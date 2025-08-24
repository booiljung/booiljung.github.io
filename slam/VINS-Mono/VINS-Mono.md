# VINS-Mono
[VINS-Mono](./index.md)




시각-관성 주행 거리계(Visual-Inertial Odometry, VIO) 기술의 등장은 각기 다른 센서가 지닌 본질적인 한계를 극복하기 위한 필연적인 결과물이다. VIO는 저렴하고 소형화된 센서 조합을 통해 고가의 장비로만 가능했던 정밀한 6자유도(Degrees-of-Freedom, DOF) 상태 추정을 가능하게 함으로써, 로보틱스와 증강현실(AR) 분야에 혁신을 가져왔다. 이러한 융합 기술의 필요성은 단일 센서가 가진 명확한 약점에서 비롯된다.1

첫째, 단안 카메라(Monocular Camera)는 풍부한 환경 정보를 제공하지만, 근본적으로 미터 단위의 실제 스케일(metric scale)을 복원할 수 없는 한계를 지닌다.3 이는 카메라가 2차원 이미지 평면에 3차원 세계를 투영하는 과정에서 깊이 정보가 소실되기 때문이다. 이로 인해 비전 단독 시스템은 객체까지의 실제 거리나 로봇의 이동 거리를 알 수 없어, 미터 단위의 상태 추정이 필수적인 실제 로봇 응용 분야에서의 활용이 제한된다.5 또한, 카메라는 조명 변화가 심하거나, 텍스처가 부족한 환경, 또는 빠른 움직임으로 인한 모션 블러(motion blur)가 발생하는 상황에서는 특징점 추적에 실패하여 위치 추정이 불가능해지는 취약점을 보인다.3

둘째, 관성 측정 장치(Inertial Measurement Unit, IMU)는 가속도계와 자이로스코프로 구성되어 로봇의 가속도와 각속도를 매우 높은 주기로 측정한다. 이 측정값을 적분하면 위치와 자세를 추정할 수 있지만, 저가형 MEMS 기반 IMU는 상당한 수준의 노이즈와 시시각각 변하는 바이어스(bias)를 포함하고 있다.7 이 오차는 적분 과정에서 누적되어 시간이 지남에 따라 위치 추정 오차가 급격하게 증가하는 드리프트(drift) 현상을 야기한다.2 따라서 IMU 단독으로는 장시간 동안 신뢰할 수 있는 위치 정보를 제공할 수 없다.

이 두 센서의 융합은 각자의 단점을 완벽하게 보완하는 상호 보완적 관계를 형성한다. IMU는 카메라가 제공할 수 없는 미터 단위의 스케일 정보를 제공하며, 동시에 롤(roll)과 피치(pitch) 각도를 관측 가능(observable)하게 만든다.3 또한, 시각 정보가 일시적으로 소실되는 구간에서도 IMU의 고주파수 측정값을 통해 움직임을 지속적으로 추정하여 시스템의 강인성을 크게 향상시킨다.3 반대로, 카메라는 외부 환경의 특징점을 관측하여 IMU의 누적 오차를 지속적으로 보정해주는 역할을 한다. 이러한 이유로 단안 카메라와 저가형 IMU의 조합은 6자유도 미터 스케일 상태 추정을 위한 최소한의 센서 구성으로 평가받으며 3, 자율 드론이나 모바일 기기 기반 증강현실과 같은 응용 분야의 핵심 기술로 자리 잡았다.3 결국 VIO 기술의 발전은 고가의 고정밀 관성 센서 없이, 저렴하고 어디서나 구할 수 있는 센서들을 지능적으로 결합하여 그 합 이상의 성능을 이끌어내는 최적의 공학적 해법이라 할 수 있다.


VIO 시스템에서 시각 정보와 관성 정보를 융합하는 방식은 크게 필터 기반(filter-based) 접근법과 최적화 기반(optimization-based) 접근법으로 나뉜다. VINS-Mono는 이 중 최적화 기반 방식을 채택하고 있으며, 각 방식의 특징을 이해하는 것은 VINS-Mono의 설계 철학을 파악하는 데 필수적이다.

필터 기반 접근법은 주로 확장 칼만 필터(Extended Kalman Filter, EKF)나 그 변형인 다중 상태 제약 칼만 필터(Multi-State Constraint Kalman Filter, MSCKF)를 사용한다.13 이 방식은 재귀적(recursive) 상태 추정 기법으로, 새로운 IMU 측정값이 들어올 때마다 시스템의 상태와 공분산을 예측(prediction)하고, 새로운 카메라 이미지가 들어오면 시각적 관측을 통해 상태를 갱신(update)한다. 필터 기반 방식의 가장 큰 장점은 계산 효율성이다. 매 시간 단계에서 이전 상태만을 기반으로 현재 상태를 추정하므로, 계산량이 일정하게 유지되어 연산 능력이 제한된 시스템에서도 실시간으로 동작할 수 있다.15 그러나 이 방식은 비선형 시스템을 선형화하는 과정에서 발생하는 오차에 취약하며, 한번 발생한 선형화 오차는 되돌릴 수 없어 정확도가 떨어질 수 있다. 또한, 시스템의 관측 가능성(observability)을 잘못 판단하여 필터가 자신의 추정치를 과신하게 되는 불일치(inconsistency) 문제가 발생할 수 있다.18

반면, 최적화 기반 접근법은 일정 시간 동안의 센서 측정값들을 모아 하나의 큰 최적화 문제를 구성하고 이를 푸는 방식이다.13 VINS-Mono와 OKVIS 등이 이 방식을 사용하며, 주로 슬라이딩 윈도우(sliding window) 기법을 활용한다. 이 기법은 최근 N개의 키프레임과 관련된 모든 상태 변수(카메라 자세, 속도, IMU 바이어스, 특징점 위치 등)를 하나의 비용 함수(cost function)로 정의하고, 이 비용 함수를 최소화하는 상태 변수들을 찾는다. 비용 함수는 주로 IMU 측정값의 잔차(residual)와 시각적 특징점의 재투영 오차(reprojection error)로 구성된다.21 최적화 기반 방식은 여러 프레임에 걸친 정보를 동시에 고려하고, 반복적인 최적화를 통해 과거 상태의 선형화 지점을 다시 계산할 수 있으므로 필터 기반 방식보다 일반적으로 더 높은 정확도를 달성한다.13 하지만 슬라이딩 윈도우 내의 모든 상태와 측정값을 처리해야 하므로 계산 복잡도가 더 높다.22

이러한 융합 방식은 센서 데이터를 처리하는 수준에 따라 강결합(tightly-coupled)과 약결합(loosely-coupled)으로도 구분된다. VINS-Mono와 같은 최신 고성능 시스템들은 대부분 강결합 방식을 채택한다. 이는 카메라의 픽셀 정보와 IMU의 원시 측정값을 하나의 최적화 문제 안에서 직접 융합하여, 센서 간의 상호작용을 최대한 활용하고 최고 수준의 정확도를 추구하는 방식이다.3



VINS-Mono는 단안 카메라와 IMU 센서로부터 입력받은 데이터를 처리하여 강인하고 정확한 6자유도 상태를 추정하는 완전한 시스템으로, 총 다섯 개의 주요 모듈이 순차적 혹은 병렬적으로 동작하는 파이프라인 구조를 가진다.4 각 모듈은 VIO 문제를 해결하기 위한 특정 과업을 담당하며 유기적으로 연결되어 있다.

1. **측정 전처리 (Measurement Preprocessing):** 이 모듈은 시스템의 가장 앞단에서 원시 센서 데이터를 처리한다. 새로운 카메라 이미지가 입력되면 특징점을 추출하고(feature detection), 이전 프레임과의 추적(feature tracking)을 수행한다. 동시에, 연속된 두 이미지 프레임 사이에서 수신된 모든 IMU 데이터를 사전적분(pre-integration)하여 단일 상대 모션 제약 조건으로 압축한다.4
2. **초기화 (Estimator Initialization):** VIO 시스템은 고도의 비선형성을 가지기 때문에, 안정적인 동작을 위해서는 양질의 초기값이 필수적이다. 이 모듈은 강인한 초기화 절차를 통해 시스템을 부트스트랩한다. 먼저, 시각 정보만을 사용하여 스케일이 부정확한 3차원 구조와 카메라 자세를 복원하는 Vision-Only SfM (Structure from Motion) 단계를 거친다. 그 후, 이 시각적 구조와 IMU 사전적분 값을 정렬(alignment)하여 실제 미터 스케일, 속도, 중력 벡터, IMU 바이어스 등의 초기값을 추정한다.4
3. **강결합 단안 VIO (Tightly-Coupled Monocular VIO):** 시스템의 핵심부로, 지역적으로 매우 정확한 주행 거리계를 제공한다. 이 모듈은 슬라이딩 윈도우 기반의 비선형 최적화 기법을 사용한다. 윈도우 내의 키프레임들과 관련된 IMU 사전적분 측정값 및 시각 특징점 관측값을 강결합하여, 카메라 자세, 속도, IMU 바이어스, 특징점의 3차원 위치 등을 공동으로 최적화한다.3
4. **재지역화 (Relocalization / Loop Closure):** 장시간 주행 시 누적되는 드리프트 오차를 해결하기 위한 모듈이다. DBoW2와 같은 장소 인식(place recognition) 기술을 사용하여 과거에 방문했던 장소를 다시 인식한다. 루프가 감지되면, 현재 프레임과 과거 키프레임 간의 시각적 제약 조건을 VIO 최적화 문제에 직접 통합하여 누적된 오차를 즉시 보정한다.3
5. **전역 포즈 그래프 최적화 (Global Pose Graph Optimization):** 재지역화가 성공적으로 수행된 후, 전체 궤적의 일관성을 보장하기 위해 전역 최적화를 수행한다. VINS-Mono는 IMU 덕분에 롤과 피치 각도의 드리프트가 없다는 점에 착안하여, 위치(x, y, z)와 요(yaw) 각도에 대한 4자유도 포즈 그래프 최적화만을 수행하여 계산 효율성을 높인다.3

이러한 모듈식 구조는 VINS-Mono의 강점이자 이후 연구의 청사진이 되었다. 각 모듈이 VIO의 특정 문제를 해결하도록 명확히 분리되어 있고 모듈 간 인터페이스가 잘 정의되어 있어, 후속 연구자들이 전체 시스템을 재설계할 필요 없이 특정 모듈(예: 프론트엔드)만을 개선하거나 교체하는 것을 용이하게 했다. 예를 들어, SuperVINS와 같은 최신 연구는 전통적인 시각 프론트엔드와 루프 클로저를 딥러닝 기반 모듈로 대체하면서도, VINS-Mono로부터 계승된 핵심 최적화 백엔드는 그대로 유지하는 전략을 취한다.6 이는 VINS-Mono의 핵심 기여가 단일 알고리즘을 넘어, 최적화 기반 VIO를 위한 강인하고 확장 가능한 *프레임워크*를 제시했다는 점을 시사한다.


VINS-Mono가 VIO 분야의 벤치마크 알고리즘으로 자리매김할 수 있었던 배경에는 독창적인 몇 가지 핵심 기여가 있다.3

- **강인한 초기화:** 단안 VIO의 가장 큰 난제 중 하나는 알 수 없는 초기 상태에서 시스템을 시작하는 것이다. VINS-Mono는 사용자의 특별한 움직임 없이, 움직이는 상태에서 즉시(on-the-fly) 시스템을 안정적으로 부트스트랩할 수 있는 강인한 초기화 절차를 제안했다.3
- **강결합 최적화 기반 VIO:** 슬라이딩 윈도우 최적화 기법을 통해 IMU 상태(자세, 속도, 바이어스), 카메라-IMU 외부 파라미터(extrinsic parameters), 그리고 특징점의 깊이 정보를 하나의 비용 함수 내에서 공동으로 추정한다. 이를 통해 센서 간의 상호작용을 극대화하여 높은 정확도를 달성하며, 특히 카메라와 IMU 간의 변환 관계 및 시간 오프셋을 온라인으로 보정하는 기능을 포함한다.3
- **재지역화 및 전역 일관성:** 루프 클로저를 통해 누적 오차를 제거하는 기능은 SLAM 시스템의 핵심이다. VINS-Mono는 루프 제약 조건을 VIO 최적화에 직접 통합하는 강결합 재지역화 방식을 제안하여 오차를 부드럽게 보정한다. 더 나아가, IMU 측정으로 인해 롤과 피치 각도의 드리프트가 없다는 관측 가능성 분석에 기반하여, 4자유도 전역 포즈 그래프 최적화라는 독창적인 해법을 제시하여 전역 일관성을 효율적으로 확보한다.3
- **완전성과 범용성:** VINS-Mono는 단순한 알고리즘 제안을 넘어, 실제 응용을 위한 완전한 시스템으로 설계되었다. 이는 소형 무인항공기(MAV)에서의 자율 비행, 스마트폰(iOS) 기반의 증강현실 시연 등 다양한 플랫폼과 시나리오에서의 성공적인 적용을 통해 입증되었다.3 또한, PC와 모바일 기기용 코드를 모두 오픈소스로 공개하여 학계와 산업계의 폭넓은 활용을 이끌었다.3




시각 프론트엔드는 연속적인 이미지 스트림으로부터 상태 추정에 필요한 핵심적인 시각 정보를 추출하고 정제하는 역할을 담당한다. 이 과정은 특징점 검출, 추적, 이상치 제거, 그리고 키프레임 선택의 단계로 구성된다.

- **특징점 검출:** VINS-Mono는 OpenCV 라이브러리의 `GoodFeaturesToTrack` 함수, 즉 Shi-Tomasi 코너 검출기를 사용하여 이미지에서 특징점을 찾는다.4 여기서 중요한 전략은 특징점들이 이미지의 특정 영역에 집중되지 않도록 최소 픽셀 간격을 설정하여 균일한 분포를 유도하는 것이다.30 이는 후속 최적화 단계에서 더 나은 기하학적 제약 조건을 제공하여 자세 추정의 정확도를 높이는 데 기여한다. 시스템은 각 이미지에서 100개에서 300개 사이의 특징점을 유지하려고 시도한다.4
- **특징점 추적:** 검출된 특징점들은 Kanade-Lucas-Tomasi (KLT) 희소 광학 흐름(sparse optical flow) 알고리즘을 통해 연속된 프레임 간에 추적된다.4 KLT는 이미지의 밝기 항상성(brightness constancy)을 가정하고, 특징점 주변의 작은 윈도우 내에서 움직임을 추정한다.
- **이상치 제거:** 잘못된 추적이나 동적 객체에 의해 발생할 수 있는 이상치(outlier) 특징점들은 시스템의 정확도를 심각하게 저하시킬 수 있다. VINS-Mono는 RANSAC(RANdom SAmple Consensus) 알고리즘과 기본 행렬(fundamental matrix) 테스트를 결합하여 이러한 이상치들을 통계적으로 제거한다.4
- **키프레임 선택:** 모든 이미지 프레임을 최적화에 사용하는 것은 계산적으로 비효율적이므로, VINS-Mono는 정보량이 풍부한 특정 프레임만을 '키프레임'으로 선택하여 관리한다. 키프레임 선택 기준은 두 가지이다.
  1. **시차(Parallax):** 현재 프레임과 마지막 키프레임 간에 추적된 특징점들의 평균 시차(픽셀 단위 이동 거리)가 특정 임계값을 초과하면 새로운 키프레임으로 선택된다. 이는 3차원 특징점의 위치를 삼각 측량(triangulation)하기에 충분한 기준선(baseline)이 확보되었음을 의미한다.4 여기서 핵심적인 기법은 순수 회전(rotation-only) 동작 중에 불필요한 키프레임이 생성되는 것을 방지하는 것이다. 순수 회전 시에는 시차가 발생하지만, 삼각 측량이 불가능하여 깊이 정보를 얻을 수 없다.30 VINS-Mono는 IMU의 단기 적분 결과를 사용하여 회전으로 인한 픽셀 이동을 보상하고, 순수한 병진 운동(translation)에 의한 시차만을 계산한다.4 이 IMU 보상 시차 계산은 시스템의 강인성을 직접적으로 향상시키는 중요한 메커니즘이다. 기하학적으로 유의미한 정보를 제공하는 프레임만을 선택함으로써, 후속 초기화 및 최적화 단계에서 발생할 수 있는 잘못된 문제 설정(ill-posed problem)을 근본적으로 방지한다.
  2. **추적 품질:** 추적 중인 특징점의 수가 특정 임계값 이하로 떨어지면, 이는 큰 장면 변화나 추적 실패를 의미하므로 해당 프레임을 새로운 키프레임으로 선택하여 새로운 특징점을 검출하도록 한다.4


IMU 사전적분(pre-integration)은 VINS-Mono와 같은 최적화 기반 VIO 시스템의 계산 효율성을 보장하는 핵심 기술이다.

- **문제점:** IMU는 카메라보다 훨씬 높은 주파수(예: 100Hz 이상)로 데이터를 출력한다.26 최적화 기반 VIO에서는 슬라이딩 윈도우 내의 과거 상태(예: 키프레임의 자세)가 반복적으로 수정된다. 만약 IMU 측정값을 직접 적분하여 상태를 전파(propagation)한다면, 과거 상태가 조금이라도 변경될 때마다 그에 의존하는 모든 IMU 측정값을 처음부터 다시 적분해야 한다. 이는 엄청난 계산량을 유발하여 실시간 처리를 불가능하게 만든다.30

- **해결책: 사전적분:** IMU 사전적분은 연속된 두 키프레임 $k$와 $k+1$ 사이의 모든 IMU 측정값을 하나의 상대적인 움직임 제약 조건으로 미리 통합해두는 기법이다. 핵심 아이디어는 이 상대적인 움직임을 첫 번째 키프레임 $k$의 몸체 좌표계(body frame) 기준으로 표현하는 것이다. 이렇게 하면, 사전적분된 값은 키프레임 $k$의 전역 자세(global pose)와 무관해지므로, 나중에 최적화 과정에서 키프레임 $k$의 자세가 변경되더라도 사전적분 값을 재계산할 필요가 없어진다.26

- 수학적 공식화: 키프레임 $b_k$와 $b_{k+1}$ 사이의 시간 간격 $[t_k, t_{k+1}]$에 대한 IMU 상태 전파 방정식은 월드 좌표계($w$)에서 다음과 같이 표현된다.
  $$
  p_{wb_{k+1}} = p_{wb_k} + v^w_{b_k} \Delta t_k + \iint_{t \in [t_k, t_{k+1}]} (R_{wt}(t)(\hat{a}t - b{a_t}) - g^w) dt^2
  $$

  $$
  v^w_{b_{k+1}} = v^w_{b_k} + \int_{t \in [t_k, t_{k+1}]} (R_{wt}(t)(\hat{a}t - b{a_t}) - g^w) dt
  $$

  $$
   q_{wb_{k+1}} = q_{wb_k} \otimes \int_{t \in [t_k, t_{k+1}]} \frac{1}{2} \Omega(\hat{\omega}t - b{g_t}) q_{b_k t}(t) dt
  $$

  여기서 $p,v,q$는 각각 위치, 속도, 자세(쿼터니언)를, $R$은 회전 행렬을, $\hat{a}, \hat{\omega}$는 IMU의 측정값을, $b_a$,$b_g$는 가속도계와 자이로스코프의 바이어스를, $g^w$는 중력 벡터를 나타낸다. 이 식들은 $p_{wb_k}, v^w_{b_k}, q_{wb_k}$에 의존하므로 재계산이 필요하다.

  전적분은 이 식들을 bk 좌표계 기준으로 재정의하여 다음과 같은 상대적인 움직임 항들 $\Delta p_{b_k b_{k+1}}, \Delta v_{b_k b_{k+1}}, \Delta q_{b_k b_{k+1}}$을 구한다.
  $$
  \Delta p_{b{_k}b{_{k+1}}} = \iint_{t\in[t_k,t_{k-1}]} R_{b_kt}(t)(\hat{a}_t - b_{a_t})dt^2
  $$

  $$
  \Delta v_{b{_k}b{_{k+1}}} = \int_{t\in[t_k,t_{k-1}]} R_{b_kt}(t)(\hat{a}_t - b_{a_t})dt
  $$

  이 사전적분 항들은 오직 바이어스 ba,bg에만 의존한다.26

- **IMU 바이어스 처리:** 최적화 과정에서 IMU 바이어스 추정치 또한 계속 갱신된다. 바이어스가 크게 변하면 사전적분 값을 재계산해야 하지만, VINS-Mono는 1차 테일러 전개(first-order Taylor expansion)를 이용하여 바이어스의 작은 변화에 대한 사전적분 값의 변화를 근사적으로 계산한다. 이를 통해 바이어스가 크게 변하지 않는 한, 전체 재적분을 피하고 계산 효율성을 유지할 수 있다.3 또한, 사전적분 과정에서 발생하는 불확실성은 공분산(covariance)으로 전파되어 최적화 시 정보 행렬(information matrix)로 사용된다.32



단안 VIO 시스템의 초기화는 매우 어려운 문제이다. 시스템이 고도의 비선형성을 띠고 있어, 상태 변수들에 대한 좋은 초기 추정값 없이는 최적화 해가 지역 최솟값(local minima)에 빠지거나 발산하기 쉽다.3 특히 단안 카메라는 스케일 정보를 직접 얻을 수 없으며, 이 스케일은 충분한 가속 운동이 있어야만 관측 가능해지기 때문에 시스템이 정지 상태가 아닌 움직이는 상태에서 초기화가 이루어져야 한다.3 VINS-Mono는 이러한 난제들을 해결하기 위해 체계적인 2단계 초기화 절차를 제안한다.


초기화의 첫 단계는 IMU 정보를 배제하고 오직 시각 정보만을 사용하여 환경의 기하학적 구조와 카메라의 상대적인 자세를 복원하는 것이다. 이 단계의 결과물은 실제 스케일을 알 수 없는, 즉 'up-to-scale' 상태의 맵이다.

- **상대 자세 복원:** 슬라이딩 윈도우 내의 키프레임들 중에서 충분한 시차와 특징점 매칭을 보이는 두 프레임을 선택한다. 이 두 프레임 간의 상대적인 회전(Rotation)과 스케일이 모호한 이동(Translation)은 5점 알고리즘(Five-point algorithm)을 통해 계산된다.3
- **삼각 측량 및 PnP:** 두 프레임 간의 상대 자세가 구해지면, 공통으로 관측된 모든 특징점들을 삼각 측량하여 3차원 위치를 추정한다. 이렇게 복원된 3D-2D 대응점을 기반으로, 윈도우 내의 다른 모든 프레임들의 자세는 PnP(Perspective-n-Point) 알고리즘을 통해 계산된다.3
- **전역 번들 조정 (Global Bundle Adjustment):** 윈도우 내 모든 프레임의 자세와 3차원 특징점들의 위치가 초기화되면, 모든 특징점 관측에 대한 재투영 오차를 최소화하는 전역 번들 조정을 수행한다. 이 과정을 통해 시각 정보만을 이용한 최적의 기하학적 구조를 얻게 되며, 이때 첫 번째 카메라 프레임($c_0$)이 기준 좌표계가 된다.3


두 번째 단계는 1단계에서 얻은 스케일이 모호한 시각적 구조와, IMU 사전적분을 통해 얻은 미터 단위의 움직임 정보를 정렬(align)하는 과정이다. 이 과정을 통해 자이로스코프 바이어스($b_g$), 각 프레임에서의 속도($v_b$), 시각 기준 좌표계에서의 중력 벡터($g^{c_0}$), 그리고 가장 중요한 미터 스케일($s$)을 추정한다.3

- **자이로스코프 바이어스 보정:** SfM을 통해 얻은 연속된 프레임 간의 상대 회전과 IMU 사전적분을 통해 얻은 상대 회전을 비교한다. 이론적으로 두 회전은 일치해야 하므로, 그 차이를 최소화하는 자이로스코프 바이어스 값을 선형 방정식을 풀어 추정한다. 이 추정된 바이어스를 사용하여 모든 IMU 사전적분 항을 다시 계산(re-propagate)한다.3
- **속도, 중력, 스케일 추정:** 자이로스코프 바이어스가 보정되면, IMU 사전적분으로 계산된 위치 및 속도 변화량과 SfM으로 계산된 카메라 움직임 사이의 관계식을 세울 수 있다. 이 관계식은 미지수인 속도, 중력, 스케일에 대해 선형 시스템을 구성하며, 최소 제곱법(least squares method)을 통해 이 변수들의 최적해를 구한다.3
- **중력 벡터 정제:** 중력 가속도의 크기는 약 $9.8 m/s^2$로 알려져 있다. 이 물리적 제약 조건을 이용하여 앞서 추정한 중력 벡터를 정제한다. 이는 중력 벡터의 자유도를 3에서 2(방향)로 줄여 더 정확한 추정을 가능하게 한다. 정제된 중력 벡터의 방향을 월드 좌표계의 $z$축과 일치시킴으로써, 시각 기준 좌표계($c_0$)에서 월드 좌표계($w$)로의 회전 변환을 최종적으로 결정한다. 이로써 모든 상태 변수들을 월드 좌표계로 변환하고 초기화를 완료한다.3

이러한 초기화 과정의 핵심은 복잡한 비선형 문제를 순차적인 선형 부분 문제들로 분해하여 푸는 데 있다. 전체 VIO 문제를 한 번에 풀려고 시도하는 대신, 먼저 상대적인 기하 구조를 복원하고(SfM), 그 다음 회전에 가장 큰 영향을 미치는 자이로 바이어스를 해결하며, 마지막으로 나머지 변수들을 선형적으로 추정하는 단계적 접근 방식을 취한다. 이 전략적인 분해 덕분에 VINS-Mono는 특별한 초기 동작 없이도 강인하게 초기화를 수행할 수 있다.



VINS-Mono의 핵심인 VIO 모듈은 슬라이딩 윈도우 내의 모든 관련 정보를 통합하여 최적의 상태를 추정한다. 이를 위해 최적화할 상태 변수 벡터 $X$를 다음과 같이 정의한다.
$$
X=[x_0,x_1,\dots,x_n,x_c^b,λ_0,λ_1,\dots,λ_m]
$$
여기서 각 항목은 다음과 같다.

- $x_k=[p_{wb_k},v_{wb_k},q_{wb_k},b_a,b_g]$: 윈도우 내 $k$번째 키프레임에서의 IMU 상태. 월드 좌표계 기준 위치($p$), 속도($v$), 자세($q$)와 가속도계 및 자이로스코프 바이어스($b_a$, $b_g$)를 포함한다.

- $x_c^b=[p_c^b,q_c^b]$: 카메라와 IMU 간의 외부 파라미터(extrinsic parameters). 상대 위치와 자세를 포함하며, 온라인으로 보정될 수 있다.3
- $λ_l$: $l$번째 특징점의 깊이 정보. 첫 관측 시점에서의 역깊이(inverse depth)로 파라미터화되어 최적화의 수렴성을 높인다.35


VIO는 모든 가용한 센서 측정값과 사전 정보로부터 발생하는 잔차(residual)의 합을 최소화하는 비선형 최소 제곱법 문제로 공식화된다. 전체 비용 함수는 다음과 같이 세 가지 주요 요소로 구성된다.34
$$
\min_{X}
\left\{ | r_p - H_p X |^2 + \sum_{k \in B} | r_B(\hat{z}{b{k+1}}^{b_k}, X) |^2_{P_{b_{k+1}}^{b_k}} + \sum_{(l,j) \in C} \rho \left( | r_C(\hat{z}l^{c_j}, X) |^2{P_l^{c_j}} \right)
\right\}
$$

- **사전 정보 항 (Prior Factor):** $\|r_p−H_pX\|^2$. 이 항은 슬라이딩 윈도우에서 마지널화(marginalization)되어 제거된 과거 상태들의 정보를 압축하여 담고 있다. 이는 과거의 모든 정보를 버리지 않고 현재 윈도우 내 상태 변수들에 대한 확률적 제약 조건으로 작용하게 하여, 정보의 손실을 최소화하고 추정의 일관성을 유지한다.35
- **IMU 측정 잔차 항:** $\sum_{k∈B}∥r_B(\hat{z}_{b_{k+1}}^{b_k},X)∥_{P_{b_{k+1}}^{b_k}} ^2$. 이 항은 연속된 두 키프레임 $k$와 $k+1$ 사이의 IMU 상태(위치, 속도, 자세) 변화량이 IMU 사전적분 모델로부터 예측된 값과 얼마나 차이가 나는지를 측정한다. 잔차는 사전적분 과정에서 계산된 공분산의 역행렬($P^{-1}$)로 가중되어, 불확실성이 낮은 측정값에 더 큰 가중치를 부여한다.35
- **시각 측정 잔차 항 (재투영 오차):** $\sum_{(l,j)∈C}ρ(∥r_C(\hat{z}_l^{cj},X)∥_{P_l^{c_j}}^2)$. 이 항은 $j$번째 키프레임에서 관측된 $l$번째 특징점의 실제 픽셀 좌표($\hat{z}_l^{c_j}$)와, 현재 추정된 3차원 특징점 위치와 카메라 자세를 이용해 계산한 예상 픽셀 좌표 사이의 오차를 측정한다. 이 오차를 재투영 오차라고 한다. $\rho(\cdot)$는 강인한 비용 함수인 후버 손실(Huber loss)을 나타내며, 이상치(outlier) 관측값이 최적화에 미치는 과도한 영향을 줄이는 역할을 한다.34


실시간 성능을 유지하기 위해 슬라이딩 윈도우의 크기는 일정하게 유지되어야 한다. 새로운 키프레임이 추가되면 가장 오래된 키프레임은 윈도우에서 제거되어야 한다.

- **마지널화의 필요성:** 오래된 키프레임을 단순히 버리면 그 프레임과 연결된 모든 측정 정보가 소실되어 시스템의 정확성과 일관성이 저하된다. 마지널화는 이 정보를 버리지 않고, 제거될 상태 변수와 관련된 제약 조건들을 현재 윈도우에 남아있는 변수들에 대한 사전 정보로 변환하는 확률적 기법이다.
- **구현 메커니즘:** VINS-Mono는 이 마지널화 과정을 효율적으로 수행하기 위해 슈어 보수(Schur complement)를 사용한다. 최적화 문제의 각 반복 단계에서 생성되는 거대한 선형 시스템(Hessian 행렬)에서, 슈어 보수를 이용하면 제거하고자 하는 상태 변수들을 해석적으로 소거하고 그 효과를 나머지 변수들에 대한 정보 행렬로 압축할 수 있다. 이를 통해 계산 복잡도를 크게 줄이면서도 정보 손실을 최소화한다.13

슬라이딩 윈도우와 마지널화의 조합은 VIO 설계에 있어 중요한 철학적 타협점을 보여준다. 이는 전체 궤적에 대해 최적화를 수행하는 완전한 번들 조정(full bundle adjustment)의 높은 정확도와, 재귀적 필터 방식의 실시간 제약 조건을 모두 만족시키기 위한 하이브리드 해법이다. 과거의 정보를 확률적 사전 정보로 요약하면서 최근 정보에 대해서는 반복적인 재선형화를 통해 높은 정확도를 유지하는 이 구조는, 정확성과 실시간성이라는 상충하는 목표 사이의 균형을 맞추려는 시도의 직접적인 결과물이다.



장시간 운용 시 발생하는 드리프트는 VIO 시스템의 고질적인 문제이다. VINS-Mono는 루프 클로저(loop closure)를 통해 이 문제를 해결하고 전역적인 궤적 일관성을 확보한다.

- **장소 인식:** VINS-Mono는 시각적 단어 가방(Bag-of-Visual-Words) 라이브러리인 DBoW2를 사용하여 효율적으로 장소를 인식한다. 시스템이 과거에 방문했던 장소와 유사한 장면을 다시 보게 되면, DBoW2는 루프 후보 프레임을 반환한다.3 루프 검출의 재현율(recall rate)을 높이기 위해, VIO 프론트엔드에서 사용하는 특징점 외에 추가적으로 BRIEF 디스크립터를 가진 특징점들을 추출하여 데이터베이스 검색에 활용한다.3
- **기하학적 검증:** 루프 후보가 식별되면, 현재 프레임과 후보 프레임 간의 특징점 대응 관계를 설정하고, 기하학적 검증을 통해 잘못된 매칭을 걸러낸다. 이 과정은 2단계 RANSAC으로 구성된다: (1) 2D-2D 기본 행렬 테스트, (2) 3D-2D PnP 테스트.3
- **강결합 융합:** 성공적인 루프가 검출되면, VINS-Mono는 과거 키프레임과의 특징점 대응 관계를 현재 슬라이딩 윈도우 최적화 문제에 *추가적인 시각 제약 조건*으로 직접 통합한다. 이때 과거 루프 클로저 프레임의 자세는 고정된 상수로 취급되며, 현재 윈도우 내의 상태 변수들이 지역적인 VIO 측정값과 전역적인 루프 제약 조건 모두를 만족하도록 최적화된다. 이 강결합 방식은 궤적 오차를 갑작스럽게 수정하는 대신, 부드럽고 즉각적으로 보정하여 시스템의 안정성을 높인다.3


- **4자유도 최적화의 근거:** 이 기법은 VINS-Mono의 핵심적인 통찰을 보여준다. IMU 가속도계는 지속적으로 중력 벡터를 측정하므로, 시스템은 전역 좌표계에 대한 롤(roll)과 피치(pitch) 각도를 항상 관측할 수 있다. 이는 두 축 방향으로는 드리프트가 누적되지 않음을 의미한다. 따라서 장기적인 드리프트는 전역적으로 관측 불가능한 4개의 자유도, 즉 3차원 위치$(x, y, z)$와 중력 벡터를 축으로 하는 회전(yaw)에 대해서만 발생한다.3

- **포즈 그래프 구조:** 시스템은 마지널화된 모든 키프레임을 노드(node)로 하는 포즈 그래프를 유지한다. 노드들은 두 종류의 엣지(edge)로 연결된다.

  1. **순차 엣지 (Sequential Edges):** 연속된 키프레임들을 연결하며, VIO에서 계산된 상대적인 4자유도 변환 정보를 담는다.
  2. **루프 클로저 엣지 (Loop Closure Edges):** 루프가 검출된 키프레임과 과거 키프레임을 연결하며, 재지역화 과정에서 계산된 4자유도 변환 정보를 담는다.3

- **최적화:** 시스템은 이 그래프의 모든 엣지에서 발생하는 오차를 최소화하여 모든 키프레임의 자세를 전역적으로 일관성 있게 조정한다. 이를 통해 전체 궤적에서 누적된 드리프트가 효과적으로 제거된다.3 이 4자유도 최적화 방식은 시각-

  *관성* 시스템의 물리적 특성을 깊이 이해하고 이를 활용하여 6자유도 최적화 문제를 더 효율적이고 제약이 강한 4자유도 문제로 단순화한 우아한 해법이다. 이는 시스템의 관측 가능성(observability)을 분석한 직접적인 결과물이다.


VINS-Mono는 생성된 맵(포즈 그래프와 특징점)을 저장하고, 추후에 다시 로드하여 기존 환경에서 재지역화를 수행하는 기능을 지원한다.9 이를 통해 맵 재사용이 가능하며, 현재 세션에서 생성된 맵을 이전에 로드한 맵과 전역 포즈 그래프 최적화를 통해 병합하는 기능도 포함하고 있다.9




어떠한 VIO 시스템의 이론적 정확도도 결국 센서 보정(calibration)의 품질에 의해 제한된다. VIO 시스템의 성공적인 운용은 세 가지 핵심적인 보정 요소에 크게 의존한다.

- **카메라 내부 파라미터 (Intrinsic Calibration):** 렌즈 왜곡 계수, 초점 거리, 주점 등 카메라 고유의 파라미터를 정확하게 측정하는 과정이다. 부정확한 내부 파라미터는 특징점의 3차원 위치를 2차원 이미지 평면에 투영하는 모델 자체를 왜곡시켜, 높은 재투영 오차를 유발하고 시스템 전체의 정확도를 저하시킨다.7 안정적인 성능을 위해서는 재투영 오차가 0.5 픽셀 미만이 되도록 정밀하게 보정하는 것이 권장된다.27
- **카메라-IMU 외부 파라미터 (Extrinsic Calibration):** VIO 시스템에서 가장 중요한 보정 요소로, 공간적(spatial) 및 시간적(temporal) 파라미터로 나뉜다.
  - **공간적 보정 (회전 및 이동):** 카메라 좌표계와 IMU 좌표계 사이의 강체 변환(rigid body transformation)을 의미한다. 이 변환이 정확하지 않으면, IMU가 측정한 움직임과 카메라가 관측한 움직임이 서로 어긋나게 되어 시스템이 빠르게 발산하는 원인이 된다.37 VINS-Mono와 VINS-Fusion은 이 외부 파라미터를 온라인으로 추정하고 보정하는 기능을 지원하여 편의성과 강인성을 높였다.3
  - **시간적 보정 (시간 오프셋):** 카메라 이미지의 타임스탬프와 IMU 측정값의 타임스탬프 사이에 존재하는 미세한 시간 차이(수 밀리초 단위)를 의미한다. 이 시간 오프셋은 특히 급격한 움직임이 있을 때 측정값 간의 불일치를 야기하여 심각한 드리프트의 원인이 된다.37 VINS 계열 시스템은 이 시간 오프셋 또한 온라인으로 추정하는 기능을 제공한다.27
- **IMU 내부 파라미터 (노이즈 모델):** IMU의 가속도계와 자이로스코프가 갖는 백색 잡음(white noise)과 바이어스 랜덤 워크(bias random walk) 특성을 정확하게 모델링하는 것이다. 이 노이즈 파라미터들은 최적화 과정에서 IMU 측정 잔차의 가중치(정보 행렬)를 결정하는 데 사용된다. 부정확한 노이즈 모델은 최적화기가 IMU 측정값을 과도하게 신뢰하거나 불신하게 만들어 전체 시스템의 성능을 저하시킨다.7


- **롤링 셔터 vs. 글로벌 셔터 카메라:** 저가형 모바일 기기에 널리 사용되는 롤링 셔터 카메라는 이미지의 각 픽셀 행을 순차적으로 노출시킨다. 이는 대부분의 VIO 알고리즘이 가정하는 '이미지는 한순간에 촬영된다'는 전역 셔터 모델을 위반하며, 특히 빠른 회전 시 심각한 이미지 왜곡을 유발하여 추적 성능을 저하시킨다.27 VINS-Mono는 롤링 셔터 지원 옵션을 제공하지만, 최상의 성능은 글로벌 셔터 카메라에서 발휘된다.27
- **IMU 품질 및 동기화:** 노이즈가 심하거나 바이어스가 불안정한 저품질 IMU는 오차의 주요 원인이다. 더 중요한 것은 카메라 셔터와 IMU 측정 간의 하드웨어 동기화 부재이다. 소프트웨어 기반 타임스탬프는 지터(jitter)와 가변적인 지연을 유발할 수 있으며, 이는 온라인 시간 보정 기능으로도 완벽하게 처리하기 어려워 시스템 발산의 주된 원인이 된다.44 따라서 강인한 성능을 위해서는 하드웨어 동기화가 필수적이다.45
- **환경적 제약:** VINS-Mono의 전통적인 프론트엔드는 다음과 같은 환경에서 실패할 가능성이 높다.
  - **저텍스처/무특징 환경:** 충분한 수의 특징점을 추적할 수 없는 환경에서는 시스템이 실패한다.3
  - **동적 환경:** 움직이는 물체는 정적 세계 가정을 위반하며, RANSAC에 의해 제거되지 않은 동적 특징점들은 자세 추정을 오염시킨다.34
  - **급격한 움직임 및 조명 변화:** 빠른 움직임은 모션 블러를, 급격한 조명 변화는 KLT 추적의 밝기 항상성 가정을 위반하여 추적 실패를 유발한다.3

VIO 시스템의 이론적 우아함과 실제 하드웨어 구현의 현실 사이에는 상당한 간극이 존재한다. 실제 환경에서의 VIO 시스템 성공은 종종 최적화 백엔드의 참신함보다는 센서 프론트엔드의 세심한 엔지니어링, 특히 보정과 동기화에 더 크게 좌우된다. VINS-Mono의 강결합 구조는 한 센서의 오차(예: 부정확한 시간의 IMU 측정)가 다른 센서의 제약 조건(예: 시각적 재투영)을 직접적으로 오염시켜 오차의 피드백 루프를 생성할 수 있음을 의미한다. 따라서 "좋은 데이터 입력이 좋은 결과를 낳는다"는 원칙이 VIO 시스템의 실용적인 성공에 결정적인 영향을 미친다.



VINS-Mono와 ORB-SLAM3는 가장 널리 알려진 오픈소스 최적화 기반 SLAM 시스템으로, 서로 다른 설계 철학을 가지고 있다.

- **프론트엔드:** VINS-Mono는 KLT 광학 흐름을 이용해 Shi-Tomasi 코너를 추적하는 반면, ORB-SLAM3는 회전과 스케일 변화에 강인한 ORB 특징점을 사용한다. ORB 특징점은 더 강인하지만 추출 및 매칭에 더 많은 계산 비용이 소모된다.17
- **IMU 통합 및 초기화:** 두 시스템 모두 사전적분을 이용한 강결합 최적화 방식을 사용한다. 그러나 ORB-SLAM3는 최대 사후 확률(Maximum a Posteriori, MAP) 추정에 기반한 새로운 초기화 기법을 도입하여 더 높은 강인성을 보고한다.52
- **맵핑 및 재지역화:** 이것이 두 시스템의 가장 큰 차이점이다. VINS-Mono는 본질적으로 드리프트 보정을 위한 루프 클로저 기능이 탑재된 VIO 시스템이다. 즉, 주된 목적은 정확한 실시간 상태 추정(주행 거리계)이며, 맵은 추정기를 보조하는 역할을 한다. 반면, ORB-SLAM3는 지속적이고 재사용 가능한 다중 세션 맵("Atlas" 시스템) 구축에 초점을 맞춘 완전한 SLAM 시스템이다. 이는 ORB-SLAM3가 장기적으로 동일한 환경에서 자율 주행하는 데 더 적합하게 만든다.52
- **성능:** 다수의 벤치마크에서 ORB-SLAM3가 더 높은 정확도와 강인성을 보이며 최신 기술(state-of-the-art)로 평가받지만, 이는 더 높은 계산 비용을 수반한다.23


이 비교는 VIO의 두 가지 주요 패러다임을 대표하는 VINS-Mono와 MSCKF를 통해 이루어진다.

- **핵심 방법론:** VINS-Mono의 슬라이딩 윈도우 내 반복적인 배치 최적화 방식과 MSCKF의 재귀적인 단일 패스 칼만 필터 추정 방식을 대조한다.13
- **정확도 vs. 효율성:** 벤치마크 결과는 일관되게 VINS-Mono와 같은 최적화 기반 방식이 더 높은 정확도를 달성하고, MSCKF와 같은 필터 기반 방식이 계산적으로 더 효율적(낮은 CPU 및 메모리 사용량)임을 보여준다.15 이로 인해 MSCKF는 VINS-Mono가 너무 무거울 수 있는 고도로 자원이 제약된 플랫폼에 더 적합하다.15
- **일관성 및 관측 가능성:** EKF 기반 방법인 MSCKF는 선형화 오차로 인해 필터가 자신의 추정치를 과신하게 되는 불일치(inconsistency) 문제에 빠지기 쉽다. 이는 선형화된 시스템 모델에서 실제로는 관측 불가능한 상태(예: yaw)가 관측 가능한 것처럼 잘못 나타나기 때문에 발생한다.19 매 반복마다 재선형화를 수행하는 최적화 기반 방식은 이러한 문제에 덜 취약하다.


OKVIS 역시 키프레임 기반의 최적화 VIO 시스템으로 VINS-Mono와 유사한 접근 방식을 취한다. 두 시스템은 강결합, 키프레임 기반 최적화라는 점에서 공통점을 가지지만, 구현상의 차이로 인해 성능이 달라진다. 벤치마크에서는 두 시스템 모두 매우 경쟁력 있는 성능을 보이며, 특정 하드웨어나 궤적에 따라 우위가 달라질 수 있다.15

| 기능                | VINS-Mono (최적화 기반)                           | ORB-SLAM3 (최적화 기반)                   | MSCKF (필터 기반)                                            |
| ------------------- | ------------------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| **핵심 방법**       | 강결합, 슬라이딩 윈도우 비선형 최적화 (번들 조정) | 강결합, 다중 맵 번들 조정                 | 다중 상태 제약을 이용한 확장 칼만 필터(EKF)                  |
| **시각 프론트엔드** | Shi-Tomasi 코너의 KLT 광학 흐름 추적              | ORB 특징점 (oriented FAST, rotated BRIEF) | 일반적인 특징점 추적                                         |
| **IMU 통합**        | 최적화 내 사전적분 인수(factor)                   | 최적화 내 사전적분 인수                   | 상태 예측을 위한 IMU 전파                                    |
| **상태 추정**       | 최대 사후 확률(MAP)                               | 최대 사후 확률(MAP)                       | 재귀적 베이즈 필터링                                         |
| **맵핑**            | 희소 특징점 맵                                    | 조밀하고 재사용 가능한 다중 맵 시스템     | 영구적인 맵 없음 (주행 거리계에 집중)                        |
| **루프 클로저**     | DBoW2 + 4-DOF 포즈 그래프                         | DBoW2 + 7-DOF 포즈 그래프 (Sim(3))        | 핵심 구성 요소 아님                                          |
| **정확도**          | 높음                                              | 매우 높음 (최신 기술)                     | 중간 ~ 높음                                                  |
| **계산 비용**       | 높음                                              | 매우 높음                                 | 낮음 ~ 중간                                                  |
| **일관성**          | 일반적으로 일관됨                                 | 일반적으로 일관됨                         | 특정 수정 없이는 불일치 경향 (관측 가능성 문제)              |
| **강인성**          | IMU 덕분에 시각적 저하에 강인함                   | 진보된 재지역화 기능으로 매우 강인함      | 계산적으로 효율적이나, 초기화/선형화 오차에 덜 강인할 수 있음 |



VINS-Fusion은 VINS-Mono의 직접적인 후속작으로, 일반적인 다중 센서 상태 추정기를 목표로 설계되었다.38

- **다중 센서 지원:** VINS-Fusion의 핵심적인 확장은 다양한 센서 구성을 지원하는 능력에 있다. 단안+IMU(VINS-Mono 모드), 스테레오+IMU, 심지어 스테레오 카메라만으로도 동작할 수 있다.38 이는 시각 프론트엔드와 비용 함수 내의 시각적 인수를 일반화하여 여러 카메라로부터의 특징점을 처리할 수 있도록 구현되었다.35
- **GPS 융합:** VINS-Fusion은 GPS와 같은 전역 센서를 통합하는 방법을 보여준다. 이는 주로 전역 포즈 그래프를 통해 약결합 방식으로 이루어진다. VIO는 지역적으로 정확한 상대 자세 제약 조건을 제공하고, GPS는 전역적으로 정확하지만 저주파수인 절대 위치 제약 조건을 제공한다. 포즈 그래프 최적화는 이 두 정보를 융합하여 지역적으로는 부드럽고 전역적으로는 드리프트가 없는 궤적을 생성한다.38


- **Semantic VIO:** 전통적인 VIO는 동적 환경에 취약하다.34 이를 극복하기 위해 Semantic VIO가 등장했다. 이 기술은 딥러닝 기반의 객체 탐지나 시맨틱 분할을 사용하여 움직이는 물체 위의 특징점을 식별하고 배제함으로써 강인성을 향상시킨다.48
- **기하학적 특징 (선):** 텍스처가 부족한 인공 환경에서는 점 특징점이 희소할 수 있다. PL-VINS, MLINE-VINS와 같은 시스템은 VINS 프레임워크를 확장하여 선 특징점(line features)을 통합한다. 선은 추가적인 구조적 제약 조건을 제공하여 이러한 도전적인 환경에서 성능을 향상시킨다.21


최근에는 전통적인 컴퓨터 비전 구성 요소를 딥러닝 모델로 대체하는 경향이 뚜렷하며, SuperVINS가 그 대표적인 예이다.6

- **학습 기반 프론트엔드:** SuperVINS는 VINS-Fusion의 Shi-Tomasi/KLT 프론트엔드를 SuperPoint(특징점 검출)와 LightGlue(매칭)와 같은 학습된 지역 특징점으로 대체한다. 이러한 모델들은 전통적인 방법이 실패하는 극심한 조명 변화나 모션 블러 상황에서 더 강인한 성능을 보인다.6
- **학습 기반 루프 클로저:** 또한, BRIEF 기반의 DBoW2를 딥러닝 디스크립터로 학습된 단어 가방 모델로 대체하여 장소 인식의 정확도와 강인성을 향상시킨다.6


- **엣지에서의 VIO:** VIO 및 기타 AI 연산을 자원이 제한된 임베디드 장치에서 직접 처리하는 엣지 AI(Edge AI)가 중요한 흐름으로 자리 잡고 있다. 이는 지연 시간과 전력 소비를 최소화하면서 실시간 성능을 보장하기 위해 고효율 알고리즘과 특수 하드웨어(예: NPU)를 필요로 한다.80
- **상태 추정의 미래:** 현재 최적화 기반 추정의 주류인 팩터 그래프를 넘어서는 새로운 패러다임이 연구되고 있다. 그중 하나는 연속 시간 추정(continuous-time estimation)이다. 이 방법은 상태를 시간의 연속적인 함수(예: 스플라인)로 모델링하여, 비동기적인 고주파수 센서 데이터를 더 자연스럽게 처리하고 유연성과 효율성을 높일 수 있다.85 또한, 필터링, 최적화, 그리고 학습 기반 방법을 결합한 하이브리드 접근법이 차세대 로봇 상태 추정의 강력한 방향으로 부상하고 있다.87

VINS-Mono에서 VINS-Fusion으로, 그리고 SuperVINS와 같은 시스템으로의 진화는 로보틱스 연구의 반복적인 패턴을 명확히 보여준다. 강인하고 범용적인 수학적 프레임워크(최적화 백엔드)가 플랫폼이 되어, 그 위에 특정 환경의 어려움을 해결하기 위한 전문화된 고성능 인식 모듈(프론트엔드)이 구축된다. VINS-Mono는 핵심적인 최적화 엔진을 제공했고, 후속 연구들은 저텍스처, 동적 환경, 조명 변화와 같은 인식 실패 사례를 직접적으로 해결하기 위해 선 특징, 시맨틱 정보, 딥러닝 모델 등을 도입했다. 이는 VIO의 미래가 팩터 그래프 자체를 대체하기보다는, 모든 조건에서 신뢰할 수 있는 제약 조건을 백엔드 최적화기에 제공할 수 있는 더 지능적이고 적응적인 프론트엔드를 개발하는 데 있음을 시사한다.


1. Visual Inertial Navigation Short Tutorial - UMN - MARS Lab, accessed July 19, 2025, https://mars.cs.umn.edu/tr/Stergios_VINS_Tutorial_IROS19.pdf
2. VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator | alphaXiv, accessed July 19, 2025, https://www.alphaxiv.org/overview/1708.03852v1
3. VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator - ar5iv - arXiv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/1708.03852
4. [Paper Review] VINS-mono 요약 및 설명 - GitHub, accessed July 19, 2025, https://taeyoung96.github.io/research/VINS_mono/
5. A Review Of Visual Inertial Odometry For Object Tracking And Measurement - International Journal of Scientific & Technology Research, accessed July 19, 2025, https://www.ijstr.org/final-print/feb2020/A-Review-Of-Visual-Inertial-Odometry-For-Object-Tracking-And-Measurement.pdf
6. SuperVINS: A visual-inertial SLAM framework integrated deep learning features - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.21348v1
7. VINS Mono Body-Of-Knowledge | ai Werkstatt, accessed July 19, 2025, https://www.aiwerkstatt.com/wp-content/uploads/2020/08/VINS-Mono-Body-Of-Knowledge.pdf
8. Review of visual odometry: types, approaches, challenges, and applications - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5084145/
9. VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator, accessed July 19, 2025, https://www.researchgate.net/publication/319122049_VINS-Mono_A_Robust_and_Versatile_Monocular_Visual-Inertial_State_Estimator
10. [1708.03852] VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator, accessed July 19, 2025, https://arxiv.org/abs/1708.03852
11. VINS-Mono: Monocular Visual-Inertial System | Robotics Institute - The Hong Kong University of Science and Technology, accessed July 19, 2025, https://ri.hkust.edu.hk/vins-mono
12. Code for VINS-Mono is now available on GitHub – HDJI / lab, accessed July 19, 2025, https://hdji.hkust.edu.hk/code-for-vins-mono-is-now-available-on-github/
13. arXiv:2312.01616v6 [cs.CV] 17 Jul 2024 - OpenReview, accessed July 19, 2025, https://openreview.net/pdf?id=OhZARx1Zdw
14. PO-MSCKF: An Efficient Visual-Inertial Odometry by Reconstructing the Multi-State Constrained Kalman Filter with the Pose-only Theory - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2407.01888
15. A Benchmark Comparison of Monocular Visual-Inertial Odometry Algorithms for Flying Robots : r/computervision - Reddit, accessed July 19, 2025, https://www.reddit.com/r/computervision/comments/85vc7n/a_benchmark_comparison_of_monocular/
16. A Review of Visual-Inertial Simultaneous Localization and Mapping from Filtering-Based and Optimization-Based Perspectives - MDPI, accessed July 19, 2025, https://www.mdpi.com/2218-6581/7/3/45
17. A Benchmark Comparison of Monocular Visual-Inertial Odometry ..., accessed July 19, 2025, https://rpg.ifi.uzh.ch/docs/ICRA18_Delmerico.pdf
18. SP-VIO: Robust and Efficient Filter-Based Visual Inertial Odometry with State Transformation Model and Pose-Only Visual Description - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.07551v1
19. Consistency of EKF-Based Visual-Inertial Odometry - Department of ..., accessed July 19, 2025, https://intra.ece.ucr.edu/~mourikis/tech_reports/VIO.pdf
20. LRPL-VIO: A Lightweight and Robust Visual–Inertial Odometry with Point and Line Features, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10892506/
21. PL-VINS: Real-Time Monocular Visual-Inertial SLAM with Point and Line Features - ar5iv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2009.07462
22. A review of visual inertial odometry from filtering and optimisation perspectives, accessed July 19, 2025, https://www.researchgate.net/publication/282429809_A_review_of_visual_inertial_odometry_from_filtering_and_optimisation_perspectives
23. Research on the Availability of VINS-Mono and ORB ... - WSEAS, accessed July 19, 2025, https://wseas.com/journals/computers/2020/a545105-048.pdf
24. (PDF) A Review of Visual-Inertial Simultaneous Localization and Mapping from Filtering-Based and Optimization-Based Perspectives - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/327064224_A_Review_of_Visual-Inertial_Simultaneous_Localization_and_Mapping_from_Filtering-Based_and_Optimization-Based_Perspectives
25. VINS Mono - Notion, accessed July 19, 2025, https://yonseidrone.notion.site/VINS-Mono-2e37da4b65f4495fb12e3c4cf7ea0079
26. Formula Derivation and Analysis of the VINS-Mono - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/1912.11986
27. HKUST-Aerial-Robotics/VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator - GitHub, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/VINS-Mono
28. SuperVINS: A visual-inertial SLAM framework integrated deep learning features, accessed July 19, 2025, https://www.researchgate.net/publication/382738929_SuperVINS_A_visual-inertial_SLAM_framework_integrated_deep_learning_features
29. SuperVINS: A Real-Time Visual-Inertial SLAM Framework for Challenging Imaging Conditions - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.21348v2
30. Technical Report: VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator - GitHub, accessed July 19, 2025, https://raw.githubusercontent.com/HKUST-Aerial-Robotics/VINS-Mono/master/support_files/paper/tro_technical_report.pdf
31. Formula Derivation and Analysis of the VINS-Mono - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/338228543_Formula_Derivation_and_Analysis_of_the_VINS-Mono
32. [SLAM] VINS-mono 논문 리뷰 (+ IMU preintegration) - A L I D A - 티스토리, accessed July 19, 2025, https://alida.tistory.com/64
33. [SLAM][En] Notes on Formula Derivation and Analysis of the VINS-mono - A L I D A, accessed July 19, 2025, https://alida.tistory.com/67
34. VINS-Dimc: A Visual-Inertial Navigation System for Dynamic Environment Integrating Multiple Constraints - MDPI, accessed July 19, 2025, https://www.mdpi.com/2220-9964/11/2/95
35. VINS-Multi: A Robust Asynchronous Multi-camera-IMU State Estimator - ICRA 2025 Construction Robotics Workshop, accessed July 19, 2025, https://construction-robots.github.io/papers/60.pdf
36. Developing a Flying Explorer for Autonomous Digital Modelling in Wild Unknowns - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/3/1021
37. Getting Started » Sensor Calibration | OpenVINS, accessed July 19, 2025, https://docs.openvins.com/gs-calibration.html
38. HKUST-Aerial-Robotics/VINS-Fusion: An optimization-based multi-sensor state estimator, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/VINS-Fusion
39. Online Time Offset Modeling Networks for Robust Temporal Alignment in High Dynamic Motion VIO - arXiv, accessed July 19, 2025, https://arxiv.org/html/2403.12504v1
40. TON-VIO: Online Time Offset Modeling Networks for Robust Temporal Alignment in High Dynamic Motion VIO - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/379079851_TON-VIO_Online_Time_Offset_Modeling_Networks_for_Robust_Temporal_Alignment_in_High_Dynamic_Motion_VIO
41. [1808.00692] Online Temporal Calibration for Monocular Visual-Inertial Systems - arXiv, accessed July 19, 2025, https://arxiv.org/abs/1808.00692
42. Theoretical question regarding global vs rolling shutter behaviour and accounting for it with ZED2 and D455 / Issue #464 / rpng/open_vins - GitHub, accessed July 19, 2025, https://github.com/rpng/open_vins/issues/464
43. cggos/vins_mono_cg: Modified version of VINS-Mono (commit 9e657be) - GitHub, accessed July 19, 2025, https://github.com/cggos/vins_mono_cg
44. VINS-FUSION drift / Issue #41 / HKUST-Aerial-Robotics/VINS-Fusion - GitHub, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/VINS-Fusion/issues/41
45. Trying to solve extreme drift with software synchronization / Issue #431 / HKUST-Aerial-Robotics/VINS-Mono - GitHub, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/VINS-Mono/issues/431
46. VINS-MONO OPTMIZED: A MONOCULAR VISUAL-INERTIAL STATE ESTIMATOR WITH IMPROVED INITIALIZATION A Thesis by LINGJIE XU Submitted t - OAKTrust, accessed July 19, 2025, https://oaktrust.library.tamu.edu/bitstream/handle/1969.1/174383/XU-THESIS-2018.pdf?sequence=1&isAllowed=y
47. Enhancing Visual–Inertial Odometry Robustness and Accuracy in Challenging Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/2218-6581/14/6/71
48. D-VINS: Dynamic Adaptive Visual–Inertial SLAM with IMU Prior and Semantic Constraints in Dynamic Scenes - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/15/15/3881
49. (PDF) PC-VINS-Mono: A Robust Mono Visual-Inertial Odometry with ..., accessed July 19, 2025, https://www.researchgate.net/publication/330526039_PC-VINS-Mono_A_Robust_Mono_Visual-Inertial_Odometry_with_Photometric_Calibration
50. PC-VINS-Mono: A Robust Mono Visual-Inertial Odometry with Photometric Calibration - Frontier Scientific Publishing, accessed July 19, 2025, https://en.front-sci.com/index.php/jai/article/view/33/21
51. A Comparison of Monocular Visual SLAM and Visual Odometry Methods Applied to 3D Reconstruction - MDPI, accessed July 19, 2025, https://www.mdpi.com/2076-3417/13/15/8837
52. [2007.11898] ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - ar5iv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2007.11898
53. [Nav2] A Comparison of Modern General-Purpose Visual SLAM Approaches, accessed July 19, 2025, https://discourse.ros.org/t/nav2-a-comparison-of-modern-general-purpose-visual-slam-approaches/21451
54. The comparison of trajectories between our method, VINS-Mono and... - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/The-comparison-of-trajectories-between-our-method-VINS-Mono-and-ORB-SLAM3-with-the_fig5_364973826
55. Comparison of the proposed method versus VINS-Mono. The three colorful... - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/Comparison-of-the-proposed-method-versus-VINS-Mono-The-three-colorful-trajectories-of_fig7_324421212
56. S-MSCKF vs VINS-Fusion(Stereo) on FlightGoggles simulator from MIT - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=s_Ol-k8rhwY
57. High-precision, consistent EKF-based visual–inertial odometry | Request PDF, accessed July 19, 2025, https://www.researchgate.net/publication/258140599_High-precision_consistent_EKF-based_visual-inertial_odometry
58. High-precision, consistent EKF-based visual-inertial odometry - Semantic Scholar, accessed July 19, 2025, https://pdfs.semanticscholar.org/0be0/c13803cd08e81b7adaada537e91222eb1491.pdf
59. EKF-based Visual-Inertial Navigation on Matrix Lie group with Improved Consistency.pdf - S-Space, accessed July 19, 2025, [https://s-space.snu.ac.kr/bitstream/10371/143100/1/EKF-based%20Visual-Inertial%20Navigation%20on%20Matrix%20Lie%20group%20with%20Improved%20Consistency.pdf](https://s-space.snu.ac.kr/bitstream/10371/143100/1/EKF-based Visual-Inertial Navigation on Matrix Lie group with Improved Consistency.pdf)
60. Fast and Robust Monocular Visua-Inertial Odometry Using Points and Lines - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC6832589/
61. A Benchmark Comparison of Monocular Visual-Inertial Odometry Algorithms for Flying Robots | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/325369350_A_Benchmark_Comparison_of_Monocular_Visual-Inertial_Odometry_Algorithms_for_Flying_Robots
62. A Vision/Inertial Navigation/Global Navigation Satellite Integrated System for Relative and Absolute Localization in Land Vehicles - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/10/3079
63. Tightly-coupled Fusion of Global Positional Measurements in Optimization-based Visual-Inertial Odometry - Robotics and Perception Group, accessed July 19, 2025, https://rpg.ifi.uzh.ch/docs/IROS20_Cioffi.pdf
64. Tightly Coupled Optimization-based GPS-Visual-Inertial Odometry with Online Calibration and Initialization - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2203.02677
65. Visual-Inertial SLAM with Tightly-Coupled Dropout-Tolerant GPS Fusion - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2208.00709
66. robust visual-inertial odometry in dynamic environments using semantic segmentation for feature selection - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/343400726_ROBUST_VISUAL-INERTIAL_ODOMETRY_IN_DYNAMIC_ENVIRONMENTS_USING_SEMANTIC_SEGMENTATION_FOR_FEATURE_SELECTION
67. Semantic SLAM for mobile robots in dynamic environments based on visual camera sensors, accessed July 19, 2025, https://qizhang-research.com/publication/journal-article/
68. Visual SLAM for Dynamic Environments Based on Object Detection and Optical Flow for Dynamic Object Removal - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/22/19/7553
69. Example of dynamic environments (TUM). | Download Scientific Diagram - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/Example-of-dynamic-environments-TUM_fig1_353476587
70. DS-SLAM: A Semantic Visual SLAM towards Dynamic Environments - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/1809.08379
71. MLINE-VINS: Robust Monocular Visual-Inertial SLAM With Flow Manhattan and Line Features - arXiv, accessed July 19, 2025, https://arxiv.org/html/2503.01571v1
72. [논문 리뷰] MLINE-VINS: Robust Monocular Visual-Inertial SLAM With Flow Manhattan and Line Features, accessed July 19, 2025, https://www.themoonlight.io/ko/review/mline-vins-robust-monocular-visual-inertial-slam-with-flow-manhattan-and-line-features
73. MLINE-VINS: Robust Monocular Visual-Inertial SLAM With Flow Manhattan and Line Features - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/389581047_MLINE-VINS_Robust_Monocular_Visual-Inertial_SLAM_With_Flow_Manhattan_and_Line_Features
74. PL-VIWO: A Lightweight and Robust Point-Line Monocular Visual Inertial Wheel Odometry - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2503.00551
75. UMPL- VINS: Generalized SLAM with Point and Line Features for Multi-scene Applications, accessed July 19, 2025, https://www.researchgate.net/publication/377629052_UMPL-_VINS_Generalized_SLAM_with_Point_and_Line_Features_for_Multi-scene_Applications
76. [IEEE Sensors Journal (JSEN) ] SuperVINS: A Real-Time Visual-Inertial SLAM Framework for Challenging Imaging Conditions (integrated deep learning features) - GitHub, accessed July 19, 2025, https://github.com/luohongk/SuperVINS
77. SuperVINS: A Real-Time Visual-Inertial SLAM Framework for Challenging Imaging Conditions | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/390512894_SuperVINS_A_Real-Time_Visual-Inertial_SLAM_Framework_for_Challenging_Imaging_Conditions
78. SuperVINS: A visual-inertial SLAM framework integrated deep learning features - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2407.21348v1/
79. [2407.21348] SuperVINS: A Real-Time Visual-Inertial SLAM Framework for Challenging Imaging Conditions - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2407.21348
80. Edge AI: Are You Ready to Lead? - Arm Newsroom, accessed July 19, 2025, https://newsroom.arm.com/blog/edge-ai-embedded-development-vdc-research-report
81. Edge AI visual processing | Vision Bot, accessed July 19, 2025, https://visionbot.com/why-edge-ai-visual-processing-is-the-future-of-real-time-industrial-automation/
82. Edge AI is a Game-Changer for Embedded Devices - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=HuOVictahCI
83. Edge AI and Vision Alliance: Home, accessed July 19, 2025, https://www.edge-ai-vision.com/
84. 2.1 Edge computing and embedded Artificial Intelligence | ECS SRIA, accessed July 19, 2025, https://ecssria.eu/2.1
85. Proprioceptive Invariant Robot State Estimation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2311.04320v2
86. Continuous-Time State Estimation Methods in Robotics: A Survey - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2411.03951
87. OptiState: State Estimation of Legged Robots using Gated Networks with Transformer-based Vision and Kalman Filtering - arXiv, accessed July 19, 2025, https://arxiv.org/html/2401.16719v1
88. Fast Decentralized State Estimation for Legged Robot Locomotion via EKF and MHE - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.20567v3


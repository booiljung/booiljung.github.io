# LVI-SAM



자율 시스템의 핵심 기술인 SLAM(Simultaneous Localization and Mapping)의 발전은 단일 센서의 한계를 극복하려는 노력과 궤를 같이 해왔다. 특히 LiDAR, 카메라, IMU(Inertial Measurement Unit)는 각각 명확한 장점과 본질적인 단점을 지니고 있어, 이들을 융합하는 것은 선택이 아닌 필연적인 연구 방향으로 자리 잡았다.1

LiDAR는 레이저를 이용하여 주변 환경과의 거리를 직접 측정하므로, 조명 변화에 강인하고 매우 정확한 3차원 기하 구조 정보를 획득할 수 있다.3 하지만 긴 복도나 넓은 벽면과 같이 기하학적 특징이 부족하거나 반복적인 구조를 가진 환경에서는 스캔 매칭(scan-matching)이 모호해져 위치 추정 성능이 급격히 저하되는 '퇴화(degeneracy)' 문제에 직면한다.5

반면, 카메라는 저렴하고 소형이며, 환경의 풍부한 텍스처와 색상 정보를 포착한다. 이는 특히 장소 인식(place recognition)이나 루프 클로저(loop closure) 검출에 매우 유리하게 작용한다.5 그러나 카메라는 조명 변화에 극도로 민감하며, 빛이 너무 많거나 적은 환경, 또는 빠른 움직임으로 인한 모션 블러(motion blur) 상황에서는 특징점 추적에 실패할 확률이 높다.3 또한, 단안 카메라의 경우, 깊이(depth) 정보를 직접 알 수 없어 스케일 모호성(scale ambiguity) 문제가 발생한다.9

IMU는 가속도계와 자이로스코프로 구성되어, 외부 환경에 의존하지 않고 수백 Hz에 달하는 높은 주파수로 시스템의 운동(가속도, 각속도)을 측정한다.10 이 고주파 정보는 LiDAR 포인트 클라우드의 모션 왜곡(motion distortion)을 보정하거나, 카메라 프레임 간의 상대적인 움직임을 예측하는 데 결정적인 역할을 한다. 또한, 중력 벡터 측정을 통해 시스템의 롤(roll)과 피치(pitch) 각도를 추정하고, VIO(Visual-Inertial Odometry) 시스템에 미터 단위의 스케일(metric scale) 정보를 제공할 수 있다.10 하지만 IMU는 측정값에 노이즈와 바이어스가 포함되어 있어, 이를 적분하여 위치를 계산하면 시간이 지남에 따라 오차가 기하급수적으로 누적되는 치명적인 단점을 가진다.11

이처럼 각 센서는 서로가 갖지 못한 정보를 제공하는 상호 보완적인 관계에 있다. LiDAR의 기하학적 정확성, 카메라의 텍스처 풍부함, 그리고 IMU의 고주파 운동 정보를 결합한 LiDAR-Visual-Inertial (LVI) 융합 SLAM은, 각 센서가 퇴화하는 시나리오에서도 다른 센서의 정보를 활용하여 시스템 전체의 강인성(robustness)과 정확성(accuracy)을 극대화할 수 있다. 이는 다양한 도전적인 실제 환경에서 자율 시스템이 안정적으로 동작하기 위한 필수적인 접근법이다.3


다중 센서 융합 방식은 크게 Loosely-coupled(느슨한 결합)와 Tightly-coupled(강한 결합)의 두 가지 아키텍처로 분류된다.5 이 둘의 선택은 시스템의 정확도와 복잡도를 결정하는 핵심적인 설계 철학의 차이를 반영한다.

Loosely-coupled 방식은 각 센서 모듈이 독립적으로 상태 추정을 수행한 후, 그 결과를 후처리 단계에서 융합하는 구조다. 예를 들어, VIO 서브시스템이 카메라와 IMU 데이터로 자세를 추정하고, LIO(Lidar-Inertial Odometry) 서브시스템이 LiDAR와 IMU 데이터로 또 다른 자세를 추정한 뒤, 이 두 개의 독립적인 자세 추정치를 칼만 필터(Kalman Filter) 등을 이용해 최종적으로 결합하는 방식이다.5 VIL-SLAM 5과 같은 초기 시스템에서는 VIO가 추정한 자세를 LIO의 초기값으로만 사용하는 제한적인 정보 교환만 이루어졌다. 이 방식은 구현이 비교적 간단하고 모듈화가 용이하다는 장점이 있지만, 각 센서의 원시 측정값(raw measurements)에 담긴 상호 제약 정보를 완전히 활용하지 못하기 때문에 최적의 정확도를 달성하기 어렵다.

반면, Tightly-coupled 방식은 모든 센서로부터 들어오는 원시 측정값 또는 전처리된 특징(features)들을 단일 최적화 문제(single optimization problem) 내에서 동시에 고려하여 시스템의 상태 변수(자세, 속도, 바이어스 등)를 추정한다.5 예를 들어, LiDAR 특징점의 기하학적 오차, 카메라 특징점의 재투영 오차, IMU의 사전 적분(pre-integration) 오차 등을 모두 포함하는 하나의 비용 함수(cost function)를 정의하고, 이를 최소화하는 해를 찾는 방식이다. 이 접근법은 센서 간의 상호 작용과 제약 조건을 최대한으로 활용하므로 이론적으로 더 높은 정확도를 얻을 수 있다.13 LVI-SAM은 바로 이 Tightly-coupled 접근법을 채택하여, LiDAR, 카메라, IMU의 정보를 하나의 팩터 그래프(Factor Graph) 위에서 긴밀하게 결합한다.3 이는 시스템의 성능을 극대화하기 위한 핵심적인 설계 결정이었다.


LVI-SAM은 완전히 새로운 알고리즘을 밑바닥부터 개발한 것이 아니라, 당시 각 분야에서 최고 수준(State-of-the-Art, SOTA)으로 인정받던 두 개의 오픈소스 프로젝트, 즉 Lidar-Inertial Odometry인 LIO-SAM과 Visual-Inertial Odometry인 VINS-Mono의 장점을 시스템 수준에서 창조적으로 통합한 결과물이다.15 이러한 접근 방식은 SLAM 연구의 발전 방향이 개별 알고리즘의 미세한 성능 개선을 넘어, 이미 검증된 이종(heterogeneous)의 SOTA 시스템들을 어떻게 효과적으로 융합하여 시너지를 창출할 것인가에 대한 시스템 엔지니어링적 고민으로 확장되었음을 보여준다.

LIO-SAM 16은 LOAM 12을 기반으로 IMU pre-integration을 긴밀하게 결합하고, 팩터 그래프 최적화를 통해 실시간으로 정확한 Lidar-Inertial 오도메트리를 구현한 성공적인 프로젝트였다. VINS-Mono 11는 단안 카메라와 IMU만을 사용하여 최적화 기반의 슬라이딩 윈도우 방식으로 매우 정확하고 강인한 VIO를 구현하여 학계에 큰 영향을 미쳤다.

LVI-SAM의 개발 목표는 이 두 강력한 시스템의 장점은 취하고, 각자가 가진 약점은 서로를 통해 보완하는 것이었다. 예를 들어, VINS-Mono는 텍스처가 풍부한 환경에서는 강하지만 초기화가 어렵고 스케일 모호성이 있으며, LIO-SAM은 기하학적 구조가 명확한 환경에서는 강하지만 특징이 없는 환경에서는 취약했다. LVI-SAM은 이 둘을 Tightly-coupled 프레임워크 안에서 융합함으로써, 한쪽 시스템이 어려움을 겪는 환경에서도 다른 쪽 시스템이 보조하여 전체 시스템이 강건하게 동작하도록 설계되었다. 따라서 LVI-SAM의 핵심적인 혁신은 새로운 알고리즘의 발명이 아닌, 검증된 모듈들을 하나의 유기적인 시스템으로 엮어낸 '시스템 통합의 승리'로 평가할 수 있으며, 그 독창성은 두 시스템을 묶는 '접착제' 역할을 하는 Tightly-coupled Factor Graph 아키텍처에 있다.3



LVI-SAM의 아키텍처는 그 핵심 철학을 명확히 보여주는 이중 서브시스템 구조로 설계되었다. 시스템은 3D LiDAR, 단안 카메라, IMU로부터 센서 데이터를 입력받아, 최종적으로 IMU의 데이터 수신율에 맞춰 실시간으로 6-DOF(Degrees of Freedom) 자세를 출력한다.3

이 시스템의 심장부에는 두 개의 핵심 모듈, 즉 **Visual-Inertial System (VIS)**와 **Lidar-Inertial System (LIS)**가 존재한다.14 이 두 서브시스템은 독립적으로 작동할 수 있는 능력을 갖추고 있으면서도, 동시에 서로의 정보를 긴밀하게 주고받으며 상호 보완적으로 동작하도록 설계되었다. 이러한 이중 구조는 한 종류의 센서 정보가 부족하거나 신뢰할 수 없는 환경에 직면했을 때, 다른 센서 기반의 서브시스템이 전체 시스템의 안정성을 유지시켜주는 역할을 한다.

모든 센서 정보의 융합과 상태 추정은 팩터 그래프(Factor Graph)라는 강력한 모델링 도구를 기반으로 이루어진다. 팩터 그래프는 상태 변수(자세, 속도, 바이어스 등)를 노드(node)로, 센서 측정값으로부터 파생된 제약 조건(constraint)을 팩터(factor)로 표현하는 그래프 모델이다.16 LVI-SAM은 Georgia Tech Smoothing and Mapping (GTSAM) 라이브러리 15를 사용하여 이 팩터 그래프를 구축하고, 비선형 최적화를 통해 모든 제약 조건을 가장 잘 만족시키는 상태 변수들을 추정한다. 이 팩터 그래프 위에서 VIS와 LIS로부터 오는 정보들이 통합되어 전역적으로 일관된 해를 찾게 된다.1


LVI-SAM 아키텍처의 진정한 힘은 두 서브시스템이 단순히 병렬로 실행되는 것을 넘어, 서로의 약점을 적극적으로 보완하는 상호작용 방식과, 각자의 신뢰도를 스스로 판단하는 실패 감지 메커니즘에 있다. 이는 정적인 파이프라인이 아닌, 환경 변화에 동적으로 대응하는 유기적인 시스템을 구현한다.


두 서브시스템 간의 정보 교환은 LVI-SAM 성능의 핵심이다.

- **LIS가 VIS를 지원:** VIO 시스템의 가장 큰 난제 중 하나는 초기화 과정이다. 움직임이 충분하지 않거나, 스케일, 중력 방향 등을 추정해야 하는 고도의 비선형 문제 때문에 초기화에 실패하는 경우가 잦다.3 LVI-SAM의 VIS는 이 문제를 LiDAR의 직접적인 깊이 측정 능력을 활용하여 해결한다. 먼저 LIS가 안정적으로 초기화되어 자세, 속도, IMU 바이어스 등을 추정하면, 이 정보를 VIS의 초기값으로 제공한다. 이로써 VIS는 복잡한 초기화 과정을 건너뛰고 빠르고 강건하게 동작을 시작할 수 있다.3
- **VIS가 LIS를 지원:** LIS의 스캔 매칭 과정은 정확한 초기 추정값(initial guess)이 주어졌을 때 훨씬 빠르고 정확하게 수렴한다. VIS는 카메라와 IMU를 통해 빠르고 연속적인 오도메트리 추정치를 생성할 수 있으므로, 이를 LIS의 스캔 매칭을 위한 초기 추정값으로 제공한다. 이는 특히 로봇이 빠르게 회전하거나 움직일 때 LIS가 매칭에 실패할 위험을 크게 줄여준다.3
- **LiDAR를 통한 VIS 정확도 향상:** 단안 VIS는 특징점의 3D 위치를 추정하기 위해 여러 프레임에 걸친 삼각측량(triangulation)이 필요하며, 이는 스케일 모호성을 내포한다. LVI-SAM의 VIS는 LIS로부터 얻은 포인트 클라우드를 이용해 현재 추적 중인 시각 특징점들의 깊이(depth)를 직접적으로 부여한다.3 이는 2D-2D 매칭 제약을 3D-2D 제약으로 강화하여 VIO의 정확도를 크게 향상시키는 역할을 한다.3
- **협력적 루프 클로저:** 전역적인 누적 오차를 보정하는 루프 클로저는 2단계로 진행된다. 먼저, 텍스처 정보에 강한 VIS가 DBoW2와 같은 장소 인식 기술을 사용해 루프 클로저 후보를 신속하게 찾아낸다. 그 후, 기하학적 정확도가 높은 LIS가 이 후보 구간에 대해 정밀한 스캔 매칭을 수행하여 루프 클로저를 최종적으로 검증하고 정제한다. 이 방식은 빠르고 정확한 루프 클로저를 가능하게 한다.3


LVI-SAM의 가장 두드러진 특징은 '신뢰도 기반의 동적 책임 분배 시스템'으로 볼 수 있는 실패 감지 메커니즘이다.3 각 서브시스템은 자신의 출력 품질을 지속적으로 모니터링하고, 신뢰도가 기준치 이하로 떨어지면 실패를 선언한다.

- **VIS 실패 감지:** VIS는 추적하는 시각 특징점의 개수가 특정 임계값 이하로 떨어지거나(e.g., 텍스처가 없는 벽면을 볼 때), 추정된 IMU 바이어스 값이 비정상적으로 발산할 경우 실패를 감지한다. 실패가 감지되면 VIS는 자신의 결과를 팩터 그래프에 추가하는 것을 중단하고, LIS로부터 새로운 정보를 받아 재초기화를 시도한다.14
- **LIS 실패 감지:** LIS는 스캔 매칭을 위한 비선형 최적화 과정에서 정보 행렬(information matrix, 근사적인 헤시안 행렬) $A^TA$의 가장 작은 고유값(eigenvalue)을 확인한다. 이 고유값이 작다는 것은 특정 방향으로의 움직임에 대한 제약 조건이 매우 약하다는 의미이며(e.g., 긴 복도에서 전진 방향), 이는 퇴화 상태를 나타낸다. LIS는 이 고유값이 임계값보다 작으면 실패를 보고하고, 해당 라이다 오도메트리 팩터를 그래프에 추가하지 않는다. 이는 신뢰할 수 없는 정보가 전체 시스템의 상태 추정을 오염시키는 것을 방지하는 중요한 안전장치다.3

이러한 상호 보완적인 정보 교환과 독립적인 실패 감지 메커니즘의 결합 덕분에, LVI-SAM은 텍스처가 없는 환경(VIS 실패, LIS 동작)과 기하학적 특징이 없는 환경(LIS 실패, VIS 동작) 모두에서 끊김 없이 강인하게 동작할 수 있다.3 이는 단순히 두 모듈을 합쳐놓은 것을 넘어, 실시간으로 각 모듈의 신뢰도를 평가하고 그에 따라 정보의 흐름과 가중치를 동적으로 조절하는 고도의 지능적인 제어 로직이 내장되어 있음을 보여준다.


LVI-SAM의 강력한 성능은 각자의 분야에서 최고 수준으로 검증된 LIO-SAM과 VINS-Mono의 핵심 알고리즘을 계승하고, 이를 유기적으로 결합한 데서 비롯된다. 두 서브시스템은 독립적으로도 완결된 구조를 갖추고 있지만, LVI-SAM 프레임워크 내에서는 서로의 약점을 직접적으로 보완하는 '적극적인 개입' 관계를 형성한다.


LVI-SAM의 LIS는 Tixiao Shan의 이전 연구인 LIO-SAM 15을 기반으로 하며, 이는 다시 3D LiDAR SLAM의 효시라 할 수 있는 LOAM 12의 아이디어를 계승한다. LIS는 LiDAR의 정밀한 기하학 정보와 IMU의 고주파 운동 정보를 긴밀하게 결합하여 정확하고 강인한 오도메트리와 맵을 생성하는 역할을 담당한다.


3D LiDAR 센서는 보통 회전하면서 레이저를 발사하기 때문에, 한 번의 스캔(sweep)이 완료되는 데 수십~수백 밀리초가 소요된다. 이 시간 동안 로봇이 움직이면 스캔의 시작 부분과 끝 부분의 포인트들이 서로 다른 위치와 자세에서 측정되어 포인트 클라우드가 휘어지거나 늘어나는 왜곡(distortion 또는 skew)이 발생한다. 이 왜곡을 보정하지 않으면 특징점의 위치가 부정확해져 스캔 매칭의 정확도가 크게 저하된다. LIS는 두 LiDAR 키프레임 사이의 IMU 측정값을 사전 적분(pre-integration)하여 얻은 고주파의 상대적인 움직임 정보를 이용해, 각 포인트가 측정된 정확한 시점의 센서 자세를 보간(interpolate)한다. 이 정보를 바탕으로 모든 포인트를 스캔의 종료 시점으로 변환하여 왜곡이 제거된(de-skewed) 포인트 클라우드를 생성한다.3 이는 LIS 파이프라인의 가장 첫 단계이자, 후속 처리의 정확성을 보장하는 필수적인 과정이다.


왜곡이 보정된 포인트 클라우드로부터 스캔 매칭에 사용할 안정적인 특징점을 추출한다. LIS는 LOAM과 유사하게 각 포인트의 주변 곡률(local curvature)을 계산하여 특징점을 분류한다.3

- **Edge Features (모서리 특징):** 곡률이 큰 포인트들로, 주로 물체의 모서리나 코너에 해당한다.
- **Planar Features (평면 특징):** 곡률이 작은 포인트들로, 벽, 바닥, 천장과 같은 평평한 표면에 해당한다.

전체 포인트 클라우드를 사용하는 대신, 이렇게 의미 있는 기하학적 특징점들만 선별하여 사용함으로써 계산 효율성을 높이고 매칭의 강인성을 확보한다.


LIS는 계산 복잡도를 제한하고 실시간 성능을 보장하기 위해, 과거의 모든 포인트 클라우드를 포함하는 전역 맵(global map) 대신, 최근의 키프레임들로 구성된 슬라이딩 윈도우(sliding window) 방식의 로컬 맵(local map)을 유지한다.3

1. **로컬 맵 구성:** 시스템은 최근 $N$개의 키프레임(논문에서는 서브-키프레임이라 칭함)을 선택하여 이들을 최적화된 자세로 변환한 후, 하나의 포인트 클라우드 맵으로 병합한다. 이 로컬 맵은 계산 효율성을 위해 복셀 그리드(voxel grid)로 다운샘플링되며, 에지 특징 맵과 평면 특징 맵으로 분리되어 관리된다.16

2. **매칭 및 최적화:** 새로운 키프레임이 들어오면, 이 키프레임에서 추출된 에지/평면 특징점들을 로컬 맵의 해당 에지/평면 구조에 정합(register)한다. 정합 과정은 각 특징점과 맵 상의 대응되는 기하 구조(선 또는 평면) 사이의 거리를 최소화하는 비선형 최적화 문제로 공식화된다.

   - **Point-to-line distance:** 새로운 에지 특징점과 로컬 맵에서 찾은 가장 가까운 에지 라인(두 점으로 정의) 사이의 수직 거리 오차를 계산한다.

   - Point-to-plane distance: 새로운 평면 특징점과 로컬 맵에서 찾은 가장 가까운 평면 패치(세 점으로 정의) 사이의 수직 거리 오차를 계산한다.

     이 두 거리 오차의 합을 최소화하는 상대 변환(자세와 위치)을 Gauss-Newton이나 Levenberg-Marquardt와 같은 최적화 기법으로 찾는다. 이 결과가 바로 Lidar Odometry Factor가 된다.16

LIS는 VIS로부터 받은 오도메트리 추정치를 이 최적화 과정의 초기 추정값으로 사용함으로써, 탐색 공간을 크게 줄이고 격렬한 움직임 속에서도 빠르고 안정적으로 수렴할 수 있게 된다.3


LVI-SAM의 VIS는 VINS-Mono 3의 검증된 파이프라인을 기반으로 구축되었다. VINS-Mono는 최적화 기반의 슬라이딩 윈도우 방식을 사용하여 단안 카메라와 저가형 IMU만으로도 SOTA 수준의 정확도를 달성한 것으로 유명하다.11 LVI-SAM의 VIS는 이 강력한 VIO 엔진을 채택하되, LiDAR 데이터를 적극적으로 활용하여 VIO의 근본적인 한계를 극복한다.


VIS의 프론트엔드는 일반적인 VIO 파이프라인을 따른다.

1. **특징점 검출:** 새로운 이미지 프레임이 들어오면, Shi-Tomasi 코너 검출기 8를 사용하여 추적에 적합한 특징점들을 찾는다. 이때, 이미지 전체에 특징점이 균일하게 분포하도록 최소 이격 거리를 강제한다.11
2. **특징점 추적:** 검출된 특징점들을 Kanade-Lucas-Tomasi (KLT) 광학 흐름(optical flow) 알고리즘 14을 사용하여 다음 프레임까지 추적한다.
3. **이상치 제거:** 추적된 특징점들 중 잘못된 매칭을 제거하기 위해, RANSAC(Random Sample Consensus)을 이용한 Fundamental Matrix 테스트를 수행한다. 이 테스트를 통과한 특징점들만이 유효한 측정값으로 간주된다.11


VIS는 단순한 VIO가 아니라, LIS와의 긴밀한 협력을 통해 'Lidar-aided VIO'로 동작한다. 이는 VIS의 성능을 극적으로 향상시키는 핵심적인 차별점이다.

- **초기화 촉진 (Robust Initialization):** 순수 VIO는 시스템의 초기 상태(자세, 속도, 중력 방향, IMU 바이어스)와 3D 특징점의 위치를 모두 추정해야 하는 매우 비선형적이고 어려운 초기화 과정을 거친다. 이 과정은 센서의 움직임이 충분하지 않으면 실패하기 쉽다. LVI-SAM의 VIS는 이 문제를 우아하게 회피한다. 먼저 LIS가 LiDAR의 절대 거리 측정 능력을 바탕으로 안정적으로 초기화되면, LIS가 추정한 자세, 속도, IMU 바이어스 값을 VIS의 초기화에 필요한 초기 추정치로 직접 사용한다. 이 '정보의 이식'은 VIS의 초기화 실패율을 크게 낮추고, 시스템 전체가 매우 빠르고 강건하게 동작을 시작할 수 있도록 만든다.3
- **시각 특징점에 깊이 정보 부여 (Depth Registration):** 단안 VIO의 가장 큰 약점은 깊이, 즉 스케일을 직접 알 수 없다는 점이다. LVI-SAM의 VIS는 이 한계를 LiDAR로 직접 보완한다. 추정된 카메라 자세를 이용해 LIS가 생성한 LiDAR 포인트 클라우드를 현재 이미지 프레임의 좌표계로 변환하여 투영한다. 그 다음, 현재 추적 중인 2D 시각 특징점들의 위치에 해당하는 3D 깊이 값을 이 투영된 포인트 클라우드로부터 직접 얻는다.3
  - 이 과정을 위해, 단일 LiDAR 스캔은 희소(sparse)할 수 있으므로 여러 개의 연속된 LiDAR 프레임을 누적하여 더 밀도 높은 깊이 맵을 생성한다.
  - 시각 특징점과 깊이 포인트들을 카메라 중심의 단위 구(unit sphere)에 투영한 뒤, 2차원 K-D 트리를 사용하여 각 시각 특징점에 가장 가까운 3개의 깊이 포인트를 효율적으로 검색한다. 이 3개의 깊이 포인트가 형성하는 평면과, 카메라 중심에서 시각 특징점으로 나아가는 광선(ray)의 교점을 계산하여 최종적인 깊이 값을 결정한다.
  - 서로 다른 시간의 LiDAR 프레임을 누적할 때 발생할 수 있는 깊이 모호성을 제거하기 위해, 검색된 3개의 깊이 포인트 간의 거리가 특정 임계값(e.g., 2m)을 초과하면 해당 깊이 추정치를 유효하지 않은 것으로 판단하고 폐기하는 검증 절차를 거친다.3

이처럼 VIS는 LiDAR의 절대 깊이 정보를 '빌려와' 단안 VIO의 가장 큰 약점인 스케일 모호성을 근본적으로 해결하고, LIS는 VIS의 빠르고 안정적인 모션 추정치를 '빌려와' 계산 비용이 큰 스캔 매칭의 탐색 공간을 줄인다. 이러한 상호 개입은 두 시스템이 단순한 병렬 처리를 넘어, 서로의 데이터 처리 파이프라인 깊숙이 관여하는 Tightly-coupled 설계의 진정한 의미를 보여주며, 시스템 전체의 성능을 각 서브시스템의 단순 합 이상으로 끌어올리는 시너지의 원천이 된다.


LVI-SAM의 핵심은 다양한 센서로부터 얻어지는 이종의 정보들을 하나의 일관된 프레임워크 안에서 통합하고 최적화하는 능력에 있다. 이 역할의 중심에는 팩터 그래프 최적화(Factor Graph Optimization)가 자리 잡고 있다.


LVI-SAM이 해결하고자 하는 문제는 주어진 모든 센서 측정값(LiDAR 스캔, 이미지, IMU 데이터)을 바탕으로, 로봇의 전체 궤적(trajectory)과 맵을 가장 잘 설명하는 상태 변수(state variables)를 추정하는 것이다. 이는 통계적으로 최대 사후 확률(Maximum A Posteriori, MAP) 추정 문제로 공식화될 수 있다.16 베이즈 정리에 따르면, 사후 확률은 가능도(likelihood)와 사전 확률(prior)의 곱에 비례한다.
$$
P(\text{States} | \text{Measurements}) \propto P(\text{Measurements} | \text{States}) \cdot P(\text{States})
$$
여기서 '가능도'는 현재 추정된 상태 변수들이 실제 센서 측정값들을 얼마나 잘 설명하는지를 나타내고, '사전 확률'은 상태 변수에 대한 이전의 믿음이나 정보를 의미한다.

만약 모든 센서 측정값의 노이즈가 독립적이고 가우시안 분포(Gaussian distribution)를 따른다고 가정하면, MAP 추정 문제는 모든 측정 오차(residual)의 마할라노비스 거리(Mahalanobis distance) 제곱의 합을 최소화하는 비선형 최소제곱(Nonlinear Least-Squares) 문제와 등가가 된다.16
$$
\min_{\mathcal{X}} \left( \sum_{i} \| f_i(\mathcal{X}_i) - z_i \|^2_{\Sigma_i} \right)
$$
여기서 $\mathcal{X}$는 최적화할 전체 상태 변수 집합(모든 시간의 자세, 속도, 바이어스 등), $z_i$는 $i$번째 센서 측정값, $f_i(\mathcal{X}_i)$는 상태 변수 $\mathcal{X}_i$로부터 예측한 측정값, 그리고 $\| \cdot \|^2_{\Sigma_i}$는 공분산 행렬 $\Sigma_i$로 정규화된 마할라노비스 노름의 제곱을 의미한다.

팩터 그래프는 이러한 복잡한 비선형 최소제곱 문제를 직관적으로 시각화하고 효율적으로 풀기 위한 강력한 모델링 도구다. 팩터 그래프에서 노드(node)는 우리가 추정하고자 하는 상태 변수(e.g., $k$ 시점의 자세 $x_k$)를 나타내고, 팩터(factor)는 이 변수들 사이에 존재하는 제약 조건, 즉 측정 오차 항을 나타낸다.14 LVI-SAM은 이 팩터 그래프 모델을 사용하여 LiDAR, 카메라, IMU로부터 오는 다양한 제약 조건들을 하나의 거대한 최적화 문제로 통합한다.


LVI-SAM의 팩터 그래프는 주로 네 가지 종류의 핵심적인 팩터(제약 조건)로 구성된다. 각 팩터는 특정 센서 측정값으로부터 파생되며, 전체 비용 함수에 하나의 항으로 추가된다.3


이 팩터는 두 개의 연속된 키프레임 사이의 IMU 측정값(각속도, 가속도)으로부터 생성되는 운동학적 제약이다.3 IMU는 매우 높은 주파수(e.g., 200 Hz 이상)로 데이터를 제공하는데, 최적화 기반 SLAM에서는 키프레임 단위(e.g., 10 Hz)로 상태를 추정한다. 만약 최적화 과정에서 키프레임의 자세가 조금씩 수정될 때마다 그 사이의 수많은 IMU 측정값들을 매번 다시 적분(re-propagation)한다면 엄청난 계산 비용이 발생한다.

IMU pre-integration은 이러한 비효율을 해결하기 위한 핵심적인 기법이다. 두 키프레임 $k$와 $k+1$ 사이의 IMU 측정값들을 미리 적분하여, 두 프레임 간의 상대적인 자세, 속도, 위치 변화량($\Delta R, \Delta v, \Delta p$)을 계산해 놓는다.11 중요한 점은 이 사전 적분값이 현재의 IMU 바이어스(

$b_a, b_g$) 추정치에 대한 일차 선형 근사식으로 표현된다는 것이다. 덕분에 최적화 과정에서 바이어스 추정치가 업데이트되더라도, 간단한 선형 보정만으로 사전 적분값을 수정할 수 있어 매번 재적분할 필요가 없다.

IMU 팩터의 잔차(residual)는 이 사전 적분된 측정값과, 팩터 그래프에서 추정 중인 두 키프레임의 상태(자세, 속도, 위치, 바이어스)로부터 계산된 예측값 사이의 차이로 정의된다.11 이 잔차를 최소화하는 것은 IMU 측정값으로 기술되는 운동 모델을 로봇의 궤적이 따르도록 강제하는 역할을 한다.


이 팩터는 LIS의 scan-to-map 매칭 결과로부터 얻어지는, 두 키프레임 간의 상대 변환에 대한 기하학적 제약이다.3 LIS가 새로운 키프레임의 특징점들을 로컬 맵에 정합하고 나면, 두 키프레임 간의 최적 상대 변환 $\Delta T_{i, i+1}$이 계산된다. 이 변환 자체가 팩터 그래프에 제약 조건으로 추가된다.

잔차는 Lidar 특징점들의 기하학적 정합 오차로 정의된다. 구체적으로, 에지 특징점에 대해서는 **point-to-line 거리** 오차, 평면 특징점에 대해서는 **point-to-plane 거리** 오차의 합으로 계산된다.16

- Point-to-line 거리 $d_{ek}$는 새로운 스캔의 에지 특징점 $p^e_{i+1,k}$와, 로컬 맵에서 찾은 대응되는 에지 라인(두 점 $p^e_{i,u}, p^e_{i,v}$로 정의됨) 사이의 최단 거리다. 이는 벡터 외적을 이용하여 계산될 수 있다:
  $$
  d_{ek} = \frac{\|(p^e_{i+1,k} - p^e_{i,u}) \times (p^e_{i+1,k} - p^e_{i,v})\|}{\|p^e_{i,u} - p^e_{i,v}\|}
  $$
  16

- Point-to-plane 거리 $d_{pk}$는 새로운 스캔의 평면 특징점 $p^p_{i+1,k}$와, 로컬 맵에서 찾은 대응되는 평면 패치(세 점 $p^p_{i,u}, p^p_{i,v}, p^p_{i,w}$로 정의됨) 사이의 최단 거리다. 이는 평면의 법선 벡터를 이용하여 계산된다.16

이러한 기하학적 거리 오차들을 최소화함으로써, Lidar Odometry Factor는 매우 정확한 상대 자세 제약을 팩터 그래프에 제공한다.


이 팩터는 VIS에서 추적된 시각 특징점들의 재투영 오차(reprojection error)로부터 생성되는 제약이다.3 어떤 3D 특징점이 여러 카메라 프레임에서 관측되었을 때, 이 3D 특징점의 추정된 위치를 각 프레임의 추정된 카메라 자세로 이미지 평면에 다시 투영(reproject)할 수 있다. 이 재투영된 2D 위치와 실제 이미지에서 관측된 2D 특징점 위치 사이의 픽셀 단위 오차가 바로 재투영 오차다.

VINS-Mono의 방식을 따라, LVI-SAM은 일반적인 핀홀 카메라 모델뿐만 아니라 어안 렌즈와 같은 광각 렌즈에도 강건하게 대응하기 위해 단위 구(unit sphere) 모델 상에서 재투영 오차를 정의한다.11 즉, 2D 이미지 픽셀 좌표를 카메라 중심을 원점으로 하는 3D 단위 벡터로 변환하여, 두 단위 벡터 간의 각도 오차를 최소화하는 방식이다.

특히 LVI-SAM에서는 LiDAR로부터 깊이 정보를 얻은 특징점과 그렇지 않은 특징점을 구분하여 처리한다.3 깊이 정보가 없는 특징점은 여러 프레임에 걸쳐 관측되어야 3D 위치가 결정되지만, 깊이 정보가 있는 특징점은 단일 프레임의 관측만으로도 3D 위치가 결정되므로 훨씬 더 강한 제약 조건을 제공한다. 이는 LVI-SAM의 정확도를 높이는 중요한 요소다.


이 팩터는 시스템이 이전에 방문했던 장소를 다시 인식했을 때 생성되는 전역 제약이다. VIS가 DBoW2 14와 같은 시각적 장소 인식 기술을 사용하여 현재 프레임과 과거의 특정 프레임이 같은 장소를 촬영했다는 루프 클로저 후보를 찾아낸다. 그 후, LIS가 이 두 시점의 LiDAR 스캔 데이터를 정밀하게 매칭하여 정확한 상대 변환을 계산하고, 이것이 루프 클로저 팩터로서 팩터 그래프에 추가된다.3

이 팩터는 시간적으로 멀리 떨어진 두 개의 상태 노드 사이에 직접적인 연결(제약)을 생성한다. 이 제약을 만족시키기 위해 최적화를 수행하면, 그동안 누적되었던 오차(drift)가 그래프 전체에 걸쳐 분산되고 보정되어 전역적으로 일관된(globally consistent) 궤적과 맵을 얻게 된다.3

이 네 가지 핵심 팩터들을 하나의 팩터 그래프에서 통합하여 최적화함으로써, LVI-SAM은 각 센서의 장점을 극대화하고 단점을 상쇄하여 매우 정확하고 강인한 SLAM 성능을 달성한다. 아래 표는 각 팩터의 역할과 특성을 요약한 것이다.

**Table 1: LVI-SAM의 핵심 Factor 요약**

| Factor Type         | Source Sensor(s)     | Residual Definition (Conceptual)                         | Purpose in Optimization                                      |
| ------------------- | -------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| IMU Pre-integration | IMU                  | IMU 측정값 기반 예측과 상태 변수 간의 운동학적 불일치 11 | 고주파 모션 제약 제공, 상태 전파(propagation)                |
| Lidar Odometry      | LiDAR, IMU           | Point-to-line & Point-to-plane 거리 오차 16              | 정확한 기하학적 구조 기반의 상대 자세 제약                   |
| Visual Odometry     | Camera, IMU, (LiDAR) | 3D 특징점의 재투영 오차 (Reprojection Error) 11          | 텍스처 정보 기반의 상대 자세 제약, 스케일 제약(w/ LiDAR depth) |
| Loop Closure        | Camera, LiDAR        | 두 원거리 포즈 간의 상대 변환 오차 3                     | 전역 일관성(Global Consistency) 확보, 누적 오차 제거         |

이 표는 LVI-SAM의 복잡한 최적화 문제를 구성하는 핵심 요소들을 한눈에 파악할 수 있게 해준다. 각 센서 데이터가 어떻게 수학적 제약 조건으로 변환되어 전체 시스템의 정확도에 기여하는지를 명확히 보여줌으로써, Tightly-coupled 융합의 본질을 이해하는 데 필수적인 정보를 제공한다.


LVI-SAM은 발표 이후 수많은 연구에서 벤치마크 및 비교 대상으로 활용되며 그 성능을 입증받았다. 시스템의 강점과 한계를 명확히 이해하고, 주요 경쟁 알고리즘과의 아키텍처적 차이를 분석하는 것은 LVI-SAM의 기술적 위상을 정확히 파악하는 데 필수적이다.


LVI-SAM의 가장 큰 강점은 다양한 스케일과 환경, 여러 플랫폼에서 수집된 데이터셋을 통해 검증된 높은 수준의 **정확도**와 **강인성**이다.3 특히 이 강인성은 센서 성능이 저하되는(sensor-degraded) 시나리오에서 빛을 발한다. 예를 들어, 긴 복도나 넓은 벽과 같이 기하학적 특징이 부족한(feature-less) 환경에서는 LIS의 성능이 저하되지만, VIS가 안정적으로 오도메트리를 제공하여 시스템이 표류하는 것을 막아준다. 반대로, 어둡거나 텍스처가 없는(texture-less) 환경에서는 VIS가 실패하지만, LIS가 주변의 구조 정보를 바탕으로 정확한 위치를 추정해낸다. 이처럼 두 서브시스템이 서로의 실패를 보완하는 능력은 LVI-SAM을 실제 환경 적용에 매우 적합하게 만드는 핵심 요소다.5

성능 비교 연구에 따르면, LVI-SAM은 LiDAR-Inertial 시스템인 LIO-SAM에 비해 궤적의 RMSE(Root Mean Square Error)를 약 81% 감소시키는 등, 단일 모달리티 시스템 대비 월등한 성능 향상을 보여준다.5 이는 시각 정보가 LIO 시스템의 초기 추정 및 루프 클로저에 기여하는 바가 매우 크다는 것을 의미한다.

하지만 LVI-SAM 역시 완벽하지 않으며 몇 가지 한계를 지닌다. 원본 LVI-SAM은 보행 속도로 수집된 데이터셋에서는 우수한 성능을 보이지만, 차량과 같이 **고속으로 주행하거나 지면이 울퉁불퉁한 환경**에서는 정확도와 생성되는 맵의 품질이 저하될 수 있다는 지적이 있다.20 이러한 한계를 극복하기 위한 후속 연구들이 활발히 진행되었다.

- 한 연구에서는 LVI-SAM의 LiDAR 프론트엔드를 Super4PCS, ICP, NDT와 같은 다양한 점군 정합 알고리즘을 결합한 방식으로 개선하여, 복잡한 야지 환경에서 원본 LVI-SAM 대비 궤적의 번역 오차(translational error)를 약 4.7% 감소시키는 성과를 거두었다.20
- 또 다른 연구에서는 VIS의 전통적인 특징점 추출 방식인 Shi-Tomasi 대신, 딥러닝 기반의 특징점 검출기인 **SuperPoint**를 적용했다. 이를 통해 조명 변화나 시점 변화에 더 강인한 특징점을 추출하여, KITTI 데이터셋의 05 시퀀스에서 RMSE를 12%, M2DGR 데이터셋의 Street07 시퀀스에서 11% 감소시키는 등 성능을 향상시켰다.22

이러한 개선 연구들은 LVI-SAM이 강력한 베이스라인 프레임워크이지만, 특정 애플리케이션이나 환경에 맞춰 프론트엔드 모듈을 교체하거나 개선함으로써 성능을 더욱 끌어올릴 수 있는 잠재력을 가지고 있음을 보여준다.


LVI-SLAM 분야에서 LVI-SAM의 주요 경쟁자로는 R2LIVE와 그 후속 버전인 R3LIVE가 꼽힌다. 이들 시스템은 사용하는 센서는 유사하지만, 상태 추정을 위한 근본적인 철학과 아키텍처에서 큰 차이를 보인다. 이 차이를 이해하는 것은 특정 목적에 맞는 SLAM 시스템을 선택하는 데 중요한 기준이 된다.


- **핵심 방식:** LVI-SAM은 **팩터 그래프 기반의 스무딩 및 매핑(Smoothing and Mapping)** 방식을 채택한다.12 이는 일정 시간 동안의 데이터(슬라이딩 윈도우)를 모아, 그 윈도우 내의 모든 측정값과 상태 변수들을 포함하는 하나의 거대한 비선형 최적화 문제를 푸는 방식이다.
- **장점:** 여러 시간대의 측정값을 동시에 고려하여 상태를 추정하므로, 시간적으로 일관된 최적의 해를 찾을 수 있다. 따라서 일반적으로 필터 기반 방식보다 더 높은 **궤적 정확도**와 **전역 일관성(global consistency)**을 달성할 수 있다. 또한, 루프 클로저와 같이 시간 순서에 맞지 않는 비순차적인 제약 조건을 팩터 그래프에 자연스럽게 추가하여 전역 오차를 보정하는 데 매우 유리하다.
- **단점:** 슬라이딩 윈도우의 크기가 커지거나 제약 조건이 많아질수록 계산 비용이 급격히 증가할 수 있다. 이로 인해 실시간성을 유지하기 위한 계산 복잡도 관리가 중요하다.


- **핵심 방식:** R2LIVE와 R3LIVE는 **필터 기반(Filter-based)** 접근법, 특히 **Error-State Iterated Kalman Filter (ESIKF)**를 핵심 추정기로 사용한다.23 이 방식은 새로운 측정값이 들어올 때마다 상태를 순차적으로 예측(prediction)하고 보정(update)하는 재귀적인 구조를 가진다.
- **시스템 구조:** 이들 시스템은 LIO와 VIO 모듈이 병렬적으로 실행되는 독특한 구조를 가진다. R2LIVE에서는 LIO 모듈이 주로 기하학적 구조 정보를 제공하고, VIO 모듈은 시각적 특징점을 이용해 백엔드 팩터 그래프 최적화에 제약 조건을 추가한다.23 R3LIVE는 여기서 더 나아가, FAST-LIO 기반의 LIO 서브시스템이 맵의 기하학적 구조(3D 포인트 위치)를 만들면, VIO 서브시스템은 이 맵에 이미지의 RGB 색상을 입히는(texturing) 역할에 집중한다.26
- **장점:** 계산량이 일정하게 유지되어 **실시간 성능**이 매우 우수하다. 이러한 효율성 덕분에 R3LIVE는 실시간으로 밀도 높은(dense) **RGB 컬러 맵**을 생성하는 데 특화될 수 있었다.26
- **단점:** 필터 방식은 한번 추정된 과거 상태를 다시 최적화하지 않으므로(marginalization), 최적화 기반 방식에 비해 이론적인 정확도나 전역 일관성 확보가 더 어렵다. 한번 발생한 추정 오차를 나중에 수정하기가 까다롭다.

결론적으로, LVI-SAM은 '전역적 궤적 정확성'을 최우선 목표로 하는 시스템이며, R2LIVE/R3LIVE는 '실시간 밀도 높은 맵핑'과 시각화에 더 큰 비중을 두는, 서로 다른 철학을 가진 시스템이라고 볼 수 있다.25 SLAM 연구자나 개발자는 자신의 애플리케이션이 요구하는 것이 최고 수준의 궤적 정확도인지, 아니면 실시간으로 생성되는 풍부한 시각적 맵인지에 따라 적합한 시스템을 선택해야 한다.

아래 표는 세 가지 주요 LVI-SLAM 알고리즘의 핵심적인 특징을 비교하여 각 시스템의 본질적인 특성을 명확히 보여준다.

**Table 2: 주요 LVI-SLAM 알고리즘 비교**

| Feature                | LVI-SAM                                               | R2LIVE                                                   | R3LIVE                                                   |
| ---------------------- | ----------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------- |
| **Core Estimator**     | Optimization-based (Factor Graph Smoothing) 14        | Filter-based (ESIKF) + Factor Graph Opt. 23              | Filter-based (ESIKF) 26                                  |
| **Fusion Strategy**    | Tightly-coupled VIS & LIS subsystems 3                | Parallel LIO & VIO, LIO provides geometry 23             | Parallel LIO & VIO, LIO for geometry, VIO for texture 25 |
| **Map Representation** | Feature-based Point Cloud 12                          | Dense Point Cloud (Geometry) 23                          | Dense RGB-colored Point Cloud / Mesh 26                  |
| **Primary Strength**   | Global consistency & robustness in degraded scenes 14 | Real-time state estimation 23                            | Real-time, high-fidelity RGB mapping 26                  |
| **VIO Role**           | Provides odometry constraints & loop candidates 3     | Provides visual landmark constraints to backend graph 25 | Renders color on LIO map via photometric error 28        |


성공적인 연구 프로젝트가 학술적 성과를 넘어 실제 기술로 자리 잡기 위해서는 강력한 오픈소스 생태계의 지원이 필수적이다. LVI-SAM은 이러한 과정을 보여주는 전형적인 사례로, 활발한 커뮤니티 활동을 통해 '살아있는 기술'로 진화하고 있다.


LVI-SAM은 GitHub를 통해 소스 코드가 공개되었으며 15, ROS(Robot Operating System) 환경(Kinetic, Melodic 테스트 완료)에서 동작하도록 설계되었다. 핵심적인 최적화 부분은 GTSAM 라이브러리를, 비선형 문제 해결에는 Ceres Solver를 의존성으로 가진다.15

이러한 오픈소스화는 LVI-SAM이 LVI-SLAM 분야의 표준 베이스라인 중 하나로 빠르게 자리매김하는 데 결정적인 역할을 했다. 전 세계의 수많은 연구자들과 개발자들은 LVI-SAM의 코드를 직접 분석하고, 자신의 하드웨어에 적용하며, 이를 기반으로 새로운 아이디어를 검증하고 확장하는 연구를 수행할 수 있게 되었다.20 이는 LVI-SAM이 단지 논문으로만 존재하는 이론이 아니라, 실제 문제를 해결하기 위한 강력하고 실용적인 도구로 채택되었음을 증명하는 가장 확실한 증거다.


LVI-SAM의 공식 저장소 외에도, 수많은 포크(fork) 버전이 등장하며 커뮤니티에 의해 자발적으로 개선되고 있다. 이러한 포크들의 수정 내용을 분석하면, 원본 알고리즘의 학술적 우수성과는 별개로, 실제 현장에서의 적용에 존재했던 간극을 엿볼 수 있다. 커뮤니티는 이 간극을 메우기 위해 실용적인 개선들을 이루어냈다.

- LVI-SAM-Easyused 30:

   이 포크는 이름에서 알 수 있듯 '사용 편의성' 개선에 집중했다. 원본 LVI-SAM 코드에서는 카메라, LiDAR, IMU 간의 상대적인 위치와 자세를 나타내는 외재 변수(extrinsic parameters) 중 상당수가 코드 내에 하드코딩되어 있어, 다른 센서 조합으로 변경하기가 매우 까다로웠다. LVI-SAM-Easyused는 이러한 변수들을 모두 설정 파일인 YAML 파일로 분리하여 사용자가 자신의 하드웨어에 맞게 쉽게 수정할 수 있도록 했다. 또한, 버그가 수정된 최신 버전의 LIO-SAM 코드를 통합하여 시스템의 안정성을 높였다.

- lviorf 31:

   이 포크는 '범용성'을 극대화하는 방향으로 발전했다. 원본 LVI-SAM의 LiDAR 특징점 추출 모듈은 특정 종류의 LiDAR(e.g., Velodyne)에 최적화되어 있었는데, lviorf는 이 모듈을 제거하여 다양한 LiDAR(e.g., Robosense)에 더 쉽게 적응할 수 있도록 했다. 뿐만 아니라, 9축 IMU만 지원하던 것을 6축 IMU도 지원하도록 확장하고, 50Hz나 100Hz 같은 저주파 IMU 및 다양한 GPS 장치와의 연동성도 개선했다.

- LVI-SAM-Simple 32:

   이 포크 역시 외재 변수 설정을 YAML 파일로 추출하여 편의성을 높였으며, 원본 코드에서 고려되지 않았던 LiDAR와 카메라 간의 미세한 변위(translation)를 보정하는 기능을 추가하여 정확도를 개선했다.

- LVI-SAM_detailed_comments 33:

   이 프로젝트는 알고리즘 자체의 수정보다는 '교육'과 '분석'에 초점을 맞췄다. 코드의 모든 라인에 상세한 중국어 주석을 추가하고, 복잡한 의존성 설치 과정을 생략할 수 있는 Docker 이미지를 제공함으로써, LVI-SAM을 처음 접하는 연구자들이 알고리즘을 더 쉽게 이해하고 분석할 수 있도록 진입 장벽을 크게 낮췄다.

이러한 포크들의 공통적인 개선 방향(외재 변수 설정의 용이성, 다양한 센서 지원)은 LVI-SAM의 핵심 알고리즘은 매우 강력하지만, 실제 현장의 다양한 하드웨어에 그대로 적용하기에는 불편함이 있었음을 시사한다. 커뮤니티의 이러한 자발적인 개선 노력은 LVI-SAM이 이론적 프레임워크를 넘어, 실제 로봇 시스템에 탑재될 수 있는 실용적인 기술로 진화하는 데 크게 기여했다.


LVI-SAM이 제공하는 강인하고 정확한 실시간 위치 추정 및 3차원 지도 작성 능력은 자율적으로 움직이는 모든 시스템의 근간이 된다.

- **자율주행 자동차:** 도심의 빌딩 숲이나 터널과 같이 GPS 신호가 끊기는 음영 지역에서 LVI-SAM은 차량의 위치를 끊김 없이 추정하는 핵심적인 역할을 한다. 생성된 고정밀 3D 포인트 클라우드 맵은 경로 계획, 장애물 회피, 차선 유지 등 자율주행에 필요한 주변 환경 인지의 기반이 된다.34
- **무인 항공기(드론):** GPS가 없는 실내 환경이나 교량 하부, 산림 등 복잡한 지형을 비행하는 드론의 자율 항법에 LVI-SAM은 필수적이다.12 드론에 탑재된 경량 LiDAR, 카메라, IMU를 이용해 실시간으로 자신의 위치를 파악하고 주변 환경 지도를 작성하여 안전한 비행 경로를 생성할 수 있다.
- **모바일 로봇:** 공장, 물류창고, 농장 등 다양한 환경에서 운용되는 모바일 로봇의 자율 이동 및 작업 수행을 위해 정확한 위치 정보는 필수적이다.36 LVI-SAM은 로봇이 자신의 위치를 알고, 작업 환경의 3D 지도를 생성하여 효율적인 경로 계획 및 탐사(exploration)를 수행할 수 있도록 지원한다.12

이처럼 LVI-SAM은 단순히 학술적인 성과에 그치지 않고, 다양한 실제 응용 분야에서 자율 시스템의 '눈'과 '균형 감각' 역할을 수행하는 핵심 기반 기술로 널리 활용되고 있다.



LVI-SAM은 다중 센서 융합 SLAM의 역사에서 중요한 이정표를 세운 알고리즘으로 평가받는다. 그 기술적 기여는 다음과 같이 요약할 수 있다.

첫째, LVI-SAM은 당시 최고 수준의 LIO(LIO-SAM)와 VIO(VINS-Mono)를 **Tightly-coupled 방식으로 성공적으로 통합**하여, 각 센서의 성능이 저하되는 도전적인 환경에서도 상호 보완을 통해 강인한 성능을 유지하는 새로운 기준을 제시했다. 이는 텍스처가 부족한 환경과 기하학적 특징이 없는 환경 모두에서 안정적인 동작을 가능하게 했다.

둘째, **팩터 그래프 최적화 프레임워크**의 유연성과 확장성을 입증했다. LiDAR의 기하학적 제약, 카메라의 시각적 제약, IMU의 운동학적 제약을 하나의 통합된 비선형 최적화 문제로 공식화함으로써, 다중 센서 정보를 체계적이고 정확하게 융합하는 방법론의 효용성을 보여주었다.

셋째, 강력한 **오픈소스 베이스라인**으로서 후속 연구의 발전에 지대한 공헌을 했다. LVI-SAM의 공개된 코드는 전 세계 수많은 연구자들에게 LVI-SLAM을 연구하고 개발하기 위한 견고한 발판을 제공했으며, 이를 기반으로 한 수많은 확장 및 개선 연구를 촉발시켰다.

이러한 기여를 통해 LVI-SAM은 LVI-SLAM 분야의 '기하학적 정밀도' 시대를 정립한 대표적인 알고리즘으로 자리매김했다. 그러나 기술의 발전은 멈추지 않으며, LVI-SAM이 해결한 문제는 이제 더 고차원적인 인공지능 작업을 위한 기반이 되고 있다. LVI-SAM의 역할은 최종 목표가 아니라, 차세대 SLAM 패러다임을 위한 '안정적인 자세 추정 엔진(Pose Engine)'으로 진화하고 있다.


기존의 SLAM은 주로 점, 선, 면과 같은 기하학적 정보만을 사용하여 환경을 이해했다. 이는 로봇이 '어디에 있는지'와 '주변의 구조가 어떤지'는 알 수 있지만, '주변에 있는 것이 무엇인지'는 이해하지 못하는 한계를 가진다. 이 한계를 극복하기 위해, 딥러닝 기반의 시맨틱 분할(semantic segmentation) 기술을 SLAM에 융합하는 연구가 차세대 SLAM의 주요 흐름으로 부상하고 있다.13

시맨틱 정보의 융합은 두 가지 중요한 이점을 제공한다.

1. **동적 환경에서의 강인성 향상:** 시맨틱 분할을 통해 이미지나 포인트 클라우드 내에서 사람, 자동차와 같이 움직일 가능성이 있는 동적 객체(dynamic objects)를 식별할 수 있다. SLAM 시스템은 이러한 동적 객체들을 특징점 추출이나 스캔 매칭 과정에서 제외함으로써, 움직이는 물체에 의해 위치 추정이 오염되는 것을 방지하고 정적인 배경만을 사용하여 더욱 안정적이고 정확한 자세를 추정할 수 있다.38
2. **고차원적 환경 이해:** 맵에 의미론적 정보를 부여함으로써(e.g., '이 영역은 도로', '이 객체는 건물', '저것은 문'), 로봇은 단순한 기하학적 지도를 넘어 환경을 인간과 유사한 수준으로 이해하게 된다. 이는 '문으로 이동하라'와 같은 고차원적인 명령을 수행하거나, 특정 종류의 객체와만 상호작용하는 등 지능적인 작업 수행의 기반이 된다.39

PSMD-SLAM 37과 같이 LVI-SAM 프레임워크 위에 Panoptic Segmentation(객체 인스턴스와 배경 클래스를 동시에 분할)을 적용하여 동적 환경에서의 성능을 개선한 연구는 이러한 추세를 명확히 보여준다. LVI-SAM이 제공하는 정확한 자세와 기하학적 맵은, 딥러닝으로 추출된 시맨틱 정보를 3D 공간에 일관되게 정합(align)하기 위한 필수적인 전제 조건이 된다. 즉, LVI-SAM은 시맨틱 SLAM을 위한 견고한 '뼈대' 역할을 수행하고 있다.


전통적인 SLAM 시스템이 생성하는 맵은 주로 포인트 클라우드, 복셀 그리드, 또는 메시(mesh) 형태였다. 이러한 표현 방식은 로봇의 네비게이션이나 기하학적 분석에는 충분하지만, 가상현실(VR), 증강현실(AR), 디지털 트윈(Digital Twin)과 같이 인간이 소비하기 위한 사실적인 시각화에는 한계가 있다.12

최근 컴퓨터 그래픽스와 비전 분야에서 등장한 NeRF(Neural Radiance Fields)와 **3D Gaussian Splatting (3DGS)**는 이러한 맵 표현의 패러다임을 바꾸고 있다. 특히 3DGS는 3D 공간을 미분 가능한 수많은 3D 가우시안(Gaussian)들의 집합으로 표현하는 방식으로, NeRF에 비해 훈련 속도와 렌더링 속도가 매우 빠르면서도 사진처럼 사실적인(photorealistic) 품질의 이미지를 실시간으로 생성할 수 있어 큰 주목을 받고 있다.40

이러한 3DGS 기술을 SLAM과 결합하여, 로봇이 이동하면서 실시간으로 주변 환경의 '사실적인 디지털 트윈'을 구축하는 연구가 활발히 진행되고 있다. LVI-GS 42와 같은 연구는 이러한 흐름의 선두에 있다. 이 시스템들은 LVI-SAM과 같은 강력한 Tightly-coupled LVI 오도메트리를 백본(backbone)으로 사용하여 매우 정확하고 안정적인 카메라 자세를 실시간으로 추정한다. 그리고 이 정확한 자세 정보를 기반으로, 입력되는 이미지의 색상 정보를 3D 가우시안 맵에 투사하고 최적화하여 사실적인 3D 맵을 점진적으로 구축한다.

이러한 진화는 LVI-SLAM의 역할이 '자율 시스템의 위치 추정'이라는 전통적인 영역을 넘어, '현실 세계를 디지털 공간에 사실적으로 복제하는' 창조적인 도구로 확장되고 있음을 시사한다. LVI-SAM이 정립한 기하학적 정확성은 이제 더 풍부하고, 더 의미 있고, 더 사실적인 세계를 구축하기 위한 필수적인 디딤돌이 되고 있다. 결국, LVI-SAM과 같은 시스템이 해결한 '어디에 있는가(Where am I?)'와 '주변의 구조는 무엇인가(What is the structure?)'라는 문제는, 이제 '주변에 있는 것이 무엇인가(What is around me?)'와 '주변이 실제로는 어떻게 보이는가(What does it look like?)'라는 더 복잡하고 고차원적인 질문을 풀기 위한 핵심적인 입력값이 되어가고 있다.


1. LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/362412641_LVI-SAM_Tightly-coupled_Lidar-Visual-Inertial_Odometry_via_Smoothing_and_Mapping
2. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 23, 2025, https://www.mdpi.com/1424-8220/25/5/1447
3. LVI-SAM: Tightly-coupled Lidar-Visual- Inertial Odometry via Smoothing and Mapping - DSpace@MIT, accessed July 23, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/144047/2104.10831.pdf?sequence=2&isAllowed=y
4. LOAM: Lidar Odometry and Mapping in Real-time - Carnegie Mellon University's Robotics Institute, accessed July 23, 2025, https://www.ri.cmu.edu/pub_files/2014/7/Ji_LidarMapping_RSS2014_v8.pdf
5. LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 23, 2025, https://arxiv.org/html/2504.20380v1
6. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 23, 2025, https://arxiv.org/html/2408.14035v1
7. Visual SLAM: What Are the Current Trends and What to Expect? - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/22/23/9297
8. (PDF) PC-VINS-Mono: A Robust Mono Visual-Inertial Odometry with Photometric Calibration, accessed July 23, 2025, https://www.researchgate.net/publication/330526039_PC-VINS-Mono_A_Robust_Mono_Visual-Inertial_Odometry_with_Photometric_Calibration
9. Monocular Visual-Inertial Depth Estimation - Vladlen Koltun, accessed July 23, 2025, http://vladlen.info/papers/VID.pdf
10. VINS Mono Body-Of-Knowledge | ai Werkstatt, accessed July 23, 2025, https://www.aiwerkstatt.com/wp-content/uploads/2020/08/VINS-Mono-Body-Of-Knowledge.pdf
11. Technical Report: VINS-Mono: A Robust and Versatile ... - GitHub, accessed July 23, 2025, https://raw.githubusercontent.com/HKUST-Aerial-Robotics/VINS-Mono/master/support_files/paper/tro_technical_report.pdf
12. Tightly-Coupled LiDAR-Visual-Inertial SLAM and Large-Scale Volumetric Occupancy Mapping - arXiv, accessed July 23, 2025, https://arxiv.org/html/2403.02280v1
13. [2106.11516] SA-LOAM: Semantic-aided LiDAR SLAM with Loop Closure - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2106.11516
14. (PDF) LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping, accessed July 23, 2025, https://www.researchgate.net/publication/351063772_LVI-SAM_Tightly-coupled_Lidar-Visual-Inertial_Odometry_via_Smoothing_and_Mapping
15. LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 23, 2025, https://github.com/TixiaoShan/LVI-SAM
16. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via ... - AMiner, accessed July 23, 2025, https://static.aminer.cn/storage/pdf/arxiv/20/2007/2007.00258.pdf
17. Code for VINS-Mono is now available on GitHub – HDJI / lab, accessed July 23, 2025, https://hdji.hkust.edu.hk/code-for-vins-mono-is-now-available-on-github/
18. A Localization and Mapping Algorithm Based on Improved LVI-SAM for Vehicles in Field Environments - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/23/7/3744
19. An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM and Scan Context for Underground Coalmine - PMC - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8778426/
20. A Localization and Mapping Algorithm Based on Improved LVI-SAM for Vehicles in Field Environments - PMC, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10098548/
21. A Localization and Mapping Algorithm Based on Improved LVI-SAM for Vehicles in Field Environments - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/369831328_A_Localization_and_Mapping_Algorithm_Based_on_Improved_LVI-SAM_for_Vehicles_in_Field_Environments
22. SLAM Algorithm for Mobile Robots Based on Improved LVI-SAM in Complex Environments, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11598613/
23. R2LIVE: A Robust, Real-time, LiDAR-Inertial-Visual tightly-coupled state Estimator and mapping - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/uploads/r2live.pdf
24. arxiv.org, accessed July 23, 2025, https://arxiv.org/abs/2102.12400
25. SR-LIVO: LiDAR-Inertial-Visual Odometry and Mapping with Sweep Reconstruction - arXiv, accessed July 23, 2025, https://arxiv.org/html/2312.16800v1
26. hku-mars/r3live: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package - GitHub, accessed July 23, 2025, https://github.com/hku-mars/r3live
27. R$^3$LIVE: A Robust, Real-time, RGB-colored, LiDAR ... - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/publication/paper_r3live/
28. A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/uploads/r3live.pdf
29. R3LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/354649365_R3LIVE_A_Robust_Real-time_RGB-colored_LiDAR-Inertial-Visual_tightly-coupled_state_Estimation_and_mapping_package
30. Cc19245/LVI-SAM-Easyused - GitHub, accessed July 23, 2025, https://github.com/Cc19245/LVI-SAM-Easyused
31. YJZLuckyBoy/lviorf: This repo is modified based on lvi-sam and liorf, which remove the feature extraction module and makes it easier to adapt different sensor. - GitHub, accessed July 23, 2025, https://github.com/YJZLuckyBoy/lviorf
32. YJZLuckyBoy/LVI-SAM-Simple - GitHub, accessed July 23, 2025, https://github.com/YJZLuckyBoy/LVI-SAM-Simple
33. electech6/LVI-SAM_detailed_comments: LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 23, 2025, https://github.com/electech6/LVI-SAM_detailed_comments
34. The principle of the LVI-SAM algorithm. - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/The-principle-of-the-LVI-SAM-algorithm_fig1_369831328
35. LVI-Fusion: A Robust Lidar-Visual-Inertial SLAM Scheme - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/16/9/1524
36. SLAM Algorithm for Mobile Robots Based on Improved LVI-SAM in Complex Environments, accessed July 23, 2025, https://www.mdpi.com/1424-8220/24/22/7214
37. PSMD-SLAM: Panoptic Segmentation-Aided Multi-Sensor Fusion Simultaneous Localization and Mapping in Dynamic Scenes - MDPI, accessed July 23, 2025, https://www.mdpi.com/2076-3417/14/9/3843
38. (PDF) LVI-Fusion: A Robust Lidar-Visual-Inertial SLAM Scheme - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/380148723_LVI-Fusion_A_Robust_Lidar-Visual-Inertial_SLAM_Scheme
39. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 23, 2025, https://arxiv.org/html/2408.14035v2
40. LVI-GS: Tightly-coupled LiDAR-Visual-Inertial SLAM using 3D Gaussian Splatting - arXiv, accessed July 23, 2025, https://arxiv.org/html/2411.02703v1
41. (PDF) VIGS SLAM: IMU-based Large-Scale 3D Gaussian Splatting SLAM - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/388354469_VIGS_SLAM_IMU-based_Large-Scale_3D_Gaussian_Splatting_SLAM
42. [Literature Review] LVI-GS: Tightly-coupled LiDAR-Visual-Inertial SLAM using 3D Gaussian Splatting - Moonlight, accessed July 23, 2025, https://www.themoonlight.io/en/review/lvi-gs-tightly-coupled-lidar-visual-inertial-slam-using-3d-gaussian-splatting
43. LVI-GS: Tightly-coupled LiDAR-Visual-Inertial SLAM using 3D Gaussian Splatting - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2411.02703




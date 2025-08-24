[SLAM 동향](./index.md)
# 라이다 기반 최신 SLAM 기술 고찰



SLAM(Simultaneous Localization and Mapping)은 로봇 공학의 핵심 과제 중 하나로, 로봇이 이전에 탐사한 적 없는 미지의 환경에서 자신의 위치와 방향을 실시간으로 추정함과 동시에, 주변 환경에 대한 지도를 작성하는 기술을 의미한다.1 이 기술은 자율주행 자동차, 무인 항공기(UAV), 실내 서비스 로봇 등 다양한 자율 시스템의 기반이 된다.

초기 SLAM 연구는 카메라를 주 센서로 사용하는 시각(Visual) SLAM에 집중되었으나, 이는 조명 변화에 민감하고 텍스처가 부족한 환경(예: 흰 벽, 어두운 복도)에서 특징점 추출에 어려움을 겪는 본질적인 한계를 가지고 있었다.4 이러한 배경에서 라이다(LiDAR) 센서가 대안으로 부상했다. 라이다는 레이저 펄스를 사용하여 주변 환경과의 거리를 직접적이고 정밀하게 측정하므로, 조명 조건과 무관하게 안정적인 3D 공간 정보를 획득할 수 있다.4 이 특성 덕분에 라이다는 현대 SLAM 시스템에서 가장 신뢰도 높은 주력 센서로 자리 잡게 되었다.


현대 라이다 SLAM 기술의 발전은 2014년 LOAM(Lidar Odometry and Mapping)의 등장을 기점으로 중요한 전환점을 맞이했다. LOAM 이전의 SLAM 알고리즘들은 주로 필터링이나 전체 스캔 매칭에 의존하여 실시간성과 정확성 사이에서 절충점을 찾아야 했다.1 LOAM은 포인트 클라우드에서 기하학적 특징(feature)을 추출하고, 이를 기반으로 오도메트리(Odometry)와 맵핑(Mapping)을 분리하여 처리하는 혁신적인 구조를 제시함으로써, 고성능 SLAM의 새로운 기준을 세웠다.

LOAM 이후의 기술 발전은 크게 세 가지 주요 경로로 분화하며 심화되었다.

1. **특화 및 경량화 (Specialization & Lightweighting):** LOAM의 기본 구조를 유지하면서 특정 플랫폼이나 환경에 맞춰 알고리즘을 최적화하고 경량화하는 방향이다. 대표적으로 지상 로봇(UGV) 환경에 특화된 LeGO-LOAM이 있다.
2. **강결합 센서 융합 (Tightly-Coupled Sensor Fusion) - 그래프 최적화:** 관성 측정 장치(IMU)와 같은 이종(heterogeneous) 센서의 측정값을 팩터 그래프(Factor Graph)라는 최적화 프레임워크 내에서 라이다 데이터와 강하게 결합(tightly-coupled)하여 전역 일관성(global consistency)과 강건성을 극대화하는 방향이다. LIO-SAM이 이 패러다임의 대표적인 예이다.
3. **강결합 센서 융합 (Tightly-Coupled Sensor Fusion) - 필터링:** 칼만 필터(Kalman Filter)를 기반으로 다중 센서를 강결합하여, 그래프 최적화 방식보다 월등한 계산 효율성과 실시간성을 확보하는 데 중점을 두는 방향이다. FAST-LIO 계열이 이 접근법을 주도하고 있다.

이러한 기술적 분화 과정은 단순히 성능을 개선하는 것을 넘어, SLAM 문제에 대한 접근 철학 자체의 변화를 보여준다. 초기 LOAM의 혁신이 프론트엔드(특징 추출)와 백엔드(맵핑)의 '구조적 분리'에 있었다면, LIO-SAM과 FAST-LIO와 같은 후속 기술들의 비약적인 발전은 백엔드 추정 엔진(estimation engine) 자체를 팩터 그래프나 칼만 필터와 같은 고도화된 프레임워크로 전면 교체하면서 이루어졌다. 이는 SLAM 성능 향상의 무게 중심이 프론트엔드의 점진적 개선에서 백엔드의 근본적인 재설계로 이동했음을 시사한다. 즉, 현대 SLAM 아키텍처 설계에서 가장 핵심적인 결정은 어떤 특징을 추출할 것인가를 넘어, 어떤 백엔드 최적화 철학(그래프 기반 전역 일관성 vs. 필터 기반 실시간 추정)을 채택할 것인가가 되었다.



- **발표:** Zhang, J. and Singh, S. (2014) LOAM: Lidar Odometry and Mapping in Real-Time. Robotics: Science and Systems Conference (RSS).8
- **의의:** LOAM은 고정밀 IMU나 고가의 외부 측정 장비 없이도 낮은 드리프트(low-drift)와 낮은 계산 복잡도를 동시에 달성한 최초의 실시간 3D 라이다 SLAM 프레임워크 중 하나로 평가받는다.6 이 알고리즘의 등장은 이후 발표된 거의 모든 특징 기반 라이다 SLAM 연구의 성능 비교 기준점이자 기술적 출발점이 되었다.


LOAM의 가장 큰 혁신은 실시간성과 정확성이라는 상충하는 목표를 달성하기 위해 SLAM 문제를 두 개의 독립적인 서브 모듈로 분리한 것이다.9

- **Lidar Odometry (고주파, 저정밀):** 이 모듈은 약 10Hz의 높은 주기로 동작한다. 연속된 두 개의 라이다 스캔(scan-to-scan) 사이의 상대적인 움직임을 빠르게 추정하여 로봇의 속도를 계산한다. 이 과정은 정확도는 다소 낮지만, 실시간으로 로봇의 모션을 추적하는 데 필수적인 역할을 한다.9
- **Lidar Mapping (저주파, 고정밀):** 이 모듈은 약 1Hz의 낮은 주기로 동작한다. Lidar Odometry 모듈에서 추정된 포즈를 초기값으로 사용하여, 현재 스캔을 지금까지 누적된 전역 맵(scan-to-map)에 정합(registration)한다. 이 과정은 계산 비용이 높지만, 오도메트리에서 발생한 오차를 보정하고 드리프트를 최소화하여 전체 시스템의 정확도를 높인다.12

이러한 이중 알고리즘 구조는 계산적으로 가벼운 오도메트리가 즉각적인 모션 추정을 제공하고, 무겁지만 정확한 맵핑 모듈이 이를 주기적으로 보정하는 형태로 상호 보완한다. 이로써 LOAM은 당시 기술로는 달성하기 어려웠던 실시간성과 정확성을 동시에 확보할 수 있었다.


LOAM은 전체 포인트 클라우드를 직접 사용하는 대신, 기하학적으로 의미 있는 소수의 점들, 즉 '특징점'만을 추출하여 계산 효율을 극대화했다.

- **특징점 추출:** 포인트 클라우드 내 각 점에 대해 주변 점들과의 관계를 분석하여 지역적 표면의 곡률(local smoothness)을 계산한다. 이 곡률 값이 특정 임계값보다 크면, 날카로운 모서리에 해당한다고 판단하여 **엣지(edge) 특징점**으로 분류한다. 반대로 곡률 값이 임계값보다 작으면, 평평한 표면에 해당한다고 보아 **평면(planar) 특징점**으로 분류한다.13
- **특징점 정합:** 최적의 변환 행렬(로봇의 움직임)을 찾기 위해, 현재 스캔의 엣지 특징점과 맵 상의 대응되는 엣지 라인 사이의 거리를 최소화하고, 동시에 현재 스캔의 평면 특징점과 맵 상의 대응되는 평면 패치 사이의 거리를 최소화하는 비선형 최적화 문제를 푼다.12
- **모션 왜곡 보정 (Motion Distortion Compensation):** 3D 라이다 센서는 한 바퀴(360도)를 스캔하는 동안에도 로봇이 계속 움직이기 때문에, 스캔 시작 시점의 포인트와 종료 시점의 포인트는 서로 다른 위치에서 측정된다. 이로 인해 포인트 클라우드가 왜곡되는 현상이 발생한다. LOAM은 Lidar Odometry에서 추정된 속도를 이용해 각 포인트가 측정된 시간을 기준으로 위치를 보정하여 이 왜곡을 제거한다.11


- **장점:** 당시 널리 사용되던 ICP(Iterative Closest Point) 기반의 방법들에 비해 월등히 높은 계산 효율성과 낮은 드리프트 성능을 보였다.13
- **해결한 문제:** 고사양 컴퓨팅 자원 없이도 6-DOF(자유도) 라이다 SLAM을 실시간으로 구동할 수 있는 가능성을 열었으며, 특징 기반 접근법의 효용성을 입증했다.


- **루프 클로저 부재 (No Loop Closure):** 원본 LOAM 알고리즘은 로봇이 과거에 방문했던 장소를 다시 인식하여 누적된 오차를 보정하는 루프 클로저(loop closure) 기능이 없다.11 이로 인해 장시간 또는 장거리 주행 시 오차(drift)가 시간에 따라 계속해서 누적되는 한계가 있다.
- **특징이 부족한 환경에서의 취약성:** 긴 복도, 넓은 공터, 터널과 같이 뚜렷한 엣지나 평면이 없는 기하학적으로 퇴화된(degenerate) 환경에서는 안정적인 특징점 추출이 어려워져 성능이 급격히 저하될 수 있다.
- **동적 환경 및 노이즈에 대한 민감성:** 알고리즘이 정적 환경을 가정하기 때문에, 움직이는 객체가 많거나 풀, 나뭇잎처럼 불규칙하고 신뢰도 낮은 객체에서 추출된 특징점은 오히려 정합 과정에서 오차를 유발하는 노이즈로 작용할 수 있다.15



- **발표:** Shan, T., & Englot, B. (2018). LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain. IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS).16
- **참고 관계:** LeGO-LOAM은 LOAM을 직접적으로 계승하고 확장한 알고리즘이다. 특히 무인 지상 차량(Unmanned Ground Vehicle, UGV)이 실제 환경에서 직면하는 두 가지 주요 문제, 즉 제한된 컴퓨팅 자원과 지면에서 발생하는 측정 노이즈 문제를 해결하는 데 초점을 맞추어 개발되었다.15


LeGO-LOAM의 핵심은 LOAM의 프론트엔드 단을 UGV 환경에 맞게 개선하여 강건성과 효율성을 높인 것이다.

- **지면 분리 (Ground Segmentation):** 특징점을 추출하기 전에, 라이다 포인트 클라우드를 거리 이미지(range image)로 투영하여 지면으로 판단되는 점들을 명시적으로 분리한다. 이 과정은 두 가지 중요한 이점을 제공한다. 첫째, 풀이나 낮은 장애물과 같이 신뢰도가 낮은 특징점을 유발할 수 있는 지면 포인트를 정합 과정에서 배제하여 정확도를 높인다. 둘째, 전체 포인트 클라우드를 지면과 비-지면 객체로 나누어 처리함으로써 후속 연산의 복잡도를 줄인다.15
- **경량화 (Lightweighting):** 지면 분리 후, 객체들을 클러스터링하고 포인트 수가 너무 적은 작은 클러스터(예: 나뭇잎, 작은 돌 등)를 제거한다. 이 과정을 통해 처리해야 할 특징점의 수를 LOAM 대비 획기적으로 감소시킨다(엣지 특징 최대 68%, 평면 특징 최대 72% 감소).15 이러한 경량화 덕분에 NVIDIA Jetson TX2와 같은 저전력 임베디드 시스템에서도 실시간 구동이 가능해졌다.15


- **2단계 Levenberg-Marquardt (L-M) 최적화:** LeGO-LOAM은 LOAM의 단일 6-DOF 최적화 과정을 계산적으로 더 효율적인 두 단계로 분리했다.15

  1. **1단계:** 지면에서 추출된 평면 특징점만을 사용하여 상대적으로 추정이 용이한 수직 이동(tz)과 roll(θroll), pitch(θpitch) 회전을 먼저 계산한다.

  2. 2단계: 1단계에서 얻은 변환 값을 고정한 상태에서, 지면을 제외한 나머지 객체들에서 추출된 엣지 특징점들을 사용하여 수평 이동(tx,ty)과 yaw(θyaw) 회전을 추정한다.

     이 접근법은 전체 6-DOF 문제를 더 작은 두 개의 문제로 나누어 푸는 방식으로, 기존 방식과 유사한 정확도를 유지하면서도 계산 시간을 약 35% 절감하는 효과를 가져온다.15

- **루프 클로저 통합:** LOAM의 가장 큰 한계였던 루프 클로저 기능을 통합하여 장시간 주행 시 누적되는 오차를 보정할 수 있게 되었다.15 하지만 초기 구현은 단순한 ICP 기반 방식으로, 로봇의 경로 오차가 클 경우 루프를 감지하지 못하는 경우가 많았다.16 이 한계를 극복하기 위해, Scan Context와 같이 더 발전된 장소 인식(place recognition) 기술을 결합한 SC-LeGO-LOAM과 같은 후속 연구들이 등장했다.20


- **장점:** UGV가 운용되는 실제 지형 환경에서 LOAM 대비 높은 정확도와 강건성을 보이며, 동시에 월등한 계산 효율성을 달성했다.15
- **해결한 문제:** LOAM의 루프 클로저 부재 문제를 해결하고, 저사양 하드웨어에서도 안정적인 실시간 성능을 확보하여 라이다 SLAM의 적용 범위를 넓혔다.


- **지면 존재 가정 (Ground Plane Assumption):** 알고리즘의 핵심 최적화 전략이 지면의 존재를 강하게 가정하므로, 지면이 없거나 감지하기 어려운 무인 항공기(UAV) 또는 다층 실내 환경에서는 성능이 저하되거나 적용이 어려울 수 있다.15
- **단순한 루프 클로저:** 내장된 루프 클로저의 성능이 제한적이어서, 대규모 복잡한 환경에서는 전역적으로 일관된 지도를 생성하기 위해 외부의 고성능 장소 인식 모듈의 도움이 필요할 수 있다.16



- **발표:** Shan, T., Englot, B., Meyers, D., Wang, W., Ratti, C., & Rus, D. (2020). LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping. arXiv preprint.21

- **참고 관계:** LIO-SAM은 LOAM의 특징 추출 방식(엣지/평면)을 프론트엔드에서 일부 활용하지만 14, 백엔드를 완전히 다른 철학인 

  **팩터 그래프(Factor Graph)** 기반의 최적화 프레임워크로 대체했다. 이를 통해 IMU, GPS 등 다중 센서를 강결합(Tightly-coupled) 방식으로 융합하는 혁신을 이루었다.21 이는 LOAM 계열의 점진적 개선이 아닌, 백엔드 아키텍처의 근본적인 전환에 해당한다.


LIO-SAM의 핵심은 IMU와 LiDAR 센서 간의 공생 관계를 구축하는 데 있다. 두 센서는 단순히 측정값을 합치는 것이 아니라, 서로의 약점을 실시간으로 보완하는 유기적인 관계를 형성한다. 이 과정은 다음과 같이 이루어진다.

1. 로봇이 빠르게 움직이면 라이다 스캔 데이터는 왜곡된다. 고주파(200Hz 이상 권장) IMU는 이 빠른 움직임을 정밀하게 포착한다.22
2. IMU 측정값을 기반으로 각 라이다 포인트가 측정된 시점의 정확한 센서 위치를 역산하여, 왜곡된 포인트 클라우드를 원래의 형태로 복원(de-skewing)한다.21
3. 왜곡이 보정된 깨끗한 스캔 데이터는 더 정확한 라이다 오도메트리 계산을 가능하게 한다.
4. 이 정확하게 계산된 라이다 오도메트리 결과는 다시 팩터 그래프 내에서 IMU의 고질적인 문제인 드리프트와 바이어스(오차 성향)를 추정하고 보정하는 데 사용된다.21

이러한 선순환 구조 덕분에, 한 센서만으로는 안정적인 추정이 불가능했던 공격적인 모션(aggressive motion)이나 특징이 부족한 환경에서도 시스템 전체의 강건성이 비약적으로 향상된다.

이 모든 과정은 팩터 그래프 최적화를 통해 이루어진다. 시스템의 상태(로봇의 포즈, 속도, IMU 바이어스 등)와 다양한 센서의 측정값 사이의 관계를 '팩터(factor)'라는 확률적 제약 조건으로 모델링하고, 모든 팩터의 오차 합을 최소화하는 전역 최적화를 수행한다.21 LIO-SAM에서 사용되는 주요 팩터는 다음과 같다 21:

1. **IMU 사전적분(Pre-integration) 팩터:** 두 라이다 키프레임 사이의 IMU 측정값들을 적분하여 상대적인 모션 제약을 생성한다. 이는 라이다 오도메트리의 초기값을 제공하고 모션 왜곡을 보정하는 핵심 역할을 한다.
2. **LiDAR 오도메트리 팩터:** 키프레임 간의 특징점(엣지/평면) 정합을 통해 얻어진 상대 포즈 제약을 나타낸다.
3. **GPS 팩터:** GPS 수신이 가능할 때, 절대 위치 정보를 팩터로 추가하여 시스템의 전역 드리프트를 효과적으로 억제한다.
4. **루프 클로저 팩터:** 과거에 방문했던 장소를 다시 인식했을 때, 두 시간대의 포즈 사이에 제약을 추가하여 그동안 누적된 오차를 전역적으로 분산시키고 보정한다.


- **슬라이딩 윈도우 최적화:** 실시간 성능을 보장하기 위해, 전체 맵이 아닌 최근의 일부 키프레임들로 구성된 지역적인 슬라이딩 윈도우 내에서만 최적화를 수행한다. 윈도우에서 벗어나는 오래된 변수들은 주변화(marginalization) 기법을 통해 계산에서 제외하여 복잡도를 제어한다.21
- **키프레임 선택:** 모든 라이다 스캔을 팩터 그래프에 추가하면 계산량이 폭발적으로 증가한다. 이를 방지하기 위해 로봇의 이동 거리나 회전 각도가 일정 임계값을 넘을 때만 해당 스캔을 키프레임으로 선택하여 그래프가 불필요하게 커지는 것을 막는다.21


- **장점:** 다중 센서의 강결합을 통해 매우 높은 정확도와 강건성을 달성했다. 특히 공격적인 움직임, 장기간 주행, 특징이 부족한 환경에서 기존 방법들보다 월등한 성능을 보인다.21 또한, 루프 클로저와 GPS 융합을 통해 전역적으로 일관된 지도 작성이 가능하다.
- **해결한 문제:** LOAM 계열의 무한 드리프트 문제를 근본적으로 해결했으며, GPS 사용 유무에 관계없이 장기간 안정적인 SLAM 운용을 가능하게 했다.


- **계산 복잡성:** 필터 기반 방식에 비해 팩터 그래프 최적화는 본질적으로 더 많은 계산 자원을 요구한다.
- **센서 의존성:** 최상의 성능을 내기 위해서는 가속도, 자이로, 지자기 센서가 모두 포함된 고품질의 9축 IMU 사용이 권장된다. 특히 초기 자세 추정에 지자기 센서의 yaw 값이 활용될 수 있다.22
- **LVI-SAM으로의 확장:** 이 프레임워크는 시각(Visual) 센서까지 통합한 LVI-SAM으로 자연스럽게 확장되었다. LVI-SAM은 라이다가 취약한 환경(예: 특징 없는 긴 벽)과 카메라가 취약한 환경(예: 어두운 공간)을 상호 보완하여 더욱 강건한 시스템을 구축하는 기반이 된다.23



- **발표:** Xu, W., & Zhang, F. (2021). FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter. IEEE Robotics and Automation Letters (RA-L).24
- **참고 관계:** FAST-LIO는 LIO-SAM과 마찬가지로 강결합 라이다-관성 오도메트리(LIO)를 목표로 하지만, 백엔드 최적화 엔진으로 그래프 최적화 대신 **반복 확장 칼만 필터(Iterated Extended Kalman Filter, iEKF)**를 채택했다. 이는 SLAM 문제에 대한 근본적으로 다른 철학적 접근을 의미한다.

LIO-SAM과 FAST-LIO의 등장은 현대 LIO SLAM이 두 가지 주요한 최적화 철학으로 분기되었음을 명확히 보여준다. 그래프 기반의 LIO-SAM은 '과거의 모든 정보를 활용하여 전역적 일관성을 맞추는 것'에 중점을 둔다. 이는 루프가 닫혔을 때 과거의 모든 경로를 재조정하여 전체 맵의 정합성을 보장하며, 정밀한 오프라인 지도 제작에 이상적이다. 반면, 필터 기반의 FAST-LIO는 '현재 시점의 상태를 가장 빠르고 정확하게 추정하는 것'에 집중한다. 과거의 상태는 요약된 확률 분포로만 유지하고, 새로운 측정값이 들어올 때마다 현재 상태만을 업데이트한다. 과거를 수정하지 않는 대신, 매우 빠른 속도와 낮은 지연 시간으로 현재 상태를 추정할 수 있어 빠른 속도로 비행하는 드론의 실시간 제어와 같이 즉각적인 피드백이 중요한 응용 분야에 더 적합하다.


- **새로운 칼만 이득 공식:** FAST-LIO의 가장 핵심적인 혁신은 칼만 필터의 업데이트 과정에서 계산 복잡도를 획기적으로 줄인 새로운 칼만 이득(Kalman Gain) 공식을 유도한 것이다. 전통적인 칼만 필터에서 계산량은 측정값의 차원, 즉 수천 개에 달하는 라이다 포인트 수에 비례하여 증가했다. 하지만 FAST-LIO의 새로운 공식은 계산량이 상태 변수의 차원(로봇의 포즈, 속도 등 수십 개의 변수)에만 의존하도록 설계되었다.24 이 덕분에 수많은 라이다 포인트를 측정값으로 사용하면서도 엄청난 계산 속도 향상을 이룰 수 있었다.
- **iEKF (반복 확장 칼만 필터):** 라이다와 IMU의 관계는 비선형성이 강하기 때문에, 표준 확장 칼만 필터(EKF)는 선형화 오차로 인해 성능이 저하될 수 있다. iEKF는 측정값 업데이트 단계를 여러 번 반복하여 선형화 지점을 재계산함으로써, 비선형 시스템에 대한 추정 정확도를 크게 향상시킨다.


FAST-LIO의 후속 버전인 FAST-LIO2는 프론트엔드에서อีก 한 번의 패러다임 전환을 이끌었다. 이는 '특징'의 개념을 추상화하고 일반화하는 과정으로 볼 수 있다. LOAM이 인간이 인지하는 기하학적 구조(엣지/평면)를 '특징'으로 정의했다면, FAST-LIO2는 이 특징 추출 단계를 완전히 제거하고, 다운샘플링된 원본 포인트(raw points) 자체를 직접 맵에 정합하는 '직접 정합(Direct Registration)' 방식을 채택했다.27 이 접근법은 SLAM 프론트엔드의 패러다임을 '어떻게 특징을 잘 정의할까'에서 '어떻게 모든 정보를 효율적으로 사용할까'로 전환시켰다. 이로 인해 뚜렷한 기하학적 구조가 없는 환경에서도 강건하게 동작하며, 회전식 라이다와 고정형(solid-state) 라이다 등 다양한 종류의 센서에 별도의 수정 없이 적용 가능한 높은 범용성을 확보하게 되었다.27

- **증분 k-d 트리 (ikd-Tree):** FAST-LIO2는 대규모 포인트 클라우드 맵을 효율적으로 관리하기 위해 'ikd-Tree'라는 새로운 동적 데이터 구조를 개발했다. ikd-Tree는 빠른 최근접 이웃 탐색(kNN search)은 물론, 맵에 새로운 포인트를 추가하거나 오래된 포인트를 삭제하고, 맵을 다운샘플링하는 점진적 업데이트(incremental update)를 매우 낮은 계산 비용으로 지원한다. 이는 실시간으로 맵을 계속 갱신해야 하는 SLAM 시스템에 필수적인 기능이다.27


- **장점:** 압도적인 계산 속도와 실시간성(최대 100Hz의 오도메트리 및 맵핑 가능)을 자랑한다.27 특히 극심한 모션(최대 1000 deg/s의 회전)에서도 안정적인 성능을 유지하며 27, 다양한 종류의 라이다 센서에 대한 높은 범용성을 가진다.
- **해결한 문제:** 강결합 LIO 프레임워크의 고질적인 문제였던 높은 계산 부하를 해결하여, 고사양 프로세서 없이도 고성능 실시간 SLAM을 가능하게 했다.


- **내재된 전역 최적화 부재:** 필터 기반 알고리즘의 특성상, 원본 FAST-LIO/FAST-LIO2 자체에는 전역 루프 클로저 기능이 포함되어 있지 않다.

- **커뮤니티 기반의 진화:** FAST-LIO의 이러한 '단점'은 오히려 커뮤니티의 폭발적인 혁신을 촉발하는 계기가 되었다. FAST-LIO의 강력한 실시간 오도메트리 엔진과 공개된 소스 코드 28를 기반으로, 전 세계의 개발자들이 각자의 루프 클로저 모듈(ScanContext, Radius-Search 등)을 결합한 수많은 파생 프로젝트들을 만들어냈다. 

  `FAST_LIO_LC` 29, 

  `FAST-LIO-SAM` 30 등이 그 대표적인 예이다. 이는 현대 SLAM 개발 패러다임이 하나의 거대한 단일 시스템을 만드는 것에서, 각자 가장 뛰어난 부분을 모듈로 만들어 결합하는 '모듈화'와 '민주화'의 방향으로 진화하고 있음을 보여준다. 이러한 오픈소스 생태계는 현재 SLAM 기술 발전의 가장 강력한 원동력 중 하나이다.



아래 표는 본 보고서에서 분석한 네 가지 핵심 라이다 SLAM 기술의 특성을 다차원적으로 비교하여 정리한 것이다.

| 항목 (Category)      | LOAM (2014)                                | LeGO-LOAM (2018)                                         | LIO-SAM (2020)                                             | FAST-LIO / FAST-LIO2 (2021)                               |
| -------------------- | ------------------------------------------ | -------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------- |
| **발표 연도 (Year)** | 2014 8                                     | 2018 16                                                  | 2020 21                                                    | 2021 24                                                   |
| **핵심 방법론**      | 특징 기반 오도메트리/맵핑 분리             | 지면 최적화 및 경량화                                    | 강결합 라이다-관성 오도메트리                              | 강결합 라이다-관성 오도메트리                             |
| **백엔드 최적화**    | 비선형 최적화 (Scan-to-Map)                | 비선형 최적화 + 루프 클로저                              | 팩터 그래프 최적화 (Smoothing)                             | 반복 확장 칼만 필터 (iEKF)                                |
| **필수 센서 구성**   | 3D LiDAR                                   | 3D LiDAR                                                 | 3D LiDAR, 9축 IMU                                          | 3D LiDAR, 6/9축 IMU                                       |
| **주요 특징**        | 엣지/평면 특징 추출, 모션 왜곡 보정 12     | 지면 분리, 2단계 최적화, 저전력 시스템 구동 15           | 다중 센서 융합 (IMU, GPS), 슬라이딩 윈도우, 전역 최적화 21 | 새로운 칼만 이득 공식, 직접 정합 (FAST-LIO2), ikd-Tree 24 |
| **장점**             | 실시간성, 낮은 드리프트 (당시 기준) 12     | UGV 환경 강건성, 높은 계산 효율, 저사양 하드웨어 지원 15 | 높은 정확도, 공격적 모션에 강건, 전역 일관성 확보 21       | 압도적인 계산 속도, 극심한 모션에 강건, 높은 범용성 27    |
| **단점 및 한계**     | 루프 클로저 부재, 특징 없는 환경에 취약 11 | 지면 존재 가정, 단순한 루프 클로저 15                    | 상대적으로 높은 계산 복잡도, 고품질 IMU 의존성 22          | 원본에 루프 클로저 부재 (커뮤니티가 보완)                 |
| **주요 적용 분야**   | 초기 실시간 3D SLAM 연구                   | 무인 지상 차량(UGV)                                      | 자율주행, 장기간/대규모 맵핑                               | 무인 항공기(UAV), 고속 이동체                             |


라이다 SLAM 기술의 발전은 **LOAM**을 공통의 뿌리로 하여 여러 갈래로 진화하는 모습을 보인다. 이 기술 계보는 다음과 같이 시각적으로 요약될 수 있다.

1. **시작점: LOAM (2014)**
   - 특징 기반(엣지/평면) 접근법과 오도메트리/맵핑 분리 구조를 확립하며 모든 후속 연구의 기준점이 되었다.
2. **분기 1: 특화 및 개선 (LeGO-LOAM, 2018)**
   - LOAM에서 직접 파생된 경로. LOAM의 기본 구조는 유지하되, 프론트엔드에 **"UGV 최적화"** (지면 분리, 경량화)를 추가하고 백엔드에 루프 클로저를 통합하여 특정 응용 분야의 성능을 극대화했다.
3. **분기 2: 백엔드의 혁신 - 그래프 기반 강결합 (LIO-SAM, 2020)**
   - LOAM의 특징 추출 아이디어는 차용했지만, 백엔드를 완전히 새로운 **"팩터 그래프 기반 강결합"** 아키텍처로 대체했다. IMU, GPS 등 다중 센서를 통합하여 전역 최적화와 강건성을 추구하는 방향으로 나아갔다.
4. **분기 3: 백엔드의 혁신 - 필터 기반 강결합 (FAST-LIO, 2021)**
   - LIO-SAM과 같은 강결합 LIO를 목표로 하지만, 백엔드로 **"칼만 필터 기반 강결합"**을 선택했다. 이는 계산 효율성과 실시간성을 최우선으로 고려하는 또 다른 주요 흐름을 형성했다.
   - **FAST-LIO2**는 여기서 한 걸음 더 나아가 프론트엔드의 특징 추출마저 제거하고 '직접 정합' 방식을 도입함으로써, SLAM의 일반성과 적용 범위를 크게 확장했다.

이 계보도는 현대 라이다 SLAM이 단일한 경로가 아닌, 해결하고자 하는 문제(경량화, 강건성, 실시간성)에 따라 다양한 철학과 아키텍처로 분화하며 발전해왔음을 명확히 보여준다.



지금까지 논의된 SLAM 알고리즘들은 대부분 곡률과 같은 수작업으로 설계된(hand-crafted) 기하학적 특징에 의존한다. 이러한 접근법은 특정 환경에서는 효과적이지만, 복잡하고 다양한 실제 환경에 대한 일반성과 강건성에는 한계가 있다.2 최근 딥러닝 기술의 발전은 이러한 한계를 극복할 새로운 가능성을 제시하고 있다.

딥러닝 기반 SLAM은 신경망을 이용해 방대한 데이터로부터 특징을 직접 학습한다. 이는 인간이 설계한 특징보다 더 추상적이고 강력한 표현력을 가질 수 있다.2 예를 들어, **DeepPointMap**과 같은 최신 연구는 DPM Encoder라는 신경망을 통해 포인트 클라우드에서 '신경 기술자(neural descriptor)'라는 고도로 압축된 특징 벡터를 추출한다.2 이 통일된 기술자는 오도메트리, 루프 클로저, 맵 표현 등 다양한 SLAM 하위 작업에 일관되게 사용되어, 전체 프레임워크를 단순화하면서도 기존 방법들보다 높은 성능을 달성한다.2 이는 SLAM의 특징 추출 패러다임이 기하학적 정의에서 데이터 기반 학습으로 이동하고 있음을 보여주는 중요한 사례이다.


기술의 발전은 알고리즘뿐만 아니라 하드웨어에서도 일어나고 있다. 기존의 기계식 회전형 라이다와 달리, 최근에는 반도체 기술을 이용한 고정형 라이다(Solid-State LiDAR)가 주목받고 있다. 고정형 라이다는 기계적 회전 부품이 없어 내구성이 뛰어나고 가격이 저렴하며 크기가 작다는 장점이 있다. 하지만 시야각(Field of View, FoV)이 좁고, 스캔 패턴이 불규칙하며, 데이터 전송률이 매우 높다는 특징을 가진다.32

이러한 특성은 기존 SLAM 알고리즘에 새로운 도전을 제기한다. 360도 회전 스캔을 가정하고 설계된 LOAM 계열 알고리즘은 좁은 FoV를 가진 고정형 라이다로 빠르게 회전할 경우, 연속된 스캔 간의 중첩 영역이 부족하여 추적을 놓치기 쉽다.34 또한, 높은 데이터 전송률은 알고리즘의 실시간성을 보장하기 위해 더 높은 계산 성능을 요구한다. 따라서 이러한 새로운 센서의 특성에 최적화된 경량화되고 강건한 SLAM 알고리즘 개발이 활발히 진행되고 있으며, FAST-LIO2와 같이 범용성이 높은 알고리즘이 대안으로 주목받고 있다.27


라이다 SLAM 기술은 LOAM이 제시한 특징 기반 프레임워크에서 출발하여, IMU와 같은 다중 센서를 강결합하는 방향으로 빠르게 발전했다. 이 과정에서 백엔드 최적화 방식에 따라 전역 일관성을 중시하는 그래프 기반(LIO-SAM)과 실시간성을 극대화하는 필터 기반(FAST-LIO)으로 명확히 분화되었다.

미래의 라이다 SLAM 기술은 다음과 같은 방향으로 진화할 것으로 전망된다.

- **하이브리드 시스템의 부상:** 실시간 오도메트리는 필터 기반(FAST-LIO)으로 빠르고 안정적으로 수행하고, 전역 최적화 및 루프 클로저는 그래프 기반(LIO-SAM)으로 처리하여 두 접근법의 장점을 모두 취하는 하이브리드 시스템이 주류가 될 가능성이 높다.
- **딥러닝의 전면적 도입:** 프론트엔드에서는 딥러닝 기반의 특징 추출 및 장소 인식이 기존의 기하학적 방식을 대체하여, 환경 변화에 더욱 강건하고 지능적인 SLAM 시스템을 구현할 것이다.
- **하드웨어-소프트웨어 동시 최적화:** 고정형 라이다와 같은 새로운 센서 하드웨어의 등장은 이에 최적화된 새로운 알고리즘의 개발을 계속해서 촉진할 것이며, 하드웨어와 소프트웨어가 함께 발전하는 선순환 구조가 더욱 강화될 것이다.

결론적으로, 현대 라이다 SLAM은 더 이상 단일 알고리즘의 성능 경쟁이 아닌, 다양한 센서, 최적화 철학, 그리고 학습 기반 방법론이 융합되는 시스템 공학의 영역으로 진화하고 있다.


1. A Review of 2D Lidar SLAM Research - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/17/7/1214
2. DeepPointMap: Advancing LiDAR SLAM with Unified Neural Descriptors - arXiv, accessed July 23, 2025, https://arxiv.org/html/2312.02684v1
3. Lidar SLAM: The Ultimate Guide to Simultaneous Localization and Mapping - Wevolver, accessed July 23, 2025, https://www.wevolver.com/article/lidar-slam
4. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar - MDPI, accessed July 23, 2025, https://www.mdpi.com/2032-6653/16/2/56
5. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar, accessed July 23, 2025, https://www.preprints.org/manuscript/202411.2260/v1
6. Tightly Coupled LiDAR-Inertial Odometry and Mapping for Underground Environments - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/23/15/6834
7. A Review of 2D Lidar SLAM Research - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/390333764_A_Review_of_2D_Lidar_SLAM_Research
8. Zhang, J. and Singh, S. (2014) LOAM Lidar Odometry and Mapping in Real-Time. Robotics Science and Systems Conference, Berkeley, July 2014. - References, accessed July 23, 2025, https://www.scirp.org/reference/referencespapers?referenceid=2606507
9. LOAM: Lidar Odometry and Mapping in Real-time, accessed July 23, 2025, https://www.ri.cmu.edu/publications/loam-lidar-odometry-and-mapping-in-real-time/
10. LOAM: Lidar Odometry and Mapping in Real-time. - DBLP, accessed July 23, 2025, https://dblp.org/rec/conf/rss/ZhangS14
11. LOAM: Lidar Odometry and Mapping in Real-time - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/282704722_LOAM_Lidar_Odometry_and_Mapping_in_Real-time
12. LOAM: Lidar Odometry and Mapping in Real-time - Carnegie Mellon ..., accessed July 23, 2025, https://www.ri.cmu.edu/pub_files/2014/7/Ji_LidarMapping_RSS2014_v8.pdf
13. F-LOAM : Fast LiDAR Odometry and Mapping - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2107.00822
14. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PMC, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
15. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry ..., accessed July 23, 2025, https://static.ux5.de/Moving-Object-Detection-with-OpenCV/archiv/learnopencv-master/LeGO-LOAM-ROS2/Shan_Englot_IROS_2018_Preprint.pdf
16. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain - GitHub, accessed July 23, 2025, https://github.com/RobustFieldAutonomyLab/LeGO-LOAM
17. LeGO-LOAM/Shan_Englot_IROS_2018_Preprint.pdf at master - GitHub, accessed July 23, 2025, https://github.com/RobustFieldAutonomyLab/LeGO-LOAM/blob/master/Shan_Englot_IROS_2018_Preprint.pdf
18. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain - Stevens Institute of Technology, accessed July 23, 2025, https://researchwith.stevens.edu/en/publications/lego-loam-lightweight-and-ground-optimized-lidar-odometry-and-map
19. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain - OpenCV, accessed July 23, 2025, https://opencv.org/blog/lego-loam/
20. An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM and Scan Context for Underground Coalmine - PMC - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8778426/
21. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed July 23, 2025, http://arxiv.org/pdf/2007.00258
22. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 23, 2025, https://github.com/TixiaoShan/LIO-SAM
23. (PDF) LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping, accessed July 23, 2025, https://www.researchgate.net/publication/351063772_LVI-SAM_Tightly-coupled_Lidar-Visual-Inertial_Odometry_via_Smoothing_and_Mapping
24. FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/349909094_FAST-LIO_A_Fast_Robust_LiDAR-inertial_Odometry_Package_by_Tightly-Coupled_Iterated_Kalman_Filter
25. Fast-lio: A fast, robust lidar-inertial odometry package by tightly-coupled iterated Kalman filter - HKU Scholars Hub, accessed July 23, 2025, https://hub.hku.hk/handle/10722/301527
26. arxiv.org, accessed July 23, 2025, https://arxiv.org/abs/2010.08196
27. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 23, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
28. zlwang7/S-FAST_LIO - GitHub, accessed July 23, 2025, https://github.com/zlwang7/S-FAST_LIO
29. yanliang-wang/FAST_LIO_LC: The tight integration of FAST-LIO with Radius-Search-based loop closure module. - GitHub, accessed July 23, 2025, https://github.com/yanliang-wang/FAST_LIO_LC
30. a SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on LIO-SAM paper - GitHub, accessed July 23, 2025, https://github.com/engcang/FAST-LIO-SAM
31. Overview of DeepPointMap++. (a) DPM Encoder extracts feature from the... | Download Scientific Diagram - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/Overview-of-DeepPointMap-a-DPM-Encoder-extracts-feature-from-the-input-point-cloud_fig1_391420366
32. Lightweight 3-D Localization and Mapping for Solid-State ... - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2102.03800
33. A Low-Cost 3D SLAM System Integration of Autonomous Exploration Based on Fast-ICP Enhanced LiDAR-Inertial Odometry - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/16/11/1979
34. Lightweight 3-D Localization and Mapping for Solid-State LiDAR - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2102.03800


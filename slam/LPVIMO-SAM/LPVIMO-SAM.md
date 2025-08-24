# LPVIMO-SAM에 대한 종합적 기술 분석
[LPVIMO-SAM](./index.md)

까다로운 환경에서의 다중 센서 융합 SLAM 발전



동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM)은 에이전트가 미지의 환경을 탐색하는 데 있어 근본적인 계산 문제로 정의된다.1 핵심 개념은 로봇이나 차량이 지도를 구축함과 동시에 그 지도 내에서 자신의 위치를 추적해야 한다는 것이다.5 이 기술은 가정용 로봇 청소기(Roomba)부터 자율주행차(Waymo, Tesla), 우주 탐사 로봇에 이르기까지 광범위한 응용 분야에서 진정한 자율성을 구현하는 핵심 요소로 간주된다.2 SLAM은 사전 정보가 없는 환경에서 로봇이 자율적으로 작동할 수 있도록 함으로써, 인간이 탐사하기에 너무 작거나 위험한 지역의 지도를 제작할 수 있게 한다.1


SLAM은 본질적으로 "닭이 먼저냐, 달걀이 먼저냐" 하는 딜레마를 내포하고 있다. 즉, 정확한 위치 추정을 위해서는 좋은 지도가 필요하고, 좋은 지도를 만들기 위해서는 정확한 위치 추정이 선행되어야 한다.8 이러한 상호 의존적인 문제를 해결하기 위해 현대 SLAM은 확률론적 접근법을 채택한다. SLAM의 목표는 센서 측정값과 제어 입력이 주어졌을 때, 로봇의 전체 경로와 지도에 대한 사후 확률(posterior probability)을 추정하는 것으로 수학적으로 정식화된다.4 이 확률적 프레임워크는 센서의 노이즈와 모션의 불확실성을 수학적으로 모델링하여 최적의 해를 찾는 기반을 제공하며, 모든 현대 SLAM 솔루션의 수학적 기초를 이룬다.


SLAM 백엔드, 즉 수집된 데이터를 처리하여 지도와 경로를 추정하는 계산 엔진은 역사적으로 큰 발전을 거듭해왔다.


초기 SLAM 시스템은 확장 칼만 필터(Extended Kalman Filter, EKF)와 같은 필터링 기법을 기반으로 했다. 이 접근법은 SLAM을 재귀적인 상태 추정 문제로 취급한다.8 매 시간 단계마다 새로운 측정값을 사용하여 현재 로봇의 자세와 랜드마크 위치에 대한 상태 및 공분산을 업데이트한다. MonoSLAM과 같은 초기 시스템들이 EKF를 성공적으로 활용했지만 11, 이 방식은 몇 가지 근본적인 한계를 가지고 있었다. 가장 큰 문제는 계산 복잡도이다. 환경의 크기가 커지고 랜드마크 수가 증가함에 따라 상태 벡터와 공분산 행렬의 크기가 기하급수적으로 커져, 계산량이 상태 수의 제곱($O(n^2$))에 비례하게 증가한다.12 이는 대규모 환경에서는 실시간 처리를 불가능하게 만드는 요인이었다.13

더욱 심각한 문제는 EKF가 본질적으로 비선형인 SLAM 문제에 적용될 때 발생하는 일관성(consistency) 문제이다. EKF는 각 단계에서 비선형 모델을 선형화하는데, 이 과정에서 발생하는 선형화 오류는 영구적으로 누적된다. 필터링 방식은 과거 정보를 요약하여 현재 상태에 반영(marginalization)하므로, 한번 잘못된 선형화로 인해 발생한 오차는 이후에 새로운 정보가 들어와도 수정할 수 없다.14 이로 인해 추정치가 실제 값에서 점차 벗어나는, 즉 일관성을 잃는 문제가 발생하며, 이는 장기적인 자율 주행에 치명적인 결함이 된다.


이러한 필터링 방식의 한계를 극복하기 위해 현대 SLAM은 그래프 기반 최적화라는 새로운 패러다임으로 전환했다. 이 접근법은 SLAM 문제를 대규모 비선형 최소제곱 최적화 문제로 재정의한다.9 그래프는 노드(node)와 엣지(edge)로 구성되는데, 노드는 로봇의 각 시점에서의 자세나 랜드마크의 위치와 같은 미지의 변수를 나타내고, 엣지는 센서 측정값(오도메트리, 랜드마크 관측 등)으로부터 파생된 변수들 간의 확률적 제약 조건을 나타낸다.8

이 방식은 '전체 SLAM(full SLAM)' 또는 '평활화(smoothing)'라고도 불리며, 현재 상태뿐만 아니라 로봇의 전체 경로를 최적화한다.14 필터링과 달리 과거의 모든 자세와 제약 조건을 유지하므로, 루프 클로저(loop closure)와 같이 과거와 현재를 연결하는 새로운 정보가 들어왔을 때 전체 경로를 재선형화하고 오차를 전역적으로 재분배하여 과거의 오류까지 수정할 수 있다.

초기에는 이 대규모 최적화 문제를 푸는 것이 계산적으로 비효율적이라고 여겨졌으나, SLAM 문제의 그래프가 본질적으로 희소(sparse)하다는 사실이 밝혀지면서 돌파구가 마련되었다. 각 센서 측정은 소수의 변수(예: 오도메트리 측정은 두 개의 연속된 자세)만을 연결하므로, 전체 문제에 대한 정보 행렬(information matrix)은 매우 희소한 구조를 가진다.8 이 희소성을 활용하는 Cholesky 분해나 QR 분해와 같은 선형대수 기법을 통해 대규모 최적화 문제를 매우 효율적으로 풀 수 있게 되었다.14

이러한 필터링에서 그래프 최적화로의 패러다임 전환은 단순한 알고리즘 개선을 넘어, SLAM 문제에 대한 근본적인 접근 방식의 변화였다. 이는 EKF-SLAM의 확장성 및 일관성 문제를 극복하고 대규모, 장기간 자율 주행을 위한 길을 열었다. LIO-SAM이나 본 보고서에서 다루는 LPVIMO-SAM과 같은 현대적인 시스템의 'SAM(Smoothing and Mapping)' 백엔드는 바로 이 그래프 기반 최적화 철학의 직계 후손이다. 이 철학 덕분에 이들 시스템은 루프 클로저, GPS, 지자기 센서 측정값 등 다양하고 비동기적인 측정값들을 단일한 전역적 일관성을 갖는 최적화 문제에 '요인(factor)'으로 통합할 수 있으며, 이는 EKF 방식으로는 다루기 매우 어려운 작업이다.




시각-관성 오도메트리(Visual-Inertial Odometry, VIO)는 카메라와 관성 측정 장치(Inertial Measurement Unit, IMU)의 데이터를 융합하여 이동체의 6자유도(6-DOF) 자세와 속도를 추정하는 기술이다.16 이 두 센서는 상호 보완적인 특성을 가진다. 카메라는 환경의 풍부한 시각적 정보(텍스처, 특징점 등)를 제공하여 위치를 인식하는 데 사용되지만 20, 조명 변화에 민감하고 빠른 움직임 시 모션 블러가 발생하기 쉽다. 반면, IMU는 가속도계와 자이로스코프를 이용해 고주파수(high-frequency)로 가속도와 각속도를 측정하여 단기적인 움직임을 정밀하게 추정하고, 카메라가 정보를 얻기 어려운 빠른 움직임 상황에서 모션 블러와 같은 문제를 보완한다.16


VIO 시스템의 성능은 카메라와 IMU 데이터를 어떻게 결합하느냐에 따라 크게 달라진다. 결합 방식은 크게 약결합(loosely-coupled)과 강결합(tightly-coupled)으로 나뉜다. 약결합 방식은 카메라와 IMU 각각에서 독립적으로 자세를 추정한 후, 그 결과를 융합하는 방식이다. 반면, 강결합 방식은 두 센서의 원시 측정값(raw measurements), 즉 시각적 특징점과 IMU 사전적분(pre-integration) 항을 하나의 최적화 문제로 묶어 공동으로 최적화한다. 일반적으로 강결합 방식이 센서 간의 상호 제약 조건을 더 잘 활용하므로 정확도와 강건성 면에서 더 우수하다.20 LPVIMO-SAM은 명시적으로 강결합 방식을 채택하고 있다.22


VIO는 강력한 기술이지만 몇 가지 내재적인 한계를 가지고 있다. 특히 단안 카메라(monocular camera)를 사용하는 VIO는 실제 세계의 미터(metric) 단위 스케일을 결정할 수 없는 스케일 모호성(scale ambiguity) 문제를 겪는다.20 즉, 추가적인 정보 없이는 생성된 지도의 실제 크기를 알 수 없다. 또한, 모든 VIO 시스템은 환경 조건에 민감하여 조명이 어둡거나, 텍스처가 부족하거나, 특징점이 거의 없는 환경에서는 성능이 크게 저하된다.11



라이다-관성 오도메트리(LiDAR-Inertial Odometry, LIO)는 3D 라이다 포인트 클라우드와 IMU 데이터를 융합하는 시스템이다. 라이다는 레이저를 이용하여 주변 환경까지의 거리를 매우 정밀하게 측정하므로, 조명 변화에 거의 영향을 받지 않는 정확한 미터 단위의 3D 구조 정보를 제공한다.7 LIO 시스템에서 IMU는 두 가지 중요한 역할을 한다. 첫째, 라이다가 한 바퀴 스캔하는 동안 발생하는 움직임으로 인해 포인트 클라우드가 왜곡되는 현상을 보정(de-skewing)한다. 둘째, 스캔 매칭(scan-matching) 알고리즘을 위한 고주파수의 움직임 예측값을 제공하여 최적화 과정을 가속화하고 정확도를 높인다.27


LIO-SAM과 같은 LIO 시스템은 일반적으로 포인트 클라우드에서 평면(planar)이나 모서리(edge)와 같은 기하학적 특징을 추출하여 연속된 스캔 간의 정합에 사용한다.27 이를 통해 계산 효율성을 높이고 강건한 추정을 가능하게 한다.


LIO 시스템은 기하학적으로 특징이 없는 환경에서 성능이 저하(degenerate)되는 약점을 가진다. 예를 들어, 특징 없는 긴 복도나 넓은 개활지와 같이 뚜렷한 기하학적 구조가 부족한 곳에서는 스캔 매칭이 실패할 수 있다.29 또한, 유리와 같이 반사율이 높거나 투명한 표면을 제대로 감지하지 못하는 문제도 있다.23

VIO와 LIO 사이의 선택은 근본적으로 의미론적 풍부함(semantic richness)과 기하학적 정밀성(geometric precision) 사이의 트레이드오프를 나타낸다. 이러한 트레이드오프는 LVI-SAM과 같은 다중 모드 융합 시스템의 주요 동기가 된다. VIO는 카메라를 사용하여 텍스처, 색상 등 풍부한 정보를 포착하여 장소 인식에 유리하지만 20, 조명과 텍스처에 민감하고 원거리 깊이 정확도가 떨어진다.16 반면 LIO는 라이다를 사용하여 조명과 무관하게 매우 정확한 3D 구조 정보를 제공하지만 7, 포인트 클라우드는 카메라 이미지의 풍부한 외형 정보가 부족하여 기하학적으로 퇴화된 장면에서 취약하다.26 이 두 센서의 장단점은 거의 완벽하게 상호 보완적이다. 라이다는 단안 VIO가 부족한 미터 단위 스케일을 제공할 수 있고, 카메라는 긴 복도에서 라이다가 부족한 풍부한 특징을 제공할 수 있다. 이는 라이다와 시각/관성을 융합하는 시스템(예: LVI-SAM 27)이 이미 강력한 조합임을 시사한다. 그러나 LPVIMO-SAM의 존재는 이러한 강력한 LVI 융합조차도 특정 까다로운 환경(예: 유리벽이 있는 길고 특징 없는 복도)에서는 불충분하다는 것을 의미한다. LPVIMO-SAM의 5개 센서 조합은 주요 시각 및 라이다 모달리티가 모두 저하되었을 때에도 강건한 시스템을 만들려는 시도이다.



SAM과 필터링의 차이점을 더 깊이 살펴보면, SAM은 '전체 SLAM' 문제를 해결하여 전체 경로와 지도를 추정하는 반면, 필터링은 현재의 자세만을 추정한다.4 SAM의 핵심 장점은 루프 클로저와 같은 새로운 정보가 도착했을 때 전체 문제를 재선형화하여 필터에서는 영구적으로 남을 과거의 오류를 수정할 수 있다는 점이다.14


요인 그래프(Factor Graph)는 SAM 문제를 표현하는 데 사용되는 강력한 도구이다.9

- **노드(Nodes):** 로봇의 자세(x), 랜드마크의 위치(l)와 같은 미지의 변수를 나타낸다.
- **요인(Factors):** 센서 측정값(예: 오도메트리, 랜드마크 관측)에서 파생된 변수 간의 확률적 제약 조건을 나타낸다.35 각 요인은 대규모 비선형 최소제곱 비용 함수의 한 항에 해당한다.14


일반적인 SLAM 문제에 대한 요인 그래프는 희소(sparse)하다. 왜냐하면 각 측정값은 소수의 변수만을 제약하기 때문이다 (예: 오도메트리 측정값은 두 개의 연속된 자세를 연결함).8 iSAM/iSAM2 (LPVIMO-SAM에서 사용 37)와 같은 솔버는 이 희소성을 활용하여 대규모 최적화 문제를 효율적으로, 종종 실시간으로 해결한다.12


3.1. 시스템 개요 및 핵심 목표

LPVIMO-SAM은 라이다, 편광 비전, IMU, 지자기 센서, 광학 흐름 센서를 강결합 방식으로 통합한 프레임워크로 정의된다.22 이 시스템의 명시된 목표는 라이다 성능 저하, 저특징, 특징 부족 지역과 같이 다른 시스템이 실패하는 까다로운 환경에서 고정밀, 고강건성의 실시간 상태 추정 및 지도 작성을 달성하는 것이다.22


약어 'LPV'를 둘러싼 잠재적 혼동을 명확히 할 필요가 있다. 논문의 전체 제목과 초록은 이 약어가 **LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow**의 약자임을 분명히 한다.22 여기서 'P'는 편광 비전(Polarization Vision)을 의미한다.

전문 독자를 위한 포괄적인 맥락을 제공하기 위해, 제어 이론의 **선형 파라미터 변동(Linear Parameter-Varying, LPV)** 시스템에 대한 간략한 개요를 제시한다. LPV 시스템은 상태 공간 행렬이 시간에 따라 변하는 측정 가능한 파라미터에 의존하는 선형 시스템으로 모델링되는 비선형 시스템의 한 종류이다.39 이는 비선형 동역학의 강건한 제어를 위해 사용되는 게인 스케줄링(gain-scheduling)의 한 형태이다.39

결론적으로, LPVIMO-SAM은 약어에도 불구하고 LPV 모델링 기법을 사용하지 **않는다**. 이 시스템의 백엔드는 요인 그래프를 통한 비선형 최소제곱 최적화이며, 파라미터 변동 상태 공간 제어기나 관측기가 아니다.


이 프레임워크는 두 개의 주요하고 상호 연결된 서브시스템으로 구성되며, 하나가 실패할 경우 다른 하나가 독립적으로 작동할 수 있어 전반적인 강건성을 향상시킨다.22

- **편광 비전-관성 시스템 (Polarized Vision-Inertial System, PVIS):** 이 서브시스템은 편광 카메라(예: Sony IMX250MZR)와 IMU의 데이터를 처리한다.37 주요 혁신은 편광 정보를 활용하여 저특징 시나리오에서 VIO의 강건성을 향상시키는 것이다.22
- **라이다/관성/지자기/광학 흐름 시스템 (LiDAR/Inertial/Magnetometer/Optical Flow System, LIMOS):** 이 서브시스템은 강화된 LIO이다. 기본 오도메트리를 위해 라이다와 IMU를 사용하지만, 결정적으로 절대 방위를 위한 지자기 센서와 누적 오차를 줄이기 위한 속도 및 높이 측정을 위한 광학 흐름 센서를 통합한다.22



이것은 가장 새로운 구성 요소이다.

- **작동 방식:** 편광 카메라는 컬러 카메라의 베이어 필터와 유사하게 이미지 센서 위에 마이크로 편광판 배열(예: 0°, 45°, 90°, 135° 필터)을 가지고 있다.44 이를 통해 한 번의 촬영으로 네 개의 편광 이미지를 캡처할 수 있다.
- **추출 정보:** 이 이미지들로부터 각 픽셀에 대해 두 가지 주요 물리적 특성, 즉 **선형 편광도(Degree of Linear Polarization, DoP)**와 **편광각(Angle of Polarization, AOP)**이 계산된다.37 DoP는 빛이 얼마나 강하게 편광되었는지를 나타내고, AOP는 편광의 방향을 나타낸다.
- **SLAM에서의 응용:** 빛은 표면, 특히 유리나 물과 같은 비금속 유전체에서 반사될 때 편광된다.46 DoP와 AOP는 물체의 표면 법선(방향)과 직접적으로 관련이 있다.44 평범한 흰 벽과 같이 표준 특징 검출기(FAST, ORB)가 실패하는 저특징 환경에서, 편광으로부터 파생된 표면 법선 정보는 SLAM 알고리즘에 풍부한 기하학적 제약 조건을 제공한다.37 또한 라이다에 보이지 않거나 표준 카메라를 혼란시키는 경면 반사를 일으키는 유리와 같은 까다로운 재질을 강건하게 감지할 수 있게 한다.32


이들은 시스템의 기하학적 구조와 자기 운동에 대한 핵심적인 이해를 제공하는 기본적인 오도메트리 센서 역할을 한다. IMU 사전적분은 센서 측정값 사이의 모션 제약을 제공하고 라이다 스캔의 왜곡을 보정할 수 있게 한다.27


이들은 장기 오도메트리에서 치명적인 문제인 무한한 오차 누적을 방지하기 위해 절대적 또는 준절대적 제약을 제공하도록 특별히 선택된 보조 센서이다.

- **지자기 센서:** 자기 북쪽을 기준으로 한 절대 방위 참조를 제공하며, 이는 그래프에 사전 요인으로 추가되어 누적된 요(yaw) 오차를 수정한다.22
- **광학 흐름 센서:** 이는 아래를 향하는 센서일 가능성이 높다. 지상에 대한 차량의 속도와 높이를 직접 측정한다. 이들은 속도와 결정적으로 Z축(높이)의 누적 오차를 수정하기 위해 관측 요인으로 추가된다. Z축 누적 오차는 긴 궤적에서 LIO-SAM과 같은 LIO 시스템의 알려진 문제이다.22

LPVIMO-SAM의 아키텍처는 "벨트와 멜빵"과 같은 이중 안전장치 설계 철학을 구현하지만, 계층적 구조를 가지고 있다. 이는 단순히 센서를 추가하는 것이 아니라, 더 간단한 시스템의 특정하고 연쇄적인 실패 모드를 목표로 하는 중복 계층을 만드는 것이다. 기본 VIO 또는 LIO 시스템은 저특징 환경이나 기하학적 퇴화 환경에서 실패할 수 있다.16 LVI 융합(LVI-SAM처럼 27)은 많은 문제를 해결하지만, 환경이 저특징이면서 동시에 기하학적으로 퇴화된 경우(예: 길고 평범한 흰색 복도)에는 두 기본 모달리티 모두 약화된다. 바로 이 지점에서 LPVIMO-SAM의 새로운 센서들이 역할을 한다. 편광 카메라는 VIO의 "저특징" 실패 모드에 대한 직접적인 대응책이며 22, 지자기 및 광학 흐름 센서는 오도메트리가 약할 때 악화되는 "장기 누적 오차" 실패 모드에 대한 직접적인 대응책이다.22 이 시스템은 동시적이고 복합적인 환경 문제에 탄력적으로 대응하도록 설계되었다. 독립적으로 작동할 수 있는 이중 서브시스템 설계(PVIS 및 LIMOS) 22는 궁극적인 안전장치로서, 한 센서 스위트 전체에 치명적인 고장(예: 라이다 고장)이 발생하더라도 전체 시스템이 다운되지 않도록 보장한다. 이는 실패가 허용되지 않는 안전이 중요한 응용 분야에 특히 적합하게 만든다.



이 섹션에서는 LPVIMO-SAM 시스템의 핵심인 공동 최적화 프레임워크를 자세히 설명한다. 문제는 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제로 정식화되며, 모든 센서 측정값이 주어졌을 때 확률을 최대화하는 상태 궤적을 찾아 해결된다.37 이는 요인 그래프로 표현되는 비선형 최소제곱 비용 함수를 최소화하는 것과 동일하다.14


그래프에 추가되는 각 유형의 제약(요인)에 대한 상세한 분석은 다음과 같다. 이는 강결합 전략의 핵심이다.

- **IMU 사전적분 요인:** 고주파 IMU 측정값을 사용하여 연속적인 로봇 상태를 제약하며, 다른 센서 판독값 사이의 움직임을 설명한다.27
- **편광 비전/관성 오도메트리 (PVIO) 요인:** PVIS 서브시스템에서 파생된 요인으로, 편광 카메라 특징과 IMU 데이터를 융합하여 얻은 모션 제약을 나타낸다.37
- **라이다 오도메트리 요인:** LIMOS 서브시스템의 스캔 매칭 결과에서 파생된 요인으로, 라이다 포인트 클라우드 등록을 기반으로 연속적인 자세를 제약한다.27
- **지자기 센서 방위 사전 요인:** 로봇 상태의 절대 방향(요)에 제약을 추가하는 단항 요인으로, 지자기 센서 판독값 쪽으로 끌어당겨 방위 누적 오차를 수정한다.22
- **광학 흐름 속도 및 높이 관측 요인:** 광학 흐름 센서의 직접 측정값을 기반으로 로봇의 속도와 Z축 위치를 제약하는 두 개의 개별 요인이다.22
- **루프 클로저 요인:** 시스템이 이전에 매핑된 위치로 돌아왔음을 인식할 때 현재 자세와 비연속적인 과거 자세 사이에 추가되는 제약이다. 이는 전체 궤적에 걸쳐 누적된 오차를 수정하는 데 매우 중요하다.27


LPVIMO-SAM은 iSAM2 솔버를 사용한다.37 매 단계마다 전체 문제를 처음부터 다시 푸는 배치 최적화와 달리, iSAM2는 증분 알고리즘이다. 새로운 측정값에 의해 영향을 받는 그래프 부분만 재선형화하고 다시 풀어 솔루션을 효율적으로 업데이트하며, 이전 계산은 캐싱하여 재사용한다.8 이것이 복잡한 전체 평활화 백엔드가 실시간으로 실행될 수 있게 하는 이유이다.

LPVIMO-SAM의 요인 그래프는 단순히 제약 조건의 집합이 아니라, 시스템의 확실성에 대한 동적이고 다중 해상도의 표현이다. 서로 다른 요인들은 다양한 수준의 영향력을 가지며 각기 다른 유형의 오차를 목표로 한다. 그래프는 **상대적 자세 제약**(IMU, PVIO, 라이다 오도메트리)과 **절대적/준절대적 제약**(지자기 센서, 광학 흐름)을 모두 포함한다. 상대적 제약은 지역적 움직임을 추적하는 데는 뛰어나지만 적분 오차에 취약하다. 반면, 절대적 제약은 낮은 빈도로 들어오지만 '기준점' 역할을 한다. 예를 들어, 지자기 센서 요인은 이전 자세와 상관없이 현재 자세의 방향을 전역 프레임에서 직접 제약한다. 광학 흐름 요인은 Z축 높이를 직접 제약하여, LIO-SAM에서 지적된 문제인 무한한 높이 누적 오차를 방지한다.22 루프 클로저 요인은 시간적으로 멀리 떨어진 두 지점을 연결하여 그 사이의 누적 오차를 효과적으로 '상쇄'하는 가장 강력한 제약이다.

최적화 과정은 이러한 다양한 요인들을 그들의 불확실성(공분산)에 따라 가중치를 부여하며 끊임없이 균형을 맞추는 작업이다. 조명이 밝고 특징이 풍부한 환경에서는 PVIO와 라이다 요인의 신뢰도가 높아 솔루션을 주도할 것이다. 반면, 길고 특징 없는 복도에서는 이들의 신뢰도가 떨어지고, 솔버는 IMU 사전적분과 누적 오차를 수정하는 지자기 및 광학 흐름 요인에 더 많이 의존하게 될 것이다. 이처럼 최적화 내에서 적응적으로 가중치를 조절하는 것이 바로 강건한 강결합 다중 센서 융합의 본질이다.


이 섹션에서는 LPVIMO-SAM을 잘 알려진 최신 기술(State-of-the-Art, SOTA) 프레임워크와 비교하여 그 아키텍처와 성능을 맥락에 맞게 평가한다.


- **기반:** LPVIMO-SAM은 개념적으로 LVI-SAM 27 및 LIO-SAM 28 제품군의 확장이다. 이들은 요인 그래프 기반 LIO 시스템이라는 핵심 아키텍처를 공유한다.
- **주요 차이점:** LPVIMO-SAM은 세 가지 중요한 센서 모달리티를 추가한다. 논문 실험에서 나타난 성능 향상을 강조할 것이다.22 예를 들어, 지그재그 궤적에서 LPVIMO-SAM은 LIO-SAM보다 정확도를 약 65.7% 향상시켰다. 긴 궤적에서는 LIO-SAM을 괴롭혔던 심각한 Z축 누적 오차를 완화했다. LIO-SAM이 완전히 실패한 라이다 성능 저하 환경에서 LPVIMO-SAM은 더 오랫동안 안정적인 추정치를 유지할 수 있었다.22 편광 비전의 추가는 LVI-SAM이 여전히 가질 수 있는 저특징 장면에 대한 강건성을 제공한다.37


- **핵심 기술:** VINS-Mono는 고도로 최적화된 최적화 기반 단안 VIO 시스템으로 10, VIO 성능의 벤치마크이다.
- **비교:** 비교는 강건성에 초점을 맞출 것이다. 순수 시각-관성 시스템인 VINS-Mono는 본질적으로 열악한 조명, 빠른 움직임, 저특징 환경에 취약하다.11 라이다와 편광 카메라를 갖춘 LPVIMO-SAM은 바로 이러한 실패 모드를 극복하도록 명시적으로 설계되었다.22 VINS-Mono는 경량이지만, LPVIMO-SAM은 이를 희생하는 대신 더 넓은 작동 조건에서 우수한 강건성을 확보했다.


- **핵심 기술:** ORB-SLAM3는 아마도 가장 진보된 특징 기반 시각-관성 SLAM 시스템으로, 강건한 장기 작동을 위한 다중 지도 아틀라스(Atlas) 시스템을 특징으로 한다.50 ORB 특징과 최대 사후 확률(MAP) 추정에 의존한다.
- **비교:** ORB-SLAM3의 강점은 충분한 시각적 특징이 있는 환경에서의 다재다능함과 높은 정확도이다. 그러나 ORB 특징에 의존하기 때문에 다른 시각 시스템과 마찬가지로 저특징 및 열악한 조명 문제에 취약하다.52 LPVIMO-SAM의 라이다 및 편광 비전 사용은 이러한 시나리오에서 근본적인 이점을 제공한다. 더욱이, ORB-SLAM3는 LPVIMO-SAM의 광학 흐름 센서처럼 Z축 누적 오차에 대한 직접적인 제약을 통합하지 않으므로, 루프 클로저가 없는 장기 실행에서 높이 누적 오차에 더 취약할 수 있다.


다음 표는 주요 SOTA SLAM 프레임워크 간의 비교를 요약하여 독자에게 빠른 참조 가이드를 제공한다.

**표 1: SOTA SLAM 프레임워크 비교 분석**

| 기능                 | **LPVIMO-SAM**                                               | **LIO-SAM / LVI-SAM**                                        | **VINS-Mono**                                          | **ORB-SLAM3**                                                |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| **주요 센서**        | 라이다, 편광 카메라, IMU, 지자기 센서, 광학 흐름 22          | 라이다, (LVI-SAM의 경우 카메라), IMU 27                      | 단안 카메라, IMU 11                                    | 단안/스테레오/RGB-D 카메라, IMU 50                           |
| **백엔드 방법**      | 요인 그래프 (iSAM2) 37                                       | 요인 그래프 (iSAM2) 28                                       | 최적화 (슬라이딩 윈도우) 10                            | MAP 추정 / 번들 조정 (g2o) 50                                |
| **결합 전략**        | 강결합 22                                                    | 강결합 27                                                    | 강결합 (시각-관성) 25                                  | 강결합 (시각-관성) 54                                        |
| **저특징 환경 대응** | **주요 강점:** 편광 비전이 기하학적 단서(표면 법선) 제공.22 라이다가 구조 제공. | 라이다 기하학에 의존. LVI-SAM의 시각 부분은 어려움을 겪음.27 | **주요 약점:** 실패 또는 누적 오차 발생 가능성 높음.11 | **주요 약점:** ORB 특징에 의존하므로 실패 가능성 높음.52     |
| **누적 오차 완화**   | **주요 강점:** 지자기 센서(방위), 광학 흐름(속도, 높이), 루프 클로저.22 | 루프 클로저, GPS(사용 가능한 경우). Z축 누적 오차에 취약.22  | 루프 클로저. 루프 없이는 스케일 누적 오차 발생.20      | **주요 강점:** 다중 지도 아틀라스 시스템과 강건한 루프 클로저.50 |
| **핵심 혁신**        | 5개 센서 융합, 특히 저하된 환경을 위한 편광 비전 22          | 요인 그래프 최적화를 통한 고효율 LIO 28                      | 온라인 보정을 통한 고강건성 및 경량 단안 VIO 25        | 강건하고 평생 사용 가능한 SLAM을 위한 다중 지도 시스템(아틀라스). 불가지론적 카메라 모델 50 |


이 섹션은 LPVIMO-SAM의 구체적인 내용을 넘어, 이처럼 복잡한 다중 모드 시스템에 내재된 더 넓고 종종 간과되는 공학적, 이론적 과제들을 논의한다.


서로 다른 주파수(예: IMU >1kHz, 라이다 10-20Hz, 카메라 20-30Hz)로 작동하는 5개의 비동기 센서로부터의 데이터를 정밀하게 시간적으로 정렬하는 것은 매우 중요하다.16 하드웨어 수준의 동기화(예: 공통 클럭 신호)나 정교한 소프트웨어 기반 타임스탬핑 및 보간법이 필수적이다. 강결합 시스템에서 작은 시간 오차는 큰 추정 오차로 이어질 수 있기 때문이다.


5개 센서 스위트에 대한 전체 외재적(공간적) 및 내재적 파라미터 세트를 정확하게 결정하는 것은 엄청난 실제적 과제이다. 이는 모든 센서 쌍 사이의 6자유도 변환을 찾는 것을 포함한다. 보정 오류는 최적화 백엔드가 극복하지 못할 수 있는 시스템적 편향을 유발할 수 있다.26 이 작업의 복잡성은 이러한 시스템의 실제적인 배포에 있어 상당한 장벽이 된다.


논문은 실시간 성능을 주장하지만, 이 섹션에서는 계산 부하를 비판적으로 분석할 것이다. 5개의 센서 스트림을 융합하고 점점 커지는 요인 그래프를 최적화하는 것은 계산 집약적이다.56 포함되는 요인의 수, 최적화 창의 크기, 그리고 달성 가능한 실시간 주파수 사이의 트레이드오프를 논의할 것이다. 이는 특히 드론과 같이 자원이 제한된 플랫폼과 더 많은 처리 능력을 가진 지상 로봇 사이에서 중요한 고려사항이다.


이 섹션에서는 시스템의 주요 설계 목표인 강건성을 분석한다. 이중 서브시스템 아키텍처(PVIS 및 LIMOS)가 어떻게 점진적 성능 저하(graceful degradation)를 가능하게 하는지 논의할 것이다.22 예를 들어, 먼지가 많은 환경에서 라이다가 실패하면 31 시스템은 PVIS로 대체 작동할 수 있다. 카메라가 태양광에 의해 눈이 멀면 LIMOS가 대신할 수 있다. 이러한 중복성은 핵심 기능이지만, 센서 고장을 감지하고 그래프에서 요인의 가중치를 다시 조정해야 하는 상태 추정 로직에 복잡성을 더한다.30

더 많은 센서를 융합하려는 움직임은 SLAM 문제의 병목 현상이 **알고리즘적 독창성**에서 **시스템 엔지니어링의 복잡성**으로 이동하고 있음을 나타낸다. 초기 SLAM은 핵심 알고리즘(EKF, 파티클 필터, 그래프 최적화)을 발명하는 데 중점을 두었다. LPVIMO-SAM과 같은 시스템은 새로운 수학적 패러다임을 발명하는 것이 아니라, 이미 확립된 것(iSAM2를 통한 요인 그래프 최적화)을 사용하고 있다. 이들의 주요 혁신은 새로운 센서(편광 카메라)의 *통합*과 *융합 아키텍처* 자체에 있다. 이는 이러한 시스템을 구축하는 가장 어려운 부분이 더 이상 수학만이 아니라, 하드웨어 선택, 강건한 다중 센서 보정, 여러 드라이버와 하드웨어에 걸친 정밀한 시간 동기화, 전체 소프트웨어 스택의 계산 예산 관리와 같은 힘든 엔지니어링 작업이라는 것을 의미한다. 따라서 강건한 SLAM의 미래는 혁신적인 새로운 최적화 알고리즘보다는 복잡한 센서 융합을 더 쉽게 접근할 수 있도록 만드는 표준화되고 잘 설계된 하드웨어/소프트웨어 플랫폼에 있을 수 있다. LPVIMO-SAM과 같은 시스템을 처음부터 복제하는 어려움은 이러한 복잡한 프레임워크를 오픈소스로 공개하는 것의 가치를 부각시킨다.38



핵심 내용을 요약하면, LPVIMO-SAM은 더 간단한 SLAM 시스템의 실패 모드를 체계적으로 해결함으로써 모든 환경에서의 자율성을 향한 중요한 진전을 나타낸다. 주요 기여는 텍스처가 없거나 반사적인 표면을 처리하기 위한 편광 비전의 새로운 통합과, 장기적인 누적 오차에 대한 절대적인 제약을 제공하기 위한 보조 센서(지자기 센서, 광학 흐름)의 사용이다.


- **딥러닝 통합:** 고전적인 구성 요소를 딥러닝 모델로 대체할 가능성을 논의한다. 예를 들어, FAST 코너 대신 SuperPoint와 같은 학습된 특징 추출기를 사용하거나 53, 깊이 완성 또는 동적 객체 분할을 위해 신경망을 사용하는 것이다.54
- **시맨틱 SLAM:** 기하학적 지도를 넘어 시맨틱 지도로 나아가는 것이다. 풍부한 센서 데이터의 융합은 벽을 매핑하는 것뿐만 아니라 "벽"으로 식별하고, 요인 그래프 프레임워크 내에서 "자동차"나 "사람"과 같은 동적 객체를 감지하고 추적하는 데 사용될 수 있다.58
- **적응형 융합:** 시스템이 실시간으로 각 센서 스트림의 품질을 자동으로 평가하고 요인 그래프 최적화에서 가중치(노이즈 공분산)를 동적으로 조정하는 더 지능적인 메커니즘을 개발하는 것이다.30
- **복잡성 단순화:** 이러한 복잡한 다중 모드 시스템의 배포 장벽을 낮추기 위한 자동 보정 및 자체 동기화 기술에 대한 연구이다.


LPVIMO-SAM과 같은 시스템은 강건성의 한계를 넓히는 동시에 복잡성이 증가하는 추세를 보여준다. 궁극적인 목표는 정확하고 강건할 뿐만 아니라 확장 가능하고 저렴하며 배포하기 쉬운 SLAM 시스템으로 남아있다. LPVIMO-SAM은 그 목표를 향한 길에서 가치 있는 데이터 포인트를 제공한다.


1. Understanding SLAM in Robotics and Autonomous Vehicles - Flyability, accessed July 19, 2025, https://www.flyability.com/blog/simultaneous-localization-and-mapping
2. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar - MDPI, accessed July 19, 2025, https://www.mdpi.com/2032-6653/16/2/56
3. Simultaneous localization and mapping - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping
4. 46. Simultaneous Localization and Mapping - CS@Columbia, accessed July 19, 2025, https://www.cs.columbia.edu/~allen/F19/NOTES/slam_paper.pdf
5. Overview of SLAM. What is SLAM? What is Simultaneous... | by Luis Bermudez | machinevision | Medium, accessed July 19, 2025, https://medium.com/machinevision/overview-of-slam-50b7f49903b7
6. Introduction to SLAM (Simultaneous Localization and Mapping) | Ouster, accessed July 19, 2025, https://ouster.com/insights/blog/introduction-to-slam-simultaneous-localization-and-mapping
7. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed July 19, 2025, https://www.mathworks.com/discovery/slam.html
8. The Types of SLAM Algorithms - Medium, accessed July 19, 2025, https://medium.com/@nahmed3536/the-types-of-slam-algorithms-356196937e3d
9. A Tutorial on Graph-Based SLAM, accessed July 19, 2025, http://www2.informatik.uni-freiburg.de/~stachnis/pdf/grisetti10titsmag.pdf
10. (PDF) A Review of Visual-Inertial Simultaneous Localization and Mapping from Filtering-Based and Optimization-Based Perspectives - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/327064224_A_Review_of_Visual-Inertial_Simultaneous_Localization_and_Mapping_from_Filtering-Based_and_Optimization-Based_Perspectives
11. LET-SE2-VINS: A Hybrid Optical Flow Framework for Robust Visual–Inertial SLAM - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12251901/
12. Graph SLAM - Washington, accessed July 19, 2025, [https://courses.cs.washington.edu/courses/cse571/23sp/slides/L09/Lecture09_Modern%20SLAM.pdf](https://courses.cs.washington.edu/courses/cse571/23sp/slides/L09/Lecture09_Modern SLAM.pdf)
13. Post-integration based point-line feature visual SLAM in low-texture environments - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12033347/
14. Square Root SAM - Frank Dellaert, accessed July 19, 2025, https://dellaert.github.io/files/Dellaert06ijrr.pdf
15. Real-time Smoothing and Mapping - Ananth Ranganathan, accessed July 19, 2025, http://www.ananth.in/Projects/Entries/2010/12/7_Real-time_Smoothing_and_Mapping.html
16. VIO Revolution in Navigation and Positioning - Inertial Labs, accessed July 19, 2025, https://inertiallabs.com/vio-revolution-in-navigation-and-positioning/
17. Visual Inertial Odometry (VIO) / PX4 User Guide, accessed July 19, 2025, https://docs.px4.io/v1.11/en/computer_vision/visual_inertial_odometry.html
18. Visual Inertial Odometry (VIO) | PX4 Guide (main), accessed July 19, 2025, https://docs.px4.io/main/en/computer_vision/visual_inertial_odometry.html
19. What is Visual-Inertial Odometry (VIO) - Activeloop, accessed July 19, 2025, https://www.activeloop.ai/resources/glossary/visual-inertial-odometry-vio/
20. A review of visual inertial odometry from filtering and optimisation perspectives, accessed July 19, 2025, https://www.researchgate.net/publication/282429809_A_review_of_visual_inertial_odometry_from_filtering_and_optimisation_perspectives
21. How Visual Inertial Odometry (VIO) Works - Think Autonomous, accessed July 19, 2025, https://www.thinkautonomous.ai/blog/visual-inertial-odometry/
22. LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.20380v1
23. PL-VIO: Tightly-Coupled Monocular Visual–Inertial Odometry Using Point and Line Features, accessed July 19, 2025, https://www.mdpi.com/1424-8220/18/4/1159
24. [2504.20380] LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2504.20380
25. SuperVINS: A visual-inertial SLAM framework integrated deep learning features - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.21348v1
26. Neural Approach to Coordinate Transformation for LiDAR–Camera Data Fusion in Coastal Observation - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/20/6766
27. MIT Open Access Articles LVI-SAM: Tightly-coupled Lidar-Visual- Inertial Odometry via Smoothing and Mapping, accessed July 19, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/144047/2104.10831.pdf?sequence=2&isAllowed=y
28. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 19, 2025, https://github.com/TixiaoShan/LIO-SAM
29. GO-LIO: Lidar-IMU SLAM Algorithm Based on Ground Optimization - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/379089633_GO-LIO_Lidar-IMU_SLAM_Algorithm_Based_on_Ground_Optimization
30. Fusion LiDAR-Inertial-Encoder data for High-Accuracy SLAM - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.11870v1
31. Present and Future of SLAM in Extreme Underground Environments - JPL Robotics, accessed July 19, 2025, https://www-robotics.jpl.nasa.gov/media/documents/2208.01787.pdf
32. Glass Detection Using Polarization Camera and LRF for SLAM in Environment with Glass | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/348471049_Glass_Detection_Using_Polarization_Camera_and_LRF_for_SLAM_in_Environment_with_Glass
33. Lidar-Monocular Visual Odometry using Point and Line Features - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/344981766_Lidar-Monocular_Visual_Odometry_using_Point_and_Line_Features
34. Improving SLAM Techniques with Integrated Multi-Sensor Fusion for 3D Reconstruction - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014387/
35. Factor Graphs for Navigation Applications: A Tutorial, accessed July 19, 2025, https://navi.ion.org/content/71/3/navi.653
36. Factor Graphs and GTSAM, accessed July 19, 2025, https://gtsam.org/tutorials/intro.html
37. [Literature Review] LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - Moonlight | AI Colleague for Research Papers, accessed July 19, 2025, https://www.themoonlight.io/en/review/lpvimo-sam-tightly-coupled-lidarpolarization-visioninertialmagnetometeroptical-flow-odometry-via-smoothing-and-mapping
38. LVIO-Fusion:Tightly-Coupled LiDAR-Visual-Inertial Odometry and Mapping in Degenerate Environments | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/378568851_LVIO-FusionTightly-Coupled_LiDAR-Visual-Inertial_Odometry_and_Mapping_in_Degenerate_Environments
39. Linear parameter-varying control - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Linear_parameter-varying_control
40. Simulate linear parameter-varying (LPV) systems - Simulink - MathWorks, accessed July 19, 2025, https://www.mathworks.com/help/control/ref/lpvsystem.html
41. Linear Parameter-Varying Models - MATLAB & Simulink - MathWorks, accessed July 19, 2025, https://www.mathworks.com/help/control/ug/linear-parameter-varying-models.html
42. (PDF) Analysis And Control Of Linear Parameter-Varying Systems - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/2915499_Analysis_And_Control_Of_Linear_Parameter-Varying_Systems
43. Peng Guo - CatalyzeX, accessed July 19, 2025, [https://www.catalyzex.com/author/Peng%20Guo](https://www.catalyzex.com/author/Peng Guo)
44. Polarimetric Dense Monocular SLAM - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content_cvpr_2018/papers/Yang_Polarimetric_Dense_Monocular_CVPR_2018_paper.pdf
45. Polarization Explained: The Sony Polarized Sensor - LUCID Vision Labs, accessed July 19, 2025, https://thinklucid.com/tech-briefs/polarization-explained-sony-polarized-sensor/
46. SLAM in Environment with Glass Using Degree of Polarization from Polarization Camera and Depth Information from LRF偏光カメラの偏光度とLRFの距離情報を用いたガラス環境対応SLAM - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/348237079_SLAM_in_Environment_with_Glass_Using_Degree_of_Polarization_from_Polarization_Camera_and_Depth_Information_from_LRFpianguangkameranopianguangdutoLRFnojuliqingbaowoyongitagarasuhuanjingduiyingSLAM
47. Robot navigation using stereo vision and polarization imaging - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/266338039_Robot_navigation_using_stereo_vision_and_polarization_imaging
48. Pop-up SLAM: Semantic Monocular Plane SLAM for Low-texture Environments - Carnegie Mellon University, accessed July 19, 2025, https://www.cs.cmu.edu/~kaess/pub/Yang16iros.pdf
49. [1703.07334] Pop-up SLAM: Semantic Monocular Plane SLAM for Low-texture Environments - arXiv, accessed July 19, 2025, https://arxiv.org/abs/1703.07334
50. [2007.11898] ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - ar5iv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2007.11898
51. VINS-Dimc: A Visual-Inertial Navigation System for Dynamic Environment Integrating Multiple Constraints - MDPI, accessed July 19, 2025, https://www.mdpi.com/2220-9964/11/2/95
52. Introduction and application of ORB-SLAM3 - MATOM.AI, accessed July 19, 2025, https://matom.ai/insights/slam/
53. [Literature Review] SuperPoint-SLAM3: Augmenting ORB-SLAM3 with Deep Features, Adaptive NMS, and Learning-Based Loop Closure - Moonlight | AI Colleague for Research Papers, accessed July 19, 2025, https://www.themoonlight.io/en/review/superpoint-slam3-augmenting-orb-slam3-with-deep-features-adaptive-nms-and-learning-based-loop-closure
54. An Adaptive ORB-SLAM3 System for Outdoor Dynamic Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/23/3/1359
55. (PDF) LIC-Fusion: LiDAR-Inertial-Camera Odometry - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/335831148_LIC-Fusion_LiDAR-Inertial-Camera_Odometry
56. Underwater SLAM Meets Deep Learning: Challenges, Multi-Sensor Integration, and Future Directions - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12157327/
57. A Comprehensive Review of Sensor Fusion Techniques in SLAM: Trends, Challenges and Future Directions | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/390220531_A_Comprehensive_Review_of_Sensor_Fusion_Techniques_in_SLAM_Trends_Challenges_and_Future_Directions
58. Multimodal Semantic SLAM with Probabilistic Data Association - DSpace@MIT, accessed July 19, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/137995.2/doherty_icra2019_revised.pdf?sequence=4&isAllowed=y
59. Top arXiv papers, accessed July 19, 2025, http://121.43.168.64:10060/s/com/scirate/G.https/?date=2025-04-30&page=15&range=3
60. Derui Shan - CatalyzeX, accessed July 19, 2025, [https://www.catalyzex.com/author/Derui%20Shan](https://www.catalyzex.com/author/Derui Shan)
61. FAST-LIVO: Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/366611115_FAST-LIVO_Fast_and_Tightly-coupled_Sparse-Direct_LiDAR-Inertial-Visual_Odometry
62. [2501.11893] DynoSAM: Open-Source Smoothing and Mapping Framework for Dynamic SLAM - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2501.11893


[항공-지상 이미지 매칭 (Aerial-Ground Image Matching)](./index.md)
# 저고도 군용 UAV를 위한 항공 지도 기반 기술의 종합 분석



현대 전쟁의 패러다임은 무인 항공기(Unmanned Aerial Vehicle, UAV), 즉 드론의 등장으로 근본적인 변화를 맞이했다. 초기 정보/감시/정찰(ISR) 자산의 역할을 넘어, 이제 드론은 군수 보급, 정밀 타격, 전자전 등 다방면에 걸쳐 없어서는 안 될 핵심 전력으로 자리매김했다.1 이러한 변화는 시장 규모의 폭발적인 성장으로 증명된다. 전 세계 군용 드론 시장은 지정학적 긴장 고조와 각국의 국방비 지출 증가에 힘입어 2032년까지 471억 6천만 달러 규모에 이를 것으로 전망된다.4 이는 단순히 군사적 수요에 국한되지 않는다. 물류, 농업, 시설 점검 등 민간 부문에서의 드론 활용이 폭발적으로 증가하며 거대한 산업 기반을 형성하고 기술 발전을 가속화하고 있다.4

이러한 군사 및 민간 부문의 동반 성장은 기술 개발의 선순환 구조를 만들어낸다. 민간 시장의 규모는 센서, 프로세서와 같은 핵심 부품의 가격을 낮추고 비행 제어, 컴퓨터 비전 등 소프트웨어 분야의 혁신을 촉진한다. 군은 이렇게 상용화된 기술(COTS) 또는 군용 상용 기술(MCOTS)을 활용하여 개발 비용과 기간을 단축할 수 있다. 반대로, 적대적 환경에서의 극단적인 신뢰성과 성능을 요구하는 군사적 필요성(예: GPS 거부 환경 항법)은 기술의 최전선을 개척하며, 이렇게 개발된 견고한 솔루션은 도심 항공 모빌리티(UAM)나 핵심 인프라 점검과 같은 고부가가치 민간 분야로 기술이전(Spin-off)되는 상호 보완적 관계를 형성한다.7 따라서 군용 UAV의 전략적 우위는 이제 광범위한 상업용 드론 및 로보틱스 생태계의 건전성과 혁신 역량에 점점 더 의존하게 되었다.


드론의 혁신적인 운용 능력에도 불구하고, 그 기반에는 치명적인 아킬레스건이 존재한다. 바로 위성항법시스템(Global Navigation Satellite System, GNSS)에 대한 과도한 의존이다. 미국의 GPS가 대표적인 GNSS는 현대 군사 작전의 신경망과도 같지만, 적대적 환경에서는 매우 취약하다.

첫째, GNSS 신호는 약 20,000 km 상공의 위성에서 송출되어 지표면에 도달할 때쯤에는 매우 미약해진다.9 이로 인해 의도적인 전파 방해(Jamming)나 허위 신호를 보내 수신기를 속이는 기만(Spoofing) 공격에 극도로 취약하다.9 둘째, 이러한 취약성은 더 이상 이론에 머무르지 않는다. 우크라이나 전쟁과 같은 최근의 분쟁들은 광범위하고 효과적인 GPS 재밍이 현실화되었음을 명백히 보여주었다.3 이로 인해 GPS에만 의존하는 드론은 사실상 무력화될 수 있으며, 이는 대체 항법 기술의 확보가 생존과 직결되는 문제임을 시사한다. 마지막으로, 정밀도의 문제도 존재한다. 일반적인 GPS는 수 미터의 오차를 가지며, 이를 보완하기 위한 실시간 이동 측위(Real-Time Kinematic, RTK) 기술은 지상 기준국에 의존하고 여전히 신호 교란에 취약하다.11 정밀 타격이나 복잡한 지형에서의 임무 수행 시, 이러한 정확도의 손실은 임무 실패로 이어진다.

이러한 현실은 군사 기술의 설계 철학 자체를 바꾸고 있다. 과거 GPS 재밍은 특정 상황에서 발생하는 고급 위협으로 간주되었지만, 이제는 GPS 거부 상황이 작전의 기본 전제 조건(Baseline Operational Condition)으로 인식되고 있다.3 따라서 비(非)GPS 항법 기술은 더 이상 '백업' 시스템이 아니라, 처음부터 시스템에 완벽하게 통합되어야 하는 '공동 주력(Co-primary)' 시스템으로 격상되었다. 미래 군용 UAV 작전의 성패는 GPS 없이도 얼마나 효과적으로 임무를 완수할 수 있느냐에 따라 결정될 것이다.


특히 '저고도' 작전 환경은 드론 항법 기술에 더욱 복잡한 도전 과제를 제기한다. 지상에 근접하여 비행하는 것은 건물, 전선, 수목 등 지형 및 장애물과의 충돌 위험을 급격히 증가시킨다.1 또한, 지상 기반의 대공 위협에 더 쉽게 노출된다. 이러한 환경에서는 GPS의 신호 지연이나 잠재적 오류를 감수할 여유가 없으며, 매우 신뢰성 높은 실시간 항법 및 장애물 회피 능력이 필수적이다. 한국을 포함한 여러 국가에서 '가상 항로(SKYWAY)'와 같은 저고도 무인기 교통관리(UTM) 체계 구축을 서두르는 이유도 바로 이 때문이다.1 결국, 저고도라는 특수한 작전 영역은 본 보고서에서 다루게 될 지도 기반 및 환경 상호작용 항법 기술 개발의 가장 강력한 동인으로 작용하고 있다.


GPS 거부 환경에서 드론의 생존과 임무 완수를 보장하기 위해 다양한 대체 항법 기술들이 연구되고 있다. 이 기술들은 크게 관성 기반의 추측 항법, 사전 제작된 지도를 활용하는 절대 위치 측정, 그리고 실시간으로 지도를 생성하며 위치를 파악하는 환경 상호작용 기술로 분류할 수 있다.


관성항법장치(Inertial Navigation System, INS)는 외부 신호 없이 드론의 위치를 추정하는 가장 기본적인 기술이다. 이 시스템의 핵심은 가속도계와 자이로스코프로 구성된 관성측정장치(Inertial Measurement Unit, IMU)이다.17 IMU는 드론의 선형 가속도와 각속도를 측정하며, 시스템은 이 값을 시간에 대해 적분하여 속도를 계산하고, 속도를 다시 적분하여 이동 거리를 산출함으로써 출발점 대비 현재 위치를 추정(Dead Reckoning)한다.18

그러나 INS는 치명적인 단점을 내포하고 있다. 바로 '누적 오차(Cumulative Drift)'이다. IMU 센서에서 발생하는 아주 미세한 측정 오차가 적분 과정을 거치면서 시간이 지남에 따라 눈덩이처럼 불어나 위치 오차를 급격히 증가시킨다.17 대부분의 드론에 탑재되는 저가의 MEMS(Micro-Electro-Mechanical Systems) 기반 IMU의 경우, 이 오차는 불과 몇 초 만에 수십 미터에 달할 수 있어, 다른 보정 수단 없이는 장시간의 정밀 항법에 부적합하다.17


이 기술군은 사전에 제작되어 드론에 저장된 지리 참조 지도와 실시간 센서 데이터를 비교하여 드론의 절대 좌표를 찾는 방식이다. 이는 크게 지형 참조 항법과 영상 기반 항법으로 나뉜다.


지형 참조 항법(Terrain Referenced Navigation, TRN)은 드론에 장착된 레이더 고도계나 라이다(LiDAR) 같은 센서가 기체 하부 지형의 고도 값을 연속적으로 측정하는 방식으로 작동한다.22 이렇게 측정된 고도 프로파일(profile)은 기체에 저장된 수치표고모델(Digital Elevation Model, DEM)과 비교된다. 시스템은 측정된 프로파일과 가장 일치하는 지형을 지도 상에서 찾아내어 드론의 현재 위치를 정확하게 특정한다.15

TRN은 레이더를 사용할 경우 기상 조건이나 주야간에 관계없이 안정적인 성능을 보이며, 전파 방해나 기만이 어렵다는 장점이 있다.15 이러한 특성 덕분에 순항 미사일이나 군용 항공기에서 오랫동안 사용되어 온 성숙한 기술이다.15 하지만 물 위나 사막, 만년설과 같이 고도 변화가 거의 없는 평탄한 지형에서는 고유한 프로파일을 얻을 수 없어 성능이 급격히 저하된다. 또한, 항법의 정확도는 전적으로 사전에 구축된 지도의 정밀도와 해상도에 의존한다는 한계가 있다.15


영상 기반 항법(Image-Based Navigation, IBN) 또는 장면 대조 항법(Scene Matching)은 TRN의 시각적 버전이라고 할 수 있다. 내장된 카메라가 지상의 모습을 촬영하면, 이 영상은 사전에 구축된 위성사진이나 항공사진 데이터베이스와 비교된다.21 시스템은 현재 촬영된 장면과 가장 일치하는 지역을 지도 데이터베이스에서 찾아내어 위치를 결정한다.

IBN은 비교적 저렴하고 보편적인 카메라 센서를 활용할 수 있다는 장점이 있다. 그러나 주야간과 같은 광원 변화, 구름이나 안개 같은 기상 조건, 그리고 계절에 따른 식생 변화(녹음, 단풍, 적설)에 매우 민감하다.24 이러한 환경 변화는 실제 지형의 모습을 참조 영상과 다르게 만들어 매칭 실패를 유발할 수 있다. 또한, 광범위한 지역을 비행하기 위해서는 방대한 용량의 영상 데이터베이스를 저장해야 하는 부담도 따른다.24


이 기술군은 드론이 미지의 주변 환경을 실시간으로 탐색하여 지도를 생성하고, 동시에 그 지도 안에서 자신의 위치를 추정하는 방식이다. 대표적인 기술이 SLAM이다.


SLAM(Simultaneous Localization and Mapping)은 이름 그대로 미지의 환경에서 '지도 작성'과 '위치 추정'을 동시에 수행하는 기술이다.10 이는 "정확한 지도를 만들려면 자신의 위치를 알아야 하고, 정확한 위치를 알려면 신뢰할 수 있는 지도가 필요하다"는 '닭과 달걀'의 문제와 같다. SLAM 알고리즘은 이 문제를 반복적인 연산을 통해 점진적으로 해결한다.

- **라이다 SLAM (LiDAR SLAM):** 라이다 센서를 사용하여 주변 환경의 3차원 점군(point cloud) 데이터를 획득한다. 연속적으로 스캔되는 점군 데이터를 정합(matching)하여 드론의 움직임을 계산하고, 이를 누적하여 지도를 구축한다. 매우 정밀하고 빛이 없는 환경에서도 작동하지만, 센서 가격이 비싸고 연산량이 많다는 단점이 있다.18
- **시각 SLAM (Visual SLAM, V-SLAM):** 하나 이상의 카메라를 사용하여 환경 내의 모서리, 질감 등 시각적 특징점(feature)을 추출하고 추적한다. 프레임 간 특징점의 움직임을 분석하여 카메라의 이동 경로를 계산하고, 이를 바탕으로 특징점의 3차원 위치를 추정하여 지도를 생성한다.20 저비용의 경량 센서를 사용하지만, 저조도나 특징이 없는(texture-less) 환경에서는 성능이 저하되며, 빠른 움직임으로 인한 모션 블러(motion blur) 현상에 취약하다.25

이러한 기술들의 특성은 임무 프로파일에 따라 항법 기술 선택이 단순한 기술적 결정이 아닌 전술적 결정임을 시사한다. 예를 들어, 사전에 정찰이 완료된 적진 깊숙한 곳으로 침투하는 임무에는 고품질의 참조 지도를 활용하는 TRN이나 IBN이 적합하다. 이러한 '지도 기반(Map-Based)' 항법은 전역 좌표계(global coordinate system) 기준으로 절대 위치를 제공한다는 장점이 있다. 반면, 시가전이나 재난 현장 수색처럼 사전 정보가 없거나 환경이 급변하는 곳에서는 SLAM과 같은 '지도 생성(Map-Building)' 기술이 필수적이다. 이는 작전의 유연성과 사전 정보 의존성 탈피라는 장점을 제공한다. 결국, 현대 군은 이러한 두 가지 유형의 기술 포트폴리오를 모두 확보해야만 다양한 작전 요구에 효과적으로 대응할 수 있다.

또한, 각각의 항법 기술은 명확한 실패 시나리오를 가지고 있다. INS는 시간이 지나면 표류하고, IBN은 날씨에, TRN은 평지에, V-SLAM은 특징 없는 환경에 취약하다. 단일 센서에 의존하는 것은 환경적 또는 적대적 요인에 의해 악용될 수 있는 단일 실패점(single point of failure)을 만드는 것과 같다. 군사 시스템에서 이는 용납될 수 없는 취약점이다. 따라서 가장 논리적인 해결책은 여러 보완적인 센서를 결합하여 하나의 약점을 다른 하나의 강점으로 보완하는 '센서 퓨전(Sensor Fusion)'이다. 미래 군용 UAV 항법의 핵심은 단 하나의 완벽한 센서를 찾는 것이 아니라, IMU, 카메라, 라이다, 레이더 등 다양한 센서로부터의 입력을 지능적으로 융합하여 단일의 견고한 항법 해(solution)를 도출하는 기술을 마스터하는 데 있다.


센서들이 드론의 '감각'이라면, 이 감각들을 통합하고 해석하여 지능적인 행동으로 이끄는 '두뇌' 역할을 하는 것이 바로 알고리즘이다. 센서 퓨전과 인공지능은 GPS 거부 환경 항법 기술의 성능을 결정하는 핵심적인 소프트웨어 기반 기술이다.


시각-관성 주행 거리 측정(Visual-Inertial Odometry, VIO)은 카메라와 IMU의 데이터를 긴밀하게 결합하여 각 센서가 단독으로 가질 수 있는 한계를 극복하고 월등한 항법 성능을 달성하는 기술이다.27 이 기술의 기본 원리는 상호 보완에 있다. IMU는 수백에서 수천 헤르츠(

Hz)에 달하는 매우 빠른 속도로 드론의 움직임(가속도, 각속도)을 측정하여 다음 위치를 예측하는 역할을 한다. 반면, 카메라는 상대적으로 느린 주파수(예: 30~60Hz)로 작동하지만, 외부 환경의 시각 정보를 통해 IMU의 누적 오차를 주기적으로 보정하여 항법의 정확성을 담보한다.17

VIO는 특히 다음과 같은 항법의 고질적인 문제들을 효과적으로 해결한다 17:

- **실제 거리 측정(Metric Scale) 문제 해결:** 단일 카메라(monocular VO)만으로는 촬영된 경로의 형태는 알 수 있지만, 실제 크기(예: 1미터를 이동했는지 10미터를 이동했는지)를 파악할 수 없다. VIO에서는 IMU의 가속도계가 물리적인 가속도를 측정하므로, 이를 통해 시각 정보에 실제 '미터(m)' 단위의 스케일을 부여하여 경로를 현실 세계에 정밀하게 정합시킬 수 있다.
- **고속 기동 및 모션 블러 대응:** 카메라의 프레임 속도는 한계가 있어 드론이 빠르게 움직이면 영상이 흐려지거나(motion blur) 프레임 간의 변화가 너무 커서 특징점 추적에 실패할 수 있다. 이때 IMU는 빠른 업데이트 속도로 프레임 사이의 '빈틈'을 채워주어 고속 기동 중에도 안정적인 위치 추적을 가능하게 한다.
- **특징 없는 환경 극복:** 벽이나 하늘처럼 시각적 특징이 부족한 환경을 마주하면 카메라는 위치 추정에 실패한다. VIO 시스템은 이런 상황에서 잠시 IMU의 추측 항법에 의존하여 해당 구간을 통과한 뒤, 다시 시각적 특징이 나타나면 오차를 보정하며 항법을 이어갈 수 있다.
- **순수 회전(Pure Rotation) 문제 해결:** 시각 항법은 서로 다른 두 시점에서 촬영한 영상의 시차(baseline)를 이용해 거리를 삼각 측량하는데, 제자리에서 회전만 할 경우 이 시차가 발생하지 않아 위치 추정에 실패한다. 그러나 VIO에서는 IMU의 자이로스코프가 회전을 직접 측정하므로, 이러한 순수 회전 상황에서도 문제없이 자세를 추정할 수 있다.

이러한 VIO 기술은 세계 유수의 연구 그룹들에 의해 발전해왔다. 펜실베이니아 대학(UPenn)의 MSCKF와 취리히 연방 공대(ETH Zurich)의 ROVIO는 확장 칼만 필터(EKF)를 기반으로 한 VIO를 개발했으며, 카네기 멜런 대학(CMU)은 여러 시점의 측정값을 한 번에 최적화하는 방식을 채택하여 더욱 정밀한 성능을 추구하고 있다.27


센서 퓨전 알고리즘은 불확실하고 노이즈가 섞인 여러 센서 데이터를 수학적으로 결합하여 가장 신뢰도 높은 단일의 추정치를 만들어내는 기술이다.

- **칼만 필터 (Kalman Filter):** 확장 칼만 필터(Extended Kalman Filter, EKF)는 센서 퓨전의 고전적인 확률 기반 도구다. EKF는 '예측'과 '수정'의 두 단계를 반복한다. 먼저 IMU 데이터로 드론의 다음 상태를 **예측**하고, 카메라나 GPS 등 새로운 측정값이 들어오면 이 정보를 이용해 예측치를 **수정**한다.22 이는 많은 '약결합' 및 '강결합' 융합 시스템의 핵심을 이룬다.
- **파티클 필터 (Particle Filter):** 주로 지형 참조 항법(TRN)에 사용되는 기법으로, 수많은 '가설 입자(particle)'들을 통해 드론의 가능한 위치를 확률적으로 표현한다. 새로운 센서 측정값이 들어올 때마다 측정값과 일치하지 않는 입자들은 제거되고, 일치하는 입자들은 강화되어 결국 가장 가능성 높은 위치로 수렴하게 된다. 이는 지형 매칭과 같이 비선형성이 매우 강한 문제에 강건한 성능을 보인다.15
- **최적화 (Bundle Adjustment):** 칼만 필터처럼 데이터를 순차적으로 처리하는 대신, V-SLAM이나 VIO에서 주로 사용하는 최적화 기반 방식은 일정 시간 동안의 과거 측정값(keyframe)들을 모아, 이 모든 측정값을 가장 잘 설명하는 단 하나의 최적 경로를 한 번에 계산한다. 연산 비용은 더 높지만, 전역적인 일관성을 고려하기 때문에 종종 더 높은 정확도를 제공한다.27

이처럼 동일한 센서 조합을 사용하더라도 어떤 퓨전 알고리즘을 선택하느냐에 따라 시스템의 성능과 특성이 크게 달라진다. EKF 기반 방식은 연산 속도가 빨라 소형 드론에 적합하고, 최적화 방식은 더 많은 연산 자원을 요구하지만 높은 정확도를 얻을 수 있다. 이는 단순히 최고의 센서를 확보하는 것만으로는 기술적 우위를 점할 수 없으며, 그 하드웨어의 잠재력을 최대한 끌어낼 수 있는 정교한 알고리즘 개발 역량과 이를 위한 인재 확보가 무엇보다 중요함을 시사한다. 소프트웨어와 알고리즘의 우위가 곧 핵심 경쟁력인 시대이다.


심층 강화 학습(Deep Reinforcement Learning, DRL)은 기존의 항법 패러다임을 근본적으로 바꾸는 기술이다. 전통적인 방식이 지도 위에서 경로를 '계획'하고 이를 '실행'하는 것이었다면, DRL은 인공지능 '에이전트'가 시뮬레이션 환경에서 수많은 시행착오를 통해 항법 '정책(policy)' 자체를 학습하는 방식이다.30

DRL의 작동 메커니즘은 다음과 같다. 에이전트는 목표 지점으로 이동하거나 장애물을 피하는 등 바람직한 행동을 할 때마다 '보상(reward)'을 받고, 충돌과 같은 원치 않는 행동을 하면 '벌점(penalty)'을 받는다. PPO, TD3, DQN과 같은 DRL 알고리즘은 수백만 번의 시뮬레이션 시행을 거치며 이 누적 보상을 최대화하도록 에이전트의 심층 신경망(Deep Neural Network)을 최적화한다.32 이 과정을 통해 인간 프로그래머가 명시적으로 설계하기 어려운 복잡하고 강건한 항법 행동이 스스로 발현된다.

DRL은 특히 빽빽한 숲이나 도심 협곡처럼 기존의 경로 계획 알고리즘이 적용되기 어려운 매우 복잡하고 동적인 환경에서 강력한 성능을 발휘한다. 드론이 예측 불가능한 장애물에 대해 반사적이고 지능적으로 대응할 수 있게 만들기 때문이다.34 현재 연구는 언리얼 엔진(Unreal Engine)과 같은 현실적인 시뮬레이터에서 정책을 학습시킨 후, 이를 실제 드론에 이식하는 방향으로 활발히 진행되고 있다.30

DRL은 자율 항법 능력의 비약적인 도약을 의미하지만, 동시에 새로운 도전 과제를 제시한다. DRL 에이전트는 인간이 생각하지 못한 창의적이고 효과적인 전략을 개발할 수 있다는 엄청난 장점을 지닌다. 그러나 신경망의 의사결정 과정이 불투명한 '블랙박스'처럼 작동하여 특정 결정의 원인을 파악하기 어렵다는 문제가 있다.33 이는 군사 시스템에 요구되는 시험, 평가, 검증 및 인증(TEV&V) 과정을 매우 복잡하게 만든다. 또한, AI를 속이기 위해 의도적으로 조작된 센서 입력을 가하는 '적대적 공격(Adversarial Attack)'은 새로운 유형의 심각한 위협으로 부상하고 있다. 따라서 DRL이 핵심 군사 시스템에 성공적으로 도입되기 위해서는 '설명가능 AI(Explainable AI, XAI)', 적대적 AI 공격에 대한 방어 기술, 그리고 신뢰성을 보장하기 위한 새로운 검증 프로토콜 개발이 병행되어야 한다. 99%의 확률로 작동하는 시스템을 넘어, 100% 신뢰할 수 있는 시스템으로 만드는 것이 핵심 과제이다.


군사 임무의 특수한 요구사항, 특히 적에게 탐지되지 않는 저피탐성(스텔스)과 네트워크 기반의 작전 회복탄력성(resilience)을 충족시키기 위해 더욱 진보된 항법 개념들이 연구되고 있다.


레이더나 라이다와 같은 능동형 센서는 전자기파를 방사하여 주변을 탐지한다. 이 신호는 적의 탐지 장비에 의해 포착될 수 있으며, 이는 드론의 위치를 노출시키고 스텔스 성능을 무력화시킨다. 적진 깊숙이 침투하는 ISR이나 타격 임무에서 이러한 전자파 신호를 최소화하는 것은 생존과 직결된 문제이다.

- **수동형 레이더 TRN (Passive Radar TRN):** 이 기술은 자체 레이더를 사용하는 대신, 상업용 FM/TV 방송 송신탑이나 다른 아군/적군의 레이더 등 제3의 '기회 표적 조사기(illuminators of opportunity)'에서 방사된 전파가 지형에 반사되어 돌아오는 신호를 수신하여 지형도를 그리는 방식이다.37 이를 통해 어떠한 신호도 방출하지 않고 지형 참조 항법을 수행할 수 있어 생존성을 크게 향상시킨다.37
- **수동형 시각 기반 항법:** 영상 기반 항법(IBN)이나 시각 SLAM(V-SLAM)은 외부의 빛을 수신하기만 할 뿐 자체적으로 신호를 방출하지 않으므로 본질적으로 수동형이며 스텔스 작전에 적합하다.24
- **신개념 수동형 기술:** 더 나아가, 드론 자체의 프로펠러 소음을 수동형 음향 탐지 소스로 활용하여 해저 지형을 영상화하거나(수중 무인정 기술이지만 원리적으로 적용 가능), SAR 위성에서 송출하는 신호를 수동으로 수신하여 활용하는 등 혁신적인 연구도 진행되고 있다.38

이러한 기술 동향은 미래 군용 항법 시스템이 '정확도'와 '전자파 신호' 사이의 스펙트럼 위에서 설계되어야 함을 보여준다. 능동형 센서는 기상 조건에 강하고 정확하지만 전자파 노출이 크며 15, 수동형 센서는 스텔스에 유리하지만 정확도가 낮거나 외부 요인에 의존적일 수 있다.37 임무의 성격이 이 스펙트럼상의 어느 지점을 요구하는지를 결정하게 될 것이다. 예를 들어, 적진 깊숙한 곳에서의 정찰 임무는 스텔스를 최우선으로 하여 수동형 IBN이나 수동형 레이더 TRN을 채택할 것이고, 비교적 안전한 후방에서의 군수 보급 임무는 전천후 신뢰성을 위해 능동형 레이더나 라이다 SLAM을 선호할 것이다. 가장 진보된 시스템은 위협 환경에 따라 능동/수동 모드를 동적으로 전환하여, 위치 보정이 꼭 필요할 때만 능동 센서를 사용하고 나머지 시간에는 침묵을 유지하는 '신호 관리(Signature Management)' 기능을 갖추게 될 것이다.


협력 항법(Cooperative Navigation)은 드론 편대나 드론-지상로봇과 같은 여러 자산이 서로의 위치와 센서 정보를 공유하여 그룹 전체의 항법 정확도를 향상시키는 개념이다.41

이 기술의 핵심 원리는 네트워크를 통한 상호 보정에 있다. 만약 편대 내의 한 드론이 양호한 GPS 신호를 수신하거나 TRN을 통해 정확한 위치를 확보했다면, 이 드론은 GPS가 거부되는 지역에 있는 다른 동료 드론들에게 이동형 '유사 위성(Pseudolite)' 역할을 할 수 있다. 다른 드론들은 이 '위치가 알려진' 드론까지의 거리와 방향을 측정하여 자신의 위치를 높은 정확도로 삼각 측량할 수 있다.41

구체적인 적용 사례로, 지상의 무인 차량(Unmanned Ground Vehicle, UGV)은 상공을 비행하는 UAV에게 매우 안정적이고 기하학적으로 유리한 위치 기준점을 제공할 수 있다. UAV는 UGV와의 상대 거리를 측정함으로써 GPS가 교란되는 환경에서도 위치 오차를 획기적으로 줄일 수 있다.41 이는 단일 플랫폼의 생존성을 넘어, 다영역(Multi-Domain) 작전 전체의 회복탄력성을 강화하는 중요한 수단이 된다.


글로벌 기술 동향 속에서, 대한민국은 민간 및 군사 분야 모두에서 독자적이고 강인한 항법 생태계를 구축하기 위한 체계적인 국가 전략을 추진하고 있다. 이는 단순한 기술 개발을 넘어, '기술 주권' 확보를 목표로 하는 거시적인 움직임이다.


- **한국형 위성항법시스템 (KPS):** 약 3조 7천억 원의 예산을 투입하여 2035년까지 독자적인 위성항법시스템을 구축하는 KPS(Korean Positioning System) 사업은 한국 국가 전략의 핵심이다.12 KPS는 한반도 및 주변 지역에 암호화되고 재밍에 강한 고유의 항법 신호를 제공함으로써, 핵심 군사 및 인프라 분야에서 해외 GNSS에 대한 의존도를 완전히 제거하는 것을 목표로 한다.12 이는 본 보고서 서론에서 지적한 GNSS의 취약성에 대한 직접적이고 근본적인 대응책이다.
- **저고도 교통관리체계 (K-UAM):** 정부는 미래의 고밀도 드론 교통을 안전하게 관리하기 위해 'SKYWAY'라는 개념의 무인기 교통관리(UTM) 체계 구축을 선제적으로 추진하고 있다.1 이는 하늘에 가상의 도로와 신호체계를 만들어 민간 배송 드론부터 군수 지원 드론에 이르기까지 모든 비행체가 안전하게 운항할 수 있는 인프라를 조성하는 사업이다.1


- **국방과학연구소 (ADD):** ADD는 군용 무인 시스템의 핵심 자율화 기술 개발을 선도하고 있다. ADD는 드론이 외부 위협과 환경 변화에 스스로 대응하여 최적의 비행 경로를 결정하는 자율 항법 및 임무 관리 기술을 성공적으로 개발했다.7 또한, 새로운 무기체계 개발 기간을 단축하기 위해 다양한 플랫폼에 적용할 수 있는 모듈화된 항법 소프트웨어 개발 플랫폼도 구축하고 있다.49
- **한국전자통신연구원 (ETRI):** ETRI는 센서 퓨전과 같은 기반 기술 연구에 특화되어 있다. ETRI의 연구는 라이다와 IMU 데이터를 EKF로 융합하여 장애물이 많은 환경에서 안정적인 항법을 구현하고, 정교한 비행 궤적 생성 알고리즘을 개발하는 등 높은 수준의 기술력을 보여준다.27 이러한 원천 기술은 ADD가 개발하는 통합 시스템의 핵심 '빌딩 블록' 역할을 한다.


- **LIG넥스원:** LIG넥스원은 KPS 사업의 핵심 파트너로서 위성 수신기, 센서 등 주요 구성품 개발을 담당하고 있다.43 또한, 축적된 국방 기술을 바탕으로 200kg급 고중량 화물 수송이 가능한 수소연료전지 기반 카고 드론을 개발하고 있으며, 이는 군수 지원에 직접적으로 활용될 수 있다.43
- **한화시스템:** 한화시스템은 국내 방산 생태계의 또 다른 한 축이다. 해양용 항법장치를 자체 개발하는 한편 52, 우주 기반 감시정찰 분야에서 독보적인 위치를 차지하고 있다. 특히 세계 최고 수준인 0.25m 해상도의 소형 SAR(합성 개구 레이더) 위성을 독자 개발하여, 주야간 및 악천후에 관계없이 고해상도 영상 정보를 획득할 수 있는 능력을 확보했다.53 이 SAR 위성이 생성하는 고품질의 수치표고모델(DEM)과 참조 영상은 TRN 및 IBN 시스템에 필수적인 국가적 핵심 자산이다.

이러한 각 부문의 활동들은 개별적인 프로젝트의 나열이 아니라, 하나의 거대한 목표를 향해 유기적으로 연결된 '수직 통합형 국가 전략'의 일부로 해석되어야 한다. 정부가 KPS라는 전략적 인프라를 구축하고 12, ADD와 ETRI 같은 국책 연구소가 핵심 알고리즘을 개발하며 7, LIG넥스원과 한화시스템 같은 방산업체가 이를 실제 하드웨어와 플랫폼으로 구현하는 구조이다.43 예를 들어, 한화의 SAR 위성이 생성한 지도는 LIG넥스원의 유도무기나 ADD의 자율 드론에 탑재될 수 있다. ETRI의 퓨전 알고리즘은 국내 업체가 제작한 비행 제어 컴퓨터에 내장될 수 있다. 그리고 이 모든 시스템은 최종적으로 국가 고유의 KPS 신호를 활용하여 외부의 영향으로부터 자유로운, 완결된 국내 기술 생태계를 형성하게 된다.44 이는 해외 기술 종속에서 벗어나 독자적인 안보 역량을 확보하고, 지정학적 환경에 최적화된 맞춤형 시스템을 개발하기 위한 강력한 '전략적 해자(Strategic Moat)'를 구축하는 과정이다.


본 보고서는 저고도 군용 드론의 GPS 거부 환경 항법 기술에 대한 다각적인 분석을 통해, 기술적 동향과 전략적 함의를 도출하였다. 이를 바탕으로 주요 기술을 비교 분석하고, 미래 연구 방향과 국가적 차원의 전략적 필수 요건을 제언하고자 한다.


특정 군사 임무에 '만능'인 단일 항법 기술은 존재하지 않는다. 임무의 성격에 따라 최적의 기술 조합은 달라지며, 다양한 능력을 갖춘 포트폴리오 구축이 필수적이다. 아래 표는 주요 항법 기술의 특성을 비교하고, 이를 바탕으로 임무별 최적의 솔루션을 제시한다.

**표 1: 적대적 환경 하 UAV 항법 기술 비교 매트릭스**

| 기술 구분           | 잠재적 정확도                  | GPS 의존성 | 전천후 성능           | 연산 비용 | 스텔스 성능 (전자파 방출) | 지도 요구사항    | 주요 군사 임무 적용              |
| ------------------- | ------------------------------ | ---------- | --------------------- | --------- | ------------------------- | ---------------- | -------------------------------- |
| **GPS/GNSS**        | 높음 (RTK 시 cm급)             | 예         | 낮음 (대기 영향)      | 낮음      | 능동 (신호 수신)          | 불필요           | 정밀 타격, 일반 비행             |
| **INS (단독)**      | 낮음 (시간에 따라 급격히 저하) | 아니요     | 높음                  | 낮음      | 수동                      | 불필요           | 단기 항법 보조                   |
| **TRN (레이더)**    | 중간~높음                      | 아니요     | 높음                  | 중간      | 능동                      | 사전 구축 (DEM)  | 전천후 침투, 군수 보급           |
| **IBN (시각)**      | 중간~높음                      | 아니요     | 낮음                  | 중간~높음 | 수동                      | 사전 구축 (영상) | 주간 정찰, 스텔스 침투           |
| **V-SLAM**          | 중간 (누적 오차 발생)          | 아니요     | 낮음                  | 높음      | 수동                      | 동적 생성        | 실내/시가전, 미지 환경 탐사      |
| **LiDAR SLAM**      | 높음                           | 아니요     | 중간 (강우/안개 영향) | 매우 높음 | 능동                      | 동적 생성        | 정밀 지도 작성, 장애물 회피      |
| **VIO (시각+관성)** | 높음                           | 아니요     | 낮음                  | 높음      | 수동                      | 동적 생성        | 고기동 비행, 시가전, 스텔스 정찰 |

이 분석을 바탕으로 임무 유형별 최적의 항법 솔루션을 다음과 같이 구체화할 수 있다.

- **적진 깊숙한 침투 및 ISR 임무:** 최대의 스텔스 성능이 요구된다. 사전 구축된 지도 데이터를 기반으로 하는 수동형 IBN과 VIO를 결합하고, 필요시 수동형 TRN으로 보강하는 방식이 이상적이다. 전자파 방출을 최소화하는 것이 핵심이다.
- **전장 군수 보급 임무:** 악천후를 포함한 모든 조건에서의 신뢰성이 최우선이다. GNSS/RTK를 기본으로 하되, 신호 두절 시 라이다 SLAM이나 레이더 기반 TRN으로 즉시 전환하여 안정적인 운항을 보장하는 강력한 증강 시스템이 필요하다.
- **시가전 및 근접항공지원 임무:** 높은 기동성과 동적인 장애물 회피 능력이 필수적이다. VIO와 라이다 센서를 퓨전하고, 복잡하고 예측 불가능한 환경에 대응하기 위해 심층 강화 학습(DRL)으로 훈련된 항법 정책을 탑재하는 것이 효과적이다.


미래 전장에서의 우위를 확보하기 위해 다음 분야에 대한 선도적인 연구 개발이 요구된다.

- **실시간 지도 업데이트 및 융합:** '지도 기반' 항법과 '지도 생성' 항법의 경계를 허무는 기술이다. TRN/IBN을 사용하는 드론이 비행 중 파괴된 다리와 같은 지형 변화를 감지했을 때, 이를 실시간으로 참조 지도에 반영하고, 이 정보를 인근의 다른 아군 자산과 공유하는 기술이 필요하다.
- **인지 및 적대적 AI:** 항법 시스템이 AI에 더 많이 의존하게 됨에 따라, 다음 기술의 격전지는 적대적 AI 공격(조작된 센서 입력으로 AI를 교란)에 대한 방어 기술과, 공역 내 다른 객체의 '의도'를 추론할 수 있는 '인지(Cognitive)' AI 개발이 될 것이다.
- **SWaP-C 최적화:** 더 작고 저렴하며 소모성인 드론 플랫폼에도 첨단 항법 기능을 탑재하기 위해 센서의 소형화, 저전력화, 그리고 알고리즘의 연산 효율성을 극대화하는 연구(크기, 무게, 전력, 비용 최적화)가 지속되어야 한다.
- **비전통적 신호원 활용:** LTE/5G 이동통신 신호, Wi-Fi 신호 등 주변에 존재하는 다양한 '기회 신호(Signals of Opportunity)'를 항법 솔루션에 통합하여, GPS가 없을 때 활용할 수 있는 중첩된 예비 항법 체계를 구축하는 연구가 필요하다.18


대한민국이 항법 기술 분야에서 지속적인 주도권을 확보하고 기술 주권을 공고히 하기 위해 다음의 전략적 노력이 요구된다.

- **기초 R&D에 대한 지속적 투자:** 센서 퓨전, 심층 강화 학습(DRL), 설명가능 AI(XAI)와 같은 장기적인 기술 경쟁력을 좌우하는 핵심 원천 기술 분야에 대한 국가 차원의 꾸준한 투자가 필수적이다.
- **민/군 기술 생태계 활성화:** 민간 시장의 규모와 혁신 속도, 그리고 군사 분야의 높은 기술적 요구사항을 동시에 활용할 수 있도록 기술의 상호 이전(Spin-off/Spin-on)을 장려하는 정책적 지원이 필요하다.8
- **국가 차원의 시험평가(T&E) 프레임워크 구축:** GPS 거부 환경 항법 및 AI 기반 자율 시스템의 신뢰성과 안전성을 검증하기 위한 국가적 표준과 전용 시험 시설을 구축해야 한다.
- **인간-기계 협업(Human-Machine Teaming) 연구:** 드론의 자율성이 높아질수록 인간 운용자의 역할은 '조종사'에서 다수의 자율 시스템을 지휘하는 '임무 지휘관'으로 변화한다. 드론 편대를 효과적으로 통제하고 AI의 의사결정을 신뢰하며 감독하기 위한 새로운 형태의 인터페이스와 제어 패러다임에 대한 연구가 시급하다.42


1. 지상도로망의 상공 확장기반 가상 SKYWAY 항법 및 관제체계(μUTM)에 관한 연구, accessed July 1, 2025, https://www.koreascience.kr/article/CFKO201634662509374.pdf
2. SE프로세스를 적용한 UTM 환경의 항법 오차 산출 필요성 검토, accessed July 1, 2025, https://www.jksaa.org/archive/view_article?pid=jksaa-28-4-47
3. [한반도 신무기 대백과] "북러 '드론협력' 본질은 무기의 북 위탁생산" - Radio Free Asia, accessed July 1, 2025, https://www.rfa.org/korean/weekly_program/c2e0bc15d55cd55cbc18b3c4c2e0bb34ae30b300bc31acfc/drone-russia-lancet-harop-gps-ukraine-weapon-02142025131226.html
4. 품목별ICT 시장동향, accessed July 1, 2025, https://www.globalict.kr/upload_file/kms/202312/46257328044971008.pdf
5. 군용 드론 시장의 성장과 지정학적 요인: 2025년 현재 동향과 미래 전망 - Goover, accessed July 1, 2025, https://seo.goover.ai/report/202503/go-public-report-ko-59530fe7-64be-42c7-97c7-298329388ff6-0-0.html
6. 『4차 산업혁명 기반 드론 산업』 국내외 동향연구 보고서, accessed July 1, 2025, https://care.gb.go.kr/Common/board_egov/download.jsp?B_STEP=70035300&B_ORDER=1
7. 군, '알아서 피해 가는' 무인기 자율항법 기술 개발 - 연합뉴스, accessed July 1, 2025, https://www.yna.co.kr/view/AKR20210511043600504
8. 드론 잡는 안티드론도 우리 손으로 만듭니다 - THE SSEN LIG, accessed July 1, 2025, https://www.thessen-lig.com:10010/svcShowWebZinDetail.do?contentsSeq=671
9. 자율항법연구실 - AutoNav Lab. - 인하대학교, accessed July 1, 2025, [http://autonav.inha.ac.kr/etc/2024%20%EC%9E%90%EC%9C%A8%ED%95%AD%EB%B2%95%EC%97%B0%EA%B5%AC%EC%8B%A4%20%ED%99%8D%EB%B3%B4%20%EB%B8%8C%EB%A1%9C%EC%85%94_%ED%95%9C%EA%B8%80ver.pdf](http://autonav.inha.ac.kr/etc/2024 자율항법연구실 홍보 브로셔_한글ver.pdf)
10. Smart Skies: New Methods for UAVs to Navigate Where GPS Fails | Newswise, accessed July 1, 2025, https://www.newswise.com/articles/smart-skies-new-methods-for-uavs-to-navigate-where-gps-fails
11. 실내 드론/무인 항만...오차 10㎝ 초정밀 항법위성이 만들 미래 - 한국경제, accessed July 1, 2025, https://www.hankyung.com/article/2024052008041
12. [기고] 시급한 위성항법시스템 구축과 제반 기술 경쟁력 강화 - 동아일보, accessed July 1, 2025, https://www.donga.com/news/It/article/all/20240403/124305937/1
13. GPS 음영지역에서 딥러닝을 활용한 드론 자율 착륙 - Korea Science, accessed July 1, 2025, https://koreascience.kr/article/CFKO202323672006095.pdf
14. 일본 스타트업, GPS수신 불가 지역서 자율비행하는 드론 개발 - 로봇신문, accessed July 1, 2025, https://www.irobotnews.com/news/articleView.html?idxno=22236
15. Drone-Based Radar Terrain-Referenced Navigation Using a Low-Cost Automotive-Class FMCW Radar to Enable GNSS-Denied Navigation - MDPI, accessed July 1, 2025, https://www.mdpi.com/2673-4591/88/1/11
16. 저고도 UAV를 위한 가상 공중도로 설계/제공/관리기술 개발 기획, accessed July 1, 2025, https://www.codil.or.kr/filebank/original/RK/OTKCRK210297/OTKCRK210297.pdf
17. VIO, IMU, INS - has,me - 티스토리, accessed July 1, 2025, https://hasme.tistory.com/3
18. 드론 정밀 측위 기술 동향 - ETRI Electronics and ..., accessed July 1, 2025, [https://ettrends.etri.re.kr/ettrends/202/0905202002/011-019.%20%EC%9D%B4%EC%A0%95%ED%98%B8_202%ED%98%B8.pdf](https://ettrends.etri.re.kr/ettrends/202/0905202002/011-019. 이정호_202호.pdf)
19. [TECHNOLOGY FOCUS] 드론 최신 기술 동향과 전망 - 헬로티 –매일 만나는 첨단 산업, 경제, IT 소식, accessed July 1, 2025, https://www.hellot.net/news/article_print.html?no=54939
20. Vision-Based Navigation Techniques for Unmanned Aerial Vehicles: Review and Challenges - MDPI, accessed July 1, 2025, https://www.mdpi.com/2504-446X/7/2/89
21. Vision-aided terrain referenced navigation for unmanned aerial vehicles = 무인항공기 영상기반 지형참조항법 알고리듬 연구 - 카이스트 도서관, accessed July 1, 2025, http://library.kaist.ac.kr/search/detail/view.do?bibCtrlNo=566071&flag=t
22. 지형 험준도를 고려한 프로파일 기반 지형참조항법과 ... - Korea Science, accessed July 1, 2025, https://koreascience.kr/article/JAKO201310559989738.pdf
23. defensetimes.co.kr, accessed July 1, 2025, [http://defensetimes.co.kr/article/downloadfile.php?type=txt&bbs_id=DefenceTimes_news&page=&doc_num=3290&PHPSESSID=6cc1abae8a5813e17b3571932f4974a2](http://defensetimes.co.kr/article/downloadfile.php?type=txt&bbs_id=DefenceTimes_news&page&doc_num=3290&PHPSESSID=6cc1abae8a5813e17b3571932f4974a2)
24. Terrain referenced navigation | Download Scientific Diagram - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/figure/Terrain-referenced-navigation_fig2_257581446
25. The Future of Drone Mapping with SLAM Technology, accessed July 1, 2025, https://www.thedroneu.com/blog/slam-technology/
26. Simultaneous Localization and Mapping (SLAM) and Data Fusion in Unmanned Aerial Vehicles: Recent Advances and Challenges - MDPI, accessed July 1, 2025, https://www.mdpi.com/2504-446X/6/4/85
27. 드론 자율비행 기술 동향 - ETRI Electronics and Telecommunications ..., accessed July 1, 2025, [https://ettrends.etri.re.kr/ettrends/189/0905189001/001-011_%EA%B9%80%EC%88%98%EC%84%B1.pdf](https://ettrends.etri.re.kr/ettrends/189/0905189001/001-011_김수성.pdf)
28. KR102268980B1 - Gps 음영지역에서 드론의 자율 주행을 위한 위치 ..., accessed July 1, 2025, https://patents.google.com/patent/KR102268980B1/ko
29. Advances and challenges in UAV navigation based on visual SLAM, accessed July 1, 2025, https://www.ewadirect.com/proceedings/tns/article/view/16389
30. Deep RL-based Autonomous Navigation of Micro Aerial Vehicles (MAVs) in a complex GPS-denied Indoor Environment - arXiv, accessed July 1, 2025, https://arxiv.org/html/2504.05918v1
31. Deep RL-based Autonomous Navigation of Micro Aerial Vehicles (MAVs) in a complex GPS-denied Indoor Environment - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2504.05918
32. Autonomous Unmanned Aerial Vehicle Navigation using Reinforcement Learning: A Systematic Review - Western Engineering, accessed July 1, 2025, https://www.eng.uwo.ca/electrical/faculty/grolinger_k/docs/RL-for-UVA-navigation.pdf
33. Autonomous Navigation of Drones Using Explainable Deep Reinforcement Learning in Complex Environments - International Institute of Informatics and Systemics, accessed July 1, 2025, https://www.iiis.org/CDs2024/CD2024Summer//papers/SA031PL.pdf
34. UAV Autonomous Navigation Based on Deep Reinforcement Learning in Highly Dynamic and High-Density Environments - MDPI, accessed July 1, 2025, https://www.mdpi.com/2504-446X/8/9/516
35. Deep-reinforcement-learning-based UAV autonomous navigation and collision avoidance in unknown environments - SciOpen, accessed July 1, 2025, https://www.sciopen.com/article/10.1016/j.cja.2023.09.033
36. Vision-Based Deep Reinforcement Learning of Unmanned Aerial Vehicle (UAV) Autonomous Navigation Using Privileged Information - MDPI, accessed July 1, 2025, https://www.mdpi.com/2504-446X/8/12/782
37. UAV Detection with Passive Radar: Algorithms, Applications, and Challenges - MDPI, accessed July 1, 2025, https://www.mdpi.com/2504-446X/9/1/76
38. System Design and Analysis for Energy-Efficient Passive UAV Radar Imaging System using Illuminators of Opportunity - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2010.00179
39. Stealth Technology Breakthrough By Navy Researchers Could Revolutionize Underwater Drone Capabilities - The Debrief, accessed July 1, 2025, https://thedebrief.org/stealth-technology-breakthrough-by-navy-researchers-could-revolutionize-underwater-drone-capabilities/
40. Terrain-following radar - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Terrain-following_radar
41. UGV-to-UAV Cooperative Ranging for Robust Navigation in GNSS-Challenged Environments, accessed July 1, 2025, https://navigationlab.wvu.edu/files/d/963db649-711e-4b2f-992a-2a72fa8f5f0b/coopnav.pdf
42. Autopilots | UAV Navigation, accessed July 1, 2025, https://www.uavnavigation.com/taxonomy/term/80
43. LIG넥스원, '장거리공대지유도무기' 어떻게 발전시키나? - 에이티엔뉴스, accessed July 1, 2025, https://www.atnnews.co.kr/news/articleView.html?idxno=55981
44. 한국형 위성항법시스템(KPS)을 활용한 군사용 드론의 위치추적 및 ..., accessed July 1, 2025, http://journal.kidet.or.kr/archive/view_article?pid=jkidt-4-3-8
45. 국방과학연구소, 무인기 스스로 대응하는 무인기 자율화 기술 개발 - 보안뉴스, accessed July 1, 2025, https://m.boannews.com/html/detail.html?idx=97371
46. ADD '알아서 척척 위험 피해가는' 무인기 자율항법기술 국내 개발 - 문화일보, accessed July 1, 2025, https://www.munhwa.com/article/11240587
47. 장애물 알아서 피해 가는 무인기 자율항법 기술 국내 개발 - 세계일보, accessed July 1, 2025, https://www.segye.com/newsView/20210511505734
48. 국방과학연구소, '무인기 자율화 기술' 개발 - 민주신문, accessed July 1, 2025, http://www.iminju.net/news/articleView.html?idxno=72365
49. ADD, 유도무기 표적 콕 찍는 '항법SW 자동 생성' 기술 확보 - 국방신문, accessed July 1, 2025, https://www.gukbangnews.com/news/articleView.html?idxno=4117
50. 유도무기 - LIG Nex1, accessed July 1, 2025, https://www.lignex1.com/business/gddWpnList.do
51. LIG넥스원, 서울 아덱스 2023에서 '유도무기 명가' 확인 - 국방신문, accessed July 1, 2025, https://www.gukbangnews.com/news/articleView.html?idxno=6600
52. 한화, 자체 개발한 항법장치 민수 분야 진출 | 한화그룹, accessed July 1, 2025, https://www.hanwha.co.kr/newsroom/media_center/news/news_view.do?seq=7010
53. 순수 우리 기술로 소형 SAR 위성 개발... 이동형 안티드론 시스템 구축 ..., accessed July 1, 2025, https://www.donga.com/news/Politics/article/all/20240928/130117369/2




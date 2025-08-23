# Livox MID-70 LiDAR 기술적 고찰


Livox MID-70은 저속 자율주행 및 모바일 로보틱스 분야를 목표로 설계된 고성능 3D LiDAR(Light Detection and Ranging) 센서다. 기존 기계식 회전형 LiDAR 센서와는 근본적으로 다른 작동 원리와 가격 정책을 통해 시장에 진입했으며, 특정 응용 분야에서 독자적인 입지를 구축하고 있다. 본 보고서는 Livox MID-70의 핵심 기술, 성능 사양, 데이터 특성, 응용 분야 및 소프트웨어 생태계를 심층적으로 분석한다. 또한, Ouster OS0, Velodyne Puck 등 주요 경쟁 제품과의 비교를 통해 시장 내 위치를 명확히 하고, 실제 시스템 통합 시 고려해야 할 기술적 한계와 과제를 종합적으로 고찰하여 잠재적 사용자에게 전략적 지침을 제공하는 것을 목표로 한다.


Livox MID-70의 경쟁력은 전통적인 LiDAR 기술의 패러다임을 벗어난 독창적인 스캐닝 메커니즘과 이를 통한 비용 효율적 설계에 기인한다. 이 섹션에서는 센서의 성능 특성과 시장 포지셔닝을 결정하는 핵심 기술들을 상세히 분석한다.


MID-70의 가장 큰 기술적 특징은 비반복 스캐닝(NRS, Non-Repetitive Scanning) 패턴을 채택했다는 점이다.1 이는 프레임마다 동일한 라인을 반복적으로 스캔하는 전통적인 기계식 LiDAR와 근본적인 차이를 보인다. NRS 방식에서는 레이저의 스캐닝 궤적이 프레임마다 반복되지 않으므로, 통합 시간(integration time)이 길어질수록 포인트 클라우드의 밀도와 커버리지가 점진적으로 증가하는 특성을 가진다.1

이 독특한 '꽃 모양' 또는 '로제트 스타일(rosette-style)' 패턴은 레이저/수신부 전체를 회전시키는 대신, 한 쌍의 리슬리 프리즘(Risley prisms, 쐐기 모양의 유리 렌즈)을 서로 반대 방향으로 회전시켜 레이저 빔의 경로를 조향함으로써 생성된다.6 이러한 반도체 기반(solid-state) 접근 방식은 기계적 구동 부품을 최소화하여 신뢰성을 높이고 비용을 절감하는 핵심 설계 철학을 반영한다.2

MID-70의 스캐닝 패턴은 극좌표 방정식 $r = a \cos(k\theta)$로 정의되는 로제트 곡선으로 수학적 모델링이 가능하다. 한 학술 연구에서는 MID-70의 k 값이 100으로, 초당 200개의 '꽃잎'을 생성하는 것으로 분석했다.6 이러한 수학적 특성화는 알려진 패턴을 활용하여 객체 추출 및 정합(registration)과 같은 고급 알고리즘을 개발하는 데 중요한 기반이 된다.6


NRS 패턴의 직접적인 결과는 시야각 커버리지(FOV 내에서 레이저 빔이 도달하는 영역의 비율)가 시간에 따라 극적으로 증가하여 100%에 근접한다는 점이다.4 이는 채널 수와 수직 해상도에 의해 커버리지가 고정되는 기계식 LiDAR와 뚜렷하게 대조된다.4

이러한 특성은 특정 응용 분야에서 MID-70의 가치를 극대화한다. 예를 들어, 단일 프레임(약 10ms)의 포인트 클라우드는 불규칙하고 희소하지만(sparse) 9, 500ms와 같은 비교적 짧은 시간 동안 데이터를 누적하면 기존 기계식 LiDAR보다 훨씬 더 조밀하고 완전한 환경 데이터를 구축할 수 있다.3 이는 시간적 해상도를 일부 희생하여 공간적 해상도를 점진적으로 확보하는 전략적 트레이드오프로 해석할 수 있다. 따라서 MID-70은 환경 변화가 통합 시간에 비해 상대적으로 느린 매핑, 정적 객체 인식, 저속 주행 시나리오에 매우 적합하다.5 반대로, 단일 프레임 내에서 작고 빠르게 움직이는 객체를 탐지하는 데는 본질적인 한계를 가진다. 레이저 빔이 해당 프레임 내에 객체를 타격한다는 보장이 없기 때문이다.5

Livox의 공식 문서 및 학술 연구는 기계식 LiDAR와의 정량적 성능 비교를 통해 이러한 특성을 명확히 보여준다.3 아래 표는 통합 시간에 따른 FOV 커버리지 변화를 요약한 것이다.

**표 1: FOV 커버리지(%) 대 통합 시간(s) 비교**

| 통합 시간 (s) | Livox MID-70 커버리지 (%) | 16-라인 기계식 커버리지 (%) | 32-라인 기계식 커버리지 (%) | 64-라인 기계식 커버리지 (%) |
| ------------- | ------------------------- | --------------------------- | --------------------------- | --------------------------- |
| **0.1**       | ~30-40 (32-라인과 유사)   | ~15-20                      | ~30-40                      | ~50-60                      |
| **0.2**       | ~50-60 (32-라인 상회)     | ~15-20                      | ~30-40                      | ~50-60                      |
| **0.5**       | ~80-90 (64-라인 상회)     | ~15-20                      | ~30-40                      | ~50-60                      |
| **1.0**       | >95                       | ~15-20                      | ~30-40                      | ~50-60                      |
| **1.5**       | ~100                      | ~15-20                      | ~30-40                      | ~50-60                      |

주: 커버리지 값은 2 자료를 기반으로 한 추정치이며, 기계식 LiDAR의 커버리지는 수직 FOV에 따라 달라질 수 있음.


MID-70의 합리적인 가격은 설계 및 제조 전략의 직접적인 산물이다.7 고가의 레이저 이미터나 미성숙한 MEMS 스캐너 대신, 저비용의 고품질 반도체 부품을 빛 생성 및 감지에 활용한다.2 스캐닝 유닛을 포함한 전체 광학 시스템은 광학 렌즈 산업에서 사용되는 것과 같이 검증되고 쉽게 구할 수 있는 부품을 사용한다.2

또한, 동축 설계(coaxial design)와 NRS 패턴은 복잡하고 시간이 많이 소요되는 다중 라인 레이저 보정(calibration) 절차를 제거하여 제조 공정을 단순화하고 자동화된 대량 생산을 가능하게 한다.7 이러한 접근은 DJI의 자회사로서 축적된 대량 생산 공급망 관리 노하우를 LiDAR 산업에 적용하여 3D LiDAR 기술의 진입 장벽을 낮추려는 전략적 목표를 반영한다. 이는 단순히 성능으로 경쟁하는 것을 넘어, 서비스 로봇, 물류 자동화, 소규모 매핑 등 기존에는 비용 문제로 LiDAR 도입이 어려웠던 새로운 시장을 창출하려는 시도다.2


이 섹션에서는 제조사가 제공하는 공식 사양과 독립적인 학술 연구 결과를 교차 검증하여 MID-70의 성능을 데이터 기반으로 정밀하게 분석한다.


MID-70의 핵심 기술 사양은 다음과 같다.

**표 2: Livox MID-70 핵심 기술 사양**

| 매개변수                   | 사양                                                       |
| -------------------------- | ---------------------------------------------------------- |
| **레이저 파장**            | 905 nm                                                     |
| **레이저 안전 등급**       | Class 1 (IEC60825-1:2014)                                  |
| **탐지 거리 (@100 klx)**   | 90 m @ 10% 반사율, 130 m @ 20% 반사율, 260 m @ 80% 반사율  |
| **시야각 (FOV)**           | 70.4° (원형)                                               |
| **거리 정밀도 (1σ @ 20m)** | ≤ 2 cm                                                     |
| **각도 정밀도 (1σ)**       | < 0.1°                                                     |
| **포인트 출력률**          | 100,000 points/s (단일 반사), 200,000 points/s (이중 반사) |
| **데이터 지연 시간**       | ≤ 2 ms                                                     |
| **IP 등급**                | IP67                                                       |
| **작동 온도**              | -20℃ ~ 65℃                                                 |
| **평균 소비 전력**         | 8 W                                                        |
| **크기**                   | 97 × 64 × 62.7 mm                                          |
| **무게**                   | 580 g                                                      |

데이터 출처: 16


제조사의 사양은 독립적인 연구를 통해 신뢰성을 검증받아야 한다. 비록 MID-70에 대한 직접적인 연구는 제한적이지만, 동일한 핵심 기술을 공유하는 Livox Mid-40 센서에 대한 정밀한 기하학적 정확도 분석 연구는 MID-70의 성능을 가늠할 중요한 근거를 제공한다.

- **기하학적 정확도:** 해당 연구에 따르면, Mid-40의 조정된 포인트 클라우드의 전체 표준편차는 **1.8 cm**로, 제조사 사양인 2 cm에 매우 근접한 결과를 보였다. 이상치(outlier)를 제거한 후에는 표준편차가 **1.3 cm**까지 향상되었다.17 이는 Livox가 제시하는 정밀도 주장의 신뢰성이 높음을 시사한다.
- **시스템 오차:** 더욱 중요한 발견은 입사각(최대 65°까지), 반사 강도, 스캔 각도 등과 연관된 **유의미한 시스템적 기하 오차(systematic geometric errors)가 발견되지 않았다**는 점이다.17 시스템 오차는 예측 가능하고 패턴 기반의 편차로, 후처리 과정에서 모델링을 통해 보정할 수 있다. 이러한 오차가 없다는 것은 센서의 원시 데이터(raw data) 품질이 높고, 복잡한 보정 모델 없이도 신뢰할 수 있음을 의미한다. 이는 특히 정밀한 기하학적 충실도가 요구되는 매핑 및 측량 분야 사용자에게 매우 중요한 장점이다.
- **근거리 성능:** MID-70은 최소 5 cm의 탐지 거리를 가지지만 7, 5 cm에서 25 cm 사이의 구간에서는 내부 필터링으로 인해 정확도가 저하된다.6 사용자 매뉴얼 역시 5 cm에서 20 cm 사이의 데이터는 정밀도가 보장되지 않으므로 참고용으로만 사용해야 한다고 명시하고 있다.3 한 평가 보고서는 신뢰할 수 있는 측정을 위해 최소 1 m 이상의 거리를 권장하기도 했다.5 이는 시스템 설계 시 반드시 고려해야 할 '성능 도넛' 현상으로, 정밀한 근접 상호작용이 필요한 로봇 팔 조작이나 도킹과 같은 작업에서는 이 불확실성 구간을 회피하도록 센서를 배치하거나, 초음파 센서와 같은 보조 센서를 활용해야 함을 시사한다.


- **방수/방진:** IP67 등급은 센서가 먼지로부터 완벽하게 보호되며 최대 1 m 깊이의 물에서 30분간 견딜 수 있음을 의미한다.16 이는 실외 로봇 및 차량 응용에 적합한 내구성을 보장한다.
- **작동 온도:** -20℃에서 65℃의 넓은 작동 온도 범위를 가지며, 저온 환경에서는 자동 예열 모드가 활성화되어 안정적인 작동을 지원한다.16
- **비와 안개:** Mid 시리즈 사용자 매뉴얼은 비, 안개, 먼지와 같은 대기 중 입자로 인한 노이즈를 효과적으로 줄이기 위한 노이즈 제거 알고리즘이 내장되어 있다고 언급한다.20 다양한 LiDAR를 기후 챔버에서 테스트한 한 학술 연구는 유사 기술을 사용하는 Livox Horizon의 성능을 통해 MID-70의 악천후 성능을 예측할 단서를 제공한다.21
  - **안개 환경:** Livox Horizon은 안개 속에서 뛰어난 성능을 보였다. 가시거리가 30 m일 때 목표물을 탐지하기 시작하여 50 m에서 100% 탐지에 성공했으며, 이는 여러 스피닝 LiDAR보다 우수한 결과였다.
  - **강우 환경:** Horizon은 시간당 80 mm의 강우량까지 80% 이상의 공칭 포인트 수를 유지하며 비교적 안정적인 성능을 나타냈다. 이는 MID-70 기술이 악천후 환경에서 상당한 강건성을 가짐을 시사하지만, 성능 저하는 여전히 고려해야 할 요소다.


MID-70의 독특한 기술적 특성은 특정 응용 분야에 최적화되어 있으며, 이를 지원하기 위한 견고한 소프트웨어 생태계가 함께 제공된다.


MID-70은 저속 자율주행 및 모바일 로보틱스 시장을 명확하게 겨냥하고 있다.7 넓은 FOV와 최소화된 사각지대는 공항, 쇼핑몰, 병원, 물류 단지와 같이 복잡하고 혼잡한 환경을 탐색하는 데 최적화된 특성이다.7

실제 적용 사례로는 가우시안 로보틱스(Gaussian Robotics)의 바닥 청소 로봇, 징둥닷컴(JD.com)의 무인 배송 차량, 생산 라인의 무인 운반차(AGV) 등이 있으며, 이는 해당 분야에서의 높은 산업 수용도를 보여준다.10 특히 가우시안 로보틱스의 1만 대 주문은 상업용 로봇 시장에서 MID-70이 '충분히 좋은(good enough)' 성능을 갖춘 비용 효율적인 3D 인식 솔루션으로 자리 잡았음을 증명한다.23


- **SLAM 및 위치 추정:** NRS 패턴은 모션 보정과 결합될 때 조밀한 지도를 생성하는 데 유리하다. Livox는 불규칙한 포인트 클라우드에서 특징점을 효과적으로 추출하여 안정적인 자세를 추정할 수 있도록 LOAM(Laser Odometry and Mapping) 알고리즘을 개량한 오픈소스 버전을 제공한다.9 다양한 SLAM 알고리즘(Livox-mapping, LOAM-Livox, FAST-LIO2)이 MID-70과 함께 구동되는 영상이 공개되었으며, 알고리즘에 따라 드리프트 수준과 성공률에 차이를 보였다.24
- **장애물 회피 및 객체 인식:** 3D 데이터는 2D LiDAR와 달리 매달린 장애물을 감지하고 경사로를 주행하는 데 본질적인 이점을 제공한다.10 수백 밀리초(예: 500ms) 동안 프레임을 누적하면 포인트 클라우드가 충분히 조밀해져, 심도 카메라가 실패할 수 있는 열악한 조명 조건에서도 팔레트나 보행자와 같은 객체의 윤곽을 정확하게 인식할 수 있다.10


- **Livox SDK:** Livox는 사용자가 센서에 연결하고 포인트 클라우드 데이터를 수신할 수 있는 C/C++ 기반 소프트웨어 개발 키트(SDK)를 제공한다.25 이는 맞춤형 애플리케이션 개발을 위한 가장 기본적인 도구다.
- **Livox Viewer:** 실시간 포인트 클라우드 시각화, 녹화, 기본 분석을 위한 전용 소프트웨어다.3 초기 설정, 테스트 및 진단에 필수적이다. 사용자들이 흔히 겪는 문제 중 하나는 MID-70에 HAP/Mid-360용인 Livox Viewer 2를 사용하려는 시도인데, 이 센서는 구버전인 Livox Viewer 1을 사용해야 한다.29
- **ROS/ROS2 드라이버:** Livox는 로보틱스 운영체제(ROS) 생태계와의 통합을 위해 공식 ROS 드라이버(`livox_ros_driver`)를 제공한다.25 이 드라이버는 데이터를 표준 `PointCloud2` 메시지 형태로 발행하여 SLAM, 내비게이션, 인식 등 방대한 ROS 기반 도구와 호환된다.31

성공적인 시스템 통합은 단순히 센서를 연결하는 것을 넘어, 비표준 데이터 형식에 대한 알고리즘 수준의 적응에 크게 의존한다. 기존의 SLAM 알고리즘들은 기계식 LiDAR의 정형화된 링(ring) 구조 데이터에 최적화되어 있어, MID-70의 불규칙한 포인트 클라우드를 입력하면 성능이 저하될 수 있다.9 따라서 사용자는 Livox가 제공하는 소프트웨어 스택을 채택하거나, 모션 블러 보정 및 희소 데이터 기반 특징점 추출과 같은 NRS 데이터의 고유한 특성을 처리하도록 기존 알고리즘을 수정하는 데 개발 노력을 투자해야 한다.


MID-70의 가치는 경쟁 제품과의 비교를 통해 더욱 명확해진다. 이 섹션에서는 주요 경쟁 센서와 MID-70을 벤치마킹하여 상대적인 강점과 약점, 그리고 시장에서의 전략적 위치를 분석한다.


- **Ouster OS0-32:** 초광각 90° 수직 FOV를 자랑하는 스피닝 기계식 LiDAR로, 근거리 인식 및 로보틱스 분야의 강력한 경쟁자다. 865 nm 파장을 사용하며, 32채널 버전 기준 최대 1,310,720 points/s의 월등히 높은 포인트 출력률을 제공한다.32
- **Velodyne Puck (VLP-16):** 16채널 스피닝 기계식 LiDAR로, 오랫동안 로보틱스 및 초기 자율주행차 개발의 산업 표준으로 자리매김했다. 360° 수평 FOV를 제공하지만 수직 FOV는 30°로 훨씬 좁고, 포인트 출력률은 약 300,000 points/s로 상대적으로 낮다.34


아래 표는 세 가지 센서의 핵심 지표를 직접 비교한 것이다.

**표 3: Livox MID-70 vs. Ouster OS0-32 vs. Velodyne Puck VLP-16 비교 분석**

| 지표                 | Livox MID-70           | Ouster OS0-32      | Velodyne Puck VLP-16 |
| -------------------- | ---------------------- | ------------------ | -------------------- |
| **스캐닝 기술**      | 비반복 스캐닝 (반도체) | 기계식 회전        | 기계식 회전          |
| **수평 FOV**         | 70.4° (원형)           | 360°               | 360°                 |
| **수직 FOV**         | 70.4° (원형)           | 90°                | 30°                  |
| **탐지 거리 (@80%)** | 260 m                  | 75 m               | 100 m                |
| **거리 정밀도 (1σ)** | ≤ 2 cm                 | ±0.8 ~ 4 cm        | ±3 cm                |
| **포인트 출력률**    | 100k ~ 200k points/s   | 1,310,720 points/s | ~300k points/s       |
| **레이저 파장**      | 905 nm                 | 865 nm             | 903 nm               |
| **평균 소비 전력**   | 8 W                    | 14 ~ 20 W          | 8 W                  |
| **크기 (mm)**        | 97×64×62.7             | ∅87×74.2           | ∅103.3×88.9          |
| **무게 (g)**         | 580                    | 430 ~ 500          | ~830                 |
| **IP 등급**          | IP67                   | IP68/IP69K         | IP67                 |
| **가격 (신품 추정)** | $1,099 ~ $1,429        | $2,150 ~           | $800 ~ $4,000        |

데이터 출처: 7

분석 결과, MID-70은 360° 스캐너의 직접적인 대체재가 아니라, 전방 감지 및 사각지대 해소가 중요한 특정 응용 분야에 최적화된 대안임이 드러난다. 로봇 주변의 완전한 상황 인식이 필요하다면 단일 Ouster OS0가 기하학적으로 우수하지만, 자율주행 셔틀이나 지게차와 같이 전방 주행이 주인 환경에서는 MID-70의 넓은 수직/수평 FOV와 중앙부 사각지대 부재가 더 효과적이다.

또한, 시장은 명확히 분화되고 있다. Ouster는 최신 칩과 높은 포인트 출력률을 내세워 프레임당 최대 데이터 품질이 중요한 고성능, 고가 시장을 공략한다.39 반면 Livox는 '비용 효율성'과 '대량 생산'을 지속적으로 강조하며, 성능이 특정 작업에 충분하고 비용이 도입의 주요 장벽인 고수요 산업 시장을 적극적으로 개척하고 있다.2 이는 LiDAR 산업이 더 이상 단일 솔루션 시장이 아니라, 응용 분야에 따라 최적화된 센서를 선택하는 성숙한 단계로 진입했음을 보여준다.


MID-70의 성공적인 도입을 위해서는 기술적 한계와 실제 통합 과정에서 발생하는 과제들을 명확히 인지해야 한다.


- **근거리 부정확성:** 앞서 언급했듯이, 센서로부터 약 25 cm 이내의 '도넛' 형태 영역에서는 정밀도가 보장되지 않는다.3 이는 정밀한 근접 제어가 필요한 작업에 제약이 될 수 있다.
- **고반사율 표면:** 반사율이 매우 높은 재질은 측정 오류를 유발할 수 있다. 한 평가에서는 반사 테이프에 조사된 포인트가 실제 위치보다 약 60 mm 멀리 측정되었다고 보고했다.5
- **빠르게 움직이는 객체 탐지:** NRS 패턴의 특성상 단일 스캔에서 작고 빠르게 움직이는 객체의 탐지를 보장하기 어렵다. 이 센서는 정적 환경이나 객체의 움직임이 센서 프레임 속도에 비해 느린 시나리오에 가장 적합하다.5
- **모션 블러 (스캔 왜곡):** 센서가 움직이는 동안 스캔이 이루어지기 때문에, 한 프레임 내의 포인트들은 서로 다른 시점에 획득되어 포인트 클라우드가 왜곡되는 현상이 발생한다. 이는 NRS LiDAR의 잘 알려진 문제로, IMU 데이터와의 결합을 통한 모션 보정 알고리즘이 필수적이다.9


- **소프트웨어 호환성:** 사용자들이 흔히 겪는 문제는 MID-70에 최신 Livox Viewer 2를 사용하려다 연결에 실패하는 경우다. 이 센서는 구버전 뷰어를 필요로 한다.29
- **네트워크 설정:** 센서와의 통신을 위해서는 컴퓨터의 IP 주소를 센서와 동일한 서브넷으로 설정하는 등 정확한 네트워크 구성이 필수적이다.5 연결 문제는 사용자 포럼에서 자주 논의되는 주제다.43
- **알고리즘 의존성:** 고유한 데이터 형식으로 인해 기존 기계식 LiDAR를 단순히 대체할 수 없으며, Livox 생태계를 채택하거나 맞춤형 알고리즘을 개발해야 하는 상당한 통합 노력이 필요하다.9

결론적으로, MID-70 도입의 주요 장벽은 하드웨어 자체의 결함보다는 알고리즘 및 시스템 통합의 복잡성에 있다. 센서의 약점은 센서 융합 및 로보틱스 인식 분야에 대한 깊은 소프트웨어 전문성을 통해 대부분 완화될 수 있다. 따라서 단순한 '플러그 앤 플레이' 장치를 기대하는 팀보다는 강력한 시스템 엔지니어링 역량을 갖춘 팀이 이 센서의 잠재력을 최대한 활용할 수 있다.



Livox MID-70은 혁신적인 기술과 공격적인 가격 정책을 결합하여 새로운 시장을 성공적으로 개척한 반도체 기반 LiDAR 센서다. 핵심 기술인 비반복 스캐닝 메커니즘은 정적 및 저속 응용 분야에서 뛰어난 공간 해상도와 거의 완벽한 FOV 커버리지를 제공하여 매핑 및 근거리 장애물 회피에 강력한 도구가 된다. 정밀도와 기하학적 정확도는 사양에 부합하는 것으로 검증되었으나, 시간에 따라 밀도를 높이는 성능 특성은 명확한 트레이드오프를 가진다. 즉, 누적된 데이터의 품질은 우수하지만 빠르게 움직이는 객체의 즉각적인 탐지에는 약점을 보인다. 신뢰성과 대량 생산성에 초점을 맞춘 설계는 대규모 상업용 로보틱스 분야에서 매력적인 부품으로 자리매김하게 한다.


- **강력 추천 분야:**
  - **저속 모바일 로봇 (AGV, AMR, 서비스 로봇):** 넓은 FOV, 최소화된 사각지대, 비용 효율성은 창고, 병원, 소매 환경 등에서 운용되는 로봇의 주력 인식 센서로서 이상적인 조합이다.
  - **정적 3D 매핑 및 측량:** 고정된 삼각대나 느리게 움직이는 플랫폼에 장착 시, 100%에 가까운 FOV 커버리지를 달성하는 능력은 기존 측량 장비 비용의 일부만으로 매우 상세한 3D 모델을 생성할 수 있게 한다.
  - **대형 차량의 사각지대 보완:** 자율주행 셔틀이나 트럭에서 장거리 주력 LiDAR가 놓치기 쉬운 근거리 사각지대를 효과적으로 커버하는 보조 센서로 활용 가치가 높다.
- **주의가 필요한 분야:**
  - **고속 자율주행 (레벨 3/4):** 센서 제품군의 일부로 활용될 수는 있으나, NRS 패턴의 특성상 고속으로 움직이는 동적 객체를 탐지하는 주력 센서로 사용하기에는 위험 부담이 크다. 이 목적에는 Horizon이나 Tele-15와 같은 다른 Livox 제품이 더 적합하다.22
  - **고정밀 로봇 조작:** 센서로부터 약 25 cm 이내의 저정밀도 구간은 매우 가까운 거리에서 정밀한 측정이 필요한 작업에는 부적합하다. 단, 시스템이 이 구간 밖에서 작동하도록 설계된 경우는 예외다.
- **핵심 구현 지침:** 성공적인 통합은 Livox 소프트웨어 생태계(SDK, ROS 드라이버)를 적극적으로 활용하고, IMU 데이터를 사용하여 모션 왜곡을 효과적으로 보정하는 데 달려 있다. 개발팀은 비반복 스캐닝 패러다임의 본질적인 과제를 완화하기 위해 소프트웨어 개발 및 시스템 수준 통합에 대한 예산을 책정해야 한다.


1. What is the difference between the repetitive scanning pattern and non-repetitive scanning pattern? - Heliguy, accessed July 23, 2025, https://www.heliguy.com/blogs/knowledge-base/what-is-the-difference-between-the-repetitive-scanning-pattern-and-non-repetitive-scanning-pattern/
2. Mid-40 lidar sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/mid-40-and-mid-100
3. Livox Mid-70 User Manual - EN - v1.2 - Scribd, accessed July 23, 2025, https://www.scribd.com/document/538276720/Livox-Mid-70-User-Manual-EN-v1-2
4. Point Cloud Characteristics | Livox, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Point%20cloud%20characteristics.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Point cloud characteristics.pdf)
5. Livox Mid-40 and Mid-70 Evaluation - AWS, accessed July 23, 2025, https://h24-files.s3.amazonaws.com/204790/1374776-iW1fn.pdf
6. Non-Repetitive Scanning LiDAR Sensor for Robust 3D Point Cloud Registration in Localization and Mapping Applications - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/24/2/378
7. Mid-70 LiDAR Sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/mid-70
8. Livox Introduces High Performance, Low Cost, Mass Market Lidar Sensors For L3/L4 Autonomous Driving Applications, accessed July 23, 2025, https://www.livoxtech.com/news/4
9. Loam livox: A fast, robust, high-precision LiDAR odometry and mapping package for LiDARs of small FoV - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/uploads/loam_livox.pdf
10. Livox Mid-70 LiDAR in Low-speed Robot Applications, accessed July 23, 2025, https://www.livoxtech.com/showcase/10
11. (PDF) 360-deg FOV Scanning LiDAR versus Non-Repetitive Scanning LiDAR: A Rover Navigation Experiment - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/372708700_360-deg_FOV_Scanning_LiDAR_versus_Non-Repetitive_Scanning_LiDAR_A_Rover_Navigation_Experiment
12. Livox Mid-360 - Sachtleben Technology, accessed July 23, 2025, https://www.sachtleben-technology.com/wp-content/uploads/sites/5/2024/07/LivoxMid-360UserManual.pdf
13. www.livoxtech.com, accessed July 23, 2025, [https://www.livoxtech.com/mid-70#:~:text=All%20of%20these%20factors%20contribute,%241099%20for%20a%20single%20unit.](https://www.livoxtech.com/mid-70#:~:text=All of these factors contribute,%241099 for a single unit.)
14. LIDAR] Livox Mid-70 3D Object Detection - J. M. Patel College, accessed July 23, 2025, https://jmpatelcollege.com/view/LIDAR-Livox-Mid-70-3D-Object-Detection/504186
15. Lidar sensors : r/UAVmapping - Reddit, accessed July 23, 2025, https://www.reddit.com/r/UAVmapping/comments/14tneya/lidar_sensors/
16. Specs - Mid-70 LiDAR Sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/mid-70/specs
17. ACCURACY ASSESSMENT AND CALIBRATION OF LOW-COST ..., accessed July 23, 2025, https://isprs-archives.copernicus.org/articles/XLIII-B1-2020/371/2020/isprs-archives-XLIII-B1-2020-371-2020.pdf
18. Buy Livox Mid-70 LiDAR - DJI Store, accessed July 23, 2025, https://store.dji.com/product/livox-mid-70-lidar
19. Buy Livox Mid-70 LiDAR at Robu.in, accessed July 23, 2025, https://robu.in/product/livox-mid-70-lidar/
20. Livox Mid Series, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20190129/Livox%20Mid%20Series%20User%20Manual%20EN%2020190129%20v1.0.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20190129/Livox Mid Series User Manual EN 20190129 v1.0.pdf)
21. A Quantitative Analysis of Point Clouds from Automotive Lidars ..., accessed July 23, 2025, https://www.mdpi.com/2073-4433/12/6/738
22. 1. Product - Livox wiki 0.1 documentation, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/introduction/production.html
23. Covering Short and Long Ranges Livox Tech Launches Two LiDAR Devices, accessed July 23, 2025, https://www.livoxtech.com/news/12
24. [SLAM, Livox MID-70 LiDAR] FAST-LIO2 vs Livox-mapping vs LOAM-Livox in real-world (test#4) - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=NmT0o268OLM
25. 1. Summary of Official Open Source Materials - Livox wiki 0.1 documentation, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/data_summary/Livox_data_summary.html
26. Livox SDK, accessed July 23, 2025, https://www.livoxtech.com/sdk
27. Livox SDK API - GitHub Pages, accessed July 23, 2025, https://livox-sdk.github.io/Livox-SDK-Doc/
28. LIVOX Mid-360 LiDAR Sensor User Guide - device.report, accessed July 23, 2025, https://device.report/manual/6363411
29. Livox Mid 70 - Faulty Unit? : r/LiDAR - Reddit, accessed July 23, 2025, https://www.reddit.com/r/LiDAR/comments/1ezdmvz/livox_mid_70_faulty_unit/
30. Livox-SDK/livox_ros_driver: Livox device driver under ros, support Lidar Mid-40, Mid-70, Tele-15, Horizon, Avia. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver
31. livox_ros_driver - ROS Wiki, accessed July 23, 2025, http://wiki.ros.org/livox_ros_driver
32. OS0 Ultra-Wide View High-Resolution Imaging Lidar - Ouster, accessed July 23, 2025, https://data.ouster.io/downloads/datasheets/datasheet-rev7-v3p0-os0.pdf
33. OS0 Ultra-Wide View High-Resolution Imaging Lidar - Ouster, accessed July 23, 2025, https://data.ouster.io/downloads/datasheets/datasheet-rev05-v2p1-os0.pdf
34. Velodyne VLP-16 - PAL SDK 23.12 documentation, accessed July 23, 2025, https://docs.pal-robotics.com/sdk/23.12/hardware/extra/velodyne.html
35. 63-9229_Rev-H_Puck _Datasheet_Web, accessed July 23, 2025, https://www.mapix.com/wp-content/uploads/2018/07/63-9229_Rev-H_Puck-_Datasheet_Web-1.pdf
36. Ouster Ultra-Wide View LiDar Sensor OS0-32 - PNW Scientific, accessed July 23, 2025, https://pnwscientific.com/products/ouster-lidar-sensor-oso-32
37. Velodyne Puck - eBay, accessed July 23, 2025, https://www.ebay.com/shop/velodyne-puck?_nkw=velodyne+puck
38. Velodyne cuts VLP-16 lidar price to $4k - Geo Week News, accessed July 23, 2025, https://www.geoweeknews.com/news/velodyne-cuts-vlp-16-lidar-price-4k
39. Ouster OS0: High-Precision Ultra-Wide Short-Range Lidar Sensor, accessed July 23, 2025, https://ouster.com/products/hardware/os0-lidar-sensor
40. Ouster Ultra-Wide FOV OS0 Lidar (Rev 7) - Génération Robots, accessed July 23, 2025, https://www.generationrobots.com/en/404054-ouster-ultra-wide-fov-os0-lidar-rev-7.html
41. Livox-SDK/LIO-Livox: A Robust LiDAR-Inertial Odometry for Livox LiDAR - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/LIO-Livox
42. Not able to capture Lidar data from Livox Mid 70 connected to Rasberry Pi 4, using Python code / Issue #147 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK/issues/147
43. MID-70 Problem!! Questions - Products - Livox Forum, accessed July 23, 2025, https://forum.livoxtech.com/thread-196-1-1.html
44. Unable to connect to ubuntu [SOLVED] - Applications - Livox Forum, accessed July 23, 2025, https://forum.livoxtech.com/thread-267-1-1.html
45. Unable to connect to Livox MID-70 using ROS launch files / Issue #156 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver/issues/156






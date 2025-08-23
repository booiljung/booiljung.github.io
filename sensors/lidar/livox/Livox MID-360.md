# Livox MID-360에 대한 기술적 고찰


DJI의 내부 인큐베이팅을 통해 탄생한 Livox는 MID360이라는 제품을 통해 저속 이동 로봇(low speed robotics) 시장에 근본적인 변화를 시도하고 있다.1 이 센서는 360도 수평 시야각(FOV)과 하이브리드 고정형(hybrid solid-state) 기술을 1,000달러 미만의 파격적인 가격대에 결합함으로써, 3D 공간 인지가 상당한 비용 장벽이었던 기존의 패러다임에 정면으로 도전한다.2 본 보고서는 이러한 저비용의 약속이 실제 현장에서 요구되는 성능 수준을 만족시키는지, 그리고 어떤 기술적 트레이드오프를 수반하는지를 심층적으로 분석하는 것을 목표로 한다.

전통적으로 로보틱스 개발자들은 어려운 선택에 직면해왔다. 저렴한 2D LiDAR를 사용하고 부족한 공간 인지 능력을 심도 카메라(depth camera)와 같은 부가적인 센서로 보완하거나, Velodyne Puck과 같은 기계식 3D LiDAR에 수천 달러를 투자해야만 했다.5 이러한 상황에서 MID360은 스스로를 '두 세계의 장점을 모두 갖춘' 해결책으로 포지셔닝한다. 즉, 여러 센서를 복잡하게 조합해야 했던 기존의 방식을 단 하나의 비용 효율적인 단일 유닛으로 대체하겠다는 야심찬 목표를 제시하는 것이다.4 이는 자율 이동 로봇(AMR), 서비스 로봇, 자율 지게차 등 가격에 민감한 시장을 정조준한 전략이다.2

이 분석은 단순한 마케팅 문구를 넘어선다. 핵심 기술의 원리를 파헤치고, 공식적으로 발표된 성능 제원을 독립적인 데이터 및 실제 사용 사례와 비교 검증하며, 소프트웨어 생태계의 성숙도와 한계를 평가하고, 시장의 주요 경쟁 제품들과의 냉정한 비교를 통해 그 위치를 명확히 할 것이다. 이 보고서의 궁극적인 목적은 기술적 배경을 가진 독자, 즉 로보틱스 엔지니어와 연구자들이 MID360의 도입 여부를 전략적으로 결정하는 데 필요한 깊이 있는 통찰력을 제공하는 것이다. MID360의 등장은 단순히 새로운 제품의 출시가 아니라, DJI와 Livox가 자사의 거대한 제조 역량과 R&D를 활용하여 복잡한 기술을 대중화하려는 전략적 움직임의 일환으로 해석될 수 있다. 저속 이동 로봇 시장은 부품 원가(BOM, Bill of Materials)가 제품의 확장성에 결정적인 영향을 미치는 분야다. Livox는 MID360의 가격을 공격적으로 책정함으로써 3, 단순한 경쟁을 넘어 3D 인지를 프리미엄 기능이 아닌 기본 사양으로 만드는 새로운 가격 대비 성능 기준을 세우려 하고 있다. 따라서 MID360의 성공은 기술적 장점뿐만 아니라, 이전에 2D 솔루션이 지배하던 시장에서 3D SLAM과 내비게이션의 채택을 얼마나 가속화할 수 있는지에 따라 평가되어야 한다.9 이는 본 보고서의 후반부에서 다룰 SLAM 적용 분석이 특히 중요한 이유다.


Livox MID360의 경쟁력과 한계는 그 핵심 기술에 깊이 뿌리박고 있다. 전통적인 기계식 LiDAR와는 근본적으로 다른 접근 방식을 채택했으며, 이는 센서의 모든 특성에 연쇄적인 영향을 미친다.


MID360은 공식적으로 "독자적인 회전 미러 하이브리드 고정형 기술(unique rotating mirror hybrid-solid technology)"을 사용한다고 명시되어 있다.2 이 방식은 Velodyne과 같은 전통적인 기계식 LiDAR가 레이저 발신기와 수신기 배열 전체를 고속으로 회전시키는 것과 대조된다. MID360의 구조에서는 레이저 발신기와 수신기가 기기에 고정되어 있으며(solid-state), 내부의 회전하는 미러 시스템이 레이저 빔의 방향을 조종하여 주변 환경을 스캔한다.10

이 설계는 몇 가지 명백한 장점을 가진다. 첫째, 복잡하고 마모에 취약한 고속 회전 부품의 수를 크게 줄여 신뢰성과 내구성을 획기적으로 향상시킨다. 회전하는 부품이 적다는 것은 물리적 충격이나 진동에 더 강하다는 것을 의미하며, 이는 산업 현장이나 실외에서 운용되는 로봇에게 매우 중요한 특성이다. 둘째, 전체 시스템을 훨씬 더 작고 가볍게 만들 수 있다. MID360이 65x65x60 mm의 크기와 265g의 무게라는 놀라운 소형화를 달성할 수 있었던 것은 바로 이 하이브리드 설계 덕분이다.2 이러한 소형화는 공간이 제한적인 소형 로봇이나 드론에 LiDAR를 통합할 때 결정적인 이점으로 작용한다.13

레이저 소스로는 Class 1 등급의 눈에 안전한(eye-safe) 905 nm 파장의 레이저를 사용한다.11 이는 단거리 및 중거리 LiDAR에서 가장 보편적으로 사용되는 비용 효율적인 선택지로, 대량 생산과 가격 경쟁력 확보에 유리하다.


MID360 기술의 가장 독특하고 핵심적인 부분은 바로 비반복 스캔 패턴이다. 이는 센서의 데이터를 이해하고 활용하는 방식 자체를 바꾸는 중요한 특징이다.

전통적인 기계식 LiDAR는 매 프레임마다 고정된 수직 각도로 레이저를 발사하여 수평으로 회전한다. 그 결과, 매번 동일한 경로를 따라 스캔하여 예측 가능하고 구조화된 형태의 포인트 클라우드를 생성한다. 반면, Livox 센서들은 "꽃잎 모양(flower-like)" 또는 "스파이로그래프(spirograph)"와 유사한 비반복적인 스캔 패턴을 채택했다.16 이는 레이저 빔이 프레임마다 다른 경로를 따라 발사되어, 시간이 지남에 따라 스캔 영역을 점차 채워나가는 방식이다.

이 방식의 가장 큰 장점은 '시간 누적에 따른 커버리지 증가'이다. 단일 프레임만 보면 포인트 클라우드가 다소 듬성듬성해 보일 수 있지만, 시간이 흐르면서 레이저 빔이 이전에 스캔하지 않았던 영역을 비추게 된다. 그 결과, 통합 시간(integration time)이 길어질수록 포인트 클라우드의 밀도와 시야각 내 커버리지가 기하급수적으로 증가한다.10 Livox의 공식 문서에 따르면, MID360은 0.1초의 통합 시간만으로도 32채널 기계식 LiDAR와 유사한 커버리지를 보이며, 0.5초가 되면 커버리지가 70%에 육박하여 64채널 기계식 LiDAR를 능가하는 성능을 보인다.10 이는 정적인 환경을 매핑하거나 저속으로 이동하면서 주변 지도를 작성하는 SLAM(Simultaneous Localization and Mapping) 애플리케이션에 매우 유리하다. 시간이 지날수록 더 상세하고 완전한 환경 정보를 얻을 수 있기 때문이다.2


이 독특한 비반복 스캔 패턴은 MID360의 가장 큰 강점이자 동시에 가장 뚜렷한 약점이기도 하다. 이 기술적 트레이드오프를 이해하는 것이 센서를 올바르게 활용하는 데 있어 핵심적이다.

장점은 앞서 언급했듯이, 저속으로 움직이거나 정지 상태에서 시간 누적을 통해 매우 상세하고 밀도 높은 3D 지도를 생성할 수 있다는 점이다. 이는 실내 서비스 로봇이나 자율 이동 로봇이 자신의 위치를 파악하고 경로를 계획하는 데 필수적인 고품질의 지도를 만드는 데 매우 효과적이다.

하지만 단점은 실시간 동적 객체 인식의 어려움이다. 가변적인 빔 패턴 때문에, 빠르게 움직이는 물체에 대해 단일 프레임 내에서 확보할 수 있는 포인트의 수가 매우 적다.24 예를 들어, 센서 앞을 빠르게 지나가는 사람이나 차량을 한 프레임의 데이터만으로는 정확하게 인식하고 추적하기 어려울 수 있다. 이 문제를 해결하기 위해서는 포인트 클라우드의 가변성을 처리할 수 있도록 특별히 설계된 인식 소프트웨어가 필요하다.24

더 나아가, 이러한 비반복 패턴은 시뮬레이션 환경에서 정확하게 구현하기가 매우 어렵다. NVIDIA의 Isaac Sim이나 Gazebo와 같은 주요 시뮬레이터들은 기본적으로 이러한 패턴을 지원하지 않으며, 이를 위해 개발자들은 커뮤니티에서 제작한 플러그인을 사용하거나 직접 개발해야 하는 부담을 안게 된다.23 이는 가상 환경에서의 알고리즘 개발 및 테스트에 크게 의존하는 팀에게는 상당한 개발 오버헤드로 작용할 수 있다.

이러한 기술적 특성을 종합해 볼 때, 비반복 스캔 패턴은 MID360의 정체성을 규정하는 핵심 요소이다. 이는 개발자에게 프레임 기반의 단편적인 인식을 넘어, 시간적으로 누적된 정보를 활용하는 인식 패러다임으로의 전환을 요구한다. 그리고 이 과정에서 내장된 IMU(관성 측정 장치)는 단순한 부가 기능이 아니라, 이 센서 데이터를 이해하기 위한 *근본적인 필수 요소*가 된다. 전통적인 LiDAR는 각 프레임이 구조적이고 예측 가능하여 프레임 간의 비교를 통해 쉽게 움직임을 계산할 수 있다. 그러나 MID360의 포인트 클라우드는 비구조적이고 프레임마다 다르기 때문에, 단순한 스캔 매칭(scan-matching) 방식의 SLAM 알고리즘(예: F-LOAM)은 제대로 작동하지 않는다.23 시스템이 자신의 움직임을 파악하기 위해서는 다른 센서의 도움이 절대적으로 필요하다. 바로 이 지점에서 내장된 ICM40609 IMU가 역할을 한다.11 이 IMU는 200 Hz의 높은 주기로 10 모션 데이터를 제공하여, LiDAR 데이터의 공백을 메워준다. 따라서 MID360을 사용한 성공적인 SLAM은 LiDAR와 IMU 데이터를 긴밀하게 결합하는 LiDAR-관성 주행 거리 측정(LIO, LiDAR-Inertial Odometry) 방식에 의존할 수밖에 없다. IMU가 고주파의 모션 추정치를 제공하고, LiDAR 데이터가 주기적으로 이 추정치의 드리프트를 보정하며 지도를 작성하는 방식이다. 이것이 바로 FAST-LIO나 LIO-SAM과 같은 알고리즘이 Livox 센서와 함께 뛰어난 성능을 보이는 이유다.27 결론적으로, 비반복 스캔 패턴이라는 설계상의 선택은 전체 시스템 아키텍처에 연쇄적인 영향을 미친다. 이는 IMU의 사용을 강제하고, 특정 LIO 기반 알고리즘의 채택을 필수적으로 만든다. 개발자는 MID360을 단순히 '더 저렴한 Velodyne'으로 취급할 수 없으며, 센서 융합에 기반한 완전히 다른 소프트웨어 및 알고리즘 철학을 받아들여야만 그 잠재력을 온전히 활용할 수 있다.


제조사가 제공하는 공식 제원(datasheet)은 센서의 잠재력을 평가하는 첫걸음이지만, 실제 애플리케이션에서의 성능을 보장하지는 않는다. 이 섹션에서는 공식 제원을 상세히 분석하고, 이를 실제 사용 환경에서의 평가 결과와 비교하여 MID360의 현실적인 성능을 가늠해본다.


MID360의 성능을 이해하기 위해 먼저 제조사가 공개한 핵심 제원들을 종합적으로 살펴볼 필요가 있다. 이 정보들은 여러 공식 문서(제품 페이지, 사양 시트, 사용자 매뉴얼)에 흩어져 있는 내용을 통합한 것으로, 기술적 평가의 기준점이 된다.2

**표 1: Livox MID360 공식 기술 제원 요약**

| 항목                 | 사양                                                | 출처 |
| -------------------- | --------------------------------------------------- | ---- |
| **모델명**           | MID-360                                             | 11   |
| **기술 방식**        | 하이브리드 고정형 (회전 미러)                       | 2    |
| **레이저 파장**      | 905 nm (Class 1 Eye-Safe)                           | 11   |
| **시야각 (FOV)**     | 수평 360° x 수직 59° (-7° ~ +52°)                   | 2    |
| **탐지 거리**        | 40 m @ 10% 반사율, 70 m @ 80% 반사율 (100 klx 환경) | 3    |
| **최소 탐지 거리**   | 0.1 m                                               | 2    |
| **포인트 출력률**    | 200,000 points/s (First Return)                     | 11   |
| **등가 채널 수**     | 40-line (등가)                                      | 2    |
| **프레임 레이트**    | 10 Hz (일반)                                        | 11   |
| **거리 정밀도 (1σ)** | ≤ 2 cm @ 10m, ≤ 3 cm @ 0.2m                         | 11   |
| **각도 정밀도 (1σ)** | < 0.15°                                             | 11   |
| **내장 IMU**         | ICM40609, 200 Hz 출력                               | 10   |
| **데이터 포트**      | 100 BASE-TX Ethernet                                | 11   |
| **시간 동기화**      | PTPv2 (IEEE 1588), GPS                              | 11   |
| **평균 소비 전력**   | 6.5 W (자체 발열 모드 시 최대 14W)                  | 11   |
| **작동 전압**        | 9V - 27V DC                                         | 11   |
| **방수/방진 등급**   | IP67                                                | 11   |
| **크기 (mm)**        | 65 x 65 x 60                                        | 2    |
| **무게**             | 265 g                                               | 2    |
| **작동 온도**        | -20°C ~ 55°C                                        | 11   |

이 제원표 외에도 실제 통합 시 반드시 고려해야 할 물리적, 환경적 제약사항들이 있다.

첫째, **열 관리(Thermal Management)** 문제다. 사용자 매뉴얼은 고온 환경에서 성능이 저하될 수 있음을 명시하며, 알루미늄 합금과 같이 열전도성이 좋은 금속판에 센서를 장착하고 원활한 공기 흐름을 위해 최소 10mm의 공간을 확보할 것을 강력히 권장한다.11 센서 외피 온도가 80°C를 초과하면 과열 보호 메커니즘이 작동하여 경고를 발생시키고, 온도가 더 높아지면 자동으로 작동을 멈춘다.11 이는 로봇 설계 시 방열 구조를 반드시 고려해야 함을 의미하는 중요한 제약 조건이다.

둘째, **자체 발열(Self-Heating)** 기능이다. -20°C에서 0°C 사이의 저온 환경에서는 센서가 자동으로 자체 발열 모드에 진입하며, 이때 최대 소비 전력이 14W까지 치솟는다.11 로봇의 전원 시스템은 이러한 순간적인 최대 부하를 감당할 수 있도록 설계되어야 하며, 이를 간과할 경우 저온 환경에서 시동 시 센서가 정상적으로 작동하지 않을 수 있다.

셋째, **커넥터 사양**이다. MID360은 신뢰성이 높은 M12 A-Code 12핀 커넥터를 사용한다.10 이는 산업용 표준 커넥터이지만, 실제 통합 시 케이블의 굵기나 커넥터의 크기가 소형 로봇의 협소한 공간에 제약을 줄 수 있으므로 기구 설계 단계에서부터 고려되어야 한다.


공식 제원이 제시하는 수치와 실제 현장에서 체감하는 성능 사이에는 종종 간극이 존재한다. MID360 역시 예외는 아니며, 특히 탐지 거리와 포인트 밀도 측면에서 이러한 차이가 두드러진다.

**유효 탐지 거리 vs. 공식 탐지 거리**: 공식 제원에는 10% 반사율을 가진 물체에 대해 40m의 탐지 거리를 명시하고 있다.11 하지만 사람 추적(people tracking)과 같은 특정 애플리케이션에 대한 독립적인 리뷰에서는 실제 *유효* 탐지 거리가 8~15m에 불과하다고 보고된다.24 이 차이는 매우 중요하다. 40m라는 수치는 이상적인 조건에서 크고 반사율이 높은 평면 타겟을 대상으로 측정한 벤치마크 값일 가능성이 높다. 실제 환경의 객체들, 예를 들어 다양한 재질의 옷을 입은 사람들은 복잡한 형상과 낮은 반사율로 인해 훨씬 약한 신호를 반사시키므로, 신뢰할 수 있는 탐지 거리가 크게 줄어든다.

**포인트 밀도의 현실**: 제조사는 MID360을 '40채널 등가' 성능을 가진다고 홍보하지만 4, 일부 리뷰어들은 이 표현이 오해의 소지가 있다고 지적한다. 이는 시간 누적에 따른 커버리지를 기반으로 한 마케팅적 표현이며, 엔지니어에게 더 실질적인 지표는 초당 200,000 포인트라는 절대적인 데이터 출력률이다.24 이 수치는 Ouster OS0와 같은 고성능 센서가 초당 520만 개 이상의 포인트를 생성하는 것에 비하면 초라하게 보일 수 있다.24 하지만 반경 10m 내외의 실내 내비게이션과 같은 목표 애플리케이션에서는 충분히 활용 가능한 수준의 밀도로 평가된다.24

**악천후 및 외부 환경에서의 성능**:

- **비, 안개, 먼지**: 사용자 매뉴얼은 안개가 끼거나 폭풍우가 치는 등 시인성이 낮은 조건에서는 탐지 거리가 감소할 수 있다고 경고하며 30, 포인트 클라우드에 노이즈가 발생할 수 있음을 인정한다.33 흥미롭게도, 일부 연구에서는 Livox의 비반복 스캔 패턴이 특정 경쟁사 제품보다 안개 속에서 더 나은 성능을 보인다고 보고하기도 했다.34 이는 패턴의 특성상 일부 레이저 빔이 장애물을 투과하거나 우회하여 목표물에 도달할 확률이 있기 때문일 수 있다.
- **직사광선**: MID360은 최대 100 klx의 강한 직사광선 환경에서도 0.01% 미만의 낮은 오경보율(False Alarm Rate)로 작동하도록 설계되었다.4 이는 많은 심도 카메라가 강한 햇빛 아래에서 성능이 급격히 저하되는 것과 비교했을 때 상당한 강점이며, 실내외를 오가는 로봇에 더 넓은 활용성을 제공한다.4
- **노이즈 필터링**: Livox는 데이터 패킷 내에 '태그(Tag)' 정보를 포함시켜, 반사된 에코의 에너지 강도를 기반으로 비, 안개, 먼지 등으로 인한 노이즈를 식별하고 필터링하는 데 도움을 준다.35 이는 매우 유용한 기능이지만, 완벽하지는 않다. 예를 들어, 물기가 있는 지면을 먼지로 오인하는 등 복잡한 환경에서는 잘못된 분류가 발생할 수 있어, 추가적인 공간적 필터링 알고리즘을 함께 사용해야 할 수도 있다.35

이러한 분석을 통해 우리는 MID360의 '스펙과 현실 사이의 간극'이 왜 발생하는지 이해할 수 있다. 이는 센서의 핵심 기술과 비용 절감을 위한 설계상의 선택에서 비롯된 필연적인 결과다. 40m @ 10% 반사율이라는 제원은 벤치마크일 뿐, 실제 객체는 이와 다르다. '40채널 등가'라는 표현은 시간 누적 커버리지를 의미하며, 실시간 장애물 회피를 고민하는 엔지니어에게 중요한 것은 순간적인 데이터 속도인 초당 20만 포인트다. 이 낮은 포인트 속도는 더 강력한 레이저와 복잡한 검출기, 처리 파이프라인을 요구하는 고비용 구조를 피하기 위한 선택이다. 비반복 스캔 패턴은 이러한 낮은 포인트 속도를 시간적으로 보상하기 위한 영리한 해결책인 셈이다. 따라서 엔지니어는 MID360을 단순히 제원표의 헤드라인 수치만으로 평가해서는 안 된다. 특정 애플리케이션 환경에서 직접 성능 검증(Proof-of-Concept)을 수행하여 *실제 유효 성능*을 파악하는 과정이 반드시 필요하다. 제원표에만 의존한 시스템 설계는, 특히 장거리 탐지나 고충실도의 동적 객체 추적이 필요한 경우, 성능 미달로 이어질 가능성이 높다.


아무리 뛰어난 하드웨어라도 강력하고 안정적인 소프트웨어 지원 없이는 그 잠재력을 발휘할 수 없다. MID360은 Livox-SDK2와 ROS/ROS2 드라이버를 중심으로 개발 환경을 제공하지만, 그 성숙도와 사용 편의성에는 몇 가지 주목할 점이 있다.


Livox는 MID360을 포함한 자사의 모든 LiDAR 제품을 위해 Livox-SDK2라는 소프트웨어 개발 키트를 제공한다.36 C/C++를 기반으로 개발되었으며, Ubuntu, Windows, ARM 등 다양한 아키텍처를 지원하여 폭넓은 개발 환경에 대응한다.37 이 SDK는 GitHub를 통해 공개되어 있으며, 지속적인 업데이트와 릴리스가 이루어지고 있어 개발자 커뮤니티의 피드백을 반영하려는 노력을 엿볼 수 있다.37

센서와의 통신은 100 BASE-TX 이더넷 연결을 통한 UDP 프로토콜을 기반으로 한다.11 이 프로토콜은 크게 두 가지 종류의 데이터를 정의한다: 센서의 파라미터를 설정하고 상태를 조회하는 '제어 명령 데이터'와, 실제 측정값인 '포인트 클라우드 데이터'다.40 UDP 데이터 패킷은 헤더와 데이터 세그먼트로 구성되며, 버전 정보, 타임스탬프, 데이터 타입(직교좌표계/구면좌표계), 그리고 실제 포인트 클라우드나 IMU 데이터 등의 핵심 정보를 포함한다.29 시간 동기화는 PTPv2(IEEE 1588)나 GPS를 통해 정밀하게 이루어질 수 있으며, 외부 동기화 소스가 없을 경우 센서의 전원이 켜진 시점을 기준으로 한 내부 시간을 사용한다.29


로보틱스 개발에서 사실상의 표준 프레임워크인 ROS(Robot Operating System)와의 통합은 센서의 활용성을 결정하는 매우 중요한 요소다. Livox는 `livox_ros_driver2`라는 공식 드라이버를 통해 ROS1(Melodic, Noetic 권장)과 ROS2(Foxy, Humble 권장)를 모두 지원한다.43

그러나 이 드라이버의 설치 및 사용 과정은 다소 까다로운 편이다. 개발자는 반드시 패키지 내에 포함된 `build.sh` 스크립트를 사용하여 컴파일해야 한다. 일반적인 ROS 패키지처럼 `catkin_make`나 `colcon build` 명령어를 직접 사용하면 빌드에 실패할 수 있다.45 또한, 빌드 전에 시스템에 설치된 특정 정적 라이브러리(`livox_sdk_static.a`)를 삭제해야 하는 등 46, 초보 개발자에게는 다소 혼란스러울 수 있는 절차들이 존재한다.

설정 과정 역시 수동적인 작업이 필요하다. 센서는 출고 시 `192.168.1.1XX` (여기서 XX는 시리얼 번호의 마지막 두 자리) 형태의 고정 IP 주소를 가지도록 설정되어 있다.30 따라서 개발자는 호스트 PC의 IP 주소 또한 동일한 서브넷의 고정 IP로 설정해야 하며, 이 정보를 `MID360_config.json`과 같은 설정 파일에 명시적으로 기입해야 한다.44

드라이버가 성공적으로 실행되면, 센서 데이터는 표준 ROS 토픽으로 발행된다. MID360의 경우, 포인트 클라우드 데이터는 `/livox/lidar` 토픽으로, IMU 데이터는 `/livox/imu` 토픽으로 발행되어 47, 다른 ROS 노드에서 이 데이터들을 구독하여 사용할 수 있게 된다. 포인트 클라우드 데이터는 Livox의 커스텀 메시지 포맷이나 표준 `sensor_msgs/PointCloud2` 포맷으로 발행되도록 선택할 수 있다.43


Livox는 자체적인 데이터 생태계를 구축하고 있으며, 이는 데이터 저장 및 변환 과정에서도 나타난다. Livox가 제공하는 `Livox Viewer` 소프트웨어를 사용하면 센서 데이터를 `.lvx`라는 독자적인 포맷으로 저장할 수 있다.48 이 포맷은 여러 디바이스의 데이터를 하나의 파일에 저장할 수 있으며, 헤더 정보, 장치 정보, 프레임별 포인트 클라우드 데이터를 체계적으로 담고 있다.

물론, 로보틱스 커뮤니티에서 널리 사용되는 표준 포맷으로의 변환도 지원된다. `livox_ros_driver`는 `.lvx` 파일을 ROS 생태계에서 사용하기 용이한 `.bag` 파일로 변환하는 기능을 제공한다.50 또한 `Livox Viewer` 자체적으로 `.lvx` 파일을 `.las`나 `.csv` 포맷으로 변환하는 툴을 내장하고 있다.50

`PointCloud2` 메시지를 포함하는 `.bag` 파일은 `pcl_ros`와 같은 표준 ROS 툴을 사용하여 프레임별 `.pcd`(Point Cloud Data) 파일로 쉽게 변환할 수 있어, PCL(Point Cloud Library)을 사용한 후처리 작업에도 용이하다.50

이러한 소프트웨어 생태계를 종합적으로 평가할 때, 기능적으로는 필요한 요소들을 갖추고 있지만 아직 성숙도가 다소 부족하다는 인상을 준다. 이는 개발자에게 더 높은 수준의 환경 설정 및 구성 부담을 지운다. 예를 들어, 특정 빌드 스크립트 사용 강제, 정적 라이브러리 수동 삭제, 호스트와 설정 파일 양쪽에서의 고정 IP 수동 설정 등은 '플러그 앤 플레이' 경험과는 거리가 멀다.43 이러한 엄격한 규칙들은 빌드 시스템의 유연성이 부족하거나 내부적인 충돌 가능성이 있음을 시사한다. 또한, GitHub의 수많은 이슈 스레드와 커뮤니티 포크들은 공식 지원이 모든 사용 사례를 포괄하지 못하며, 개발자들이 스스로 문제를 해결하고 해결책을 공유하며 생태계를 보완해나가고 있음을 보여준다.25

결론적으로, MID360을 도입하는 개발팀은 초기 설정, 환경 구성, 그리고 잠재적인 문제 해결을 위한 추가적인 시간을 예산에 반드시 포함해야 한다. 공식적으로 문서화된 Ubuntu/ROS 버전을 벗어나는 경우, 이러한 부담은 더욱 커질 수 있다. 하드웨어의 낮은 가격은, 엔지니어링 시간으로 환산되는 초기 소프트웨어 통합 비용의 증가로 일부 상쇄된다고 볼 수 있다.


MID360의 진정한 가치는 기술적 제원 자체보다는 시장 내에서의 독보적인 포지셔닝에서 나온다. 가격, 성능, 기술적 특성을 종합적으로 고려할 때, MID360은 기존의 경쟁 구도를 재편하고 새로운 시장을 창출하는 잠재력을 지니고 있다.


MID360의 가장 강력한 무기는 단연 가격이다. 공식 스토어 가격은 979달러이며, 여러 온라인 리테일러를 통해 약 750달러에서 1,245달러 사이에서 구매할 수 있다.3 이 가격대는 360도 3D 인지를 제공하는 LiDAR 센서로서는 파격적이며, 시장에 상당한 충격을 주었다.

이 가격을 바탕으로 한 MID360의 가치 제안은 명확하다: 기존의 고성능 LiDAR 대비 극히 일부의 비용으로 완전한 360도 3D 인지, 초소형/초경량 폼팩터, 그리고 내장 IMU까지 제공하는 것이다.12 이는 비용에 극도로 민감한 저속 이동 로봇, 서비스 로봇, 교육용 플랫폼 시장에서 '게임 체인저' 역할을 할 수 있음을 의미한다. 이전에는 비용 문제로 2D LiDAR에 머물러야 했던 많은 애플리케이션들이 3D 공간 인지의 문턱을 넘을 수 있게 된 것이다.


MID360의 시장 내 위치를 더 명확히 이해하기 위해, 주요 경쟁 제품인 Ouster OS0와 Velodyne Puck(VLP-16)과의 직접적인 비교는 필수적이다. 이 비교는 개발자가 자신의 애플리케이션에 가장 적합한 센서를 선택하는 데 있어 어떤 트레이드오프를 고려해야 하는지를 명확하게 보여준다.

**표 2: 주요 경쟁 제품 사양 비교 - Livox MID360 vs. Ouster OS0 vs. Velodyne Puck (VLP-16)**

| 기능                       | **Livox MID360**    | **Ouster OS0-128 (Rev7)** | **Velodyne Puck (VLP-16)**      |
| -------------------------- | ------------------- | ------------------------- | ------------------------------- |
| **기술 방식**              | 하이브리드 고정형   | 회전식 디지털 LiDAR       | 회전식 기계식 LiDAR             |
| **수평 FOV**               | 360° 2              | 360° 55                   | 360° 7                          |
| **수직 FOV**               | 59° (-7° ~ +52°) 11 | **90° (±45°)** 55         | 30° (±15°) 7                    |
| **포인트 출력률 (최대)**   | 200,000 pts/s 11    | **5,242,880 pts/s** 55    | ~600,000 pts/s (Dual Return) 7  |
| **탐지 거리 (10% 반사율)** | 40 m 11             | 35 m 55                   | ~40 m (유효, 추정)¹             |
| **탐지 거리 (80% 반사율)** | 70 m 11             | 75 m 55                   | 100 m 7                         |
| **최소 탐지 거리**         | **0.1 m** 11        | 0.5 m 55                  | ~0.5 m (유효)                   |
| **채널/라인 수**           | 40 (등가) 4         | **128** 55                | 16 7                            |
| **크기 (mm)**              | **65 x 65 x 60** 11 | Ø87 x 74.2 55             | Ø103 x 72 7                     |
| **무게 (g)**               | **265 g** 11        | 500 g (캡 포함) 55        | ~830 g 7                        |
| **평균 소비 전력 (W)**     | **6.5 W** 11        | 14-20 W 56                | 8 W 7                           |
| **가격 (근사치)**          | **~$980** 3         | ~$4,500 - $17,000+²       | ~$4,000 (과거), 현재 ~$400-$3k³ |

- **주석**:
  1. Velodyne VLP-16의 공식 탐지 거리는 100m이지만, 이는 주로 높은 반사율 기준이다. 낮은 반사율 대상에 대한 성능은 동급 센서들과 유사하거나 약간 나은 수준으로 알려져 있다.
  2. Ouster OS0의 가격은 채널 수(32/64/128)와 판매처에 따라 매우 큰 편차를 보인다. 중고 128채널 제품이 약 4,500달러에 거래되기도 하며 57, 신제품은 17,000유로를 초과할 수 있다.58
  3. Velodyne VLP-16의 가격은 변동이 심하다. 과거 8,000달러에 판매되다가 2018년에 4,000달러로 인하되었다.59 현재 시장에서는 부품용 중고 제품이 수백 달러, 신품 키트는 수천 달러에 이르는 등 넓은 가격대를 형성하고 있다.60

vs. Ouster OS0: 비용과 성능의 대결

이 비교는 본질적으로 비용 대 성능의 트레이드오프를 보여준다. Ouster OS0는 MID360 대비 26배 이상의 압도적인 포인트 출력률, 더 넓은 수직 시야각, 그리고 더 성숙한 디지털 LiDAR 아키텍처를 자랑한다.55 하지만 이는 훨씬 높은 가격, 더 큰 무게, 그리고 상당한 전력 소비를 대가로 한다. 결론적으로, 두 센서는 서로 다른 시장을 공략한다. MID360은 '충분히 좋은(good enough)' 3D 인지 능력이 필요하고 비용이 가장 중요한 제약 조건인 애플리케이션을 위한 것이다. 반면, OS0는 비용이 부차적인 문제이고 고충실도의 정밀한 인지가 최우선 과제인 애플리케이션(예: 고속 자율주행, 정밀 매핑)을 위한 선택지다.56

vs. Velodyne Puck (VLP-16): 신기술과 구기술의 대결

이 비교는 유사한 (과거의) 가격대에서 신기술과 구기술이 어떻게 경쟁하는지를 보여준다. VLP-16은 오랫동안 360도 LiDAR 시장의 산업 표준이었다.7 하지만 MID360은 VLP-16에 비해 크기가 훨씬 작고, 무게는 약 1/3 수준이며, 소비 전력도 더 낮고, 수직 시야각은 거의 두 배(59° vs 30°)에 달한다. VLP-16이 포인트 출력률(~600k vs 200k)에서 앞서지만, MID360의 비반복 스캔은 시간 누적을 통해 더 나은 커버리지를 달성할 수 있다. 즉, MID360은 가격 대비 크기, 무게, 수직 커버리지 측면에서 세대교체를 이룬 제품이라고 할 수 있다.

이러한 비교 분석을 통해 우리는 MID360의 시장 전략을 명확히 이해할 수 있다. MID360은 Ouster OS0와 같은 고성능 센서와 직접적으로 경쟁하는 것이 아니다. 대신, Velodyne VLP-16이 한때 지배했던 시장, 특히 새롭게 부상하는 로보틱스 분야에서 '카테고리 킬러(category killer)' 역할을 하고자 한다. 사양 비교표는 MID360과 OS0가 서로 다른 리그에 속해 있음을 명백히 보여준다. 포인트 밀도가 최우선인 엔지니어는 MID360을 선택하지 않을 것이다. OS0는 고속 자율주행이나 고정밀 매핑을 위한 것이고, MID360은 저속 내비게이션과 장애물 회피를 위한 것이다.2

이제 2023년에 새로운 AMR을 설계하는 로봇 스타트업의 입장에서 VLP-16과 MID360을 비교해보자. MID360은 무게가 1/3에 불과하고, 돌출된 장애물이나 연석을 감지하는 데 중요한 수직 시야각은 거의 두 배이며, 더 작고, VLP-16의 과거 가격의 일부만으로 신제품을 구매할 수 있다. VLP-16의 유일한 장점인 높은 포인트 출력률마저도, 많은 저속 로봇에게는 MID360의 20만 pps와 우수한 수직 FOV, 누적 커버리지의 조합으로도 *충분하다*. SWaP-C(Size, Weight, and Power - Cost)에서의 압도적인 이점은 새로운 설계에 있어 MID360을 훨씬 더 합리적인 선택으로 만든다. 따라서 MID360의 시장 전략은 고성능 세그먼트는 Ouster와 같은 플레이어에게 내어주고, 대신 자신의 SWaP-C 이점을 활용하여 VLP-16과 같은 이전 세대의 기계식 스피너를 시장에서 도태시키는 것이다. 이는 저렴한 3D 인식의 기준을 재정의하는 움직임이다.


MID360의 매력적인 가격과 제원은 실제 로봇 시스템에 통합하는 과정에서 마주하게 될 여러 기술적 과제들을 가릴 수 있다. 성공적인 도입을 위해서는 알고리즘 선택부터 물리적 설치, 시뮬레이션 환경 구축에 이르기까지 다방면에 걸친 신중한 고려가 필요하다.


MID360 통합에서 가장 먼저 부딪히는 큰 장벽은 SLAM 알고리즘의 선택이다.

**핵심 문제**: 앞서 분석했듯이, MID360의 비반복 스캔 패턴은 일관되고 구조적인 스캔 라인을 가정하는 다수의 전통적인 스캔 매칭 기반 SLAM 알고리즘과 호환되지 않는다. LOAM, F-LOAM, LeGo-LOAM과 같은 유명한 알고리즘들은 MID360의 비구조적인 포인트 클라우드를 처리하지 못해 성능이 저하되거나 아예 작동하지 않을 수 있다.23

**해결책 - LiDAR-관성 주행 거리 측정(LIO)**: 이 문제를 해결하기 위한 표준적인 접근법은 이러한 비구조적 포인트 클라우드에 특화된 최신 LIO 알고리즘을 사용하는 것이다. 이 알고리즘들은 LiDAR 데이터와 IMU 데이터를 긴밀하게 결합(tightly-coupled)하여 강인하고 높은 주기의 상태 추정을 수행한다.

- **FAST-LIO / FAST-LIO2**: Livox 센서와 뛰어난 궁합을 보여주기 위해 특별히 개발된 알고리즘 군이다. IMU의 고주파 출력을 기반으로 모션을 예측하고, LiDAR 포인트 클라우드를 사용해 이를 보정함으로써 정확하고 부드러운 궤적을 생성한다.27
- **LIO-SAM**: 또 다른 널리 사용되는 강력한 LIO 프레임워크로, MID360에 맞게 설정을 조정하여 성공적으로 적용할 수 있다.
- **커뮤니티의 검증**: 여러 포럼과 GitHub 이슈 트래커에서 개발자들은 이러한 LIO 알고리즘들이 MID360과 잘 작동함을 확인해주고 있으며, 반대로 다른 알고리즘을 사용하려다 어려움을 겪는 사례들도 공유되고 있다.28

**2D SLAM으로의 변환**: 만약 최종 목표가 2D 점유 격자 지도(occupancy map)를 생성하는 것이라면, 3D 포인트 클라우드를 2D 레이저 스캔 데이터로 변환하는 과정이 필요하다. ROS의 `pointcloud_to_laserscan` 패키지가 이 역할을 수행할 수 있지만, 이 과정은 생각보다 간단하지 않다. 올바른 TF(Transform) 트리 구조를 설정해야 하고, 변환할 포인트 클라우드의 높이 범위를 신중하게 지정해야 하는 등, 초보자들이 흔히 실패하는 지점 중 하나다.27


소프트웨어적인 문제 외에도, 하드웨어를 물리적으로 시스템에 통합하는 과정에서도 몇 가지 중요한 과제들이 존재한다.

**열 관리**: 이는 선택이 아닌 필수 사항이다. 센서는 작동 시 발생하는 열을 효과적으로 방출하기 위해 반드시 방열판 역할을 할 수 있는 금속 표면에 장착되어야 한다.11 유사한 센서에 대한 Reddit 사용자 리뷰에 따르면, 센서가 최대 80°C까지 뜨거워질 수 있으며, 금속 마운트가 온도를 약 15°C 낮추는 데 도움이 되었다고 한다.67 열 관리에 실패하면 성능 저하는 물론, 센서에 영구적인 손상을 초래할 수 있다.11

**전원 공급**: 전원 시스템은 평균 소비 전력인 6.5W뿐만 아니라, 저온 환경에서 자체 발열 모드가 작동할 때의 **최대 14W** 부하까지 안정적으로 공급할 수 있도록 설계되어야 한다.11 전원 공급 용량이 부족하면 추운 날씨에 로봇을 시동할 때 센서가 작동하지 않는 심각한 문제로 이어질 수 있다.

**네트워크 설정**: 센서는 호스트 컴퓨터에 고정 IP 주소 설정을 요구한다.30 이는 동적인 네트워크 환경에서는 번거로울 수 있으며, 특히 여러 대의 센서를 동시에 사용할 경우 신중한 시스템 관리가 필요하다.

**기구적 통합**: 센서 자체는 작지만, M12 커넥터와 함께 제공되는 1-to-3 스플리터 케이블은 상대적으로 부피가 크고 뻣뻣하여 기구 설계 시 반드시 고려해야 할 요소다.67 케이블의 두께와 최소 곡률 반경은 공간이 협소한 소형 로봇 내부에서 배선 문제를 야기할 수 있다.


알고리즘 개발 및 테스트를 위해 시뮬레이션에 크게 의존하는 팀에게 MID360은 또 다른 과제를 안겨준다.

**문제점**: 비반복적이고 시간 누적적인 스캔 패턴을 가상 환경에서 정확하고 효율적으로 시뮬레이션하는 것은 계산적으로 매우 복잡하며, Isaac Sim이나 Gazebo와 같은 주요 시뮬레이터에서 기본적으로 지원되지 않는다.25

**공식 플러그인 vs. 커뮤니티 솔루션**: Livox는 자체적으로 레이저 시뮬레이션 플러그인을 공개했지만, 일부 연구자들은 이 플러그인이 실제 센서에서는 나타나지 않는 왜곡을 발생시켜 SLAM 성능을 심각하게 저하시킨다고 보고했다.23 이러한 문제로 인해, 커뮤니티에서는 더 현실적인 시뮬레이션을 위해 자체적으로 Gazebo용 커스텀 플러그인을 개발하여 공유하고 있다.23

**시사점**: 이는 시뮬레이션 기반 개발팀에게 상당한 장벽이 될 수 있음을 의미한다. 이들은 커스텀 시뮬레이션 플러그인을 찾고, 검증하고, 혹은 직접 개발하는 데 상당한 시간을 투자해야 하거나, 센서의 독특한 특성을 완전히 반영하지 못하는 낮은 충실도의 시뮬레이션에 만족해야 하는 상황에 놓일 수 있다.

이러한 통합 과제들을 종합해 보면, MID360의 낮은 '표면 가격'은 더 높은 '총 통합 비용'이라는 이면을 감추고 있음을 알 수 있다. 특히 Livox 생태계나 LIO 기반 SLAM에 익숙하지 않은 팀에게는 더욱 그렇다. 하드웨어 비용은 약 980달러로 저렴하지만, 새로운 소프트웨어 스택을 학습하고 통합하는 데 드는 엔지니어링 시간, 적절한 방열 및 전원 솔루션을 설계하는 데 드는 노력, 그리고 시뮬레이션 환경을 구축하는 데 필요한 잠재적 비용까지 고려하면 '총 소유 비용'은 결코 낮지 않다. 이는 프로젝트 관리자가 센서 도입을 결정할 때 반드시 고려해야 할 중요한 현실이다. 저렴하게 구매할 수는 있지만, 성공적으로 통합하기 위해서는 상당한 전문성과 노력이 필요하다.


지금까지 Livox MID360의 핵심 기술, 성능, 소프트웨어 생태계, 시장 경쟁력, 그리고 통합 과제에 대해 다각도로 심층 분석했다. 이를 바탕으로 MID360의 강점과 약점을 명확히 요약하고, 최적의 적용 분야를 제시하며, 도입을 고려하는 엔지니어들을 위한 최종적인 전략적 제언을 하고자 한다.


**강점 (Strengths):**

- **압도적인 가격 경쟁력**: 360도 3D 인지를 제공하는 LiDAR로서는 전례 없는 가격대를 형성하며, 시장의 진입 장벽을 극적으로 낮추었다.3
- **뛰어난 SWaP-C (Size, Weight, Power, and Cost)**: 극도로 작고 가벼우며 전력 효율이 높아 소형 이동 로봇이나 드론에 통합하기에 이상적이다.2
- **넓은 수직 시야각 (Vertical FOV)**: 59도의 넓은 수직 시야각은 Velodyne VLP-16의 30도와 같은 동급 가격대의 경쟁 제품들보다 우수하여, 로봇 주변의 상하부 장애물에 대한 인지 능력을 크게 향상시킨다.2
- **우수한 내환경성**: IP67 등급의 방수/방진 성능과 강한 직사광선 아래에서도 안정적으로 작동하는 능력은 실내외를 아우르는 다양한 환경에서의 활용성을 보장한다.4

**약점 (Weaknesses):**

- **낮은 순간 포인트 밀도**: 초당 20만 포인트라는 출력률은 작거나 빠르게 움직이는 객체를 실시간으로 정밀하게 탐지하는 데 한계가 있다.24
- **복잡한 스캔 패턴**: 비반복 스캔 패턴은 개발에 있어 높은 진입 장벽으로 작용한다. 특화된 SLAM 알고리즘을 요구하며, 정확한 시뮬레이션 환경을 구축하기 어렵게 만든다.23
- **높은 통합 오버헤드**: 신중한 열 관리, 전원 설계, 네트워크 설정이 필수적이며, 소프트웨어 설정 과정이 성숙한 시스템들에 비해 '플러그 앤 플레이'와는 거리가 멀다.11
- **제원과 실제 성능의 간극**: 특히 탐지 거리와 같은 주요 제원은 마케팅 수치와 실제 애플리케이션에서의 유효 성능 간에 상당한 차이가 있을 수 있어, 반드시 자체적인 성능 검증이 필요하다.24


이러한 강점과 약점을 고려할 때, MID360이 가장 큰 가치를 발휘할 수 있는 분야는 다음과 같다.

- **저속 실내 AMR / 서비스 로봇**: 이 센서의 핵심 타겟 시장이다. 창고, 슈퍼마켓, 병원 등 속도가 낮고 비용이 중요한 환경에서의 내비게이션 및 장애물 회피에 완벽하게 부합한다.1
- **저비용 휴대용 3D 스캐너/매퍼**: 가벼운 무게, 작은 크기, 그리고 우수한 누적 커버리지 성능은 준공 도면 작성이나 부피 측정 등을 위한 저가형 시스템을 구축하는 데 이상적이다.22
- **학술 및 연구 플랫폼**: 저렴한 가격 덕분에 대학이나 연구소에서 다수의 로봇에 3D 인지 능력을 탑재할 수 있게 하여, 다중 에이전트 시스템이나 군집 로봇 연구를 가속화할 수 있다.12
- **단거리 사람 추적**: 반경 10m 내의 실내 공간에서 사람을 측정하는 용도로 적합하다. 예를 들어, 리테일 분석 분야에서 스테레오스코픽 카메라의 비용 효율적인 대안을 제공할 수 있다.24


MID360의 도입을 검토하는 엔지니어 또는 프로젝트 관리자는 다음의 네 가지 전략적 조언을 반드시 숙고해야 한다.

**1. 기술 스택을 먼저 평가하라 (Evaluate Your Tech Stack First)**: 단지 저렴하다는 이유만으로 MID360을 구매해서는 안 된다. 가장 먼저, 개발팀이 LIO 기반 SLAM 프레임워크(FAST-LIO 등)에 대한 경험이 있거나 이를 도입할 준비가 되어 있는지 평가해야 한다. 만약 기존의 모든 파이프라인이 전통적인 스캔 매칭 방식에 기반하고 있다면, 새로운 패러다임으로의 전환 비용은 예상보다 훨씬 클 수 있다.

**2. 데이터시트를 믿지 말고, 직접 테스트하라 (Don't Trust the Datasheet, Test It Yourself)**: 단일 유닛을 구매하여 목표 애플리케이션 환경에서 철저한 성능 검증(Proof-of-Concept)을 수행해야 한다. 특정 타겟에 대한 *유효* 탐지 거리와 밀도를 직접 측정하라. 시스템 설계를 마케팅 제원표에만 의존하는 우를 범해서는 안 된다.

**3. 통합 비용을 예산에 포함시켜라 (Budget for the Total Cost of Integration)**: 하드웨어 가격 외에, 새로운 소프트웨어 스택 학습, 적절한 방열 및 전원 솔루션 설계, 그리고 잠재적인 커스텀 시뮬레이션 툴 개발에 필요한 엔지니어링 시간을 예산에 명시적으로 반영해야 한다. 하드웨어는 저렴하지만, 엔지니어의 시간은 그렇지 않다.

**4. 강점을 활용하고 약점을 회피하라 (Leverage its Strengths, Avoid its Weaknesses)**: MID360의 SWaP-C와 넓은 수직 FOV가 명백한 이점을 제공하는 분야에 집중적으로 사용하라. 고속 동적 객체 인식이나 장거리 탐지가 요구되는 애플리케이션에 억지로 적용하려 하지 마라. 센서의 한계를 명확히 이해하고, 그 한계 내에서 시스템을 설계하는 것이 성공의 열쇠다. MID360은 강력한 도구이지만, 올바른 작업에 사용될 때만 그 진가를 발휘한다.


1. Mid-360 LiDAR: "All-Seeing Eyes" for Robot Dog - Livox, accessed July 23, 2025, https://www.livoxtech.com/showcase/13
2. Livox Mid-360, accessed July 23, 2025, https://livoxtech.com/mid-360
3. Buy Livox Mid-360 - DJI Store, accessed July 23, 2025, https://store.dji.com/product/livox-mid-360
4. Livox Revolutionizes Smart Robotics with a Compact, Omnidirectional LiDAR Sensor Mid-360, accessed July 23, 2025, https://www.livoxtech.com/news/mid360_launch
5. Livox Mid 360 Livox Lidar 3d Lidar - Speed Sensor - AliExpress, accessed July 23, 2025, https://www.aliexpress.com/i/1005005694581580.html
6. LiDAR News: Velodyne, Aeye, Livox, Robosense, Hesai - Image Sensors World, accessed July 23, 2025, http://image-sensors-world.blogspot.com/2019/08/lidar-news-velodyne-aeye-livox-velodyne.html
7. 63-9229_Rev-J_Puck _Datasheet_Web - NET, accessed July 23, 2025, https://hexagondownloads.blob.core.windows.net/public/AutonomouStuff/wp-content/uploads/2019/05/Puck__Datasheet_whitelabel.pdf
8. Livox Technology | NEXTY Electronics, a trading company for electronic components and semiconductors., accessed July 23, 2025, https://www.nexty-ele.com/en/product/livox/
9. Precise navigation: Mid-360 empowers Wimsha cleaning robots to be smarter - Livox, accessed July 23, 2025, https://www.livoxtech.com/showcase/19
10. Livox Mid-360 - Sachtleben Technology, accessed July 23, 2025, https://www.sachtleben-technology.com/wp-content/uploads/sites/5/2024/07/LivoxMid-360UserManual.pdf
11. Specs - Mid-360 LiDAR Sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/mid-360/specs
12. Unitree LIVOX MID360 LiDAR - STEMfinity, accessed July 23, 2025, https://stemfinity.com/products/livox-mid360-lidar
13. High-Speed Micro Aerial Vehicle Uses Lidar for Obstacle Avoidance | In the Scan, accessed July 23, 2025, https://blog.lidarnews.com/super-mav-livox-lidar-obstacle-avoidance/
14. LIDAR Livox MID-360 incl. Cable - owlRobotics-Shop, accessed July 23, 2025, https://owlrobotics-store.company.site/products/LIDAR-Livox-MID-360-incl-Cable-p605042897
15. Buy Livox Mid-360 (Line-16) at Robu.in, accessed July 23, 2025, https://robu.in/product/livox-mid-360-line-16-v1-2-laser-detection-rangefinder/
16. What is the difference between the repetitive scanning pattern and non-repetitive scanning pattern? - Heliguy, accessed July 23, 2025, https://www.heliguy.com/blogs/knowledge-base/what-is-the-difference-between-the-repetitive-scanning-pattern-and-non-repetitive-scanning-pattern/
17. Point Cloud Characteristics | Livox, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Point%20cloud%20characteristics.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Point cloud characteristics.pdf)
18. Understanding How Lidar Works - Insights | Outsight, accessed July 23, 2025, https://insights.outsight.ai/how-does-lidar-work/
19. Repetitive vs non-repetitive setting - ROCK R360 R2A - ROCK robotic, accessed July 23, 2025, https://community.rockrobotic.com/t/repetitive-vs-non-repetitive-setting/606
20. Discover LiDAR scan patterns with prisms & mirrors - YellowScan, accessed July 23, 2025, https://www.yellowscan.com/knowledge/what-are-the-different-scan-patterns-of-lidar-systems/
21. LIVOX AVIA - OxTS, accessed July 23, 2025, https://www.oxts.com/wp-content/uploads/2021/08/Livox-AVIA-User-Manual_EN_compressed.pdf
22. Mid-360 Assists Handheld Scanners in High-Precision Measurement - Livox, accessed July 23, 2025, https://www.livoxtech.com/showcase/18
23. SIMULATION OF LOW-COST MEMS-LIDAR AND ANALYSIS OF ITS EFFECT ON THE PERFORMANCES OF STATE-OF-THE-ART SLAMS - ISPRS - The International Archives of the Photogrammetry, Remote Sensing and Spatial Information Sciences, accessed July 23, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-1-W1-2023/539/2023/isprs-archives-XLVIII-1-W1-2023-539-2023.pdf
24. LiDAR Sensors for People Measurement Review Series: Livox Mid-360 - Digital Mortar, accessed July 23, 2025, https://digitalmortar.com/lidar-sensors-for-people-measurement-review-series-livox-mid-360/
25. Livox MID360 - Isaac Sim - NVIDIA Developer Forums, accessed July 23, 2025, https://forums.developer.nvidia.com/t/livox-mid360/283074
26. Simulation of a Livox Mid360 LiDAR sensor - CoppeliaSim forums, accessed July 23, 2025, https://forum.coppeliarobotics.com/viewtopic.php?t=10701
27. Slam_toolbox: map is not creating - External Requests - The Construct ROS Community, accessed July 23, 2025, https://get-help.theconstruct.ai/t/slam-toolbox-map-is-not-creating/33780
28. Using Livox Mid-360 with Slam Toolbox for lifelong mapping and localization #129 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver2/issues/129
29. 1.2.1.1. Livox LiDAR Communication Protocol–Mid360, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/tutorials/new_product/mid360/livox_eth_protocol_mid360.html
30. Livox Mid-360 - DJI, accessed July 23, 2025, https://dl.djicdn.com/downloads/Livox/Mid-360/QSG/Livox_Mid-360_Quick_Start_Guide_multi.pdf
31. LIVOX Mid-360 LiDAR Sensor User Guide - Manuals.plus, accessed July 23, 2025, https://manuals.plus/livox/mid-360-lidar-sensor-manual
32. LIVOX Mid-360 Lidar 3D LiDAR Minimal Detection Range User Guide - Manuals.plus, accessed July 23, 2025, https://manuals.plus/livox/mid-360-lidar-3d-lidar-minimal-detection-range-manual
33. Mid-360 FAQ - Livox, accessed July 23, 2025, https://www.livoxtech.com/support/mid-360
34. Captures of pointclouds from Livox lidar. - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/Captures-of-pointclouds-from-Livox-lidar_fig5_352237326
35. How to use Tag Information in Livox LiDAR Point Cloud, accessed July 23, 2025, https://www.livoxtech.com/showcase/livox-tag
36. CN-XSH/Livox-SDK2 - Gitee, accessed July 23, 2025, https://gitee.com/cn-xsh/Livox-SDK2
37. Livox-SDK/Livox-SDK2: Drivers for receiving LiDAR data and controlling lidar, support Lidar HAP and Mid-360. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK2
38. Releases / Livox-SDK/Livox-SDK2 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK2/releases
39. Activity / Livox-SDK/Livox-SDK2 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK2/activity
40. Livox-SDK API Reference - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK-Doc
41. Livox SDK Communication Protocol - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK/wiki/Livox-SDK-Communication-Protocol
42. Livox SDK API: LivoxEthPacket Struct Reference - GitHub Pages, accessed July 23, 2025, https://livox-sdk.github.io/Livox-SDK-Doc/struct_livox_eth_packet.html
43. Livox-SDK/livox_ros_driver2: Livox device driver under Ros(Compatible with ros and ros2), support Lidar HAP and Mid-360. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver2
44. livox_ros_driver2/README.md / ccf25d6dbb2c5f0c62353002ad2ff8a96cbef74a / CREATE Lab / RoboCup at home / epfl_robocup - GitLab EPFL, accessed July 23, 2025, https://gitlab.epfl.ch/create-lab/robocup_at_home/epfl-robocup/-/blob/ccf25d6dbb2c5f0c62353002ad2ff8a96cbef74a/livox_ros_driver2/README.md
45. Trying out the Livox Mid-360 with ROS1/ROS2 - Tech Blog - Fixstars Corporation, accessed July 23, 2025, https://blog.us.fixstars.com/trying-out-the-livox-mid-360-with-ros1-ros2/
46. Livox-SDK/livox_ros2_driver: Livox device driver under Ros2, support Lidar Mid-40, Mid-70, Tele-15, Horizon, Avia. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros2_driver
47. How to get odometry data from livox mid 360 / Issue #167 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver2/issues/167
48. LVX Specifications v1.0.0.0 - Livox, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20200107/Lvx%20Specifications_v1.0.0.0%20for%20View%200.5.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20200107/Lvx Specifications_v1.0.0.0 for View 0.5.pdf)
49. LVX Specifications v1.1.0.0 Approved by Livox 08/20/2019, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Livox%20Viewer/LVX%20Specifications%20EN_20190924.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Livox Viewer/LVX Specifications EN_20190924.pdf)
50. 2.3. Data format and conversion - Livox wiki - Read the Docs, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/tutorials/other_product/format_conversion.html
51. looking for a guidance on using ouster os1 lidars and livox avia / Issue #197 / koide3/glim, accessed July 23, 2025, https://github.com/koide3/glim/issues/197
52. 360 Lidar - eBay, accessed July 23, 2025, https://www.ebay.com/shop/360-lidar?_nkw=360+lidar
53. LIVOX MID360 LiDAR - RoboStore, accessed July 23, 2025, https://robostore.com/products/livox-mid-360-lidar
54. stemfinity.com, accessed July 23, 2025, [https://stemfinity.com/products/livox-mid360-lidar#:~:text=The%20Livox%20Mid%2D360%20LiDAR,capabilities%20of%20their%20robotic%20systems.](https://stemfinity.com/products/livox-mid360-lidar#:~:text=The Livox Mid-360 LiDAR,capabilities of their robotic systems.)
55. OS0 Ultra-Wide View High-Resolution Imaging Lidar - Ouster, accessed July 23, 2025, https://data.ouster.io/downloads/datasheets/datasheet-rev7-v3p0-os0.pdf
56. Ouster OS0 - AutonomouStuff, accessed July 23, 2025, https://autonomoustuff.com/products/ouster-os0
57. Ouster 3D LiDAR OS0-128-U 128 Channel Laser Scanner | eBay, accessed July 23, 2025, https://www.ebay.com/itm/335349559105
58. Ouster OS0 128 Rev 7 Ultra Wide Field of View Lidar Sensor - RobotShop Europe, accessed July 23, 2025, https://eu.robotshop.com/products/ouster-os0-128-rev-6-ultra-wide-field-of-view-lidar-sensor
59. Velodyne cuts VLP-16 lidar price to $4k - Geo Week News, accessed July 23, 2025, https://www.geoweeknews.com/news/velodyne-cuts-vlp-16-lidar-price-4k
60. Velodyne Puck - eBay, accessed July 23, 2025, https://www.ebay.com/shop/velodyne-puck?_nkw=velodyne+puck
61. Velodyne LiDAR Puck Kit VLP-16 - PNW Scientific, accessed July 23, 2025, https://pnwscientific.com/products/velodyne-lidar-puck-kit-vlp-16
62. Ouster OS0: High-Precision Ultra-Wide Short-Range Lidar Sensor, accessed July 23, 2025, https://ouster.com/products/hardware/os0-lidar-sensor
63. Lidar Variability: A Novel Dataset and Comparative Study of Solid-State - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2507.04321
64. A Novel Dataset and Comparative Study of Solid-State and Spinning Lidars - arXiv, accessed July 23, 2025, https://arxiv.org/html/2507.04321v1
65. Livox + SLAM - Google Groups, accessed July 23, 2025, https://groups.google.com/g/livox-lidars/c/8MmvjOwotw8
66. Create a map using Lidar : r/robotics - Reddit, accessed July 23, 2025, https://www.reddit.com/r/robotics/comments/1i2ocj2/create_a_map_using_lidar/
67. Livox Mid-360 or Robosense Airy : r/LiDAR - Reddit, accessed July 23, 2025, https://www.reddit.com/r/LiDAR/comments/1li59kb/livox_mid360_or_robosense_airy/
68. Laboratory Tests of Metrological Characteristics of a Non-Repetitive Low-Cost Mobile Handheld Laser Scanner - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/24/18/6010


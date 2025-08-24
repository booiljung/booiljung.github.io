# Livox HAP에 대한 기술적 고찰
[Livox Technology](./index.md)


Livox HAP (Horizon Automotive Platform)는 DJI의 기술 인큐베이팅 프로그램을 통해 탄생한 Livox의 첫 번째 차량용 등급(Automotive-Grade) 라이다(LiDAR) 센서다.1 이 센서는 단순히 고성능을 지향하는 연구용 장비를 넘어, 첨단운전자보조시스템(Advanced Driver-Assistance Systems, ADAS) 및 레벨 3/4 자율주행 시장의 대중화를 목표로 설계된 전략적 제품이다. HAP의 핵심 철학은 높은 신뢰성, 대량 생산성, 그리고 합리적인 가격을 통해 기존 고가 라이다 기술의 높은 진입 장벽을 극복하는 데 있다.4 HAP의 출시는 기술적 진보뿐만 아니라, DJI가 보유한 거대한 제조 및 공급망 역량을 라이다 시장에 본격적으로 투입하는 신호탄으로 해석될 수 있다. Livox가 DJI의 'Open Innovation Program'을 통해 설립되었고, DJI의 제조 전문성을 활용하여 가격 경쟁력을 확보했다는 점은 HAP가 단순한 스타트업의 제품이 아님을 명확히 보여준다.6 라이다 산업의 주요 병목 현상으로 지적되어 온 높은 가격, 낮은 생산 확장성, 그리고 부족한 신뢰성 문제를 해결하기 위해, DJI는 드론 시장에서 대량 생산을 통해 가격을 혁신하고 시장을 장악했던 경험과 인프라를 Livox에 이식했다. 이로 인해 HAP는 기술적 성능뿐만 아니라 '생산 가능하고 저렴한' 라이다라는 강력한 시장 포지션을 확보하게 되었으며, 이는 기술적 우위만을 주장하는 다른 경쟁사들과 Livox를 차별화하는 결정적인 요소로 작용한다.

HAP는 이미 중국의 전기차 제조사 Xpeng의 P5 모델과 중국 최대 상용차 제조사 FAW Jiefang의 J7 L3 자율주행 트럭에 탑재되어 실제 양산차 환경에서 그 성능과 신뢰성을 입증했다.2 이는 HAP가 개발 키트를 넘어, 실제 OEM(Original Equipment Manufacturer)의 까다로운 요구사항을 충족하는 양산 부품으로서의 자격을 갖추었음을 의미한다.

HAP는 두 가지 주요 모델로 제공되며, 핵심적인 차이는 데이터 인터페이스에 있다.13 HAP (T1) 모델은 차량 내부 네트워크 표준인 `100BASE-T1 Automotive Ethernet`을 채택하여 OEM의 차량 시스템 통합에 최적화되어 있다.13 반면, HAP (TX) 모델은 일반적인 `100BASE-TX Ethernet`을 사용하여 연구 및 개발 단계에서 보다 용이한 접근성을 제공한다.13 이 외의 핵심적인 성능 지표는 두 모델이 동일하여, 사용자는 적용 환경과 목적에 따라 적합한 인터페이스를 선택할 수 있다. 본 보고서는 Livox HAP의 핵심 기술, 성능, 시스템 통합 환경, 그리고 시장 경쟁력에 대해 심층적으로 분석하여 이 센서가 자율주행 기술의 미래에 미칠 영향을 고찰하고자 한다.


Livox HAP의 기술적 우위는 독창적인 스캐닝 방식에서 비롯된다. 기존 기계식 라이다의 구조적 한계를 극복하고, 효율성과 안전성을 동시에 추구하는 HAP의 스캐닝 아키텍처는 하이브리드 솔리드 스테이트 구조, 비반복 스캔 패턴, 그리고 지능적인 관심 영역 집중 기술로 요약될 수 있다.


HAP는 하이브리드 솔리드 스테이트(Hybrid Solid-State) 또는 세미-솔리드 스테이트(Semi-Solid-State)로 분류되는 구조를 채택한다. 이는 레이저를 발사하는 송신부와 반사된 빛을 감지하는 수신부 등 핵심 전자 장치는 내부에 완전히 고정되어 움직이지 않으며, 회전하는 미러 또는 프리즘(rotating-mirror/prism)을 이용해 레이저 빔의 방향을 제어하는 방식이다.2 센서 전체가 360도 고속 회전하는 전통적인 기계식 라이다와 비교할 때, 움직이는 부품을 최소화함으로써 기계적 마모나 충격으로 인한 고장 가능성을 현저히 낮추고, 내구성과 장기적인 신뢰성을 크게 향상시킨다.14

HAP의 가장 독특한 기술적 특징은 비반복 스캔 패턴(Non-Repetitive Scanning Pattern, NRS)이다.14 기존의 기계식 라이다가 매 프레임마다 동일한 수직 경로를 반복적으로 스캔하는 것과 달리, HAP의 레이저는 시간이 지남에 따라 시야각(Field of View, FOV) 내에서 이전에 스캔하지 않았던 새로운 공간을 지속적으로 탐색하며 스캔 경로를 채워나간다. 이로 인해 생성되는 포인트 클라우드 패턴은 마치 꽃잎이 겹쳐지며 피어나는 듯한 독특한 'flower-like' 형태를 띤다.19 이 방식은 짧은 시간 동안에는 포인트 분포가 불균일해 보일 수 있지만, 누적 시간이 길어질수록 전체 FOV를 촘촘하게 채워나가는 장점을 가진다.


NRS 방식의 가장 큰 장점은 누적 시간(integration time)이 길어질수록 FOV 커버리지가 점진적으로 증가하여 거의 100%에 수렴한다는 점이다.14 HAP의 경우, 0.1초라는 매우 짧은 시간 내에 FOV의 약 80%를 커버하는데, 이는 동일 시간 동안 고정된 라인만을 스캔하는 기존의 64채널 기계식 라이다보다 우수한 성능이다.14 만약 누적 시간을 0.5초까지 늘리면, FOV 커버리지는 99%에 근접하여 이론적으로 FOV 내에 스캔되지 않는 사각지대가 거의 사라지게 된다.14

이는 전통적인 기계식 라이다와의 근본적인 차이점이다. 기계식 라이다는 고정된 수직 채널(예: 64개)을 수평으로 360도 회전시키기 때문에, 스캔 라인과 라인 사이에는 영구적인 사각지대가 존재한다. 아무리 오랜 시간 동안 센서를 작동시켜도 이 빈 공간은 결코 채워지지 않는다.19 이러한 사각지대에 작은 물체나 멀리 있는 보행자가 위치할 경우, 시스템이 이를 전혀 인지하지 못하는 치명적인 문제가 발생할 수 있다. 반면, HAP의 NRS 방식은 시간이 지남에 따라 이 빈 공간을 지속적으로 채워나가므로, 작은 장애물이나 멀리 있는 객체를 놓칠 확률을 획기적으로 줄여 자율주행 시스템의 안전성을 크게 향상시킨다.


HAP는 제한된 자원(포인트 생성 능력)을 가장 효율적으로 사용하기 위해 지능적인 스캔 전략을 사용한다. 전체 FOV(수평 120°, 수직 25°)에 걸쳐 포인트를 균일하게 분배하는 대신, 주행 안전에 가장 중요한 전방 중앙 영역을 관심 영역(Region of Interest, ROI)으로 설정하여 이 부분의 스캔 밀도를 극대화한다.25 HAP의 ROI는 수평 40°, 수직 12°의 범위를 가지며, 이 영역 내에서 0.18°(수평) × 0.23°(수직)의 매우 높은 각도 분해능을 달성한다. 이는 144채널 라이다에 상응하는 포인트 클라우드 밀도로, 멀리 있는 작은 물체(보행자, 자전거, 교통 콘 등)를 정밀하게 인식하고 그 형태를 파악하는 데 결정적인 역할을 한다.2

이러한 공간적 집중 전략에 더해, HAP는 'Ultra FPS (Frames Per Second)'라는 개념을 통해 시간적 집중도 또한 높인다.2 HAP는 기본적으로 10Hz의 프레임 속도로 동작하지만, Ultra FPS 모드가 활성화되면 ROI 내의 객체를 나머지 영역보다 두 배 더 자주 스캔한다. 이는 사실상 ROI 영역에 대해 20Hz에 해당하는 데이터 업데이트 속도를 제공하는 것과 같은 효과를 낸다.11 고속도로 주행이나 보행자가 많은 복잡한 도심 환경과 같이 전방 상황이 급변할 수 있는 시나리오에서, 이 기능은 위험 요소를 더 빨리 인지하고 시스템이 대응할 수 있는 시간을 벌어주어 안전성을 한층 더 강화한다.

HAP의 이러한 스캔 방식(NRS + ROI + Ultra FPS)은 '효율성'과 '안전'이라는, 때로는 상충될 수 있는 두 가지 목표를 지능적으로 조화시킨 결과물이다. 자율주행 시스템의 인지(Perception) 모듈은 제한된 연산 자원을 가지며, 처리해야 할 포인트 수가 많아질수록 지연 시간(latency)이 늘어나 안전에 치명적일 수 있다. HAP는 모든 영역을 균일하게 고밀도로 스캔하여 연산 부하를 가중시키는 대신, '모든 곳을 보는 것'보다 '중요한 곳을 더 잘, 그리고 더 자주 보는 것'이 더 중요하다는 설계 철학을 채택했다. 연산 부하가 많이 걸리는 원거리 및 중앙 영역에 스캔 자원을 공간적으로 집중(ROI)하고, 특히 위험 감지가 시급한 고속 주행 시에는 시간적 자원까지 집중(Ultra FPS)하는 것이다. 이는 HAP가 단순히 물리적 센서를 넘어, '상황 인지'라는 상위 레벨의 요구사항을 고려하여 설계된 '지능형 센서'임을 의미한다. 따라서 개발자는 이러한 스캔 특성을 깊이 이해하고 ROI 내의 고밀도 데이터를 우선적으로 활용하도록 인식 알고리즘을 설계해야만 HAP의 성능을 100% 이끌어낼 수 있다.


HAP는 차량용 등급의 요구사항을 만족시키기 위해 탐지 거리, 정밀도, 데이터 출력률 등 여러 핵심 성능 지표에서 높은 수준의 사양을 갖추고 있다. 이러한 정량적 성능은 실제 주행 환경에서의 객체 탐지 능력으로 이어지며, 동시에 기술적 한계와 악천후 대응 능력도 명확히 인지해야 한다.


Livox HAP의 핵심 기술 사양은 아래 표와 같이 정리할 수 있다. T1과 TX 모델은 데이터 포트를 제외한 모든 성능 지표를 공유한다.

| 매개변수             | 사양                                     | 비고 / 측정 조건                    |
| -------------------- | ---------------------------------------- | ----------------------------------- |
| **모델**             | HAP (T1) / HAP (TX)                      | -                                   |
| **레이저 파장**      | 905 nm                                   | -                                   |
| **레이저 안전 등급** | Class 1 (IEC60825-1:2014)                | Eye Safety (눈에 안전)              |
| **탐지 거리**        | 150 m @ 10% 반사율                       | 100 klx 조도 환경 기준 13           |
|                      | 200 m @ 35% 반사율                       | 회색 콘크리트 벽 등 14              |
| **시야각 (FOV)**     | 120° (수평) × 25° (수직)                 | -                                   |
| **거리 무작위 오차** | <2 cm (1σ)                               | 20m 거리, 80% 반사율 타겟 13        |
| **각도 무작위 오차** | <0.1º (1σ)                               | -                                   |
| **각도 분해능 @ROI** | 0.18° (수평) × 0.23° (수직)              | 144채널 라이다 상당의 밀도 2        |
| **포인트율**         | 452,000 points/s                         | 첫 번째 또는 가장 강한 반사 기준 2  |
| **데이터 포트**      | HAP (T1): 100BASE-T1 Automotive Ethernet | HAP (TX): 100BASE-TX Ethernet 13    |
| **데이터 동기화**    | gPTP                                     | -                                   |
| **오탐지율**         | <0.01%                                   | 100 klx 조도 환경 기준 13           |
| **작동 온도**        | -40℃ to 85℃                              | -                                   |
| **소비 전력**        | 12 W (일반), 26 W (시동 시)              | 저온 시동 시 소비 전력 증가 가능 13 |
| **IP 등급**          | IP67                                     | -                                   |
| **크기**             | 105 × 131.6 × 65 mm                      | -                                   |
| **무게**             | 약 1120 g                                | -                                   |

HAP의 가장 중요한 성능 지표 중 하나는 **탐지 거리**다. 10%의 낮은 반사율을 가진 어두운 색상의 물체에 대해서도 150m의 탐지 거리를 보장하는 것은 고속도로 주행 시 전방의 차량이나 장애물을 충분한 시간적 여유를 두고 인지할 수 있음을 의미한다.1 반사율이 35% 이상인 회색 콘크리트 벽과 같은 일반적인 구조물은 최대 200m까지도 탐지가 가능하여, 주변 지형지물을 이용한 정밀 측위(Localization)에도 유리하다.14

**정밀도** 측면에서, 20m 거리의 80% 반사율 타겟을 측정했을 때 거리 무작위 오차(Random Error)가 2cm 미만(1σ)이라는 것은 객체의 위치와 형태를 매우 신뢰성 있게 파악할 수 있음을 나타낸다.13 이는 차선 변경 시 옆 차량과의 거리, 전방 차량과의 간격, 주차 시 장애물과의 거리 등을 정밀하게 측정해야 하는 ADAS 기능에 필수적인 성능이다.

초당 452,000개의 포인트를 생성하는 **포인트율**은 주변 환경을 상세한 3D 포인트 클라우드로 재구성하기 위한 기본 데이터 양을 결정한다.2 특히 ROI 내에서 144채널 라이다에 상응하는 고밀도 데이터를 생성하는 능력은 멀리 있는 작은 객체의 윤곽을 명확하게 식별하는 데 기여한다.


HAP의 사양은 실제 주행 환경에서의 탐지 성능으로 뒷받침된다. Livox가 공개한 테스트 결과에 따르면, HAP는 150m 거리의 흰색 세단을 효과적으로 탐지했으며, 200m 거리에서도 차량 형태를 식별할 수 있는 7개 이상의 포인트를 포착했다. 반사율이 더 낮은 검은색 차량에 대해서도 150m에서 9개, 200m에서 5개의 포인트를 감지하는 성능을 보였다.25 이는 다양한 색상과 재질의 차량을 안정적으로 인식할 수 있음을 시사한다. 또한, ADAS 시스템의 중요한 과제 중 하나인 작은 장애물 인식 능력도 입증되었다. 110m 거리의 작은 교통 콘에 대해 5개의 포인트를, 45m 거리의 축구공에 대해 6개의 포인트를, 그리고 30m 거리의 15cm 높이 물병에 대해서도 3개 이상의 포인트를 감지했다.25

HAP의 전신 모델인 Livox Horizon에 대한 벤치마크 테스트 결과는 HAP의 성능을 간접적으로 유추하는 데 참고가 될 수 있다.29 해당 테스트에서 Horizon은 경쟁 모델인 Velodyne Velarray나 Robosense M1에 비해 동적 차량 탐지 시나리오에서 250m 이상의 월등한 최대 탐지 거리를 기록했다. 이는 Livox의 비반복 스캔 패턴이 원거리 객체에 일부 빔을 도달시킬 확률이 높기 때문으로 분석된다. 다만, 해당 테스트에서 Horizon은 위치 정확도와 포인트 밀도 면에서는 경쟁 모델 대비 다소 부족한 모습을 보였다. HAP는 Horizon 대비 탐지 거리와 포인트율이 모두 향상된 모델이므로 18, 이러한 단점을 상당 부분 개선했을 것으로 기대된다.


모든 센서와 마찬가지로 HAP 역시 기술적 한계를 가지고 있다. 가장 중요한 것은 **블라인드 존(Blind Zone)**이다. HAP는 센서 전면으로부터 0.5m 이내의 물체는 정밀하게 탐지할 수 없다.15 또한 0.5m에서 2m 사이의 비교적 가까운 거리에 있는 객체에 대해서는 포인트 클라우드가 어느 정도 왜곡될 수 있다.15 이 특성은 센서를 차량 범퍼 전면에 장착했을 때, 차량 바로 앞 지면이나 매우 근접한 장애물을 감지하는 데 영향을 줄 수 있는 요소이므로 시스템 설계 시 반드시 고려해야 한다.

**악천후 성능**은 자율주행 시스템의 강건성을 결정하는 핵심 요소다. HAP의 사용자 매뉴얼은 안개나 폭풍우와 같은 저시정성 환경에서는 레이저 빔의 산란 및 흡수로 인해 유효 탐지 거리가 감소할 수 있음을 명시하고 있다.18 이는 905nm 파장대 레이저를 사용하는 모든 라이다가 공통으로 가지는 물리적 한계다. 하지만 Livox는 이를 보완하기 위한 독자적인 솔루션을 제공한다. HAP가 출력하는 포인트 클라우드 데이터의 각 포인트에는 'Tag'라는 추가 정보가 포함되어 있는데, 이 정보는 반사된 에코(echo)의 신뢰도 수준을 나타낸다.30 Livox는 이 Tag 정보를 이용하여 비, 안개, 먼지 등 악천후 상황에서 발생하는 노이즈 포인트를 식별하고 필터링하는 기능을 내장했다. 예를 들어, 에코 에너지 강도가 약한 포인트는 '먼지일 확률이 높음', 중간 정도인 포인트는 '비나 안개일 확률이 높음'으로 분류된다.30 이를 통해 후처리 알고리즘 단에서 악천후로 인한 허상(false positive)을 효과적으로 제거하고, 다른 센서(카메라, 레이더)와의 융합을 통해 악천후 상황에서의 인지 강건성을 높일 수 있다.


HAP는 단순히 하드웨어 센서만 제공하는 것을 넘어, 개발자가 차량 및 로봇 시스템에 쉽게 통합하고 활용할 수 있도록 포괄적인 하드웨어 인터페이스와 풍부한 소프트웨어 생태계를 함께 제공한다. 그러나 HAP의 독특한 스캔 방식은 기존 알고리즘에 새로운 과제를 제시하며, Livox는 이를 해결하기 위한 특화된 솔루션을 적극적으로 지원한다.


HAP는 105 × 131.6 × 65 mm의 컴팩트한 크기와 약 1120 g의 무게를 가지며, 차량 외부에 장착하기에 적합하도록 설계되었다.13 M6 관통 홀과 M3 마운팅 홀을 제공하여 다양한 브라켓을 통해 차량에 견고하게 통합될 수 있다.16

데이터 인터페이스는 모델에 따라 명확히 구분된다. **HAP (T1)** 모델은 `100BASE-T1 Automotive Ethernet`을 사용한다.13 이는 단일 연선(single twisted pair)을 기반으로 하는 차량 내 통신 표준으로, 기존 이더넷 케이블보다 무게와 부피가 작고 배선이 간편하여 양산차 적용에 매우 유리하다. 반면, **HAP (TX)** 모델은 표준 `100BASE-TX Ethernet` (일반적인 RJ45 커넥터)을 사용하므로, 개발 및 테스트 단계에서 별도의 변환기 없이 일반적인 PC나 네트워크 스위치에 직접 연결하여 데이터를 확인할 수 있는 편리함을 제공한다.13

전원 공급은 9V에서 18V DC의 넓은 범위를 지원하여 차량의 전압 변동에 안정적으로 대응할 수 있으며, 일반 작동 시 12W, 저온 환경에서의 시동 시에는 최대 26W의 전력을 소비한다.13 여러 개의 센서를 사용하는 자율주행 시스템에서 각 센서 데이터의 시간 정합성은 매우 중요하다. HAP는 이를 위해 `gPTP (generalized Precision Time Protocol)`를 지원하여, 네트워크에 연결된 모든 센서들이 마이크로초 단위의 정밀도로 시간을 동기화할 수 있도록 한다.13


Livox는 강력한 소프트웨어 생태계를 통해 HAP의 활용성을 극대화한다. 핵심은 **Livox SDK2**로, HAP와 Mid-360과 같은 최신 센서들을 지원하는 차세대 소프트웨어 개발 키트다.32 C/C++ 기반의 API를 제공하며, Windows, Linux (Ubuntu 18.04/20.04), 그리고 로봇 개발의 표준 플랫폼인 ROS/ROS2 환경을 포괄적으로 지원한다.14 개발자는 이 SDK를 통해 라이다 센서에 쉽게 연결하고, 포인트 클라우드 데이터를 수신하며, 스캔 모드나 프레임 속도와 같은 다양한 파라미터를 제어할 수 있다.

**Livox Viewer 2**는 HAP 전용 시각화 및 분석 도구다.14 이 소프트웨어를 통해 사용자는 실시간으로 수신되는 포인트 클라우드 데이터를 3D로 확인할 수 있으며, 데이터를 lvx 파일 형식으로 녹화하고 재생할 수 있다. 또한, 센서의 파라미터를 GUI 환경에서 편리하게 설정하거나, 여러 센서를 함께 사용할 때 필요한 외부 파라미터 보정(Extrinsics Calibration) 작업을 수행하는 기능도 제공한다.

단순한 드라이버와 뷰어를 넘어, Livox는 GitHub를 통해 HAP의 특성을 활용할 수 있는 다양한 **오픈소스 알고리즘**을 적극적으로 공개하고 있다.34 이는 개발자들이 HAP를 활용하여 SLAM(동시적 위치추정 및 지도작성), 객체 탐지, 차선 인식 등의 애플리케이션을 더 빠르고 쉽게 개발할 수 있도록 지원하는 강력한 생태계 전략의 일환이다.


HAP의 혁신적인 비반복 스캐닝 방식은 장점이 많은 만큼, 기존 라이다 센서를 기준으로 개발된 전통적인 알고리즘에는 새로운 과제를 제시한다.

첫째, **SLAM에서의 퇴화(Degeneracy) 문제**다. HAP의 비균일 스캔 패턴과 상대적으로 좁은 수직 FOV는 긴 복도, 터널, 또는 특징 없는 넓은 평면 벽과 같이 기하학적 특징점이 부족한(feature-poor) 환경에서 SLAM 알고리즘의 '퇴화' 현상을 유발할 수 있다.36 퇴화란, 특정 방향으로의 움직임을 측정할 충분한 기하학적 제약조건을 포인트 클라우드로부터 얻지 못해 위치 추정의 불확실성이 급격히 증가하고 심각한 오차(drift)가 누적되는 현상을 의미한다.

이 문제를 해결하기 위해 Livox는 **LIO-Livox**라는 강력한 라이다-관성 주행 거리 측정(LiDAR-Inertial Odometry) 시스템을 오픈소스로 제공한다.36 LIO-Livox는 라이다 데이터와 센서에 내장된 IMU(관성측정장치) 데이터를 강결합(Tightly-coupled) 방식으로 융합한다. 라이다가 특징점 부족으로 퇴화 현상을 겪는 순간에도, IMU는 독립적으로 고주파의 가속도와 각속도 정보를 제공하여 모션을 추정할 수 있다. LIO-Livox는 이 두 정보를 최적화 기법을 통해 통합하여, 한쪽 센서의 약점을 다른 쪽 센서의 정보로 보완함으로써 시스템 전체의 강건성(robustness)을 크게 향상시킨다.36

둘째, **외부 파라미터 보정(Extrinsic Calibration)의 어려움**이다. HAP를 카메라와 같은 다른 종류의 센서와 함께 사용하려면, 두 센서 간의 상대적인 위치와 방향(Extrinsic Parameters)을 정밀하게 측정하는 보정 과정이 필수적이다. 일반적으로 체커보드와 같은 보정 타겟을 사용하여 공통된 특징점(모서리, 평면 등)을 각 센서에서 추출하고 이를 매칭하여 외부 파라미터를 계산한다. 하지만 HAP의 비균일하고 시간에 따라 계속 변하는 포인트 클라우드 분포는 보정 타겟의 특징점을 안정적으로, 그리고 충분한 밀도로 추출하는 것을 매우 어렵게 만든다.40 이 문제를 해결하기 위해, 짧은 시간 동안 포인트를 누적하여 타겟 표면의 밀도를 인위적으로 높이거나 42, 포인트의 반사율 정보를 활용하여 체커보드의 흑백 패턴 경계를 추정하는 등 HAP의 데이터 특성에 맞는 특화된 보정 알고리즘들이 연구되고 있다.40

Livox의 이러한 소프트웨어 및 알고리즘 제공 전략은 단순한 '기술 지원'을 넘어선다. 이는 자사 하드웨어의 '내재적 약점'을 '시스템 레벨의 강점'으로 전환시키는 고도의 엔지니어링 접근법으로 볼 수 있다. HAP의 NRS는 사각지대 제거에 유리하지만, 순간적인 데이터 불균일성이라는 명백한 단점을 소프트웨어 개발자에게 전가한다. Livox는 이 문제를 회피하는 대신, LIO-Livox나 자동 보정 알고리즘 연구와 같이 정면으로 돌파하는 솔루션을 직접 개발하고 공개한다. 이는 "우리 센서는 이런 단점이 있지만, 우리가 제공하는 이 소프트웨어를 사용하면 문제가 해결될 뿐만 아니라, IMU 융합을 통해 오히려 더 강건한 시스템을 만들 수 있다"는 강력한 메시지를 전달한다. 즉, 하드웨어의 단점을 인정하고 이를 해결하는 우수한 소프트웨어 솔루션을 함께 제공함으로써 전체 시스템의 신뢰도를 높이는 역설적인 효과를 창출하는 것이다. 이는 단순히 하드웨어 스펙 경쟁에 몰두하는 경쟁사들과 차별화되는 지점이며, '하드웨어+소프트웨어 통합 솔루션'을 통해 개발자 생태계를 구축하고 고객의 최종 목표 달성을 직접적으로 돕는, HAP의 시장 침투를 가속화하는 핵심 동력으로 작용한다.


Livox HAP의 가장 중요한 정체성은 '차량용 등급(Automotive-Grade)'과 '대량 생산(Mass Production)'이라는 두 가지 키워드로 요약된다. 이는 HAP가 단순한 기술 시연용 프로토타입이 아니라, 실제 자동차의 수명 주기 동안 혹독한 환경에서도 안정적으로 작동해야 하는 핵심 부품임을 의미한다. 이러한 목표를 달성하기 위해 Livox는 엄격한 신뢰성 표준을 충족하고, 고도로 자동화된 양산 체계를 구축했다.


HAP는 차량용 부품에 요구되는 국제 표준인 ISO 16750을 포함하여, 총 70개 이상의 차량용 등급 신뢰성 테스트를 통과했다.2 이 테스트들은 실제 차량이 겪을 수 있는 모든 종류의 스트레스 상황을 모사한다. 예를 들어, -40℃의 혹한에서 85℃의 폭염에 이르는 극단적인 온도 변화, 비포장도로 주행 시 발생하는 지속적인 진동, 갑작스러운 충격, 그리고 차량 내 다른 전자 장비와의 전자파 간섭(EMC) 등이 포함된다.12 HAP가 이러한 테스트를 모두 통과했다는 것은 10년 이상의 차량 수명 주기 동안 안정적인 성능을 유지할 수 있는 내구성과 신뢰성을 공식적으로 검증받았음을 의미한다.

이러한 높은 신뢰성은 HAP의 하이브리드 솔리드 스테이트 구조에 크게 기인한다. 센서 전체가 고속으로 회전하는 기계식 라이다는 장시간 사용 시 모터 베어링의 마모나 회전부의 불균형으로 인한 고장 가능성이 상존한다. 반면, HAP는 레이저 송수신부와 같은 핵심 전자 부품이 모두 고정되어 있고, 상대적으로 작고 가벼운 미러/프리즘만이 회전하기 때문에 기계적 마모나 고장 가능성을 원천적으로 줄여 장기적인 신뢰성 확보에 결정적인 이점을 가진다.14


Livox는 HAP의 대량 공급을 위해 연간 최대 20만 개의 라이다를 생산할 수 있는 완전 자동화된 조립 라인을 구축했다.1 이는 Xpeng과 같은 대형 OEM의 대량 주문에 안정적으로 대응하고, 규모의 경제를 통해 생산 원가를 절감하는 기반이 된다.

이러한 대량 생산의 핵심에는 Livox의 특허 기술인 **DL-Pack (Direct Laser-Pack)**이 있다.4 전통적인 라이다 생산 공정에서 가장 많은 시간과 비용이 소요되는 부분은 여러 개의 레이저 다이오드와 수광 센서를 정밀하게 정렬하고 초점을 맞추는 수동 보정(calibration) 작업이다. DL-Pack 기술은 다수의 레이저 송신기와 수신기를 하나의 반도체 패키지처럼 통합하고, 이 과정을 자동화하여 보정 작업을 수행한다. 이를 통해 생산 효율을 기존 방식 대비 수십 배 향상시키고, 수작업으로 인한 제품 간 품질 편차를 최소화하여 균일한 성능을 보장한다.12

HAP의 가격 경쟁력은 모회사인 DJI와의 강력한 시너지 효과에서 비롯된다. Livox는 세계 최대의 드론 제조사인 DJI의 성숙한 공급망 관리(SCM) 시스템, 대량 생산 노하우, 그리고 ppm(parts per million) 단위로 불량률을 관리하는 엄격한 품질 관리 시스템을 적극적으로 활용한다.6 이를 통해 고품질의 광학 및 전자 부품을 저렴한 가격에 대량으로 조달하고, 고도로 자동화된 생산 라인에서 높은 수율로 제품을 생산함으로써 HAP의 가격 경쟁력을 극대화한다.

HAP가 '차량용 등급' 인증을 받고 '양산 체계'를 갖추었다는 것은 단순한 품질 보증을 넘어, 라이다 기술의 패러다임이 '고가의 연구 장비'에서 '표준화된 자동차 부품'으로 전환되고 있음을 상징한다. 전통적인 자동차 산업의 Tier-1 공급업체와 OEM들은 수백만 대 단위의 안정적인 공급 능력, 10년 이상의 내구성, 그리고 ppm 단위의 엄격한 불량률 관리를 요구한다. 이는 소량 생산되던 기존의 고가 연구용 라이다와는 완전히 다른 차원의 요구사항이다. Livox는 프로젝트 초기부터 이 '자동차 부품' 시장을 목표로 삼았기 때문에, 센서의 성능뿐만 아니라 생산 기술과 품질 관리 프로세스에 막대한 투자를 감행했다. DL-Pack과 같은 기술은 단순히 원가를 낮추는 것을 넘어, 대량 생산 시에도 모든 제품이 동일한 성능을 내도록 보장하는 핵심 기술이다. 이것이 바로 HAP가 Xpeng이나 FAW Jiefang과 같은 대형 OEM의 까다로운 공급망에 성공적으로 편입될 수 있었던 근본적인 이유다. 경쟁사들이 프로토타입의 뛰어난 성능을 홍보할 때, Livox는 '언제든, 원하는 만큼, 동일한 품질로, 합리적인 가격에 공급할 수 있음'을 실제 양산을 통해 증명했다. 이 능력이야말로 라이다 기술이 ADAS에 보편적으로 적용되기 위한 가장 중요한 전제조건이며, HAP가 시장의 게임 체인저가 될 수 있는 잠재력의 원천이다.


Livox HAP는 기술적 혁신과 양산 능력을 바탕으로 빠르게 성장하는 차량용 라이다 시장에 진입했지만, 이미 강력한 경쟁자들이 시장을 선점하기 위해 치열하게 경쟁하고 있다. HAP의 시장에서의 위치를 이해하기 위해서는 가격 전략, 주요 경쟁 기술 및 업체, 그리고 거시적인 기술 트렌드를 종합적으로 분석할 필요가 있다.


HAP는 2022년 7월, 일반 개발자 및 소규모 프로젝트를 대상으로 1,599달러라는 공격적인 가격으로 판매를 시작했다.9 이는 대량으로 공급되는 OEM 계약 시 가격이 1,000달러 이하, 잠재적으로는 500달러 수준까지 하락할 수 있음을 시사한다.46 이러한 가격 정책은 라이다를 일부 고가의 프리미엄 차량에만 적용되던 기술에서, Xpeng P5와 같은 중가형 전기차나 FAW의 상용 트럭과 같이 판매량이 많은 볼륨 모델로까지 채택을 확산시키려는 명확한 전략을 보여준다.2 즉, HAP의 핵심 목표 시장은 ADAS 기능의 대중화가 이루어지는 메인스트림 자동차 시장이다.


HAP는 시장에서 여러 강력한 경쟁자들과 마주하고 있다. 특히 중국 시장을 중심으로 빠르게 성장한 RoboSense와 Hesai가 가장 직접적인 경쟁 상대이며, 기술적 차별점을 가진 Luminar 역시 중요한 플레이어다.

| 특징                   | Livox HAP                                     | RoboSense M1 Plus                    | Luminar Iris                                     |
| ---------------------- | --------------------------------------------- | ------------------------------------ | ------------------------------------------------ |
| **스캐닝 기술**        | 하이브리드 솔리드 스테이트 (회전 미러/프리즘) | 하이브리드 솔리드 스테이트 (2D MEMS) | 하이브리드 솔리드 스테이트 (2축 갈바노미터 미러) |
| **레이저 파장**        | 905 nm                                        | 905 nm                               | 1550 nm                                          |
| **탐지 거리 (@10%)**   | 150 m                                         | 180 m                                | 250 m                                            |
| **시야각 (수평x수직)** | 120° × 25°                                    | 120° × 25°                           | 120° × 26°                                       |
| **각도 분해능 (ROI)**  | 0.18° × 0.23°                                 | 0.2° × 0.1° (ROI)                    | 0.05° × 0.05° (ROI)                              |
| **포인트율**           | 452,000 pts/s                                 | 1,575,000 pts/s (이중 반사)          | -                                                |
| **핵심 기능**          | 비반복 스캐닝, Ultra FPS                      | 지능형 "GAZE" 기능                   | 고출력, 악천후 성능 우위                         |
| **목표 가격 (양산)**   | < $1,000                                      | < $500 (MX 시리즈: < $200)           | $500 - $1,000                                    |

- **RoboSense (M 시리즈):** HAP와 가장 직접적인 경쟁 구도를 형성하는 업체다. MEMS(미세전자기계시스템) 미러를 이용한 스캐닝 방식을 사용하며, HAP와 동일한 905nm 파장대와 120° x 25°의 FOV를 제공한다.48 RoboSense의 핵심 기술은 특정 영역의 해상도를 동적으로 높이는 'GAZE' 기능으로, 이는 HAP의 ROI 스캔과 유사한 개념으로 경쟁한다.51 RoboSense는 2024년 글로벌 승용차 라이다 시장에서 26%의 점유율로 1위를 차지했으며, 수많은 중국 및 글로벌 OEM과의 디자인 윈(Design Win)을 통해 시장을 주도하고 있다.55
- **Hesai (AT 시리즈):** RoboSense와 함께 시장을 양분하는 또 다른 강자다. 특히 로보택시 시장에서 압도적인 점유율(74%)을 바탕으로 기술력을 입증했으며, 이를 기반으로 ADAS 시장에서도 빠르게 점유율을 확대하고 있다.58 Hesai의 핵심 경쟁력은 자체적인 ASIC 칩 개발 능력과 고도로 자동화된 생산 라인을 통한 원가 절감 능력에 있다.61 이를 통해 성능과 가격 경쟁력을 동시에 확보하는 전략을 구사한다.
- **Luminar (Iris):** 기술적으로 HAP와 차별화된 경쟁자다. 1550nm 파장대의 레이저를 사용하여 더 높은 출력을 낼 수 있어, 905nm 대비 더 먼 거리의 어두운 물체를 탐지하는 데 유리하며 악천후 투과성도 더 뛰어나다.62 하지만 1550nm 기술은 인듐갈륨비소(InGaAs) 기반의 고가 부품을 필요로 하기 때문에, HAP, RoboSense, Hesai 등이 주도하는 905nm 기반 센서들의 가격 경쟁에서는 다소 불리한 위치에 있다. 주로 Volvo, Mercedes-Benz 등 기술 선도를 중시하는 프리미엄 브랜드와 파트너십을 맺고 있다.


현재 차량용 라이다 시장은 몇 가지 중요한 기술적 변곡점을 맞이하고 있다. 첫째는 **905nm와 1550nm 파장대 간의 경쟁**이다. HAP가 속한 905nm 진영은 성숙한 실리콘 기반 검출기 기술을 사용하여 비용 효율성이 매우 높기 때문에 현재 ADAS 시장의 주류를 형성하고 있다.63 반면, 1550nm 진영은 더 높은 성능과 안전성을 제공하지만, 고가의 부품으로 인해 가격이 높다.62 장기적으로 1550nm 기술의 가격 하락이 예상되지만, ADAS 대중화를 이끄는 향후 수년간은 905nm 기반 센서들이 가격 경쟁력을 바탕으로 시장을 주도할 것으로 보인다.

둘째는 **진정한 솔리드 스테이트로의 진화**다. HAP와 같은 하이브리드 방식은 기계식 라이다의 신뢰성 문제를 해결한 중요한 진보이지만, 여전히 미세한 움직이는 부품을 포함하는 과도기적 기술이다. 장기적으로는 MEMS를 넘어 움직이는 부품이 전혀 없는 OPA(Optical Phased Array)나 Flash LiDAR가 궁극적인 솔루션으로 여겨진다.65 이 기술들은 반도체 공정을 통해 생산되므로 소형화, 저전력화, 그리고 극적인 원가 절감이 가능하다. 하지만 2025년 현재, 이 기술들은 아직 성능(특히 탐지 거리와 FOV), 가격, 신뢰성 측면에서 대량 양산 차량에 적용되기에는 성숙도가 부족한 상태다.67

이러한 시장 환경 속에서 Livox는 **시장 침투 과제**에 직면해 있다. Xpeng, FAW와의 초기 파트너십을 통해 시장 진입에는 성공했으나, RoboSense와 Hesai가 수많은 중국 및 글로벌 OEM과 대규모 디자인 윈을 확보하며 빠르게 시장을 장악하고 있다.55 Yole Group의 2023년 시장 분석에 따르면, Livox는 XPeng과의 파트너십 덕분에 8%의 점유율을 차지했으나, Hesai와 RoboSense는 이미 시장의 선두 그룹을 형성했다.70 Livox가 이들과의 경쟁에서 시장 점유율을 확대하기 위해서는 더 많은 OEM 파트너십을 확보하고, DJI와의 시너지를 극대화하여 지속적인 가격 경쟁력을 유지하는 것이 향후 가장 중요한 과제가 될 것이다.


Livox HAP는 자율주행 기술의 역사에서 중요한 변곡점을 대표하는 센서다. 이는 단순히 또 하나의 고성능 라이다가 아니라, 첨단 기술을 대중이 접근 가능한 영역으로 끌어내리려는 명확한 목표를 가지고 설계된 제품이기 때문이다.

HAP의 **기술적 성취**는 비반복 스캐닝(NRS)이라는 혁신적인 기술에 집약되어 있다. 이 기술은 기존 기계식 라이다의 고질적인 한계였던 스캔 라인 간의 영구적인 사각지대 문제를 해결하고, 누적 시간을 통해 FOV 커버리지를 100%에 가깝게 채워나가는 독창적인 해법을 제시했다. 여기에 더해, 주행 안전에 가장 중요한 전방 중앙 영역에 스캔 자원을 집중하는 ROI 및 Ultra FPS 기능은 제한된 자원을 가장 효율적으로 사용하여 인지 성능을 극대화하는 지능적인 접근법이다. 이는 하드웨어가 소프트웨어의 요구사항과 제약을 깊이 이해하고 설계되었을 때 얼마나 강력한 시너지를 낼 수 있는지를 보여주는 대표적인 사례다.

그러나 HAP의 진정한 **시장 경쟁력**은 기술적 성능에만 국한되지 않는다. 그 핵심에는 모회사 DJI의 강력한 제조 인프라를 기반으로 한 '차량용 등급 신뢰성', '대규모 양산 능력', 그리고 '합리적인 가격'이라는 세 가지 축이 있다. 이 세 요소의 결합은 라이다를 소수의 고가 차량이나 연구용 플랫폼을 위한 특수 장비에서, 수백만 대의 양산차에 탑재 가능한 표준화된 '자동차 부품'으로 전환시키는 결정적인 역할을 한다. 이는 ADAS 기술이 대중에게 보편적으로 보급되기 위한 가장 중요한 전제조건이며, HAP가 시장의 '게임 체인저'가 될 수 있는 가장 큰 잠재력이다.

물론 HAP 앞에는 **향후 과제** 또한 명확하다. 비균일 스캔 패턴은 여전히 소프트웨어 개발에 높은 허들로 작용하며, 이는 개발자 생태계의 폭넓은 지지를 얻기 위해 지속적으로 해결해야 할 문제다. Livox가 LIO-Livox와 같은 강력한 오픈소스 솔루션을 제공하는 것은 매우 긍정적이지만, 더 많은 개발자들이 쉽게 접근하고 활용할 수 있도록 생태계를 지속적으로 확장하고 지원해야 한다. 또한, 이미 시장을 선점하고 있는 RoboSense와 Hesai가 주도하는 치열한 시장 경쟁 속에서 더 많은 OEM 파트너십을 확보하고, 지속적인 원가 절감을 통해 가격 경쟁력을 유지하는 것이 시장 점유율 확대의 관건이 될 것이다.

결론적으로, Livox HAP는 기술과 양산성의 절묘한 균형을 통해 ADAS 및 자율주행 기술의 대중화를 이끄는 중요한 촉매제 역할을 할 잠재력을 충분히 갖추고 있다. HAP의 성공 여부는 단순히 센서 하나의 성능을 넘어, 라이다 기술이 진정한 의미의 대중화 시대로 진입할 수 있는지를 가늠하는 중요한 척도가 될 것이다.


1. Livox HAP (TX) - DJI Store, accessed July 23, 2025, https://store.dji.com/hk-en/product/livox-hap
2. Livox HAP LiDAR, accessed July 23, 2025, https://www.livoxtech.com/hap
3. 1. Product - Livox wiki 0.1 documentation, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/introduction/production.html
4. Empowering Xpeng P5: Livox Officially Releases HAP Lidar - PR Newswire, accessed July 23, 2025, https://www.prnewswire.com/news-releases/empowering-xpeng-p5-livox-officially-releases-hap-lidar-301268675.html
5. DJI-funded Livox marks leap for self-driving cars with two new, low-cost Lidar sensors, accessed July 23, 2025, https://www.thedronegirl.com/2020/01/06/dji-livox-lidar/
6. DJI Drone LiDAR: Introducing Livox and Avia - DRONELIFE, accessed July 23, 2025, https://dronelife.com/2020/08/25/dji-drone-lidar-introducing-livox-and-avia/
7. DJI Showcases Wide Product Portfolio At CES 2020 Including New Incubated Livox Lidar Technology, accessed July 23, 2025, https://www.dji.com/newsroom/news/dji-showcases-wide-product-portfolio-at-ces-2020
8. The Future Has Arrived: Livox Helps JD Logistics Build Its L4 Unmanned Delivery System, accessed July 23, 2025, https://www.livoxtech.com/showcase/11
9. The preferred choice by Xpeng P5 is extending its reach to spatial intelligence developers worldwide - Livox, accessed July 23, 2025, https://www.livoxtech.com/showcase/autonomous/3
10. Livox Car Lidar Now Up For Ordering - DVN - Driving Vision News, accessed July 23, 2025, https://www.drivingvisionnews.com/news/2022/08/03/livox-hap-automotive-grade-lidar-available-for-online-ordering/
11. Xpeng Partners with Livox to Deploy Lidar Technology in the New 2021 Production Model, accessed July 23, 2025, https://ir.xiaopeng.com/news-releases/news-release-details/xpeng-partners-livox-deploy-lidar-technology-new-2021-production/
12. Joint Collaboration Between Livox, Zhito and FAW Jiefang Propels Autonomous Heavy-Duty Truck Into the Smart Driving Era, accessed July 23, 2025, https://www.livoxtech.com/news/11
13. Specs - HAP LiDAR Sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/hap/specs
14. User Manual - LIVOX HAP, accessed July 23, 2025, https://dl.djicdn.com/downloads/Livox/HAP/HAP(T1)_User_Manual_V1.2_EN.pdf
15. LIVOX HAP T1 Automotive Grade Lidar for Serial Production User Manual - device.report, accessed July 23, 2025, https://device.report/manual/3994406
16. LIVOX HAP T1 Automotive Grade Lidar for Serial Production User Manual, accessed July 23, 2025, https://manuals.plus/livox/hap-t1-automotive-grade-lidar-for-serial-production-manual
17. Livox HAP (TX) Lidar - DeviEstore, accessed July 23, 2025, https://deviestore.com/product/livox-hap-lidar/
18. LIVOX HAP High Performance LiDAR Sensor User Guide - Manuals.plus, accessed July 23, 2025, https://manuals.plus/livox/hap-high-performance-lidar-sensor-manual
19. Livox Introduces High Performance, Low Cost, Mass Market Lidar Sensors For L3/L4 Autonomous Driving Applications, accessed July 23, 2025, https://www.livoxtech.com/jp/news/4
20. DJI's Livox introduces high performance, low cost mass market lidar sensors - DroneDJ, accessed July 23, 2025, https://dronedj.com/2020/01/06/djis-livox-introduces-high-performance-low-cost-mass-market-lidar-sensors/
21. LIVOX HAP - DJI, accessed July 23, 2025, https://dl.djicdn.com/downloads/Livox/HAP/HAP(TX)_User_Manual_V1.2_EN.pdf
22. Point Cloud Characteristics | Livox, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Point%20cloud%20characteristics.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Point cloud characteristics.pdf)
23. 2. Introduction to Livox Scanning Features, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/introduction/livox_scanning_pattern.html
24. Livox Mid-40 lidar first review! with pictures : r/SelfDrivingCars - Reddit, accessed July 23, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/aimdqz/livox_mid40_lidar_first_review_with_pictures/
25. Livox's First Automotive-grade LiDAR HAP is Open for Public Ordering at $1,599, accessed July 23, 2025, https://www.livoxtech.com/news/hap_spec
26. Livox HAP (TX IMU) - HPDRONES, accessed July 23, 2025, https://store.hp-drones.com/en/livox/2918-EALVHPD937223-6941565937223.html
27. Buy Livox Horizon HAP at Robu.in, accessed July 23, 2025, https://robu.in/product/livox-horizon-hap-lidar/
28. An Introduction guide on setting up LIVOX HAP Lidar | by Janet Lin - Medium, accessed July 23, 2025, https://medium.com/@janetlinw/an-introduction-guide-on-setting-up-livox-hap-lidar-54881600c26a
29. Benchmarking of Various LiDAR Sensors for Use in Self-Driving Vehicles in Real-World Environments - PMC, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9572247/
30. How to use Tag Information in Livox LiDAR Point Cloud, accessed July 23, 2025, https://www.livoxtech.com/showcase/livox-tag
31. 3. Livox Point Cloud and Coordinate System - Livox wiki 0.1 documentation, accessed July 23, 2025, [https://livox-wiki-en.readthedocs.io/en/latest/introduction/Point_Cloud_Characteristics_and_Coordinate_System%20.html](https://livox-wiki-en.readthedocs.io/en/latest/introduction/Point_Cloud_Characteristics_and_Coordinate_System .html)
32. Livox-SDK - GitHub, accessed July 23, 2025, https://github.com/livox-sdk
33. Livox-SDK/Livox-SDK2: Drivers for receiving LiDAR data and controlling lidar, support Lidar HAP and Mid-360. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/Livox-SDK2
34. Livox SDK, accessed July 23, 2025, https://www.livoxtech.com/sdk
35. Livox Viewer User Manual, accessed July 23, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Livox%20Viewer/Livox%20Viewer%20User%20Manual.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Livox Viewer/Livox Viewer User Manual.pdf)
36. Livox-SDK/LIO-Livox: A Robust LiDAR-Inertial Odometry for ... - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/LIO-Livox
37. GM-Livox: An Integrated Framework for Large-Scale Map Construction with Multiple Non-repetitive Scanning LiDARs - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/355222210_GM-Livox_An_Integrated_Framework_for_Large-Scale_Map_Construction_with_Multiple_Non-repetitive_Scanning_LiDARs
38. A Robust Framework for Simultaneous Localization and Mapping with Multiple Non-Repetitive Scanning Lidars - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/13/10/2015
39. Map Construction Based on LiDAR Vision Inertial Multi-Sensor Fusion - MDPI, accessed July 23, 2025, https://www.mdpi.com/2032-6653/12/4/261
40. [2011.08516] ACSC: Automatic Calibration for Non-repetitive Scanning Solid-State LiDAR and Camera Systems - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2011.08516
41. Automatic Calibration for Livox LiDAR and Camera from ACSC - Products, accessed July 23, 2025, https://forum.livoxtech.com/thread-48-1-1.html
42. (PDF) ACSC: Automatic Calibration for Non-repetitive Scanning Solid-State LiDAR and Camera Systems - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/345989415_ACSC_Automatic_Calibration_for_Non-repetitive_Scanning_Solid-State_LiDAR_and_Camera_Systems
43. Livox | In-house Product Reliability Testing for Automotive-grade Performance - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=ErG04a1uSGU
44. Livox's First Automotive-grade LiDAR HAP is Open for Public Ordering at $1,599, accessed July 23, 2025, https://www.livoxtech.com/news/hap_order
45. Livox's First Automotive-grade LiDAR, Livox HAP Available for Online Order at $1599, accessed July 23, 2025, https://www.reddit.com/r/LiDAR/comments/vwhcvb/livoxs_first_automotivegrade_lidar_livox_hap/
46. Luminar raises $100M to scale LiDAR for autonomous vehicles - The Robot Report, accessed July 23, 2025, https://www.therobotreport.com/luminar-raises-100m-lidar-autonomous-vehicles/
47. Luminar lidar to hit the road in 2022 with 'sub-$1000' price tag - Optics.org, accessed July 23, 2025, https://optics.org/news/10/7/22
48. Robosense-RS-LiDAR-M1-solid-state-datasheet.pdf, accessed July 23, 2025, https://www.generationrobots.com/media/Robosense-RS-LiDAR-M1-solid-state-datasheet.pdf
49. Product Advantages Al Perception*, accessed July 23, 2025, https://www.generationrobots.com/media/robosense/RS-LiDAR-M1-Brochure-EN.pdf
50. RoboSense RS-LiDAR-M1 Plus - RobotShop, accessed July 23, 2025, https://cdn.robotshop.com/media/U/UTR/RB-Utr-08/pdf/robosense-rs-lidar-m1-plus-advanced-2d-mems-smart-chip-lidar-200m-range-brochure.pdf
51. RS-LiDAR M1 (M1) - RoboSense | Safer world, Smarter life, accessed July 23, 2025, https://www.robosense.ai/en/RS-LiDAR-M1
52. RoboSense Unveils The Smart "GAZE" Function Of Its RS-LiDAR-M1 Zoom Functionality Introduced in the LiDAR - Business Wire, accessed July 23, 2025, https://www.businesswire.com/news/home/20210629005665/en/RoboSense-Unveils-The-Smart-GAZE-Function-Of-Its-RS-LiDAR-M1-Zoom-Functionality-Introduced-in-the-LiDAR
53. RoboSense Unveils The Smart "GAZE" Function Of Its RS-LiDAR-M1 Zoom Functionality Introduced in the LiDAR | Financial Post, accessed July 23, 2025, https://financialpost.com/pmn/press-releases-pmn/business-wire-news-releases-pmn/robosense-unveils-the-smart-gaze-function-of-its-rs-lidar-m1-zoom-functionality-introduced-in-the-lidar
54. RoboSense launches LiDAR System with "Gaze" function similar to human eyes, accessed July 23, 2025, https://geospatialworld.net/news/robosense-launches-lidar-system-with-gaze-function/
55. RoboSense Ranked No. 1 in Global Passenger Car LiDAR Market Share, Annual and Cumulative Sales in 2024 | Yole Annual Report - PR Newswire, accessed July 23, 2025, https://www.prnewswire.com/news-releases/robosense-ranked-no-1-in-global-passenger-car-lidar-market-share-annual-and-cumulative-sales-in-2024--yole-annual-report-302426396.html
56. RoboSense Ranked No. 1 in Global Passenger Car LiDAR Market Share, Annual and Cumulative Sales in 2024 | Yole Annual Report, accessed July 23, 2025, https://www.robosense.ai/en/news-show-1894
57. BriefCASE: Chinese suppliers gain strong foothold in global lidar market, accessed July 23, 2025, https://www.spglobal.com/mobility/en/research-analysis/briefcase-chinese-suppliers-gain-strong-foothold-lidar-market.html
58. Lidar Industry Enters Mass Adoption Phase, Hesai Remains Leader In Global Market Share, accessed July 23, 2025, https://lidarmag.com/2025/04/17/lidar-industry-enters-mass-adoption-phase-hesai-remains-leader-in-global-market-share/
59. Hesai Tops the Global Automotive Lidar Ranking for the Third Consecutive Year, accessed July 23, 2025, https://investor.hesaitech.com/news-releases/news-release-details/hesai-tops-global-automotive-lidar-ranking-third-consecutive/
60. Hesai tops the global automotive LiDAR ranking for the third consecutive year - Yole Group, accessed July 23, 2025, https://www.yolegroup.com/industry-news/hesai-tops-the-global-automotive-lidar-ranking-for-the-third-consecutive-year/
61. Hesai Group (HSAI US), accessed July 23, 2025, https://www.cmbi.com.hk/upload/202412/20241226711948.pdf
62. LiDAR Comparison: 905nm vs. 1550nm - LumiMetriC, accessed July 23, 2025, https://www.lumimetric.com/en/new/905nm-and-1550nm-LiDAR-Laser-Comparison.html
63. Differences Between 905nm and 1550nm Laser Wavelengths in Distance Measurement, accessed July 23, 2025, https://www.meskernel.com/news/differences-between-905nm-and-1550nm-laser-wavelengths-in-distance-measurement-262321.html
64. 905 and 1550 nm Automotive Lidar Lasers Market's Consumer Landscape: Insights and Trends 2025-2033, accessed July 23, 2025, https://www.datainsightsmarket.com/reports/905-and-1550-nm-automotive-lidar-lasers-167788
65. Things you need to know about LiDAR: solid-state and hybrid solid-state, what's the difference? | Hesai, accessed July 23, 2025, https://www.hesaitech.com/things-you-need-to-know-about-lidar-solid-state-and-hybrid-solid-state-whats-the-difference/
66. All-Solid-State Beam Steering via Integrated Optical Phased Array Technology - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-666X/13/6/894
67. OPA Solid State LiDAR 2025-2033 Analysis: Trends, Competitor Dynamics, and Growth Opportunities, accessed July 23, 2025, https://www.archivemarketresearch.com/reports/opa-solid-state-lidar-831720
68. Automotive Flash Blind Spot LiDAR Analysis 2025 and Forecasts 2033: Unveiling Growth Opportunities - Data Insights Market, accessed July 23, 2025, https://www.datainsightsmarket.com/reports/automotive-flash-blind-spot-lidar-142275
69. Automotive LiDAR 2025 - Yole Group - Follow the latest trend news in the Semiconductor Industry, accessed July 23, 2025, https://www.yolegroup.com/product/report/automotive-lidar-2025/
70. LiDAR for automotive: China is propelling the market forward - Yole Group, accessed July 23, 2025, https://www.yolegroup.com/press-release/lidar-for-automotive-china-is-propelling-the-market-forward/


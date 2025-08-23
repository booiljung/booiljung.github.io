# Livox AVIA에 대한 기술적 고찰


본 보고서는 Livox AVIA LiDAR 센서에 대한 심층적인 기술 및 시장 분석을 제공한다. Livox AVIA는 솔리드 스테이트(Solid-State) 설계, 파괴적인 가격 대비 성능, 그리고 독창적인 스캐닝 기술을 특징으로 하는 LiDAR 시장의 핵심 제품으로 평가된다. AVIA는 단순한 점진적 개선을 넘어, DJI와 Livox의 시너지 효과를 통해 탄생한 전략적 제품이다. 이는 UAV(무인 항공기) 매핑 및 저속 로보틱스와 같은 대중 시장 애플리케이션을 위한 LiDAR 기술의 민주화를 목표로 한다. 본 보고서는 경량 설계, 듀얼 모드 스캐닝, DJI 생태계와의 깊은 통합과 같은 AVIA의 핵심 강점을 조명하는 동시에, 절대 정확도의 한계와 새로운 스캔 패턴이 전통적인 SLAM(동시적 위치추정 및 지도작성) 알고리즘에 제기하는 도전 과제 등 약점 또한 분석한다. AVIA는 LiDAR 기술의 접근성을 획기적으로 높여 새로운 시장을 창출한 제품으로, 그 기술적 특성과 시장에서의 위치를 종합적으로 고찰하는 것이 본 보고서의 목적이다.


이 섹션에서는 AVIA의 하드웨어와 그 기반이 되는 기술 원리를 심층적으로 분석한다. 단순한 기능 나열을 넘어, 설계 선택의 배경과 이유를 설명하여 센서의 본질을 파악하고자 한다.


AVIA의 기술 데이터시트는 UAV 및 로보틱스 애플리케이션에서의 역할을 규정하는 핵심 요소들로 구성된다.

- **핵심 성능 지표:** AVIA는 파장 $905 \text{ nm}$의 레이저를 사용하며, 눈에 안전한 Class 1 등급(IEC60825-1:2014)을 획득했다.1 또한, IP67 등급의 방진/방수 성능을 갖추어 다양한 야외 환경에서 작동이 가능하다.1 센서의 가장 큰 특징 중 하나는 $91 \times 61.2 \times 64.8 \text{ mm}$의 소형 크기와 $498 \text{ g}$에 불과한 가벼운 무게로, 이는 드론의 비행 시간을 연장하고 탑재 가능한 드론의 선택 폭을 넓히는 결정적인 장점으로 작용한다.1
- **탐지 거리의 미묘함:** AVIA의 탐지 거리는 단일 값으로 표현되지 않으며, 주변광의 세기(0 klx vs. 100 klx)와 목표물의 반사율(10%, 20%, 80%)에 따라 달라진다. 최대 탐지 거리인 450 m는 구름 낀 날이나 야간과 같은 저조도(0 klx) 환경에서 반사율 80%인 목표물을 대상으로 할 때만 달성 가능하다.1 반면, 맑은 대낮(100 klx)에 반사율 10%의 일반적인 목표물을 탐지할 경우, 유효 거리는 190 m로 감소한다.1 이러한 조건별 성능 차이는 실제 임무 계획 수립 시 반드시 고려해야 할 중요한 변수다.
- **정확도 및 정밀도:** 센서의 거리 정밀도(Range Precision)는 20 m 거리에서 1σ 기준으로 $\pm 2 \text{ cm}$로 명시되어 있으며, 각도 정밀도(Angular Precision)는 0.05∘ 미만이다.1 이 수치는 25℃ 환경에서 반사율 80%인 목표물을 20 m 거리에서 측정했을 때의 결과이므로, 실제 필드 환경에서는 달라질 수 있다는 점을 인지해야 한다.1
- **데이터 출력:** 포인트율(Point Rate)은 단일 리턴 모드에서 초당 240,000 포인트이며, 듀얼 리턴 모드에서는 480,000 포인트/초, 트리플 리턴 모드에서는 최대 720,000 포인트/초까지 확장된다.1 데이터는 100 Mbps 이더넷을 통해 전송되며, 지연 시간은 2 ms 이하로 매우 짧다.1


AVIA의 파괴적인 시장 경쟁력은 Livox의 기술 혁신과 DJI의 제조 역량이라는 두 축의 시너지에서 비롯된다. Livox는 2016년 DJI의 개방형 혁신 프로그램(Open Innovation Program)을 통해 설립된 독립 회사로, DJI의 방대한 제조 전문성, 공급망 관리 능력, 그리고 재정적 자원을 활용할 수 있는 독보적인 위치에 있다.6

이러한 시너지의 핵심에는 'DL-Pack' 기술이 있다.2 이는 Livox가 특허를 보유한 다중 레이저 송수신기 통합 및 패키징 기술로, 고도로 모듈화되고 자동화된 생산 라인 설계의 기반이 된다. DL-Pack 기술은 LiDAR 센서의 교정(calibration) 과정을 자동화하고, 숙련된 기술 인력의 필요성을 줄여준다. 이는 전통적인 LiDAR 제조 방식과 비교할 때 생산 수율을 높이고 단위 비용을 극적으로 낮추는 결과로 이어진다.8 결과적으로 Livox는 경쟁사 가격의 일부에 불과한 비용으로 고품질 센서를 대량 생산할 수 있는 능력을 갖추게 되었으며, 이는 AVIA의 가격 경쟁력의 근간을 이룬다.8 이처럼 Livox의 경쟁 우위는 단순히 특정 성능 지표가 아닌, 방어 가능한 제조 기반의 비용 우위에서 나온다고 분석할 수 있다.


AVIA에는 보쉬(Bosch)의 BMI088 관성측정장치(IMU)가 내장되어 있다.1 이 IMU는 소비자 또는 산업용 등급의 부품으로, 고가의 전술 등급(tactical-grade)이나 항법 등급(navigation-grade) 장치와는 구별된다. 내장 IMU의 주된 목적은 센서가 빠르게 움직일 때 발생하는 포인트 클라우드의 왜곡을 보정하고, 공간 자세 및 가속도 정보를 지속적으로 출력하여 '중복 데이터'를 제공하는 것이다.2

하지만 이 IMU는 고정밀 측량에 필요한 고성능 관성항법장치(INS)를 대체할 수 없다. 센티미터 수준의 정확도를 요구하는 매핑 작업을 위해서는 반드시 외부의 고등급 GNSS-INS 장치와 결합해야 한다.10 내장 IMU 데이터만을 이용한 추측항법(dead reckoning)은 시간에 따라 오차가 빠르게 누적되어 위치 추정의 신뢰도가 급격히 저하된다.12

이러한 설계는 Livox의 전략적 트레이드오프를 명확히 보여준다. 저렴한 IMU를 내장함으로써 로보틱스와 같은 기본 모션 보정이 필요한 응용 분야에서는 추가 비용 없이 가치를 제공하는 동시에, 어차피 자체적인 고성능 INS를 사용할 측량 시장의 사용자에게는 불필요한 가격 상승을 유발하지 않는다. 이처럼 의도된 타협을 통해 AVIA는 더 넓은 잠재 고객층을 확보할 수 있었다.


AVIA는 하나의 레이저 펄스당 최대 3개의 리턴 신호를 수신할 수 있다.2 이는 복잡하고 다층적인 환경을 스캔할 때 매우 중요한 기능이다. 예를 들어, 산림 지역을 스캔할 경우 첫 번째 리턴은 나뭇잎이 무성한 수관(canopy)의 최상단에서 반사되고, 두 번째와 세 번째 리턴은 중간층의 잎사귀를 투과하여 최종적으로 지표면에 도달할 수 있다. 이 덕분에 무성한 식생 아래의 실제 지형을 모델링하는 디지털 지형 모델(DTM) 생성에 탁월한 성능을 발휘하며, 특히 산림 측량 분야에서 그 가치가 극대화된다.2

------

**표 1: Livox AVIA 상세 제원**

| **분류**      | **항목**               | **사양**                                                     | **출처** |
| ------------- | ---------------------- | ------------------------------------------------------------ | -------- |
| **광학**      | 레이저 파장            | 905 nm                                                       | 1        |
|               | 레이저 안전 등급       | Class 1 (IEC60825-1:2014)                                    | 1        |
|               | 탐지 거리 (80% 반사율) | 320 m (@ 100 klx), 450 m (@ 0 klx)                           | 1        |
|               | 탐지 거리 (10% 반사율) | 190 m (@ 100 klx), 190 m (@ 0 klx)                           | 1        |
|               | 거리 정밀도            | ±2 cm (1σ @ 20m)¹                                            | 1        |
|               | 각도 정밀도            | <0.05∘ (1σ)                                                  | 1        |
|               | FOV (비반복 스캔)      | 70.4∘ (수평) × 77.2∘ (수직)                                  | 1        |
|               | FOV (반복 스캔)        | 70.4∘ (수평) × 4.5∘ (수직)                                   | 1        |
|               | 빔 발산각              | 0.03∘ (수평) × 0.28∘ (수직)                                  | 1        |
|               | 최대 리턴 수           | 3                                                            | 1        |
| **데이터**    | 포인트율               | 240,000 pts/s (단일), 480,000 pts/s (듀얼), 720,000 pts/s (트리플) | 1        |
|               | 데이터 인터페이스      | 100 Mbps Ethernet                                            | 1        |
|               | 데이터 지연 시간       | ≤2 ms                                                        | 1        |
| **기계/전기** | 내장 IMU               | Bosch BMI088                                                 | 1        |
|               | 전력 소비              | 8W (비반복), 9W (반복) / 16W (시동)                          | 1        |
|               | 작동 전압              | 10 ~ 15 V DC                                                 | 1        |
|               | 방진/방수 등급         | IP67                                                         | 1        |
|               | 무게                   | 498 g (케이블 제외)                                          | 1        |
|               | 크기                   | 91×61.2×64.8 mm                                              | 1        |

¹ 25℃ 환경에서 80% 반사율을 가진 목표물을 20m 거리에서 측정한 값.

------


이 섹션에서는 AVIA의 가장 독특한 특징인 듀얼 스캐닝 모드의 원리, 성능, 그리고 장단점을 심층적으로 분석한다. 비반복 패턴과 반복 패턴의 기술적 차이를 이해하는 것은 센서의 잠재력을 최대한 활용하기 위한 필수 과정이다.


- **원리:** 이 모드는 전통적인 기계식 LiDAR가 매번 동일한 라인을 반복적으로 스캔하는 것과 근본적으로 다르다. 대신, '스파이로그래프(spirograph)' 또는 '로제트(rosette)' 패턴으로 묘사되는 독특한 비반복적, 준-무작위 스캐닝 패턴을 사용한다.13 이 방식은 리즐리 프리즘(Risley prisms)을 이용해 구현되는 것으로 추정되며, 시간이 지남에 따라 시야각(FOV) 내의 빈 공간을 지속적으로 채워나간다.4
- **FOV 및 포인트 밀도:** 비반복 스캔 모드는 수평 70.4∘, 수직 77.2∘의 매우 넓은 원형에 가까운 FOV를 제공한다.1 포인트 밀도는 균일하지 않으며, FOV의 중심부에서 가장 높고 주변부로 갈수록 점차 감소하는 특징을 보인다.5
- **커버리지 진화:** 이 모드의 핵심은 시간 누적에 따른 커버리지의 극적인 향상이다. 짧은 시간 동안에는 데이터가 성기게 보일 수 있지만, 통합 시간이 길어질수록 FOV 커버리지는 100%에 가깝게 수렴한다.
  - 0.1초 이내에 FOV 중심부(반경 10° 이내)의 포인트 밀도는 전통적인 32라인 LiDAR 센서에 필적한다.5
  - 0.2초가 지나면 FOV 전체 영역의 밀도가 32라인 LiDAR 수준에 도달한다.5
  - 0.8초 후에는 FOV 커버리지가 거의 100%에 근접하여, 장면의 거의 모든 부분이 레이저 빔에 의해 조사된다.5 이는 정적인 환경을 매핑하거나 저속으로 이동하는 모바일 애플리케이션에서 사진과 유사한 고밀도 포인트 클라우드를 생성하는 데 이상적이다.2


- **원리:** 이 모드에서는 수직 방향의 넓은 커버리지를 포기하는 대신, 스캔의 일관성과 밀도를 극대화한다. 센서는 매우 집중된 수평 라인 스캔을 반복적으로 생성한다.13
- **FOV 및 포인트 밀도:** 수평 FOV는 비반복 모드와 동일한 70.4∘를 유지하지만, 수직 FOV는 4.5∘라는 매우 좁은 폭으로 극적으로 줄어든다.1 초당 240,000개의 모든 포인트가 이 좁은 밴드 안에 집중되므로, 스캔 라인을 따라 극도로 높은 포인트 밀도를 얻을 수 있다.
- **이상적인 사용 사례:** 이 모드는 전력선, 철도, 도로와 같은 회랑 매핑, 농경지, 건설 현장 검사 등 높은 정밀도와 균일한 포인트 분포가 요구되는 측량 시나리오를 위해 명시적으로 설계되었다.2

이러한 듀얼 모드 기능은 AVIA의 활용성을 크게 높이는 전략적 선택이다. 비반복 스캔 패턴은 그 자체로 혁신적이지만, 불규칙성이나 기존 워크플로우와의 호환성 문제 등 단점을 내포한다. 특히 정밀 측량 분야에서는 예측 가능하고 일관된 밀도의 데이터가 무엇보다 중요하다. 반복 라인 스캔 모드는 바로 이 지점에서 전통적인 센서의 동작을 모방하여, 확립된 처리 워크플로우를 가진 사용자들에게 친숙하고 신뢰할 수 있는 데이터 구조를 제공한다. 따라서 듀얼 모드는 혁신적인 기술을 탐구하려는 R&D 사용자와 안정적인 결과물을 필요로 하는 상업적 측량 사용자 모두를 만족시키기 위한 영리한 헤지(hedge) 전략이라고 볼 수 있다.


| **특징**                 | **비반복 스캔 모드 (Non-Repetitive Mode)**                   | **반복 스캔 모드 (Repetitive Mode)**                  |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| **원리**                 | 스파이로그래프(Spirograph) 형태의 패턴으로 시간이 지남에 따라 빈 공간을 채움 | 집중된 수평 라인을 반복적으로 스캔                    |
| **FOV (수평 x 수직)**    | 70.4∘×77.2∘                                                  | 70.4∘×4.5∘                                            |
| **포인트 밀도 분포**     | 중앙부에서 매우 높고, 주변부로 갈수록 감소                   | 좁은 수평 라인 내에서 매우 높고 균일함                |
| **시간에 따른 커버리지** | 통합 시간이 길어질수록 커버리지가 100%에 근접                | 매 스캔마다 동일한 영역을 커버 (변화 없음)            |
| **핵심 장점**            | 최대 영역 커버리지, 정적 환경에서 고밀도 3D 모델 생성        | 최대 선형 밀도, 예측 가능성, 균일한 데이터 분포       |
| **핵심 단점**            | 짧은 시간 내에는 데이터가 성기고 불규칙함, 동적 환경에서 불리 | 수직 커버리지가 극히 제한적임                         |
| **추천 애플리케이션**    | 스마트 시티, 저속 자율주행, 정적 3D 매핑, 산림 측량          | 전력선/철도/도로 등 회랑 매핑, 농경지, 건설 현장 검사 |


이 섹션에서는 AVIA가 목표로 하는 주요 응용 분야에서의 실제 성능을 평가한다. 기술 제원과 사용자 피드백, 그리고 경쟁 제품과의 비교 분석을 통합하여 각 분야별 적합성을 심도 있게 고찰한다.


- **강점:** 이 분야에서 AVIA의 가장 큰 강점은 498g의 가벼운 무게와 소형 크기다. 이는 드론의 비행 시간을 늘리고, DJI Matrice 시리즈와 같이 긴밀하게 통합된 플랫폼을 포함한 더 넓은 범위의 드론에 탑재될 수 있게 한다.3 특히 DJI와 Livox의 관계는 복잡한 커스텀 통합 과정 없이 거의 플러그 앤 플레이(plug-and-play)에 가까운 사용자 경험을 제공하는데, 이는 경쟁사 대비 상당한 이점이다.7
- **약점:** 명시된 $\pm 2\text{cm}$의 정확도는 논쟁의 여지가 있다. 사용자 포럼과 비교 리뷰에 따르면, 측량 등급의 정확도(예: 3cm 미만)를 일관되게 달성하는 것은 도전적인 과제일 수 있다.20 한 사용자는 유사 등급의 센서로 단단한 표면에서 0.4-0.5 피트(약 12-15cm)의 노이즈를 보고하기도 했다.20 따라서 AVIA는 최고의 정확도보다는 비용과 운영 효율성을 우선시하는 애플리케이션에 더 적합하다.
- **통합:** RESEPI와 같은 페이로드 시스템은 AVIA를 고성능 INS 및 GNSS 수신기와 결합하여 완전한 매핑 솔루션을 구성한다.10 이는 본격적인 측량 작업을 위해서는 외부 고성능 부품과의 통합이 필수적임을 보여준다.


- **강점:** 이 분야는 AVIA가 가장 두각을 나타내는 활용 사례 중 하나다. 반복 라인 스캔 모드는 레이저 포인트를 목표물(전선, 철로 등)에 집중시켜 매우 높은 밀도의 데이터를 얻는 데 이상적이다.2 또한, 센서의 특별한 빔 스팟 형태는 전선과 같은 가느다란 객체를 탐지하는 데 최적화되어 있다.2 긴 탐지 거리는 더 높은 고도에서 안전하게 비행하며 효율적으로 데이터를 수집할 수 있게 해준다.2
- **비교 관점:** AVIA가 매우 유능하지만, 전선 주변의 식생 투과가 중요한 유틸리티 작업의 경우, 5개의 리턴을 지원하는 Zenmuse L2(동일한 Livox 기반 센서 사용)를 선호하는 사용자도 있다.21


- **강점:** 트리플 리턴 기능이 이 분야의 핵심 동력이다. 이 기능 덕분에 센서는 수관 상단, 중간층 식생, 그리고 지표면의 데이터를 동시에 수집할 수 있다.2 이는 식생 아래의 정확한 디지털 지형 모델(DTM)을 생성하는 데 매우 중요하다.
- **비교 관점:** Hesai XT-32와의 직접적인 비교에서, AVIA가 트리플 리턴을 지원하지만, XT-32의 360° FOV와 더 높은 전체 포인트율은 충분한 중첩도를 가진 촘촘한 비행 계획 하에서 식생 투과 성능이 더 우수할 수 있다.18 어떤 센서가 더 나은지는 비행 파라미터와 구체적인 목표에 따라 달라진다.


- **도전 과제:** 비반복 스캔 패턴은 전통적인 SLAM 알고리즘에 상당한 어려움을 제기한다.
  - **특징점 추출:** 안정적이고 뚜렷한 스캔 라인에서 특징점을 추출하는 데 의존하는 방법들은 AVIA의 불규칙한 꽃 모양 패턴에 적용하기 어렵다.14
  - **스캔 정합:** 매 프레임마다 포인트 분포가 바뀌기 때문에 연속적인 스캔을 정합하는 것이 까다롭다. 이는 긴 복도와 같이 특징이 부족한 환경에서 위치 추정의 성능 저하(degeneration)를 유발할 수 있다.14
  - **루프 폐쇄:** 포인트 클라우드의 일관된 2D 투영에 의존하는 Scan Context와 같은 장소 인식 방법은 불규칙한 패턴으로 인해 성능이 저하될 수 있다.16
- **해결책:** 연구 커뮤니티는 이러한 문제들을 해결하기 위해 새로운 알고리즘을 활발히 개발하고 있다. FAST-LIO2나 FASTER-LIO와 같은 방법들은 서펠(surfel) 기반 특징점과 긴밀하게 결합된 관성 융합(tightly-coupled inertial fusion)을 사용하여 AVIA의 데이터 구조에 더 강건한 성능을 보인다.22 스캔 패턴 자체를 명시적으로 모델링하여 정합을 돕는 연구도 진행 중이다.14

AVIA의 시장 성공은 '충분히 좋은(good-enough)' 기술의 민주화가 가져온 직접적인 결과다. 이 센서는 20만 달러가 넘는 Riegl 센서의 정확도를 따라잡지는 못하지만 20, 건설, 소규모 유틸리티, 농업 분야의 수많은 신규 사용자들이 LiDAR 데이터에 처음으로 접근할 수 있게 함으로써 시장 자체를 근본적으로 확장시켰다. 이전에는 가격 장벽 때문에 LiDAR 도입을 고려조차 할 수 없었던 새로운 시장 부문을 창출한 것이 AVIA의 진정한 영향력이다.

또한, AVIA의 적합성은 항공 애플리케이션과 지상 애플리케이션 간에 뚜렷한 차이를 보인다. 가벼운 무게와 DJI 통합이라는 강점은 항공 분야에서 극대화되는 반면, 가장 큰 약점인 까다로운 스캔 패턴은 고속 지상 SLAM에서 가장 두드러지게 나타난다. 따라서 잠재 구매자는 자신의 주된 사용 분야가 항공인지 지상인지를 가장 중요한 평가 기준으로 삼아야 한다. AVIA는 가격 대비 매우 훌륭한 *항공 매핑* 센서이지만, 더 도전적인 *지상 로보틱스* 센서라고 할 수 있다.


이 섹션에서는 전통적인 기계식 회전형 LiDAR 패러다임을 대표하는 두 경쟁 제품, 벨로다인 퍽(Velodyne Puck, VLP-16) 및 아우스터 OS0(Ouster OS0)와의 직접 비교를 통해 AVIA를 시장 맥락 속에 위치시킨다.


- **Livox AVIA (솔리드 스테이트):** 리즐리 프리즘으로 추정되는 비기계식 빔 조향 방식을 사용하여 움직이는 부품을 최소화했다. 이 설계는 본질적으로 더 견고하고, 소형이며, 저비용 대량 생산에 유리하다.22 AVIA의 핵심 혁신은 스캐닝 패턴과 제조 공정에 있다.
- **Velodyne Puck (기계식 회전형):** 회전형 LiDAR의 원형과 같은 제품이다. 레이저 어셈블리 전체를 회전시켜 360° 수평 FOV를 달성한다. 이 방식은 기술적으로 성숙하고 잘 알려져 있으며, 고정된 수평 라인으로 구성된 매우 예측 가능하고 구조화된 포인트 클라우드를 생성한다.24
- **Ouster OS0 (디지털 회전형):** 기계식 접근법의 진화된 형태다. 이 센서 역시 회전하지만, 단일 레이저와 맞춤형 SPAD(Single-Photon Avalanche Diode) 검출기 배열을 사용하는 독점적인 '디지털 LiDAR' 아키텍처를 채택했다. 이를 통해 포인트 클라우드와 함께 카메라급의 근적외선 이미지를 생성할 수 있으며, 매우 넓은 수직 FOV를 제공한다.27


- **FOV:** AVIA의 비반복 모드 FOV(70.4∘×77.2∘)는 넓지만 지향성이다. 반면 Puck(360∘×30∘)과 OS0(360∘×90∘)는 완전한 서라운드 뷰를 제공하여 지상 로봇의 장애물 회피에 핵심적인 이점을 가진다.1
- **정확도:** AVIA는 ±2cm, Puck은 ±3cm, Ouster OS0는 $\pm 3\text{cm}$에서 $\pm 5\text{cm}$의 정확도를 주장한다. 그러나 이는 서로 다른 조건에서 측정된 값이므로 신중하게 해석해야 하며, 실제 환경에서의 성능은 크게 달라질 수 있다.1
- **무게 및 전력:** AVIA(498g, ~8-9W)는 표준 Puck(830g, ~8W)보다 훨씬 가볍고, Puck LITE(590g)와 비슷하며, Ouster OS0(445g)보다는 약간 무겁다.1 세 제품 모두 UAV에 탑재 가능하지만, AVIA의 가벼운 무게는 여전히 경쟁 우위로 작용한다.
- **리턴 수:** AVIA의 트리플 리턴 기능은 Puck과 Ouster OS0(Rev7)의 듀얼 리턴 기능에 비해 식생 투과 측면에서 뚜렷한 장점을 제공한다.2
- **가격:** 가격은 AVIA의 가장 강력한 무기다. 센서 단품 가격은 약 1,700달러에서 2,500달러 사이에서 형성된다.30 Velodyne Puck VLP-16은 2018년에 약 4,000달러로 대폭 가격을 인하했음에도 불구하고 33, 신품은 여전히 수천 달러 이상에 판매되며 중고품은 eBay에서 300달러에서 3,000달러 사이에 거래된다.34 Ouster OS0 역시 수천 달러/유로에 달하는 더 높은 가격대에 위치한다.36

이러한 경쟁 구도는 LiDAR 시장이 세분화되고 있음을 보여준다. AVIA는 기존의 고가 센서를 직접 대체하기보다는, 기존 업체들이 비용 구조와 비즈니스 모델 때문에 경쟁하기 어려운 새로운 저가 시장을 창출하고 지배하고 있다. Livox는 DJI의 대량 생산 능력에 기반한 근본적으로 다른 비용 구조를 통해 가격 파괴를 주도하고 있으며, 이는 기존 업체들에게 저가 시장을 포기하거나 자신들의 비즈니스 모델을 근본적으로 바꾸도록 압박하는 비대칭적 경쟁을 유발하고 있다.


| **항목**                        | **Livox AVIA**                | **Velodyne Puck (VLP-16)**  | **Ouster OS0 (Rev7)**       |
| ------------------------------- | ----------------------------- | --------------------------- | --------------------------- |
| **기술**                        | 솔리드 스테이트 (비기계식)    | 기계식 회전형               | 디지털 회전형               |
| **스캔 패턴**                   | 비반복 로제트 / 반복 라인     | 반복 수평 라인              | 반복 수평 라인              |
| **수평 FOV**                    | 70.4∘                         | 360∘                        | 360∘                        |
| **수직 FOV**                    | 77.2∘ (비반복) / 4.5∘ (반복)  | 30∘                         | 90∘                         |
| **최대 탐지 거리 (80% 반사율)** | 450 m                         | 100 m                       | 75 m                        |
| **탐지 거리 (10% 반사율)**      | 190 m                         | 제공되지 않음               | 35 m                        |
| **정확도**                      | ±2 cm                         | ±3 cm                       | ±3−5 cm                     |
| **포인트율**                    | 최대 720k pts/s (트리플 리턴) | 최대 600k pts/s (듀얼 리턴) | 최대 5.2M pts/s (듀얼 리턴) |
| **리턴 수**                     | 3                             | 2                           | 2                           |
| **무게**                        | 498 g                         | 830 g (Lite: 590 g)         | 445 g                       |
| **전력 소비**                   | ~8-9 W                        | ~8 W                        | ~14-20 W                    |
| **내장 IMU**                    | 예 (Bosch BMI088)             | 아니요                      | 예 (InvenSense)             |
| **대략적 가격 (센서 단품)**     | $1,700 - $2,500               | $4,000+ (신품)              | $4,500+ (신품)              |


이 섹션에서는 AVIA를 실제로 활용하는 데 필요한 소프트웨어 도구, 라이브러리, 그리고 커뮤니티 지원 등 개발 환경을 평가한다.


AVIA와의 통합을 위한 핵심 도구는 Livox SDK이다. 이는 C/C++ 기반의 라이브러리로, 센서와의 통신 및 데이터 파싱을 담당한다.40 SDK는 Linux(Ubuntu 14.04/16.04/18.04, x86/ARM)와 Windows 환경을 모두 지원한다.40

통신은 UDP(User Datagram Protocol)를 통해 이루어지며, SDK는 제어 명령과 포인트 클라우드 데이터 스트림을 처리하는 핸들러를 제공한다.40 개발에서 중요한 개념은 '브로드캐스트 코드(Broadcast Code)'로, 각 LiDAR 장치를 식별하는 15자리의 고유 코드다. 개발자는 네트워크상의 모든 센서에 연결하거나, 특정 브로드캐스트 코드 목록을 지정하여 원하는 장치에만 선택적으로 연결할 수 있다.40


Livox는 `livox_ros_driver`라는 공식 ROS1 드라이버를 제공하며, 이는 매우 안정적으로 유지 관리되고 있다.41 이 드라이버는 ROS Melodic, Kinetic, Indigo 버전을 지원하며, AVIA를 비롯한 Mid-40, Horizon, Tele-15 등 다양한 Livox 모델과 호환된다.41 현재로서는 AVIA를 로보틱스 프로젝트에 통합하는 가장 성숙하고 권장되는 경로다.


ROS2 지원은 AVIA의 결정적인 약점 중 하나다. Livox의 공식 ROS2 드라이버인 `livox_ros_driver2`는 HAP, Mid-360과 같은 최신 센서를 위해 설계되었으며, **AVIA를 지원하지 않는다**.42

따라서 AVIA의 ROS2 지원은 전적으로 서드파티, 즉 커뮤니티가 유지 관리하는 포크(fork)에 의존해야 한다. GitHub의 `ASIG-X/livox_ros2_avia`가 대표적인 예다.44 이러한 포크를 사용하기 위해서는 종종 수동적인 개입이 필요하다. 예를 들어, 사용자는 시스템에 설치된 정적 Livox SDK 라이브러리(`livox_sdk_static.a`)를 삭제해야 ROS2 드라이버가 자체적으로 공유 라이브러리 버전을 빌드할 수 있다.44 이러한 파편화는 ROS2 통합을 위한 단일하고 공식적이며 신뢰할 수 있는 경로가 없음을 의미하며, 이는 상용 시스템 개발에 있어 상당한 위험 요소로 작용할 수 있다.


Livox는 자사 GitHub를 통해 LIO-Livox(SLAM), 객체 탐지, 캘리브레이션 등 다양한 오픈소스 알고리즘 예제를 제공하며 개발자 커뮤니티를 활성화하고 있다.46 또한, 커뮤니티에서 제공하는 Gazebo 시뮬레이션 패키지(ROS1 및 ROS2용)를 사용하면 물리적인 하드웨어 없이도 개발 및 테스트를 진행할 수 있다.46

이러한 개발자 생태계의 현황은 Livox의 전략적 우선순위를 엿볼 수 있게 한다. 안정적인 ROS1 지원은 기존의 성숙한 로보틱스 시장을 반영하는 반면, AVIA에 대한 공식적인 ROS2 지원 부재는 Livox의 핵심 엔지니어링 자원이 HAP과 같은 최신 자동차 등급 센서로 이동했음을 시사한다. 이는 AVIA와 같은 구형(그러나 여전히 인기 있는) 센서의 지원은 커뮤니티에 맡겨두고, 회사는 자동차 시장에 집중하려는 전략으로 해석될 수 있다. 따라서 새로운 ROS2 기반 프로젝트를 위해 센서를 선택하는 개발자에게 이는 중요한 고려사항이 된다.


이 마지막 섹션에서는 전체 분석을 종합하여 AVIA의 시장 내 역할에 대한 미래 지향적 관점을 제시하고, 잠재적 도입자를 위한 실행 가능한 조언을 제공한다.


AVIA의 성공은 DJI 생태계 내에서의 탄생이라는 배경 없이는 이해할 수 없다. Livox의 혁신적인 센서 설계와 DJI의 세계적 수준의 대량 생산 능력이 결합하여 LiDAR 시장의 경제성을 근본적으로 바꾸는 제품이 탄생했다.2 이 시너지는 앞으로도 전통적인 LiDAR 제조업체에 지속적인 압박을 가하는 지속 가능한 경쟁 우위로 작용할 것이다.


AVIA는 LiDAR의 상품화(commoditization)라는 더 큰 흐름의 전조다. 솔리드 스테이트 기술과 대량 생산에 힘입어 비용이 계속 하락함에 따라, LiDAR는 틈새시장의 고가 센서에서 로보틱스, 드론, 심지어 소비자 기기의 표준 부품으로 이동할 것이다.48 AVIA와 그 후속 제품들은 이전에는 경제적으로 불가능했던 농업, 건설, 물류 분야의 수많은 애플리케이션을 가능하게 할 것이다.51


- **UAV 측량 및 매핑 전문가:** AVIA는 특히 RESEPI와 같은 시스템에 통합되거나 DJI Matrice 드론과 함께 사용될 때 뛰어난 가성비를 제공한다. 가벼운 무게와 특화된 스캔 모드가 큰 자산이 되는 전력선 검사, 산림 관리, 건설 현장 모니터링과 같은 애플리케이션에 강력히 추천된다. 그러나 법적으로 방어 가능해야 하는 최고 수준의 정확도를 요구하는 프로젝트의 경우, 여전히 더 높은 등급의 시스템을 고려하고 AVIA의 결과물을 지상기준점(GCP)과 비교하여 검증하는 과정을 거쳐야 한다.
- **지상 로보틱스 및 SLAM 개발자:** AVIA는 양날의 검과 같다. 저렴한 비용과 높은 포인트 밀도는 R&D에 매력적이다. 그러나 개발자들은 비반복 스캔 패턴이 제기하는 상당한 알고리즘적 도전과 파편화된 ROS2 지원 상태를 명확히 인지해야 한다. FAST-LIO와 같은 최첨단 SLAM 알고리즘을 다룰 의향이 있는 강력한 소프트웨어 엔지니어링 역량을 갖춘 팀에게 추천된다. 정해진 기한 내에 강건한 위치 추정 솔루션을 플러그 앤 플레이 방식으로 구현해야 하는 팀은, 더 비싸더라도 전통적인 회전형 LiDAR가 더 간단한 선택이 될 수 있다.
- **교육자 및 취미 개발자:** AVIA의 낮은 가격대와 사용 가능한 SDK/ROS 드라이버는 로보틱스 및 3D 인식 분야의 교육과 실험을 위한 훌륭한 도구다. 이전에는 학술 및 개인 프로젝트에서 접근하기 어려웠던 고품질 LiDAR 데이터에 대한 접근성을 제공한다.


Livox AVIA는 단순한 센서를 넘어 시장을 재편하는 제품이다. 이 센서는 절대적인 성능과 예측 가능성을 일부 희생하는 대신, 경제성과 접근성에서 막대한 이득을 성공적으로 얻어냈다. AVIA의 유산은 그것이 어떤 사양을 만족시켰는지가 아니라, 어떤 새로운 시장과 애플리케이션의 문을 열었는지로 정의될 것이다.


1. Specs - Avia LiDAR sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/avia/specs
2. Avia LiDAR sensor - Livox, accessed July 23, 2025, https://www.livoxtech.com/avia
3. Livox Avia LiDAR LIVOXAVIA | C.R.Kennedy Geospatial Solutions, accessed July 23, 2025, https://survey.crkennedy.com.au/products/livoxavia/livox-avia-lidar
4. The Avia LiDAR Sensor-Livox LiDAR: For Advanced Precision - Mavdrones, accessed July 23, 2025, https://www.mavdrones.com/product/avia-sensor-livox-lidar/
5. LIVOX AVIA - OxTS, accessed July 23, 2025, https://www.oxts.com/wp-content/uploads/2021/08/Livox-AVIA-User-Manual_EN_compressed.pdf
6. 1. Product - Livox wiki 0.1 documentation, accessed July 23, 2025, https://livox-wiki-en.readthedocs.io/en/latest/introduction/production.html
7. DJI Drone LiDAR: Introducing Livox and Avia - DRONELIFE, accessed July 23, 2025, https://dronelife.com/2020/08/25/dji-drone-lidar-introducing-livox-and-avia/
8. DJI Showcases Wide Product Portfolio At CES 2020 Including New Incubated Livox Lidar Technology, accessed July 23, 2025, https://www.dji.com/newsroom/news/dji-showcases-wide-product-portfolio-at-ces-2020
9. DJI-funded Livox marks leap for self-driving cars with two new, low-cost Lidar sensors, accessed July 23, 2025, https://www.thedronegirl.com/2020/01/06/dji-livox-lidar/
10. RESEPI Livox Avia Drone LiDAR Payload - Canal Geomatics, accessed July 23, 2025, https://canalgeomatics.com/product/resepi-livox-avia-drone-lidar-payload/
11. Livox Showcase - UAV 3D Mapping, accessed July 23, 2025, https://www.livoxtech.com/showcase/1
12. new livox LIDAR AVIA / Issue #15 / ryan-brazeal-ufl/OpenPyLivox - GitHub, accessed July 23, 2025, https://github.com/ryan-brazeal-ufl/OpenPyLivox/issues/15
13. Repetitive vs non-repetitive setting - ROCK R360 R2A - ROCK robotic, accessed July 23, 2025, https://community.rockrobotic.com/t/repetitive-vs-non-repetitive-setting/606
14. (PDF) Non-Repetitive Scanning LiDAR Sensor for Robust 3D Point ..., accessed July 23, 2025, https://www.researchgate.net/publication/377253234_Non-Repetitive_Scanning_LiDAR_Sensor_for_Robust_3D_Point_Cloud_Registration_in_Localization_and_Mapping_Applications
15. Detection and Tracking of MAVs Using a Rosette Scanning Pattern LiDAR - arXiv, accessed July 23, 2025, [https://arxiv.org/pdf/2408.08555?](https://arxiv.org/pdf/2408.08555)
16. A Robust Framework for Simultaneous Localization and Mapping with Multiple Non-Repetitive Scanning Lidars - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/13/10/2015
17. Technical specifications of the LiVOX Avia and Faro Focus 3D. - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/Technical-specifications-of-the-LiVOX-Avia-and-Faro-Focus-3D_tbl1_369523267
18. Drone Surveying Mapping: Comparing RESEPI XT-32 vs Livox Avia LiDAR - Inertial Labs, accessed July 23, 2025, https://inertiallabs.com/comparing-resepi-xt-32-vs-livox-avia-lidar-for-drone-surveying-mapping/
19. AirWorks 2020 Recap: Innovation, Resilience, and a Future Beyond Fear - Insights - DJI, accessed July 23, 2025, https://enterprise-insights.dji.com/blog/dji-airworks-2020-keynote-recap
20. Looking for LiDAR Options : r/UAVmapping - Reddit, accessed July 23, 2025, https://www.reddit.com/r/UAVmapping/comments/kbebvy/looking_for_lidar_options/
21. Thoughts on LiDAR sensors - Zenmuse L2 : r/UAVmapping - Reddit, accessed July 23, 2025, https://www.reddit.com/r/UAVmapping/comments/1cwjq18/thoughts_on_lidar_sensors_zenmuse_l2/
22. Lidar Variability: A Novel Dataset and Comparative Study of Solid-State - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2507.04321
23. A Novel Dataset and Comparative Study of Solid-State and Spinning Lidars - arXiv, accessed July 23, 2025, https://arxiv.org/html/2507.04321v1
24. Velodyne Puck - AutonomouStuff, accessed July 23, 2025, https://autonomoustuff.com/products/velodyne-puck-vlp-16
25. 63-9229_Rev-H_Puck _Datasheet_Web, accessed July 23, 2025, https://www.mapix.com/wp-content/uploads/2018/07/63-9229_Rev-H_Puck-_Datasheet_Web-1.pdf
26. 63-9229_Rev-J_Puck _Datasheet_Web - NET, accessed July 23, 2025, https://hexagondownloads.blob.core.windows.net/public/AutonomouStuff/wp-content/uploads/2019/05/Puck__Datasheet_whitelabel.pdf
27. OS0 Ultra-Wide View High-Resolution Imaging Lidar - Ouster, accessed July 23, 2025, https://data.ouster.io/downloads/datasheets/datasheet-rev05-v2p1-os0.pdf
28. Ouster OS0: High-Precision Ultra-Wide Short-Range Lidar Sensor, accessed July 23, 2025, https://ouster.com/products/hardware/os0-lidar-sensor
29. OS0 Ultra-Wide View High-Resolution Imaging Lidar - Ouster, accessed July 23, 2025, https://data.ouster.io/downloads/datasheets/datasheet-rev7-v3p0-os0.pdf
30. Buy Livox Avia - DJI Store, accessed July 23, 2025, https://store.dji.com/product/livox-avia
31. Livox Avia Laser Scanner - DJI Matrice 300/600 Drone Sensor - Alibaba.com, accessed July 23, 2025, https://www.alibaba.com/product-detail/New-original-Livox-Avia-Laser-scanner_1600862284637.html
32. livox AVIA scanner LIDAR and post-processing software available for sale 3D mapping system - AliExpress, accessed July 23, 2025, https://www.aliexpress.com/item/1005006655549082.html
33. Velodyne cuts VLP-16 lidar price to $4k - Geo Week News, accessed July 23, 2025, https://www.geoweeknews.com/news/velodyne-cuts-vlp-16-lidar-price-4k
34. Velodyne Puck - eBay, accessed July 23, 2025, https://www.ebay.com/shop/velodyne-puck?_nkw=velodyne+puck
35. Velodyne LiDAR Puck Kit VLP-16 - PNW Scientific, accessed July 23, 2025, https://pnwscientific.com/products/velodyne-lidar-puck-kit-vlp-16
36. Ouster 3D LiDAR OS0-128-U 128 Channel Laser Scanner | eBay, accessed July 23, 2025, https://www.ebay.com/itm/335349559105
37. Ouster Ultra-Wide View LiDar Sensor OS0-32 - PNW Scientific, accessed July 23, 2025, https://pnwscientific.com/products/ouster-lidar-sensor-oso-32
38. Ouster Ultra-Wide FOV OS0 Lidar (Rev 7) - Génération Robots, accessed July 23, 2025, https://www.generationrobots.com/en/404054-ouster-ultra-wide-fov-os0-lidar-rev-7.html
39. Ouster OS0 128 Rev 7 Ultra Wide Field of View Lidar Sensor - RobotShop Europe, accessed July 23, 2025, https://eu.robotshop.com/products/ouster-os0-128-rev-6-ultra-wide-field-of-view-lidar-sensor
40. Livox SDK API - GitHub Pages, accessed July 23, 2025, https://livox-sdk.github.io/Livox-SDK-Doc/
41. Livox-SDK/livox_ros_driver: Livox device driver under ros ... - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver
42. Livox-SDK/livox_ros_driver2: Livox device driver under Ros(Compatible with ros and ros2), support Lidar HAP and Mid-360. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver2
43. Releases / Livox-SDK/livox_ros_driver2 - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros_driver2/releases
44. ASIG-X/livox_ros2_avia: ROS2 driver for Livox Avia - GitHub, accessed July 23, 2025, https://github.com/ASIG-X/livox_ros2_avia
45. Livox-SDK/livox_ros2_driver: Livox device driver under Ros2, support Lidar Mid-40, Mid-70, Tele-15, Horizon, Avia. - GitHub, accessed July 23, 2025, https://github.com/Livox-SDK/livox_ros2_driver
46. Livox SDK, accessed July 23, 2025, https://www.livoxtech.com/sdk
47. stm32f303ret6/livox_laser_simulation_RO2: A ROS2 package to provide gazebo-classic plugin for Livox Series LiDAR - GitHub, accessed July 23, 2025, https://github.com/stm32f303ret6/livox_laser_simulation_RO2
48. Solid-State LiDAR Market | Forecast 2025 To 2033 - Business Research Insights, accessed July 23, 2025, https://www.businessresearchinsights.com/market-reports/solid-state-lidar-market-122558
49. Solid-State LiDAR Market - Forecasts from 2025 to 2030 - GII Research, accessed July 23, 2025, https://www.giiresearch.com/report/ksi1742680-solid-state-lidar-market-forecasts-from.html
50. The Future of Robotics: How Lidar is Changing the Game - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/future-robotics-lidar-technology
51. Exploring LiDAR Drones: Applications & Industry Uses - Flyability, accessed July 23, 2025, https://www.flyability.com/blog/lidar-drone
52. LiDAR and Drones: A New Era in Topographical Mapping & Surveying - Advexure, accessed July 23, 2025, https://advexure.com/blogs/news/lidar-and-drones-a-new-era-in-topographical-mapping-surveying


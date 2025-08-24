# Trimble CenterPoint RTX
[GNSS](./index.md)



위성항법시스템(GNSS)은 다양한 수준의 정확도를 제공하며, 그 활용 분야를 결정한다. 가장 기본적인 단계는 수 미터(m) 수준의 정확도를 제공하는 자율(Autonomous) 측위이며, 이를 보완하는 위성기반 보정시스템(SBAS)이나 차분 위성항법시스템(DGPS)을 사용하면 서브미터(sub-meter) 수준으로 정확도가 향상된다. 그러나 측량, 정밀 농업, 건설과 같은 전문 분야에서는 센티미터(cm) 수준의 고정밀도가 필수적으로 요구된다.1 이러한 센티미터 수준의 정확도를 달성하는 기술은 크게 두 가지로 나뉜다. 첫째는 기준국과 이동국 간의 상대적 위치를 계산하여 공통 오차를 소거하는 차분(differential) 방식의 실시간 이동 측위(RTK, Real-Time Kinematic) 기술이다. 둘째는 모든 GNSS 오차 요인을 정밀하게 모델링하여 이동국 단독으로 절대 위치를 결정하는 정밀 단독 측위(PPP, Precise Point Positioning) 기술이다. 이 두 기술의 근본적인 차이를 이해하는 것은 CenterPoint RTX와 같은 서비스의 존재 이유와 그 가치를 파악하는 데 있어 핵심적인 출발점이다.


Trimble CenterPoint RTX는 Trimble의 독점적인 실시간 확장(RTX, Real-Time eXtended) 기술에 기반한 고정밀 PPP 보정 서비스이다.2 이 서비스는 전 세계에 분포된 기준국 네트워크를 통해 생성된 정밀 보정 정보를 위성 또는 인터넷을 통해 수신하여, 실시간으로 센티미터 수준의 위치 정확도를 제공한다.4 사용자는 호환되는 Trimble 수신기와 연간 구독 기반의 라이선스를 통해 이 서비스를 이용할 수 있으며 7, 이를 통해 기존의 복잡한 인프라 없이도 고정밀 측위가 가능해진다.


CenterPoint RTX의 가장 중요한 가치는 로컬 RTK 기준국이나 가상기준점(VRS) 네트워크에 의존하지 않고도 센티미터 수준의 정확도를 달성할 수 있다는 점이다.1 이러한 '제약 없는(untethered)' 특성은 사용자에게 진정한 이동성을 부여하며, 특히 광범위한 지역이나 원격지에서의 작업 흐름을 획기적으로 단순화한다.9 이는 전통적인 RTK 방식이 가진 물류적 복잡성, 즉 기지국 설치의 번거로움, 무선 신호 손실, 셀룰러 통신 음영 지역, 기선 길이(baseline) 제한 등의 근본적인 한계를 직접적으로 해결하는 강력한 대안이 된다.9


CenterPoint RTX의 등장은 고정밀 측위 시장의 비즈니스 모델에 근본적인 변화를 의미한다. 전통적인 RTK 시스템은 기준국 장비 구매라는 막대한 초기 자본 지출(CapEx)을 필요로 하는 모델이었다.12 반면, CenterPoint RTX는 호환 수신기와 연간 구독료만으로 운영되므로, 운영 지출(OpEx) 중심의 모델로 전환시킨다.1 이러한 변화는 여러 측면에서 전략적 의미를 가진다. 첫째, 고정밀 측위 기술에 대한 진입 장벽을 크게 낮춘다. 둘째, 기업은 프로젝트 수요에 따라 고정밀 수신기(로버)의 수를 유연하게 확장하거나 축소할 수 있으며, 이때마다 값비싼 기준국을 추가로 구매할 필요가 없다.9 이는 자원 배분의 효율성을 극대화한다. 최근 Trimble이 신규 수신기에 1년 구독권을 포함하여 제공하는 'RTX on Point' 프로그램은 이러한 OpEx 모델의 채택을 가속화하고, 사용자의 작업 흐름에 RTX를 처음부터 내재시키려는 명확한 전략적 움직임이다.11 결과적으로 CenterPoint RTX는 센티미터급 정확도를 일부 전문가의 전유물이 아닌, 더 넓은 사용자층이 접근 가능한 주류 기술로 변모시키고 있다.



CenterPoint RTX의 핵심은 PPP 기술이다. PPP는 이동국 수신기 위치에서 위성 궤도 오차, 위성 시계 오차, 대기층(전리층 및 대류층) 지연 등 GNSS 신호에 포함된 모든 오차 요소를 직접 모델링하여 보정하는 '절대' 측위 기법이다.1 이는 기준국과 이동국 간의 공통 오차를 상쇄하여 상대 위치를 계산하는 RTK와 근본적으로 다르다. RTK는 매우 높은 정확도를 제공하지만, 오차 상쇄 효과를 극대화하기 위해 기준국과 이동국이 물리적으로 가까운 거리(짧은 기선)를 유지해야 한다는 제약이 따른다.12 반면 PPP는 전 지구적 모델을 사용하므로 이러한 거리 제약에서 자유롭다.


CenterPoint RTX의 성능은 전 세계에 촘촘하게 분포된 100개 이상의 Trimble 글로벌 기준국 네트워크에 의해 뒷받침된다.1 이 기준국들은 24시간 내내 모든 GNSS 위성군(constellation)의 신호를 추적하며 16, 수집된 원시 데이터를 Trimble의 데이터 처리 센터로 실시간 전송한다.3 또한, 이 네트워크에는 보정 정보의 무결성을 검증하기 위한 별도의 모니터링 기준국들이 포함되어 있어, 서비스의 신뢰성과 안정성을 보장하는 이중 안전장치 역할을 한다.15


데이터 처리 센터에서는 수신된 방대한 양의 데이터를 정교한 알고리즘으로 분석하여, 위성의 정밀 궤도, 시계, 그리고 전 지구적 또는 지역적 대기 상태 모델(전리층 및 대류층 지연 모델)을 생성한다.1 이 모델링된 정보는 압축된 형태의 보정 데이터 스트림으로 패키징되어 사용자에게 전송된다. 전송 방식은 두 가지다.

- **위성 (L-Band)**: 보정 정보는 정지궤도 위성군을 통해 L-Band 주파수로 전 세계에 방송된다.3 이 방식 덕분에 셀룰러 통신이 불가능한 원격지나 해상에서도 서비스 이용이 가능하다.
- **인터넷 프로토콜 (IP/Cellular)**: 동일한 보정 정보가 인터넷을 통해서도 제공된다(NTRIP 프로토콜). 셀룰러 통신이 가능한 지역에서는 이 방식을 사용할 수 있다.5

최신 Trimble 시스템은 이 두 가지 전송 방식을 자동으로 전환하는 기능을 갖추고 있다. 인터넷 연결이 끊어지면 수신기는 L-Band 위성 신호로 매끄럽게 전환하고, 인터넷이 다시 연결되면 원래 방식으로 복귀하여 작업 중단을 최소화한다.8


Trimble RTX 네트워크는 한번 구축되고 끝나는 정적인 인프라가 아니라, 지속적으로 발전하고 학습하는 동적인 시스템이라는 점을 이해하는 것이 중요하다. 기준국 네트워크의 지속적인 확장과 밀도 증가는 곧바로 최종 사용자의 성능 향상으로 이어진다. PPP 보정의 품질은 기준국 데이터의 품질과 밀도에 정비례하기 때문이다.1 실제로 Trimble은 북미나 유럽처럼 기준국 네트워크가 조밀한 지역에서는 더 정밀한 '지역' 대기 모델을 제공할 수 있다고 밝히고 있다.15 이러한 고밀도 지역 모델이 바로 1~3분 내의 빠른 수렴 시간을 자랑하는 'RTX Fast' 지역을 가능하게 하는 핵심 기술이다.8 따라서 Trimble이 글로벌 네트워크를 계속 확장함에 따라 더 많은 지역이 'RTX Fast' 등급으로 상향될 가능성이 높다. 이는 사용자가 구매한 구독 서비스의 가치가 시간이 지남에 따라 별도의 조치 없이도 자연스럽게 증가할 수 있음을 의미한다. 이는 정적인 성능을 가진 로컬 RTK 기준국과 비교했을 때, 글로벌 관리형 서비스만이 제공할 수 있는 강력하고 종종 간과되는 장점이다. 사용자는 단순한 서비스를 구매하는 것이 아니라, 지속적인 개선이 이루어지는 생태계에 투자하는 셈이다.



CenterPoint RTX의 성능은 다양한 산업 분야에서 요구하는 정밀도를 충족시킨다.

- **수평 정확도**: 일관되게 수평 정확도 2cm 미만(<2cm) 또는 2.5cm (1인치) 미만으로 명시된다.2 이 수치는 통상적으로 RMS(Root Mean Square) 또는 95% 신뢰수준을 기준으로 한다.
- **수직 정확도**: 수직 정확도는 5cm 미만(RMS)으로 제공된다.2
- **반복성**: 이 서비스는 매년 동일한 위치에서 높은 수준의 반복성을 보장하며, 이는 정밀 농업과 같이 매년 동일한 경로 작업이 중요한 분야에서 필수적인 요소다.7

이러한 성능은 업계에서 'RTK 수준(RTK-level)'으로 통용되며, CenterPoint RTX의 핵심적인 성능 지표이자 마케팅 포인트다.9


수렴(Convergence)은 수신기의 측위 알고리즘이 모든 모호정수(ambiguity)를 해결하고 명시된 최종 정확도에 도달하는 데 걸리는 시간을 의미한다.7 이 시간은 CenterPoint RTX의 실용성을 결정하는 매우 중요한 요소이며, 지역 및 수신기 성능에 따라 달라진다.

- **RTX Fast 지역**: 고밀도 기준국 네트워크가 구축된 특정 지역에서는 수렴 시간이 1분 미만 9 또는 

  2분 미만 7으로 매우 빠르다. 이는 RTK의 즉각적인 초기화와 견줄 수 있는 수준으로, RTX의 활용 범위를 크게 넓혔다.

- **표준 지역 (ProPoint 엔진 탑재)**: 최신 ProPoint 엔진이 탑재된 수신기의 경우, 전 세계 대부분 지역에서 3∼5분 이내에 수렴이 완료된다.8

- **표준 지역 (구형 수신기)**: 구형 수신기는 15∼30분 정도의 수렴 시간이 소요될 수 있다.10

주목할 점은 수렴 시간이 기술 발전에 따라 과거 20분 이상에서 현재 5분 미만으로 극적으로 단축되었다는 사실이다.19 이는 Trimble의 기술력이 빠르게 진보하고 있음을 보여주는 지표다.


CenterPoint RTX는 현존하는 대부분의 GNSS 위성군을 지원한다. 여기에는 GPS(미국), GLONASS(러시아), Galileo(유럽), BeiDou(중국), QZSS(일본)가 포함된다.5 모든 가용 위성군을 활용하면 추적 가능한 위성의 수가 크게 증가하여, 부분적으로 하늘이 가려진 까다로운 환경에서도 측위의 안정성과 신뢰성, 그리고 가용성을 높이는 데 결정적인 역할을 한다.11 이는 일부 위성군만 지원하는 구형 RTK 시스템에 비해 명백한 이점이다.


- **네트워크 가동 시간**: 전 세계 네트워크는 완벽한 이중화 인프라를 갖추고 있으며, 운영 엔지니어에 의해 24/7 모니터링되어 매우 높은 서비스 가용성을 보장한다.12
- **신호 단절 대응**: 보정 신호가 일시적으로 중단되더라도 최대 200초 동안 작업을 지속할 수 있다.7
- **태양 활동 저항성**: Trimble은 진보된 대기 모델과 최신 IonoGuard 기술을 통해 태양 활동 극대기나 전리층 교란 시기에도 측위 성능 저하를 최소화하도록 설계했다.15 이는 현재 정점을 향해 가는 태양 활동 25주기(Solar Cycle 25)에서 특히 중요한 기능이다.


RTX 기술의 발전 과정에서 수렴 시간의 단축은 Trimble이 RTK의 아성에 도전하기 위한 가장 중요한 전략적 지표였다. RTK의 가장 큰 장점은 전원을 켜자마자 거의 즉각적으로 고정해(fixed solution)를 얻을 수 있다는 신속성이었다. 이는 짧은 시간 동안 여러 지점을 측정해야 하는 작업에 절대적으로 유리했다. 초기의 PPP/RTX 기술은 수렴에 수십 분이 걸려 12 이러한 작업에는 부적합했고, 장시간 이동하며 측정하는 일부 응용에 국한되었다.10 Trimble이 R&D 역량을 집중하여 수렴 시간을 20분대에서 5분 미만, 나아가 'Fast' 지역에서는 1분 미만으로 단축시킨 것은 9 RTK의 핵심 강점인 '신속성'을 정면으로 겨냥한 것이다. '정확도 도달 시간'의 격차를 좁힘으로써, Trimble은 '기준국 없는 편리함'과 '무제한의 작업 반경'이라는 RTX 본연의 장점을 훨씬 더 넓은 범위의 사용자들에게 매력적으로 어필할 수 있게 되었다. 또한 'Standard'와 'Fast'라는 등급 구분은 7 핵심 시장에서는 프리미엄급 경험을 제공하고, 그 외 지역에서는 여전히 가치 있는 서비스를 제공하는 영리한 시장 세분화 전략이기도 하다.



- **RTX**: 호환되는 로버 수신기와 구독 라이선스만 있으면 된다. 이는 현장 물류를 단순화하고, 운반할 장비를 줄이며, 기준국 전원이나 무선 링크 같은 잠재적 고장 지점을 원천적으로 제거한다.1
- **RTK**: 무선(Radio)으로 보정 신호를 송신하는 기준국, 또는 셀룰러 모뎀을 통해 VRS 네트워크에 접속해야 한다. 이는 추가적인 하드웨어와 복잡성을 야기하며, 기준국 자체가 단일 장애점(Single Point of Failure)이 될 수 있다.12


- **RTX**: 물리적 인프라로부터의 거리 제약이 없다. 사용자가 도심에서 1 km 떨어져 있든, 1000 km 떨어진 외딴곳에 있든 일관된 정확도를 제공한다. 이는 광활한 지역, 원격지 작업, 파이프라인이나 도로 같은 선형 프로젝트에 이상적이다.9
- **RTK**: 기준국과 로버 간의 기선 거리에 근본적으로 제약을 받는다. 거리가 멀어질수록 정확도가 저하되며(PPM 오차), 무선 통신 거리는 보통 수 킬로미터에 불과하다. VRS 네트워크 역시 서비스 커버리지 영역이 한정되어 있다.9


- **데이텀 기반**: RTX 위치는 기본적으로 측정 시점의 ITRF2020이라는 글로벌 기준 좌표계(Global Reference Frame)에서 계산된다.10
- **좌표 변환**: Trimble Access 필드 소프트웨어나 TBC 오피스 소프트웨어는 이 글로벌 좌표를 사용자의 로컬 좌표계로 실시간 자동 변환한다. 이때 높은 정확도를 위해 시간 종속 변환(time-dependent transformation)과 지역 변위 모델(local deformation model)과 같은 고급 기술이 사용된다.16
- **RTX-RTK 오프셋**: 하나의 프로젝트에서 RTX와 로컬 RTK 데이터를 함께 사용할 경우, 만약 로컬 RTK 기준국이 글로벌 데이텀에 완벽하게 연결되어 있지 않다면 두 데이터 간에 미세한 불일치가 발생할 수 있다. Trimble Access는 이러한 불일치를 보정하기 위해 동일 지점의 RTX와 RTK 측량을 통해 'RTX-RTK 오프셋'을 계산하고, 이를 모든 RTX 측정값에 적용하여 데이터를 조화시키는 기능을 제공한다.8 이는 전환기 사용자나 혼합 환경에서 작업하는 사용자에게 매우 중요한 기능이다.


- **xFill® 기술**: Trimble xFill은 RTK 보정 신호(무선 또는 셀룰러)가 잠시 끊겼을 때 최대 5분간 그 공백을 메워주는 기술이다.14
- **xFill Premium (RTX 기반)**: 수신기에 유효한 CenterPoint RTX 구독이 활성화되어 있으면, 이 xFill 서비스가 *무제한*으로 확장된다. RTK 신호가 끊기면 시스템은 즉시 RTX 기반 측위로 전환하여, 사용자가 주 RTK 링크가 복구될 때까지 로컬 좌표계에서 센티미터급 정확도로 작업을 계속할 수 있게 해준다.9
- **전략적 가치**: 이 기능은 RTX를 단순한 RTK의 '대안'에서 RTK의 '보완재'이자 '강화재'로 격상시킨다. 통신 두절로 인한 작업 중단 시간을 원천적으로 제거하는 완벽한 이중화, 즉 '측위 보험'을 제공하는 셈이다.


- **RTX 비용**: 주로 연간 구독료로 구성되며, 다년 구독 시 할인이 제공되기도 한다.7 일부 수신기 기능 해제(unlock) 비용이 필요할 수 있다.7
- **RTK 비용**: 기준국, 삼각대, 전원 공급 장치, 무선 장비 등에 대한 상당한 초기 자본 투자가 필요하다. 또한 설치 및 해체에 드는 인건비, 유지보수 비용, 도난 및 파손의 위험도 감수해야 한다.12 VRS의 경우 구독료 방식이지만 지리적 제약이 따른다.
- **분석**: 단일 작업자에게는 3~5년간의 TCO가 비슷할 수 있다. 하지만 여러 팀을 운영하는 조직의 경우, 각 로버마다 별도의 기준국 없이 구독만 추가하면 되므로 RTX 모델이 훨씬 확장성이 높고 비용 효율적이다.9 2019년 가격 기준으로 CenterPoint RTX 1년 구독료가 $1,095 USD였으며 현재도 유사한 수준에서 제공되고 있어 22, 이는 VRS 구독료나 기준국 장비의 감가상각 비용과 충분히 경쟁력 있는 수준이다. (자세한 비용 구조는 섹션 5.5 참조)


| 기능                   | CenterPoint RTX                 | RTK (기준국/이동국)               | RTK (VRS 네트워크)        |
| ---------------------- | ------------------------------- | --------------------------------- | ------------------------- |
| **필요 인프라**        | 로버 + 구독                     | 기준국 + 로버 + 무선장비          | 로버 + 모뎀 + 구독        |
| **초기 비용**          | 낮음 (구독료)                   | 높음 (장비 구매)                  | 낮음 (구독료)             |
| **작업 반경**          | 전 세계 / 무제한                | 무선 통신 거리 제한 (통상 <10 km) | 네트워크 서비스 영역 제한 |
| **초기화 시간**        | <1분 (Fast) ~ <5분 (전 세계)    | 거의 즉시                         | 거의 즉시                 |
| **정확도 (수평/수직)** | <2cm / <5cm                     | <2cm / <3cm (일반적)              | <2cm / <3cm (일반적)      |
| **물류 복잡성**        | 매우 낮음                       | 높음 (설치/보안)                  | 낮음                      |
| **이중화/안정성**      | 높음 (글로벌 네트워크)          | 낮음 (단일 기준국)                | 중간 (네트워크)           |
| **이상적 활용 사례**   | 원격/선형 프로젝트, 장비 확장성 | 소규모 현장, 반복 작업            | 도심/교외 지역            |
| **주요 약점**          | 수렴 시간                       | 거리 제약, 물류 부담              | 셀룰러 통신 의존          |



CenterPoint RTX는 Trimble 및 자회사인 Spectra Geospatial의 광범위한 수신기 포트폴리오와 호환된다.

- **측량용 수신기**: R-시리즈 (R12i, R12, R10, R2), R780, R580 2
- **건설용 수신기**: SPS-시리즈 (SPS986, SPS855, SPS785), R750, MPS865 2
- **농업용 수신기**: NAV-900, AG-372, AG-382, AG-392 7
- **OEM/UAV용 모듈**: APX-시리즈 (APX-15, 18, 20, 30), BD-시리즈 (BD992 등) 7
- **디스플레이/컨트롤러**: GFX-시리즈 (GFX-1260, GFX-1060), TMX-2050, CFX-750 7


- **필드 소프트웨어**: 측량 및 건설 분야의 주력 필드 소프트웨어인 Trimble Access는 RTX를 완벽하게 지원한다. 보정 소스 선택(인터넷/위성), 좌표계 변환, RTX-RTK 오프셋 관리 등 모든 기능이 소프트웨어 내에서 매끄럽게 처리된다.8
- **오피스 소프트웨어**: Trimble Business Center(TBC)는 데이터 후처리 및 관리에 사용된다. TBC는 RTX로 수집된 데이터를 정확하게 인식하고 처리하며, Access와 동일한 고급 데이텀 변환 기능을 적용하여 현장과 사무실 간의 데이터 일관성을 보장한다.16
- **후처리 서비스**: 실시간 작업이 아닌 경우, 사용자는 웹 기반 후처리 서비스에 GNSS 관측 데이터를 업로드하여 RTX 기술로 처리된 정밀 위치를 얻을 수도 있다.24


Trimble은 사용자의 다양한 정확도 요구사항과 예산에 맞춰 여러 등급의 RTX 서비스를 제공한다.

- **CenterPoint RTX**: 측량 등급의 최고 정밀도(<2.5cm)를 제공하는 플래그십 서비스.2
- **FieldPoint RTX**: 약 10cm 수준의 정확도를 제공하며, GIS 데이터 수집 및 자산 매핑에 적합.2
- **RangePoint RTX**: 약 15∼50cm 수준의 광역 정확도를 제공하며, 정밀도가 덜 요구되는 농업 작업 등에 사용.2
- **ViewPoint RTX**: 약 30∼50cm 수준의 입문용 서비스로, 기본적인 매핑 및 안내에 활용.2


| 서비스 등급         | 수평 정확도 | 수직 정확도 | 일반적 수렴 시간 | 주요 활용 분야                         | 대상 산업        |
| ------------------- | ----------- | ----------- | ---------------- | -------------------------------------- | ---------------- |
| **CenterPoint RTX** | <2.5cm      | <5cm        | <1분 ~ <20분     | 지형/지적 측량, 기준점 측량, 공사 측설 | 측량, 건설, 농업 |
| **FieldPoint RTX**  | 10cm        | 정보 없음   | 정보 없음        | 자산 위치 매핑, 대규모 경계 지역 결정  | GIS, 유틸리티    |
| **RangePoint RTX**  | 15cm∼50cm   | 정보 없음   | <5분             | 광역 농업 (파종, 방제 등)              | 농업             |
| **ViewPoint RTX**   | 30cm∼50cm   | 정보 없음   | <5분             | 필드 정보 수집, 자산 관리, 기본 안내   | 농업, GIS        |

참고: 정확도 및 수렴 시간은 수신기, 지역, 작업 환경에 따라 달라질 수 있음. 2


RTX 서비스는 주로 연간 구독 방식으로 제공되며, 다년 계약 시 할인 혜택이 있다.7 구독은 수신기의 고유 시리얼 번호에 귀속된다.7 'RTX on Point'는 Trimble의 핵심적인 사업 전략으로, 특정 신규 수신기(예: 2024년 4월 이후 구매한 R-시리즈, NAV-900)에 1년간의 CenterPoint RTX 구독권을 무료로 활성화하여 제공하는 프로그램이다.11 이 프로그램은 초기 구독 비용이라는 진입 장벽을 제거함으로써 사용자가 RTX 기술을 쉽게 시험해보고 일상적인 작업 흐름에 통합하도록 유도한다. 이는 전형적인 서비스형 소프트웨어(SaaS) 산업의 '랜드 앤 익스팬드(Land and Expand)' 전략을 지오스페이셜 하드웨어 시장에 적용한 것으로, 1년 후 사용자가 서비스의 가치와 편리함을 체감하고 자연스럽게 유료 구독으로 전환하게 만드는 것을 목표로 한다.


Trimble RTX 서비스의 비용은 주로 연간 구독료(OpEx) 모델을 기반으로 하며, 이는 사용자가 필요한 서비스 등급과 기간을 유연하게 선택할 수 있도록 한다.12

- **CenterPoint RTX**: 최고 등급의 이 서비스는 북미 지역 리셀러 기준으로 연간 약 $1,095 USD에 제공된다.7 이는 2019년에 'Fast'와 'Standard' 옵션의 가격이 통합된 이후 비교적 일관되게 유지된 가격이다.22
- **기타 서비스 등급**: 더 넓은 범위의 정확도를 제공하는 서비스도 있다.
  - **RangePoint RTX**: 약 $475 USD부터 시작한다.38
  - **ViewPoint RTX**: 약 $400 USD부터 시작한다.38
- **다년 구독 할인**: Trimble은 2년, 3년, 5년 등 다년 구독 계약 시 할인 혜택을 제공하여 장기 사용자의 총 소유 비용을 절감해준다. 예를 들어, 2019년 가격 기준으로 CenterPoint RTX의 5년 구독료는 연간 $876 USD 수준으로 인하되었다.22
- **추가 비용**: 구독료 외에 고려해야 할 비용이 있다. 특정 정확도 수준(예: CenterPoint RTX Fast)을 활성화하기 위해서는 수신기 자체에 대한 추가적인 '정확도 라이선스' 구매가 필요할 수 있다.7 이는 초기 하드웨어 투자 비용에 영향을 미치는 요소다.

이러한 가격 정책은 사용자가 초기 자본 지출(CapEx) 부담 없이 필요에 따라 고정밀 측위 기술을 도입하고 확장할 수 있도록 지원하는 RTX 생태계의 핵심 전략이다.


| 서비스 등급         | 예상 연간 구독료 (1년 기준) | 주요 특징                              |
| ------------------- | --------------------------- | -------------------------------------- |
| **CenterPoint RTX** | ~$1,095 7                   | 센티미터(<2.5cm) 수준의 최고 정밀도    |
| **RangePoint RTX**  | ~$475 38                    | 서브미터(15cm∼50cm) 수준의 광역 정확도 |
| **ViewPoint RTX**   | ~$400 38                    | 서브미터(30cm∼50cm) 수준의 기본 정확도 |

참고: 위 가격은 북미 지역 리셀러의 공시 가격(MSRP)을 기준으로 하며, 지역, 프로모션, 다년 계약 및 필요한 수신기 라이선스에 따라 달라질 수 있다.7



CenterPoint RTX는 지형 및 지적 측량, 공사 측설, 자산 관리 등 다양한 측량 분야에서 활용된다.2 특히 VRS/셀룰러 통신이 불가능한 원격지나 산악 지역, 광범위한 선형 프로젝트(파이프라인, 송전선로 등)에서 그 진가를 발휘한다.10 고프 섬(Gough Island) 복원 프로젝트 사례에서처럼, 험난한 환경에서 기준점 설치의 어려움 없이 신속하게 정밀 지형 데이터를 수집하는 데 결정적인 역할을 했다.27 이는 시간과 비용이 많이 드는 기준점 측량 작업을 생략하고, 모든 수신기를 이동국(로버)으로 활용하여 현장 인력의 효율성을 극대화할 수 있게 한다.9


정밀 농업 분야에서 CenterPoint RTX는 스트립 틸링, 파종, 방제, 시비 등 모든 작업 단계에 적용된다.2 센티미터급의 반복 가능한 정확도는 작업 경로의 중복을 최소화하여 종자, 비료, 약품과 같은 투입 비용을 최대 5%까지 절감할 수 있게 한다.18 또한, 매년 동일한 경로로 농기계를 주행시키는 제어 경로 농법(Controlled Traffic Farming)을 가능하게 하여 토양 다짐을 줄이고 토양 건강을 개선하는 데 기여한다.13 복잡한 장비 설정 없이 위성 신호만으로 RTK급 정확도를 얻을 수 있다는 단순함은 신뢰할 수 있는 기술을 필요로 하는 농업인들에게 큰 장점으로 작용한다.12 Trimble은 농업용 NAV-900 컨트롤러에 1년 무료 구독을 번들로 제공하며 이 시장에서의 채택을 강력하게 추진하고 있다.13


건설 현장에서 CenterPoint RTX는 초기 지형 측량, 설계 검토, 부지 정리, 준공 측량, 개략적인 측설 등에 사용된다.2 특히 현장 감독관이나 엔지니어가 기준국 설치 없이 RTX가 탑재된 로버만으로 광대한 현장 어디에서든 신속하게 검측이나 물량 계산을 수행할 수 있게 해준다.1 이는 전문 측량팀과 고가의 장비를 더 중요한 작업에 집중시킬 수 있게 하여 프로젝트 전반의 효율성을 향상시킨다.


전통적으로 UAV를 이용한 고정밀 매핑은 지상에 다수의 지상기준점(GCP)을 설치하거나, 통신 링크와 기선 거리 제약이 있는 RTK를 이용해야 하는 등 시간과 비용이 많이 드는 작업이었다.20 CenterPoint RTX는 이러한 패러다임을 바꾸고 있다. Trimble APX RTX 포트폴리오는 소형 GNSS-관성측정장치(IMU) 시스템과 CenterPoint RTX 보정 서비스를 UAV에 직접 통합한다.23 이를 통해 '직접 지리 참조(Direct Georeferencing)'가 가능해지는데, 이는 카메라의 모든 픽셀이나 LiDAR 센서의 모든 점에 GCP나 기준국 없이도 실시간으로 정확한 지리적 위치를 부여하는 기술이다. 이 혁신은 비행 계획부터 최종 성과물 제작에 이르는 전체 매핑 작업 흐름을 극적으로 간소화하여 막대한 시간과 비용을 절감시킨다.20 이는 RTX 기술의 가장 파급력 있는 응용 분야 중 하나로 평가받는다.



- **과제**: 모든 GNSS 기술과 마찬가지로, RTX의 성능은 '도심 협곡(urban canyon)'이라 불리는 고층 빌딩 밀집 지역이나 울창한 삼림 지대와 같이 위성 신호를 가리는 장애물에 의해 저하될 수 있다.15 이러한 환경은 신호 손실과 다중경로 오차(multipath error)를 유발한다.
- **완화 전략 - Trimble ProPoint® 엔진**: 이는 Trimble의 최신 수신기에 통합된 첨단 GNSS 처리 엔진이다.14 ProPoint 엔진은 이러한 까다로운 환경에서 더 안정적이고 정확한 위치를 제공하도록 특별히 설계되었다.14 지능적으로 신호를 필터링하고, 특정 위성군에 얽매이지 않고 가용한 모든 위성 신호를 최적으로 활용한다. Trimble은 이전 세대 엔진에 비해 까다로운 환경에서 정확도, 정밀도, 신뢰성 등 주요 지표가 최소 30% 향상되었다고 주장한다.32


- **과제**: 수렴 시간은 비약적으로 단축되었지만, 여전히 RTK의 즉각적인 고정해와는 차이가 있다. 표준 지역에서는 최종 정확도를 얻기 위해 몇 분간의 대기 시간이 필요할 수 있다.17
- **완화 전략**: 적절한 작업 계획이 필요하다. 도심 지역에서 다수의 개별점을 측설하는 것처럼 짧은 관측을 자주 반복해야 하는 작업에는 여전히 RTK가 더 효율적일 수 있다. 반면, 지형 측량이나 장비 제어처럼 지속적으로 이동하거나 한 지점에서 오래 머무는 작업의 경우, 초기 수렴 시간은 전체 작업 시간에서 미미한 비중을 차지하게 된다. 핵심은 사용자가 당면한 특정 작업에 가장 적합한 도구를 선택하는 것이다.9


- **과제**: 지구의 대기(특히 전리층)는 GNSS 오차의 가장 큰 원인이다. 특히 현재와 같은 태양 활동 극대기에는 강력한 태양 폭풍이 전리층을 심하게 교란하여 GNSS 측위를 저하시키거나 불가능하게 만들 수 있다.15
- **완화 전략 - 고급 모델링 및 IonoGuard**: RTX는 글로벌 네트워크를 사용하여 대기 효과를 적극적으로 모델링하므로, 단일 기준점 주변의 대기가 동일하다고 가정하는 RTK보다 본질적으로 이러한 영향에 더 강하다.15 최근 Trimble은 태양 폭풍으로 인한 극심한 전리층 교란 시에도 신호 무결성과 측위 성능을 유지하도록 특별히 설계된 차세대 기술인 IonoGuard를 도입했다.21


ProPoint 엔진이나 IonoGuard와 같은 기술의 개발은 GNSS의 내재적 한계와 이를 극복하려는 기술 솔루션 간의 끊임없는 '기술 경쟁'을 보여준다. 이는 단순히 이상적인 조건에서의 정확도를 높이는 것을 넘어, 불완전한 실제 환경에서 기술의 신뢰성을 보장하는 '성능 범위(performance envelope)'를 확장하는 것이다. RTX의 초기 가치 제안은 '기준국 없는 정확도'였지만, 긴 수렴 시간과 음영 지역에서의 성능 저하라는 단서가 붙었다. 경쟁사들과 RTK 지지자들은 이러한 약점을 지적했다. 시장을 확대하기 위해 Trimble은 이러한 문제들을 정면으로 해결해야만 했다. ProPoint 엔진 개발은 '도심 협곡/삼림' 문제에 대한 직접적인 해답이었고 14, 수렴 시간의 지속적인 단축은 '정확도 도달 시간' 문제에 대한 해답이었다.19 그리고 IonoGuard의 도입은 예측 가능한 '태양 활동 극대기' 문제에 대한 선제적 대응이다.21 이는 Trimble의 R&D가 핵심 PPP 알고리즘뿐만 아니라, 사용자가 기술 채택을 주저하게 만드는 요인들을 체계적으로 식별하고 제거하는 총체적인 '사용자 경험'에 초점을 맞추고 있음을 보여준다. 이러한 한계 완화에 대한 노력은 서비스의 성숙도와 장기적인 생존 가능성을 가늠하는 핵심 지표다.



업계의 궁극적인 목표는 PPP의 장점(글로벌 일관성, 기준국 불필요)과 RTK의 장점(즉각적인 수렴)을 결합하는 것이다. 이는 종종 PPP-RTK로 불린다. Hexagon과 같은 경쟁사들도 'RTK From the Sky' 기술을 통해 전 세계적으로 1분 미만의 수렴을 목표로 이 분야에서 경쟁하고 있다.33 미래에는 CenterPoint RTX와 같은 서비스의 수렴 시간이 전 세계적으로 계속 단축되어, 결국 RTK의 즉각적인 성능에 근접하게 될 것이며, 이는 시장에 궁극적인 파괴적 혁신을 가져올 것이다.


- **한국 (KT)**: 2025년 KT와의 파트너십 발표는 매우 중요한 의미를 가진다. 이는 통신 서비스와 정밀 측위 서비스를 결합하여 제공하는 것으로 34, 한국의 강력한 5G 네트워크를 RTX 보정 정보의 안정적인 주 전송 채널로 활용하여 한층 강화된 서비스 등급을 제공하려는 전략으로 분석된다.

- **자동차/IoT (TDK, 퀄컴)**: TDK의 IMU 센서와 RTX를 결합하고 36, 퀄컴과 협력하는 34 최근의 움직임은 자동차 및 IoT 시장으로의 본격적인 진출을 시사한다. 이는 자율주행차, 첨단 운전자 보조 시스템(ADAS), 산업용 로봇 등에 고정밀, 고신뢰성 측위 솔루션을 제공하는 것을 목표로 한다.

  이러한 파트너십은 Trimble이 RTX를 전통적인 시장(측량, 건설, 농업)을 위한 도구를 넘어, 훨씬 더 광범위한 초연결 세계를 위한 기반 기술 플랫폼으로 보고 있음을 보여준다.


Trimble은 시장 선도 기업이지만 유일한 플레이어는 아니다. Hexagon(NovAtel)의 TerraStar 서비스와 'RTK From the Sky' 기술 33, 그리고 Galileo의 고정밀 서비스(HAS)와 같은 공공 서비스의 등장은 37 역동적이고 경쟁적인 환경을 조성하고 있다. 이러한 경쟁은 지속적인 기술 혁신을 촉진하고 가격 인하 압력으로 작용하여 최종 사용자에게 혜택을 줄 것이다.


CenterPoint RTX는 단순한 제품을 넘어, 고정밀 측위의 물류, 경제성, 접근성을 근본적으로 바꾸고 있는 전략적 플랫폼이다. 수렴 시간과 안정성의 끊임없는 개선을 통해 원격지를 위한 틈새 솔루션에서 RTK의 주류 경쟁자로 성공적으로 자리매김했다. 이 기술의 진정한 장기적 가치는 자율주행차, UAV, IoT와 같은 신흥 대중 시장을 위한 핵심 기술로서, 물리적 세계와 디지털 세계를 센티미터 수준의 확실성으로 연결하는 역할에 있다.9 특히 한국 시장의 사용자에게는 성숙한 글로벌 서비스와 현지화된 파트너십(KT)의 결합이 측위 기술에 대한 매력적이고 미래 지향적인 투자를 의미한다. 이 서비스는 한국의 발전된 인프라를 활용하여 세계 최고 수준의 경험을 제공할 준비가 되어 있다.

**참고 자료**

1. Maximizing Accuracy with Trimble RTX: A Comprehensive Guide - SITECH Southwest, accessed July 17, 2025, https://www.sitechsw.com/trimble-rtx-whats-in-it-for-me/
2. Trimble RTX Correction Services, accessed July 17, 2025, https://positioningservices.trimble.com/en/rtx
3. Trimble CenterPoint RTX – A First Study on Supporting Galileo - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/280085552_Trimble_CenterPoint_RTX_-_A_First_Study_on_Supporting_Galileo
4. help.fieldsystems.trimble.com, accessed July 17, 2025, [https://help.fieldsystems.trimble.com/trimble-access/latest/ko/gnss-survey-rtx.htm#:~:text=Trimble%20Centerpoint%20RTX%E2%84%A2%20%EB%B3%B4%EC%A0%95,precise%20point%20positioning)%20%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9E%85%EB%8B%88%EB%8B%A4.](https://help.fieldsystems.trimble.com/trimble-access/latest/ko/gnss-survey-rtx.htm#:~:text=Trimble Centerpoint RTX™ 보정,precise point positioning) 시스템입니다.)
5. Trimble Real-Time CenterPoint RTX | Applanix, accessed July 17, 2025, https://applanix.trimble.com/en/services/real-time-centerpoint-rtx
6. Trimble CenterPoint RTX Post-Processing Service, accessed July 17, 2025, https://www.trimblertx.com/
7. Trimble | Precision Ag | CenterPoint RTX - VANTAGE NORTHEAST, accessed July 17, 2025, https://vantagenortheast.com/centerpoint-rtx-standard-or-fast/
8. RTX 보정 서비스 - Trimble Field Systems Help Portal, accessed July 17, 2025, https://help.fieldsystems.trimble.com/trimble-access/latest/ko/gnss-survey-rtx.htm
9. 5 Uses for Trimble CenterPoint RTX: Adding Value to a Surveyor's Toolset, accessed July 17, 2025, https://geospatial.trimble.com/en/resources/blog/five-uses-for-trimble-centerpoint-rtx-adding-value-to-a-surveyors-toolset
10. RTX correction service - Trimble Field Systems Help Portal, accessed July 17, 2025, https://help.fieldsystems.trimble.com/trimble-access/latest/en/gnss-survey-rtx.htm
11. Trimble CenterPoint RTX Correction Service - SITECH Construction Systems, accessed July 17, 2025, https://sitechcs.com/docs/trimble-centerpoint-rtx-and-gnss-receivers-datasheet.pdf
12. Trimble CenterPoint RTX Fast: one-inch accuracy in under a minute - Future Farming, accessed July 17, 2025, https://www.futurefarming.com/smart-farming/tools-data/trimble-centerpoint-rtx-fast-one-inch-accuracy-in-under-a-minute/
13. Trimble Introduces Integrated Precise Point Positioning as a Subscription for Agriculture Applications - Oct. 26, 2023, accessed July 17, 2025, https://news.trimble.com/2023-10-26-Trimble-Introduces-Integrated-Precise-Point-Positioning-as-a-Subscription-for-Agriculture-Applications
14. RTK, Trimble RTX, Trimble xFill, and Trimble ProPoint - Trimble Field Systems Help Portal, accessed July 17, 2025, https://help.fieldsystems.trimble.com/r980/position-modes-rtk-rtx-xfill-propoint.htm
15. Frequently Asked Questions Correction Service - Trimble Geospatial, accessed July 17, 2025, https://geospatial.trimble.com/downloads/7EN0wFRAk234kbJgk4FZOV/b47339a9dd4437b25f9e6104050d149f/Trimble_RTX_Correction_Services_FAQ.pdf
16. Coordinate System Enhancements for CenterPoint RTX Corrections in Trimble Access and TBC, accessed July 17, 2025, https://geospatial.trimble.com/en/resources/blog/coordinate-system-enhancements-for-centerpoint-rtx-corrections
17. Trimble Positioning & Correction Services | Guidance | Agriculture, accessed July 17, 2025, https://ptxtrimble.com/en/positioning-services
18. Trimble CenterPoint RTX - KOREC Group, accessed July 17, 2025, https://www.korecgroup.com/product/trimble-centerpoint-rtx/
19. Trimble CenterPoint RTX Upgrade Offers RTK Accuracy Without Complexity - No-Till Farmer, accessed July 17, 2025, https://www.no-tillfarmer.com/articles/10965-upgrade-to-trimbles-centerpoint-rtx-makes-convergence-time-75-faster
20. Airborne Direct Georeferencing with Trimble Centerpoint RTX - an upgrade that pays off in more ways than one, accessed July 17, 2025, https://applanix.trimble.com/en/resources/external-publications/airborne-direct-georeferencing-with-trimble-centerpoint-rtx-an-upgrade-that-pays-off-in-more-ways
21. Trimble and PTx Trimble Expand Innovative Technology to Maintain Precision and Continuous Operations in the Agriculture Industry - AGCO, accessed July 17, 2025, https://news.agcocorp.com/2025-03-20-Trimble-and-PTx-Trimble-Expand-Innovative-Technology-to-Maintain-Precision-and-Continuous-Operations-in-the-Agriculture-Industry
22. Low Mu Tech Dust New Trimble RTX Correction Pricing - Linco Precision, accessed July 17, 2025, https://www.lincoprecision.com/wp-content/uploads/2019/01/1-21-19-Newsletter.pdf
23. Trimble APX RTX - Applanix, accessed July 17, 2025, https://applanix.trimble.com/en/products/hardware/trimble-apx-rtx
24. Trimble CenterPoint RTX Post-Processing Service, accessed July 17, 2025, https://trimblertx.com/UploadForm.aspx
25. Applanix POSPac PP-RTX - Trimble, accessed July 17, 2025, https://applanix.trimble.com/en/services/applanix-pospac-pp-rtx
26. Plans & Pricing for Agriculture - RTX Corrections - Trimble RTX, accessed July 17, 2025, https://rtxcorrections.vantage-au.com/gnss-correction-services/
27. Trimble CenterPoint RTX Correction Service Helps Solve Century-Old Problem on an Isolated Island, accessed July 17, 2025, https://www.trimble.com/en/our-commitment/people-and-planet/climate-action/case-studies/saving-gough-islands-birds
28. Farming in Kentucky with Trimble CenterPoint RTX correction service - YouTube, accessed July 17, 2025, https://www.youtube.com/watch?v=9AK6mzCWlBQ
29. Base Station-Free LiDAR Mapping with GeoCue's and Trimble's RTX Technology, accessed July 17, 2025, https://geocue.com/resources/articles/base-station-free-lidar-mapping-with-geocues-and-trimbles-rtx-technology/
30. Trimble Introduces New Direct Georeferencing Portfolio for UAV Mapping, accessed July 17, 2025, https://news.trimble.com/2024-09-24-Trimble-Introduces-New-Direct-Georeferencing-Portfolio-for-UAV-Mapping
31. Trimble Introduces New Direct Georeferencing Portfolio for UAV Mapping, accessed July 17, 2025, https://insideunmannedsystems.com/trimble-introduces-new-direct-georeferencing-portfolio-for-uav-mapping/
32. Keeping connected on the road - Trimble Geospatial, accessed July 17, 2025, https://geospatial.trimble.com/en/resources/blog/keeping-connected-on-the-road
33. RTK From Sky | Hexagon, accessed July 17, 2025, https://hexagon.com/company/divisions/autonomy-and-positioning/rtk-from-sky
34. Trimble Mediaroom - News Releases, accessed July 17, 2025, https://news.trimble.com/news-releases?l=100
35. News Releases - Trimble Mediaroom, accessed July 17, 2025, https://news.trimble.com/news-releases
36. Trimble and TDK Join Forces to Accelerate Precision Navigation - Jun 24, 2025, accessed July 17, 2025, https://news.trimble.com/2025-06-24-Trimble-and-TDK-Join-Forces-to-Accelerate-Precision-Navigation
37. PPP-RTK market and technology report - EU Agency for the Space Programme, accessed July 17, 2025, https://www.euspa.europa.eu/sites/default/files/calls_for_proposals/rd.03_-_ppp-rtk_market_and_technology_report.pdf




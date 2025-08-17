# NovAtel TerraStar-X

## 1.  개요

NovAtel의 TerraStar-X는 기존의 지역 기지국이나 셀룰러 통신망 연결 없이도 실시간 이동 측위(Real-Time Kinematic, RTK) 수준의 성능을 제공하는 프리미엄 위성 기반 GNSS 보정 서비스이다.1 이 서비스의 핵심 가치는 "하늘에서 직접 수신하는 RTK(RTK From the Sky™)"라는 개념으로 요약될 수 있으며, 이는 전통적인 고정밀 측위 방식의 제약을 극복하려는 기술적 지향점을 명확히 보여준다.2

TerraStar-X의 핵심 성능 지표는 센티미터 수준의 정확도(수평 2 cm RMS)와 60초 미만의 수렴 시간(Convergence Time)이다.3 특히 이 빠른 수렴 시간은 기존의 정밀 단독 측위(Precise Point Positioning, PPP) 서비스와 비교했을 때 가장 두드러지는 차별점이다. 이러한 성능은 전 지구 위성 궤도 및 시각 보정 정보에 지역별 대기층 보정 정보를 결합하는 하이브리드 PPP-RTK 아키텍처를 통해 구현된다.6

주요 목표 시장은 자동 조향 및 파종과 같은 정밀 농업 분야와 첨단 운전자 보조 시스템(ADAS) 및 자율주행(AD)과 같은 빠르게 성장하는 자율주행차 분야이다.7 특히 자율주행 분야에서는 차선 수준의 연속적인 위치 정보를 제공하며, ISO 26262와 같은 기능 안전 표준을 충족하여 안전이 중요한 애플리케이션의 요구 사항에 부응한다.4

전략적 서비스 확장 측면에서 주목할 점은 'TerraStar-X Enterprise'라는 특화된 서비스 모델이다. 대한민국에서 문화방송(MBC)과의 파트너십을 통해 기존 지역 RTK 인프라를 활용하여 첨단 기술 시장에 신속하게 서비스를 배포한 사례는 TerraStar-X의 글로벌 확장 전략을 보여주는 대표적인 예이다.9

결론적으로, TerraStar-X는 RTK의 정확성과 속도, 그리고 PPP의 편리성과 확장성을 결합하여 시장의 요구에 부응하는 GNSS 보정 기술의 중요한 진화를 대표한다. 이는 고정밀 위치 정보가 필수적인 다양한 산업 분야에서 새로운 가능성을 열어주는 핵심 기술로 평가된다.

## 2.  정밀도의 기반: GNSS 보정 방법론 입문

### 2.1  단독 GNSS의 내재적 오차

보정 서비스의 필요성을 이해하기 위해서는 먼저 단독 GNSS 수신기가 가지는 본질적인 한계를 파악해야 한다. 위성항법시스템(GNSS) 신호는 위성에서 수신기까지 도달하는 과정에서 다양한 오차 요인에 의해 정확도가 저하된다. 주요 오차 원인으로는 위성에 탑재된 원자시계의 미세한 오차, 위성의 실제 궤도와 예측 궤도 간의 차이(천체력 오차), 그리고 신호가 지구의 전리층과 대류층을 통과하면서 발생하는 전파 지연 등이 있다.4 이러한 오차들이 누적되면 단독 GNSS 수신기의 위치 정확도는 수 미터 수준에 머무르게 되어, 자율주행이나 정밀 농업과 같은 고정밀 애플리케이션에는 부적합하다.

### 2.2  보정 서비스의 분류

GNSS 오차를 보정하기 위해 다양한 기술이 개발되었으며, 각 기술은 정확도, 수렴 시간, 인프라 요구사항 등에서 뚜렷한 특징을 보인다. TerraStar-X의 기술적 위상을 이해하기 위해 주요 보정 기법들을 비교 분석할 필요가 있다.

#### 2.2.1  기본 방법론 (DGNSS 및 SBAS)

DGNSS(Differential GNSS)와 SBAS(Satellite-Based Augmentation System)는 코드 기반(code-based) 보정 기법의 초기 형태이다. DGNSS는 정확한 위치를 아는 기준국에서 수신한 GNSS 신호의 오차를 계산하여 주변의 이동국(rover)에 전송하는 방식이다. SBAS는 광역의 기준국 망에서 생성된 보정 정보를 정지궤도 위성을 통해 사용자에게 방송하는 시스템이다. 두 방식 모두 미터에서 서브미터 수준의 정확도를 제공하지만, 센티미터급 정밀도를 요구하는 애플리케이션에는 한계가 있다.5

#### 2.2.2  고정밀의 기준: 실시간 이동 측위 (RTK)

RTK는 센티미터 수준 정확도의 전통적인 표준 기술로 간주된다.13 이 기술은 코드 신호뿐만 아니라 위성 신호의 반송파 위상(carrier phase)까지 측정하여 기준국과 이동국 간의 상대 위치를 매우 정밀하게 계산한다. 그러나 RTK는 치명적인 운영상의 제약을 갖는데, 바로 기준국과 이동국이 40 km 이내의 비교적 가까운 거리에 위치해야 한다는 점이다.13 이는 광범위한 지역에서 이동하는 자율주행차나 농기계에 적용하기에는 인프라 구축 및 유지보수에 큰 부담을 준다.

#### 2.2.3  글로벌 접근법: 정밀 단독 측위 (PPP)

PPP는 전 세계에 분포된 기준국 네트워크를 이용하여 위성의 궤도와 시각 오차를 정밀하게 모델링하고, 이 보정 정보를 사용자에게 제공하는 기술이다.5 PPP는 별도의 지역 기준국 없이도 전 세계 어디서나 센티미터 수준의 정확도를 달성할 수 있다는 장점이 있다. 하지만 가장 큰 단점은 수렴 시간(convergence time)이 20분에서 45분까지 매우 길다는 점이다.5 이러한 긴 대기 시간은 즉각적인 작업 시작이 필수적인 동적 애플리케이션의 생산성을 저해하는 주요 요인이었다.

#### 2.2.4  하이브리드 기술: PPP-RTK (SSR)

PPP-RTK는 TerraStar-X의 기술적 근간을 이루는 차세대 보정 기술이다. 이 방식은 PPP의 전 지구적 위성 궤도 및 시각 보정 정보와 RTK와 유사한 지역별 대기층 보정 정보를 결합하여 사용자에게 방송한다.4 이 보정 모델은 상태 공간 표현(State Space Representation, SSR)이라는 형식으로 제공되기도 한다.15 이 하이브리드 접근법은 RTK의 빠른 수렴 속도와 PPP의 광역 서비스 가능성이라는 두 가지 장점을 모두 취함으로써, 기존 기술들의 한계를 극복한다.18

이러한 기술의 발전 과정은 단순히 정확도를 높이는 선형적 진보가 아니라, 시장의 운영 효율성 요구에 대한 전략적 대응의 결과물이다. PPP에서 PPP-RTK로의 전환은 긴 수렴 시간이 경제적 손실이나 안전 문제로 직결되는 동적 애플리케이션(자율주행차, 정밀 농업 등)의 부상에 의해 촉발되었다. 농부가 30분 이상을 대기해야 하거나 5, 자율주행차가 즉시 차선 수준의 위치를 파악하지 못하는 8 상황은 용납될 수 없다. RTK는 시간 문제를 해결했지만, 기지국이라는 물리적 제약을 낳았다.1 이는 빠르면서도 인프라에 덜 의존적인 솔루션에 대한 명확한 시장 수요를 창출했고, TerraStar-X의 기반 기술인 PPP-RTK는 바로 이 특정하고 가치 있는 시장의 문제점을 해결하기 위한 직접적인 기술적 해답이다.

**표 1: GNSS 보정 방법론 비교**

| 특성                   | SBAS               | DGNSS            | RTK                   | PPP              | PPP-RTK (SSR)        |
| ---------------------- | ------------------ | ---------------- | --------------------- | ---------------- | -------------------- |
| **보정 방식**          | 코드 기반          | 코드 기반        | 반송파 위상           | 반송파 위상      | 반송파 위상          |
| **일반적 수평 정확도** | < 1 m              | 0.4 m ~ 1 m      | 1-2 cm                | 2-5 cm           | 2-5 cm               |
| **수렴 시간**          | 즉시               | 즉시             | 수 초 ~ 수 분         | 20-45 분         | < 1 분               |
| **인프라 요구사항**    | 광역 기준국 (무료) | 지역 기준국      | 지역 기준국 (<40 km)  | 글로벌 기준국    | 글로벌 + 지역 기준국 |
| **비용 모델**          | 무료               | 장비/서비스 비용 | 장비/서비스 비용      | 구독료           | 구독료               |
| **주요 사용 사례**     | 일반 항법, 항공    | 매핑, GIS        | 측량, 건설, 정밀 농업 | 해양, 측량, 농업 | 자율주행, 정밀 농업  |

## 3.  NovAtel TerraStar 에코시스템: 정밀도 포트폴리오

### 3.1  TerraStar 글로벌 인프라

모든 TerraStar 서비스는 Hexagon이 소유하고 운영하는 강력한 글로벌 인프라를 기반으로 한다. 이 인프라는 세 가지 핵심 요소로 구성된다.

- **기준국 네트워크:** 전 세계에 전략적으로 배치된 100개 이상의 GNSS 기준국 네트워크가 모든 GNSS 위성군을 24시간 모니터링하며 원시 데이터를 수집한다.2
- **처리 및 제어:** 지리적으로 분리된 3개의 네트워크 제어 센터(NCC)가 수집된 데이터를 처리하여 정밀한 위성 궤도 및 시각 보정 정보를 생성한다. 이러한 이중화 구조는 99.9% 이상의 높은 서비스 가용성을 보장한다.2
- **전송 시스템:** 생성된 보정 정보는 6개 이상의 정지궤도 통신 위성을 통해 전 세계에 L-Band 주파수로 방송된다. 이는 지구상 어디에서든 최소 2개 이상의 위성 신호를 수신(이중 또는 삼중 빔 추적)할 수 있도록 중첩된 커버리지를 제공하며, IP/셀룰러를 통한 전송 옵션도 지원한다.2

### 3.2  차별화된 서비스 등급: 전략적 포트폴리오

NovAtel은 시장의 다양한 요구에 대응하기 위해 여러 등급의 TerraStar 서비스를 제공하는 계층적 전략을 구사한다.

- **TerraStar-L:** 데시미터 수준(40-50 cm)의 정확도를 5분 미만의 수렴 시간으로 제공하는 보급형 서비스이다. 무료 SBAS 서비스보다 월등한 성능을 제공하여 정밀 농업의 기본 작업에 적합하다.5
- **TerraStar-C PRO:** 전문가급 글로벌 서비스로, 센티미터 수준(2.5-3 cm)의 정확도를 제공한다. 최신 펌웨어와 다중 위성군 지원을 통해 수렴 시간이 기존 18분 미만에서 3-5분까지 단축되었다.5 GPS, GLONASS, Galileo, BeiDou 등 모든 주요 GNSS 위성군을 지원하여 까다로운 환경에서도 안정적인 성능을 발휘한다.20
- **TerraStar-X:** C PRO와 동일한 수준의 정확도(수평 2.5 cm 95%)를 제공하지만, 수렴 시간을 1분 미만으로 극적으로 단축시킨 프리미엄 지역 서비스이다.5 이는 TerraStar-C PRO의 글로벌 보정 정보에 지역별 대기층 보정 정보를 추가함으로써 가능하다. 지정된 서비스 지역을 벗어나면 자동으로 TerraStar-C PRO 모드로 전환되어 끊김 없는 서비스를 보장한다.2

### 3.3  핵심 지원 기술

- **NovAtel CORRECT®:** NovAtel 수신기에 내장된 독점적인 측위 알고리즘으로, RTK, PPP, SBAS 등 다양한 소스로부터 수신된 보정 정보를 지능적으로 처리하여 최적의 위치 해를 산출한다.5
- **RTK ASSIST™ / RTK ASSIST PRO™:** 사용자의 주 RTK 보정 링크(예: 셀룰러 통신)가 끊겼을 때, 위성으로 전송되는 TerraStar 보정 정보를 백업으로 사용하여 측위를 유지하는 기능이다. 특히 RTK ASSIST PRO는 무제한으로 백업을 제공하여, RTK 신호가 없는 지역에서는 완전한 PPP 솔루션으로 작동한다.20

이러한 계층화된 서비스 포트폴리오는 단순한 제품 라인업이 아니라, 정교하게 설계된 고객 확보 및 수익화 전략이다. NovAtel은 TerraStar-L을 통해 다양한 가격대와 성능 요구를 가진 사용자를 유입시키고, 고객의 운영 요구가 정밀해짐에 따라 C PRO, 그리고 최종적으로 X로 업그레이드할 수 있는 명확한 경로를 제공한다. 예를 들어, GPS 7500 수신기를 구매한 농업 사용자는 1년간 무료로 제공되는 TerraStar-L로 시작할 수 있다.7 파종과 같이 더 높은 정밀도가 필요해지면 TerraStar-C PRO 구독으로 전환하고, 수렴 시간을 기다릴 여유가 없는 경우 지원 지역 내에서 TerraStar-X로 업그레이드할 수 있다.20 또한 RTK ASSIST PRO는 기존 RTK 사용자에게도 강력한 '보험' 역할을 하여, RTK 통신 두절에 대한 불안을 해소하고 이들을 NovAtel의 구독 생태계로 끌어들인다.24

**표 2: NovAtel TerraStar 서비스 포트폴리오 비교**

| 구분                    | TerraStar-L   | TerraStar-C PRO         | TerraStar-X         |
| ----------------------- | ------------- | ----------------------- | ------------------- |
| **수평 정확도 (95%)**   | 50 cm         | 2.5 cm                  | 2.5 cm              |
| **수직 정확도 (95%)**   | 75 cm         | 5 cm                    | 5 cm                |
| **패스간 정확도 (95%)** | 15 cm         | 2 cm                    | 2 cm                |
| **수렴 시간**           | < 5 분        | 3 분 (최신 펌웨어 기준) | < 1 분              |
| **지원 위성군**         | GPS, GLO      | GPS, GLO, GAL, BDS      | GPS, GLO            |
| **서비스 지역**         | 글로벌        | 글로벌                  | 지역                |
| **지원 플랫폼**         | OEM7, OEM6    | OEM7                    | OEM7                |
| **주요 적용 분야**      | 기본 가이던스 | 정밀 농업, 측량         | 자율주행, 정밀 농업 |

자료 출처: 5

## 4.  심층 분석: NovAtel TerraStar-X의 아키텍처와 성능

### 4.1  기술적 프레임워크: "하늘에서 직접 수신하는 RTK"의 해부

#### 4.1.1  글로벌과 지역 데이터의 융합

TerraStar-X의 핵심 기술은 전 지구적 데이터와 지역적 데이터를 융합하는 데 있다. 기본적으로 TerraStar-C PRO가 제공하는 전 지구 위성 시각 및 궤도 보정 정보를 활용한다. 여기에 Hexagon이 보유한 4,500개 이상의 촘촘한 HxGN SmartNet 기준국 네트워크로부터 수집된 지역별 전리층 보정 데이터를 추가한다.6 바로 이 지역화된 대기층 정보가 수신기로 하여금 반송파 위상의 미지정수(ambiguity)를 신속하게 결정할 수 있게 하여, 거의 즉각적인 수렴을 가능하게 하는 핵심 요소이다.18

#### 4.1.2  전송 채널과 이중화

주요 마케팅은 셀룰러 통신망 없이 L-Band 위성을 통해 보정 정보를 수신하는 편리함을 강조하지만 1, 기술적으로는 셀룰러/IP를 통한 하이브리드 전송도 지원한다.6 특히 대량의 차량에 서비스를 제공해야 하는 TerraStar-X Enterprise 모델은 IP 전송에 의존하여 확장성을 확보한다.8 이러한 유연성은 다양한 시장 요구에 대응하는 데 매우 중요하다.

### 4.2  성능 사양 및 실제 벤치마크

#### 4.2.1  정확도 분석

TerraStar-X의 공식적인 정확도 사양은 다음과 같다:

- **수평 정확도:** 2 cm (RMS) / 2.5 cm (95%) 5
- **수직 정확도:** 5 cm (RMS) 5
- **패스간 정확도(Pass-to-Pass Accuracy):** 2 cm (sub-inch). 이는 파종이나 스트립 틸과 같이 반복적인 경로 주행이 중요한 농업 분야에서 핵심적인 지표이다.1

#### 4.2.2  수렴, 재수렴 및 지연 시간

- **초기 수렴:** 60초 미만이라는 획기적인 수렴 시간은 이 서비스의 가장 큰 특징이다.2 일부 연구에서는 이상적인 조건 하에서 수렴 시간이 "수 초"에 불과할 수 있다고 보고한다.18
- **재수렴:** 다리 밑을 통과하는 등 일시적인 신호 단절 후에는 즉각적인 재수렴이 가능하다.3 이는 고속의 보정 정보 업데이트 덕분이며, 일반적으로 60초 이내에 완전한 성능을 회복한다.5
- **지연 시간 및 업데이트 속도:** 보정 정보 스트림의 구체적인 지연 시간(latency) 값은 명시적으로 제공되지 않지만, 서비스는 "높은 전송률과 낮은 지연 시간의 보정(high-rate, low-latency corrections)"을 제공한다고 설명된다.20 기반이 되는 OEM7 수신기는 최대 100 Hz의 위치 업데이트 속도를 지원하므로 29, 보정 스트림은 이러한 고주파 위치 출력을 저해하지 않을 만큼 충분히 빠르게 설계되었음을 추론할 수 있다. 이는 차량 제어 루프와 같은 실시간 시스템에 매우 중요하다.

#### 4.2.3  시스템 가용성 및 무결성

99.9% 이상의 가용성 및 가동 시간은 이중화된 네트워크 제어 센터와 중첩된 위성 커버리지를 통해 보장된다.4 또한, 기준국에서부터 최종 사용자인 차량에 이르기까지 전 과정에 걸친 무결성(integrity) 모니터링은 자율주행과 같은 안전이 중요한 애플리케이션에 필수적인 기능이다.4

### 4.3  시스템 통합 요구사항

#### 4.3.1  호환 하드웨어

TerraStar-X 서비스를 이용하기 위해서는 NovAtel의 OEM7 제품군 수신기(예: OEM7700, OEM719, OEM729, PwrPak7, SMART7)가 필요하다.3 위성을 통해 보정 정보를 수신하려면 L-Band 수신이 가능한 GNSS 안테나 또한 필수적이다.31

#### 4.3.2  펌웨어 및 모델 전제 조건

특정 버전 이상의 펌웨어가 요구된다. TerraStar-X의 경우, 펌웨어 버전 7.06.03 이상이 필요하며, 최상의 성능을 위해서는 7.08.10 이상이 권장된다.31 더 중요한 요구사항은 수신기의 '모델' 또는 라이선스이다. TerraStar-X를 사용하려면 수신기에 'R' 모델(예: DDN-R 또는 FFN-R)이 활성화되어 있어야 하며, 이는 RTK 기능 라이선스에 해당한다. 이 'RTK 언락(unlock)'은 해당 기능을 활성화하는 소프트웨어 키이다.1

TerraStar-X 사용을 위해 'RTK 언락'이 필요하다는 점은 매우 중요한 전략적 선택을 시사한다.1 이는 TerraStar-X를 단순히 빠른 PPP 서비스가 아닌, RTK의 한 *가지 방식*으로 포지셔닝하려는 의도를 보여준다. 기술적으로 볼 때, 수신기는 RTK와 동일한 처리 엔진을 사용하되, 보정 정보를 지역 기지국이 아닌 '하늘에서' 수신하는 것이다. 사업적 관점에서 이는 NovAtel이 RTK 기능에 걸맞은 프리미엄 가격을 유지하고 제품 라이선스 모델을 단순화할 수 있게 한다. 즉, TerraStar-X는 기존의 고가 RTK 하드웨어 시장을 잠식하는 대신, RTK의 가치를 확장하는 새로운 서비스로 자리매김한다. 이는 기존 제품 프레임워크 내에서 신기술을 포장하고 가격을 책정하는 영리한 방법이다.

**표 3: TerraStar-X 시스템 통합 요구사항 체크리스트**

| 항목                   | 요구사항                                    | 관련 자료 |
| ---------------------- | ------------------------------------------- | --------- |
| **호환 수신기 제품군** | NovAtel OEM7                                | 3         |
| **구체적 수신기 모델** | OEM7700, OEM719, OEM729, PwrPak7, SMART7 등 | 3         |
| **최소 펌웨어 버전**   | 7.06.03                                     | 31        |
| **권장 펌웨어 버전**   | 7.08.10 이상                                | 31        |
| **안테나 요구사항**    | L-Band 수신 가능 GNSS 안테나                | 31        |
| **필수 펌웨어 모델**   | 'R' 모델 디자인 (RTK 기능 언락 필요)        | 1         |

## 5.  TerraStar-X Enterprise: 글로벌 배포와 대한민국 사례 연구

### 5.1  TerraStar-X Enterprise의 정의

'Enterprise' 서비스는 표준 TerraStar-X와 몇 가지 중요한 차이점을 가진다. 이 서비스는 대규모 프로그램을 위해 특별히 설계되었다.

- **핵심 특징:** TerraStar-X Enterprise는 특정 하드웨어에 종속되지 않으며(hardware-agnostic), 모든 차량의 전자 제어 장치(ECU)에 원활하게 통합되도록 설계되었다. 보정 정보는 주로 저대역폭의 셀룰러/IP 통신을 통해 전달된다.4
- **기능 안전:** 가장 결정적인 차별점은 자동차 기능 안전 표준인 ISO 26262 ASIL B를 준수한다는 것이다. 이는 ADAS 및 AD와 같은 안전이 중요한(safety-critical) 애플리케이션에 이 서비스를 사용할 수 있게 하는 핵심 요건이다.4

### 5.2  대한민국의 전략적 확장

Hexagon의 글로벌 확장 야망을 보여주는 대표적인 사례는 대한민국 시장 진출이다.

- **Hexagon-MBC 파트너십:** 2023년 10월, Hexagon의 오토노미 & 포지셔닝 사업부와 MBC는 TerraStar-X Enterprise 서비스를 한국에 도입하기 위한 협약을 발표했다.9
- **시너지 효과:** 이 파트너십은 MBC가 이미 전국적으로 구축한 RTK 네트워크를 활용하여 서비스에 필요한 촘촘한 대기층 데이터를 확보하는 방식이다. 이는 Hexagon이 한국 지역에서 네트워크의 이중화와 신뢰성을 강화하고 신속하게 서비스를 출시할 수 있게 한다.9 한국의 RTK 측위 분야를 선도해온 MBC는 글로벌 기술 리더와의 협력을 통해 자사의 인프라를 더욱 혁신하고 새로운 수익을 창출할 기회를 얻게 된다.9
- **한국 시장에 대한 시사점:** 고성능, 기능 안전을 갖춘 보정 서비스의 등장은 한국의 첨단 기술 분야에서 ADAS, 자율주행차, 마이크로 모빌리티 및 산업용 애플리케이션 개발을 가속화할 것으로 기대된다.9 이 서비스는 현재 한국 전역에서 테스트가 가능하다.35

### 5.3  아시아 태평양 및 글로벌 출시

한국과의 파트너십은 더 넓은 글로벌 전략의 일환이다. 일본(히타치 조선과의 파트너십), 중국, 유럽, 북미 등 주요 자동차 시장에도 테스트베드가 구축되었다.10 이는 자동차 OEM 및 기타 대규모 사용자를 위해 전 세계적으로 일관된 고성능 측위 서비스를 제공하려는 명확한 로드맵을 보여준다.10

'TerraStar-X Enterprise'의 탄생과 파트너십 기반의 글로벌 확장은 Hexagon의 비즈니스 모델이 하드웨어 중심(수신기 판매)에서 확장 가능한 구독 기반 서비스 제공업체로 전환되고 있음을 의미한다. 이러한 변화는 거대한 자동차 시장을 공략하기 위한 필수적인 전략이다. 표준 TerraStar-X는 NovAtel의 OEM7 하드웨어에 연동되지만 31, 자동차 산업은 다양한 ECU와 GNSS 칩셋을 사용한다. 이 시장에 침투하기 위해서는 하드웨어에 구애받지 않는 솔루션이 필요하다.4 따라서 'Enterprise'를 IP 기반의 안전 인증 서비스로 별도 출시한 것이다.8 MBC나 히타치 조선과 같은 현지 파트너와의 협력 모델은 막대한 자본 투자 없이도 필요한 지역 네트워크 밀도를 신속하게 구축하고 현지 전문성을 활용할 수 있는 유일하고 효율적인 방법이다. 이는 '제품' 판매에서 '서비스형 데이터(Data-as-a-Service)' 판매로의 전략적 전환이며, 자율성의 미래에 훨씬 더 확장 가능하고 수익성 높은 모델이다.

## 6.  주요 적용 분야 분석

### 6.1  자동차 및 자율주행차 (AD/ADAS)

- **필요성:** 자율 시스템은 안전한 항법과 운행을 위해 지속적이고 신뢰할 수 있는 차선 수준의 위치 정보를 요구한다.4
- **솔루션:** TerraStar-X Enterprise는 이러한 요구를 충족시키는 즉각적인 센티미터 수준의 정확도와 99.9% 이상의 높은 가용성을 제공한다.4 특히 ISO 26262 기능 안전 인증과 종단 간 무결성 모니터링은 생명과 직결된 이 애플리케이션에 매우 중요하다.4 테스트 결과, 표준 SBAS 솔루션 대비 64-93%의 수평 정확도 개선을 보였다.6

### 6.2  정밀 농업

- **필요성:** 현대 농업은 수확량을 극대화하고 연료, 비료 등 투입 비용을 절감하기 위해 자동 조향, 파종, 스트립 틸, 분무와 같은 작업에서 정밀한 위치 정보에 의존한다.7
- **솔루션:** TerraStar-X는 이러한 작업에 필수적인 2 cm(sub-inch) 패스간 정확도를 제공한다.7 가장 큰 장점은 RTK 수준의 성능을 지역 RTK 기지국 설치 및 유지보수에 드는 비용과 번거로움 없이 제공한다는 점이다.1 또한 1분 미만의 빠른 수렴 시간은 중요한 파종 및 수확 시기에 가동 중단 시간을 최소화하여 생산성을 극대화한다.19

### 6.3  기타 및 신흥 애플리케이션

높은 정확도, 이동성, 빠른 수렴 시간의 조합은 TerraStar-X를 다양한 분야에 적합하게 만든다. 여기에는 무인 항공기(UAV) 및 드론, 산업용 로봇, 토지 측량, 건설, 해양 작업 등이 포함된다.5 특히 RTK 네트워크 커버리지가 없는 지역에서 작업하는 측량사에게는 무제한 'RTK ASSIST PRO' 솔루션을 제공하여 작업의 연속성을 보장한다.5

## 7.  경쟁 환경 및 시장 포지셔닝

### 7.1  Trimble CenterPoint RTX

- **비교:** Trimble의 프리미엄 서비스인 CenterPoint RTX Fast는 1분 미만의 수렴 시간과 2 cm 수평 정확도 등 TerraStar-X와 매우 유사한 성능 사양을 제공하는 직접적인 경쟁자이다.40
- **기술:** TerraStar-X와 마찬가지로 글로벌 기준국 네트워크를 사용하며 위성과 IP를 통해 보정 정보를 방송한다.41 최근에는 BeiDou-III 위성군을 활용하여 수렴 시간을 단축했다.40
- **시장 초점:** 농업 및 건설 분야에서 강점을 보이며, 자동차 시장도 적극적으로 공략하고 있다 (예: GM의 SuperCruise는 Trimble RTX 사용).42 이 분야의 경쟁은 생태계, 지역 커버리지 밀도, 상업적 파트너십의 싸움이다.

### 7.2  John Deere StarFire™ (SF-RTK)

- **비교:** John Deere의 SF-RTK 신호는 StarFire 7500 수신기를 통해 2.5 cm 패스간 정확도를 제공하며, 대폭 개선된 수렴 시간으로 '하늘에서 받는' RTK급 솔루션을 제공한다.43
- **전략:** 이는 수직적으로 통합된 경쟁자이다. StarFire 시스템은 John Deere의 트랙터, 콤바인, 디스플레이 등 자체 생태계에 깊숙이 통합되어 있다.43
- **차별점:** 농업 사용자의 선택은 종종 개방형 생태계 접근 방식(예: Ag Leader 디스플레이와 NovAtel 수신기를 사용한 TerraStar-X)과 폐쇄적인 단일 공급업체 John Deere 생태계 사이에서 이루어진다.46 TerraStar-X의 장점은 여러 장비 브랜드와의 호환성이다.

### 7.3  기타 시장 참여자

- **Topcon:** Starpoint Pro(PPP, 3-6 cm 정확도, <5분 수렴)와 Realpoint(네트워크 RTK)를 포함하는 Topnet Live 포트폴리오를 제공한다.48 Skybridge 서비스는 RTK ASSIST와 유사하게 RTK 공백을 메우는 역할을 한다.
- **신흥 주자:** Point One Navigation과 같은 회사들은 Polaris와 같은 현대적이고 개발자 친화적인 RTK 네트워크로 시장에 진입하고 있다. 이들은 통합의 용이성과 투명한 가격 정책을 강조하며 로봇 및 자동차 스타트업을 공략한다.51

### 7.4  NovAtel의 전략적 차별점

- **종단 간 소유권:** 자체 네트워크, 처리, 하드웨어, 펌웨어의 긴밀한 통합은 심층적인 최적화와 종단 간 무결성 모니터링을 가능하게 한다.4
- **기능 안전:** TerraStar-X Enterprise의 ISO 26262 인증은 안전이 중요한 자동차 시장에서 강력한 경쟁 우위를 제공한다.8
- **파트너십 모델:** 신속한 확장을 위해 기존 지역 RTK 제공업체(예: MBC)와 협력하는 전략은 민첩하고 자본 효율적인 글로벌 배포 방식이다.9

경쟁 환경은 양분화되고 있다. 한쪽에는 글로벌 규모, 신뢰성, 깊은 수직 통합을 기반으로 경쟁하는 기존의 거대 기업들(Hexagon/NovAtel, Trimble, Topcon)이 있다. 다른 한쪽에는 개발자 경험, API 접근성, 파괴적인 비즈니스 모델을 무기로 경쟁하는 새롭고 민첩한 기업들(Point One, Swift Navigation)이 있다. Trimble과 John Deere의 최상위 서비스 사양이 TerraStar-X와 거의 동일하다는 점은 41 이 기술 분야가 성숙기에 접어들었으며, 이제 차별화는 다른 곳에서 이루어져야 함을 시사한다. Hexagon/NovAtel은 기능 안전 8과 종단 간 무결성 4에, John Deere는 자사의 농업 생태계 43에 베팅하고 있다. 한편, Point One은 '현대적인 사용자 인터페이스', '오픈 소스 API', '단일 NTRIP 마운트 포인트'와 같은 단순함을 내세워 기존 기업들의 복잡성과 불투명한 가격 정책에 불만을 가진 새로운 고객층(개발자, 스타트업)에게 어필하고 있다.51 따라서 NovAtel은 규모와 신뢰성 측면에서 기존 경쟁사와, 사용 편의성과 비즈니스 모델 혁신 측면에서 신규 경쟁사와 양면 전쟁을 치러야 하는 상황이다.

**표 4: PPP-RTK 보정 서비스 경쟁사 비교**

| 구분                 | NovAtel TerraStar-X                                  | Trimble CenterPoint RTX                 | John Deere SF-RTK       |
| -------------------- | ---------------------------------------------------- | --------------------------------------- | ----------------------- |
| **서비스 제공사**    | Hexagon / NovAtel                                    | Trimble                                 | John Deere              |
| **공시 수평 정확도** | 2.5 cm (95%)                                         | 2 cm (RMS)                              | 2.5 cm                  |
| **공시 수렴 시간**   | < 1 분                                               | < 1 분                                  | 73% 단축 (SF3 대비)     |
| **전송 방식**        | 위성 (L-Band), IP                                    | 위성 (L-Band), IP                       | 위성                    |
| **커버리지 전략**    | 지역별 확장 (글로벌 C PRO 백업)                      | 글로벌 네트워크 기반 지역별 Fast 서비스 | 글로벌                  |
| **핵심 차별화 전략** | 기능 안전 (ISO 26262), 종단 간 무결성, 하드웨어 연계 | 글로벌 네트워크, 농업/건설 생태계       | 농기계 수직 통합 생태계 |

자료 출처: 5

## 8.  전략적 평가 및 권장 사항

### 8.1  강점, 약점, 기회 분석 (SWOT)

- **강점(Strengths):** 기지국 없는 RTK급 성능, 1분 미만의 빠른 수렴, 신뢰성을 보장하는 강력한 수직 통합, 자동차 시장을 위한 명확한 기능 안전 경로.
- **약점(Weaknesses):** 최고 성능 등급의 지역적 커버리지 한계(확장 중), 표준 서비스의 OEM7 하드웨어 종속성, 유사 사양을 가진 기존 경쟁사.
- **기회(Opportunities):** ADAS/AD 시장의 폭발적 성장, 파트너십을 통한 신규 지역 확장, 기존 TerraStar-C PRO 사용자 기반의 업셀링, 기능 안전을 핵심 경쟁 우위로 활용.

### 8.2  미래 전망

TerraStar-X와 같은 PPP-RTK 서비스는 고정밀 모바일 애플리케이션의 새로운 표준이 되고 있다. 미래에는 서비스 커버리지의 지속적인 확장, 수렴 시간의 추가 단축, 그리고 완전 자율 운행에 대한 신뢰를 구축하기 위한 무결성 및 보안 기능(예: GNSS 인증)의 심층적인 통합이 이루어질 것으로 전망된다.

### 8.3  잠재적 도입자를 위한 권장 사항

- **자동차/ADAS 개발자:** TerraStar-X Enterprise는 성능과 기능 안전 준수 측면에서 최고의 후보 중 하나이다. 핵심 평가 요소는 주요 개발 및 배포 지역(북미, 유럽, 한국, 일본, 중국 등)에서의 서비스 성숙도와 네트워크 밀도가 될 것이다.
- **정밀 농업 사용자:** 여러 브랜드의 장비를 혼용하거나 John Deere 이외의 장비를 사용하는 경우, TerraStar-X는 RTK 기지국에 대한 강력한 대안을 제공한다. 결정은 구독료와 RTK 인프라 구축 및 운영 비용 간의 비용-편익 분석에 따라 달라질 것이다.
- **일반 권장 사항:** 잠재 사용자는 자신의 특정 운영 지역에서 서비스 커버리지를 반드시 확인하고, 올바른 하드웨어(OEM7 'R' 모델)와 펌웨어를 갖추었는지 확인해야 한다. 장기 구독을 결정하기 전에, 여러 공급업체가 제공하는 시험 구독 7을 통해 목표 환경에서 성능을 직접 검증하는 것이 강력히 권장된다.

#### **참고 자료**

1. TerraStar X for GPS7500 - Ag Leader, accessed July 17, 2025, https://portal.agleader.com/community/s/article/2262
2. Terrastar X by Novatel's correction service - WALDYTECH, accessed July 17, 2025, https://www.waldytech.com/?CategoryID=153&ArticleID=175
3. TerraStar-X Services - NovAtel, accessed July 17, 2025, https://novatel.com/products/gps-gnss-correction-services/terrastar-correction-services/terrastar-x-services
4. Hexagon Autonomy & Positioning - Automotive Solutions - TerraStar-X - NovAtel, accessed July 17, 2025, https://hexagonautomotive.novatel.com/en/solutions/terrastar-x
5. TerraStar-X - NavtechGPS, accessed July 17, 2025, https://www.navtechgps.com/terrastar-x-global-positioning/
6. TerraStar X Technology, RTK From the Sky™ | NovAtel, accessed July 17, 2025, https://novatel.com/tech-talk/papers/terrastar-x-technology-rtk-from-the-sky
7. What is TerraStar? - Ag Leader, accessed July 17, 2025, https://portal.agleader.com/community/s/article/whatisterrastar
8. TerraStar-X Enterprise GNSS Correction Services - NovAtel, accessed July 17, 2025, https://novatel.com/products/automotive-solutions/terrastar-x-technology
9. 헥사곤 (Hexagon), 테라스타-X (TerraStar-X) 정밀측위 GNSS 보정 ..., accessed July 17, 2025, https://novatel.com/about-us/news-releases/hexagon-expands-terrastar-x-precise-positioning-gnss-correction-service-to-south-korea-kr
10. Hexagon expands TerraStar-X precise positioning GNSS correction service to South Korea, accessed July 17, 2025, https://hexagon.com/company/newsroom/press-releases/2023/hexagon-expands-terrastar-x-precise-positioning-gnss-correction-service-to-south-korea
11. 헥사곤, 테라스타-X 정밀측위 GNSS 보정 서비스 국내 확대, accessed July 17, 2025, https://www.autoelectronics.co.kr/article/articleView.asp?idx=5325
12. How TerraStar Works, accessed July 17, 2025, https://terrastar.net/services/how-terrastar-works
13. Which correction method? | NovAtel, accessed July 17, 2025, https://novatel.com/an-introduction-to-gnss/resolving-errors/which-correction-method
14. What are the Different GNSS Correction Methods? | Swiftnav, accessed July 17, 2025, https://www.swiftnav.com/resource/blog/what-are-the-different-gnss-correction-methods
15. GNSS Corrections Demystified - Septentrio, accessed July 17, 2025, https://www.septentrio.com/en/learn-more/insights/gnss-corrections-demystified
16. NovAtel CORRECT - NavtechGPS, accessed July 17, 2025, https://www.navtechgps.com/wp-content/uploads/NovAtelCORRECT-Brochure-1.pdf
17. GNSS Corrections Methods & When to Use Them (Plus Top GNSS Correction Services) - pointonenav.com, accessed July 17, 2025, https://pointonenav.com/news/gnss-corrections/
18. TerraStar X: Precise Point Positioning with Fast Convergence and Integrity, accessed July 17, 2025, https://www.ion.org/publications/abstract.cfm?articleID=17092
19. Precision Agriculture - TerraStar, accessed July 17, 2025, https://terrastar.net/industries/precision-agriculture
20. GLOBAL CORRECTION SERVICES - Canal Geomatics, accessed July 17, 2025, https://canalgeomatics.com/wp-content/uploads/2019/12/terrastar-brochure.pdf
21. TerraStar Correction Services - NovAtel, accessed July 17, 2025, https://novatel.com/products/gps-gnss-correction-services/terrastar-correction-services
22. NovAtel Launches New TerraStar-C PRO Multi-Constellation Correction Service - Inside GNSS - Global Navigation Satellite Systems Engineering, Policy, and Design, accessed July 17, 2025, https://insidegnss.com/novatel-launches-new-terrastar-c-pro-multi-constellation-correction-service/
23. RTK From the Sky™ Technology Transforms Hexagon | NovAtel TerraStar-C PRO Service With Three Minute Global Convergence - Inside Autonomous Vehicles, accessed July 17, 2025, https://insideautonomousvehicles.com/rtk-from-the-sky-technology-transforms-hexagon-novatel-terrastar-c-pro-service-with-three-minute-global-convergence/
24. TerraStar for Precision Agriculture, accessed July 17, 2025, https://terrastar.net/services/terrastar-service-options/precision-agriculture
25. TerraStar Land and Airborne Industries Service Options, accessed July 17, 2025, https://terrastar.net/services/terrastar-service-options/land-airborne
26. TerraStar Correction Services - gnssystems, accessed July 17, 2025, https://gnssystems.se/products/terrastar-correction-services
27. TerraStar X: Precise Point Positioning with Fast Convergence and Integrity - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/336451886_TerraStar_X_Precise_Point_Positioning_with_Fast_Convergence_and_Integrity
28. How TerraStar-X™ Works - YouTube, accessed July 17, 2025, https://www.youtube.com/watch?v=hisJ8I_Zztg
29. OEM729 Product Sheet - NET, accessed July 17, 2025, https://hexagondownloads.blob.core.windows.net/public/Novatel/assets/Documents/Papers/OEM729-Product-Sheet/OEM729-Product-Sheet.pdf
30. OEM719 Performance Specifications - NovAtel Documentation Portal, accessed July 17, 2025, https://docs.novatel.com/oem7/Content/Technical_Specs_Receiver/OEM719_Performance_Specs.htm
31. Application Note - TerraStar on OEM7 - NET, accessed July 17, 2025, https://hexagondownloads.blob.core.windows.net/public/Novatel/assets/Documents/Papers/APN-087-Advanced-TerraStar-Services/APN-087-Advanced-TerraStar-Services.pdf
32. Enabling TerraStar Correction Services - NovAtel Documentation Portal, accessed July 17, 2025, [https://docs.novatel.com/OEM7/Content/Operation/Enable_TerraStar_SMART7.htm?TocPath=Operation%7CSMART7%20Operation%7C_____8](https://docs.novatel.com/OEM7/Content/Operation/Enable_TerraStar_SMART7.htm?TocPath=Operation|SMART7+Operation|_____8)
33. 헥사곤, 테라스타-X 정밀측위 GNSS 보정서비스 국내 확대 - 뉴스와이어, accessed July 17, 2025, https://www.newswire.co.kr/newsRead.php?no=976038
34. Hexagon Expands TerraStar-X Precise Positioning GNSS Correction Service to South Korea, accessed July 17, 2025, https://insidegnss.com/hexagon-expands-terrastar-x-precise-positioning-gnss-correction-service-to-south-korea/
35. 헥사곤 (Hexagon), 테라스타-X (TerraStar-X) 정밀측위 GNSS 보정서비스 국내 확대, accessed July 17, 2025, https://hexagon.com/ko/company/newsroom/press-releases/2023/hexagon-expands-terrastar-x-precise-positioning-gnss-correction-service-to-south-korea
36. Hexagon and Hitachi Zosen sign agreement to provide TerraStar-X Enterprise corrections in Japan, accessed July 17, 2025, https://hexagon.com/company/newsroom/press-releases/2023/hexagon-hitachi-zosen-sign-agreement-to-provide-terrastar-x-enterprise-corrections-in-japan
37. What is TerraStar-X™ "RTK From the Sky™" Corrections for Precision Agriculture | Hexagon | NovAtel - YouTube, accessed July 17, 2025, https://www.youtube.com/watch?v=V7bxVJC8YNM
38. Autonomous Robotics - TerraStar, accessed July 17, 2025, https://terrastar.net/industries/robotic-autonomous-vehicles
39. TerraStar-X Correction Service - Geo-matching, accessed July 17, 2025, https://geo-matching.com/products/terrastar-x-correction-service
40. Upgrade to Trimble CenterPoint RTX Makes RTK-Level Accuracy Easy - Strip-Till Farmer, accessed July 17, 2025, https://www.striptillfarmer.com/articles/4050-upgrade-to-trimbles-centerpoint-rtx-makes-convergence-time-75-faster
41. Are Satellite-Based Correction Services the "Next Utility"? - The American Surveyor, accessed July 17, 2025, https://amerisurv.com/2021/06/20/are-satellite-based-correction-services-the-next-utility/
42. Can anyone recommend INS/GNSS please? : r/SelfDrivingCars - Reddit, accessed July 17, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/ew1i2y/can_anyone_recommend_insgnss_please/
43. Guidance | StarFire™ 7500 with SF-RTK | John Deere US, accessed July 17, 2025, https://www.deere.com/en/technology-products/precision-ag-technology/guidance/starfire-7500-receiver/
44. Riechmann Bros News, accessed July 17, 2025, https://www.riechmannbros.com/news?blogid=29
45. Why Choose a John Deere StarFire™ Receiver as a Precision Agriculture Solution?, accessed July 17, 2025, https://blog.koenigequipment.com/why-choose-john-deere-starfiretm-receiver-precision-agriculture-solution
46. Ag Leader TerraStar-X - Youngblut Ag, Inc., accessed July 17, 2025, https://youngblutag.com/product/ag-leader-terrastar-x/
47. Ag Leader's New DualTrac featuring NovAtel's TerraStar-X Provides RTK Level Accuracy, accessed July 17, 2025, https://www.youtube.com/watch?v=3Dq4VUxvlKw
48. Topnet Live correction services for surveying and construction - Topcon Positioning Systems, accessed July 17, 2025, https://www.topconpositioning.com/us/en/solutions/technology/infrastructure-software-and-services/gnss-correction-services
49. Topnet Live - GNSS positioning correction service - myTopcon, accessed July 17, 2025, https://mytopcon.topconpositioning.com/na/office-software-and-services/gnss-correction-services/topnet-live
50. Infrastructure projects need GNSS correction services | Topcon Positioning - myTopcon, accessed July 17, 2025, https://mytopcon.topconpositioning.com/na/office-software-and-services/gnss-correction-services/correction-services-infrastructure
51. NovAtel Review and Top GNSS Correction Service Alternatives [2024], accessed July 17, 2025, https://pointonenav.com/news/novatel-review/
52. HxGN SmartNet: Complete Review & Top HxGN Alternatives [2024], accessed July 17, 2025, https://pointonenav.com/news/smartnet-review-hxgn-alternatives/
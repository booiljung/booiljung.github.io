# Xsens MTi-3 시리즈 AHRS 모듈

- MTI-3-DK
- MTi-3-5A-DK
- MTi-3-0I-T
- MTi-3-5A-T



Xsens MTi 1-시리즈는 초소형(12.1 x 12.1 mm), 저전력, 산업용 등급의 표면 실장(Surface Mountable Device, SMD) 모션 트래킹 솔루션 포트폴리오이다.1 이 시리즈는 크기, 무게, 전력 소비(Size, Weight, and Power, SWaP)가 핵심적인 제약 조건으로 작용하는 다양한 임베디드 애플리케이션에 최적화되어 설계되었다.3 자율 이동 로봇(AMR), 무인 항공기(UAV), 플랫폼 안정화, 해양 부표, 정밀 농업 등 광범위한 분야에서 활용된다.3

이 시리즈 내에서 MTi-3 모듈은 AHRS(Attitude and Heading Reference System) 기능을 담당한다. AHRS는 관성 측정 장치(Inertial Measurement Unit, IMU)의 기본 구성 요소인 3축 가속도계, 3축 자이로스코프, 3축 지자기센서의 원시 데이터를 입력받아, 내장된 고도의 센서 퓨전 알고리즘을 통해 이를 통합 처리하는 시스템이다.5 그 결과, 외부 환경의 동적인 움직임과 자기장 왜곡 속에서도 안정적이고 신뢰도 높은 3D 자세(Roll, Pitch) 및 절대 방위각(True North-referenced Yaw) 정보를 실시간으로 제공한다. MTi-3는 이러한 복잡한 기능을 별도의 외부 프로세서 없이 단일 모듈 내에서 모두 처리하는 완전한(self-contained) 솔루션으로, 시스템 설계자가 개별 센서를 통합하고 복잡한 칼만 필터 알고리즘을 직접 개발 및 유지보수해야 하는 부담을 덜어준다.1


본 보고서는 Xsens MTi-3 제품군에 속하는 네 가지 특정 파트 넘버, 즉 `MTI-3-DK`, `MTi-3-5A-DK`, `MTi-3-0I-T`, `MTi-3-5A-T`에 대한 심층 비교 분석을 제공하는 것을 목표로 한다.

본 분석은 단순한 데이터시트 사양의 나열을 넘어, 다음과 같은 핵심적인 질문에 대한 해답을 제공하고자 한다.

1. 제품명에 포함된 접미사(`-DK`, `-T`)와 코어 모델 식별자(`-0I`, `-5A`)는 각각 무엇을 의미하며, 이는 제품의 용도와 세대를 어떻게 구분하는가?
2. 단종된 구형 모델과 이를 대체하는 신형 모델 사이에는 어떠한 기술적 차이가 존재하며, 이는 하드웨어 버전의 진화와 어떻게 연결되는가?
3. 개발 키트(Development Kit)와 양산용 OEM 모듈은 구성과 목적에서 어떠한 차이를 보이며, 각 단계에서 엔지니어는 무엇을 고려해야 하는가?

이러한 분석을 통해, 연구 개발자 및 시스템 통합 엔지니어가 각 제품의 특성과 차이점을 명확히 이해하고, 자신의 애플리케이션에 가장 적합한 모델을 선정하며, 특히 구형 모델에서 신형 모델로의 마이그레이션 과정에서 발생할 수 있는 잠재적 문제를 사전에 파악하고 대비할 수 있도록 실질적이고 깊이 있는 기술 정보를 제공하는 것을 최종 목적으로 한다.


Xsens MTi-3 시리즈의 파트 넘버는 제품의 형태, 패키징, 그리고 세대를 구분하는 체계적인 정보를 담고 있다. 이를 정확히 이해하는 것은 올바른 제품 선택의 첫걸음이다.


파트 넘버의 마지막에 붙는 접미사는 제품의 제공 형태와 주된 사용 단계를 나타낸다.


`-DK`는 개발 키트(Development Kit)를 의미하며, 사용자가 제품을 처음 도입하거나 시스템에 본격적으로 통합하기 전, 성능을 평가하고 초기 프로토타입을 제작하는 R&D 단계를 위해 고안된 패키지이다.10 Xsens는 MTi-3 모듈을 처음 사용하는 경우, 개발 키트로 시작할 것을 공식적으로 권장하고 있다.3

- **구성품:** 개발 키트는 일반적으로 다음과 같은 요소들로 구성된다.6
  - **실드 보드 (Shield Board):** MTi-3 모듈이 PLCC-28 소켓에 장착된 형태의 소형 PCB이다. 이 보드에는 USB-to-UART 브릿지 회로(예: FTDI FT230X)와 전원 레귤레이터 등이 포함되어 있어, 사용자가 별도의 하드웨어 회로를 구성할 필요 없이 제공된 USB 케이블만으로 PC와 즉시 연결하여 통신할 수 있다.11
  - **USB 케이블:** PC와 실드 보드를 연결하기 위한 표준 케이블이다.
  - **MT Software Suite:** Xsens가 제공하는 강력한 소프트웨어 도구 모음으로, 데이터 시각화 및 로깅을 위한 GUI 프로그램인 `MT Manager`, 사용자 애플리케이션 개발을 위한 `MT SDK`, 그리고 자기장 교정을 위한 `Magnetic Field Mapper(MFM)` 등이 포함된다.1

개발 키트는 단순한 편의성 도구를 넘어, 프로젝트의 기술적 위험을 사전에 완화하는 핵심적인 역할을 수행한다. 엔지니어는 개발 키트를 통해 실제 제품이 사용될 환경과 유사한 조건에서 센서의 성능을 검증할 수 있다. 예를 들어, 애플리케이션의 동적 특성에 따른 자세 정확도를 평가하고, 주변의 모터나 배터리로 인해 발생하는 자기장 간섭의 영향을 분석하며, `MT Manager`의 MFM 기능을 사용하여 이러한 자기장 왜곡을 보상하는 교정 작업을 수행할 수 있다.3 이러한 검증 및 교정 과정을 커스텀 PCB 설계 이전에 완료함으로써, 개발 후반에 발생할 수 있는 값비싼 재설계 비용과 시간을 절약하고, 최종 제품이 요구 성능을 만족할 것이라는 확신을 얻을 수 있다. 개발 키트의 실드 보드 자체는 최종 제품에 통합하기 위한 용도로 설계되지 않았다는 점을 명심해야 한다.15


`-T`는 트레이(Tray) 패키징을 의미하며, OEM(Original Equipment Manufacturer)이 최종 상용 제품에 MTi-3 모듈을 대량으로 실장할 목적으로 구매할 때 제공되는 형태이다.2

- **특징:** 모듈은 표면 실장 기술(SMD/SMT) 공정에서 사용되는 자동화된 픽 앤 플레이스(Pick-and-place) 장비와 완벽하게 호환되도록 표준 트레이에 담겨 제공된다.17 이는 수백, 수천 개의 제품을 생산하는 양산 라인에서 조립 공정의 효율성을 극대화하고 인건비를 절감하는 데 직접적으로 기여한다. Xsens는 `-T`(트레이 20개입), `-C`(트레이 100개입), `-R`(릴 250개입) 등 다양한 수량의 패키징 옵션을 제공한다.16

`-DK`와 `-T` 접미사는 현대 전자 제품 개발의 두 가지 핵심 단계인 연구개발(R&D)과 양산(Mass Production)을 명확하게 반영한다. `-DK`는 R&D 단계에서 빠른 반복 테스트와 기술 검증을 가능하게 하고, `-T`는 검증이 완료된 부품을 사용하여 확장 가능하고 비용 효율적인 생산을 지원한다. 한 프로젝트가 `-DK` 평가 단계를 통과하지 못하고 `-T` 양산 단계로 넘어가지 못하는 경우는, 평가 과정에서 해결하기 어려운 근본적인 기술적 문제(예: 과도한 자기장 간섭, 요구 정확도 미달)가 발견되었음을 의미할 수 있다. 이는 개발 키트를 이용한 철저한 사전 검증 과정의 중요성을 다시 한번 강조한다.


파트 넘버의 핵심 부분인 `-0I`와 `-5A`는 단순한 제품 변종(variant)이 아닌, 명확한 세대 교체를 나타내는 중요한 식별자이다.

- **세대 교체 및 제품 수명 주기:** 여러 유통사 및 공식 문서에서 `MTi-3-0I-T`는 단종(End of Life, EOL) 또는 단종 예정으로 명시되어 있으며, `MTi-3-5A-T`가 이를 대체하는 공식적인 후속 모델로 지정되어 있다.17 따라서 모든 신규 설계는 `MTi-3-5A` 모델을 기반으로 해야 한다.
- **하드웨어 버전과의 직접적 연관성:** 이 모델명의 변경은 MTi 1-시리즈의 내부 하드웨어(HW) 버전 진화와 직접적으로 연결된다. Movella(Xsens)가 발표한 공식 마이그레이션 문서에 따르면, `-0I`로 끝나는 파트 넘버는 **하드웨어 버전 3(HW v3)**에 해당하며, `-5A`로 끝나는 새로운 파트 넘버는 **하드웨어 버전 5(HW v5)**에 해당한다.16

이 명명법의 발견은 본 보고서의 전체 분석 방향을 결정하는 핵심 열쇠이다. 사용자가 질의한 네 가지 파트 넘버에 대한 비교는 사실상 두 개의 쌍, 즉 구세대 제품군(`MTi-3-0I-T` 모듈과 이를 포함하는 `MTI-3-DK`)과 신세대 제품군(`MTi-3-5A-T` 모듈과 이를 포함하는 `MTi-3-5A-DK`) 간의 비교로 단순화될 수 있다. 따라서 두 모델 간의 기술적 차이는 곧 HW v3와 HW v5 간의 하드웨어 변경점에서 비롯되는 차이와 동일하다. 이 관점은 단순한 기능 비교를 넘어, 기술의 진화, 부품 공급망 관리, 그리고 마이그레이션 전략이라는 더 깊은 차원의 분석을 가능하게 한다.


`MTi-3-0I-T`와 `MTi-3-5A-T`의 비교는 곧 하드웨어 버전 3(HW v3)과 하드웨어 버전 5(HW v5)의 비교이다. 이 섹션에서는 하드웨어의 진화 과정과 그로 인한 기술 사양의 변화를 심층적으로 분석한다.


Xsens MTi 1-시리즈는 시장 출시 이후, 핵심 부품의 단종(EOL)에 대응하고 지속적인 성능 개선을 추구하기 위해 여러 차례의 하드웨어 업데이트를 거쳐왔다. 이 마이그레이션 경로는 v1 -->> v2 -->> v3 -->> v5로 요약될 수 있다.16

- **초기 버전(v1 -->> v2/v3)의 주요 개선:** HW v1에서 v2로의 마이그레이션(2018년 6월)은 새로운 가속도계, 자이로스코프, 지자기센서를 도입하면서 이루어졌다.20 이 과정에서 자이로스코프의 노이즈 밀도는 약 0.007 °/s/√Hz에서 0.005 °/s/√Hz로, 바이어스 안정성은 약 10-12 °/h에서 6-8 °/h로 크게 향상되었다.20 가속도계 역시 노이즈 특성이 개선되었다.20 HW v2에서 v3로의 마이그레이션(2021년 8월)에서는 가속도계와 자이로스코프가 다시 한번 교체되어 가속도계의 노이즈 밀도가 120 µg/√Hz에서 70 µg/√Hz로 개선되는 등의 변화가 있었다.22
- **HW v3 (`-0I`)에서 HW v5 (`-5A`)로의 핵심 변경점:** 본 보고서의 핵심 비교 대상인 이 마이그레이션은 2023년 5월에 시작되었다. 가장 주된 변경 원인은 HW v3에서 사용하던 **지자기센서 부품의 단종**이었다.16
  - **변경점 1: 지자기센서 (Magnetometer):** 단종된 부품을 대체하기 위해 새로운 지자기센서가 도입되었다. 하지만 Movella는 새로운 부품이 기존 부품과 동일한 성능 사양(측정 범위, 노이즈, 비선형성, 분해능 등)을 갖도록 선정했음을 명시했다.16
  - **변경점 2: 최소 공급 전압 (Minimum Supply Voltage):** 교체된 지자기센서는 기존 부품보다 더 높은 전력을 요구한다. 이로 인해 모듈의 안정적인 동작을 위한 **최소 공급 전압이 기존 2.16V (HW v3)에서 2.8V (HW v5)로 상향 조정**되었다.16 이는 기존 시스템에 신형 모듈을 적용하려는 하드웨어 설계자에게 가장 중요하고 직접적인 영향을 미치는 변경 사항이다.
  - **불변 사항:** 가속도계와 자이로스코프 부품은 HW v3와 HW v5 간에 동일하게 유지되었다.16 또한, 모듈의 기계적 폼팩터(12.1 x 12.1 x 2.55 mm)와 PLCC-28 패키지의 핀아웃(pinout) 역시 변경되지 않아 물리적인 드롭인(drop-in) 교체가 가능하다.16

여기서 주목할 점은, 하드웨어 부품이 여러 세대에 걸쳐 지속적으로 변경 및 개선되었음에도 불구하고, 최종 제품의 핵심 성능 지표인 센서 퓨전 성능(Roll, Pitch, Yaw 정확도)은 동일하게 유지되었다는 사실이다.16 예를 들어, 자이로스코프의 바이어스 안정성이 10 deg/h에서 6 deg/h로 개선되었음에도 1, 최종 Yaw 정확도는 2 deg RMS로 동일하게 명시된다.

이는 시스템의 최종 정확도가 개별 MEMS 센서의 원시 데이터 품질에만 의존하지 않음을 시사한다. 일단 특정 임계 수준 이상의 품질을 가진 센서가 사용되면, 시스템의 성능 한계는 센서 자체의 노이즈나 바이어스 안정성보다는 **센서 퓨전 알고리즘의 정교함**에 의해 더 크게 좌우된다. Xsens의 독자적인 칼만 필터(XKF)는 센서의 각종 오차(바이어스, 스케일 팩터, 축 정렬 오차 등)를 실시간으로 모델링하고 보상하며, 외부 환경의 노이즈(진동, 자기장 왜곡)를 효과적으로 제거하는 데 매우 뛰어나다. 따라서 최종 정확도는 중력과 선형 가속도를 구분하는 것의 내재적 모호성, 복잡한 실제 환경의 자기장을 정확히 매핑하는 것의 어려움과 같은 더 근본적인 물리적, 알고리즘적 한계에 의해 결정된다.

결론적으로, HW v3에서 v5로의 진화와 같은 하드웨어 업그레이드는 최종 자세 정확도를 높이기 위한 목적보다는, 부품 단종에 따른 공급망 안정성 확보, 생산 공정의 일관성 향상, 기계적 스트레스에 대한 내구성 강화와 같은 실용적인 목표를 위해 이루어졌다고 해석할 수 있다.16 이는 Xsens의 센서 퓨전 코어 기술이 이미 높은 수준의 성숙도에 도달했음을 보여주는 방증이기도 하다.


다음 표는 단종된 `MTi-3-0I-T` (HW v3 기반)와 이를 대체하는 `MTi-3-5A-T` (HW v5 기반)의 주요 기술 사양을 비교하여, 마이그레이션 시 엔지니어가 고려해야 할 핵심 사항들을 명확히 제시한다.

**Table 1: MTi-3-0I-T (HW v3 기반) vs. MTi-3-5A-T (HW v5 기반) 상세 기술 사양 비교**

| **분류 (Category)** | **항목 (Parameter)**     | **MTi-3-0I-T (HW v3 기반)** | **MTi-3-5A-T (HW v5 기반)** | **주요 차이점 및 고찰 (Key Differences & Analysis)**         |
| ------------------- | ------------------------ | --------------------------- | --------------------------- | ------------------------------------------------------------ |
| **센서 퓨전 성능**  | Roll, Pitch 정확도 (RMS) | 0.5 deg 9                   | 0.5 deg 1                   | **변경 없음.** 성능은 하드웨어 부품보다 센서 퓨전 알고리즘에 의해 결정됨을 시사한다.16 |
|                     | Yaw/Heading 정확도 (RMS) | 2 deg 9                     | 2 deg 1                     | **변경 없음.** 자기장 환경 및 MFM 교정이 정확도에 더 큰 영향을 미친다.14 |
| **자이로스코프**    | In-run bias stability    | 10 deg/h 9*                 | 6 deg/h 1                   | *9의 사양은 HW v1/v2 시절의 것으로 보이며, HW v3와 v5는 6 deg/h 사양을 공유할 가능성이 높다.16 |
|                     | Noise Density            | 0.007 °/s/√Hz 9*            | 0.003 °/s/√Hz 1             | 위와 동일한 이유로, HW v3/v5는 더 개선된 사양을 공유할 것이다. |
| **가속도계**        | In-run bias stability    | 30 µg 9                     | 40 µg 1                     | 사양상 약간의 차이가 있으나, 이는 측정 조건의 차이일 수 있으며 최종 성능에 영향을 주지 않는다.16 |
|                     | Noise Density            | 120 µg/√Hz 9                | 70 µg/√Hz 1                 | HW v3에서 이미 개선된 사양(70 µg/√Hz)이 적용되었을 가능성이 높다.22 |
| **지자기센서**      | 핵심 부품                | 구형 부품                   | 신형 부품 (EOL로 인한 교체) | **부품 변경.** 하지만 모든 성능 지표(범위, 노이즈, 분해능 등)는 동일하게 유지되었다.16 |
| **기계적 사양**     | 크기 (Dimensions)        | 12.1 x 12.1 x 2.55 mm 9     | 12.1 x 12.1 x 2.55 mm 1     | **변경 없음.** 물리적 드롭인(Drop-in) 교체가 가능하다.       |
|                     | 패키지 (Package)         | PLCC-28 (SMD/SMT) 17        | PLCC-28 (SMD/SMT) 2         | **변경 없음.**                                               |
| **전기적 사양**     | **최소 공급 전압**       | **2.16 V** 17               | **2.8 V** 1                 | **가장 중요한 변경점.** 기존 시스템의 전원 레일이 2.8V 미만일 경우 호환성 문제가 발생할 수 있다. |
|                     | 최대 공급 전압           | 3.6 V 17                    | 3.6 V 1                     | 변경 없음.                                                   |
|                     | 소비 전력 (Typical)      | 44 mW @ 3V 9                | <100 mW @ 3V 1              | 표기 방식의 차이로 보이며, 실제 소비 전력은 유사한 수준일 것으로 추정된다. |
| **인터페이스**      | 통신 프로토콜            | I²C, SPI, UART / Xbus 9     | I²C, SPI, UART / Xbus 1     | **변경 없음.** 소프트웨어 프로토콜 호환성이 유지된다.        |
| **제품 수명 주기**  | 상태 (Status)            | **단종 (End of Life)** 17   | **양산 중 (Active)** 24     | **신규 설계 시 `-5A` 모델 사용이 필수적이다.**               |


MTi-3의 성능은 단순히 개별 센서의 사양만으로 결정되지 않으며, 내장된 정교한 알고리즘과 외부 환경 요인의 상호작용을 통해 이해해야 한다.

- **알고리즘의 역할:** MTi-3의 핵심에는 두 가지 주요 알고리즘 엔진이 있다.

  - `AttitudeEngine™`: 이 엔진은 1 kHz의 높은 속도로 스트랩다운 적분(Strapdown Integration, SDI)을 수행한다.1 이는 자이로스코프에서 측정된 각속도를 시간에 대해 적분하여 짧은 시간 동안의 자세 변화를 계산하는 과정으로, 고주파의 빠른 움직임을 놓치지 않고 포착하는 데 필수적이다.

  - `XKF3™`: 이 알고리즘은 Xsens의 독자적인 확장 칼만 필터(Extended Kalman Filter)의 구현체이다.6

    `AttitudeEngine™`에서 계산된 단기적인 자세 변화량(Δq)을 중력 방향(가속도계 측정값)과 지구 자기장 방향(지자기센서 측정값)이라는 두 개의 절대적인 외부 기준 정보와 최적으로 융합한다. 이 과정을 통해 자이로스코프 적분 과정에서 필연적으로 발생하는 누적 오차(drift)를 효과적으로 보상하고, 장시간에 걸쳐 안정적이고 드리프트 없는(drift-free) 자세(Roll, Pitch) 및 방위각(Yaw)을 계산해낸다.25

- **동적 정확도에 영향을 미치는 요인:** 데이터시트에 명시된 동적 정확도(Roll/Pitch: 0.5 deg RMS, Yaw: 2 deg RMS)를 달성하기 위해서는 다음 요인들을 반드시 고려해야 한다.

  - **자기장 환경:** AHRS의 방위각(Yaw) 정확도는 지구 자기장을 기준으로 하므로, 주변 환경의 자기장 균일성(homogeneity)에 크게 의존한다.14 시스템에 포함된 모터, 배터리, 전원 케이블, 철 구조물 등은 지구 자기장을 왜곡시키는 주된 원인이 된다. 이러한 왜곡을 보상하지 않으면 심각한 방위각 오차가 발생한다. 

    `MT Software Suite`에 포함된 MFM(Magnetic Field Mapper) 도구는 이러한 왜곡을 측정하고 보상하는 교정 프로파일을 생성하는 필수적인 절차이다.3

  - **진동 및 가속:** MTi-3 모듈은 진동에 강한 설계를 바탕으로 하며, `AttitudeEngine™`의 신호 처리 파이프라인에는 진동 제거(vibration-rejection) 기능이 포함되어 있다.6 또한, 코닝/스컬링(coning/sculling) 보상 알고리즘을 통해 진동이 심한 환경에서도 적분 오차를 최소화한다.27 하지만, 가속도계의 측정 범위(±16 g)를 초과하는 매우 강한 충격이나 진동은 센서의 포화(saturation)를 일으킬 수 있으며, 이는 일시적으로 부정확한 Roll/Pitch 값으로 나타날 수 있다.27 따라서 가능한 한 진동이 적은 위치에 모듈을 장착하거나, 필요한 경우 진동 댐퍼를 사용하는 것이 좋다.

- **데이터 출력:** 모듈은 Xbus라는 독자적인 바이너리 프로토콜을 사용하여 호스트 프로세서와 통신한다.1 사용자는 이 프로토콜을 통해 출력할 데이터의 종류(예: 오일러 각, 쿼터니언, 보정된 센서 데이터, 상태 정보 등)와 각 데이터의 출력 주파수(최대 1 kHz)를 매우 유연하게 설정할 수 있다.1 이는 시스템의 통신 대역폭과 호스트 프로세서의 부하를 최적화하는 데 큰 장점을 제공한다.


올바른 모델을 선택하고 이를 시스템에 성공적으로 통합하기 위해서는 개발 단계와 양산 단계를 구분하여 체계적인 전략을 수립해야 한다.


- **모델 선택의 당위성:** 모든 신규 프로젝트는 반드시 최신 세대인 **`MTi-3-5A-DK`** 를 사용하여 시작해야 한다. `MTI-3-DK`는 단종된 구형 `MTi-3-0I-T` 모듈을 포함하고 있으므로, 더 이상 구매하거나 신규 개발에 사용해서는 안 된다.10

  `MTi-3-5A-DK`는 AHRS(MTi-3) 기능뿐만 아니라, 동일한 하드웨어에서 펌웨어 설정 변경을 통해 VRU(MTi-2) 및 IMU(MTi-1) 기능까지 평가할 수 있는 다목적 도구이다.11

- **권장 워크플로우:**

  1. **초기 설정 및 연결:** `MTi-3-5A-DK`를 제공된 USB 케이블로 PC에 연결하고, Xsens 웹사이트에서 최신 버전의 `MT Software Suite`를 다운로드하여 설치한다.3
  2. **기본 성능 확인:** `MT Manager`를 실행하여 장치가 정상적으로 인식되는지 확인한다. 실시간 데이터 뷰어의 그래프를 통해 Roll, Pitch, Yaw 데이터가 안정적으로 출력되는지 시각적으로 검증한다.3
  3. **대표 움직임 기록 및 분석:** 데이터 로깅 기능을 사용하여 실제 애플리케이션에서 발생할 수 있는 대표적인 움직임(representative motion)을 기록한다. 이를 통해 특정 동작(급가속, 급선회 등)에서 센서의 반응 특성을 파악할 수 있다.3
  4. **자기장 교정 (MFM):** **이 단계는 AHRS의 성능을 결정하는 가장 중요한 과정이다.** 최종 제품에서 MTi-3 모듈이 장착될 실제 위치와 동일한 환경에서 `Magnetic Field Mapper`를 실행한다. MFM은 장치를 천천히 여러 방향으로 회전시켜 주변의 고정된 자기장 왜곡(hard and soft iron effects)을 측정하고, 이를 보상하기 위한 교정 파라미터를 계산한다. 이 교정 프로파일을 생성하고 장치에 저장해야만 데이터시트에 명시된 2 deg RMS의 방위각 정확도를 기대할 수 있다.3
  5. **SDK를 이용한 프로토타이핑:** `MT SDK`에 포함된 다양한 언어(C++, Python, MATLAB, ROS 등)의 예제 코드를 활용하여 호스트 프로세서의 통신 및 데이터 처리 로직을 개발하고 테스트한다.1 이를 통해 양산용 펌웨어 개발에 필요한 기반을 마련한다.


개발 및 평가 단계에서 기술 검증이 완료되면, 양산을 위해 `-T` 모델을 시스템에 통합한다.

- **하드웨어 통합:**

  - **모델 선택:** 양산용 모델로 **`MTi-3-5A-T`** 를 사용한다.

  - **마운팅:** MTi-3 모듈은 PLCC-28 패키지로 제공된다. 이론적으로는 PLCC-28 소켓을 사용할 수 있지만, Xsens는 최상의 성능과 기계적 안정성을 위해 리플로우 솔더링(reflow soldering) 공정을 통해 PCB에 직접 실장할 것을 강력히 권장한다. 소켓을 사용하면 PCB에 미세한 기계적 응력을 유발할 수 있으며, 이는 민감한 MEMS 센서의 성능을 저하시키는 요인이 될 수 있다.15

  - **전원 설계 검증:** **가장 중요한 검증 사항**이다. `MTi-3-5A-T`의 최소 공급 전압은 2.8V이다.16 시스템의 전원 공급 회로가 노이즈와 전압 강하(voltage drop)를 고려하더라도 이 최소 요구 전압 이상을 안정적으로 공급하는지 오실로스코프 등으로 반드시 실측하여 검증해야 한다. 특히 구형 

    `-0I` 모델(최소 2.16V)을 사용하던 시스템을 마이그레이션할 경우, 이 부분을 확인하지 않으면 모듈이 비정상적으로 동작하거나 아예 부팅되지 않는 문제가 발생할 수 있다.

- **소프트웨어 통합:**

  - **인터페이스 선택:** 호스트 MCU의 가용 포트와 시스템의 통신 요구사항에 맞춰 UART, SPI, I²C 중 최적의 통신 인터페이스를 선택한다. 선택된 인터페이스는 모듈의 PSEL 핀들의 논리 상태(logic state)에 따라 모듈 부팅 시 결정된다.12
  - **펌웨어 구현:** MT SDK의 소스 코드나 저수준 통신 프로토콜 문서([LLCP])를 참조하여, 호스트 펌웨어에 Xbus 프로토콜을 파싱하고 원하는 데이터를 추출하는 루틴을 구현한다.25
  - **교정 데이터 적용:** 개발 단계에서 생성했던 MFM 교정 데이터를 양산 공정에서 각 개별 모듈에 기록(Write to device)하는 절차를 반드시 포함해야 한다. 이 과정이 누락되면 제품마다 다른 방위각 오차를 보이게 되어 제품 품질의 일관성을 확보할 수 없다.


본 보고서는 Xsens의 AHRS 모듈인 `MTI-3-DK`, `MTi-3-5A-DK`, `MTi-3-0I-T`, `MTi-3-5A-T`에 대한 다각적이고 심층적인 비교 분석을 수행했다. 분석 결과를 바탕으로 다음과 같은 결론 및 최종 권장 사항을 제시한다.


분석 대상인 네 가지 파트 넘버의 관계는 복잡하지 않으며, 두 가지 기준, 즉 **패키징 형태**와 **제품 세대**로 명확하게 구분된다.

- **패키징 형태:** `-DK` 접미사는 개발 및 평가를 위한 **개발 키트**를, `-T` 접미사는 양산을 위한 **트레이 패키징 모듈**을 의미한다.
- **제품 세대:** `-0I` 식별자는 단종된 **구형(HW v3 기반)** 모델을, `-5A` 식별자는 현재 양산 중인 **신형(HW v5 기반)** 모델을 나타낸다.

`MTi-3-0I-T`와 `MTi-3-5A-T` 간의 근본적인 차이는 내부 하드웨어 버전(HW v3 vs. HW v5)의 차이에서 비롯되며, 이는 사용자에게 직접적인 영향을 미치는 **최소 공급 전압의 상향 조정(2.16V -->> 2.8V)** 이라는 결정적인 전기적 사양 변경으로 귀결된다.16

주목할 점은, 지자기센서 부품 교체라는 중요한 하드웨어 변경에도 불구하고 기계적 폼팩터와 핀아웃, 그리고 가장 중요한 최종 센서 퓨전 성능(자세 및 방위각 정확도)은 동일하게 유지되었다는 것이다.16 이는 기존 사용자가 하드웨어의 전원 호환성 문제만 해결하면, 별도의 소프트웨어나 알고리즘 수정 없이도 신형 모델로 원활하게 마이그레이션할 수 있음을 의미한다.


- **신규 설계 및 개발:** 어떠한 경우에도 최신 세대 제품을 사용하는 것이 원칙이다. 따라서 **`MTi-3-5A-DK`** 를 구매하여 개발 및 평가를 시작하고, 양산 단계에서는 **`MTi-3-5A-T`** 를 시스템에 통합해야 한다. 단종된 `-0I` 모델 및 관련 개발 키트(`MTI-3-DK`)는 고려 대상에서 완전히 제외해야 한다.
- **기존 `MTi-3-0I-T` 사용자의 마이그레이션:** 부품 수급의 안정성을 위해 `MTi-3-5A-T`로의 마이그레이션 계획을 수립해야 한다. 마이그레이션 과정에서 가장 시급하고 중요한 조치는 현재 시스템의 **전원 공급 회로가 새로운 최소 전압 요구사항인 2.8V를 안정적으로 충족하는지 검증**하는 것이다. 이 조건만 만족된다면, 물리적, 성능적으로 큰 문제 없이 원활한 대체가 가능하다.


MTi-3 모듈의 진화 과정은 순수한 성능 지표의 향상보다는, 부품 단종에 대응하기 위한 공급망 안정성 확보와 생산 공정의 견고성 및 일관성 향상에 더 큰 초점을 맞추고 있다. 이는 Xsens의 센서 퓨전 기술이 해당 제품 등급에서 이미 매우 안정되고 최적화된 상태에 도달했음을 간접적으로 보여준다.

따라서, 사용자는 하드웨어 자체의 미세한 사양 변화에 집중하기보다는, 센서가 가진 잠재 성능을 100% 이끌어내기 위한 **올바른 통합 방법론**과 **환경에 맞는 설정**에 더욱 집중해야 한다. 구체적으로는, 기계적 스트레스를 최소화하는 **리플로우 솔더링 방식의 실장**과, 정확한 방위각 확보를 위한 필수 절차인 **자기장 교정(MFM)**을 철저히 수행하는 것이 성공적인 시스템 개발의 핵심이 될 것이다.


1. MTi-3 - Movella.com, accessed August 11, 2025, https://www.xsens.com/hubfs/Downloads/Leaflets/MTi-3.pdf
2. MTi-3-5A-T Movella / Xsens - Mouser Electronics, accessed August 11, 2025, [https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-5A-T?qs=3Rah4i%252BhyCEVHzvLS1mcLw%3D%3D](https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-5A-T?qs=3Rah4i%2BhyCEVHzvLS1mcLw%3D%3D)
3. Xsens MTi-3 AHRS - Movella.com, accessed August 11, 2025, https://www.movella.com/sensor-modules/xsens-mti-3-ahrs
4. How to choose the best Xsens inertial measurement unit - Movella.com, accessed August 11, 2025, https://www.movella.com/resources/blog/how-to-choose-the-best-xsens-inertial-measurement-unit
5. Xsens Sensor Modules - Movella.com, accessed August 11, 2025, https://www.movella.com/sensor-modules
6. Xsens / Movella MTi-3-DK Development Kit - Mouser Electronics, accessed August 11, 2025, https://www.mouser.com/new/movella/xsens-mti-3-dk-development-kit/
7. MTi Family Reference Manual, accessed August 11, 2025, https://www.xsens.com/hubfs/Downloads/Manuals/MTi_familyreference_manual.pdf
8. Xsens MTi-3 (9-Axis IMU + AHRS) - Lulu's blog, accessed August 11, 2025, https://lucidar.me/en/inertial-measurement-unit/imu-xsens-mti-3-ahrs/
9. Xsens MTi-3 AHRS Sensor Module - Canal Geomatics, accessed August 11, 2025, https://canalgeomatics.com/product/xsens-mti-3-ahrs-sensor-module/
10. MTI-3-DK Movella / Xsens - Mouser Electronics, accessed August 11, 2025, https://www.mouser.com/ProductDetail/Movella-Xsens/MTI-3-DK?qs=55YtniHzbhBGvlIRYNoxgQ%3D%3D
11. MTi-3-5A-DK Movella / Xsens - Mouser Electronics, accessed August 11, 2025, https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-5A-DK?qs=9vOqFld9vZVm8SUT4T13Og%3D%3D
12. XSENS MTi-3 Click - MIKROE, accessed August 11, 2025, https://www.mikroe.com/xsens-mti-3-click
13. XSENS MTi-3 Click, accessed August 11, 2025, https://static6.arrow.com/aropdfconversion/1566ba1b2cf2ab488954c2909bd950f9ba868164/xsens_mti-3_click.pdf
14. Reasons why the heading/yaw is not stable or incorrect - Movella.com, accessed August 11, 2025, https://base.movella.com/s/article/Reasons-why-the-heading-yaw-is-not-stable-or-incorrect?language=en_US
15. Altium library and STEP-file for MTi 1-series - Movella.com, accessed August 11, 2025, https://base.movella.com/s/article/Altium-library-and-STEP-file-for-MTi-1-series-1605870241073?language=en_US
16. Migration to Hardware Version 5 of the MTi 1-series - Movella.com, accessed August 11, 2025, https://base.movella.com/s/article/Migration-to-Hardware-Version-5-of-the-MTi-1-series
17. MTi-3-0I-T Movella / Xsens | Mouser, accessed August 11, 2025, [https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-0I-T?qs=pBJMDPsKWf0vHDkgc01%252BrQ%3D%3D](https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-0I-T?qs=pBJMDPsKWf0vHDkgc01%2BrQ%3D%3D)
18. MTI-3-0I Xsens a Movella brand | Sensors, Transducers | DigiKey, accessed August 11, 2025, https://www.digikey.com/en/products/detail/xsens-a-movella-brand/MTI-3-0I/14834329
19. MTi 1-series Hardware Version 1.x Firmware Updates - Movella.com, accessed August 11, 2025, https://base.movella.com/s/article/MTi-1-series-Hardware-Version-1-x-Firmware-Updates
20. Migration to Hardware Version 2.0 of the MTi 1-series - Login | Movella Trial Site, accessed August 11, 2025, https://movella.my.site.com/XsensKnowledgebase/s/article/Migration-to-Hardware-Version-2-0-of-the-MTi-1-series-1605870241149
21. Links to the documentation, leaflets and changelogs for the MTi series - Movella.com, accessed August 11, 2025, https://base.movella.com/s/article/Online-links-to-manuals-from-the-MT-Software-Suite
22. Migration to Hardware Version 3 of the MTi 1-series - DigiKey, accessed August 11, 2025, [https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/2576/Migration%20MTi%201-series.pdf](https://mm.digikey.com/Volume0/opasdata/d220001/medias/docus/2576/Migration MTi 1-series.pdf)
23. XSENS MTI-3-5A-T - Spezial Electronic, accessed August 11, 2025, https://www.spezial.com/en/mti-3-5a-t-p10212/
24. MTI-3-5A-T Xsens a Movella brand | Sensors, Transducers | DigiKey, accessed August 11, 2025, https://www.digikey.com/en/products/detail/xsens-a-movella-brand/MTI-3-5A-T/18718048
25. MTi User Manual, accessed August 11, 2025, https://www.xsens.com/hubfs/Downloads/usermanual/MTi_usermanual.pdf
26. Industrial grade MEMS 3D motion trackers maintain accuracy under vibrations, accessed August 11, 2025, https://www.eenewseurope.com/en/industrial-grade-mems-3d-motion-trackers-maintain-accuracy-under-vibrations/
27. Best Practices for Automotive Applications - Movella.com, accessed August 11, 2025, https://base.movella.com/s/article/Best-Practices-for-Automotive-Applications?language=en_US
28. Software Downloads | Movella.com, accessed August 11, 2025, https://www.movella.com/support/software-documentation


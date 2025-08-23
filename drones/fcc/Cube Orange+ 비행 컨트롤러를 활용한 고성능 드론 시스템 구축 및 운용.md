# Cube Orange+ 비행 컨트롤러를 활용한 고성능 드론 시스템 구축 및 운용



Cube Orange+ 비행 컨트롤러(Flight Controller, FC)는 단순한 비행 제어 장치를 넘어, 상업 및 연구용 무인 시스템을 위한 고도로 통합된 자율 비행의 핵심 모듈이라 정의할 수 있다. 이는 과거 Pixhawk 2.1로 알려진 플랫폼의 직계 후손으로서, 수년간 축적된 오픈소스 하드웨어 생태계의 기술적 정점에 위치한다.1

CubePilot 생태계는 Cube 모듈을 중심으로 캐리어 보드(Carrier Board), 고정밀 GPS 모듈(Here 시리즈), 통합 텔레메트리 및 제어 시스템(Herelink) 등 다양한 액세서리가 유기적으로 결합된 종합 솔루션을 제공한다.5 이러한 접근 방식은 사용자가 특정 임무 요구사항에 맞춰 시스템을 유연하게 확장하고 맞춤화할 수 있는 모듈식 설계의 근간을 이룬다.9

본 컨트롤러의 적용 범위는 매우 광범위하여 멀티로터, 고정익 항공기, 수직이착륙기(VTOL), 지상 로버, 무인 보트, 수중 잠수정 등 다양한 형태의 무인 이동체에 탑재될 수 있다.12 특히 정밀 농업, 지리 정보 시스템(GIS)을 위한 측량 및 매핑, 물류 배송, 비무장 전술 임무와 같은 고도의 신뢰성과 정밀성을 요구하는 전문적인 분야에서 그 진가를 발휘한다.1


Cube Orange+의 뛰어난 성능은 정교하게 설계된 하드웨어 아키텍처에 기반한다. 각 핵심 구성 요소는 높은 신뢰성과 처리 능력을 목표로 선정 및 통합되었다.


주 비행 관리 장치(Flight Management Unit, FMU)는 고성능 32비트 ARM Cortex-M7 코어 기반의 **STM32H757** 듀얼코어 마이크로컨트롤러를 탑재했다.1 이 프로세서는 400MHz의 주 클럭 속도, 2MB의 플래시 메모리, 1MB의 RAM을 제공하여 복잡한 비행 알고리즘, 센서 융합, 자동 임무 수행 등을 지연 없이 처리할 수 있는 강력한 연산 능력을 보장한다.10 이는 이전 세대 모델인 Cube Black의 STM32F4 프로세서나 Cube Orange의 STM32H753에 비해 월등한 성능이다.19

또한, 시스템 안정성을 극대화하기 위해 별도의 **STM32F103** 또는 **STM32F100** 보조 프로세서(I/O Processor)를 탑재했다.13 이 보조 프로세서는 주 프로세서에 장애가 발생할 경우 비행 제어권을 즉시 인수하거나, 수동 조종 입력을 처리하는 등 페일세이프(Failsafe) 기능을 수행한다. 이러한 이중화 설계 철학은 시스템의 단일 지점 장애(Single Point of Failure) 가능성을 최소화하는 핵심 요소다.21


Cube Orange+는 다중화 및 환경 적응성을 고려한 정교한 센서 시스템을 갖추고 있다.

- **3중 관성 측정 장치 (Triple Redundant IMU)**: 3개의 독립적인 관성 측정 장치(IMU)를 탑재하여 특정 센서에서 오류가 발생하더라도 나머지 센서 데이터를 기반으로 안정적인 비행 자세 추정이 가능하다.9 이는 비행 안전성과 직결되는 핵심적인 이중화 기능이다.
- **진동 절연 및 온도 제어**: 3개의 IMU 중 2개는 물리적인 댐퍼가 포함된 보드에 실장되어 기체 프레임에서 발생하는 고주파 진동이 센서에 전달되는 것을 기계적으로 차단한다.9 또한, 내장된 히팅 저항을 통해 IMU를 최적의 작동 온도(-10°C ~ 55°C)로 유지함으로써, 저온 환경에서도 일관되고 신뢰성 있는 센서 데이터를 보장한다.11
- **센서 모델**: 가속도계 및 자이로스코프로는 ICM42688p, ICM20948, ICM20649 등이 사용되며, 고도 측정을 위한 기압계는 2개의 MS5611 센서가 이중으로 탑재된다. 방향 탐지를 위한 지자계(나침반)는 ICM20948 IMU에 내장된 센서를 활용한다.12 모든 센서는 데이터 전송 지연을 최소화하기 위해 고속 SPI(Serial Peripheral Interface) 버스로 주 프로세서와 연결된다.10


Cube Orange+는 다양한 주변기기와의 연결성과 안정적인 전원 공급을 위한 폭넓은 인터페이스를 제공한다.

- **전원 시스템**: 2개의 독립적인 전원 입력 포트(POWER1, POWER2)를 통해 전원을 공급받으며, 주 전원에 문제가 발생할 경우 자동으로 보조 전원으로 전환되는 페일오버 기능을 지원한다.6
- **입출력 포트**: 총 14개의 PWM 서보 출력(I/O 프로세서에서 8개, FMU에서 6개)을 제공하며, 5개의 UART 시리얼 포트, 2개의 CAN(Controller Area Network) 버스 포트, 2개의 I2C 포트, SPI 포트 등 풍부한 확장성을 갖추고 있다.10
- **PWM 전압 가변성**: 서보 레일의 PWM 출력 신호 전압을 지상관제시스템(GCS) 소프트웨어를 통해 3.3V와 5V 사이에서 전환할 수 있다. 이는 다양한 종류의 서보 모터나 ESC와의 호환성을 크게 향상시킨다.12

| 항목 (Category)             | 상세 사양 (Detailed Specification)                           |
| --------------------------- | ------------------------------------------------------------ |
| **주 프로세서 (FMU)**       | STM32H757 (32bit ARM Cortex-M7 @ 400MHz + M4 @ 200MHz) 13    |
| **보조 프로세서 (IO)**      | STM32F103 (32bit ARM Cortex-M3 @ 24MHz) 18                   |
| **메모리 (Memory)**         | 2MB Flash, 1MB RAM 10                                        |
| **센서 (Sensors)**          | **IMU**: 3x (ICM42688p, ICM20948, ICM20649) 18<br />기압계: 2x MS5611 18<br />지자계: 1x ICM20948 (내장) 18 |
| **센서 특징**               | 2x IMU 진동 절연 및 온도 제어 9                              |
| **전원 입력 (Power Input)** | 4.1V - 5.7V / 2.5A (정격 14W) 1                              |
| **서보 레일 전압**          | 3.3V / 5V (소프트웨어 전환 가능) 12                          |
| **인터페이스**              | 14x PWM 출력, 5x UART, 2x CAN, 2x I2C, 1x SPI 10             |
| **RC 입력**                 | S.BUS, PPM, Spektrum DSM/DSM2/DSM-X 18                       |
| **크기 (Dimensions)**       | Cube: 38.25 x 38.25 x 22.3 mm표준 캐리어 보드: 94.5 x 44.3 x 17.3 mm 18 |
| **무게 (Weight)**           | 약 73g (캐리어 보드 포함) 15                                 |
| **하우징 재질**             | Cube: CNC 가공 알루미늄 합금캐리어 보드: ABS 성형 플라스틱 1 |
| **작동 온도**               | -10°C ~ 55°C 1                                               |


Cube Orange+의 설계는 단순한 성능 향상을 넘어, 실제 운용 환경에서의 안정성과 안전성을 최우선으로 고려하는 철학을 반영한다.

- **이중화 (Redundancy)**: 시스템의 핵심 요소인 IMU, 기압계, 전원 입력, 그리고 주/보조 프로세서에 다중화 및 이중화 개념을 적용했다. 이는 특정 부품의 고장이 전체 시스템의 실패로 이어지는 것을 방지하여, 상업용 드론에 요구되는 높은 수준의 신뢰성을 확보하기 위한 핵심적인 설계 원칙이다.6

- **ADS-B 통합 (ADS-B Integration)**: 표준 캐리어 보드에는 uAvionix 사의 1090MHz ADS-B(Automatic Dependent Surveillance-Broadcast) 수신기가 기본적으로 통합되어 있다.1 이를 통해 드론은 주변을 비행하는 유인 항공기(항공기, 헬리콥터 등)가 송출하는 위치, 고도, 속도 정보를 실시간으로 수신할 수 있다. 이 정보는 GCS 지도에 표시되어 조종사에게 상황 인식을 제공하며, 사전에 설정된 배제 구역 내에 항공기가 탐지될 경우 자동으로 회피 기동을 수행하도록 설정할 수도 있다.1 ADS-B 수신기의 통합은 단순한 충돌 회피 기능의 추가를 넘어선다. 전 세계 항공 규제 기관들은 무인 항공기와 유인 항공기가 공역을 안전하게 공유하기 위한 DAA(Detect and Avoid) 기술을 요구하고 있다.26 ADS-B는 유인 항공 분야에서 널리 사용되는 표준 기술이므로, 이를 드론에 탑재하는 것은 미래의 항공 규제 변화에 선제적으로 대응하고, 복잡한 공역에서의 합법적인 상업적 운용을 위한 전략적 포석으로 해석할 수 있다. 따라서 Cube Orange+를 선택하는 것은 기술적 우위뿐만 아니라, 장기적인 운용 및 상업적 가치를 고려한 결정이 된다.

- **모듈성 및 확장성 (Modularity and Extensibility)**: Cube 모듈과 캐리어 보드는 80핀 DF17 고밀도 커넥터를 통해 연결 및 분리되는 모듈식 구조를 채택했다.9 이는 제조사들이 물류, 농업, 측량 등 특정 임무에 최적화된 기능을 갖춘 맞춤형 캐리어 보드를 직접 설계하고 제작할 수 있는 길을 열어준다.6 이러한 개방성과 확장성은 CubePilot 생태계가 지속적으로 성장하고 다양한 산업 요구에 대응할 수 있게 하는 핵심적인 강점이다.

- **펌웨어 호환성 (Firmware Compatibility)**: Cube Orange+는 대표적인 오픈소스 비행 제어 펌웨어인 ArduPilot과 PX4를 모두 지원한다.12 그러나 커뮤니티의 논의와 실제 사용 사례를 종합해 보면, 

  **ArduPilot**에서 가장 안정적이고 폭넓은 기능 지원을 제공받는다.26 특히 Cube Orange+에 탑재된 새로운 STM32H757 프로세서는 ArduPilot Copter 4.2.3 버전 이상에서 공식적으로 지원되므로, 구버전 펌웨어를 사용할 경우 하드웨어가 올바르게 인식되지 않거나 예상치 못한 호환성 문제가 발생할 수 있다.28 따라서 최상의 성능과 안정성을 위해서는 항상 최신 안정 버전의 ArduPilot 펌웨어를 사용하는 것이 강력히 권장된다.


Cube Orange+의 시장 내 위치를 명확히 이해하기 위해서는 다른 주요 비행 컨트롤러와의 비교 분석이 필수적이다.

- **Cube Orange vs. Cube Orange+**: Cube Orange+는 기존 Cube Orange 출시 이후 발생한 전 세계적인 반도체 부품 수급난에 대응하여 일부 부품을 변경하여 출시된 모델이다. 가장 큰 차이점은 프로세서가 STM32H753에서 STM32H757로 변경되었고, 일부 IMU 센서 모델이 업데이트되었다는 점이다.18 이러한 변경을 통해 생산 단가를 낮출 수 있었지만, 하드웨어 정의 파일(hwdef)이 달라졌기 때문에 구버전 펌웨어와는 호환되지 않는 문제가 발생할 수 있다.28
- **Cube Black vs. Cube Orange+**: Cube Black은 STM32F4 계열 프로세서를 사용하는 이전 세대 모델이다. H7 프로세서를 탑재한 Cube Orange+는 연산 능력, 센서 성능, ADS-B 통합 등 모든 면에서 Cube Black을 압도하며, 현재 Cube 시리즈의 표준 모델로 확고히 자리 잡았다.12
- **"Pixhawk" 브랜드의 모호성**: "Pixhawk"라는 명칭은 특정 기술 사양이나 표준을 지칭하기보다는, 시장에서 널리 알려진 일종의 브랜드명처럼 사용되고 있다.30 CubePilot, CUAV, Holybro 등 여러 제조사가 "Pixhawk"라는 이름을 마케팅에 활용하지만, 실제 하드웨어 구성, 부품 품질, 커뮤니티 지원 수준은 제조사 및 모델별로 큰 차이를 보인다.3 따라서 사용자는 "Pixhawk"라는 이름에 의존하기보다, 'Cube Orange+', 'Pixhawk 6X'와 같은 구체적인 모델명과 그 기술 사양을 기준으로 제품을 평가하고 선택해야 한다.
- **vs. CUAV / Holybro**:
  - **CUAV**: CUAV 제품군(예: V6X, X7)은 케이블, 커넥터 등 부속품의 물리적인 품질과 만듦새가 견고하다는 긍정적인 평가를 받는다.31 일부 모델은 Cube Orange보다 많은 수의 DShot 프로토콜 지원 출력 포트를 제공하는 등 특정 기능에서 강점을 보이기도 한다.31 그러나 공식 문서나 커뮤니티 지원은 CubePilot에 비해 상대적으로 부족하여, 문제 발생 시 해결에 더 많은 노력이 필요할 수 있다.
  - **Holybro**: Holybro의 Pixhawk 6X와 같은 모델은 Cube Orange+와 유사한 H7 프로세서와 다중 IMU를 탑재하면서도 훨씬 저렴한 가격에 판매되어 높은 가성비를 자랑한다.32 이는 예산이 중요한 연구 프로젝트나 개인 개발자에게 매우 매력적인 대안이 될 수 있다.

결론적으로, 비행 컨트롤러의 선택은 단순히 '최고의 제품'을 찾는 것이 아니라, 프로젝트의 목표와 제약 조건에 '최적화된 제품'을 찾는 과정이다. Cube Orange+는 강력한 성능과 신뢰성, 잘 구축된 생태계를 제공하지만 상대적으로 가격이 높다.32 반면 Holybro는 유사한 핵심 성능을 더 낮은 비용으로 제공하며, CUAV는 특정 하드웨어 품질에서 강점을 보인다. 사용자는 기술 사양, 생태계 지원, 비용, 문서화 수준과 같은 다차원적인 요소를 종합적으로 고려하여 자신의 프로젝트에 가장 적합한 트레이드오프를 결정해야 한다. 예를 들어, 안정적인 상업용 제품 개발이나 문서 의존도가 높은 초심자에게는 지원이 풍부한 Cube Orange+가 적합할 수 있다. 반면, 예산 제약이 크거나 특정 하드웨어 기능이 중요한 숙련된 개발자는 Holybro나 CUAV를 선택하여 비용 효율적인 맞춤형 시스템을 구축하는 것을 고려할 수 있다.

| 항목               | Cube Orange+                        | Holybro Pixhawk 6X               | CUAV V6X                            |
| ------------------ | ----------------------------------- | -------------------------------- | ----------------------------------- |
| **주 프로세서**    | STM32H757 18                        | STM32H753 34                     | STM32H743 34                        |
| **IMU 이중화**     | 3중 (ICM42688p 등) 18               | 3중 (ADIS16470, IIM-42652 등) 34 | 2중 (ICM-42688-P, BMI055) 34        |
| **핵심 차별점**    | ADS-B 수신기 통합, CubePilot 생태계 | 산업 등급 IMU, 높은 가성비       | 견고한 부품 품질, 다수의 DShot 출력 |
| **생태계 및 지원** | 매우 강력함 (CubePilot)             | 우수함 (Holybro)                 | 발전 중 (CUAV)                      |
| **가격대**         | 고가                                | 중가                             | 중고가                              |
| **주요 출처**      | 18                                  | 32                               | 31                                  |


Cube Orange+를 중심으로 한 드론 시스템을 구축하기 위해서는 비행 컨트롤러 외에도 다양한 구성 요소를 체계적으로 선정하고 통합하는 과정이 필요하다. 이 장에서는 ArduPilot 펌웨어를 기반으로 한 시스템 설계를 위한 핵심 요소와 선정 기준을 다룬다.


ArduPilot 기반의 안정적인 드론 시스템을 구축하기 위해서는 다음의 핵심 구성 요소들이 필수적으로 요구된다.35

- **기체**: 프레임, 모터, 전자 변속기(ESC), 프로펠러
- **비행 제어 시스템**: 비행 컨트롤러(본 보고서에서는 Cube Orange+), GPS 및 지자계 모듈
- **원격 제어 및 통신**: 6채널 이상의 RC 조종기 및 수신기, 지상관제시스템(GCS)과의 데이터 통신을 위한 텔레메트리 라디오
- **전원**: 리튬 폴리머(LiPo) 배터리 및 전용 충전기, 전원 모듈
- **지상국**: Mission Planner와 같은 GCS 소프트웨어가 설치된 PC 또는 태블릿

부품을 선정할 때는 ArduPilot 공식 문서에 명시된 호환 하드웨어 목록을 적극적으로 참조하는 것이 중요하다. 이 목록에는 검증된 GPS, ESC, 전원 모듈, 레인지파인더 등 방대한 주변기기 정보가 포함되어 있어 호환성 문제를 최소화하는 데 큰 도움이 된다.36


기체의 비행 성능은 프레임, 모터, 프로펠러의 상호작용에 의해 결정된다. 이들 구성 요소는 기체의 총 중량과 목표 비행 특성을 고려하여 신중하게 선택해야 한다.

- **추력 대 중량비 (Thrust-to-Weight Ratio, TWR)**: 안정적인 기동과 충분한 상승력을 확보하기 위해, 시스템의 총 추력은 기체의 총 중량(페이로드 포함)의 최소 2배 이상이 되는 것이 일반적이다. 이상적으로는 약 50%의 스로틀에서 호버링(정지 비행)이 가능해야 한다. 이는 다음과 같은 수식으로 표현될 수 있다.
  $$
  TWR = \frac{T_{total}}{W_{total}} \geq 2.0
  $$
  여기서 $T_{total}$은 모든 모터가 발생시키는 최대 추력의 합이며, $W_{total}$은 배터리와 페이로드를 포함한 기체의 총 이륙 중량이다.

- **선정 프로세스**:

  1. **총 중량 예측**: 카메라, 센서, 배터리 등 모든 부품을 포함한 최종 기체의 총 중량($W_{total}$)을 예측한다.
  2. **필요 추력 계산**: 목표 TWR(예: 안정적인 비행을 위해 2.5)을 설정하고, 필요한 총 추력($T_{total} = W_{total} \times TWR$)을 계산한다.
  3. **모터당 추력 계산**: 계산된 총 추력을 모터의 수로 나누어 각 모터가 담당해야 할 최소 추력을 산출한다.
  4. **모터 및 프로펠러 선정**: 모터 제조사(예: T-Motor)가 제공하는 성능 테스트 데이터를 참조하여, 사용할 배터리 전압(예: 6S, 22.2V)에서 계산된 추력을 가장 효율적으로 발생시키는 모터와 프로펠러 조합을 선택한다.

- **진동 문제**: 프로펠러의 재질은 비행 안정성에 큰 영향을 미칠 수 있다. 상대적으로 유연한 플라스틱 프로펠러는 특정 프레임 및 모터와의 공진으로 인해 비행 중 원치 않는 진동을 유발하고, 이는 FC의 센서 데이터를 오염시켜 비행 성능을 저하시킬 수 있다. 반면, 강성이 높은 카본 파이버(CF) 프로펠러는 이러한 진동 문제를 완화하고 더 안정적인 비행을 가능하게 하는 데 도움이 될 수 있다.38


ESC는 배터리로부터 전력을 공급받아 모터의 회전 속도를 제어하는 핵심 부품이다. 최신 ESC 기술은 단순한 속도 제어를 넘어 양방향 통신을 통한 고급 기능을 제공한다.

- **ESC 통신 프로토콜**: ArduPilot은 전통적인 아날로그 방식인 PWM, OneShot부터 최신 디지털 방식인 DShot까지 다양한 프로토콜을 지원한다.40
- **DShot 프로토콜의 장점**: DShot은 디지털 통신을 기반으로 하므로 아날로그 방식에 비해 여러 장점을 가진다.43
  - **ESC 보정 불필요**: 최소 및 최대 스로틀 값을 ESC에 학습시키는 보정 과정이 필요 없어 설정이 간편하다.
  - **높은 노이즈 내성**: 디지털 신호는 전자기 간섭(EMI)에 강해 더 신뢰성 있는 모터 제어가 가능하다.
  - **정밀한 제어**: 더 높은 해상도로 스로틀 값을 전달하여 정밀하고 빠른 모터 응답성을 구현할 수 있다.
- **Cube Orange+와 DShot 연결**: Cube Orange+와 같이 주 프로세서(FMU)와 별도의 I/O 보조 프로세서를 갖춘 보드의 경우, DShot 프로토콜은 I/O 프로세서를 거치지 않는 **AUX 출력 포트**에 연결해야만 정상적으로 작동한다. MAIN 출력 포트에 연결할 경우, DShot으로 설정하더라도 실제로는 PWM 신호로 출력되므로 반드시 주의해야 한다.41
- **ESC 텔레메트리**: BLHeli32, AM32와 같은 최신 펌웨어를 탑재한 ESC는 모터의 RPM, 전압, 전류, 온도 등의 상태 데이터를 실시간으로 FC에 전송하는 텔레메트리 기능을 지원한다.40 이 기능은 시스템의 상태를 진단하고 유지보수하는 데 매우 중요한 역할을 한다. 기존의 단방향 제어 방식에서는 특정 모터나 ESC에 문제가 발생해도 FC가 이를 인지할 방법이 없었지만, ESC 텔레메트리를 통해 각 모터의 상태를 개별적으로 모니터링할 수 있게 되었다. 이는 비행 중 특정 모터의 RPM 저하, 과열, 과전류 등의 이상 징후를 조기에 감지하여 심각한 고장이나 추락을 예방하는 데 결정적인 역할을 한다. 또한, 비행 후 로그 분석 시 각 모터의 데이터를 비교하여 미세한 성능 저하의 원인을 파악하고 예방 정비를 수행할 수 있게 해, 시스템의 전반적인 신뢰성과 안정성을 크게 향상시킨다.
  - **Bi-directional DShot**: 별도의 텔레메트리 배선 없이 기존의 DShot 신호선을 통해 양방향 통신을 구현하여 RPM과 같은 데이터를 전송하는 기술이다.40 이 RPM 데이터는 특히 제5부에서 다룰 하모닉 노치 필터 설정에 필수적이다.
  - **CAN ESC**: DroneCAN 프로토콜을 사용하는 ESC는 더 풍부한 텔레메트리 데이터와 높은 통신 신뢰성을 제공하지만, 배선이 더 복잡하고 비용이 높다는 단점이 있다.40


정확한 위치 정보는 자율 비행의 가장 기본이 되는 요소다. Cube Orange+는 다양한 수준의 정밀도를 제공하는 항법 시스템을 지원한다.

- **표준 GPS**: u-blox M8N, M9N, M10 칩셋을 기반으로 하는 일반적인 GPS 모듈은 수 미터 수준의 위치 정확도를 제공한다.47 CubePilot 생태계에서는 **Here 시리즈** GPS가 Cube 컨트롤러와 최적의 호환성을 보여준다.17
- **RTK (Real-Time Kinematic)**: RTK는 지상에 설치된 기준국(Base Station)이 수신한 위성 신호의 오차 정보를 이동국(Rover, 드론에 장착)으로 전송하여, 이동국이 자신의 위치 오차를 실시간으로 보정하는 기술이다. 이를 통해 위치 정확도를 센티미터 수준까지 획기적으로 향상시킬 수 있다.51 CubePilot의 **Here3+, Here4, HerePro**와 같은 제품은 RTK 기능을 지원하는 고정밀 GNSS(Global Navigation Satellite System) 모듈이다.17
- **연결 방식**:
  - **UART**: 전통적인 시리얼 통신 방식으로, FC의 GPS 또는 TELEM 포트에 연결한다.52
  - **CAN (DroneCAN)**: Here3, Here4와 같은 최신 GPS 모듈은 CAN 버스를 통해 연결된다. CAN 통신은 UART에 비해 더 긴 케이블 길이를 허용하고, 전자기 노이즈에 대한 내성이 강하며, 버스 형태로 여러 장치를 쉽게 연결할 수 있는 장점을 가진다.48
- **고급 항법 기능**:
  - **이중 GPS (Dual GPS / GPS Blending)**: 2개의 GPS 모듈을 장착하여 하나의 GPS 신호가 불량할 때 다른 하나를 사용하거나, 두 GPS의 측정값을 융합(Blending)하여 위치 정보의 신뢰성과 정확도를 높이는 기능이다.48
  - **GPS for Yaw (Moving Baseline)**: 일정 거리를 두고 장착된 2개의 RTK GPS 모듈 간의 위상차를 계산하여, 지자계(나침반) 센서 없이도 매우 정확한 기체의 방위각(Yaw)을 결정하는 기술이다. 이는 고압선 주변이나 철골 구조물 근처와 같이 자기장 간섭이 심한 환경에서 안정적인 비행을 가능하게 한다.48


안정적인 전원 시스템은 드론의 모든 전자 장치가 올바르게 작동하기 위한 가장 근본적인 전제 조건이다.

- **전원 모듈 (Power Module)**: 배터리와 기체의 전력 분배 보드(PDB) 사이에 연결되어, 배터리의 전압과 소모 전류를 측정하여 FC에 전달하고, FC 및 기타 주변기기(GPS, 수신기 등)에 안정화된 5V 전원을 공급하는 핵심 부품이다.1 Cube Orange+ 표준 세트에는 보통 'Power Brick Mini'가 포함된다.1
- **이중화 전원 공급**: Cube Orange+는 2개의 독립된 전원 입력 포트(POWER1, POWER2)를 지원한다. 주 전원 모듈을 POWER1에 연결하고, 별도의 BEC(Battery Eliminator Circuit)나 보조 전원 모듈을 POWER2에 연결하면, 주 전원 공급 라인에 문제가 발생했을 때 시스템이 자동으로 보조 전원으로 전환하여 비행을 지속할 수 있다.6
- **배선 및 관리**: 높은 전류가 흐르는 배터리 및 ESC 전원선은 가능한 한 굵고 짧게 배선하고, GPS나 RC 수신기와 같은 민감한 신호선과는 물리적으로 최대한 멀리 떨어뜨려 전자기 간섭을 최소화해야 한다. 모든 납땜 부위는 비행 중의 진동에도 견딜 수 있도록 견고하게 처리해야 한다.57

전원 시스템의 안정성은 단순히 부품을 이중으로 구성하는 것만으로 보장되지 않는다. 사용자 포럼에서 보고된 FC 재부팅 문제의 로그를 분석한 결과, 높은 스로틀 상태에서 배터리 전압이 급격히 강하(Voltage Sag)하면서 FC에 공급되는 5V 전압(VCC)이 불안정해지는 현상이 관찰되었다.57 이 문제의 근본 원인은 저품질의 전원 모듈 불량이었으며, 검증된 제품으로 교체하자 문제가 해결되었다. 또 다른 사례에서는 부적절한 배선으로 인해 두 개의 다른 FC를 하나의 배터리에 연결했다가 그라운드 루프 또는 전압 역류로 추정되는 현상으로 인해 Cube Orange와 텔레메트리 모듈이 영구적으로 손상되기도 했다.58 이러한 사례들은 전원 시스템이 드론에서 가장 치명적인 단일 장애 지점이 될 수 있음을 명확히 보여준다. 따라서, 초기 투자 비용이 더 들더라도 Mauch와 같이 신뢰성이 검증된 고품질 전원 모듈을 사용하고, 배선 규격을 철저히 준수하며, 비행 후에는 반드시 로그 파일의 전압 관련 데이터를 분석하여 시스템의 건강 상태를 지속적으로 확인하는 것이 매우 중요하다. 이중화는 하나의 안전장치일 뿐, 근본적인 전원 공급의 품질을 대체할 수는 없다.


조종사와 드론, 그리고 지상관제시스템 간의 원활한 통신은 안전하고 효율적인 임무 수행의 필수 요소다.

- **RC 시스템**: 수동 조종 및 비행 모드 변경을 위해 최소 6채널 이상의 RC 조종기 및 수신기가 필요하다.35 특히 FrSky Taranis 시리즈(Q X7, X9D)와 같이 OpenTX/EdgeTX 펌웨어를 사용하는 조종기는 높은 설정 유연성과 Yaapu 텔레메트리 스크립트와 같은 고급 기능을 지원하여 ArduPilot 사용자들 사이에서 널리 사용된다.59
- **수신기 연결**: S.BUS, FPort, CRSF(Crossfire, ELRS)와 같은 시리얼 기반의 디지털 프로토콜을 사용하는 것이 권장된다. 이들은 단일 케이블로 다수의 채널 정보를 전달하고 응답 속도가 빠르다. 수신기는 Cube Orange+의 RCIN 포트 또는 설정된 UART 포트에 연결할 수 있다.53
- **텔레메트리 라디오**: 비행 중인 드론의 상태(위치, 고도, 속도, 배터리 잔량 등)를 실시간으로 지상관제시스템에 전송하고, GCS에서 새로운 명령(예: 임무 변경)을 드론으로 전송하기 위해 필요하다. 900MHz 또는 433MHz 대역을 사용하는 SiK 라디오(예: RFD900x, Holybro Telemetry Radio)가 일반적으로 사용된다.17
- **통합 솔루션 (Herelink)**: CubePilot 생태계의 일부인 Herelink은 RC 조종, 텔레메트리 데이터 통신, 그리고 고화질(HD) 영상 전송 기능을 하나의 통합된 장치로 제공한다. 이는 복잡한 배선을 단순화하고 사용자 편의성을 극대화하는 강력한 솔루션이다.7


정확한 하드웨어 조립과 배선은 드론 시스템의 신뢰성을 보장하는 가장 기본적인 단계다. 작은 배선 오류 하나가 값비싼 부품의 손상이나 추락으로 이어질 수 있으므로, 각별한 주의가 요구된다.


Cube Orange+ 표준 ADS-B 캐리어 보드의 각 포트와 핀 배열을 정확히 이해하는 것은 모든 배선의 시작점이다. 부정확한 연결은 부품 손상의 주된 원인이 되므로, 아래의 핀맵을 반드시 참조하여 작업을 진행해야 한다.

| 포트 (Port)         | 핀 번호 | 신호명 (Signal) | 전압 (Volt) | 설명 (Description)                           |
| ------------------- | ------- | --------------- | ----------- | -------------------------------------------- |
| **TELEM1 / TELEM2** | 1       | VCC             | +5V         | 텔레메트리, 컴패니언 컴퓨터 등 주변기기 전원 |
|                     | 2       | TX (OUT)        | +3.3V       | 데이터 송신 (FC -> 주변기기)                 |
|                     | 3       | RX (IN)         | +3.3V       | 데이터 수신 (주변기기 -> FC)                 |
|                     | 4       | CTS             | +3.3V       | 하드웨어 흐름 제어 (Clear to Send)           |
|                     | 5       | RTS             | +3.3V       | 하드웨어 흐름 제어 (Request to Send)         |
|                     | 6       | GND             | GND         | 접지                                         |
| **GPS1**            | 1       | VCC             | +5V         | GPS 모듈 전원                                |
|                     | 2       | TX (OUT)        | +3.3V       | 데이터 송신 (FC -> GPS)                      |
|                     | 3       | RX (IN)         | +3.3V       | 데이터 수신 (GPS -> FC)                      |
|                     | 4       | SCL (I2C2)      | +3.3V       | I2C 클럭 (외부 나침반용)                     |
|                     | 5       | SDA (I2C2)      | +3.3V       | I2C 데이터 (외부 나침반용)                   |
|                     | 6       | Safety Button   | GND         | 안전 스위치 입력                             |
|                     | 7       | Button LED      | GND         | 안전 스위치 LED 출력                         |
|                     | 8       | GND             | GND         | 접지                                         |
| **CAN1 / CAN2**     | 1       | VCC             | +5V         | CAN 장치 전원                                |
|                     | 2       | CAN_H           | +12V        | CAN High 신호                                |
|                     | 3       | CAN_L           | +12V        | CAN Low 신호                                 |
|                     | 4       | GND             | GND         | 접지                                         |
| **POWER1 / POWER2** | 1       | VCC             | +5V         | FC 주 전원 입력                              |
|                     | 2       | VCC             | +5V         | FC 주 전원 입력 (이중화)                     |
|                     | 3       | CURRENT         | +3.3V       | 전류 센서 신호 입력                          |
|                     | 4       | VOLTAGE         | +3.3V       | 전압 센서 신호 입력                          |
|                     | 5       | GND             | GND         | 접지                                         |
|                     | 6       | GND             | GND         | 접지 (이중화)                                |
| **USB**             | 1       | VCC             | +5V         | USB 전원                                     |
|                     | 2       | D+ (DP)         | +3.3V       | USB 데이터 +                                 |
|                     | 3       | D- (DM)         | +3.3V       | USB 데이터 -                                 |
|                     | 4       | GND             | GND         | 접지                                         |
|                     | 5       | BUZZER          | Battery     | 부저 출력 (배터리 전압)                      |
| **I2C (외부)**      | 1       | VCC             | +5V         | I2C 장치 전원                                |
|                     | 2       | SCL             | +3.3V       | I2C 클럭                                     |
|                     | 3       | SDA             | +3.3V       | I2C 데이터                                   |
|                     | 4       | GND             | GND         | 접지                                         |

주요 출처: 6


각 구성 요소를 올바른 포트에 정확하게 연결하는 것이 중요하다.

- **GPS + Compass**:
  - **UART 기반 GPS**: GPS 모듈의 6핀 커넥터를 FC의 **GPS1** 포트에 연결한다. 이때, FC의 TX 핀은 GPS의 RX 핀으로, FC의 RX 핀은 GPS의 TX 핀으로 교차 연결해야 한다.53 GPS 모듈에 내장된 I2C 나침반은 GPS1 포트의 SCL/SDA 핀을 통해 자동으로 연결된다. 만약 별도의 I2C 나침반을 사용한다면 외부 I2C 포트에 연결한다.52
  - **CAN 기반 GPS (Here3/4 등)**: 모듈의 4핀 커넥터를 FC의 **CAN1** 또는 **CAN2** 포트에 직접 연결한다. 배선이 매우 단순하며, TX/RX 교차 연결이 필요 없어 오류 가능성이 적다.54
- **RC 수신기**:
  - **S.BUS/PPM**: 수신기의 출력 신호선을 FC의 **RCIN** 포트에 연결한다.
  - **CRSF/FPort (Crossfire, ELRS 등)**: 이 프로토콜들은 양방향 통신을 위해 완전한 UART 포트가 필요하다. **TELEM1** 또는 **TELEM2** 포트에 연결하고, Mission Planner에서 해당 시리얼 포트의 `SERIALx_PROTOCOL` 파라미터를 23 (RCIN)으로 설정해야 한다.53
- **전자 변속기 (ESC)**:
  - **PWM/OneShot**: 모터 번호에 맞춰 **MAIN OUT** 또는 **AUX OUT** 포트의 신호 핀에 연결한다.
  - **DShot**: Cube Orange+와 같이 IOMCU가 있는 보드에서는 반드시 **AUX OUT** 포트(AUX 1-6)에 연결해야 한다. MAIN OUT 포트는 DShot 프로토콜을 지원하지 않는다.43 모터 번호와 AUX 출력 번호를 정확히 매핑하여 연결한다.
- **텔레메트리 라디오**: 안정적인 장거리 데이터 통신을 위해 하드웨어 흐름 제어(CTS/RTS)를 지원하는 **TELEM1** 또는 **TELEM2** 포트에 연결하는 것이 가장 이상적이다.19
- **전원 모듈**: 배터리에서 나온 주 전원선을 전원 모듈의 입력단(Battery)에 연결하고, 출력단(ESC/PDB)을 전력 분배 보드에 연결한다. 전원 모듈에 포함된 6핀 신호 케이블을 FC의 **POWER1** 포트에 연결한다. 이 케이블을 통해 FC에 전원이 공급되고 배터리 전압/전류 정보가 전달된다.19


비행 성능과 안정성은 기체의 기계적, 전기적 완성도에 크게 좌우된다.

- **진동 관리**:
  - Cube Orange+는 내부에 진동 절연 IMU를 갖추고 있지만, 기체 자체의 진동을 최소화하는 것이 근본적인 해결책이다. 프로펠러와 모터의 밸런스를 정밀하게 맞추고, 프레임의 모든 나사를 단단히 조여 유격이 없도록 한다.
  - FC에 연결되는 케이블들이 너무 팽팽하게 당겨지거나 헐거워져서 비행 중 떨리면서 진동을 FC에 전달하지 않도록 케이블 타이 등으로 적절히 정리하고 고정해야 한다.56
  - 초기 비행 후 로그 파일을 분석하여 진동 수준(`VIBE` 메시지)을 반드시 확인하고, 필요하다면 제5부에서 다룰 하모닉 노치 필터를 설정하여 특정 주파수의 진동을 억제해야 한다.38
- **전자기 간섭 (EMI) 관리**:
  - **GPS/나침반 배치**: GPS 및 지자계(나침반) 모듈은 EMI에 가장 민감한 부품이다. 모터, ESC, 배터리 전원선과 같이 강한 자기장을 발생시키는 부품들로부터 물리적으로 최대한 멀리, 그리고 높이 이격시켜야 한다. 이를 위해 GPS 마스트를 사용하는 것이 강력히 권장된다.52
  - **배선 분리**: 고전류가 흐르는 전원선(배터리-PDB-ESC)과 저전압 신호선(RC, GPS, 텔레메트리 등)은 서로 겹치거나 나란히 배치하지 말고 최대한 분리한다. 신호선의 경우, 가능하다면 신호선과 접지선을 함께 꼬아서(twisted pair) 외부 노이즈의 유입을 상쇄시키는 것이 좋다.52
  - **접지(Ground) 관리**: 시스템의 모든 전자 부품은 공통 접지(common ground)를 공유해야 한다. 부적절한 접지 연결은 그라운드 루프를 형성하여 노이즈를 유발하고, 심각한 경우 부품을 손상시킬 수 있다.58 모든 접지선이 PDB 또는 공통 접지 포인트에 안정적으로 연결되었는지 반드시 확인해야 한다.


하드웨어 조립이 완료되면, 비행 컨트롤러에 생명을 불어넣는 소프트웨어 설정 단계로 넘어간다. 이 과정은 Mission Planner 지상관제시스템을 통해 이루어진다.


- **Mission Planner 설치**: ArduPilot 프로젝트의 공식 GCS인 Mission Planner를 PC에 다운로드하여 설치한다. Mission Planner는 펌웨어 설치, 파라미터 설정, 비행 계획 수립, 실시간 모니터링 등 모든 작업을 수행하는 통합 환경이다.35
- **펌웨어 설치**:
  1. Cube Orange+와 PC를 마이크로 USB 케이블로 연결한다.54
  2. Mission Planner를 실행하고 초기 화면에서 'Setup' 탭으로 이동한 뒤, 좌측 메뉴에서 'Install Firmware'를 선택한다.66
  3. 화면 우측 상단의 COM 포트 드롭다운 메뉴에서 Cube Orange+에 해당하는 포트를 선택하고, 전송 속도(Baud rate)는 115200으로 설정한다.
  4. 화면에 표시되는 기체 아이콘(멀티콥터, 비행기, 로버 등) 중에서 자신의 기체 타입에 맞는 아이콘을 클릭한다.
  5. Mission Planner가 서버에서 최신 안정 버전의 펌웨어를 자동으로 다운로드하고, 보드를 부트로더 모드로 전환한 뒤 펌웨어를 설치한다. 이 과정에서 여러 확인 창이 나타날 수 있으며, 지시에 따라 진행한다.54
- **펌웨어 버전 확인**: Cube Orange+의 하드웨어는 ArduCopter 4.2.3 버전 이상에서 공식적으로 지원된다. 따라서, 특별한 이유가 없는 한 항상 최신 안정(Stable) 버전의 펌웨어를 설치하여 하드웨어와의 완벽한 호환성을 확보해야 한다.28


- **FC 연결**: 펌웨어 설치가 완료되면, Mission Planner 우측 상단의 'Connect' 버튼을 클릭하여 FC와 연결한다. 연결이 성공하면 FC의 각종 파라미터를 읽어오기 시작한다.54
- **프레임 타입 설정**: 'Setup' -> 'Mandatory Hardware' -> 'Frame Type' 메뉴로 이동한다. 여기서 자신의 기체에 맞는 프레임 클래스(예: Quad, Hexa, Octo)와 물리적인 모터 배열 형태(예: X, +)를 정확하게 선택해야 한다. 이 설정은 각 모터에 추력을 어떻게 분배할지를 결정하는 '모터 믹서(Motor Mixer)'를 정의하므로, 가장 먼저 수행해야 할 필수적인 단계다.67
- **초기 파라미터 설정**: 'Config/Tuning' -> 'Standard Params' 메뉴에서 'Initial Parameter Setup' 또는 유사한 기능을 사용할 수 있다. 이 기능은 기체의 크기(주로 프로펠러 직경)를 입력받아, 해당 크기에 적합한 기본적인 PID 게인 값과 필터 값들을 자동으로 설정해 준다. 이는 복잡한 튜닝 과정의 좋은 시작점을 제공하여 초기 비행의 안정성을 높이는 데 도움이 된다.38


정확한 센서 보정은 안정적인 비행 자세 제어의 전제 조건이다.

- **가속도계 보정**: 'Setup' -> 'Mandatory Hardware' -> 'Accel Calibration' 메뉴에서 진행한다. 'Calibrate Accel' 버튼을 클릭한 후, 화면의 지시에 따라 기체를 수평, 왼쪽 면이 아래로, 오른쪽 면이 아래로, 기수가 아래로, 기수가 위로, 뒤집은 상태 등 6가지의 정적인 자세로 고정시킨다. 각 자세에서 'Click when Done' 버튼을 눌러 중력 가속도 벡터를 측정하고, 이를 통해 센서의 오프셋과 스케일 오차를 보정한다.65

- **지자계(나침반) 보정**: 'Setup' -> 'Mandatory Hardware' -> 'Compass' 메뉴에서 진행한다.

  - **절차**: 'Start' 버튼을 누른 후, 기체를 들어 모든 축(Roll, Pitch, Yaw) 방향으로 천천히 그리고 부드럽게 360도 회전시켜, 지구 자기장에 대한 센서의 반응을 다각도에서 수집한다. Mission Planner가 충분한 데이터를 수집하면 보정이 자동으로 완료된다.68

  - **주의사항**: 이 과정은 컴퓨터, 스피커, 철제 가구 등 자기장을 발생시키는 물체로부터 멀리 떨어진 야외나 넓은 실내 공간에서 수행해야 한다.69

  - **나침반 문제 해결**: 나침반 보정 실패는 하드웨어 결함보다는 환경적 요인이나 설정 오류일 가능성이 훨씬 높다. 사용자 포럼에서는 Cube Orange의 내장 나침반이 기체 내부의 전원선이나 다른 부품들로부터 발생하는 자기장 간섭에 취약하다는 보고가 자주 있다.70 보정 후 오프셋 값이 비정상적으로 높게 나오거나(일반적으로 600 이상), 비행 준비 중 'Inconsistent Compass' 오류가 발생한다면, 이는 자기장 간섭이 심각하다는 신호다. 이 경우, 문제 해결을 위해 다음과 같은 체계적인 접근이 필요하다. 첫째, 나침반의 물리적 위치를 재검토하여 간섭원으로부터 최대한 이격시킨다. 둘째, Mission Planner의 나침반 설정 페이지에서 외부 GPS 모듈에 내장된 나침반을 최우선(Primary)으로 사용하도록 설정(

    `COMPASS_PRIO1_ID` 파라미터 등)하고, 간섭이 의심되는 내장 나침반은 사용하지 않도록 비활성화한다. 셋째, 비행 후 로그 파일의 `MAG` 필드를 분석하여 비행 중 자기장 세기의 변화를 확인하고 간섭의 원인을 추적한다. 이러한 접근은 하드웨어 교체와 같은 성급한 결정을 내리기 전에 문제를 정확히 진단하고 해결하는 데 도움이 된다.


조종기와의 정확한 연동은 수동 제어 및 비상 상황 대응의 핵심이다.

- **조종기 보정**: 'Setup' -> 'Mandatory Hardware' -> 'Radio Calibration' 메뉴에서 'Calibrate Radio' 버튼을 클릭한다. 조종기의 모든 스틱(Roll, Pitch, Yaw, Throttle)과 할당된 스위치들을 최대 및 최소 범위까지 여러 번 움직여 각 채널의 PWM 신호 범위를 FC에 정확하게 인식시킨다.65
- **비행 모드 설정**: 'Setup' -> 'Mandatory Hardware' -> 'Flight Modes' 메뉴에서 조종기의 특정 스위치 위치에 원하는 비행 모드를 할당한다.67
  - **PWM 값 매핑**: ArduPilot은 특정 PWM 값의 범위에 따라 비행 모드를 인식한다. 예를 들어, 6개의 비행 모드를 하나의 채널에 할당하려면, 조종기에서 해당 채널이 6개의 서로 다른 PWM 값을 출력하도록 설정(믹싱 기능 활용)해야 한다.59
  - **권장 모드 구성**: 초기 설정 시에는 안전을 위해 **Stabilize**(자세 안정화), **AltHold**(고도 유지), **Loiter**(위치 유지)와 같이 조종사의 개입이 용이한 기본 모드들을 반드시 포함해야 한다. 기체에 대한 확신이 생긴 후에 **Auto**(자동 임무), **Guided**(GCS 유도), **RTL**(자동 복귀)과 같은 고급 자동 비행 모드를 추가하는 것이 바람직하다.73

| 비행 모드     | 설명                                                         | 제어 수준            | GPS 필요 여부   |
| ------------- | ------------------------------------------------------------ | -------------------- | --------------- |
| **Stabilize** | 기체의 수평 자세를 자동으로 유지. 스로틀, 롤, 피치, 요는 수동 제어. | 수동 자세 제어       | 아니요          |
| **AltHold**   | Stabilize 모드에 고도 유지가 추가됨.                         | 고도 자동, 자세 수동 | 아니요 (기압계) |
| **Loiter**    | GPS를 사용하여 현재 위치와 고도를 유지. 스틱 조작 시 해당 방향으로 이동. | 위치 및 고도 자동    | 예              |
| **PosHold**   | Loiter와 유사하나, 스틱을 놓으면 현재 위치를 유지. 스틱 조작 시 수동처럼 제어. | 위치 및 고도 자동    | 예              |
| **RTL**       | 이륙 지점 또는 사전에 설정된 지점으로 자동 복귀 후 착륙.     | 완전 자동            | 예              |
| **Auto**      | Mission Planner로 사전에 계획된 임무(Waypoint)를 자동으로 수행. | 완전 자동            | 예              |
| **Guided**    | GCS 지도에서 클릭한 지점으로 이동하도록 명령.                | 완전 자동            | 예              |
| **Acro**      | 자동 수평 유지 기능이 없는 완전 수동 비행 모드. 곡예 비행용. | 완전 수동            | 아니요          |

주요 출처: 73


- **ESC 프로토콜 설정**: DShot 프로토콜을 사용하기 위해서는 'Config/Tuning' -> 'Full Parameter List'에서 관련 파라미터를 설정해야 한다. `SERVO_BLH_MASK` (또는 `MOT_PWM_TYPE` 등 펌웨어 버전에 따라 다를 수 있음) 파라미터를 사용하여 DShot을 사용할 출력 채널을 비트마스크 형태로 지정한다. 예를 들어, AUX 1~4번 포트를 사용한다면 해당 채널들에 대한 비트를 활성화해야 한다.43
- **ESC 보정 (PWM의 경우)**: 만약 전통적인 PWM 또는 OneShot 방식의 ESC를 사용한다면, 반드시 ESC 보정 절차를 거쳐야 한다. Mission Planner의 'Setup' -> 'Mandatory Hardware' -> 'ESC Calibration' 메뉴의 'All-at-once' 보정 기능을 사용하거나, 각 ESC를 수신기 스로틀 채널에 직접 연결하여 수동으로 보정할 수 있다. DShot 프로토콜을 사용하는 ESC는 이 과정이 전혀 필요 없다.69
- **모터 테스트**: 모든 설정이 완료되면, **프로펠러를 반드시 제거한 상태**에서 'Setup' -> 'Optional Hardware' -> 'Motor Test' 유틸리티를 사용한다. 이 유틸리티를 통해 각 모터를 개별적으로 구동시켜 모터의 번호 순서와 회전 방향이 기체 프레임 타입에 맞게 설정되었는지 최종적으로 확인한다.
  - **순서 확인**: ArduPilot 공식 문서에 명시된 프레임 타입별 모터 순서 다이어그램과 실제 모터의 구동 순서가 일치하는지 비교한다.53
  - **방향 수정**: 만약 모터의 회전 방향이 잘못되었다면, 물리적으로 모터와 ESC를 연결하는 3개의 전선 중 2개의 위치를 바꾸거나, BLHeliSuite 또는 웹 기반 ESC-Configurator와 같은 소프트웨어를 사용하여 ESC의 내부 파라미터를 변경하여 회전 방향을 반대로 설정할 수 있다.69


- **페일세이프 설정**: 비상 상황 발생 시 기체의 손실을 방지하기 위한 자동 대응 절차를 설정하는 것은 매우 중요하다. 'Config/Tuning' -> 'Standard Params' -> 'Failsafe' 메뉴에서 설정할 수 있다.
  - **RC Failsafe**: 조종기와의 통신이 끊어졌을 때의 동작을 설정한다. 일반적으로 RTL(Return to Launch) 또는 Land(즉시 착륙)로 설정하여 기체가 통제 불능 상태에 빠지는 것을 방지한다.67
  - **Battery Failsafe**: 비행 중 배터리 전압이 사전에 설정된 임계값 이하로 떨어졌을 때의 동작을 설정한다. 1단계(Low Battery)에서는 RTL을, 2단계(Critical Battery)에서는 Land를 수행하도록 설정하는 것이 일반적이다.24
- **배터리 모니터 설정**: 'Setup' -> 'Optional Hardware' -> 'Battery Monitor' 메뉴에서 사용 중인 전원 모듈의 종류를 선택하고, 실제 멀티미터로 측정한 배터리 전압과 비교하여 전압 분배기(Voltage Divider) 값을 보정한다. 전류 센서 또한 보정을 통해 정확한 배터리 소모량을 측정할 수 있도록 설정한다. 정확한 배터리 모니터링은 안전한 비행 시간 예측과 배터리 페일세이프 기능의 정상적인 작동을 위해 필수적이다.56


기본적인 비행 준비가 완료된 드론은 고급 센서와 소프트웨어 튜닝을 통해 더욱 정교하고 지능적인 시스템으로 발전할 수 있다. 이 장에서는 고급 기능의 통합 방법과 시스템 운용 중 발생할 수 있는 일반적인 문제들의 해결 방안을 다룬다.


- **LiDAR (Light Detection and Ranging)**:

  - **용도**: 레이저를 이용하여 거리를 정밀하게 측정하는 센서로, 지면과의 거리를 정확하게 측정하여 정밀한 고도를 유지(Altitude Hold)하거나, 전방의 장애물을 감지하여 회피(Obstacle Avoidance)하는 데 사용된다.79

  - **연결 및 설정**: 대부분의 LiDAR는 UART 또는 I2C 인터페이스를 통해 FC와 연결된다. Lightware SF20, Benewake TF03, RPLIDAR 등 다양한 제품이 ArduPilot에서 지원된다.79 연결 후에는 Mission Planner에서 해당 시리얼 포트의 

    `SERIALx_PROTOCOL` 파라미터를 9 (Rangefinder)로 설정하고, `RNGFNDx_TYPE` 파라미터에서 사용 중인 LiDAR 모델을 선택해야 한다. 또한, 센서의 최소/최대 측정 거리(`RNGFNDx_MIN_CM`, `RNGFNDx_MAX_CM`)와 장착 방향(`RNGFNDx_ORIENT`)을 기체에 맞게 정확히 설정해야 한다.79

  - **전원 문제**: 일부 고성능 LiDAR는 FC의 5V 출력만으로는 순간적인 전류 소모량을 감당하지 못해 불안정하게 작동할 수 있다. 이 경우, 배터리 전압을 5V로 낮춰주는 별도의 BEC(Battery Eliminator Circuit)를 사용하여 LiDAR에 독립적이고 안정적인 전원을 공급하는 것이 권장된다.81

- **컴패니언 컴퓨터 (Companion Computer)**:

  - **용도**: Raspberry Pi, NVIDIA Jetson Nano, 또는 Qualcomm QRB5165 기반 보드와 같은 컴패니언 컴퓨터는 FC의 연산 능력을 초과하는 복잡한 작업을 수행한다.83 예를 들어, 인공지능(AI) 기반 객체 인식, 시각적 동시 위치 추정 및 지도 작성(VSLAM), 복잡한 경로 계획 등의 알고리즘을 실행하고, 그 결과를 MAVLink 프로토콜을 통해 FC에 전달하여 드론을 제어한다.
  - **연결 및 통합**: 컴패니언 컴퓨터는 일반적으로 FC의 텔레메트리 포트(UART)에 연결되어 MAVLink 메시지를 주고받는다. Cube Orange+는 5개의 UART 포트를 제공하여 컴패니언 컴퓨터와의 통합이 매우 용이하다.22 Mistral사의 MRD5165 Eagle Kit와 같이 Cube Orange+와 컴패니언 컴퓨터가 하나의 보드에 통합된 형태로 제공되는 제품도 있어 개발 편의성을 높여준다.83


최상의 비행 성능을 끌어내기 위해서는 물리적인 조립을 넘어 소프트웨어적인 튜닝이 필수적이다.

- **진동 분석**: 비행 후 생성된 로그 파일(`.bin`)을 Mission Planner에서 열어 `VIBE` 메시지를 분석한다. X, Y, Z축의 진동 수준이 권장 허용치(일반적으로 15m/s/s 이하)를 초과하는지 확인한다. 과도한 진동은 가속도계와 자이로스코프 센서 데이터를 오염시켜 제어 루프를 불안정하게 만들고, 심한 경우 비행 불능 상태를 초래할 수 있다.38

- **하모닉 노치 필터 (Harmonic Notch Filter)**: 하모닉 노치 필터는 현대 고성능 드론 튜닝에서 선택이 아닌 필수 과정으로 자리 잡았다. 과거에는 물리적인 댐퍼에 의존하여 진동을 줄였지만, 이는 모든 주파수 대역의 진동을 효과적으로 제거하지 못했다. 하모닉 노치 필터는 모터의 회전으로 인해 발생하는 특정 주파수의 진동만을 정밀하게 목표하여 감쇠시키는 디지털 필터다. 모터에서 발생하는 진동은 스로틀 레벨에 따라 주파수가 변하는 특징을 가지는데, 이 필터는 모터의 RPM과 연동하여 동적으로 필터의 중심 주파수를 변경함으로써 매우 효과적으로 노이즈를 제거한다.

  - **설정**: 이 필터를 최적으로 사용하기 위해서는 실시간 모터 RPM 데이터가 필수적이다. 제2.3절에서 언급된 Bi-directional DShot 기능이 있는 ESC를 사용하면, 별도의 센서 없이 ESC 텔레메트리를 통해 이 RPM 정보를 얻을 수 있다.45

    `INS_HNTCH_ENABLE` 파라미터를 1로 설정하여 필터를 활성화하고, `INS_HNTCH_MODE`를 1(Throttle-based) 또는 4(ESC Telemetry-based)로 설정한다. ESC 텔레메트리를 사용하는 것이 훨씬 더 정확하고 효과적이다.

  - **효과**: 하모닉 노치 필터를 올바르게 설정하면 자이로 센서로 유입되는 노이즈가 극적으로 감소한다. 이는 더 깨끗한 제어 루프를 의미하며, 결과적으로 더 높은 PID 게인 값을 설정할 수 있게 하여 기체의 반응성과 안정성을 모두 크게 향상시킨다. 이는 하드웨어(ESC) 선택이 소프트웨어 튜닝의 성능 한계를 결정하는 대표적인 사례다.


드론 제작 및 운용 과정에서는 다양한 문제가 발생할 수 있다. 다음 표는 커뮤니티 포럼 등에서 자주 보고되는 일반적인 문제들과 그에 대한 체계적인 진단 및 해결 방안을 정리한 것이다.

| 증상 (Symptom)                                        | 예상 원인 (Probable Cause)                                   | 해결 방안 (Solution)                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **FC가 비행 중 또는 높은 스로틀에서 재부팅됨**        | 전원 모듈 불량, 배선 접촉 불량, 배터리 성능 저하로 인한 순간적인 전압 강하(Voltage Sag). | 비행 로그에서 VCC 및 배터리 전압을 확인한다. 신뢰성 있는 고품질 전원 모듈로 교체한다. 모든 전원 관련 배선 및 납땜 상태를 점검한다. 57 |
| **Mission Planner/QGC에 연결되지 않음**               | PC의 USB 드라이버 문제, 잘못된 COM 포트 선택, USB 케이블 불량, FC 펌웨어 손상. | PC의 장치 관리자에서 드라이버를 확인하고 재설치한다. 다른 USB 포트와 케이블을 사용해 본다. Mission Planner에서 펌웨어를 재설치(필요시 Full chip erase 옵션 사용)한다. 66 |
| **나침반 보정 실패 또는 'Inconsistent Compass' 오류** | 주변의 강한 자기장 간섭(전원선, 모터, 철 구조물), 잘못된 나침반 우선순위 설정, 외부 나침반 미사용. | 자기장 간섭이 없는 장소에서 다시 보정을 시도한다. 나침반을 간섭원으로부터 최대한 멀리 재배치한다. 외부 나침반을 주(Primary)로 설정하고 내장 나침반을 비활성화한다. 70 |
| **모터가 떨리거나(pulsing) 비행이 불안정함**          | 높은 기계적 진동, 부적절한 PID/필터 튜닝, 프로펠러/모터 밸런스 불량. | 프로펠러와 모터의 밸런스를 확인한다. 비행 로그에서 진동 수준(`VIBE`)을 분석한다. 하모닉 노치 필터를 설정한다. Autotune 기능을 수행하여 PID 값을 최적화한다. 38 |
| **LiDAR, 카메라 등 특정 주변기기가 작동하지 않음**    | 잘못된 배선(TX/RX 교차 연결 오류), 주변기기 전원 부족, 호환되지 않는 펌웨어, 잘못된 파라미터 설정. | 공식 문서의 배선도를 다시 확인한다. 주변기기에 별도의 안정적인 전원을 공급해 본다. `SERIALx_PROTOCOL`, `RNGFNDx_TYPE` 등 관련 파라미터가 올바르게 설정되었는지 확인한다. 81 |


Cube Orange+는 강력한 듀얼코어 프로세서, 다중화된 고신뢰성 센서 시스템, 탁월한 확장성, 그리고 성숙한 ArduPilot 오픈소스 생태계의 지원을 바탕으로, 현재 시장에서 가장 진보된 상업용 및 연구용 비행 컨트롤러 중 하나로 평가된다.

ADS-B 수신기 통합, 센티미터급 정밀도의 RTK 항법 지원, 그리고 컴패니언 컴퓨터와의 원활한 연동 기능은 Cube Orange+ 기반의 드론이 단순한 원격 조종 비행체를 넘어, 복잡한 임무를 스스로 판단하고 수행하는 지능형 자율 시스템으로 발전할 수 있는 무한한 잠재력을 보여준다.

향후 무인 항공 기술은 인공지능 기반의 완전 자율 비행, 다수의 드론이 협력하는 군집 비행, 그리고 GPS 신호가 없는 환경에서의 항법 기술 등으로 빠르게 발전할 것이다. Cube Orange+의 강력한 하드웨어 성능과 개방형 소프트웨어 생태계는 이러한 미래 기술을 연구하고 구현하는 데 이상적인 플랫폼을 제공한다. 본 보고서에서 다룬 기술적 고찰을 바탕으로, 개발자들은 더 안전하고, 더 지능적이며, 더 효율적인 차세대 무인 항공 시스템을 구축하여 다양한 산업 분야의 혁신을 이끌어 나갈 수 있을 것이다.


1. The Cube Orange+ Standard Set - Aero Systems West, 8월 14, 2025에 액세스, https://aerosystemswest.com/product/the-cube-orange-standard-set/
2. Cube Orange+ Standard Set ADS-B | CubePilot - IR-LOCK, 8월 14, 2025에 액세스, https://irlock.com/products/cube-orange-plus-standard-set
3. The Advantages of the Pixhawk Orange Cube Plus over Other Flight Controllers, 8월 14, 2025에 액세스, https://techmaharaja.com/2023/01/31/the-advantages-of-the-pixhawk-orange-cube-plus-over-other-flight-controllers-2/
4. Cube Orange+ Mini Set, Flight Controller - World Drone Market, 8월 14, 2025에 액세스, https://www.worldronemarket.com/product/cube-orange-mini-set/
5. CubePilot | Autopilot-on-Module | Blue Manufactured in USA | Blue Assembled in USA | Pixhawk Original Team, 8월 14, 2025에 액세스, https://www.cubepilot.com/
6. The Cube Black - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-thecube-overview.html
7. CubePilot | CubePilot, 8월 14, 2025에 액세스, https://docs.cubepilot.org/user-guides
8. The CubePilot Hardware & Software Ecosystem for UAVs - Unmanned Systems Technology, 8월 14, 2025에 액세스, https://www.unmannedsystemstechnology.com/2021/06/the-cubepilot-hardware-software-ecosystem-for-uavs/
9. The Cube Orange+ - IR-LOCK, 8월 14, 2025에 액세스, https://irlock.com/products/the-cube-orange-plus
10. CubePilot The Cube Orange+ - Lumenier, 8월 14, 2025에 액세스, https://www.lumenier.com/products/cubepilot-the-cube-orange
11. The Cube Orange+ - HeliEngadin, 8월 14, 2025에 액세스, https://www.heliengadin.com/products/the-cube-orange-plus
12. The Cube Orange+ : Drones, UAV, OnyxStar, MikroKopter, ArduCopter, RPAS : AltiGator, drones, radio controlled aircrafts: aerial survey, inspection, video & photography, 8월 14, 2025에 액세스, https://drones.altigator.com/the-cube-orange-p-42757.html
13. The Cube Orange+ Standard Set (ADS-B Carrier Board) - BZB UAS, 8월 14, 2025에 액세스, https://www.bzbuas.com/shop/hx4-06222-the-cube-orange-standard-set-ads-b-carrier-board-548
14. CubePilot The Cube Orange+ Standard Set - GetFPV, 8월 14, 2025에 액세스, https://www.getfpv.com/cubepilot-the-cube-orange-standard-set.html
15. Cube Orange+ UAV Autopilot & Set | Precision Drone Flight System - Aeroboticshop.com, 8월 14, 2025에 액세스, https://aeroboticshop.com/en-na/products/cube-orange-standard-set-ads
16. Cubepilot The Cube Orange+ - Aero Systems West, 8월 14, 2025에 액세스, https://aerosystemswest.com/product/cubepilot-the-cube-orange-pixhawk-2-1/
17. Cubepilot The Cube Orange+ full bundle with Here3+ GNSS, RFD 900x-US, 8월 14, 2025에 액세스, https://aerosystemswest.com/product/cubepilot-the-cube-orange-full-bundle-with-here3-gnss-rfd-900x-us/
18. CubePilot Cube Orange+ Flight Controller | PX4 自动驾驶用户指南 ..., 8월 14, 2025에 액세스, https://docs.px4.io/v1.14/zh/flight_controller/cubepilot_cube_orangeplus.html
19. The Cube Orange/+ With ADSB-In Overview - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-thecubeorange-overview.html
20. CubePilot Cube Orange Flight Controller | PX4 User Guide (v1.14), 8월 14, 2025에 액세스, https://docs.px4.io/v1.14/en/flight_controller/cubepilot_cube_orange.html
21. CubePilot The Cube Orange+ (IMU V8) - GetFPV, 8월 14, 2025에 액세스, https://www.getfpv.com/cubepilot-the-cube-orange-imu-v8.html
22. CubePilot Cube Orange+ Flight Controller | PX4 Guide (main), 8월 14, 2025에 액세스, https://docs.px4.io/main/en/flight_controller/cubepilot_cube_orangeplus
23. Cube Orange+ | Advanced open-source autopilot for drones & robotics, 8월 14, 2025에 액세스, https://www.unmannedsystemstechnology.com/company/cubepilot/cube-orange/
24. CubePilot The Cube Orange + Standard Set - RobotShop, 8월 14, 2025에 액세스, https://www.robotshop.com/products/cube-orange-standard-set
25. CubePilot PixHawk Cube Orange+ Mini Set - ReadyMadeRC, 8월 14, 2025에 액세스, https://www.readymaderc.com/products/details/pix-hawk-cube-orange-plus-mini-set
26. How to choose what Cube to use? - Cubepilot, 8월 14, 2025에 액세스, https://discuss.cubepilot.org/t/how-to-choose-what-cube-to-use/4318
27. ROB498-flight/instructions/hardware/orange_cube_plus.md at main - GitHub, 8월 14, 2025에 액세스, https://github.com/utiasSTARS/ROB498-flight/blob/main/instructions/hardware/orange_cube_plus.md
28. Differences Cube Orange vs Orange+ - Pixhawk family - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/differences-cube-orange-vs-orange/92535
29. Cube Orange+ Standard Set - Foxtech drone store, 8월 14, 2025에 액세스, https://www.foxtechfpv.com/pixhawk-cube-orange-standard-set.html
30. Functionality differences among Pixhawk Flight Controllers - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/functionality-differences-among-pixhawk-flight-controllers/101025
31. Cube Orange vs. CUAV Pixhawk V6X : r/Multicopter - Reddit, 8월 14, 2025에 액세스, https://www.reddit.com/r/Multicopter/comments/128p3zi/cube_orange_vs_cuav_pixhawk_v6x/
32. Should I use Pixhawk 6x? - PX4 Autopilot, 8월 14, 2025에 액세스, https://discuss.px4.io/t/should-i-use-pixhawk-6x/46160
33. Pixhawk 5x vs CubePilot Orange? : r/diydrones - Reddit, 8월 14, 2025에 액세스, https://www.reddit.com/r/diydrones/comments/ul6tij/pixhawk_5x_vs_cubepilot_orange/
34. Controller Comparison - Holybro Docs, 8월 14, 2025에 액세스, https://docs.holybro.com/autopilot/controller-comparison
35. What You Need to Build a MultiCopter - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/what-you-need.html
36. Peripheral Hardware - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-optional-hardware.html
37. Choosing an Autopilot - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-autopilots.html
38. Cube Orange+ IB version - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/cube-orange-ib-version/126797
39. Cube Orange + Vs Cube Orange + (IB) - Cube Autopilot, 8월 14, 2025에 액세스, https://discuss.cubepilot.org/t/cube-orange-vs-cube-orange-ib/14442
40. ESC (Electronic Speed Controls) - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-esc-guide.html
41. Ardupilot --> ESC : r/diydrones - Reddit, 8월 14, 2025에 액세스, https://www.reddit.com/r/diydrones/comments/qmityd/ardupilot_esc/
42. ESC Protocols - Which are supported and stable? What are you running?, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/esc-protocols-which-are-supported-and-stable-what-are-you-running/40647
43. DShot ESCs - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-dshot-escs.html
44. ESCs and Motors - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-escs-and-motors.html
45. ESC Telemetry - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-esc-telemetry.html
46. DroneCAN ESCs - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-uavcan-escs.html
47. Ardupilot GPS - eBay, 8월 14, 2025에 액세스, https://www.ebay.com/shop/ardupilot-gps?_nkw=ardupilot+gps
48. Setup & Getting Started (Ardupilot) - Holybro Docs, 8월 14, 2025에 액세스, https://docs.holybro.com/gps-and-rtk-system/zed-f9p-h-rtk-series/setup-and-getting-started-ardupilot
49. ArduPilot APM 2.6 GPS Module w Compass for Drone / UAV Flight Controller - Tinkersphere, 8월 14, 2025에 액세스, https://tinkersphere.com/arduino-compatible-components/374-ardupilot-apm-26-gps-module-w-compass-for-drone-uav-flight-controller.html
50. GPS/Compass (landing page) - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-positioning-landing-page.html
51. CubePilot Here+ RTK GPS - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-here-plus-gps.html
52. UBlox GPS + Compass Module - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-installing-3dr-ublox-gps-compass-module.html
53. Typical Autopilot Wiring Connections - Copter documentation, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-flight-controller-wiring.html
54. The CubeOrange Platform - CSE Senior Design Knowledge Base, 8월 14, 2025에 액세스, https://cseseniordesignkb.uta.edu/the-cubeorange-platform/
55. Here 3+ GPS With Cube orange - Setup Tutorial - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=iEwf2ZefkT8
56. Cube Orange Power Monitor Configuration - Copter 4.3 - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/cube-orange-power-monitor-configuration/108383
57. Cube Orange Automatic Reboot Issue, 8월 14, 2025에 액세스, https://discuss.cubepilot.org/t/cube-orange-automatic-reboot-issue/8676
58. Common power source to Pixhawk 2.4.8 and CubePilot CubeOrange - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/common-power-source-to-pixhawk-2-4-8-and-cubepilot-cubeorange/133519
59. How To: Calibration of FrSky Taranis X9D Plus with S3 6 position switch - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=3IQUlDd8JAc
60. Mission Planner Radio Calibration - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/mission-planner-radio-calibration/80512
61. Orange Cube / Mission Planner/ x7 Taranisq - ArduCopter - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/orange-cube-mission-planner-x7-taranisq/89875
62. Can't get manual flight with Taranis X9D + 2019, X8R, Cube Orange - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/cant-get-manual-flight-with-taranis-x9d-2019-x8r-cube-orange/81954
63. Hexacopter Drone Build Project – Part 4 Cube Orange Plus and Kore Carrier Board Initial Setup - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=5t4QqKYQOWc
64. Herelink For Pixhawk & The Cube - Initial Setup and Connection Overview - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=s39eUckPx14
65. Arduplane Build 2025, Video 2: Flashing a CubePilot Orange+ and performing the basic settings - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=YwfjLcbF5N0
66. Cube orange solid orange light - PX4 Autopilot, 8월 14, 2025에 액세스, https://discuss.px4.io/t/cube-orange-solid-orange-light/39984
67. Mandatory Hardware Configuration - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/configuring-hardware.html
68. Part 1 - Hardware and Setup: Complete ArduPilot Tuning Guide (ArduCopter) - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=4pkSnBqA_m4
69. Getting Started Guide for Ardupilot - MicoAir Tech, 8월 14, 2025에 액세스, https://micoair.com/docs/getting-started-guide-for-ardupilot/
70. Orange Cube Compass Calibration - ArduCopter - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/orange-cube-compass-calibration/67770
71. RC Transmitter Flight Mode Configuration - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/common-rc-transmitter-flight-mode-configuration.html
72. RC Transmitter Flight Mode Configuration - Mission Planner documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/planner/docs/common-rc-transmitter-flight-mode-configuration.html
73. Flight Modes - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/flight-modes.html
74. Guided Mode - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/ac2_guidedmode.html
75. Mission Planner Flight PLAN, 8월 14, 2025에 액세스, https://ardupilot.org/planner/docs/mission-planner-flight-plan.html
76. Pixhawk Orange cube Remote controller settings. - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=qC67iixUE5E
77. Electronic Speed Controller (ESC) Calibration - Copter documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/copter/docs/esc-calibration.html
78. ESC Calibration - Plane documentation - ArduPilot, 8월 14, 2025에 액세스, https://ardupilot.org/plane/docs/common-esc-calibration.html
79. Configuring TF03 with UART Interface on Ardupilot Flight Stack using Cube Orange Flight Controller - Evelta, 8월 14, 2025에 액세스, [https://evelta.com/content/datasheets/Configuring%20TF03%20on%20Ardupilot%20using%20Cub%20Orange%20(UART).pdf](https://evelta.com/content/datasheets/Configuring TF03 on Ardupilot using Cub Orange (UART).pdf)
80. How to connect rplidar S2 to cube pilot - ArduCopter - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/how-to-connect-rplidar-s2-to-cube-pilot/97258
81. Rangefinders (I2C) won't work with Cube Orange but do work with Cube Black - ArduCopter - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/rangefinders-i2c-wont-work-with-cube-orange-but-do-work-with-cube-black/85817
82. Orange cube+ connecting with Lidar issue - ArduCopter - ArduPilot Discourse, 8월 14, 2025에 액세스, https://discuss.ardupilot.org/t/orange-cube-connecting-with-lidar-issue/104531
83. MRD5165 Eagle Kit w/ Cube Pilot (Cube Orange) - Mistral Solutions, 8월 14, 2025에 액세스, https://www.mistralsolutions.com/media_coverage/mistral-mrd5165-cubepilot/
84. AI-powered Companion Computer for Autopilot Modules | UST, 8월 14, 2025에 액세스, https://www.unmannedsystemstechnology.com/2024/08/ai-powered-companion-computer-for-autopilot-modules/
85. CubePilot The Cube Orange+ - GetFPV, 8월 14, 2025에 액세스, https://www.getfpv.com/cubepilot-the-cube-orange.html
86. Camera Hotshoe feedback not working on Cube Orange / Issue #17807 / PX4/PX4-Autopilot, 8월 14, 2025에 액세스, https://github.com/PX4/PX4-Autopilot/issues/17807
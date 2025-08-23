# 자동차 및 임베디드 비전 시스템을 위한 FPD-Link III 인터페이스


현대 자동차 기술의 발전은 첨단 운전자 보조 시스템(ADAS), 자율 주행, 고해상도 인포테인먼트 디스플레이의 급격한 확산을 이끌었으며, 이는 차량 내 데이터 처리량의 폭발적인 증가를 야기했습니다.1 이러한 시스템의 핵심은 차량 주변에 배치된 다수의 센서(카메라, RADAR, LIDAR, ToF)로부터 수집된 방대한 양의 데이터를 중앙 처리 장치(ECU)로 안정적으로 전송하는 것입니다.2 이 과정에서 발생하는 핵심적인 기술적 과제는 최대 15미터에 달하는 긴 거리에서, 혹독한 자동차 환경의 전자기 간섭(EMI)과 진동 속에서도 압축되지 않은 고품질 비디오 및 센서 데이터를 손실 없이 전송하는 것입니다.1 직렬 변환기/역직렬 변환기(Serializer/Deserializer, SerDes) 기술은 이러한 문제를 해결하기 위한 핵심 솔루션으로 부상했으며, 넓은 병렬 데이터 버스를 고속 직렬 스트림으로 변환하여 단일 케이블을 통해 전송함으로써 배선 복잡성, 무게, 비용을 획기적으로 줄입니다.1


자동차 산업의 패러다임이 '기계'에서 '전자 장치'로 전환되면서, 차량은 단순한 이동 수단을 넘어 정교한 센서와 컴퓨팅 시스템의 집합체가 되었습니다. 서라운드 뷰 시스템(SVS), 차선 유지 보조(LKA), 자동 긴급 제동(AEB)과 같은 ADAS 기능은 실시간으로 고해상도 비디오 데이터를 처리해야 하며, 이는 수 기가비트(Gbps)에 달하는 대역폭을 요구합니다.1 기존의 병렬 인터페이스 방식으로는 이러한 대용량 데이터를 전송하기 위해 수십 가닥의 배선이 필요하며, 이는 차량의 무게와 비용을 증가시키고 케이블 하네스의 복잡성을 가중시키는 주요 원인이 됩니다. 또한, 길어진 배선은 EMI에 취약해져 데이터의 신뢰성을 저해하는 심각한 문제를 야기합니다. SerDes 기술은 이러한 병렬 데이터를 직렬화하여 단 하나의 차동 쌍 또는 동축 케이블로 전송함으로써 물리적인 제약을 극복하고, 고속 데이터 전송의 신뢰성과 효율성을 보장하는 필수적인 기술로 자리 잡았습니다.


FPD-Link의 발전 과정은 단순한 기술적 진보를 넘어, 자동차 산업의 비용 및 무게 절감이라는 강력한 요구에 부응하기 위한 공격적인 통합과 단순화의 역사입니다. 각 세대는 속도를 높이는 것을 넘어, 물리적 복잡성을 근본적으로 줄이는 방향으로 진화했습니다.

- **FPD-Link I (LVDS SerDes 시대):** 1996년에 처음 등장한 FPD-Link는 저전압 차동 신호(LVDS) 기술을 사용하여 여러 개의 꼬임쌍선 케이블(데이터용, 클럭용)을 통해 직렬화된 데이터를 전송했습니다.9 이는 SerDes 기술의 가능성을 열었지만, 분리된 클럭과 데이터 라인 간의 타이밍 스큐(skew) 문제로 인해 케이블 길이가 제한되고, 여러 가닥의 케이블로 인한 부피와 비용 문제가 여전히 남아있었습니다.9
- **FPD-Link II (클럭 임베딩):** 2006년에 도입된 FPD-Link II는 클럭 신호를 데이터 스트림에 내장(embedding)하는 혁신을 통해 타이밍 스큐 문제를 근본적으로 해결했습니다. 이로써 단일 꼬임쌍선만으로 비디오 데이터 전송이 가능해졌고, 더 길고 저렴한 케이블을 사용할 수 있게 되었습니다.9 또한 AC 커플링을 도입하여 노이즈 내성을 향상시켰습니다.
- **FPD-Link III (궁극의 통합):** 이 보고서의 핵심 주제인 FPD-Link III는 2010년에 등장하며 또 한 번의 도약을 이루었습니다. 가장 큰 혁신은 **양방향 제어 채널**의 통합과 고속 신호를 위한 **전류 모드 로직(CML)** 으로의 전환입니다. 이를 통해 비디오, 제어 신호, 그리고 Power-over-Coax(PoC) 기술을 통한 전력까지 단일 동축 케이블 또는 차폐 꼬임쌍선(STP) 케이블로 통합할 수 있게 되었습니다.1 이는 배선 복잡성, 무게, 비용을 극적으로 감소시키는 패러다임의 전환이었으며, 원격 센서 및 디스플레이 모듈의 설계를 근본적으로 단순화시켰습니다.


FPD-Link III 시스템은 기본적으로 한 쌍의 칩셋으로 구성됩니다. 센서나 비디오 소스 측에 위치한 직렬 변환기(Serializer, SER), 예를 들어 DS90UB953-Q1은 MIPI CSI-2와 같은 병렬 인터페이스로부터 데이터를 받아 고속 FPD-Link III 직렬 스트림으로 변환합니다.13 이 직렬 스트림은 단일 케이블을 통해 전송되어 프로세서나 디스플레이 측에 위치한 역직렬 변환기(Deserializer, DES), 예를 들어 DS90UB954-Q1에 도달합니다. DES는 수신된 직렬 스트림을 다시 원래의 병렬 인터페이스로 변환하여 출력합니다.13 이 SER/DES 쌍은 시스템 설계자에게 고속 직렬 전송의 복잡성을 추상화하여 '투명한(transparent)' 링크를 제공하며, 마치 병렬 인터페이스가 직접 연결된 것처럼 시스템을 설계할 수 있게 해줍니다.5


FPD-Link III의 기술적 우수성은 세 가지 핵심 요소, 즉 고속 순방향 채널, 양방향 제어 채널, 그리고 Power-over-Coax(PoC) 메커니즘의 유기적인 결합에 있습니다. 이 섹션에서는 각 기술적 기둥을 심층적으로 분석합니다.


순방향 채널은 카메라의 고해상도 비디오 데이터를 ECU로 전송하는 주된 통로입니다.

- **데이터 속도 및 비디오 지원:** FPD-Link III는 5-85 MHz 16에서 최대 100 MHz 5에 이르는 넓은 범위의 픽셀 클럭을 지원합니다. 이를 통해 HD (720p) 4, Full HD (1080p) 1, 그리고 최대 4MP 해상도의 카메라까지 지원 가능하며 17, 순방향 채널 대역폭은 최대 4.16 Gbps에 달합니다.17
- **임베디드 클럭 및 인코딩:** DC 밸런싱 및 스크램블링된 데이터와 함께 클럭을 임베딩하는 방식을 사용합니다.14 이는 신호의 DC 성분을 제거하여 AC 커플링 인터커넥트를 가능하게 하고, 길고 손실이 많은 케이블을 통해서도 DES에서 안정적인 클럭 복구를 보장하는 핵심 기술입니다.5
- **물리 계층(Physical Layer):** FPD-Link III는 고속 신호를 위해 기존의 LVDS에서 CML(Current-Mode Logic)로 전환했습니다. CML은 3 Gbit/s 이상의 데이터 속도에 더 적합하며, 특히 동축 케이블의 단일 도체를 구동하는 데 효과적이어서 임피던스 불연속에 대한 내성이 뛰어납니다.11
- **데이터 프레이밍:** 순방향 채널 프레임은 35비트 페이로드로 구성됩니다. 여기에는 픽셀 데이터를 위한 24비트(RGB888), 클럭 및 DC 밸런싱 비트, 그리고 I2C, GPIO, HDCP 등 제어 데이터를 다중화하기 위한 7비트가 포함됩니다.19 이 구조는 제어 데이터가 비디오 데이터와 근본적인 수준에서 어떻게 인터리빙되는지를 보여줍니다.


FPD-Link III의 가장 큰 특징 중 하나는 동일한 물리적 매체를 통해 순방향 채널과 동시에 독립적으로 작동하는 저지연, 전이중(full-duplex) 역방향 제어 채널입니다.4 이 채널은 시간 다중화 방식이 아니므로 실시간 제어 애플리케이션에 매우 중요합니다.2

- **지원 프로토콜:**
  - **I2C:** 가장 일반적인 사용 사례로, 호스트 프로세서(마스터)가 원격 주변 장치(예: 이미지 센서)를 마치 로컬 I2C 버스에 연결된 것처럼 투명하게 제어할 수 있게 합니다.9 역방향 채널은 최대 400kHz의 I2C 속도를 지원합니다.5
  - **GPIO:** 프레임 동기화 트리거, 상태 표시 LED 등 범용 입출력 신호를 양방향으로 전달하는 데 필수적입니다.6
  - **SPI:** 일부 듀얼 채널 장치에서 사용 가능하며, 더 빠른 제어 데이터 전송을 위해 두 번째 FPD-Link 채널을 활용합니다.19
- **I2C 구현 상세:** 제어 채널의 '투명성'은 실제로는 상당한 복잡성을 우아하게 추상화한 결과물입니다. 호스트 프로세서는 간단한 I2C 버스를 보는 반면, FPD-Link 칩셋은 내부 오실레이터와 클럭 스트레칭을 사용하여 두 개의 분리된 버스 도메인을 관리하는 복잡한 실시간 프로토콜 변환을 수행합니다.15 이 추상화는 큰 가치를 제공하지만, 설계자는 효과적인 디버깅을 위해 그 기저의 메커니즘과 잠재적인 타이밍 변동을 이해해야 합니다.
  - **클럭 스트레칭(Clock Stretching):** 시스템은 로컬 및 원격 I2C 버스 간의 통신을 관리하기 위해 클럭 스트레칭을 사용합니다. 메시지가 원격 버스로 반복될 때 로컬 버스는 스트레칭 상태로 유지되며, 그 반대의 경우도 마찬가지입니다.15
  - **장치 별칭(Device Aliasing):** 단일 DES 허브에 여러 개의 동일한 장치가 연결될 때 주소 충돌을 피하기 위해, FPD-Link III는 각 원격 장치에 고유한 '별칭(alias)' 주소를 할당하는 기능을 제공합니다.6 호스트는 이 별칭을 사용하여 통신하고, SerDes는 이를 장치의 실제 ID로 변환합니다.
  - **패스스루 모드(Pass-Through Modes):** `I2C_PASS_THROUGH` 및 `I2C_PASS_ALL` 레지스터 설정은 링크를 통해 전파되는 I2C 메시지를 세밀하게 제어하여 불필요한 버스 트래픽을 방지합니다.19


PoC는 단일 케이블 솔루션의 정점을 이루는 기술로, 데이터와 전력을 하나의 동축 케이블에 통합합니다.

- **기본 원리:** PoC는 DES(ECU 측)에서 원격 SER 및 센서 모듈로 DC 전력을 데이터 전송에 사용되는 동일한 동축 케이블을 통해 공급하는 기술입니다.4 이는 주파수 영역을 분리함으로써 가능해집니다. 즉, DC는 전력, 저주파 AC는 역방향 채널, 고주파 AC는 순방향 채널에 할당됩니다.23
- **PoC 필터 네트워크의 핵심 부품:**
  - **인덕터 (LPOC):** SER와 DES 양단의 DC 전원 공급 라인에 직렬로 배치됩니다. 이들의 기능은 AC 데이터 신호(순방향 및 역방향 모두)에 대해 **높은 임피던스**를 제공하여 신호가 전원 레일로 단락되는 것을 방지하는 동시에, DC 전류에 대해서는 **낮은 임피던스**를 제공하는 것입니다.17
  - **AC 커플링 커패시터 (CAC):** 데이터 경로에 직렬로 배치됩니다. 이들의 기능은 AC 데이터 신호에 대해 **낮은 임피던스**를 제공하여 SER와 DES 간에 신호가 통과하도록 하는 동시에, DC 전력에 대해서는 **높은 임피던스**(개방 회로)를 제공하여 DC가 SerDes 데이터 핀으로 유입되는 것을 차단하는 것입니다.17
- **PoC 네트워크의 핵심 설계 고려사항:** PoC 네트워크 설계는 FPD-Link III 시스템에서 가장 중요한 하드웨어 설계 과제 중 하나입니다. 단일 케이블의 단순성이라는 목표는 부품 선택과 PCB 레이아웃에 매우 민감한 복잡한 광대역 수동 필터 네트워크를 설계해야 하는 과제를 직접적으로 만들어냅니다. 여기서의 실패는 성능 저하가 아닌 전체 링크의 실패로 이어집니다.
  - **광대역 임피던스:** 가장 중요한 과제는 PoC 인덕터 네트워크가 역방향 채널의 MHz 대역부터 순방향 채널의 GHz 대역에 이르는 방대한 주파수 범위(예: 10 MHz ~ 2.2 GHz)에 걸쳐 높은 임피던스를 제공해야 한다는 것입니다.17 단일 인덕터는 기생 커패시턴스로 인한 낮은 자기 공진 주파수(SRF) 때문에 이를 효과적으로 수행할 수 없습니다.
  - **다중 부품 필터:** 해결책은 저-중 주파수 대역을 위한 큰 값의 인덕터(예: 10µH)와 고주파수 대역(>1 GHz)을 위한 페라이트 비드를 결합한 다단 필터를 사용하여 필요한 광대역 고임피던스를 달성하는 것입니다.17
  - **인덕터 포화 전류:** 선택된 인덕터는 카메라 모듈의 예상 DC 부하 전류보다 훨씬 높은 포화 전류 정격을 가져야 합니다. 전류가 이 정격을 초과하면 인덕터 코어가 포화되어 인덕턴스가 급격히 감소하고 더 이상 AC 신호를 효과적으로 차단하지 못해 링크 실패를 초래합니다.17
  - **DC 저항 (IR 강하):** 케이블(15m 길이의 경우 9-10 옴에 달할 수 있음)과 인덕터의 DC 저항은 전압 강하(IR drop)를 유발하여 전력 전송 효율을 저하시킵니다. 이를 완화하기 위해 더 높은 PoC 전압(예: >10V)을 사용하고 카메라 모듈에서 스텝다운 DC-DC 컨버터를 사용하여 필요한 저전압(1.8V, 3.3V 등)을 생성하는 것이 권장됩니다.17


이 섹션은 이론을 실제 애플리케이션으로 전환하여, 실제 시스템을 설계하는 엔지니어에게 실용적이고 실행 가능한 지침을 제공합니다.


- **체계적인 설계 흐름:** 복잡한 링크 분석으로 넘어가기 전에 기본적인 전원 인가 확인부터 시작하는 체계적인 설계 접근 방식의 중요성이 강조됩니다.13 이 과정은 플로우차트를 참조하여 안내될 수 있습니다.
- **신호 무결성 및 전송 라인:**
  - FPD-Link III 신호는 4Gbps 이상의 고속 신호이므로 반사를 방지하기 위해 제어된 임피던스 전송 라인(동축의 경우 일반적으로 50옴 단일 종단)으로 라우팅해야 합니다.13
  - **시간 영역 반사 측정법(TDR):** TDR 분석을 사용하여 PCB 레이아웃, 커넥터, 케이블 전환부의 임피던스 불연속성을 식별하고 수정해야 합니다.13
  - **삽입 손실 및 반사 손실:** 전체 채널(PCB 트레이스, PoC 필터, 커넥터, 케이블)은 DES의 이퀄라이저가 신호를 복구할 수 있도록 특정 삽입 손실 및 반사 손실 마스크를 충족해야 합니다.13
- **전원 공급 및 디커플링:**
  - SerDes 칩에 저잡음 전원 공급 장치를 제공해야 하며, 특히 PLL 전원 핀(예: DS90UB953의 LPF1/LPF2 핀)에는 특정 필터 커패시터가 필요합니다.13
  - 장치 전원 핀 가까이에 커패시터를 배치하는 적절한 디커플링 전략을 사용해야 합니다.
- **PoC 필터 레이아웃:** 이는 매우 중요한 레이아웃 문제입니다.
  - 가장 높은 주파수 필터 부품(페라이트 비드)을 동축 커넥터에 가장 가깝게 배치해야 합니다.17
  - 인덕터 패드 아래에 "안티패드(anti-pads)"(접지면의 빈 공간)를 사용하여 기생 커패시턴스를 줄이고 인덕터의 고주파 임피던스를 보존해야 합니다.17
- **물리적 상호 연결:**
  - **케이블 선택:** 동축 케이블이 일반적으로 사용되며, 우수한 신호 무결성과 낮은 비용을 제공합니다. 차폐 꼬임쌍선(STP)도 지원되며 더 유연할 수 있습니다.4
  - **커넥터:** 자동차 등급의 FAKRA 커넥터가 표준으로 사용되며, 진동이 심한 환경을 위해 설계된 견고하고 안전한 연결을 제공합니다.3


FPD-Link III의 진단 기능은 초기 설계 및 디버그뿐만 아니라, 자동차 기능 안전(FuSa) 및 장기 신뢰성을 위한 핵심 가치 제안입니다. 이 진단 제품군은 SerDes 링크를 단순한 데이터 파이프에서 자가 모니터링 하위 시스템으로 변환하며, 이는 안전 필수 자동차 애플리케이션에 대한 협상 불가능한 요구 사항입니다.

- **링크 설정("Lock"):** 시스템 가동의 첫 단계는 DES가 들어오는 직렬 스트림에 "Lock"을 달성했는지 확인하는 것입니다. 이는 일반적으로 상태 핀이나 레지스터 비트로 표시됩니다.13
- **내장 자가 테스트(BIST):** 필수적인 진단 도구입니다. DES는 SER에게 알려진 의사 난수 비트 시퀀스(PRBS) 패턴을 보내도록 명령할 수 있습니다. 그런 다음 DES는 수신된 패턴의 오류를 확인하여 실제 비디오 소스 없이 물리적 링크 무결성을 최고 속도로 테스트할 수 있습니다.2
- **적응형 이퀄라이제이션(AEQ):**
  - DES에는 케이블 손실로 인한 신호 저하(심벌 간 간섭, ISI)를 보상하기 위한 적응형 이퀄라이저가 포함되어 있습니다.12
  - 이 기능은 전원 인가 시 다양한 케이블 길이와 유형에 자동으로 조정됩니다.2
  - 중요한 것은, 이 기능이 온도 주기와 진동으로 인한 차량 수명 동안의 케이블 **노후화** 및 성능 저하도 보상한다는 점입니다. AEQ 레벨은 레지스터에서 읽을 수 있어 채널의 "상태"나 품질을 평가하는 강력한 진단 도구를 제공합니다.2 이를 통해 링크 마진이 사라져 고장이 임박했음을 나타내는 예측 유지 보수가 가능합니다.
- **패턴 생성:** SER 및 DES 칩 모두 내부 테스트 패턴을 생성할 수 있습니다. 이를 통해 SER-이미저 링크와 DES-프로세서 링크를 독립적으로 검증하여 시스템의 결함을 분리하는 데 도움이 됩니다.13


- **아키텍처:** DS90UB960-Q1 또는 DS90UB964A-Q1과 같은 쿼드 포트 DES 허브를 사용하여 최대 4개의 카메라 입력을 집계합니다.2
- **출력 집계:** 허브는 4개의 들어오는 비디오 스트림을 결합하여 주 프로세서(예: TI TDAx SoC)를 위해 하나 또는 두 개의 MIPI CSI-2 인터페이스로 출력합니다.6 프로세서는 제한된 수의 CSI-2 입력만 가지고 있기 때문에 이 기능은 매우 중요합니다.
- **MIPI 가상 채널(Virtual Channels):** DES 허브는 각 카메라 입력을 다른 MIPI 가상 채널(VC) ID에 매핑할 수 있습니다. 이를 통해 프로세서는 단일 물리적 CSI-2 포트를 통해 도착하는 4개의 개별 비디오 스트림을 구별할 수 있습니다.7
- **프레임 동기화:** 서라운드 뷰 시스템과 같은 애플리케이션에서는 모든 카메라가 정확히 같은 순간에 프레임을 캡처하는 것이 절대적으로 중요합니다. DES 허브는 모든 연결된 SER에 역방향 채널을 통해 동기화된 프레임 동기화 신호를 동시에 전송하여 이를 관리합니다. 이 트리거는 DES에서 내부적으로 생성되거나 외부 신호에서 공급될 수 있습니다.4

아래 표는 FPD-Link III 시스템, 특히 PoC 회로를 설계할 때 하드웨어 엔지니어를 위한 실용적인 체크리스트를 제공합니다. 이는 복잡한 RF/아날로그 이론을 실행 가능한 설계 규칙으로 변환하여 설계 위험을 줄이고 개발을 가속화합니다.

**표 1: Power-over-Coax (PoC) 필터 설계 체크리스트**

| 설계 매개변수                        | 요구사항/권장사항           | 근거                                                | 참조 |
| ------------------------------------ | --------------------------- | --------------------------------------------------- | ---- |
| 인덕터 임피던스 (10MHz-1GHz)         | >2kΩ                        | 저/중 주파수 신호 감쇠 방지. 10µH 인덕터 사용 권장. | 17   |
| 페라이트 비드 임피던스 (1GHz-2.2GHz) | >2kΩ                        | 고주파수 신호 감쇠 방지.                            | 24   |
| 인덕터 포화 전류 (Isat)              | Isat>1.5× 최대 부하 전류    | 부하 시 인덕턴스 강하 방지.                         | 17   |
| AC 커플링 커패시터 (CAC) 값          | 0.033µF (4Gbps 모드용)      | DC를 차단하면서 AC 신호 통과.                       | 13   |
| CAC 전압 정격                        | >2× 최대 PoC 전압           | DC 바이어스 디레이팅 고려.                          | 17   |
| PoC 입력 전압                        | >10V 권장                   | IR 강하 및 인덕터 전류 요구량 감소.                 | 17   |
| PCB 레이아웃                         | 인덕터 아래에 안티패드 사용 | 기생 커패시턴스를 줄여 SRF 증가.                    | 17   |
| PoC 입력 디커플링                    | >20µF                       | 스위처/센서의 저주파 노이즈 필터링.                 | 17   |


FPD-Link III는 자동차 환경의 혹독한 요구 사항을 충족시키기 위해 개발된 기술이지만, 그 견고함과 신뢰성은 다른 산업 분야로의 확장을 가능하게 했습니다. 자동차를 위해 요구된 '과도한 엔지니어링'은 다른 까다로운 산업 애플리케이션에 완벽하게 부합하는 특성들을 만들어냈습니다.


- **서라운드 뷰 시스템 (SVS):** FPD-Link III의 전형적인 애플리케이션입니다. 4개 이상의 동기화된 광각(어안) 카메라가 필요하며, 데이터는 프로세서(예: TI TDAx)로 전송되어 왜곡 보정, 스티칭을 통해 가상의 버드아이 뷰를 생성합니다.4 저지연 및 동기화 기능이 여기서 매우 중요합니다.
- **전방 및 후방 카메라 (FVC/RVC):** 자동 긴급 제동(AEB), 차선 유지 보조(LKA), 어댑티브 크루즈 컨트롤(ACC), 주차 보조와 같은 안전 기능에 사용됩니다.1 고해상도와 신뢰성이 가장 중요합니다.
- **운전자 및 실내 모니터링 시스템 (DMS):** 운전자 졸음 및 행동을 모니터링하거나 제스처 인식을 위해 카메라를 사용하는 새로운 애플리케이션입니다.1 반응성이 좋은 시스템을 위해 저지연이 핵심입니다.
- **차세대 센서 연결:** 높은 대역폭과 견고한 링크는 원격 위성 RADAR, LIDAR, ToF(Time-of-Flight) 센서 연결에도 적합하여, 단순한 카메라 링크를 넘어 범용 고속 센서 링크로서의 다재다능함을 보여줍니다.2
- **인포테인먼트 및 디스플레이:** 카메라에 중점을 두지만, FPD-Link III는 뒷좌석 엔터테인먼트나 디지털 계기판용 고화질 디스플레이를 구동하는 데에도 사용되며, 종종 HDCP 복사 방지 기능과 함께 사용됩니다.1


이러한 애플리케이션들은 자동차와 유사한 요구 사항을 공유합니다: 긴 케이블 길이(1m 이상), 혹독한 환경(진동, 온도, EMI)에서의 작동, 그리고 높은 신뢰성입니다.3 자동차 시장을 위해 이루어진 막대한 R&D 투자와 규모의 경제를 활용하여, 산업 및 로봇 분야의 기업들은 자신들의 까다로운 애플리케이션을 위한 고도로 신뢰할 수 있고 사전 검증된 솔루션을 얻을 수 있습니다.

- **로봇 공학:** 카메라가 중앙 처리 장치에서 멀리 떨어진 대형 농업용 로봇, 트랙터, 로봇 크레인, 지게차 및 배달 로봇에 사용됩니다.7
- **드론:** 긴 팔 끝에 달린 카메라가 가볍고 신뢰할 수 있는 장거리 링크를 필요로 하는 내비게이션 및 장애물 회피에 사용됩니다.7
- **머신 비전 및 보안:** 견고하고 장거리 카메라 연결이 필요한 공장 자동화 및 감시 시스템에 사용됩니다.5
- **의료 영상:** 원격 영상 헤드를 중앙 콘솔에 연결하는 등 잠재적인 응용 분야로 언급됩니다.18


SerDes 시장은 독점 솔루션의 이중 독점에서 강력한 개방형 표준을 포함하는 3자 경쟁으로 전환되는 변곡점에 있습니다. 이 마지막 섹션에서는 FPD-Link III를 경쟁 기술 및 후속 기술과 비교하여 시장에서의 위치와 미래 생존 가능성을 이해하는 데 도움이 되는 전략적 분석을 제공합니다.


자동차 SerDes 시장은 Texas Instruments(TI)의 FPD-Link와 Analog Devices(구 Maxim Integrated)의 GMSL(Gigabit Multimedia Serial Link)이라는 두 가지 독점 솔루션이 지배해 왔습니다.30

- **비교 분석:**
  - **대역폭:** GMSL2는 일반적으로 FPD-Link III의 약 4.16 Gbps / 50 Mbps에 비해 더 높은 순방향 채널 대역폭(약 6 Gbps)과 역방향 채널 대역폭(약 187 Mbps)을 제공합니다. 이는 초고해상도(4K 이상) 애플리케이션에서 GMSL2에 이점을 줍니다.18
  - **케이블 거리:** GMSL2는 종종 FPD-Link III의 약 15m에 비해 약간 더 긴 케이블 도달 거리(약 20m)를 주장합니다.31
  - **기능:** 두 기술 모두 PoC를 지원하고 강력한 EMI 성능을 갖추고 있습니다. GMSL2는 더 많은 GPIO와 역방향 채널에서 네이티브 UART를 지원하는 등의 기능적 이점이 있는 반면, FPD-Link III는 I2C/SPI에 중점을 둡니다.18 GMSL2는 또한 다중 스트림 집계에 대한 지원이 더 우수합니다.31
  - **결론:** 선택은 트레이드오프입니다. GMSL2가 원시 사양에서 종종 앞서지만, FPD-Link III는 방대한 주류 자동차 애플리케이션(HD ~ 2MP/4MP 카메라)에 충분하거나 최적인 성숙하고 비용 효율적이며 매우 유능한 솔루션입니다.33


FPD-Link IV는 증가하는 대역폭 요구에 대한 TI의 해답이며, FPD-Link III를 대체하는 것이 아니라 더 높은 성능 계층을 제공합니다.34 TI는 FPD-Link IV를 통해 개방형 표준과 직접 경쟁하는 대신, 표준화하기 어려운 복잡한 디스플레이 토폴로지와 같은 하이엔드 기능에서 혁신을 추구함으로써 시장 점유율을 방어하는 전략을 취하고 있습니다.

- **정량적 개선 사항:**
  - **순방향 채널:** 약 4.16 Gbps 대비 약 7.55 Gbps (6 Gbps 비디오 페이로드).29
  - **MIPI 지원:** MIPI D-PHY v2.1 및 최대 16개의 가상 채널을 지원하여 크게 향상되었습니다.29
- **새로운 기능 (디스플레이 중심):** FPD-Link IV는 데이지 체이닝 및 스플리팅과 같은 초고해상도 디스플레이를 위한 고급 기능을 도입했습니다. 단일 SER가 체인으로 여러 DES 칩을 구동하여 필러-투-필러 디스플레이에 전원을 공급할 수 있으며, 단일 DES는 동기화된 듀얼 DisplayPort 출력을 가질 수 있습니다.2
- **하위 호환성:** FPD-Link IV 직렬 변환기는 일부 FPD-Link III 역직렬 변환기(예: DS90UB954-Q1, DS90UB960-Q1)와 호환되도록 설계되어 기존 시스템에 대한 업그레이드 경로를 제공합니다.29


MIPI A-PHY는 업계 최초의 표준 기반, 비독점 장거리 자동차 SerDes 물리 계층으로, TI(FPD-Link)와 ADI(GMSL)의 독점 생태계에 근본적인 도전을 제기합니다.39

- **개방형 표준의 주요 이점:**
  - **상호 운용성:** 이론적으로 A사(Vendor A)의 A-PHY SER가 장착된 센서는 B사(Vendor B)의 A-PHY DES가 있는 ECU에 연결될 수 있어, 벤더 종속(vendor lock-in)을 해결합니다.4
  - **생태계 및 비용:** 더 넓은 다중 벤더 생태계는 경쟁을 촉진하고 비용을 절감하며 혁신을 가속화할 것으로 예상됩니다.4
  - **네이티브 프로토콜 지원:** A-PHY는 MIPI를 FPD-Link/GMSL로 변환하는 독점적인 "브리지" 칩의 필요성을 없애고, 기존 MIPI 프로토콜(예: CSI-2, DSI-2)을 위한 네이티브 장거리 물리 계층으로 설계되었습니다.40
- **성능:** A-PHY는 32 Gbps 이상으로 확장되는 로드맵, 초저 패킷 오류율(<10−19), 높은 순 데이터 효율(>90%) 등 매우 높은 성능을 목표로 설계되었습니다.39

아래 표는 기술 관리자나 시스템 아키텍트가 프로젝트의 특정 요구 사항에 따라 데이터 기반 결정을 내릴 수 있도록 주요 기술 선택 사항을 다각적으로 비교합니다.

**표 2: 자동차 SerDes 인터페이스 비교 분석**

| 기능                        | FPD-Link III      | GMSL2            | FPD-Link IV              | MIPI A-PHY v1.1            |
| --------------------------- | ----------------- | ---------------- | ------------------------ | -------------------------- |
| **개발사**                  | Texas Instruments | Analog Devices   | Texas Instruments        | MIPI Alliance (다중 벤더)  |
| **생태계 유형**             | 독점              | 독점             | 독점                     | 개방형 표준                |
| **최대 순방향 속도**        | ~4.16 Gbps        | ~6 Gbps          | ~7.55 Gbps               | 최대 32 Gbps (STQ 사용 시) |
| **최대 역방향 속도**        | ~50 Mbps          | ~187 Mbps        | ~47 Mbps                 | 200 Mbps                   |
| **최대 케이블 거리 (동축)** | ~15m              | ~20m             | ~15m (추정)              | ~15m                       |
| **PoC 지원**                | 예                | 예               | 예                       | 예                         |
| **다중 카메라 지원**        | 우수 (허브 사용)  | 매우 우수 (집계) | 매우 우수 (허브 사용)    | 매우 우수 (네이티브)       |
| **핵심 차별점**             | 성숙/비용 효율성  | 고대역폭/기능    | 고급 디스플레이 토폴로지 | 상호 운용성/표준           |


- **강점:** FPD-Link III는 성숙하고, 입증되었으며, 매우 신뢰할 수 있고 비용 효율적인 기술입니다. 방대한 설치 기반과 TI의 광범위한 문서 및 지원을 보유하고 있으며, 오늘날의 자동차 카메라 애플리케이션(최대 4MP) 대부분에 완벽하게 최적화되어 있습니다. 특히 진단 기능은 기능 안전을 위한 핵심 요소입니다.1
- **한계:** 독점적인 단일 벤더 솔루션으로, 일부 OEM에게는 전략적 위험(벤더 종속)이 될 수 있습니다.33 대역폭은 최신 세대(FPD-Link IV, GMSL2/3) 및 A-PHY 표준에 의해 추월당하고 있어, 미래의 최고급 8K 디스플레이나 8MP+ 카메라에는 덜 적합합니다.
- **미래 생존 가능성:** FPD-Link III는 성숙도, 비용 효율성, 그리고 자동차 산업의 느린 설계 주기 덕분에 앞으로 수년간 주류 자동차 시장에서 지배적인 위치를 유지할 가능성이 높습니다. 그러나 차세대 아키텍처(구역형 ECU, 8MP+ 카메라)의 경우, FPD-Link IV와 MIPI A-PHY가 명확한 미래 방향입니다. FPD-Link III의 역할은 신뢰할 수 있는 대량 생산의 주력 제품으로 남는 반면, 최첨단 기술은 새로운 기술로 이동할 것입니다.


1. TI FPDLink III Automotive Cameras & Camera Modules - Leopard Imaging, accessed July 11, 2025, https://leopardimaging.com/product-category/automotive-cameras/cameras-by-interface/ti-ftplink-iii-cameras-cameras-by-interface/
2. FPD-Link learning center | TI.com - Texas Instruments, accessed July 11, 2025, https://www.ti.com/video/series/fpd-link-learning-center.html
3. Cameras with FDP-Link: An Introduction - VIA optronics, accessed July 11, 2025, https://via-optronics.com/en/products-technology/camera-portfolio/fpd-link-camera.html
4. FPD-Link III -- doing more with less - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/pdf/SLYT581
5. DS90UB633A-Q1 FPD-Link III Serializer - TI - Mouser Electronics, accessed July 11, 2025, https://www.mouser.com/new/texas-instruments/ti-ds90ub633a-q1-serializer/
6. ADAS Multi-Sensor Hub Design With Quad 4 ... - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/pdf/tiduct0
7. FPD-Link III Cameras – Working Principle and Applications in Embedded Vision, accessed July 11, 2025, https://www.technexion.com/resources/fpd-link-iii-cameras-working-principle-and-applications-in-embedded-vision/
8. FPD-Link III Cameras for Embedded Vision Systems - The Imaging Source, accessed July 11, 2025, https://www.theimagingsource.com/en-us/embedded/fpd-link-iii/
9. FPD-Link - Wikipedia, accessed July 11, 2025, https://en.wikipedia.org/wiki/FPD-Link
10. Serializer targets ADAS applications - Electronic Specifier, accessed July 11, 2025, https://www.electronicspecifier.com/industries/automotive/serializer-targets-adas-applications
11. Difference between LVDS SerDes and FPD-Link III SerDes? - Electronics Stack Exchange, accessed July 11, 2025, https://electronics.stackexchange.com/questions/511972/difference-between-lvds-serdes-and-fpd-link-iii-serdes
12. What is an FPD-link? - MD Elektronik, accessed July 11, 2025, https://www.md-elektronik.com/en/fpd-link/
13. How to Design a FPD-Link III System Using ... - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/pdf/snla267
14. DS90UB933-Q1 FPD-Link III Serializer for 1-MP/60-fps Cameras 10/12 Bits,100 MHz datasheet (Rev. E), accessed July 11, 2025, https://file.elecfans.com/web2/M00/21/6B/pYYBAGGkLreATmb2ACsDKxvF5d4399.pdf
15. I2C Communication Over FPD-Link III with Bidirectional Control Channel (Rev. A) - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/pdf/snla131
16. DS90UB927Q-Q1 5-MHz to 85-MHz 24-bit Color FPD-Link III Serializer With Bidirectional Control Channel, accessed July 11, 2025, https://www.tij.co.jp/lit/ds/symlink/ds90ub927q-q1.pdf
17. Power Over Coax Design Guidelines for ... - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/pdf/snla272
18. GMSL2 cameras vs. FPD-Link III cameras – a detailed study - e-con Systems, accessed July 11, 2025, https://www.e-consystems.com/blog/camera/technology/gmsl2-cameras-vs-fpd-link-iii-cameras-a-detailed-study/
19. Infotainment (IVI) back channel basics | Video | TI.com - Texas Instruments, accessed July 11, 2025, https://www.ti.com/video/5503353885001
20. What is FPD-Link? | Video | TI.com, accessed July 11, 2025, https://www.ti.com/video/6094247787001
21. FPD-Link III SerDes Bidirectional Control Channel - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=oexymmf3Zi4
22. Sending Power Over Coax in FPD-Link Designs (Rev. A) - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/pdf/snla224
23. Power over Coax (PoC) Design | Video | TI.com - Texas Instruments, accessed July 11, 2025, https://www.ti.com/video/5503332273001
24. Automotive 2-MP Camera Module Reference Design With MIPI CSI-2, PMIC, FPD-Link III and POC, accessed July 11, 2025, https://www.tij.co.jp/lit/pdf/TIDUEB1
25. Computer vision for intralogistics: Deploying FPD-Link III and GMSL2, accessed July 11, 2025, https://www.alliedvision.com/fileadmin/content/documents/landing-pages/AlliedVision-Whitepaper_Computer-vision-for-intralogistics.pdf
26. Deserializer Board FPD-Link III Coax to CSI-2 User Guide - Allied Vision, accessed July 11, 2025, https://cdn.alliedvision.com/fileadmin/content/documents/products/accessories/embedded/user-guide/19559_Deserializer-Board_FP3-Coax-CSI-2_User-Guide.pdf
27. Automotive ADAS Reference Design for Four-Camera Hub With MIPI CSI-2 Output, accessed July 11, 2025, http://www.tij.co.jp/jp/lit/ug/tiducm4/tiducm4.pdf
28. FPD-Link III Cameras for Embedded Vision Applications - e-con Systems, accessed July 11, 2025, https://www.e-consystems.com/fpd-link-iii-cameras.asp
29. Texas Instruments DS90UB971-Q1 FPD-Link IV Serializers - Mouser Electronics, accessed July 11, 2025, https://www.mouser.com/new/texas-instruments/ti-ds90ub971-q1-serializers/
30. Difference Between GMSL Camera and FPD-Link Camera-Industrial ..., accessed July 11, 2025, https://www.premier-cable-mfg.com/Blog-Detail/86.html
31. FPD-Link III vs GMSL2: Compare Camera Interfaces & Cables for ..., accessed July 11, 2025, https://www.pcm-cable.com/info/fpd-link-iii-vs-gmsl2-camera-cables-103029004.html
32. FPD Link-III vs GMSL2 for cameras at a distance from FPGA - element14 Community, accessed July 11, 2025, https://community.element14.com/technologies/fpga-group/f/forum/49965/fpd-link-iii-vs-gmsl2-for-cameras-at-a-distance-from-fpga
33. FPD-Link Vs GMSL: A Quick Comparison Guide - Supertek, accessed July 11, 2025, https://www.supertekmodule.com/fpd-link-vs-gmsl/
34. DS90UB954-Q1: Difference between FPD-LINK III and IV - Interface forum - TI E2E, accessed July 11, 2025, https://e2e.ti.com/support/interface-group/interface/f/interface-forum/966831/ds90ub954-q1-difference-between-fpd-link-iii-and-iv
35. DS90UB971 -Q1 FPD-Link IV 7.55-Gbps ... - Texas Instruments, accessed July 11, 2025, https://www.ti.com/lit/gpn/DS90UB971-Q1
36. FPD-Link Specification - IEEE 802, accessed July 11, 2025, https://www.ieee802.org/3/dm/public/adhoc/090524/razavi_dm_01a_09042024.pdf
37. FPD-Link video output synchronization demo | Video | TI.com - Texas Instruments, accessed July 11, 2025, https://www.ti.com/video/6347682213112
38. FPD-Link IV - Daisy chain audio demo | Video | TI.com, accessed July 11, 2025, https://www.ti.com/video/6314892285112
39. A-PHY Frequently Asked Questions | MIPI - MIPI Alliance, accessed July 11, 2025, https://www.mipi.org/resources/a-phy-frequently-asked-questions
40. MIPI A-PHY, accessed July 11, 2025, https://www.mipi.org/specifications/a-phy
41. MIPI Alliance Releases A-PHY SerDes Interface for Automotive - SemiWiki, accessed July 11, 2025, https://semiwiki.com/forum/threads/mipi-alliance-releases-a-phy-serdes-interface-for-automotive.13028/
42. In-depth analysis: MIPI A-PHY, a new generation of high-speed serial communication technology - EEWorld, accessed July 11, 2025, https://en.eeworld.com.cn/mp/qckfq/a391952.jspx




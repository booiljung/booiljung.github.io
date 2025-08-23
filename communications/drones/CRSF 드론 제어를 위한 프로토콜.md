# 드론 제어를 위한 CRSF (Crossfire) 프로토콜



CRSF(Crossfire)는 Team BlackSheep(TBS)사가 개발한 독점 통신 프로토콜로, 현대 드론 시스템에서 원격 제어(RC)와 원격 측정(Telemetry) 데이터 통신을 위한 사실상의 표준(de-facto standard)으로 자리매김했다.1 이 프로토콜의 핵심은 단일 UART(범용 비동기 송수신기) 직렬 연결을 통해 조종기와 기체 간의 데이터를 양방향으로, 그리고 동시에 교환할 수 있다는 점이다.3 과거의 RC 프로토콜, 예를 들어 PWM(Pulse Width Modulation), PPM(Pulse Position Modulation), SBUS(Serial Bus) 등은 주로 조종기에서 기체로 향하는 단방향 제어 신호 전송에 국한되었다.3 텔레메트리 데이터를 수신하기 위해서는 FrSky의 SmartPort와 같은 별도의 하드웨어와 프로토콜이 필요했으며, 이는 배선과 설정을 복잡하게 만드는 요인이었다.

CRSF는 이러한 한계를 극복하고 제어 신호(Uplink)와 텔레메트리 데이터(Downlink)를 하나의 통합된 프로토콜로 처리함으로써 시스템 구성을 획기적으로 단순화했다. 이를 통해 파일럿은 기체의 배터리 전압, 전류 소모량, GPS 위치, 링크 품질(Link Quality) 등 핵심 비행 정보를 조종기 화면이나 OSD(On-Screen Display)를 통해 실시간으로 확인할 수 있게 되었다.5 이러한 양방향 통신 기능은 비행 안전성을 크게 향상시켰을 뿐만 아니라, 드론 시스템 운용의 패러다임을 바꾸는 계기가 되었다.


CRSF의 기술적 의의는 단순히 장거리(Long Range) 통신을 구현한 것을 넘어, 낮은 지연 시간(Low Latency), 외부 전파 간섭에 대한 강인함(Robustness), 그리고 사용자 편의성을 하나의 시스템 안에 완벽하게 통합했다는 데 있다.8 TBS는 이를 "그냥 작동한다(It Just Works)"라는 철학으로 요약하며, 복잡한 설정 과정 없이도 누구나 안정적인 고성능 비행을 경험할 수 있도록 하는 데 초점을 맞추었다.8

이러한 철학은 CRSF를 단순한 통신 프로토콜을 넘어 하나의 거대한 **통합 생태계(Integrated Ecosystem)**로 발전시키는 원동력이 되었다. CRSF의 진정한 경쟁력은 개별 기술의 우수성이 아니라, 하드웨어(송신기, 수신기, 영상송신기), 소프트웨어(OpenTX/EdgeTX LUA 스크립트, TBS Agent), 그리고 비행 컨트롤러 펌웨어(Betaflight, iNav, ArduPilot/PX4)를 아우르는 유기적인 연동성에 있다.

이 생태계의 구축 과정을 살펴보면 CRSF의 혁신성을 명확히 이해할 수 있다. 첫째, CRSF는 제어와 텔레메트리를 단일 직렬 프로토콜로 통합하여 물리적 배선을 단순화하는 1차적 이점을 제공했다.1 둘째, TBS는 이 양방향 링크를 단순한 데이터 모니터링 채널을 넘어 

**기기 제어 및 관리 채널**로 확장했다. 조종기의 LUA 스크립트나 내장 메뉴를 통해 수신기(RX)의 출력 채널 매핑을 변경하고, 영상송신기(VTX)의 채널과 출력 세기를 조절하며, 심지어 비행 컨트롤러의 PID 값까지 수정할 수 있는 길을 열었다.8 이는 더 이상 기체 설정을 위해 매번 PC에 연결해야 했던 번거로움을 없애고, 비행 현장에서 조종기만으로 대부분의 설정을 완료할 수 있게 만든 혁신적인 변화였다.

결과적으로 CRSF는 '수신기 프로토콜'에서 '드론 시스템 관리 프로토콜'로 진화했다. 이러한 압도적인 편의성과 긴밀한 통합성은 Betaflight, iNav, ArduPilot/PX4 등 주요 오픈소스 비행 컨트롤러 펌웨어 개발자들이 CRSF를 최우선으로 지원하게 만들었고 9, 이는 다시 CRSF의 시장 지배력을 공고히 하는 선순환 구조를 만들었다. 심지어 강력한 경쟁 기술로 부상한 오픈소스 프로젝트 ExpressLRS(ELRS)마저도 비행 컨트롤러와의 통신에는 CRSF 프로토콜을 채택할 정도로 4, CRSF는 현대 드론 통신의 표준으로 확고히 자리 잡았다. 본 보고서는 이처럼 드론 기술의 발전에 지대한 영향을 미친 CRSF 프로토콜의 기술적 원리를 심층적으로 분석하고, 실제 시스템 구성 방법을 상세히 기술하여 사용자들이 그 잠재력을 최대한 활용할 수 있도록 돕는 것을 목표로 한다.


CRSF 프로토콜의 성공은 견고한 물리 계층 기술과 효율적인 데이터 링크 계층 기술의 정교한 결합에 기반한다. 장거리 통신을 위한 물리적 특성과 저지연 제어를 위한 프로토콜 설계를 분리하여 분석함으로써 그 핵심 원리를 파악할 수 있다.


CRSF는 명확하게 정의된 패킷 프레임(Packet Frame) 구조를 기반으로 동작하는 전이중(Full-duplex) 직렬 통신 프로토콜이다.


모든 CRSF 통신은 일관된 프레임 포맷을 따른다. 각 프레임은 동기화를 위한 시작 바이트, 데이터의 길이와 종류를 명시하는 헤더, 실제 정보를 담은 페이로드, 그리고 데이터 무결성을 보장하기 위한 오류 검출 코드로 구성된다.13

- **Start Byte (시작 바이트)**: 프레임의 시작을 알리는 1바이트의 고유 식별자이다. 예를 들어, `0xC8`은 수신기에서 비행 컨트롤러로 전송되는 패킷의 시작을 나타낼 수 있다.13

- **Length (길이)**: `Type` 필드부터 `CRC` 필드까지의 총 바이트 수를 나타내는 1바이트 필드이다. 이 값을 통해 수신 측은 페이로드의 길이를 동적으로 파악할 수 있다.

- **Type (종류)**: 패킷에 담긴 데이터의 종류를 정의하는 1바이트 필드이다. 예를 들어, `0x16`은 RC 채널 데이터를 의미하며, 다른 값들은 GPS 데이터, 링크 통계, VTX 설정 등 다양한 정보를 나타낸다.13 이 

  `Type` 필드는 새로운 기능을 추가할 때 프로토콜을 유연하게 확장할 수 있게 하는 핵심 요소이다.

- **Payload (페이로드)**: 실제 데이터가 담기는 부분으로, `Type`에 따라 그 구조와 길이가 가변적이다. RC 채널 데이터의 경우, 각 채널의 스틱 값을 정밀하게 표현하기 위해 11비트를 사용하는 등 효율적인 데이터 압축 기법이 적용된다.

- **CRC (Cyclic Redundancy Check)**: `Type`과 `Payload` 데이터를 기반으로 계산된 1바이트(8비트) 오류 검출 코드이다.13 CRSF는 다항식 

  `0xD5`를 사용하는 CRC-8 알고리즘을 채택하여 데이터 무결성을 검증한다.85 수신 측은 동일한 계산을 수행하여 수신된 CRC 값과 비교함으로써 전송 중 발생한 데이터 손상 여부를 판별한다.


CRSF는 하나의 UART 포트에 있는 TX(송신)와 RX(수신) 핀을 모두 사용하여 데이터의 송신과 수신을 동시에 수행하는 전이중 통신 방식을 채택했다.3 이는 다음과 같은 데이터 흐름을 가능하게 한다.

1. **Uplink (제어 신호)**: 지상의 조종기(Transmitter)는 스틱 입력, 스위치 상태 등의 제어 신호를 생성하여 Crossfire 송신 모듈로 전달한다. 송신 모듈은 이 데이터를 CRSF 패킷으로 인코딩하여 공중으로 전송한다.
2. **Downlink (텔레메트리)**: 기체의 Crossfire 수신기(Receiver)는 비행 컨트롤러(FC)의 UART TX 핀으로부터 배터리 전압, GPS 좌표, 비행 모드 등의 텔레메트리 정보를 수신한다. 수신기는 이 데이터를 다시 CRSF 패킷으로 인코딩하여 지상의 송신 모듈로 전송한다.

이 두 과정이 동시에 이루어지기 때문에, 파일럿은 기체를 제어하면서 동시에 기체의 상태를 실시간으로 피드백 받을 수 있다. 이는 기존의 반이중(Half-duplex) 또는 단방향 프로토콜에 비해 월등히 높은 효율성과 안정성을 제공한다.


Crossfire 시스템이 수십 킬로미터에 달하는 통신 거리를 달성할 수 있는 비결은 주파수 선택, 변조 방식, 그리고 간섭 회피 기술의 전략적인 조합에 있다.

- **ISM 대역 활용**: Crossfire는 일반적으로 사용되는 2.4GHz 대역 대신 900MHz 대역의 ISM(Industrial, Scientific, and Medical) 밴드를 주력으로 사용한다.10 전파는 주파수가 낮을수록 파장이 길어져 회절(Diffraction) 특성이 좋아지고, 건물이나 나무와 같은 장애물을 투과하거나 우회하는 능력이 향상된다. 이 원리 덕분에 900MHz 대역은 비가시선(NLOS, Non-Line-of-Sight) 환경에서 2.4GHz 대역보다 훨씬 안정적인 링크를 제공한다. 지역별 전파 규정에 따라 유럽 및 러시아에서는 868MHz 대역을, 미국, 아시아, 호주 등에서는 915MHz 대역을 사용하도록 설정할 수 있다.3

- **LoRa 변조 방식**: Crossfire 시스템의 핵심에는 LoRa(Long Range) 통신 칩이 있다.3 LoRa는 처프 확산 스펙트럼(CSS, Chirp Spread Spectrum) 기술을 사용하는 독점적인 변조 방식으로, 신호를 넓은 주파수 대역에 걸쳐 확산시킨다. 이 기술의 가장 큰 장점은 매우 낮은 신호 대 잡음비(SNR, Signal-to-Noise Ratio) 환경, 즉 잡음 레벨보다 신호의 세기가 훨씬 약한 상황에서도 데이터를 성공적으로 복원할 수 있다는 점이다. 이 덕분에 Crossfire 수신기는 

  `-130dBm`이라는 경이로운 수신 감도를 달성할 수 있으며, 이는 곧 극단적인 장거리 통신 능력으로 이어진다.8

- **자가 치유 주파수 호핑 (Self-healing Frequency Hopping)**: Crossfire는 DSSS(Direct Sequence Spread Spectrum)와 FHSS(Frequency Hopping Spread Spectrum) 기술을 복합적으로 활용하여 외부 간섭에 대한 강인함을 확보한다.7 시스템은 지속적으로 사용 가능한 주파수 채널들의 상태를 모니터링한다. 만약 현재 사용 중인 채널에서 강한 간섭이 감지되면, 시스템은 통신 두절 없이 순간적으로 가장 깨끗한 다른 채널로 이동한다. 이러한 '자가 치유' 메커니즘은 Wi-Fi, 휴대폰 기지국, 기타 RC 시스템 등 다양한 전파 간섭원으로부터 링크를 보호하고 비행 내내 안정적인 연결을 유지하는 핵심적인 역할을 한다.


Crossfire는 장거리 성능뿐만 아니라, FPV 레이싱과 프리스타일 비행에서 요구하는 빠른 반응성을 충족시키기 위한 정교한 데이터 링크 계층 기술을 갖추고 있다. 이는 '적응형 변조(Adaptive Modulation)' 전략으로 요약될 수 있다.

많은 사용자들이 Crossfire를 단순히 'LoRa를 사용하는 장거리 시스템'으로 이해하지만, 실제 핵심 경쟁력은 비행 상황에 맞춰 **LoRa(장거리용)와 FSK(저지연용) 변조 방식을 동적으로 혹은 선택적으로 사용하는 전략**에 있다. 이는 '통신 거리'와 '지연 시간'이라는 상충하는 두 가지 목표 사이에서 최적의 균형점을 찾으려는 TBS의 정교한 엔지니어링의 결과물이다. FPV 시장이 장거리 비행뿐만 아니라, 즉각적인 반응성이 중요한 프리스타일/레이싱 시장도 포괄해야 함을 인지한 TBS는, '최고의 거리'가 필요할 때는 LoRa를, '최고의 반응성'이 필요할 때는 더 빠른 FSK를 사용하는 하이브리드 접근 방식을 채택했다. 이는 경쟁 기술인 ELRS가 모든 패킷 속도에서 LoRa를 사용하는 것과 대조되는 지점이다.19 이 '적응성'이 바로 Crossfire가 수년간 다양한 FPV 파일럿들의 요구를 만족시킬 수 있었던 기술적 비결이다.

- **업데이트 속도 (RF Profile)**: 사용자는 비행 목적에 따라 RF 프로파일을 선택할 수 있다.5
  - **50Hz 모드**: 이 모드에서는 LoRa 변조를 사용하여 최고의 통신 거리와 장애물 투과 성능을 제공한다.19 하지만 패킷 전송 간격이 20ms로, 지연 시간이 상대적으로 길어 정밀한 조종보다는 안정적인 장거리 비행에 적합하다.20
  - **150Hz 모드**: 이 모드에서는 더 빠른 데이터 전송을 위해 FSK(Frequency-Shift Keying) 변조 방식으로 전환한다.19 패킷 전송 간격이 약 6.7ms로 줄어들어 매우 낮은 지연 시간을 구현하며, 조종 입력에 대한 기체의 반응이 즉각적이다.20 하지만 FSK 변조는 LoRa만큼의 수신 감도를 제공하지 못하므로 통신 거리는 상대적으로 짧아진다. 프리스타일 및 레이싱 비행에 최적화되어 있다.
  - **Dynamic Mode**: 링크 품질(LQ)이 일정 수준 이하로 떨어지면, 시스템이 자동으로 150Hz 모드에서 50Hz 모드로 전환하여 링크의 안정성을 확보하는 기능이다.10
- **CRSF 프로토콜 버전과 CRSFShot**: 지연 시간을 더욱 줄이고 조종감의 일관성을 높이기 위해 프로토콜은 지속적으로 개선되었다.
  - **CRSF V2**: 150Hz 모드 지원이 공식화된 버전이다.
  - **CRSFShot**: CRSF V2에 도입된 핵심 기술로, 조종기의 스틱 입력 샘플링 시점과 송신 모듈의 무선 패킷 전송 시점을 정밀하게 동기화한다.7 이를 통해 전송 과정에서 발생하는 가변적인 지연 시간(jitter)을 제거하여, 마치 기체와 조종기가 하나로 '잠긴(locked-in)' 듯한 매우 일관되고 직관적인 조종감을 제공한다.
  - **CRSF V3**: CRSFShot 기술을 포함하면서, 조종기와 송신 모듈 간의 내부 직렬 통신 속도 자체를 더욱 높여 전체 시스템의 종단 간(end-to-end) 지연 시간을 극한까지 줄인 최신 버전이다.7

이처럼 CRSF는 물리 계층에서는 LoRa와 주파수 호핑으로 장거리와 안정성을 확보하고, 데이터 링크 계층에서는 적응형 변조와 CRSFShot 기술로 저지연과 반응성을 구현하는 이중 전략을 통해 FPV 드론이 요구하는 다양한 비행 시나리오에 효과적으로 대응한다.


TBS Crossfire 시스템은 다양한 사용자의 요구와 조종기 폼팩터에 대응하기 위해 여러 종류의 송신(TX) 모듈과 수신(RX) 모듈로 구성된 생태계를 갖추고 있다. 각 하드웨어의 특징을 이해하는 것은 자신의 비행 스타일과 기체에 맞는 최적의 시스템을 구성하는 첫걸음이다.


송신 모듈은 조종기 후면의 모듈 베이에 장착되어 CRSF 신호를 생성하고 송출하는 역할을 한다. 크기, 출력, 기능에 따라 크게 세 가지 라인업으로 나뉜다.

- **Full-size Crossfire TX**:
  - **특징**: Crossfire 시스템의 시초가 된 오리지널 모델이다. 내장된 OLED 디스플레이와 조그 다이얼을 통해 PC나 조종기 LUA 스크립트 없이도 모든 설정을 직관적으로 변경할 수 있다.3 최대 송신 출력이 2W(2000mW)에 달해 현존하는 RC 링크 중 가장 강력한 출력을 자랑한다.8 또한, Bluetooth 모듈이 내장되어 있어 태블릿이나 노트북의 지상관제소프트웨어(GCS)와 무선으로 연결하여 MAVLink 텔레메트리 데이터를 스트리밍하는 고급 기능을 지원한다.8
  - **적합 대상**: 15-20km를 넘어서는 초장거리 비행, MAVLink 연동이 필수적인 전문적인 항공 촬영 또는 측량 임무, 그리고 설정 변경의 편의성을 최우선으로 여기는 사용자에게 적합하다.9
  - **고려사항**: 가격이 가장 비싸고 부피가 크며, 1W 이상의 고출력 사용 시 외부 전원(XT30)을 연결해야 한다.8
- **Crossfire Micro TX (V1/V2)**:
  - **특징**: Radiomaster TX16S, FrSky Taranis 등 표준 JR 모듈 베이를 사용하는 대부분의 조종기에 맞춰 설계된 컴팩트한 모델이다.3 OLED 화면이 없는 대신, 모든 설정은 조종기 화면의 LUA 스크립트(TBS Agent Lite)를 통해 이루어진다.10 V1 모델은 최대 출력이 250mW였으나, V2 모델은 최대 1W(1000mW)로 대폭 향상되어 Full-size 모델 못지않은 장거리 성능을 제공한다.25 또한 V2는 구형 USB 포트 대신 USB-C 커넥터를 채택하여 편의성을 높였다.9 단, MAVLink 블루투스 스트리밍 기능은 지원하지 않는다.9
  - **적합 대상**: 가격과 성능의 균형을 중시하는 대다수의 FPV 파일럿에게 최적의 선택이다. 건물 내부 비행(Penetration), 미니 쿼드 프리스타일, 수 킬로미터 반경의 중장거리 탐사 비행에 필요한 모든 성능을 제공하면서도 Full-size 모델의 절반 이하 가격으로 구매할 수 있다.9
- **Crossfire Nano TX**:
  - **특징**: Micro TX와 기술적으로 동일한 코어를 공유하며 최대 1W의 출력을 내지만, TBS Tango 2, Taranis X-Lite, Radiomaster Zorro 등 'Lite' 모듈 베이를 사용하는 소형 조종기를 위해 특별히 설계된 모델이다.3
  - **적합 대상**: 'Lite' 폼팩터의 소형 조종기를 사용하는 파일럿.


수신 모듈은 드론 기체에 장착되어 지상으로부터 제어 신호를 수신하고, 기체의 텔레메트리 데이터를 지상으로 송신하는 역할을 한다. 기체의 크기, 비행 목적, 요구되는 신뢰성 수준에 따라 적합한 모델을 선택해야 한다.

- **Crossfire Nano RX**:
  - **특징**: 11mm x 18mm의 크기와 0.5g에 불과한 무게를 가진 초소형, 초경량 수신기이다.30 이는 널리 사용되던 FrSky XM+ 수신기보다도 작은 크기로, 공간과 무게가 극도로 제한적인 현대 FPV 레이싱 및 프리스타일 드론에 이상적이다. 단일 안테나(주로 Immortal T 안테나 사용) 구성이며, CRSF, SBUS, PPM 등 다양한 프로토콜 출력을 지원하여 구형 FC와의 호환성도 갖추었다.33
  - **적합 대상**: 크기와 무게가 중요한 거의 모든 종류의 FPV 드론. 레이싱, 프리스타일, 경량 장거리 기체 등 범용성이 매우 높다.
- **Crossfire Diversity Nano RX**:
  - **특징**: 이름에서 알 수 있듯이, 두 개의 독립적인 RF 수신 회로와 두 개의 안테나를 갖춘 진정한 칩 기반 다이버시티(True Diversity) 시스템을 구현한 것이 가장 큰 특징이다.34 이는 안테나의 '널(null)' 지점, 즉 특정 방향에서 신호 수신 감도가 급격히 떨어지는 현상으로 인한 순간적인 신호 손실(glitch) 위험을 원천적으로 차단한다. 시스템은 항상 두 안테나 중 더 강한 신호를 수신하는 쪽을 자동으로 선택하여 링크의 안정성과 신뢰성을 극대화한다.37
  - **추가 기능**:
    - **비콘 모드 (Beacon Mode)**: 이 수신기의 가장 독보적인 기능이다. 별도의 1S LiPo 배터리를 수신기에 직접 연결해두면, 기체의 주 배터리가 분리되거나 전원이 차단되는 최악의 추락 상황에서도 수신기가 자체 전력으로 최대 하루 동안 살아남는다.34 이 시간 동안 수신기는 마지막 GPS 좌표와 함께 RF 비콘 신호를 주기적으로 송출하여, 수백 미터 밖에서도 방향 탐지기를 이용해 추락한 기체의 위치를 정밀하게 찾아낼 수 있도록 돕는다.
    - **FLARM 지원**: 경비행기 및 UAV를 위한 항공 교통 인식 및 충돌 회피 시스템인 FLARM 네트워크에 방송할 수 있는 기능을 제공하여, 비행 안전성을 한 단계 더 높일 수 있다.36
  - **적합 대상**: 링크 신뢰성이 무엇보다 중요한 고가의 장거리 비행용 고정익(Fixed-wing)이나 시네마틱 촬영용 대형 드론에 최적화되어 있다. 안테나를 서로 다른 각도와 위치에 배치하여 다이버시티 효과를 극대화할 수 있는 기체에 특히 유리하다.
- **Crossfire Sixty9**: Crossfire 수신기와 5.8GHz 아날로그 영상송신기(VTX)를 하나의 보드에 통합한 AIO(All-in-One) 제품이다.38 별도의 수신기와 VTX를 구매하고 연결하는 번거로움을 줄여주어, 빌드 과정을 간소화하고 기체 내부를 깔끔하게 정리할 수 있는 장점이 있다.


사용자의 합리적인 의사결정을 돕기 위해 각 모듈의 핵심 사양을 아래 표로 정리하였다.

**Table 1: TBS Crossfire 송신(TX) 모듈 비교**

| 특징         | Full-size TX        | Micro TX V2      | Nano TX        |      |
| ------------ | ------------------- | ---------------- | -------------- | ---- |
| 폼팩터       | 외부 장착형         | JR 모듈 베이     | Lite 모듈 베이 |      |
| 최대 출력    | 2W                  | 1W               | 1W             |      |
| 디스플레이   | OLED                | 없음             | 없음           |      |
| 설정 방식    | 내장 조그, LUA      | LUA 스크립트     | LUA 스크립트   |      |
| Bluetooth    | 지원 (MAVLink)      | 미지원           | 미지원         |      |
| 커넥터       | XT30, 구형 USB      | USB-C            | USB-C          |      |
| 권장 용도    | 초장거리, 전문 임무 | 범용 FPV, 장거리 | 소형 조종기용  |      |
| 자료 출처: 3 |                     |                  |                |      |

**Table 2: TBS Crossfire 수신(RX) 모듈 비교**

| 특징          | Nano RX                            | Diversity Nano RX                  |      |
| ------------- | ---------------------------------- | ---------------------------------- | ---- |
| 크기 / 무게   | 11x18mm / 0.5g                     | 24x18mm / 1.8g                     |      |
| 다이버시티    | 없음 (단일 안테나)                 | True Diversity (듀얼 안테나)       |      |
| 비콘 모드     | 미지원                             | 지원 (별도 배터리 필요)            |      |
| FLARM         | 미지원                             | 지원                               |      |
| 가격 (MSRP)   | 약 $25                             | 약 $50                             |      |
| 권장 용도     | 소형 쿼드, 레이싱, 일반 프리스타일 | 장거리 비행, 고정익, 고신뢰성 임무 |      |
| 자료 출처: 30 |                                    |                                    |      |


Crossfire 시스템의 성능을 온전히 활용하기 위해서는 조종기(Transmitter)와 드론의 비행 컨트롤러(FC) 양쪽에서 정확한 설정이 이루어져야 한다. 본 장에서는 각 단계별 설정 과정을 상세히 안내한다.


조종기는 Crossfire 시스템의 제어 센터 역할을 한다. 펌웨어 준비부터 모듈 활성화, 세부 파라미터 설정까지 순차적으로 진행해야 한다.


최신 Crossfire 기능, 특히 LUA 스크립트를 완벽하게 사용하기 위해서는 조종기의 운영체제 펌웨어(OpenTX 또는 EdgeTX)와 SD카드 콘텐츠를 최신 버전으로 유지하는 것이 매우 중요하다.10 이는 TBS Agent Lite 스크립트와의 호환성을 보장하고, 잠재적인 버그를 예방하는 가장 기본적인 단계이다. TBS Agent X나 Agent M을 사용하여 Crossfire 송신 모듈의 펌웨어 또한 최신 안정 버전으로 업데이트해야 한다.7


1. Crossfire 송신 모듈을 조종기 후면의 모듈 베이에 정확하게 장착한다.
2. 조종기의 `Model Setup` 메뉴로 진입한다.10
3. `Internal RF` (내장 RF 모듈) 항목을 `OFF`로 설정하여 내장 모듈과의 충돌을 방지한다.
4. `External RF` (외장 RF 모듈) 섹션에서 `Mode`를 `CRSF`로 선택한다.10 이 설정을 통해 조종기는 외장 모듈 베이를 통해 CRSF 프로토콜로 통신을 시작한다.
5. `Channel Range`는 `CH1-16`으로 설정하여 12채널 모드 사용 시 모든 보조 채널(AUX)을 활용할 수 있도록 한다.10
6. 최신 EdgeTX 펌웨어는 `OneBit Mode`를 지원하여, 과거 일부 구형 조종기에서 400k baud rate 통신을 위해 필요했던 하드웨어 개조 없이도 소프트웨어적으로 안정적인 고속 통신을 가능하게 한다.41


모든 세부 설정은 조종기의 `Tools` 메뉴에서 `TBS Agent Lite` LUA 스크립트를 실행하여 진행한다.10

1. **송신기(XF TX) 설정**:
   - **Region**: `Open`으로 설정해야 국가별 규제에 따른 출력 제한 없이 모듈의 최대 성능을 사용할 수 있다.10
   - **Frequency**: 비행하는 국가의 ISM 대역에 맞춰 `868MHz` 또는 `915MHz`를 정확히 선택해야 한다. 잘못된 주파수 선택은 통신 불량 및 법규 위반의 원인이 될 수 있다.10
   - **Max Power**: 비행 환경과 목적에 따라 송신 출력을 10mW, 25mW, 100mW, 250mW, 500mW, 1W, 2W 등으로 설정한다.
   - **Dynamic Power**: `On`으로 설정하는 것을 권장한다. 링크 품질(LQ)에 따라 송신 출력을 자동으로 조절하여 불필요한 전력 소모를 줄이고, 다른 파일럿과의 간섭을 최소화한다.10
   - **Mode**: `12ch` 모드를 선택하여 더 많은 AUX 채널을 활용하는 것이 일반적인 FPV 드론 설정에 유리하다.10
   - **RF Profile**: 비행 스타일에 맞춰 선택한다. 장거리 비행 시에는 `50Hz` 또는 `Dynamic`을, 저지연이 중요한 프리스타일/레이싱에서는 `150Hz`를 선택한다.10
2. **수신기(XF RX) 설정**:
   - **Output Map**: **가장 중요한 설정 단계 중 하나이다.** 비행 컨트롤러와 CRSF 프로토콜로 정상적으로 통신하기 위해서는 수신기의 각 채널 출력을 정의해야 한다. `Output 1`을 `CRSF TX`로, `Output 2`를 `CRSF RX`로 설정해야 한다.10 이 설정이 잘못되면 FC에서 수신기 입력을 전혀 감지하지 못한다.
   - **Telemetry**: `On`으로 설정하여 기체로부터 텔레메트리 데이터를 수신할 수 있도록 한다.
   - **Failsafe Mode**: `Cut`으로 설정하는 것이 일반적이다. 이는 신호 손실 시 수신기가 모든 채널의 출력을 중단시켜 FC가 페일세이프 상태임을 명확히 인지하도록 한다.


FC 설정은 물리적 연결과 소프트웨어 구성으로 나뉜다.


- Crossfire 수신기의 `5V` 및 `GND` 핀을 FC의 사용 가능한 5V 및 GND 패드에 연결한다.
- 수신기의 `TX` 핀(보통 CH1으로 표기)을 FC의 비어있는 UART 포트의 `RX` 패드에 연결한다.
- 수신기의 `RX` 핀(보통 CH2으로 표기)을 FC의 동일한 UART 포트의 `TX` 패드에 연결한다.10
- **핵심 원칙**: 신호는 한쪽의 송신(TX, Transmit) 포트에서 나와 다른 쪽의 수신(RX, Receive) 포트로 들어가야 한다. 따라서 **수신기 TX -->> FC RX**, **수신기 RX -->> FC TX**로 반드시 교차 연결해야 한다.46 이는 초보자들이 가장 흔하게 저지르는 실수 중 하나이다.
- 사용 가능한 UART 포트와 핀 배열은 FC 모델(F4, F7 등)마다 다르므로, 반드시 해당 FC의 공식 배선도(Wiring Diagram)를 참조해야 한다.45


1. **Ports 탭**: Betaflight Configurator에 연결한 후, 수신기를 물리적으로 연결한 UART 번호를 찾아 해당 줄의 `Serial RX` 슬라이더를 활성화한다. 다른 기능(예: MSP)과 중복되지 않도록 주의해야 한다. 설정을 마친 후 `Save and Reboot`를 클릭한다.10
2. **Configuration 탭**:
   - `Receiver` 섹션으로 이동하여 `Receiver Mode` 드롭다운 메뉴에서 `Serial (via UART)`를 선택한다.
   - 바로 아래 `Serial Receiver Provider` 드롭다운 메뉴에서 `CRSF`를 선택한다.10
   - `Telemetry` 기능이 비활성화되어 있다면 활성화한다. 다시 `Save and Reboot`를 클릭한다.
3. **Receiver 탭**:
   - 모든 설정이 올바르게 되었다면, 조종기의 스틱(Throttle, Yaw, Pitch, Roll)을 움직였을 때 해당 채널의 막대그래프가 부드럽게 움직이는 것을 확인할 수 있다.57
   - 만약 롤 스틱을 움직였는데 피치 막대가 움직이는 등 채널이 뒤섞여 있다면, `Channel Map`을 조정해야 한다. 대부분의 조종기는 `AETR1234` 또는 `TAER1234` 순서를 따르므로, 맞는 것을 선택하면 된다.58
4. **OSD 탭**: 비행 중 중요한 링크 정보를 확인하기 위해 `RSSI dBm`, `Link Quality` 등의 항목을 OSD 화면에 배치한다.61


iNav의 설정 과정은 Betaflight와 매우 유사하다.

1. **Ports 탭**: Betaflight와 동일하게, 수신기가 연결된 UART의 `Serial RX`를 활성화한다.62
2. **Configuration 탭**: `Receiver` 섹션에서 `Receiver Type`을 `Serial`로, `Serial Provider`를 `CRSF`로 선택한다.62
3. **Receiver 탭**: 조종기 입력이 정상적으로 들어오는지 확인하고, 필요시 채널 순서를 조정한다.
4. GPS와 함께 사용하는 경우, RTH(Return to Home)와 같은 자율 비행 기능을 위해 페일세이프 설정을 더욱 정교하게 구성하는 것이 중요하다.62


바인딩은 송신기와 수신기를 짝지어주는 과정이며, 페일세이프는 링크가 끊어졌을 때의 비상 동작을 정의하는 핵심 안전 기능이다.


1. 드론에 배터리를 연결하여 수신기의 전원을 켠다.
2. 수신기에 있는 작은 바인드 버튼을 짧게 눌러 바인드 모드(녹색 LED가 느리게 점멸)로 진입시킨다.10
3. 조종기의 `TBS Agent Lite` 스크립트에서 `Bind` 메뉴를 선택하여 실행하거나, 송신 모듈의 바인드 버튼을 짧게 누른다.10
4. 바인딩이 시작되면 수신기와 송신기의 펌웨어 버전이 일치하는지 확인한다. 버전이 다를 경우, 조종기 화면에 수신기 펌웨어를 무선(OTA, Over-the-Air)으로 업데이트할 것인지 묻는 메시지가 나타난다. `Confirm`을 선택하여 업데이트를 진행한다.10
5. 업데이트와 바인딩이 성공적으로 완료되면 수신기의 LED가 녹색으로 계속 켜져 있는 상태가 된다.10


- **개념**: 페일세이프는 조종 신호가 1.5초(기본값) 이상 끊겼을 때(Failsafe), 드론이 사전에 정의된 절차에 따라 동작하도록 하는 가장 중요한 안전장치이다.70
- **설정 위치**: Betaflight Configurator의 `Failsafe` 탭.
- **Stage 2 절차 (Failsafe Procedure)**:
  - **Drop (기본값)**: 가장 권장되는 방식. 즉시 모터를 정지시키고(Disarm), 그 자리에서 추락한다. 예측 불가능한 비행으로 인한 2차 사고(인명, 재산 피해)를 방지하는 가장 안전한 방법이다.70
  - **Land**: 기체의 수평을 잡고 천천히 하강하여 착륙을 시도한다. 지면이 평평하지 않거나 바람이 부는 등 예측 불가능한 환경에서는 오히려 위험할 수 있다.
  - **GPS Rescue**: GPS 모듈이 장착되고 정확하게 설정된 경우에만 사용할 수 있다. 신호가 끊기면 사전에 설정된 고도로 상승한 후, 이륙 지점으로 자동 복귀(RTH)를 시도한다. 주변 장애물, GPS 위성 수, 자기장 간섭 등 고려해야 할 변수가 많아 신중한 설정과 테스트가 필수적이다.70
- **권장 설정 및 테스트**:
  - 일반적인 FPV 프리스타일 및 레이싱 드론에서는 **`Drop`** 모드가 가장 안전하고 확실한 선택이다.
  - 장거리 비행을 목적으로 하는 GPS 장착 기체에서는 `GPS Rescue`를 신중하게 설정하여 활용할 수 있다.
  - **테스트는 필수이다.** **반드시 기체에서 모든 프로펠러를 제거한 상태에서** 배터리를 연결하고, 조종기에서 Arm 스위치를 켜 모터를 회전시킨다. 그 후, 조종기의 전원을 강제로 꺼서 1-2초 내에 모터가 완전히 정지하는지 반드시 확인해야 한다.70


Crossfire 시스템의 기본 설정을 마쳤다면, 다음 단계는 비행 스타일과 목적에 맞게 성능을 최적화하고, 발생할 수 있는 문제에 대처하는 방법을 숙지하는 것이다.


Crossfire는 다양한 RF 프로파일과 출력 설정을 제공하므로, 비행 목적에 따라 최적의 조합을 선택할 수 있다.

- **장거리 비행 (Long Range)**:
  - **RF Profile**: 링크의 안정성과 도달 거리를 극대화하기 위해 `50Hz` 모드로 고정하거나, `Dynamic` 모드를 사용한다. 50Hz 모드는 LoRa 변조를 사용하여 최고의 수신 감도를 제공하므로, 신호가 미약한 한계 상황에서 링크를 유지하는 데 가장 유리하다.19
  - **TX Power**: `Dynamic Power`를 `On`으로 설정하고, `Max Power`를 250mW, 500mW 또는 그 이상으로 충분히 높게 설정한다. 이렇게 하면 평소에는 낮은 출력으로 비행하여 전력을 아끼다가, 거리가 멀어지거나 장애물로 인해 신호가 약해지면 시스템이 자동으로 출력을 높여 링크를 복원한다.10
  - **안테나 시스템**: 송신기에는 TBS Diamond와 같은 고이득(High-gain) 지향성 안테나를 사용하여 특정 방향으로 신호 에너지를 집중시키는 것이 효과적이다. 수신기 측에서는 Diversity RX를 사용하여 두 개의 안테나로 신호를 수신함으로써 '널(null)' 지점으로 인한 신호 손실을 방지하고 링크 신뢰도를 비약적으로 향상시킬 수 있다.37 안테나의 편파(Polarization)를 일치시키고, 기체의 카본 프레임이나 배터리로부터 안테나 소자를 최대한 멀리, 그리고 서로 다른 각도로 배치하는 것이 매우 중요하다.22
- **프리스타일 및 레이싱 (Freestyle & Racing)**:
  - **RF Profile**: 지연 시간을 최소화하여 조종 입력에 대한 기체의 반응성을 극대화하는 것이 가장 중요하다. 따라서 RF 프로파일을 `150Hz`로 고정해야 한다.20 CRSFShot 기능이 포함된 최신 펌웨어(CRSF V2 이상)를 사용하면 가변 지연 시간(jitter)이 제거되어 더욱 일관되고 예측 가능한 조종감을 얻을 수 있다.7
  - **TX Power**: 수백 미터 이내의 근거리에서 비행하는 경우가 대부분이므로, 25mW 또는 100mW의 낮은 출력으로도 충분하다. 낮은 출력은 배터리 소모를 줄여줄 뿐만 아니라, 여러 명의 파일럿이 동시에 비행하는 레이싱 환경에서 채널 간 상호 간섭을 줄이는 데 매우 중요한 역할을 한다.
  - **Telemetry**: 극단적으로 지연 시간에 민감한 레이싱 파일럿의 경우, LUA 스크립트에서 수신기의 `Telemetry` 기능을 `Off`로 설정하기도 한다.42 이는 다운링크 데이터 전송에 사용되는 대역폭을 모두 업링크 제어 신호 전송에 할당하여, 이론적으로나마 지연 시간을 미세하게 줄이는 효과를 기대할 수 있다.


Crossfire는 다양한 텔레메트리 데이터를 제공하며, 이 값들을 올바르게 해석하는 것은 안전한 비행의 핵심이다.

- **주요 텔레메트리 값의 의미**:
  - **LQ (Link Quality)**: 수신된 패킷 중 유효한 패킷의 비율을 백분율(%)로 나타낸다. 100%가 최상의 상태이며, 70% 이하로 떨어지기 시작하면 링크가 불안정해지고 있다는 신호이므로 주의해야 한다. LQ는 단순히 신호의 세기(RSSI)가 아닌, 실제 데이터 통신의 성공률을 보여주기 때문에 링크의 건강 상태를 판단하는 가장 직접적이고 신뢰할 수 있는 지표이다.5
  - **RSSI (Received Signal Strength Indicator)**: 수신 신호의 절대적인 세기를 dBm 단위로 나타낸다. 0에 가까울수록 신호가 강하며, 음수 값이 커질수록 신호가 약해짐을 의미한다. Crossfire의 이론적인 한계 수신 감도는 `-130dBm`이다.5 RSSI는 주변 환경의 노이즈 레벨에 따라 같은 값이라도 실제 링크 품질이 다를 수 있어, LQ와 함께 종합적으로 판단해야 한다.
  - **RFMD (RF Mode)**: 현재 사용 중인 RF 프로파일을 숫자로 보여준다 (예: `1` = 50Hz, `2` = 150Hz). OSD에 이 값을 표시해두면, Dynamic 모드 사용 시 현재 링크가 장거리 모드로 작동하는지, 저지연 모드로 작동하는지 실시간으로 파악할 수 있다.5
- **텔레메트리 링크의 비대칭성 이해**:
  - Crossfire 시스템에서 지상 송신기(TX)의 제어 신호 송신 출력은 최대 1W 또는 2W까지 설정할 수 있지만, 기체 수신기(RX)의 텔레메트리 데이터 송신 출력은 그보다 훨씬 낮다. 예를 들어, 표준 Crossfire Nano RX의 텔레메트리 출력은 약 25mW에서 40mW 수준에 불과하다.73
  - 이러한 출력의 비대칭성 때문에, 거리가 멀어지면 제어 신호(Uplink)는 정상이지만 텔레메트리 신호(Downlink)가 먼저 약해져 조종기에서 "Telemetry Lost" 경고가 발생하는 것은 지극히 정상적인 현상이다.74 이 경고가 떴다고 해서 즉시 조종 불능 상태에 빠지는 것은 아니므로 당황할 필요는 없다. 하지만 이는 링크가 한계에 가까워지고 있다는 중요한 신호이므로, 기체를 회항시키는 판단의 근거로 삼아야 한다.
  - TBS는 이러한 비대칭성 문제를 해결하기 위해 최대 500mW의 높은 텔레메트리 출력을 지원하는 Crossfire Nano RX Pro 모델을 출시했다.7


- **문제: 수신기 인식 불가 / Betaflight Receiver 탭에서 입력 없음**
  1. **배선 재확인**: 가장 흔한 원인이다. 수신기의 TX 핀이 FC의 RX 핀으로, 수신기의 RX 핀이 FC의 TX 핀으로 **교차 연결**되었는지 다시 한번 확인한다.48
  2. **FC 포트 설정 확인**: Betaflight의 `Ports` 탭에서 수신기를 연결한 UART에 `Serial RX`가 올바르게 활성화되었는지 확인한다.10
  3. **FC 프로토콜 설정 확인**: `Configuration` 탭에서 수신기 모드가 `Serial (via UART)`로, 프로토콜이 `CRSF`로 정확히 설정되었는지 확인한다.10
  4. **수신기 출력 맵 확인**: 조종기 LUA 스크립트에서 수신기의 `Output Map`이 `CRSF TX` 및 `CRSF RX`로 설정되어 있는지 반드시 확인한다. 이 설정이 SBUS나 PPM으로 되어 있으면 FC는 CRSF 신호를 수신할 수 없다.10
  5. **UART 포트 변경**: 드물지만 FC의 특정 UART 포트가 물리적으로 손상되었을 수 있다. 다른 비어있는 UART 포트로 수신기를 옮겨 납땜한 후, `Ports` 탭 설정을 변경하여 테스트해본다.75
- **문제: 텔레메트리 유실(Telemetry Lost) 경고가 너무 자주 발생**
  1. **정상 동작 범위 이해**: 앞서 설명했듯이, 제어 링크보다 텔레메트리 링크가 먼저 끊어지는 것은 정상일 수 있다. 특히 저출력으로 비행 시 근거리에서도 발생할 수 있다.74
  2. **펌웨어 버전 불일치**: 송신기, 수신기, 조종기(OpenTX/EdgeTX)의 펌웨어 버전이 서로 호환되지 않을 때 통신 오류가 발생할 수 있다. TBS Agent를 통해 모든 장치를 최신 안정(Stable) 버전으로 업데이트하고 다시 바인딩한다.77
  3. **지역(Region) 설정 오류**: LUA 스크립트에서 `Region`이 `Open`으로 설정되어 있는지 확인한다. 특정 지역 제한 모드(예: EU LBT)로 설정되면 출력 제한으로 인해 통신 거리가 짧아질 수 있다.77
  4. **안테나 상태 및 위치 점검**: 수신기 안테나(특히 Immortal T 안테나의 끝부분)가 손상되지 않았는지, 커넥터가 헐거워지지 않았는지 확인한다. 또한 안테나가 카본 프레임이나 배터리 케이블과 같은 전도성 물체에 직접 닿아 신호가 차폐되고 있는지 점검하고, 최대한 이격시켜 배치한다.22



TBS Crossfire는 지난 수년간 장거리 및 고신뢰성 RC 링크 시장을 개척하고 선도해 온, 기술적으로 성숙하고 검증된 시스템이다. 900MHz 대역의 물리적 이점과 LoRa 변조 기술을 바탕으로 한 뛰어난 통신 거리, 상황에 따라 저지연 모드로 전환하는 유연성, 그리고 무엇보다 '그냥 작동하는' 직관적인 사용자 경험과 강력한 생태계는 Crossfire가 FPV 파일럿들로부터 깊은 신뢰를 얻은 핵심 요인이다.19 특히 비행 데이터의 무결성을 보장하는 데이터 암호화 기능은 상업용 드론이나 보안이 중요한 임무에서 다른 프로토콜이 제공하지 못하는 독보적인 가치를 제공한다.19


2020년대에 들어서면서, 오픈소스 프로젝트인 ExpressLRS(ELRS)는 Crossfire의 강력한 경쟁자로 부상하며 RC 링크 시장의 판도를 바꾸고 있다. 두 시스템은 기술 철학에서부터 성능, 가격에 이르기까지 뚜렷한 차이를 보이며, 이는 사용자의 선택에 중요한 기준이 된다.

- **기술 철학과 개발 방식**: Crossfire가 TBS라는 단일 기업에 의해 개발되고 관리되는 '안정성과 편의성' 중심의 상용 시스템이라면, ELRS는 전 세계 개발자 커뮤니티가 참여하여 '최고 성능과 개방성'을 추구하는 오픈소스 프로젝트이다.19 이 차이는 신기능 도입 속도와 하드웨어 가격 경쟁력에서 극명하게 드러난다.
- **성능 비교**:
  - **지연 시간 (Latency)**: ELRS는 2.4GHz 대역에서 최대 1000Hz, 900MHz 대역에서 200Hz에 달하는 압도적으로 높은 패킷 속도를 지원한다. 이는 Crossfire의 최대 150Hz보다 훨씬 빠르며, 레이싱과 같이 극도의 반응성이 요구되는 분야에서 ELRS가 명백한 우위를 점하는 이유이다.19
  - **통신 거리 (Range)**: 이론적으로는 900MHz 대역이 2.4GHz보다 장애물 투과와 장거리 통신에 유리하다. 하지만 ELRS는 모든 패킷 속도에서 고효율 LoRa 변조를 사용하는 반면, Crossfire는 150Hz 모드에서 FSK 변조로 전환한다. 이 때문에 다수의 실증 테스트에서 동일 출력 조건일 때 ELRS가 Crossfire보다 더 나은 거리 성능을 보여준다는 결과가 보고되고 있다.19 900MHz 대역 내에서의 비교에서도 ELRS는 더 다양한 LoRa 대역폭 옵션을 활용하여 Crossfire보다 우수한 성능을 보이는 것으로 평가된다.82
  - **안정성 및 신뢰성**: Crossfire는 수년간 축적된 데이터와 사용자 경험을 통해 그 안정성을 입증받았으며, 데이터 암호화 기능은 외부의 악의적인 공격으로부터 링크를 보호한다.19 반면, ELRS는 암호화 기능이 없어 이론적으로는 재밍(jamming)이나 하이재킹에 더 취약할 수 있다.19 하지만 일반적인 취미 비행 환경에서는 이 차이가 문제 되는 경우는 드물며, ELRS 역시 매우 견고한 링크를 제공한다는 것이 중론이다.
- **사용 편의성 및 가격**: Crossfire는 직관적인 바인딩 과정과 무선(OTA) 펌웨어 업데이트 등 사용자 경험 측면에서 더 매끄럽다는 평가를 받는다.83 ELRS는 초기 설정과 펌웨어 업데이트 과정(WiFi를 이용한 플래싱 등)이 상대적으로 더 복잡하게 느껴질 수 있다. 그러나 가격 면에서는 오픈소스 기반의 ELRS가 다양한 제조사에서 저렴한 하드웨어를 출시하면서 Crossfire 대비 압도적인 우위를 점하고 있다.19


현재 FPV 시장은 두 시스템의 경쟁 구도로 재편되고 있다. Crossfire는 검증된 신뢰성과 사용 편의성, 암호화 기능을 바탕으로 고가의 촬영 장비를 운용하는 상업용 드론 시장이나, 고정익 장거리 비행 등 안정성이 최우선시되는 분야에서 여전히 강력한 입지를 유지할 것으로 보인다.

반면, 일반적인 취미용 FPV 시장의 주도권은 월등한 성능과 저렴한 가격을 앞세운 ELRS로 빠르게 넘어가고 있다.79 TBS는 이러한 변화에 대응하여 수신기와 VTX를 통합한 Sixty9과 같은 혁신적인 제품을 출시하거나, 기존 시스템의 안정성과 편의성을 더욱 강화하는 방향으로 경쟁력을 유지하려 할 것이다.

궁극적으로 CRSF와 ELRS의 경쟁은 FPV RC 링크 기술의 발전을 가속화하는 긍정적인 동력으로 작용하고 있다. 사용자는 자신의 비행 스타일, 예산, 그리고 기술적 선호도에 따라 두 훌륭한 시스템 중 하나를 선택할 수 있는 풍요로운 시대에 살고 있다.


**Table 3: CRSF vs. ELRS 핵심 비교**

| 구분                | TBS Crossfire                 | ExpressLRS (ELRS)                      |      |
| ------------------- | ----------------------------- | -------------------------------------- | ---- |
| **개발 주체**       | TBS (상용, 독점)              | 커뮤니티 (오픈소스)                    |      |
| **주파수 대역**     | 868 / 915 MHz                 | 2.4 GHz, 868 / 915 MHz                 |      |
| **변조 방식**       | 50Hz: LoRa / 150Hz: FSK       | 모든 패킷 속도에서 LoRa                |      |
| **최대 패킷 속도**  | 150 Hz                        | 1000 Hz (2.4G), 200 Hz (900M)          |      |
| **지연 시간**       | 상대적으로 높음               | 매우 낮음                              |      |
| **통신 거리**       | 매우 김 (검증됨)              | 매우 김 (동일 출력에서 우위)           |      |
| **데이터 암호화**   | 지원                          | 미지원                                 |      |
| **가격**            | 고가                          | 저가                                   |      |
| **사용 편의성**     | 높음 (직관적, OTA 업데이트)   | 상대적으로 복잡 (WiFi 플래싱)          |      |
| **하드웨어 생태계** | TBS 독점                      | 다수 제조사 참여                       |      |
| **핵심 장점**       | 검증된 신뢰성, 간편함, 생태계 | 최고 성능, 저렴한 가격, 빠른 개발 속도 |      |

자료 출처: 19


1. docs.px4.io, accessed August 11, 2025, [https://docs.px4.io/main/en/telemetry/crsf_telemetry#:~:text=CRSF%20Telemetry%20(TBS%20Crossfire%20Telemetry)%20%E2%80%8B,on%20a%20compatible%20RC%20transmitter.](https://docs.px4.io/main/en/telemetry/crsf_telemetry#:~:text=CRSF Telemetry (TBS Crossfire Telemetry) ,on a compatible RC transmitter.)
2. CRSF Protocol Repo / Issue #26 / tbs-fpv/freedomtx - GitHub, accessed August 11, 2025, https://github.com/tbs-fpv/freedomtx/issues/26
3. TBS Crossfire: ALL Of Your Questions Answered In ONE Post ..., accessed August 11, 2025, https://noirfpv.com/tbs-crossfire-faq/
4. CRSF Telemetry (TBS Crossfire Telemetry) | PX4 User Guide (v1.14), accessed August 11, 2025, https://docs.px4.io/v1.14/en/telemetry/crsf_telemetry.html
5. CRSF Telemetry (TBS Crossfire Telemetry) | PX4 Guide (main) - PX4 docs, accessed August 11, 2025, https://docs.px4.io/main/en/telemetry/crsf_telemetry
6. FPV Protocols Explained (CRSF, SBUS, DSHOT, ACCST, PPM, PWM and more), accessed August 11, 2025, https://oscarliang.com/rc-protocols/
7. TBS CROSSFIRE R/C System - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/media/files/tbs-crossfire-manual.pdf
8. TBS Crossfire TX - Long Range R/C Transmitter - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/products/prod:crossfire_tx
9. TBS Crossfire Micro TX V2 - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/products/prod:crossfire_micro_tx
10. How to Setup TBS Crossfire and Tracer - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/crossfire-betaflight/
11. TBS Crossfire Telemetry - Plane documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/plane/docs/common-crsf-telemetry.html
12. Crossfire and ELRS RC Systems - Copter documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/copter/docs/common-tbs-rc.html
13. Documentation on the Crossfire RX protocol. : r/fpv - Reddit, accessed August 12, 2025, https://www.reddit.com/r/fpv/comments/1bqwyvj/documentation_on_the_crossfire_rx_protocol/
14. Mini Quad Long Range RC Options: TBS Crossfire & FrSky R9M - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/mini-quad-long-range-rc-options-tbs-crossfire-frsky-r9m/
15. TBS Crossfire Tx - BlackSheep 팀 868MHZ / 915MHZ 1.1W 3.2W 76g 긴 분노 R/C 송신기, accessed August 11, 2025, https://rcdrone.top/ko/products/tbs-crossfire-tx
16. TBS CROSSFIRE MICRO TX V2 - Team BlackSheep 868MHz / 915MHZ 1.1W 2W 48, accessed August 11, 2025, https://rcdrone.top/ko/products/tbs-crossfire-micro-tx-v2
17. TBS 다이아몬드 안테나 868 915MHz Crossfire CRSF TX 35KM TBS Tracer Crossfire 송신기 용 장거리 FPV Drone Radio System - AliExpress, accessed August 11, 2025, https://ko.aliexpress.com/item/1005005644893890.html
18. (1/3) TBS Crossfire Series: Overview and Introduction - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=_2Xu14sFUss
19. ExpressLRS vs Crossfire: Which Radio Link is Best? - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/expresslrs/
20. Crossfire freestyle : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/lrd15x/crossfire_freestyle/
21. Review: R9M-Lite Module & Latency Testing vs TBS Crossfire - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/r9m-lite-crossfire-latency-testing/
22. Poor Crossfire Range - Team BlackSheep, accessed August 11, 2025, https://team-blacksheep.freshdesk.com/support/solutions/articles/4000099977-poor-crossfire-range
23. TBS TRACER 2.4GHz RC System - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/media/files/tbs-tracer-manual.pdf
24. TBS Crossfire Vs. FrSkt R9 - Initial thoughts | Mr. D - Falling with Style - MrD-RC.com, accessed August 11, 2025, https://www.mrd-rc.com/blog/tbs-crossfire-vs-frskt-r9-initial-thoughts/
25. TBS crossfire micro version 1 vs version 2 : r/fpvracing - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpvracing/comments/lsvnbt/tbs_crossfire_micro_version_1_vs_version_2/
26. TBS Crossfire Micro TX V2 - GetFPV, accessed August 11, 2025, https://www.getfpv.com/tbs-crossfire-micro-tx-v2.html
27. TBS Crossfire Micro TX Module V2 - NDAA - Lumenier, accessed August 11, 2025, https://www.lumenier.com/products/tbs-crossfire-micro-tx-module-v2-ndaa
28. New to TBS protocols: TBS Crossfire vs TBS Nano? : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/1asjn8o/new_to_tbs_protocols_tbs_crossfire_vs_tbs_nano/
29. TBS Crossfire Nano TX - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/products/prod:xf_nano_tx
30. TBS Crossfire Nano Rx - FPV LONG RANGE DRONE RECEIVER - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/products/prod:crossfire_nano_rx
31. TBS Crossfire Nano Rx Special Edition FPV Long Range Drone Receiver - Pyrodrone, accessed August 11, 2025, https://pyrodrone.com/products/tbs-crossfire-nano-rx
32. TBS Crossfire Nano Rx - GetFPV, accessed August 11, 2025, https://www.getfpv.com/tbs-crossfire-nano-rx.html
33. TBS CROSSFIRE Nano RX - Team BlackSheep, accessed August 11, 2025, https://www.team-blacksheep.com/media/files/tbs-crossfire-nano-quickstart.pdf
34. TBS Crossfire Diversity Nano Rx ... - Team BlackSheep Online Store, accessed August 11, 2025, https://www.team-blacksheep.com/products/prod:xf_nano_div_rx
35. Nano Diversity 915MHz Receiver For Crossfire Protocol - NDAA - Rotor Riot, accessed August 11, 2025, https://rotorriot.com/products/nano-diversity-915mhz-receiver-for-crossfire-protocol-ndaa
36. TBS Crossfire Diversity Nano Rx - GetFPV, accessed August 11, 2025, https://www.getfpv.com/tbs-crossfire-diversity-nano-rx.html
37. Diversity crossfire worth it? : r/Multicopter - Reddit, accessed August 11, 2025, https://www.reddit.com/r/Multicopter/comments/oa6gsg/diversity_crossfire_worth_it/
38. TBS Crossfire - Pyrodrone, accessed August 11, 2025, https://pyrodrone.com/collections/tbs-crossfire
39. Internal / External RF - EdgeTX User Manual, accessed August 11, 2025, https://manual.edgetx.org/color-radios/model-settings/model-setup/internal-external-rf
40. Radio Preparation - ExpressLRS, accessed August 11, 2025, https://www.expresslrs.org/quick-start/transmitters/tx-prep/
41. Hardware - EdgeTX User Manual, accessed August 11, 2025, https://manual.edgetx.org/color-radios/radio-settings/hardware
42. Crossfire Lua Scripts - YouTube, accessed August 11, 2025, https://m.youtube.com/watch?v=ZvQVqa7HmqI&t=62s
43. How to Install and Run TBS Agent Lite on any OpenTX Crossfire Radio - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=w-dAIvKdDx4
44. TBS Crossfire Beginner Guide (2023 Update) - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=Ypn71lIu8l8
45. Kakute F7 - Avifly.pl, accessed August 11, 2025, [https://avifly.pl/img/cms/zdjecia%20pod%20opisy/Holybro/Holybro_Kakute_F7_Manual.pdf](https://avifly.pl/img/cms/zdjecia pod opisy/Holybro/Holybro_Kakute_F7_Manual.pdf)
46. How to set up TBS or ELRS receiver in Betaflight configurator on SpeedyBee F7mini flight controller?, accessed August 11, 2025, https://speedybee.zendesk.com/hc/en-us/articles/22326784370843-How-to-set-up-TBS-or-ELRS-receiver-in-Betaflight-configurator-on-SpeedyBee-F7mini-flight-controller
47. How to Set Up Crossfire in Betaflight - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=8Btui5iGmfs
48. Troubleshooting | Betaflight, accessed August 11, 2025, https://betaflight.com/docs/wiki/getting-started/troubleshooting
49. BF11372_BLITZ Mini F4 FC Wiring Diagram_20221215 - fixfly.ru, accessed August 11, 2025, [https://fixfly.ru/files/BF11372_BLITZ%20Mini%20F4%20FC%20Wiring%20Diagram_20221215.pdf](https://fixfly.ru/files/BF11372_BLITZ Mini F4 FC Wiring Diagram_20221215.pdf)
50. iFlight BLITZ MINI F4 Wiring Diagram - DJI Digital VTX + Radio, accessed August 11, 2025, https://www.lacameraembarquee.fr/img/cms/fiches-produit/Drone-FPV/Schema-Montage/schema_montage_fc_blitz_mini_f4.pdf
51. USER MANUAL - Robu.in, accessed August 11, 2025, https://robu.in/wp-content/uploads/2025/01/F7-Stack.pdf
52. BF15459_BLITZ F7 V1.2 FC Wiring Diagram_20250313 - RC Maniak, accessed August 11, 2025, https://rcmaniak.pl/pl/p/file/bb51ef88d30454863309b9a768ddd340/BF15459_BLITZ-F7-V1.2-FC-Wiring-Diagram_20250317-2.pdf
53. JBardwell F7 - Joshua Bardwell, accessed August 11, 2025, https://www.fpvknowitall.com/wp-content/uploads/2021/09/RDQ-Bardwell-F7-Manual-v1.3.pdf
54. How to Setup Betaflight Firmware on FPV Drones - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/betaflight-firmware-setup/
55. Finally! Quick Guide To Setting Up Crossfire Nano! How To Wire & Install & Program, accessed August 11, 2025, https://www.youtube.com/watch?v=05pDkrUHI8w
56. Crossfire no input in betaflight 4.3 : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/zc7jci/crossfire_no_input_in_betaflight_43/
57. Receivers (RX) - Betaflight, accessed August 11, 2025, https://betaflight.com/docs/development/Rx
58. Betaflight Channel Map Explained (And Ideal Channel Order in EdgeTX Radio), accessed August 11, 2025, https://oscarliang.com/channel-map/
59. Need some Betaflight help here - Looks like my controls are all jumbled somehow. The quad just spins in the receiver preview section. Anyone know how to set those and calibrate? Using lite radio and nanohawk x : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/xkk3oo/need_some_betaflight_help_here_looks_like_my/
60. Receiver Configuration Issues In Betaflight - Help - DroneTrest, accessed August 11, 2025, https://www.dronetrest.com/t/receiver-configuration-issues-in-betaflight/5355
61. Configuring Crossfire - Betaflight, accessed August 11, 2025, https://betaflight.com/docs/wiki/guides/current/Configuring-Crossfire-LQ-for-use-as-RSSI-on-the-Betaflight-OSD
62. How to Setup iNav on an FPV Drone - Converting From Betaflight - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/setup-inav-fpv-drone/
63. How To Setup iNavflight on a Multirotor - Best FPV Goggles, accessed August 11, 2025, https://fpvfrenzy.com/how-to-setup-inavflight/
64. [FR] Add CRSF mirror/output on another UART / Issue #7269 / iNavFlight/inav - GitHub, accessed August 11, 2025, https://github.com/iNavFlight/inav/issues/7269
65. INAV 7 Quad Setup: full 'step by step' guide and flight demo of POS HOLD and GPS RTH!, accessed August 11, 2025, https://www.youtube.com/watch?v=7Z8QPXio-i8
66. A complete guide to setting up iNav 4 | Mr. D - Falling with Style - MrD-RC.com, accessed August 11, 2025, https://www.mrd-rc.com/tutorials-tools-and-testing/flight-controller-therapy/a-complete-guide-to-setting-up-inav-4/
67. How to Bind with External TBS Crossfire Receiver - BETAFPV Support, accessed August 11, 2025, https://support.betafpv.com/hc/en-us/articles/900003638046-How-to-Bind-with-External-TBS-Crossfire-Receiver
68. Need Help: Binding TBS Crossfire Nano Rx to TBS Crossfire Micro Tx V2 : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/qkpohv/need_help_binding_tbs_crossfire_nano_rx_to_tbs/
69. How-to: Binding - TBS Nano RX - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=-iNkVcOLITM
70. What's Failsafe and How to Setup on FPV Drone (Betaflight & EdgeTX Radio) - Oscar Liang, accessed August 11, 2025, https://oscarliang.com/setup-failsafe/
71. Your TBS Crossfire Questions Covered (Part 1) - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=eI_ft5vhDKA
72. TBS Crossfire Telemetry - SAVE YOUR QUAD! - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=bKgPltHrdJs
73. Crossfire RC-link good range, telemetry very poor - Other Hardware - ArduPilot Discourse, accessed August 11, 2025, https://discuss.ardupilot.org/t/crossfire-rc-link-good-range-telemetry-very-poor/122710
74. Telemetry on Crossfire : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/10jwrfe/telemetry_on_crossfire/
75. Crossfire transmitter bound but Betaflight receiver tab not working : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/10e73eu/crossfire_transmitter_bound_but_betaflight/
76. TBS Crossfire - Setup Help : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/11i0j47/tbs_crossfire_setup_help/
77. Crossfire not connecting after seemingly successful bind : r/Multicopter - Reddit, accessed August 11, 2025, https://www.reddit.com/r/Multicopter/comments/hqeauw/crossfire_not_connecting_after_seemingly/
78. TBS Crossfire Keeps Losing Bind WORKAROUND!!! | CRSFShot Nightly Build Issue, accessed August 11, 2025, https://www.youtube.com/watch?v=8Mu2Sp38H6Y
79. ELRS VS CROSSFIRE : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/w20yr8/elrs_vs_crossfire/
80. New to FPV.... ELRS or Crossfire? - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/y5nquc/new_to_fpv_elrs_or_crossfire/
81. ExpressLRS 2.4 vs Crossfire and TBS Tracer: range and penetration | 360 Rumors, accessed August 11, 2025, https://360rumors.com/expresslrs-2-4-vs-crossfire-tbs-tracer-range-penetration/
82. CRSF vs. ELRS? : r/fpv - Reddit, accessed August 11, 2025, https://www.reddit.com/r/fpv/comments/1dho3tr/crsf_vs_elrs/
83. Ever since ELRS 3 came out, I haven't met many people that argue that Crossfire - Hacker News, accessed August 11, 2025, https://news.ycombinator.com/item?id=40457051
84. FPV Receiver: difference between 2.4GHz 915MHz and TBS ELRS - Mepsking.com, accessed August 11, 2025, https://www.mepsking.shop/blog/fpv-receiver-difference-between-2-4ghz-and-915mhz-and-tbs-elrs.html
85. CRSF Rev07 | PDF | Parameter (Computer Programming) | Duplex (Telecommunications), accessed August 12, 2025, https://www.scribd.com/document/781193820/CRSF-Rev07
86. Betaflight, ELRS CRC8 calculation for CRSF packets - Stack Overflow, accessed August 12, 2025, https://stackoverflow.com/questions/79248715/betaflight-elrs-crc8-calculation-for-crsf-packets
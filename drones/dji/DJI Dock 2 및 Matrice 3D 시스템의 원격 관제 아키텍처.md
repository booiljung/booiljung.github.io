---
layout: page
title: DJI Dock 2 및 Matrice 3D 시스템의 원격 관제 아키텍처
permalink: /drones/dji/DJI Dock 2 및 Matrice 3D 시스템의 원격 관제 아키텍처
---


산업 현장에서 드론의 역할은 단순 항공 촬영을 넘어, 자동화된 데이터 수집 및 원격 임무 수행 플랫폼으로 빠르게 진화하고 있다. 이러한 패러다임 전환의 중심에는 '드론 인 어 박스(Drone-in-a-Box)' 솔루션이 있으며, DJI Dock 2는 이 분야를 대표하는 제품이다.1 Dock 2 시스템은 기체의 자동 이착륙, 충전, 데이터 관리를 통합하여 최소한의 인력 개입으로 상시적인 드론 운영을 가능하게 한다.

본 보고서는 DJI Dock 2와 Matrice 3D 시스템을 서드파티(third-party) 클라우드 플랫폼에 통합하고자 하는 시스템 아키텍트 및 개발자를 위해 작성되었다. 보고서의 목적은 시스템의 핵심 통신 아키텍처인 DJI Cloud API, 특히 사물 인터넷(IoT) 통신의 표준으로 자리 잡은 MQTT(Message Queuing Telemetry Transport) 프로토콜의 기술적 사양을 심층적으로 분석하는 것이다. 단순한 기능 나열을 넘어, 아키텍처의 설계 철학과 구체적인 구현 방식을 해부함으로써, 성공적인 시스템 통합에 필요한 근본적인 이해와 실질적인 가이드를 제공하고자 한다.3


DJI Dock 2 시스템의 강력함은 정교하게 설계된 하드웨어와 이를 유기적으로 제어하는 계층화된 소프트웨어 생태계의 결합에서 비롯된다. 전체 시스템의 동작 원리를 이해하기 위해서는 먼저 이 두 가지 구성 요소의 거시적 구조를 파악해야 한다.


DJI Dock 2는 자율 운영을 위한 물리적 기반을 제공하는 고내구성 자동화 플랫폼이다. IP55 등급의 방수방진 성능과 −25°C 에서 45°C 에 이르는 넓은 작동 온도 범위를 갖추어, 혹독한 산업 현장에서도 안정적인 상시 운영이 가능하도록 설계되었다.4 이는 군용 등급(−55°C ~ 125°C)에는 미치지 못하지만, 대부분의 산업 및 공공 안전 분야의 요구사항을 충족하는 수준이다.2

Dock에 탑재되는 Matrice 3D/3TD 기체는 특정 임무에 최적화된 페이로드를 장착하고 있다. Matrice 3D는 고정밀 매핑 및 측량 임무를 위해 기계식 셔터가 장착된 광각 카메라를 탑재했으며, Matrice 3TD는 광각, 망원 카메라와 함께 열화상 카메라를 탑재하여 보안, 감시, 수색 및 구조 임무에 특화되어 있다.2 두 기체 모두 IP54 등급의 방수방진 성능을 제공하여 악천후 속에서도 임무 수행이 가능하다.2

Dock 2 자체는 기체의 자동 이착륙과 무선 충전, 데이터 전송을 담당할 뿐만 아니라, 자율 운영의 신뢰성을 높이기 위한 다양한 보조 장치를 내장하고 있다. 고정밀 위치 결정을 위한 RTK(Real-Time Kinematic) 기지국, 현장 기상 상황을 실시간으로 파악하기 위한 풍속계 및 강우량계, 그리고 외부 보안 카메라 등은 시스템이 주변 환경을 스스로 인지하고 안전하게 임무를 수행하기 위한 핵심 구성 요소이다.4

**표 1: DJI Dock 2 및 Matrice 3D/3TD 핵심 하드웨어 제원**

| 구분              | 항목                     | DJI Dock 2                 | DJI Matrice 3D/3TD             |
| ----------------- | ------------------------ | -------------------------- | ------------------------------ |
| **물리적 제원**   | 무게                     | 34 kg (기체 제외)          | 1410 g                         |
|                   | 크기 (L×W×H)             | 570×583×465 mm (덮개 닫힘) | 335×398×153 mm (프로펠러 제외) |
| **환경 내구성**   | IP 등급                  | IP55                       | IP54                           |
|                   | 작동 온도                | −25°C ~ 45°C               | −20°C ~ 45°C                   |
| **운영 성능**     | 최대 작동 반경           | 10 km                      | -                              |
|                   | 최대 이착륙 허용 풍속    | 8 m/s                      | 8 m/s                          |
|                   | 최대 임무 수행 저항 풍속 | -                          | 12 m/s                         |
|                   | 최대 비행 시간           | -                          | 50 분                          |
| **카메라 (M3D)**  | 광각 카메라              | -                          | 4/3 CMOS, 20 MP, 기계식 셔터   |
|                   | 망원 카메라              | -                          | 1/2-inch CMOS, 12 MP           |
| **카메라 (M3TD)** | 광각 카메라              | -                          | 1/1.32-inch CMOS, 48 MP        |
|                   | 망원 카메라              | -                          | 1/2-inch CMOS, 12 MP           |
|                   | 열화상 카메라            | -                          | 640×512@30fps                  |

자료: 1

이러한 하드웨어 제원은 개발자가 시스템의 물리적, 환경적 제약을 명확히 인지하고 운영 시나리오를 설계하는 데 중요한 기준이 된다. 예를 들어, 최대 이착륙 허용 풍속(8 m/s)과 임무 수행 중 최대 저항 풍속(12 m/s)의 차이는, 임무 계획 시 이착륙 시점과 비행 중의 기상 조건을 별도로 고려해야 함을 의미한다.


DJI는 다양한 개발 목적과 기술 수준에 맞춰 여러 계층의 SDK(Software Development Kit) 및 API를 제공하여 강력한 개발자 생태계를 구축하고 있다. 이는 Mobile SDK(모바일 앱 개발), Payload SDK(커스텀 페이로드 개발), Onboard SDK(기체 내 온보드 컴퓨터 연동), 그리고 본 보고서의 핵심 주제인 Cloud API와 Edge SDK로 구성된다.9

**Cloud API**는 서드파티 클라우드 서버가 DJI Pilot 2 애플리케이션 또는 DJI Dock을 게이트웨이 삼아 원격으로 기기를 제어하고 데이터를 동기화할 수 있도록 설계된 최상위 추상화 계층이다. Cloud API의 가장 큰 장점은 별도의 모바일 앱 개발 없이, 기존에 보유한 클라우드 인프라와 웹 기술만으로 DJI 하드웨어를 통합하여 원격 관제 솔루션을 구축할 수 있다는 점이다.3

**Payload SDK (PSDK)**는 서드파티 하드웨어 개발자를 위한 도구로, 특정 산업용 센서나 낙하산 시스템과 같은 커스텀 페이로드를 기체에 물리적으로 통합하고 소프트웨어적으로 제어할 수 있는 인터페이스를 제공한다.11 이를 통해 기본 제공 페이로드만으로는 수행할 수 없는 특수 임무 수행이 가능해져 시스템의 확장성이 극대화된다.

**Edge SDK**는 DJI Dock 내부에 탑재된 컴퓨팅 자원을 활용하여 엣지 애플리케이션을 개발하기 위한 도구다. 클라우드와의 통신 지연을 최소화하고, 네트워크 연결이 불안정한 상황에서도 독립적으로 데이터 처리 및 의사결정을 수행할 수 있어 실시간 반응성이 중요한 시나리오에 적합하다.9

DJI가 이처럼 계층화된 API를 제공하는 것은 단순한 기술 지원을 넘어, 개방형 플랫폼 생태계를 구축하려는 전략적 의도를 보여준다. 각 산업 분야의 고객들은 고유한 요구사항을 가지고 있으며, DJI가 이 모든 것을 단일 솔루션으로 제공하는 것은 불가능하다. 대신, DJI는 안정적인 비행 플랫폼이라는 핵심 기술을 제공하고, 각기 다른 개발자층을 겨냥한 API를 통해 외부 개발자들이 그 위에 자신들의 전문 솔루션을 쉽게 구축할 수 있도록 지원한다.

예를 들어, 드론 안전 솔루션 기업 Dronavia는 PSDK를 이용해 Matrice 3D용 낙하산 시스템을 개발했다. 이 시스템은 비행 중 이상 상황 발생 시 자동으로 전원을 차단하고 낙하산을 전개한다. 여기서 더 나아가, 이 PSDK 기반 페이로드는 Cloud API를 통해 DJI의 공식 관제 소프트웨어인 FlightHub 2와 연동된다. 이를 통해 운영자는 원격으로 지오펜싱(Geofencing) 기능을 설정하여 특정 구역을 벗어날 경우 낙하산이 자동으로 작동하도록 설정할 수 있다.11 이는 PSDK를 통한 하드웨어 확장과 Cloud API를 통한 원격 관제 기능이 어떻게 시너지를 내어 새로운 가치를 창출하는지 보여주는 명확한 사례다. 따라서 개발자는 자신의 솔루션이 요구하는 제어 수준과 데이터 처리 위치(클라우드, 엣지, 페이로드)를 종합적으로 고려하여 최적의 API 조합을 선택하고 시스템을 설계해야 한다.


DJI Cloud API는 단일 통신 프로토콜에 의존하지 않고, 각기 다른 목적에 최적화된 여러 표준 프로토콜을 조합하여 사용하는 하이브리드 아키텍처를 채택했다.3 이는 시스템의 효율성과 실시간성, 안정성을 모두 확보하기 위한 설계적 선택이다.

- **MQTT (Message Queuing Telemetry Transport):** 경량의 발행/구독(Publish/Subscribe) 메시징 프로토콜로, 기기의 상태 정보(Telemetry), 원격 제어 명령, 이벤트 알림 등 저용량 데이터를 고빈도로 교환하는 데 사용된다. TCP/IP 기반으로 동작하며, 불안정한 네트워크 환경에서도 신뢰성 있는 메시지 전달을 지원하여 IoT 통신의 핵심 프로토콜로 널리 사용된다.15
- **HTTPS (Hypertext Transfer Protocol Secure):** 클라이언트-서버 간의 요청/응답(Request/Response) 모델을 기반으로 하는 프로토콜이다. Cloud API에서는 주로 인증 토큰 교환, 조직(Organization) 정보 바인딩, 미디어 파일 목록 조회 및 업로드 URL 획득과 같이 비실시간성이지만 보안이 중요한 상호작용에 사용된다.16
- **WebRTC/RTMP (Web Real-Time Communication / Real-Time Messaging Protocol):** 실시간 영상 및 음성 스트리밍을 위한 프로토콜이다. MQTT는 스트리밍 세션의 시작, 중지, 화질 변경과 같은 '제어 신호(Signaling)'를 전달하는 역할을 하고, 실제 대용량 미디어 데이터는 WebRTC 또는 RTMP와 같은 전용 스트리밍 프로토콜을 통해 서버로 직접 전송된다.17

이처럼 각 프로토콜의 역할을 명확히 분담함으로써, 시스템은 전체적인 통신 효율을 최적화한다. 예를 들어, 드론의 현재 배터리 잔량을 1초마다 확인하는 기능은 경량의 MQTT를 사용하고, 어제 촬영한 고해상도 사진 파일을 다운로드하는 기능은 HTTPS를 통해 얻은 임시 URL을 사용하는 방식이다.

**표 2: Cloud API 통신 프로토콜별 역할 및 특징**

| 프로토콜        | 주요 역할                         | 통신 모델                    | 주요 사용 사례                                               | 특징                                     |
| --------------- | --------------------------------- | ---------------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| **MQTT**        | 상태 보고, 원격 제어, 이벤트 알림 | 발행/구독 (Pub/Sub)          | 기체 OSD, 카메라 상태, 비행 명령, HMS 경고                   | 경량성, 저전력, 신뢰성 있는 메시징 (QoS) |
| **HTTPS**       | 인증, 파일 메타데이터 관리        | 요청/응답 (Request/Response) | 사용자 로그인, 조직 바인딩, 미디어 목록 조회, 업로드 URL 획득 | 보안성 (TLS 암호화), 표준 웹 기술        |
| **WebRTC/RTMP** | 실시간 미디어 전송                | 스트리밍 (Streaming)         | 드론 카메라 영상 실시간 중계                                 | 저지연, 실시간성                         |

자료: 3


DJI Cloud API 아키텍처의 근간을 이루는 핵심 개념은 '사물 모델(Thing Model)'이다. 이는 드론, 카메라, Dock과 같은 복잡한 물리적 하드웨어의 기능과 데이터를 표준화된 디지털 형식으로 표현하는 일종의 추상화 계층이다. 개발자는 사물 모델을 통해 하드웨어의 구체적인 동작 방식에 대한 깊은 이해 없이도, 표준화된 인터페이스를 통해 기기를 제어하고 데이터를 수신할 수 있다.


사물 모델은 **TSL(Thing Specification Language)** 이라는 JSON 형식의 언어로 기술된다. TSL 파일은 특정 제품이 무엇이고(속성), 무엇을 할 수 있으며(서비스), 어떤 정보를 외부에 알릴 수 있는지(이벤트)를 정의한 명세서다. 이는 물리적 공간에 존재하는 개체를 디지털 세계에서 표현하는 '디지털 트윈(Digital Twin)'의 설계도와 같다.20 개발자는 이 TSL 파일을 참조하여 특정 기기 모델이 제공하는 데이터의 종류와 형식, 실행 가능한 명령의 목록과 파라미터를 사전에 명확히 파악할 수 있다. 이는 하드웨어 모델별로 코드를 수정해야 하는 종속성을 제거하고, 일관성 있는 개발을 가능하게 한다.


사물 모델은 모든 기능을 세 가지 핵심 요소로 분류하여 정의한다.20

- **속성 (Properties):** 기기의 현재 상태를 나타내는 데이터 값을 의미한다. 예를 들어, 기체의 배터리 잔량(`capacity_percent`), 현재 GPS 좌표(`longitude`, `latitude`), 짐벌의 기울기 각도(`gimbal_pitch`) 등이 여기에 해당한다. 속성은 읽기만 가능한(read-only, `r`) 속성과 읽고 쓰는 것이 모두 가능한(read-write, `rw`) 속성으로 구분된다. `rw` 속성의 경우, 클라우드에서 MQTT 메시지를 통해 값을 변경할 수 있다.21
- **서비스 (Services):** 기기가 수행할 수 있는 동작이나 명령을 의미한다. 이는 클라우드에서 기기로 보내는 요청에 해당하며, 일반적으로 비동기적으로 처리된다. 예를 들어, '이륙하라'(`flight_takeoff`), '사진을 한 장 촬영하라'(`camera_take_photo`), '라이브 스트리밍을 시작하라'(`live_start_push`)와 같은 명령들이 서비스로 정의된다.17
- **이벤트 (Events):** 기기에서 특정 조건이 충족되었을 때 자발적으로 발생하는 알림을 의미한다. 이는 기기가 클라우드로 능동적으로 보고하는 정보다. 예를 들어, '예약된 항로 비행 임무가 완료됨', '배터리 잔량이 20% 미만으로 떨어짐', '모터에 오류가 발생함'(HMS, Health Management System)과 같은 상황들이 이벤트로 정의된다.24


사물 모델에서 정의된 각 '속성'에는 `pushMode`라는 중요한 메타데이터가 포함되어 있다. 이 값은 해당 속성 데이터가 어떤 방식으로, 그리고 어떤 채널을 통해 클라우드에 보고될지를 결정하는 핵심적인 역할을 한다.20

- `pushMode = 0`: **주기적 보고(Periodic Reporting).** 기기는 0.5Hz(2초에 한 번)와 같은 사전에 정의된 고정된 주기로 데이터를 클라우드에 보고한다. 이는 비행 고도, 속도, 자세와 같이 지속적으로 값이 변하는 OSD(On-Screen Display)성 데이터에 주로 사용된다.
- `pushMode = 1`: **상태 변경 시 보고(On-Change Reporting).** 데이터의 값이 이전과 달라졌을 때만 클라우드에 보고한다. 이는 비행 모드(예: Normal, Sport), 카메라 촬영 모드(예: Single, Timed)와 같이 상태가 불규칙하게 변하는 데이터에 적합하다.
- `pushMode = 2`: **능동적 보고 없음(No Active Reporting).** 기기가 자발적으로 데이터를 보고하지 않으며, 클라우드에서 명시적으로 요청할 때만 해당 값을 응답한다.

DJI Cloud API는 이 `pushMode` 값을 MQTT 토픽 채널과 직접적으로 매핑하여 통신 효율을 극대화하는 영리한 아키텍처를 채택했다. 드론과 같은 IoT 기기는 초당 수많은 원격 측정 데이터를 생성한다. 이 모든 데이터를 무분별하게 클라우드로 전송하면 막대한 네트워크 대역폭을 소모하고, 서버는 불필요한 데이터를 처리하느라 리소스를 낭비하게 된다.

이 문제를 해결하기 위해, DJI는 데이터의 성격에 따라 각기 다른 MQTT 토픽 채널로 데이터를 분리하여 전송한다.

- `pushMode = 0`으로 정의된 속성들은 `thing/product/{device_sn}/osd` 토픽으로 발행된다.25 이 채널은 고빈도 데이터 스트림을 위한 채널로, 약간의 데이터 유실이 발생하더라도 다음 주기에 새로운 값으로 갱신되므로 전체 시스템에 큰 영향을 주지 않는 데이터에 적합하다.
- `pushMode = 1`로 정의된 속성들은 `thing/product/{device_sn}/state` 토픽으로 발행된다.25 이 채널은 이벤트 기반의 상태 변경 알림을 위한 채널로, 데이터의 변경 자체가 중요한 의미를 가지는 경우에 사용된다.

이러한 설계 덕분에 개발자는 자신의 애플리케이션 요구사항에 맞춰 필요한 토픽만 선택적으로 구독할 수 있다. 예를 들어, 지도 위에 드론의 실시간 비행 경로를 시각화하는 기능은 `osd` 토픽을 구독하고, 드론의 비행 모드가 '임무 수행'에서 '자동 복귀'로 변경될 때 관리자에게 알림을 보내는 기능은 `state` 토픽을 구독하면 된다. 이는 단순한 기술 구현을 넘어, 수많은 기기가 연결되는 대규모 IoT 시스템을 설계하는 데 있어 매우 중요한 확장성 및 효율성 확보 아키텍처 패턴이다.


이 장에서는 DJI Cloud API가 MQTT 5.0 표준을 기반으로 원격 관제 시스템을 어떻게 구현했는지, 인증 및 연결 절차부터 토픽 구조, 데이터 페이로드 스키마에 이르기까지 구체적인 기술 사양을 상세하게 분석한다.


DJI Cloud API는 서드파티 개발자가 자체 클라우드 인프라와 MQTT 브로커를 운영하는 것을 전제로 설계되었다. DJI Dock 2 또는 DJI Pilot 2가 MQTT 클라이언트 역할을 수행하여 개발자의 서버에 접속하는 구조다.18

연결에 필요한 정보(브로커 주소, 포트, 사용자 이름, 암호 등)를 기기에 안전하고 유연하게 전달하기 위해, DJI는 웹뷰(WebView)와 JSBridge(JavaScript Bridge)를 결합한 독특한 메커니즘을 사용한다.

1. **웹뷰 로딩:** 사용자가 DJI Pilot 2 앱에서 '클라우드 서비스 연결'을 시작하면, 앱은 개발자가 사전에 지정한 URL의 웹 페이지를 내장된 웹뷰에 로드한다.16
2. **사용자 인증:** 개발자는 이 웹 페이지에서 자사의 로그인 및 인증 절차를 구현한다. 사용자가 성공적으로 로그인하면, 개발자의 백엔드 서버는 해당 세션에 대해 유효한 MQTT 연결 정보를 동적으로 생성한다.
3. **정보 주입:** 서버는 생성된 MQTT 연결 정보(주소, 임시 토큰 등)를 웹 페이지로 전달한다. 웹 페이지의 자바스크립트 코드는 이 정보를 받아 JSBridge 인터페이스(`window.djiBridge.platformLoadComponent`)를 호출하여 Pilot 앱의 네이티브(Native) 영역으로 안전하게 전달한다.26
4. **MQTT 연결:** Pilot 앱은 전달받은 정보를 사용하여 MQTT 브로커에 연결을 시도한다.

인증 정보 페이로드는 `address`(서버 주소 및 포트), `client_id`(클라이언트 식별자), `username`(사용자 이름), `password`(암호), `expire_time`(인증 정보 만료 시간), `enable_tls`(TLS 암호화 사용 여부) 등의 필드를 포함하는 JSON 객체다.28 보안을 위해 `password` 필드에는 영구적인 암호 대신, JWT(JSON Web Token)와 같이 만료 시간이 있는 단기 토큰을 사용하는 것이 강력히 권장된다.

이러한 방식은 DJI가 각기 다른 서드파티의 고유한 인증 시스템(OAuth2, API Key 등)을 모두 지원해야 하는 복잡성을 피하는 동시에, 민감한 보안 자격 증명이 Pilot 앱이나 DJI 서버를 거치지 않도록 하여 보안 수준을 높이는 효과를 가진다. 개발자는 기존의 인증 시스템을 그대로 활용하면서도, 동적으로 생성된 단기 자격 증명을 통해 기기를 안전하게 연결할 수 있다. 이는 클라우드 통합의 복잡성을 개발자 측에 위임하는 대신, 높은 수준의 유연성과 보안을 제공하는 매우 효율적인 설계라 할 수 있다.


DJI Cloud API의 MQTT 토픽은 일관되고 계층적인 구조를 따른다. 이를 통해 수많은 기기로부터 발생하는 메시지를 체계적으로 관리하고 라우팅할 수 있다. 모든 토픽은 다음과 같은 표준화된 구조를 가진다.25

```
thing/product/{device_sn}/{channel}
```

- `thing/product`: 모든 DJI Cloud API 토픽의 최상위 네임스페이스를 나타내는 고정된 접두사다.
- `{device_sn}`: 메시지를 발행하거나 수신하는 기기의 고유 시리얼 번호(SN)가 위치하는 가변 필드다. 이를 통해 각 기기는 독립적인 토픽 네임스페이스를 가지게 되어 메시지 충돌을 방지한다.
- `{channel}`: 메시지의 목적과 종류를 나타내는 채널 이름이다.

주요 채널의 역할은 사물 모델의 구성 요소와 밀접하게 연관되어 있다.

**표 3: MQTT 주요 토픽 채널 정의**

| 채널명             | 전체 토픽 예시                      | 방향            | 주요 메시지 내용                              | 관련 사물 모델 요소 |
| ------------------ | ----------------------------------- | --------------- | --------------------------------------------- | ------------------- |
| **osd**            | `thing/product/{sn}/osd`            | 기기 → 클라우드 | 고빈도 원격 측정 데이터 (위치, 속도, 자세 등) | 속성 (PushMode=0)   |
| **state**          | `thing/product/{sn}/state`          | 기기 → 클라우드 | 상태 변경 데이터 (비행 모드, 카메라 모드 등)  | 속성 (PushMode=1)   |
| **services**       | `thing/product/{sn}/services`       | 클라우드 → 기기 | 원격 제어 명령 (이륙, 촬영, 줌 등)            | 서비스 (요청)       |
| **services_reply** | `thing/product/{sn}/services_reply` | 기기 → 클라우드 | 서비스 요청에 대한 응답 (성공, 실패, 결과)    | 서비스 (응답)       |
| **events**         | `thing/product/{sn}/events`         | 기기 → 클라우드 | 비동기적 이벤트 알림 (임무 완료, 오류 발생)   | 이벤트              |
| **property/set**   | `thing/product/{sn}/property/set`   | 클라우드 → 기기 | 쓰기 가능한 속성 값 변경 요청                 | 속성 (`rw`)         |

자료: 20

이처럼 체계적인 토픽 구조는 개발자가 특정 목적의 통신을 위해 어떤 토픽을 구독(subscribe)하거나 발행(publish)해야 하는지를 명확하게 알려주며, MQTT 기반 애플리케이션을 설계하는 데 가장 기본적인 청사진 역할을 한다.


MQTT를 통해 교환되는 모든 데이터 페이로드는 JSON 형식을 사용한다. 메시지는 `tid`(Transaction ID), `bid`(Business ID), `timestamp` 등 메타데이터를 포함하는 표준 봉투(envelope) 구조로 래핑될 수 있으며, 실제 데이터는 `data` 필드 내에 포함된다.29

DJI Dock 2 속성 페이로드

Dock 2의 상태를 나타내는 페이로드는 운영에 필수적인 다양한 정보를 포함한다. state 토픽을 통해 보고되는 주요 속성은 다음과 같다.21

**표 4: DJI Dock 2 주요 MQTT 속성(Properties) 페이로드 스키마**

| 속성 이름 (JSON Key)      | 설명               | 데이터 타입    | 제약 조건 / 예시                               | 접근 모드 | PushMode |
| ------------------------- | ------------------ | -------------- | ---------------------------------------------- | --------- | -------- |
| `home_position_is_valid`  | 홈포인트 유효성    | enum_int       | `{"0":"Invalid", "1":"Valid"}`                 | r         | 0        |
| `heading`                 | Dock의 헤딩 각도   | double         | `{"min":"-180", "max":"180", "unit_name":"°"}` | r         | 0        |
| `rtcm_info`               | RTK 보정 소스 정보 | struct         | -                                              | r         | 1        |
| `» source_type`           | 보정 유형          | enum_int       | `{"1":"자체 수렴", "3":"네트워크 RTK"}`        | r         | 0        |
| `air_conditioner`         | 에어컨 작동 상태   | struct         | -                                              | r         | 0        |
| `» air_conditioner_state` | 에어컨 모드        | enum_int       | `{"0":"유휴", "1":"냉방", "2":"난방"}`         | r         | 0        |
| `silent_mode`             | 저소음 모드        | enum_int       | `{"0":"비활성", "1":"활성"}`                   | rw        | 1        |
| `dongle_infos`            | 4G 동글 정보       | array (struct) | -                                              | r         | 1        |
| `» imei`                  | 동글 IMEI          | text           | -                                              | r         | 0        |

자료: 21

Matrice 3D/3TD 속성 페이로드

기체의 페이로드는 고빈도로 전송되는 OSD 데이터와 상태 변경 시 전송되는 데이터로 나뉜다.

**표 5: Matrice 3D/3TD 주요 MQTT 속성(Properties) 페이로드 스키마**

| 속성 이름 (JSON Key) | 설명               | 데이터 타입    | 제약 조건 / 예시                                 | 접근 모드 | PushMode |
| -------------------- | ------------------ | -------------- | ------------------------------------------------ | --------- | -------- |
| `longitude`          | 경도               | double         | `unit_name: "°"`                                 | r         | 0        |
| `latitude`           | 위도               | double         | `unit_name: "°"`                                 | r         | 0        |
| `height`             | 고도 (해발)        | float          | `unit_name: "m"`                                 | r         | 0        |
| `velocity_x`         | X축 속도 (북쪽)    | float          | `unit_name: "m/s"`                               | r         | 0        |
| `attitude_pitch`     | 피치 각도          | float          | `unit_name: "°"`                                 | r         | 0        |
| `capacity_percent`   | 배터리 잔량        | int            | `{"min":0, "max":100, "unit_name":"%"}`          | r         | 0        |
| `flight_status`      | 비행 상태          | enum_int       | `{"0":"대기", "3":"자동 이륙", "7":"자동 착륙"}` | r         | 1        |
| `payloads`           | 페이로드 정보 배열 | array (struct) | -                                                | r         | 1        |
| `» payload_index`    | 페이로드 식별자    | text           | `1-15-0` (M3D 광각)                              | r         | 0        |
| `» zoom_factor`      | 줌 배율            | float          | `{"min":2, "max":200}`                           | rw        | 1        |
| `» camera_mode`      | 카메라 모드        | enum_int       | `{"0":"촬영", "1":"녹화"}`                       | rw        | 1        |
| `» wide_iso`         | 광각 렌즈 ISO      | enum_int       | `{"0":"Auto", "3":"100", "4":"200",...}`         | rw        | 1        |

자료: 22 (M4D/M4TD 및 기타 모델 속성 참조)

이러한 상세 스키마는 개발자가 MQTT 메시지를 정확히 파싱하고, 수신된 데이터를 애플리케이션 로직에서 올바르게 활용하기 위한 필수적인 정보를 제공한다.


MQTT 프로토콜의 사양을 이해했다면, 이를 바탕으로 실제 원격 관제 시나리오에서 비행, 페이로드, 스트리밍 등을 어떻게 제어하는지 구체적인 명령(서비스 호출)을 통해 살펴볼 수 있다.


모든 실시간 비행 제어 명령은 `thing/product/{gateway_sn}/services` 토픽으로 JSON 페이로드를 발행하여 수행된다. 페이로드 내 `method` 필드는 수행할 동작을 지정하고, `data` 필드는 해당 동작에 필요한 파라미터를 담는다.23

- **이륙 (`flight_takeoff`):** 기체를 지정된 안전 고도까지 자동으로 이륙시킨다.
- **자동 복귀 (`flight_rth`):** 기체를 사전에 설정된 홈포인트(Dock)로 자동 복귀시킨다.
- **지점 이동 (`flight_flyto`):** `data` 필드에 목표 지점의 위도(`latitude`), 경도(`longitude`), 고도(`altitude`) 및 목표 헤딩(`heading`)을 지정하여 기체를 해당 지점으로 이동시킨다.
- **강제 착륙 (`force_landing`):** 현재 위치에서 즉시 착륙을 시도하는 비상 명령이다.31

여기서 주목할 점은, 제어 명령 토픽의 `{device_sn}` 부분에 제어 대상인 드론의 시리얼 번호가 아닌, 게이트웨이(Gateway)의 시리얼 번호가 사용된다는 것이다.23 DJI Dock 2 시스템에서 게이트웨이는 Dock 본체를 의미한다. 드론은 인터넷에 직접 연결되지 않고, Dock을 통해 클라우드와 통신하기 때문이다. 클라우드는 게이트웨이인 Dock에 명령을 보내고, Dock은 이 명령을 수신하여 DJI의 사설 무선 프로토콜(예: OcuSync)을 통해 실제 드론에 전달한다. 이 아키텍처는 클라우드 입장에서 통신 대상을 Dock으로 일원화하여 구조를 단순화하고, Dock 또는 조종기(수동 개입 시)에서 클라우드의 원격 제어 요청을 승인하거나 거부하는 등 제어 권한을 중재하는 역할을 수행할 수 있게 하여 안전성을 높인다. 따라서 개발자는 제어 명령을 보낼 때, 제어 대상인 드론의 SN이 아닌, 현재 드론과 통신하고 있는 게이트웨이(Dock)의 SN을 토픽에 사용해야 한다.


페이로드 제어 역시 `services` 토픽을 통해 이루어진다. 여러 개의 카메라가 장착된 경우, `data` 필드 내의 `payload_index` 파라미터를 통해 제어할 카메라(예: 광각, 줌, 열화상)를 특정한다.24

- **짐벌 제어 (`gimbal_rotate`):** `data` 필드에 `pitch`, `roll`, `yaw` 값을 지정하여 짐벌의 자세를 정밀하게 제어한다.
- **카메라 줌 (`camera_zoom`):** `zoom_factor` 값을 설정하여 광학 또는 디지털 줌 배율을 조정한다.
- **사진/영상 촬영:** `camera_take_photo` (사진 촬영), `camera_start_record` (영상 녹화 시작), `camera_stop_record` (영상 녹화 중지) `method`를 사용하여 원격으로 미디어를 캡처한다.
- **카메라 설정 변경:** `camera_set_exposure_mode` (노출 모드), `camera_set_iso` (감도), `camera_set_shutter_speed` (셔터 속도) 등의 `method`를 사용하여 현장 상황에 맞게 카메라의 촬영 파라미터를 실시간으로 변경할 수 있다.32


라이브 스트리밍 기능은 제어 신호와 미디어 데이터 전송 채널이 분리된 구조로 동작한다. MQTT는 스트리밍 세션을 관리하는 제어 신호를 주고받는 데 사용되며, 실제 대용량 비디오 데이터는 RTMP나 WebRTC와 같은 별도의 스트리밍 프로토콜을 통해 개발자가 구축한 미디어 서버로 전송된다.

- **스트림 시작 (`live_start_push`):** `services` 토픽으로 전송된다. `data` 페이로드에는 스트리밍 프로토콜을 지정하는 `video_type`(예: `rtmp`, `webrtc`), 미디어 서버의 주소인 `url`, 그리고 전송 화질을 결정하는 `video_quality` 등의 정보가 포함되어야 한다.17
- **스트림 중지 (`live_stop_push`):** 현재 진행 중인 스트리밍 세션을 종료하는 명령이다.
- **렌즈 전환 (`live_lens_change`):** 스트리밍을 중단하지 않고, 실시간으로 영상 소스를 광각, 줌, 열화상 카메라 간에 전환할 수 있다. 이는 임무 수행 중 다양한 화각의 영상이 필요할 때 유용하다.17

**표 6: 주요 기능별 MQTT 서비스(Services) 호출 명령 및 페이로드 구조**

| 기능 분류       | `method` 이름       | 요청 페이로드 (`data` 필드) 구조                             |
| --------------- | ------------------- | ------------------------------------------------------------ |
| **비행 제어**   | `flight_flyto`      | `{"latitude": 34.05, "longitude": -118.24, "altitude": 100.0, "heading": 90}` |
| **짐벌 제어**   | `gimbal_rotate`     | `{"payload_index": "1-15-0", "pitch": -90, "yaw": 0}`        |
| **카메라 제어** | `camera_take_photo` | `{"payload_index": "1-15-0"}`                                |
| **스트리밍**    | `live_start_push`   | `{"video_type": "rtmp", "url": "rtmp://your-server.com/live/stream_key", "video_quality": 2}` |

자료: 17


DJI Dock 2 시스템은 Cloud API를 통한 원격 관제 기능 외에도, PSDK와 Edge SDK를 통해 하드웨어와 소프트웨어 양면에서 높은 확장성을 제공한다. 이는 DJI 플랫폼을 기반으로 다양한 산업 맞춤형 솔루션을 구축할 수 있는 가능성을 열어준다.


Matrice 3D/3TD 기체는 서드파티 페이로드를 장착할 수 있는 E-Port를 지원하며 8, 개발자는 PSDK를 사용하여 자신만의 특수 센서나 액추에이터를 드론에 통합할 수 있다. PSDK는 페이로드가 드론의 전원을 공급받고, 비행 제어 데이터에 접근하며, 기체와 데이터를 교환할 수 있는 표준화된 인터페이스를 제공한다. PSDK 3.8 버전부터 Matrice 3D 시리즈를 공식적으로 지원하기 시작했다.12

PSDK의 진정한 가치는 Cloud API와의 연동을 통해 발현된다. Cloud API는 `PSDK Interconnection`이라는 기능을 제공하여, 클라우드 플랫폼에서 PSDK로 개발된 커스텀 페이로드와 직접 데이터를 주고받거나 명령을 내릴 수 있는 통로를 마련한다.31 예를 들어, PSDK로 개발된 대기 질 측정 센서를 장착한 드론이 특정 지역의 오염도를 측정하면, 이 데이터를 `PSDK Interconnection` 채널을 통해 클라우드로 실시간 전송할 수 있다. 반대로, 클라우드에서 명령을 보내 드론에 장착된 샘플 채취 장치를 원격으로 작동시키는 것도 가능하다.

이처럼 PSDK가 물리적, 전기적 통합을 담당하고 Cloud API가 원격 제어 및 데이터 통신 인터페이스를 담당함으로써, 서드파티 하드웨어는 DJI 순정 페이로드와 거의 동일한 수준의 원격 운영성을 확보할 수 있다. 이는 DJI가 단순히 드론을 제조하여 판매하는 것을 넘어, 다양한 산업 솔루션이 탄생하고 성장할 수 있는 개방형 '항공 로보틱스 플랫폼' 제공자로 진화하고 있음을 보여주는 가장 강력한 증거다.


Edge SDK는 DJI Dock 내부에 내장된 컴퓨팅 자원을 활용하여 맞춤형 애플리케이션을 개발하고 실행할 수 있게 한다.9 이는 모든 데이터를 클라우드로 전송하여 처리하는 대신, 데이터가 생성되는 지점(엣지)에서 즉시 처리하는 엣지 컴퓨팅 아키텍처를 구현할 수 있게 한다.

주요 활용 시나리오는 다음과 같다.

- **저지연 실시간 분석:** 드론이 촬영한 고화질 영상을 클라우드로 전송하기 전에 Dock에서 1차적으로 AI 기반 객체 탐지(예: 균열, 침입자)를 수행한다. 분석 결과, 이상 상황이 감지되었을 때만 경보와 함께 해당 영상 클립을 클라우드로 전송하여 네트워크 대역폭을 절약하고 반응 속도를 높일 수 있다.
- **네트워크 불안정성 대응:** 클라우드와의 네트워크 연결이 일시적으로 끊어지더라도, Dock의 엣지 애플리케이션이 자체적인 판단에 따라 사전에 정의된 임무를 지속하거나 안전 조치(예: 비상 착륙)를 수행하여 운영의 연속성과 안정성을 보장한다.

Edge SDK로 개발된 애플리케이션은 Cloud API의 `CloudAPI_SendCustomEventsMessage`와 같은 함수를 통해 클라우드와 상호작용할 수 있다.13 이를 통해 엣지에서 처리한 분석 결과나 주요 이벤트 로그를 클라우드 관제 시스템에 보고하여, 중앙 관제와 엣지 자율 운영의 장점을 결합한 하이브리드 시스템을 구축할 수 있다.


DJI Dock 2 및 Matrice 3D 시스템의 원격 관제 아키텍처는 현대적인 IoT 시스템 설계 원칙을 충실히 따르고 있다. 분석을 통해 도출된 핵심적인 설계 특징은 다음과 같다.

- **표준 기술 채택과 추상화:** 표준 IoT 프로토콜인 MQTT와 웹 기술인 HTTPS를 기반으로, 복잡한 하드웨어의 기능을 사물 모델(Thing Model)이라는 개념으로 추상화했다. 이는 개발자가 하드웨어의 내부 동작에 대한 깊은 지식 없이도 표준화된 인터페이스를 통해 시스템을 쉽게 제어하고 통합할 수 있도록 한다.
- **통신 효율 최적화:** 데이터의 성격(`pushMode`)에 따라 MQTT 토픽 채널을 `osd`와 `state`로 분리하여, 고빈도 데이터와 이벤트성 데이터를 각기 다른 채널로 전송한다. 이는 불필요한 네트워크 트래픽과 서버 부하를 줄여 시스템의 확장성과 효율성을 높인다.
- **하이브리드 프로토콜 아키텍처:** 제어 신호(MQTT)와 대용량 미디어 데이터(WebRTC/RTMP) 전송에 각기 다른 프로토콜을 사용하는 하이브리드 방식을 통해, 제어의 용이성과 미디어 전송의 실시간성을 모두 효과적으로 확보했다.
- **개방형 플랫폼 지향:** PSDK, Edge SDK와 같은 강력한 확장 도구를 제공함으로써, DJI는 단순한 드론 제품 제조사를 넘어, 다양한 서드파티 개발자들이 참여하여 새로운 가치를 창출할 수 있는 개방형 항공 로보틱스 플랫폼 생태계를 지향하고 있다.

이러한 아키텍처를 기반으로 서드파티 관제 소프트웨어를 개발하는 개발자들을 위해 다음과 같은 기술적 사항을 권장한다.

- **견고한 상태 관리 로직 구현:** MQTT는 본질적으로 상태를 저장하지 않는(stateless) 메시징 프로토콜이다. 따라서 클라우드 애플리케이션은 각 기기의 최신 상태(위치, 배터리, 비행 모드 등)를 자체적으로 추적하고 관리하는 '디지털 트윈' 로직을 견고하게 구현해야 한다.
- **비동기 처리 모델 채택:** 모든 서비스 호출(명령 전송)은 비동기적으로 처리된다. 명령을 전송한 후에는 `services_reply` 토픽을 통해 즉각적인 응답을 확인하고, `state` 토픽을 통해 명령 실행으로 인한 기기의 최종 상태 변화를 지속적으로 추적하는 로직이 필수적이다.
- **포괄적인 예외 처리 시나리오 설계:** 네트워크 단절, 기기 오류(HMS 이벤트), 명령 실패 응답 등 다양한 예외 상황에 대한 처리 시나리오를 상세히 설계해야 한다. DJI Dock 2는 정전 시 5시간 이상 작동하는 백업 배터리를 갖추고 있으나, 이 동안에는 기체 충전, 에어컨, 풍속계 가열 등 일부 기능이 제한된다는 점도 시스템 설계에 반영해야 한다.4
- **보안 프로토콜 준수:** JSBridge를 통한 인증 정보 주입 과정에서는 반드시 단기 토큰 기반의 안전한 인증 메커니즘을 구현해야 한다. 또한, 모든 MQTT 통신 시 TLS 암호화를 활성화하여 중간자 공격(Man-in-the-middle attack)으로부터 통신 내용을 보호해야 한다.


1. DJI Dock 2, accessed August 21, 2025, https://enterprise.dji.com/dock-2
2. MATRICE 3D SERIES - Unmanned Aircraft Flight Manual - DJI, accessed August 21, 2025, https://dl.djicdn.com/downloads/DJI_Dock_2/User_Manual/20240814/DJI_Dock_2_User_Manual_V2.01_en.pdf
3. Low threshold access to third-party cloud platform - DJI Developer, accessed August 21, 2025, https://developer.dji.com/cloud-api
4. Support for DJI Dock 2, accessed August 21, 2025, https://www.dji.com/support/product/dock-2
5. Installation and Setup Manual - DJI, accessed August 21, 2025, https://dl.djicdn.com/downloads/DJI_Dock_2/User_Manual/20240407/DJI_Dock_2_Installation_And_Setup_Manual_v1.0_EN.pdf
6. DJI Matrice 3D - Terrestrial Imaging, accessed August 21, 2025, https://www.terrestrialimaging.com/products/dji-matrice-3d
7. DJI Dock 2 - Specs, accessed August 21, 2025, https://enterprise.dji.com/dock-2/specs
8. DJI Dock 2: Revolutionize Remote Operations | Advexure, accessed August 21, 2025, https://advexure.com/pages/dji-dock-2
9. DJI Developer, accessed August 21, 2025, https://developer.dji.com/
10. Product Introduction - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/product-introduction.html
11. DJI Matrice 3D & DJI Dock 2 - Kronos M3D - Dronavia, accessed August 21, 2025, https://www.dronavia.com/fr/product/dji-matrice-3d-kronos-3d/?v=82a9e4d26595
12. Releases · dji-sdk/Payload-SDK - GitHub, accessed August 21, 2025, https://github.com/dji-sdk/Payload-SDK/releases
13. DJI Cloud API - Edge SDK, accessed August 21, 2025, https://developer.dji.com/doc/edge-sdk-api-reference/en/cloud/cloud-api.html
14. DJI Matrice 3D & DJI Dock 2 - Kronos M3D - Dronavia, accessed August 21, 2025, https://www.dronavia.com/product/dji-matrice-3d-kronos-3d/
15. MQTT - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/mqtt.html
16. https - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/https.html
17. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-livestream.html
18. Live Stream - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/pilot-feature-set/pilot-livestream.html
19. Service | Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock3/live.html
20. Thing Model - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/thing-model.html
21. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock2/properties.html
22. M4D/M4TD Properties - MQTT - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/aircraft/m4d-properties.html
23. Cloud API - Live Flight Controls/Remote Control - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/pilot-feature-set/drc.html
24. Remote Control - Dock - Event | Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock3/remote-control.html
25. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/topic-definition.html
26. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-access-to-cloud.html
27. JSBridge - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/jsbridge.html
28. Live Flight Controls - Event | Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock2/drc.html
29. Event | Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/mqtt/rc-pro/drc.html
30. drc - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/drc.html
31. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/
32. Cloud-API-Doc/docs/en/00.index.md at master - GitHub, accessed August 21, 2025, https://github.com/dji-sdk/Cloud-API-Doc/blob/master/docs/en/00.index.md
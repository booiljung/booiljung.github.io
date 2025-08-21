# DJI Dock 3 및 Matrice 4D 시스템의 원격 관제 아키텍처

## 1. 차세대 자율 비행 솔루션의 아키텍처 고찰

DJI Dock 3 및 Matrice 4D(M4D) 시스템은 단순한 'Drone-in-a-Box' 솔루션을 넘어선다. 이 시스템은 산업 표준 사물 인터넷(IoT) 프로토콜을 기반으로 구축된 개방형 자율 운영 플랫폼으로서, 전례 없는 수준의 자동화, 유연성, 그리고 확장성을 제공한다.1 기존의 폐쇄적인 드론 시스템과 달리, DJI는 MQTT(Message Queuing Telemetry Transport)라는 경량 메시징 프로토콜을 통신 아키텍처의 핵심으로 채택함으로써 서드파티 개발자와의 통합을 위한 문을 활짝 열었다.

본 보고서는 DJI Dock 3 및 Matrice 4D 시스템의 기술적 사양을 심층적으로 분석하는 것을 목표로 한다. 특히, 시스템의 핵심 통신 프로토콜인 MQTT의 구현 방식에 초점을 맞추어 관제 소프트웨어의 아키텍처, 데이터 흐름, 그리고 서드파티 개발 생태계에서의 확장 가능성을 규명하고자 한다. 이를 위해 먼저 자율 운영의 물리적 기반이 되는 하드웨어 아키텍처를 분석하고, DJI가 제공하는 소프트웨어 생태계의 구조를 살펴본다. 이후 MQTT 프로토콜의 구체적인 구현 방식, 즉 토픽 구조와 데이터 페이로드를 상세히 해부하고, 이를 통해 실시간 비행 제어, 임무 관리, 영상 스트리밍과 같은 핵심 기능이 어떻게 구현되는지 그 메커니즘을 분석한다. 마지막으로, 이러한 MQTT 기반 아키텍처가 갖는 기술적 함의와 개발자 관점에서의 고려사항을 종합적으로 고찰하여 향후 발전 방향을 제시한다.

## 2.  시스템 하드웨어 아키텍처 분석: 자율 운영의 물리적 기반

소프트웨어 아키텍처는 그것이 구동되는 물리적 시스템의 성능과 한계에 깊이 의존한다. DJI Dock 3와 Matrice 4D 시스템의 24/7 무인 자율 운영이라는 목표는 열악한 환경에서도 안정적으로 임무를 수행할 수 있는 견고한 하드웨어에 의해 뒷받침된다.

### 2.1  DJI Dock 3: 전천후 자율 운영 기지

DJI Dock 3는 기체의 자동 이착륙, 충전, 그리고 상태 관리를 수행하는 핵심 기반 시설이다. 이전 세대인 DJI Dock 2와 비교하여 운영 환경의 한계를 크게 확장하고 배포 유연성을 극대화하는 방향으로 진화했다.

#### 2.1.1 내환경성 및 물리적 제원

Dock 3는 IP56 등급의 방진방수 성능을 갖추고 있어 이전 세대(IP55)보다 향상된 보호 기능을 제공한다.3 또한, 작동 온도 범위가 -30°C에서 50°C로 확장되어 혹한이나 혹서의 극한 환경에서도 안정적인 운영이 가능하다.1 이는 더 넓은 지리적 범위와 다양한 기후 조건에서의 배치를 가능하게 하여 자율 드론 시스템의 적용 가능성을 넓히는 중요한 요소이다. 최대 12 m/s의 풍속에서도 안전한 이착륙을 지원하여 기상 변화에 대한 대응력을 높였다.3

#### 2.1.2 충전 및 전력 시스템

Dock 3의 고속 충전 시스템은 기체 배터리를 15%에서 95%까지 단 27분 만에 충전할 수 있어 임무 간 대기 시간을 획기적으로 단축시킨다.3 이는 임무 연속성을 극대화하고 긴급 상황 발생 시 신속한 재출동을 가능하게 한다. 또한, 12 Ah 용량의 납산 백업 배터리가 내장되어 있어 갑작스러운 정전이 발생하더라도 4시간 이상 핵심 기능을 유지할 수 있다.6 이를 통해 외부 전력 공급이 불안정한 원격지에서도 시스템의 안정성을 확보할 수 있다.

#### 2.1.3 네트워크 및 센서

안정적인 클라우드 연결은 원격 자율 운영의 필수 조건이다. Dock 3는 10/100/1000Mbps 적응형 이더넷 포트를 기본으로 제공하며, 별매품인 DJI Cellular Dongle 2를 통해 4G 네트워크에 접속할 수 있는 옵션을 제공한다.4 이는 유선 네트워크 설치가 어려운 지역에서도 시스템을 배포할 수 있게 한다. 또한, 풍속, 강우, 주변 온도, 침수, 캐빈 내부 온도 및 습도 센서 등 다양한 환경 센서를 통합하여 원격지의 환경 상태를 실시간으로 모니터링한다.6 이 데이터는 클라우드 플랫폼으로 전송되어 비행 가능 여부를 판단하고 임무 수행의 안전성을 높이는 데 활용된다.

#### 2.1.4 배포 유연성

Dock 3의 가장 주목할 만한 발전 중 하나는 차량 탑재 배포(Mobile Deployment)를 공식적으로 지원한다는 점이다.1 이를 위해 진동 방지 설계가 적용되었으며, 클라우드 기반으로 도크의 위치를 보정하는 기능이 추가되었다.1 이는 시스템이 고정된 위치에서만 운영되던 한계를 넘어, 이동하는 플랫폼(예: 트럭, 선박)에서도 자율 임무를 수행할 수 있음을 의미한다. 재난 현장, 광범위한 건설 부지, 국경 순찰 등 동적인 시나리오에서 새로운 활용 가능성을 제시한다.

### 2.2  DJI Matrice 4D/4TD: 고성능 다중 센서 항공 플랫폼

DJI Matrice 4D(M4D) 및 Matrice 4TD(M4TD)는 Dock 3 시스템에 최적화된 고성능 드론으로, 장시간 비행 능력과 뛰어난 내구성, 그리고 강력한 다중 센서 페이로드를 특징으로 한다.

#### 2.2.1 비행 성능 및 내구성

M4D 시리즈는 최대 54분의 긴 비행 시간을 제공하여 이전 세대인 Matrice 3D 시리즈(50분)보다 넓은 영역을 한 번의 비행으로 커버할 수 있다.5 IP55 등급의 방수방진 성능과 결빙 방지 기능이 적용된 저소음 프로펠러는 비나 눈이 오는 악천후 속에서도 안정적인 비행을 보장한다.1 작동 온도 범위 또한 -20°C에서 50°C로 넓어 다양한 환경에서 임무 수행이 가능하다.13

#### 2.2.2 다중 센서 페이로드

M4D 시리즈는 임무 목적에 따라 두 가지 모델로 제공된다.

- **Matrice 4D (M4D):** 고정밀 매핑 및 측량, 자산 검사 임무에 최적화되어 있다. 4/3인치 CMOS 센서와 기계식 셔터를 탑재한 20MP 광각 카메라, 48MP 중망원 카메라, 그리고 48MP 망원 카메라로 구성된 트리플 카메라 시스템을 갖추고 있다.12 또한, 최대 1800m 거리까지 측정이 가능한 레이저 거리 측정기(Laser Rangefinder)를 통합하여 정밀한 거리 및 면적 측정이 가능하다.3
- **Matrice 4TD (M4TD):** M4D의 강력한 가시광선 카메라 구성에 더해, 640x512 해상도의 열화상 카메라를 추가로 탑재했다.3 이 열화상 카메라는 UHR(Ultra-High Resolution) 적외선 이미지 모드를 통해 1280x1024 해상도까지 지원하여 미세한 온도 차이를 식별할 수 있다.16 이를 통해 공공 안전, 수색 및 구조, 야간 감시, 태양광 패널 검사 등 다양한 특수 임무에 대응할 수 있다.

#### 2.2.3 정밀 측위 및 장애물 감지

기체에 통합된 RTK(Real-Time Kinematic) 모듈은 GNSS와 연동하여 수직 $2 \text{ cm} + 1 \text{ ppm (RMS)}$, 수평 $1 \text{ cm} + 1 \text{ ppm (RMS)}$ 수준의 센티미터급 위치 정확도를 제공한다.5 이는 정밀 매핑 및 반복적인 검사 임무의 정확성을 보장하는 핵심 기술이다. 또한, 기체 전방위에 배치된 쌍안 비전 센서와 하단의 3D 적외선 센서는 복잡하고 장애물이 많은 환경에서도 안전한 자율 비행을 가능하게 한다.16

### 2.3  이전 세대와의 비교 분석: 진화의 방향성

DJI Dock 3와 M4D 시리즈는 이전 세대인 Dock 2와 M3D 시리즈와 비교했을 때 단순한 성능 향상을 넘어, 시스템의 운용 개념(CONOPS) 자체를 혁신하는 방향으로 진화했다. Dock 2가 주로 고정된 장소에 설치되어 정적인 감시 및 순찰 임무를 수행하는 것을 전제로 설계되었다면, Dock 3는 '이동성'과 '전천후 운영 능력'을 극대화하여 동적이고 예측 불가능한 현장으로 자율 드론 시스템의 적용 범위를 확장하려는 명확한 전략적 의도를 보여준다.

예를 들어, Dock 3의 향상된 IP 등급(IP56)과 넓어진 작동 온도 범위(-30°C ~ 50°C)는 더 혹독한 기후 조건에서의 배치를 가능하게 한다.3 여기에 '차량 탑재 배포' 기능이 공식적으로 추가되면서 1, 시스템은 더 이상 고정된 자산이 아니라 필요에 따라 어디든 이동하여 즉시 임무를 수행할 수 있는 유연한 자산이 되었다. M4D의 향상된 비행 시간(54분)과 내구성(IP55)은 이러한 이동형 임무를 완벽하게 뒷받침한다.11 이 두 하드웨어의 진화 방향을 종합해 보면, 시스템 전체가 '고정된 지점'에서의 반복 임무를 넘어 '필요한 모든 지점'으로 이동하여 자율 임무를 수행할 수 있도록 설계되었음을 알 수 있다. 이는 단순한 기술적 진보가 아닌, 자율 드론 시스템의 활용 패러다임을 바꾸는 근본적인 변화를 의미한다.

다음 표는 DJI Dock 3 및 Matrice 4D 시리즈가 이전 세대 대비 주요하게 개선된 항목을 요약한 것이다.

| 항목                    | DJI Dock 2     | DJI Dock 3     | Matrice 3D/3TD  | Matrice 4D/4TD |
| ----------------------- | -------------- | -------------- | --------------- | -------------- |
| **총 중량 (기체 제외)** | 34 kg          | 55 kg          | 1410 g          | 1850 g         |
| **방진방수 등급**       | IP55           | IP56           | IP54            | IP55           |
| **작동 온도**           | -25° to 45°C   | -30° to 50°C   | -20° to 45°C    | -20° to 50°C   |
| **최대 허용 착륙 풍속** | 8 m/s          | 12 m/s         | 8 m/s (착륙 시) | 12 m/s         |
| **충전 시간**           | 32분 (20%→90%) | 27분 (15%→95%) | -               | -              |
| **최대 비행 시간**      | -              | -              | 50분            | 54분           |
| **차량 탑재 배포**      | 미지원         | 지원           | -               | -              |
| **비디오 전송 시스템**  | O3 Enterprise  | O4+ Enterprise | O3 Enterprise   | O4+ Enterprise |

## 3.  관제 소프트웨어 생태계: 개방성과 확장성

DJI Dock 3 시스템의 진정한 잠재력은 하드웨어의 성능을 최대한 활용하고 외부 시스템과의 유연한 통합을 가능하게 하는 소프트웨어 생태계에서 발현된다. DJI는 공식 플랫폼인 FlightHub 2를 통해 강력한 통합 관제 기능을 제공하는 동시에, 포괄적인 SDK(Software Development Kit)를 공개하여 개발자들이 자체적인 솔루션을 구축할 수 있는 개방형 생태계를 조성하고 있다.

### 3.1  공식 플랫폼: DJI FlightHub 2

DJI FlightHub 2는 다수의 드론과 도크를 원격에서 통합 관리하기 위한 클라우드 기반 관제 플랫폼이다.6 이 플랫폼은 Dock 3 및 M4D 시리즈와 긴밀하게 통합되어 다음과 같은 핵심 기능을 제공한다.

- **실시간 상황 인식:** 지도 위에 드론의 실시간 위치와 비행 경로, 그리고 라이브 비디오 스트림을 오버레이하여 현장 상황을 직관적으로 파악할 수 있다.2
- **원격 임무 관리:** 클라우드에서 직접 비행 경로를 계획하고, 임무를 예약하며, 원클릭으로 드론을 이륙시켜 자동 임무를 수행하게 할 수 있다.10
- **지능형 데이터 분석:** 실시간 객체 탐지 기능을 통해 영상 속 차량이나 사람의 수를 자동으로 계산하거나, 변화 탐지 기능을 통해 동일 지역의 이전 이미지와 현재 이미지를 비교하여 변화된 부분을 자동으로 식별하고 보고서를 생성하는 등 고도의 분석 기능을 제공한다.10
- **데이터 중앙 관리:** 비행 중 촬영된 사진과 영상은 자동으로 클라우드에 업로드되고, 임무별로 정리되어 손쉬운 데이터 관리 및 공유를 지원한다.2

### 3.2  개발자 생태계: DJI SDK의 역할

DJI는 단순한 완제품 제조사를 넘어, 개발자들이 자사의 하드웨어 플랫폼 위에서 혁신적인 솔루션을 만들 수 있도록 다양한 SDK를 제공한다.18 Dock 3 시스템과 관련된 주요 SDK는 다음과 같다.

- **Cloud API:** 본 보고서의 핵심 주제로, 서드파티 클라우드 플랫폼이 DJI Dock 및 드론과 직접 통신할 수 있도록 하는 표준 인터페이스이다. MQTT, HTTPS, WebSocket과 같은 웹 표준 프로토콜을 기반으로 설계되어 개발자들이 낮은 진입 장벽으로 자사의 서비스에 DJI 하드웨어를 통합할 수 있게 한다.18
- **Edge SDK:** Dock 3에 내장된 엣지 컴퓨팅 모듈에서 애플리케이션을 개발하기 위한 SDK이다.18 이를 통해 드론이 수집한 대용량 데이터를 클라우드로 전송하기 전에 현장에서 실시간으로 분석하고 처리할 수 있다. 예를 들어, 영상에서 특정 객체를 탐지하거나 이상 징후를 감지하여 결과값만 클라우드로 전송함으로써 네트워크 대역폭을 절약하고 응답 시간을 단축할 수 있다.
- **Payload SDK (PSDK):** 서드파티 개발사가 자체적으로 개발한 페이로드(예: 특수 목적 카메라, 환경 센서, 스피커, 탐조등 등)를 Matrice 4D/4TD 기체에 물리적으로 장착하고 소프트웨어적으로 통합하여 제어할 수 있게 해준다.18 이는 드론의 활용 범위를 무한히 확장할 수 있는 가능성을 열어준다.
- **Mobile SDK (MSDK):** 안드로이드나 iOS 기반의 모바일 애플리케이션을 개발하여 조종기를 통해 드론을 직접 제어하거나, 비행 데이터를 수신하고, 페이로드 기능을 활용하는 데 사용된다.18

### 3.3  서드파티 플랫폼 통합 구조

DJI의 소프트웨어 전략의 핵심은 Cloud API를 통해 '게이트웨이'(DJI Dock 또는 RC 조종기)와 '서드파티 클라우드' 간의 통신을 표준화하는 것이다. 이러한 접근 방식은 DJI가 하드웨어 플랫폼의 기술적 우위는 유지하면서, 각 산업 분야에 특화된 정교한 소프트웨어 솔루션 개발은 해당 분야의 전문성을 가진 서드파티 개발자들에게 맡기는 개방형 생태계 전략을 추구하고 있음을 명확히 보여준다.

이러한 전략이 성공적으로 작동하고 있음은 DroneDeploy 21나 FlytBase 8와 같은 유수의 기업들이 DJI Cloud API를 기반으로 자사의 상용 드론 자동화 솔루션을 성공적으로 출시하고 있다는 사실에서 확인할 수 있다. 이들 기업은 건설, 농업, 보안 등 특정 산업의 요구사항에 맞는 맞춤형 사용자 인터페이스, 데이터 분석 도구, 워크플로우 통합 기능을 제공한다. DJI가 모든 산업의 세분화된 요구를 직접 충족시키려 하기보다, 개발자들이 자유롭게 솔루션을 창조할 수 있는 '기술적 토대'를 제공하는 데 집중하고 있는 것이다. 이 개방형 전략의 중심에는 바로 표준 프로토콜인 MQTT가 자리 잡고 있으며, 이는 DJI가 하드웨어 제조사를 넘어 플랫폼 사업자로 진화하려는 의도를 명확히 드러내는 기술적 선택이라 할 수 있다.

## 4.  DJI Cloud API와 MQTT 프로토콜 아키텍처

DJI Dock 3 시스템의 원격 관제 및 자동화 기능의 근간을 이루는 것은 DJI Cloud API이며, 그 중에서도 실시간 양방향 통신을 담당하는 MQTT 프로토콜이 핵심적인 역할을 수행한다. 이 장에서는 MQTT가 DJI의 자율 운영 시스템 아키텍처 내에서 어떻게 구조화되고 작동하는지 기술적으로 심층 분석한다.

### 4.1  디바이스-게이트웨이-클라우드 3계층 구조

DJI Cloud API는 현대 사물 인터넷(IoT) 시스템에서 널리 사용되는 3계층 아키텍처를 채택하고 있다.25 이 구조는 각 계층의 역할을 명확히 분리하여 시스템의 모듈성, 확장성, 그리고 보안성을 높인다.

- **디바이스 계층 (Device Layer):** 데이터 수집 및 물리적 동작을 수행하는 최종 단말기로, Matrice 4D/4TD 드론과 그에 탑재된 카메라, 센서 등의 페이로드가 이 계층에 해당한다.
- **게이트웨이 계층 (Gateway Layer):** 디바이스와 클라우드 사이의 통신을 중개하는 핵심 요소이다. DJI Dock 3 또는 DJI RC Plus 조종기가 게이트웨이 역할을 수행한다. 이 계층은 드론과의 근거리 통신(DJI의 사설 프로토콜인 O4+ Enterprise AirLink 사용)과 외부 클라우드와의 원거리 통신(MQTT, HTTPS 등 표준 프로토콜 사용)을 변환하고 연결하는 책임을 진다.
- **클라우드 계층 (Cloud Layer):** 서드파티 개발자가 구축하고 운영하는 애플리케이션 서버와 데이터베이스, 분석 엔진 등이 위치하는 곳이다. 이 계층에서 모든 드론과 도크의 상태를 모니터링하고, 임무를 계획하며, 수집된 데이터를 처리하고 사용자에게 서비스를 제공한다.

이러한 3계층 아키텍처는 드론을 인터넷에 직접 노출시키지 않고 신뢰할 수 있는 게이트웨이를 통해 연결함으로써 보안을 강화하는 중요한 이점을 제공한다. 또한, 복잡한 하드웨어 제어 로직(예: 비행 안정화, 모터 제어)을 펌웨어와 게이트웨이 수준에서 추상화하여 클라우드 개발자가 표준 MQTT 메시지만으로 드론의 고수준 기능을 제어할 수 있게 함으로써, 개발의 복잡성을 크게 낮추고 개발 속도를 향상시킨다.20

### 4.2  통신 프로토콜 스택: MQTT, HTTPS, WebSocket의 역할 분담

DJI Cloud API는 단일 프로토콜에 의존하지 않고, 각 통신 작업의 특성에 가장 적합한 프로토콜을 선택하여 사용하는 하이브리드 방식을 채택하고 있다.

- **MQTT 5.0:** 실시간성이 요구되는 대부분의 통신에 사용되는 주 프로토콜이다. 드론의 텔레메트리 데이터(위치, 속도, 고도 등) 전송, 시스템 상태 업데이트, 실시간 비행 제어 명령(DRC), 라이브 스트리밍 세션 제어 등 저지연 양방향 통신이 필수적인 기능들은 모두 MQTT를 통해 이루어진다.25
- **HTTPS:** 비동기적이며 대용량 데이터 전송이나 보안이 중요한 요청-응답 상호작용에 사용된다. 예를 들어, 웨이포인트 임무 파일(.wpml)을 클라우드에서 다운로드하거나, 촬영된 고해상도 미디어 파일을 클라우드 스토리지에 업로드하기 위한 임시 자격 증명을 획득하는 등의 작업에 HTTPS API가 활용된다.19
- **WebSocket:** 지도 위에 여러 드론이나 객체의 위치를 표시하는 상황 인식(Situational Awareness) 기능과 같이, 서버에서 웹 브라우저와 같은 클라이언트로 비동기적인 데이터를 지속적으로 푸시해야 할 때 사용될 수 있다.20

### 4.3  MQTT 발행/구독(Publish/Subscribe) 모델 분석

MQTT는 '발행/구독'이라는 독특한 메시징 모델을 사용한다. 이 모델에서는 메시지를 보내는 클라이언트(Publisher)와 메시지를 받는 클라이언트(Subscriber)가 서로를 직접 알지 못한다. 모든 통신은 '브로커(Broker)'라고 불리는 중앙 서버를 통해 이루어진다.26

1. **연결 (Connect):** 모든 클라이언트(DJI Dock, 클라우드 애플리케이션)는 통신을 시작하기 위해 MQTT 브로커에 연결한다.
2. **구독 (Subscribe):** 메시지를 수신하고자 하는 클라이언트는 특정 '토픽(Topic)'을 브로커에 구독 신청한다. 토픽은 메시지를 분류하는 주소와 같은 역할을 한다 (예: `thing/product/dock_sn_123/state`).
3. **발행 (Publish):** 메시지를 전송하고자 하는 클라이언트는 특정 토픽에 메시지를 발행한다.
4. **중개 (Forward):** 브로커는 발행된 메시지를 해당 토픽을 구독하고 있는 모든 클라이언트에게 전달한다.

이 모델은 송신자와 수신자를 완전히 분리(decoupling)시키기 때문에 시스템의 유연성과 확장성을 극대화한다. 예를 들어, 하나의 드론이 발행하는 텔레메트리 데이터를 지도 표시용 클라이언트, 데이터 로깅용 클라이언트, 이상 감지용 클라이언트 등 다수의 수신자가 동시에 받아 처리할 수 있다.29

### 4.4  서비스 품질(QoS) 레벨 및 보안 프로토콜

MQTT는 불안정한 네트워크 환경에서도 메시지 전달의 신뢰성을 보장하기 위해 3단계의 서비스 품질(Quality of Service, QoS) 레벨을 제공한다. DJI Cloud API 문서에서도 이 세 가지 레벨을 명시적으로 언급하며, 개발자가 통신의 중요도에 따라 신뢰성과 시스템 부하 간의 균형을 맞출 수 있도록 지원한다.26

- **QoS 0 (At most once):** "최대 한 번 전송". 메시지 전달을 보장하지 않는 'fire and forget' 방식이다. 전송 속도가 가장 빠르고 오버헤드가 적어, 일부 유실되어도 큰 문제가 없는 고빈도의 텔레메트리 데이터 전송에 적합하다.31
- **QoS 1 (At least once):** "최소 한 번 전송". 메시지가 수신자에게 최소 한 번은 전달되는 것을 보장한다. 네트워크 문제로 인해 중복 전송될 가능성이 있다. 드론의 상태 변경 알림이나 일반적인 제어 명령에 사용될 수 있다.31
- **QoS 2 (Exactly once):** "정확히 한 번 전송". 4단계 핸드셰이크 과정을 통해 메시지가 유실되거나 중복되지 않고 정확히 한 번만 전달되는 것을 보장한다. 가장 신뢰성이 높지만 전송 속도가 가장 느리고 오버헤드가 크다. 비행 임무 시작/종료, 원격 잠금 해제와 같이 절대적으로 신뢰성이 요구되는 중요한 명령에 필수적이다.31

보안 측면에서, 민감한 기업 데이터를 다루는 엔터프라이즈 환경에서는 통신 암호화가 필수적이다. DJI Cloud API는 표준 보안 프로토콜인 SSL/TLS를 통해 MQTT 통신 채널 전체를 암호화하는 것을 지원한다. DJI Pilot 2와 Dock은 Godaddy에서 발급한 인증서를 지원하며, 이를 통해 서드파티 클라우드와 상호 인증 기반의 안전한 MQTT SSL 연결을 구축할 수 있다.33

## 5.  MQTT 토픽 구조 및 데이터 페이로드 심층 분석

MQTT 기반 시스템을 연동하고 개발하기 위해서는 실제 데이터가 어떤 형식과 구조로 교환되는지 이해하는 것이 필수적이다. DJI Cloud API는 체계적인 토픽 구조와 표준화된 JSON 페이로드 형식을 사용하여 다양한 기능과 데이터를 효율적으로 관리한다.

### 5.1  토픽 네이밍 규칙 및 계층 구조

DJI Cloud API는 슬래시(`/`)로 구분되는 계층적 토픽 구조를 사용하여 메시지를 체계적으로 분류한다. 일반적인 토픽 형식은 `thing/product/{device_sn}/{function}`의 구조를 따른다.35 여기서 `{device_sn}`은 통신을 중개하는 게이트웨이(Dock 또는 RC) 또는 데이터를 발생시키는 디바이스(드론)의 고유 시리얼 번호로 대체된다.

주요 기능(`{function}`)에 따른 토픽은 다음과 같이 명확하게 구분되어 있다.35

- **`osd` (On-Screen Display):** 기체의 위치, 속도, 고도, 자세 등 고빈도(기본 0.5Hz)로 주기적으로 보고되는 실시간 텔레메트리 데이터를 위한 토픽이다.
- **`state` (State):** 디바이스의 상태가 변경될 때만 비주기적으로 보고되는 데이터를 위한 토픽이다. 펌웨어 버전, 네트워크 연결 상태, 도크 커버의 개폐 상태 등이 여기에 해당한다.
- **`services` (Services):** 클라우드 플랫폼이 디바이스(드론 또는 도크)에 특정 동작을 수행하도록 명령을 보낼 때 사용하는 토픽이다. 디바이스는 이 명령에 대한 응답을 `services_reply` 토픽으로 보낸다.
- **`events` (Events):** 디바이스에서 비정상적인 상황이나 중요한 사건이 발생했을 때(예: HMS 알람, 임무 단계 변경, 미디어 업로드 완료) 이를 클라우드에 비동기적으로 알리기 위한 토픽이다.
- **`property/set` (Property Set):** 클라우드에서 디바이스의 특정 속성(읽기/쓰기가 가능한)을 설정할 때 사용하는 토픽이다.

`osd`와 `state` 토픽을 분리한 것은 네트워크 대역폭과 클라우드 서버의 처리 부하를 최적화하기 위한 매우 정교한 설계 결정이다. 모든 데이터를 고빈도로 전송하는 대신, 실시간성이 절대적으로 중요한 핵심 비행 데이터(`osd`)와 상태 변화의 감지가 중요한 데이터(`state`)를 분리함으로써 시스템은 효율성과 실시간성을 동시에 확보한다. 예를 들어, 수백 대의 드론으로 구성된 대규모 플릿을 운영하는 시나리오를 가정해 보자. 모든 드론이 자신의 모든 상태 정보를 0.5Hz로 전송한다면 MQTT 브로커와 백엔드 시스템에 엄청난 부하가 발생할 것이다. 하지만 이 설계 덕분에, 관제 시스템은 실시간 위치 추적을 위해 `osd` 토픽만 구독하고, 배터리 부족이나 기기 오류와 같은 중요한 상태 변화 알림은 `state` 또는 `events` 토픽을 통해 효율적으로 수신할 수 있다. 이는 대규모 드론 플릿 운영의 확장성 문제를 해결하는 데 결정적인 역할을 하는 아키텍처적 지혜라 할 수 있다.

### 5.2  OSD 텔레메트리 데이터 페이로드 분석

`thing/product/{device_sn}/osd` 토픽을 통해 전송되는 데이터 페이로드는 JSON 형식이며, 기체의 핵심 비행 상태 정보를 담고 있다. 주요 필드는 다음과 같다.36

- `longitude`, `latitude`, `height`: 기체의 경도, 위도, 고도 (WGS84 타원체고).
- `att_head`, `att_pitch`, `att_roll`: 기체의 자세 정보 (Heading, Pitch, Roll).
- `horizontal_speed`, `vertical_speed`: 수평 및 수직 속도.
- `battery`: 배터리 잔량(%), 전압 등 배터리 상태 정보.
- `position_state`: GPS 위성 수, RTK 상태 등 위치 정보의 품질.
- `wind_speed`, `wind_direction`: 풍속 및 풍향.
- 페이로드 정보: 짐벌 각도(`gimbal_pitch`, `gimbal_yaw`), 레이저 거리 측정기 데이터(`measure_target_distance`) 등 탑재된 페이로드의 실시간 데이터.

### 5.3  상태(State) 및 이벤트(Events) 메시지 페이로드 분석

`state`와 `events` 토픽은 비주기적으로 발생하지만 운영에 중요한 정보를 전달한다.

- **`state` 페이로드:** 펌웨어 버전(`firmware_version`), 네트워크 상태(`network_state`), 도크 커버 상태(`cover_state`), 도크 내 드론 유무(`drone_in_dock`), 미디어 파일 업로드 대기 수량(`media_file_detail`) 등 시스템의 전반적인 상태 정보를 포함한다.36
- **`events` 페이로드:** 비행 임무의 현재 진행 상태(예: `fly_to_point_progress`, `status`: `wayline_progress`), HMS(Health Management System) 알람, 조종기 제어권 상실 알림(`joystick_invalid_notify`) 등 특정 사건의 발생을 알리는 정보를 담고 있다.38

### 5.4  서비스(Services) 요청 및 응답 페이로드 분석

클라우드가 디바이스에 특정 동작을 요청할 때 `thing/product/{gateway_sn}/services` 토픽을 사용한다. 이 요청 메시지의 페이로드는 표준화된 JSON 구조를 가진다.39

- `tid` (Transaction ID): 각 요청과 응답을 짝짓기 위한 고유 식별자.
- `bid` (Business ID): 웨이포인트 임무와 같이 여러 단계로 이루어진 비즈니스 로직을 식별하기 위한 ID.
- `timestamp`: 메시지 생성 시간.
- `method`: `live_start_push`, `takeoff_to_point`, `wayline_start` 등 수행할 구체적인 명령을 지정하는 문자열.
- `data`: 해당 명령을 수행하는 데 필요한 파라미터들을 담고 있는 JSON 객체. 예를 들어, `takeoff_to_point` 명령의 `data` 객체에는 목표 지점의 위도, 경도, 고도 정보가 포함된다.

디바이스는 이 요청을 처리한 후, `thing/product/{gateway_sn}/services_reply` 토픽을 통해 응답 메시지를 발행한다. 이 응답 페이로드에는 원본 요청의 `tid`와 함께 처리 결과(성공/실패 코드)가 포함되어, 클라우드가 비동기적으로 명령 수행 결과를 확인할 수 있게 한다.

다음 표는 주요 MQTT 토픽의 목적과 데이터, 그리고 예상되는 QoS 레벨을 요약한 것이다.

| 토픽 방향               | 토픽 이름 형식                      | 목적                   | 주요 `method` 또는 데이터                             | QoS 추정   |
| ----------------------- | ----------------------------------- | ---------------------- | ----------------------------------------------------- | ---------- |
| **디바이스 → 클라우드** | `thing/product/{sn}/osd`            | 고빈도 텔레메트리 전송 | 위치, 속도, 자세, 배터리 등                           | QoS 0      |
| **디바이스 → 클라우드** | `thing/product/{sn}/state`          | 상태 변경 시 정보 보고 | 펌웨어 버전, 네트워크 상태, 도크 상태                 | QoS 1      |
| **디바이스 → 클라우드** | `thing/product/{sn}/events`         | 비동기 이벤트 알림     | HMS 알람, 임무 진행 상태, 미디어 업로드 완료          | QoS 1 or 2 |
| **클라우드 → 디바이스** | `thing/product/{sn}/services`       | 명령 하달              | `takeoff_to_point`, `fly_to_point`, `live_start_push` | QoS 1 or 2 |
| **디바이스 → 클라우드** | `thing/product/{sn}/services_reply` | 명령에 대한 응답       | 성공/실패 결과 코드, 요청 데이터                      | QoS 1 or 2 |
| **클라우드 → 디바이스** | `thing/product/{sn}/property/set`   | 디바이스 속성 설정     | `commander_flight_height` 등 쓰기 가능한 속성         | QoS 1 or 2 |

## 6.  MQTT를 통한 핵심 기능 구현 메커니즘

앞서 분석한 MQTT 토픽과 페이로드는 실제 드론 관제 기능에서 유기적으로 결합되어 사용된다. 이 장에서는 실시간 비행 제어, 임무 관리, 영상 스트리밍과 같은 핵심 기능들이 MQTT를 통해 어떻게 구현되는지 구체적인 시나리오를 통해 그 메커니즘을 설명한다.

### 6.1  실시간 비행 제어(Live Flight Controls, DRC)

DRC(Drone Remote Control)는 자동 임무 수행 중 긴급 상황이 발생하거나 정밀한 수동 조작이 필요할 때, 원격지에서 MQTT를 통해 저지연으로 드론을 직접 제어하는 기능이다.40

1. **제어권 요청 및 획득:** 클라우드 플랫폼은 `services` 토픽을 통해 드론 제어권 획득을 요청한다.
2. **명령 전송:** 제어권을 획득한 후, 클라우드는 `drc` 관련 토픽을 통해 실시간 제어 명령을 전송한다. 예를 들어, `fly_to_point` 명령을 통해 특정 좌표로 이동시키거나, 짐벌 제어 명령으로 카메라 시점을 바꾸고, `camera_zoom` 명령으로 확대/축소할 수 있다.38
3. **상태 피드백:** 드론은 명령을 수행하면서 자신의 상태 변화(위치, 속도 등)를 `osd` 토픽을 통해 실시간으로 클라우드에 보고한다. 또한, `fly_to_point`와 같은 특정 임무의 진행 상태는 `events` 토픽의 `fly_to_point_progress` 메시지를 통해 상세히 보고된다.38

이 DRC 기능은 MQTT의 저지연 특성을 최대로 활용한 대표적인 예시이다. 특히, DJI는 일반적인 MQTT 통신 채널 외에 DRC 통신을 위해 별도의 EMQX 브로커를 할당하는 아키텍처를 채택하여 전송 속도와 응답성을 극대화했다.40 이를 통해 원격지의 관제사는 마치 현장에서 조종기를 직접 조작하는 것과 유사한 수준의 정밀하고 즉각적인 제어 경험을 얻을 수 있다.

### 6.2  임무 및 웨이포인트(Wayline) 관리

웨이포인트 임무는 정해진 경로를 따라 비행하며 데이터를 수집하는 자동화 임무의 핵심이다. 이 과정은 대용량 파일 전송을 위한 HTTPS와 실시간 제어를 위한 MQTT가 결합된 하이브리드 방식으로 구현된다.

1. **임무 파일 생성 및 업로드:** 관제사는 웹 인터페이스에서 비행 경로, 고도, 카메라 동작 등을 설정하여 웨이포인트 파일(.wpml)을 생성한다. 이 파일은 HTTPS API를 통해 클라우드 서버에 업로드된다.28
2. **임무 실행 명령:** 임무를 시작할 때, 클라우드는 MQTT의 `services` 토픽과 `wayline_start` `method`를 사용하여 드론에 임무 실행을 명령한다.28 이 메시지에는 실행할 임무 파일의 ID와 같은 메타데이터가 포함된다.
3. **실시간 진행 상황 보고:** 드론은 임무를 수행하면서 현재 몇 번째 웨이포인트를 통과하고 있는지, 남은 비행 시간과 거리는 얼마인지 등의 진행 상황을 `events` 토픽을 통해 실시간으로 클라우드에 보고한다.28 이를 통해 관제사는 임무가 계획대로 진행되고 있는지 원격으로 모니터링할 수 있다. 임무 일시 중지, 재개, 취소 등의 제어 역시 MQTT를 통해 이루어진다.

### 6.3  실시간 영상 스트리밍(Live Streaming) 제어

원격 관제에서 실시간 영상은 현장 상황을 파악하는 가장 중요한 수단이다. DJI 시스템은 영상 데이터 자체는 표준 스트리밍 프로토콜을 사용하고, 스트리밍 세션의 제어는 MQTT를 사용하는 효율적인 방식을 채택했다.

1. **스트리밍 시작 명령:** 클라우드는 `services` 토픽에 `live_start_push` `method`를 포함한 메시지를 발행하여 스트리밍 시작을 명령한다. 이 메시지의 `data` 필드에는 영상 데이터를 수신할 서버의 주소(예: RTMP URL)와 스트리밍 프로토콜 타입(RTMP, RTSP, WebRTC 등) 정보가 포함된다.42
2. **영상 데이터 전송:** 명령을 수신한 게이트웨이(Dock 또는 RC)는 드론으로부터 받은 영상 데이터를 지정된 프로토콜로 인코딩하여 해당 스트리밍 서버로 전송하기 시작한다. 실제 대용량 영상 데이터는 MQTT 채널이 아닌 별도의 스트리밍 프로토콜 채널(예: RTMP over TCP)을 통해 전송된다.44
3. **스트리밍 제어:** 스트리밍이 진행되는 동안, 클라우드는 MQTT를 통해 추가적인 제어 명령을 보낼 수 있다. 예를 들어, `live_set_quality` `method`로 영상 화질을 변경하거나, `live_lens_change` `method`로 광각, 망원, 열화상 카메라 간의 전환을 명령할 수 있다. 스트리밍을 종료할 때는 `live_stop_push` 명령을 사용한다.42

이처럼 실제 데이터가 흐르는 '데이터 평면(Data Plane)'과 제어 신호가 오가는 '제어 평면(Control Plane)'을 분리하는 것은 대규모 분산 시스템에서 널리 사용되는 매우 효율적인 아키텍처 설계 패턴이다. MQTT는 가볍고 신뢰성 있는 제어 신호를 신속하게 전달하는 데 최적화되어 있고, RTMP와 같은 프로토콜은 대용량 미디어 데이터를 지속적으로 전송하는 데 특화되어 있다. DJI는 각 프로토콜의 장점을 최적으로 활용하여 안정적이면서도 유연한 라이브 스트리밍 기능을 구현했다.

### 6.4  미디어 파일 업로드 및 관리

임무 중 촬영된 고해상도 사진이나 영상 파일은 크기가 크기 때문에 MQTT로 직접 전송하는 것은 비효율적이다. 따라서 이 과정 역시 MQTT와 HTTPS가 협력하여 수행된다.

1. **업로드 요청 및 자격 증명 획득:** 클라우드는 MQTT `services` 토픽을 통해 특정 임무(`flight_id`)에서 촬영된 미디어 파일의 우선 업로드를 명령할 수 있다 (`upload_flighttask_media_prioritize`).46 파일을 실제로 업로드하기 전에, 디바이스(Dock)는 MQTT 

   `requests` 토픽을 통해 클라우드 스토리지(Amazon S3, MinIO 등)에 접근하기 위한 임시 자격 증명(Access Key, Secret Key, Session Token)을 클라우드에 요청한다.46

2. **파일 업로드:** 클라우드는 요청에 대한 응답으로 임시 자격 증명을 `requests_reply` 토픽을 통해 디바이스에 전달한다. 디바이스는 이 자격 증명을 사용하여 HTTPS(S3 API)를 통해 미디어 파일을 클라우드 스토리지에 직접 업로드한다.

3. **업로드 완료 보고:** 파일 업로드가 성공적으로 완료되면, 디바이스는 `events` 토픽을 통해 업로드 완료 사실과 해당 파일의 메타데이터(촬영 시간, 위치, 짐벌 각도 등)를 클라우드에 보고한다.46 이를 통해 클라우드 애플리케이션은 파일이 성공적으로 업로드되었음을 인지하고 후속 처리(예: 데이터베이스에 메타데이터 저장, 3D 모델링 작업 시작)를 진행할 수 있다.

## 7.  종합 고찰 및 기술적 함의

지금까지 DJI Dock 3 및 Matrice 4D 시스템의 하드웨어 사양부터 소프트웨어 생태계, 그리고 MQTT 기반 통신 아키텍처의 구체적인 구현 방식까지 심층적으로 분석했다. 이 장에서는 앞선 분석 내용을 종합하여 MQTT 기반 아키텍처가 갖는 기술적 의미와 한계, 그리고 서드파티 개발자 관점에서의 과제와 향후 발전 가능성을 논한다.

### 7.1  MQTT 기반 아키텍처의 장단점 평가

DJI가 자사의 주력 엔터프라이즈 솔루션에 MQTT를 핵심 통신 프로토콜로 채택한 것은 여러 가지 중요한 장점을 제공한다.

#### 7.1.1 장점

- **높은 확장성 (High Scalability):** MQTT의 발행/구독 모델과 경량 메시지 구조는 중앙 브로커를 통해 수만, 수십만 개의 디바이스로 연결을 확장하는 데 매우 용이하다. 이는 단일 조직이 수백, 수천 개의 드론 도크를 전국 또는 전 세계에 배포하여 운영하는 대규모 플릿 관리 시나리오에 필수적인 특성이다.47
- **실시간 양방향 통신 (Real-time Bidirectional Communication):** TCP 기반의 영구적인 연결을 유지하므로 서버가 클라이언트에 데이터를 즉시 푸시(push)할 수 있다. 이는 HTTP의 요청/응답 모델이 갖는 폴링(polling) 방식의 지연 문제를 해결하며, 원격 제어(DRC) 및 실시간 상태 모니터링에 매우 효과적이다.50
- **네트워크 효율성 (Network Efficiency):** 최소 2바이트의 작은 헤더 크기를 가지는 경량 프로토콜로, 동일한 데이터를 전송할 때 HTTP에 비해 훨씬 적은 네트워크 대역폭을 소모한다.52 이는 대역폭이 제한적이거나 비용이 비싼 셀룰러 네트워크 환경에서 운영되는 Dock 3 시스템에 큰 이점을 제공한다.53
- **개방성과 상호운용성 (Openness and Interoperability):** MQTT는 널리 사용되는 개방형 표준 프로토콜이다. 이를 채택함으로써 DJI는 개발자들이 특정 벤더의 독점 기술에 종속되지 않고, 기존에 사용하던 다양한 프로그래밍 언어와 플랫폼을 사용하여 DJI 시스템과 쉽게 통합할 수 있는 길을 열었다.19

#### 7.1.2 단점

- **브로커 의존성 (Broker Dependency):** 모든 통신이 중앙 MQTT 브로커를 거치므로, 브로커의 성능과 안정성이 전체 시스템의 단일 장애점(Single Point of Failure) 및 성능 병목 지점이 될 수 있다. 따라서 실제 운영 환경에서는 브로커의 고가용성(High Availability)을 보장하기 위한 클러스터링 구성이 필수적이다.47
- **보안 구현의 복잡성 (Security Implementation Complexity):** MQTT 프로토콜 자체는 인증이나 암호화 기능을 포함하지 않는다. 엔터프라이즈급 보안을 확보하기 위해서는 전송 계층에서 TLS를 적용하고, 클라이언트 인증서를 관리하며, 토픽별 접근 제어를 위한 ACL(Access Control List)을 정교하게 설정하는 등 추가적인 보안 계층의 구현과 관리가 필요하다.54
- **대용량 파일 전송의 비효율성 (Inefficiency in Large File Transfer):** MQTT는 작은 제어 메시지와 데이터를 빈번하게 교환하는 데 최적화되어 있다. 수십 메가바이트(MB)에서 기가바이트(GB)에 이르는 고해상도 미디어 파일을 전송하는 데는 적합하지 않다. 이 때문에 DJI 아키텍처는 미디어 파일 전송을 위해 HTTPS를 병행하는 복합적인 구조를 채택했다.51

### 7.2  서드파티 개발자 관점에서의 기술적 과제 및 고려사항

DJI Cloud API를 사용하여 자체적인 드론 관제 솔루션을 구축하려는 개발자는 다음과 같은 기술적 과제들을 고려해야 한다.

- **MQTT 브로커 선택 및 운영:** 솔루션의 규모와 요구사항에 따라 적합한 MQTT 브로커를 선택해야 한다. 소규모 테스트나 개발 단계에서는 Mosquitto와 같은 오픈소스 브로커를 자체 서버에 구축(Self-hosted)할 수 있다. 반면, 대규모 상용 서비스를 위해서는 EMQX나 HiveMQ와 같은 고성능 브로커를 클러스터로 구성하거나, AWS IoT Core, Azure IoT Hub, DroneMQTT와 같은 완전 관리형 클라우드 서비스(Managed Service)를 사용하는 것이 운영 부담을 줄일 수 있다.48
- **실시간 데이터 처리 파이프라인 구축:** `osd` 토픽을 통해 수많은 드론으로부터 쏟아지는 대량의 시계열 텔레메트리 데이터를 실시간으로 처리하고 저장하기 위한 견고한 데이터 파이프라인 설계가 필수적이다. 일반적으로 MQTT 브로커에서 수신한 데이터를 Apache Kafka와 같은 메시지 큐로 전달하고, 이를 InfluxDB나 TimescaleDB와 같은 시계열 데이터베이스에 저장하여 분석 및 시각화에 활용하는 아키텍처가 널리 사용된다.48
- **견고한 상태 관리:** 드론과 도크의 상태는 비동기적인 MQTT 메시지를 통해 전달된다. 따라서 클라우드 애플리케이션은 각 디바이스의 최신 상태(예: 비행 중, 충전 중, 오류 발생 등)를 정확하게 추적하고 관리하기 위한 정교한 상태 머신(State Machine)을 내부적으로 구현해야 한다. 또한, 네트워크 단절 후 재연결 시 상태를 동기화하는 로직도 반드시 고려해야 한다.39

### 7.3  향후 발전 방향 및 권고 사항

DJI Dock 3 시스템의 MQTT 기반 아키텍처는 향후 더욱 고도화된 자율 운영 애플리케이션으로 발전할 수 있는 강력한 기반을 제공한다.

- **엣지 AI 통합 (Edge AI Integration):** DJI Edge SDK 18와 MQTT를 결합하는 것은 가장 유망한 발전 방향 중 하나이다. 드론이 촬영하는 고화질 영상을 모두 클라우드로 스트리밍하는 대신, Dock 3의 엣지 컴퓨팅 모듈에서 AI 모델을 실행하여 실시간으로 영상을 분석할 수 있다. 그리고 그 분석 결과(예: '균열 발견', '미승인 차량 진입')만을 작고 의미 있는 MQTT 

  `events` 메시지로 클라우드에 전송하는 것이다. 이 방식은 클라우드로 전송되는 데이터 양을 획기적으로 줄여 통신 비용을 절감하고, 현장에서 즉각적인 의사결정을 가능하게 한다.60

- **디지털 트윈 구현 (Digital Twin Implementation):** MQTT를 통해 실시간으로 수신되는 `osd` 및 `state` 데이터를 활용하여, 클라우드 상에 실제 드론과 도크의 상태 및 위치를 완벽하게 동기화하는 가상의 복제본, 즉 디지털 트윈을 구축할 수 있다. 이 디지털 트윈은 과거 비행 데이터와 결합하여 임무 시뮬레이션, 고장 예측을 통한 예지 정비(Predictive Maintenance), 원격 진단 등 기존의 단순 관제를 넘어서는 고부가가치 서비스를 창출하는 기반이 될 것이다.

개발자에게는 DJI가 제공하는 Cloud API Demo 소스 코드 61를 면밀히 분석하여 MQTT 토픽 구독 및 메시지 처리 로직을 완전히 이해하고, 자체 브로커 환경을 구축하여 소규모 테스트를 우선적으로 수행할 것을 권고한다. 또한, 실제 운영 환경을 구축할 때는 애플리케이션의 요구사항에 맞춰 각 통신(텔레메트리, 제어 명령, 이벤트 알림)에 대한 최적의 QoS 레벨을 신중하게 선택하고, TLS 암호화 및 강력한 클라이언트 인증(예: 인증서 기반 인증)을 반드시 적용하여 시스템의 보안을 최고 수준으로 강화해야 한다.

마지막으로, 다음 표는 드론 관제 시스템 구축 시 MQTT와 HTTP/REST 프로토콜의 특성을 비교하고, DJI 시스템이 이 두 프로토콜을 어떻게 효과적으로 조합하여 사용하고 있는지를 분석한 것이다.

| 특성               | MQTT                                    | HTTP/REST                                        | DJI 시스템에서의 적용                                        |
| ------------------ | --------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| **통신 모델**      | 발행/구독 (Publish/Subscribe)           | 요청/응답 (Request/Response)                     | **MQTT:** 실시간 텔레메트리, 상태 보고, 양방향 제어. **HTTP:** 파일 관리, 인증 등 비동기 작업. |
| **연결성**         | 영구적 연결 (Persistent Connection)     | 비연결성 (Connectionless)                        | **MQTT:** 한 번의 연결로 지속적인 데이터 교환, 오버헤드 적음. **HTTP:** 매 요청마다 연결 생성/해제 오버헤드 발생. |
| **데이터 흐름**    | 양방향 (Bi-directional), 서버 푸시 가능 | 단방향 (클라이언트 → 서버 요청)                  | **MQTT:** 서버(클라우드)가 드론에 즉시 명령 푸시 가능. **HTTP:** 드론이 주기적으로 폴링(polling)해야 함. |
| **메시지 크기**    | 경량 헤더 (최소 2바이트)                | 상대적으로 무거운 헤더                           | **MQTT:** 저대역폭 셀룰러망에서 효율적. **HTTP:** 대용량 페이로드(파일) 전송에 적합. |
| **신뢰성 보장**    | QoS 0, 1, 2 레벨 제공                   | TCP 레벨에서 보장, 애플리케이션 레벨 재시도 필요 | **MQTT:** 중요한 명령(예: 임무 시작)은 QoS 2로 신뢰성 확보.  |
| **주요 사용 사례** | 실시간 제어, 모니터링, 알림             | 자원(Resource) 조회 및 관리                      | **MQTT:** `DRC`, `osd`, `events`. **HTTP:** 웨이포인트 파일, 미디어 파일 관리. |

#### 참고 자료

1. DJI Dock 3 Delivers “Drone in a Box” Enterprise Solution For 24/7 Remote Operations, accessed August 21, 2025, https://enterprise.dji.com/news/detail/dock-3-release
2. DJI Dock 3 - Stand-alone docking station for drones - Drone Parts Center, accessed August 21, 2025, https://drone-parts-center.com/en/product/dji-dock-3-the-stand-alone-docking-station-for-professional-drones/
3. DJI Dock 3 for Matrice 4D and 4TD - Advexure, accessed August 21, 2025, https://advexure.com/products/dji-dock-3-for-matrice-4d-and-4td
4. DJI Dock 3 - FAQ, accessed August 21, 2025, https://enterprise.dji.com/dock-3/faq
5. Support for DJI Dock 3, accessed August 21, 2025, https://www.dji.com/support/product/dock-3
6. DJI Dock 3 | DJI Store Sofia, accessed August 21, 2025, https://store.dji.bg/en/dji-dock-3.html
7. Buy DJI Dock 3 from COPTERS.GR, accessed August 21, 2025, https://www.copters.gr/en/dji-dock-3.html
8. DJI Dock 3 + FlytBase | Mobile Drone Operations for Fast, Secure Missions, accessed August 21, 2025, https://www.flytbase.com/dji-dock-3
9. DJI Matrice 3D Drone For DJI Dock 2 - DJI Retail, accessed August 21, 2025, https://dji-retail.co.uk/products/dji-matrice-3d-drone-for-dji-dock-2
10. DJI Dock 3: Everything You Need to Know - Heliguy, accessed August 21, 2025, https://www.heliguy.com/blogs/posts/dji-dock-3-everything-you-need-to-know/
11. DJI Dock Comparison, accessed August 21, 2025, https://www.dji.com/products/comparison-dock
12. DJI Matrice 4D for Dock 3 w/ DJI Care Enterprise Plus | Advexure, accessed August 21, 2025, https://advexure.com/products/dji-matrice-4d-for-dock-3-w-dji-care-enterprise
13. DJI Matrice 4D Drone Standalone Combo (with RC, Battery and Charger), accessed August 21, 2025, https://dji-retail.co.uk/products/dji-matrice-4d-standalone-combo-with-rc-and-charger
14. DJI Matrice 4D (Overseas Edition) - Halo Robotics, accessed August 21, 2025, https://halorobotics.com/shop/drones/dji-matrice-4d-overseas-edition/
15. DJI Matrice 4 Series vs Matrice 4D Series – Comparison for Enterprise Users - Candrone, accessed August 21, 2025, https://candrone.com/blogs/news/dji-matrice-4-series-vs-matrice-4d-series-in-depth-comparison-for-enterprise-users
16. DJI Dock 3 - Specs, accessed August 21, 2025, https://enterprise.dji.com/dock-3/specs
17. DJI Matrice 3D - Terrestrial Imaging, accessed August 21, 2025, https://www.terrestrialimaging.com/products/dji-matrice-3d
18. DJI Developer, accessed August 21, 2025, https://developer.dji.com/
19. SDK Guide for DJI's Enterprise Drone Ecosystem - Insights, accessed August 21, 2025, https://enterprise-insights.dji.com/blog/dji-sdk-guide
20. Low threshold access to third-party cloud platform - DJI Developer, accessed August 21, 2025, https://developer.dji.com/cloud-api
21. Configuration | Robotics Toolkit Documentation - DroneDeploy, accessed August 21, 2025, https://docs-automate.dronedeploy.com/robotics-toolkit/agent-plugins/dji/configuration
22. DJI Dock Provisioning and Set Up - DroneDeploy, accessed August 21, 2025, https://help.dronedeploy.com/hc/en-us/articles/30927637908247-DJI-Dock-Provisioning-and-Set-Up
23. DJI Dock Powered by FlytBase, accessed August 21, 2025, https://www.flytbase.com/dji-dock
24. Register Your DJI Dock 3 - FlytBase, accessed August 21, 2025, https://docs.flytbase.com/device-management/add-and-setup-your-device/register-your-dji-dock-3
25. Product Architecture - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/product-architecture.html
26. MQTT - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/mqtt.html
27. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/product-introduction.html
28. Tutorial Reading Guide - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/tutorial-map.html
29. MQTT fundamentals - Navixy, accessed August 21, 2025, https://docs.navixy.com/expert-center/mqtt-fundamentals
30. MQTT Vs. HTTP for IoT - HiveMQ, accessed August 21, 2025, https://www.hivemq.com/blog/mqtt-vs-http-protocols-in-iot-iiot/
31. What is MQTT Quality of Service (QoS) 0,1, & 2? – MQTT Essentials: Part 6 - HiveMQ, accessed August 21, 2025, https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels/
32. MQTT QoS Guide - Quality of Service 0, 1, 2 Explained | Cedalo, accessed August 21, 2025, https://cedalo.com/blog/understanding-mqtt-qos/
33. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-access-to-cloud.html
34. Development of an Efficiency Platform Based on MQTT for UAV Controlling and DoS Attack Detection, accessed August 21, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9460209/
35. Topic Definition - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/mqtt/topic-definition.html
36. Properties - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock1/properties.html
37. pktiuk/DJI_Cloud_API_minimal: Minimal working example using DJI Cloud API - GitHub, accessed August 21, 2025, https://github.com/pktiuk/DJI_Cloud_API_minimal
38. Live Flight Controls - Event | Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock2/drc.html
39. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/topic-definition.html
40. Live Flight Controls/Remote Control - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/drc.html
41. Cloud API - Live Flight Controls/Remote Control - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/pilot-feature-set/drc.html
42. Service | Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/mqtt/others/rc/live.html
43. Service | Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock3/live.html
44. Live Stream - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/pilot-feature-set/pilot-livestream.html
45. Live Stream - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-livestream.html
46. Media Management - Event | Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock3/media.html
47. HiveMQ MQTT Broker - Enterprise ready to move IoT data, accessed August 21, 2025, https://www.hivemq.com/products/mqtt-broker/
48. MQTT Broker: How It Works, Popular Options, and Quickstart | EMQ - EMQX, accessed August 21, 2025, https://www.emqx.com/en/blog/the-ultimate-guide-to-mqtt-broker-comparison
49. The Guide to MQTT Broker - Catchpoint, accessed August 21, 2025, https://www.catchpoint.com/network-admin-guide/mqtt-broker
50. MQTT: The Critical Communication Protocol Powering Modern Drone Operations, accessed August 21, 2025, https://decentcybersecurity.eu/mqtt-the-critical-communication-protocol-powering-modern-drone-operations/
51. MQTT vs REST in IoT [Which Should You Choose?] - Nabto, accessed August 21, 2025, https://www.nabto.com/mqtt-vs-rest-iot/
52. MQTT V/s HTTP (REST) - PROLIM, accessed August 21, 2025, https://www.prolim.com/mqtt-v-s-http-rest/
53. Design And Implementation Of An Mqtt-Based Internet Of Drone Things For Swarm Drone - Telkom University Open Library, accessed August 21, 2025, https://openlibrary.telkomuniversity.ac.id/pustaka/files/196596/jurnal_eproc/design-and-implementation-of-an-mqtt-based-internet-of-drone-things-for-swarm-drone.pdf
54. (PDF) Secure Drone Communications using MQTT protocol - ResearchGate, accessed August 21, 2025, https://www.researchgate.net/publication/386548115_Secure_Drone_Communications_using_MQTT_protocol
55. Secure Drone Communications using MQTT protocol | International Journal of Computational and Experimental Science and Engineering, accessed August 21, 2025, https://www.ijcesen.com/index.php/ijcesen/article/view/685
56. Secure Drone Communications using MQTT protocol - International Journal of Computational and Experimental Science and Engineering, accessed August 21, 2025, https://www.ijcesen.com/index.php/ijcesen/article/download/685/405/2377
57. Choose Between REST API and MQTT API - MATLAB & - MathWorks, accessed August 21, 2025, https://www.mathworks.com/help/thingspeak/choose-between-rest-and-mqtt.html
58. MQTT for DJI Dock - DroneMQTT, accessed August 21, 2025, https://dronemqtt.com/dock
59. Eclipse Mosquitto, accessed August 21, 2025, https://mosquitto.org/
60. MQTT, once the cornerstone of IoT communication, is starting to show its age. - InfinyOn, accessed August 21, 2025, https://infinyon.com/blog/2024/10/the-future-of-iot/
61. Source Code Deployment Steps - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/quick-start/source-code-deployment-steps.html


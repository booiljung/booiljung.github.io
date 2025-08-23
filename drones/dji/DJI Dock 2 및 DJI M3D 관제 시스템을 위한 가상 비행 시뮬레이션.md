# DJI Dock2 및 M3D 관제 시스템을 위한 가상 비행 시뮬레이션


맞춤형 관제 소프트웨어 개발 시, 실제 기체를 사용하지 않고 비행 로직과 데이터 처리를 검증하는 능력은 프로젝트의 성공에 매우 중요하다. DJI Cloud API는 이러한 요구사항을 충족시키기 위해 가장 직접적이고 공식적으로 지원되는 방법을 제공한다. 이 API에 내장된 시뮬레이션 기능은 단순한 테스트 도구를 넘어, 개발자가 실제 하드웨어 없이도 완전한 엔드-투-엔드(end-to-end) 시스템을 검증할 수 있도록 설계된 핵심 아키텍처 요소이다. 이는 DJI가 하드웨어 중심의 생태계에서 벗어나, 개발자들이 클라우드 기반 서비스 개발에 집중할 수 있도록 지원하는 전략적 방향성을 명확히 보여준다. 따라서 Cloud API의 시뮬레이션 기능을 활용하는 것은 가장 안정적이고 미래 지향적인 개발 방법론이라 할 수 있다.


DJI Cloud API 문서에서는 DJI Dock 2 및 Dock 3의 '경로 관리(Wayline Management)' 기능이 "가상 조종석(virtual cockpit)을 통한 비행 경로 배포"를 지원한다고 명시하고 있다.1 또한, DJI의 공식 관제 소프트웨어인 FlightHub 2 역시 3D 모델을 활용하여 운영자가 "시각적으로 경로를 편집하고 촬영 결과를 시뮬레이션"할 수 있도록 지원한다.2

이 '가상 조종석' 개념은 단순한 시각화 도구가 아니라, API 설계의 핵심적인 부분을 차지한다. 이는 개발 중인 클라우드 플랫폼이 실제 임무를 배포하기 전에 3D 환경 모델을 기반으로 비행 경로를 사전 시각화하고 유효성을 검증할 수 있게 해준다. 개발자의 관점에서 이는 경로 생성부터 시뮬레이션 비행 실행에 이르는 전체 임무 계획 및 수행 워크플로우를 오직 API 호출만으로 테스트할 수 있음을 의미한다. 이 방식은 실제 운영 환경을 완벽하게 모방하므로, 개발 초기 단계부터 시스템의 통합성과 안정성을 높은 수준으로 확보할 수 있게 한다. 즉, API를 통해 가상 환경에서 임무를 생성하고, 시뮬레이션하고, 그 결과를 분석하는 모든 과정이 실제 하드웨어 운영과 동일한 로직으로 처리되므로, 개발자는 하드웨어의 제약 없이 소프트웨어 로직 검증에만 집중할 수 있다.


Cloud API 문서는 '경로 관리'와 '실시간 비행 제어(Live Flight Controls)' 기능 모두에서 `simulate_mission`이라는 명령어를 통한 "시뮬레이션 비행 디버깅"을 지원한다고 명시적으로 언급하고 있다.3 특히 실시간 비행 제어 관련 문서에서는 "시뮬레이터가 활성화되면, 기체는 독 커버 개방 및 시동과 같은 비행 임무 준비 단계를 거치게 된다"고 구체적으로 설명한다.4

이 `simulate_mission` 명령어는 개발자의 시뮬레이션 요구사항을 해결하는 핵심적인 역할을 한다. 이 메소드는 개발자가 MQTT 프로토콜을 통해 특정 토픽(예: `thing/product/{gateway_sn}/services`)으로 발행(publish)하는 서비스 호출이다. 이 기능이 사전에 계획된 임무(Wayline)와 실시간 조종(Live Flight Controls) 두 가지 주요 운영 모드 모두에서 제공된다는 점은, API가 포괄적인 시뮬레이션 환경을 제공함을 증명한다.

특히 독 커버 개방과 같은 물리적 준비 단계까지 시뮬레이션한다는 사실은 이 기능의 충실도가 매우 높다는 것을 보여준다. 이는 단순히 비행 부분만 테스트하는 것을 넘어, 임무 시작부터 종료까지의 전체 운영 시퀀스를 검증할 수 있게 해준다. 따라서 개발자는 자신의 관제 소프트웨어가 독의 상태 변화, 기체 준비 과정, 비행 실행, 그리고 임무 완료 후 복귀 및 충전까지 이어지는 모든 단계를 정확하게 처리하는지 API 레벨에서 완벽하게 테스트할 수 있다.


시뮬레이션이 실행되는 동안, 가상의 드론은 실제 드론과 동일한 MQTT 토픽으로 원격 측정(telemetry) 데이터를 발행한다. 문서는 속성(properties) 데이터를 위한 두 가지 주요 토픽 유형을 정의한다. 첫째는 고주파 데이터(0.5Hz)를 위한 `thing/product/{device_sn}/osd`이고, 둘째는 상태 변경 시에만 데이터를 보고하는 `thing/product/{device_sn}/state`이다.5

이 구조는 테스트 환경의 피드백 루프를 완성하는 핵심 요소이다. 개발자의 맞춤형 관제 소프트웨어는 이미 자체 MQTT 브로커의 해당 토픽들을 구독(subscribe)하고 있으므로, DJI 클라우드 서비스에서 실행되는 시뮬레이션으로부터 JSON 페이로드 스트림을 자연스럽게 수신하게 된다. 이 데이터에는 가상 드론의 위치, 자세, 속도 등 실제 비행과 동일한 형식의 정보가 포함된다. 이를 통해 개발자는 자신의 소프트웨어가 상태 관리, 데이터 로깅, 사용자 인터페이스(UI)의 지도 표시 등 모든 구성 요소를 실제 M3D 기체에 연결된 것처럼 검증할 수 있다. 결과적으로, API 기반 시뮬레이션은 실제 비행 없이도 실제와 거의 동일한 조건에서 전체 시스템의 통합성을 검증할 수 있는 강력하고 효율적인 방법을 제공한다.


DJI Assistant 2 (Enterprise)는 펌웨어 관리, 데이터 로깅, 그리고 강력한 시뮬레이션 기능을 제공하는 DJI의 공식 데스크톱 소프트웨어이다. 이 도구는 특정 시나리오, 특히 기체 자체의 비행 동역학이나 저수준(low-level) 제어 로직을 테스트하는 데 매우 유용하다. 하지만, 사용자의 클라우드 기반 관제 시스템 아키텍처와는 근본적인 불일치가 존재하여 엔드-투-엔드 시스템 검증을 위한 주력 도구로 사용하기에는 한계가 명확하다. 이 시뮬레이터의 역할은 시스템 전체의 통합성을 검증하는 것이 아니라, 드론 펌웨어 레벨의 동작을 독립적으로 검증하는 데 있다.


DJI는 여러 버전의 Assistant 2 소프트웨어를 제공하므로, 올바른 버전을 사용하는 것이 매우 중요하다. Matrice 시리즈 및 Mavic Enterprise 시리즈를 포함한 전문가용 드론을 위해서는 "DJI Assistant 2 (Enterprise Series)" 버전을 사용해야 한다.8 일반 소비자용이나 다른 특수 목적용 버전을 사용할 경우, 엔터프라이즈급 기체의 비행 컨트롤러나 펌웨어를 인식하지 못해 시뮬레이션 기능을 포함한 주요 기능들을 사용할 수 없다.


DJI Matrice 3D (M3D) 기체는 "Mavic 3 Enterprise (M3E)의 IP 등급 강화 버전"으로 설명된다.2 이는 두 기체가 핵심적인 비행 컨트롤러와 펌웨어 아키텍처를 공유할 가능성이 매우 높다는 것을 시사한다. DJI Assistant 2 (Enterprise Series)의 공식 릴리스 노트는 DJI Mavic 3E 및 Mavic 3T 모델에 대한 "시뮬레이터" 기능이 추가되었음을 명확히 하고 있다.13

이러한 사실들을 종합해 볼 때, Assistant 2 시뮬레이터는 M3D의 핵심 비행 시스템과 호환될 가능성이 매우 높다. 실제 M3D 기체를 PC에 USB로 연결하면, 소프트웨어는 이를 Mavic 3 Enterprise 기종으로 인식하고 시뮬레이션 모듈에 대한 접근을 허용할 것이다. 이를 통해 사용자는 M3D의 실제 비행 동역학, 센서 동작, 그리고 장애물 회피 시스템과 같은 저수준 기능들을 매우 높은 충실도로 시뮬레이션할 수 있다.


DJI Assistant 2 시뮬레이터는 데스크톱 컴퓨터에서 실행되며, 물리적으로 USB를 통해 연결된 드론(또는 비행 컨트롤러)과 직접 통신하는 방식으로 작동한다.14 반면, 개발자가 구축 중인 시스템은 인터넷을 통해 자체 MQTT 서버와 통신하는 클라우드 기반 아키텍처를 가지고 있다. 현재까지 공개된 자료에서는 Assistant 2 시뮬레이터의 원격 측정 데이터를 외부 애플리케이션이나 MQTT 브로커로 내보낼 수 있는 공식 API나 네트워크 브리지 기능이 존재한다는 증거를 찾을 수 없다.10

이는 근본적인 아키텍처의 차이점을 드러낸다. Assistant 2 시뮬레이터는 분산된 클라우드 시스템에 통합되기보다는, 로컬 환경에서 하드웨어 중심의 테스트를 위해 설계되었다. 이 간극을 메우기 위해서는 시뮬레이터와 동일한 PC에서 실행되는 복잡하고 비공식적인 '브리지' 애플리케이션을 직접 개발해야 한다. 이 브리지는 공식 API가 없기 때문에 화면 스크래핑이나 메모리 검사와 같은 비정상적인 방법으로 시뮬레이션 데이터를 캡처하고, 이를 Cloud API 사양에 맞는 JSON 페이로드 구조로 변환한 뒤, MQTT 브로커로 발행해야 하는 매우 어려운 과제를 안고 있다. 이러한 비공식적인 통합 방식은 DJI가 Assistant 2 소프트웨어를 업데이트할 때마다 브리지 코드가 손상될 위험이 크며, 이는 전체 테스트 파이프라인을 중단시킬 수 있는 심각한 프로젝트 리스크를 초래한다. 따라서 이 방법은 시스템 전체를 테스트하는 주된 워크플로우보다는, Cloud API 시뮬레이션으로는 검증이 불가능한 특정 저수준 비행 특성이나 실패 시나리오를 검증하는 제한적인 목적으로만 고려되어야 한다.


DJI의 공식 시뮬레이션 도구들을 보완하고 전문적인 소프트웨어 개발 수명 주기를 완성하기 위해서는, 맞춤형 원격 측정 모의 엔진(Custom Telemetry Mocking Engine)을 구축하는 것이 필수적이다. 이 접근법은 DJI 시뮬레이터를 대체하는 것이 아니라, 시스템의 견고성, 확장성, 그리고 자동화를 달성하기 위한 필수적인 보조 테스트 도구로서의 역할을 수행한다. 모의 엔진은 개발팀이 DJI의 외부 서비스나 하드웨어에 대한 의존성 없이 독립적으로 백엔드 인프라를 개발하고 테스트할 수 있게 해주는 강력한 자산이다.


DJI Cloud API는 MQTT, HTTPS, WebSocket과 같은 표준적이고 개방된 프로토콜을 기반으로 설계되어 서드파티 플랫폼과의 통합을 용이하게 한다.16 DJI가 제공하는 공식 Cloud API 데모 프로젝트는 실제 운영 환경을 위한 솔루션이 아닌, API 호출 로직을 이해하기 위한 참조용으로 제공되므로 개발자는 자체적으로 견고한 시스템을 구축해야 한다.1

이러한 배경에서 맞춤형 모의 엔진은 전문적인 개발 환경의 표준적인 구성 요소가 된다. 이 엔진을 통해 DJI 도구만으로는 수행하기 어렵거나 불가능한 다음과 같은 중요한 테스트를 수행할 수 있다:

- **부하 테스트 (Load Testing):** 수백 대의 가상 드론이 동시에 MQTT 브로커에 접속하는 상황을 시뮬레이션하여 서버의 성능 한계와 안정성을 테스트한다.
- **오류 주입 (Failure Injection):** 비정상적인 데이터, 네트워크 단절, 잘못된 형식의 페이로드 등을 의도적으로 발생시켜 시스템의 오류 처리 능력과 복원력을 검증한다.
- **CI/CD 통합 (Continuous Integration/Continuous Deployment):** 외부 DJI 서비스나 데스크톱 소프트웨어에 대한 의존 없이 개발 파이프라인 내에서 테스트를 자동화한다.


모의 엔진은 다음과 같은 단계별 아키텍처로 구성될 수 있다:

1. **MQTT 연결:** 엔진의 핵심은 MQTT 클라이언트이다. Python의 `paho-mqtt`와 같은 표준 라이브러리를 사용하여 실제 Dock 2와 동일한 인증 정보로 자체 MQTT 브로커에 연결한다.21 다수의 오픈소스 예제에서 이러한 구성을 확인할 수 있다.22
2. **비행 경로 및 원격 측정 데이터 생성:** 엔진은 실제와 유사한 비행 데이터를 생성해야 한다. 이는 다음과 같은 방법으로 구현할 수 있다:
   - `Flight Plan Database` 23 또는 `Great Circle Map` 25과 같은 비행 경로 생성 도구를 사용하여 일련의 경유지(waypoints)를 생성한다.
   - 생성된 경유지 사이를 보간(interpolate)하여 시계열(time-series) 형태의 GPS 좌표, 자세(롤, 피치, 요), 속도, 고도 데이터를 생성한다. 이를 위해 `dronekit` (시뮬레이션 모드) 26, `drone_sim` 27과 같은 Python 기반 드론 시뮬레이션 라이브러리를 사용하거나, 간단한 물리 법칙을 적용한 맞춤형 스크립트를 작성할 수 있다.28`gps-sdr-sim` 30과 같은 GPS 데이터 시뮬레이터나 Python GPS 추적 예제 31는 현실적인 데이터 생성 모델을 제공한다.
3. **페이로드 형식화 및 발행:** 각 시간 단계에서 생성된 원격 측정 데이터는 섹션 4에서 자세히 설명할 `osd` 및 `state` 토픽의 JSON 구조에 맞게 형식화되어야 한다. 그 후, 스크립트는 이 JSON 문자열을 MQTT 브로커의 해당 토픽으로 발행한다.

이러한 아키텍처는 백엔드 팀이 드론 하드웨어나 DJI의 시뮬레이션 서비스와 완전히 독립적으로 데이터 수집, 처리, 저장 파이프라인을 개발하고 테스트할 수 있게 해준다. 동시에 프론트엔드 팀은 동일한 모의 데이터 스트림을 사용하여 사용자 인터페이스, 지도, 원격 측정 디스플레이 등을 개발하고 테스트할 수 있다. 이러한 병렬 워크플로우는 현대 소프트웨어 공학의 표준적인 관행이며 프로젝트 타임라인을 극적으로 단축시킬 수 있다.


모의 엔진 구축을 위해 다음과 같은 도구와 라이브러리를 권장한다:

- **MQTT 브로커:** 로컬 개발 환경에서는 Docker를 통해 쉽게 배포할 수 있는 EMQX가 권장된다.22

- **MQTT 클라이언트 라이브러리 (Python):** `paho-mqtt`가 산업 표준이다.

- **원격 측정 데이터 생성:**

  - **간단한 경로:** 경유지 간의 선형 보간으로 충분할 수 있다.

  - **복잡한 동역학:** 센서 오류 모델링을 위해 `py-gnss-ins-sim` 33을, 강체 동역학 모델링을 위해 

    `drone_sim` 27 라이브러리를 활용할 수 있다.

궁극적으로, 맞춤형 모의 엔진은 DJI Assistant 2나 Cloud API 시뮬레이터를 자동화된 GitLab/Jenkins 파이프라인에 통합하는 것이 거의 불가능한 반면, Python 기반 스크립트는 매우 쉽게 통합할 수 있다는 점에서 진정한 DevOps 및 자동화 테스트를 가능하게 한다. 가상 드론을 실행하고, API를 통해 명령을 내린 다음, 올바른 원격 측정 데이터가 수신되고 처리되었는지 자동으로 검증하는 통합 테스트를 작성할 수 있게 되며, 이는 현대적이고 신뢰성 높은 개발 및 배포 프로세스의 초석이 된다.


이 섹션은 보고서의 기술적인 핵심으로, 실제 원격 측정 데이터를 파싱하거나 섹션 3에서 설명한 맞춤형 모의 엔진을 구축하는 데 필요한 필수 데이터 사전을 제공한다. 이 구조는 Cloud API 문서의 "사물 모델(Thing Model)" 개념에서 직접 파생되었다. 이 정보를 정확히 이해하고 활용하는 것은 개발 중인 관제 소프트웨어와 DJI 생태계 간의 원활한 데이터 교환을 보장하는 데 매우 중요하다.


Cloud API는 드론, 독 등 복잡한 하드웨어를 표준화된 "사물 모델(Thing Model)"로 추상화한다.5 데이터는 두 가지 주요 속성 토픽을 통해 발행된다. 첫째는 고주파 데이터를 위한 `thing/product/{sn}/osd`이고, 둘째는 이벤트 기반 데이터를 위한 `thing/product/{sn}/state`이다.6 여기서 `{sn}` 플레이스홀더는 데이터를 발행하는 장치(이 경우 M3D 기체)의 고유 시리얼 번호로 대체된다.

이 구조는 MQTT 구독 로직을 설계하는 데 있어 매우 중요하다. 관제 소프트웨어의 MQTT 구독 클라이언트는 `thing/product/+/osd` 및 `thing/product/+/state`와 같은 와일드카드 토픽을 구독하여 시스템에 연결된 모든 드론의 원격 측정 데이터를 수신해야 한다. 각 메시지 내의 JSON 페이로드에는 구체적인 데이터 포인트들이 포함된다.


`osd` 토픽은 "On-Screen Display" 데이터를 의미하며, 안정적인 주기(일반적으로 0.5Hz)로 푸시된다.5 페이로드는 `tid`, `bid`, `timestamp` 등을 포함하는 더 큰 메시지 구조 내의 JSON 객체이다.6 공개된 최소 실행 예제에서는 `Lat`, `Lon`, `height`, `att_head`, `gimbal_pitch`와 같은 핵심 필드를 확인할 수 있다.22 이 토픽은 실시간 비행 추적 및 상태 표시에 필요한 핵심 데이터를 제공한다. 애플리케이션은 주로 이 메시지의 `data` 객체 내에 있는 정보를 사용하여 지도에 드론의 위치와 방향을 렌더링하게 된다.


`state` 토픽은 값이 변경될 때만 보고되는 속성들을 위해 사용된다.5 여기에는 현재 비행 모드를 나타내는 `mode_code`나 SD 카드의 저장 공간 상태와 같은 정보가 포함된다. 이 토픽을 구독하는 것은 시스템의 효율성을 높이고 중요한 상태 변화를 즉시 포착하는 데 필수적이다. 예를 들어, 드론의 `mode_code`가 `RTH`(Return to Home)로 변경될 때 관제 시스템은 즉각적으로 반응해야 하며, 이 정보는 `state` 토픽을 통해 전달될 것이다.


개발자는 실제 데이터를 파싱하거나 모의 엔진을 구축할 때 가장 중요한 데이터 필드에 대한 빠르고 신뢰할 수 있는 참조 자료가 필요하다. 아래 표는 여러 문서 페이지에 흩어져 있는 M3D/M3TD 기체의 핵심 원격 측정 속성들을 하나의 참조 테이블로 통합한 것이다.3 이 표는 데이터 파싱팀과 모의 엔진 구축팀 모두에게 "진실의 원천(source of truth)" 역할을 하여 일관성을 보장하고 오류를 줄이는 데 기여할 것이다.

| 속성명 (Property Name)   | MQTT 토픽 | 데이터 타입 | 단위 / 열거형 | 설명                                                    |
| ------------------------ | --------- | ----------- | ------------- | ------------------------------------------------------- |
| `longitude`              | `osd`     | float       | degrees       | 기체의 WGS-84 경도.                                     |
| `latitude`               | `osd`     | float       | degrees       | 기체의 WGS-84 위도.                                     |
| `height`                 | `osd`     | float       | meters        | 이륙 지점 기준 기체의 상대 고도.                        |
| `attitude_head`          | `osd`     | float       | degrees       | 진북(True North) 기준 기체의 기수 방향(요, Yaw).        |
| `attitude_pitch`         | `osd`     | float       | degrees       | 기체의 피치(Pitch) 각도.                                |
| `attitude_roll`          | `osd`     | float       | degrees       | 기체의 롤(Roll) 각도.                                   |
| `horizontal_speed`       | `osd`     | float       | m/s           | 기체의 수평 지면 속도.                                  |
| `vertical_speed`         | `osd`     | float       | m/s           | 기체의 수직 속도.                                       |
| `mode_code`              | `state`   | enum        | 문서 참조     | 현재 비행 모드 (예: 0: Manual, 1: Atti).                |
| `gimbal_pitch`           | `osd`     | float       | degrees       | 주 짐벌의 피치 각도.                                    |
| `remain_photo_num`       | `osd`     | integer     | count         | 촬영 가능한 예상 사진 매수.                             |
| `remain_record_duration` | `osd`     | integer     | seconds       | 예상 잔여 비디오 녹화 시간.                             |
| `total_flight_time`      | `osd`     | integer     | seconds       | 기체의 누적 비행 시간.                                  |
| `wind_speed`             | `osd`     | float       | m/s           | 추정된 현재 풍속.                                       |
| `battery`                | `state`   | object      | -             | `capacity_percent` 등 배터리 상세 정보를 포함하는 객체. |


지금까지 분석한 세 가지 가상 비행 시뮬레이션 방법론—Cloud API, Assistant 2, 맞춤형 모의 엔진—은 각각 고유한 장단점을 가지며, 특정 개발 단계에서 서로 다른 목적을 수행한다. 따라서 단일 도구를 선택하기보다는, 각 방법의 강점을 활용하는 다층적 테스트 전략을 수립하는 것이 프로젝트의 성공 가능성을 극대화하는 길이다. 이 섹션에서는 각 방법을 비교 분석하고, 이를 바탕으로 체계적인 테스트 전략을 제시한다.


- **Cloud API 시뮬레이션:**
  - **장점:** 고충실도의 엔드-투-엔드 테스트 가능, DJI 공식 지원, 추가 하드웨어 불필요, 경로 비행 및 실시간 제어 모드 모두 지원.
  - **단점:** 내부 물리 엔진을 제어할 수 없는 "블랙박스" 형태, 모든 실패 시나리오를 모델링하지 못할 수 있음, DJI 클라우드 인프라에 의존적.
- **Assistant 2 시뮬레이터:**
  - **장점:** 실제 비행 컨트롤러 펌웨어 시뮬레이션, 저수준 비행 동역학 및 실패 대응 테스트에 탁월.
  - **단점:** 클라우드 시스템과 아키텍처적으로 호환되지 않음, 복잡하고 비공식적인 맞춤형 브리지 필요, 자동화 또는 대규모 테스트에 부적합.
- **맞춤형 모의 엔진:**
  - **장점:** 데이터와 시나리오에 대한 완전한 제어, 부하 테스트 및 오류 주입 가능, CI/CD 자동화에 완벽하게 부합, 개발팀 간의 의존성 분리.
  - **단점:** 실제 비행 동역학을 시뮬레이션하지 않음(데이터를 재생하거나 생성할 뿐), 구축 및 유지보수에 개발 노력 필요.


위 분석을 바탕으로 다음과 같은 단계별 다층적 테스트 전략을 수립할 것을 강력히 권장한다. 이 전략은 개발 초기 단계부터 최종 배포까지 시스템의 신뢰성을 체계적으로 확보하도록 설계되었다.

- **1단계: 백엔드 개발 및 단위 테스트 (맞춤형 모의 엔진 활용)**
  - 백엔드 개발팀은 Python 기반의 모의 엔진을 사용하여 서비스를 테스트해야 한다. 이를 통해 다양한 시나리오에 대한 데이터 파싱, 데이터베이스 저장, API 응답, 오류 처리 등을 검증하는 단위 테스트를 작성하고 자동화할 수 있다. 이 단계에서는 DJI의 외부 서비스에 대한 의존 없이 백엔드 로직의 견고성을 확보하는 데 집중한다.
- **2단계: 엔드-투-엔드 시스템 통합 테스트 (Cloud API 시뮬레이션 활용)**
  - 개별 구성 요소가 안정화되면, Cloud API의 `simulate_mission` 기능을 사용하여 전체 시스템 통합 테스트를 수행한다. 이 단계에서는 사용자가 UI에서 "임무 시작" 버튼을 클릭하는 순간부터 지도에 시뮬레이션된 드론의 비행 경로가 정확하게 표시되기까지의 전체 워크플로우가 예상대로 작동하는지 검증한다. 이는 시스템의 모든 부분이 유기적으로 연동되는지를 확인하는 가장 중요한 과정이다.
- **3.단계: 비행 동역학 및 엣지 케이스 검증 (Assistant 2 제한적 활용)**
  - DJI Assistant 2 시뮬레이터는 API를 통해 검증할 수 없는 특정하고 목표 지향적인 테스트를 위해 아껴두어야 한다. 이는 전체 시스템을 위한 QA 도구가 아니라, 엔지니어링 도구이다. "호버링 중인 M3D 펌웨어가 갑작스러운 강풍 명령에 어떻게 반응하는가?"와 같은 저수준의 질문에 답하기 위해 이 도구를 활용한다.
- **4단계: 실제 비행 검증 (최종 단계)**
  - 시뮬레이션은 결코 실제 환경을 완벽하게 대체할 수 없다. 릴리스 주기의 마지막 단계에서는 반드시 실제 DJI Dock 2와 M3D 기체를 사용하여 검증을 수행해야 한다. 이를 통해 RF 간섭, 실제 기상 조건, 하드웨어의 미세한 결함 등 현실 세계의 변수들을 최종적으로 확인할 수 있다.


단일 "가상 비행 시뮬레이션" 방법을 찾는 사용자의 질문에 대한 최상의 답변은 단일 도구가 아닌, 전략적인 접근법이다. **Cloud API 시뮬레이션**을 시스템 수준의 무결성 검증에 주력으로 사용하고, 이를 **맞춤형 모의 엔진**으로 보완하여 개발 민첩성과 견고성을 확보함으로써, 포괄적이고 전문적인 테스트 프레임워크를 구축할 수 있다. 이 이중적 전략은 프로젝트의 리스크를 크게 줄이고 개발 속도를 가속화하며, 최종적으로 훨씬 더 신뢰성 있고 확장 가능한 제품을 만들어낼 것이다.


1. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/
2. DJI Unveils the All-New DJI Dock 2 and Matrice 3D Series Drones - Advexure, accessed August 21, 2025, https://advexure.com/blogs/news/dji-unveils-dock-2-and-matrice-3d-series-drones
3. Cloud-API-Doc/docs/en/00.index.md at master - GitHub, accessed August 21, 2025, https://github.com/dji-sdk/Cloud-API-Doc/blob/master/docs/en/00.index.md
4. Live Flight Controls/Remote Control - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/drc.html
5. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/aircraft/m3d-properties.html
6. Topic Definition - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/mqtt/topic-definition.html
7. Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock2/properties.html
8. DJI Dock 2 - Downloads - DJI - DJI Enterprise, accessed August 21, 2025, https://enterprise.dji.com/dock-2/downloads
9. Virtual Flight - Download Center - DJI, accessed August 21, 2025, https://www.dji.com/downloads/djiapp/dji-virtual-flight
10. DJI Assistant 2 (Enterprise Series) - Download Center - DJI, accessed August 21, 2025, https://www.dji.com/downloads/softwares/assistant-dji-2-for-matrice
11. DJI Mavic 3 Enterprise - Download Center, accessed August 21, 2025, https://www.dji.com/downloads/products/mavic-3-enterprise
12. DJI Download Including Firmware Software Manuals and Updates, accessed August 21, 2025, https://www.dji.com/downloads
13. DJI Assistant 2(Enterprise Series) Release Note, accessed August 21, 2025, https://dl.djicdn.com/downloads/dji_assistant/20250609/DJI+Assistant+2+(Enterprise+Series)+Release+Notes(V2.1.17).pdf
14. Customize 3D environment for Assistant 2 Simulator [dji-sdk] - Stack Overflow, accessed August 21, 2025, https://stackoverflow.com/questions/52687377/customize-3d-environment-for-assistant-2-simulator-dji-sdk
15. Simulating Drone Flights using DJI Assistant 2 and Hammer Missions - YouTube, accessed August 21, 2025, https://www.youtube.com/watch?v=RKUOEqF0-Nk
16. Low threshold access to third-party cloud platform - DJI Developer, accessed August 21, 2025, https://developer.dji.com/cloud-api
17. SDK Guide for DJI's Enterprise Drone Ecosystem - Insights, accessed August 21, 2025, https://enterprise-insights.dji.com/blog/dji-sdk-guide
18. Product Architecture - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/product-architecture.html
19. Product Introduction - Cloud API, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/product-introduction.html
20. MQTT - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/mqtt.html
21. silvamleandro/UAV_Platform: MQTT platform for UAV controlling - GitHub, accessed August 21, 2025, https://github.com/silvamleandro/uav_platform
22. pktiuk/DJI_Cloud_API_minimal: Minimal working example ... - GitHub, accessed August 21, 2025, https://github.com/pktiuk/DJI_Cloud_API_minimal
23. Flight Plan Database, accessed August 21, 2025, https://flightplandatabase.com/
24. Flight Planner - Flight Plan Database, accessed August 21, 2025, https://flightplandatabase.com/planner
25. Great Circle Map, accessed August 21, 2025, https://www.greatcirclemap.com/
26. How to program a drone using Python: A beginner's ... - The Drone Girl, accessed August 21, 2025, https://www.thedronegirl.com/2025/07/28/program-drone-python/
27. SuhrudhSarathy/drone_sim: A Simple drone simulator written in Python - GitHub, accessed August 21, 2025, https://github.com/SuhrudhSarathy/drone_sim
28. bobzwik/Quadcopter_SimCon: Quadcopter Simulation and Control. Dynamics generated with PyDy. - GitHub, accessed August 21, 2025, https://github.com/bobzwik/Quadcopter_SimCon
29. Write a Python Dronekit Script to Control a Simulated Drone - YouTube, accessed August 21, 2025, https://www.youtube.com/watch?v=-RO9qgZVqBc
30. osqzss/gps-sdr-sim: Software-Defined GPS Signal Simulator - GitHub, accessed August 21, 2025, https://github.com/osqzss/gps-sdr-sim
31. Gps Tracker Using Python - GeeksforGeeks, accessed August 21, 2025, https://www.geeksforgeeks.org/python/gps-tracker-using-python/
32. How to Use GPS for Location Tracking in Python | Visualize GPS Data with Folium (2025), accessed August 21, 2025, https://www.youtube.com/watch?v=ZVfeIWk6974&pp=0gcJCdgAo7VqN5tD
33. Open-Source, Python-based GNSS-INS Simulation | by Mike Horton | Medium, accessed August 21, 2025, https://medium.com/@mikehorton/open-source-python-based-gnss-ins-simulation-dd38d7dc729a
34. Properties - Cloud API - DJI Developer, accessed August 21, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/mqtt/m4-series/properties.html


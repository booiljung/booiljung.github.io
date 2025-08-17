# QGroundControl 외부 임무 주입 방법

## 서론

본 보고서는 QGroundControl(QGC)의 핵심 소스 코드를 수정하지 않고 외부 파일이나 인터페이스를 통해 비행 경로(임무)를 주입하는 방법에 대한 심층적인 기술 분석을 제공합니다. MAVLink 생태계의 핵심 지상관제시스템(Ground Control Station, GCS)으로서 1, QGC가 외부 시스템과 상호 운용하는 능력은 고급 및 자동화된 운영에 매우 중요합니다. 본 보고서는 이 과업을 위한 네 가지 주요 접근 벡터를 탐구합니다: 직접적인 파일 가져오기, 네트워크 기반 MAVLink 통신, 외부 개발자 SDK를 통한 프로그래밍 방식 제어, 그리고 QGC의 플러그인 아키텍처를 활용한 고급 통합이 그것입니다. 각 방법은 복잡성, 유연성, 그리고 이상적인 사용 사례에 대해 평가될 것이며, QGC의 기능을 확장하고자 하는 시스템 통합자 및 개발자를 위한 결정적인 가이드를 제공할 것입니다.

## 1.  파일 포맷을 통한 직접적인 임무 주입

이 섹션에서는 가장 직접적인 임무 주입 방법, 즉 QGC가 기본적으로 이해하고 로드할 수 있는 형식의 파일을 생성하는 방법을 분석합니다. 이러한 방법은 주로 비행 전 계획에 적합하며, 단순성과 QGC 사용자 인터페이스 또는 특정 파일 시스템 기반 자동화 기능에 의존하는 특징을 가집니다.

### 1.1  현대적 표준: JSON `.plan` 파일

QGC의 기본적이고 가장 기능이 풍부한 임무 파일 형식은 JSON 기반의 `.plan` 파일입니다.3 이 형식은 QGC 내에서 현대적인 임무 저장 및 로딩의 초석입니다. JSON에 기반한 구조적인 특성 덕분에 외부 도구에 의한 프로그래밍 방식의 생성이 이상적입니다. 이 스키마는 단순한 임무 항목의 순서뿐만 아니라 복잡한 패턴(측량, 코리더 스캔), 지오펜스, 집결 지점까지 지원하므로 전체 작전 계획을 위한 포괄적인 컨테이너 역할을 합니다.3

이 파일 형식은 QGC 개발자 가이드에 명시적으로 문서화되어 있습니다.3 최상위 구조는 `mission`, `geoFence`, `rallyPoints` 객체를 포함합니다. `mission` 객체 자체는 `firmwareType`, `vehicleType`, `cruiseSpeed`와 같은 메타데이터와 개별 임무 명령을 나열하는 핵심적인 `items` 배열을 포함합니다.3

JSON과 같이 구조화되고 기계가 읽을 수 있는 형식을 채택한 것은 드론 임무의 복잡성이 증가한 직접적인 결과입니다. 측량(Survey) 5이나 구조물 스캔(Structure Scan) 3과 같은 기능들은 단순한 웨이포인트 목록으로는 적절히 표현될 수 없습니다. 초기 드론 임무는 단순한 웨이포인트 간 순차 비행이었고, 이를 위해 일반 텍스트 파일로도 충분했습니다.6 그러나 드론 사용 사례가 복잡한 항공 사진 촬영 및 3D 모델링으로 진화하면서 QGC는 '패턴' 도구와 같은 고수준 기능을 도입했습니다. 이러한 패턴은 단일 MAVLink 명령이 아니라 QGC의 플래너가 일련의 웨이포인트와 카메라 트리거로 해석하는 추상화입니다. 이러한 복잡한 추상화를 충실하게 저장하고 로드하기 위해서는 더 표현력이 풍부한 파일 형식이 필요했습니다. 중첩된 객체와 복잡한 데이터 구조를 표현할 수 있는 JSON의 능력은 이를 위한 논리적인 선택이었고, 이는 `.plan` 형식 사양으로 이어졌습니다.3 이 형식을 통해 QGC는 결과적인 웨이포인트뿐만 아니라 사용자의 고수준 *의도*(예: "이 다각형 영역을 70% 중첩률로 측량")까지 저장할 수 있습니다.

**표 1: QGC `.plan` JSON 스키마 참조**

| 최상위 키             | 타입     | 설명                                                   |
| --------------------- | -------- | ------------------------------------------------------ |
| `fileType`            | String   | "Plan"으로 고정되어야 함.                              |
| `groundStation`       | String   | 파일을 생성한 GCS (예: "QGroundControl").              |
| `version`             | Number   | 계획 파일 형식의 버전. 현재는 1.                       |
| `mission`             | Object   | 임무 항목과 관련 메타데이터를 포함하는 핵심 객체.      |
| `geoFence`            | Object   | 원형 및 다각형 지오펜스 정의를 포함하는 객체.          |
| `rallyPoints`         | Object   | 집결 지점(Rally Points) 정의를 포함하는 객체.          |
| **Mission 객체 키**   | **타입** | **설명**                                               |
| `firmwareType`        | Number   | 임무가 생성된 펌웨어 유형 (`MAV_AUTOPILOT` 열거형 값). |
| `vehicleType`         | Number   | 임무가 생성된 기체 유형 (`MAV_TYPE` 열거형 값).        |
| `cruiseSpeed`         | Number   | 고정익 또는 VTOL 기체의 기본 순항 속도.                |
| `hoverSpeed`          | Number   | 멀티로터 기체의 기본 호버링 속도.                      |
| `plannedHomePosition` | Array    | [위도, 경도, 고도] 형식의 계획된 홈 위치.              |
| `items`               | Array    | `SimpleItem` 또는 `ComplexItem` 객체의 목록.           |
| **SimpleItem 키**     | **타입** | **설명**                                               |
| `command`             | Number   | 이 임무 항목에 대한 MAVLink 명령 ID (`MAV_CMD`).       |
| `frame`               | Number   | 위치 좌표 프레임 (`MAV_FRAME`).                        |
| `params`              | Array    | `param1`부터 `param7`(x, y, z)까지의 명령별 파라미터.  |
| `autoContinue`        | Boolean  | 다음 웨이포인트로 자동 진행 여부.                      |
| `type`                | String   | "SimpleItem"으로 고정.                                 |

### 1.2  레거시 지원: 일반 텍스트 임무 파일 (`.mission`, `.waypoints`)

QGC는 `.mission` 또는 `.waypoints`와 같은 파일 확장자를 가진 더 단순한 탭으로 구분된 일반 텍스트 임무 형식과의 하위 호환성을 유지합니다.7 이 형식은 MAVLink `MISSION_ITEM` 메시지 구조를 직접적으로 표현합니다.

이 형식은 첫 줄에 `QGC WPL <VERSION>`으로 정의되며, 그 다음 줄부터는 `MISSION_ITEM` 필드에 해당하는 탭으로 구분된 파라미터들이 나열됩니다: 인덱스, 현재 WP, 좌표 프레임, 명령, 파라미터 1-4, 위도, 경도, 고도, 그리고 자동진행(autocontinue).6 GCS 애플리케이션 간에 버전 불일치 이력이 있으며, Mission Planner는 버전 110을, QGC는 120을 사용하는 것으로 알려져 있습니다.6 DroneKit 또한 이 "Waypoint 파일 형식"을 참조합니다.8

`.plan` 파일의 우수성에도 불구하고 이러한 단순한 형식이 계속 지원되는 것은 MAVLink 생태계의 핵심 설계 원칙인 상호 운용성과 하위 호환성을 강조합니다. MAVLink 생태계는 QGC, Mission Planner, APM Planner 등 다양한 공급업체와 오픈 소스 프로젝트의 여러 도구로 구성되어 있습니다.9 이 도구들은 각기 다른 속도로 발전했기 때문에, 사용자들이 기본 비행 계획을 교환할 수 있도록 하는 단순하고 공통된 분모 형식이 필수적이었습니다. QGC가 `.plan` 형식으로 발전했지만, 일반 텍스트 형식에 대한 지원을 중단하면 많은 수의 오래된 도구, 스크립트, 워크플로우와의 호환성이 깨지고 사용자들이 소외될 수 있습니다. 따라서 이 레거시 형식을 지원하는 것은 `.plan` 형식보다 적은 기능을 지원하는 비용을 감수하더라도, 광범위한 생태계 호환성을 유지하기 위한 의도적인 선택입니다.

### 1.3  임무 경계 정의: KML 및 SHP 파일 통합

QGC는 KML(Keyhole Markup Language) 및 SHP(Shapefile) 파일을 가져오는 메커니즘을 제공합니다. 그러나 이 기능은 특정 목적을 가지며 종종 오해를 받습니다. 이는 범용 오버레이가 아니라, 측량(Survey), 코리더 스캔(Corridor Scan) 또는 구조물 스캔(Structure Scan) 패턴의 경계를 정의하기 위해 다각형을 가져오는 데 사용됩니다.5

이 과정은 계획(Plan) 뷰에서 측량 패턴을 생성한 다음 "KML/SHP" 버튼을 사용하여 파일을 선택하는 것을 포함합니다.10 시스템은 KML 파일 내에서 `Polygon` 노드를 찾을 것으로 예상하며, 만약 선이나 점만 포함되어 있다면 가져오기는 "Unable to find Polygon node in KML" 오류와 함께 실패합니다.12 이 제한은 여러 포럼 게시물에서 문서화된 바와 같이 사용자 혼란의 빈번한 원인입니다.12

KML 가져오기 기능은 임무 주입 도구가 아니라 *임무 생성 유틸리티*입니다. GIS나 일반 매핑 배경을 가진 사용자들은 KML 파일을 임의의 지리적 데이터(점, 선, 다각형)를 표시하는 방법으로 익숙하게 사용하며, 자연스럽게 QGC의 "KML 가져오기" 기능도 동일하게 작동할 것이라고 가정합니다.12 그러나 QGC 개발자들은 이 기능을 매우 특정한 워크플로우를 위해 구현했습니다: 사용자가 외부 GIS 도구10에서 복잡한 측량 영역을 정의한 다음 그 경계를 QGC의 측량 도구로 가져올 수 있도록 하는 것입니다. 그러면 QGC는 이 가져온 다각형을 사용하여 고도, 카메라 사양, 이미지 중첩률과 같은 사용자 정의 파라미터를 기반으로 실제 임무 웨이포인트와 카메라 트리거를 *생성*합니다.5 따라서 KML 파일은 임무 자체가 아니라 임무 생성 과정에 대한 *입력*입니다. 이 구별이 이 기능의 동작과 한계를 이해하는 열쇠이며, 미리 정의된 비행 경로(선 또는 일련의 웨이포인트)를 주입하는 데는 사용할 수 없습니다.

### 1.4  자동화된 주입: `AutoLoad#.plan` 메커니즘

QGC는 UI 내에서 사용자 상호 작용을 완전히 우회하여 기체에 임무를 자동으로 업로드할 수 있는 강력한 자동화 기능을 포함합니다.

이 기능은 "Autoload Missions" 설정을 통해 활성화됩니다. 활성화되면 QGC는 애플리케이션 데이터 디렉토리에서 `AutoLoad#.plan`이라는 이름의 파일을 찾습니다. 여기서 `#`은 기체의 MAVLink 시스템 ID로 대체됩니다.15 해당 ID를 가진 기체가 연결될 때 이 파일이 존재하면, QGC는 즉시 기체에 임무 업로드 프로세스를 시작합니다.

이 기능은 외부 애플리케이션이 프로그래밍 방식의 사전 비행 컨텍스트에서 기체들의 임무를 관리할 수 있는 간단하면서도 효과적인 "API"를 제공합니다. 예를 들어, 한 조직이 각각 고유한 MAVLink 시스템 ID를 가진 수십 대의 드론을 보유하고 있을 수 있습니다. 별도의 외부 임무 관리 애플리케이션(예: 웹 서비스)이 운영상의 필요에 따라 각 드론에 대한 일일 비행 계획을 생성하는 책임을 맡을 수 있습니다. 이 외부 애플리케이션은 적절한 `.plan` 파일을 생성하여 기체 ID에 따라 이름(예: `AutoLoad1.plan`, `AutoLoad2.plan` 등)을 지정하고 QGC 애플리케이션 디렉토리에 배치할 수 있습니다. 현장 운영자가 QGC를 시작하고 드론을 연결하면, 잘못된 파일을 선택하는 것과 같은 인적 오류의 가능성 없이 올바른 임무가 자동으로 로드됩니다. 이는 QGC를 수동 계획 도구에서 단순한 "비행 실행" 인터페이스로 변환시키며, 복잡한 임무 로직은 외부 시스템에서 처리됩니다. 이는 프로그래밍 방식의 사전 비행 주입을 위한 견고한 방법입니다.

### 1.5  종합: 올바른 파일 형식 선택

이 하위 섹션은 1.1부터 1.4까지의 결과를 종합하여 다양한 시나리오에 어떤 파일 형식을 사용해야 하는지에 대한 명확한 지침을 제공합니다.

**표 2: 임무 파일 형식 비교**

| 기능               | `.plan` (JSON)                             | `.mission` / `.waypoints`   | KML / SHP                        | `AutoLoad#.plan`             |
| ------------------ | ------------------------------------------ | --------------------------- | -------------------------------- | ---------------------------- |
| **기본 형식**      | JSON                                       | 탭으로 구분된 텍스트        | XML / 바이너리                   | `.plan` 형식 사용            |
| **지원 기능**      | 웨이포인트, 복합 패턴, 지오펜스, 집결 지점 | 기본 웨이포인트만           | 패턴 경계 정의만                 | `.plan` 파일의 모든 기능     |
| **주입 방법**      | QGC UI를 통한 수동 로드                    | QGC UI를 통한 수동 로드     | QGC 패턴 도구를 통한 로드        | 기체 연결 시 자동 로드       |
| **복잡성**         | 중간 (JSON 생성 필요)                      | 낮음 (단순 텍스트 생성)     | 낮음 (GIS 도구로 생성)           | 중간 (파일 시스템 관리 필요) |
| **주요 사용 사례** | 모든 기능을 갖춘 현대적인 임무 계획        | 다른 GCS와의 기본 임무 교환 | 외부 GIS 데이터로 측량 영역 정의 | 자동화된 사전 비행 임무 할당 |
| **유연성**         | 높음                                       | 낮음                        | 매우 낮음 (특정 목적)            | 높음 (프로그래밍 가능)       |

## 2.  MAVLink 프로토콜을 통한 네트워크 기반 주입

이 섹션은 MAVLink 프로토콜을 사용하여 네트워크를 통해 QGC와 직접 통신함으로써 임무를 주입하는 더 동적이고 유연한 접근 방식을 탐구합니다. 이 방법은 QGC를 네트워크 노드로 취급하여 실시간, 자동화된 상호 작용의 가능성을 엽니다.

### 2.1  QGroundControl UDP MAVLink 엔드포인트

기본적으로 QGC는 UDP 소켓을 열고 포트 14550에서 들어오는 MAVLink 트래픽을 수신 대기합니다.17 이는 네트워크상의 MAVLink 지원 기체와 자동 연결하기 위한 기본적인 설계 기능입니다.

QGC는 UDP 포트 14550에 바인딩하고 하트비트(heartbeat)를 기다립니다.19 MAVLink 패킷이 수신되면 QGC의 `LinkManager`는 발신자의 소스 IP 주소와 포트를 식별합니다. 이후 QGC에서 해당 "기체"로의 모든 통신은 그 특정 소스 주소와 포트로 다시 전달됩니다.20 만약 포트 14550이 다른 애플리케이션(여러 GCS나 MAVLink 라우터가 실행될 때 흔한 문제)에 의해 이미 사용 중이라면, QGC는 바인딩에 실패하고 오류를 보고합니다.22

QGC의 기본 UDP 리스너는 의도적으로 개방된 인터페이스로, 최대의 상호 운용성과 자동 탐지를 위해 설계되었습니다. 이는 외부 애플리케이션이 단순히 기체를 모방하여 QGC와 MAVLink 링크를 설정하는 데 활용될 수 있습니다. MAVLink 프로토콜은 잠재적으로 애드혹(ad-hoc) 네트워크로 구성된 장치들(드론, GCS, 컴패니언 컴퓨터)을 위해 설계되었습니다. UDP 브로드캐스트/멀티캐스트와 같은 간단한 비연결형 프로토콜은 탐지에 이상적입니다. 기체는 단순히 `HEARTBEAT` 메시지를 브로드캐스팅하기 시작할 수 있습니다. QGC와 같은 네트워크상의 모든 GCS는 사전 구성 없이 사용 가능한 기체를 발견하고 표시하기 위해 이러한 하트비트를 수신 대기할 수 있습니다. 이러한 설계는 통신을 시작하기 위해 인증이나 핸드셰이크가 필요 없음을 의미합니다. 외부 스크립트는 `localhost:14550`으로 `HEARTBEAT` 메시지를 보낼 수 있으며, QGC는 충실하게 새 기체를 목록에 추가하고 통신을 시도할 것입니다. 이것은 데이터를 주입하기 위한 강력하지만 안전하지 않은 진입점을 제공합니다.

### 2.2  MAVLink 임무 업로드 프로토콜: 저수준 관점

MAVLink를 통한 임무 전송은 임무 프로토콜(Mission Protocol)로 알려진 특정 상태 기반 마이크로서비스에 의해 관리됩니다. 이 프로토콜을 이해하는 것은 네트워크를 통해 임무를 주입하려는 모든 도구에 필수적입니다.

이 프로토콜은 GCS(클라이언트)가 기체(서버)에 `MISSION_COUNT` 메시지를 보내는 것으로 시작됩니다.24 그러면 기체는 수신할 준비가 된 각 임무 항목에 대해 하나씩 `MISSION_REQUEST_INT` 메시지 시리즈로 응답합니다. GCS는 각 요청에 해당하는 `MISSION_ITEM_INT` 메시지로 회신합니다. 모든 항목이 전송될 때까지 이 과정이 계속되며, 마지막으로 기체는 성공 또는 실패를 확인하는 최종 `MISSION_ACK`를 보냅니다.25

임무를 QGC로 *주입*하기 위해서는 외부 애플리케이션이 MAVLink "기체"(서버) 역할을 해야 하고 QGC가 "GCS"(클라이언트)가 되어야 합니다. 표준 프로토콜 흐름은 GCS가 업로드를 시작하는 것입니다. 그러나 사용자 질의는 임무를 QGC로 *주입*하는 것에 관한 것입니다. QGC는 임의의 네트워크 서비스로부터 임무를 "요청"하는 메커니즘을 가지고 있지 않습니다. 대신, 외부 도구는 먼저 하트비트를 보내 링크를 설정해야 합니다(2.1절 참조). 그러면 QGC는 이 새로운 "기체"를 보고 표준 연결 절차를 시작하며, 여기에는 기체로부터 임무를 요청하는 과정이 포함됩니다.19 따라서 외부 스크립트는 MAVLink 임무 프로토콜의 *서버 측*을 구현해야 합니다. QGC의 `MISSION_COUNT`(QGC가 다운로드를 요청하므로 아마도 0일 것임)를 수신 대기한 다음, 스크립트에서 QGC로 임무를 가져오기 위해 적절하게 응답해야 합니다. 더 견고한 방법은 QGC가 `MISSION_REQUEST_LIST`를 보내 임무 항목을 요청할 때까지 기다린 다음, 스크립트가 업로드 시퀀스를 시작하는 것입니다. 이러한 역할 반전은 원하는 결과를 달성하기 위한 명백하지 않지만 중요한 아키텍처 개념입니다.

**표 3: MAVLink 임무 업로드 프로토콜 시퀀스**

| 단계 | 발신자 (역할)    | 수신자 (역할)    | 메시지                | 설명                                                         |
| ---- | ---------------- | ---------------- | --------------------- | ------------------------------------------------------------ |
| 1    | GCS (클라이언트) | 기체 (서버)      | `MISSION_COUNT`       | 업로드할 임무 항목의 총 수를 알립니다.                       |
| 2    | 기체 (서버)      | GCS (클라이언트) | `MISSION_REQUEST_INT` | 시퀀스 번호 0부터 시작하여 특정 임무 항목을 요청합니다.      |
| 3    | GCS (클라이언트) | 기체 (서버)      | `MISSION_ITEM_INT`    | 요청된 시퀀스 번호에 해당하는 임무 항목 데이터를 보냅니다.   |
| 4    | ...              | ...              | ...                   | 모든 항목이 전송될 때까지 2단계와 3단계를 반복합니다.        |
| 5    | 기체 (서버)      | GCS (클라이언트) | `MISSION_ACK`         | 전체 업로드 트랜잭션의 성공(`MAV_MISSION_ACCEPTED`) 또는 실패를 알립니다. |

### 2.3  실용적인 구현: MAVLink 프록시를 통한 임무 주입

프록시 또는 라우터 아키텍처는 데이터를 가로채고 주입하는 강력하고 덜 침해적인 방법을 제공합니다. MAVProxy는 이러한 도구의 대표적인 예입니다.

MAVProxy는 기체와 GCS 사이에 위치할 수 있는 MAVLink 라우터입니다.27 직렬 포트(기체)에서 MAVLink를 수신하여 하나 이상의 UDP 엔드포인트(QGC 등)로 전달할 수 있습니다. MAVProxy는 모듈식 아키텍처를 가지고 있어 27, 사용자가 NTRIP GPS 보정 정보 29 또는 GPS 데이터 31를 주입하는 것과 같은 특정 작업을 위한 플러그인을 로드할 수 있습니다.

프록시 아키텍처는 GCS나 기체 전체를 사칭하지 않고 데이터 스트림을 조작할 수 있기 때문에 실시간 임무 주입에 가장 유연하고 강력한 방법입니다. 기체를 QGC에 사칭하는 것(2.2절에서처럼)은 스크립트가 QGC로부터의 *모든* 통신을 처리해야 하므로 복잡할 수 있습니다. SDK로 기체를 직접 제어하는 것(3.0절에서처럼)은 임무가 기체에서 다시 다운로드될 때까지 QGC에서 보이지 않거나 편집할 수 없음을 의미합니다. 프록시 아키텍처는 이러한 문제들을 피합니다. 프록시는 대부분의 MAVLink 메시지(하트비트, 자세, 위치)를 실제 기체와 QGC 사이에서 투명하게 전달할 수 있습니다. 그러나 프록시는 특정 메시지를 가로채도록 프로그래밍될 수 있습니다. 예를 들어, QGC가 기체에 `MISSION_REQUEST_LIST`를 보내는 것을 보면, 프록시는 이 메시지를 가로채 기체에 도달하지 못하게 막고, 대신 외부에서 생성된 자체 임무로 QGC에 응답할 수 있습니다. QGC는 주입된 임무를 수신하고 그것이 기체에서 온 것이라고 믿습니다. 기체는 이 트랜잭션이 일어났다는 사실조차 모릅니다. 이는 다른 통신에 최소한의 방해를 주면서 임무 데이터 스트림에 대한 정밀한 제어를 제공합니다. Pymavlink를 사용하는 맞춤형 Python 스크립트로 이를 달성할 수 있습니다.32

## 3.  외부 개발자 SDK를 통한 프로그래밍 방식 제어

이 섹션은 임무를 프로그래밍 방식으로 관리하고 업로드하기 위해 고수준 소프트웨어 개발 키트(SDK)를 사용하는 것을 평가합니다. 이러한 도구들은 MAVLink 프로토콜의 저수준 세부 사항을 추상화합니다.

### 3.1  고수준 추상화: MAVSDK를 이용한 임무 관리

MAVSDK는 Dronecode 재단에서 지원하는 MAVLink 시스템과 상호 작용하기 위한 공식적인 고수준 SDK입니다. Python, C++, Swift, Java와 같은 언어로 현대적인 비동기 API를 제공합니다.

MAVSDK는 임무 관리를 위한 명시적인 함수를 가지고 있습니다. 45의 Python 예제는 `.plan` 파일을 파싱할 수 있는 `import_qgroundcontrol_mission` 함수와 전체 MAVLink 임무 프로토콜 핸드셰이크를 자동으로 처리하는 `upload_mission` 함수를 보여줍니다.

MAVSDK와 같은 SDK는 *기체*를 제어하도록 설계되었다는 점이 중요한 구별점입니다. 이들은 QGC 애플리케이션 자체를 직접 조작하는 API를 제공하지 않습니다. MAVSDK의 목적은 프로그래밍 방식의 비행 제어를 가능하게 하여 드론을 프로그래밍 가능한 플랫폼으로 만드는 것입니다. 통신 대상은 비행 컨트롤러(또는 시뮬레이터)의 MAVLink 엔드포인트입니다. 스크립트가 MAVSDK의 `upload_mission`을 사용하면, 임무를 기체로 보내는 것입니다. 만약 QGC도 해당 기체에 연결되어 있다면, 기체의 임무가 변경되었음을 감지하고 일반적으로 사용자에게 새로운 임무를 다운로드하여 표시하라는 메시지를 표시합니다. 따라서 MAVSDK를 사용하는 것은 임무를 *시스템* 전체에 주입하는 것이며, QGC는 이 변경 사항을 간접적으로 반영합니다. 이는 프록시 방법과는 다른 중요한 아키텍처상의 구별입니다.

### 3.2  저수준 세분성: Pymavlink를 이용한 임무 제작

Pymavlink는 MAVLink 프로토콜의 기초적인 Python 구현체입니다.33 MAVSDK에 비해 복잡성이 높지만, 모든 MAVLink 메시지를 생성, 전송, 수신하는 도구를 제공하여 궁극적인 유연성을 제공합니다.

Pymavlink는 UDP 또는 TCP 연결을 설정하는 데 사용될 수 있습니다.33 스크립트는 하트비트를 기다려 링크를 설정(`wait_heartbeat()`)한 다음, 조작된 메시지를 전송(`mav.send()`)할 수 있습니다.32 이를 통해 개발자는 MAVLink 임무 프로토콜(또는 다른 MAVLink 마이크로서비스)을 처음부터 구현할 수 있으며, 이는 사용자 정의 메시지 전송 36 또는 전달 라우터 구현 32 예제에서 볼 수 있습니다.

Pymavlink는 단순히 기체를 제어하기 위한 것이 아니라 MAVLink 언어를 구사하기 위한 도구입니다. 이는 이전에 논의된 더 진보된 주입 아키텍처를 구축하는 데 이상적인 도구임을 의미합니다. MAVSDK는 편리한 `upload_mission` 함수를 제공하지만, 2.3절의 프록시 아키텍처를 구현하고 싶다면 어떨까요? MAVSDK에는 "프록시" 모드가 없습니다. 더 저수준 라이브러리인 Pymavlink는 이를 생성할 수 있는 구성 요소를 제공합니다. 32에서 보여주듯이 두 개의 연결(하나는 기체로, 다른 하나는 QGC로)을 열고 메시지를 전달하는 루프를 작성할 수 있습니다. 2.2절의 기체 사칭 아키텍처를 구현하고 싶다면 어떨까요? Pymavlink는 QGC를 속이는 데 필요한 정확한 하트비트 및 임무 프로토콜 응답을 만들고 보낼 수 있게 해줍니다. 따라서 Pymavlink는 단순한 기체 제어를 넘어선 가장 정교한 네트워크 기반 주입 방법을 가능하게 하는 기술입니다.

### 3.3  제어 구별: QGC에 주입 vs. 기체 명령

이 하위 섹션에서는 두 가지 주요 SDK 기반 접근 방식의 아키텍처 차이점과 그 의미를 명시적으로 요약하여 3.1절의 분석을 구체화할 것입니다. "기체 직접 제어" 접근 방식(SDK를 사용하여 드론을 명령하고 QGC는 수동적인 모니터 역할)과 "GCS 대상" 접근 방식(Pymavlink 스크립트를 사용하여 기체를 사칭하고 QGC에 임무를 업로드)을 대조할 것입니다. 전자는 시스템의 상태를 직접 변경하고 QGC가 이를 따르도록 하는 반면, 후자는 QGC의 내부 상태를 직접 조작하는 것을 목표로 합니다.

## 4.  고급 통합: 플러그인 및 사용자 정의 빌드 아키텍처

이 섹션은 QGC의 기능을 확장하는 가장 깊이 통합된 방법을 다룹니다. 개발 환경이 필요하지만, 이는 사용자 정의를 위해 공식적으로 승인된 방법이며 핵심 QGC 소스 파일을 수정하지 않으므로 사용자의 제약 조건을 준수합니다.

### 4.1  경계 정의: 사용자 정의 빌드 vs. 소스 코드 해킹

"사용자 정의 빌드(custom build)"와 단순히 QGC 소스 코드를 포크(fork)하여 수정하는 것을 구별하는 것이 중요합니다. QGC 개발팀은 후자를 권장하지 않으면서 전자를 용이하게 하기 위한 특정 아키텍처를 만들었습니다.

사용자 정의 빌드는 제3자가 주 업스트림 저장소의 변경 사항을 쉽게 따라갈 수 있는 방식으로 자신만의 브랜드화된, 특화된 버전의 QGC를 만들 수 있도록 합니다.37 메인라인 코드를 해킹하는 주된 위험은 QGC 팀의 향후 업데이트를 병합하는 것이 거의 불가능해져서 사용자 정의 버전이 오래되고 지원되지 않는 기반에 갇히게 된다는 것입니다.39 권장되는 접근 방식은 QGC 저장소를 서브모듈이나 종속성으로 사용하는 사용자 정의 코드를 위한 별도의 저장소를 만드는 것입니다.39

플러그인/사용자 정의 빌드 아키텍처는 근본적으로 유지보수성에 관한 것입니다. 이는 QGC를 확장하기 위한 API이지, 단순히 사용자 정의 코드를 위한 폴더가 아닙니다. 한 회사가 그들의 브랜딩과 사용자 정의 임무 유형을 가진 특별한 버전의 QGC를 탑재한 드론을 출시하고 싶다고 가정해 봅시다. 만약 그들이 단순히 QGC를 포크하여 코드를 곳곳에 추가한다면, 그들은 이제 전체 코드베이스를 유지할 책임이 있습니다. QGC가 중요한 보안 패치를 출시할 때, 심하게 수정된 포크에 이를 병합하는 것은 악몽과 같을 것입니다. 플러그인 아키텍처는 안정적인 인터페이스(`QGCCorePlugin`, `FirmwarePlugin` 등)를 제공합니다.41 회사는 이러한 플러그인 서브클래스 내에서 사용자 정의 로직을 구현할 수 있습니다. 새로운 버전의 QGC가 출시되면, 그들은 단순히 QGC 서브모듈을 업데이트하면 됩니다. 플러그인 API 인터페이스가 변경되지 않는 한, 그들의 사용자 정의 빌드는 최소한의 노력으로 새로운 QGC 코어와 함께 다시 컴파일되어야 합니다. 이것이 사용자 정의를 지속 가능하고 장기적인 전략으로 만드는 이유입니다.

### 4.2  QGC 플러그인 프레임워크: 개요

QGC는 사용자 정의 빌드가 기본 동작을 재정의하거나 확장하기 위해 서브클래싱할 수 있는 여러 플러그인 클래스를 노출합니다.

주요 플러그인은 `QGCCorePlugin`(애플리케이션 수준 기능, 예: 설정, 브랜딩, UI 요소), `FirmwarePlugin`(펌웨어별 MAVLink 해석), `AutoPilotPlugin`(기체 설정 UI)입니다.41 사용자 정의 빌드는 예를 들어 새로운 UI 요소를 추가하거나, 복잡성을 줄이기 위해 단일 비행 스택을 정의하거나, 사용자 정의 비행 전 체크리스트를 구현하기 위해 이러한 플러그인의 자체 버전을 구현할 수 있습니다.37 UI 자체는 QML로 작성되며 C++ "컨트롤러" 클래스와 통신합니다.44

### 4.3  사용자 정의 임무 주입 플러그인의 실현 가능성

이 하위 섹션에서는 플러그인 아키텍처를 사용하여 원활하고 통합된 임무 주입 시스템을 만드는 방법을 이론화할 것입니다.

사용자 정의 빌드는 자체 QtQuick 인터페이스 모듈, 툴바, UI 내비게이션을 구현할 수 있습니다.37

사용자 정의 플러그인은 QGC 사용자 인터페이스에 직접 통합될 수 있기 때문에 임무 주입을 위한 가장 우아하고 사용자 친화적인 솔루션을 나타냅니다. 다른 모든 방법(파일, 네트워크 스크립트)은 QGC 애플리케이션 외부에 있습니다. 사용자는 별도의 스크립트를 실행하거나 특정 폴더의 파일을 관리해야 합니다. `QGCCorePlugin`을 사용하면 개발자는 계획(Plan) 뷰의 툴바에 직접 새 버튼을 추가할 수 있습니다. 이 버튼은 "클라우드 서비스에서 가져오기"와 같이 레이블링될 수 있습니다. 클릭하면 사용자 정의 플러그인 내의 C++ 코드가 트리거되어 외부 서버에 REST API 호출을 하여 임무 데이터를 가져올 수 있습니다. 그런 다음 플러그인 코드는 응답을 파싱하고 QGC의 내부 C++ 클래스(파일 로딩 메커니즘이 사용하는 것과 동일한 클래스)를 사용하여 임무를 계획 뷰의 데이터 모델에 직접 로드합니다. 이는 다른 어떤 방법으로도 달성할 수 없는 완전히 원활한 사용자 경험을 제공합니다. 컴파일이 필요하지만, QGC 애플리케이션의 상태에 내부에서 임무를 진정으로 "주입"하는 유일한 방법입니다.

## 5.  종합 및 권장 사항

이 마지막 섹션은 전체 보고서의 결과를 종합하여 비교 분석 및 다양한 사용 사례에 대한 실행 가능한 권장 사항을 제공합니다.

### 5.1  주입 방법론 비교 분석

모든 탐색된 방법에 대한 요약 및 직접적인 비교를 통해 각 방법의 강점과 약점을 강조합니다.

**표 4: 임무 주입 방법론 종합 비교**

| 평가 기준     | 파일 기반 (`.plan`)          | 네트워크 (프록시)                                        | SDK (MAVSDK)                          | 플러그인 (사용자 정의 빌드)        |
| ------------- | ---------------------------- | -------------------------------------------------------- | ------------------------------------- | ---------------------------------- |
| **복잡성**    | 낮음                         | 높음                                                     | 중간                                  | 매우 높음                          |
| **유연성**    | 중간                         | 매우 높음                                                | 높음                                  | 최상                               |
| **실시간성**  | 비실시간 (사전 비행)         | 실시간                                                   | 실시간                                | 실시간                             |
| **통합 수준** | 외부 (파일 시스템)           | 외부 (네트워크)                                          | 외부 (애플리케이션)                   | 내부 (애플리케이션 UI)             |
| **필요 기술** | 파일 생성 (예: 스크립팅)     | MAVLink 프로토콜, 네트워크 프로그래밍 (Python/Pymavlink) | SDK 사용 (Python, C++)                | QGC 아키텍처, C++, Qt, QML         |
| **주요 장점** | 단순성, 신뢰성               | 동적 임무 수정, 투명성                                   | 빠른 개발, 고수준 추상화              | 완벽한 UI 통합, 최고의 사용자 경험 |
| **주요 단점** | 수동 개입 또는 제한된 자동화 | 구현 복잡성, 잠재적 불안정성                             | 기체 제어 중심, 간접적인 GCS 업데이트 | 높은 초기 개발 비용, 컴파일 필요   |

### 5.2  일반적인 시나리오에 대한 권장 아키텍처

이 섹션은 일반적인 사용자 요구를 가장 적절한 주입 방법에 매핑하여 구체적인 조언을 제공합니다.

- **시나리오 1: 외부 도구를 사용한 사전 비행 계획.**
  - **권장 사항:** `.plan` 파일을 생성하고 수동 로딩 또는 자동화를 위해 `AutoLoad#.plan` 기능을 사용합니다. 이는 가장 간단하고 신뢰할 수 있는 방법입니다.
- **시나리오 2: 비행 중 실시간 동적 임무 생성.**
  - **권장 사항:** MAVLink 프록시 아키텍처(Pymavlink로 구축)를 사용하여 임무 세그먼트를 가로채고 주입합니다. 이는 비행 중 임무를 수정해야 하는 고급 애플리케이션에 가장 큰 유연성을 제공합니다.
- **시나리오 3: 테스트/자동화를 위한 간단한 스크립트 제어.**
  - **권장 사항:** MAVSDK와 같은 고수준 SDK를 사용하여 기체를 직접 명령합니다. 이는 QGC를 수동 모니터로 사용하면서 빠른 프로토타이핑 및 자동화된 테스트에 이상적입니다.
- **시나리오 4: 통합된 임무 서비스를 갖춘 상용 브랜드 GCS 제작.**
  - **권장 사항:** QGC 플러그인 아키텍처를 사용하여 사용자 정의 빌드를 만드는 데 투자합니다. 이는 가장 전문적이고 사용자 친화적인 최종 제품을 제공하지만 가장 많은 개발 노력이 필요합니다.

### 5.3  최종 평가

분석 결과를 최종적으로 요약합니다. 본 보고서는 QGroundControl의 핵심 소스 코드를 수정하지 않고도 비행 계획을 주입하는 것이 가능할 뿐만 아니라, 이를 위한 여러 잘 지원되는 방법이 존재한다고 결론 내립니다. 방법의 선택은 전적으로 요구되는 통합 수준, 동적성, 그리고 구현 팀의 기술적 역량에 달려 있습니다. 간단한 파일 드롭에서부터 정교한 네트워크 프록시 및 완전히 통합된 플러그인에 이르기까지, QGC의 개방형 아키텍처는 외부 시스템 상호 운용성을 위한 다양한 솔루션을 제공합니다.

#### **참고 자료**

1. qgroundcontrol/README.md at master - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/blob/master/README.md
2. QGroundControl – Drone Control – Ground Control Station for Small Air – Land – Water Autonomous Unmanned Systems, accessed July 10, 2025, https://qgroundcontrol.com/
3. Plan File Format | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/file_formats/plan.html
4. Document .plan file format / Issue #5234 / mavlink/qgroundcontrol - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/5234
5. Survey (Plan Pattern) | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/plan_view/pattern_survey.html
6. Mission file format clarifications / Issue #2243 / mavlink/qgroundcontrol - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/2243
7. File Formats - MAVLink Guide, accessed July 10, 2025, https://mavlink.io/en/file_formats/
8. Example: Mission Import/Export - DroneKit-Python's documentation! - Read the Docs, accessed July 10, 2025, https://dronekit-python.readthedocs.io/en/latest/examples/mission_import_export.html
9. Mission Planner Overview - ArduPilot, accessed July 10, 2025, https://ardupilot.org/planner/docs/mission-planner-overview.html
10. Uploading KMLs into QGC - INSPIRED DOCUMENTATION, accessed July 10, 2025, https://docs.inspiredflight.com/inspired-documentation/products/additional-software/qgc/uploading-kmls-into-qgc
11. Release Notes | QGC Guide (4.4), accessed July 10, 2025, https://docs.qgroundcontrol.com/v4.4.3/en/qgc-user-guide/releases/release_notes.html
12. KML file format for QGC - QGroundControl - PX4 Discussion Forum, accessed July 10, 2025, https://discuss.px4.io/t/kml-file-format-for-qgc/36680
13. How to upload a .kml into QGroundControl - PX4 Discussion Forum, accessed July 10, 2025, https://discuss.px4.io/t/how-to-upload-a-kml-into-qgroundcontrol/6146
14. Load KML into QGC - QGroundControl - ArduPilot Discourse, accessed July 10, 2025, https://discuss.ardupilot.org/t/load-kml-into-qgc/131688
15. General Settings (Settings View) | QGC Guide (4.3) - QGroundControl Guide, accessed July 10, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/tr/qgc-user-guide/settings_view/general.html
16. General Settings (Settings View) | QGC Guide (4.3) - QGroundControl Guide, accessed July 10, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/zh/qgc-user-guide/settings_view/general.html
17. We really shouldn't ask our users to manage UDP ports themselves / Issue #1681 / mavlink/qgroundcontrol - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/1681
18. MavLink UDP connections - Discussions - diydrones, accessed July 10, 2025, https://diydrones.com/forum/topics/mavlink-udp-connections
19. Communication Flow | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/communication_flow.html
20. UDP listening ports - Development - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 10, 2025, https://discuss.px4.io/t/udp-listening-ports/22939
21. What is the proper UDP port for mavlink - PX4 Discussion Forum, accessed July 10, 2025, https://discuss.px4.io/t/what-is-the-proper-udp-port-for-mavlink/33142
22. QGroundControl on Herelink - "Error Binding UDP" - Cubepilot, accessed July 10, 2025, https://discuss.cubepilot.org/t/qgroundcontrol-on-herelink-error-binding-udp/14793
23. UDP Port: address is already bound issue on Qgroundcontrol v4.4.3 - HereLink - Cubepilot, accessed July 10, 2025, https://discuss.cubepilot.org/t/udp-port-address-is-already-bound-issue-on-qgroundcontrol-v4-4-3/14738
24. Mission Upload / Download - Dev documentation - ArduPilot, accessed July 10, 2025, https://ardupilot.org/dev/docs/mavlink-mission-upload-download.html
25. Mission Protocol / ham_mavdevguide - Hamish Willee, accessed July 10, 2025, https://hamishwillee.gitbooks.io/ham_mavdevguide/en/services/mission.html
26. MAVLink Mission transfers are not transactional as they ought to be / Issue #138 - GitHub, accessed July 10, 2025, https://github.com/ArduPilot/ardupilot/issues/138
27. Modules - MAVProxy documentation - ArduPilot, accessed July 10, 2025, https://ardupilot.org/mavproxy/docs/modules/index.html
28. ArduCopter novice learns from pymavlink tools (and mistakes) - Blogs - diydrones, accessed July 10, 2025, https://diydrones.com/profiles/blogs/arducopter-novice-learns-from-pymavlink-tools-and-mistakes
29. NTRIP Injection - MAVProxy documentation - ArduPilot, accessed July 10, 2025, https://ardupilot.org/mavproxy/docs/modules/ntrip.html
30. How to send NTRIP corrections to ArduPilot with MissionPlanner, QGroundControl and MAVProxy - ArduSimple, accessed July 10, 2025, https://www.ardusimple.com/send-ntrip-corrections-to-ardupilot-with-missionplanner-qgroundcontrol-and-mavproxy/
31. GPS Input - MAVProxy documentation - ArduPilot, accessed July 10, 2025, https://ardupilot.org/mavproxy/docs/modules/GPSInput.html
32. receiving and sending mavlink messages using pymavlink library - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/53394660/receiving-and-sending-mavlink-messages-using-pymavlink-library
33. Pymavlink / GitBook - GitHub Pages, accessed July 10, 2025, https://williangalvani.github.io/ardusub-gitbook/developers/pymavlink.html
34. Using Pymavlink Libraries (mavgen) - MAVLink Guide, accessed July 10, 2025, https://mavlink.io/en/mavgen_python/
35. Pymavlink / GitBook - ArduSub, accessed July 10, 2025, https://www.ardusub.com/developers/pymavlink.html
36. Send and receive custom message over mavlink - ArduCopter - ArduPilot Discourse, accessed July 10, 2025, https://discuss.ardupilot.org/t/send-and-receive-custom-message-over-mavlink/116148
37. Custom Builds | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/custom_build/custom_build.html
38. Custom Builds | QGC Guide (4.4), accessed July 10, 2025, https://docs.qgroundcontrol.com/Stable_V4.4/en/qgc-dev-guide/custom_build/custom_build.html
39. Custom Build Implementation for CMake / Issue #11436 / mavlink/qgroundcontrol - GitHub, accessed July 10, 2025, https://github.com/mavlink/qgroundcontrol/issues/11436
40. Initial Repository Setup For Custom Build | QGC Guide (4.3), accessed July 10, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-dev-guide/custom_build/create_repos.html
41. Plugin Architecture | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/firmware_plugin.html
42. Custom Build Plugins | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/custom_build/plugins.html
43. Plugin Architecture | QGC Guide (4.4), accessed July 10, 2025, https://docs.qgroundcontrol.com/v4.4.3/zh/qgc-dev-guide/firmware_plugin.html
44. User Interface Design | QGC Guide (master), accessed July 10, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/user_interface_design/index.html
45. Import a QGroundControl missions in JSON .plan format. / Issue #439 / mavlink/MAVSDK-Python - GitHub, accessed July 10, 2025, https://github.com/mavlink/MAVSDK-Python/issues/439


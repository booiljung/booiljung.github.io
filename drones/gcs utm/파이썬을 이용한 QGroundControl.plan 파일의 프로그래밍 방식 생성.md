# 파이썬을 이용한 QGroundControl.plan 파일의 프로그래밍 방식 생성

## 서론

무인 항공기(UAV) 기술이 발전함에 따라, 자율 비행 임무의 복잡성과 규모 또한 기하급수적으로 증가하고 있습니다. 단순한 지점 간 비행을 넘어, 정밀 농업을 위한 광역 서베이, 인프라 검사를 위한 3차원 구조물 스캔, 실시간 데이터에 기반한 동적 수색 및 구조 활동 등 고도로 복잡한 임무 프로파일이 요구되고 있습니다. 이러한 복잡한 임무를 수동으로 계획하는 것은 비효율적일 뿐만 아니라, 인간의 실수에 취약하며 확장성이 떨어집니다. 따라서, 비행 계획을 프로그래밍 방식으로 자동 생성하는 기술은 현대 UAV 운영의 핵심적인 요구사항으로 부상했습니다.

QGroundControl(QGC)은 PX4 및 ArduPilot 기반 기체를 위한 강력하고 직관적인 지상 관제 시스템(GCS)으로, `.plan`이라는 파일 형식을 사용하여 비행 계획을 저장합니다.1 이 보고서는 Python 프로그래밍 언어를 사용하여 QGroundControl용 `.plan` 파일을 자동으로 생성하는 최신 방법론에 대한 심층적인 기술 분석을 제공하는 것을 목표로 합니다.

본 문서는 먼저 `.plan` 파일의 근간을 이루는 JSON 구조를 해부하여 그 해부학적 구조를 명확히 하고, 각 필드의 의미와 역할을 상세히 설명합니다. 이어서, 비행 계획의 실질적인 내용을 구성하는 MAVLink 프로토콜의 핵심 요소인 임무 커맨드(`MAV_CMD`)와 좌표계(`MAV_FRAME`)를 분석하여, `.plan` 파일과 자율주행 컨트롤러 간의 관계를 규명합니다.

이를 바탕으로, Python의 표준 라이브러리와 `pymavlink`를 활용하여 `.plan` 파일을 생성하는 실용적인 방법론을 제시합니다. 단순한 임무 생성부터 시작하여, 동적 서베이 패턴과 같은 복잡한 임무를 생성하는 고급 기법까지 단계적으로 다룰 것입니다. 마지막으로, 생성된 파일의 검증 방법, 일반적인 문제 해결 방안, 그리고 안전하고 신뢰할 수 있는 자동화 임무 생성을 위한 최상의 실천 방안을 제안함으로써, 개발자와 연구자가 현장에서 마주할 수 있는 실질적인 문제들에 대한 해결책을 제공하고자 합니다. 이 보고서는 UAV 소프트웨어 개발자, 로보틱스 엔지니어, 그리고 관련 분야 연구자들에게 프로그래밍 방식의 임무 생성에 대한 깊이 있는 이해와 실용적인 지침을 제공하는 권위 있는 기술 자료가 될 것입니다.

------

## 1.  QGroundControl.plan 파일의 해부학적 구조

QGroundControl의 `.plan` 파일은 비행 임무의 모든 세부 사항을 담고 있는 핵심적인 데이터 구조입니다. 이 파일의 구조를 정확히 이해하는 것은 프로그래밍 방식의 임무 생성을 위한 첫걸음입니다. 최신 QGC는 과거의 텍스트 기반 형식에서 벗어나, 구조화되고 확장 가능한 JSON(JavaScript Object Notation) 형식을 채택했습니다.3 이 선택은 파일을 인간이 쉽게 읽고 이해할 수 있게 할 뿐만 아니라, Python을 포함한 거의 모든 최신 프로그래밍 언어에서 별도의 파서 없이 내장 라이브러리만으로 손쉽게 다룰 수 있게 합니다.

### 1.1  기본 개념: JSON 구조

`.plan` 파일은 하나의 거대한 JSON 객체로 구성됩니다. 이 객체는 여러 키-값 쌍을 포함하며, 각 키는 비행 계획의 특정 구성 요소를 나타냅니다. 프로그래밍 방식으로 `.plan` 파일을 생성하는 작업은 본질적으로 Python의 딕셔너리(dictionary) 자료구조를 사용하여 이 JSON 객체를 올바르게 구성하고, 최종적으로 파일에 쓰는 과정입니다.

가장 최소한의 유효한 `.plan` 파일은 다음과 같은 기본 구조를 가집니다. 이 구조는 하나의 임무 아이템을 반드시 포함해야 합니다.

JSON

```
{
    "fileType": "Plan",
    "groundStation": "QGroundControl",
    "mission": {
        "items":,
                "autoContinue": true,
                "doJumpId": 1
            }
        ],
        "plannedHomePosition": [47.6, -122.1, 0.0],
        "firmwareType": 3,
        "vehicleType": 2,
        "version": 2
    },
    "version": 1
}
```

### 1.2  최상위 객체 명세

`.plan` 파일의 루트 레벨에는 파일 전체의 메타데이터를 정의하는 몇 가지 핵심 키가 존재합니다.

- `fileType`: 이 파일의 용도를 식별하는 '매직 넘버'와 같은 역할을 합니다. 값은 반드시 문자열 `"Plan"`이어야 합니다.

- `version`: `.plan` 파일 형식 자체의 버전을 나타냅니다. 현재 이 값은 `1`입니다.3

- `groundStation`: 이 파일을 생성한 GCS의 이름을 명시합니다. 일반적으로 `"QGroundControl"`이 사용됩니다.3 이 필드는 단순한 정보 제공을 넘어, 임무 파일의 호환성과 해석에 중요한 단서를 제공합니다. 

  `SimpleItem`은 대부분의 GCS에서 상호 교환이 가능하지만, `Survey`와 같은 `ComplexItem`은 이를 생성한 GCS에 특화된 정보를 담고 있을 수 있습니다. 따라서 다른 GCS가 이 파일을 읽을 때, `groundStation` 필드를 보고 자신이 이해할 수 없는 `ComplexItem`을 어떻게 처리할지(예: 무시하고 기본 `SimpleItem`만 해석) 결정할 수 있습니다. 이는 다양한 드론 생태계 내에서 기본적인 호환성을 유지하면서도 각 GCS의 고유한 고급 기능을 허용하는 지능적인 설계 방식입니다.

- `geoFence`: (선택 사항) 비행 금지 구역을 정의하는 지오펜스 정보를 포함합니다.

- `rallyPoints`: (선택 사항) 비상 상황 시 복귀할 수 있는 랠리 포인트(안전 지점) 목록을 포함합니다.

### 1.3  핵심 `mission` 객체: 상세 분석

`mission` 객체는 실제 비행 경로와 행동을 정의하는 심장부입니다. 이 객체 내부의 각 키는 임무의 성격을 규정하는 중요한 메타데이터를 담고 있습니다.

- `version`: `mission` 객체 자체의 버전으로, 현재 `2`입니다.3
- `firmwareType`: 임무가 설계된 기체의 펌웨어 유형을 나타내는 정수 값입니다. 이 값은 MAVLink의 `MAV_AUTOPILOT` 열거형(enum)에 해당하며, 예를 들어 ArduPilot은 `3`, PX4 Pro는 `12`입니다.3
- `vehicleType`: 기체의 유형(예: 고정익 `1`, 멀티로터 `2`, VTOL 등)을 나타내는 정수입니다. QGC는 이 정보를 바탕으로 임무 시뮬레이션 및 유효성 검사를 수행합니다.6
- `cruiseSpeed` / `hoverSpeed`: 기체가 연결되지 않은 상태에서 임무를 계획할 때, 총 비행 시간을 추정하기 위해 사용되는 속도 값(m/s)입니다.6
- `plannedHomePosition`: `[latitude, longitude, altitude]` 형식의 배열로, 기체 연결 없이 임무를 계획할 때 사용되는 가상의 홈 위치입니다. 이 위치는 상대 고도 계산, 임무 통계(예: 가장 먼 웨이포인트까지의 거리) 산출 등 모든 계획의 기준점이 되므로 매우 중요합니다.4
- `items`: 임무의 각 단계를 정의하는 `SimpleItem` 또는 `ComplexItem` 객체들의 배열입니다. 이 배열에 대한 자세한 내용은 2장에서 다룹니다.3

여기서 `firmwareType`과 `plannedHomePosition`은 선택 사항이 아닌, 안전과 정확성을 위한 필수 요소입니다. ArduPilot과 PX4 펌웨어는 홈 위치(MAVLink 임무 아이템 시퀀스 0)를 처리하는 방식에 근본적인 차이가 있기 때문입니다. ArduPilot은 GCS가 시퀀스 0으로 홈 위치를 보내주기를 기대하는 반면, PX4는 그렇지 않습니다. `.plan` 파일의 `plannedHomePosition` 키는 이러한 펌웨어 간의 차이를 GCS 레벨에서 흡수하기 위한 설계입니다. 즉, 파일 생성 스크립트는 '홈 위치가 어디인지'와 '어떤 펌웨어를 위한 것인지'라는 *의도*만 파일에 명시하면 됩니다. 그러면 QGC는 이 정보를 바탕으로 실제 기체에 업로드할 때, MAVLink 프로토콜에 맞춰 홈 위치를 포함하거나 제외하는 *구현*을 책임집니다. 이 두 필드를 정확하게 설정하지 않은 `.plan` 파일은 모호하고 잠재적으로 위험한 파일이 될 수 있습니다.

### 1.4  부가 임무 데이터: `geoFence`와 `rallyPoints`

이 객체들은 선택적으로 포함될 수 있으며, 임무의 안전성을 크게 향상시키는 역할을 합니다.3

- `geoFence`: 비행 영역을 제한하는 경계를 정의합니다. 이 객체는 `version` 키와 함께, `circles`(원형 펜스)와 `polygons`(다각형 펜스) 배열을 포함할 수 있습니다. 각 다각형은 위도와 경도 쌍의 배열로 정의됩니다.3
- `rallyPoints`: 비상 착륙 또는 복귀 지점을 정의합니다. 이 객체는 `version` 키와 `points` 배열을 포함하며, 각 포인트는 `[latitude, longitude, altitude]` 형식의 배열입니다.3

이러한 구조적 이해를 바탕으로, 프로그래밍 스크립트는 각 필드의 데이터 타입과 요구사항에 맞춰 Python 딕셔너리를 체계적으로 구성해야 합니다.

------

## 2.  임무 아이템 해부: 비행의 구성 요소

`mission` 객체 내의 `items` 배열은 비행 계획의 실질적인 내용을 담고 있는 순차적인 명령어 목록입니다. 이 배열은 `SimpleItem`과 `ComplexItem`이라는 두 가지 유형의 객체를 포함할 수 있으며, 이 둘의 조합으로 기체의 구체적인 행동이 결정됩니다.

### 2.1  `SimpleItem` 객체: MAVLink 커맨드의 직접적 표현

`SimpleItem`은 `.plan` 파일에서 가장 기본적인 임무 단위로, MAVLink 프로토콜의 `MISSION_ITEM` 또는 `MISSION_ITEM_INT` 메시지 하나를 직접적으로 표현합니다. 이는 자동항법장치가 이해할 수 있는 가장 낮은 수준의 명령 추상화입니다. `SimpleItem` 객체의 구조는 다음과 같습니다.3

- `type`: 객체의 유형을 나타내며, 반드시 문자열 `"SimpleItem"`이어야 합니다.
- `command`: 수행할 MAVLink 임무 커맨드를 나타내는 정수 ID입니다. 예를 들어, 웨이포인트 비행은 `16` (`MAV_CMD_NAV_WAYPOINT`), 이륙은 `22` (`MAV_CMD_NAV_TAKEOFF`)입니다.
- `frame`: `params`에 포함된 위치 정보(위도, 경도, 고도)가 어떤 좌표계를 기준으로 하는지를 정의하는 정수 ID입니다. 이는 MAVLink의 `MAV_FRAME` 열거형에 해당합니다.
- `params`: 해당 `command`에 필요한 7개의 파라미터 값을 담고 있는 숫자 배열입니다. MAVLink 메시지의 `param1`부터 `param7`까지 순서대로 매핑됩니다. 사용되지 않는 파라미터는 보통 `0` 또는 `null` 값으로 채워집니다.
- `autoContinue`: 해당 아이템을 완료한 후 다음 아이템으로 자동으로 진행할지 여부를 나타내는 불리언(boolean) 값으로, 일반적으로 `true`입니다.
- `doJumpId`: 임무 내에서 각 아이템의 고유한 순차 번호(1부터 시작)입니다. 이 ID는 `MAV_CMD_DO_JUMP`와 같이 임무 흐름을 제어하는 명령에서 특정 아이템을 참조할 때 사용됩니다.

### 2.2  `ComplexItem` 객체: QGroundControl의 고수준 추상화

`ComplexItem`은 QGC가 사용자 편의를 위해 제공하는 고수준의 임무 계획 도구입니다. 사용자가 지도 위에 다각형을 그리면 자동으로 비행 경로를 생성해주는 `Survey`(서베이), `CorridorScan`(코리더 스캔), `StructureScan`(구조물 스캔) 등이 여기에 해당합니다.3

`ComplexItem`의 구조는 그 유형에 따라 다르지만, 공통적으로 다음과 같은 특징을 가집니다. 예를 들어, `StructureScan`의 경우 `complexItemType`, `polygon`, `StructureHeight`, `CameraCalc`와 같은 고유한 키를 포함합니다.

여기서 가장 중요한 점은 `ComplexItem`과 `SimpleItem`의 공생 관계입니다. `ComplexItem` 자체는 비행 경로, 즉 웨이포인트 목록을 직접 포함하지 않습니다. 대신, 웨이포인트들을 *생성하는 데 필요한 메타데이터* (예: 서베이 영역의 폴리곤, 카메라 사양, 이미지 겹침률 등)를 저장합니다. 그리고 이 메타데이터를 기반으로 계산된 실제 웨이포인트들은 `SimpleItem` 객체들로 변환되어 `items` 배열에 함께 저장됩니다.

이 설계의 이유는 호환성에 있습니다. QGC가 아닌 다른 GCS가 이 `.plan` 파일을 열었을 때, `ComplexItem`의 복잡한 구조를 이해하지 못하더라도 `items` 배열에 있는 표준 `SimpleItem` 목록은 해석할 수 있습니다. 덕분에 최소한의 기본 비행 경로는 공유가 가능합니다. 이는 `ComplexItem`을 임무 세그먼트의 '소스 코드'로, 그리고 `SimpleItem`들을 '컴파일된 실행 파일'로 비유할 수 있습니다. 따라서 프로그래밍 방식으로 서베이 임무를 생성하려면, `ComplexItem` 객체를 만드는 것만으로는 부족합니다. 반드시 해당 서베이 패턴에 맞는 `MAV_CMD_NAV_WAYPOINT` `SimpleItem`들을 직접 계산하여 `items` 배열에 추가해주어야 합니다.

### 2.3  `items` 배열: 순서와 임무 흐름

`items` 배열은 전체 임무에 대한 모든 `SimpleItem`들을 포함하는 평탄한(flat) 리스트입니다.3 배열에 나열된 순서가 곧 임무의 기본 실행 순서를 정의합니다. 첫 번째 아이템(인덱스 0)은 보통 홈 위치를 나타내거나(ArduPilot의 경우) 이륙 명령으로 시작됩니다.

각 아이템의 `doJumpId` 필드는 임무의 흐름 제어에 결정적인 역할을 합니다. 예를 들어, `MAV_CMD_DO_JUMP` 커맨드는 `param1`에 특정 `doJumpId`를 지정하여 해당 아이템으로 실행 순서를 건너뛰게 할 수 있습니다. 이를 통해 임무를 반복하거나 특정 조건에 따라 다른 경로를 타는 등의 복잡한 로직 구현이 가능해집니다.

------

## 3.  MAVLink 프로토콜 기반

`.plan` 파일은 결국 MAVLink라는, 드론과 GCS 간의 통신 언어로 번역되어야 합니다. 따라서 안전하고 유효한 `.plan` 파일을 생성하기 위해서는 MAVLink 프로토콜의 핵심 개념, 특히 임무 커맨드(`MAV_CMD`)와 좌표계(`MAV_FRAME`)에 대한 깊은 이해가 필수적입니다.

### 3.1  임무 커맨드 열거형 (`MAV_CMD`): 자동항법장치의 언어

`MAV_CMD`는 기체가 임무 중에 수행할 수 있는 모든 가능한 행동을 정의한 열거형(enumeration)입니다.9

`SimpleItem`의 `command` 필드에 들어가는 정수 값은 바로 이 `MAV_CMD`의 ID에 해당합니다. 예를 들어, `16`은 `MAV_CMD_NAV_WAYPOINT`를, `22`는 `MAV_CMD_NAV_TAKEOFF`를, `21`은 `MAV_CMD_NAV_LAND`를 의미합니다.10

이러한 커맨드들은 단순한 위치 이동뿐만 아니라, 기체의 속도 변경(`MAV_CMD_DO_CHANGE_SPEED`), 관심 지역 설정(`MAV_CMD_DO_SET_ROI`), 카메라 제어(`MAV_CMD_DO_DIGICAM_CONTROL`) 등 매우 다양한 동작을 포함합니다. 정확한 임무 생성을 위해서는 생성하려는 동작에 해당하는 `MAV_CMD` ID를 정확히 사용해야 합니다. 전체 커맨드 목록은 MAVLink 공식 문서에서 확인할 수 있습니다.11

### 3.2  좌표계 (`MAV_FRAME`): 공간적 문맥의 중요성

`MAV_FRAME`은 임무 생성 시 가장 중요하면서도 실수가 잦은 부분 중 하나입니다. `SimpleItem`의 `frame` 필드는 해당 아이템의 `params` 배열에 포함된 모든 공간 관련 파라미터(위도, 경도, 고도)가 어떤 기준에 따라 해석되어야 하는지를 명시합니다.13 잘못된 프레임을 사용하면 기체가 의도치 않은 고도나 위치로 비행하여 심각한 사고로 이어질 수 있습니다.

예를 들어, 해발고도(AMSL) 100m와 지상고도(AGL) 100m는 산악 지형에서 전혀 다른 의미를 가집니다. 또한, GCS마다, 펌웨어마다 특정 프레임을 해석하는 방식에 미묘한 차이가 있을 수 있어 혼란을 야기하기도 합니다.14 프로그래밍 시 혼동을 피하고 안전을 확보하기 위해, 임무 계획에 주로 사용되는 

`MAV_FRAME`의 정의를 명확히 이해해야 합니다.

**표 1: 임무 계획을 위한 주요 `MAV_FRAME` 참조**

| `MAV_FRAME` 열거형              | 값   | 고도 정의                                  | 좌표계   | 일반적인 사용 사례                                           |
| ------------------------------- | ---- | ------------------------------------------ | -------- | ------------------------------------------------------------ |
| `MAV_FRAME_GLOBAL`              | 0    | 평균 해수면 기준 고도 (AMSL)               | WGS84    | 일정한 기압 고도로 비행할 때.                                |
| `MAV_FRAME_GLOBAL_RELATIVE_ALT` | 3    | 홈 위치 기준 상대 고도                     | WGS84    | 동일한 장소에서 이착륙하는 표준 임무. **주의:** `alt=0`은 홈 위치의 고도를 의미함.15 |
| `MAV_FRAME_GLOBAL_TERRAIN_ALT`  | 10   | 지상 기준 고도 (AGL)                       | WGS84    | 지형 추적 임무 (예: 저고도 서베이). 지형 데이터베이스 필요.16 |
| `MAV_FRAME_LOCAL_NED`           | 1    | 로컬 원점 기준 상대 고도 (North-East-Down) | 로컬 NED | 정밀 착륙 또는 시뮬레이션 환경에서 사용. 원점은 보통 부팅 위치.17 |

### 3.3  `MAV_CMD` 파라미터를 `SimpleItem.params` 배열에 매핑하기

`SimpleItem`의 `params` 배열은 MAVLink 커맨드 메시지의 `param1`부터 `param7`까지 7개 필드에 대한 직접적이고 순차적인 매핑입니다. 각 커맨드마다 각 파라미터가 의미하는 바가 모두 다릅니다. 예를 들어, `MAV_CMD_NAV_WAYPOINT`에서 `param1`은 해당 지점에서의 체공 시간(초)을 의미하지만, `MAV_CMD_DO_CHANGE_SPEED`에서 `param1`은 변경할 속도의 유형(대지 속도 또는 대기 속도)을 의미합니다.

개발자가 모든 커맨드의 파라미터 구성을 암기하는 것은 비현실적입니다. 따라서 자주 사용되는 핵심 커맨드들의 파라미터 매핑을 정리해두는 것은 오류를 줄이고 개발 속도를 높이는 데 매우 유용합니다.

**표 2: 주요 `MAV_CMD` 파라미터 매핑**

| `MAV_CMD` ID | 커맨드 이름       | `params` (p1)            | `params` (p2)  | `params` (p3)     | `params` (p4)         | `params` (p5) | `params` (p6) | `params` (p7) |
| ------------ | ----------------- | ------------------------ | -------------- | ----------------- | --------------------- | ------------- | ------------- | ------------- |
| 16           | `NAV_WAYPOINT`    | 체공 시간 (s)            | 수용 반경 (m)  | 통과 반경 (m)     | Yaw (deg)             | 위도          | 경도          | 고도 (m)      |
| 22           | `NAV_TAKEOFF`     | 최소 피치 (deg)          | 0              | 0                 | Yaw (deg)             | 위도          | 경도          | 고도 (m)      |
| 21           | `NAV_LAND`        | 중단 고도 (m)            | 0              | 0                 | Yaw (deg)             | 위도          | 경도          | 고도 (m)      |
| 178          | `DO_CHANGE_SPEED` | 속도 유형(0:대기,1:대지) | 속도 (m/s)     | 스로틀 (%)        | 상대적(0:절대,1:상대) | 0             | 0             | 0             |
| 201          | `DO_SET_ROI`      | ROI 모드                 | 임무 인덱스    | ROI 인덱스        | 0                     | 위도          | 경도          | 고도 (m)      |
| 115          | `CONDITION_YAW`   | 목표 각도 (deg)          | 각속도 (deg/s) | 방향(-1:CCW,1:CW) | 상대적(0:절대,1:상대) | 0             | 0             | 0             |

이 표는 ArduPilot 문서를 기준으로 작성되었으며, PX4에서는 일부 파라미터의 해석이 다를 수 있습니다.10 따라서 대상 펌웨어의 문서를 반드시 참조해야 합니다. 이처럼 MAVLink 프로토콜에 대한 명확한 이해는 

`.plan` 파일의 추상적인 JSON 구조에 생명을 불어넣어, 기체가 의도한 대로 정확하게 움직이게 하는 열쇠입니다.

------

## 4.  Python 구현: 실용 가이드

이론적 기반을 갖추었으니, 이제 Python을 사용하여 `.plan` 파일을 실제로 생성하는 구체적인 방법을 알아볼 차례입니다. 이 장에서는 권장 도구 체인을 소개하고, 단계별 코드 예제를 통해 기본적인 임무 생성 과정을 안내하며, 확장성을 고려한 코드 구조화 방안을 제시합니다.

### 4.1  권장 도구 체인: Python `json` 라이브러리와 `pymavlink`

`.plan` 파일을 생성하는 가장 직접적이고 효율적인 방법은 Python의 내장 `json` 라이브러리를 사용하는 것입니다. 이 라이브러리를 통해 Python 딕셔너리 객체를 손쉽게 JSON 형식의 문자열로 변환하고 파일에 저장할 수 있습니다.

여기서 `pymavlink` 라이브러리의 역할에 대해 명확히 할 필요가 있습니다. 드론 임무 생성을 위해 Python 라이브러리를 찾는 개발자는 자연스럽게 `pymavlink`를 접하게 되고, 그 안에 포함된 `mavwp` 모듈과 `MAVWPLoader` 클래스를 발견할 것입니다.20 언뜻 보면 이 클래스가 임무 파일을 로드하고 저장하는 데 완벽한 도구처럼 보입니다.21 하지만 

`mavwp` 모듈의 소스 코드를 분석해보면, "QGC WPL 110"과 같은 헤더를 파싱하도록 설계되어 있음을 알 수 있습니다.22 이는 구식 텍스트 기반의 웨이포인트 파일 형식이며, 현대의 JSON 기반 

`.plan` 파일과는 호환되지 않습니다.

따라서, **`pymavlink.mavwp`는 최신 `.plan` 파일을 생성하는 데 적합한 도구가 아닙니다.** 이 모듈의 올바른 용도는 MAVLink 프로토콜을 통해 자동항법장치와 직접 통신하여 임무를 업로드/다운로드하거나9, 오래된 형식의 파일을 다룰 때입니다.

최신 `.plan` 파일 생성 작업에서 `pymavlink`의 진정한 가치는 `MAV_CMD`와 `MAV_FRAME`에 대한 권위 있는 정수 상수(constant)를 제공하는 데 있습니다. 코드에 `16`이나 `3`과 같은 '매직 넘버'를 직접 하드코딩하는 대신, `mavutil.mavlink.MAV_CMD_NAV_WAYPOINT`나 `mavutil.mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT`와 같은 명시적인 상수를 사용함으로써 코드의 가독성과 유지보수성을 크게 향상시키고, 잠재적인 오류를 방지할 수 있습니다.

`pymavlink`는 다음 명령어로 간단히 설치할 수 있습니다 20:

Bash

```
pip install pymavlink
```

### 4.2  단계별 기본 임무 계획 생성

다음은 Python을 사용하여 간단한 이륙 후 웨이포인트 비행, 그리고 착륙하는 기본 임무를 담은 `.plan` 파일을 생성하는 전체 코드 예제입니다.

Python

```
import json
from pymavlink.dialects.v20 import common as mavlink

def create_simple_item(command, frame, params, seq):
    """SimpleItem 딕셔너리를 생성하는 헬퍼 함수"""
    return {
        "type": "SimpleItem",
        "command": command,
        "frame": frame,
        "params": params,
        "autoContinue": True,
        "doJumpId": seq
    }

def generate_basic_mission():
    """기본적인 비행 계획을 생성하고 파일로 저장합니다."""
    
    # 1. 최상위.plan 구조 정의
    plan_dict = {
        "fileType": "Plan",
        "geoFence": {"circles":, "polygons":, "version": 2},
        "groundStation": "PythonScript",
        "mission": {
            "cruiseSpeed": 15,
            "firmwareType": 3,  # ArduPilot
            "hoverSpeed": 5,
            "items":,
            "plannedHomePosition": [47.6366, -122.1419, 25.0],
            "vehicleType": 2,  # Multirotor
            "version": 2
        },
        "rallyPoints": {"points":, "version": 2},
        "version": 1
    }

    mission_items =
    seq_counter = 1

    # 2. 홈 위치 설정 (ArduPilot에서는 아이템 0으로 사용될 수 있음)
    # QGC는 plannedHomePosition을 사용하므로, items은 실제 명령으로 시작.
    # 참고: QGC는 plannedHomePosition을 기준으로 첫 번째 아이템을 0으로 간주함.
    # doJumpId는 1부터 시작하는 것이 명확함.
    home_pos = plan_dict["mission"]["plannedHomePosition"]

    # 3. 임무 아이템 생성
    # Item 1: 이륙 (Takeoff)
    takeoff_alt = 30.0
    takeoff_item = create_simple_item(
        mavlink.MAV_CMD_NAV_TAKEOFF,
        mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT,
        [0, 0, 0, 0, home_pos, home_pos, takeoff_alt],
        seq_counter
    )
    mission_items.append(takeoff_item)
    seq_counter += 1

    # Item 2: 웨이포인트 1
    wp1_item = create_simple_item(
        mavlink.MAV_CMD_NAV_WAYPOINT,
        mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT,
        [0, 0, 0, 0, 47.6375, -122.1425, takeoff_alt],
        seq_counter
    )
    mission_items.append(wp1_item)
    seq_counter += 1

    # Item 3: 웨이포인트 2
    wp2_item = create_simple_item(
        mavlink.MAV_CMD_NAV_WAYPOINT,
        mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT,
        [0, 0, 0, 0, 47.6360, -122.1430, takeoff_alt],
        seq_counter
    )
    mission_items.append(wp2_item)
    seq_counter += 1

    # Item 4: 착륙 (Land)
    land_item = create_simple_item(
        mavlink.MAV_CMD_NAV_LAND,
        mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT,
        [0, 0, 0, 0, home_pos, home_pos,
        seq_counter
    )
    mission_items.append(land_item)

    # 4. 생성된 아이템들을 mission 객체에 추가
    plan_dict["mission"]["items"] = mission_items

    # 5. 딕셔너리를 JSON 파일로 저장
    file_path = "basic_mission.plan"
    with open(file_path, 'w') as f:
        json.dump(plan_dict, f, indent=4)
        
    print(f"'{file_path}' 파일이 성공적으로 생성되었습니다.")

if __name__ == "__main__":
    generate_basic_mission()
```

### 4.3  재사용성과 확장성을 위한 코드 구조화

단순한 임무를 넘어 복잡한 비행 계획을 생성하기 시작하면, 위와 같은 절차적 코드는 금방 관리하기 어려워집니다. QGC의 내부 코드 구조가 그러하듯, 객체 지향 접근 방식을 채택하는 것이 바람직합니다.26

`PlanGenerator`와 같은 클래스를 만들어 임무 생성 로직을 캡슐화하면 코드의 재사용성과 확장성을 크게 높일 수 있습니다.

Python

```
class PlanGenerator:
    def __init__(self, ground_station="PythonClass", firmware_type=mavlink.MAV_AUTOPILOT_ARDUPILOTMEGA, vehicle_type=2):
        self.plan = {
            "fileType": "Plan",
            "groundStation": ground_station,
            "mission": {
                "items":,
                "firmwareType": firmware_type,
                "vehicleType": vehicle_type,
                "version": 2
            },
            "version": 1
        }
        self.seq_counter = 1

    def set_home_position(self, lat, lon, alt):
        self.plan["mission"]["plannedHomePosition"] = [lat, lon, alt]

    def add_waypoint(self, lat, lon, alt, hold_time=0):
        item = create_simple_item(
            mavlink.MAV_CMD_NAV_WAYPOINT,
            mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT,
            [hold_time, 0, 0, 0, lat, lon, alt],
            self.seq_counter
        )
        self.plan["mission"]["items"].append(item)
        self.seq_counter += 1
    
    #... 다른 아이템 추가 메서드들 (add_takeoff, add_land 등)...

    def save(self, filename="mission.plan"):
        if "plannedHomePosition" not in self.plan["mission"]:
            raise ValueError("홈 위치가 설정되지 않았습니다. set_home_position()을 호출해주세요.")
        
        with open(filename, 'w') as f:
            json.dump(self.plan, f, indent=4)
        print(f"'{filename}' 파일이 성공적으로 생성되었습니다.")

# 사용 예시
# planner = PlanGenerator()
# planner.set_home_position(47.6366, -122.1419, 25.0)
# planner.add_waypoint(47.6375, -122.1425, 30.0)
#...
# planner.save("class_based_mission.plan")
```

이러한 클래스 기반 접근 방식은 임무 생성 과정을 더욱 구조화하고, 복잡한 로직을 각 메서드에 분리하여 관리하기 용이하게 만듭니다.

------

## 5.  고급 프로그래밍 방식 임무 생성

기본적인 임무 생성 방법을 익혔다면, 이제 이를 응용하여 GIS 데이터 연동, 동적 패턴 생성 등 보다 현실적이고 복잡한 요구사항을 해결할 수 있습니다. 이 장에서는 서베이(격자 비행) 패턴을 동적으로 생성하는 사례를 통해 고급 임무 생성 기법을 심도 있게 다룹니다.

### 5.1  사례 연구: 동적 서베이 (격자) 패턴 생성

항공 매핑 및 정밀 농업 분야에서는 특정 영역을 꼼꼼하게 촬영하기 위한 격자 비행 패턴이 필수적입니다.28 사용자가 정의한 폴리곤 영역과 카메라 사양, 촬영 조건에 맞춰 비행 경로를 동적으로 생성하는 것은 프로그래밍 방식 임무 생성의 가장 강력한 활용 사례 중 하나입니다.

이 과정은 2.2절에서 설명한 `ComplexItem`과 `SimpleItem`의 관계를 실제로 구현하는 것입니다. 즉, 서베이에 대한 메타데이터를 담는 `ComplexItem`과, 실제 비행 경로인 `SimpleItem` 웨이포인트들을 모두 생성해야 합니다.

다음은 주어진 폴리곤 경계와 서베이 파라미터를 기반으로 서베이 임무 `.plan` 파일을 생성하는 Python 스크립트의 개념적 로직입니다. (참고: 실제 구현에는 폴리곤 내부의 격자점 계산을 위한 기하학 라이브러리, 예: `shapely`, `pyproj`가 필요할 수 있습니다.)

**개념적 구현 단계:**

1. **입력 파라미터 정의**:
   - `boundary_polygon`: `[[lat1, lon1], [lat2, lon2],...]` 형태의 폴리곤 꼭짓점 리스트.
   - `altitude`: 비행 고도 (AGL).
   - `overlap`: 이미지 겹침률 (예: 70%는 `0.7`).
   - `sidelap`: 비행 경로 간 겹침률 (예: 60%는 `0.6`).
   - `angle`: 격자 비행 경로의 각도.
   - `camera_spec`: 카메라 센서 너비, 초점 거리 등.
2. **비행 경로 계산**:
   - 카메라 사양과 고도를 이용해 지상에서의 이미지 footprint 크기를 계산합니다.
   - `overlap`과 `sidelap`을 고려하여 사진 촬영 지점 간 거리(spacing)와 비행 라인 간 거리(line spacing)를 계산합니다.
   - `boundary_polygon`과 `angle`을 이용해 전체 서베이 영역을 감싸는 최소 경계 사각형(minimum bounding rectangle)을 계산합니다.
   - 계산된 사각형과 라인 간격을 바탕으로, 지그재그 형태의 비행 경로(웨이포인트 목록)를 생성합니다.
3. **`.plan` 파일 객체 생성**:
   - 4장에서 구현한 `PlanGenerator`와 같은 클래스를 사용하여 기본 `.plan` 구조를 초기화합니다.
   - 계산된 웨이포인트 목록을 순회하며 `add_waypoint`와 같은 메서드를 호출하여 `SimpleItem`들을 `items` 배열에 추가합니다.
   - 이때, 카메라 촬영을 위해 각 웨이포인트 사이에 `MAV_CMD_DO_SET_CAM_TRIGG_DIST` 커맨드를 추가하여 일정 거리마다 사진을 찍도록 설정하거나, 웨이포인트 도착 시 `MAV_CMD_IMAGE_START_CAPTURE` 등의 커맨드를 사용할 수 있습니다.
4. **`ComplexItem` (Survey) 생성 및 추가**:
   - 서베이 메타데이터를 담을 `ComplexItem` 딕셔너리를 생성합니다.
   - 이 객체에는 `complexItemType: "Survey"`, `polygon` (입력받은 폴리곤), `grid` (계산된 격자 정보), `CameraCalc` (카메라 및 촬영 설정) 등의 필드가 포함됩니다.
   - 생성된 `ComplexItem`을 `items` 배열의 시작 부분에 추가합니다. QGC는 이를 보고 후속 `SimpleItem`들이 서베이의 일부임을 인지하고 시각적으로 표현해줍니다.
5. **파일 저장**: 최종적으로 완성된 딕셔너리를 `.plan` 파일로 저장합니다.

이러한 동적 생성 방식은 수동 계획으로는 불가능한 수준의 정밀도와 효율성을 제공하며, 다양한 현장 조건에 맞춰 즉석에서 최적의 비행 계획을 수립할 수 있게 합니다.

### 5.2  지오펜스 및 랠리 포인트 구현

프로그래밍 방식으로 지오펜스와 랠리 포인트를 추가하는 것은 간단합니다. `PlanGenerator` 클래스에 다음과 같은 메서드를 추가하여 구현할 수 있습니다.

Python

```
# PlanGenerator 클래스 내부에 추가될 메서드 예시

def set_geofence(self, polygon_points):
    """
    다각형 지오펜스를 설정합니다.
    polygon_points: [[lat1, lon1], [lat2, lon2],...] 형태의 리스트
    """
    fence_obj = {
        "version": 2,
        "circles":,
        "polygons": [polygon_points]
    }
    self.plan["geoFence"] = fence_obj

def add_rally_point(self, lat, lon, alt):
    """랠리 포인트를 추가합니다."""
    if "rallyPoints" not in self.plan:
        self.plan["rallyPoints"] = {"version": 2, "points":}
    
    self.plan["rallyPoints"]["points"].append([lat, lon, alt])
```

이 메서드들을 사용하면 비행 계획 생성 시 안전 경계와 비상 복귀 지점을 코드 레벨에서 체계적으로 관리할 수 있습니다.

### 5.3  펌웨어별 동작 처리

1.3절에서 언급했듯이, ArduPilot과 PX4는 일부 커맨드나 파라미터를 다르게 처리할 수 있습니다. QGC는 내부적으로 `Mission Command Tree`라는 JSON 메타데이터 계층 구조를 사용하여 이러한 펌웨어/기체별 차이를 처리합니다.27

우리의 `PlanGenerator` 클래스도 이러한 개념을 적용할 수 있습니다. 생성자에서 `firmware_type`을 인자로 받고, 특정 커맨드를 추가하는 메서드 내에서 조건부 로직을 사용하는 것입니다.

Python

```
# PlanGenerator 클래스 내 메서드 예시

def add_takeoff(self, lat, lon, alt):
    # 펌웨어 유형에 따라 이륙 커맨드의 파라미터를 다르게 설정
    if self.plan["mission"] == mavlink.MAV_AUTOPILOT_ARDUPILOTMEGA:
        # ArduCopter는 param 1~4를 사용하지 않음
        params = [0, 0, 0, 0, lat, lon, alt]
    elif self.plan["mission"] == mavlink.MAV_AUTOPILOT_PX4:
        # PX4는 param 1(최소 피치)을 사용할 수 있음
        params = [15.0, 0, 0, 0, lat, lon, alt]
    else:
        # 기본값
        params = [0, 0, 0, 0, lat, lon, alt]
        
    item = create_simple_item(
        mavlink.MAV_CMD_NAV_TAKEOFF,
        mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT,
        params,
        self.seq_counter
    )
    self.plan["mission"]["items"].append(item)
    self.seq_counter += 1
```

이러한 접근 방식은 생성되는 `.plan` 파일이 특정 대상 펌웨어에 최적화되도록 보장하여 호환성 문제를 예방하고 임무의 신뢰성을 높입니다.

------

## 6.  검증, 문제 해결 및 최상의 실천 방안

프로그래밍 방식으로 생성된 `.plan` 파일은 코드의 버그나 논리적 오류로 인해 잠재적인 위험을 내포할 수 있습니다. 따라서 실제 기체에 적용하기 전에 철저한 검증 과정을 거치는 것이 무엇보다 중요합니다. 이 장에서는 생성된 파일의 유효성을 확인하는 방법, 발생할 수 있는 일반적인 문제들과 해결 방안, 그리고 안전한 자동화 임무 생성을 위한 최상의 실천 방안을 제시합니다.

### 6.1  생성된 `.plan` 파일 검증 방법

1. **QGroundControl에서의 시각적 검증**: 생성된 `.plan` 파일을 검증하는 가장 기본적이고 효과적인 첫 단계는 QGC에 직접 로드해보는 것입니다.7 QGC의 

   `Plan View`에서 임무를 열었을 때, 의도한 대로 비행 경로, 고도, 임무 아이템들이 지도 위에 시각적으로 올바르게 표시되는지 확인해야 합니다. 웨이포인트가 엉뚱한 곳에 찍히거나, 고도 그래프가 비정상적이거나, QGC가 오류 메시지를 표시한다면 파일 생성 로직에 문제가 있다는 명백한 신호입니다.

2. **SITL (Software In The Loop) 시뮬레이션**: 시각적 검증을 통과했다면, 다음 단계는 시뮬레이션을 통해 임무를 가상으로 비행해보는 것입니다. PX4나 ArduPilot은 SITL 환경을 제공하여 실제 하드웨어 없이 컴퓨터 상에서 펌웨어를 실행하고 GCS와 연동할 수 있게 합니다.24 생성된 

   `.plan` 파일을 SITL 기체에 업로드하고 임무를 실행시켜보면, 기체가 각 웨이포인트를 정상적으로 따라가는지, 속도나 고도 변경 등의 명령이 올바르게 수행되는지, 예상치 못한 동작은 없는지 등을 실제 비행과 거의 동일한 환경에서 안전하게 테스트할 수 있습니다.

### 6.2  일반적인 문제 및 해결 방안 (Troubleshooting)

프로그래밍 방식의 생성 과정에서 흔히 발생하는 오류들은 다음과 같습니다.

- **부정확한 좌표계 (`MAV_FRAME`) 사용**: 가장 위험한 오류입니다. 상대 고도(`MAV_FRAME_GLOBAL_RELATIVE_ALT`)를 사용해야 할 곳에 절대 고도(`MAV_FRAME_GLOBAL`)를 사용하거나 그 반대의 경우, 기체는 예상과 전혀 다른 고도로 비행하게 됩니다. 3.2절의 참조표를 다시 확인하고, 각 웨이포인트의 고도가 어떤 기준(해수면, 홈 위치, 지표면)으로 설정되어야 하는지 명확히 해야 합니다.
- **파라미터 불일치**: `MAV_CMD`에 맞지 않는 파라미터를 사용한 경우입니다. 예를 들어 `MAV_CMD_NAV_WAYPOINT`의 `params` 배열에 위도, 경도 대신 다른 값을 넣거나, 7개가 아닌 다른 개수의 파라미터를 제공하는 경우입니다. 3.3절의 파라미터 매핑표와 MAVLink 공식 문서를 참조하여 각 커맨드에 맞는 정확한 파라미터 구조를 사용해야 합니다.
- **유효하지 않은 JSON 형식**: 괄호나 쉼표 누락 등 단순한 구문 오류로 인해 파일 전체를 읽지 못하는 경우입니다. Python의 `json.dump()`를 사용하면 대부분의 구문 오류는 방지되지만, 딕셔너리 구조 자체의 오류는 발생할 수 있습니다. JSON 유효성 검사기(linter/validator)를 사용하거나, 코드를 실행하기 전에 생성될 딕셔너리 구조를 점검하는 것이 좋습니다.
- **좌표 순서 오류**: `.plan` 파일의 좌표 배열은 `[latitude, longitude, altitude]` 순서를 따릅니다. 많은 GIS 시스템이나 다른 라이브러리에서는 `[longitude, latitude]` 순서를 사용하는 경우가 많아 혼동하기 쉽습니다. 좌표를 다룰 때는 항상 순서를 재확인해야 합니다.
- **펌웨어 불일치**: ArduPilot용으로 생성된 계획을 PX4 기체에 로드하는 경우처럼, `firmwareType`과 실제 기체의 펌웨어가 일치하지 않을 때 예기치 않은 동작이 발생할 수 있습니다. 생성 스크립트에서 대상 펌웨어를 명확히 지정하고, 필요한 경우 5.3절에서 설명한 펌웨어별 분기 로직을 구현해야 합니다.

### 6.3  안전과 신뢰성을 위한 최상의 실천 방안

- **필수 메타데이터 명시**: 항상 `plannedHomePosition`, `firmwareType`, `vehicleType`을 명시적으로 설정하십시오. 이는 QGC가 임무를 정확하게 해석하고 시뮬레이션하는 데 필수적인 정보입니다.
- **매직 넘버 사용 금지**: `command`나 `frame` ID에 `16`, `3`과 같은 정수 리터럴을 직접 사용하지 마십시오. 반드시 `pymavlink.mavutil.mavlink`에서 제공하는 `MAV_CMD_*`, `MAV_FRAME_*` 상수를 사용하여 코드의 가독성을 높이고 실수를 방지해야 합니다.
- **자체 유효성 검사 구현**: 생성 코드 내에 기본적인 제정신 검사(sanity check) 로직을 포함시키십시오. 예를 들어, 생성된 웨이포인트의 고도가 음수는 아닌지, 연속된 웨이포인트 간의 거리가 비정상적으로 멀지는 않은지, 서베이 영역이 홈 위치에서 수백 km 떨어져 있지는 않은지 등을 검사하는 로직을 추가하면 잠재적인 오류를 조기에 발견할 수 있습니다.
- **점진적 개발**: 처음부터 복잡한 임무를 생성하려 하지 말고, 이륙-착륙과 같은 가장 단순한 임무부터 시작하여 웨이포인트 추가, 속도 변경 등 점진적으로 기능을 확장해나가십시오. 각 단계마다 QGC와 SITL을 통해 철저히 검증하는 것이 중요합니다.
- **실 비행 전 최종 검증**: 어떤 경우에도, 프로그래밍 방식으로 생성된 임무를 시뮬레이션 검증 없이 실제 기체에 바로 적용해서는 안 됩니다. 실 비행 직전에는 항상 QGC에 최종 파일을 로드하여 숙련된 조종사가 전체 계획을 검토하는 절차를 거쳐야 합니다.

------

## 7. 결론: 자동화된 임무 계획의 미래

이 보고서는 Python을 사용하여 QGroundControl용 `.plan` 파일을 프로그래밍 방식으로 생성하는 최신 방법론을 심층적으로 분석하고 제시했습니다. 핵심적인 방법론은 **MAVLink 프로토콜 사양에 대한 명확한 이해를 바탕으로, Python의 표준 `json` 라이브러리를 사용하여 `.plan` 파일의 JSON 구조를 직접 구성하는 것**입니다. 우리는 `pymavlink`의 `mavwp` 모듈이 최신 JSON 형식 생성에는 적합하지 않으며, 대신 MAVLink 상수 제공자로서의 역할이 중요함을 밝혔습니다.

`.plan` 파일의 해부학적 구조부터 `SimpleItem`과 `ComplexItem`의 관계, 그리고 그 근간을 이루는 `MAV_CMD`와 `MAV_FRAME`의 원리까지 단계적으로 분석함으로써, 개발자들은 이제 단순한 웨이포인트 나열을 넘어 복잡하고 동적인 임무를 안전하고 신뢰성 있게 생성할 수 있는 견고한 이론적, 실용적 기반을 갖추게 되었습니다.

이러한 프로그래밍 방식의 임무 생성 기술은 단순히 편의성을 높이는 차원을 넘어섭니다. 이는 임무 계획을 정적이고 수동적인 작업에서 동적이고 확장 가능하며 다른 시스템과 통합 가능한 소프트웨어 컴포넌트로 전환시키는 패러다임의 변화를 의미합니다.

이 기술은 차세대 자율 시스템을 구현하는 핵심적인 동력원(enabler)입니다. 예를 들어, 위성 이미지 분석을 통해 실시간으로 병충해 지역을 파악하고, 해당 지역만을 정밀하게 방제하는 서베이 임무를 즉석에서 생성하는 AI 기반 정밀 농업 시스템을 상상해볼 수 있습니다. 또한, 재난 현장에서 구조대의 피드백에 따라 수색 패턴을 동적으로 수정하며 최적의 경로를 탐색하는 지능형 수색 및 구조 드론, 또는 물류 센터의 수요에 맞춰 수백 대의 드론에게 각기 다른 배송 경로를 할당하고 관제하는 대규모 드론 플릿 관리 시스템 등은 모두 수동 임무 계획으로는 실현이 불가능합니다.

프로그래밍 방식의 임무 생성은 바로 이러한 고수준의 지능(AI, 데이터 분석, 최적화 알고리즘)과 저수준의 기체 제어(자동항법장치)를 연결하는 핵심적인 API 역할을 수행합니다. GIS 및 고급 애플리케이션과의 연동 사례들은 이러한 추세의 시작에 불과합니다.32 본 보고서에서 제시된 원리와 기법들은 개발자와 연구자들이 이러한 미래 자율 시스템의 문을 여는 열쇠를 제공할 것입니다. 결국, 비행을 코드로 정의하는 능력은 무인 항공 기술의 잠재력을 최대한으로 이끌어내고, 진정한 의미의 자율성을 실현하는 길입니다.

#### **참고 자료**

1. mavlink/qgroundcontrol: Cross-platform ground control station for drones (Android, iOS, Mac OS, Linux, Windows) - GitHub, accessed July 12, 2025, https://github.com/mavlink/qgroundcontrol
2. QGroundControl – Drone Control – Ground Control Station for Small Air – Land – Water Autonomous Unmanned Systems, accessed July 12, 2025, https://qgroundcontrol.com/
3. Plan File Format | QGC Guide (master) - QGroundControl Guide, accessed July 12, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/file_formats/plan.html
4. Mission: JSON file format · Issue #3800 · mavlink/qgroundcontrol ..., accessed July 12, 2025, https://github.com/mavlink/qgroundcontrol/issues/3800
5. Best way to convert mission from qgc json format to txt - ArduPilot Discourse, accessed July 12, 2025, https://discuss.ardupilot.org/t/best-way-to-convert-mission-from-qgc-json-format-to-txt/69347
6. QGC Plan Format Example · GitHub, accessed July 12, 2025, https://gist.github.com/SamuelDudley/c6a5e1aa5a49ce2b1690b980fc248563
7. Plan View | QGC Guide (4.3) - QGroundControl Guide, accessed July 12, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/plan_view/plan_view.html
8. Missions | PX4 Guide (main), accessed July 12, 2025, https://docs.px4.io/main/en/flying/missions.html
9. Mission Protocol - MAVLink Guide, accessed July 12, 2025, https://mavlink.io/en/services/mission.html
10. Mission Commands — Copter documentation - ArduPilot, accessed July 12, 2025, https://ardupilot.org/copter/docs/common-mavlink-mission-command-messages-mav_cmd.html
11. MavCmd in mavlink::common - Rust - Docs.rs, accessed July 12, 2025, https://docs.rs/mavlink/latest/mavlink/common/enum.MavCmd.html
12. MAVLINK Common Message Set (common.xml), accessed July 12, 2025, https://mavlink.io/en/messages/common.html
13. Command Protocol - MAVLink Guide, accessed July 12, 2025, https://mavlink.io/en/services/command.html
14. Wrong coordinate frame in waypoint files? · Issue #953 · ArduPilot/MissionPlanner - GitHub, accessed July 12, 2025, https://github.com/ArduPilot/MissionPlanner/issues/953
15. Missions with MAV_FRAME_GLOBAL_RELATIVE_ALT ignore alt=0 - ArduPilot Discourse, accessed July 12, 2025, https://discuss.ardupilot.org/t/missions-with-mav-frame-global-relative-alt-ignore-alt-0/66791
16. mavros_msgs/SetMavFrame Documentation - the official ROS docs, accessed July 12, 2025, http://docs.ros.org/en/kinetic/api/mavros_msgs/html/srv/SetMavFrame.html
17. MavFrame in mavlink::common - Rust - Docs.rs, accessed July 12, 2025, https://docs.rs/mavlink/latest/mavlink/common/enum.MavFrame.html
18. How does MAV_FRAME_LOCAL_NED work? - Robotics Stack Exchange, accessed July 12, 2025, https://robotics.stackexchange.com/questions/80742/how-does-mav-frame-local-ned-work
19. Mission Commands — Plane documentation - ArduPilot, accessed July 12, 2025, https://ardupilot.org/plane/docs/common-mavlink-mission-command-messages-mav_cmd.html
20. Using Pymavlink Libraries (mavgen) - MAVLink Guide, accessed July 12, 2025, https://mavlink.io/en/mavgen_python/
21. mavlink: pymavlink.mavwp.MAVWPLoader Class Reference, accessed July 12, 2025, http://docs.ros.org/jade/api/mavlink/html/classpymavlink_1_1mavwp_1_1MAVWPLoader.html
22. pymavlink/mavwp.py at master - GitHub, accessed July 12, 2025, https://github.com/ArduPilot/pymavlink/blob/master/mavwp.py
23. pymavlink/mavwp.py at master - GitHub, accessed July 12, 2025, https://github.com/meee1/pymavlink/blob/master/mavwp.py
24. How to implement mission load through pymavlink - Blue Robotics Community Forums, accessed July 12, 2025, https://discuss.bluerobotics.com/t/how-to-implement-mission-load-through-pymavlink/8910
25. ArduPilot/pymavlink: python MAVLink interface and utilities - GitHub, accessed July 12, 2025, https://github.com/ArduPilot/pymavlink
26. Vehicle.cc - mavlink/qgroundcontrol - GitHub, accessed July 12, 2025, https://github.com/mavlink/qgroundcontrol/blob/master/src/Vehicle/Vehicle.cc
27. Mission Command Tree | QGC Guide (4.3), accessed July 12, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-dev-guide/plan/mission_command_tree.html
28. UAV Mapping Path Generator (for Litchi) — QGIS Python Plugins Repository, accessed July 12, 2025, https://plugins.qgis.org/plugins/drone_path/
29. Auto grid & drawing polygon for survey mission in python - ArduPilot Discourse, accessed July 12, 2025, https://discuss.ardupilot.org/t/auto-grid-drawing-polygon-for-survey-mission-in-python/80941
30. Mission Command Tree | QGC Guide (master), accessed July 12, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/plan/mission_command_tree.html
31. offDaCharts/qgroundcontrol: qgroundcontrol software source code - GitHub, accessed July 12, 2025, https://github.com/offDaCharts/qgroundcontrol
32. UCANR-IGIS/ParseDroneImgs: Python script to parse a folder of images from a drone mapping mission into separate flights, & make point Shapefile of image locations - GitHub, accessed July 12, 2025, https://github.com/UCANR-IGIS/ParseDroneImgs
33. WIP Python tool for mapping water surfaces with aerial images : r/UAVmapping - Reddit, accessed July 12, 2025, https://www.reddit.com/r/UAVmapping/comments/1g821s1/wip_python_tool_for_mapping_water_surfaces_with/


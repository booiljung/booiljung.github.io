# QGroundControl을 위한 Plan 파일 생성 방법

## 1.  QGroundControl 미션 계획의 기초 원리

본 섹션에서는 QGroundControl(QGC)이 지상관제국(GCS)으로서 가지는 근본적인 역할과 미션 계획 기능의 기본 원칙을 다룹니다.

### 1.1  MAVLink 기반 GCS로서의 QGroundControl 소개

QGroundControl(QGC)은 MAVLink 프로토콜을 사용하는 모든 드론을 위한 직관적이고 강력한 지상관제국(GCS)으로, 전문 사용자와 개발자 모두를 위해 설계되었습니다.1 QGC의 핵심 기능은 MAVLink가 활성화된 기체에 대한 완전한 비행 제어, 설정 및 자율 비행 미션 계획을 제공하는 것입니다.2

QGC의 핵심적인 아키텍처 철학 중 하나는 폭넓은 호환성입니다. 이는 PX4 Pro와 ArduPilot 같은 여러 비행 스택(Flight Stacks), 콥터, 비행기, 로버, 잠수정 등 다양한 기체 유형, 그리고 Windows, macOS, Linux, iOS, Android 등 광범위한 운영 체제를 지원하는 것에서 잘 드러납니다.3 이러한 크로스 플랫폼 및 크로스 펌웨어 지원은 QGC의 중심 설계 사상입니다.

### 1.2  자율 운영에서 미션 계획의 역할

미션 계획은 기체의 비행 컨트롤러에 업로드되어 실행될 사전 정의된 자율 비행 임무, 즉 '비행 계획(flight plan)'을 생성하는 과정입니다.6 효과적인 미션 계획은 단순한 수동 조종을 넘어, 항공 측량, 인프라 점검, 3D 모델링과 같은 복잡하고 반복적이며 정밀한 작업을 가능하게 하는 데 필수적입니다.8 이는 즉흥적인 비행에 비해 위험을 완화하고 효율성을 높입니다.10

### 1.3  미션 기능에 대한 비행 스택(PX4 vs. ArduPilot)의 영향

QGC는 통합된 인터페이스를 제공하지만, 사용 가능한 미션 명령어와 기능은 기체에서 실행되는 펌웨어에 따라 달라집니다.6 예를 들어, 'Structure Scan'과 같은 특정 고급 패턴은 PX4에서는 지원되지만 ArduPilot에서는 아직 지원되지 않는 것으로 명시되어 있습니다.9 QGC는 기체에 연결되었을 때 펌웨어와 기체 유형을 동적으로 파악하여 사용 가능한 미션 명령어 옵션을 그에 맞게 조정합니다. 연결되지 않은 상태에서는 사용자가 이러한 매개변수를 수동으로 지정하여 정확한 오프라인 미션 계획을 수립할 수 있습니다.6

이러한 아키텍처 분석을 통해 QGC의 근본적인 역할이 단순한 지상관제국을 넘어선다는 점을 알 수 있습니다. QGC는 PX4와 ArduPilot를 모두 지원하고 연결된 기체의 펌웨어에 따라 인터페이스를 동적으로 조정함으로써, 사실상 MAVLink 생태계를 위한 보편적인 '추상화 계층(abstraction layer)'으로 기능합니다. 이 설계 철학은 '미션 명령어 트리(Mission Command Tree)'에서 명확하게 드러나는데, 이 트리는 일반적인 MAVLink 명령어 세트에 펌웨어별 및 기체별 재정의(override)를 체계적으로 적용합니다.11 이러한 접근 방식은 운영자가 각기 다른 스택에 대한 개별 GCS를 마스터할 필요 없이, 단일하고 일관된 인터페이스를 통해 다양한 기체들을 관리할 수 있게 해줍니다. 결과적으로 이는 훈련 장벽을 낮추고 운영 유연성을 높여, ArduPilot에 주로 초점을 맞춘 Mission Planner와 같은 펌웨어 특정 GCS 대안에 비해 확장성이 뛰어난 솔루션으로 자리매김하게 합니다.5

### 1.4  미션의 핵심 구성 요소: 웨이포인트, 액션, 미션 아이템

미션은 근본적으로 '미션 아이템(mission items)'의 순차적인 목록입니다.6 이 아이템들은 지리적 좌표와 고도를 가진 공간적 요소인 '웨이포인트(Waypoint)'일 수도 있고, 이미지 촬영, 속도 변경, 착륙과 같은 비공간적 요소인 '액션(Actions)'일 수도 있습니다.6 각 아이템은 QGC와 기체 간의 통신에 사용되는 기본 언어인 MAVLink 명령어에 해당합니다.1

## 2.  Plan View: 미션 생성 인터페이스 종합 분석

본 섹션에서는 미션 생성을 위한 기본 사용자 인터페이스를 해부하고 각 구성 요소의 기능과 상호 작용을 설명합니다.

### 2.1  Plan View: 목적 및 개괄적 워크플로우

Plan View는 QGC 내에서 자율 비행 미션을 계획하고 이를 기체에 업로드하기 위한 전용 작업 공간입니다. 계획이 완료되면 사용자는 Fly View로 전환하여 미션을 실행합니다.6 또한 이 뷰는 지오펜스(GeoFence) 및 랠리 포인트(Rally Points)와 같은 안전 기능을 구성하는 데에도 사용됩니다.6 전체적인 워크플로우는 다음과 같습니다: 1) Plan View로 전환, 2) 미션 명령어 추가 및 편집, 3) 기체에 업로드, 4) Fly View로 전환하여 실행.6

### 2.2  UI 구성 요소 상세 분석

Plan View UI는 사용자가 두 가지 수준에서 미션과 상호 작용할 수 있도록 설계되었습니다. 하나는 지도 위에서 웨이포인트를 드래그하는 것과 같은 직관적인 공간적 수준이고, 다른 하나는 미션 명령어 목록에서 매개변수를 편집하는 것과 같은 정밀한 수치적 수준입니다. 이 '이중 모드 편집(dual-modality editing)' 방식은 사용자가 개략적인 기하학적 계획과 세부적인 명령어 튜닝 사이를 원활하게 전환할 수 있게 해줍니다. 두 방식 모두 단일한 통합 미션 모델을 업데이트하며, 이는 신속한 프로토타이핑과 정밀한 미션 미세 조정을 모두 지원합니다.

- **Map:** 웨이포인트, 계획된 홈(Planned Home), 비행 경로에 대한 번호가 매겨진 표시기를 보여주는 중심 시각 요소입니다. 지도상의 항목들은 상호작용이 가능하여 드래그 앤 드롭으로 위치를 재조정할 수 있습니다.6
- **Plan Toolbar:** 선택된 웨이포인트에 대한 실시간 상태 정보(예: 이전 지점과의 거리)와 전체 미션에 대한 통계(총 거리, 예상 시간)를 제공합니다. 기체가 연결되면 중요한 'Upload' 버튼도 여기에 표시됩니다.6
- **Plan Tools:** 미션을 생성하고 관리하기 위한 도구 세트입니다. 웨이포인트 추가, 복잡한 패턴 생성, 그리고 계획을 저장, 로드, 업로드, 다운로드하기 위한 파일 작업(Sync)이 포함됩니다.6
- **Mission Command List:** 뷰의 오른쪽에 있는 수직 목록으로, 모든 미션 아이템을 순서대로 표시합니다. 여기서 아이템을 선택하면 해당 매개변수를 상세하게 편집할 수 있습니다.6
- **Terrain Altitude Overlay:** 각 미션 명령어의 상대 고도를 지형 위에 시각적으로 보여주어 안전한 지형 회피를 보장하는 데 중요한 시각 보조 도구입니다.6

### 2.3  "Planned Home Position": 오프라인 계획을 위한 핵심 개념

계획된 홈(Planned Home, 지도에 H로 표시)은 기체가 연결되지 않았을 때 QGC가 사용하는 시뮬레이션된 시작점입니다.6 그 목적은 순전히 Plan View 내에서의 추정 및 시각화를 위한 것입니다. 이는 미션 시간을 계산하고 궤적선을 그리는 데 사용되며, 기체 자체의 실제 홈 위치(기체가 시동을 걸 때 설정됨)에는 영향을 미치지 않습니다.18 사용자는 정확한 사전 비행 분석을 위해 계획된 홈을 대략적인 이륙 위치로 드래그해야 합니다.6

#### 2.3.1 표 2.1: Plan View UI 구성 요소의 기능적 분석

| UI 구성 요소                 | 주요 기능                           | 핵심 사용자 액션                                  | 관련 자료 |
| ---------------------------- | ----------------------------------- | ------------------------------------------------- | --------- |
| **Map**                      | 미션 경로의 시각적 표현             | 웨이포인트 드래그, 경로 확인                      | 6         |
| **Plan Toolbar**             | 실시간 미션 통계 및 상태 정보 제공  | 총 비행 시간 확인, 미션 업로드                    | 6         |
| **Plan Tools**               | 미션 생성 및 관리를 위한 도구 모음  | 웨이포인트 추가, Survey 패턴 생성, 파일 저장/로드 | 6         |
| **Mission Command List**     | 상세한 아이템 편집을 위한 순차 목록 | 웨이포인트 고도 변경, 카메라 액션 설정            | 6         |
| **Terrain Altitude Overlay** | 지형 대비 고도 시각화               | 지형 충돌 위험 확인                               | 6         |

## 3.  기본 미션 구성: 절차적 심층 분석

본 섹션에서는 이륙부터 착륙까지 표준적인 웨이포인트 기반 미션을 생성하는 과정을 상세하고 단계적으로 분석합니다.

### 3.1  미션 시작: 이륙(Takeoff) 명령어

미션은 일반적으로 이륙 명령어로 시작됩니다. 멀티콥터의 경우, 미션이 시작되면 QGC가 자동으로 이륙 단계를 수행할 수 있습니다.14 이륙 명령어(`MAV_CMD_NAV_TAKEOFF`)는 기체가 첫 번째 웨이포인트로 이동하기 전에 도달해야 할 고도를 정의합니다.21 이 고도는 미션 아이템의 매개변수에서 설정할 수 있습니다.

기본 미션 생성 과정은 '상태 기반(stateful)' 상호작용 모델을 따릅니다. 특정 액션(예: 웨이포인트 추가)의 사용 가능 여부는 계획의 현재 상태에 따라 결정됩니다. 예를 들어, 사용자가 웨이포인트 버튼이 비활성화되어 있다고 보고하는 경우가 있는데 22, 이는 미션이 허공에 떠 있는 웨이포인트로만 구성될 수 없기 때문입니다. 해결책은 먼저 '이륙' 명령어를 추가하거나 'Survey'와 같은 패턴을 선택하여 유효한 시작점을 정의하는 것입니다. 즉, QGC의 미션 플래너는 계획의 현재 유효성에 따라 UI 요소를 활성화 또는 비활성화함으로써 논리적인 미션 구조를 강제합니다. 이는 신뢰성을 높이는 일종의 유도된 사용자 입력 방식입니다.

### 3.2  웨이포인트 추가 및 설정

Plan Tools의 'Add Waypoint' 도구는 지도를 클릭하여 웨이포인트를 배치하는 데 사용됩니다.6 웨이포인트가 배치되면 지도나 Mission Command List에서 선택하여 매개변수를 편집할 수 있습니다.6

- **주요 매개변수:**
  - **고도(Altitude):** 홈 기준 상대 고도, 지형 기준 상대 고도 또는 절대 해발 고도(MSL)로 설정할 수 있습니다. 첫 번째 웨이포인트의 기본 고도는 'Mission Settings' 패널에서 설정됩니다.6
  - **비행 속도(Flight Speed):** 미션의 기본 순항 속도를 재정의하도록 지정할 수 있습니다.6
  - **카메라 액션(Camera Actions):** 웨이포인트는 단일 사진 촬영, 비디오 녹화 시작/중지, 또는 시간/거리 기반 사진 촬영 간격 시작과 같은 카메라 액션을 트리거할 수 있습니다.6

### 3.3  미션 종료: 착륙(Land) 및 복귀(RTL) 명령어

미션은 마지막 웨이포인트 위치에서 'Land' 명령어(`MAV_CMD_NAV_LAND`)로 마무리될 수 있습니다.14 또는 미션이 종료된 후 기체가 자동으로 'Return to Launch(RTL)' 모드로 전환되어 홈 위치로 복귀하여 착륙할 수도 있습니다.14 RTL의 동작은 랠리 포인트(Rally Points)에 의해 변경될 수 있습니다.14 일부 펌웨어 설정에서는 미션에 착륙 아이템이 반드시 포함되어야 합니다(`MIS_TKO_LAND_REQ` 파라미터).7

### 3.4  미션 생명주기: 업로드, 실행, 완료

계획이 완료되면 Plan Toolbar나 File (Sync) 도구의 'Upload' 버튼을 사용하여 기체에 업로드해야 합니다.6 그 후 사용자는 'Fly View'로 전환합니다.6 미션 경로는 지도에 표시되며 기체에 하드코딩됩니다.26 'Start Mission'을 누르고 슬라이더로 확인하면 미션이 시작됩니다.14 기체는 시동을 걸고 이륙하여 미션 아이템을 순차적으로 실행합니다. 완료 시, 사용자에게 '기체에서 계획 제거(Remove the plan from the vehicle)' 또는 '기체에 계획 남기기(Leave the plan on vehicle)' 옵션이 제공되어 향후 사용 여부를 결정할 수 있습니다.21

## 4.  자동화된 패턴을 이용한 고급 미션 설계

본 섹션에서는 복잡한 비행 기동을 간단한 그래픽 인터페이스로 추상화하는 QGC의 강력한 `Pattern` 도구를 탐색합니다.

### 4.1  `Pattern` 도구: 개요

Plan Tools에 위치한 `Pattern` 도구는 복잡한 비행 패턴을 그래픽으로 생성할 수 있게 해줍니다.8 사용 가능한 패턴은 연결된 기체의 유형과 펌웨어 지원 여부에 따라 달라집니다.8 이 도구는 단일 'Pattern' 아이템이 수십 또는 수백 개의 기저 MAVLink 명령어를 생성할 수 있는 더 높은 수준의 추상화를 나타냅니다.

이 패턴 도구는 명시적인 명령어 기반 계획에서 '목표 지향적 계획(goal-oriented planning)'으로의 패러다임 전환을 의미합니다. 사용자는 "이 영역을 70% 중첩률로 측량하라"와 같이 *무엇을* 원하는지 정의하고, QGC는 정밀한 웨이포인트, 속도, 카메라 트리거를 계산하는 등 *어떻게* 달성할지를 처리합니다. 예를 들어, `Structure Scan`에서는 사용자가 "필요 이미지 해상도"나 "이미지 중첩률"과 같은 높은 수준의 목표를 지정하면 9, QGC의 내부 로직(`CameraCalc` 객체 28)이 기체와 구조물 간의 거리, 각 레이어의 높이, 사진 트리거 간격과 같은 저수준 세부 사항을 자동으로 계산합니다. 이러한 추상화는 운영자의 인지 부하와 복잡한 임무에서의 인적 오류 가능성을 극적으로 줄여줍니다.

### 4.2  사례 연구: 항공 매핑을 위한 `Survey` 패턴

- **목적:** 사진 측량 및 매핑을 위한 지오태깅된 이미지를 생성하는 데 최적화된 다각형 영역 위의 그리드 기반 비행 패턴을 만듭니다.8
- **워크플로우:** 사용자가 지도에 다각형을 정의하면, `Survey` 도구는 카메라 설정, 이미지 중첩률(전방 및 측면), 그리드 라인 각도와 같은 매개변수를 구성할 수 있게 해줍니다. QGC는 원하는 커버리지를 달성하기 위해 필요한 비행 경로와 카메라 트리거 지점을 자동으로 계산합니다.8
- **경계 가져오기:** 측량 경계는 KML 또는 SHP 파일에서 가져올 수 있으며, 이는 고객으로부터 관심 영역을 받는 전문 측량사에게 매우 중요한 워크플로우입니다.6

### 4.3  사례 연구: 3D 모델링을 위한 `Structure Scan`

- **목적:** 시각적 검사 또는 3D 모델 생성을 위해 수직 표면(예: 건물 벽, 타워)의 이미지를 캡처하는 비행 패턴을 만듭니다.8
- **워크플로우:** 사용자가 구조물 바닥 주위에 다각형이나 원을 정의하면, 스캔은 구조물의 높이를 여러 "레이어"로 나눕니다. 기체는 각 레이어 고도에서 구조물 주위를 비행하며 이미지를 캡처합니다.9
- **주요 매개변수:**
  - `Structure height` 및 `Scan distance`: 전체적인 기하학적 구조를 정의합니다.
  - `Camera Orientation` 및 `Overlap`: 사전 정의된 카메라의 경우, 이 설정들은 3D 재구성에 충분한 데이터를 보장하기 위해 레이어 높이와 트리거 거리를 자동으로 계산합니다.9
  - `Manual Mode`: 사용자 지정 설정을 위해 `Layer height`, `Trigger Distance`, `Gimbal Pitch`를 직접 제어할 수 있습니다.9
- **펌웨어 제한:** 이 기능은 PX4에서 지원되지만 ArduPilot에서는 아직 지원되지 않는다고 명시되어 있습니다.12

### 4.4  사례 연구: 선형 구조물 점검을 위한 `Corridor Scan`

- **목적:** 도로, 철도, 파이프라인과 같은 선형 인프라를 측량하는 데 이상적인 폴리라인을 따라가는 비행 패턴을 만듭니다.8
- **워크플로우:** 사용자가 지도에 복도의 중심을 나타내는 선을 그리면, 도구는 복도의 너비와 기타 측량 매개변수를 지정하여 정의된 경로를 따라 왕복하는 패턴을 생성합니다.8

## 5.  `.plan` 파일 해부: JSON 스키마 기술 분석

본 섹션에서는 QGC 인터페이스 외부에서 계획을 생성하거나 조작하려는 개발자 및 고급 사용자에게 필수적인 파일 형식 자체에 대한 심층적인 기술 분석을 제공합니다.

### 5.1  최상위 JSON 객체 구조

`.plan` 파일은 사람이 읽을 수 있는 JSON 형식으로 저장됩니다.31 루트 객체는 여러 키-값 쌍을 포함합니다:

- `fileType`: 항상 `"Plan"`이어야 합니다.31
- `version`: 플랜 파일 형식의 버전입니다.31
- `groundStation`: 파일을 생성한 GCS로, 일반적으로 `"QGroundControl"`입니다.31
- `mission`: 핵심 비행 계획을 포함하는 JSON 객체입니다.31
- `geoFence`: 하나 이상의 지오펜스를 정의하는 선택적 객체입니다.31
- `rallyPoints`: 대체 착륙/체공 지점을 정의하는 선택적 객체입니다.31

### 5.2  `mission` 객체 법의학적 분석

이 객체는 미션 전반의 설정과 미션 아이템 목록을 포함합니다.

- **전역 매개변수:** `cruiseSpeed`, `hoverSpeed`, `firmwareType`, `vehicleType`.31
- **`plannedHomePosition`:** 오프라인 계획에 사용되는 시뮬레이션된 홈 위치를 나타내는 `[위도, 경도, 고도]` 배열입니다.31
- **`items`:** 미션 아이템 객체의 배열입니다. 이는 미션의 핵심으로, 작업 순서를 정의합니다.31

### 5.3  `SimpleItem` 대 `ComplexItem`: 미션의 구성 블록

`.plan` 파일 형식은 단순한 명령어 목록이 아니라, 미션 '컴파일' 프로세스를 위한 '소스 파일'과 같습니다. `SimpleItem`과 `ComplexItem`의 구분은 프로그래밍에서 어셈블리 코드와 고수준 함수 호출의 차이와 유사합니다. `.plan` 파일은 "Survey"와 같은 `ComplexItem`을 포함할 수 있지만 31, 기체의 비행 컨트롤러는 `SimpleItem`에 해당하는 기본적인 MAVLink `MISSION_ITEM` 명령어 시퀀스만 이해합니다.15 따라서 QGC는 업로드 전에 중첩률, 카메라 사양 등의 매개변수를 사용하여 `ComplexItem`을 '컴파일'하고, 이를 통해 다수의 `SimpleItem`(웨이포인트, 카메라 트리거 등) 목록을 생성해야 합니다. 이로 인해 `.plan` 파일은 사용자 의도의 고수준 표현이 되며, 드론에 전송되는 최종 실행 가능한 미션은 이 파일의 저수준 컴파일된 버전이 됩니다. 이 분리 구조는 견고한 MAVLink 미션 프로토콜과의 호환성을 유지하면서도 'Pattern'과 같은 강력하고 사용자 친화적인 기능을 가능하게 합니다.

- **`SimpleItem`:** 단일하고 직접적인 MAVLink `MISSION_ITEM` 명령어를 나타냅니다.31

  `command`(MAV_CMD ID), `frame`(좌표계), 그리고 명령어에 특화된 7개의 매개변수 배열인 `params`와 같은 필드를 포함합니다.31 이는 플랜 파일에서 볼 수 있는 가장 낮은 수준의 미션 구성 블록입니다.

- **`ComplexItem`:** Survey나 Structure Scan과 같은 패턴을 캡슐화하는 고수준 객체입니다.31 측량 경계를 위한 

  `polygon` 배열이나 스캔을 위한 `StructureHeight`와 같은 패턴별 메타데이터를 포함합니다.31 QGC는 기체에 업로드하기 전에 이 단일 

  `ComplexItem`을 내부적으로 일련의 `SimpleItem` 객체로 변환합니다.

### 5.4  `Mission Command Tree`: 동적 UI와 펌웨어 추상화의 열쇠

QGC는 `Mission Command Tree`라고 불리는 JSON 메타데이터 계층 구조로부터 미션 아이템 편집을 위한 UI를 동적으로 생성합니다.11 이 트리 구조는 서로 다른 펌웨어(PX4 대 ArduPilot)와 기체 유형의 특이점을 관리하는 데 필수적입니다. 이들은 동일한 MAVLink 명령어를 다른 매개변수나 동작으로 지원할 수 있기 때문입니다.11 이 트리는 일반적인 MAVLink 명령어의 루트를 가지며, 기체 유형과 펌웨어 유형에 따라 특정 재정의가 적용됩니다. 이를 통해 QGC는 사용자에게 특정 하드웨어에 대해 관련 있고 지원되는 옵션만을 제시할 수 있습니다.11 UI는 `MissionCommandUIInfo`(전체 명령어용)와 `MissionCmdParamInfo`(각 매개변수용) 클래스의 메타데이터를 사용하여 구축됩니다.11

#### 5.4.1 표 5.1: `.plan` JSON 파일 형식의 상세 스키마

| 키 경로                 | 데이터 타입     | 설명                                           | 필수/선택           | 관련 자료 |
| ----------------------- | --------------- | ---------------------------------------------- | ------------------- | --------- |
| `fileType`              | `String`        | 파일 유형. 항상 "Plan"이어야 함.               | 필수                | 31        |
| `version`               | `Integer`       | 파일 형식 버전.                                | 필수                | 31        |
| `groundStation`         | `String`        | 파일을 생성한 GCS 이름.                        | 필수                | 31        |
| `mission`               | `Object`        | 핵심 비행 계획을 포함하는 객체.                | 필수                | 31        |
| `mission.vehicleType`   | `Integer`       | 기체 유형 (`MAV_TYPE` 열거형).                 | 필수                | 31        |
| `mission.cruiseSpeed`   | `Number`        | 미션의 기본 순항 속도 (m/s).                   | 필수                | 31        |
| `mission.items`         | `Array[Object]` | `SimpleItem` 또는 `ComplexItem` 객체의 배열.   | 필수                | 31        |
| `mission.items.type`    | `String`        | 아이템 유형 ("SimpleItem" 또는 "ComplexItem"). | 필수                | 31        |
| `mission.items.command` | `Integer`       | MAVLink 명령어 ID (`MAV_CMD`).                 | `SimpleItem`에 필수 | 31        |
| `mission.items.params`  | `Array[Number]` | 명령어에 대한 7개의 매개변수 배열.             | `SimpleItem`에 필수 | 31        |
| `geoFence`              | `Object`        | 지오펜스 정보를 포함하는 객체.                 | 선택                | 31        |
| `rallyPoints`           | `Object`        | 랠리 포인트 정보를 포함하는 객체.              | 선택                | 31        |

## 6.  프로그래밍 방식 및 자동화된 Plan 생성

본 섹션에서는 QGC 인터페이스를 넘어 코드를 사용하여 미션 계획을 생성하고 조작하는 고급 방법을 탐구합니다.

### 6.1  MAVSDK를 이용한 프로그래밍 방식의 미션 생성

MAVSDK(Python, C++ 등에서 사용 가능)는 MAVLink 기체와 상호 작용하기 위한 고수준 API를 제공합니다.33 개발자는 `.plan` 파일을 직접 생성하는 대신, 코드에서 `MissionItem` 객체 목록을 만들 수 있습니다.35 MAVSDK의 `MissionItem` 클래스는 위도, 경도, 고도, 속도, 카메라 액션 등 미션 아이템의 개념을 반영합니다.36 그런 다음 `drone.mission.upload_mission(mission_items)`와 같은 함수를 사용하여 미션을 기체에 직접 업로드합니다.35 이는 물리적인 `.plan` 파일의 필요성을 우회합니다.

### 6.2  기존 `.plan` 파일 가져오기 및 파싱

MAVSDK-Python은 `import_qgroundcontrol_mission`이라는 특정 함수를 제공하여 디스크에서 `.plan` 파일을 읽고 그 내용을 MAVSDK 미션 아이템 목록으로 변환한 후 업로드할 수 있게 합니다.38 이는 QGC GUI에서 이전에 설계된 미션의 실행을 자동화하는 강력한 워크플로우입니다.34

미션 자동화를 위한 프로그래밍 워크플로우는 두 가지로 나뉩니다: '직접-기체(direct-to-vehicle)' 방식과 '파일 기반(file-based)' 방식입니다. MAVSDK 예제는 메모리에서 `MissionItem` 객체를 생성하여 `upload_mission`을 통해 드론에 직접 전송하는 실시간 '직접-기체' 접근법을 보여줍니다.35 반면, CSV에서 변환하는 것처럼 먼저 디스크에 

`.plan` 파일을 생성한 후 QGC에 로드하거나 스크립트로 가져오는 것은 '파일 기반' 접근법입니다.38 개발자는 작업의 성격에 따라 적절한 워크플로우를 선택해야 합니다. '직접-기체' 방식은 동적인 미션 생성에 이상적이며, '파일 기반' 방식은 사전 계획, 로깅 및 상호 운용성에 더 적합합니다. 이 프로그래밍 능력은 완전 자율 시스템을 위한 핵심 기술로, 상위 레벨의 AI 시스템이 실시간 데이터에 기반하여 비행 계획을 생성하고 기체에 명령을 내리는 것을 가능하게 합니다.41

### 6.3  외부 데이터 형식(CSV, KML)을 미션으로 변환

외부 데이터 소스에서 비행 계획을 생성하는 것은 일반적인 요구 사항입니다.

- **CSV에서 미션으로:** Python 스크립트를 작성하여 위도, 경도, 고도 열이 포함된 CSV 파일을 읽고, 올바른 JSON 구조를 가진 `.plan` 파일이나 QGC가 가져올 수 있는 일반 텍스트 웨이포인트 파일을 생성할 수 있습니다.40 이를 위한 오픈 소스 Python 파서도 존재합니다.43

- **KML에서 미션으로:** QGC는 `Survey` 다각형을 정의하기 위해 KML 파일을 로드하는 내장 기능을 가지고 있습니다.29

- **KML 가져오기 제한 사항:** QGC는 KML에 `Polygon` 노드가 포함될 것으로 예상하며, 파일에 선이나 점만 포함된 경우 "Unable to find Polygon node in KML" 오류와 함께 실패할 수 있습니다.30 이는 단순한 비행 경로를 선으로 가져오려는 사용자에게 혼란을 주는 지점입니다.30 주된 사용 사례는 비행 경로 자체가 아닌 측량 

  *경계*를 정의하는 것입니다.

## 7.  미션 생명주기 관리 및 문제 해결

본 섹션에서는 미션 파일 관리의 운영 측면과 계획 및 실행 단계에서 발생하는 일반적인 문제를 진단하는 방법을 다룹니다.

### 7.1  파일 관리: 계획 저장, 로드 및 공유

Plan View의 `File (Sync)` 도구를 사용하면 미션을 `.plan` 파일로 저장하고, 파일에서 로드하며, 기체에서 현재 미션을 다운로드할 수 있습니다.6 또한 QGC는 기체 연결 시 `AutoLoad#.plan`(#은 기체 ID)이라는 이름의 미션 파일을 자동으로 로드하도록 구성할 수 있습니다.24

### 7.2  미션 업로드/다운로드 실패

미션 업로드/다운로드 실패의 주된 원인은 높은 MAVLink 패킷 손실률로 특징지어지는 불량한 통신 링크입니다.6 QGC는 이 문제에 대해 Mission Planner보다 더 민감합니다. QGC는 신뢰할 수 없는 연결은 비행에 안전하지 않다는 철학에 따라 파라미터 다운로드를 3번만 재시도하고 실패 처리합니다. 반면 Mission Planner는 무한정 재시도합니다.45 일부 장거리 무선 통신과 같이 느린 원격 측정 링크도 문제를 일으킬 수 있는데, QGC가 링크가 처리할 수 있는 것보다 높은 데이터 스트림 속도를 구성하려고 시도하여 포화 및 패킷 손실을 유발할 수 있기 때문입니다.44

### 7.3  일반적인 미션 오류 문제 해결

미션의 신뢰성은 계획 파일의 정확성뿐만 아니라 전체 통신 생태계의 건전성에 크게 의존합니다. 사용자들이 보고하는 미션 업로드 및 파라미터 다운로드 실패는 45 QGC의 소프트웨어 버그가 아니라 높은 패킷 손실을 동반하는 "잡음이 많은 통신 연결"이 근본 원인인 경우가 많습니다.44 QGC의 설계 철학은 안전 조치로서 이러한 문제에 대해 다른 GCS보다 덜 관용적입니다.45 따라서 QGC에서의 문제 해결은 전체 시스템 수준의 접근이 필요합니다. 운영자는 소프트웨어에만 집중할 것이 아니라 물리적인 무선 링크를 진단하고, 간섭을 확인하며, 링크 대역폭에 맞게 MAVLink 스트림 속도(`SRn_` 파라미터)를 조정해야 합니다.44

- **"웨이포인트를 추가할 수 없음(Unable to add waypoints)":** 미션에 유효한 시작점이 없을 때 자주 발생합니다. 사용자는 먼저 이륙 아이템이나 패턴을 추가해야 합니다.22
- **"파라미터 다운로드 실패(Parameter Download Failure)":** 위에서 논의한 바와 같이, 이는 거의 항상 통신 링크 품질 문제입니다.45 해결책은 소프트웨어를 재시작하는 것이 아니라 물리적 링크를 개선하는 것입니다.
- **"KML에서 다각형 노드를 찾을 수 없음(Unable to find Polygon node in KML)":** QGC의 KML 가져오기 기능이 주로 측량 경계를 정의하기 위해 설계되었기 때문에 다각형이 포함되지 않은 KML 파일을 가져오려고 할 때 발생합니다.29 해결책은 KML이 유효한 다각형 피처로 생성되었는지 확인하는 것입니다.29
- **미션 실행 오류(예: 웨이포인트에서 체공):** 좁은 회전에 비해 너무 빠른 속도와 같이 기체의 비행 동역학과 호환되지 않는 미션 파라미터로 인해 발생할 수 있습니다.46

### 7.4  "미션 재개(Resume Mission)" 기능: 기능 및 실패 모드

기체가 미션 중간에 착륙하면(예: 배터리 교체), QGC는 "미션 재개" 옵션을 제공합니다.26 이 기능은 남은 웨이포인트만 포함하는 *새로운* 미션 계획을 생성하여 작동합니다.26

- **실패 모드:** 이 기능은 문제가 발생하기 쉽습니다. 일반적인 문제로는 재개 대화 상자가 나타나지 않거나, 새로 생성된 미션이 부정확한 경우(예: 카메라 명령어 누락)가 있습니다.47
- **디버깅:** 재개 실패를 디버깅하려면 특정 절차가 필요합니다: `GuidedActionsControllerLog` 콘솔 로깅을 활성화하고, 원격 측정 로깅을 활성화한 후 문제를 재현합니다. 그런 다음 콘솔 로그, 원격 측정 로그, 원본 계획 파일, 그리고 수정된(재개된) 계획 파일을 저장하여 분석해야 합니다.47

#### 7.4.1 표 7.1: 일반적인 미션 계획 오류 및 진단 해결책

| 오류 메시지 / 증상                                         | 유력한 근본 원인                  | 진단 단계 및 해결책                                | 관련 자료 |
| ---------------------------------------------------------- | --------------------------------- | -------------------------------------------------- | --------- |
| **"Parameter download failed"**                            | 원격 측정 링크의 높은 패킷 손실   | 설정에서 Mavlink 패킷 손실률 확인; 무선 링크 개선. | 45        |
| **"Waypoint button is greyed out"**                        | 미션 계획에 이륙 명령어가 없음    | 먼저 이륙 아이템을 추가.                           | 22        |
| **"KML file load failed. Unable to find Polygon node..."** | KML 파일에 다각형 대신 선/점 포함 | GIS 소프트웨어에서 KML을 다각형으로 다시 내보내기. | 29        |
| **"Resume Mission dialog does not appear"**                | QGC의 재개 로직 상태 추적 버그    | 47의 상세 로깅 절차를 따르고 이슈 보고.            | 47        |

## 8.  결론 분석 및 권장 사항

본 마지막 섹션에서는 보고서의 결과를 종합하고, 비교 분석을 제공하며, 모범 사례 및 향후 방향에 대한 권장 사항을 제시합니다.

### 8.1  결과 종합: 계획 생성 방법론 대조

세 가지 주요 계획 생성 방법론의 장단점을 요약하면 다음과 같습니다:

1. **GUI 기반:** 직관적이고 시각적이지만 복잡하거나 대규모 미션에는 지루할 수 있습니다.
2. **패턴 기반:** 매핑, 점검과 같은 표준화된 작업에 매우 효율적이지만 사용 가능한 패턴으로 제한됩니다.
3. **프로그래밍 방식:** 최고의 유연성과 자동화를 제공하지만 개발 기술이 필요합니다.

### 8.2  비교 분석: QGroundControl 대 Mission Planner

- **UI 철학 및 사용성:** QGC는 일반적으로 더 현대적이고 직관적이며 초보자 친화적인 UI를 가지고 있으며, 크로스 플랫폼 일관성에 중점을 둡니다.5 Mission Planner는 더 밀집되고 기능이 풍부한 인터페이스를 가지고 있어 압도적일 수 있지만 고급 사용자에게는 강력합니다.5
- **플랫폼 및 펌웨어 지원:** QGC의 핵심 장점은 Windows, macOS, Linux, iOS, Android 및 PX4와 ArduPilot 펌웨어 모두를 폭넓게 지원한다는 것입니다.5 Mission Planner는 주로 Windows 애플리케이션이며(Linux에서 Mono/에뮬레이션을 통해 실행 가능) ArduPilot에 특화되어 있습니다.5
- **기능 깊이:** Mission Planner는 종종 파라미터 튜닝, 로그 분석 및 스크립팅에서 더 고급 기능을 가지고 있다고 평가됩니다.5 QGC의 기능 세트는 포괄적이지만 MP에서 볼 수 있는 일부 고도로 전문화된 도구는 부족할 수 있습니다.5

### 8.3  모범 사례 권장 사항

- **전문 측량사:** `Survey` 및 `Corridor Scan` 패턴을 활용하십시오. 고객 데이터로부터 미션 경계를 정의하기 위해 KML/SHP 가져오기 워크플로우를 숙달하십시오.
- **3D 모델러/검사 전문가:** `Structure Scan` 패턴(PX4 펌웨어와 함께)을 사용하십시오. 고품질 데이터 캡처를 보장하기 위해 카메라 및 중첩 설정에 세심한 주의를 기울이십시오.
- **개발자/연구원:** 프로그래밍 방식의 미션 생성 및 자동화를 위해 MAVSDK를 사용하십시오. `.plan` JSON 형식을 분석하여 맞춤형 미션 생성 도구를 구축하십시오.

### 8.4  미래 전망: AI 기반 동적 미션 계획

QGC의 현재 미션 계획 패러다임은 대체로 정적입니다. 즉, 계획을 생성하고 업로드한 후 실행합니다. 미래는 동적 미션 계획에 있으며, 컴패니언 컴퓨터나 클라우드의 AI 시스템이 센서 데이터를 기반으로 실시간으로 미션을 생성하거나 수정할 수 있습니다.50

예를 들어, AI가 물체를 감지하고 이를 추적하기 위해 연속적인 웨이포인트 스트림을 생성하는 '시각적 따라가기(visual follow-me)' 모드 41나, 새로 감지된 장애물 또는 갑자기 나타난 비행 금지 구역을 중심으로 측량 미션을 재계획하는 시스템이 있습니다.50 QGC 및 MAVLink 생태계 내의 프로그래밍 인터페이스(섹션 6)와 아키텍처 추상화(섹션 5)는 이러한 차세대 지능형 자율 시스템을 가능하게 하는 기초적인 요소입니다.

#### **참고 자료**

1. QGroundControl – Drone Control – Ground Control Station for Small Air – Land – Water Autonomous Unmanned Systems, accessed July 11, 2025, https://qgroundcontrol.com/
2. Q Ground Control, accessed July 11, 2025, https://dmna.ny.gov/nynm/manuals/QGC_User_Guide.pdf
3. QGC Guide (4.4) - QGroundControl User Guide, accessed July 11, 2025, https://docs.qgroundcontrol.com/v4.4.3/en/qgc-user-guide/index.html
4. QGC Guide (master) - QGroundControl Guide, accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/
5. QGroundControl vs. Mission Planner: Which One is Right for You? - Flymore Drone, accessed July 11, 2025, https://www.flymoredrone.in/blog-details/qgroundcontrol-vs-mission-planner-which-one-is-right-for-you
6. Plan View | QGC Guide (4.3) - QGroundControl Guide, accessed July 11, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/plan_view/plan_view.html
7. Mission Mode (Fixed-Wing) | PX4 Guide (main), accessed July 11, 2025, https://docs.px4.io/main/en/flight_modes_fw/mission.html
8. Pattern | QGC Guide (master) - QGroundControl Guide, accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/plan_view/pattern.html
9. Structure Scan (Plan Pattern) | QGC Guide (4.3), accessed July 11, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/plan_view/pattern_structure_scan_v2.html
10. Top Drone Mission Planning Software in 2025 - The Dronedesk Blog, accessed July 11, 2025, https://blog.dronedesk.io/drone-mission-planning-software/
11. Mission Command Tree | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/plan/mission_command_tree.html
12. Structure Scan (Plan Pattern) | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/plan_view/pattern_structure_scan_v2.html
13. Mission Planner Overview - ArduPilot, accessed July 11, 2025, https://ardupilot.org/planner/docs/mission-planner-overview.html
14. Fly View | QGC Guide (4.3) - QGroundControl Guide, accessed July 11, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/fly_view/fly_view.html
15. Mission Protocol - MAVLink Guide, accessed July 11, 2025, https://mavlink.io/en/services/mission.html
16. Plan View - GeoFence | QGC Guide (4.3), accessed July 11, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/zh/qgc-user-guide/plan_view/plan_geofence.html
17. Plan View - Rally Points | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/plan_view/plan_rally_points.html
18. Plan View | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/plan_view/plan_view.html
19. Fly View | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/fly_view/hud.html
20. Fly Tools | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/fly_view/fly_tools.html
21. How To Setup Waypoint Mission QGround Control - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=3bizwuySQlg
22. Waypoint icon greyed out - Using QGroundControl - PX4 Discussion Forum, accessed July 11, 2025, https://discuss.px4.io/t/waypoint-icon-greyed-out/21986
23. Plan · QGroundControl User Guide, accessed July 11, 2025, https://qgroundcontrol.gitbooks.io/qgc-user-guide/quickstart_plan.html
24. General Settings (Settings View) | QGC Guide (4.3) - QGroundControl Guide, accessed July 11, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/zh/qgc-user-guide/settings_view/general.html
25. Rally Points — Copter documentation - ArduPilot, accessed July 11, 2025, https://ardupilot.org/copter/docs/common-rally-points.html
26. Mission Planning in QGroundControl - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=zBmC0EeP54Q
27. "Getting Started" guide for QGroundControl (Planning a Mission) - YouTube, accessed July 11, 2025, https://m.youtube.com/watch?v=0d23O_RUOmI&t=395s
28. Plan: Structure Scan discussion · Issue #5694 · mavlink/qgroundcontrol - GitHub, accessed July 11, 2025, https://github.com/mavlink/qgroundcontrol/issues/5694
29. Uploading KMLs into QGC - INSPIRED DOCUMENTATION, accessed July 11, 2025, https://docs.inspiredflight.com/inspired-documentation/products/additional-software/qgc/uploading-kmls-into-qgc
30. How to upload a .kml into QGroundControl - PX4 Discussion Forum, accessed July 11, 2025, https://discuss.px4.io/t/how-to-upload-a-kml-into-qgroundcontrol/6146
31. Plan File Format | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/file_formats/plan.html
32. QGC Plan Format Example - GitHub Gist, accessed July 11, 2025, https://gist.github.com/SamuelDudley/c6a5e1aa5a49ce2b1690b980fc248563
33. Actions (Take-off, Landing, Arming, etc) | MAVSDK Guide, accessed July 11, 2025, https://mavsdk.mavlink.io/main/en/cpp/guide/taking_off_landing.html
34. Example: Fly QGroundControl Plan Mission - Hamish Willee, accessed July 11, 2025, https://hamishwillee.gitbooks.io/dronecore/content/en/examples/fly_mission_qgc_plan.html
35. A mission example using MAVSDK-Python · GitHub, accessed July 11, 2025, https://gist.github.com/JonasVautherin/66eacd179f7fff45bb09da2b64df2d3e
36. mavsdk::Mission::MissionItem Struct Reference, accessed July 11, 2025, https://mavsdk.mavlink.io/main/en/cpp/api_reference/structmavsdk_1_1_mission_1_1_mission_item.html
37. MAVSDK-Python/mavsdk/mission.py at main · mavlink/MAVSDK-Python - GitHub, accessed July 11, 2025, https://github.com/mavlink/MAVSDK-Python/blob/main/mavsdk/mission.py
38. MAVSDK -- Import a QGroundControl missions in JSON .plan format ..., accessed July 11, 2025, https://github.com/mavlink/MAVSDK-Python/issues/439
39. QGC Mission Upload using MAVSDK python for multiple aircraft - PX4 Discussion Forum, accessed July 11, 2025, https://discuss.px4.io/t/qgc-mission-upload-using-mavsdk-python-for-multiple-aircraft/29812
40. KML files for mission plans in QGC - QGroundControl - Blue Robotics Community Forums, accessed July 11, 2025, https://discuss.bluerobotics.com/t/kml-files-for-mission-plans-in-qgc/15978
41. GSoC24: Visual Follow-me using AI - Blog - ArduPilot Discourse, accessed July 11, 2025, https://discuss.ardupilot.org/t/gsoc24-visual-follow-me-using-ai/119527
42. KML files for mission plans in QGC - #2 by zhrandell - Blue Robotics Community Forums, accessed July 11, 2025, https://discuss.bluerobotics.com/t/kml-files-for-mission-plans-in-qgc/15978/2
43. geoffreynyaga/mission-planner-and-QgroundControl-flight-plan-parser - GitHub, accessed July 11, 2025, https://github.com/geoffreynyaga/mission-planner-and-QgroundControl-flight-plan-parser
44. The qground control can't received full set. of parameters - ArduPilot Discourse, accessed July 11, 2025, https://discuss.ardupilot.org/t/the-qground-control-cant-received-full-set-of-parameters/48773
45. Failure to download parameters upon startup · Issue #10749 ..., accessed July 11, 2025, https://github.com/mavlink/qgroundcontrol/issues/10749
46. Error in mission mode with USV - QGroundControl - PX4 Discussion Forum, accessed July 11, 2025, https://discuss.px4.io/t/error-in-mission-mode-with-usv/39226
47. Resume Mission Failures | QGC Guide (master), accessed July 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/troubleshooting/resume_mission.html
48. Discussion Does Anyone Use QGroundControl In Lieu of Mission Planner And If So, Why?, accessed July 11, 2025, https://www.rcgroups.com/forums/showthread.php?2562324-Does-Anyone-Use-QGroundControl-In-Lieu-of-Mission-Planner-And-If-So-Why
49. Comapring QGround Control and Mission Planner - QGroundControl - ArduPilot Discourse, accessed July 11, 2025, https://discuss.ardupilot.org/t/comapring-qground-control-and-mission-planner/104208
50. Extending QGroundControl for Automated Mission Planning of UAVs - MDPI, accessed July 11, 2025, https://www.mdpi.com/1424-8220/18/7/2339
51. Machine Learning Approach to Real-Time 3D Path Planning for Autonomous Navigation of Unmanned Aerial Vehicle - MDPI, accessed July 11, 2025, https://www.mdpi.com/2076-3417/11/10/4706
52. Dynamic mission planning for drones with Azure Maps | Microsoft Azure Blog, accessed July 11, 2025, https://azure.microsoft.com/en-us/blog/dynamic-mission-planning-for-drones-with-azure-maps/


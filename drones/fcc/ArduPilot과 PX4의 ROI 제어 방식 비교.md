# ArduPilot과 PX4의 ROI 제어 방식 비교

멀티콥터 드론의 임무 수행 중 특정 지점을 지속적으로 바라보도록 지시하는 관심 지역(Region of Interest, ROI) 기능은 항공 촬영, 감시, 정찰 등 다양한 분야에서 핵심적인 역할을 수행합니다. 지상 관제 시스템(Ground Control Station, GCS)에서 ROI를 설정했을 때, ArduPilot과 PX4라는 두 주요 오픈소스 비행 스택(Flight Stack)이 보이는 상이한 거동은 많은 개발자와 시스템 통합 전문가들에게 중요한 기술적 질문을 제기합니다. ArduPilot은 기체 전체의 Yaw를 변경하여 ROI를 향하는 반면, PX4는 기본적으로 기체의 Yaw를 변경하지 않아 짐벌 제어에만 의존하는 것처럼 보입니다.

본 보고서는 이 두 비행 스택의 ROI 구현 방식에 대한 심층적인 기술 분석을 제공합니다. 이를 위해 공통 통신 프로토콜인 MAVLink의 관련 명령어들을 먼저 분석하고, 각 비행 스택의 내부 파라미터와 제어 로직, 그리고 그 기저에 깔린 설계 철학을 비교합니다. 최종적으로, 이러한 차이가 실제 드론 시스템 통합 및 운용에 미치는 실질적인 영향과 각 시스템에서 원하는 포인팅(pointing) 동작을 구현하기 위한 구체적인 방안을 제시하는 것을 목표로 합니다.

## 1.  MAVLink 프로토콜: 공통된 제어 언어

ArduPilot과 PX4는 모두 MAVLink라는 공통 프로토콜을 사용하여 GCS 및 온보드 컴퓨터와 통신합니다.1 따라서 두 시스템의 동작 차이를 이해하기 위해서는 먼저 MAVLink가 ROI 및 Yaw 제어를 위해 어떤 명령어들을 제공하는지 명확히 파악해야 합니다. 두 비행 스택은 동일한 명령어 세트를 사용하지만, 각 명령어의 해석과 우선순위 적용 방식에서 차이를 보이며, 이것이 관찰되는 거동 차이의 근본 원인이 됩니다.

### 1.1  `DO_SET_ROI` 명령어: 상세 분석

사용자 질의의 핵심에 있는 명령어는 `MAV_CMD_DO_SET_ROI` (명령어 ID 201)입니다. 이 명령어는 MAVLink에서 "DO" 명령 그룹에 속하며, 이는 기체의 위치에 직접적인 영향을 주지 않는 보조 기능을 수행하는 명령어들을 의미합니다.2 MAVLink 공식 문서에 따르면, 이 명령어의 목적은 "센서 세트 또는 기체 자체를 위한 관심 지역(ROI)을 설정하는 것"이며, 이렇게 설정된 ROI는 "기체의 자세(attitude)와 카메라와 같은 다양한 센서의 자세를 제어하기 위해 차량의 제어 시스템에 의해 사용될 수 있습니다".3

이 정의에서 가장 중요한 부분은 "기체의 자세와 센서의 자세"를 모두 제어할 수 있다고 명시한 점입니다. MAVLink 프로토콜은 의도적으로 이 부분을 모호하게 남겨두어 각 비행 스택이 자체적인 설계 철학에 따라 구현할 수 있는 유연성을 부여했습니다. 이 명령어는 `MAV_ROI_LOCATION` (특정 GPS 좌표를 향함), `MAV_ROI_WPNEXT` (다음 웨이포인트를 향함), `MAV_ROI_NONE` (ROI 설정을 취소함) 등 여러 하위 모드를 포함합니다.4 결국, `DO_SET_ROI`는 "이곳을 보라"는 고수준의 추상적인 지시이며, 이를 수행하기 위해 기체를 돌릴 것인지, 짐벌을 돌릴 것인지, 아니면 둘 다 사용할 것인지에 대한 구체적인 결정은 각 비행 스택의 내부 로직에 달려있습니다.

### 1.2  직접적인 Yaw 제어: `CONDITION_YAW`와 웨이포인트 헤딩

MAVLink는 `DO_SET_ROI`와 별개로 기체의 Yaw를 직접적으로 제어하기 위한 명시적인 명령어들을 제공합니다.

첫째, `MAV_CMD_CONDITION_YAW` (명령어 ID 115)는 기체가 특정 목표 Yaw 각도에 도달하도록 지시하는 명령어입니다. 이 각도는 북쪽을 기준으로 하는 절대 각도이거나 현재 기체의 헤딩을 기준으로 하는 상대 각도일 수 있으며, 회전 속도와 방향(시계방향/반시계방향)까지 지정할 수 있습니다.5 이 명령어는 임무 수행 중 동적인 회전 기동을 구현하는 데 사용됩니다.

둘째, `MAV_CMD_NAV_WAYPOINT` 명령어 자체에 기체의 Yaw를 설정하는 기능이 포함되어 있습니다. 이 명령어의 `param4` 필드는 해당 웨이포인트에 도달했을 때 기체가 가져야 할 목표 Yaw 각도를 지정하는 데 사용됩니다.7 이는 임무의 각 구간에서 정적인 헤딩을 유지하고자 할 때 유용합니다.

이러한 직접적인 Yaw 제어 명령어들의 존재는 `DO_SET_ROI`가 기체 Yaw 제어를 위한 유일한 수단이 아님을 시사합니다. 개발자는 임무의 목적에 따라 `DO_SET_ROI`를 통한 동적 포인팅과 `CONDITION_YAW` 또는 웨이포인트 헤딩 설정을 통한 정적/동적 Yaw 제어를 조합하여 사용할 수 있습니다.

### 1.3  전용 페이로드 포인팅: `DO_MOUNT_CONTROL` 명령어

MAVLink는 기체 자세 제어와 완전히 분리된 페이로드(짐벌) 제어를 위한 전용 명령어도 제공합니다. `MAV_CMD_DO_MOUNT_CONTROL` (명령어 ID 205)은 카메라나 안테나 마운트를 명시적으로 제어하기 위해 설계되었으며, Pitch, Roll, Yaw 각도를 직접 지정할 수 있습니다.3

이 명령어는 `DO_SET_ROI`와 명확히 구분되며, 기체 제어 로직을 거치지 않고 직접적으로 짐벌 하드웨어를 목표로 합니다. ArduPilot 문서는 이 명령어를 "Mavlink targeting mode"라고 부르며, `DO_SET_ROI`와 연관된 "GPS point mode"와 구분하고 있습니다.9

`DO_MOUNT_CONTROL`의 존재는 MAVLink 프로토콜 설계자들이 기체 자세와 독립적으로 페이로드를 제어해야 할 필요성을 명확히 인지하고 있었음을 보여줍니다. 이 명령어는 PX4가 암묵적으로 선호하는 "페이로드 중심" 접근 방식의 근간을 이룹니다.

이처럼 MAVLink 프로토콜은 경직된 명령어 집합이 아니라 유연한 프레임워크입니다. `DO_SET_ROI`와 같은 명령어에 내재된 모호성은 의도된 설계 선택으로, ArduPilot과 PX4 같은 비행 스택이 각자의 고유한 "성격"을 개발하고 서로 다른 사용자 철학(예: 연구용 대 취미용, 복잡한 하드웨어 대 단순한 하드웨어)에 부응할 수 있도록 하는 다양성을 촉진합니다.10 하지만 이러한 유연성은 양날의 검과 같아서, 시스템 간 전환 시 학습 곡선을 가파르게 만들고 동일한 GCS 임무 계획이 비행 스택에 따라 극적으로 다른 결과를 낳는 파편화의 원인이 되기도 합니다.

## 2.  ArduPilot의 구현: 기체 중심 접근 방식

ArduPilot의 ROI 처리 방식은 짐벌이 없는 단순한 하드웨어부터 복잡한 시스템에 이르기까지 광범위한 환경에서 견고하고 예측 가능한 동작을 제공하는 것을 우선시하는 설계 역사와 철학의 직접적인 결과물입니다.

### 2.1  핵심 철학: 주 포인팅 장치로서의 기체

ArduPilot은 2009년부터 개발되어 온 오랜 역사를 가지고 있으며, 실제 환경에서의 성능과 풍부한 기능 세트를 갖춘 "성숙한" 비행 스택으로 평가받습니다.1 2013년에서 2014년 사이의 개발자 포럼 토론을 살펴보면, 

`DO_SET_ROI`의 동작을 웨이포인트 간에 "지속되도록(persistent)" 의도적으로 발전시킨 것을 확인할 수 있습니다. 이는 한번 ROI가 설정되면 명시적으로 취소될 때까지 기체가 계속해서 해당 지점을 바라보도록 하는 것을 의미합니다.12 이는 기체 자체를 포인팅 작업 수행의 주체로 삼으려는 강력한 설계 의도를 보여줍니다. 한 사용자의 의견은 이를 명확히 요약합니다: 2축 짐벌을 사용할 경우, ArduPilot은 "관심 지역을 마주 보도록 콥터를 회전시킬 것"입니다.13

이러한 배경은 ArduPilot의 동작 이면에 있는 "이유"를 설명합니다. 이는 우연이 아니라, 3축 짐벌 없이도 "관심 지점(Point of Interest)" 임무를 간단하고 효과적으로 수행할 수 있는 방법을 제공하기 위해 수년간 사용자 피드백을 바탕으로 개선된 기능입니다.

### 2.2  `WP_YAW_BEHAVIOR` 파라미터: 기본 임무 헤딩 정의

ArduPilot에서 자동 임무 중 Yaw를 제어하는 마스터 파라미터는 `WP_YAW_BEHAVIOR`입니다.14 이 파라미터는 아래 표와 같이 여러 모드를 제공합니다. 중요한 점은 `DO_SET_ROI`나 `CONDITION_YAW` 명령어가 활성화되면, 이 `WP_YAW_BEHAVIOR` 설정을 일시적으로 무시하고 우선적으로 적용된다는 것입니다.12

| 값   | 이름                          | 설명                                                         |      |
| ---- | ----------------------------- | ------------------------------------------------------------ | ---- |
| 0    | Never change yaw              | 임무 중 기체의 Yaw를 절대 변경하지 않습니다. 이륙 시의 헤딩을 유지합니다. |      |
| 1    | Face next waypoint            | 기수가 항상 다음 웨이포인트를 향하도록 Yaw를 제어합니다.     |      |
| 2    | Face next waypoint except RTL | 'Face next waypoint'와 동일하지만, RTL(Return to Launch) 모드에서는 이 동작을 수행하지 않습니다. |      |
| 3    | Face along GPS course         | 기체의 이동 방향(GPS 경로)과 기수의 방향을 일치시킵니다. 측풍이 있을 경우 기수 방향이 다음 웨이포인트와 다를 수 있습니다. |      |

표 1: ArduPilot `WP_YAW_BEHAVIOR` 파라미터 옵션 14

이 표는 ROI 명령어가 개입하기 전의 기준 Yaw 동작을 이해하는 데 필수적이며, 사용자가 드론의 기본 헤딩을 설정하기 위한 명확하고 실행 가능한 참조를 제공합니다.

### 2.3  `DO_SET_ROI`의 실제 동작: 논리 흐름 분석

ArduPilot 기체가 `DO_SET_ROI` 명령을 수신하면, 비행 컨트롤러는 현재 위치에서 ROI 좌표까지의 방위각을 계산하여 기수의 목표 Yaw 값을 결정합니다. 이 계산된 Yaw 값은 `WP_YAW_BEHAVIOR` 파라미터보다 높은 우선순위를 가집니다. 역사적으로 이 기능은 다음 웨이포인트에만 영향을 미치는 "원샷(one-shot)" 명령어였지만, 사용자들의 강력한 요구에 따라 지속적인(persistent) 기능으로 변경되었습니다.12 이 지속적인 ROI 모드를 취소하고 `WP_YAW_BEHAVIOR`에 의해 정의된 기본 동작으로 돌아가려면, 모든 좌표값이 0으로 설정된 새로운 `DO_SET_ROI` 명령어를 전송해야 합니다.4 이 논리 흐름은 ArduPilot의 `DO_SET_ROI`에 대한 주된 반응이 기체의 Yaw를 변경하는 것임을 명확히 하여 사용자의 질문에 직접적으로 답합니다.

### 2.4  최적의 포인팅을 위한 기체와 짐벌의 협응

ArduPilot은 기체 중심적 접근 방식을 취하면서도 짐벌을 완벽하게 지원합니다. 짐벌이 장착된 경우, `DO_SET_ROI`는 기체와 짐벌을 모두 제어합니다. 기체는 목표물이 대체로 전방에 위치하도록 Yaw를 조정하고, 짐벌은 정밀한 포인팅과 안정화를 제공합니다.13 3축 짐벌이 사용될 경우 시스템은 이를 지능적으로 활용합니다. 그러나 만약 사용자가 3축 짐벌을 가지고 있으면서도 영상 촬영의 미학적 이유 등으로 의도적으로 기체 Yaw를 강제하고 싶다면, 짐벌의 Yaw 제어 기능을 비활성화할 수 있습니다. 이는 비행 컨트롤러의 관점에서 짐벌을 사실상 2축 짐벌처럼 만들고, 결과적으로 콥터가 Yaw를 하도록 강제합니다.13

이는 ArduPilot이 단순히 기체 Yaw에만 의존하는 것이 아니라, 기체와 페이로드가 조화롭게 작동하는 통합 시스템임을 보여줍니다. ArduPilot의 접근 방식은 계층적입니다. 기체 Yaw가 기초를 이루고, 그 위에 짐벌 제어가 층을 이룹니다. 이러한 "점진적 기능 저하(graceful degradation)" 철학은 가장 단순한 하드웨어(고정형 카메라)에서도 핵심 기능(객체 포인팅)을 제공하고, 하드웨어가 정교해짐에 따라(2축, 3축 짐벌) 기능을 추가하는 방식으로 시스템을 설계합니다. 이는 ArduPilot을 매우 견고하고 다재다능하게 만들지만, 때로는 제어 로직이 서로 얽혀 있어 PX4의 분리된 모델보다 구조적으로 덜 "깨끗하게" 보일 수 있습니다.

## 3.  PX4의 구현: 분리된 시스템 아키텍처

PX4는 근본적으로 다른 접근 방식을 취하며, 모듈성과 비행 제어로부터 페이로드 제어를 분리하는 아키텍처에 의존합니다. 사용자가 관찰한 동작은 의도된 설계의 결과이며, 이는 비행 제어와 페이로드 제어를 분리하는 소프트웨어 아키텍처에 뿌리를 두고 있습니다.

### 3.1  핵심 철학: 기체 자세와 페이로드 제어의 분리

PX4는 모듈식 아키텍처로 유명하며, 맞춤형 통합이 핵심인 연구 및 상업용 애플리케이션에서 종종 선호됩니다.10 그 설계 철학은 개발자의 의견에서 명확히 드러납니다: "모든 사용자가 기체를 ROI 쪽으로 돌리기를 원하지는 않을 것입니다. Yaw가 가능한 짐벌이 있다면, 기체는 전방을 향하게 하고 카메라만 돌리기를 원할 수 있습니다".18 이는 기체의 Yaw를 자동으로 제어하지 않으려는 설계 의도를 명시적으로 보여주며, 유능한 짐벌이 주된 포인팅 도구라는 가정을 전제로 합니다. 이 분리는 버그가 아니라 핵심적인 기능입니다.

### 3.2  마운트 드라이버의 우선권: `MNT_MODE_IN`과 `MNT_MODE_OUT`

PX4의 문서는 마운트(짐벌) 드라이버의 구성을 매우 강조합니다.19 PX4가 짐벌 명령을 수신하는 방법(`MNT_MODE_IN`, 예: MAVLink)과 PX4가 짐벌에 명령을 전송하는 방법(`MNT_MODE_OUT`, 예: MAVLink v2 또는 PWM)을 정의하는 파라미터들은 그 작동의 중심에 있습니다.19 Gremsy 짐벌에 대한 문서는 PX4의 경우 사용자가 `MNT_MODE_OUT`을 `MAVLINK`로 설정해야 한다고 명시적으로 언급합니다.9

이는 철학 뒤에 있는 메커니즘을 보여줍니다. PX4는 단순히 짐벌이 존재한다고 가정하는 것이 아니라, 모든 포인팅 명령에 대한 중개자 역할을 하는 전용의 설정 가능한 소프트웨어 드라이버를 가지고 있습니다. `DO_SET_ROI` 명령이 수신되면, 이 명령은 이 마운트 드라이버를 통해 라우팅됩니다. 만약 드라이버가 비활성화되었거나 올바르게 구성되지 않았다면, 이 명령은 아무런 효과가 없을 수 있습니다.

### 3.3  기본 Yaw 관리: `MPC_YAW_MODE` 파라미터

PX4에서 ArduPilot의 `WP_YAW_BEHAVIOR`에 해당하는 것은 `MPC_YAW_MODE`입니다.21 이 파라미터는 다른 Yaw 명령이 활성화되지 않았을 때 임무 중 기본 Yaw 동작을 제어합니다. 옵션에는 "Towards Waypoint", "Along Trajectory", "Towards Home" 등이 포함됩니다.22 ArduPilot과 달리, `DO_SET_ROI` 명령은 일반적으로 기체 Yaw에 대해 이 파라미터를 재정의하지 않습니다. 대신, 기체는 `MPC_YAW_MODE` 동작을 계속 따르는 동안 짐벌(있는 경우)이 ROI를 향하게 됩니다.

| 값   | 이름                         | 설명                                                         |      |
| ---- | ---------------------------- | ------------------------------------------------------------ | ---- |
| 0    | Towards Waypoint             | 기체의 Yaw가 지속적으로 다음 웨이포인트를 향하도록 방향을 잡습니다. |      |
| 1    | Towards Home                 | 기체는 홈 위치를 향하는 Yaw 방향을 유지합니다.               |      |
| 2    | Away From Home               | 기체의 Yaw가 홈 위치에서 멀어지는 방향으로 향합니다.         |      |
| 3    | Along Trajectory             | 기체의 Yaw가 현재 속도 벡터와 정렬됩니다. 즉, 현재 이동하는 방향을 가리킵니다. |      |
| 4    | Towards Waypoint (Yaw First) | 'Towards Waypoint'와 유사하지만, 궤적을 따라 이동하기 전에 Yaw 설정값을 먼저 달성하는 것을 우선시합니다. |      |

표 2: PX4 `MPC_YAW_MODE` 파라미터 옵션 22

이 표는 PX4의 기본 Yaw 로직과 그것이 ROI 명령어와 어떻게 독립적으로 유지되는지를 사용자가 이해하는 데 필수적이며, 이는 핵심적인 차이점입니다.

### 3.4  `DO_SET_ROI`: 짐벌 우선 해석

PX4에서 `DO_SET_ROI` 명령이 실행되면, 비행 스택의 주된 동작은 목표 좌표를 구성된 마운트 드라이버에 전달하는 것입니다.19 기체의 Yaw는 `MPC_YAW_MODE`에 의해 계속 제어됩니다. 만약 짐벌이 구성되지 않았다면(`MNT_MODE_IN` 비활성화), 이 명령은 종종 무시되어 사용자가 관찰한 것과 같은 동작으로 이어집니다. 이는 기체가 ROI를 향하지 않는다고 보고하는 GitHub 이슈 토론에서도 확인됩니다.18 즉, PX4는 `DO_SET_ROI`를 `DO_MOUNT_CONTROL_BY_LOCATION` (위치 기반 마운트 제어)으로 해석합니다.

### 3.5  예외: PX4에서 기체 Yaw 강제하기

PX4에서 기체가 ROI를 향해 Yaw하도록 만드는 것이 가능하기는 하지만, 이는 기본 동작이 아니며 간단하지 않을 수 있습니다. 한 개발자는 `MIS_YAWMODE` 파라미터(현재의 `MPC_YAW_MODE`의 이전 이름)의 "heading towards ROI" 설정이 "짐벌에 Yaw 기능이 없음" 설정으로 간주될 수 있다고 제안합니다.18 이는 시스템이 짐벌 Yaw 기능이 없다는 것을 인지하면 기체 Yaw를 사용하는 것으로 대체할 수 있음을 시사합니다. 그러나 다른 GitHub 토론에서는 이 기능이 일관되게 구현되지 않았으며, 특히 VTOL과 같은 특정 기체 유형의 경우 이 동작을 달성하기 위해 코드 수정에 의존해야 했다고 밝히고 있습니다.24 이는 기본적으로 짐벌 전용이지만, 기체 Yaw를 달성하기 위한 경로(및 해결 방법)가 있다는 중요한 뉘앙스를 제공합니다.

### 3.6  `CONDITION_YAW` 구현의 어려움

포럼 게시물들은 PX4 임무에서 `MAV_CMD_CONDITION_YAW`를 사용하는 데 있어 상당한 사용자 혼란과 어려움이 있음을 보여줍니다.6 사용자들은 오류 없이 명령을 전송했지만 드론이 회전하지 않는다고 보고합니다. 이는 PX4의 임무 모드에서 이 명령어에 대한 지원이 불완전하거나, 버그가 있거나, 직관적이지 않음을 시사합니다. 한 GitHub 이슈에서 개발자들은 이 명령어의 모호성과 해석 방식에 대해 논의하며, Yaw 오프셋을 명령하는 더 나은 방법으로 새로운 ROI 유형을 제안하기도 했습니다.26

이는 명시적인 기체 Yaw 제어를 원하는 사용자에게 PX4 생태계의 주요 약점을 드러냅니다. 주된 ROI 명령어(`DO_SET_ROI`)가 Yaw를 제어하지 않고, 명시적인 Yaw 명령어(`CONDITION_YAW`)가 임무에서 신뢰할 수 없다면, 사용자는 원하는 동작을 달성하기 위한 간단하고 견고한 방법 없이 남겨지게 됩니다. 이는 ArduPilot에 비해 심각한 기능적 격차입니다. PX4의 아키텍처적 "순수성"은 개발자에게는 예측 가능한 시스템을 만들지만, 단순한 하드웨어를 가진 운영자에게는 덜 직관적인 시스템을 만듭니다. 이러한 설계 선택은 기능적 격차(신뢰할 수 없는 `CONDITION_YAW`)와 원래 아키텍처의 우아함을 훼손하는 해결 방법에 대한 의존으로 이어지는 파급 효과를 낳습니다.

## 4.  비교 분석 및 종합

이 섹션에서는 이전 섹션들의 내용을 종합하여 두 시스템을 직접 비교하고, 명확하고 고수준의 개요를 제공합니다.

### 4.1  기능 심층 분석: 기본 동작 비교

ArduPilot과 PX4의 `DO_SET_ROI`에 대한 반응은 그들의 근본적인 설계 철학을 반영합니다. ArduPilot은 기본적으로 기체 Yaw를 제어하며, 이는 지속적인 상태로 유지되고 짐벌이 있는 경우 짐벌과 협응합니다. 반면, PX4는 기본적으로 짐벌을 제어하며, 기체 Yaw는 `MPC_YAW_MODE`에 의해 설정된 대로 유지됩니다. PX4에서 기체 Yaw를 이용한 ROI 포인팅은 예외적인 경우로, 특정 파라미터 설정이나 코드 변경이 필요할 수 있습니다. 아래 표는 이러한 핵심적인 차이점을 요약합니다.

| 항목                                    | ArduPilot                | PX4                                                   |
| --------------------------------------- | ------------------------ | ----------------------------------------------------- |
| **핵심 철학**                           | 기체 중심, 통합 제어     | 페이로드 중심, 분리된 제어                            |
| **주 포인팅 명령어**                    | `DO_SET_ROI` (기체+짐벌) | `DO_SET_ROI` (마운트/짐벌에 위임)                     |
| **`DO_SET_ROI` 수신 시 기본 기체 동작** | ROI를 향해 Yaw           | `MPC_YAW_MODE`에 따라 Yaw 유지                        |
| **`DO_SET_ROI` 수신 시 기본 짐벌 동작** | ROI를 향해 Pitch         | ROI를 향해 Pitch 및 Yaw                               |
| **핵심 Yaw 파라미터**                   | `WP_YAW_BEHAVIOR`        | `MPC_YAW_MODE`                                        |
| **ROI를 위한 기체 Yaw 구현 용이성**     | 기본 동작으로 매우 쉬움  | 기본 동작이 아니며, 해결 방법이 필요할 수 있어 어려움 |

*표 3: ArduPilot 대 PX4의 ROI 동작 - 비교 요약*

### 4.2  역사적 배경과 아키텍처의 분기

두 프로젝트가 서로 다른 철학에 도달한 배경을 이해하는 것은 중요합니다. ArduPilot은 DIY/취미 커뮤니티에서 시작되었으며 1, 이는 다양한 기체 유형에 대해 실용적이고 풍부하며 성숙한 기능 세트로 이어졌습니다.10 GPL 라이선스는 커뮤니티에 대한 기여 공유를 장려합니다.1 반면, PX4는 연구와 보다 공식적인 소프트웨어 아키텍처에 중점을 두고 시작되었으며 27, 허용적인 BSD 라이선스는 자신들의 수정 사항을 독점적으로 유지하고자 하는 상업 기업들에게 더 매력적입니다.1 ROI 동작의 차이는 이러한 더 큰 분기의 완벽한 축소판입니다. ArduPilot의 실용적이고 기능이 풍부한 접근 방식은 커뮤니티 주도의 반복적인 진화의 산물이며, PX4의 깨끗하고 모듈화된 아키텍처는 보다 학술적이고 상업 지향적인 기원의 산물입니다.

### 4.3  드론 시스템 통합 전문가를 위한 실질적 시사점

시스템 통합 전문가의 관점에서, ArduPilot을 선택하는 것은 POI 임무와 같은 일반적인 작업을 간단한 하드웨어로도 "즉시(out of the box)" 작동하는 시스템을 얻는 것을 의미합니다. 그 대가는 더 단일체적이고 얽혀있는 제어 시스템입니다.10 PX4를 선택하는 것은 복잡하고 맞춤화된 애플리케이션(예: 고급 센서나 온보드 컴퓨터 통합)을 위한 더 유연하고 모듈화된 기반을 제공하지만, 하드웨어가 "이상적인" 구성(3축 짐벌을 갖춘 기체)과 일치하지 않을 경우 기본 동작을 구성하는 데 더 많은 노력이 필요합니다.1 PX4 임무에서 간단한 기체 Yaw를 달성하는 데 따르는 어려움은 반복적으로 제기되는 문제입니다.6

결론적으로, 특정 프로젝트에 대해 ArduPilot과 PX4 중 하나를 선택하는 것은 단순히 기술적인 결정이 아니라 프로젝트의 자원 제약, 하드웨어 철학 및 개발 방법론을 반영하는 전략적인 결정입니다. ROI 동작 문제는 이러한 더 깊은 전략적 차이를 드러내는 "탄광 속의 카나리아"와 같습니다. 이 단일 문제를 통해 전체 개발 경험을 추론할 수 있습니다. 프로젝트가 촉박한 일정에 있고 일반적인 상용 하드웨어를 사용한다면, ArduPilot의 "모든 것이 포함된(batteries-included)" 접근 방식이 위험 부담이 적을 가능성이 높습니다. 프로젝트가 새로운 하드웨어, 심층적인 맞춤화, 그리고 독점 IP 보호를 포함한다면, PX4의 모듈성과 BSD 라이선스가 더 유리하지만, ArduPilot의 "즉시 사용 가능한" 기능 중 일부를 복제하는 데 필요한 추가 통합 노력에 대한 예산을 책정해야 합니다.

## 5.  권장 사항 및 고급 구성

이 마지막 섹션에서는 사용자가 각 비행 스택에서 원하는 동작을 달성하기 위한 실행 가능한 조언을 제공합니다.

### 5.1  PX4에서 ArduPilot의 동작 모방하기

PX4 기체가 ROI를 향해 Yaw하도록 만들려면, 시스템이 "Yaw 기능이 없는 짐벌"을 가지고 있다고 인지하도록 구성하는 것이 주된 방법입니다.

**단계:**

1. 물리적인 짐벌이 없더라도 마운트 드라이버를 활성화해야 합니다 (`MNT_MODE_IN` 및 `MNT_MODE_OUT` 구성). 이는 직관에 반하지만, 포인팅 로직을 작동시키기 위해 필요합니다.
2. `MPC_YAW_MODE` 파라미터를 4번 옵션인 "Towards Waypoint (Yaw First)"로 설정하거나, 특정 펌웨어 버전에서 사용 가능한 경우 ROI를 향해 헤딩을 강제하는 다른 값으로 설정합니다.18
3. VTOL의 경우 또는 위 방법이 실패할 경우, 개발자들이 보여준 가장 신뢰할 수 있는 방법은 표준 임무 명령어가 이 작업에 신뢰할 수 없으므로 코드 수정이나 스크립팅을 사용하여 기본 Yaw 동작을 재정의하는 것입니다.24
4. 대안으로 `DO_SET_ROI`를 포기하고, 주기적으로 필요한 Yaw를 계산하여 `MAV_CMD_CONDITION_YAW` 명령을 보내는 방법이 있지만, 이는 복잡하고 임무 모드에서 신뢰성 있게 작동하지 않을 수 있습니다.6

### 5.2  ArduPilot에서 PX4의 동작 모방하기

ArduPilot 기체가 Yaw를 하지 않고 오직 짐벌만 사용하여 ROI를 처리하도록 하려면, 사용자가 이를 명시적으로 명령해야 합니다.

**단계:**

1. `WP_YAW_BEHAVIOR` 파라미터를 0 ("Never change yaw")으로 설정합니다.
2. `DO_SET_ROI` 명령어 대신 `DO_MOUNT_CONTROL` 명령어를 사용합니다. 이 명령어는 특별히 짐벌을 대상으로 하며 기체 자세에 영향을 주지 않습니다. 이는 "GPS point mode" 대신 "Mavlink targeting mode"를 직접 사용하는 것입니다.9
3. 이 접근 방식은 기체와 페이로드 제어를 깨끗하게 분리하여 ArduPilot 프레임워크 내에서 PX4와 유사한 동작을 달성합니다.

### 5.3  특정 기체 유형(멀티로터 대 VTOL)에 대한 고려 사항

PX4의 VTOL 헤딩 제어에 대한 GitHub 이슈는 특히 시사하는 바가 큽니다.24 표준 멀티로터에서 작동하는 방법이 VTOL의 멀티콥터 모드에서는 실패할 수 있음을 보여줍니다. 개발자는 "VTOL에서 헤딩은 MC 모드일 때 중요한 파라미터로 간주되지 않는다"고 언급했습니다.

따라서 PX4에서 VTOL과 같은 비표준 기체로 작업하는 통합 전문가는 기본 동작이 적용되지 않을 수 있다고 가정해야 합니다. 특정 헤딩 제어를 달성하기 위해 상당한 테스트와 잠재적으로 펌웨어 수준의 맞춤화에 대한 예산을 책정해야 합니다. ArduPilot의 보다 통합된 제어 시스템은 일반적으로 다른 콥터 기반 기체 유형에 걸쳐 더 일관된 동작을 제공합니다.

## 6. 결론: 두 가지 철학, 두 가지 해결책

결론적으로, ROI 동작의 차이는 한 시스템이 "옳고" 다른 시스템이 "그르다"는 문제가 아닙니다. 이는 두 개의 서로 다르지만 타당한 설계 철학의 직접적이고 논리적인 결과입니다. ArduPilot은 통합된, 즉시 사용 가능한 기능성을 우선시하는 반면, PX4는 모듈성과 아키텍처적 분리를 우선시합니다.

이 핵심적인 차이점을 이해하면 시스템 통합 전문가는 자신의 프로젝트에 적합한 플랫폼을 선택하고, 기체 Yaw, 짐벌 제어 또는 이 둘의 조화로운 조합을 통해 원하는 포인팅 동작을 효과적으로 구성할 수 있는 역량을 갖추게 됩니다. 사용자의 초기 관찰, 즉 ArduPilot은 기체를 돌리고 PX4는 짐벌만 돌린다는 것은 두 오픈소스 거인이 드론 제어라는 동일한 문제에 대해 각기 다른 경로를 통해 어떻게 정교한 해결책을 제시했는지를 보여주는 명백한 증거입니다.

#### **참고 자료**

1. ArduPilot VS PX4 | Which Is Better? - Drone Dojo, accessed July 14, 2025, https://dojofordrones.com/ardupilot-vs-px4/
2. Missions (AUTO Mode) - DroneKit-Python's documentation!, accessed July 14, 2025, https://dronekit-python.readthedocs.io/en/latest/guide/auto_mode.html
3. mavros_msgs/CommandCode Message - ROS 2, accessed July 14, 2025, http://docs.ros.org/en/noetic/api/mavros_msgs/html/msg/CommandCode.html
4. Yaw orientation during mission - Discussions - diydrones, accessed July 14, 2025, https://diydrones.com/forum/topics/yaw-orientation-during-mission
5. Can I use condition_yaw(mission planner command) to rotate it like the picture?, accessed July 14, 2025, https://discuss.ardupilot.org/t/can-i-use-condition-yaw-mission-planner-command-to-rotate-it-like-the-picture/83275
6. Yaw Control with MAV_CMD_CONDITION_YAW - MAVLink - PX4 Discussion Forum, accessed July 14, 2025, https://discuss.px4.io/t/yaw-control-with-mav-cmd-condition-yaw/44948
7. Mission Commands — Copter documentation - ArduPilot, accessed July 14, 2025, https://ardupilot.org/copter/docs/common-mavlink-mission-command-messages-mav_cmd.html
8. Missions | PX4 User Guide (v1.12), accessed July 14, 2025, https://docs.px4.io/v1.12/en/flying/missions.html
9. How to setup Gremsy gimbal with Pixhawk Cube?, accessed July 14, 2025, https://support.gremsy.com/support/discussions/topics/36000012579
10. PX4 vs ArduPilot - when to choose what - General, accessed July 14, 2025, https://discuss.ardupilot.org/t/px4-vs-ardupilot-when-to-choose-what/14262
11. PX4 vs. ArduPilot: Choosing the Right Open-Source Flight Stack |, accessed July 14, 2025, https://www.thedroningcompany.com/blog/px4-vs-ardupilot-choosing-the-right-open-source-flight-stack
12. ArduCopter: make ROI persistent across waypoints · Issue #569 - GitHub, accessed July 14, 2025, https://github.com/ArduPilot/ardupilot/issues/569
13. Gimbal Control in Auto Mode | Do Set ROI - Copter 3.5 - ArduPilot Discourse, accessed July 14, 2025, https://discuss.ardupilot.org/t/gimbal-control-in-auto-mode-do-set-roi/23009
14. Complete Parameter List — Sub documentation - ArduPilot, accessed July 14, 2025, https://ardupilot.org/sub/docs/parameters-Sub-stable-V4.1.2.html
15. Complete Parameter List — Copter documentation - ArduPilot, accessed July 14, 2025, https://ardupilot.org/copter/docs/parameters-Copter-stable-V4.1.0.html
16. WP_YAW_BEHAVIOR not affecting orientation - ArduCopter - ArduPilot Discourse, accessed July 14, 2025, https://discuss.ardupilot.org/t/wp-yaw-behavior-not-affecting-orientation/21174
17. Gimbal control with ROI using Pixhawk - Archive - ArduPilot Discourse, accessed July 14, 2025, https://discuss.ardupilot.org/t/gimbal-control-with-roi-using-pixhawk/4900
18. Mission: MAV_CMD_DO_SET_ROI not working · Issue #8541 · PX4/PX4-Autopilot - GitHub, accessed July 14, 2025, https://github.com/PX4/Firmware/issues/8541
19. Gimbal Configuration | PX4 Guide (main) - PX4 docs, accessed July 14, 2025, https://docs.px4.io/main/en/advanced/gimbal_control.html
20. Gimbal Configuration | PX4 Guide (v1.15), accessed July 14, 2025, https://docs.px4.io/v1.15/en/advanced/gimbal_control.html
21. Missions | PX4 User Guide - GitBook, accessed July 14, 2025, https://px4.gitbook.io/px4-user-guide/flying/missions
22. Parameter Reference | PX4 User Guide (v1.12) - PX4 docs, accessed July 14, 2025, https://docs.px4.io/v1.12/en/advanced_config/parameter_reference.html#MPC_YAW_MODE
23. MPC_YAW_MODE 0 vs 3 - PX4 Autopilot, accessed July 14, 2025, https://discuss.px4.io/t/mpc-yaw-mode-0-vs-3/21162
24. Heading issue on PX4 in mission · Issue #13984 · PX4/PX4-Autopilot, accessed July 14, 2025, https://github.com/PX4/Firmware/issues/13984
25. Set vehicle yaw without cancelling running commands - MAVSDK - PX4 Discussion Forum, accessed July 14, 2025, https://discuss.px4.io/t/set-vehicle-yaw-without-cancelling-running-commands/39044
26. Add MAV_CMD_CONDITION_YAW · Issue #8228 · PX4/PX4 ..., accessed July 14, 2025, https://github.com/PX4/Firmware/issues/8228
27. Request:. Feature-by-Feature comparison of Ardupilot and PX4 firmware stacks : r/diydrones, accessed July 14, 2025, https://www.reddit.com/r/diydrones/comments/4wzxtv/request_featurebyfeature_comparison_of_ardupilot/


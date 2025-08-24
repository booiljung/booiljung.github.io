[PX4-Autopilot](./index.md)
# PX4 Autopilot v1.15 포괄적 기술 검토



PX4 오토파일럿은 전문가 및 연구용 애플리케이션을 위한 선도적인 오픈 소스 비행 제어 시스템으로, 멀티콥터부터 수직이착륙기(VTOL), 로버, 잠수정에 이르기까지 광범위한 무인 기체에 동력을 제공합니다.1 이러한 생태계에서 v1.15는 v1.14에 이은 주요 안정화 릴리스(stable release)로, 2024년 9월경에 공식 발표되었습니다.4 이 릴리스는 개발자 경험 향상, 외부 로보틱스 생태계(특히 ROS 2)와의 심층적인 통합, 상태 추정 시스템의 근본적인 개편, 그리고 혁신적인 실험적 기능 도입이라는 명확한 주제를 중심으로 구성되었습니다.4

v1.15는 단순히 기능을 추가하는 것을 넘어, PX4 플랫폼의 전략적 방향성을 제시하는 중요한 이정표입니다. 이전 릴리스인 v1.14가 동적 제어 할당(Dynamic Control Allocation)이나 uXRCE-DDS 미들웨어 도입과 같이 시스템의 내부 기반을 다지는 데 중점을 두었다면 8, v1.15는 이 기반 위에서 외부 시스템과의 연결성 및 오프보드(offboard) 인텔리전스 역량을 강화하는 데 초점을 맞추었습니다. 이는 비행 컨트롤러의 핵심 기능 개선을 넘어, 더 크고 복잡한 로보틱스 시스템 내에서 PX4를 원활한 구성 요소로 자리매김하려는 전략적 변화를 의미합니다.


PX4 v1.15는 개발자 중심의 전략적 전환을 명확히 보여줍니다. v1.14에서 ROS 2와의 견고한 통신을 위한 인프라인 uXRCE-DDS 브리지를 도입한 후 8, v1.15는 이를 적극적으로 활용하여 ROS 2 통합을 위한 고수준 C++ 라이브러리를 선보였습니다.4 이 라이브러리는 외부 ROS 2 노드를 PX4 내부 비행 모드와 동등한 '피어(peer)'로 취급할 수 있게 하는 아키텍처적 선언이며, 이는 PX4가 단순한 비행 제어기를 넘어 고성능 컴패니언 컴퓨터에서 실행되는 고급 AI 및 컴퓨터 비전 애플리케이션의 비행 제어 허브로 자리매김하려는 의도를 드러냅니다.10 실험적인 Zenoh Pico 프로토콜 지원 추가 4 역시 이러한 경향을 뒷받침하며, 차세대 경량 통신 프로토콜을 적극적으로 탐색하고 있음을 보여줍니다.

그러나 v1.15는 '최첨단 기능'과 '기반 안정성' 사이의 뚜렷한 이중성을 보입니다. 릴리스 노트는 'Throw Mode'나 'ROS 2 인터페이스 라이브러리'와 같은 혁신적이지만 '실험적(experimental)' 딱지가 붙은 기능들을 대대적으로 홍보합니다.4 동시에 커뮤니티 포럼과 GitHub 이슈 트래커에는 GPS 탐지 실패, 오프보드 제어 중 추락 등 기본적인 기능의 심각한 오류들이 다수 보고되었습니다.12 이는 v1.15가 연구개발(R&D) 환경에서는 강력한 도구가 될 수 있지만, 상용 환경에서 요구되는 높은 수준의 신뢰성을 확보하는 데에는 상당한 어려움을 겪었음을 시사합니다. 결과적으로 v1.15는 기능적 도약을 이루었지만, 그 과정에서 일시적인 안정성 저하를 감수한 전환기적 릴리스로 평가할 수 있습니다. 이러한 평가는 기술 도입을 고려하는 전문가들에게 중요한 시사점을 제공합니다.



PX4 v1.15의 가장 중요한 내부 아키텍처 변화는 새로운 오차 상태 칼만 필터(Error-State Kalman Filter, ESKF) 유도 방식으로의 전환입니다.4 이는 단순한 개선이 아닌, 상태 추정 시스템의 근본적인 재설계에 가깝습니다.


새로운 EKF는 여러 이론적, 실용적 이점을 목표로 설계되었습니다.

- **수렴성 향상**: 초기 헤딩(heading) 정확도에 대한 민감도를 낮춰, 열악한 조건에서도 필터가 더 빠르고 안정적으로 수렴하도록 설계되었습니다.4
- **수치적 안정성**: 'Joseph 안정화 공분산 업데이트 알고리즘'을 도입하여 공분산의 안정성을 높였습니다. 또한, SymForce와 같은 도구를 사용하여 공분산 예측, 측정 야코비안 등을 자동으로 생성함으로써 사람의 실수로 인한 오류 가능성을 줄이고 수학적 정확성을 보장했습니다.4
- **계산 효율성**: 이전 필터 대비 계산 비용을 절감하여 제한된 성능의 비행 컨트롤러에서도 더 효율적으로 동작할 수 있도록 했습니다.4
- **이론적 정확성**: 리 군(Lie group) 이론을 사용하여 쿼터니언(quaternion) 불확실성을 수학적으로 올바르게 표현함으로써, 자세 추정의 정확도를 근본적으로 개선했습니다.4


비행 안정성에 큰 영향을 미치는 자력계(magnetometer) 처리 방식 또한 크게 개선되었습니다.

- **새로운 'Init' 모드**: `EKF2_MAG_TYPE` 파라미터에 'Init' 모드가 추가되었습니다. 이 모드에서는 자력계가 오직 초기 헤딩을 정렬하는 데만 사용되고, 이후에는 자이로와 GNSS 데이터에 의존하여 헤딩을 추정합니다. 이는 자기장 간섭이 심한 환경에서 비행 안정성을 획기적으로 높일 수 있는 중요한 기능입니다.4
- **자력계 상태 검사 강화**: `EKF2_MAG_CHECK` 관련 파라미터가 세분화되어, 자기장 강도 허용 오차(`EKF2_MAG_CHK_STR`)와 경사각 검사(`EKF2_MAG_CHK_INC`)를 개별적으로 설정할 수 있게 되어 더 정밀한 자력계 상태 모니터링이 가능해졌습니다.4
- **헤딩 수렴 속도 개선**: GNSS 보조를 사용할 때, 부정확한 초기 헤딩 값으로 시작했더라도 더 빠르게 정확한 헤딩으로 수렴하도록 알고리즘이 개선되었습니다.4


기존의 복잡했던 `SYS_MC_EST_GROUP` 파라미터가 제거되고, `EKF2_EN`, `ATT_EN`, `LPE_EN`과 같이 각 추정기를 직접 활성화하는 전용 파라미터가 도입되었습니다. 이는 사용자가 자신의 시스템에 맞는 추정기를 더 직관적이고 명확하게 선택할 수 있도록 하여 설정 오류를 줄여줍니다.4


v1.15는 컴패니언 컴퓨터와의 연동을 강화하여 PX4를 더 큰 로보틱스 시스템의 핵심 요소로 통합하는 데 주력했습니다.

- **PX4 ROS 2 인터페이스 라이브러리**: 실험적으로 도입된 이 C++ 라이브러리는 uXRCE-DDS 통신의 복잡성을 추상화하고, 차량 제어 및 센서 데이터 접근을 위한 고수준 인터페이스를 제공합니다.4 이 라이브러리의 가장 혁신적인 점은 ROS 2 애플리케이션으로 작성된 비행 모드를 PX4 내부 모드와 동등한 '피어'로 취급한다는 것입니다. 즉, 외부에서 작성된 비행 모드가 페일세이프(failsafe) 상태 머신에 통합되고 지상관제시스템(GCS)에서 선택 가능해져, 사실상 네이티브 모드처럼 동작합니다.4 이는 컴패니언 컴퓨터의 강력한 연산 능력을 활용한 복잡한 자율 비행 로직 개발을 가능하게 합니다.
- **uXRCE-DDS 미들웨어 향상**: DDS 토픽 YAML 파일에서 `subscription_multi` 옵션을 사용할 수 있게 되었습니다. 이를 통해 ROS 2에서 오는 메시지를 별도의 uORB 토픽 인스턴스로 분리하여, 내부 uORB 메시지와의 충돌을 방지하고 데이터 흐름을 정밀하게 제어할 수 있게 되었습니다.4
- **실험적 Zenoh Pico 지원**: MAVLink나 DDS를 넘어 차세대 통신 아키텍처를 모색하는 일환으로, 경량의 pub/sub/query 프로토콜인 Zenoh에 대한 실험적 지원이 추가되었습니다.4


- **플랫폼 독립적인 UART 인터페이스**: 새로운 통합 UART 인터페이스가 도입되어, 하드웨어 포팅 및 드라이버 개발 과정을 단순화했습니다.4 이는 오랫동안 PX4 개발의 복잡성 요인 중 하나였던 직렬 통신 처리를 표준화하려는 중요한 시도입니다.14
- **uORB 메시지 용량 확장**: 기존 255개였던 uORB 메시지 최대 개수 제한이 해제되어, 더 많은 센서와 모듈을 사용하는 복잡한 시스템 설계가 가능해졌습니다.4

표 1: PX4 v1.15의 주요 파라미터 변경 사항 요약

| **파라미터 이름**             |
| ----------------------------- |
| `EKF2_EN`, `ATT_EN`, `LPE_EN` |
| `SYS_HAS_NUM_ASPD`            |
| `FW_USE_AIRSPD`               |
| `LNDMC_ALT_MAX`               |
| `VT_BT_TILT_DUR`              |
| `EKF2_MAG_TYPE`               |

이러한 아키텍처 변화는 PX4의 역할을 재정의합니다. 과거에는 모든 핵심 비행 로직이 비행 컨트롤러 내부에 존재하는 단일체(monolithic) 시스템에 가까웠지만 11, v1.15는 외부 컴퓨팅 자원을 안전하고 깊게 통합하는 분산 제어 시스템의 구성 요소로 진화하고 있음을 보여줍니다. 이는 PX4가 첨단 로보틱스 연구 및 개발을 위한 최고의 비행 스택으로 자리매김하는 데 결정적인 역할을 할 것입니다.

그러나 EKF2의 전면적인 개편은 양날의 검으로 작용했습니다. 이론적으로는 견고성을 목표로 했지만, 실제로는 새로운 형태의 미묘한 버그를 유발하는 주된 원인이 되었습니다. 예를 들어, GPS 융합이 비활성화된 상태에서만 고정익 착륙 감지가 실패하는 문제 16나, 양호한 GPS 신호에도 불구하고 EKF의 속도 정확도 검사 실패로 인해 'local position invalid' 페일세이프가 발생하는 문제 17 등은 EKF2 로직의 변경이 예상치 못한 부작용을 낳았음을 명확히 보여줍니다. 이는 장기적으로는 더 안정적인 시스템을 위한 투자이지만, 단기적으로는 v1.15를 비전문가 사용자가 채택하기에 위험한 릴리스로 만들었습니다.


v1.15는 핵심 아키텍처의 급진적인 변화와는 대조적으로, 사용자 대면 기능 측면에서는 기존 역량을 점진적으로 개선하고 다듬는 데 중점을 두었습니다.


- **실험적 'Throw Mode'**: 멀티콥터를 공중으로 던져서 시동을 거는 새로운 이륙 방식입니다. 특정 시나리오에서 수동 이륙 절차를 생략하고 신속하게 기체를 배포할 수 있는 잠재력을 가집니다.7 다만 '실험적' 기능인 만큼, 사용 시 각별한 주의가 필요합니다.
- **'Position Slow Mode'**: 표준 'Position Mode'의 저속 버전으로, 사용자가 최대 수평/수직 속도 및 요(yaw) 속도를 낮게 설정할 수 있습니다. 이는 정밀한 제어가 요구되는 실내 비행, 근접 촬영, 또는 섬세한 페이로드 운반 시 안전성과 제어성을 높여줍니다.7
- **단방향 킬 제스처(One-Way Kill Gesture)**: 안전 기능으로, 어떠한 상황에서도 모터를 즉시 정지시킬 수 있는 새로운 제스처가 추가되었습니다. 오작동을 방지하기 위해 기본적으로는 비활성화되어 있습니다.4


- **VTOL 성능 향상**:
  - 후방 전환(back-transition) 시 틸팅 시간을 파라미터(`VT_BT_TILT_DUR`)로 노출하여, 전진 비행에서 호버링으로의 전환을 더 빠르고 부드럽게 튜닝할 수 있게 되었습니다.4
  - 테일시터(Tailsitter) 기체의 'Stabilized' 모드에서 자동 피치 램프(pitch ramp) 기능이 추가되어, 수동 제어 시 전환 과정의 안정성이 향상되었습니다.4
  - 비상 상황 시 호버링 모드로 전환하는 'Quad-Chute' 페일세이프 로직이 개선되었습니다.4
- **고정익 성능 향상**:
  - 에어스피드 센서 구성을 `CBRK_AIRSPD_CHK` 대신 `SYS_HAS_NUM_ASPD` 파라미터를 사용하도록 변경하여, 다른 센서들과의 일관성을 맞추고 설정을 더 직관적으로 만들었습니다.4
  - '강풍 대응(high wind hardening)' 기능이 개선되어 까다로운 기상 조건에서도 더 안정적인 비행이 가능해졌습니다.7


- **지오펜스(Geofence) 개선**: 지오펜스 시스템이 재작성되어 경계 유지 기능이 더 견고해지고 복잡한 형태의 금지 구역 설정이 개선되었습니다.4 이 과정에서 불필요했던 

  `LNDMC_ALT_MAX` 파라미터가 지오펜스 기반의 `GF_MAX_VER_DIST`로 통합되어 설정이 간소화되었습니다.15

- **임무 재개 기능 강화**: 측량 임무 등이 중단되었다가 재개될 때, 이전의 짐벌 및 카메라 명령을 다시 재생하는 기능이 추가되었습니다. 이는 데이터 수집 페이로드의 작동 신뢰도를 높여 임무 실패율을 줄여줍니다.4

- **대형 기체 나침반 보정**: 모든 축으로 회전시키기 어려운 대형 기체를 위해 설계된 새로운 나침반 보정 루틴이 도입되었습니다. 이는 산업용 및 중량물 운송 드론에 필수적인 기능입니다.4

이러한 기능들은 대부분 기존 기능의 사용성을 개선하고 완성도를 높이는 데 초점이 맞춰져 있습니다. 이는 v1.15의 개발 자원이 새로운 비행 행동을 광범위하게 추가하기보다는, EKF2 재설계나 ROS 2 아키텍처와 같은 심층적인 내부 작업에 집중적으로 투입되었음을 시사합니다. 이는 장기적으로 플랫폼의 건강성을 위한 올바른 전략이지만, v1.15 릴리스 자체에서는 혁신적인 신규 비행 모드가 상대적으로 부족하게 보이는 원인이 되었습니다.



v1.15는 최신 하드웨어에 대한 지원을 확장하여 생태계의 성장을 지속했습니다.

- **비행 컨트롤러**: FMUv6X-RT(Pixhawk 6X-RT), MicoAir H743 등 새로운 보드에 대한 공식 지원이 추가되었습니다.4
- **센서**: ARK Mosaic-X5 RTK GPS, TDK IIM42653 IMU와 같은 고성능 신규 센서들이 통합되었습니다.4
- **주변기기**: LP5562 RGBLED 드라이버, Auterion 파워 모듈 등 다양한 주변기기에 대한 지원이 추가되었습니다.4


하드웨어 지원 확장에도 불구하고, v1.15는 드라이버 레벨에서 심각한 안정성 문제를 드러냈습니다.

- **GPS 탐지 버그**: v1.15에서 가장 광범위하게 보고된 치명적인 버그입니다.
  - **문제 현상**: v1.15로 펌웨어를 업그레이드하자, Cube Orange, PixRacer Pro 등 다양한 비행 컨트롤러에서 기존에 잘 작동하던 GPS 모듈이 더 이상 인식되지 않는 문제가 발생했습니다.13
  - **근본 원인**: 이 문제의 원인은 파라미터 충돌이었습니다. 새롭게 추가된 Septentrio GPS 드라이버가 기본적으로 `SEP_PORT1_CFG` 파라미터를 'GPS 1'로 설정하면서, 기존의 모든 GPS 드라이버가 사용하던 `GPS_1_CONFIG` 파라미터와 충돌을 일으켜 포트 초기화에 실패한 것입니다.13
  - **임시 해결책과 그 결과**: 커뮤니티에서 `SEP_PORT1_CFG`를 수동으로 'Disabled'로 설정하는 해결책이 발견되었습니다.13 하지만 이 해결책을 적용한 일부 사용자들은 GPS 스푸핑 경고, 비행 중 불안정 등 더 심각한 문제를 겪었고, 결국 추락 후 안정적인 v1.14.4로 롤백해야 했습니다.13 이는 임시방편이 증상은 해결했지만, 더 깊은 근본 원인을 건드렸음을 보여줍니다. 이후 패치 릴리스를 통해 공식적인 수정이 이루어졌습니다.13
- **MAVLink 주변기기 비호환성**: v1.14에서 완벽하게 작동하던 낙하산 시스템과 같은 MAVLink 기반 주변기기들이 v1.15로 업그레이드 후 통신이 두절되는 사례가 보고되었습니다.21 이는 MAVLink 프로토콜 처리나 UART 드라이버 설정에 미묘하지만 호환성을 깨뜨리는 변화가 있었음을 시사합니다.
- **SD 카드 타임아웃 오류**: Kakute H7 mini와 같이 SD 카드 슬롯이 없는 보드에서 'Operation timeout, aborting transfer' 오류가 발생하여 QGC를 통한 파라미터 설정이 불가능해지는 버그가 v1.15에서 새롭게 나타났습니다.22

GPS 탐지 버그는 의도는 좋았지만(새 드라이버의 즉각적인 사용성 확보) 예기치 못한 부작용을 낳은 대표적인 사례입니다. 이는 복잡하게 결합된 시스템에서 작은 변화가 어떻게 전체 시스템에 치명적인 영향을 미칠 수 있는지 보여줍니다. 하나의 새 드라이버 기본 설정이 다른 모든 GPS 사용자의 시스템을 마비시킨 이 사건은, 방대한 하드웨어 생태계를 가진 오픈 소스 프로젝트가 새로운 기여 요소가 기존 설정 공간과 충돌하지 않도록 보장하는 것이 얼마나 중요한지를 명확히 보여줍니다. 이는 안정화 릴리스 선언 전에 더 광범위하고 철저한 통합 테스트의 필요성을 강조하는 사례입니다.


v1.15의 진정한 평가는 릴리스 노트가 아닌, 실제 사용자들이 마주한 현실에 기반해야 합니다. 커뮤니티 피드백은 이 릴리스의 강점과 약점을 가장 명확하게 보여주는 바로미터입니다.


표 2: PX4 v1.15에서 보고된 주요 문제점 종합

| **문제 범주**            |
| ------------------------ |
| GPS 탐지 실패            |
| 오프보드 모드 불안정     |
| 고정익 착륙 감지 실패    |
| 광학 흐름 위치 유지 실패 |
| SD 카드 타임아웃         |

- **오프보드 모드 불안정성**: 사용자들이 보고한 가장 심각한 문제 중 하나는 오프보드 모드에서의 치명적인 오작동입니다. 기체가 목표 고도에 도달한 후 첫 번째 수평 이동 명령을 받자마자 급강하하며 추락하는 현상이 보고되었습니다.12 개발팀의 초기 분석은 모터/ESC 문제를 지적했지만, 사용자는 수동 모드에서는 문제가 없음을 확인했습니다. 특히 이 문제는 시뮬레이션에서는 전혀 재현되지 않았다는 점에서 12, 시뮬레이션과 실제 하드웨어 간의 괴리를 명확히 보여줍니다. 이는 v1.15의 제어 로직이 실제 센서 노이즈와 지연 시간 하에서 불안정해지는 문제가 있음을 시사합니다.
- **고정익 착륙 감지 실패**: GitHub 이슈 #23746에서 보고된 이 문제는 EKF2 개편의 직접적인 부작용입니다. v1.14에서는 문제가 없었으나, v1.15에서는 GPS 없이 비행하는 고정익 기체가 착륙을 감지하지 못하는 심각한 회귀(regression) 버그가 발생했습니다.16 이는 착륙 감지 로직이 자신도 모르게 전역 위치 추정값에 의존하게 되었음을 의미하며, GPS가 없는 환경에서의 운용에 치명적입니다.
- **광학 흐름 위치 유지 실패**: v1.15에서 광학 흐름 센서를 이용한 실내 위치 유지가 불가능했다가, 동일한 하드웨어로 v1.16으로 업그레이드하자마자 정상 작동했다는 보고 23는 v1.15의 광학 흐름 융합 파이프라인에 명백한 버그가 존재했음을 증명합니다.


- **롤백(Rollback) 경향**: 다수의 사용자들이 v1.15의 불안정성 때문에 결국 안정적인 비행 운용을 위해 v1.14.4로 펌웨어를 되돌렸다고 보고했습니다.13 이는 v1.15가 특정 사용자 그룹에게는 실사용에 부적합했다는 가장 강력한 증거입니다.
- **설정의 어려움**: GPS 탐지 문제나 SD 카드 타임아웃 버그와 같은 기본적인 문제들은 사용자들이 v1.15의 새로운 기능들을 경험하기도 전에 큰 진입 장벽으로 작용했습니다.

v1.15에서 드러난 여러 심각한 버그들은 시뮬레이션 기반 테스트의 한계를 명확히 보여줍니다. 오프보드 모드 추락이나 고정익 착륙 감지 실패와 같은 문제들은 실제 하드웨어와 실제 비행 환경에서만 나타났습니다. 이는 Ascend Engineering과의 비행 테스트 협력 24과 같은 노력이 왜 중요한지를 역설하며, 안정화 릴리스 선언 전에 다양한 하드웨어 조합에 대한 광범위한 커뮤니티 베타 테스트 프로그램의 필요성을 제기합니다.



PX4 v1.15는 개발자 중심 기능과 아키텍처 측면에서 기념비적인 릴리스였습니다. 특히 ROS 2와의 깊은 통합과 핵심 상태 추정 엔진의 전면적인 개편은 플랫폼의 미래를 위한 과감한 투자였습니다. 하지만 이러한 야심 찬 시도는 초기 릴리스의 심각한 안정성 문제와 기능 회귀라는 대가를 치렀습니다. 이로 인해 v1.15는 상용 및 프로덕션 환경에서는 채택하기 어려운, '강력하지만 결함이 있는' 버전으로 평가됩니다. 이 릴리스는 플랫폼의 잠재력을 한 단계 끌어올렸지만, 그 안정성을 확보하기 위해서는 후속 패치 릴리스와 차기 버전(v1.16)의 노력이 반드시 필요했습니다.


- **연구개발(R&D) 및 학계**: 이 그룹에게 v1.15는 불안정성에도 불구하고 매우 가치 있는 릴리스였습니다. 새로운 ROS 2 도구와 EKF2의 개선점은 첨단 자율 비행 연구를 위한 강력한 플랫폼을 제공했습니다. 이들에게는 기능적 이점이 안정성 위험을 상회했을 가능성이 높습니다.
- **상업용 운영자 및 시스템 통합 업체**: 이 그룹에게는 v1.15에 대한 신중한 접근이 권장되었어야 합니다. 미션 크리티컬한 실패를 피하기 위해서는 v1.14의 장기 지원 버전을 유지하거나, v1.15.4와 같이 충분히 안정화된 후기 패치 릴리스를 기다리는 것이 현명한 선택이었습니다.


v1.15에서 드러난 문제들은 v1.16 이후의 로드맵을 명확하게 제시합니다. 최우선 과제는 v1.15에서 실험적으로 도입된 기능들을 안정화하고, 보고된 회귀 버그들을 수정하며, 새로운 EKF를 다양한 엣지 케이스에 대해 강화하는 것이 될 것입니다.

장기적으로 ROS 2 및 기타 미들웨어와의 심층 통합은 PX4의 핵심 전략 방향으로 지속될 것입니다.10 이는 PX4를 더 넓은 로보틱스 생태계의 필수 불가결한 부분으로 만들 것입니다. 주간 개발자 회의는 이러한 미래 작업을 조율하는 핵심적인 장으로 계속 기능할 것입니다.26 v1.15는 비록 험난한 과정을 거쳤지만, PX4가 단순한 오토파일럿을 넘어 진정한 '전문가용 자율비행 플랫폼'으로 나아가는 데 있어 필수적인 성장통이었습니다.


1. PX4 Autopilot: Open Source Autopilot for Drones, accessed July 7, 2025, https://px4.io/
2. PX4 User Guide (v1.14) - PX4 docs - PX4 Autopilot, accessed July 7, 2025, https://docs.px4.io/v1.14/en/
3. PX4 Autopilot User Guide | PX4 Guide (v1.15), accessed July 7, 2025, https://docs.px4.io/v1.15/en/
4. PX4-Autopilot v1.15 Release Notes | PX4 Guide (main), accessed July 7, 2025, https://docs.px4.io/main/en/releases/1.15.html
5. PX4 v1.15.0 Stable Release - PX4 Autopilot, accessed July 7, 2025, https://discuss.px4.io/t/px4-v1-15-0-stable-release/40846
6. Blog - PX4 Autopilot, accessed July 7, 2025, https://px4.io/blog/
7. PX4 Autopilot Release v1.15: What You Need To Know, accessed July 7, 2025, https://px4.io/px4-autopilot-release-v1-15-what-you-need-to-know/
8. PX4-Autopilot v1.14 Release Notes | PX4 Guide (main), accessed July 7, 2025, https://docs.px4.io/main/en/releases/1.14.html
9. PX4 Autopilot Release v1.14: What You Need to Know, accessed July 7, 2025, https://px4.io/px4-autopilot-release-v1-14-what-you-need-to-know/
10. PX4 Development | PX4 Guide (main) - PX4 docs, accessed July 7, 2025, https://docs.px4.io/main/en/development/development.html
11. PX4 System Architecture | PX4 Guide (v1.15), accessed July 7, 2025, https://docs.px4.io/v1.15/en/concept/px4_systems_architecture.html
12. Offboard Mode Issues on PX4-Autopilot 1.15 - Flight Testing & Log ..., accessed July 7, 2025, https://discuss.px4.io/t/offboard-mode-issues-on-px4-autopilot-1-15/42323
13. Firmware v1.15 stable _ GPS not detected _ CUBE orange _ Herelink - PX4 Autopilot, accessed July 7, 2025, https://discuss.px4.io/t/firmware-v1-15-stable-gps-not-detected-cube-orange-herelink/40854
14. generalize serial port configuration / Issue #8302 / PX4/PX4-Autopilot - GitHub, accessed July 7, 2025, https://github.com/PX4/Firmware/issues/8302
15. PX4 v1.15 public changes - what needs docs? - PX4 Autopilot, accessed July 7, 2025, https://discuss.px4.io/t/px4-v1-15-public-changes-what-needs-docs/39850
16. [Bug] Fixed-wing UAV not detecting landing in PX4 1.15.0 when GPS fusion is disabled / Issue #23746 - GitHub, accessed July 7, 2025, https://github.com/PX4/PX4-Autopilot/issues/23746
17. Frequent local position loss despite good GPS - PX4 Discussion Forum, accessed July 7, 2025, https://discuss.px4.io/t/frequent-local-position-loss-despite-good-gps/45362
18. PX4 Autopilot User Guide | PX4 Guide (main), accessed July 7, 2025, https://docs.px4.io/main/en/
19. Releases | PX4 Guide (v1.15), accessed July 7, 2025, https://docs.px4.io/v1.15/en/releases/index.html
20. [Bug] GPS does not show up for PixRacer Pro in v1.15.0 / Issue #23714 / PX4/PX4-Autopilot, accessed July 7, 2025, https://github.com/PX4/PX4-Autopilot/issues/23714
21. Parachute issues after upgrading from px4 1.14 to 1.15 - MAVLink, accessed July 7, 2025, https://discuss.px4.io/t/parachute-issues-after-upgrading-from-px4-1-14-to-1-15/43856
22. [Bug] "Operation timeout, aborting transfer" alert on flight controller ..., accessed July 7, 2025, https://github.com/PX4/PX4-Autopilot/issues/23904
23. [Bug] It does not work properly in PX4 version 1.15 / Issue #24325 - GitHub, accessed July 7, 2025, https://github.com/PX4/PX4-Autopilot/issues/24325
24. PX4 Sync / Q&A: Jan 15, 2025 - PX4 Discussion Forum - PX4 Autopilot, accessed July 7, 2025, https://discuss.px4.io/t/px4-sync-q-a-jan-15-2025/43274
25. PX4 Roadmap 2019 - PX4 Autopilot - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 7, 2025, https://discuss.px4.io/t/px4-roadmap-2019/9347
26. PX4 Maintainers Call: February 06, 2024 - PX4 Discussion Forum - PX4 Autopilot, accessed July 7, 2025, https://discuss.px4.io/t/px4-maintainers-call-february-06-2024/36562
27. Latest Coordination Calls topics - Discussion Forum for PX4 ..., accessed July 7, 2025, https://discuss.px4.io/c/weekly-dev-call/14


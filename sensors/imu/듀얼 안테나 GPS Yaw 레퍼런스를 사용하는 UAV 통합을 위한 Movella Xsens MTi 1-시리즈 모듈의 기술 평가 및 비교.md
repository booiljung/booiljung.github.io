---
layout: page
title: 듀얼 안테나 GPS Yaw 레퍼런스를 사용하는 UAV 통합을 위한 Movella Xsens MTi 1-시리즈 모듈의 기술 평가 및 비교
permalink: /sensors/imu/듀얼 안테나 GPS Yaw 레퍼런스를 사용하는 UAV 통합을 위한 Movella Xsens MTi 1-시리즈 모듈의 기술 평가 및 비교
---



본 보고서는 실외 드론 애플리케이션을 위한 Movella Xsens MTi 1-시리즈 관성 센서 모듈(MTi-1, MTi-2, MTi-3)에 대한 심층적인 기술 검토를 제공합니다. 분석의 핵심 전제는 사용자의 시스템 아키텍처, 즉 드론의 Yaw(헤딩)를 듀얼 안테나 GPS 시스템을 통해 추정한다는 것입니다. 이 결정적인 제약 조건을 고려할 때, **Movella Xsens MTi-2 (Vertical Reference Unit)**가 이 애플리케이션에 가장 적합하고 비용 효율적인 모듈이라는 것이 명백한 결론입니다. 이 모듈은 중력 참조 롤(Roll) 및 피치(Pitch) 데이터를 온보드에서 처리하여 제공하며, "비참조 저드리프트(unreferenced, low drift)" Yaw 출력은 듀얼 안테나 GPS와 같은 외부 절대 Yaw 소스와의 고주파 융합에 이상적으로 설계되었습니다.


이 권장 사항은 아키텍처 시너지 원칙에 기반합니다. 사용자의 시스템은 이미 자기장에 영향을 받지 않는 정확하고 절대적인 헤딩 소스를 갖추고 있습니다. 따라서 관성 모듈의 역할은 이 절대 소스를 보완하는 것이지, 대체하거나 충돌하는 것이 아닙니다. MTi-2는 호스트 프로세서의 롤 및 피치 추정이라는 계산 집약적인 작업을 오프로드하여 통합을 단순화합니다. 동시에, MTi-3(AHRS)의 핵심 기능인 자력계 기반 헤딩은 사용자의 우수한 듀얼 안테나 GPS 시스템으로 인해 불필요하며, 드론의 전자기 간섭 환경에서는 오히려 잠재적인 오류 소스가 될 수 있습니다. MTi-2는 이러한 중복성과 잠재적 충돌을 피할 수 있습니다. 가장 저렴한 옵션인 MTi-1(IMU)은 상당한 소프트웨어 개발 및 실시간 처리 부담을 호스트 프로세서에 부과하며, MTi-2로의 소폭의 가격 인상은 이러한 부담을 효과적으로 해결해 줍니다.


본 보고서는 세 모듈 모두 동일한 핵심 관성 하드웨어 플랫폼을 공유하며, 오직 펌웨어 기반의 센서 융합 기능에서만 차이가 있음을 입증할 것입니다. ROS2 Humble에 대한 지원은 공식적으로, 그리고 견고하게 제공됩니다. 따라서 최종 결정은 어떤 모듈의 온보드 처리 기능이 지정된 외부 Yaw 레퍼런스를 충돌 없이 가장 잘 보완하는지에 달려 있습니다. MTi-2는 이 기준에서 명백한 승자입니다.


Movella Xsens MTi-1, MTi-2, MTi-3 모듈을 비교하기 전에, 이 세 제품이 모두 동일한 물리적, 전기적, 센서 하드웨어 기반 위에 구축되었다는 점을 이해하는 것이 중요합니다. 이러한 공통 플랫폼은 물리적 통합 관점에서 상호 교환성을 보장하며, 모듈 간의 진정한 차이점이 펌웨어 수준의 데이터 처리 능력에 있음을 명확히 합니다.


세 모듈은 모두 동일한 하드웨어 플랫폼을 기반으로 하므로 물리적 통합 관점에서 상호 교환이 가능합니다.

- **치수 및 폼 팩터:** 모든 모듈은 12.1 x 12.1 x 2.55 mm 크기의 초소형 표면 실장 장치(SMD)이며, JEDEC PLCC-28 호환 풋프린트를 가집니다.1 0.6 g에 불과한 가벼운 무게와 작은 크기는 크기, 무게, 전력(SWaP)이 제한적인 UAV 애플리케이션에 이상적입니다.4
- **전기적 사양:** 모든 모듈은 2.8V에서 3.6V 사이의 공급 전압으로 작동하며, 3V에서 100 mW 미만의 일반적인 전력 소비를 특징으로 합니다.1 이러한 낮은 전력 소모는 배터리로 구동되는 드론에 매우 유리합니다.
- **인터페이스:** UART, SPI, I²C를 포함한 다용도 통신 인터페이스 세트가 모든 모듈에서 제공되어 다양한 마이크로컨트롤러 및 비행 컴퓨터와의 유연한 통합 옵션을 제공합니다.1
- **환경 등급:** 모듈은 IP00 등급을 가지며 -40°C에서 85°C의 넓은 작동 온도 범위를 지원하여 다양한 실외 환경 조건에 적합합니다.1


기저 MEMS 센서의 성능은 시리즈 전체에 걸쳐 일관됩니다. 차이점은 이 센서들로부터 얻은 데이터를 처리하는 방식에 있습니다. 사용자는 MTi-1에서 MTi-3로 업그레이드할 때 더 나은 하드웨어를 구매하는 것이 아니라, 더 정교한 온보드 처리 능력을 구매하는 것입니다.

- **자이로스코프:**

  - 표준 전체 범위(Standard Full Range): 2000 deg/s

  - 인런 바이어스 안정성(In-run bias stability): 6 deg/h

  - 노이즈 밀도(Noise Density): $0.003 \text{ º/s/}\sqrt{\text{Hz}}$

    1

- **가속도계:**

  - 표준 전체 범위(Standard Full Range): 16 g

  - 인런 바이어스 안정성(In-run bias stability): 40 µg

  - 노이즈 밀도(Noise Density): $70 \text{ µg/}\sqrt{\text{Hz}}$

    1

- **자력계:** 세 모듈 모두 온보드 3D 자력계를 포함하고 있습니다.1 이 자력계의 활용 여부와 방식이 각 모듈의 펌웨어 기능을 차별화하는 핵심 요소입니다.


Movella는 전체 MTi 시리즈를 지원하는 포괄적인 무료 소프트웨어 스위트를 제공하여, 선택한 모듈에 관계없이 일관된 개발 경험을 보장합니다.

- **MT Manager:** 장치 구성, 실시간 데이터 시각화 및 기록을 위한 Windows/Linux용 GUI입니다.1 특히, 기록된 로그 파일을 다른 필터 설정으로 재처리하는 강력한 기능은 개발 및 튜닝 과정에서 매우 유용합니다.9
- **MT SDK:** C++, C#, Python, MATLAB 및 ROS를 위한 프로그래밍 예제와 라이브러리를 제공하여 맞춤형 애플리케이션 개발을 용이하게 합니다.1
- **Magnetic Field Mapper (MFM):** 드론 자체 구조물에서 발생하는 정적 자기 왜곡을 보상하기 위한 교정 도구입니다.5

아래 표는 MTi 1-시리즈 모듈들이 공유하는 공통적인 하드웨어 및 소프트웨어 사양을 요약한 것입니다. 이 기준 정보를 통해 각 모듈의 고유한 기능적 차이점에 더 집중할 수 있습니다.

**표 1: MTi 1-시리즈 공통 플랫폼 사양**

| 사양 항목                         | 값                                          | 출처 |
| --------------------------------- | ------------------------------------------- | ---- |
| **물리적 특성**                   |                                             |      |
| 폼 팩터                           | 12.1 x 12.1 x 2.55 mm SMD                   | 1    |
| 무게                              | 0.6 g                                       | 4    |
| **전기적 특성**                   |                                             |      |
| 입력 전압                         | 2.8 V ~ 3.6 V                               | 1    |
| 전력 소비 (Typ. @ 3V)             | < 100 mW                                    | 4    |
| **핵심 센서 성능**                |                                             |      |
| 자이로스코프 인런 바이어스 안정성 | 6 deg/h                                     | 2    |
| 자이로스코프 노이즈 밀도          | $0.003 \text{º/s/}\sqrt{\text{Hz}}$         | 2    |
| 가속도계 인런 바이어스 안정성     | 40 µg                                       | 2    |
| 가속도계 노이즈 밀도              | $70 \text{ µg/}\sqrt{\text{Hz}}$            | 2    |
| **인터페이스 및 환경**            |                                             |      |
| 통신 인터페이스                   | UART, SPI, I²C                              | 1    |
| 작동 온도                         | -40 °C ~ 85 °C                              | 1    |
| IP 등급                           | IP00                                        | 1    |
| **소프트웨어 지원**               |                                             |      |
| 공통 소프트웨어                   | MT Software Suite (MT Manager, MT SDK, MFM) | 1    |


MTi 1-시리즈의 공통 하드웨어 플랫폼을 이해했다면, 이제 각 모듈을 차별화하는 펌웨어 기반 기능에 대해 심층적으로 분석할 차례입니다. 이 기능적 차이는 호스트 프로세서와의 처리 부담 분담, 통합의 복잡성, 그리고 최종적으로 사용자의 특정 아키텍처에 대한 적합성을 결정합니다.


- **기능:** MTi-1은 순수한 IMU입니다. 이 모듈은 보정되고 시간 동기화된 3D 각속도, 3D 선형 가속도, 3D 자기장 데이터 스트림을 제공합니다.1 모듈 자체적으로는 자세 솔루션을 계산하기 위한 어떠한 센서 융합도 수행하지 않습니다. 출력은 순수한 센서 측정값입니다.
- **통합에 대한 시사점:** 이 모듈을 사용하면, 호스트 프로세서(예: 드론의 비행 컴퓨터)가 이 원시 데이터를 안정적인 자세 추정치로 융합하기 위한 상태 추정 필터(예: 확장 칼만 필터 - EKF)를 전적으로 구현해야 합니다. 이는 상당한 처리 능력과 센서 융합 알고리즘 개발 및 튜닝에 대한 깊은 전문 지식을 요구합니다.13
- **사용 사례 분석:** 이 모듈은 이미 고도로 맞춤화되었거나 독점적인 센서 융합 파이프라인을 보유한 전문가 사용자 또는 팀에 가장 적합합니다. 최대의 유연성을 제공하지만, 동시에 호스트 시스템에 가장 큰 통합 노력과 계산 오버헤드를 요구합니다.


- **기능:** MTi-2는 VRU입니다. 이 모듈은 Xsens의 AttitudeEngine™이라는 온보드 센서 융합 알고리즘을 내장하고 있으며, 가속도계 데이터를 사용하여 중력 방향을 참조합니다. 이를 통해 드리프트가 없는 고정밀 롤 및 피치 추정치(0.5 deg RMS)를 계산하고 출력할 수 있습니다.2 결정적으로, 이 모듈은 "비참조, 저드리프트(unreferenced, low drift)"로 명시된 Yaw 각도도 출력합니다.2
- **통합에 대한 시사점:** MTi-2는 안정적인 롤과 피치를 계산하는 복잡한 작업을 호스트 프로세서로부터 오프로드합니다. 여기서 "비참조" Yaw는 자력이나 진북을 참조하지 않고, 단순히 자이로스코프의 회전율을 바이어스 보정하여 적분한 값입니다. "저드리프트" 특성은 고품질 자이로스코프(6 deg/h 바이어스 안정성)와 정교한 온보드 바이어스 추정 알고리즘의 직접적인 결과입니다.2
- **사용 사례 분석:** 이 기능은 사용자의 시스템에 대한 이상적인 아키텍처적 조화를 이룹니다. 드론의 메인 EKF는 MTi-2로부터 고품질의 롤과 피치를 직접 수신할 수 있습니다. MTi-2의 "비참조" Yaw는 듀얼 안테나 GPS로부터의 저주파(일반적으로 1-10 Hz) 절대 헤딩 업데이트 사이의 부드러운 제어를 위해 필요한 고주파(최대 1 kHz) 상대 헤딩 정보를 제공합니다. 이러한 구조는 현대적인 센서 융합 시스템에서 매우 효율적이고 강력한 접근 방식입니다.


- **기능:** MTi-3는 완전한 AHRS입니다. 이 모듈은 MTi-2의 기능에 더해, 온보드 자력계를 절대 참조(나침반처럼)로 사용하여 북쪽을 기준으로 하는 Yaw/헤딩 추정치(일반 정확도 2 deg RMS)를 계산합니다.3 즉, 모듈에서 직접 완전한 3D 자세 솔루션(롤, 피치, 헤딩)을 제공합니다.
- **통합에 대한 시사점:** 사용자의 아키텍처에서는 두 개의 독립적인 절대 헤딩 소스, 즉 MTi-3의 자기 헤딩과 GPS의 기하학적 헤딩이 존재하게 됩니다. 이 두 소스는 드론의 자기 교란과 측정 원리의 본질적인 차이로 인해 필연적으로 충돌하게 됩니다. 호스트의 융합 필터는 한 소스를 다른 소스보다 신뢰하도록 구성해야 하며, 이는 사실상 사용자가 MTi-3의 주요 기능을 비활성화하거나 무시하기 위해 비용을 지불하는 것과 같습니다.
- **사용 사례 분석:** AHRS는 더 나은 절대 헤딩 소스가 없는 애플리케이션에 가치가 있습니다. 그러나 UAV에서는 강력한 모터, 고전류 전력 분배 보드 및 기타 전자 장치가 심각하고 동적으로 변화하는 자기장을 생성하여, 신뢰할 수 있는 자력계 기반 헤딩을 극도로 어렵게 만듭니다.16 사용자가 듀얼 안테나 GPS를 선택한 것은 바로 이 문제에 대한 직접적이고 견고한 해결책이며, 이는 MTi-3의 AHRS 기능을 중복되고 신뢰성이 떨어지는 것으로 만듭니다.


각 모듈의 기능적 차이를 이해한 후, 이제 사용자의 특정 요구 사항인 '듀얼 안테나 GPS Yaw를 사용하는 드론'이라는 맥락에서 이들을 직접 비교 분석해야 합니다. 이 섹션에서는 아키텍처 적합성, 가격 대비 가치, 그리고 ROS2 Humble과의 통합 경로를 중심으로 각 모듈을 평가합니다.


ROS 환경에서 가장 견고한 접근 방식은 `robot_localization` 패키지를 사용하는 것입니다.18 이 패키지는 EKF 또는 UKF를 사용하여 임의의 수의 센서 입력을 융합하도록 설계되었습니다.

- **MTi-1 통합:** 사용자는 `/imu/data` 토픽에서 나오는 각속도와 선형 가속도를 융합하도록 `robot_localization`을 구성해야 합니다. 또한 듀얼 안테나 GPS Yaw를 별도의 메시지(예: 다른 `sensor_msgs/Imu` 또는 `geometry_msgs/PoseWithCovarianceStamped`)로 입력하고 필터가 이를 융합하도록 설정해야 합니다. 이 과정은 필터의 프로세스 노이즈 및 측정 공분산 행렬에 대한 신중한 튜닝을 요구하며, 상당한 전문 지식이 필요합니다.14
- **MTi-2 통합:** 이것이 가장 간단하고 효율적인 경로입니다. 사용자는 MTi-2의 `/imu/data` 토픽에서 나오는 절대 롤과 피치, 그리고 각속도를 융합하도록 `robot_localization`을 구성합니다. 이때 MTi-2에서 나오는 Yaw 측정값은 필터 설정에서 *비활성화*합니다. 듀얼 안테나 GPS의 절대 Yaw가 필터가 융합하는 *유일한* 절대 Yaw 소스가 됩니다. 이 방식은 필터가 두 개의 경쟁적인 Yaw 추정치를 조율할 필요가 없으므로 튜닝을 크게 단순화합니다.
- **MTi-3 통합:** 이는 도전적인 과제를 제시합니다. `/imu/data` 토픽에는 북쪽 기준 Yaw가 포함됩니다. 사용자는 MTi-3의 Yaw를 무시하고 GPS Yaw를 사용하거나(이 경우 MTi-3는 비싼 MTi-2가 됨), 두 가지를 모두 융합하려고 시도해야 합니다. 후자는 필터의 불안정성과 진동을 유발할 수 있어 강력히 권장되지 않습니다.20 고-EMI 환경에서 듀얼 안테나 GPS 헤딩의 우월성을 고려할 때, MTi-3의 자기 헤딩을 무시하는 것이 유일한 논리적 선택입니다.

아래는 MTi-2와 듀얼 GPS Yaw를 `robot_localization` 패키지와 함께 사용하는 개념적인 설정 예시입니다.

```YAML
# MTi-2 + 듀얼 GPS Yaw를 위한 예시
ekf_global_node:
  ros__parameters:
    #... 기타 파라미터
    imu0: /imu/data  # MTi-2로부터
    imu0_config: # X 가속 바이어스, Y 가속 바이어스, Z 가속 바이어스
    imu0_differential: false
    imu0_relative: false

    imu1: /gps/yaw_imu # 듀얼 안테나 GPS로부터
    imu1_config:
    imu1_differential: false
    imu1_relative: false
```


엔지니어링 부품의 비용은 단순히 구매 가격이 아니라 총 통합 비용으로 평가해야 합니다.

- **가격 데이터:**
  - MTi-1 (MTI-1-5A-T): 약 €145 / $145 21
  - MTi-2 (MTI-2-5A-T): 약 €222 22
  - MTi-3 (MTI-3-5A-T): 약 $349 / €300-€340 23
- **가치 분석:**
  - MTi-1에서 MTi-2로의 약 €77 가격 인상은 전문적으로 개발되고 공장에서 튜닝된 롤 및 피치용 자세 필터를 구매하는 것과 같습니다. 이는 상당한 엔지니어링 시간과 호스트의 계산 부하를 절약해 주므로 높은 투자 수익을 나타냅니다.
  - MTi-3에 대한 추가적인 약 €80-€120의 비용은 사용자의 기존 GPS 솔루션보다 성능이 떨어지고 드론의 고-EMI 환경에 취약한 자기 헤딩 기능을 제공합니다. 이는 이 특정 애플리케이션에 대해 낮은 가치 제안을 의미합니다.
  - 결론적으로, MTi-2는 이 애플리케이션에서 가장 낮은 총 소유 비용(구매 가격 + 엔지니어링 노력)을 제공합니다.


- **공식 드라이버 지원:** Movella는 GitHub에서 공식 ROS 드라이버(`xsenssupport/Xsens_MTi_ROS_Driver_and_Ntrip_Client`)를 유지 관리하며, 전용 `ros2` 브랜치를 제공합니다.27
- **Humble 호환성:** 문서 및 설치 지침은 `ros-humble-nmea-msgs`와 같은 종속성을 요구하며 ROS2 Humble에 대한 지원을 명시적으로 언급합니다.27 이는 직접적이고 지원되는 통합 경로를 확인시켜 줍니다.
- **드라이버 기능:** 이 드라이버는 YAML 파일을 통해 광범위한 데이터 출력 및 구성을 지원하며, Xsens/Movella 지원 엔지니어에 의해 적극적으로 유지 관리됩니다.29 이는 고품질의 신뢰할 수 있는 인터페이스를 보장합니다.

아래 표는 각 모듈의 기능, 성능, 가격 및 특정 애플리케이션에 대한 적합성을 한눈에 비교할 수 있도록 요약한 것입니다.

**표 2: 모듈 기능 및 성능 비교 매트릭스**

| 항목                                              | MTi-1 (IMU)       | MTi-2 (VRU)          | MTi-3 (AHRS)                  |
| ------------------------------------------------- | ----------------- | -------------------- | ----------------------------- |
| **기능**                                          | 원시 관성 데이터  | 롤/피치 + 비참조 Yaw | 완전한 3D 자세 (롤/피치/헤딩) |
| **롤/피치 정확도 (RMS)**                          | N/A (호스트 처리) | 0.5 deg              | 0.5 deg                       |
| **Yaw 출력 유형**                                 | N/A (호스트 처리) | 비참조, 저드리프트   | 북쪽 참조                     |
| **Yaw 정확도 (RMS)**                              | N/A (호스트 처리) | N/A                  | 2 deg                         |
| **예상 단가**                                     | ~€145 / $145      | ~€222                | ~€349 / $349                  |
| **통합 노력**                                     | 높음              | 낮음                 | 중간 (Yaw 충돌 해결 필요)     |
| **듀얼 GPS Yaw 아키텍처 적합성 점수 (10점 만점)** | 6/10              | **10/10**            | 3/10                          |


최적의 모듈을 선택한 후에는 드론이라는 특수한 환경에 맞게 올바르게 통합하는 것이 중요합니다. 이 섹션에서는 기계적 장착, 진동 완화, 전자기 간섭 및 필터 설정과 같은 실제적인 고려 사항을 다룹니다.


- **장착 방향:** 모듈은 장착 방향에 제한이 없습니다.1 그러나 소프트웨어에서 좌표 변환을 단순화하기 위해 센서의 축을 드론의 전방-우측-하향 또는 전방-좌측-상향 프레임과 정렬하는 것이 가장 좋습니다. 기계적 제약으로 인해 비표준 장착이 필요한 경우, `RotSensor` 및 `RotLocal` 정렬 행렬을 사용하여 사용자 정의 방향을 정의할 수 있습니다.32
- **진동 감쇠:** UAV는 모터와 프로펠러로 인해 진동이 심한 환경입니다.34 고주파 진동은 가속도계를 포화시키고 앨리어싱을 유발하여 잘못된 자세 추정으로 이어질 수 있습니다.7 따라서 특수 진동 감쇠 마운트를 사용하여 MTi 모듈을 드론의 주 프레임에서 기계적으로 분리하는 것이 매우 중요합니다. Xsens 사용자 매뉴얼은 효과가 입증된 특정 댐퍼(예: Norelem P/N 26102-00800855)를 권장합니다.7


- **EMI 차폐:** 사용자가 헤딩을 위해 자력계에 의존하지 않더라도, 모터 컨트롤러 및 무선 송신기에서 발생하는 일반적인 EMI는 여전히 민감한 전자 장치에 영향을 줄 수 있습니다. 고전류 전선과의 물리적 분리 및 노이즈가 심한 부품 근처에 모듈을 배치할 경우 보드 레벨 차폐를 고려하는 것이 모범 사례입니다.36
- **자기장 매퍼(MFM) 유틸리티:** MFM 도구는 *정적*인 하드 아이언 및 소프트 아이언 왜곡을 보정하도록 설계되었습니다.10 드론에서는 모터 스로틀에 따라 자기장이 매우 *동적*으로 변합니다. 따라서 완전히 조립되고 전원이 꺼진 드론에서 프레임 및 정적 부품을 고려하여 MFM 보정을 수행해야 하지만, 비행 중 동적 필드를 보상할 수는 없습니다.17 사용자가 Yaw를 위해 듀얼 안테나 GPS를 사용하기로 한 결정은 다중 로터 UAV에서 동적 자기 간섭이라는 다루기 힘든 문제를 완전히 우회하는 가장 중요한 설계 선택이며, 이는 매우 현명한 아키텍처적 결정입니다.


- **사용 가능한 프로파일:** MTi 1-시리즈는 다양한 동적 조건에 맞게 센서 융합 알고리즘의 동작을 조정하는 여러 온보드 필터 프로파일을 제공합니다.29 프로파일에는 "General", "High_mag_dep", "Dynamic", "VRU_general"(MTi-2용) 등이 포함됩니다.
- **드론 권장 사항:** 동적인 항공기의 경우, 빠른 변화와 급격한 움직임을 가정하는 "Dynamic" 프로파일이 강력한 후보입니다. 권장되는 MTi-2의 경우, "VRU_general" 프로파일은 자기장을 신뢰할 수 없는 환경을 위해 특별히 설계되었으며, 깨끗하고 드리프트가 적은 비참조 Yaw를 제공하기 위해 AHS(Active Heading Stabilization) 기능과 효과적으로 작동합니다.40 특정 드론의 비행 특성에 가장 적합한 프로파일을 찾기 위해서는 MT Manager의 재처리 기능을 사용한 실험이 핵심이 될 것입니다.9


- **중요성:** 모듈은 공장에서 보정되지만, 자이로스코프 바이어스는 온도와 시간에 따라 약간씩 변할 수 있습니다.41 안정적인 자이로 바이어스는 MTi-2의 "저드리프트" Yaw 성능을 달성하는 데 매우 중요합니다.
- **절차:** 센서를 5-10분 동안 예열하는 것이 가장 좋습니다. 비행 전 드론이 지상에서 정지해 있는 동안 `수동 자이로 바이어스 추정(Manual Gyro Bias Estimation)` 절차를 수행해야 합니다. 이는 MT Manager를 통해 트리거하거나, 드론 소프트웨어의 비행 전 점검 목록에 통합된 저수준 명령(`SetNoRotation`)을 통해 수행할 수 있습니다.7 공식 ROS 드라이버는 이 절차를 주기적으로 트리거하는 것을 지원합니다.43


본 보고서의 모든 분석을 종합하여, 사용자의 특정 요구사항에 맞는 최종적이고 명확한 권장 사항을 제시합니다.


**MTi-2는 외부 헤딩 소스를 사용하는 아키텍처를 위해 온보드 처리와 원시 데이터 출력의 최적 균형을 제공합니다.** 이 모듈은 깨끗한 롤/피치 솔루션을 제공하여 소프트웨어 작업을 단순화하는 동시에, 듀얼 안테나 GPS와의 견고한 융합에 필요한 이상적인 고주파 상대 Yaw 데이터를 제공합니다. 이는 최고의 가치와 가장 낮은 총 통합 비용을 제공하므로, 이 애플리케이션에 대한 명확한 엔지니어링 선택입니다.


**광범위한 사내 센서 융합 전문 지식을 보유하고 있으며 하드웨어 비용 최소화를 최우선으로 하는 팀에게는 MTi-1이 실행 가능한 대안입니다.** 이 모듈은 동일한 고품질 핵심 센서 데이터를 제공하지만, 상태 추정 필터의 개발, 구현 및 실시간 실행에 대한 모든 부담을 사용자가 져야 합니다. 이 경로는 엔지니어링 리소스가 충분하고 비용 절감이 중요한 경우에만 선택해야 합니다.


**MTi-3는 이 애플리케이션에 명시적으로 권장되지 않습니다.** 주요 기능인 자력계 기반 헤딩 솔루션은 우수한 듀얼 안테나 GPS 시스템으로 인해 중복되며, 일반적인 UAV의 전자기 환경에서는 본질적으로 신뢰할 수 없습니다. 추가 비용은 실질적인 이점을 제공하지 않으며, 소프트웨어에서 필터링해야 하는 잠재적인 데이터 충돌 소스를 도입할 뿐입니다.


관성 모듈의 역할을 저주파 절대 레퍼런스를 보완하는 고주파 보완재로 올바르게 식별함으로써, 사용자는 단일 센서 유형에 의존하는 시스템보다 더 견고하고 정확하며 신뢰할 수 있는 내비게이션 시스템을 구축할 수 있습니다. MTi-2는 이러한 현대적인 하이브리드 상태 추정 접근 방식을 위해 특별히 제작된 구성 요소이며, 이는 이 프로젝트의 명백한 기술적 선택이 됩니다.


1. MTi-1 - Movella.com, accessed July 30, 2025, https://www.xsens.com/hubfs/Downloads/Leaflets/MTi-1.pdf
2. MTi-2 - Movella.com, accessed July 30, 2025, https://www.xsens.com/hubfs/Downloads/Leaflets/MTi-2.pdf
3. MTi-3 - Movella.com, accessed July 30, 2025, https://www.xsens.com/hubfs/Downloads/Leaflets/MTi-3.pdf
4. Xsens MTi-1 IMU - Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules/xsens-mti-1-imu
5. Xsens MTi-2 VRU - Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules/xsens-mti-2-vru
6. Xsens MTi-3 AHRS - Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules/xsens-mti-3-ahrs
7. MTi User Manual, accessed July 30, 2025, https://www.xsens.com/hubfs/Downloads/usermanual/MTi_usermanual.pdf
8. MT Manager User Manual, accessed July 30, 2025, https://www.xsens.com/hubfs/Downloads/usermanual/MT_Manager_user_manual.pdf
9. Xsens MTi for Mobile Warehouse Robotics - Movella.com, accessed July 30, 2025, [https://www.movella.com/hubfs/Automation%20and%20Mobility%20-%20Application%20Notes/Indoor%20Mobile%20Robots/Xsens%20MTi%20Series%20for%20AGV-AMR%20Applications.pdf](https://www.movella.com/hubfs/Automation and Mobility - Application Notes/Indoor Mobile Robots/Xsens MTi Series for AGV-AMR Applications.pdf)
10. Xsens MTi-670 GNSS/INS - Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules/xsens-mti-670-gnss-ins
11. Xsens MTi-630 AHRS - Movella.com, accessed July 30, 2025, https://www.movella.com/products/sensor-modules/xsens-mti-630-ahrs
12. MTi 1-series User Manual - Farnell, accessed July 30, 2025, https://www.farnell.com/datasheets/4385415.pdf
13. Roadtest Review: Xsens MTi-680-DK - element14 Community, accessed July 30, 2025, https://community.element14.com/products/roadtest/rv/roadtest_reviews/1624/roadtest_review_xsen
14. AI-Driven Dynamic Covariance for ROS 2 Mobile Robot Localization - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12114818/
15. MTi-2 VRU - Applied Measurement, accessed July 30, 2025, https://appliedmeasurement.com.au/product/mti-2-vru/
16. Xsens Sensor Modules - Movella.com, accessed July 30, 2025, https://www.movella.com/sensor-modules
17. Yaw drifts regardless of the filter settings - Movella.com, accessed July 30, 2025, https://base.movella.com/s/question/0D50900002tggCKCAY/yaw-drifts-regardless-of-the-filter-settings?language=en_US
18. weihsinc/robot_localization: A ROS package for real-time nonlinear state estimation for robots moving in 3D space. It contains two state estimation nodes which use Kalman filters (EKF/UKF) for real-time sensor fusion. - GitHub, accessed July 30, 2025, https://github.com/weihsinc/robot_localization
19. Setting covarince matrix for Robot Localization - ROS Answers archive, accessed July 30, 2025, https://answers.ros.org/question/351488/
20. Fusing Wheel Odometry, IMU Data, and GPS Data Using robot_localization in ROS, accessed July 30, 2025, https://blog.abdurrosyid.com/2021/07/22/fusing-wheel-odometry-imu-data-and-gps-data-using-robot_localization-in-ros/
21. XSENS MTI-1-5A-T - Spezial Electronic, accessed July 30, 2025, https://www.spezial.com/en/mti-1-5a-t-p10208/
22. MTI-1-5A-T Xsens a Movella brand | Sensors, Transducers | DigiKey, accessed July 30, 2025, https://www.digikey.at/en/products/detail/xsens-a-movella-brand/MTI-1-5A-T/18718044
23. MTI-3-0I Xsens a Movella brand | Sensors, Transducers | DigiKey, accessed July 30, 2025, https://www.digikey.com/en/products/detail/xsens-a-movella-brand/MTI-3-0I/14834329
24. Mti-3 Ahrs Module V3 - Electromaker.io, accessed July 30, 2025, https://www.electromaker.io/shop/product/mti-3-ahrs-module-v3
25. MTi-3-5A-T Movella / Xsens - Mouser Electronics, accessed July 30, 2025, [https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-5A-T?qs=3Rah4i%252BhyCEVHzvLS1mcLw%3D%3D](https://www.mouser.com/ProductDetail/Movella-Xsens/MTi-3-5A-T?qs=3Rah4i%2BhyCEVHzvLS1mcLw%3D%3D)
26. MTi-3-0I-T Movella / Xsens | Mouser Finland, accessed July 30, 2025, [https://www.mouser.fi/ProductDetail/Movella-Xsens/MTi-3-0I-T?qs=pBJMDPsKWf0vHDkgc01%252BrQ%3D%3D](https://www.mouser.fi/ProductDetail/Movella-Xsens/MTi-3-0I-T?qs=pBJMDPsKWf0vHDkgc01%2BrQ%3D%3D)
27. UTFR/xsens_ros: Xsens MTi ROS Driver and Ntrip Client, modified based on the 2023 official ROS Driver and added Ntrip Client - GitHub, accessed July 30, 2025, https://github.com/UTFR/xsens_ros
28. How to Run two MTi sensors with ROS 2 Driver? - Movella.com, accessed July 30, 2025, https://base.movella.com/s/question/0D5Tc00000AOcBWKA1/how-to-run-two-mti-sensors-with-ros-2-driver?language=en_US
29. All MTi Related Documentation Links - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/All-MTi-Related-Documentation-Links
30. Interfacing MTi devices with the Raspberry Pi - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Interfacing-MTi-devices-with-the-Raspberry-Pi
31. Xsense MTI -3 NOT connecting with Jetpack 6 - NVIDIA Developer Forums, accessed July 30, 2025, https://forums.developer.nvidia.com/t/xsense-mti-3-not-connecting-with-jetpack-6/328236
32. Changing or Resetting the MTi reference co-ordinate systems - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Changing-or-Resetting-the-MTi-reference-co-ordinate-systems-1605869706643?language=en_US
33. Example: How to determine the correct alignment matrices (RotSensor/RotLocal) for your application - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Example-How-to-determine-the-correct-alignment-matrices-RotSensor-RotLocal-for-your-application
34. Reducing Vibration in UAVs and Thrust Stands - Tyto Robotics, accessed July 30, 2025, https://www.tytorobotics.com/blogs/articles/reducing-vibration-in-drones-and-test-stands
35. Best Practices for Automotive Applications - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Best-Practices-for-Automotive-Applications?language=en_US
36. 2017 EMI SHIELDING GUIDE | Interference Technology, accessed July 30, 2025, https://interferencetechnology.com/wp-content/uploads/2017/06/2017_IT_EMI-Shielding-Guide_Low-Res.pdf
37. How to perform a Magnetic Field Mapping on Movella DOT, accessed July 30, 2025, https://base.movella.com/s/article/How-to-perform-a-Magnetic-Field-Mapping-on-Movella-DOT
38. Magnetic immunity of the MTi AHRS - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Magnetic-immunity-of-the-MTi-AHRS-1605869708053
39. MTi Filter Profiles - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/MTi-Filter-Profiles-1605869708823
40. Filter profiles for MTi 1-series - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Filter-profiles-for-MTi-1-series-1605870241087
41. Recalibrations - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Recalibrations
42. Understanding Sensor Bias (offset) - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/article/Understanding-Sensor-Bias-offset?language=zh_TW
43. Manual Gyro Bias Estimation (MGBE) - Movella.com, accessed July 30, 2025, https://base.movella.com/s/article/Manual-Gyro-Bias-Estimation


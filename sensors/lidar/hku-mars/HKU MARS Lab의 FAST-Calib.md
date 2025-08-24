# HKU MARS Lab의 FAST-Calib에 대한 비판적 분석
[HKU-MaRS](./index.md)


자율 주행 자동차, 로봇 공학, 3차원 장면 재구성 등 다양한 첨단 기술 분야에서 주변 환경을 정확하고 강건하게 인식하는 능력은 시스템의 성능과 안전성을 결정하는 핵심 요소입니다. 이를 위해 단일 센서의 한계를 극복하고 상호 보완적인 정보를 얻기 위해 여러 종류의 센서를 융합하는 다중 모드(multi-modal) 접근 방식이 표준으로 자리 잡았습니다. 특히, LiDAR(Light Detection and Ranging)와 카메라는 자율 시스템에서 가장 보편적으로 사용되는 센서 조합으로, 이 두 센서의 데이터를 효과적으로 융합하기 위한 전제 조건이 바로 '외재적 캘리브레이션(Extrinsic Calibration)'입니다.


LiDAR와 카메라는 각각 고유한 장점을 가지며, 서로의 단점을 보완하는 이상적인 관계를 형성합니다. LiDAR는 레이저 펄스를 사용하여 주변 환경까지의 거리를 매우 정밀하게 측정함으로써, 조명 조건에 관계없이 정확한 3차원 기하학적 정보를 담은 포인트 클라우드(point cloud)를 생성합니다.1 이는 장애물 감지, 지도 작성, 측위 등에서 핵심적인 역할을 합니다. 하지만 LiDAR 데이터는 본질적으로 희소(sparse)하고, 색상이나 질감과 같은 시각적 정보를 포함하지 않는다는 한계가 있습니다.3

반면, 카메라는 저렴한 비용으로 고해상도의 2차원 이미지를 제공하며, 여기에는 풍부한 색상, 질감, 그리고 딥러닝 모델을 통해 추출할 수 있는 의미론적(semantic) 정보가 담겨 있습니다.6 이는 객체 인식, 분류, 장면 이해에 필수적입니다. 그러나 카메라는 조명 변화에 민감하고, 이미지로부터 직접 3차원 깊이 정보를 얻는 것이 어렵다는 본질적인 제약을 가집니다.3

이 두 센서의 데이터를 융합하면, LiDAR의 정밀한 3차원 구조에 카메라의 풍부한 시각적 정보를 결합하여, 각 센서가 단독으로 제공할 수 있는 것보다 훨씬 더 완전하고 강건한 환경 인식이 가능해집니다. 예를 들어, LiDAR 포인트 클라우드에 카메라 이미지의 색상을 입혀 '컬러 포인트 클라우드(colored point cloud)'를 생성하면, 3차원 공간 내 객체의 형태와 시각적 특징을 동시에 파악할 수 있어 SLAM(Simultaneous Localization and Mapping)이나 3D 객체 감지 알고리즘의 성능을 크게 향상시킬 수 있습니다.2


LiDAR-카메라 데이터 융합의 핵심은 두 센서가 서로 다른 좌표계에서 데이터를 수집한다는 사실을 극복하는 것입니다. 외재적 캘리브레이션은 바로 이 문제를 해결하는 과정으로, 한 센서의 좌표계를 다른 센서의 좌표계로 변환하는 6자유도(6-DoF) 강체 변환(rigid body transformation) 행렬을 찾는 것을 목표로 합니다.8 이 변환 행렬은 3x3 회전 행렬(R)과 3x1 평행 이동 벡터(t)로 구성되며, LiDAR 좌표계의 한 점 PL을 카메라 좌표계의 점 PC로 변환하는 관계를 PC=R⋅PL+t 와 같이 수학적으로 정의합니다.2

이 외재적 파라미터(R,t)가 정확하게 추정되어야만, LiDAR 포인트 클라우드의 각 점을 카메라 이미지 평면에 정확하게 투영할 수 있으며, 이를 통해 두 데이터 스트림 간의 의미 있는 연관성을 찾고 융합할 수 있습니다. 만약 이 캘리브레이션에 작은 오차라도 존재한다면, 데이터 융합 과정에서 불일치가 발생하여 포인트 클라우드 색상화가 어긋나거나, SLAM 시스템의 데이터 연관(data association) 실패, 3D 객체 감지 성능 저하 등 심각한 문제를 야기할 수 있습니다. 따라서, 외재적 캘리브레이션의 정확도는 모든 후속 융합 기반 인식 알고리즘의 성능 상한선을 결정하는 매우 중요한 선행 단계라 할 수 있습니다.


LiDAR-카메라 외재적 캘리브레이션이 어려운 근본적인 이유는 두 가지 매우 이질적인 데이터 양식(modality) 사이에서 신뢰할 수 있는 대응 관계(correspondence)를 찾는 것이 어렵기 때문입니다.3 카메라는 조밀한 픽셀 그리드로 구성된 2차원 이미지를 생성하는 반면, LiDAR는 비정형적이고 희소한 3차원 포인트 클라우드를 생성합니다. 이 두 데이터 간의 해상도, 시야각(Field of View, FoV), 데이터 구조의 근본적인 차이는 대응점 탐색을 매우 도전적인 과제로 만듭니다.2 예를 들어, 이미지에서 쉽게 검출되는 체커보드의 모서리(corner)는 희소한 LiDAR 포인트 클라우드에서는 정확하게 특정하기 매우 어렵습니다. 이러한 본질적인 난제를 해결하기 위해 다양한 캘리브레이션 방법론이 제안되어 왔으며, FAST-Calib 역시 이러한 맥락에서 개발된 도구 중 하나입니다.


HKU MARS Lab의 FAST-Calib은 단순히 독립적인 유틸리티 소프트웨어로 평가될 수 없습니다. 이 도구의 진정한 가치와 전략적 중요성을 이해하기 위해서는, 홍콩대학교(HKU) 기계공학과 소속의 메카트로닉스 및 로봇 시스템 연구실(Mechatronics and Robotic Systems Laboratory, MARS Lab)이 구축해 온 "FAST"라는 이름의 고성능 SLAM 알고리즘 생태계 안에서 그 위치를 파악해야 합니다.11 FAST-Calib은 이 생태계의 성능을 극대화하고 접근성을 높이기 위해 전략적으로 개발된 필수 구성 요소입니다.


MARS Lab은 지난 몇 년간 로봇 공학, 특히 LiDAR 기반 SLAM 분야에서 세계적인 명성을 쌓아왔습니다. 이들의 연구는 'FAST'라는 브랜딩으로 대표되는 일련의 빠르고(Fast) 강건한(Robust) 알고리즘들을 통해 구체화되었습니다.

- **FAST-LIO 및 FAST-LIO2:** 이들은 고효율의 강결합(tightly-coupled) 반복 칼만 필터(iterated Kalman filter)를 기반으로 하는 LiDAR-관성측정장치(IMU) 주행 거리 측정(odometry) 프레임워크입니다.12 특히 FAST-LIO2는 특징점 추출 과정 없이 원시 포인트(raw points)를 직접 지도에 등록하는 방식을 채택하여 환경의 미묘한 특징까지 활용함으로써 정확도를 높였습니다.12 또한, 증분 k-d 트리(incremental k-d tree, ikd-Tree)라는 새로운 데이터 구조를 도입하여 대규모 지도를 효율적으로 유지하고 업데이트할 수 있게 함으로써, 열악한 환경에서도 높은 계산 효율성과 강건성을 달성했습니다.12
- **FAST-LIVO2:** 이는 FAST-LIO2를 한 단계 더 발전시킨 LiDAR-관성-시각 주행 거리 측정(LiDAR-Inertial-Visual Odometry) 시스템입니다.15 LiDAR, IMU, 카메라 세 가지 센서의 측정값을 효율적인 오류 상태 반복 칼만 필터(Error-State Iterated Kalman Filter, ESIKF) 내에서 순차적으로 융합합니다.17 이 다중 모드 융합을 통해 한 센서의 성능이 저하되는 환경(예: LiDAR의 특징이 부족한 복도, 카메라의 조명이 어두운 곳)에서도 다른 센서의 정보로 보완하여 전체 시스템의 정확성과 강건성을 극대화합니다.16


FAST-Calib과 "FAST" 생태계 간의 긴밀한 관계는 FAST-LIVO2의 공식 GitHub 저장소에서 명확하게 드러납니다. FAST-LIVO2 문서는 사용자들에게 **FAST-Calib 툴킷 사용을 권장**하며, "FAST-Calib의 출력 외재적 파라미터를 YAML 설정 파일에 직접 채워 넣을 수 있다"고 명시하고 있습니다.16 이는 두 프로젝트가 단지 같은 연구실에서 개발되었다는 사실을 넘어, 기능적으로 완벽하게 연동되도록 설계되었음을 보여주는 강력한 증거입니다. FAST-Calib은 FAST-LIVO2 시스템을 구동하기 위한 '공식' 사전 준비 도구로서의 역할을 수행합니다.


세계 최고 수준의 SLAM 알고리즘을 개발하는 연구실이 왜 상대적으로 '기초적인' 캘리브레이션 도구를 직접 개발하고 배포하는지에 대한 질문은 이 생태계의 전략을 이해하는 데 핵심적입니다. 그 이유는 SLAM 시스템의 성능이 전적으로 정확한 외재적 파라미터에 의존하기 때문입니다. 사용자들이 복잡하고 오류가 발생하기 쉬운 캘리브레이션 과정에서 어려움을 겪는다면, 그들은 FAST-LIVO2와 같은 우수한 SLAM 시스템의 진정한 성능을 경험하거나 제대로 평가할 수 없습니다. 이는 곧 알고리즘의 채택과 확산에 심각한 병목 현상을 초래합니다.

MARS Lab은 이러한 문제를 해결하기 위해 직접 나섰습니다. "단 2초 만에" 19 고정밀 캘리브레이션을 수행할 수 있는 자동화된 도구인 FAST-Calib을 제공함으로써, 사용자들이 겪는 가장 큰 진입 장벽 중 하나를 제거한 것입니다. "FAST"라는 브랜딩을 공유하는 것은 이 도구가 연구실의 다른 고성능 알고리즘들과 동일한 철학(속도와 효율성)을 공유하며, 원활하게 작동하도록 설계되었음을 사용자에게 알리는 의도적인 신호입니다. 결과적으로, FAST-Calib은 단순한 유틸리티를 넘어, MARS Lab의 핵심 연구 성과물인 SLAM 시스템의 접근성과 성능을 보장하고, 그 영향력을 극대화하는 핵심적인 '활성화 기술(enabling technology)'로서 기능합니다. 이는 연구 성과를 실제 사용자가 쉽게 활용할 수 있는 완전한 생태계로 발전시키려는 연구실의 성숙한 전략을 보여줍니다.


FAST-Calib은 사용자 편의성과 빠른 성능에 초점을 맞춘 실용적인 도구로, 그 설계 철학은 아키텍처, 캘리브레이션 타겟, 그리고 알고리즘 워크플로우 전반에 걸쳐 일관되게 나타납니다. 이 섹션에서는 FAST-Calib의 기술적 구성 요소를 심층적으로 분석하여 그 작동 원리와 설계상의 선택을 탐구합니다.


FAST-Calib의 아키텍처는 명확하고 간결한 목표를 중심으로 구축되었습니다. 공식 GitHub 저장소에서 강조하는 핵심 특징들은 이 도구의 설계 원칙을 잘 보여줍니다.19

- **핵심 목표:**
  1. **광범위한 LiDAR 지원:** 솔리드 스테이트(solid-state) LiDAR와 기계식 회전(mechanical spinning) LiDAR를 모두 지원하여 다양한 하드웨어 구성에 대한 범용성을 확보합니다.
  2. **초기값 불필요:** 캘리브레이션을 위해 사용자가 어떠한 초기 외재적 파라미터 추정치도 제공할 필요가 없어, 사용 편의성을 극대화합니다.
  3. **고속 및 고정밀:** 단 2초라는 매우 짧은 시간 안에 높은 정확도의 캘리브레이션 결과를 도출하는 것을 목표로 합니다.
- **시스템 의존성:** FAST-Calib은 로봇 공학 연구 및 개발의 표준 플랫폼인 ROS(Robot Operating System) 내에서 작동하도록 설계되었습니다. 포인트 클라우드 처리를 위해서는 PCL(Point Cloud Library)을, 이미지 처리를 위해서는 OpenCV(Open Source Computer Vision Library)를 핵심 라이브러리로 사용합니다.19 이러한 표준 라이브러리 채택은 대부분의 로봇 공학 연구자들이 쉽게 접근하고 사용할 수 있도록 합니다.
- **라이선스:** 이 프로젝트는 GPL-2.0 라이선스 하에 배포됩니다.19 이는 소스 코드를 사용, 수정, 배포할 수 있는 자유를 보장하지만, 파생된 작업물 역시 동일한 GPL 라이선스를 따라야 하는 '카피레프트(copyleft)' 조항을 포함합니다. 이는 학술적 연구 및 커뮤니티 기여를 장려하는 반면, 상업적 활용 시에는 라이선스 준수에 대한 법적 검토가 필요함을 의미합니다.


FAST-Calib의 성공적인 작동은 특별히 설계된 캘리브레이션 타겟에 크게 의존합니다. 이 타겟의 설계는 단순한 물리적 형태를 넘어, 알고리즘의 강건성과 정확성을 보장하기 위한 공학적 고려가 담겨 있습니다.

- **물리적 설계:** FAST-Calib이 사용하는 타겟은 여러 개의 원형 패턴(circular pattern)과 식별을 위한 QR 코드가 인쇄된 평평한 보드입니다.19 공식 저장소에는 이 타겟의 정확한 치수가 명시된 기술 도면이 제공되어 사용자가 직접 제작할 수 있도록 안내합니다.

- **알고리즘적 유산:** FAST-Calib의 README 파일은 "캘리브레이션 타겟 설계가 `velo2cam_calibration`에 기반한다"고 명시적으로 밝히고 있습니다.19 이는 FAST-Calib이 완전히 새로운 방법론을 창조한 것이 아니라, 이미 학계에서 검증되고 널리 사용되는 기존의 성공적인 연구를 기반으로 하고 있음을 시사합니다. 

  `velo2cam_calibration`은 LiDAR와 카메라 캘리브레이션을 위한 잘 알려진 오픈소스 패키지로, 그 신뢰성이 입증되었습니다.21 이러한 접근 방식은 "바퀴를 재발명하지 않는" 실용적인 공학적 선택으로, 검증된 기반 위에서 개발 기간을 단축하고 안정성을 확보하는 데 큰 이점을 가집니다.

- **원형 패턴의 이론적 근거:** 체커보드와 같은 다른 타겟 대신 원형 패턴을 선택한 것은 LiDAR 데이터의 특성을 고려한 전략적인 결정입니다. 체커보드의 모서리는 고해상도 이미지에서는 명확하게 검출되지만, 점들이 희소하게 분포하는 LiDAR 포인트 클라우드에서는 그 위치를 정확하게 특정하기 매우 어렵습니다.3 반면, 원형 패턴의 경우 LiDAR 포인트들이 원의 둘레를 따라 감지되므로, 이 점들을 이용하여 기하학적 원 모델(예: RANSAC을 이용한 원 피팅)을 강건하게 피팅하고 그 중심점을 매우 높은 정밀도로 추정할 수 있습니다.7 이 선택은 LiDAR-카메라 간의 대응점을 설정하는 과정에서 발생하는 가장 큰 난제 중 하나를 효과적으로 완화합니다.


FAST-Calib의 전체적인 워크플로우는 데이터 수집부터 최종 변환 행렬 계산까지 체계적인 단계로 구성됩니다.

1. **데이터 입력:** 사용자는 캘리브레이션 타겟이 포함된 정적인 장면을 촬영하여 포인트 클라우드 메시지가 담긴 `rosbag` 파일과 이에 해당하는 동기화된 이미지를 준비해야 합니다.19
2. **특징 추출 (카메라):** 시스템은 OpenCV를 사용하여 입력 이미지를 처리합니다. 이미지에서 원형 패턴들을 검출하고 각 원의 중심점 픽셀 좌표(2D)를 계산합니다. 이와 동시에, 사용자가 `qr_params.yaml` 파일에 제공한 카메라의 내부 파라미터(intrinsic matrix)와 왜곡 계수(distortion coefficients)를 사용하여 이미지 왜곡을 보정하고, 검출된 2D 중심점들을 정규화된 이미지 좌표계로 변환합니다.19
3. **특징 추출 (LiDAR):** PCL을 사용하여 `rosbag`의 포인트 클라우드 데이터를 처리합니다. 먼저, `qr_params.yaml`에 설정된 거리 필터(distance filter)를 적용하여 캘리브레이션 보드 평면에 속하지 않는 불필요한 포인트들을 제거합니다. 그 후, 남은 포인트들로부터 원형 패턴에 해당하는 포인트 클러스터를 분리하고, 각 클러스터에 RANSAC과 같은 강건한 모델 피팅 알고리즘을 적용하여 3차원 공간상의 원 중심점 좌표(3D)를 정밀하게 추출합니다.19
4. **최적화/정합 (Optimization/Registration):** 이 단계에서는 카메라와 LiDAR에서 각각 추출된 대응되는 원 중심점들의 집합을 사용합니다. 즉, 카메라에서 얻은 3차원 방향 벡터(2D 점을 3D로 역투영)와 LiDAR에서 직접 얻은 3차원 좌표점들 간의 대응 관계가 성립됩니다. 문제는 이 두 포인트 집합 간의 유클리드 거리를 최소화하는 최적의 강체 변환(회전 행렬 R과 평행 이동 벡터 t)을 찾는 고전적인 포인트 집합 정합(point-set registration) 문제로 귀결됩니다. FAST-Calib이 초기 추정값 없이 작동한다는 점 19은 특이값 분해(Singular Value Decomposition, SVD) 기반의 비반복적 해법이나 전역 최적화(global optimization) 기법이 사용되었을 가능성을 시사합니다. 이 과정을 통해 최종적인 외재적 파라미터가 계산됩니다.

이처럼 FAST-Calib은 검증된 알고리즘적 유산을 바탕으로, LiDAR 데이터의 특성에 최적화된 특징(원형 패턴)을 선택하고, 이를 표준 라이브러리를 통해 효율적으로 처리하는 실용적이고 강건한 워크플로우를 구현하고 있습니다.


FAST-Calib의 가치를 온전히 평가하기 위해서는 이 도구를 현재 존재하는 다양한 LiDAR-카메라 캘리브레이션 기술들의 광범위한 맥락 속에 위치시켜야 합니다. 이를 통해 FAST-Calib이 채택한 접근법의 상대적인 강점과 약점을 명확히 파악하고, 그 기술적 선택의 의미를 심도 있게 이해할 수 있습니다. 캘리브레이션 기술은 크게 타겟 기반(target-based), 타겟리스(target-free), 그리고 딥러닝 기반(deep learning-based) 패러다임으로 분류할 수 있습니다.


FAST-Calib은 명백히 타겟 기반 방법론에 속합니다. 이 패러다임은 특별히 제작된 인공적인 객체(타겟)를 사용하여 센서 데이터 간의 명확한 대응 관계를 설정하는 것을 특징으로 합니다.

- **타겟 종류에 따른 비교:**
  - **체커보드(Checkerboard):** 가장 전통적인 타겟으로, 카메라 내부 파라미터 캘리브레이션에 널리 사용됩니다.7 이미지에서는 모서리(corner) 검출이 용이하지만, 앞서 언급했듯이 희소한 LiDAR 포인트 클라우드에서는 모서리나 평면의 경계를 정확히 추출하기 어렵다는 단점이 있습니다.10
  - **선(Line) 특징:** 특정 환경(예: 건물 모서리)에서 자연적으로 발견되는 선 특징을 이용하거나 인공적인 선 타겟을 사용합니다.1 구현이 비교적 간단할 수 있으나, 선이 특정 방향으로만 치우쳐 있을 경우 모든 6-DoF 파라미터를 완전히 구속하지 못할 수 있습니다.
  - **원형 패턴(Circular Pattern):** FAST-Calib이 채택한 방식으로, LiDAR 데이터 처리에 특히 강점을 보입니다. 원의 중심은 희소한 포인트 데이터로부터 강건하게 추정될 수 있으며, 이는 체커보드 모서리보다 안정적인 3D 특징점을 제공합니다.7

이러한 비교를 통해 볼 때, FAST-Calib의 원형 패턴 타겟 채택은 LiDAR 센서의 데이터 특성을 깊이 고려한, 매우 합리적이고 효과적인 선택임을 알 수 있습니다.


캘리브레이션 분야의 또 다른 주요한 구분은 타겟의 사용 유무입니다. FAST-Calib의 타겟 기반 접근법은 타겟을 사용하지 않는 타겟리스 방법론과 뚜렷한 장단점을 보입니다.

- **타겟리스 방법론:** 이 접근법은 주변 환경에 자연적으로 존재하는 특징(예: 평면, 선, 모서리 등)을 활용하거나 1, 센서의 움직임 정보(motion)를 기반으로 캘리브레이션을 수행합니다.27
  - **장점:** 별도의 타겟을 설치할 필요가 없어 유연성이 높고, 환경에 구애받지 않으며, 잠재적으로 온라인(online) 또는 지속적인(continuous) 캘리브레이션이 가능합니다.1
  - **단점:** 환경에 충분한 기하학적 특징이 존재해야 한다는 제약이 있으며, 특징이 부족하거나 모호한 환경에서는 성능이 저하될 수 있습니다. 또한, 대응 관계 설정이 타겟 기반 방식보다 더 복잡하고 오류에 취약할 수 있습니다.
- **타겟 기반 방법론 (FAST-Calib):**
  - **장점:** 기하학적으로 명확하게 정의된 타겟을 사용하므로, 매우 정확하고 신뢰할 수 있는 대응 관계를 설정할 수 있습니다. 이는 일반적으로 타겟리스 방법보다 높은 정밀도를 달성할 수 있게 합니다.1 결과 검증 또한 상대적으로 용이합니다.
  - **단점:** 캘리브레이션을 위해 타겟을 특정 위치에 설치해야 하는 번거로움이 있으며, 타겟을 설치할 수 없는 넓은 야외나 특정 환경에서는 사용이 제한됩니다.

흥미롭게도, HKU MARS Lab은 FAST-Calib과 같은 타겟 기반 도구뿐만 아니라, 적응형 복셀화(adaptive voxelization)를 이용한 타겟리스 다중 LiDAR/카메라 캘리브레이션 도구(`mlcc`) 29나, 환경의 모서리 특징을 이용하는 타겟리스 캘리브레이션 도구(`livox_camera_calib`) 26도 개발하여 공개했습니다. 이는 연구실이 두 패러다임 모두에 깊은 전문성을 가지고 있음을 보여줍니다. 이러한 이중적인 접근은 "공짜 점심은 없다(no free lunch)"는 공학적 원리를 잘 이해하고 있음을 시사합니다. 즉, 특정 사용 사례에 따라 최적의 도구는 달라진다는 것입니다. 연구실 환경에서 새로운 SLAM 알고리즘을 테스트하기 위해 빠르고 신뢰성 있는 캘리브레이션이 필요할 때는 FAST-Calib과 같은 타겟 기반 방식이 우수합니다. 반면, 실제 주행 환경에서 자율적으로 캘리브레이션을 수행해야 하는 자율주행차에는 타겟리스 방식이 필수적입니다. 따라서 MARS Lab의 캘리브레이션 도구 포트폴리오는 중복이 아니라, 기초적인 실험실 설정부터 현장 적용까지 다양한 요구를 충족시키는 포괄적인 툴킷으로 보아야 합니다.


최근 몇 년간, 딥러닝 기술이 캘리브레이션 분야에도 활발히 도입되고 있습니다.2 기하학적 원리에 기반한 고전적인 도구인 FAST-Calib은 이러한 새로운 패러다임과 비교했을 때 뚜렷한 차이를 보입니다.

- **딥러닝 기반 방법론:** 이 방법들은 명시적인 특징 추출 및 정합 과정 대신, 심층 신경망을 사용하여 원시 센서 데이터(예: LiDAR 포인트 클라우드를 투영한 깊이 맵과 카메라 RGB 이미지)로부터 외재적 파라미터를 직접 회귀(regress)하는 방식으로 작동합니다.2
  - **정확성 및 강건성:** 이상적인 조건에서 기하학적 제약이 명확한 FAST-Calib은 매우 높은 정밀도를 보장하며 결정론적(deterministic)인 결과를 제공합니다. 반면, 딥러닝 모델은 학습 데이터에 포함되지 않은 새로운 환경에서는 일반화 성능이 저하될 수 있으며(generalization issue), 그 작동 원리를 해석하기 어려운 '블랙박스' 모델이라는 한계가 있습니다.2
  - **데이터 의존성:** FAST-Calib은 특정 타겟을 촬영한 소수의 정적 데이터만으로 작동합니다. 딥러닝 모델은 대규모의 다양한 학습 데이터셋을 필요로 하며, 이러한 데이터셋을 구축하는 것은 매우 어렵고 비용이 많이 듭니다.2
  - **복잡성 및 자동화:** FAST-Calib의 파이프라인은 각 단계가 명확하여 디버깅이 용이합니다. 딥러닝 모델은 완전 자동화가 가능하지만, 학습 과정이 복잡하고 많은 계산 자원을 요구합니다.

결론적으로, FAST-Calib은 고전적인 기하학 기반 방법론의 장점을 극대화한, 매우 잘 최적화된 도구입니다. 딥러닝이라는 새로운 패러다임이 부상하고 있음에도 불구하고, 연구 개발 환경에서 빠르고, 신뢰할 수 있으며, 해석 가능한 캘리브레이션이 필요할 때 FAST-Calib과 같은 접근법은 여전히 강력한 유효성과 경쟁력을 가집니다.

**표 1: LiDAR-카메라 캘리브레이션 패러다임 비교**

| 패러다임                   | 예시 도구/방법                           | 방법론                                                       | 타겟 요구사항                      | 자동화 수준                      | 주요 장점                                                    | 주요 단점                                                    |
| -------------------------- | ---------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **타겟 기반 (원형 패턴)**  | **HKU FAST-Calib**, velo2cam_calibration | 이미지와 포인트 클라우드에서 원형 패턴의 중심점을 추출하여 3D-3D 대응 관계를 설정하고, 포인트 집합 정합을 통해 변환 행렬 최적화. | 특정 디자인의 원형 패턴 보드 필요. | 높음 (데이터 수집 후 자동 처리). | 높은 정밀도와 신뢰성, 명확한 기하학적 제약, 결과 검증 용이.  | 타겟 설치의 번거로움, 타겟이 없는 환경에서는 사용 불가.      |
| **타겟리스 (기하학 기반)** | HKU mlcc, livox_camera_calib             | 환경에 자연적으로 존재하는 평면, 선, 모서리 등의 기하학적 특징을 추출하여 센서 데이터 간의 대응 관계 설정. | 필요 없음.                         | 중간~높음 (장면 특징에 의존).    | 높은 유연성, 온라인/지속적 캘리브레이션 가능, 환경 제약 적음. | 환경에 충분한 특징이 필요, 특징이 부족하면 성능 저하, 타겟 기반보다 정밀도가 낮을 수 있음. |
| **타겟리스 (딥러닝 기반)** | LCC-Net, DXQ-Net                         | 심층 신경망을 사용하여 원시 센서 데이터로부터 외재적 파라미터를 직접 회귀. 특징을 암시적으로 학습. | 필요 없음.                         | 매우 높음 (End-to-End).          | 완전 자동화, 복잡한 환경에 대한 잠재적 강건성.               | 대규모 학습 데이터 필요, 일반화 문제, 결과의 해석 불가능성(블랙박스), 결정론적이지 않음. |


이론적 분석을 넘어, FAST-Calib을 실제로 사용하는 연구자와 엔지니어를 위해 이 섹션에서는 구체적인 구현 절차와 커뮤니티에서 보고된 일반적인 문제들에 대한 해결 방안을 제시합니다. 이는 FAST-Calib의 성공적인 활용을 위한 실질적인 지침이 될 것입니다.


성공적인 캘리브레이션의 첫걸음은 올바른 환경 설정과 양질의 데이터 수집입니다.

- **시스템 설정:**
  1. **소프트웨어 환경:** FAST-Calib은 ROS를 기반으로 하므로, Ubuntu 운영체제(18.04 또는 20.04 권장)에 ROS(Melodic 또는 Noetic)가 설치되어 있어야 합니다.19
  2. **핵심 라이브러리 설치:** PCL(버전 1.8 이상)과 OpenCV(버전 4.0 이상)가 필요합니다. 이들은 ROS 설치 시 함께 설치되거나, `apt`를 통해 쉽게 추가할 수 있습니다.19
  3. **소스 코드 빌드:** FAST-Calib 저장소를 `catkin` 작업 공간의 `src` 폴더에 클론한 후, `catkin_make` 또는 `catkin build` 명령어를 사용하여 패키지를 빌드합니다.
- **데이터 수집 프로토콜 (Best Practices):**
  1. **정적 장면 캡처:** 캘리브레이션 중에는 LiDAR와 카메라 센서, 그리고 캘리브레이션 타겟이 모두 완전히 정지된 상태여야 합니다.
  2. **타겟 가시성 확보:** 타겟이 카메라의 시야에 완전히 들어오고, 조명이 충분하여 이미지에서 원형 패턴이 선명하게 보여야 합니다.
  3. **충분한 포인트 밀도:** LiDAR가 타겟 표면에 충분히 많은 포인트를 조사하도록 타겟을 적절한 거리에 배치해야 합니다. 너무 멀리 있으면 포인트가 희소해져 원 중심 추출이 불안정해질 수 있습니다.
  4. **다양한 포즈 캡처:** 캘리브레이션의 정확도를 높이기 위해, 센서와 타겟 간의 상대적인 위치와 방향을 여러 번 변경하며 데이터를 수집하는 것이 매우 중요합니다. 단일 포즈로 캘리브레이션을 수행하면 특정 축(특히 Z축)에 대한 구속 조건이 약해져 부정확한 결과를 초래할 수 있습니다. 최소 3~5개 이상의 다양한 포즈에서 데이터를 수집하는 것을 권장합니다.
  5. **데이터 기록:** 각 포즈에 대해 `rosbag record` 명령어를 사용하여 LiDAR의 포인트 클라우드 토픽과 카메라의 이미지 토픽을 동시에 기록합니다.


FAST-Calib GitHub 저장소의 'Issues' 섹션에는 사용자들이 겪는 다양한 문제들이 보고되어 있습니다.30 이를 분석하여 일반적인 실패 원인과 해결책을 정리하면 다음과 같습니다.

- **문제: 큰 캘리브레이션 오차 (특히 Z축 평행 이동 값)**
  - **증상:** "캘리브레이션 결과 오차가 매우 큰 문제에 관하여", "캘리브레이션 결과 평행 이동 Z값이 큰 문제"와 같은 이슈가 보고됨.30
  - **원인 분석 (Z축의 저주):** 이는 평면형 타겟을 사용할 때 발생하는 고전적인 문제입니다. 센서가 타겟을 정면에서 바라볼 때, 타겟 평면에 수직인 방향(일반적으로 Z축)의 평행 이동 값에 대한 관측 가능성(observability)이 매우 낮습니다. 즉, Z축으로 약간의 오차가 있어도 투영된 이미지 상의 변화가 미미하기 때문에, 최적화 과정에서 이 값을 정확하게 결정하기 어렵습니다.
  - **완화 전략:** **다양한 거리와 각도에서 데이터를 수집**하는 것이 가장 효과적인 해결책입니다. 타겟을 센서에 가깝게, 멀게, 그리고 비스듬한 각도로 여러 번 촬영하면 Z축 평행 이동을 구속하는 기하학적 기저(baseline)가 강화되어 훨씬 더 정확한 결과를 얻을 수 있습니다.
- **문제: 하드웨어 호환성 및 비이상적 설치**
  - **증상:** "더 많은 카메라 모델에 대한 지원을 추가할 수 있습니까?", "LiDAR 기울여 배치".30
  - **원인 분석:** 소프트웨어가 표준 핀홀(pinhole) 카메라 모델을 가정하고 있거나, 센서가 수평으로 설치되었다고 가정하는 로직이 내재되어 있을 수 있습니다.
  - **완화 전략:** 사용 중인 카메라가 어안(fisheye) 렌즈 등 비표준 모델인 경우, 캘리브레이션 전에 OpenCV 등을 사용하여 이미지를 핀홀 모델에 맞게 **사전 왜곡 보정(pre-rectify)**하는 것이 좋습니다. LiDAR가 기울어져 설치된 경우, 알고리즘의 강건성이 보장되지 않을 수 있으므로 가능한 한 **센서를 수평으로 맞추어 설치**하는 것이 최선입니다.
- **문제: 캘리브레이션 보드 자체의 결함**
  - **증상:** "SVG로 인쇄한 QR 코드가 더 작음", "보드 가장자리의 약간의 패임이 캘리브레이션에 영향을 줍니까?".30
  - **원인 분석:** 알고리즘은 타겟의 물리적 치수가 정확하다는 가정 하에 작동합니다. 인쇄된 패턴의 크기가 다르거나 보드가 평평하지 않으면, 최종 변환 행렬의 스케일(scale)이나 정확도에 직접적인 오류를 유발합니다.
  - **완화 전략:** 기술 도면에 명시된 **정확한 치수로 타겟을 인쇄**하고, 왜곡이나 구부러짐이 없는 **완벽하게 평평하고 단단한 표면(예: 포맥스, 아크릴)에 부착**해야 합니다.24 타겟이 손상되지 않도록 주의하여 보관하고 사용해야 합니다.
- **문제: 데이터 품질 및 알고리즘 민감도**
  - **증상:** "포인트 클라우드 크기가 일치하지 않음", "캘리브레이션 알고리즘의 LiDAR 포인트 클라우드 밀도 요구 사항에 대해 문의합니다.".30
  - **원인 분석:** 알고리즘이 원형 특징을 강건하게 검출하려면 일정 수준 이상의 포인트 밀도가 필요합니다. "Point cloud sizes do not match" 오류는 특징점 추출 또는 대응점 매칭 단계에서 실패했음을 의미하며, 이는 주로 데이터 품질 문제(예: 너무 희소한 포인트)에서 기인합니다.
  - **완화 전략:** 데이터 수집 시 **타겟이 LiDAR에 충분히 가까워** 원형 패턴 위에 조밀한 포인트가 맺히도록 해야 합니다. 각 센서의 데이터 스트림이 시간적으로 잘 동기화되었는지 확인하는 것도 중요합니다.

5.3. 결과 해석 및 검증

캘리브레이션이 완료되면, 그 결과를 신뢰할 수 있는지 반드시 검증해야 합니다.

- **정량적 검증:**
  - **변환 행렬 확인:** 출력된 회전 행렬과 평행 이동 벡터가 물리적인 센서 배치와 일치하는지 대략적으로 확인합니다. 예를 들어, 카메라가 LiDAR보다 10cm 오른쪽에 있다면, 평행 이동 벡터의 해당 축 성분이 약 0.1m 근처의 값을 가져야 합니다.
  - **RMSE (Root Mean Square Error):** 최적화 과정에서 계산된 RMSE 값은 정합된 포인트들 간의 평균 오차를 나타냅니다. 이 값이 낮을수록 정합이 잘 되었다고 볼 수 있습니다.
- **정성적 검증:**
  - **재투영 (Reprojection):** 가장 직관적이고 효과적인 검증 방법입니다. 계산된 외재적 파라미터를 사용하여 LiDAR 포인트 클라우드를 카메라 이미지 평면에 투영합니다. 그 후, 투영된 포인트에 이미지의 해당 픽셀 색상 값을 입혀 3D 뷰어(예: Rviz)에서 시각화합니다. 성공적인 캘리브레이션은 이미지 속 객체의 경계선과 색상이 입혀진 포인트 클라우드의 경계가 매우 정확하게 일치하는 모습을 보여줍니다.19 이 시각적 일치 여부가 캘리브레이션 품질의 최종적인 판단 기준이 됩니다.

**표 2: FAST-Calib 문제 해결 가이드**

| 증상 / 오류 메시지                                 | 가능한 원인                                                  | 권장 조치 / 완화 전략                                        |
| -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **캘리브레이션 오차가 큼 (특히 Z축 평행 이동 값)** | 1. 단일 또는 유사한 포즈에서만 데이터 수집.  2. Z축에 대한 기하학적 구속 조건 부족. | 1. 센서와 타겟 간의 거리와 각도를 다양하게 변경하며 최소 3~5개 이상의 포즈에서 데이터를 수집.  2. 타겟을 정면뿐만 아니라 비스듬한 각도에서도 촬영. |
| **"Point cloud sizes do not match" 오류 발생**     | 1. LiDAR 포인트 클라우드 밀도가 너무 낮아 원형 특징 검출 실패.  2. 이미지에서 조명 불량 등으로 원형 패턴 검출 실패.  3. LiDAR와 카메라의 시야가 거의 겹치지 않음. | 1. 타겟을 LiDAR에 더 가깝게 배치하여 포인트 밀도 확보.  2. 타겟에 충분한 조명을 제공하여 이미지 품질 향상.  3. 데이터 수집 전, 두 센서의 시야가 타겟을 포함하여 충분히 겹치는지 확인. |
| **비표준 카메라 모델(예: 어안 렌즈) 사용 시 실패** | 소프트웨어가 표준 핀홀 카메라 모델을 가정하고 있음.          | 1. OpenCV의 `cv::fisheye::undistortImage`와 같은 함수를 사용하여 이미지를 핀홀 모델에 맞게 사전 왜곡 보정.  2. 보정된 이미지를 사용하여 캘리브레이션 수행. |
| **인쇄된 타겟의 크기나 형태가 정확하지 않음**      | 1. 인쇄 설정 오류.  2. 타겟을 평평하지 않은 표면에 부착.     | 1. 제공된 기술 도면에 명시된 치수와 정확히 일치하도록 100% 스케일로 인쇄.  2. 인쇄물을 포맥스, 아크릴 등 단단하고 평평한 보드에 기포 없이 부착. |
| **LiDAR가 기울어져 설치된 경우 정확도 저하**       | 알고리즘이 센서가 수평으로 설치되었다는 암묵적 가정을 포함할 수 있음. | 1. 가능한 한 LiDAR 센서를 수평으로 정확하게 설치.  2. 기울기가 불가피한 경우, 결과의 정확도가 저하될 수 있음을 인지하고, 재투영을 통한 철저한 시각적 검증 수행. |


지금까지 HKU MARS Lab의 FAST-Calib을 방법론, 생태계 내에서의 역할, 그리고 실용적 관점에서 다각적으로 분석했다. 이 마지막 섹션에서는 분석 내용을 종합하여 FAST-Calib의 강점과 약점, 그리고 전략적 가치를 비판적으로 평가하고, 이를 바탕으로 센서 캘리브레이션 기술의 미래 발전 방향을 전망하고자 한다.


FAST-Calib은 여러 뚜렷한 강점을 가지고 있지만, 동시에 그 접근법에서 비롯되는 본질적인 한계도 명확합니다.

- **강점:**

  - **속도와 자동화:** "단 2초"라는 매우 빠른 처리 속도와 초기값 없이 작동하는 자동화된 프로세스는 이 도구의 가장 큰 장점입니다.19 이는 연구 개발 과정에서 반복적으로 수행해야 하는 캘리브레이션 작업의 부담을 크게 줄여줍니다.
  - **생태계 통합:** FAST-LIVO2와 같은 MARS Lab의 주력 SLAM 시스템과 완벽하게 연동되도록 설계되어, 사용자에게 원활하고 일관된 경험을 제공합니다.16 이는 단순한 편의성을 넘어, 전체 "FAST" 생태계의 완성도와 채택률을 높이는 핵심 요소입니다.
  - **강건한 특징 선택:** 희소한 LiDAR 데이터에 적합한 원형 패턴을 타겟 특징으로 선택한 것은 기술적으로 매우 현명한 결정으로, 알고리즘의 안정성과 정확성을 높이는 데 기여합니다.7

- **약점:**

  - **유연성 부족:** 물리적인 타겟에 의존하는 방식은 타겟을 설치하기 어려운 환경에서의 사용을 제한하며, 이는 타겟리스 방법론에 비해 유연성이 떨어진다는 명백한 한계입니다.1
  - **사용자 의존성:** GitHub 이슈 분석에서 드러났듯이, 캘리브레이션의 성공 여부는 양질의 데이터 수집, 정확한 타겟 제작, 올바른 하드웨어 설치 등 사용자의 숙련도와 주의력에 상당히 의존합니다.30 이는 완전 자동화라는 목표와는 다소 거리가 있습니다.
  - **기술적 독창성:** 이 도구는 `velo2cam_calibration`이라는 기존의 성공적인 연구를 기반으로 하므로, 방법론 자체의 학술적 독창성은 MARS Lab의 다른 타겟리스 캘리브레이션 연구(`mlcc`, `livox_camera_calib`)에 비해 상대적으로 낮습니다.19

- 전략적 가치:

  FAST-Calib의 진정한 가치는 학술적 독창성보다는 그 전략적 역할에 있습니다. 이 도구는 MARS Lab의 핵심 연구 자산인 고성능 SLAM 알고리즘을 더 많은 연구자들이 쉽게 사용하고 검증할 수 있도록 만드는 '실용적인 조력자(pragmatic enabler)'입니다. 복잡한 캘리브레이션 과정을 단순화함으로써 진입 장벽을 낮추고, 이를 통해 자신들의 핵심 연구 성과가 더 넓은 커뮤니티로 확산되도록 촉진하는 매우 효과적인 전략입니다. 즉, FAST-Calib은 생태계를 강화하고 연구의 영향력을 극대화하기 위한 필수적인 인프라 구축의 일환으로 평가해야 합니다.


FAST-Calib이 고전적인 패러다임의 우수한 구현체임은 분명하지만, 센서 캘리브레이션 분야의 연구 최전선은 이미 새로운 방향으로 나아가고 있습니다.

- **온라인, 타겟리스, 자가 캘리브레이션으로의 전환:** 미래의 자율 시스템은 실험실의 통제된 환경을 벗어나, 실제 세계의 동적인 변화에 스스로 적응해야 합니다. 이를 위해, 특정 시점에 오프라인으로 수행되는 타겟 기반 캘리브레이션에서 벗어나, 시스템이 작동하는 동안 지속적으로 환경 정보를 활용하여 스스로 캘리브레이션 오차를 감지하고 보정하는 **온라인(online), 타겟리스(target-free), 자가 캘리브레이션(self-calibration)** 기술이 핵심 연구 주제로 부상하고 있습니다.1
- **딥러닝의 역할 증대:** 딥러닝은 특징을 수동으로 설계할 필요 없이 데이터로부터 직접 학습할 수 있다는 강력한 장점을 가지고 있습니다. 앞으로 딥러닝은 더욱 정교한 특징을 학습하고, 다양한 환경 변화에 강건한 캘리브레이션 모델을 구축하는 데 중심적인 역할을 할 것입니다. 하지만, 학습 데이터 의존성, 일반화 성능, 결과의 해석 가능성 등 해결해야 할 과제 역시 명확합니다.2
- **하이브리드 접근법의 대두:** 미래의 가장 유망한 방향은 고전적인 기하학 기반 방법의 정밀성과 해석 가능성, 그리고 딥러닝 기반 방법의 강력한 특징 학습 능력과 자동화를 결합하는 **하이브리드(hybrid) 접근법**이 될 가능성이 높습니다. 예를 들어, 딥러닝으로 환경에서 강건한 특징 대응 관계를 대략적으로 찾은 후, 기하학적 최적화 기법으로 이를 정밀하게 다듬는 방식이 있을 수 있습니다.

결론적으로, HKU MARS Lab의 FAST-Calib은 빠르고 신뢰할 수 있는 캘리브레이션이 최우선 과제인 연구 개발 및 시스템 초기 설정 단계에서 여전히 매우 유효하고 강력한 도구입니다. 이는 고전적 패러다임이 얼마나 잘 최적화될 수 있는지를 보여주는 훌륭한 사례입니다. 동시에, 로봇 공학의 궁극적인 목표인 완전 자율화를 향한 여정 속에서, 캘리브레이션 기술은 더욱 지능적이고 적응적인 온라인 방식으로 끊임없이 진화해 나갈 것입니다. FAST-Calib은 그 진화의 여정에서 굳건한 디딤돌 역할을 수행하는, 시대를 대표하는 중요한 성과물로 기록될 것입니다.


1. Online, Target-Free LiDAR-Camera Extrinsic Calibration via Cross-Modal Mask Matching, accessed July 19, 2025, https://arxiv.org/html/2404.18083v2
2. A Review of Deep Learning-Based LiDAR and Camera Extrinsic ..., accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/12/3878
3. Extrinsic Calibration for LiDAR–Camera Systems Using Direct 3D–2D Correspondences, accessed July 19, 2025, https://www.mdpi.com/2072-4292/14/23/6082
4. You Only Calibrate Once for Accurate Extrinsic Parameter in LiDAR-Camera Systems - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.18043v1
5. A Novel Calibration Board and Experiments for 3D LiDAR and Camera Calibration - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7070607/
6. A Review of Deep Learning-Based LiDAR and Camera Extrinsic Calibration - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/381497701_A_Review_of_Deep_Learning-Based_LiDAR_and_Camera_Extrinsic_Calibration
7. Camera–LiDAR Calibration Using Iterative Random Sampling and Intersection Line-Based Quality Evaluation - MDPI, accessed July 19, 2025, https://www.mdpi.com/2079-9292/13/2/249
8. Deephome/Awesome-LiDAR-Camera-Calibration - GitHub, accessed July 19, 2025, https://github.com/Deephome/Awesome-LiDAR-Camera-Calibration
9. Survey of Extrinsic Calibration on LiDAR-Camera System for Intelligent Vehicle: Challenges, Approaches, and Trends | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=HAhJkhLUAl
10. Geometric calibration for LiDAR-camera system fusing 3D-2D and 3D-3D point correspondences, accessed July 19, 2025, https://opg.optica.org/abstract.cfm?URI=oe-28-2-2122
11. MARS LAB HKU - YouTube, accessed July 19, 2025, https://www.youtube.com/channel/UC_sj0oCXnrBzl2QDn3G38Ww/about
12. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 19, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
13. [2010.08196] FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2010.08196
14. FAST-LIO2: Fast Direct LiDAR-inertial Odometry, accessed July 19, 2025, https://arxiv.org/abs/2107.06829
15. MαRS Lab at HKU - YouTube, accessed July 19, 2025, https://www.youtube.com/playlist?list=PLBS7MEwCM1ZZ-56BjyCAyY2YtJClISqtD
16. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-LIVO2
17. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.14035v2
18. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/388354428_FAST-LIVO2_on_Resource-Constrained_Platforms_LiDAR-Inertial-Visual_Odometry_with_Efficient_Memory_and_Computation
19. hku-mars/FAST-Calib: A Handy Extrinsic Calibration Tool for LiDAR-camera Systems., accessed July 19, 2025, https://github.com/hku-mars/FAST-Calib
20. README.md - hku-mars/FAST-Calib - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-Calib/blob/main/README.md
21. beltransen/velo2cam_calibration: Automatic Extrinsic ... - GitHub, accessed July 19, 2025, https://github.com/beltransen/velo2cam_calibration
22. Automatic Extrinsic Calibration Method for LiDAR and Camera, accessed July 19, 2025, https://paperswithcode.com/paper/automatic-extrinsic-calibration-method-for
23. Improvement on LiDAR-Camera Calibration Using Square Targets - arXiv, accessed July 19, 2025, https://arxiv.org/html/2506.18294
24. Tips for calibrating / aligning the camera - Resources - LightBurn Software Forum, accessed July 19, 2025, https://forum.lightburnsoftware.com/t/tips-for-calibrating-aligning-the-camera/9365
25. Automatic LiDAR-Camera Calibration of Extrinsic Parameters Using a Spherical Target, accessed July 19, 2025, https://www.researchgate.net/publication/344981931_Automatic_LiDAR-Camera_Calibration_of_Extrinsic_Parameters_Using_a_Spherical_Target
26. hku-mars/livox_camera_calib: This repository is used for automatic calibration between high resolution LiDAR and camera in targetless scenes. - GitHub, accessed July 19, 2025, https://github.com/hku-mars/livox_camera_calib
27. Automatic Targetless Monocular Camera and LiDAR External Parameter Calibration Method for Mobile Robots - Semantic Scholar, accessed July 19, 2025, https://pdfs.semanticscholar.org/5b8e/d0ac60f2c3adb9032f52736799b2fa4c9603.pdf
28. Deep Learning-Based Online Automatic Targetless LiDAR–Camera Calibration with Iterative and Attention-Driven Post - arXiv, accessed July 19, 2025, [https://arxiv.org/pdf/2502.17648?](https://arxiv.org/pdf/2502.17648)
29. hku-mars/mlcc: Fast and Accurate Extrinsic Calibration for Multiple LiDARs and Cameras - GitHub, accessed July 19, 2025, https://github.com/hku-mars/mlcc
30. Issues / hku-mars/FAST-Calib - GitHub, accessed July 19, 2025, https://github.com/hku-mars/FAST-Calib/issues

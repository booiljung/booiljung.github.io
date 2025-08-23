# HKU-MaRS의 MLCC(Multiple LiDARs and Cameras extrinsic Calibration)에 대한 심층 고찰



자율주행 자동차, 로봇, 무인 항공기 등 현대 지능형 시스템의 핵심은 주변 환경을 정확하게 인식하는 능력에 있다. 이를 위해 단일 센서의 한계를 극복하고 강건성과 정밀도를 높이고자 상호 보완적인 특성을 지닌 여러 종류의 센서를 함께 사용하는 센서 융합(Sensor Fusion) 기술이 필수적으로 요구된다. 특히, 3차원 공간 구조 정보를 정밀하게 측정하는 LiDAR(Light Detection and Ranging)와 풍부한 색상 및 텍스처 정보를 제공하는 카메라는 가장 대표적인 조합으로, 이 둘의 데이터를 융합함으로써 시스템은 기하학적 정보와 의미론적 정보를 동시에 파악할 수 있게 된다.1

이러한 센서 융합이 성공적으로 이루어지기 위한 가장 근본적인 전제 조건은 바로 '외재적 캘리브레이션(Extrinsic Calibration)'이다. 외재적 캘리브레이션이란, 각기 다른 센서가 자신의 고유한 좌표계를 기준으로 데이터를 수집할 때, 이들 좌표계 간의 상대적인 위치(translation)와 회전(rotation) 관계, 즉 강체 변환(rigid-body transformation)을 정확하게 추정하는 과정을 의미한다. 이 변환 행렬을 알아야만 서로 다른 센서로부터 들어온 데이터를 하나의 통일된 좌표계로 정렬하여 의미 있는 정보로 결합할 수 있다. 따라서 정밀한 외재적 캘리브레이션은 전체 인식 파이프라인의 성능을 좌우하는 '초석(cornerstone)'과 같은 역할을 한다.5 만약 캘리브레이션에 미세한 오차라도 존재한다면, 이는 장애물의 위치를 잘못 인식하거나 주변 환경을 왜곡하여 해석하는 심각한 결과로 이어질 수 있으며, 궁극적으로는 시스템의 안전성을 치명적으로 위협하게 된다.5


다수의 LiDAR와 카메라로 구성된 현대적인 센서 시스템은 그 복잡성으로 인해 캘리브레이션 과정에서 여러 가지 기술적 난제에 직면한다.

첫째, **비중첩 또는 최소한의 시야(Non-overlapping or Minimal Field-of-View, FoV)** 문제이다. 과거에는 360도 회전하는 기계식 LiDAR가 주로 사용되어 다른 센서와의 시야각 중첩을 가정하기 용이했다. 그러나 최근에는 비용, 내구성, 크기 등의 이점으로 인해 제한된 시야각을 갖는 고정형(Solid-state) LiDAR가 여러 대 장착되는 추세이다.8 이러한 구성에서는 센서 간 시야가 전혀 겹치지 않는 경우가 흔히 발생하여, 두 센서에서 동시에 관측되는 공통 특징점을 찾아 정합하는 전통적인 캘리브레이션 방식의 적용이 근본적으로 불가능해진다.10 홍콩 과학기술대학교 MaRS 연구소(HKU-MaRS)의 MLCC는 바로 이 문제를 해결하는 것을 주요 목표 중 하나로 삼는다.8

둘째, **데이터 비동기성(Asynchronicity)** 문제이다. 시스템에 장착된 여러 센서는 하드웨어적으로 완벽하게 동기화되지 않는 한, 각기 다른 시점에 데이터를 획득한다. 특히 센서의 움직임을 기반으로 캘리브레이션을 수행하는 동작 기반(Motion-based) 방법에서 이러한 시간 불일치는 오차 누적의 주된 원인이 되어 캘리브레이션의 정밀도를 저하시킨다.12

셋째, **데이터 이질성(Heterogeneity)** 문제이다. LiDAR가 제공하는 3차원 희소 포인트 클라우드(sparse point cloud)와 카메라가 제공하는 2차원 조밀 픽셀(dense pixels) 데이터는 그 양식, 해상도, 노이즈 특성이 근본적으로 다르다.5 이처럼 서로 다른 모달리티(modality)의 데이터로부터 신뢰할 수 있는 공통 특징(correspondence)을 추출하고 정합하는 것은 캘리브레이션 과정에서 가장 어렵고 핵심적인 과제이다.


이러한 도전 과제들을 해결하기 위한 캘리브레이션 방법론은 크게 두 가지 패러다임으로 나눌 수 있다.

**타겟 기반(Target-based) 방식**은 체커보드(checkerboard), ChArUco 보드, 또는 특정 패턴이 인쇄된 평면 보드와 같은 인공적인 '타겟'을 사용한다.2 이 방식은 타겟의 기하학적 정보를 사전에 정확히 알고 있으므로, 각 센서 데이터에서 타겟의 특징(코너, 평면 등)을 검출하여 명확한 대응 관계를 수립할 수 있다. 그 결과, 매우 높은 정확도와 재현성을 확보할 수 있다는 장점이 있다. 하지만, 이는 조명이 일정하고 반사되는 물체가 없는 통제된 환경을 요구하며, 타겟을 설치하고 촬영하는 과정에 많은 수동적 개입이 필요하다.7 이러한 제약은 대규모 센서 시스템을 캘리브레이션하거나, 진동이나 온도 변화로 인해 센서의 정렬이 틀어졌을 때 현장에서 신속하게 재캘리브레이션하는 것을 거의 불가능하게 만든다. 이는 단순한 불편함을 넘어, 자율 시스템의 대규모 상용화와 지속적인 운영에 있어 시스템적인 장벽으로 작용한다.7

**타겟리스(Targetless) 방식**은 이러한 한계를 극복하기 위해 등장했다. 이 방식은 인공적인 타겟 대신, 주변 환경에 자연적으로 존재하는 특징들, 예를 들어 건물의 평면, 벽과 바닥이 만나는 모서리(에지), 코너 등을 활용한다.4 별도의 장비나 통제된 환경 없이, 시스템이 작동하는 실제 환경에서 캘리브레이션을 수행할 수 있으므로 온라인(online) 또는 현장(in-field) 캘리브레이션에 매우 적합하며, 확장성과 편의성이 뛰어나다. MLCC는 바로 이 타겟리스 패러다임에 속하며, 그중에서도 센서 플랫폼의 '동작(motion)'을 통해 시야가 겹치지 않는 센서들 간의 공통 관측 정보를 인위적으로 생성하는 독창적인 접근법을 취한다.11 이러한 접근 방식의 전환은 단순한 기술적 호기심이 아닌, 실제 현장에서의 운영 및 경제적 요구에 부응하기 위한 필연적인 진화 과정이라 할 수 있다.


HKU-MaRS의 MLCC(Multiple LiDARs and Cameras extrinsic Calibration)는 다중 센서, 특히 비중첩 시야각(non-overlapping FoV)을 갖는 센서 시스템의 외재적 캘리브레이션을 위해 설계된 타겟리스(targetless) 프레임워크이다. 이 방법론의 핵심은 정교하게 설계된 2단계 파이프라인과 효율적인 최적화 기법에 있다.


MLCC 방법론의 근간을 이루는 철학은 '동작을 통한 공통 특징 생성(Creating Co-visible Features through Motion)'이다. 전통적인 타겟리스 방법들은 센서들 간에 공통으로 관측되는 영역이 존재한다고 가정하지만, 다수의 소형 FoV 고정형 LiDAR를 사용하는 최신 시스템에서는 이러한 가정이 성립하지 않는다.8 MLCC는 이 문제를 해결하기 위해, 센서 플랫폼 자체에 의도적인 움직임, 특히 회전 운동을 가한다.10 이 움직임을 통해 시야가 전혀 겹치지 않던 센서들이 시간차를 두고 동일한 공간 영역을 스캔하게 되며, 결과적으로 시공간적인 공통 관측 정보(spatiotemporal co-visible features)가 생성된다. 이 접근법은 특정 센서 배치에 구애받지 않고 하드웨어 구성의 유연성을 극대화하며, 다양한 종류의 LiDAR와 카메라 조합에 효과적으로 적용될 수 있다.8


MLCC는 복잡한 다중 센서 캘리브레이션 문제를 보다 다루기 쉬운 두 개의 하위 문제로 분리하는 '분할 정복(divide and conquer)' 전략을 채택한다. 이는 단일 모달리티(LiDAR-LiDAR) 문제를 먼저 해결하여 고품질의 3D 참조 모델을 구축하고, 이를 기반으로 더 어려운 교차 모달리티(LiDAR-Camera) 문제를 푸는 방식으로, 전체 프로세스의 안정성과 강건성을 높인다.


캘리브레이션의 첫 번째 단계는 시스템에 장착된 모든 LiDAR 센서 간의 상대적인 기하학적 관계를 정립하는 것이다.

1. **문제 정식화:** 여러 LiDAR 중 하나를 기준(base) LiDAR로 설정한다. 그 후, 다른 모든 LiDAR의 기준 LiDAR에 대한 상대적 변환 행렬(외재적 파라미터) $E_L$과, 데이터 수집 동안의 기준 LiDAR의 궤적 $S$를 동시에 최적화하는 문제로 정식화한다.11

2. **LiDAR 번들 조정(Bundle Adjustment, BA):** 이 최적화 문제는 SLAM(Simultaneous Localization and Mapping) 분야에서 맵의 전역적 일관성을 확보하기 위해 사용되는 번들 조정(BA) 기법을 캘리브레이션에 적용한 것이다.8 핵심 아이디어는, 모든 LiDAR로부터 수집된 포인트 클라우드를 현재 추정된 궤적 

   $S$와 외재적 파라미터 $E_L$을 이용해 하나의 전역 맵으로 정합했을 때, 이 맵의 기하학적 일관성(consistency)이 최대가 되도록 하는 $S$와 $E_L$을 찾는 것이다.11 이 일관성은 주로 환경에 존재하는 평면(plane) 특징들이 얼마나 잘 정렬되는지로 측정된다.


LiDAR 간의 관계가 정립되면, 다음 단계는 이 LiDAR 시스템 전체와 각 카메라 간의 외재적 파라미터를 찾는 것이다.

1. **고품질 3D 맵 생성:** 1단계에서 최적화된 LiDAR 궤적 $S^*$와 LiDAR 간 외재적 파라미터 $E_L^*$를 사용하여, 모든 LiDAR의 포인트 클라우드를 하나의 좌표계로 통합한다. 이를 통해 매우 정밀하고 밀도 높은 전역 3D 포인트 클라우드 맵이 생성된다.11 이 맵은 2단계 캘리브레이션을 위한 안정적인 '가상 타겟(virtual target)' 역할을 한다.
2. **교차 모달리티 특징 정합:**
   - **3D 에지 추출:** 생성된 고품질 3D 맵에서 기하학적 특징, 특히 인접한 두 평면이 만나는 경계선인 '깊이 연속 에지(depth-continuous edge)'를 추출한다. 이 에지는 조명 변화에 강건하고 명확한 3D 구조를 나타내므로 LiDAR-카메라 정합에 효과적이다.11
   - **2D 에지 추출:** 각 카메라 이미지에서는 Canny 에지 검출기와 같은 표준 알고리즘을 사용하여 2D 에지 맵을 생성한다.11
   - **최적화:** 3D 에지 포인트들을 현재 추정된 LiDAR-카메라 변환 행렬 $E_C$를 이용해 2D 이미지 평면에 투영한다. 그 후, 투영된 3D 에지 포인트와 가장 가까운 2D 에지 라인까지의 거리(point-to-edge distance)를 잔차(residual)로 정의하고, 이 잔차들의 제곱 합을 최소화하는 비선형 최적화를 수행하여 최적의 $E_C^*$를 찾는다.11

**표 1: MLCC 핵심 특징 요약**

| 특징 (Feature)            | 설명 (Description)                                           | 핵심 장점 (Key Advantage)                                    | 관련 자료 (Relevant Sources) |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------- |
| **캘리브레이션 패러다임** | 타겟리스(Targetless), 동작 기반(Motion-based)                | 별도의 캘리브레이션 타겟이나 통제된 환경이 불필요하여 현장 적용성 및 편의성 극대화 | 11                           |
| **핵심 해결 과제**        | 비중첩 또는 최소 시야각(Non/Minimal FoV) 센서 시스템의 외재적 캘리브레이션 | 소형 FoV 고정형 LiDAR 등 다양한 센서 구성에 대한 높은 유연성 및 확장성 제공 | 8                            |
| **핵심 기술 1**           | 적응형 복셀화(Adaptive Voxelization)                         | 포인트 클라우드를 동적으로 분할하여 평면 특징 추출 및 정합 과정의 계산 속도를 획기적으로 향상 | 8                            |
| **핵심 기술 2**           | LiDAR 번들 조정(LiDAR Bundle Adjustment)                     | SLAM의 전역 최적화 기법을 적용하여 LiDAR 궤적과 외재적 파라미터를 동시에 최적화함으로써 높은 정확도 달성 | 10                           |
| **최종 산출물**           | LiDAR-LiDAR 및 LiDAR-Camera 쌍에 대한 외재적 변환 행렬(SE(3)) | 다중 센서 데이터를 하나의 통일된 좌표계로 융합할 수 있는 기반 제공 | 11                           |
| **구현**                  | 오픈소스 ROS 패키지                                          | 연구 및 개발 커뮤니티의 높은 접근성과 재현성, 폭넓은 활용 가능성 | 20                           |


MLCC의 높은 성능과 효율성은 두 가지 핵심 기술, 즉 '적응형 복셀화'와 '2차 미분을 활용한 번들 조정 최적화'에 기반한다. 이 두 기술은 계산 효율성을 극대화하면서도 높은 정확도를 달성하기 위한 전략적 선택의 결과물이다.


타겟리스 캘리브레이션에서 가장 계산 비용이 많이 드는 부분 중 하나는 서로 다른 시점이나 다른 센서에서 얻은 포인트 클라우드 간의 대응점(correspondence)을 찾는 과정이다. MLCC는 이 문제를 해결하기 위해 '적응형 복셀화(Adaptive Voxelization)'라는 독창적인 공간 분할 기법을 도입했다.

- **문제 정의와 해결책:** 포인트 클라우드를 일정한 크기의 복셀(voxel)로 나누는 기존 방식은 해상도 설정에 대한 근본적인 딜레마를 안고 있다. 복셀 크기를 작게 하면(고해상도) 공간을 지나치게 잘게 쪼개어 계산량이 폭증하고, 복셀 크기를 크게 하면(저해상도) 하나의 복셀 안에 여러 개의 작은 평면이나 복잡한 구조가 섞여 들어가 특징을 제대로 분리할 수 없게 된다.11 MLCC는 이 문제를 해결하기 위해, 초기에 큰 크기의 복셀로 공간을 분할한 뒤, 각 복셀에 포함된 포인트들의 분포를 분석하여 평면성을 검사한다. 만약 포인트들이 평면을 이루지 않는다고 판단되면, 해당 복셀을 8개의 작은 자식 복셀(octants)로 재귀적으로 분할한다. 이 과정은 모든 복셀이 평면 특징을 갖거나 미리 설정된 최소 크기에 도달할 때까지 반복된다.11
- **수학적 원리:** 특정 복셀의 평면성은 그 안에 포함된 포인트들의 3차원 좌표에 대한 공분산 행렬(covariance matrix)을 계산하고, 이 행렬의 고유값(eigenvalue)을 분석함으로써 판단한다. 포인트들이 이상적인 평면에 분포한다면, 평면의 법선 벡터(normal vector) 방향으로의 분산은 매우 작고, 평면을 이루는 두 주축 방향으로의 분산은 상대적으로 클 것이다. 이는 공분산 행렬의 세 고유값($λ_1≥λ_2≥λ_3$) 중 하나($λ_3$)가 다른 두 값에 비해 현저히 작은 형태로 나타난다. MLCC는 가장 큰 고유값에 대한 가장 작은 고유값의 비율이 특정 임계값(`feat_eigen_limit`)보다 작은지를 검사하여 평면성을 판단한다.11
- **기대 효과:** 이 적응형 방식은 MLCC의 성능에 두 가지 결정적인 기여를 한다. 첫째, 각 평면 특징의 크기에 맞게 복셀 크기가 동적으로 조절되므로, 크고 작은 평면들을 모두 효과적으로 분할할 수 있다. 둘째, 각 복셀이 단 하나의 평면 특징만을 포함하도록 보장함으로써, 대응점 탐색 과정을 "복셀 간의 1:1 매칭" 문제로 단순화한다. 이는 매 최적화 반복마다 k-d 트리와 같은 복잡한 자료구조를 생성하고 광범위한 탐색을 수행해야 했던 기존 방식의 계산 비용을 획기적으로 절감시킨다.8 즉, 적응형 복셀화는 최적화 루프 내에서 가장 큰 병목이었던 대응점 탐색 과정을 가속화하기 위한 핵심적인 전처리 단계이다.


적응형 복셀화를 통해 효율적으로 평면 특징과 그 대응 관계를 찾았다면, 다음은 이를 이용해 실제 외재적 파라미터를 추정하는 최적화 단계이다.

- **비용 함수(Cost Function) 정의:** LiDAR-LiDAR 캘리브레이션의 목표는 모든 LiDAR에서 관측된 포인트들이 하나의 전역 좌표계에서 동일한 평면 위에 최대한 가깝게 정렬되도록 하는 것이다. 이는 수학적으로, 각 평면 복셀(l)에 대해, 해당 복셀에 속한 포인트(Gpk)와 복셀이 정의하는 평면(법선 벡터 $n_l$, 평면 상의 한 점 $q_l$) 사이의 거리 제곱의 합을 최소화하는 문제로 정의할 수 있다.
  $$
  \arg \min_{S,E_L,n_l,q_l} \sum_l \frac{1}{N_l} \sum_{k=1}^{N_l} \| n_l^T (G_{p_k} - q_l) \|^2
  $$
  MLCC 논문에서는 이 비용 함수가 결국 각 평면 복셀의 공분산 행렬 $A_l$의 가장 작은 고유값 $\lambda_3(A_l)$의 합을 최소화하는 것과 동일함을 보인다.11 따라서 최종적인 최적화 문제는 기준 LiDAR의 궤적 

  $S$와 LiDAR 간 외재적 파라미터 $E_L$에 대한 함수로 다음과 같이 간결하게 표현된다.
  $$
  S^*, E_L^* = \arg \min_{S,E_L} \sum_l \lambda_3(A_l)
  $$
  **2차 미분을 이용한 최적화 가속:** 위 비용 함수는 최적화 변수 $S$와 $E_L$에 대해 고도의 비선형성을 가지므로, Levenberg-Marquardt(LM)와 같은 반복적 최적화 기법으로 해를 구해야 한다. 일반적인 1차 최적화 방법(경사 하강법 등)은 비용 함수의 기울기(Jacobian)만을 사용하여 수렴이 느리거나 지역 최적해에 빠지기 쉽다. MLCC의 핵심적인 수학적 기여는 이 비용 함수 $\lambda_3$를 최적화 변수에 대해 2차 미분한 값, 즉 헤시안 행렬(Hessian Matrix)을 해석적으로 유도한 데 있다.11 헤시안 행렬은 비용 함수의 곡률(curvature) 정보를 담고 있어, 이를 활용하면 최적해를 향해 훨씬 더 빠르고 정확한 방향으로 나아갈 수 있다. 이는 뉴턴 방법(Newton's method)의 원리를 LM 알고리즘에 접목한 것으로, 더 적은 반복 횟수로 안정적인 수렴을 가능하게 하여 전체 캘리브레이션 시간을 단축하는 데 결정적인 역할을 한다. 이러한 접근은 초기 계산의 복잡성을 감수하더라도 반복적인 최적화 과정에서의 실행 시간 성능을 우선시하는 MLCC의 핵심 설계 철학을 보여준다.

- **LiDAR-카메라 최적화:** LiDAR-카메라 캘리브레이션 단계에서는 다른 종류의 비용 함수가 사용된다. 1단계에서 생성된 3D 맵의 에지 포인트들을 카메라의 내재적 파라미터(핀홀 또는 어안 모델)를 이용해 2D 이미지 평면에 투영한다. 이 투영된 포인트와 이미지에서 검출된 가장 가까운 2D 에지 라인까지의 거리를 잔차(residual)로 정의한다. Google에서 개발한 비선형 최적화 라이브러리인 Ceres Solver를 사용하여, 이 잔차들의 제곱 합이 최소가 되는 LiDAR-카메라 외재적 변환 행렬 $E_C^*$를 반복적으로 찾아낸다.11


MLCC의 유효성은 관련 논문에서 제시된 정량적, 정성적 평가를 통해 입증되었다. 평가는 주로 캘리브레이션의 속도, 정확도, 그리고 다양한 조건 하에서의 강건성에 초점을 맞추고 있다.


MLCC의 가장 두드러진 성과는 계산 효율성의 극적인 향상이다.

- **캘리브레이션 속도:** 논문에 따르면, MLCC는 당시의 최신 기술(State-of-the-art, SOTA)로 여겨지던 방법들과 비교했을 때, LiDAR-LiDAR 외재적 캘리브레이션에 소요되는 시간을 평균 **15배** 단축시켰다. LiDAR-카메라 캘리브레이션의 경우에도 평균 **1.5배**의 속도 향상을 보였다.8 이 수치들은 각각 100회 및 50회의 독립적인 실험을 통해 얻어진 평균값으로, 그 신뢰도가 높다.8 이러한 극적인 속도 향상은, 비교 대상이 되었던 기존 연구들이 대응점 탐색을 위해 매 반복마다 k-d 트리 검색과 같이 계산 비용이 높은 방식을 사용했던 반면, MLCC는 적응형 복셀화를 통해 이 병목 구간을 효과적으로 제거했기 때문으로 분석된다. GitHub 리포지토리에서도 적응형 복셀화가 "이전 연구의 k-d 트리 검색 실행 시간을 충분히 절약한다"고 명시하고 있어, 이 성능 향상이 알고리즘의 핵심 설계에 기인함을 알 수 있다.20 반면, LiDAR-카메라 캘리브레이션의 속도 향상이 1.5배로 상대적으로 낮은 것은, 해당 단계의 병목이 3D 포인트 탐색뿐만 아니라 2D 이미지 처리(Canny 에지 검출) 및 교차 모달리티 최적화 등 적응형 복셀화만으로는 크게 개선되기 어려운 다른 요소들에 있기 때문으로 보인다.11 이는 알고리즘의 단계별 계산 프로파일에 대한 중요한 통찰을 제공한다.
- **캘리브레이션 정확도:** MLCC는 이러한 속도 향상을 "정확도를 유지하면서(while remaining accurate)" 달성했다고 강조한다.8 논문의 요약 정보에서는 재투영 오차(reprojection error)와 같은 구체적인 정확도 수치를 명시하고 있지는 않지만, 최종적으로 캘리브레이션된 파라미터를 이용해 생성한 컬러 포인트 클라우드의 시각적 품질을 통해 정성적인 정확도를 입증하고 있다.11 이미지의 색상 정보가 LiDAR 포인트 클라우드의 3D 구조에 이질감 없이 정확하게 투영된 결과물은 캘리브레이션이 성공적으로 이루어졌음을 시각적으로 보여준다.
- **강건성 (Robustness):** 알고리즘의 안정성을 평가하기 위해, 연구진은 8가지의 서로 다른 초기 추정값에서 출발하여 각각 100회의 독립적인 실험을 수행했다고 밝혔다.8 이는 알고리즘이 다소 부정확한 초기값에도 불구하고 안정적으로 올바른 해를 찾아갈 수 있는 능력을 갖추고 있음을 시사한다.


MLCC 방법론의 일반화 가능성을 입증하기 위해, 연구는 다양한 조건 하에서 수행되었다.

- **다양한 센서 구성:** 실험에는 전통적인 360도 회전형 LiDAR뿐만 아니라, Livox Avia와 같이 스캔 패턴, 포인트 밀도, FoV가 완전히 다른 최신 고정형(Solid-state) LiDAR가 포함되었다.20 이는 MLCC가 특정 종류의 센서에 국한되지 않고 다양한 하드웨어 구성에 적용될 수 있음을 보여준다.
- **실외 환경 테스트:** 모든 실험은 실제 자율주행 시나리오와 유사한 실외 테스트 장면(outdoor test scenes)에서 수행되었다.11 이는 통제된 실험실 환경을 넘어, 복잡하고 다양한 특징이 존재하는 실제 환경에서의 알고리즘의 유효성을 검증했다는 점에서 의의가 있다.


MLCC의 가장 큰 장점 중 하나는 연구 결과를 실제 사용자가 활용할 수 있도록 오픈소스 코드로 공개했다는 점이다. 이는 학술적 기여를 넘어 커뮤니티의 기술 발전에 직접적으로 이바지하는 중요한 요소이다.


MLCC의 소스 코드는 GitHub 리포지토리(hku-mars/mlcc)를 통해 공개되어 있으며, 로봇 공학 분야의 표준 플랫폼인 ROS(Robot Operating System) 환경에서 동작하도록 개발되었다.20 이를 통해 전 세계의 연구자들과 개발자들이 손쉽게 코드를 다운로드하여 자신의 시스템에 적용하고 검증하며, 나아가 개선에 기여할 수 있는 생태계가 구축되었다. 라이선스는 GPLv2를 따르며, 상업적 활용을 위해서는 연구 책임자에게 별도의 문의가 필요하다고 명시되어 있다.20


MLCC를 구동하기 위한 개발 환경과 의존성은 다음과 같다.

- **운영체제 및 ROS:** Ubuntu 16.04 (ROS Kinetic), 18.04 (ROS Melodic), 20.04 (ROS Noetic) 환경에서 테스트가 완료되었다.20
- **핵심 라이브러리:** 비선형 최적화를 위한 **Ceres Solver**, 포인트 클라우드 처리를 위한 **PCL(Point Cloud Library)**, 이미지 처리를 위한 **OpenCV**, 그리고 행렬 및 벡터 연산을 위한 **Eigen** 라이브러리가 필수적으로 요구된다.20 이들은 로보틱스 및 컴퓨터 비전 분야에서 널리 사용되는 표준 라이브러리들이다.
- **빌드 과정:** 설치는 `git clone` 명령어로 소스 코드를 내려받고, `catkin_make` 명령어를 통해 빌드하는 표준적인 ROS 패키지 설치 절차를 따른다.20


MLCC 패키지는 제공된 예제 데이터뿐만 아니라 사용자의 고유 데이터를 이용한 캘리브레이션을 지원한다.

- **실행 파이프라인:**
  - **다중 LiDAR 캘리브레이션:** 사용자의 편의와 디버깅을 위해 3단계로 분리되어 제공된다. 각 단계는 별도의 `roslaunch` 파일로 실행된다:
    1. `pose_refine.launch`: 기준 LiDAR의 자세(궤적)를 우선 최적화한다.
    2. `extrinsic_refine.launch`: 최적화된 기준 LiDAR 궤적을 고정한 채, 다른 LiDAR들과의 외재적 파라미터를 최적화한다.
    3. `global_refine.launch`: 기준 LiDAR의 자세와 외재적 파라미터를 동시에 최적화하여 최종 결과를 도출한다.20
  - **LiDAR-카메라 캘리브레이션:** `calib_camera.launch` 파일을 실행하여 수행한다.20
  - **단일 LiDAR-카메라 캘리브레이션:** 사용자의 편의를 위해 핀홀(pinhole) 및 어안(fisheye) 카메라 모델을 모두 지원하는 단일 센서 쌍 캘리브레이션 기능도 추가적으로 제공된다.20
- **사용자 데이터 적용 가이드:**
  - **데이터 수집:** 최적의 정밀도를 얻기 위해서는 센서 플랫폼이 **정지한 상태**에서 LiDAR 포인트 클라우드와 이미지를 수집하는 것이 권장된다. LiDAR 데이터는 `.pcd` 형식으로 저장해야 한다.20
  - **초기값 제공:** MLCC는 완전 자동 방식이 아니므로, 사용자는 **기준 LiDAR의 초기 궤적**과 **센서 간의 대략적인 초기 외재적 변환값**을 제공해야 한다. 이 초기값들은 A-LOAM과 같은 일반적인 SLAM 알고리즘이나 간단한 수동 측정, 혹은 hand-eye calibration을 통해 얻을 수 있다.20
  - **파라미터 튜닝:** 사용자는 자신의 데이터와 환경 특성에 맞게 주요 파라미터를 조정해야 한다. 특히 `adaptive_voxel_size`(적응형 복셀의 초기 크기), `feat_eigen_limit`(평면성 판단을 위한 고유값 비율 임계값), `downsmp_sz_base`(포인트 클라우드 다운샘플링 크기) 등은 캘리브레이션의 속도와 정밀도에 직접적인 영향을 미치는 중요한 변수들이다.20

이러한 요구사항들은 MLCC가 초심자를 위한 '플러그 앤 플레이' 솔루션이라기보다는, 알고리즘의 원리를 이해하고 적절한 초기값을 제공할 수 있는 전문가를 위한 강력한 도구임을 시사한다. 특히 초기값 제공 요구는 비선형 최적화 기법이 좋은 시작점 없이는 지역 최적해(local minima)에 수렴하기 쉬운 수학적 특성에 기인하며, 이는 실제 사용 가이드가 알고리즘의 이론적 기반과 깊이 연결되어 있음을 보여준다.


MLCC는 타겟리스 캘리브레이션 분야에서 중요한 진전을 이루었지만, 모든 기술과 마찬가지로 명확한 한계와 개선의 여지를 가지고 있다. 이러한 한계는 개발자 스스로 인지한 부분과 실제 사용자 커뮤니티를 통해 보고된 문제점, 그리고 방법론 자체에 내재된 근본적인 제약으로 나누어 살펴볼 수 있다.


개발팀은 GitHub 리포지토리의 '알려진 문제점(Known Issues)' 섹션을 통해 한 가지 사용성의 문제를 명시적으로 언급했다. 다중 LiDAR 캘리브레이션 과정이 현재 디버깅 목적으로 3개의 개별적인 단계로 나뉘어 있어, 사용자가 여러 번의 `roslaunch` 명령을 실행해야 하는 번거로움이 있다는 점이다. 개발팀은 향후 릴리즈에서 이 단계들을 하나로 통합하여 사용 편의성을 개선할 계획이라고 밝혔다.20


오픈소스 프로젝트의 진정한 가치는 사용자 커뮤니티의 피드백을 통해 그 실제적인 강점과 약점이 드러나는 데 있다. MLCC의 GitHub 'Issues' 탭에는 알고리즘의 이론적 한계가 실제 사용 환경에서 어떻게 발현되는지를 보여주는 중요한 보고들이 존재한다.

- **초기 추정값에 대한 민감성(Sensitivity to Initial Values):** 가장 빈번하게 제기되는 문제 중 하나는 LiDAR-카메라 캘리브레이션 결과가 사용자가 제공하는 초기 외재적 변환값에 따라 크게 달라진다는 점이다(Issue #30).23 이는 MLCC가 사용하는 비선형 최적화 알고리즘이 전역 최적해(global optimum)를 보장하지 못하고, 시작점에 따라 다른 지역 최적해(local minima)에 수렴할 수 있음을 명확하게 보여주는 실제 사례이다. 즉, 이론적으로 예측 가능한 지역 최적화의 함정이 실제 사용자들이 겪는 문제로 직접 나타난 것이다.
- **특정 하드웨어와의 호환성 문제(Hardware-Specific Issues):** 한 사용자는 상대적으로 포인트 밀도가 낮은 Velodyne-16 LiDAR로 수집한 데이터에서 양질의 복셀이 생성되지 않는 문제를 보고했다(Issue #23).23 이는 MLCC의 핵심 기술인 적응형 복셀화가 포인트 클라우드의 밀도와 스캔 패턴에 민감하게 반응할 수 있음을 시사한다. 알고리즘은 복셀 내 포인트들의 공분산 행렬과 고유값 분석을 통해 평면성을 판단하는데, 포인트가 너무 희소하면 공분산 행렬이 불안정해지거나, 평면을 정확하게 표현하지 못해 평면성 판단에 실패하게 된다. 따라서 특정 LiDAR의 특성이 알고리즘의 근본적인 가정(충분한 밀도의 평면 존재)과 맞지 않을 경우, 전체 캘리브레이션 파이프라인의 첫 단추부터 잘못 끼워질 수 있다.
- **파라미터 튜닝의 어려움:** 고유값 비율 임계값(`feat_eigen_limit`)의 계산 방식에 대한 질문(Issue #19) 등은 사용자들이 알고리즘의 내부 파라미터가 갖는 물리적, 수학적 의미를 완전히 이해하고 자신의 환경에 맞게 튜닝하는 데 상당한 어려움을 겪고 있음을 보여준다.24 이는 MLCC가 전문가용 도구로서의 성격이 강함을 다시 한번 확인시켜 준다.


사용자들이 보고한 문제들 외에도, MLCC의 방법론 자체에는 다음과 같은 근본적인 한계가 존재한다.

- **기하학적 특징에 대한 강한 의존성:** MLCC의 성공은 본질적으로 주변 환경에 **풍부하고 명확한 기하학적 특징**(LiDAR-LiDAR 단계에서는 '평면', LiDAR-카메라 단계에서는 '에지')이 존재한다는 가정에 기반한다. 따라서 긴 복도나 건물 외벽이 많은 구조적인(structured) 환경에서는 뛰어난 성능을 보일 수 있다. 하지만 식생이 무성한 숲길, 특징이 거의 없는 넓은 개활지, 또는 복잡한 비정형(unstructured) 환경에서는 신뢰할 만한 평면이나 에지를 충분히 추출하지 못해 성능이 급격히 저하되거나 캘리브레이션에 실패할 가능성이 높다. 이러한 한계는 MFCalib과 같이 깊이 불연속 에지, 강도 불연속 에지 등 더 다양하고 강건한 특징을 활용하려는 후속 연구들의 중요한 동기가 되었다.8
- **동작 모델링의 한계:** MLCC는 IMU와 같은 별도의 동작 센서 없이, 순수하게 LiDAR SLAM(논문에서는 LOAM을 사용)의 결과로부터 기준 센서의 초기 궤적을 추정한다.11 이는 LiDAR SLAM이 잘 동작하는 환경에서는 문제가 없지만, 특징이 부족하여 SLAM의 성능이 저하되는 환경(e.g., 긴 복도에서의 드리프트)이나 센서의 움직임이 매우 빠른 경우에는 부정확한 초기 궤적을 생성하게 된다. 부정확한 초기 궤적은 번들 조정 과정의 실패로 직결될 수 있으며, 이는 전체 캘리브레이션의 신뢰도를 떨어뜨리는 요인이 된다.

결론적으로, GitHub 이슈들은 단순한 버그 리포트가 아니라, MLCC가 가진 이론적 가정들이 실제 세계의 다양성과 불확실성을 만났을 때 발생하는 문제점들을 보여주는 귀중한 자료이다. 따라서 MLCC에 대한 비판적 고찰은 이러한 실제적 실패 사례들을 그 이론적 근원과 연결하여 분석하는 데서 출발해야 한다.


MLCC는 2021년에서 2022년 사이에 발표된 연구로서, 비중첩 FoV 문제를 해결하고 타겟리스 캘리브레이션의 속도를 획기적으로 개선한 중요한 기술적 성취이다.8 하지만 기술의 발전 속도는 매우 빠르며, MLCC가 발표된 이후 IMU와 딥러닝 기술이 캘리브레이션 분야에 본격적으로 도입되면서 새로운 패러다임이 등장했다. 따라서 MLCC의 현재 위치를 정확히 파악하기 위해서는 이러한 최신 기술 동향과의 비교 분석이 필수적이다. MLCC는 '기하학 기반 방법론의 정점'에 위치하면서도, 동시에 새로운 기술들이 해결하고자 하는 한계를 명확히 보여주는 과도기적 혁신으로 평가할 수 있다.


MLCC가 직면한 동작 모델링의 한계와 데이터 비동기성 문제를 해결하기 위한 가장 강력한 대안은 IMU(Inertial Measurement Unit)를 통합하는 것이다.

- **핵심 아이디어:** IMU는 외부 환경과 무관하게 매우 높은 주파수(수백 Hz)로 가속도와 각속도를 측정하므로, 센서의 움직임을 매우 정밀하게 모델링할 수 있다. 최신 연구들은 B-스플라인(B-spline)과 같은 연속 시간(continuous-time) 함수를 이용해 IMU 측정값으로 센서의 궤적을 표현한다.12 이 연속적인 궤적 함수가 있으면, 서로 다른 시간에 비동기적으로 측정된 LiDAR나 카메라 데이터라 할지라도 해당 측정 시점의 정확한 센서 자세를 오차 없이 보간하여 계산할 수 있다.
- **MLCC 한계 극복:**
  - **강건한 동작 추정:** MLCC가 환경 특징에 의존하는 LiDAR SLAM으로 초기 궤적을 추정하는 것과 달리, IMU Pre-integration 기법을 사용하면 특징이 거의 없는 환경이나 매우 빠른 움직임 속에서도 강건하게 동작을 추정할 수 있다.10
  - **시공간 동기화:** 연속 시간 표현은 최적화 변수에 센서 간의 시간 오프셋(time offset)을 명시적으로 포함시켜 함께 추정하는 것을 가능하게 한다. 이를 통해 공간적 외재적 파라미터뿐만 아니라 시간적 관계까지 완벽하게 교정하는 진정한 의미의 시공간(spatiotemporal) 캘리브레이션을 달성할 수 있다.12
- **대표 연구:** arXiv:2501.02821 12은 IMU 데이터를 기반으로 한 연속 시간 최적화를 통해 다수의 LiDAR와 카메라의 내재적/외재적 파라미터 및 시간 오프셋을 단일 최적화 프레임워크 내에서 동시에 추정하는 방법을 제안한다. 이는 MLCC의 다음 세대 기술이 나아갈 방향을 명확히 제시한다.


MLCC가 기하학적 특징의 부재로 인해 어려움을 겪는 문제를 해결하기 위해, 딥러닝을 활용하는 접근법이 활발히 연구되고 있다.

- **핵심 아이디어:** 사람이 직접 설계한 기하학적 특징(hand-crafted features)인 평면이나 에지 대신, 딥러닝 신경망을 통해 데이터로부터 직접 캘리브레이션에 유용한 특징을 학습(learned features)하거나, 더 나아가 '자동차', '사람', '표지판'과 같은 객체 수준(object-level)의 의미론적(semantic) 정보를 이용해 센서 간의 정합을 수행한다.29
- **MLCC 한계 극복:**
  - **특징 부족 환경에서의 강건성:** MLCC가 실패하는 비정형 환경이나 특징이 부족한 도로에서도, 딥러닝 모델은 이미지와 포인트 클라우드에서 공통적으로 나타나는 객체를 탐지하고, 이들의 외형, 상대적 위치, 의미론적 클래스 등을 종합적으로 고려하여 강건한 대응 관계를 찾아낼 수 있다.29
  - **자동화 및 사용 편의성:** 일부 딥러닝 기반 방법들은 초기 추정값 없이 원시 데이터(raw data)를 입력받아 외재적 파라미터를 직접 회귀(regress)하는 종단간(end-to-end) 방식으로 설계되어, 사용자의 개입을 최소화하고 편의성을 극대화한다.34
- **대표 연구:**
  - **MFCalib:** 깊이 연속/불연속 에지, 강도 불연속 에지 등 사람이 설계 가능한 다양한 종류의 특징을 종합적으로 활용하여 단 한 번의 데이터 획득(single-shot)만으로 강건한 캘리브레이션을 수행하는, 기하학 기반 방법의 확장된 형태를 보여준다.25
  - **CalibRefine:** YOLOv8과 같은 최신 객체 탐지기를 사용하여 LiDAR와 카메라에서 각각 객체를 검출하고, 학습된 판별기(discriminator)를 통해 두 센서에서 검출된 객체가 동일한지를 판단한다. 이렇게 수립된 객체 수준의 대응 관계를 바탕으로 초기 캘리브레이션을 수행하고, 이를 온라인으로 계속해서 정제해 나가는 정교한 프레임워크를 제시한다.29


각 캘리브레이션 패러다임은 고유의 장단점을 가지며, 이는 기술의 선택이 결국 해결하고자 하는 문제와 운영 환경에 따라 달라져야 함을 의미한다.

------

**표 2: 주요 캘리브레이션 패러다임 비교 분석**

| 패러다임 (Paradigm)      | 핵심 원리 (Core Principle)                                   | 장점 (Pros)                                                  | 단점 (Cons)                                                  | 대표 방법 (Example Method) |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------- |
| **타겟 기반**            | 인공적인 캘리브레이션 타겟의 알려진 기하학적 특징을 이용한 정합 | 높은 정확도, 높은 재현성, 명확한 성능 검증                   | 수동 개입 필요, 통제된 환경 요구, 현장 적용 및 확장성 낮음   | ChArUco-based methods 6    |
| **기하학 기반 타겟리스** | 환경의 자연적인 기하학적 특징(평면, 에지)과 센서의 움직임을 이용한 정합 | 타겟 불필요, 높은 편의성, 구조적 환경에서 높은 효율성        | 특징 부족 환경에서 실패, 초기 추정값에 민감, 동작 모델링의 정확성에 의존 | **MLCC** 8                 |
| **IMU 활용 연속 시간**   | IMU의 고주파 측정값과 연속 시간 궤적을 이용한 시공간적 정합  | 강건한 동작 추정, 센서 비동기 문제 해결, 시간 오프셋 캘리브레이션 가능 | IMU 바이어스 및 노이즈에 민감, 추가적인 센서(IMU) 요구       | arXiv:2501.02821 12        |
| **딥러닝 기반**          | 학습된 특징 또는 객체 수준의 의미론적 정보를 이용한 정합     | 특징 부족 환경에서 강건, 높은 수준의 자동화, 초기값 불필요 가능 | 대규모 학습 데이터 필요, 학습되지 않은 환경/객체에 대한 일반화 성능 문제, 결과의 해석 어려움 | CalibRefine 29, MFCalib 25 |

이러한 분석을 통해 볼 때, 캘리브레이션 기술의 미래는 어느 한 가지 방법론이 다른 것을 완전히 대체하기보다는, 각 방법론의 장점을 결합하는 하이브리드(hybrid) 형태로 나아갈 가능성이 높다. 예를 들어, IMU로 강건한 동작 사전 정보(motion prior)를 확보하고, 환경에 기하학적 특징이 풍부할 때는 MLCC와 같은 방법으로 정밀한 제약 조건을 추가하며, 특징이 부족한 구간에서는 딥러닝 기반의 객체 정합으로 보완하는 식의 통합 프레임워크를 구상할 수 있다. 이러한 관점에서 MLCC는 단독으로 사용될 때의 한계에도 불구하고, 그 핵심 기술인 적응형 복셀화 등은 미래의 하이브리드 시스템에서 중요한 모듈로 계속해서 활용될 수 있는 잠재력을 지니고 있다.



홍콩 과학기술대학교 MaRS 연구소에서 개발한 MLCC는 다중 센서 외재적 캘리브레이션 분야, 특히 타겟리스 방식의 발전에 있어 중요한 이정표를 제시한 연구이다. 본 고찰을 통해 확인된 MLCC의 핵심적인 학술적, 실용적 기여는 다음과 같이 요약할 수 있다.

첫째, MLCC는 센서 간 시야가 겹치지 않는(non-overlapping FoV) 현실적인 문제에 대한 효과적인 해결책을 제시했다. 의도적인 움직임을 통해 공통 관측 영역을 생성하는 접근법은 소형 고정형 LiDAR가 다수 사용되는 최신 자율 시스템의 하드웨어 구성에 높은 유연성을 부여했다.

둘째, '적응형 복셀화'와 '2차 미분을 활용한 번들 조정'이라는 두 가지 핵심 기술을 통해 타겟리스 캘리브레이션의 계산 효율성을 획기적으로 개선했다. 특히 LiDAR-LiDAR 캘리브레이션 속도를 15배 향상시킨 것은, 복잡한 최적화 문제의 병목 구간을 정확히 파악하고 이를 알고리즘 수준에서 해결한 뛰어난 성과이다.

셋째, 연구 결과를 ROS 패키지 형태의 오픈소스로 공개함으로써, 전 세계 연구 및 개발 커뮤니티가 해당 기술을 쉽게 활용하고 검증하며, 이를 기반으로 새로운 연구를 수행할 수 있는 토대를 마련했다.

물론, MLCC는 명확한 한계 또한 가지고 있다. 환경 내에 존재하는 평면이나 에지와 같은 기하학적 특징에 대한 강한 의존성은 비정형 환경에서의 적용을 어렵게 만들며, 비선형 최적화에 내재된 문제로 인해 사용자가 제공하는 초기 추정값에 결과가 민감하게 반응한다. 이러한 한계점들은 MLCC가 특정 조건 하에서 매우 강력한 도구이지만, 모든 상황에 적용 가능한 만능 해결책은 아님을 시사한다.


MLCC와 그 이후에 등장한 최신 기술 동향을 종합적으로 분석할 때, 미래의 타겟리스 캘리브레이션 기술은 다음과 같은 방향으로 발전할 것으로 전망된다.

- **하이브리드 접근법의 보편화:** 미래의 가장 강건한 캘리브레이션 시스템은 단일 패러다임에 의존하기보다 여러 기술을 융합하는 하이브리드 형태가 될 것이다. IMU의 관성 측정으로 정밀한 동작 모델을 수립하고, 환경에 명확한 기하학적 특징이 존재할 때는 이를 활용해 높은 정확도를 확보하며, 특징이 부족한 상황에서는 딥러닝 기반의 의미론적 정보로 보완하는 방식이 대표적이다. 각 기술이 서로의 약점을 보완하며 모든 시나리오에서 안정적인 성능을 제공하는 것을 목표로 할 것이다.
- **온라인 및 자가 캘리브레이션(Online & Self-Calibration)의 중요성 증대:** 자율 시스템이 장시간 운용되면서 발생하는 진동, 충격, 온도 변화 등으로 인해 센서의 정렬은 미세하게 틀어질 수 있다. 미래의 시스템은 이러한 변화를 실시간으로 감지하고 스스로 캘리브레이션을 수정하는 능력을 갖추어야 한다. CalibRefine과 같은 연구는 이러한 온라인 자가 보정 패러다임의 가능성을 보여주는 초기 단계의 연구라 할 수 있다.29
- **정밀도와 강건성의 조화:** 궁극적인 목표는 타겟 기반 방식이 제공하는 높은 수준의 정밀도와 재현성을, 타겟리스 방식이 제공하는 유연성과 편의성으로 달성하는 것이다. 이를 위해서는 각 센서의 노이즈 특성을 정밀하게 모델링하고, 캘리브레이션 결과의 불확실성을 정량적으로 추정하며, 잘못된 특징 정합(outlier)을 효과적으로 제거하는 기술들이 더욱 정교하게 발전해야 할 것이다.

결론적으로, MLCC는 기하학 기반 타겟리스 캘리브레이션 기술의 가능성과 한계를 동시에 보여준 중요한 연구이다. MLCC가 해결한 문제와 남긴 과제들은 이후 등장한 IMU 및 딥러닝 기반의 새로운 접근법들이 탄생하는 자양분이 되었으며, 이 모든 기술의 융합을 통해 자율 시스템의 '눈'은 더욱더 정확하고 신뢰할 수 있는 방향으로 진화해 나갈 것이다.


1. What Is Lidar-Camera Calibration? - MATLAB & Simulink - MathWorks, accessed August 7, 2025, https://www.mathworks.com/help/lidar/ug/lidar-camera-calibration.html
2. A Review of Deep Learning-Based LiDAR and Camera Extrinsic Calibration - PMC, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11207430/
3. Online Extrinsic Calibration on LiDAR-Camera System with LiDAR Intensity Attention and Structural Consistency Loss - MDPI, accessed August 7, 2025, https://www.mdpi.com/2072-4292/14/11/2525
4. Automatic Targetless LiDAR-Camera Calibration: A Survey - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/363244175_Automatic_Targetless_LiDAR-Camera_Calibration_A_Survey
5. A Target-based Multi-LiDAR Multi-Camera Extrinsic Calibration System - arXiv, accessed August 7, 2025, https://arxiv.org/html/2507.16621v1
6. A Target-based Multi-LiDAR Multi-Camera Extrinsic Calibration System - arXiv, accessed August 7, 2025, http://arxiv.org/pdf/2507.16621
7. How does multi-sensor calibration work in autonomous vehicles? - EE World Online, accessed August 7, 2025, https://www.eeworldonline.com/how-does-multi-sensor-calibration-work-in-autonomous-vehicles/
8. Targetless Extrinsic Calibration of Multiple Small FoV LiDARs and Cameras Using Adaptive Voxelization | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/360817362_Targetless_Extrinsic_Calibration_of_Multiple_Small_FoV_LiDARs_and_Cameras_Using_Adaptive_Voxelization
9. Multi-LiCa: A Motion- and Targetless Multi - LiDAR-to-LiDAR Calibration Framework - arXiv, accessed August 7, 2025, https://arxiv.org/html/2501.11088v1
10. Efficient and accurate LiDAR mapping : from extrinsic calibration to global optimization - HKU Scholars Hub, accessed August 7, 2025, https://hub.hku.hk/handle/10722/341601
11. (PDF) Fast and Accurate Extrinsic Calibration for Multiple LiDARs ..., accessed August 7, 2025, https://www.researchgate.net/publication/354597587_Fast_and_Accurate_Extrinsic_Calibration_for_Multiple_LiDARs_and_Cameras
12. [2501.02821] Targetless Intrinsics and Extrinsic Calibration of Multiple LiDARs and Cameras with IMU using Continuous-Time Estimation - arXiv, accessed August 7, 2025, https://arxiv.org/abs/2501.02821
13. Targetless Intrinsics and Extrinsic Calibration of Multiple LiDARs and Cameras with IMU using Continuous-Time Estimation - arXiv, accessed August 7, 2025, https://arxiv.org/html/2501.02821v1
14. (PDF) A Target-based Multi-LiDAR Multi-Camera Extrinsic Calibration System, accessed August 7, 2025, https://www.researchgate.net/publication/393923560_A_Target-based_Multi-LiDAR_Multi-Camera_Extrinsic_Calibration_System
15. [2507.16621] A Target-based Multi-LiDAR Multi-Camera Extrinsic Calibration System - arXiv, accessed August 7, 2025, https://arxiv.org/abs/2507.16621
16. A Review of Deep Learning-Based LiDAR and Camera Extrinsic Calibration - MDPI, accessed August 7, 2025, https://www.mdpi.com/1424-8220/24/12/3878
17. Automatic Targetless Monocular Camera and LiDAR External Parameter Calibration Method for Mobile Robots - MDPI, accessed August 7, 2025, https://www.mdpi.com/2072-4292/15/23/5560
18. Adaptive Point-Line Fusion: A Targetless LiDAR–Camera Calibration Method with Scheme Selection for Autonomous Driving - PMC, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10891561/
19. Extrinsic Calibration of Multiple LiDARs of Small FoV in Targetless Environments | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/349538566_Extrinsic_Calibration_of_Multiple_LiDARs_of_Small_FoV_in_Targetless_Environments
20. hku-mars/mlcc: Fast and Accurate Extrinsic Calibration for Multiple LiDARs and Cameras - GitHub, accessed August 7, 2025, https://github.com/hku-mars/mlcc
21. ZhaoXiaoyu/mlcc - Gitee, accessed August 7, 2025, https://gitee.com/zhaoxiaoyu001/mlcc?skip_mobile=true
22. [2109.06550] Targetless Extrinsic Calibration of Multiple Small FoV LiDARs and Cameras using Adaptive Voxelization - arXiv, accessed August 7, 2025, https://arxiv.org/abs/2109.06550
23. Issues / hku-mars/mlcc / GitHub, accessed August 7, 2025, https://github.com/hku-mars/mlcc/issues
24. judge_eigen / Issue #19 / hku-mars/mlcc / GitHub, accessed August 7, 2025, https://github.com/hku-mars/mlcc/issues/19
25. MFCalib: Single-shot and Automatic Extrinsic Calibration for LiDAR and Camera in Targetless Environments Based on Multi-Feature Edge | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/383701021_MFCalib_Single-shot_and_Automatic_Extrinsic_Calibration_for_LiDAR_and_Camera_in_Targetless_Environments_Based_on_Multi-Feature_Edge
26. MFCalib: Single-shot and Automatic Extrinsic Calibration for LiDAR and Camera in Targetless Environments Based on Multi-Feature Edge - arXiv, accessed August 7, 2025, https://arxiv.org/html/2409.00992v1
27. [Literature Review] Targetless Intrinsics and Extrinsic Calibration of Multiple LiDARs and Cameras with IMU using Continuous-Time Estimation - Moonlight, accessed August 7, 2025, https://www.themoonlight.io/en/review/targetless-intrinsics-and-extrinsic-calibration-of-multiple-lidars-and-cameras-with-imu-using-continuous-time-estimation
28. Targetless Intrinsics and Extrinsic Calibration of Multiple LiDARs and Cameras with IMU using Continuous-Time Estimation - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/387767088_Targetless_Intrinsics_and_Extrinsic_Calibration_of_Multiple_LiDARs_and_Cameras_with_IMU_using_Continuous-Time_Estimation
29. Deep Learning-Based Online Automatic Targetless LiDAR–Camera Calibration with Iterative and Attention-Driven Post - arXiv, accessed August 7, 2025, [https://arxiv.org/pdf/2502.17648?](https://arxiv.org/pdf/2502.17648)
30. Deep Learning based Online Extrinsic Calibration of IMU and Monocular Camera Using Smartphone Devices | Technical Program - IEEE/ION PLANS 2023, accessed August 7, 2025, https://www.ion.org/plans/abstracts.cfm?paperID=12035
31. Deep Learning for Camera Calibration and Beyond: A Survey - arXiv, accessed August 7, 2025, https://arxiv.org/html/2303.10559v2
32. CFNet: LiDAR-Camera Registration Using Calibration Flow Network - MDPI, accessed August 7, 2025, https://www.mdpi.com/1424-8220/21/23/8112
33. CalibRefine: Deep Learning-Based Online Automatic Targetless LiDAR–Camera Calibration with Iterative and Attention-Driven Post-Refinement - arXiv, accessed August 7, 2025, https://arxiv.org/html/2502.17648v3
34. Automatic Extrinsic Calibration of 3D LIDAR and Multi-Cameras Based on Graph Optimization - PMC, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8954836/
35. [R] Deep Learning-Based Single Image Camera Calibration : r/MachineLearning - Reddit, accessed August 7, 2025, https://www.reddit.com/r/MachineLearning/comments/idnsm2/r_deep_learningbased_single_image_camera/
36. [Literature Review] MFCalib: Single-shot and Automatic Extrinsic Calibration for LiDAR and Camera in Targetless Environments Based on Multi-Feature Edge - Moonlight, accessed August 7, 2025, https://www.themoonlight.io/en/review/mfcalib-single-shot-and-automatic-extrinsic-calibration-for-lidar-and-camera-in-targetless-environments-based-on-multi-feature-edge
37. CalibRefine: Deep Learning-Based Online Automatic Targetless LiDAR–Camera Calibration with Iterative and Attention-Driven Post-Refinement - arXiv, accessed August 7, 2025, https://arxiv.org/html/2502.17648v4
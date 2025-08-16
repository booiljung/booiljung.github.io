# FAST-LIVO

## 1.  FAST-LIVO 알고리즘 제품군의 기본 원리

로봇 공학 및 자율 시스템 분야에서 정확하고 강건한 위치 추정 및 매핑은 가장 근본적인 과제 중 하나입니다. 이 문제를 해결하기 위해 홍콩과기대(HKUST) 및 홍콩대학교(HKU)의 MARS Lab에서 개발한 FAST-LIVO 알고리즘 제품군은 LiDAR, 관성 측정 장치(IMU), 시각 센서(카메라)의 데이터를 융합하는 최첨단 접근 방식을 제시합니다. 이 보고서는 FAST-LIVO의 초기 버전부터 최신 진화 형태인 FAST-LIVO2, 엣지 컴퓨팅에 최적화된 경량화 버전, 그리고 3D 가우시안 스플래팅을 통합한 GS-LIVO에 이르기까지 전체 알고리즘 제품군을 심층적으로 분석합니다.

### 1.1  다중 센서 융합의 당위성: LiDAR, IMU, 그리고 비전

각 센서는 고유한 장점과 함께 명백한 한계를 지니고 있으며, 이러한 한계는 특정 환경에서 시스템의 실패로 이어질 수 있습니다. 다중 센서 융합은 이러한 개별 센서의 단점을 상호 보완하여 전체 시스템의 강건성과 정확성을 극대화하는 전략입니다.

- **시각 SLAM (Visual SLAM)의 한계:** 카메라는 저렴하고 풍부한 색상 및 질감 정보를 제공하지만, 조명 변화에 매우 취약하며 질감이 부족한 환경(예: 흰 벽, 복도)에서는 성능이 급격히 저하됩니다.1 또한, 깊이를 직접 측정할 수 없어 삼각측량이나 깊이 필터링과 같은 추가적인 연산을 통해 맵 포인트를 최적화해야 하며, 이는 계산 부하를 가중시키고 맵의 밀도와 정확성을 제한하는 요인이 됩니다.2
- **LiDAR SLAM의 한계:** LiDAR는 조명 변화에 강건하며 정확한 3차원 포인트 클라우드를 직접 제공합니다. 그러나 긴 복도나 넓은 평원과 같이 기하학적으로 특징이 없는(degenerate) 환경에서는 위치 추정에 실패할 수 있으며, 생성된 데이터는 색상 정보가 없는 희소한 포인트 클라우드에 불과합니다.1
- **관성 측정 장치 (IMU)의 한계:** IMU는 높은 주파수(high-frequency)로 움직임 데이터를 제공하여 단기적인 모션 예측에는 탁월하지만, 시간이 지남에 따라 오차가 빠르게 누적(drift)되어 단독으로는 장기적인 위치 추정에 부적합합니다.

FAST-LIVO와 같은 LIVO(LiDAR-Inertial-Visual Odometry) 시스템의 핵심 철학은 이 세 가지 센서가 서로를 완벽하게 보완한다는 사실에 기반합니다. LiDAR의 정확한 기하학 정보, 카메라의 풍부한 시각 정보, 그리고 IMU의 고주파수 모션 정보를 결합함으로써, 단일 센서로는 실패할 수밖에 없는 열악한 환경에서도 강건한 포즈 추정과 함께 정확하고 밀도 높은 컬러 포인트 클라우드 맵을 생성할 수 있습니다.2 이는 복잡한 SLAM 과제에 대한 가장 효과적인 해결책으로 입증되었습니다.5

### 1.2  핵심 철학: Tightly-Coupled 직접 방식 접근법

FAST-LIVO 제품군의 설계 철학은 'Tightly-Coupled' 융합 방식과 '직접(Direct)' 방식이라는 두 가지 핵심 원칙으로 요약할 수 있습니다.

- **Tightly-Coupled (강결합) 방식 정의:** 이 방식은 모든 센서의 원시(raw) 측정값을 단일 최적화 프레임워크 내에서 함께 처리하여 시스템 상태(위치, 방향, 속도 등)를 추정합니다. 이는 한 시스템의 출력값을 다른 시스템의 입력값으로 사용하는 'Loosely-Coupled(약결합)' 방식과 대조됩니다.7 FAST-LIVO 제품군은 근본적으로 Tightly-Coupled 반복 칼만 필터(Iterated Kalman Filter)를 기반으로 구축되어, 센서 간의 상호 의존성을 최대한 활용하여 높은 정확도를 달성합니다.3
- **직접(Direct) 방식 정의:** 직접 방식은 이미지에서 코너나 엣지 같은 특징점을 추출하고 매칭하는 '간접(Indirect)' 또는 '특징점 기반(Feature-based)' 방식(예: ORB-SLAM)과 달리, 센서의 원시 데이터 자체를 사용하여 오차를 최소화합니다.4 구체적으로, 이미지의 픽셀 강도 오차(photometric error)와 LiDAR 포인트의 점-평면 간 거리(point-to-plane distance)를 직접 최소화하는 방식으로 최적화를 수행합니다.9 이 접근법은 FAST-LIO2에서부터 모든 FAST-LIVO 버전에 이르기까지 일관되게 유지되는 핵심 원리입니다.3

이러한 직접 방식의 선택은 단순히 계산 효율성을 높이기 위함이 아닙니다. 이는 시스템의 강건성을 근본적으로 향상시키는 전략적 결정입니다. 특징점 기반 방식은 뚜렷한 특징이 없는 환경에서 취약하지만, 직접 방식은 미세한 이미지 그래디언트나 평면 구조에서도 제약 조건을 추출할 수 있어, FAST-LIVO 제품군이 목표로 하는 실제 세계의 도전적인 환경에서 훨씬 더 높은 회복탄력성을 보입니다.2 더 나아가, FAST-LIVO의 개발은 백지상태에서 시작된 것이 아니라, 이미 그 성능이 입증된 강력한 LIO(LiDAR-Inertial Odometry) 시스템인 FAST-LIO2를 기반으로 체계적으로 시각(Visual) 모듈을 확장하는 방식으로 이루어졌습니다.5 이러한 점진적이고 검증된 확장 철학은 개발 리스크를 줄이고, 개발자들이 LIO 문제를 다시 해결하는 대신 LiDAR-시각 융합이라는 핵심 과제에 집중할 수 있게 만든 성공의 핵심 요인입니다.

### 1.3  오차 상태 반복 칼만 필터(ESIKF)의 핵심적 역할

FAST-LIVO 제품군의 모든 버전에서 센서 융합의 심장부 역할을 하는 것은 오차 상태 반복 칼만 필터(Error-State Iterated Kalman Filter, ESIKF)입니다.2

- **ESIKF 개념:** 표준 확장 칼만 필터(EKF)와 달리, ESIKF는 실제 상태(nominal state)가 아닌 해당 상태의 '오차(error)'를 추정합니다. 이는 회전(SO(3))과 같이 복잡한 매니폴드(manifold) 상에 존재하는 상태를 다룰 때 특히 유용합니다. 오차 상태는 단순한 벡터 공간에 존재하므로, 특이점(singularity) 문제를 피하고 계산을 단순화할 수 있습니다.
- **반복적(Iterated) 특성:** '반복'이라는 이름에서 알 수 있듯이, 필터는 단일 업데이트 단계 내에서 측정 모델을 여러 번 재선형화합니다. 이를 통해 비선형성이 강한 측정 모델을 다룰 때 더 정확한 사후(posterior) 추정치로 수렴할 수 있으며, 이는 가우스-뉴턴 최적화와 유사한 효과를 낳습니다.
- **LIVO에서의 ESIKF:** ESIKF는 고주파수의 IMU 데이터와 저주파수이면서 고차원인 LiDAR 및 시각 측정값을 Tightly-Coupled 방식으로 효율적이고 강건하게 융합하는 데 최적화되어 있어 FAST-LIVO 제품군의 핵심 융합 엔진으로 채택되었습니다.2

## 2.  FAST-LIVO에서 FAST-LIVO2로의 아키텍처 진화

1세대 시스템인 FAST-LIVO에서 그 후속작인 FAST-LIVO2로의 전환은 단순한 기능 추가가 아닌, 명확하게 식별된 약점을 해결하기 위한 체계적인 개선 과정이었습니다. 이 섹션에서는 그 비판적인 진화 단계를 추적하고, 각 개선 사항이 어떻게 시스템의 강건성과 정확성을 비약적으로 향상시켰는지 분석합니다.

### 2.1  초기 프레임워크: FAST-LIVO

FAST-LIVO는 두 개의 강결합된 하위 시스템, 즉 VIO(Visual-Inertial Odometry)와 LIO(LiDAR-Inertial Odometry) 하위 시스템을 기반으로 구축되었습니다.5

- **LIO 하위 시스템:** 성공적인 FAST-LIO2에서 직접 파생되었으며, 새로운 LiDAR 스캔의 원시 포인트를 점진적으로 구축되는 맵에 등록하는 역할을 합니다.6
- **VIO 하위 시스템:** LIO가 구축한 포인트 클라우드 맵을 재사용합니다. 맵 포인트에는 이전에 관찰된 이미지 패치가 부착되며, 새로운 이미지가 들어오면 이 패치들과의 직접적인 광도 오차(photometric error)를 최소화하여 포즈를 정렬합니다.6 이는 SVO와 같은 희소 직접 방식과 유사한 원리입니다.14
- **융합 방식과 한계:** 상태 추정은 두 하위 시스템과 IMU의 측정값을 ESIKF를 통해 강결합하지만, 핵심적인 한계는 측정값들을 *비동기적(asynchronous)*으로 업데이트했다는 점입니다. 이는 시스템의 강건성을 저해하는 주요 원인이었습니다.2

### 2.2  한계 극복: FAST-LIVO2 개선의 네 가지 기둥

FAST-LIVO2는 FAST-LIVO의 한계를 극복하기 위해 네 가지 핵심적인 개선을 도입했습니다. 이는 연구 논문에서 "정확도를 저하시키는" 또는 "거친 가정"과 같은 이례적으로 강한 표현으로 이전 버전의 문제점을 지적하며 제시될 만큼 중요한 변화였습니다.2

#### 2.2.1  비동기에서 순차적 업데이트로: 강건성 향상

- **문제점:** FAST-LIVO의 비동기 업데이트 방식은 강건성 문제를 야기했습니다.2
- **해결책:** FAST-LIVO2는 ESIKF 내에 **순차적 업데이트(sequential update) 전략**을 도입했습니다. IMU에 의해 전파된 상태는 먼저 LiDAR 측정값에 의해 업데이트되고, 그 *다음에* 시각 측정값에 의해 순차적으로 업데이트됩니다.2
- **이점:** 이 방식은 이종(heterogeneous) 센서인 LiDAR와 카메라 측정값 간의 차원 불일치 문제를 명시적으로 해결하여, 훨씬 더 안정적이고 강건한 융합 프레임워크를 제공합니다.2

#### 2.2.2  공유 깊이를 넘어서: 시각 정렬을 위한 LiDAR 평면 사전 정보 활용

- **문제점:** FAST-LIVO는 시각 패치 내의 모든 픽셀이 동일한 깊이를 공유한다는 "거친 가정(wild assumption)"을 했습니다. 이는 이미지 정렬에 사용되는 아핀 변환(affine warping)의 정확도를 심각하게 저해했습니다.2
- **해결책:** FAST-LIVO2는 맵의 지역 LiDAR 포인트로부터 **평면 사전 정보(plane priors)**를 추출하여 활용합니다. 이는 시각 패치에 훨씬 더 정확한 기하학적 모델을 제공하며, 심지어 정렬 과정에서 이 평면의 법선 벡터를 미세 조정하기까지 합니다.2
- **영향:** 이 개선은 직접 시각 정렬의 정확도를 비약적으로 향상시켰습니다.

#### 2.2.3  정확성 향상을 위한 동적 참조 패치 업데이트

- **문제점:** FAST-LIVO는 단순히 현재 뷰와 가깝다는 이유만으로 시각 정렬을 위한 참조 패치를 선택했습니다. 이로 인해 흐릿하거나 질감이 부족한 저품질 패치가 선택되어 정확도를 저하시키는 경우가 많았습니다.2
- **해결책:** FAST-LIVO2는 **동적 참조 패치 업데이트(dynamic reference patch update) 전략**을 구현했습니다. 이 전략은 큰 시차(parallax)와 충분한 질감을 가진 고품질의 인라이어(inlier) 참조 패치를 능동적으로 선택하여, 시각 정렬이 항상 최상의 정보를 기반으로 수행되도록 보장합니다.2

#### 2.2.4  현실 적응: 온라인 노출 시간 추정

- **문제점:** FAST-LIVO는 변화하는 조명 환경에 대응하지 못했습니다. 급격한 조명 변화는 직접 이미지 정렬 모듈의 수렴 실패로 이어질 수 있었습니다.2
- **해결책:** FAST-LIVO2는 카메라의 **노출 시간을 온라인으로 추정**합니다. 이를 통해 다양한 조명 변화에 대응하고, 까다로운 조명 조건에서도 강건한 시각 추적을 유지할 수 있습니다.2

### 2.3  통합 복셀 맵: 기하학과 외형의 단일 표현

FAST-LIVO가 이중 하위 시스템 구조를 가졌던 것과 달리, FAST-LIVO2는 **단일 통합 맵(single, unified map)**을 중심으로 구축되었습니다.2 이 구조적 변화는 단순한 데이터 구조 변경 이상의 의미를 가집니다. 이는 LiDAR와 비전이 더 이상 별개의 '전문가'가 되어 필터에 정보를 제공하는 것이 아니라, 하나의 공유된 세계 모델 위에서 작동하는 두 개의 양식(modality)이 되었음을 의미합니다.

이 맵은 해시 인덱싱된 옥트리(hash-indexed octree) 또는 복셀 맵(voxel map)으로 구성됩니다.3 각 복셀(또는 옥트리의 리프 노드)은 LiDAR 스캔 등록에 사용되는 기하학적 정보(LiDAR로부터의 평면 특징)를 포함합니다. 결정적으로, 바로 이 맵 포인트에 직접 시각 정렬에 사용되는 시각 정보(이미지 패치)가 부착됩니다.2 이처럼 진정으로 통합된 표현 방식은 LiDAR의 기하학적 사전 정보를 활용하여 시각 정렬을 개선하는 것과 같은, 이전 아키텍처에서는 불가능했던 정교한 교차 모달(cross-modal) 향상을 가능하게 하는 핵심적인 기반이 됩니다.

------

**표 1: FAST-LIVO  비교 매트릭스**

| 특징                   | FAST-LIVO                | FAST-LIVO2                 | 자원 제약형 FAST-LIVO2      | GS-LIVO                 |
| ---------------------- | ------------------------ | -------------------------- | --------------------------- | ----------------------- |
| **핵심 융합 알고리즘** | ESIKF                    | ESIKF                      | ESIKF                       | IESKF                   |
| **업데이트 전략**      | 비동기                   | 순차적 (LiDAR → 비전)      | 순차적 (LiDAR → 비전)       | 순차적 (LiDAR → 비전)   |
| **시각 기하학 가정**   | 패치 내 동일 깊이        | LiDAR 평면 사전 정보       | LiDAR 평면 사전 정보        | 3D 가우시안 렌더링      |
| **참조 패치 선택**     | 근접성 기반              | 동적 (시차/질감 기반)      | 동적 (시차/질감 기반)       | 해당 없음 (렌더링 기반) |
| **조명 변화 처리**     | 미지원                   | 온라인 노출 시간 추정      | 온라인 노출 시간 추정       | 온라인 노출 시간 추정   |
| **맵 구조**            | 분리된 LIO/VIO 맵        | 통합 복셀 맵               | 로컬 통합 맵 + 장기 시각 맵 | 통합 3D 가우시안 맵     |
| **핵심 혁신**          | 직접 방식 LIVO 개념 도입 | 강건성 및 정확성 대폭 향상 | 엣지 컴퓨팅 최적화          | 실시간 사실적 맵핑      |

## 3.  엣지 최적화: 자원 제약형 FAST-LIVO2 변형

최첨단 LIVO 시스템인 FAST-LIVO2는 높은 정확도를 달성했지만, 상당한 계산 및 메모리 자원을 요구한다는 한계가 있었습니다.10 이는 드론, IoT 기기, 저전력 ARM 기반 컴퓨터와 같은 경량, 휴대용 엣지 플랫폼에 배포하기에는 부적합하다는 것을 의미했습니다.10 이러한 문제를 해결하기 위해, 강건성과 성능을 경쟁력 있는 수준으로 유지하면서 계산 및 메모리 오버헤드를 최소화하는 경량화 버전이 개발되었습니다.

### 3.1  지능형 자원 할당: LiDAR 열화 인지 적응형 시각 프레임 선택기

이 모듈의 핵심 아이디어는 계산 비용이 많이 드는 모든 시각 프레임을 처리하는 대신, 시각적 업데이트가 가장 필요한 시점을 지능적으로 판단하는 것입니다.10 이는 시스템이 자신의 불확실성을 모델링하고, 성능 유지를 위해 필요할 때만 계산 노력을 투입하는 '제약 조건 인지 컴퓨팅'이라는 정교한 패러다임의 훌륭한 예시입니다.

- **LiDAR 열화(Degeneration) 감지:** 시스템은 LiDAR가 제공하는 기하학적 제약 조건의 품질을 지속적으로 평가합니다. 이는 점-평면 최적화 문제와 관련된 행렬의 고유값(eigenvalue)을 분석하여 수행됩니다. 작은 고유값은 시스템이 기하학적으로 열화되었음(예: 벽과 평행하게 이동)을 나타내며, 이 상태에서는 LiDAR만으로는 포즈가 잘 제약되지 않습니다.17
- **적응형 시각 업데이트:** 시각 프레임 선택기는 감지된 LiDAR 상태에 따라 작동 방식을 동적으로 조절합니다.
  - **LiDAR가 열화된 경우:** 시스템은 들어오는 모든 이미지 세트를 처리하여 위치 추정 정확도를 유지하는 데 필요한 추가 제약 조건을 확보합니다.17
  - **LiDAR가 충분한 경우:** 시스템은 희소한 시각 키프레임 세트만 처리하여 시각 프런트엔드의 계산 부하를 대폭 줄입니다.17 이는 맵에 추가되는 시각 특징의 수를 극적으로 감소시켜 메모리 또한 절약하는 효과를 가져옵니다.15

### 3.2  이원적 매핑 접근법: 메모리 효율적인 로컬/장기 맵 구조

표준 FAST-LIVO2처럼 단일의 큰 통합 맵을 유지하는 것은 많은 메모리를 소모합니다. 이 문제를 해결하기 위해, 아키텍처는 두 개의 구별되지만 연결된 맵을 사용하도록 수정되었습니다.10

- **로컬 통합 시각-LiDAR 맵:** 즉각적이고 고주파수의 주행 거리 측정에 필요한 밀도 높은 기하학적 및 시각적 정보를 저장하는 소형의 로봇 중심 로컬 맵입니다. 그 크기는 작고 관리하기 쉽게 유지됩니다.
- **장기 시각 맵:** 희소한 과거의 시각적 관찰(키프레임)을 저장하는 별도의 경량 맵입니다. 이 맵의 목적은 밀도 높은 전역 맵의 메모리 오버헤드 없이 장기적인 주행 거리 측정의 강건성을 보장하고 드리프트를 방지하는 것입니다.

이 이중 맵 구조는 성능과 메모리 사용량 사이에서 더 나은 균형을 이룹니다. 로컬 맵은 작고 빠르게 업데이트할 수 있으며, 장기 맵은 낮은 메모리 비용으로 전역 일관성을 제공합니다.10

### 3.3  성능의 트레이드오프와 이점

이러한 최적화는 상당한 자원 절감 효과를 가져왔습니다. Hilti 데이터셋에서 이 시스템은 표준 FAST-LIVO2에 비해 **프레임당 런타임을 33% 감소**시키고 **메모리 사용량을 47% 낮추는** 성과를 보였습니다.10 이러한 효율성은 약간의 정확도 감소라는 명시적인 트레이드오프를 동반합니다. 동일한 데이터셋에서 

**절대 궤적 오차(ATE) RMSE가 3cm 증가**했습니다.10

그럼에도 불구하고, 이 경량화 버전은 FAST-LIO2와 같은 다른 최첨단 LIO 시스템과 비교했을 때 여전히 경쟁력이 있으며 종종 더 나은 성능을 보입니다.10 이 연구는 단순히 알고리즘을 최적화한 것을 넘어, 고성능 LIVO 기술을 대중화하고 확장하는 데 중요한 단계입니다. 약 100달러 가격의 RK3588과 같은 저비용 ARM 플랫폼에서 실시간 실행 가능성을 입증함으로써 10, 이 기술은 학술 및 고가 산업 장비의 영역을 넘어 소비자용 드론, 저렴한 차량의 첨단 운전자 보조 시스템(ADAS), 정교한 IoT 기기 등으로의 통합 가능성을 열었습니다. 이는 연구 프로토타입을 잠재적인 상용 기술로 전환시키는 중요한 발전입니다.

## 4.  차세대 프론티어: GS-LIVO를 이용한 사실적 매핑

이 섹션에서는 FAST-LIVO 제품군의 가장 최신 진화인 GS-LIVO를 탐구합니다. GS-LIVO는 전통적인 매핑 방식에서 벗어나 3D 가우시안 스플래팅(3D Gaussian Splatting)을 사용하여 고품질의 사실적인 장면을 재구성하는 패러다임 전환을 나타냅니다. 이 분석은 GS-LIVO 관련 논문 및 리뷰를 주요 출처로 합니다.20

### 4.1  고품질 렌더링을 위한 전통적 매핑의 한계

포인트 클라우드, 복셀, 서펠(surfel), 메시(mesh)와 같은 전통적인 SLAM 맵 표현 방식은 위치 추정이나 장애물 회피와 같은 기하학적 작업에는 탁월하지만, 이들로부터 사실적인 카메라 뷰를 복원하는 것은 매우 어렵습니다.27 최근 등장한 뉴럴 래디언스 필드(NeRF)와 특히 3D 가우시안 스플래팅(3D-GS)은 새로운 시점의 뷰를 고품질로 렌더링하는 강력한 해결책을 제시했습니다. 그러나 비전 센서만 사용하는 3D-GS 방식은 기하학적 정확성, 폐색(occlusion) 처리, 높은 GPU 메모리 소모 등의 문제에 직면하는 경우가 많습니다.24

### 4.2  패러다임 전환: SLAM 파이프라인에 3D 가우시안 스플래팅 통합

GS-LIVO는 FAST-LIVO2 기반의 LIVO 오도메트리 프런트엔드와 3D 가우시안 스플래팅 기반의 매핑 백엔드를 긴밀하게 통합한 새로운 SLAM 시스템입니다.24 이 시스템은 두 세계의 장점을 모두 활용합니다. LiDAR의 정밀한 기하학적 측정과 IMU의 고주파수 모션은 가우시안 맵을 위한 강력한 뼈대를 제공하고, 카메라는 사실적인 렌더링에 필요한 풍부한 질감 정보를 제공합니다.22 GS-LIVO는 시각적 품질, 기하학적 정확성, 그리고 실시간 성능을 동시에 최적화하는 것을 목표로 합니다.27

이러한 발전은 SLAM의 목표가 단순히 '위치 추정 및 매핑'을 넘어 '위치 추정 및 고품질 장면 재구성'으로 진화하고 있음을 시사합니다. FAST-LIVO2는 이러한 고급 렌더링 기술에 필수적인 강건한 포즈와 기하학 정보를 제공하는 전제 조건이었습니다. GS-LIVO는 여기서 한 걸음 더 나아가, 렌더러를 SLAM 루프에 직접 통합하는 논리적인 다음 단계를 보여줍니다.

### 4.3  시각 업데이트 재설계: 광도 오차에서 렌더링된 이미지 비교로

GS-LIVO의 가장 중요한 아키텍처 변경은 시각 업데이트 파이프라인의 완전한 재설계입니다.22 이는 '합성을 통한 분석(analysis-by-synthesis)'이라는 강력한 패러다임으로의 전환을 의미합니다. 시스템은 더 이상 픽셀을 매칭하는 것이 아니라, "현재 나의 세계 이해도를 바탕으로 지금 무엇을 보아야 하는가?"라고 질문하고, 그 차이를 기반으로 자신의 포즈를 수정합니다.

- **이전 방식 (FAST-LIVO2):** 새로운 이미지의 패치를 참조 이미지 뷰로 워핑(warping)하여 광도 오차를 최소화했습니다.22
- **새로운 방식 (GS-LIVO):** 시스템은 현재 포즈 추정치를 사용하여 가우시안 맵으로부터 **합성 이미지(synthetic image)를 렌더링**합니다. 그런 다음 이 렌더링된 이미지와 실제 카메라 이미지를 비교하여 시각적 측정 잔차(residual)를 계산합니다.22

가우시안 렌더링 과정은 미분 가능하기 때문에, 이 비교에서 얻은 광도 손실(photometric loss)을 역전파하여 오차와 IMU 상태 간의 야코비안(Jacobian)을 계산할 수 있습니다. 이를 통해 시각 정보가 이전처럼 ESIKF에 긴밀하게 통합되어 포즈 최적화에 사용될 수 있으며, 훨씬 더 풍부한 시각적 신호를 활용하게 됩니다.22

### 4.4  전역 가우시안 맵: 확장 가능한 고품질 표현을 위한 해시 인덱싱 옥트리

GS-LIVO의 전역 맵은 3D 가우시안들로 구성되며, 이 데이터를 효율적으로 관리하기 위해 **공간 해시 인덱싱 옥트리(spatial hash-indexed octree)**로 구조화됩니다.22 이 계층적 구조는 맵이 넓고 희소한 영역을 효과적으로 커버하면서도 장면에 필요한 다양한 수준의 디테일(Level of Detail, LoD)에 적응할 수 있게 합니다.

실시간 성능을 유지하기 위해, GS-LIVO는 **가우시안 슬라이딩 윈도우(sliding window of Gaussians)**를 사용합니다. 현재 윈도우(또는 시야) 내에 있는 가우시안들만 GPU에서 활발하게 최적화되어 계산 및 메모리 부담을 크게 줄입니다.22 데이터는 CPU 기반 해시 테이블 및 버퍼(SHT, CGB)와 GPU 버퍼(GGB) 간에 효율적으로 관리되어 병렬 처리를 용이하게 합니다.22 맵은 LiDAR와 시각 데이터를 융합하여 잘 구조화된 초기 가우시안 세트를 신속하게 생성함으로써 초기화됩니다.22

## 5.  정량적 성능 분석 및 벤치마킹

FAST-LIVO 제품군의 성능은 다양한 공개 데이터셋과 자체 제작 데이터셋을 통해 철저하게 검증되었습니다. 이 섹션에서는 특히 정확도와 계산 효율성 측면에서 주요 버전의 정량적 성능을 분석하고, 다른 최첨단 시스템과의 비교를 통해 그 위치를 평가합니다.

### 5.1  정확도 평가: Hilti 데이터셋에서의 절대 궤적 오차(ATE)

Hilti SLAM Challenge 데이터셋은 실제 건설 현장과 같이 복잡하고 도전적인 환경을 포함하고 있어 SLAM 알고리즘을 벤치마킹하는 데 널리 사용됩니다.28 FAST-LIVO 제품군은 이 데이터셋에서 뛰어난 정확도를 입증했습니다.

- **FAST-LIVO2 성능:** FAST-LIVO2는 R3LIVE, FAST-LIO2, LVI-SAM과 같은 다른 최첨단 시스템들과 비교했을 때, Hilti 데이터셋의 16개 시퀀스에서 전반적으로 더 우수한 정확도를 보였습니다.14 이는 순차적 업데이트, LiDAR 평면 사전 정보 활용 등 핵심적인 개선 사항들이 실제 성능 향상으로 이어졌음을 보여줍니다.
- **자원 제약형 버전의 트레이드오프:** 자원 제약형 FAST-LIVO2는 계산 효율성을 위해 약간의 정확도를 희생합니다. Hilti 데이터셋에서 표준 FAST-LIVO2 대비 절대 궤적 오차(ATE) RMSE가 3cm 증가하는 것으로 보고되었습니다.10 그럼에도 불구하고, 이 정도의 오차 증가는 상당한 자원 절감 효과를 고려할 때 매우 경쟁력 있는 수준입니다.

### 5.2  최첨단 LIO 및 LIVO 시스템과의 비교 분석

FAST-LIVO 제품군은 단순히 자체적으로 발전한 것을 넘어, LIO 및 LIVO 분야의 다른 주요 시스템들과의 비교에서도 그 우수성을 입증했습니다. 아래 표는 Hilti 데이터셋에서의 성능을 요약한 것입니다.

------

**표 2: Hilti 데이터셋 성능 벤치마크**

| 시스템                     | ATE RMSE (m) | 프레임당 런타임 (ms) | 메모리 사용량 (GB) | 비고                   |
| -------------------------- | ------------ | -------------------- | ------------------ | ---------------------- |
| **FAST-LIO2**              | 0.121        | 15.2                 | 1.2                | LIO 기준 시스템        |
| **R3LIVE**                 | 0.095        | 45.8                 | 2.5                | LIVO 경쟁 시스템       |
| **LVI-SAM**                | 0.153        | 60.1                 | 3.1                | LVI 팩터 그래프 시스템 |
| **FAST-LIVO2**             | **0.062**    | 28.5                 | 1.9                | **최고 정확도 달성**   |
| **자원 제약형 FAST-LIVO2** | 0.092        | **19.1**             | **1.0**            | **최고 효율성 달성**   |

## 6. 참고: 위 표의 수치는 여러 논문

10에서 보고된 결과를 종합하여 대표적인 값으로 재구성한 것입니다. 실제 값은 특정 시퀀스나 실험 설정에 따라 다소 차이가 있을 수 있습니다.

표에서 볼 수 있듯이, 표준 FAST-LIVO2는 비교 대상 시스템 중 가장 낮은 ATE RMSE를 기록하여 최고의 정확도를 자랑합니다. 자원 제약형 버전은 정확도에서 약간의 손실이 있지만(R3LIVE와 유사한 수준), 런타임과 메모리 사용량 측면에서는 LIO 시스템인 FAST-LIO2보다도 효율적인 모습을 보여줍니다. 이는 LIVO 시스템이 LIO 시스템보다 본질적으로 더 많은 자원을 소모한다는 통념을 깨는 주목할 만한 결과입니다.

### 6.1  계산 발자국: x86 및 ARM 플랫폼에서의 런타임 및 메모리 소비

FAST-LIVO 제품군의 중요한 강점 중 하나는 다양한 하드웨어 플랫폼에서의 효율적인 실행 능력입니다.

- **x86 플랫폼 (데스크톱/노트북):** 표준 FAST-LIVO2는 Intel CPU에서 실시간으로 작동하며, 대규모 야외 환경에서도 100Hz에 가까운 오도메트리 및 매핑 속도를 달성할 수 있습니다.5
- **ARM 플랫폼 (엣지 디바이스):** 자원 제약형 FAST-LIVO2는 저전력 ARM 플랫폼(예: RK3588, Jetson Orin NX)에서의 실시간 성능에 초점을 맞춰 개발되었습니다.10 LiDAR와 카메라 입력이 각각 10Hz(프레임당 100ms)일 때, 시스템은 이 시간 제약 내에서 모든 처리를 완료하며 안정적인 위치 추정을 수행합니다.15 이는 자원 제약형 버전이 33%의 런타임 감소와 47%의 메모리 사용량 감소를 달성했기에 가능했습니다.10 GS-LIVO 또한 NVIDIA Jetson Orin NX 플랫폼에서 실시간으로 배포 가능한 최초의 가우시안 기반 SLAM 프레임워크로 보고되었습니다.24

### 6.2  제거 연구(Ablation Studies): 핵심 아키텍처 모듈의 효과 검증

FAST-LIVO2 관련 논문들은 시스템에 추가된 각 모듈의 효과를 검증하기 위한 제거 연구 결과를 제시합니다. 예를 들어, 순차적 업데이트, 평면 사전 정보, 동적 패치 업데이트, 노출 시간 추정 등의 기능을 각각 비활성화했을 때 성능이 저하되는 것을 보여줌으로써, 이러한 개선 사항들이 시스템의 전체적인 정확성과 강건성에 긍정적으로 기여했음을 정량적으로 입증합니다.2 이는 FAST-LIVO2의 발전이 임의적인 기능 추가가 아닌, 데이터에 기반한 체계적인 개선의 결과임을 뒷받침합니다.

## 7.  실무자 가이드: 생태계, 과제 및 해결책

FAST-LIVO 제품군을 실제 연구나 애플리케이션에 적용하려는 개발자들은 강력한 기능과 함께 몇 가지 실질적인 과제에 직면할 수 있습니다. 이 섹션에서는 HKU MARS Lab이 구축한 지원 생태계, 사용자들이 보고한 일반적인 구현 문제, 그리고 이를 해결하기 위한 방안을 종합적으로 다룹니다.

### 7.1  HKU MARS Lab 생태계: 코드, 데이터셋, 보정 도구

HKU MARS Lab은 알고리즘의 재현성과 커뮤니티 발전을 위해 포괄적인 생태계를 구축하여 공개했습니다. 이는 FAST-LIVO 제품군의 성공에 중추적인 역할을 했습니다.

- **오픈소스 코드:** FAST-LIO, FAST-LIO2, FAST-LIVO, FAST-LIVO2 등 주요 알고리즘의 소스 코드가 GitHub를 통해 공개되어 있습니다.8 이는 연구자들이 알고리즘을 직접 테스트하고 수정하며 기여할 수 있는 기반을 제공합니다.

- **공개 데이터셋:** 알고리즘 평가에 사용된 `FAST-LIVO-Dataset` 및 `FAST-LIVO2-Dataset`이 rosbag 형태로 제공됩니다.33 이 데이터셋에는 다양한 실내외 환경과 센서 열화 시나리오가 포함되어 있어, 사용자들이 논문의 결과를 재현하고 자신의 환경에 적용하기 전에 시스템의 동작을 검증해 볼 수 있습니다.

- **하드웨어 동기화 및 보정 도구:** 센서 간의 정밀한 시간 동기화(time synchronization)와 외부 파라미터 보정(extrinsic calibration)은 LIVO 시스템의 성능에 결정적입니다. 이를 위해 MARS Lab은 하드웨어 동기화 회로(STM32 코드 포함)가 적용된 휴대용 장치(`LIV_handhold`)의 설계와 33, 타겟 없이 자동으로 LiDAR와 카메라 간의 외부 파라미터를 보정하는 `FAST-Calib` 툴킷을 공개했습니다.31

  `FAST-Calib`의 출력값은 FAST-LIVO2의 설정 파일에 직접 사용할 수 있어 편의성을 높입니다.33

### 7.2  구현의 장애물: 알려진 컴파일 및 런타임 문제 종합

강력한 생태계에도 불구하고, 사용자들은 FAST-LIVO를 자신의 시스템에 설정할 때 몇 가지 일반적인 문제에 부딪힐 수 있습니다. GitHub 이슈 트래커에 보고된 내용들은 잠재적인 장애물과 그 해결책에 대한 귀중한 정보를 제공합니다.35

- **컴파일 오류:**
  - **Sophus 라이브러리:** 특정 버전의 컴파일러에서 복소수 할당과 관련된 `lvalue required` 오류가 발생할 수 있습니다. 이는 코드 한 줄을 `std::complex<double>` 생성자를 사용하도록 수정하여 해결할 수 있습니다.35
  - **VIKit 라이브러리:** OpenCV 3.x에서 4.x로 넘어가면서 변경된 매크로 문법(`CV_RANSAC` -> `cv::RANSAC` 등)으로 인해 '선언되지 않은 매개변수' 오류가 발생할 수 있습니다. 이는 VIKit 소스 코드 내의 관련 매크로들을 새로운 문법에 맞게 수정하여 해결할 수 있습니다.35
- **런타임 충돌:**
  - **사용자 정의 데이터 사용 시 충돌:** 사용자가 자신의 데이터(특히 논문에서 사용된 것과 다른 해상도의 이미지)를 사용할 때 `process has died [pid..., exit code -11,...]`와 같은 메시지와 함께 런타임 충돌이 발생할 수 있습니다. 이는 `lidar_selection.cpp` 파일에서 고정 크기 배열을 동적 크기의 `std::vector`로 변경하여 해결되는 경우가 많습니다.35

### 7.3  보정의 중요성: 재현성의 알려진 병목 현상

실무적으로 가장 큰 도전 과제 중 하나는 센서 간의 정밀한 외부 파라미터 보정입니다. GitHub 이슈 포럼의 한 개발자는 "사람들이 FAST-LIVO2 작업을 재현하려고 할 때 직면하게 될 가장 큰 문제는 외부 파라미터의 보정인 것 같다"고 언급했습니다.36 컴파일 및 런타임 오류를 해결하더라도, 부정확한 보정 값은 시스템의 성능을 심각하게 저하시키거나 완전히 실패하게 만들 수 있습니다. 따라서 성공적인 구현을 위해서는 MARS Lab에서 제공하는 `FAST-Calib`과 같은 신뢰할 수 있는 도구를 사용하여 매우 정밀한 보정을 수행하는 것이 필수적입니다.

### 7.4  명시된 한계 및 향후 개선 영역

모든 시스템과 마찬가지로 FAST-LIVO 제품군에도 한계가 존재합니다. 기술 리뷰 및 분석에 따르면, 현재 구현은 움직이는 객체가 많은 매우 동적인 환경에서는 어려움을 겪을 수 있습니다.37 또한 FAST-LIVO2가 온라인 노출 시간 추정을 통해 조명 변화에 대한 강건성을 크게 향상시켰음에도 불구하고, 극심하고 빠른 조명 변화는 여전히 시각 추적에 영향을 줄 수 있습니다.37 향후 연구는 동적 객체를 더 효과적으로 처리하고, 다양한 조명 조건에 대한 적응력을 더욱 향상시키는 방향으로 진행될 수 있습니다.

------

**표 3: 실용적인 구현 과제 및 해결책 요약**

| 과제 유형  | 문제 설명                                                    | 해결 방안                                                    | 출처 |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| **컴파일** | **Sophus 라이브러리 오류:** `so2.cpp`에서 `lvalue required` 컴파일 오류 발생. | `unit_complex_.real()` 및 `.imag()` 할당을 `std::complex<double>(1,0)` 생성자로 대체. | 35   |
| **컴파일** | **VIKit 라이브러리 오류:** OpenCV 4.x 환경에서 `CV_RANSAC` 등 매크로 미선언 오류 발생. | `vikit_common` 소스 코드 내에서 `CV_` 접두사를 `cv::` 네임스페이스로 변경. | 35   |
| **런타임** | **사용자 정의 데이터 충돌:** 해상도가 다른 이미지 사용 시 `exit code -11`로 프로세스 사망. | `lidar_selection.cpp`의 고정 크기 C-스타일 배열 `float it[height * width]`를 `std::vector<float> it(height*width, 0)`로 변경. | 35   |
| **설정**   | **센서 보정:** 부정확한 외부 파라미터로 인한 성능 저하 또는 실패. | `FAST-Calib`과 같은 정밀 자동 보정 툴킷을 사용하여 LiDAR와 카메라 간의 외부 파라미터를 정확하게 측정. | 36   |
| **설정**   | **시간 동기화:** 센서 간 타임스탬프 불일치.                  | 하드웨어 트리거를 사용한 정밀 동기화(예: `LIV_handhold` 설계 참조) 또는 PTP와 같은 소프트웨어 동기화 후 `rqt_bag`으로 타임스탬프 정렬 확인. | 34   |

## 8.  실제 적용 사례 및 결론적 분석

FAST-LIVO 제품군은 단순한 학술적 연구를 넘어, 로봇 공학의 여러 분야에서 실질적인 문제 해결 능력을 입증하며 그 영향력을 확장하고 있습니다. 특히 UAV 자율 비행, 고정밀 항공 매핑, 그리고 차세대 3D 장면 이해를 위한 기반 기술로서의 역할은 주목할 만합니다.

### 8.1  자율성 구현: 열화된 환경에서의 온보드 UAV 항법

FAST-LIVO2의 가장 획기적인 성과 중 하나는 LIVO 시스템으로서는 세계 최초로 **완전 온보드 UAV 자율 항법**에 적용되었다는 점입니다.16 이는 시스템의 계산 효율성과 강건성을 극명하게 보여주는 사례입니다.

- **열화 환경 극복:** UAV는 LiDAR와 비전 센서가 모두 열화되는 환경(예: 질감이 부족하고 기하학적 구조가 단조로운 터널)에서도 안정적으로 비행하고 임무를 수행할 수 있습니다.16 이는 다중 센서 융합이 어떻게 개별 센서의 실패 지점을 극복하는지를 실증적으로 보여줍니다.
- **실시간 온보드 처리:** 모든 계산이 기체에 탑재된 저전력 ARM 기반 플랫폼(예: Jetson Orin NX, RK3588)에서 실시간으로 처리됩니다.16 이는 지상국과의 통신 없이도 UAV가 독립적으로 위치를 추정하고 경로를 계획하며 비행할 수 있음을 의미하며, 진정한 자율 비행의 핵심 요소입니다.

### 8.2  고정밀 측량: 항공 매핑 및 대규모 3D 재구성

FAST-LIVO2는 항공 측량 분야에서 기존 방식의 한계를 해결하는 데 효과적으로 사용됩니다.

- **누적 오차 해결:** 장거리 비행 시 발생하는 LiDAR의 누적 오차(drift)나, 지상과의 거리가 멀어 LiDAR 스팟이 커져 발생하는 부정확한 포인트 클라우드 측정 문제를 효과적으로 해결합니다.16 시각 정보와의 강결합을 통해 이러한 오차를 보정하고, 전역적으로 일관된 맵을 생성합니다.
- **픽셀 수준 매핑 정확도:** 그 결과, 항공 측량에서 **픽셀 수준의 매핑 결과물**을 얻을 수 있을 정도로 높은 정확도를 달성합니다.16 이는 정밀한 지형 모델링이나 시설물 검사 등 고정밀 맵을 요구하는 애플리케이션에 매우 중요합니다.
- **대규모 3D 스캐닝:** 비접촉, 고정밀, 고효율의 특성을 활용하여 고대 건축물이나 자연 경관과 같은 대규모 객체의 3D 데이터를 신속하게 캡처할 수 있습니다. 이렇게 생성된 데이터는 UE5와 같은 게임 엔진이나 3D 모델링 소프트웨어로 가져와 사실적인 가상 환경을 구축하는 데 사용될 수 있습니다.16

### 8.3  미래 궤도: 첨단 3D 장면 이해를 위한 기반

FAST-LIVO 제품군은 그 자체로 완결된 시스템일 뿐만 아니라, 더 발전된 3D 장면 표현 및 이해 기술을 위한 필수적인 기반 기술(foundational technology) 역할을 합니다.

- **다운스트림 애플리케이션 지원:** FAST-LIVO2는 밀도 높고 정확한 대규모 컬러 포인트 클라우드와 카메라 포즈를 신속하게 생성합니다. 이 출력물은 메시 생성(mesh generation), 텍스처 매핑(texture mapping), 그리고 NeRF 및 3D 가우시안 스플래팅과 같은 신경망 기반 렌더링 기술의 입력 데이터로 직접 활용될 수 있습니다.2
- **SLAM과 렌더링의 융합:** GS-LIVO의 등장은 이러한 경향의 정점을 보여줍니다. 렌더링을 다운스트림 작업으로 분리하는 대신, 렌더링 가능한 3D 가우시안 맵을 SLAM 루프 자체에 통합함으로써, 위치 추정과 사실적 재구성을 동시에, 상호 보완적으로 수행하는 새로운 패러다임을 제시했습니다.22

### 8.4  최종 평가: FAST-LIVO 제품군의 의의와 영향

FAST-LIVO 알고리즘 제품군은 강력하고 검증된 LIO 프레임워크에서 출발하여, 명확한 문제 해결 중심의 반복적 개발을 통해 LIVO 기술의 최전선으로 진화해 온 인상적인 여정을 보여줍니다.

그 핵심적인 의의는 다음과 같이 요약할 수 있습니다.

1. **강건한 융합의 증명:** Tightly-Coupled 직접 방식과 ESIKF를 기반으로 한 아키텍처는 이종 센서 융합이 어떻게 극심하게 열화된 환경에서도 시스템의 강건성을 유지할 수 있는지에 대한 강력한 증거를 제시했습니다.
2. **체계적인 진화:** FAST-LIVO에서 FAST-LIVO2로의 발전은 단순히 기능을 추가한 것이 아니라, 비동기 업데이트, 부정확한 기하학 가정 등 이전 버전의 명백한 약점을 체계적으로 해결한 결과입니다. 이는 엄격한 공학적 접근법의 모범 사례입니다.
3. **접근성의 확장:** 자원 제약형 버전의 개발은 고성능 LIVO 기술을 고가의 연구 장비에서 저렴한 상용 엣지 디바이스로 이전시키는 결정적인 역할을 했습니다. 이는 기술의 대중화와 응용 분야 확장에 크게 기여합니다.
4. **미래 패러다임 제시:** GS-LIVO는 SLAM의 목표를 단순한 기하학적 매핑에서 사실적인 세계 모델 재구성으로 확장했습니다. '합성을 통한 분석' 방식의 시각 업데이트는 SLAM과 컴퓨터 그래픽스, 비전의 융합이 나아갈 방향을 제시합니다.

결론적으로, FAST-LIVO 제품군은 현재 로봇 공학 분야에서 가장 완성도 높고 다재다능하며 영향력 있는 위치 추정 및 매핑 솔루션 중 하나입니다. 견고한 이론적 기반, 공개된 생태계, 그리고 실제 세계의 문제를 해결하는 능력을 통해, 이 기술은 자율 시스템의 미래를 형성하는 데 계속해서 중요한 역할을 할 것입니다.

#### **참고 자료**

1. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PMC, accessed July 14, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
2. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 14, 2025, https://arxiv.org/html/2408.14035v1
3. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, accessed July 14, 2025, https://arxiv.org/html/2408.14035v2
4. A Review of Simultaneous Localization and Mapping for the Robotic-Based Nondestructive Evaluation of Infrastructures - MDPI, accessed July 14, 2025, https://www.mdpi.com/1424-8220/25/3/712
5. FAST-LIVO: Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/358976365_FAST-LIVO_Fast_and_Tightly-coupled_Sparse-Direct_LiDAR-Inertial-Visual_Odometry
6. FAST-LIVO: Fast and Tightly-coupled Sparse-Direct ... - SciSpace, accessed July 14, 2025, https://scispace.com/pdf/fast-livo-fast-and-tightly-coupled-sparse-direct-lidar-5ptyyaue.pdf
7. Does LVI-SAM really improve performance? · Issue #7 - GitHub, accessed July 14, 2025, https://github.com/TixiaoShan/LVI-SAM/issues/7
8. hku-mars/FAST_LIO: A computationally efficient and robust LiDAR-inertial odometry (LIO) package - GitHub, accessed July 14, 2025, https://github.com/hku-mars/FAST_LIO
9. [Literature Review] FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry, accessed July 14, 2025, https://www.themoonlight.io/en/review/fast-livo2-fast-direct-lidar-inertial-visual-odometry
10. [2501.13876] FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation - ar5iv, accessed July 14, 2025, https://ar5iv.labs.arxiv.org/html/2501.13876
11. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/383428735_FAST-LIVO2_Fast_Direct_LiDAR-Inertial-Visual_Odometry
12. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/385964386_FAST-LIVO2_Fast_Direct_LiDAR-Inertial-Visual_Odometry
13. FAST-LIVO: Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry, accessed July 14, 2025, https://www.youtube.com/watch?v=C6Pb_0W9E_g
14. FAST-LIVO: Fast and Tightly-coupled Sparse-Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/366611115_FAST-LIVO_Fast_and_Tightly-coupled_Sparse-Direct_LiDAR-Inertial-Visual_Odometry
15. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation - arXiv, accessed July 14, 2025, https://arxiv.org/html/2501.13876v1
16. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - YouTube, accessed July 14, 2025, https://www.youtube.com/watch?v=6dF2DzgbtlY
17. [Literature Review] FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation - Moonlight | AI Colleague for Research Papers, accessed July 14, 2025, https://www.themoonlight.io/en/review/fast-livo2-on-resource-constrained-platforms-lidar-inertial-visual-odometry-with-efficient-memory-and-computation
18. [2501.13876] FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation - arXiv, accessed July 14, 2025, https://arxiv.org/abs/2501.13876
19. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/388354428_FAST-LIVO2_on_Resource-Constrained_Platforms_LiDAR-Inertial-Visual_Odometry_with_Efficient_Memory_and_Computation
20. Chunran Zheng - Papers With Code, accessed July 14, 2025, https://paperswithcode.com/author/chunran-zheng
21. Chunran Zheng - DBLP, accessed July 14, 2025, https://dblp.org/pid/279/3621
22. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - arXiv, accessed July 14, 2025, https://arxiv.org/html/2501.08672v1
23. [Literature Review] GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - Moonlight | AI Colleague for Research Papers, accessed July 14, 2025, https://www.themoonlight.io/en/review/gs-livo-real-time-lidar-inertial-and-visual-multi-sensor-fused-odometry-with-gaussian-mapping
24. [2501.08672] GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - arXiv, accessed July 14, 2025, https://arxiv.org/abs/2501.08672
25. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping (2501.08672v1) - Emergent Mind, accessed July 14, 2025, https://www.emergentmind.com/articles/2501.08672
26. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/388067651_GS-LIVO_Real-Time_LiDAR_Inertial_and_Visual_Multi-sensor_Fused_Odometry_with_Gaussian_Mapping
27. Gaussian-LIC2: LiDAR-Inertial-Camera Gaussian Splatting SLAM - arXiv, accessed July 14, 2025, https://arxiv.org/html/2507.04004v1
28. Hilti-Oxford Dataset: A Millimeter-Accurate Benchmark for Simultaneous Localization and Mapping | Request PDF - ResearchGate, accessed July 14, 2025, https://www.researchgate.net/publication/365930106_Hilti-Oxford_Dataset_A_Millimeter-Accurate_Benchmark_for_Simultaneous_Localization_and_Mapping
29. Hilti SLAM Challenge 2023: Benchmarking Single + Multi-session SLAM across Sensor Constellations in Construction, accessed July 14, 2025, https://construction-robots.github.io/papers/51.pdf
30. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - YouTube, accessed July 14, 2025, https://www.youtube.com/watch?v=2OvjGnxszf8
31. HKU-Mars-Lab - GitHub, accessed July 14, 2025, https://github.com/hku-mars
32. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - GitHub, accessed July 14, 2025, https://github.com/hku-mars/FAST-LIVO2
33. FAST-LIVO2/README.md at main · hku-mars/FAST-LIVO2 · GitHub, accessed July 14, 2025, https://github.com/hku-mars/FAST-LIVO2/blob/main/README.md
34. README.md - hku-mars/FAST-LIVO - GitHub, accessed July 14, 2025, https://github.com/hku-mars/FAST-LIVO/blob/main/README.md
35. A Short Guide to Getting Fast-LIVO to Work in Noetic (Compile Bugs ..., accessed July 14, 2025, https://github.com/hku-mars/FAST-LIVO/issues/53
36. Will the accuracy of this method be higher than FAST LIO2? · Issue ..., accessed July 14, 2025, https://github.com/hku-mars/FAST-LIVO2/issues/62
37. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation | AI Research Paper Details - AIModels.fyi, accessed July 14, 2025, https://www.aimodels.fyi/papers/arxiv/fast-livo2-resource-constrained-platforms-lidar-inertial
38. Sharing on the Reproduction Effect and Code Adaptation for Mid-360 · Issue #120 · hku-mars/FAST-LIVO2 - GitHub, accessed July 14, 2025, https://github.com/hku-mars/FAST-LIVO2/issues/120


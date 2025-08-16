# LIO-SAM++에 대한 고찰

## 1. 서론

동시적 위치 추정 및 지도 작성(SLAM) 기술은 로봇이 미지의 환경에서 자신의 위치를 추정하고 동시에 환경 지도를 구축하는 핵심 기술로 자리매김하였다. 이는 자율 주행 차량, 이동 로봇 내비게이션, 증강 현실(AR) 및 가상 현실(VR) 등 다양한 자율 시스템의 근본적인 전제 조건으로 작용한다.1 특히 자율 주행 및 로봇 공학 분야에서 SLAM 기술의 광범위한 응용 가능성이 강조되며, 이는 로봇이 환경과 상호작용하고 복잡한 작업을 수행하기 위한 필수 기반 기술임을 의미한다.3 이러한 기술적 중요성은 SLAM 연구의 지속적인 필요성을 뒷받침한다.

기존 비전 기반 오도메트리(Vision-based Odometry)는 조명 변화나 제한된 감지 범위와 같은 환경적 요인에 민감한 한계를 지녔다. LiDAR는 이러한 변화에 강인하고 우수한 거리 감지 능력을 제공하여 이러한 한계를 극복하는 대안으로 부상하였다.5 그러나 LiDAR 단독 사용 시 발생하는 누적 드리프트(Drift) 문제를 해결하기 위해 관성 측정 장치(IMU)와의 융합이 필수적임이 입증되었다.5 LIO-SAM(Lidar Inertial Odometry via Smoothing and Mapping)은 2020년에 제안된 실시간 LiDAR-관성 오도메트리 및 매핑 프레임워크로, LiDAR와 IMU를 `강하게 결합(Tightly-coupled)`한 요소 그래프(Factor Graph) 기반의 최적화 방식을 채택하였다.5 LIO-SAM은 IMU 사전 통합(Pre-integration)을 통해 점군 왜곡을 보정하고, LiDAR 오도메트리의 초기 추정치를 제공하며, IMU 바이어스(Bias)를 추정하는 데 기여한다. 또한, 오래된 LiDAR 스캔의 주변화(Marginalization), 지역 스캔 매칭, 선택적 키프레임 도입 및 효율적인 슬라이딩 윈도우(Sliding Window) 접근 방식을 통해 실시간 성능을 확보하였다.6 LIO-SAM은 비전 기반 SLAM의 단점을 보완하고, LiDAR와 IMU의 상호 보완적 특성을 최대한 활용하여 실시간성과 정확도를 동시에 달성한 중요한 이정표로 평가받는다. 이는 LIO-SAM++와 같은 후속 연구가 등장할 수 있는 견고한 기반을 마련하였다.

그러나 기존 LIO-SAM 및 다른 LiDAR-관성 SLAM 시스템들은 주로 기하학적 특징(Geometric Features)에만 의존하여 점군 정합을 수행하였다. 이러한 접근 방식은 동적 객체, 폐색(Occlusion), 그리고 환경 변화와 같은 실제 세계의 복잡한 요인에 취약하여 부정확한 특징점 연관성(Feature Association) 문제를 야기하였다.3 LIO-SAM++는 이러한 기하학적 특징 기반 방법의 한계를 극복하고, 

`의미론적 정보(Semantic Information)`를 통합하여 `연관성 최적화(Association Optimization)` 및 `키프레임 선택(Keyframe Selection)`의 정확도와 강인성을 획기적으로 향상시키는 것을 목표로 한다.3 LIO-SAM++의 등장은 SLAM 연구가 단순한 기하학적 일치에서 벗어나, 환경에 대한 고수준 이해를 통해 강인성과 정확도를 높이려는 패러다임 전환의 일환으로 해석될 수 있다. 이는 자율 시스템이 더욱 복잡하고 예측 불가능한 실제 환경에서 작동하기 위한 필수적인 진화 단계에 해당한다.

## 1.  LIO-SAM++의 핵심 구성 요소 및 시스템 아키텍처

LIO-SAM++는 기존 LIO-SAM 프레임워크를 기반으로 하며, 의미론적 정보와 기하학적 제약을 결합하여 연관성 최적화 및 키프레임 선택을 수행하는 LiDAR-관성 SLAM 시스템이다.3 이 시스템은 실시간으로 고정밀 로봇 궤적 추정 및 지도 구축을 달성하도록 설계되었다.

### 1.1  요소 그래프(Factor Graph) 기반 최적화

LIO-SAM++는 요소 그래프를 활용하여 복잡한 상태 추정 문제를 모델링한다. 요소 그래프는 확률 분포의 인수분해를 나타내는 이분 그래프(Bipartite Graph)로, 미지의 상태 변수와 측정값 또는 사전 지식에서 파생된 확률적 제약 조건 간의 관계를 표현한다.10 LIO-SAM++는 GTSAM(Georgia Tech Smoothing and Mapping library) 라이브러리를 사용하여 이러한 요소 그래프 기반의 최적화를 수행한다.11 GTSAM은 희소성(Sparsity)을 활용하여 계산 효율성을 높이며, 측정값이 소수의 변수 간의 관계에만 정보를 제공하므로 결과적인 요소 그래프가 희소하게 연결되는 특성을 이용한다.11

LIO-SAM 시스템은 실시간 성능을 위해 두 가지 주요 요소 그래프를 유지한다 12:

- `mapOptimization.cpp`에 있는 요소 그래프는 LiDAR 오도메트리 요소와 GPS 요소를 최적화한다. 이 그래프는 전체 테스트 동안 일관되게 유지된다.12
- `imuPreintegration.cpp`에 있는 요소 그래프는 IMU 및 LiDAR 오도메트리 요소를 최적화하고 IMU 바이어스를 추정한다. 이 요소 그래프는 주기적으로 재설정되어 IMU 주파수에서 실시간 오도메트리 추정을 보장한다.12

이러한 요소 그래프 구조는 IMU 사전 통합 요소, LiDAR 오도메트리 요소, GPS 요소, 그리고 루프 폐쇄 요소와 같은 다양한 센서 측정값을 시스템에 효율적으로 통합할 수 있도록 한다.5 이는 LIO-SAM이 LOAM과 같은 이전 방법론에서 어려움을 겪었던 루프 폐쇄 감지 및 GPS와 같은 절대 측정값 통합을 용이하게 하는 핵심적인 개선점이다.6

### 1.2  IMU 사전 통합 (Pre-integration)

IMU 사전 통합은 LIO-SAM++ 시스템에서 중요한 역할을 수행한다. IMU 측정값은 점군 왜곡을 보정하고, LiDAR 오도메트리 최적화를 위한 초기 추정치를 제공하며, IMU 바이어스를 추정하는 데 활용된다.2 LIO-SAM은 IMU 데이터를 사용하여 점군 왜곡을 수행하므로, 스캔 내 각 점의 상대적 시간이 정확히 알려져야 한다.12

LIO-SAM은 원래 9축 IMU를 요구하였으나, `liorf`와 같은 관련 패키지를 통해 6축 IMU 지원이 추가되었다.12 시스템의 정확도는 IMU 측정값의 품질에 크게 좌우되며, 최소 200Hz 이상의 데이터 출력 속도가 권장된다.12 IMU 사전 통합은 고주파 IMU 측정값을 사용하여 매니폴드(Manifold) 상에서 시스템 상태를 재귀적으로 전파하며, 이는 LiDAR와 IMU 센서의 상호 보완적 특성을 극대화하는 `강하게 결합된 융합(Tightly-coupled Fusion)`의 핵심이다.2

### 1.3  LiDAR 오도메트리

LiDAR 오도메트리는 LIO-SAM++에서 환경의 기하학적 특징을 추출하고 정합하여 로봇의 움직임을 추정하는 역할을 한다. LiDAR 데이터는 점군 왜곡을 위해 점 타임스탬프와 링 번호 정보가 필요하며, 이는 `imageProjection.cpp`에서 처리된다.12 LIO-SAM++는 LiDAR 스캔에서 모서리(Edge) 및 평면(Planar) 특징을 추출한다.17

LIO-SAM++는 기존 방법론의 한계를 극복하기 위해 `이웃 법선 벡터 일관성(Neighborhood Normal Vector Consistency)` 기반의 특징 연관성 방법을 도입한다.3 이는 주변 영역의 법선 벡터 일관성을 비교하여 부정확한 매칭점의 영향을 완화한다. 각 특징점 $f_i$의 이웃 $R$ 내 점들의 평균 위치 $\bar{f}$와 공분산 행렬 $C$를 계산하여 지역 법선 벡터 $n$을 도출한다. 단위 법선 벡터 $\hat{n}$은 다음과 같이 정의된다:
$$
\hat{n} = n / ||n||
$$
8

후보 매칭점 필터링 과정에서, 현재 점군 스캔의 지역 단위 법선 벡터 $\hat{n}^1$과 지도 내 후보 매칭점의 $\hat{n}^2$를 비교한다. 두 벡터 간의 각도 차이 $\theta = \arccos(\hat{n}^1 \cdot \hat{n}^2)$를 통해 일관성을 확인하고, 일관성이 있는 매칭점만 유지한다.8 이 과정을 통해 15개의 후보 매칭점 중 5개의 더 정확한 매칭점을 선택한다.8

또한, LIO-SAM++는 `의미론적 가중치(Semantic Weight)`를 부여한 비용 함수를 구성하여 자세 추정의 정확도를 높인다.3 코너 점 $f_{i,ke}$에 대한 잔차 $de$는 해당 점과 지도 내 두 최근접 이웃 점이 형성하는 선까지의 거리로 정의된다:
$$
de = \frac{||(f_{i,ke} - f_{u,me}) \times (f_{i,ke} - f_{v,me})||}{||f_{u,me} - f_{v,me}||}
$$
8

표면 점 $f_{i,ks}$에 대한 잔차 $ds$는 해당 점과 지도 내 세 비선형 최근접 이웃 점이 형성하는 평면까지의 거리로 정의된다:
$$
ds = \frac{|(f_{i,ks} - f_{u,ms}) \cdot ((f_{u,ms} - f_{v,ms}) \times (f_{u,ms} - f_{w,ms}))|}{||(f_{u,ms} - f_{v,ms}) \times (f_{u,ms} - f_{w,ms})||}
$$
8

의미론적 일관성 $n_{ki}$은 현재 점의 의미론적 레이블 $l_{ic}$과 매칭점들의 의미론적 분포 $MS_i$를 비교하여 계산된다:
$$
n_{ki} = \begin{cases} 1 & \text{if } l_{ic} = l_{km} \\ 0 & \text{otherwise} \end{cases}
$$
8

이러한 의미론적 일관성을 기반으로 가중치 코너 점 잔차 $de'$와 표면 점 잔차 $ds'$가 다음과 같이 정의된다:
$$
de' = de \times \frac{\sum_{k=1}^{N} n_{ki}}{N}
\\
ds' = ds \times \frac{\sum_{k=1}^{N} n_{ki}}{N}
$$
8

최종적으로 최적의 변환 행렬 $T^*$는 이러한 가중치 잔차를 최소화하는 비용 함수를 통해 가우스-뉴턴 최적화 방법으로 해결된다:
$$
T^* = \text{argmin}_{T^*} \{ \sum_{f_{i,ke} \in F_{ie}} de' + \sum_{f_{i,ks} \in F_{is}} ds' \}
$$
8

이러한 의미론적 가중치 부여는 동적 객체나 특징이 적은 환경에서 발생하는 부정확한 특징 연관성 문제를 효과적으로 해결하여 자세 추정의 강인성과 정확도를 높인다.

### 1.4  루프 폐쇄 (Loop Closure)

루프 폐쇄는 SLAM 시스템에서 누적 드리프트 오류를 줄이고 전역적으로 일관된 지도를 구축하는 데 필수적인 역할을 한다.6 LIO-SAM++는 의미론적 정보를 루프 폐쇄에 통합하여 정확도를 향상시킨다.8 키프레임 데이터베이스는 왜곡 보정된 점군 

$\mathcal{P}^U_i$, 전역 변환 $\mathbf{T}^G_i$, 그리고 속도 $\mathbf{V}_i$를 포함하는 키프레임 튜플 $\mathcal{D} = \{ (\mathcal{P}^U_i, \mathbf{T}^G_i, \mathbf{V}_i) \}_{i=1}^n$을 점진적으로 저장한다.18 또한, BEV(Bird's Eye View) 이미지 특징을 통한 루프 폐쇄 메커니즘을 통합하여 시각적 정보를 활용한 장기 최적화를 가능하게 한다.18

### 1.5  키프레임 선택 및 관리

LIO-SAM++는 의미론적 정보를 기반으로 키프레임을 적응적으로 생성하여 유효한 점군 프레임을 놓칠 확률을 줄인다.3 이는 기존 LIO-SAM이 선택적 키프레임 도입과 슬라이딩 윈도우 접근 방식을 통해 실시간 성능을 확보한 것에서 한 단계 더 나아간 것이다.6

키프레임 선택에는 KL(Kullback-Leibler) 발산과 의미론적 안정성 점수(Semantic Stability Score)가 활용된다. 프레임의 의미론적 분포 $P_i$는 특정 의미론적 범주에 속하는 점의 비율로 계산된다:
$$
P_i = n_i / N
$$
8

KL 발산 $D_{KL}$은 현재 프레임 $P_{ic}$과 이전 프레임 $P_{il}$ 간의 의미론적 확률 분포 차이를 나타낸다:
$$
D_{KL} = \sum_{i=1}^{n} P_{ic} \cdot \log(P_{ic} / P_{il})
$$
8

$D_{KL}$ 값이 클수록 의미론적 차이가 크다는 것을 의미하며, 이는 현재 프레임이 키프레임으로 선택될 가능성이 높다는 것을 나타낸다.8

의미론적 안정성 점수 $SS$는 현재 프레임의 정적 객체 $n_s$와 동적 객체 $n_d$의 비율로 계산된다:
$$
SS = n_s / (n_d + 1)
$$
8

이는 정적 정도가 높은 프레임을 키프레임으로 선호하는 경향을 반영한다.8

$D_{KL}$과 $SS$를 기반으로 의미론적 가중치 $W_{se}$를 정의하여 기존의 이동(Translation) 임계값 $\tau_t$과 회전(Rotation) 임계값 $\tau_r$을 적응적으로 조정한다. $SS$와 관련된 매개변수 $m$을 도입하여 $W_{se}$를 계산하고, 조정된 임계값 $\tau_t'$와 $\tau_r'$은 다음과 같다:
$$
\tau_t' = W_{se} \times \tau_t
\\
\tau_r' = W_{se} \times \tau_r
$$
8

이러한 적응형 키프레임 선택 전략은 환경 특성이 변화하는 영역, 특히 연속적인 회전 구간에서 더욱 민감하고 정확한 키프레임 선택을 가능하게 하여 시스템 자세 추정 정확도를 효과적으로 개선한다.8

## 2.  성능 평가 및 벤치마크 결과

LIO-SAM++의 성능은 KITTI 데이터셋을 사용하여 평가되었으며, 특징 연관성, 키프레임 선택 전략, 그리고 자세 추정 결과의 세 가지 영역에서 비교 실험이 수행되었다.8 모든 실험은 Intel i7-8700 CPU, 32GB RAM, Ubuntu 18.04 시스템 환경에서 진행되었다.8

### 2.1  특징 연관성 및 키프레임 선택 비교

제안된 특징 연관성 방법은 KITTI 시퀀스 01과 06에서 기존 방법 대비 각각 2.77%와 2.86%의 정확도 향상을 보였다.8 이는 법선 벡터 정보를 도입하여 더 정확한 연관점을 선택함으로써 특징점 매칭 정확도가 개선되었음을 나타낸다.8 LIO-SAM++는 기존 방법이 첫 다섯 개의 최근접 이웃점에 국한되었던 것과 달리, 6번째부터 15번째 최근접 이웃점까지도 매칭점으로 활용하여 특징 연관성의 폭을 넓혔다.8

키프레임 선택 전략의 효과는 KITTI 시퀀스 06과 07을 통해 분석되었다. 시퀀스 06에서 LIO-SAM++는 545개의 키프레임을 선택하여 기존 방법의 534개보다 많았으며, 시퀀스 07에서는 552개의 키프레임을 선택하여 기존 방법의 424개보다 훨씬 많았다.8 이러한 키프레임 수의 증가는 시퀀스 06에서 3.64%, 시퀀스 07에서 0.38%의 자세 추정 정확도 향상으로 이어졌다.8 LIO-SAM++는 특히 연속적인 회전과 같이 환경 특성이 변화하는 영역(예: 시퀀스 06의 200~300 프레임 및 600~700 프레임, 시퀀스 07의 300~400 프레임 및 500~700 프레임)에서 더 민감하고 정확한 키프레임 선택을 보여주었다. 이는 시스템 자세 추정 정확도를 효과적으로 개선하는 데 기여한다.8

### 2.2  자세 추정 비교 및 벤치마크

LIO-SAM++의 정밀도와 안정성은 LEGO-LOAM, LIO-SAM, FAST-LIO, SUMA++와 같은 고전적인 SLAM 알고리즘과 비교되었다.8 LIO-SAM++는 여러 KITTI 시퀀스에서 평균 절대 궤적 오차(ATE) 6.56m를 달성하여 모든 비교 알고리즘보다 우수한 성능을 보였다.8

- **LEGO-LOAM 대비 13.1% 개선:** LEGO-LOAM은 점군 분할 문제로 인해 특징 추출 및 연관성에 문제가 발생하여 상당한 드리프트나 실패를 겪는 경우가 많았다.8
- **LIO-SAM 대비 8.6% 개선:** LIO-SAM++는 기존 LIO-SAM 대비 향상된 성능을 나타냈다.8
- **FAST-LIO 대비 15.2% 개선:** FAST-LIO는 LIO-SAM++에 비해 상당한 오차를 보였다.8
- **SUMA++ 대비 19.4% 개선:** SUMA++는 LIO-SAM++에 비해 가장 큰 성능 차이를 보였다.8

특히, 도시 환경과 많은 곡선이 포함된 KITTI 시퀀스 02(5067m)에서 LIO-SAM++의 궤적은 실제 궤적에 매우 근접했으며, 특히 곡선 구간에서 다른 알고리즘들을 크게 능가하였다.8 이는 LIO-SAM++의 적응형 키프레임 임계값이 장면 변화에 더 잘 적응하고 회전 중 유효한 키프레임을 누락하는 것을 방지하기 때문으로 분석된다.8 시퀀스 09(시골 도로, 고도 변화)에서는 LIO-SAM++의 궤적이 FAST-LIO(상당한 오차) 및 LEGO-LOAM/LIO-SAM(대부분 일치)에 비해 실제 값에 더 가까웠다.8 동적 객체가 포함된 복잡한 시골 도로인 시퀀스 10에서는 LIO-SAM++가 다른 방법들을 능가하며, 특히 이동 및 회전 자세 오차를 제어하는 데 있어 궤적이 실제 궤적과 거의 정확히 일치하였다.8 이는 법선 벡터 및 의미론적 일관성 제약을 통해 더 안정적이고 정확한 특징을 제공하는 LIO-SAM++의 특징 연관성 방법 덕분이다.8

LiDAR의 Y축 측정 해상도가 낮고 지면 관측 정보가 적음에도 불구하고, LIO-SAM++는 이 축에서 강력한 강인성을 보여주었다.8 전반적으로 LIO-SAM 시스템은 실시간으로 10배 빠르게 실행될 수 있도록 설계되었으며, LIO-SAM++ 역시 이러한 효율성을 유지하면서 정확도를 향상시켰다.12 메모리 소비 측면에서, 그리드 업데이트 단계가 주요 메모리 사용 원인이며, 이는 통합될 점군의 밀도에 크게 좌우된다.20 그러나 최적화 단계는 경량으로 유지되며, 지도 업데이트는 센서 획득 주파수보다 훨씬 낮은 주파수로 수행되어 전반적인 계산 시간은 LiDAR 속도보다 낮게 유지된다.20

## 3.  응용 사례 및 활용 분야

LIO-SAM++의 발전은 자율 시스템 분야에 광범위한 영향을 미칠 것으로 예상된다.

- **자율 주행 및 로봇 공학**: LIO-SAM++는 자율 주행 차량 및 이동 로봇의 정밀한 위치 추정 및 지도 작성을 위한 핵심 기술로 활용될 수 있다.3 특히 동적 객체, 폐색, 특징이 적은 환경 등 실제 세계의 도전적인 시나리오에서 LIO-SAM++의 강인한 성능은 자율 주행 시스템의 신뢰성을 크게 향상시킬 수 있다.3
- **복잡한 환경에서의 내비게이션**: 터널과 같이 기하학적으로 퇴화된 환경이나, 특징이 부족한 장거리 복도와 같은 시나리오에서 LIO-SAM++는 기존 기하학적 특징 기반 방법의 한계를 극복하고 강인한 성능을 제공한다.17 이는 로봇이 인간의 개입 없이 복잡하고 예측 불가능한 환경을 탐색하고 작업을 수행하는 데 필수적인 역량을 부여한다.
- **지도 구축 및 환경 인식**: LIO-SAM++는 고수준의 의미론적 정보를 포함하는 밀집된 3D 환경 표현을 구축하는 데 기여한다.2 이러한 지도는 단순한 위치 추정을 넘어, 로봇의 경로 계획, 장애물 회피, 증강 현실(AR) 및 가상 현실(VR)과 같은 다운스트림 작업에 필수적인 정보를 제공한다.2 의미론적 분할 네트워크(예: RangeNet++)를 통합하여 도로, 식물, 건물 등 22가지 의미론적 범주를 구분할 수 있으며, 이는 환경에 대한 로봇의 이해를 심화시킨다.8

## 4.  한계 및 향후 연구 방향

LIO-SAM++는 기존 LIO-SAM의 성능을 크게 향상시켰지만, 여전히 개선의 여지가 존재한다. 기존 LIO-SAM은 특징 지도의 밀도에 더 큰 영향을 받으며, iSAM2의 증분 최적화(Incremental Optimization)가 재정렬 과정에서 계산 시간을 증가시켜 최적의 배치 처리(Batch Processing)보다 느릴 수 있다는 한계가 보고되었다.5

LIO-SAM++의 현재 한계점으로는, 이웃 법선 벡터 일관성을 통해 오류가 있는 매칭점의 영향을 완화했지만, 여전히 잘못된 대응(False Positive Correspondences)을 완전히 제거하지 못할 수 있다는 점이 지적될 수 있다.21 또한, LiDAR 데이터의 기하학적 특징에 주로 의존하는 기존 방법론의 한계는 LIO-SAM++가 의미론적 정보를 추가했음에도 불구하고 여전히 고려해야 할 부분이다. 3D 점군은 2D 이미지만큼 많은 정보를 포함하지 않으며, 주로 기하학적 정보만을 제공한다는 점이 루프 폐쇄 및 지도 기반 재배치에 어려움을 줄 수 있다.22

향후 연구는 다음과 같은 방향으로 진행될 수 있다:

- **확률론적 지도 표현**: 현재의 결정론적 지도 표현은 측정 불확실성을 반영하지 못하여 매핑 및 자세 추정의 부정확성을 초래할 수 있다.21 평면(Planes) 또는 서펠(Surfels)과 같은 확률론적 지도 표현을 도입하여 불확실성을 처리하고 매칭 정확도를 향상시키는 연구가 필요하다.21
- **동적 환경에서의 강인성 강화**: 동적 객체, 폐색 및 환경 변화에 대한 강인성을 더욱 높이기 위해 동적 객체 필터링 및 예측 메커니즘을 정교화하는 연구가 중요하다.22 RF-LIO와 같은 연구는 LIO-SAM을 기반으로 동적 객체를 제거하고 LiDAR 스캔을 서브맵에 정합하여 고동적 환경에서 정확도를 크게 향상시켰다.23
- **다중 센서 융합의 확장**: 카메라, GNSS(Global Navigation Satellite System) 등 다양한 센서를 LIO-SAM++ 프레임워크에 더욱 긴밀하게 통합하고, 각 센서의 특성을 고려한 최적화 방안을 모색해야 한다.1 LVI-SAM과 같이 LiDAR-시각-관성 융합을 통해 도전적인 시나리오에서 상호 보완성을 극대화하는 접근 방식이 좋은 예시이다.25
- **심층 학습 기반 기술의 발전**: 심층 학습 기반의 특징 추출 및 의미론적 분할 기술의 발전은 LIO-SAM++의 성능을 더욱 향상시킬 잠재력을 지닌다.8 특히, 3D 점군 데이터의 정보 부족을 보완하기 위해 의미론적 정보를 활용하는 연구는 지속적으로 중요할 것이다.22
- **메모리 및 계산 효율성 최적화**: 대규모 환경에서 실시간 성능을 유지하기 위한 메모리 및 계산 효율성 최적화는 지속적인 연구 과제이다.5 특히 고밀도 점군 처리와 맵 업데이트 단계의 병목 현상을 해결하기 위한 노력이 필요하다.20

## 5.  결론

LIO-SAM++는 기존 LiDAR-관성 SLAM 시스템의 한계를 극복하고, 의미론적 정보를 통합하여 특징 연관성 최적화 및 키프레임 선택의 정확도와 강인성을 획기적으로 향상시킨 중요한 진보를 이루었다. 요소 그래프 기반의 강하게 결합된 센서 융합, IMU 사전 통합을 통한 점군 왜곡 보정 및 초기 추정치 제공, 그리고 의미론적 정보를 활용한 적응형 키프레임 선택은 LIO-SAM++의 핵심적인 기여이다. 특히, 이웃 법선 벡터 일관성 기반의 특징 연관성 방법과 의미론적 가중치 잔차를 포함하는 비용 함수 최적화는 동적 객체나 특징이 적은 복잡한 환경에서 자세 추정의 신뢰성을 크게 높였다.

KITTI 데이터셋을 통한 성능 평가는 LIO-SAM++가 기존의 LEGO-LOAM, LIO-SAM, FAST-LIO, SUMA++와 같은 최첨단 SLAM 알고리즘 대비 평균 절대 궤적 오차에서 우수한 성능을 보였음을 명확히 입증한다. 이는 LIO-SAM++가 자율 주행 차량, 이동 로봇 내비게이션 등 실제 세계의 다양한 응용 분야에서 고정밀하고 강인한 SLAM 솔루션을 제공할 수 있음을 시사한다.

향후 연구는 확률론적 지도 표현을 통한 불확실성 처리, 동적 환경에서의 강인성 추가 개선, 그리고 카메라와 같은 다른 센서와의 다중 센서 융합 확장에 초점을 맞출 필요가 있다. 또한, 심층 학습 기반의 특징 추출 및 의미론적 분할 기술의 발전과 통합은 LIO-SAM++의 성능을 더욱 극대화할 잠재력을 지닌다. LIO-SAM++는 SLAM 연구가 단순한 기하학적 일치를 넘어 환경에 대한 고수준 이해를 통해 더욱 지능적이고 강인한 자율 시스템을 구현하는 방향으로 나아가고 있음을 보여주는 중요한 이정표이다.

#### **참고 자료**

1. Factor graph structure of LIO-SAM. - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/figure/Factor-graph-structure-of-LIO-SAM_fig6_361293274
2. KN-LIO: Geometric Kinematics and Neural Field Coupled LiDAR-Inertial Odometry - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2501.04263v1
3. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, 8월 2, 2025에 액세스, https://www.mdpi.com/1424-8220/24/23/7546
4. IA-LIO-SAM - Autoware Universe Documentation, 8월 2, 2025에 액세스, https://autowarefoundation.github.io/autoware-documentation/pr-279/how-to-guides/integrating-autoware/creating-maps/open-source-slam/ia-lio-slam/
5. Performance Analysis of LIO-SAM - Ronia Peterson, 8월 2, 2025에 액세스, http://www.roniapeterson.com/LIO-SAM.pdf
6. (PDF) LIO-SAM: Tightly-coupled Lidar Inertial Odometry via ..., 8월 2, 2025에 액세스, https://www.researchgate.net/publication/362412728_LIO-SAM_Tightly-coupled_Lidar_Inertial_Odometry_via_Smoothing_and_Mapping
7. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/2007.00258
8. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association ..., 8월 2, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
9. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/386200068_LIO-SAM_A_Lidar-Inertial_Semantic_SLAM_with_Association_Optimization_and_Keyframe_Selection
10. Factor Graph Accelerator for LiDAR-Inertial Odometry (Invited Paper) - Horizon Lab, 8월 2, 2025에 액세스, https://horizon-lab.org/pubs/iccad22.pdf
11. Factor Graphs and GTSAM, 8월 2, 2025에 액세스, https://gtsam.org/tutorials/intro.html
12. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, 8월 2, 2025에 액세스, https://github.com/TixiaoShan/LIO-SAM
13. [2101.02874] A general framework for modeling and dynamic simulation of multibody systems using factor graphs - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/2101.02874
14. LCDNet: Deep Loop Closure Detection and Point Cloud ... - SciSpace, 8월 2, 2025에 액세스, https://scispace.com/pdf/lcdnet-deep-loop-closure-detection-and-point-cloud-3k0ohx7i.pdf
15. Marked-LIEO: Visual Marker-Aided LiDAR/IMU/Encoder Integrated Odometry - PMC, 8월 2, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9269198/
16. JokerJohn/LIO_SAM_6AXIS: LIO_SAM for 6-axis IMU and GNSS. - GitHub, 8월 2, 2025에 액세스, https://github.com/JokerJohn/LIO_SAM_6AXIS
17. BEV-LIO(LC): BEV Image Assisted LiDAR-Inertial Odometry with Loop Closure - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2502.19242v2
18. BEV-LIO(LC): BEV Image Assisted LiDAR-Inertial Odometry with Loop Closure - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2502.19242v1
19. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, 8월 2, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8654169/
20. D-LIO: 6DoF Direct LiDAR-Inertial Odometry based on Simultaneous Truncated Distance Field Mapping - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2505.16726v1
21. LIO-GVM: an Accurate, Tightly-Coupled Lidar-Inertial Odometry with Gaussian Voxel Map, 8월 2, 2025에 액세스, https://arxiv.org/html/2306.17436v3
22. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, 8월 2, 2025에 액세스, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053
23. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/2206.09463
24. [PDF] LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping, 8월 2, 2025에 액세스, https://www.semanticscholar.org/paper/LIO-SAM%3A-Tightly-coupled-Lidar-Inertial-Odometry-Shan-Englot/e49902d212374c81b48d23c3cac1cee6e2339edd
25. Tightly-Coupled LiDAR-Visual-Inertial SLAM and Large-Scale Volumetric Occupancy Mapping - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2403.02280v1
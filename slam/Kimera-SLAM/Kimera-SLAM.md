[Kimera-SLAM](./index.md)
# Kimera-SLAM (메트릭-시맨틱 인식부터 분산형 다개체 시스템)



Kimera는 단순한 동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM) 라이브러리를 넘어, 실시간 메트릭-시맨틱(Metric-Semantic) 공간 인식을 위한 통합적 C++ 프레임워크로 정의된다. MIT SPARK Lab에서 개발한 이 오픈소스 프로젝트는 기존 SLAM 시스템이 주로 기하학적(geometric) 맵핑에 국한되었던 한계를 극복하고, 환경에 대한 의미론적(semantic) 이해를 융합하는 것을 핵심 철학으로 삼는다.1 Kimera의 중요한 특징 중 하나는 고가의 그래픽 처리 장치(GPU) 없이 표준 중앙 처리 장치(CPU)에서 실시간으로 구동되도록 설계되었다는 점이며, 이는 시스템의 접근성과 실용성을 크게 향상시킨다.2


ORB-SLAM, VINS-Mono, OKVIS와 같은 기존의 저명한 시각-관성 SLAM(Visual-Inertial SLAM, VI-SLAM) 시스템들은 주로 3D 점(point) 또는 서펠(surfel) 형태의 희소(sparse)하거나 반밀도(semi-dense) 기하학적 맵을 생성하는 데 중점을 두었다.2 이러한 시스템들은 로봇의 위치를 추정하고 이동 가능한 경로를 파악하는 데에는 효과적이었으나, 환경 내 객체들의 정체성이나 의미를 이해하는 데에는 한계가 있었다. Kimera는 이러한 시스템들의 강점을 계승하면서도, 딥러닝 기반의 2D 시맨틱 분할(semantic segmentation) 결과를 3D 공간에 실시간으로 융합하는 혁신을 도입하였다. 이를 통해 '이것이 무엇인가'라는 질문에 답할 수 있는 고밀도(dense) 3D 메시(mesh) 맵을 생성함으로써, SLAM의 역할을 단순한 '측위 및 지도 작성'에서 '공간 지능(Spatial AI)'의 영역으로 확장시킨다.2 이는 로봇이 단순히 공간을 탐색하는 것을 넘어, 공간을 이해하고 인간과 상호작용하며 복잡한 임무를 수행할 수 있는 기반을 마련한다.


본 보고서는 Kimera의 아키텍처, 핵심 수학적 원리, 성능, 그리고 다개체 시스템 및 동적 장면 그래프로의 확장을 심층적으로 분석하는 것을 목표로 한다. 먼저, 각 모듈의 유기적 상호작용을 통해 Kimera의 정교한 아키텍처를 해부한다. 다음으로, 그 기반이 되는 요인 그래프 최적화와 IMU 사전적분 이론을 수학적으로 탐구한다. 이어서, 표준 벤치마크 데이터셋을 통한 정량적 성능 평가를 통해 Kimera의 객관적 위상을 조명하고, 마지막으로 Kimera가 다개체 협업 SLAM과 고차원적 공간 표현인 동적 장면 그래프로 확장되며 지향하는 차세대 로봇 인식 시스템의 비전을 고찰한다.

Kimera의 설계 철학을 더 깊이 들여다보면, 그 모듈식 구조가 단순한 소프트웨어 공학적 우수성을 넘어, 연구 커뮤니티를 위한 촉매제 역할을 하도록 의도되었음을 알 수 있다. Kimera는 Kimera-VIO, Kimera-RPGO, Kimera-Semantics 등 각 기능이 독립적으로 실행될 수 있는 별개의 저장소로 구성되어 있다.2 이러한 설계는 학술 연구의 특성과 밀접하게 연관된다. 로보틱스 및 컴퓨터 비전 연구는 종종 전체 시스템의 특정 구성 요소(예: 새로운 루프 클로저 알고리즘, 더 효율적인 VIO 백엔드)를 개선하는 데 집중된다. Kimera의 모듈성은 특정 분야의 연구자가 전체 SLAM 파이프라인을 처음부터 구축할 필요 없이, 자신이 개발한 모듈을 기존의 검증된 Kimera 모듈과 손쉽게 결합하여 테스트할 수 있는 환경을 제공한다. 예를 들어, VIO 백엔드 연구자는 Kimera의 고성능 프론트엔드를 그대로 사용하면서 백엔드만 교체하여 성능을 벤치마킹할 수 있다. 이처럼 Kimera는 최첨단 통합 시스템을 견고한 기준점으로 제공함과 동시에, 광범위한 연구 커뮤니티가 각자의 전문 분야에서 기여할 수 있는 확장 가능한 연구 플랫폼으로서 기능한다. 이는 전체 공간 인식 분야의 발전을 가속화하는 선순환 구조를 만들어낸다.2



Kimera는 네 개의 핵심 모듈이 다중 스레드 환경에서 유기적으로 연동되는 정교한 파이프라인 구조를 가진다. 시스템은 입력으로 스테레오 또는 단안 카메라 이미지와 관성 측정 장치(IMU) 데이터를 받아, 최종적으로 (a) IMU 측정 속도에 준하는 고정밀 상태 추정치, (b) 전역적으로 일관된 궤적, (c) 저지연 로컬 메시, 그리고 (d) 전역 메트릭-시맨틱 메시라는 네 가지 핵심 결과물을 생성한다.1 각 모듈은 특정 작업을 담당하며, 서로 다른 속도의 스레드에서 동작하여 실시간 성능과 전역 최적화 사이의 균형을 맞춘다.


Kimera-VIO는 전체 시스템의 가장 앞단에서 로봇의 지역적(local)인 에고모션(ego-motion)을 빠르고 정확하게 추정하는 시각-관성 주행계(VIO)의 역할을 수행한다. 이는 두 개의 주요 스레드로 구성된다.

- **프론트엔드 (Fast Thread):** 고속 스레드에서 동작하는 프론트엔드는 IMU 측정값의 사전적분(preintegration)을 수행하여 연속적인 상태 추정치를 IMU의 높은 측정 속도(예: 200Hz 이상)로 발행한다.6 동시에, 수신되는 이미지 프레임 간의 시각적 특징점(feature)을 추적하여 움직임을 계산한다. 이 고속 추정치는 즉각적인 제어나 반응이 필요한 작업에 활용될 수 있다.
- **백엔드 (Slower Thread):** 상대적으로 느린 스레드에서 동작하는 백엔드는 VIO의 핵심적인 정밀도 향상을 담당한다. 이는 GTSAM(Georgia Tech Smoothing and Mapping) 라이브러리를 기반으로 한 요인 그래프(factor graph) 최적화 기법을 사용한다.2 백엔드는 일정 시간 윈도우 내의 키프레임(keyframe)들과 그 사이의 IMU 사전적분 값, 그리고 특징점 재투영 오차를 요인으로 구성하여 최적의 상태(자세, 속도, IMU 바이어스)를 추정한다. 특히, 연산 복잡도를 일정하게 유지하기 위해 고정 지연 스무딩(fixed-lag smoothing) 방식을 채택하여, 오래된 변수를 한계화(marginalize)함으로써 실시간성을 유지하면서도 높은 정확도를 달성한다.2


이 모듈들은 VIO 단계에서 필연적으로 누적되는 드리프트(drift) 오차를 보정하고, 맵 전체의 전역적인 궤적 일관성을 확보하는 SLAM 백엔드의 역할을 담당한다.

- **Kimera-RPGO (Robust Pose Graph Optimization):** Kimera-VIO로부터 생성된 키프레임 간의 주행계(odometry) 제약과, DBoW2 라이브러리를 이용한 루프 클로저(loop closure) 검출 모듈로부터 얻은 제약을 포즈 그래프의 엣지(edge)로 구성한다.1 이 모듈의 핵심적인 기능은 지각적 모호성(perceptual aliasing), 즉 서로 다른 장소가 비슷하게 보이는 문제로 인해 발생하는 잘못된 루프 클로저(outlier)를 강인하게(robustly) 제거하는 것이다. 이를 위해 Pairwise Consistent Measurement (PCM)와 같은 최신 이상치 제거 기법을 활용하여, 잘못된 제약이 전체 최적화에 미치는 악영향을 차단하고 SLAM 실패를 방지한다.2
- **Kimera-PGMO (Pose Graph and Mesh Optimizer):** PGMO는 RPGO의 개념을 한 단계 발전시켜, 포즈 그래프와 3D 메시를 **동시에 최적화**하는 독창적인 접근 방식을 제공한다.9 루프 클로저가 발생하여 과거의 궤적이 크게 수정될 때, 기존의 맵 데이터를 단순히 새로운 궤적에 맞춰 재투영하면 맵이 왜곡되거나 깨질 수 있다. PGMO는 이 문제를 해결하기 위해 '변형 그래프(deformation graph)'라는 자료 구조를 생성한다. 이 그래프는 포즈 그래프의 노드(로봇의 자세)와 3D 메시의 정점(vertex)들 간의 시간적 연결 관계를 모델링한다. 최적화 과정에서 궤적이 보정되면, 이 변형 그래프를 통해 메시 전체에 걸쳐 부드럽고 물리적으로 일관된 변형(deformation)이 적용된다. 결과적으로, 궤적과 맵이 항상 일관성을 유지하게 된다.9


Kimera-Mesher는 환경의 기하학적 모델링을 담당하며, 로봇의 임무 수행에 필요한 다양한 시간적 스케일의 메시를 생성한다.

- **단일 프레임 및 다중 프레임 로컬 메시:** 이 메시는 Kimera-VIO 백엔드와 동일한 스레드에서 동작하여 매우 낮은 지연 시간(<20ms)으로 현재 키프레임 주변의 3D 메시를 생성한다.6 이는 로봇의 실시간 장애물 회피나 지역 경로 계획과 같이 즉각적인 반응이 요구되는 작업에 필수적인 정보를 제공한다.2
- **전역 메시:** Kimera-RPGO 또는 PGMO에 의해 최적화된 전역 궤적을 기반으로, 그동안 생성된 로컬 메시들을 통합하여 전역적으로 일관된 기하학적 맵을 최종적으로 구축한다. 이 맵은 탐사된 전체 환경에 대한 포괄적인 기하학적 정보를 담고 있다.


Kimera-Semantics는 기하학적 3D 메시에 의미론적 정보를 융합하는 모듈로서, Kimera를 다른 SLAM 시스템과 차별화하는 가장 핵심적인 구성 요소이다.

- **융합 파이프라인:** 이 모듈은 외부의 딥러닝 기반 시맨틱 분할 모델(예: MobileNet)로부터 제공된 2D 이미지의 픽셀별 시맨틱 레이블(예: '벽', '바닥', '의자')과, VIO를 통해 추정된 깊이(depth) 정보를 결합한다.10 이 결합된 3D 시맨틱 포인트 클라우드는 Kimera-VIO의 정밀한 포즈 추정치를 이용해 3D 공간에 정확하게 투영되고, 이를 전역 볼륨(global volume)에 점진적으로 통합한다.1
- **기반 기술:** 내부적으로는 Voxblox 또는 OpenChisel과 같은 효율적인 volumetric fusion 라이브러리를 재사용한다.11 이들은 환경을 복셀(voxel) 그리드로 표현하고, 각 복셀에 Truncated Signed Distance Field (TSDF) 값을 저장하여 표면을 부드럽게 표현한다. Kimera-Semantics는 여기에 더해 각 복셀에 시맨틱 레이블의 확률 분포를 함께 저장한다. 최종적으로 Marching Cubes와 같은 알고리즘을 통해 이 볼륨 데이터로부터 시맨틱 정보가 색상으로 표현된 고품질의 3D 메시를 추출한다.

Kimera의 아키텍처에서 주목할 점은 기하학적 추정과 시맨틱 맵핑의 의도적인 분리이다. 고밀도 시맨틱 정보를 매 프레임마다 융합하는 작업은 연산 비용이 매우 높다. 만약 이 과정이 시간 제약이 엄격한 VIO 최적화 루프 내에서 수행된다면, CPU 기반의 실시간 성능을 달성하는 것은 거의 불가능하다. 또한, SLAM 시스템의 근간은 기하학적 정확도이다. 부정확한 궤적은 아무리 우수한 시맨틱 분할 정보가 있더라도 무의미한 시맨틱 맵을 생성할 뿐이다. 따라서 시스템은 기하학적 추정을 최우선으로 처리해야 한다. Kimera는 이 두 가지를 아키텍처 수준에서 분리함으로써 이러한 문제를 해결한다.6 VIO와 RPGO 모듈은 실시간으로 최고 수준의 기하학적 정확도를 달성하는 데 모든 연산 자원을 집중한다. 한편, Kimera-Semantics 모듈은 이렇게 생성된 고품질의 포즈 추정치를 비동기적으로 받아, 별도의 저속 스레드에서 시맨틱 맵을 구축한다. 이러한 분리 구조는 시맨틱 맵핑 과정을 확장 가능하고 적응적으로 만든다. 예를 들어, 더 강력한 CPU가 있다면 시맨틱 업데이트 주기를 높일 수 있고, 미래에 더 효율적인 volumetric fusion 알고리즘이 개발된다면 Kimera-Semantics 모듈만 교체하면 된다. 이처럼 핵심 SLAM 엔진에 영향을 주지 않으면서도 시맨틱 구성 요소를 독립적으로 발전시킬 수 있는 구조는 Kimera 아키텍처의 장기적인 생명력과 확장성을 보장하는 핵심적인 설계 결정이라 할 수 있다.11


| 모듈 (Module)        | 주요 기능 (Primary Function) | 핵심 기술 (Core Technology)                | 출력 (Output)                    | 동작 스레드 (Thread) |
| -------------------- | ---------------------------- | ------------------------------------------ | -------------------------------- | -------------------- |
| **Kimera-VIO**       | 실시간 시각-관성 주행계      | IMU 사전적분, 요인 그래프 최적화 (GTSAM)   | 고속 상태 추정, 키프레임 궤적    | 고속/중속            |
| **Kimera-RPGO**      | 전역 궤적 최적화             | 포즈 그래프 최적화, 이상치 제거 (PCM)      | 전역 일관성을 갖춘 궤적          | 저속                 |
| **Kimera-PGMO**      | 궤적 및 메시 동시 최적화     | 변형 그래프(Deformation Graph) 기반 최적화 | 최적화된 궤적 및 변형된 메시     | 저속                 |
| **Kimera-Mesher**    | 3D 기하학적 재구성           | 델라우니 삼각분할(Delaunay Triangulation)  | 저지연 로컬 메시, 전역 기하 메시 | 중속                 |
| **Kimera-Semantics** | 3D 시맨틱 맵핑               | 2D-3D 레이블 융합, TSDF 통합 (Voxblox)     | 메트릭-시맨틱 3D 메시            | 저속                 |




시각-관성 주행계(VIO)는 본질적으로 확률적 추론 문제로 정의될 수 있다. 시스템의 목표는 시간에 따라 수집된 일련의 IMU 측정값 $U$와 시각적 특징점 관측값 $Z$가 주어졌을 때, 로봇의 상태 변수 $X$ (자세, 속도, IMU 바이어스 등을 포함)의 전체 시퀀스를 가장 가능성 높은 값으로 추정하는 것이다. 이는 통계학적으로 최대 사후 확률(Maximum-a-Posteriori, MAP) 추정 문제로 귀결된다.12


베이즈 정리에 따라, 사후 확률 $P(X|Z,U)$는 우도(likelihood) $P(Z,U|X)$와 사전 확률(prior) $P(X)$의 곱에 비례한다. 센서 측정 노이즈가 일반적으로 영평균 가우시안 분포를 따른다고 가정하면, MAP 추정 문제는 확률의 곱을 최대화하는 대신 로그 확률의 합을 최대화하는 문제로 변환된다. 이는 다시 음의 로그 확률의 합을 최소화하는 비선형 최소 제곱 문제(Non-linear Least Squares Problem)를 푸는 것과 등가가 된다. 최적화하고자 하는 상태 변수 $X^*$는 다음의 비용 함수를 최소화함으로써 얻을 수 있다.13
$$
X^* = \underset{X}{\text{argmax}} P(X|Z,U) \propto \underset{X}{\text{argmin}} \left( \lVert r_{\text{prior}}(X_0) \rVert^2_{\Sigma_{\text{prior}}} + \sum_{i,j} \lVert r_{\text{IMU}}(X_i, X_j) \rVert^2_{\Sigma_{\text{IMU}}} + \sum_{k,l} \lVert r_{\text{vision}}(X_k, L_l) \rVert^2_{\Sigma_{\text{vision}}} \right)
$$
여기서 $r(\cdot)$는 각 측정 모델과 상태 변수 간의 오차, 즉 잔차(residual)를 나타낸다. $r_{\text{prior}}$는 초기 상태에 대한 사전 정보, $r_{\text{IMU}}$는 IMU 측정값에 의한 제약, $r_{\text{vision}}$은 시각적 특징점 재투영 오차를 의미한다. $\lVert \cdot \rVert^2_{\Sigma}$는 측정 노이즈의 공분산 행렬 $\Sigma$로 정규화된 마할라노비스 거리를 나타낸다.


이러한 복잡한 최적화 문제는 요인 그래프(factor graph)라는 그래픽 모델을 통해 직관적이고 효율적으로 표현될 수 있다. 요인 그래프에서 로봇의 상태 변수(예: 각 시점의 자세 $R_k$, 위치 $p_k$, 속도 $v_k$, 바이어스 $b_k$)는 변수 노드(variable node)로 표현된다. 각 측정값(IMU, 시각, 사전 정보)으로부터 발생하는 잔차항은 해당 변수 노드들을 연결하는 요인 노드(factor node)로 표현된다. Kimera-VIO는 Georgia Tech Smoothing and Mapping (GTSAM) 라이브러리를 활용하여 이 요인 그래프를 구성하고, Levenberg-Marquardt와 같은 반복적인 최적화 알고리즘을 통해 비용 함수를 최소화하는 해를 효율적으로 찾는다.2



VIO 시스템에서 IMU와 카메라는 서로 다른 속도로 동작한다. IMU는 일반적으로 수백 Hz의 매우 높은 빈도로 측정값을 출력하는 반면, 카메라는 수십 Hz로 상대적으로 느리게 이미지를 캡처한다.16 만약 모든 IMU 측정 시점의 로봇 상태를 최적화 변수로 포함한다면, 변수의 수가 급격히 증가하여 실시간 최적화가 불가능해진다. 더 큰 문제는, 요인 그래프 최적화 과정에서 과거 상태의 선형화 지점(linearization point)이 변경될 때마다, 해당 상태에 의존하는 모든 미래의 IMU 측정값을 반복적으로 다시 적분해야 한다는 점이다. 이는 엄청난 연산 낭비를 초래한다.


IMU 사전적분(preintegration)은 이러한 문제를 해결하기 위한 핵심적인 이론이다. 이 기법의 핵심 아이디어는 두 개의 연속된 키프레임 $i$와 $j$ 사이의 모든 IMU 측정값을 하나의 상대적인 움직임 제약으로 미리 '사전적분'해두는 것이다. 결정적으로, 이 적분은 고정된 월드 좌표계가 아닌, 시작 키프레임 $i$ 시점의 로컬 바디(body) 좌표계에서 수행된다. 이로써 최적화 과정에서 키프레임 $i$의 글로벌 포즈 추정치가 변경되더라도, $i$와 $j$ 사이의 '상대적인' 움직임을 나타내는 사전적분된 값 자체는 재계산할 필요가 없어진다. 단지 바이어스 추정치가 업데이트될 때만 사전적분 값을 선형적으로 보정해주면 된다.12


키프레임 $i$와 $j$ 사이의 시간 간격 $\Delta t_{ij}$ 동안의 상대적 자세 변화 $\Delta R_{ij}$, 속도 변화 $\Delta v_{ij}$, 위치 변화 $\Delta p_{ij}$는 $i$ 시점의 바디 좌표계를 기준으로 다음과 같이 이산적으로 적분된다.
$$
\Delta R_{ij} = \prod_{k=i}^{j-1} \text{Exp}((\tilde{\omega}_k - b_g(t_k))\Delta t)
$$

$$
\Delta v_{ij} = \sum_{k=i}^{j-1} \Delta R_{ik}(\tilde{a}_k - b_a(t_k))\Delta t
$$

$$
\Delta p_{ij} = \sum_{k=i}^{j-1}
\left [
\Delta \mathbf{v}_{ik}
\Delta t 
+
\frac{1}{2} \Delta \mathbf{R}_{ik} (\tilde{\mathbf{a}}(t_k)-\mathbf{b}_{a_i})(\Delta)^2
\right ]
$$

여기서 $\tilde{\omega}_k$와 $\tilde{a}_k$는 $k$ 시점에서 IMU가 측정한 각속도와 가속도이며, $b_g$와 $b_a$는 각각 자이로스코프와 가속도계의 시변(time-varying) 바이어스이다.19 자세는 회전 그룹 $SO(3)$라는 매니폴드(manifold) 상에 존재하므로, 적분은 벡터 공간에서의 덧셈이 아닌 리 대수(Lie algebra) $so(3)$ 상에서의 지수 사상(exponential map) $\text{Exp}(\cdot)$을 통해 수행된다. 이는 회전의 비선형적 특성을 올바르게 다루기 위함이다.12


이렇게 계산된 사전적분된 측정값 $(\Delta \tilde{R}_{ij}, \Delta \tilde{v}_{ij}, \Delta \tilde{p}_{ij})$은 요인 그래프에서 두 키프레임의 상태 $(R_i, v_i, p_i, b_i)$와 $(R_j, v_j, p_j, b_j)$를 연결하는 강력한 제약, 즉 IMU 요인을 형성한다. 이 요인의 잔차 $r_{ij}$는 예측된 상태 변화와 사전적분된 측정값 간의 차이로 정의된다.
$$
r_{ij} = \begin{bmatrix} \text{Log}((\Delta \tilde{R}_{ij}(b_g))^{-1} (R_i^T R_j)) \\ R_i^T(v_j - v_i - g\Delta t_{ij}) - \Delta \tilde{v}_{ij}(b_g, b_a) \\ R_i^T(p_j - p_i - v_i\Delta t_{ij} - \frac{1}{2}g\Delta t_{ij}^2) - \Delta \tilde{p}_{ij}(b_g, b_a) \end{bmatrix}
$$
여기서 $g$는 월드 좌표계에서의 중력 벡터이며, $\text{Log}(\cdot)$는 $SO(3)$에서 $so(3)$로의 로그 사상(logarithm map)으로, 두 회전 간의 차이를 벡터로 표현한다. 최적화기는 이 잔차 벡터의 마할라노비스 거리를 최소화함으로써, IMU 측정값과 가장 일치하는 상태 시퀀스를 찾아낸다.



Kimera-VIO에서 계산된 연속적인 키프레임 간의 상대 변환(자세 및 위치 변화)은 Kimera-RPGO의 포즈 그래프에서 기본적인 제약, 즉 주행계 요인을 구성한다. 이 요인들은 로봇 궤적의 '뼈대'를 형성하지만, 시간이 지남에 따라 오차가 누적되어 드리프트를 유발한다.


로봇이 과거에 방문했던 장소를 재인식하면(예: DBoW2와 같은 장소 인식 기술 사용), 현재 키프레임과 과거의 키프레임 사이에 새로운 상대 변환 제약, 즉 루프 클로저 요인이 포즈 그래프에 추가된다.5 이 요인은 그래프에 큰 루프를 형성하여, 그동안 누적된 모든 오차를 루프 전체에 걸쳐 분산시키고 보정하는 결정적인 역할을 한다. 이는 맵의 전역 일관성을 크게 향상시키고 드리프트를 극적으로 줄여준다.


실제 환경에서는 비슷하게 생긴 다른 방이나 복도를 같은 장소로 오인하는 '지각적 모호성'으로 인해 잘못된 루프 클로저가 빈번하게 발생한다. 이러한 이상치(outlier) 요인은 최적화 과정을 오염시켜 전체 맵을 심각하게 왜곡시킬 수 있다. Kimera-RPGO는 이러한 문제를 해결하기 위해 Graduated Non-Convexity (GNC)나 Pairwise Consistent Measurement (PCM)와 같은 강인한 추정(robust estimation) 기법을 사용한다.4 이 기법들은 M-추정(M-estimation)과 유사한 원리로 동작하며, 최적화 과정에서 잔차가 비정상적으로 큰 측정값(이상치로 의심되는 요인)의 가중치를 동적으로 낮추어 최적화에 미치는 영향을 줄인다. 이를 통해 소수의 강력한 이상치가 존재하더라도 시스템이 안정적인 해를 찾을 수 있도록 보장한다.

IMU 사전적분 이론과 요인 그래프 최적화는 VIO 시스템에서 서로를 가능하게 하는 공생 관계에 있다. 요인 그래프의 가장 큰 장점은 루프 클로저와 같은 새로운 정보가 들어왔을 때, 전체 문제의 이력을 다시 선형화하고 최적화하여 전역적인 일관성을 달성할 수 있다는 점이다.13 그러나 이 과정은 과거 상태들 간의 제약(요인)을 다시 평가해야 함을 의미한다. 만약 IMU 제약이 과거 상태의 추정치가 업데이트될 때마다 수백, 수천 개의 원시 IMU 측정값을 다시 적분해야 한다면, 재선형화에 드는 연산 비용은 천문학적으로 증가하여 실시간 적용이 불가능해질 것이다. IMU 사전적분은 바로 이 문제를 해결하는 계산적 열쇠이다. 이는 IMU 데이터를 월드 좌표계와 무관한 간결한 요약본으로 변환한다. 요인 그래프가 키프레임 

$i$의 자세를 업데이트하더라도, 사전적분된 제약 $\Delta R_{ij}$ 자체는 변하지 않는다. 단지 $R_i$를 포함하는 최종 잔차 계산만 다시 수행하면 된다. 이처럼 IMU 사전적분은 수천 개의 IMU 측정값에 대한 최적화라는 다루기 힘든 문제를, 훨씬 적은 수의 키프레임 상태와 사전적분된 요인에 대한 최적화라는 다루기 쉬운 문제로 변환한다. 이 공생 관계야말로 Kimera와 같은 고성능 스무딩 기반 VIO 시스템의 이론적 초석이라 할 수 있다.12




마이크로 비행 로봇(MAV)으로 실내 환경에서 수집된 EuRoC 데이터셋은 다양한 조명 조건과 움직임 속도를 포함하여 VI-SLAM 알고리즘의 성능을 평가하기 위한 표준 벤치마크로 널리 사용된다. Kimera는 이 데이터셋에서 여러 최신 시스템들과의 비교를 통해 최상위권의 성능을 입증하였다.2


성능 평가는 주로 절대 궤적 오차(Absolute Trajectory Error, ATE)의 제곱평균제곱근(Root Mean Squared Error, RMSE)을 기준으로 이루어진다. 아래 표 2는 Kimera를 VINS-Mono, OKVIS, 그리고 또 다른 최첨단 시스템인 ORB-SLAM3와 비교한 결과를 종합한 것이다. Kimera-VIO (full smoothing, 모든 키프레임을 최적화)와 Kimera-RPGO (loop closure, 루프 클로저 적용)는 대부분의 시퀀스에서 가장 낮은 오차 수준을 기록하며 탁월한 정확도를 보여준다. 특히 Kimera-RPGO는 강인한 이상치 제거 메커니즘 덕분에 루프 클로저 관련 파라미터 튜닝에 덜 민감하고, 다양한 조건에서 안정적인 성능을 유지하는 장점을 가진다.2 한편, 22와 22의 데이터를 종합하면, ORB-SLAM3는 특히 가장 어려운 시퀀스인 `MH_05`에서 Kimera보다 월등히 낮은 오차를 기록하며 극도의 강인성을 보여주었다. 전반적으로 Kimera와 ORB-SLAM3는 현존하는 최고 수준의 정확도를 가진 시스템들로 평가될 수 있다.


TUM-VI 데이터셋은 빠른 핸드헬드 모션, 급격한 조명 변화, 큰 시점 변화 등을 포함하여 증강/가상현실(AR/VR) 시나리오를 모사한 고난도 벤치마크이다. ORB-SLAM3는 이 데이터셋의 실내 시퀀스에서 평균 9mm 수준의 놀라운 정확도를 달성했다고 보고하며, 해당 분야에서의 기술적 우위를 과시했다.2322에 따르면 Kimera의 TUM-VI 데이터셋 결과는 해당 문헌에서 직접적으로 비교되지 않았으나, ORB-SLAM3의 성능은 이 까다로운 데이터셋에서의 최첨단(State-of-the-Art, SOTA) 성능 기준으로 참조될 수 있다.


| 시스템 (System)                 | MH_01 (easy) | MH_05 (difficult) | V1_01 (easy) | V2_02 (medium) |
| ------------------------------- | ------------ | ----------------- | ------------ | -------------- |
| OKVIS                           | 0.16         | 0.40              | 0.11         | 0.14           |
| VINS-Mono                       | 0.15         | 0.38              | 0.11         | 0.16           |
| **Kimera-VIO (Full Smoothing)** | **0.04**     | 0.34              | **0.04**     | **0.06**       |
| **Kimera-RPGO (Loop Closure)**  | 0.08         | -                 | 0.05         | 0.06           |
| ORB-SLAM3                       | 0.03         | **0.04**          | 0.03         | -              |

주: 데이터는 2에서 종합되었으며, '-'는 해당 문헌에 결과가 보고되지 않았음을 의미한다. 굵은 글씨는 각 시퀀스에서 가장 우수한 성능을 나타낸다.



Kimera의 가장 중요한 설계상 장점 중 하나는 GPU와 같은 특수 하드웨어 없이, 모든 모듈이 표준 CPU에서 실시간으로 동작한다는 점이다.2 이는 고밀도 3D 재구성을 위해 GPU의 병렬 처리 능력에 의존하는 다른 많은 시맨틱 SLAM 시스템들과 비교했을 때, 하드웨어 요구사항이 낮고 다양한 플랫폼에 적용하기 용이함을 의미한다.


Kimera의 실시간 성능은 각 모듈의 연산 시간을 정밀하게 분석함으로써 확인할 수 있다. 아래 표 3은 EuRoC 데이터셋 실험에서 측정된 각 모듈의 평균 처리 시간을 보여준다.2 IMU 프론트엔드는 약 40µs 만에 사전적분을 완료하여 IMU 측정 속도에 맞춰 상태 추정이 가능하다. VIO의 핵심 연산인 특징점 추적(4.5ms), 백엔드 최적화(40ms 미만) 및 Kimera-Mesher의 로컬 메시 생성(5-15ms)은 모두 비디오 프레임 속도 내에서 충분히 처리 가능한 수준이다. 상대적으로 연산 시간이 긴 Kimera-RPGO(평균 55ms)와 Kimera-Semantics(평균 100ms)는 시스템의 실시간성에 영향을 주지 않도록 별도의 저속 스레드에서 비동기적으로 동작한다. 이러한 다중 스레드 아키텍처는 시간 제약이 엄격한 작업과 전역 일관성을 위한 무거운 작업을 효과적으로 분리하여 전체 시스템의 반응성과 처리량을 극대화한다.


| 모듈 (Module)    | 작업 (Task)                  | 평균 처리 시간 (Average Processing Time) | 비고 (Notes)                          |
| ---------------- | ---------------------------- | ---------------------------------------- | ------------------------------------- |
| IMU Front-end    | Preintegration               | ~40 µs                                   | IMU 속도(>200Hz)에서 상태 추정 가능   |
| Vision Front-end | Feature Tracking             | ~4.5 ms                                  | 매 프레임마다 수행                    |
|                  | Feature Detection & Matching | ~45 ms                                   | 키프레임에서만 수행                   |
| VIO Back-end     | Factor Graph Optimization    | < 40 ms                                  | 키프레임에서만 수행                   |
| Kimera-Mesher    | Per-frame Mesh Generation    | < 5 ms                                   | 저지연 로컬 맵                        |
|                  | Multi-frame Mesh Update      | ~15 ms                                   | 로컬 맵 확장                          |
| Kimera-RPGO      | Pose Graph Optimization      | ~55 ms                                   | 저속 스레드, 그래프 크기에 따라 변동  |
| Kimera-Semantics | Global Semantic Mesh Update  | ~100 ms                                  | 저속 스레드, 720x480 깊이 이미지 기준 |

주: 데이터는 2의 EuRoC 데이터셋 실험 결과를 기반으로 한다.




수색 및 구조, 대규모 환경 감시, 행성 탐사와 같은 응용 분야에서는 단일 로봇의 능력만으로는 넓은 영역을 신속하게 탐사하고 상황을 인식하는 데 한계가 있다.4 다개체 SLAM은 여러 로봇이 협력하여 공동의 맵을 생성함으로써 이러한 한계를 극복하는 것을 목표로 한다. Kimera-Multi는 중앙 서버나 기지국에 의존하지 않고, 로봇 간의 P2P(Peer-to-Peer) 통신에만 의존하는 완전 분산형(fully distributed) 시스템을 지향한다.24 이러한 접근 방식은 중앙 서버와의 통신 병목 현상을 근본적으로 해결하고, 통신이 단절될 수 있는 실제 환경에서의 시스템 확장성과 강인성을 크게 향상시킨다.26


Kimera-Multi 시스템에서 각 로봇은 독립적으로 완전한 Kimera 파이프라인을 실행하여 자신만의 로컬 맵과 궤적을 실시간으로 생성한다. 두 로봇이 통신 가능 범위에 들어오면, 이들은 서로의 시각적 정보를 교환하여 협업을 시작한다. 구체적으로, 각 로봇은 자신의 키프레임에서 추출한 시각적 Bag-of-Words(BoW) 벡터를 상대방에게 전송한다. 수신 로봇은 이 BoW 벡터를 자신의 데이터베이스와 비교하여 잠재적인 로봇 간 루프 클로저, 즉 두 로봇이 동일한 장소를 방문했음을 나타내는 후보를 식별한다.25


이렇게 검출된 로봇 간 루프 클로저 후보에는 높은 비율의 이상치(outlier)가 포함될 수 있다. Kimera-Multi는 이러한 문제를 해결하기 위해 분산 GNC(Distributed Graduated Non-Convexity) 알고리즘에 기반한 강인한 분산 포즈 그래프 최적화(DPGO) 기술을 개발하였다.21 이 알고리즘은 각 로봇이 자신의 지역적인 정보와 이웃 로봇으로부터 받은 정보만을 사용하여, 전역 포즈 그래프에서 이상치로 의심되는 제약들을 식별하고 그 영향을 점진적으로 줄여나간다. 이 과정을 통해 모든 로봇의 궤적은 이상치의 방해 없이 전역적으로 일관된 단일 좌표계로 정렬된다.


실험 결과, Kimera-Multi는 중앙 집중식 서버가 모든 정보를 취합하여 최적화하는 시스템과 비교했을 때 거의 동등한 수준의 정확도를 달성하면서도, 통신 대역폭을 매우 효율적으로 사용함을 보여주었다.4 또한, 기존의 다른 분산 SLAM 시스템들과 비교했을 때 이상치에 대한 강인성과 최종 궤적의 정확도 측면에서 월등한 성능을 입증하였다.27



3D 동적 장면 그래프(Dynamic Scene Graph, DSG)는 Kimera 프로젝트가 지향하는 궁극적인 목표를 보여주는 개념이다. 이는 환경을 단순한 점, 선, 면의 기하학적 집합으로 표현하는 것을 넘어, 인간이 세상을 인식하는 방식과 유사하게 계층적이고 관계적인 구조로 표현하려는 시도이다. DSG는 '어디에'와 '무엇이'라는 정보를 넘어 '누가', 그리고 이들 간의 '관계는 어떠한가'까지 모델링하는 것을 목표로 한다.5


DSG는 여러 추상화 수준에 해당하는 계층적 그래프로 구성된다.5

1. **Layer 1 (Metric-Semantic Mesh):** 가장 낮은 계층으로, Kimera-Semantics가 생성한 정적 환경의 메트릭-시맨틱 메시이다. 이는 모든 고차원적 이해의 기하학적, 의미론적 기반이 된다.
2. **Layer 2 (Objects & Agents):** 메시로부터 분리된 개별 정적 객체(책상, 의자 등)와 동적 행위자(사람 등)를 노드로 표현한다. Kimera는 외부의 객체 탐지 및 인간 자세 추정 알고리즘을 통합하여 이들을 실시간으로 탐지하고 그래프에 노드로 추가한다.5
3. **Layer 3 (Places):** 방, 복도, 주방과 같이 기능적이고 의미 있는 공간 단위를 노드로 표현한다. 이는 로봇이 인간의 지시를 이해하는 데 중요한 추상화 수준이다.
4. **Layer 4 (Structures):** 건물과 같이 장소들의 집합으로 구성된 더 큰 구조물을 표현한다.


각 계층의 노드들은 공간적('~의 옆에'), 시간적, 그리고 의미론적('~의 일부이다') 관계를 나타내는 엣지(edge)로 연결된다. 예를 들어, '사람 A' 노드는 '주방' 노드와 '안에 있다(is in)'라는 엣지로 연결되고, '의자 B' 노드는 '테이블 C' 노드와 '옆에 있다(is next to)'라는 엣지로 연결될 수 있다.


DSG는 로봇이 "2층에서 생존자를 수색하라" 또는 "주방에 있는 내 가방을 가져오라"와 같은 고수준의 인간 명령을 이해하고 실행 가능한 계획으로 변환하기 위해 필수적인 내부 표현(internal representation)을 제공한다.5 이는 로봇 지능을 단순히 A 지점에서 B 지점으로 이동하는 내비게이션 수준에서, 환경의 맥락을 이해하고 복잡한 임무를 수행하며 인간과 효과적으로 상호작용하는 고차원적 수준으로 발전시키는 핵심 기술이다.

Kimera 프로젝트의 발전 과정을 살펴보면, 단순한 SLAM 라이브러리 개선을 넘어 공간 AI를 위한 기초 인식 엔진을 구축하려는 명확하고 야심 찬 지적 궤적을 발견할 수 있다. 프로젝트는 기하학(SLAM)과 의미론(segmentation)을 결합한 'Kimera'에서 시작했다.1 다음 논리적 단계는 인식 범위를 확장하는 것이었고, 이는 다개체 협업을 다루는 'Kimera-Multi'로 이어졌다.4 그러나 이 두 단계는 근본적으로 '더 나은 맵을 만드는 것'에 초점을 맞추고 있다. DSG의 도입은 이러한 패러다임의 전환을 의미한다.5 이제 맵은 최종 결과물이 아니라, 훨씬 더 추상적인 지식 기반의 

**첫 번째 계층**이 된다. DSG의 목표는 단순히 세계를 표현하는 것을 넘어, 지능형 에이전트가 **행동할 수 있는(actionable)** 형태로 만드는 것이다. 이는 계획, 추론, 자연어 이해와 같은 고차원적 인지 기능에 필요한 상징적 기반(symbolic grounding)을 제공한다. 결론적으로, Kimera 프로젝트의 진화는 체계적인 노력을 통해 진정한 공간 AI를 위한 기초 인식 파이프라인을 구축하는 과정으로 해석할 수 있다. Kimera-VIO는 미터법 기반을 제공하고, Kimera-Semantics는 의미론적 레이블을 추가하며, Kimera-Multi는 인식 규모를 확장하고, 최종적으로 DSG는 이 모든 것을 종합하여 지능형 로봇의 '뇌'에 입력될 수 있는 계층적, 관계적 세계 모델을 합성한다. 따라서 Kimera는 단순한 SLAM 라이브러리가 아니라, 공간 인지를 위한 진행형 엔진이라 할 수 있다.



Kimera가 제공하는 풍부한 메트릭-시맨틱 정보와 강인한 성능은 다양한 첨단 기술 분야에 적용될 수 있다.

- **자율 로봇:** 정밀한 자기 위치 추정, 3D 메시를 이용한 장애물 회피는 기본이며, 나아가 시맨틱 맵을 활용하여 "복도 끝에 있는 파란 문으로 가라"와 같은 고수준의 작업 계획 및 실행이 가능하다. 이는 특히 수색 및 구조, 물류, 공장 자동화와 같은 분야에서 로봇의 자율성을 획기적으로 향상시킨다.4
- **증강 현실 (AR):** AR 애플리케이션이 가상 객체를 실제 공간에 정확하게 정합하기 위해서는 카메라의 6-DoF 자세를 실시간으로 추정하는 것이 필수적이다. Kimera-VIO는 이를 위한 강력한 기반을 제공한다. 더 나아가 Kimera-Semantics는 '벽', '바닥', '테이블'과 같은 표면의 의미를 인식하여, 가상 캐릭터가 바닥 위를 걷거나 가상 물체가 테이블 위에 놓이는 등, 훨씬 더 현실감 있고 깊이 있는 상호작용을 가능하게 한다.23
- **자율 주행:** Kimera는 자율 주행 차량을 위한 고정밀 지도(HD Map)를 생성하고, 변화하는 도로 환경에 맞춰 이를 지속적으로 업데이트하는 데 사용될 수 있다.28 또한, DSG 개념을 확장하면 정적인 지도 정보를 넘어 다른 차량이나 보행자와 같은 동적 객체들의 움직임을 인지하고 예측하여 더욱 안전한 주행 계획을 수립하는 데 기여할 수 있다.


Kimera는 최첨단 기술임에도 불구하고, 모든 SLAM 시스템이 직면하는 근본적인 도전 과제들로부터 자유롭지 않다.

- **고도의 동적 환경:** Kimera의 핵심인 VIO는 기본적으로 정적인 환경을 가정하고 특징점을 추적한다. DSG가 동적 행위자를 별도로 모델링하지만, 사람이나 차량이 많은 혼잡한 환경에서는 배경의 정적 특징점이 가려지거나 부족해져 VIO의 추적 성능이 저하될 수 있다.8
- **장기 운용 신뢰성:** 조명의 극심한 변화, 계절이나 시간에 따른 환경의 외형 변화 등 장기간에 걸쳐 발생하는 변화에 대한 강인성은 여전히 SLAM 분야의 주요 난제이다. 이러한 변화는 장소 인식의 실패를 유발하여 루프 클로저 성능을 저하시킬 수 있다.31
- **시맨틱 분할 의존성:** 3D 메트릭-시맨틱 맵의 품질은 입력으로 사용되는 2D 시맨틱 분할 모델의 성능에 직접적으로 의존한다. 2D 분할 모델이 객체를 잘못 인식하거나 경계를 부정확하게 판단하면, 이러한 오류는 3D 맵에 그대로 전파되어 시맨틱 맵의 신뢰도를 떨어뜨린다.


Kimera가 제시한 비전과 현재의 기술적 한계는 다음과 같은 미래 연구 방향을 제시한다.

- **동적 SLAM과의 통합:** 환경 내 동적 객체들을 명시적으로 탐지하고 추적하며, 이들의 움직임을 상태 추정 모델에 통합하여 동적인 환경에서의 VIO 강인성을 근본적으로 높이는 연구가 필요하다.
- **생성형 모델 및 신경망 렌더링 활용:** 최근 각광받는 Gaussian Splatting이나 NeRF(Neural Radiance Fields)와 같은 기술을 SLAM의 맵 표현 방식으로 통합하는 연구가 활발히 진행되고 있다. 이는 맵의 사실성과 충실도를 극적으로 높일 뿐만 아니라, 새로운 시점에서의 뷰 합성을 통해 데이터 연관(data association) 성능을 강화할 잠재력을 가진다.
- **DSG 기반 고수준 추론:** 현재의 DSG는 환경을 구조적으로 표현하는 데 중점을 두고 있다. 구축된 DSG를 기반으로 인과 관계를 추론하거나, 에이전트의 행동에 따른 미래 상태를 예측하고, 자연어를 통해 복잡한 임무를 계획하는 등, 진정한 의미의 공간 지능을 구현하는 방향으로 연구가 확장되어야 한다.5
- **자원 제약 환경에서의 최적화:** Kimera-Multi에서 다루었듯이, 제한된 연산 능력과 통신 대역폭을 가진 다수의 로봇이 효율적으로 협업 SLAM을 수행하기 위한 자원 관리 및 정보 선택(information selection) 기법에 대한 연구는 여전히 중요한 과제이다. 어떤 정보를 언제 누구와 공유할 것인지를 지능적으로 결정하는 기술은 대규모 로봇 군집의 실용화를 위해 필수적이다.26


1. MIT-SPARK/Kimera: Index repo for Kimera code - GitHub, 8월 17, 2025에 액세스, https://github.com/MIT-SPARK/Kimera
2. Kimera: an Open-Source Library for Real-Time Metric-Semantic Localization and Mapping - MIT, 8월 17, 2025에 액세스, https://www.mit.edu/~arosinol/papers/Rosinol20icra-Kimera.pdf
3. Kimera: An Open-Source Library for Real-Time Metric-Semantic Localization and Mapping | Request PDF - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/355988882_Kimera_An_Open-Source_Library_for_Real-Time_Metric-Semantic_Localization_and_Mapping
4. Kimera-Multi: Robust, Distributed, Dense Metric-Semantic SLAM for Multi-Robot Systems - DSpace@MIT, 8월 17, 2025에 액세스, https://dspace.mit.edu/bitstream/handle/1721.1/145301/2106.14386.pdf?sequence=2&isAllowed=y
5. Kimera: from SLAM to Spatial Perception with 3D Dynamic Scene Graphs - ar5iv - arXiv, 8월 17, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2101.06894
6. Kimera's architecture. Kimera uses images and IMU data as input (shown... - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/figure/Kimeras-architecture-Kimera-uses-images-and-IMU-data-as-input-shown-on-the-left-and_fig1_336316753
7. MIT-SPARK - GitHub, 8월 17, 2025에 액세스, https://github.com/mit-spark
8. Kimera's architecture. Kimera uses images and IMU data as input (shown... | Download Scientific Diagram - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/figure/Kimeras-architecture-Kimera-uses-images-and-IMU-data-as-input-shown-on-the-left-and_fig1_344982175
9. MIT-SPARK/Kimera-PGMO - GitHub, 8월 17, 2025에 액세스, https://github.com/MIT-SPARK/Kimera-PGMO
10. SLAM – GHAR - MRSD Projects, 8월 17, 2025에 액세스, https://mrsdprojects.ri.cmu.edu/2023teami/slam/
11. MIT-SPARK/Kimera-Semantics: Real-Time 3D Semantic ... - GitHub, 8월 17, 2025에 액세스, https://github.com/MIT-SPARK/Kimera-Semantics
12. IMU Preintegration on Manifold for Efficient Visual-Inertial Maximum-a-Posteriori Estimation - Robotics, 8월 17, 2025에 액세스, https://www.roboticsproceedings.org/rss11/p06.pdf
13. Feature Selection and Tracking for Multi-Camera Visual-Inertial Odometry - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/pdf/2109.05975
14. Factor Graphs for Navigation Applications: A Tutorial, 8월 17, 2025에 액세스, https://navi.ion.org/content/navi/71/3/navi.653.full.pdf
15. Visual-inertial SLAM using a monocular camera and detailed map data, 8월 17, 2025에 액세스, https://liu.diva-portal.org/smash/get/diva2:1746879/FULLTEXT01.pdf
16. A Comprehensive Introduction of Visual-Inertial Navigation - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/pdf/2307.11758
17. Visual-Inertial Odometry of Aerial Robots - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/pdf/1906.03289
18. IMU Preintegration on Manifold for Efficient Visual ... - Infoscience, 8월 17, 2025에 액세스, https://infoscience.epfl.ch/server/api/core/bitstreams/e23a9898-f5a4-44ce-bbb0-e70854b170d2/content
19. Continuous Integration over SO(3) for IMU Preintegration - Robotics, 8월 17, 2025에 액세스, https://www.roboticsproceedings.org/rss17/p078.pdf
20. Lecture 13 Visual Inertial Fusion - Robotics and Perception Group, 8월 17, 2025에 액세스, https://rpg.ifi.uzh.ch/docs/teaching/2020/13_visual_inertial_fusion.pdf
21. Kimera-Multi: Robust, Distributed, Dense Metric-Semantic SLAM for Multi-Robot Systems - DCIST CRA, 8월 17, 2025에 액세스, https://www.dcist-cra.org/2022/02/09/kimera-multi-robust-distributed-dense-metric-semantic-slam-for-multi-robot-systems/
22. [2108.01654] Comparison of modern open-source Visual SLAM ..., 8월 17, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2108.01654
23. ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/343179441_ORB-SLAM3_An_Accurate_Open-Source_Library_for_Visual_Visual-Inertial_and_Multi-Map_SLAM
24. Kimera-Multi: Robust, Distributed, Dense Metric-Semantic SLAM for Multi-Robot Systems, 8월 17, 2025에 액세스, https://dspace.mit.edu/handle/1721.1/145301
25. Index repo for Kimera-Multi system - GitHub, 8월 17, 2025에 액세스, https://github.com/MIT-SPARK/Kimera-Multi
26. Swarm-SLAM: Sparse Decentralized Collaborative Simultaneous Localization and Mapping Framework for Multi-Robot Systems - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2301.06230v3
27. Kimera-Multi: Robust, Distributed, Dense Metric-Semantic SLAM for Multi-Robot Systems, 8월 17, 2025에 액세스, https://mit.edu/sparklab/2023/08/25/Kimera-Multi__Robust_Distributed_Dense_Metric-Semantic_SLAM_for_Multi-Robot-Systems.html
28. Performance Enhancements to Visual-Inertial SLAM for Robots and Autonomous Vehicles Marcus Abate - DSpace@MIT, 8월 17, 2025에 액세스, https://dspace.mit.edu/bitstream/handle/1721.1/151684/Abate-mabate-SM-AeroAstro-2023-thesis.pdf?sequence=1&isAllowed=y
29. Kimera-Multi: a System for Distributed Multi-Robot Metric-Semantic Simultaneous Localization and Mapping | Request PDF - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/355432213_Kimera-Multi_a_System_for_Distributed_Multi-Robot_Metric-Semantic_Simultaneous_Localization_and_Mapping
30. Simultaneous Localization and Mapping (SLAM) for Autonomous Driving: Concept and Analysis - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2072-4292/15/4/1156
31. What is SLAM? A Beginner to Expert Guide - Kodifly, 8월 17, 2025에 액세스, https://kodifly.com/what-is-slam-a-beginner-to-expert-guide


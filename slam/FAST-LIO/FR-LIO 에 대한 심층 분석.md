# FR-LIO (Fast and Robust Lidar-Inertial Odometry)에 대한 심층 분석

## I. 서론: LiDAR-관성 주행 거리계(LIO)의 발전과 기술적 과제

### 0.1 자율 시스템의 근간, 정밀 측위 기술

자율주행차, 무인 항공기(UAV), 모바일 로봇과 같은 현대 자율 플랫폼 기술의 핵심에는 주변 환경을 인지하고 자신의 위치를 정확하게 파악하는 능력이 자리 잡고 있다.1 이러한 고정밀 위치 인식 및 매핑(SLAM, Simultaneous Localization and Mapping) 기술은 자율 시스템이 복잡하고 예측 불가능한 환경에서 안전하고 효율적으로 임무를 수행하기 위한 가장 근본적인 전제 조건이다. 다양한 측위 기술 중에서도 LiDAR-관성 주행 거리계(Lidar-Inertial Odometry, LIO)는 위성 항법 시스템(GPS) 신호가 도달하지 않는 실내, 도심 빌딩 숲, 터널과 같은 음영 지역에서도 강인하고 연속적인 위치 추정 정보를 제공할 수 있어 핵심 기술로 부상하였다.3

### 0.2 LIO의 기본 원리 및 핵심 과제

LIO 시스템의 기본 원리는 서로 다른 특성을 가진 두 센서, 즉 LiDAR(Light Detection and Ranging)와 IMU(Inertial Measurement Unit)의 장점을 결합하여 단점을 상호 보완하는 데 있다. LiDAR는 레이저를 이용하여 주변 환경에 대한 정밀하고 밀도 높은 3차원 형상 정보를 제공하지만, 상대적으로 낮은 데이터 수집 주기(일반적으로 10-20 Hz)를 가지며 빠른 움직임에 의해 포인트 클라우드 내부에 왜곡이 발생하기 쉽다. 반면, IMU는 가속도와 각속도를 수백 Hz 이상의 높은 주기로 측정하여 플랫폼의 동역학 정보를 제공하지만, 측정값에 포함된 노이즈와 바이어스가 시간에 따라 적분되면서 오차가 빠르게 누적되는 단점이 있다.4 LIO는 IMU의 고주파 측정값을 이용해 LiDAR 스캔 사이의 움직임을 예측하고 포인트 클라우드의 모션 왜곡을 보정하며, LiDAR의 정밀한 외부 환경 측정값을 이용해 IMU의 누적 오차를 주기적으로 보정하는 강결합(Tightly-coupled) 방식을 통해 안정적이고 정확한 궤적을 추정한다.

이러한 LIO 시스템이 궁극적으로 지향하는 목표는 **정확성(Accuracy)**, **강인성(Robustness)**, 그리고 **효율성(Efficiency)**이라는 세 가지 핵심 성능 지표를 동시에 극대화하는 것이다. 하지만 이 세 가지 목표는 종종 상충 관계에 있어, 이를 모두 만족시키는 것은 상당한 기술적 난제를 수반한다. 특히, 긴 복도나 넓은 평원과 같이 주변 환경에 유의미한 기하학적 특징이 부족한 퇴화 환경(degenerate environments)이나, 로봇 플랫폼이 급가속, 급정지, 급회전과 같은 격렬한 움직임(aggressive motion)을 보이는 상황에서 강인성을 잃지 않고 정확한 위치를 추정하는 것은 LIO 연구 분야의 가장 중요한 도전 과제 중 하나로 남아있다.6

### 0.3 기존 LIO 시스템의 접근 방식과 한계

초기 LIO 시스템의 대표주자인 LOAM(Lidar Odometry and Mapping) 2은 포인트 클라우드에서 곡률(curvature)을 기반으로 모서리(edge)점과 평면(planar)점과 같은 기하학적 특징점을 추출하고, 이들 간의 거리를 최소화하는 방식으로 위치를 추정했다. 이 접근 방식은 계산 효율성이 높고 잘 구조화된 환경에서 뛰어난 성능을 보였지만, 앞서 언급한 퇴화 환경에서는 추출할 특징점이 부족하여 성능이 급격히 저하되는 명백한 한계를 가졌다.

이러한 한계를 극복하기 위해 등장한 것이 FAST-LIO2 9와 같은 '직접(direct)' 방식의 LIO 시스템이다. 이들은 특징점 추출 단계를 생략하고 원본 포인트 클라우드(raw point cloud)를 그대로 사용하여 맵과 정합함으로써, 미세한 환경적 특징까지 모두 활용하여 퇴화 환경에서의 정확도와 강인성을 크게 향상시켰다. 그러나 이러한 진보된 시스템들조차 플랫폼의 움직임이 극도로 격렬해지는 상황, 예를 들어 초당 수백 도에 달하는 고속 회전이나 급격한 가속이 발생하는 경우, IMU 측정만으로는 LiDAR 스캔 한 프레임 내에서 발생하는 비선형적인 움직임을 완벽하게 보상하지 못한다. 이로 인해 발생하는 모션 왜곡 보상 오차는 결국 포인트 클라우드의 정합 실패와 궤적 추적 소실로 이어진다.7

LIO 기술의 발전 과정을 살펴보면, 이는 단순히 선형적인 성능 개선의 역사가 아니라, 점차 구체적이고 도전적인 '특수 상황(corner case)'을 해결하기 위한 전문화의 과정임을 알 수 있다. 초기 LOAM과 같은 시스템이 일반적인 환경에서의 LIO라는 기초를 다졌다면, FAST-LIO2와 같은 2세대 시스템은 특징이 부족한 환경이라는 1차적인 실패 지점을 해결했다. 그리고 기술이 더욱 성숙함에 따라, 연구자들은 아직 해결되지 않은 특정 실패 모드들을 공략하기 시작했다. Dynamic-LIO는 동적 객체가 많은 도심 환경에 11, PG-LIO는 기하학적 특징이 극도로 부족한 환경에 LiDAR의 강도(intensity) 값을 활용하여 대응하며 5, FR-LIO는 바로 이 '공격적인 움직임'이라는, 이전 시스템들이 다루지 못했던 중대하고 특수한 운영 영역을 목표로 한다. 따라서 FR-LIO는 모든 LIO를 대체하는 만능 시스템이 아니라, 특정 고난도 영역에서 탁월한 성능을 발휘하도록 설계된 고도로 전문화된 도구로 이해해야 한다.

### 0.4 FR-LIO의 등장 배경 및 핵심 기여점 소개

FR-LIO(Fast and Robust Lidar-Inertial Odometry)는 바로 이 '공격적 움직임'이라는 명확하고도 중요한 실패 지점을 해결하기 위해 제안된 혁신적인 LIO 시스템이다. 본 보고서는 FR-LIO가 어떻게 이 문제를 해결했는지 그 핵심 원리를 심층적으로 분석하고자 한다. FR-LIO의 핵심 기여는 두 가지로 요약할 수 있다. 첫째, 플랫폼의 움직임 강도에 따라 LiDAR 스캔을 동적으로 여러 개의 서브프레임으로 분할하고, 이로 인해 발생할 수 있는 제약 조건 약화 문제를 Tightly-Coupled Iterated Kalman **Smoother**(ESKS)라는 강력한 추정 기법을 통해 해결함으로써 극한의 운동 환경에서도 전례 없는 강인성을 확보했다. 둘째, Robocentric Voxel Map(RC-Vox)이라는 혁신적인 맵 관리 기법을 도입하여, 기존의 최첨단(SOTA, State-Of-The-Art) LIO 시스템들을 능가하는 압도적인 계산 효율성을 달성했다.7 본 보고서는 이 두 가지 핵심 기술을 중심으로 FR-LIO의 아키텍처, 알고리즘, 성능을 상세히 분석하고, LIO 기술의 미래 발전 방향 속에서 FR-LIO가 갖는 기술적 의의를 고찰할 것이다.

## 1. II. FR-LIO 프레임워크 상세 분석: 아키텍처와 핵심 알고리즘

### 1.1 A. 시스템 전체 아키텍처

FR-LIO의 시스템 아키텍처는 강인성과 속도라는 두 가지 핵심 목표를 달성하기 위해 유기적으로 설계된 데이터 처리 파이프라인으로 구성된다. 전체 데이터 흐름은 다음과 같은 단계로 이루어진다. 먼저, 외부로부터 LiDAR 포인트 클라우드와 IMU 측정값이 시스템에 입력된다. IMU 데이터는 전처리 단계를 거쳐 바이어스가 보정된 후, 상태 예측(propagation)에 사용된다. 동시에, 하나의 전체 LiDAR 스캔(sweep)은 IMU 측정값을 기반으로 한 실시간 운동 강도 분석을 통해 여러 개의 서브프레임(sub-frame)으로 적응적으로 분할된다. 이렇게 생성된 각 서브프레임은 Tightly-Coupled Iterated Error State Kalman Smoother (ESKS) 프레임워크 내에서 순차적으로 처리된다. 이 과정에서 각 서브프레임은 현재 맵과의 정합을 통해 측정 잔차(residual)를 생성하고, 이는 칼만 필터의 업데이트 단계에서 상태 변수를 보정하는 데 사용된다. 상태 추정이 완료되면, 새로운 포인트 클라우드 정보가 Robocentric Voxel Map (RC-Vox)을 이용해 맵에 효율적으로 업데이트된다. RC-Vox는 다음 서브프레임의 처리를 위해 필요한 주변 맵 포인트를 매우 빠른 속도로 쿼리하는 역할도 수행한다. 이 모든 과정을 거쳐, 시스템은 최종적으로 로봇의 실시간 궤적(trajectory) 정보를 출력한다. 이 파이프라인의 각 구성 요소는 공격적인 움직임 하에서도 안정적인 추적을 유지하고, 동시에 전체 처리 시간을 최소화하도록 긴밀하게 연동된다.7

### 1.2 B. 상태 추정: Tightly-Coupled Iterated Error State Kalman Smoother (ESKS)

#### 1.2.1  상태 벡터 및 모델의 수학적 정의

FR-LIO의 상태 추정기는 오차 상태 칼만 필터(Error State Kalman Filter, ESKF)를 기반으로 하며, 공칭 상태(nominal state)와 작은 오차 상태(error state)를 분리하여 다루는 접근 방식을 취한다. 이는 회전과 같이 비유클리드 공간에 속하는 상태를 효과적으로 처리하고 선형화 오차를 줄이는 데 유리하다. 시스템의 핵심적인 수학적 모델은 다음과 같이 정의된다.13

- **상태 벡터 (State Vector):** 시스템이 추정하고자 하는 전체 상태는 전역 좌표계(G)에서의 IMU 본체(b)의 위치(‘pGb‘), 속도(‘vGb‘), 자세(‘qGb‘)와 IMU의 가속도계 바이어스(‘ba‘), 자이로스코프 바이어스(‘bg‘), 그리고 전역 좌표계에 표현된 중력 벡터(‘gG‘)를 포함한다.
  $$
  \mathbf{x} \triangleq^T \in \mathcal{M}
  $$
  여기서 상태 공간 $`\mathcal{M}`$은 $`\mathcal{M} \triangleq \mathbb{R}^6 \times SO(3) \times \mathbb{R}^9`$로 정의되며, 자세는 3차원 회전 그룹 $`SO(3)`$에 속한다.

- **오차 상태 벡터 (Error State Vector):** 공칭 상태에 대한 작은 섭동(perturbation)을 나타내는 오차 상태 벡터는 18차원의 유클리드 공간 벡터로 정의된다. 자세 오차(‘δθb‘)는 3차원 벡터로 근사된다.
  $$
  \delta\mathbf{x} \triangleq^T \in \mathbb{R}^{18}
  $$
  **운동 모델 (Motion Model - Propagation):** IMU 측정값(‘uk‘)을 이용하여 현재 상태(‘xk‘)로부터 다음 시점의 상태(‘xk+1‘)를 예측한다. 이 과정은 공칭 상태에 대한 비선형 전파와 오차 상태에 대한 선형 전파로 나뉜다.
  $$
  \mathbf{x}_{k+1} = f(\mathbf{x}_k, \mathbf{u}_k)
  $$

  $$
  \delta\mathbf{x}_{k+1} = (\mathbf{I} + \mathbf{F}_k\Delta t)\delta\mathbf{x}_k + (\mathbf{C}_k\Delta t)\mathbf{w}
  $$

  여기서 $`f(\cdot)`$는 비선형 운동 방정식, $`\mathbf{F}_k`$와 $`\mathbf{C}_k`$는 각각 오차 상태 전파와 노이즈($`\mathbf{w}`$)에 대한 자코비안 행렬이다.

- **측정 모델 (Measurement Model - Update):** LiDAR 측정 업데이트는 현재 서브프레임의 각 포인트(‘pjl‘)가 맵 상의 가장 가까운 국소 평면(local plane)에 놓여야 한다는 기하학적 제약, 즉 point-to-plane 잔차를 최소화하는 방식으로 이루어진다. $`\kappa`$번째 반복에서의 잔차 $`h_j`$는 다음과 같이 정의된다.
  $$
  h_j(\mathbf{x}_k^\kappa) = {\mathbf{n}_j^G}^T ( \mathbf{R}(\mathbf{q}_{G}^{b_k^\kappa}) (\mathbf{R}(\mathbf{q}_b^l) \mathbf{p}_j^l + \mathbf{p}_b^l) + \mathbf{p}_{G}^{b_k^\kappa} - \mathbf{c}_j^G )
  $$
  여기서 $`\mathbf{n}_j^G`$와 $`\mathbf{c}_j^G`$는 맵에서 찾은 평면의 법선 벡터와 중심점, $`\mathbf{q}_b^l`$와 $`\mathbf{p}_b^l`$는 사전에 보정된 LiDAR-IMU 간의 외부 파라미터(extrinsic parameters)이다. 이 잔차와 잔차에 대한 자코비안 행렬 $`\mathbf{H}`$를 이용하여 칼만 게인을 계산하고 오차 상태를 업데이트한다.

#### 1.2.2  공격적 움직임 대응: 적응형 서브프레임 분할 (Adaptive Sub-frame Division)

FR-LIO가 공격적인 움직임에 강인한 가장 핵심적인 이유는 '적응형 서브프레임 분할' 전략에 있다. 전통적인 LIO 시스템은 하나의 전체 LiDAR 스캔(sweep)이 수집되는 동안(예: 100ms) 플랫폼이 등속 운동을 한다고 가정하고 모션 왜곡을 보정한다. 하지만 플랫폼이 급회전하거나 급가속하는 경우 이 가정은 깨지게 되고, 이는 심각한 왜곡 보상 오차로 이어진다.

FR-LIO는 이 문제를 해결하기 위해, IMU 측정값의 표준편차를 실시간으로 계산하여 움직임의 강도를 정량화한다. 움직임이 격렬하다고 판단되면, 하나의 전체 스캔을 시간 순서에 따라 여러 개의 작은 서브프레임으로 동적으로 분할한다.7 예를 들어, 100ms짜리 전체 스캔을 10ms짜리 서브프레임 10개로 나누는 식이다. 이렇게 짧은 시간 간격 동안에는 플랫폼이 등속 운동을 한다는 가정이 훨씬 더 잘 만족되므로, 각 서브프레임에 대한 모션 왜곡 보상이 매우 정확해진다. 이 간단하면서도 강력한 아이디어는 공격적인 움직임으로 인해 발생하는 LIO 실패의 근본 원인을 효과적으로 제거한다.7

#### 1.2.3  ESKS의 원리 및 장점

적응형 서브프레임 분할 전략은 모션 왜곡 문제를 해결하는 동시에 새로운 문제를 야기한다. 바로 '제약 조건의 약화(degradation)'이다. 하나의 서브프레임은 전체 스캔에 비해 훨씬 적은 수의 포인트만을 포함하므로, 해당 서브프레임이 우연히 기하학적으로 특징이 없는 평평한 벽이나 바닥만을 스캔했을 경우, 로봇의 6-DOF(자유도) 위치 및 자세를 완전히 결정할 만큼 충분한 기하학적 제약 조건을 제공하지 못할 수 있다.10

이러한 설계상의 트레이드오프는 FR-LIO가 왜 전통적인 칼만 **필터**가 아닌, 더 강력한 칼만 **스무더**를 채택했는지를 명확히 보여준다. 이 두 기법의 선택은 독립적인 혁신이 아니라, 하나의 문제를 해결하기 위해 다른 문제를 감수하고, 그 문제를 다시 더 고도화된 방법으로 해결하는 정교한 설계 철학의 결과물이다. 그 과정을 따라가 보면 다음과 같다.

1. **초기 문제:** 공격적인 움직임 중에는 하나의 전체 LiDAR 스캔 내에서 등속 운동 가정이 깨져 모션 왜곡 보상에 실패한다.10

2. **1차 해결책:** 스캔을 짧은 시간 단위의 서브프레임으로 분할한다. 이 짧은 시간 동안에는 등속 운동 가정이 성립하므로 왜곡 보상 오차가 줄어든다.7

3. **파생된 문제:** 분할된 서브프레임은 포인트 수가 적어 기하학적 제약 조건이 불충분할 위험(degradation)이 있다.10 전통적인 칼만 필터는 각 서브프레임을 독립적으로 처리하므로, 제약이 약한 서브프레임을 만나면 부정확하고 불확실성이 큰 상태 추정값을 내놓을 수밖에 없다.

4. **궁극적 해결책 (스무더 도입):** 칼만 스무더는 특정 시간 창(window) 내의 데이터를 한꺼번에 처리한다. 이는 현재 시점(‘k‘)의 상태를 추정할 때, 과거의 정보뿐만 아니라 미래(‘k+1,k+2,...‘)의 정보까지 활용할 수 있음을 의미한다.13 만약 

   ‘k‘ 시점의 서브프레임이 제약 조건이 약하더라도, 이후 ‘k+1‘ 시점에서 기하학적 특징이 풍부한 서브프레임이 들어온다면, 스무더는 이 '좋은' 미래 정보를 이용해 과거 ‘k‘ 시점의 불확실했던 상태 추정값을 '사후에(retrospectively)' 보정하여 훨씬 더 정확하게 다듬는다. 이 후방 스무딩(Backward Smooth) 과정은 다음 수식으로 표현된다.
   $$
   \mathbf{x}_k^s = \mathbf{x}_k + \mathbf{G}_k (\mathbf{x}_{k+1}^s - \mathbf{x}_{k+1}^-)
   $$
   여기서 $`\mathbf{x}_k^s`$는 스무딩된 상태, $`\mathbf{x}_k`$는 필터링된 상태, $`\mathbf{G}_k`$는 스무더 게인(smoother gain)이다.13

결론적으로, 서브프레임 분할과 ESKS의 채택은 서로가 서로를 필요로 하는 공생 관계에 있다. 서브프레임 분할은 스무더라는 강력한 추정기를 필요로 했고, 스무더는 서브프레임 분할이라는 전략 덕분에 공격적인 움직임이라는 난제를 해결할 수 있었다. 이는 필터의 단순함을 포기하는 대신 스무더의 강인함을 택한 전략적 선택이며, FR-LIO가 극한의 환경에서도 안정성을 유지하는 비결이다.

### 1.3 C. 맵 관리: Robocentric Voxel Map (RC-Vox)

#### 1.3.1  RC-Vox의 자료구조

FR-LIO가 달성한 또 다른 중요한 성과는 압도적인 계산 효율성이며, 그 비결은 바로 RC-Vox라는 독창적인 맵 관리 기법에 있다. RC-Vox는 로봇을 중심으로 하는(robocentric) 고정된 크기의 2계층 3D 배열(two-layer 3D array) 자료구조를 사용한다.7 이는 맵을 표현하기 위해 동적으로 크기가 변하고 복잡한 재구성 과정이 필요한 k-d 트리나 해시 테이블 기반의 접근법과는 근본적으로 다르다. RC-Vox는 로봇 주변의 일정 공간(예: 50m x 50m x 30m)을 큰 복셀(voxel)로 나누고, 각 큰 복셀은 다시 작은 복셀들의 배열을 가리키는 포인터를 갖는 2계층 구조로 되어 있다. 어떤 3차원 점이 주어졌을 때, 이 점이 속하는 복셀의 인덱스는 간단한 정수 나눗셈으로 즉시 계산될 수 있다. 이는 배열 자료구조가 가진 상수 시간($O(1)$) 접근 특성 덕분이며, 포인트 정합에 필수적인 k-최근접 이웃(k-Nearest Neighbor, k-NN) 탐색과 새로운 포인트를 맵에 추가하는 작업을 극도로 빠르게 수행할 수 있게 한다.

#### 1.3.2  기존 맵 관리 기법과의 비교

기존의 SOTA LIO 시스템들이 사용하는 맵 관리 기법과 비교했을 때 RC-Vox의 장점은 더욱 명확해진다. FAST-LIO2에서 사용하는 ikd-Tree는 점진적인 포인트 추가 및 삭제를 효율적으로 지원하지만, 트리의 균형이 깨지면 이를 다시 맞추기 위한 재균형(re-balancing) 작업에 여전히 $O(m \log m)$의 상당한 계산 비용이 소요된다.2 Faster-LIO에서 제안된 iVox는 해시 테이블을 사용하여 속도를 개선했지만, 서로 다른 위치의 포인트들이 동일한 해시 키를 갖게 되는 해시 충돌(hash collision)이 발생할 경우 성능 저하의 원인이 될 수 있다.14 RC-Vox는 이러한 복잡한 문제들을 '고정 크기 배열'이라는 단순하고 예측 가능한 자료구조를 통해 원천적으로 회피한다. 그 결과, FR-LIO는 단일 스레드 환경에서조차 다중 코어를 활용하여 병렬 처리된 경쟁 시스템들을 능가하는 경이로운 처리 속도를 보여준다.13

#### 1.3.3  로보센트릭(Robocentric) 접근의 장단점

RC-Vox의 설계는 '전역 일관성'을 희생하여 '지역 오도메트리의 속도'를 극대화하는 근본적인 트레이드오프를 보여준다. '로보센트릭'이라는 이름에서 알 수 있듯이, 이 맵은 항상 로봇을 중심으로 함께 움직이며 일정한 크기를 유지한다. 이는 전통적인 SLAM에서 궤적이 길어짐에 따라 맵의 크기가 무한히 커지고, 이로 인해 맵 탐색 시간이 점차 증가하는 계산량 폭증 문제를 영리하게 회피하는 방법이다.15 이러한 설계는 프레임 간의 상대적인 움직임을 최대한 빠르고 정확하게 계산하는 '주행 거리계(Odometry)'의 성능을 최적화하는 데 완벽하게 부합한다.

하지만 이 설계에는 필연적인 대가가 따른다. 시스템은 로봇의 현재 주변 환경 외에 과거에 방문했던 장소에 대한 정보를 본질적으로 가지고 있지 않다. 따라서, 이전에 방문했던 장소로 돌아왔을 때 이를 인식하는 '루프 폐쇄(loop closure)'를 자체적으로 수행할 수 없다. 루프 폐쇄는 장시간 운용 시 누적되는 궤적 오차(drift)를 보정하고 전역적으로 일관된 맵을 구축하는 데 필수적인 기능이다.16 결국 FR-LIO는 이름 그대로 매우 뛰어난 '주행 거리계(Odometry)' 엔진이지만, 그 자체만으로는 완전한 'SLAM' 시스템이라고 보기는 어렵다. 전역 일관성을 갖춘 맵을 만들고 그 안에서 자신의 위치를 찾는 완전한 SLAM 기능을 구현하기 위해서는, Dynamic-LIO가 백엔드로 SC-A-LOAM을 사용하는 것처럼 12, 별도의 포즈 그래프 최적화(pose graph optimization) 및 루프 폐쇄 모듈과 결합되어야 한다.

## 2. III. 성능 비교 분석: 정량적 및 정성적 평가

### 2.1 A. 공개 데이터셋을 이용한 정확도 평가

FR-LIO의 성능은 널리 사용되는 여러 공개 데이터셋을 통해 다른 최첨단(SOTA) LIO 시스템들과 비교하여 정량적으로 검증되었다. 평가에는 무인 항공기(UAV)로 수집된 NTU VIRAL, 지상 로봇으로 수집된 M2DGR, 그리고 도심 환경에서 수집된 UrbanNav 데이터셋 등이 사용되었다.13 정확도 평가는 주로 절대 궤적 오차(Absolute Trajectory Error, ATE)와 상대 궤적 오차(Relative Trajectory Error, RTE)를 지표로 사용한다.

Table 1: 궤적 오차 비교 (ATE & RTE)

아래 표는 주요 데이터셋에서 FR-LIO와 대표적인 경쟁 시스템인 FAST-LIO2, Faster-LIO, Point-LIO의 궤적 오차를 요약하여 비교한 것이다. 결과는 FR-LIO가 대부분의 시퀀스에서 기존 SOTA 시스템들과 동등하거나 더 우수한 정확도를 달성했음을 보여준다. 특히 M2DGR 데이터셋의 street 07 시퀀스와 같이 로봇이 많은 회전을 수행하는 도전적인 시나리오에서 다른 시스템 대비 눈에 띄게 낮은 오차를 기록했는데, 이는 FR-LIO의 적응형 서브프레임 분할과 ESKS 프레임워크가 공격적인 움직임을 효과적으로 처리하고 있음을 입증하는 강력한 증거이다.13

| 데이터셋 / 시퀀스 요약         | 메트릭       | FAST-LIO2     | Faster-LIO    | Point-LIO     | **FR-LIO (Ours)** |
| ------------------------------ | ------------ | ------------- | ------------- | ------------- | ----------------- |
| NTU VIRAL (eee, nya, sbs 평균) | ATE RMSE (m) | 0.140         | 0.145         | 0.183         | **0.139**         |
| M2DGR (street 01-10 평균)      | ATE RMSE (m) | 0.686         | 0.701         | 1.833         | **0.654**         |
| UrbanNav (Mongkok, TST)        | RTE (m/10m)  | 0.638 / 0.401 | 0.655 / 0.403 | 0.470 / 0.410 | **0.657 / 0.380** |

주: 위 표의 값은 13에 제시된 각 시퀀스별 결과의 경향성을 나타내기 위해 요약 또는 평균된 값이며, 특정 시퀀스에서는 순위가 변동될 수 있음.

### 2.2 B. 효율성 평가

FR-LIO의 또 다른 핵심 기여는 계산 효율성이다. RC-Vox 맵 관리 기법 덕분에 FR-LIO는 놀라운 처리 속도를 자랑한다.

**Table 2: 프레임당 처리 시간 비교**

아래 표는 여러 데이터셋에서 단일 스캔을 처리하는 데 소요되는 평균 시간을 비교한 것이다. 가장 주목할 만한 점은 FR-LIO의 단일 스레드(Single-thread) 버전이, 다중 코어를 활용하여 병렬 처리(Parallel-accelerated)된 FAST-LIO2나 Faster-LIO보다도 빠르다는 사실이다. 이는 FR-LIO의 속도가 단순히 하드웨어 자원을 더 잘 활용한 결과가 아니라, 알고리즘 자체가 근본적으로 더 효율적이라는 것을 의미하며, 이는 상당한 학술적, 기술적 성취이다.13

| 데이터셋             | FAST-LIO2 (병렬) | Faster-LIO (병렬) | **FR-LIO (단일)** |
| -------------------- | ---------------- | ----------------- | ----------------- |
| NTU VIRAL (평균, ms) | 16.3             | 11.1              | **7.9**           |
| M2DGR (평균, ms)     | 25.8             | 16.5              | **12.7**          |
| UrbanNav (평균, ms)  | 24.0             | 12.8              | **8.8**           |

**Table 3: RC-Vox 성능 기여도 분석**

FR-LIO의 이러한 압도적인 효율성이 어디에서 비롯되는지를 확인하기 위해 내부 처리 시간을 단계별로 분석한 결과는 RC-Vox의 역할을 명확히 보여준다. 아래 표는 UrbanNav 데이터셋에서의 처리 시간 분석 예시이다. 전체 처리 시간의 대부분은 상태 추정(Propagation + Update)에 소요되지만, 맵을 업데이트하고 다음 처리를 위해 맵 포인트를 삭제하는 Incremental mapping + Map delete 단계는 RC-Vox 덕분에 1ms도 채 걸리지 않는다. 이는 복잡한 트리 재구성이나 해시 연산 없이 배열 인덱싱만으로 맵 관리가 이루어지기 때문이다. 이처럼 효율적인 맵 관리는 상태 추정 단계에서 필요한 k-NN 탐색 시간까지 단축시켜 전체 시스템의 속도를 비약적으로 향상시킨다.13

| 처리 단계                         | 시간 소요 (ms, UrbanNav-Mongkok 예시) |
| --------------------------------- | ------------------------------------- |
| 서브프레임 생성                   | 1.04                                  |
| 상태 전파 및 업데이트             | 6.25                                  |
| 후방 스무딩 및 제약 분석          | 0.53                                  |
| **증분 매핑 및 맵 삭제 (RC-Vox)** | **0.47**                              |
| 총 실행 시간                      | 8.31                                  |

### 2.3 C. 강인성 테스트: 공격적 움직임 시나리오

FR-LIO의 진정한 가치는 공개 데이터셋의 수치 비교를 넘어, 다른 시스템들이 실패하는 극한의 환경에서 드러난다. 저자들은 자체 제작한 데이터셋을 통해 최대 각속도 21.8 rad/s (약 1250 deg/s)에 달하는 극심한 회전 환경에서 시스템을 테스트했다.7 이러한 조건에서 FAST-LIO2와 같은 유수의 시스템들은 모션 왜곡을 감당하지 못하고 궤적 추적에 실패하여 맵이 심하게 왜곡되거나 소실되는 결과를 보였다. 반면, FR-LIO는 동일한 상황에서도 안정적으로 궤적을 추정하고 일관성 있는 맵을 성공적으로 생성해냈다. 이는 적응형 서브프레임 분할과 ESKS 스무딩의 조합이 이론적으로만 우수한 것이 아니라, 실제 물리적 한계에 가까운 격렬한 움직임 속에서도 효과적으로 작동함을 보여주는 가장 강력하고 정성적인 증거이다.7

## 3. IV. LIO 기술 동향 속 FR-LIO의 위상과 의의

### 3.1 LIO 시스템의 다변화와 전문화

현대의 LIO 연구 동향은 모든 시나리오에 완벽한 단일 만능 시스템을 개발하기보다는, 특정 환경이나 주요 실패 원인에 대응하기 위한 전문화된 솔루션을 개발하는 방향으로 나아가고 있다. 이러한 전문화된 시스템들은 LIO 기술의 적용 범위를 더욱 넓히고 강인성을 한 차원 높은 수준으로 끌어올리고 있다.

- **기하학적 퇴화 환경 대응:** PG-LIO와 같은 시스템은 터널, 평원, 안개 낀 환경처럼 기하학적 특징이 부족하여 전통적인 LIO가 실패하는 상황에 주목한다. 이 시스템은 LiDAR가 수집하는 3D 위치 정보에 더해, 각 포인트가 반사하는 레이저의 강도(intensity) 또는 반사율(reflectivity) 정보를 추가적인 제약 조건으로 활용한다. 이는 기하학적 구조가 아닌 텍스처(texture) 정보를 통해 위치를 추정하는 것과 유사하여, 기하학적으로는 동일해 보이는 환경에서도 강인성을 유지할 수 있게 한다.5
- **동적 환경 대응:** Dynamic-LIO는 복잡한 도심 주행 시나리오에서 LIO의 성능을 저하하는 주된 요인인 움직이는 차량이나 보행자와 같은 동적 객체(dynamic objects) 문제에 집중한다. 이 시스템은 현재 스캔된 포인트와 기존 맵 데이터 간의 일관성을 검사하여 동적 객체로 판단되는 포인트들을 실시간으로 식별하고 제거한다. 이를 통해 오염된 측정값이 상태 추정에 미치는 악영향을 최소화하여, 동적인 환경에서도 정확한 궤적을 유지한다.11
- **센서 융합을 통한 강인성 확보:** LIR-LIVO와 같은 다중 센서 융합 시스템은 특정 센서가 약점을 보이는 환경을 다른 센서로 보완하는 전략을 취한다. 예를 들어, 어두운 환경에서는 카메라 성능이 저하되고, 특징 없는 흰 벽 앞에서는 LiDAR 성능이 저하될 수 있다. LiDAR, 카메라(Vision), IMU를 강결합으로 융합함으로써, 한 센서의 정보가 불확실해지더라도 다른 센서들의 정보를 통해 안정적인 위치 추정을 이어갈 수 있다.17 또한, SR-LIO는 LiDAR 스캔을 시간적으로 재구성(Sweep Reconstruction)하여 LIO 시스템의 업데이트 주파수를 스캔 주파수 이상으로 높임으로써, IMU 예측 오차의 누적을 줄여 정확도를 개선하는 독자적인 접근법을 제시한다.18

### 3.2 FR-LIO의 독자적 위상

이러한 다양한 전문 시스템들과 비교할 때, FR-LIO는 '플랫폼 자체의 격렬한 움직임'이라는 매우 구체적이고 중요한 문제에 대해 가장 효과적이고 완성도 높은 해결책을 제시하는 시스템으로서 독자적인 위상을 확고히 하고 있다. 다른 시스템들이 외부 환경(퇴화, 동적 객체)이나 센서의 한계에 초점을 맞출 때, FR-LIO는 플랫폼 내부의 동역학적 한계 상황을 정면으로 돌파했다.

이러한 전문화된 시스템들은 서로 배타적인 경쟁 관계에 있는 것이 아니라, 상호 보완적인 도구 상자(toolbox)를 구성하는 것으로 이해해야 한다. 미래의 가장 강인한 자율 시스템은 아마도 이러한 전문 모듈들을 통합한 형태가 될 것이다. 예를 들어, 도심을 고속으로 주행하는 자율주행차를 위한 '궁극의 LIO' 시스템을 상상해 본다면, 이는 FR-LIO의 격렬한 움직임 처리 능력, Dynamic-LIO의 동적 객체 필터링 기능, 그리고 만일의 사태를 대비한 PG-LIO의 광도 정보 활용 능력을 모두 갖춘 하이브리드 시스템일 가능성이 높다. 이 시나리오에서 FR-LIO는 다른 어떤 시스템도 대체할 수 없는 '고기동성 대응'이라는 필수 모듈을 제공한다. 따라서 FR-LIO의 등장은 LIO 기술의 모듈화 및 조합을 통한 성능 극대화라는 미래 발전 방향에 중요한 초석을 놓은 것으로 평가할 수 있다.

## 4. V. 비판적 고찰 및 향후 전망

### 4.1 A. FR-LIO의 내재적 한계 및 잠재적 단점

FR-LIO는 특정 문제에 대해 탁월한 해결책을 제시하지만, 그 설계 철학상 몇 가지 내재적인 한계 또한 가지고 있다.

- **전역 일관성 문제:** 가장 명백한 한계는 RC-Vox의 로보센트릭 설계에서 비롯된다. 이 설계는 지역 오도메트리의 속도를 극대화하는 대신, 전역 맵 정보를 의도적으로 소실시킨다. 따라서 FR-LIO는 장시간, 대규모 환경을 운용할 때 필연적으로 누적되는 궤적 오차를 보정할 수 있는 루프 폐쇄(loop closure) 및 전역 포즈 그래프 최적화(global pose graph optimization) 기능이 내장되어 있지 않다.16 이는 FR-LIO가 그 자체만으로는 완전한 SLAM 시스템이 아닌, 고성능 '오도메트리' 시스템임을 의미한다.
- **파라미터 튜닝의 민감성:** 시스템의 핵심 기능 중 일부는 수동으로 설정해야 하는 파라미터에 의존한다. 예를 들어, 적응형 서브프레임 분할은 움직임의 강도를 판단하는 가속도 및 각속도 표준편차의 임계값에 따라 그 동작이 달라진다.13 또한, RC-Vox의 성능과 메모리 사용량은 복셀의 크기 설정에 민감하게 반응할 수 있다.20 이러한 파라미터들은 사용되는 LiDAR 센서의 종류, IMU의 사양, 그리고 주행하는 환경의 특성에 따라 최적의 값을 찾기 위한 튜닝 과정이 필요할 수 있다.
- **기하학적 퇴화 문제의 잔존:** FR-LIO는 플랫폼의 '움직임'으로 인한 문제에 대해서는 매우 강인하지만, 위치 추정의 근본적인 단서는 여전히 환경의 '기하학적 정보'에 의존한다. 따라서, 끝이 보이지 않는 긴 터널이나 복도, 특징 없는 넓은 평원과 같이 극단적으로 기하학적 제약 조건이 부족한 퇴화 환경에 장시간 머무를 경우, 다른 LIO 시스템과 마찬가지로 궤적 오차가 누적되거나 추정에 실패할 가능성을 완전히 배제할 수는 없다.6

### 4.2 B. LIO 연구의 미래 방향

FR-LIO와 같은 선구적인 연구들을 바탕으로, 미래의 LIO 기술은 다음과 같은 방향으로 더욱 발전해 나갈 것으로 전망된다.

- **다중 센서 및 의미론적 정보 융합:** LiDAR와 IMU의 융합을 넘어, 카메라, 레이더, 그리고 이벤트 카메라와 같은 이종 센서들과의 더욱 깊고 강건한 융합이 중요한 연구 주제가 될 것이다.3 특히, 딥러닝 기술을 활용하여 환경 내의 객체들을 의미론적으로 이해하고(semantic information), 이를 LIO 프레임워크에 통합하는 연구가 활발해질 것이다. 예를 들어, '자동차'나 '사람'과 같이 움직일 가능성이 높은 객체와 '건물'이나 '도로'와 같이 정적인 구조물을 의미적으로 구분하여, 동적 객체는 필터링하고 정적 구조물만을 위치 추정에 활용하는 방식은 LIO의 강인성을 한 차원 더 높은 수준으로 끌어올릴 수 있다.11
- **경량화 및 엣지 컴퓨팅 (SWaP 제약):** 소형 드론, 웨어러블 기기, 휴머노이드 로봇과 같이 크기(Size), 무게(Weight), 전력(Power)의 제약이 극심한 플랫폼(SWaP-constrained platforms)에 LIO를 탑재하기 위한 경량화 연구가 더욱 중요해질 것이다.22 QLIO와 같이 측정 데이터와 잔차를 양자화(quantization)하거나, 환경에 따라 포인트 클라우드를 적응적으로 샘플링하여 처리해야 할 데이터의 양과 계산량을 획기적으로 줄이는 연구가 대표적인 예이다.22
- **장기 운용 및 라이프롱 SLAM:** 현재의 LIO 시스템들은 대부분 사전에 정의된 특정 임무 동안 작동하는 것을 가정한다. 그러나 미래의 로봇은 변화하는 환경 속에서 장기간에 걸쳐 안정적으로 작동해야 한다. 이를 위해, 시간이 지남에 따라 변하는 환경(예: 계절의 변화, 새로운 건물의 건설)에 적응하여 맵을 지속적으로 업데이트하고, 과거의 경험을 통해 학습하며, 시간이 지나도 맵의 일관성을 유지하는 라이프롱(Lifelong) SLAM 기술이 LIO 연구의 궁극적인 목표 중 하나가 될 것이다.16

FR-LIO의 아키텍처는 이러한 미래 기술 동향에 중요한 시사점을 제공한다. 특히, 빠른 지역 오도메트리 계산과 전역 맵 관리를 분리한 설계는 미래 로봇 시스템에서 보편화될 분산 컴퓨팅 아키텍처의 초기 모델로 볼 수 있다. SWaP 제약이 있는 로봇(엣지 디바이스)은 FR-LIO의 핵심 엔진과 같은 경량 오도메트리를 실행하여 실시간 제어와 장애물 회피를 수행한다. 동시에, 압축된 맵 정보나 주요 프레임(keyframe)을 주기적으로 강력한 성능의 클라우드 서버나 지상 관제소(백엔드)로 전송한다. 백엔드는 이 데이터를 받아 전역 포즈 그래프 최적화, 루프 폐쇄, 의미론적 맵 생성과 같이 계산 비용이 높은 작업을 비실시간으로 처리한다. 이러한 구조에서 FR-LIO의 오도메트리 모듈은 엣지 단에서 실시간 항법을 책임지는 이상적인 컴포넌트가 된다. 따라서 FR-LIO는 단순히 빠른 오도메트리 시스템을 넘어, 그 설계 철학 자체가 미래 자율 시스템의 발전 방향과 깊이 연관되어 있어 앞으로도 오랫동안 그 개념과 구성 요소들이 중요한 참고 자료가 될 것이다.

## 5. VI. 결론

본 보고서는 FR-LIO(Fast and Robust Lidar-Inertial Odometry)가 LIO 기술 분야에서 이룩한 두 가지 핵심적인 성과를 심층적으로 분석했다. 첫째, FR-LIO는 플랫폼의 움직임 강도에 따라 LiDAR 스캔을 동적으로 분할하는 '적응형 서브프레임' 전략과, 과거 상태를 미래 정보로 보정하는 'ESKS(Iterated Error State Kalman Smoother)'를 유기적으로 결합하였다. 이를 통해 기존의 최첨단 시스템들이 추적에 실패하던 극심한 고속 회전 및 가속 환경, 즉 '공격적인 움직임' 속에서도 전례 없는 수준의 강인성을 달성했다. 둘째, 로봇 중심의 고정 크기 배열 구조를 사용하는 'RC-Vox(Robocentric Voxel Map)'라는 혁신적인 맵 관리 기법을 도입하여, 복잡한 자료구조에 의존하던 기존 시스템들의 계산 효율성을 뛰어넘는 새로운 기준을 제시했다.

FR-LIO의 기술적 가치와 의의는 단순히 학술적 기여에만 머무르지 않는다. 이는 LIO 기술이 해결해야 할 핵심 난제 중 하나인 '고기동성' 문제를 정면으로 돌파함으로써, 고속으로 비행하는 드론, 한계 속도에 도전하는 자율 레이싱 차량, 신속한 초기 대응이 필수적인 재난 구조 로봇 등, 이전에는 안정적인 운용이 어려웠던 더 넓고 도전적인 응용 분야로 자율 이동체의 활동 영역을 확장하는 실질적인 가치를 지닌다. FR-LIO가 제시한 강인한 상태 추정 기법과 효율적인 맵 관리 패러다임은 향후 개발될 차세대 LIO 시스템에 중요한 영감을 제공하며, 자율 이동 로봇 기술의 발전에 있어 하나의 중요한 이정표가 될 것이다.

#### **참고 자료**

1. Towards Robust Sensor-Fusion Ground SLAM: A Comprehensive Benchmark and A Resilient Framework - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2507.08364v1
2. VOX-LIO: An Effective and Robust LiDAR-Inertial Odometry System Based on Surfel Voxels, 8월 2, 2025에 액세스, https://www.mdpi.com/2072-4292/17/13/2214
3. A Comprehensive Evaluation of LiDAR Odometry Techniques - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/393924022_A_Comprehensive_Evaluation_of_LiDAR_Odometry_Techniques
4. RTLIO: Real-Time LiDAR-Inertial Odometry and Mapping for UAVs - PMC, 8월 2, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8226800/
5. PG-LIO: Photometric-Geometric fusion for Robust LiDAR-Inertial Odometry - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2506.18583v1
6. A Tightly Coupled LiDAR-Inertial SLAM for Perceptually Degraded Scenes - MDPI, 8월 2, 2025에 액세스, https://www.mdpi.com/1424-8220/22/8/3063
7. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
8. Robust Lidar-Inertial Odometry with Ground Condition Perception and Optimization Algorithm for UGV - MDPI, 8월 2, 2025에 액세스, https://www.mdpi.com/1424-8220/22/19/7424
9. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/pdf/2107.06829
10. Fast and Robust LiDAR-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/380301592_Fast_and_Robust_LiDAR-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
11. LiDAR-Inertial Odometry in Dynamic Driving Scenarios using Label Consistency Detection, 8월 2, 2025에 액세스, https://arxiv.org/html/2407.03590v2
12. ZikangYuan/dynamic_lio: [IROS 2025] A LiDAR-inertial odometry for dynamic environments, 8월 2, 2025에 액세스, https://github.com/ZikangYuan/dynamic_lio
13. [2302.04031] FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/2302.04031
14. SS-LIO: Robust Tightly Coupled Solid-State LiDAR–Inertial Odometry for Indoor Degraded Environments - MDPI, 8월 2, 2025에 액세스, https://www.mdpi.com/2079-9292/14/15/2951
15. Robocentric SLAM - University of Texas at Austin, 8월 2, 2025에 액세스, https://sites.utexas.edu/renato/files/2024/01/main-1.pdf
16. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, 8월 2, 2025에 액세스, https://www.mathworks.com/discovery/slam.html
17. LIR-LIVO: A Lightweight,Robust LiDAR/Vision/Inertial Odometry with Illumination-Resilient Deep Features - arXiv, 8월 2, 2025에 액세스, [https://arxiv.org/pdf/2502.08676?](https://arxiv.org/pdf/2502.08676)
18. ZikangYuan/sr_lio: [IROS 2024] A LiDAR-inertial odometry (LIO) package that can adjust the execution frequency beyond the sweep frequency - GitHub, 8월 2, 2025에 액세스, https://github.com/ZikangYuan/sr_lio
19. A Review of Visual-LiDAR Fusion based Simultaneous Localization and Mapping - PMC, 8월 2, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC7181037/
20. Faster-LIO: Lightweight Tightly Coupled Lidar-inertial Odometry using Parallel Sparse Incremental Voxels - GitHub, 8월 2, 2025에 액세스, https://github.com/gaoxiang12/faster-lio
21. Strong Robust LIO System Based on Event Camera Assistance - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/385064798_Strong_Robust_LIO_System_Based_on_Event_Camera_Assistance
22. QLIO: Quantized LiDAR-Inertial Odometry - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2503.07949v1


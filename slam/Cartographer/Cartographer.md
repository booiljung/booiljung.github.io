# Cartographer에 대한 고찰
[Cartographer](./index.md)



Cartographer는 구글(Google)이 2016년 10월에 오픈소스로 공개한 실시간 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 라이브러리다.1 이 시스템은 2D 및 3D 환경 모두에서 작동하며, 다양한 플랫폼과 센서 구성에 걸쳐 적용될 수 있도록 설계되었다.2 Cartographer의 개발 목적은 학계와 산업계에서 널리 사용되는 센서 구성을 활용하여, 실시간으로 전역적으로 일관된(globally consistent) 지도를 구축하는 데 있었다.2 특히, 개발 초기부터 구글은 독일 박물관(Deutsches Museum)과 같은 대규모 실내 공간을 매핑하기 위해 센서를 장착한 백팩 형태의 휴대용 플랫폼을 운용했으며, 이는 제한된 컴퓨팅 자원 하에서도 실시간 성능을 보장해야 하는 목표를 명확히 보여준다.2

이 시스템의 핵심은 C++로 작성된 독립형 라이브러리이며, 로봇 운영체제(Robot Operating System, ROS)와의 원활한 통합을 위한 `cartographer_ros` 패키지를 제공함으로써 사용 편의성을 극대화했다.1 이러한 접근성 덕분에 TurtleBot, Toyota HSR, PR2와 같은 다양한 상용 및 연구용 로봇 플랫폼에 신속하게 적용될 수 있었다.2


Cartographer의 가장 근본적인 설계 철학은 '실시간성'과 '정확성'이라는 상충될 수 있는 두 가지 목표를 동시에 달성하기 위한 이중적 아키텍처에 있다. 이 시스템은 계산 과정을 두 개의 독립적이면서도 상호 연관된 하위 시스템, 즉 로컬 SLAM(Local SLAM, 프론트엔드)과 글로벌 SLAM(Global SLAM, 백엔드)으로 분리한다.6

이러한 분리 구조는 계산 부하를 시간적으로 분산시키는 영리한 전략이다. 로봇이 이동하는 동안 즉각적으로 필요한 위치 정보는 로컬 SLAM이 빠르게 계산하여 제공한다. 이 계산은 오차가 누적될 수 있지만, 즉각적인 반응성을 보장한다. 그 사이, 더 많은 계산 자원을 요구하는 전역 최적화 작업은 글로벌 SLAM이 백그라운드에서 비동기적으로 처리한다. 이 과정은 시간이 걸리지만 로봇의 실시간 동작에는 영향을 미치지 않는다. 결과적으로 사용자는 시스템이 끊김 없이 실시간으로 작동하는 것처럼 느끼면서도, 최종적으로는 누적 오차가 보정된 고품질의 전역 지도를 얻게 된다. 이 아키텍처는 Cartographer의 성능뿐만 아니라, 파라미터 튜닝 철학까지 지배하는 가장 핵심적인 설계 원리라 할 수 있다.

- **로컬 SLAM (Local SLAM / Frontend):** 이 하위 시스템은 프론트엔드 또는 로컬 궤적 빌더(local trajectory builder)라고도 불린다.6 주된 임무는 실시간으로 센서 데이터를 처리하여 연속적인 서브맵(submap)을 생성하는 것이다. 각 서브맵은 그 자체로 지역적으로는 매우 일관된 지도를 형성한다. 하지만 로컬 SLAM은 제한된 정보만을 바탕으로 빠르게 포즈를 추정하므로, 시간이 지남에 따라 오차(드리프트, drift)가 필연적으로 누적된다.4 이 단계의 핵심은 속도를 위해 전역적 정확도를 일부 희생하는 것이다.
- **글로벌 SLAM (Global SLAM / Backend):** 백엔드라고도 불리는 이 하위 시스템은 별도의 백그라운드 스레드에서 실행된다.6 주된 역할은 로컬 SLAM이 생성한 여러 서브맵들을 하나의 일관된 전역 지도로 정렬하는 것이다. 이를 위해 루프 클로저(loop closure) 제약을 탐지하고, 이를 포즈 그래프(pose graph)에 추가하여 전체 맵의 누적된 오차를 최적화 기법으로 보정한다.8 이 단계는 정확성을 위해 시간을 투자하는 과정이다.


Cartographer는 다음과 같은 주요 특징을 통해 기존 SLAM 기술들과 차별화된다.

- **2D/3D 지원 및 3D 인지 2D SLAM:** Cartographer는 2D와 3D 환경 모두에서 SLAM을 수행할 수 있는 유연성을 제공한다.1 특히 주목할 만한 점은 2D SLAM을 수행할 때에도 3D 세계를 인지한다는 것이다. 예를 들어, 로봇이 경사로를 오르내릴 때 LiDAR 센서가 기울어지더라도, Cartographer는 IMU 데이터를 활용하여 이 기울기를 보정하고 스캔 데이터를 수평면에 정확하게 투영한다.11 이는 센서가 항상 완벽하게 수평을 유지해야 한다는 제약을 가진 GMapping과 같은 전통적인 2D SLAM 알고리즘에 비해 훨씬 높은 강건성을 제공한다.
- **실시간 루프 클로저:** 로봇이 이전에 방문했던 장소를 다시 지나갈 때, 시스템은 이를 실시간으로 인식하고 맵의 불일치(드리프트)를 즉시 수정한다.2 이는 마치 긴 여행 후 출발점으로 돌아왔을 때 전체 경로의 오차를 한 번에 바로잡는 것과 같다. 이러한 실시간성은 후술할 분기 한정법(Branch-and-Bound)과 같은 고효율 검색 알고리즘을 통해 달성되며, 사용자가 매핑 과정을 직관적으로 이해하고 신뢰할 수 있게 만드는 핵심 기능이다.4
- **다양한 센서 융합:** Cartographer는 단일 센서에 의존하지 않고 LiDAR, IMU, 주행 거리계(Odometry), 카메라 등 여러 센서로부터 들어오는 데이터를 유기적으로 융합하여 위치 추정의 정확성과 강건성을 극대화한다.2 예를 들어, LiDAR는 정밀한 기하학적 정보를 제공하고, IMU는 회전과 중력 방향에 대한 신뢰도 높은 정보를 제공하여 상호 보완적으로 작동한다. 특히 3D SLAM에서는 지면과 중력 방향을 파악하는 것이 매우 중요하므로 IMU 데이터의 사용이 사실상 필수적이다.14
- **오픈소스 현황 및 한계:** 2024년 기준, 구글의 공식 `cartographer` GitHub 저장소는 더 이상 활발하게 유지보수되지 않고 있다. README 파일에는 "On rare occasions critical pull requests may be merged, but no new development is currently taking place"라고 명시되어 있으며, 이는 새로운 기능 추가나 적극적인 버그 수정이 이루어지지 않음을 의미한다.3 ROS 사용자들이 주로 이용하는 `cartographer_ros` 패키지는 커뮤니티에 의해 포크(fork)되어 제한된 수준에서 유지보수가 이루어지고 있다.3 이는 Cartographer를 상용 제품이나 장기 연구 프로젝트에 적용하고자 할 때, 잠재적인 기술 지원의 부재와 유지보수의 책임을 사용자가 직접 감당해야 한다는 점을 시사하는 중요한 고려사항이다.


Cartographer의 성능은 '최적화 문제'를 얼마나 잘 정의하고 강건하며 빠르게 푸느냐에 달려있다. 로컬 SLAM의 Ceres 스캔 매칭과 글로벌 SLAM의 포즈 그래프 최적화는 모두 비선형 최소제곱 문제라는 동일한 수학적 프레임워크를 공유한다. 하지만 전자는 '속도'를 위해 작은 문제(스캔 대 서브맵)를 실시간으로 풀고, 후자는 '정확성'을 위해 큰 문제(전체 포즈 그래프)를 백그라운드에서 푼다. 이 과정에서 Huber 손실 함수와 분기 한정법은 현실 세계의 노이즈와 계산 제약 하에서 최적화 과정이 '강건하게' 그리고 '빠르게' 작동하도록 만드는 핵심적인 공학적 장치다. 따라서 Cartographer의 아키텍처는 단순히 로컬과 글로벌로 나뉜 것이 아니라, '최적화'라는 일관된 수학적 틀 안에서 '강건성'과 '효율성'이라는 현실적 제약을 해결하기 위한 정교한 장치들이 유기적으로 결합된 시스템으로 이해해야 한다.


로컬 SLAM의 목표는 드리프트가 누적되는 것을 감수하더라도, 실시간으로 들어오는 센서 데이터를 처리하여 지역적으로 일관된 지도 조각, 즉 서브맵을 빠르고 정확하게 만들어내는 것이다.


로봇의 센서로부터 수집된 원시 데이터는 SLAM 알고리즘에 직접 사용되기 전에 여러 단계의 전처리 과정을 거친다.

- **범위 필터링 (Range Filtering):** LiDAR와 같은 거리 측정 센서는 측정 가능 범위를 벗어나는 매우 가깝거나 매우 먼 거리에서 노이즈를 포함할 수 있다. Cartographer는 이러한 노이즈를 제거하기 위해 사용자가 지정한 최소 및 최대 거리(`min_range`, `max_range`) 사이의 유효한 측정값만 남기는 밴드패스 필터를 가장 먼저 적용한다.6
- **다운샘플링 (Downsampling):** 고해상도 LiDAR는 초당 수십만 개의 포인트를 생성하여 엄청난 계산 부하를 유발할 수 있다. 이를 해결하기 위해 Cartographer는 복셀 그리드 필터(Voxel Grid Filter)를 사용하여 포인트 클라우드의 밀도를 낮춘다. 이 필터는 3D 공간을 작은 정육면체(복셀)로 나눈 뒤, 각 복셀 내에 포함된 모든 포인트들의 무게 중심(centroid) 하나만을 남겨 전체 포인트 수를 줄인다.9 Cartographer는 고정된 크기의 복셀 필터뿐만 아니라, 필터링 후 목표 포인트 수를 유지하기 위해 복셀의 크기를 동적으로 조절하는 적응형 복셀 필터(Adaptive Voxel Filter)도 사용하여 데이터의 밀도에 따라 유연하게 대응한다.6
- **IMU 데이터 처리:** IMU는 로봇의 각속도와 선형 가속도를 측정하여 중력 방향과 로봇의 회전 상태에 대한 중요한 정보를 제공한다. 하지만 IMU 센서 자체의 노이즈가 심할 수 있으므로, Cartographer는 일정 시간 동안의 IMU 측정값을 필터링하여 안정적인 중력 방향 벡터를 추정한다.6 이 정보는 특히 3D SLAM에서 스캔 데이터를 올바른 방향으로 정렬하는 데 결정적으로 사용된다. 2D SLAM에서는 IMU 사용이 선택 사항이지만, 사용하는 경우 경사로나 고르지 않은 지면에서도 더 강건한 성능을 보인다.6


로컬 SLAM의 핵심 출력물은 일련의 서브맵(submap)이다.

- **서브맵의 정의:** 서브맵은 여러 개의 연속적인 스캔(scan) 데이터가 누적되어 만들어진 작은 크기의 확률 격자 지도(probability grid map)다.4 각 서브맵은 그 자체로 완결된 작은 지도로, 지역적으로는 매우 정확하고 일관성이 높다.
- **확률 격자 업데이트:** 새로운 스캔 데이터가 들어오면, 이는 현재 구축 중인 서브맵에 통합된다. 각 스캔 포인트는 레이저가 도달한 '히트(hit)' 지점과, 그 지점까지 레이저가 방해받지 않고 통과한 '미스(miss)' 영역으로 구성된다. 서브맵의 각 격자 셀은 이 히트와 미스 정보를 기반으로 점유 확률(occupancy probability)을 갱신한다. Cartographer는 확률을 직접 다루는 대신, 계산적으로 더 효율적이고 안정적인 오즈(odds) 형태로 변환하여 업데이트를 수행한다.4
  - 어떤 사건의 확률이 $p$일 때, 오즈는 $\text{odds}(p) = p / (1 - p)$로 정의된다.
  - 새로운 측정값(히트 또는 미스)에 대한 확률($p_{hit}$, $p_{miss}$)이 주어지면, 기존 격자 셀의 값 $M_{old}(x)$는 $M_{new}(x) = \text{clamp}(\text{odds}^{-1}(\text{odds}(M_{old}(x)) \cdot \text{odds}(p_{hit/miss})))$ 와 같이 갱신된다.4
- **서브맵의 완성:** 하나의 서브맵은 설정된 개수(`submaps.num_range_data`)만큼의 스캔 데이터를 수신하면 '완성(finished)'된 것으로 간주된다.6 완성된 서브맵은 더 이상 새로운 스캔이 추가되지 않으며, 이후 글로벌 SLAM 단계에서 루프 클로저 탐지를 위한 불변의 대상으로 사용된다.4


스캔 매칭은 새로운 스캔 데이터를 현재 구축 중인 서브맵에 가장 잘 정합시키는 최적의 포즈(위치와 방향)를 찾는 과정이다. Cartographer는 이 과정을 위해 두 가지 전략을 제공한다.

- **Ceres 스캔 매처 (CeresScanMatcher):** 이것이 Cartographer의 기본이자 핵심적인 스캔 매칭 알고리즘이다.6

  - **작동 원리:** 포즈 추정기(IMU와 주행 거리계 데이터를 통합한 extrapolator)가 예측한 대략적인 초기 포즈를 사전 확률(prior)로 사용한다. 그 다음, 이 초기 위치 주변에서 스캔 데이터가 서브맵의 확률 격자 값과 가장 잘 일치하는 미세한 위치 조정을 찾는다. 이 과정은 비선형 최소제곱 문제(non-linear least squares problem)로 모델링되며, 구글에서 개발한 최적화 라이브러리인 Ceres Solver를 통해 해를 구한다.6

  - **비용 함수 (Cost Function):** 최적화의 목표는 비용 함수를 최소화하는 포즈 $\xi$를 찾는 것이다. 비용 함수는 변환된 스캔의 각 포인트 $h_k$가 서브맵 상에서 차지된 공간(높은 확률 값)에 위치하도록 정의된다. 구체적인 비용 함수는 다음과 같다 4:

    Code snippet

    ```
    $$\underset{\xi}{\text{argmin}} \sum_{k=1}^{K} (1 - M_{\text{smooth}}(T_{\xi}h_k))^2$$
    ```

    여기서 $\xi = (\xi_x, \xi_y, \xi_{\theta})$는 최적화할 스캔의 포즈, $h_k$는 스캔 프레임에서의 k번째 포인트, $T_{\xi}$는 포즈 $\xi$에 의한 변환 행렬이다. $M_{\text{smooth}}$는 서브맵의 이산적인 확률 값을 연속적인 함수로 만든 것으로, 쌍삼차 보간법(bicubic interpolation)을 사용하여 서브픽셀(sub-pixel) 수준의 정밀한 매칭을 가능하게 한다. 이 비용 함수는 스캔 포인트가 점유 확률 1인 곳에 위치할 때 0이 되어 최소화된다.

  - **특징:** 이 방식은 매우 빠르고 효율적이지만, 초기 추정값이 실제 위치와 크게 벗어난 경우에는 지역 최솟값(local minimum)에 빠져 정확한 위치를 찾지 못할 수 있다.6 따라서 IMU나 주행 거리계와 같이 신뢰할 수 있는 다른 센서 정보가 있을 때 가장 좋은 성능을 보인다.

- **실시간 상관 스캔 매처 (RealTimeCorrelativeScanMatcher):**

  - **작동 원리:** 이 매처는 IMU나 주행 거리계 데이터가 없거나 신뢰할 수 없을 때를 위한 강건한 대안이다.6 루프 클로저에서 사용하는 방식과 유사하게, 현재 스캔과 현재 서브맵 간의 상관관계(correlation)를 넓은 검색 범위에 걸쳐 계산하여 가장 높은 상관관계를 보이는 위치를 찾는다.
  - **특징:** 계산 비용이 매우 높기 때문에 실시간성을 저해할 수 있지만, 초기 추정값이 크게 틀려도 올바른 위치를 찾아낼 수 있는 강건함을 제공한다. 이 매처를 통해 찾은 최적의 위치는 다시 Ceres 스캔 매처의 초기 추정값으로 전달되어 최종적인 미세 조정을 거치게 된다.6

### B. 글로벌 SLAM (백엔드): 전역적 일관성을 위한 최적화

글로벌 SLAM의 목표는 로컬 SLAM에서 생성된, 드리프트가 누적된 서브맵들을 모아 전역적으로 일관된 하나의 지도를 완성하는 것이다. 이 과정은 백그라운드에서 주기적으로 실행된다.

#### 1. 포즈 그래프 최적화

글로벌 SLAM의 수학적 기반은 포즈 그래프 최적화(Pose Graph Optimization)다.7

- **포즈 그래프의 구성:**

  - **노드 (Nodes):** 그래프의 각 노드는 특정 시점의 로봇 포즈(스캔 포즈) 또는 서브맵의 포즈를 나타낸다.
  - **엣지 (Edges):** 노드들을 연결하는 엣지는 두 노드 간의 상대적인 위치 관계, 즉 제약(constraint)을 의미한다. 이 제약은 스캔 매칭을 통해 얻어진 상대 포즈 측정값($\hat{\xi}_{ij}$)과 그 측정의 불확실성을 나타내는 공분산 행렬($\Sigma_{ij}$)로 구성된다.
  - **제약의 종류:**
    1. **연속 제약 (Sequential Constraints):** 로컬 SLAM에서 연속적으로 생성된 스캔들 사이의 제약. 이는 주행 거리계 정보와 유사한 역할을 한다.
    2. **루프 클로저 제약 (Loop Closure Constraints):** 로봇이 이전에 방문했던 장소를 재방문했을 때, 현재 스캔과 과거의 서브맵 사이에 생성되는 제약. 이 제약은 시간적으로 멀리 떨어진 노드들을 연결하여 누적된 드리프트를 극적으로 줄이는 역할을 한다.

- **희소 포즈 조정 (Sparse Pose Adjustment, SPA):** Cartographer는 모든 스캔과 서브맵의 포즈를 전역적으로 최적화하기 위해 희소 포즈 조정(SPA)이라는 접근 방식을 따른다.4 최적화 문제는 모든 제약 조건(엣지)들을 가장 잘 만족시키는 노드들의 포즈 집합(

  $\Xi^m = \{\xi_{m_i}\}$, $\Xi^s = \{\xi_{s_j}\}$)을 찾는 비선형 최소제곱 문제로 공식화된다.
  $$
  \underset{\Xi^m, \Xi^s}{\text{argmin}} \frac{1}{2} \sum_{ij} \rho(E^2(\xi_{m_i}, \xi_{s_j}; \Sigma_{ij}, \hat{\xi}_{ij}))
  $$
  여기서 $\xi_{m_i}$와 $\xi_{s_j}$는 각각 최적화할 서브맵 $i$와 스캔 $j$의 전역 포즈다. $E^2$는 측정된 상대 포즈 $\hat{\xi}_{ij}$와 현재 추정된 전역 포즈들($\xi_{m_i}, \xi_{s_j}$)로부터 계산된 상대 포즈 간의 오차(잔차, residual)의 제곱이다. $\rho$는 후술할 손실 함수다. 이 식의 의미는, 모든 제약 조건들의 오차를 종합했을 때 그 총합이 최소가 되는 전역 포즈들을 찾으라는 것이다.

- **이상치(Outlier) 대응을 위한 Huber 손실 함수:** 포즈 그래프 최적화에서 가장 큰 위협은 잘못된 루프 클로저 제약, 즉 이상치(outlier)다. 예를 들어, 서로 다른 두 복도를 구조가 비슷하다는 이유로 같은 장소라고 잘못 판단하면, 이 잘못된 제약 하나가 전체 그래프를 심각하게 왜곡시켜 최적화가 실패하게 된다.15

  Cartographer는 이러한 문제에 강건하게 대처하기 위해 Huber 손실 함수($\rho$)를 사용한다.4 Huber 손실은 오차의 크기에 따라 다르게 행동하는 영리한 함수다.16

  - **정의:** Huber 손실 함수 $L_{\delta}(a)$는 파라미터 $\delta$를 기준으로 다음과 같이 정의된다.16
    $$
    L_{\delta}(a) = \begin{cases} \frac{1}{2}a^2 & \text{for } |a| \le \delta \\ \delta(|a| - \frac{1}{2}\delta) & \text{otherwise} \end{cases}
    $$
    **작동 방식:**

    - 오차($a$)가 작을 때 ($|a| \le \delta$): 제곱 오차(L2 loss)처럼 작동한다. 이는 가우시안 노이즈 가정 하에서 최적의 해를 제공하며, 정밀한 최적화를 가능하게 한다.
    - 오차($a$가 클 때 ($|a| > \delta$): 절대 오차(L1 loss)처럼 선형적으로 증가한다. 이는 오차가 큰 이상치에 대해 과도한 페널티를 부과하지 않음으로써, 이상치가 전체 최적화에 미치는 영향을 크게 줄여준다. 이처럼 Huber 손실 함수를 사용함으로써 Cartographer의 백엔드는 잘못된 데이터 연관에 대해 높은 수준의 강건성을 확보하게 된다.4

#### 2. 실시간 루프 클로저

루프 클로저는 누적된 드리프트를 해결하는 가장 효과적인 방법이다. Cartographer는 이 과정을 실시간으로 수행하기 위해 매우 효율적인 검색 알고리즘을 사용한다.

- **분기 한정법 (Branch-and-Bound) 기반 스캔 매칭:**
  - **목표:** 글로벌 SLAM에서는 완성된 모든 서브맵을 대상으로 현재 스캔과 일치하는 곳이 있는지 탐색해야 한다. 이는 엄청난 계산량을 요구하는 검색 문제다. Cartographer는 이 문제를 실시간으로 풀기 위해 분기 한정법(Branch-and-Bound Scan matching, BBS)이라는 최적화된 검색 기법을 사용한다.4
  - **핵심 아이디어:** 분기 한정법의 핵심은 '전체 검색 공간을 무식하게 다 뒤지지 말고, 해가 될 가능성이 없는 영역(branch)은 조기에 포기하자(가지치기, pruning)'는 것이다.
  - 알고리즘 과정 4:
    1. **검색 공간 구성:** 현재 포즈 추정치 주변에 정의된 넓은 검색 윈도우($W$, 예: x, y로 7m, 각도로 30도)를 설정한다. 이 넓은 공간을 픽셀 크기 $r$과 계산된 각도 단계 $\delta_{\theta}$를 이용해 유한한 개수의 후보 포즈들로 이산화한다.
    2. **계층적 트리 구조:** 이산화된 검색 공간을 계층적인 트리 구조로 표현한다. 트리의 루트 노드는 전체 검색 공간을 의미하고, 깊이가 깊어질수록 더 작은 영역을 나타낸다. 리프 노드는 개별 후보 포즈 하나에 해당한다.
    3. **상한(Upper Bound) 계산:** 분기 한정법의 핵심은 각 노드(즉, 특정 부분 검색 공간)에 대해, 그 안에 포함된 모든 후보 포즈들이 얻을 수 있는 **'최대 가능한 점수(상한)'**를 매우 효율적으로 계산하는 것이다. Cartographer는 이를 위해 **'사전 계산된 그리드($M_{precomp}$)'**라는 기법을 사용한다. 서브맵이 완성되면, 다양한 해상도 레벨($ch$)에 대해 각 픽셀 위치에서 시작하는 특정 크기($2^{ch} \times 2^{ch}$)의 영역 내 픽셀 값의 최대값을 미리 계산하여 저장해 둔다. 덕분에 넓은 영역의 최대 점수를 단 한 번의 조회로 근사할 수 있어 상한 계산이 극도로 빨라진다.
    4. **가지치기 (Pruning):** 깊이 우선 탐색(DFS) 방식으로 트리를 탐색한다. 탐색 중인 현재 노드의 상한 점수가 지금까지 발견된 최적해의 점수(best score)보다 낮다면, 그 노드 아래에 있는 모든 자식 노드들은 현재의 최적해보다 더 좋은 해를 포함할 가능성이 전혀 없다. 따라서 해당 하위 트리는 더 이상 탐색하지 않고 즉시 버린다(가지치기).
  - **실시간성 확보:** 이 분기 한정법 덕분에 Cartographer는 수많은 후보 위치를 직접 다 테스트해보지 않고도 최적의 루프 클로저 위치를 매우 빠르게 찾아낼 수 있다. 이는 새로운 스캔이 추가되는 속도보다 루프 클로저 탐색이 더 빨리 완료되도록 보장하며, 사용자가 느끼기에 '즉시' 루프가 닫히는 경험을 가능하게 하는 기술적 기반이 된다.4

## 3. 주요 SLAM 알고리즘과의 비교 분석

Cartographer의 기술적 위상과 특징을 명확히 이해하기 위해, 다른 주요 오픈소스 SLAM 알고리즘들과의 비교 분석은 필수적이다. 이 섹션에서는 GMapping, Hector SLAM, ORB-SLAM, LIO-SAM과의 비교를 통해 각 알고리즘의 기반 기술, 센서 요구사항, 그리고 성능상의 장단점을 심층적으로 분석한다.

### A. Cartographer vs. GMapping (파티클 필터 기반)

- **기술적 원리 비교:**
  - **GMapping:** GMapping은 Rao-Blackwellized 파티클 필터(RBPF)를 사용하는 대표적인 2D LiDAR SLAM 알고리즘이다.18 이 접근법의 핵심은 로봇의 궤적을 다수의 가설(파티클)로 표현하고, 각 파티클이 독립적으로 자신의 지도와 포즈를 추정하는 것이다. 새로운 관측이 들어오면, 각 파티클의 가설이 관측과 얼마나 일치하는지에 따라 가중치가 부여되고, 가중치가 낮은 파티클은 도태되고 높은 파티클은 복제되는 과정을 반복한다(재샘플링).20
  - **Cartographer:** 반면, Cartographer는 그래프 최적화(Graph Optimization)에 기반한다.19 단일 최적해를 찾는 것을 목표로 하며, 로봇의 전체 궤적을 하나의 그래프로 모델링한다. 루프 클로저와 같은 제약을 통해 그래프 전체의 일관성을 유지하도록 최적화하여 누적 오차를 보정한다.
  - 이 근본적인 차이는 확장성에서 큰 성능 차이를 야기한다. GMapping은 환경이 커질수록 필요한 파티클의 수가 기하급수적으로 증가하여 계산량이 폭증하고, 모든 파티클이 소수의 유망한 가설로 수렴해버리는 파티클 고갈(particle depletion) 문제에 직면할 수 있다.4 Cartographer의 그래프 기반 접근 방식은 노드와 엣지의 수에 비례하여 계산량이 증가하므로 대규모 환경에서 훨씬 뛰어난 확장성을 보인다.4
- **성능 및 센서 요구사항 비교:**
  - **성능:** 실험적 비교에서 Cartographer는 일반적으로 GMapping보다 더 낮은 RMSE(평균 제곱근 오차)를 보이며, 특히 대규모 환경에서 더 정확하고 일관된 지도를 생성한다.19 GMapping은 로봇의 이동 속도에 대한 제한이 상대적으로 적어 특정 상황에서는 더 넓은 적용 범위를 가질 수 있다.19 하지만 GMapping은 장기적으로 드리프트가 누적되는 것을 피할 수 없는 반면, Cartographer는 강력한 루프 클로저 기능으로 이를 효과적으로 제거한다.
  - **센서 요구사항:** GMapping은 정확한 주행 거리계(odometry) 데이터에 대한 의존도가 매우 높다. 주행 거리계 정보가 없거나 부정확하면 파티클의 예측 단계에서 심각한 오류가 발생한다.19 Cartographer는 주행 거리계를 선택적으로 사용할 수 있으며, IMU 데이터를 적극적으로 활용하여 로봇의 회전을 보정하고 기울어진 환경에 대응하는 등 더 유연한 센서 융합 전략을 취한다.11

### B. Cartographer vs. Hector SLAM (스캔 매칭 기반)

- **기술적 원리 비교:**
  - **Hector SLAM:** Hector SLAM은 고주파, 고정밀 LiDAR의 빠른 스캔 업데이트 속도를 신뢰하고, 연속된 스캔 간의 정합(scan matching)만으로 로봇의 포즈를 추정한다.19 가우시안-뉴턴(Gauss-Newton) 최적화 기법을 사용하여 현재 스캔이 이전 맵에 가장 잘 들어맞는 위치를 찾는다. 주행 거리계 데이터가 필요 없다는 것이 가장 큰 특징이다.20
  - **Cartographer:** Cartographer 역시 스캔 매칭을 사용하지만, 스캔-대-스캔이 아닌 스캔-대-서브맵(scan-to-submap) 매칭을 수행한다.4 이는 단일 스캔이 아닌, 여러 스캔이 누적된 더 안정적인 지도 조각에 현재 스캔을 정합하므로 노이즈에 더 강건하다. 결정적으로, Hector SLAM에는 없는 글로벌 루프 클로저 및 백엔드 최적화 단계가 존재한다.
- **성능 및 센서 요구사항 비교:**
  - **성능:** Hector SLAM은 특징이 풍부하고 로봇이 저속으로 움직이는 환경에서 매우 정밀하고 깔끔한 지도를 생성할 수 있다.19 하지만 특징이 부족한 긴 복도나, 로봇의 빠른 회전 및 이동 시에는 매칭에 실패하고 심각한 드리프트를 유발한다.19 루프 클로저 기능이 없기 때문에 한번 발생한 오차는 절대 보정되지 않는다.21 Cartographer는 로봇의 이동 속도에 더 많은 제약이 있을 수 있지만19, 서브맵 기반 매칭과 루프 클로저 덕분에 장기적인 안정성과 대규모 환경 매핑 능력 면에서 Hector SLAM을 압도한다.
  - **자원 및 센서:** Hector SLAM은 CPU 사용량이 적다는 장점이 있으나 20, 이는 루프 클로저와 같은 복잡한 백엔드 처리를 수행하지 않기 때문이다. Cartographer는 백그라운드 최적화로 인해 더 많은 계산 자원을 소모한다.19 센서 측면에서 Hector SLAM은 LiDAR의 성능에 절대적으로 의존하는 반면, Cartographer는 IMU, 주행 거리계 등 다른 센서와의 융합을 통해 LiDAR의 단점을 보완한다.

### C. Cartographer vs. ORB-SLAM (Visual SLAM)

- **기술적 원리 및 센서 비교:**
  - **ORB-SLAM:** ORB-SLAM은 카메라 이미지를 주 입력으로 사용하는 대표적인 Visual SLAM(VSLAM) 알고리즘이다.13 이미지에서 ORB(Oriented FAST and Rotated BRIEF) 특징점을 추출하고, 이 특징점들의 위치를 추적하여 카메라의 포즈를 추정하고 3D 맵을 생성한다. Cartographer와 마찬가지로 키프레임 기반의 그래프 최적화와 강력한 루프 클로저, 재지역화(re-localization) 기능을 갖추고 있다.22
  - **근본적 차이:** 가장 큰 차이는 주력 센서다. Cartographer는 LiDAR, ORB-SLAM은 카메라를 사용한다. 이로 인해 작동 환경과 성능 특성이 근본적으로 달라진다. LiDAR는 조명 변화에 영향을 받지 않고, 물체까지의 거리를 직접적이고 정확하게 측정할 수 있어 기하학적으로 정밀한 맵을 생성하는 데 유리하다.23 카메라는 저렴하고 풍부한 시각적 정보(색상, 텍스처)를 제공하여 장소 인식에 강점을 보이지만, 조명 변화, 모션 블러, 텍스처가 없는 벽이나 반복적인 패턴에 매우 취약하다.13
- **성능 비교:**
  - ORB-SLAM(특히 최신 버전인 ORB-SLAM3)은 VSLAM 분야에서 최고 수준의 정확도와 강건성을 자랑하는 벤치마크로 인정받는다.25 텍스처가 풍부한 환경에서는 LiDAR SLAM 못지않은 성능을 보여준다.
  - 두 알고리즘을 직접 비교하는 것은 '사과와 오렌지'를 비교하는 것과 같을 수 있다. 적용 분야에 따라 우위가 갈린다. 예를 들어, 기하학적 구조의 정밀한 측정이 중요한 산업 환경이나 어두운 환경에서는 Cartographer가 유리하고, 저렴한 비용으로 넓은 지역의 시각적 맵을 구축하거나 AR(증강 현실) 애플리케이션에서는 ORB-SLAM이 더 적합하다. 두 알고리즘 모두 그래프 최적화와 루프 클로저라는 동일한 백엔드 철학을 공유하지만, 프론트엔드에서 처리하는 데이터의 종류와 방식이 완전히 다르다.

### D. Cartographer vs. LIO-SAM (LiDAR-Inertial SLAM)

- **기술적 원리 비교:**
  - **LIO-SAM:** LIO-SAM은 LiDAR, IMU, 그리고 선택적으로 GPS 데이터를 강하게 결합(tightly-coupled)하는 최신 3D SLAM 알고리즘이다.23 LiDAR 스캔에서 평면이나 모서리와 같은 특징점을 추출하여 계산량을 줄이고, IMU 측정값을 사전적분(pre-integration)하여 스캔 간의 초기 포즈 추정 및 모션 왜곡 보정에 사용한다. 모든 센서 측정값은 하나의 요인 그래프(factor graph) 내에서 동시에 최적화된다.
  - **Cartographer:** Cartographer도 LiDAR와 IMU를 융합하지만, LIO-SAM과 같은 강한 결합 방식보다는 느슨한 결합(loosely-coupled)에 가깝다. 로컬 SLAM에서는 IMU 데이터가 스캔 매칭의 초기 추정값을 제공하는 데 사용되고, 글로벌 SLAM에서는 포즈 그래프의 제약 조건으로 추가된다. 로컬과 글로벌이 분리된 구조를 가진다.
- **성능 비교:**
  - **정확도 및 강건성:** 여러 비교 연구에서 LIO-SAM은 Cartographer에 비해 더 낮은 궤적 오류를 기록하며, 특히 LiDAR 데이터만으로는 포즈를 특정하기 어려운 퇴화(degeneracy) 환경(예: 긴 터널)에서 더 강건한 성능을 보인다.27 LIO-SAM의 강한 결합 방식은 각 센서의 정보를 최대한 활용하여 상호 보완 효과를 극대화하기 때문이다.27
  - **적용 환경:** LIO-SAM은 GPS 요소를 자연스럽게 통합할 수 있어 자율주행과 같은 대규모 실외 환경에서 드리프트를 효과적으로 억제하는 데 매우 강력하다.23 반면, Cartographer는 GPS 통합보다는 실내 환경이나 백팩 매핑과 같이 GPS가 없는 환경에서의 성능에 더 초점을 맞추어 개발되었다. 따라서 대규모 실외 환경에서는 LIO-SAM이, 복잡한 실내 환경에서는 Cartographer가 각각의 강점을 보일 수 있다.

### Table 1: 주요 SLAM 알고리즘 비교 분석

이 표는 각 SLAM 알고리즘의 핵심적인 차이점을 한눈에 파악할 수 있도록 구조화된 정보를 제공한다. 사용자는 자신의 애플리케이션(예: 실내 vs. 실외), 사용 가능한 센서, 계산 자원, 요구되는 정확도 수준에 따라 어떤 알고리즘이 가장 적합할지 신속하게 판단할 수 있다. 텍스트로 흩어진 정보를 응축하여 직접적인 비교를 가능하게 함으로써, 기술 선택 과정에서의 의사결정을 지원하는 데 핵심적인 역할을 한다.

| 알고리즘 (Algorithm) | 핵심 원리 (Core Principle)        | 주요 센서 (Key Sensors)           | 장점 (Advantages)                                            | 단점 (Disadvantages)                                         |
| -------------------- | --------------------------------- | --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Cartographer**     | 그래프 최적화 (로컬/글로벌 분리)  | LiDAR, IMU (Odometry 선택)        | 대규모 환경 확장성, 강력한 실시간 루프 클로저, 2D/3D 지원, IMU 활용으로 기울기 보정 4 | 파라미터 튜닝 복잡, 상대적으로 높은 계산 자원 소모, 빠른 움직임에 민감 13 |
| **GMapping**         | Rao-Blackwellized 파티클 필터     | 2D LiDAR, Odometry                | 구현 용이, 낮은 속도에서 안정적, 속도 제한이 적음 19         | 대규모 환경에서 파티클 고갈 및 계산량 폭증, 누적 오차에 취약, 루프 클로저 없음 4 |
| **Hector SLAM**      | 스캔-투-스캔 매칭 (가우시안-뉴턴) | 고속/고정밀 LiDAR                 | Odometry 불필요, 낮은 CPU 사용량, 특징 풍부한 환경에서 고정밀 지도 생성 19 | 빠른 움직임/회전 및 특징 없는 환경에 매우 취약, 루프 클로저 부재로 장기 드리프트 19 |
| **ORB-SLAM3**        | 그래프 최적화 (특징점 기반)       | 카메라 (단안/스테레오/RGB-D), IMU | VSLAM 분야 최고 수준의 정확도 및 강건성, 뛰어난 재지역화 및 루프 클로저, 다중 맵 지원 13 | 조명 변화, 빠른 움직임, 텍스처 없는 환경에 취약, LiDAR에 비해 기하학적 정확도 낮을 수 있음 13 |
| **LIO-SAM**          | 요인 그래프 최적화 (강한 결합)    | 3D LiDAR, IMU, GPS (선택)         | LiDAR/IMU/GPS 강한 결합으로 고정밀/강건성, 퇴화 문제에 강함, 대규모 실외 환경에 최적화 23 | 높은 계산 자원 요구, 고가 센서(3D LiDAR) 필요, 실내 환경에서는 과도할 수 있음 27 |

## 4. 실제 적용 및 튜닝 가이드

Cartographer의 강력한 성능을 실제 로봇 시스템에서 온전히 활용하기 위해서는 ROS와의 통합 방법과 복잡한 파라미터들을 올바르게 설정하는 과정에 대한 깊은 이해가 필수적이다. Cartographer 튜닝은 단순히 숫자를 바꾸는 행위가 아니라, 개발자가 자신의 로봇 시스템(센서의 특성, 노이즈 수준)과 작동 환경(크기, 구조, 동적 요소)에 대한 이해를 바탕으로, Cartographer의 내부 의사결정 과정에 개입하는 고도의 공학적 행위다. 모든 튜닝 파라미터는 결국 "어떤 정보를 얼마나 신뢰하고(가중치), 얼마나 자주, 얼마나 많은 데이터를 대상으로 계산할 것인가(샘플링)?"라는 근본적인 질문에 대한 답을 시스템에 알려주는 과정이다. 이 섹션에서는 실제 적용을 위한 설정 방법과 목표별 튜닝 전략을 상세히 다룬다.

### A. ROS(Robot Operating System) 통합 및 설정

Cartographer는 `cartographer_ros`라는 전용 패키지를 통해 ROS 생태계와 긴밀하게 통합되어 있다.1

- **설치:** ROS 환경에서는 `sudo apt install ros-<distro>-cartographer-ros`와 같은 명령어를 통해 바이너리 패키지를 간편하게 설치할 수 있으며, 특정 버전이나 커스터마이징이 필요한 경우 소스 코드를 직접 클론하여 빌드할 수도 있다.28

- **실행 구조:** Cartographer의 실행은 ROS의 `launch` 파일을 통해 관리된다. 이 `launch` 파일은 Cartographer의 핵심 노드인 `cartographer_node`를 실행하는 것 외에도, LiDAR, IMU 등 필요한 센서의 드라이버 노드, 로봇의 각 부분(base_link, laser_link 등)의 좌표 관계를 정의하는 `tf` 변환, 그리고 실시간으로 지도와 로봇의 위치를 시각화하기 위한 RViz 실행 등을 포함한다.8

- **핵심 설정 파일 (`.lua`):** Cartographer의 모든 세부 동작은 Lua 스크립트 언어로 작성된 설정 파일들을 통해 제어된다.28 이 파일들은 

  `map_builder.lua`, `trajectory_builder.lua`, `pose_graph.lua` 등으로 구성되며, 각각 맵 생성의 전반적인 옵션, 로컬 SLAM(궤적 생성), 글로벌 SLAM(포즈 그래프)에 관련된 파라미터들을 담고 있다.21 튜닝 작업의 대부분은 이 

  `.lua` 파일들을 수정하는 것을 의미한다.

- **지도 저장 및 변환:** 매핑 작업이 완료되면, 생성된 지도를 영구적으로 저장해야 한다.

  1. 먼저 `rosservice call /finish_trajectory 0` 명령으로 현재 궤적을 종료하고 더 이상 데이터를 받지 않음을 알린다.8
  2. 그 다음 `rosservice call /write_state "{filename: '~/map.pbstream'}"` 명령으로 현재까지의 모든 SLAM 상태(지도, 포즈 그래프 등)를 `.pbstream`이라는 자체 형식 파일로 직렬화하여 저장한다.8
  3. 이 `.pbstream` 파일은 Cartographer 내부 형식이라 다른 ROS 패키지에서 직접 사용할 수 없다. 따라서 ROS 네비게이션 스택과 같은 표준 ROS 도구에서 사용하기 위해 `rosrun cartographer_ros cartographer_pbstream_to_ros_map` 스크립트를 사용하여 점유 격자 지도 형식인 `.pgm`(이미지 파일)과 메타데이터를 담은 `.yaml` 파일로 변환해야 한다.7

### B. 주요 튜닝 파라미터 해설 (`.lua` 설정 파일 중심)

Cartographer의 튜닝은 매우 어렵고, 파라미터들이 상호 의존적이라 체계적인 접근이 요구된다.30 주요 파라미터는 다음과 같다.

- **로컬 SLAM (`trajectory_builder_\*.lua`):**
  - `submaps.num_range_data`: 하나의 서브맵을 구성하는 데 사용될 LiDAR 스캔의 개수를 정의한다.30 이 값이 크면 서브맵의 크기가 커져 더 넓은 영역을 커버하므로 지역적으로 안정적인 스캔 매칭이 가능하지만, 메모리 사용량이 늘어나고 새로운 서브맵이 생성되는 주기가 길어진다. 반대로 값이 작으면 드리프트가 증가할 수 있다. 환경의 크기와 특징에 맞춰 조절해야 한다.
  - `voxel_filter_size`: 포인트 클라우드를 다운샘플링하는 복셀의 크기다.9 값을 키우면 포인트 수가 줄어들어 계산 속도는 빨라지지만, 벽의 모서리와 같은 세밀한 기하학적 특징을 잃어버려 매칭 정확도가 떨어질 수 있다.
  - `ceres_scan_matcher.translation_weight`, `rotation_weight`: 로컬 SLAM의 스캔 매칭 시, 다른 센서(IMU, Odometry)로부터 예측된 초기 포즈(prior)를 얼마나 신뢰할지를 결정하는 가중치다.30 이 값이 높으면, 스캔 매칭 알고리즘은 LiDAR 데이터가 약간 불일치하더라도 초기 예측 포즈에서 크게 벗어나지 않으려고 한다. 이는 LiDAR 데이터의 노이즈가 심하거나 특징이 부족한 환경에서 안정성을 높여주지만, 반대로 양질의 LiDAR 데이터가 있음에도 불구하고 부정확한 IMU/Odometry 예측에 얽매이게 할 수도 있다. 이 가중치들은 시스템의 강건성과 정확성 사이의 균형을 맞추는 핵심 파라미터다.
- **글로벌 SLAM (`pose_graph.lua`):**
  - `optimize_every_n_nodes`: 글로벌 포즈 그래프 최적화를 얼마나 자주 수행할지를 노드(스캔) 개수 기준으로 결정한다.30 값이 작을수록 루프 클로저가 더 자주, 빠르게 반영되어 맵의 드리프트가 실시간으로 수정되는 것을 볼 수 있지만, CPU 사용량이 크게 증가한다. 0으로 설정하면 글로벌 SLAM이 비활성화되어 루프 클로저가 작동하지 않는다.
  - `constraint_builder.min_score`: 스캔 매칭 결과가 루프 클로저 제약으로 인정받기 위해 필요한 최소 점수다.30 이 값을 높이면, 매칭 품질이 낮은(불확실한) 후보들을 걸러내어 잘못된 루프 클로저가 추가될 위험을 줄인다. 하지만 너무 높게 설정하면, 실제 유효한 루프 클로저조차 놓칠 수 있다.
  - `matcher_translation_weight`, `matcher_rotation_weight`: 글로벌 최적화 과정에서 루프 클로저 제약 조건이 전체 그래프에 미치는 영향력(가중치)을 조절한다.33

### C. 목표별 튜닝 전략

애플리케이션의 요구사항에 따라 튜닝의 목표와 전략은 달라져야 한다.

#### 1. 저지연(Low Latency) 환경

실시간으로 로봇을 제어하거나 외부와 상호작용해야 하는 경우, SLAM의 지연 시간은 매우 중요하다. 목표는 로컬 SLAM의 처리 시간을 줄이고, 글로벌 SLAM의 계산 부하가 현재 데이터 처리 속도를 따라잡지 못해 백로그가 쌓이는 것을 방지하는 것이다.30

- **로컬 SLAM 지연 시간 줄이기:**
  - 계산량을 줄이는 것이 핵심이다. `voxel_filter_size`나 `submaps.resolution`을 높여 처리할 데이터의 양을 줄인다.30
  - `submaps.num_range_data`를 줄여 서브맵의 크기를 작게 유지한다.30
- **글로벌 SLAM 지연 시간 줄이기:**
  - 최적화의 빈도와 범위를 줄인다. `optimize_every_n_nodes` 값을 늘려 최적화를 덜 자주 수행하게 한다.30
  - 최적화에 사용될 데이터의 양을 줄인다. `global_sampling_ratio` (전체 노드 샘플링 비율)와 `constraint_builder.sampling_ratio` (제약 생성 샘플링 비율)를 낮춘다.30
  - 루프 클로저 검색 공간을 줄인다. `linear_xy_search_window`와 같은 검색 창 크기를 줄이면 탐색 시간이 단축된다.30

#### 2. 순수 지역화(Pure Localization) 모드

이미 잘 만들어진 지도(`.pbstream` 파일)가 있는 상태에서, 매핑은 중단하고 로봇의 위치만 지속적으로 추정하는 모드다.34

- **활성화:** `trajectory_builder.lua` 파일에서 `TRAJECTORY_BUILDER.pure_localization = true`로 설정한다.30
- **핵심 파라미터 조정:**
  - 새로운 맵을 생성하지 않으므로, 로봇의 위치는 기존 맵과의 비교를 통해 최대한 자주 갱신되어야 한다. 따라서 `POSE_GRAPH.optimize_every_n_nodes`를 3과 같이 매우 작은 값으로 설정하여 거의 모든 스캔마다 전역 최적화가 수행되도록 한다.30
  - 이렇게 자주 최적화를 수행하면 계산량이 폭증하므로, 이를 상쇄하기 위해 `global_sampling_ratio`와 `constraint_builder.sampling_ratio`를 대폭 낮춰야 한다.30
  - **가장 중요한 주의사항:** 순수 지역화 모드에서는 `submaps.resolution` 파라미터 값이 사용하려는 기존 맵(`.pbstream`)이 생성될 때 사용된 해상도와 **정확히 일치해야 한다**. 해상도가 다를 경우, 스캔 매칭이 실패하여 위치 추정이 불가능해질 수 있다.30

#### 3. 주행 거리계(Odometry) 데이터 활용

휠 인코더나 다른 SLAM 시스템으로부터 주행 거리계 정보를 입력받는 경우(`use_odometry = true`), 글로벌 최적화에서 이 정보의 신뢰도를 조절할 수 있다.

- **가중치 조절:** `pose_graph.lua`의 `POSE_GRAPH.optimization_problem` 섹션에 있는 4개의 가중치 파라미터를 사용한다: `local_slam_pose_translation_weight`, `local_slam_pose_rotation_weight`, `odometry_translation_weight`, `odometry_rotation_weight`.30
- **튜닝 전략:** 이 가중치들은 로컬 SLAM(LiDAR 스캔 매칭)의 결과와 외부 주행 거리계 정보 중 무엇을 더 신뢰할지를 결정한다. 예를 들어, 휠 인코더 기반의 주행 거리계는 직진 이동(translation)은 비교적 정확하지만, 제자리 회전이나 미끄러짐으로 인해 회전(rotation) 오차가 누적되기 쉽다. 이런 경우, `odometry_rotation_weight`를 낮추거나 0으로 설정하여 회전 추정은 주로 IMU나 LiDAR 스캔 매칭에 의존하도록 하고, 직진 이동 정보만 신뢰하도록 시스템을 구성할 수 있다.30

## 5. 한계점 및 향후 연구 방향

Cartographer는 그래프 기반 LiDAR SLAM의 발전에 큰 획을 그었지만, 기술적 한계와 오픈소스 프로젝트로서의 현실적인 제약을 가지고 있다. 이러한 한계점들은 역설적으로 현재 SLAM 기술이 나아가야 할 방향을 제시한다. Cartographer가 정점에 있던 '기하학적, 그래프 기반 SLAM'의 시대에서, 이제 SLAM 연구는 '의미론적, 학습 기반 SLAM'이라는 새로운 패러다임으로 전환하고 있다. Cartographer의 한계는 바로 이 새로운 시대가 해결하고자 하는 문제들과 정확히 일치하며, 이는 기술 발전의 자연스러운 과정이다.

### A. Cartographer의 기술적 한계

- **동적 환경 대응의 취약성:** Cartographer는 기본적으로 환경이 정적(static)이라는 강한 가정을 기반으로 설계되었다.35 따라서 환경 내에 움직이는 사람, 차량, 다른 로봇과 같은 동적 객체(dynamic objects)가 많을 경우, 이를 고정된 장애물로 오인하여 지도에 영구적으로 그리거나, 잘못된 스캔 매칭을 유발하여 위치 추정의 정확도를 심각하게 저하시킬 수 있다.13 실제 상용 시스템에서는 이를 보완하기 위해 동적 객체를 탐지하고 제거하는 별도의 필터링 모듈을 전처리단에 추가해야 한다.9
- **비정형/특수 환경에서의 성능 저하:** Cartographer는 건물 내부와 같이 구조화된 환경에 최적화되어 있다. 이로 인해 다음과 같은 비정형 환경에서는 성능이 크게 저하될 수 있다.
  - **특징 없는 환경:** 특징이 거의 없는 긴 복도나 넓은 홀과 같이 기하학적 정보가 부족한 곳에서는 스캔 매칭이 모호해져 드리프트가 쉽게 발생한다.
  - **대칭적 구조:** 완전히 동일한 구조의 복도나 방이 반복되는 환경(예: 호텔, 아파트 복도)에서는 잘못된 장소를 같은 곳으로 인식하는 '지각적 모호성(perceptual aliasing)' 문제가 발생하여, 치명적인 루프 클로저 오류를 일으킬 수 있다.36
  - **거친 지형:** 붕괴된 건물 잔해나 울퉁불퉁한 야외 지형과 같이 2D 평면 가정이 깨지는 수색 및 구조(Search and Rescue) 환경에는 근본적으로 부적합하다.20
  - **특수 매질:** 물 표면의 반사나 빛의 굴절이 심한 수중 환경에서는 LiDAR 센서 자체가 정상적으로 작동하기 어려워 적용이 제한된다.37
- **높은 계산 자원 요구:** 실시간 성능을 위해 최적화되었음에도 불구하고, Cartographer는 여전히 상당한 CPU와 메모리 자원을 필요로 한다. 특히 백그라운드에서 지속적으로 실행되는 글로벌 SLAM의 포즈 그래프 최적화는 계산 부하가 크다.19 이로 인해 라즈베리 파이와 같은 저사양 임베디드 시스템에서 원활하게 구동하기에는 부담이 따를 수 있다.19
- **초기화 및 장기 안정성의 어려움:** 성공적인 맵 생성을 위해서는 센서 데이터의 품질과 더불어, 수많은 `.lua` 파라미터에 대한 정밀한 튜닝이 필수적이다. 이는 SLAM에 대한 깊은 전문 지식을 요구하므로, 초보자에게는 초기 맵 생성 자체가 큰 장벽이 될 수 있다.13 또한, 시간이 지남에 따라 가구 재배치나 구조물 변경 등 환경이 변할 경우, 기존에 생성된 맵과의 불일치가 발생하여 장기적인 안정성을 유지하기 어렵다는 문제도 있다.13

### B. 오픈소스 프로젝트로서의 현황

앞서 언급했듯이, 구글이 주도하던 공식 `cartographer` GitHub 저장소는 2022년 이후 사실상 개발이 중단된 상태다.3 이는 Cartographer가 더 이상 최첨단 연구의 대상이 아니라, 성숙되고 안정화된 기술 단계에 접어들었음을 의미한다. 하지만 이는 동시에 다음과 같은 현실적인 제약을 야기한다.

- **기술 지원의 부재:** 새로운 센서에 대한 공식 지원이나 최신 운영체제(OS) 및 라이브러리와의 호환성 문제, 발견된 버그에 대한 신속한 수정 등을 기대하기 어렵다.
- **유지보수의 책임:** 사용자는 잠재적인 문제에 대해 커뮤니티의 도움을 받거나 스스로 해결해야 한다. 이는 상용 제품 개발이나 장기적인 연구 프로젝트에 Cartographer를 핵심 기술로 채택할 때 신중하게 고려해야 할 중요한 위험 요소다. 현재 `cartographer_ros`는 커뮤니티에 의해 제한적으로 유지되고 있지만, 이는 구글의 공식 지원과는 거리가 멀다.3

### C. SLAM 기술의 도전 과제와 미래

Cartographer의 한계는 현재 SLAM 연구 커뮤니티가 집중하고 있는 도전 과제들과 맥을 같이한다.

- **장기 자율성 (Long-term Autonomy):** 로봇이 한 번 지도를 만들고 끝나는 것이 아니라, 수 주, 수 개월, 혹은 수 년에 걸쳐 변화하는 환경(예: 계절 변화에 따른 식생 변화, 조명 변화, 가구 재배치)에 강건하게 대처하며 자신의 위치를 파악하고 지도를 지속적으로 갱신하는 '평생 SLAM(Lifelong SLAM)' 기술이 핵심 연구 주제다.39
- **의미론적 이해 (Semantic Understanding):** 기존의 SLAM이 '어디에 무엇이 있다'는 기하학적 정보(점, 선, 면)에 집중했다면, 미래의 SLAM은 '그것이 무엇인지'를 이해하는 의미론적(semantic) 정보를 지도에 통합하는 방향으로 나아가고 있다.35 예를 들어, 지도의 특정 영역이 '문'임을 인식하면 로봇은 그곳을 통과할 수 있다고 판단하고, '사람'을 인식하면 동적 객체로 분류하여 회피 경로를 계획할 수 있다. 이는 로봇이 환경과 더 지능적으로 상호작용하기 위한 필수 기술이다.
- **딥러닝과의 통합 (Integration with Deep Learning):** 딥러닝 기술은 SLAM의 거의 모든 구성 요소를 혁신하고 있다.24
  - **모듈 교체/개선:** SuperPoint와 같은 학습 기반 특징점 추출기는 수작업으로 설계된 특징점(e.g., ORB)보다 조명 변화나 시점 변화에 더 강건한 성능을 보이며 45, NetVLAD와 같은 딥러닝 기반 장소 인식(place recognition) 기술은 루프 클로저의 정확도를 획기적으로 높인다. 또한, 딥러닝을 통해 단안 카메라 이미지로부터 깊이를 추정하거나 동적 객체를 분리하는 연구도 활발하다.43
  - **End-to-End SLAM:** 센서의 원시 데이터 입력으로부터 로봇의 포즈를 직접 출력하는 종단간(end-to-end) 학습 모델을 구축하려는 시도도 이루어지고 있다. 이는 전체 SLAM 파이프라인을 데이터 기반으로 최적화하려는 궁극적인 목표를 가진다.43
- **새로운 3D 표현 방식 (New Representations):** 전통적인 점유 격자 지도나 포인트 클라우드를 넘어, NeRF(Neural Radiance Fields)나 3D 가우시안 스플래팅(Gaussian Splatting)과 같은 새로운 3D 표현 방식이 SLAM에 통합되고 있다.46 이러한 '신경망 렌더링(neural rendering)' 기술들은 매우 사실적인 3D 장면을 생성할 수 있어, 단순한 기하학적 지도를 넘어 고품질의 시각적 지도를 구축하는 새로운 가능성을 열고 있다.
- **강건성과 일반화:** 여전히 수많은 최첨단 SLAM 알고리즘들이 현실 세계의 다양한 도전적인 시나리오(악천후, 극단적인 조명 변화, 심한 센서 노이즈)에서 쉽게 실패한다. 다양한 실제 환경 데이터를 포함하는 대규모 벤치마크 데이터셋을 구축하고, 이를 통해 어떤 환경에서도 안정적으로 작동하는 강건한 알고리즘을 개발하는 것이 중요한 과제로 남아있다.36

결론적으로, Cartographer는 그 자체로 매우 완성도 높은 시스템이지만, SLAM 기술의 여정에서 하나의 중요한 이정표다. 이제 SLAM 연구는 Cartographer가 남긴 기반 위에서, 더 동적이고, 더 의미론적이며, 더 지능적인 방향으로 나아가고 있다.

## 6. 결론: Cartographer의 의의와 전망

### A. Cartographer의 기술사적 의의

Cartographer는 현대 SLAM 기술의 역사에서 중요한 전환점을 마련한 알고리즘으로 평가받는다. 그 기술사적 의의는 다음과 같이 요약할 수 있다.

첫째, **그래프 기반 SLAM의 대중화와 표준화에 결정적으로 기여했다.** GMapping과 같은 파티클 필터 기반 SLAM이 가진 대규모 환경에서의 확장성 문제를, 로컬/글로벌 SLAM의 이중 구조와 서브맵 개념을 통해 효과적으로 해결했다. 이는 복잡한 이론과 높은 계산 비용으로 인해 주로 학계의 연구 영역에 머물러 있던 그래프 기반 SLAM을, 일반 개발자나 산업계에서도 쉽게 접근하고 안정적으로 활용할 수 있는 수준으로 끌어올렸다.

둘째, **실시간 성능과 전역적 정확성이라는 두 마리 토끼를 잡는 성공적인 아키텍처를 제시했다.** 프론트엔드에서 빠른 스캔 매칭으로 즉각적인 위치를 제공하고, 백그라운드에서 분기 한정법과 같은 고효율 알고리즘으로 루프 클로저를 찾아 전체 지도를 최적화하는 방식은, 제한된 자원 내에서 실시간성과 정확성을 모두 달성하기 위한 매우 정교하고 실용적인 해법임을 입증했다.4

셋째, **다중 센서 융합의 중요성과 효용성을 명확히 보여주었다.** LiDAR 데이터에만 의존하지 않고 IMU, 주행 거리계 등의 정보를 유기적으로 통합함으로써, 기울어진 지면이나 센서의 단기적인 오류와 같은 현실 세계의 문제에 대해 훨씬 강건한 시스템을 구축할 수 있음을 보여주었다.2 이는 후속 LiDAR-Inertial SLAM 연구에 큰 영향을 미쳤다.

결론적으로 Cartographer는 학술적 성취와 실용적 구현 사이의 간극을 성공적으로 메운 사례로서, 강력하고 신뢰성 있는 LiDAR SLAM의 산업 표준 중 하나로 자리매김했다.

### B. 현재적 가치와 미래 전망

비록 구글의 공식적인 유지보수가 활발하지 않은 현 상황에서도 Cartographer의 가치는 여전히 유효하며, 다음과 같은 전망을 기대할 수 있다.

- **안정적인 산업용 베이스라인:** 활발한 개발이 멈췄다는 것은 역설적으로 기술이 성숙하고 안정화되었음을 의미한다. 이로 인해 Cartographer는 이미 수많은 상용 로봇 제품, 특히 실내 물류 로봇(AMR)이나 서비스 로봇 분야에서 검증된 기술로 널리 사용되고 있다.49 Fetch Robotics(현 Zebra)나 Clearpath Robotics와 같은 기업의 로봇들이 3D SLAM 기술을 활용하는 사례는 Cartographer와 같은 그래프 기반 SLAM의 산업적 중요성을 방증한다.50 앞으로도 새로운 기능 개발보다는 안정성이 우선시되는 분야에서 꾸준히 활용될 것이다.
- **차세대 SLAM 연구의 중요한 벤치마크:** SLAM 연구가 딥러닝 기반의 의미론적, 동적 환경 인식으로 나아감에 따라, 새로운 알고리즘의 성능을 평가하기 위한 견고한 '기하학적' 베이스라인의 필요성은 더욱 커지고 있다. Cartographer는 그 정확성과 강건성으로 인해, 새로운 알고리즘들이 기하학적 지도 작성 능력 면에서 얼마나 뛰어난지를 비교 측정하는 중요한 벤치마크로서의 역할을 계속 수행할 것이다.52
- **하이브리드 SLAM 시스템의 백본(Backbone)으로서의 잠재력:** 미래의 SLAM 시스템은 Cartographer와 같은 전통적인 기하학적 SLAM의 강점과, 딥러닝 기반의 상황 인식 및 의미 이해 능력의 강점을 결합한 하이브리드 형태가 될 가능성이 높다. 예를 들어, 딥러닝 모듈이 동적 객체를 제거하고 의미론적 정보를 추출하여 전처리된 데이터를 제공하면, Cartographer는 이를 입력받아 강건하고 정밀한 기하학적 맵과 궤적을 생성하는 '기하학적 백본(geometric backbone)'의 역할을 수행할 수 있다. 이러한 구조는 각 기술의 장점을 극대화하는 효과적인 접근 방식이 될 것이다.

종합적으로, Cartographer는 SLAM 기술 발전의 중요한 이정표를 세웠으며, 최첨단 연구의 선봉에서는 물러났지만 산업 현장과 차세대 연구의 든든한 기반으로서 그 가치를 이어나갈 것으로 전망된다.

#### **참고 자료**

1. Cartographer - Cartographer documentation, accessed July 31, 2025, https://google-cartographer.readthedocs.io/
2. Introducing Cartographer | Google Open Source Blog, accessed July 31, 2025, https://opensource.googleblog.com/2016/10/introducing-cartographer.html
3. Cartographer is a system that provides real-time simultaneous localization and mapping (SLAM) in 2D and 3D across multiple platforms and sensor configurations. - GitHub, accessed July 31, 2025, https://github.com/cartographer-project/cartographer
4. Real-Time Loop Closure in 2D LIDAR SLAM - Google Research, accessed July 31, 2025, https://research.google.com/pubs/archive/45466.pdf
5. cartographer-project/cartographer_ros: Provides ROS integration for Cartographer. - GitHub, accessed July 31, 2025, https://github.com/cartographer-project/cartographer_ros
6. Algorithm walkthrough for tuning - Cartographer ROS Integration - Read the Docs, accessed July 31, 2025, https://google-cartographer-ros.readthedocs.io/en/latest/algo_walkthrough.html
7. Cartographer Mapping Algorithm - Yahboom, accessed July 31, 2025, [http://www.yahboom.net/public/upload/upload-html/1733884335/Cartographer%20Mapping%20Algorithm.html](http://www.yahboom.net/public/upload/upload-html/1733884335/Cartographer Mapping Algorithm.html)
8. Mapping using Cartographer - AgileX LIMO Documentation, accessed July 31, 2025, https://docs.trossenrobotics.com/agilex_limo_docs/demos/slam_nav/cartographer.html
9. Building Maps Using Google Cartographer and the OS1 Lidar Sensor | Ouster, accessed July 31, 2025, https://ouster.com/insights/blog/building-maps-using-google-cartographer-and-the-os1-lidar-sensor
10. IMPROVING GOOGLE'S CARTOGRAPHER 3D MAPPING BY CONTINUOUS-TIME SLAM, accessed July 31, 2025, https://isprs-archives.copernicus.org/articles/XLII-2-W3/543/2017/isprs-archives-XLII-2-W3-543-2017.pdf
11. Cartographer SLAM ROS Integration - Robotics Knowledgebase, accessed July 31, 2025, https://roboticsknowledgebase.com/wiki/state-estimation/Cartographer-ROS-Integration/
12. Cartographer 2D SLAM Loop Closure Demo - YouTube, accessed July 31, 2025, https://www.youtube.com/watch?v=-EQAJOoRqEQ
13. Visual SLAM and Cartographer - Intermodalics, accessed July 31, 2025, https://www.intermodalics.ai/expertise/visual-slam-and-cartographer
14. cartographer 3d loop closure - Google Groups, accessed July 31, 2025, https://groups.google.com/g/google-cartographer/c/2xvEbnuSf6I
15. Towards a robust back-end for pose graph SLAM - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/239763448_Towards_a_robust_back-end_for_pose_graph_SLAM
16. Huber loss - Wikipedia, accessed July 31, 2025, https://en.wikipedia.org/wiki/Huber_loss
17. Real-Time Loop Closure in 2D LIDAR SLAM: Wolfgang Hess, Damon Kohler, Holger Rapp, Daniel Andor | PDF - Scribd, accessed July 31, 2025, https://www.scribd.com/document/484099947/cartographer
18. Comparison of Various SLAM Systems for Mobile Robot in an Indoor Environment - arXiv, accessed July 31, 2025, https://arxiv.org/html/2501.09490v1
19. Comparison of SLAM Mapping Algorithms and Possible Navigational Strategies on Simulated and Real World Conditions, accessed July 31, 2025, https://me336.ancorasir.com/wp-content/uploads/2023/06/Team7-Report.pdf
20. Evaluation of SLAM algorithms for search and rescue applications, accessed July 31, 2025, https://eprints.whiterose.ac.uk/id/eprint/201928/1/Evaluation_of_SLAM_algorithms_for_Search_and_Rescue_applications.pdf
21. snktshrma/Ardupilot-Cartographer-SLAM - GitHub, accessed July 31, 2025, https://github.com/snktshrma/Ardupilot-Cartographer-SLAM
22. Benchmark of Visual SLAM Algorithms: ORB-SLAM2 vs RTAB-Map* | Request PDF, accessed July 31, 2025, https://www.researchgate.net/publication/335345350_Benchmark_of_Visual_SLAM_Algorithms_ORB-SLAM2_vs_RTAB-Map
23. A Simultaneous Localization and Mapping System Using the Iterative Error State Kalman Filter Judgment Algorithm for Global Navigation Satellite System - MDPI, accessed July 31, 2025, https://www.mdpi.com/1424-8220/23/13/6000
24. Review of Visual Simultaneous Localization and Mapping Based on Deep Learning - MDPI, accessed July 31, 2025, https://www.mdpi.com/2072-4292/15/11/2740
25. What is the current state-of-the-art for SLAM right now? : r/robotics - Reddit, accessed July 31, 2025, https://www.reddit.com/r/robotics/comments/1h1o9v6/what_is_the_current_stateoftheart_for_slam_right/
26. [2108.01654] Comparison of modern open-source Visual SLAM approaches - ar5iv - arXiv, accessed July 31, 2025, https://ar5iv.labs.arxiv.org/html/2108.01654
27. 2DLIW-SLAM:2D LiDAR-Inertial-Wheel Odometry with Real-Time Loop Closure - arXiv, accessed July 31, 2025, https://arxiv.org/html/2404.07644v5
28. 2D Mapping using Google Cartographer and RPLidar with Raspberry Pi - Medium, accessed July 31, 2025, https://medium.com/robotics-weekends/2d-mapping-using-google-cartographer-and-rplidar-with-raspberry-pi-a94ce11e44c5
29. Cartographer Map Building - Waveshare Wiki, accessed July 31, 2025, https://www.waveshare.com/wiki/Cartographer_Map_Building
30. Tuning methodology - Cartographer ROS documentation, accessed July 31, 2025, https://google-cartographer-ros.readthedocs.io/en/latest/tuning.html
31. docs/source/tuning.rst / ba33291392d846926633375a5a6f8db547bfbf34 / brian.lee / cartographer_ros - GitLab for research at UTS, accessed July 31, 2025, https://code.research.uts.edu.au/13110756/cartographer_ros/-/blob/ba33291392d846926633375a5a6f8db547bfbf34/docs/source/tuning.rst
32. Cartographer Modular Resource | SLAM - Viam Documentation, accessed July 31, 2025, https://docs.viam.com/operate/reference/services/slam/cartographer/
33. Tuning 3D Cartographer to work with a HDL-32 / Issue #462 - GitHub, accessed July 31, 2025, https://github.com/googlecartographer/cartographer_ros/issues/462
34. Going further - Cartographer ROS documentation, accessed July 31, 2025, https://google-cartographer-ros.readthedocs.io/en/latest/going_further.html
35. Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey - arXiv, accessed July 31, 2025, https://arxiv.org/html/2505.12384v1
36. Challenges of Indoor SLAM: A multi-modal multi-floor dataset for SLAM evaluation - arXiv, accessed July 31, 2025, https://arxiv.org/html/2306.08522
37. Summary of the Pros and Cons of the SLAM Methods - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/figure/Summary-of-the-Pros-and-Cons-of-the-SLAM-Methods_tbl1_329873195
38. Cartographer SLAM Method for Optimization with an Adaptive Multi-Distance Scan Scheduler - MDPI, accessed July 31, 2025, https://www.mdpi.com/2076-3417/10/1/347
39. [2505.14068] Place Recognition Meet Multiple Modalitie: A Comprehensive Review, Current Challenges and Future Directions - arXiv, accessed July 31, 2025, https://arxiv.org/abs/2505.14068
40. [2302.06365] Evolution of SLAM: Toward the Robust-Perception of Autonomy - arXiv, accessed July 31, 2025, https://arxiv.org/abs/2302.06365
41. [2103.05041] Advances in Inference and Representation for Simultaneous Localization and Mapping - arXiv, accessed July 31, 2025, https://arxiv.org/abs/2103.05041
42. SemanticSLAM: Learning based Semantic Map Construction and Robust Camera Localization - arXiv, accessed July 31, 2025, https://arxiv.org/pdf/2401.13076
43. Review on VSLAM based on deep learning - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/385516006_Review_on_VSLAM_based_on_deep_learning
44. Deep Learning for Visual SLAM: The State-of-the-Art and Future Trends - MDPI, accessed July 31, 2025, https://www.mdpi.com/2079-9292/12/9/2006
45. Deep learning-assisted SLAM to reduce computational : r/computervision - Reddit, accessed July 31, 2025, https://www.reddit.com/r/computervision/comments/1m188vk/deep_learningassisted_slam_to_reduce_computational/
46. This repository primarily organizes papers, code, and other relevant materials related to Active SLAM and Robotic Exploration. - GitHub, accessed July 31, 2025, https://github.com/DoongLi/awesome-Active-SLAM
47. 3D-Vision-World/awesome-NeRF-and-3DGS-SLAM - GitHub, accessed July 31, 2025, https://github.com/3D-Vision-World/awesome-NeRF-and-3DGS-SLAM
48. Benchmarking SLAM Algorithms in the Cloud: The SLAM Hive Benchmarking Suite - arXiv, accessed July 31, 2025, https://arxiv.org/html/2406.17586v2
49. Freight100 OEM Base - Zebra Technologies, accessed July 31, 2025, https://www.zebra.com/content/dam/archive_zebra_dam/en_gb/brief/application/oem-brief-application-robotics-automation-en-gb.pdf
50. Fetch Robotics vendor profile in the Mobile Robot Directory, accessed July 31, 2025, https://www.mobile-robots.com/manufacturer/fetch-robotics/
51. IndoorNav - Clearpath Robotics, accessed July 31, 2025, https://clearpathrobotics.com/indoornav/
52. [2302.13613] Evaluation of Lidar-based 3D SLAM algorithms in SubT environment - arXiv, accessed July 31, 2025, https://arxiv.org/abs/2302.13613


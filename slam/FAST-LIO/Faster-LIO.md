# Faster-LIO (iVox 기반)
[FAST-LIO](./index.md)



LiDAR-Inertial Odometry (LIO)는 현대 자율 이동 로봇 및 자율주행 시스템의 핵심 기술로 부상했다. 이 기술은 3D LiDAR (Light Detection and Ranging) 센서가 제공하는 정밀하고 밀도 높은 3차원 환경 정보와 관성 측정 장치(IMU, Inertial Measurement Unit)가 제공하는 고주파의 운동 정보를 융합하여, 위성 항법 시스템(GNSS)의 신호가 도달하지 않는 실내나 도심 협곡과 같은 환경에서도 강건하고 정확한 6자유도(6-DOF) 상태 추정을 가능하게 한다.1 LiDAR는 주변 환경의 기하학적 구조를 정확하게 측정할 수 있지만, 상대적으로 낮은 측정 주파수(일반적으로 10-20 Hz)로 인해 빠른 움직임이 발생할 경우 측정된 포인트 클라우드에 운동 왜곡(motion distortion)이 발생하기 쉽다. 반면, IMU는 수백 Hz의 높은 주파수로 가속도와 각속도를 측정하여 빠른 움직임을 포착할 수 있지만, 측정값에 포함된 노이즈와 바이어스가 시간에 따라 적분되면서 드리프트(drift)가 빠르게 누적되는 단점을 가진다. LIO는 이 두 센서의 상보적인 특성을 활용하여 단일 센서의 한계를 극복하고, 정확성과 강건성을 동시에 확보하는 것을 목표로 한다.

초기 LiDAR 기반 SLAM 시스템들은 LOAM(LiDAR Odometry and Mapping) 3과 같이 환경에서 평면이나 엣지와 같은 기하학적 특징점(feature)을 추출하고, 연속된 스캔 간에 이러한 특징점들을 정합하여 움직임을 추정하는 방식을 주로 사용했다. 이 접근법은 연산량을 줄이는 데 효과적이었으나, 복도나 터널과 같이 특징이 부족한(feature-less) 환경에서는 정합 성능이 급격히 저하되는 근본적인 한계를 내포했다. 또한, 특징점 추출 과정 자체가 상당한 연산 부하를 유발하며, LiDAR 센서의 종류나 스캔 패턴에 따라 특징 추출 알고리즘을 조정해야 하는 번거로움도 존재했다.3 이러한 기술적 난제들은 LIO 기술이 더 넓은 범위의 응용 분야로 확장되기 위해 해결해야 할 중요한 과제로 남았다.


특징점 기반 방식의 한계를 극복하기 위한 노력은 LIO 기술의 발전을 촉진했다. LIO-SAM 4과 같은 시스템은 특징점 기반 접근법을 유지하면서도 IMU 사전적분(pre-integration)을 팩터 그래프 최적화(factor graph optimization)에 긴밀하게 결합(Tightly-coupled)하여 정확도를 크게 향상시켰다. 그러나 LIO 연구의 패러다임을 바꾼 중요한 전환점은 필터 기반의 FAST-LIO 계열 6의 등장이었다. 이 시스템들은 비선형 최적화 대신 반복 확장 칼만 필터(Iterated Extended Kalman Filter, IEKF)를 사용하여 높은 수준의 정확도를 유지하면서도 뛰어난 실시간성을 달성했다.

그 정점에 있는 시스템이 바로 **FAST-LIO2** 3이다. FAST-LIO2는 LIO 분야의 새로운 기준(State-of-the-Art, SOTA)을 제시하며 두 가지 혁신적인 아이디어를 도입했다. 첫째, **특징점 없는 직접 정합(Direct Registration without Features)** 방식을 채택했다.3 이는 LiDAR 스캔에서 얻은 원시 포인트(raw points)를 특징 추출 과정 없이 직접적으로 지도에 정합하는 방식이다. 이로써 특징점 추출 모듈을 완전히 제거하여 연산 효율성을 높이고, 알고리즘의 복잡도를 낮추었으며, 환경에 존재하는 미세한 기하학적 구조까지 모두 활용하여 정합 정확도를 극대화했다. 둘째, 맵 관리를 위한 새로운 자료구조로 **증분적 k-d 트리(incremental k-d Tree, `ikd-Tree`)**를 제안했다.1 기존 LIO 시스템에서는 새로운 포인트가 맵에 추가될 때마다 k-d 트리를 처음부터 다시 구축해야 했으며, 이는 상당한 연산 병목을 유발했다. 

`ikd-Tree`는 점의 삽입, 삭제, 그리고 트리 구조의 재균형(re-balancing)을 매우 효율적으로 지원하여, 실시간으로 대규모 포인트 클라우드 맵을 유지 관리하는 문제를 해결했다. 이 두 가지 혁신 덕분에 FAST-LIO2는 높은 정확도와 강건성, 그리고 빠른 처리 속도를 동시에 달성하며 LIO 연구의 새로운 방향을 제시했다.


FAST-LIO2는 LIO 기술을 한 단계 진보시켰지만, 그 성공은 역설적으로 새로운 연산 병목을 부각시켰다. 특징점 없는 직접 정합 방식은 정합에 사용되는 포인트의 수를 크게 증가시켰고, 이는 각 포인트에 대해 맵에서 대응점(correspondence)을 찾기 위한 최근접 이웃(k-Nearest Neighbor, k-NN) 탐색의 연산 부하를 가중시키는 결과를 낳았다. `ikd-Tree`가 맵의 *업데이트* 비용을 획기적으로 줄였지만, 수많은 포인트에 대한 k-NN *탐색* 자체는 여전히 시스템의 전체 처리 속도를 제한하는 핵심적인 병목으로 남게 된 것이다.1 LIO 기술의 발전 과정은 이처럼 하나의 문제를 해결하면 또 다른 문제가 수면 위로 떠오르는 연쇄적인 과정으로 볼 수 있다. 특징점 기반 방식의 환경 의존성 문제를 해결하기 위해 도입된 직접 정합 방식이 k-NN 탐색이라는 새로운 연산 병목을 낳은 것이다.

이러한 배경 속에서 **Faster-LIO** 10가 등장했다. Faster-LIO는 이 새로운 병목 현상을 정면으로 겨냥한 시스템이다. 그 핵심 목표는 FAST-LIO2의 강건하고 검증된 IESKF 상태 추정 프레임워크의 장점은 그대로 계승하되, k-NN 탐색과 맵 관리를 담당하는 자료구조를 `ikd-Tree`에서 **`iVox`(incremental Voxel)**라는 새로운 구조로 대체하여 처리 속도를 극한까지 끌어올리는 것이었다.10 이 접근법은 LIO 시스템의 전체 성능이 상태 추정 알고리즘뿐만 아니라, 그 알고리즘에 데이터를 공급하는 맵 자료구조와의 유기적인 상호작용에 의해 결정된다는 점을 명확히 보여준다. Faster-LIO는 맵 자료구조의 혁신만으로도 전체 시스템의 성능을 어떻게 획기적으로 개선할 수 있는지를 입증한 사례라 할 수 있다. 아래 표는 주요 LIO 시스템의 핵심 특징을 비교하여 Faster-LIO의 기술적 위치를 명확히 보여준다.

| 시스템 (System) | 백엔드 방식 (Backend Method)                   | 맵 자료구조 (Map Data Structure)   | 특징점 추출 (Feature Extraction) | 핵심 기여 (Key Contribution)                                 |
| --------------- | ---------------------------------------------- | ---------------------------------- | -------------------------------- | ------------------------------------------------------------ |
| **LOAM**        | Odometry + Mapping (Scan-to-Scan, Scan-to-Map) | k-d Tree                           | Yes                              | 엣지/평면 특징점 기반 고정밀 Odometry 및 Mapping 분리        |
| **LIO-SAM**     | Factor Graph Optimization (iSAM2)              | Voxel-based Downsampling, k-d Tree | Yes                              | IMU 사전적분을 활용한 Tightly-coupled 팩터 그래프 최적화     |
| **FAST-LIO2**   | Iterated Error State Kalman Filter (IESKF)     | **ikd-Tree**                       | No                               | 직접 포인트 정합 및 증분적 k-d 트리를 통한 고속/고정밀 LIO   |
| **Faster-LIO**  | Iterated Error State Kalman Filter (IESKF)     | **iVox (incremental Voxel)**       | No                               | `iVox`를 이용한 병렬 근사 k-NN 탐색으로 `FAST-LIO2` 대비 속도 극대화 |


Faster-LIO의 혁신을 이해하기 위해서는 그 기반이 되는 FAST-LIO2의 상태 추정 프레임워크를 먼저 이해해야 한다. Faster-LIO는 이 프레임워크를 거의 그대로 계승하여, 검증된 수학적 견고함 위에서 속도 향상에 집중했다.10 이 아키텍처의 핵심은 LiDAR와 IMU 측정값을 긴밀하게 결합하는 Tightly-Coupled 방식의 IESKF이다.


Tightly-Coupled 융합 방식은 LiDAR와 IMU의 원시 측정값(또는 이로부터 파생된 잔차)을 단일 최적화 문제 내에서 함께 처리하는 것을 의미한다.9 이는 각 센서의 측정 오차 모델과 상태 변수 간의 모든 상호 상관관계를 명시적으로 고려할 수 있게 해주어, Loosely-coupled 방식(각 센서의 추정 결과를 사후에 결합하는 방식)에 비해 일반적으로 더 높은 정확도와 강건성을 제공한다. Faster-LIO가 채택한 IESKF(Iterated Error State Kalman Filter)는 이러한 Tightly-Coupled 융합을 필터 기반으로 구현한 것이다. 특히 오차 상태(error state)를 추정함으로써 회전(rotation)과 같은 비유클리드 공간에 존재하는 상태 변수를 효과적으로 다룰 수 있으며, 반복(iteration)을 통해 비선형적인 측정 모델의 선형화 오차를 최소화하여 최적화 기반 방식에 근접하는 정확도를 달성한다.13


시스템이 추정하고자 하는 모든 변수는 상태 벡터(state vector) $\mathbf{x}$로 정의된다. Faster-LIO는 FAST-LIO2의 상태 벡터 정의를 따르며, 이는 시스템의 운동학적 상태와 IMU 센서의 내부 파라미터를 포함한다.14 일반적인 상태 벡터는 다음과 같이 구성된다.
$$
\mathbf{x} \triangleq \begin{bmatrix} {}^{G}{\mathbf{R}}_{I} \\ {}^{G}\mathbf{p}_{I} \\ {}^{G}\mathbf{v}_{I} \\ \mathbf{b}_{\omega} \\ \mathbf{b}_{\mathbf{a}} \\ {}^{G}\mathbf{g} \end{bmatrix} \in SO(3) \times \mathbb{R}^{15}
$$
여기서 각 항목은 다음과 같다:

- ${}^{G}{\mathbf{R}}_{I} \in SO(3)$: 전역 좌표계(G, Global frame)를 기준으로 한 IMU의 자세(회전 행렬).
- ${}^{G}\mathbf{p}_{I} \in \mathbb{R}^{3}$: 전역 좌표계 기준의 IMU 위치.
- ${}^{G}\mathbf{v}_{I} \in \mathbb{R}^{3}$: 전역 좌표계 기준의 IMU 속도.
- $\mathbf{b}_{\omega} \in \mathbb{R}^{3}$: 자이로스코프의 바이어스.
- $\mathbf{b}_{\mathbf{a}} \in \mathbb{R}^{3}$: 가속도계의 바이어스.
- ${}^{G}\mathbf{g} \in \mathbb{R}^{3}$: 전역 좌표계에서 표현된 중력 벡터.

일부 구현에서는 LiDAR와 IMU 간의 외부 파라미터(extrinsic parameters) ${}^{I}\mathbf{R}_{L}, {}^{I}\mathbf{p}_{L}$도 상태 벡터에 포함하여 온라인으로 보정하기도 한다.14






칼만 필터의 예측(prediction) 단계에서는 운동 모델을 사용하여 현재 상태와 제어 입력을 기반으로 다음 시점의 상태를 예측한다. LIO 시스템에서 운동 모델은 고주파로 수신되는 IMU 측정값을 통해 구현된다. IMU가 측정한 각속도 $\omega_m$과 가속도 $\mathbf{a}_m$을 사용하여 상태 벡터의 동역학을 다음과 같은 연속 시간 미분 방정식으로 모델링할 수 있다.13

```
\begin{aligned}
{}^{G}\dot{\mathbf{p}}_{I} &= {}^{G}\mathbf{v}_{I} \\
{}^{G}\dot{\mathbf{v}}_{I} &= {}^{G}{\mathbf{R}}_{I}(\mathbf{a}_m - \mathbf{b}_{\mathbf{a}} - \mathbf{n}_{\mathbf{a}}) + {}^{G}\mathbf{g} \\
{}^{G}\dot{\mathbf{R}}_{I} &= {}^{G}{\mathbf{R}}_{I} \lfloor \omega_m - \mathbf{b}_\omega - \mathbf{n}_\omega \rfloor_{\wedge} \\
\dot{\mathbf{b}}_{\omega} &= \mathbf{n}_{\mathbf{b}\omega}, \quad \dot{\mathbf{b}}_{\mathbf{a}} = \mathbf{n}_{\mathbf{b}\mathbf{a}}
\end{aligned}
```

여기서 $\mathbf{n}$ 항들은 각 측정값 및 바이어스의 노이즈를 나타내며, $\lfloor \cdot \rfloor_{\wedge}$는 벡터를 반대칭 행렬(skew-symmetric matrix)로 변환하는 연산자이다. 이 연속 시간 모델을 IMU 측정 주기 $\Delta t$에 맞춰 이산화하고, 마지막 LiDAR 스캔이 처리된 시점부터 현재 시점까지의 모든 IMU 측정값을 순차적으로 적분하여 현재 상태를 전파(propagate)한다. 이 과정은 새로운 LiDAR 스캔이 도착했을 때 각 포인트의 운동 왜곡을 보정하고, 포인트 클라우드 정합을 위한 정확한 초기 추정치를 제공하는 데 결정적인 역할을 한다.






칼만 필터의 갱신(update) 단계는 측정 모델을 통해 예측된 상태와 실제 측정값 간의 오차, 즉 잔차(residual)를 계산하고 이를 최소화하는 방향으로 상태를 보정하는 과정이다. Faster-LIO에서는 새로운 LiDAR 스캔의 각 포인트가 측정값의 역할을 한다. 먼저, IMU 전파를 통해 운동 왜곡이 보정된 각 LiDAR 포인트 ${L}\mathbf{p}_j$는 현재 추정된 상태(자세 ${G}\widehat{\mathbf{R}}_{I_k}$, 위치 ${G}\widehat{\mathbf{p}}_{I_k}$)를 사용하여 전역 좌표계로 변환된다.

```
{}^{G}\mathbf{p}_j = {}^{G}\widehat{\mathbf{R}}_{I_k} ({}^{I}\mathbf{R}_{L} {}^{L}\mathbf{p}_j + {}^{I}\mathbf{p}_{L}) + {}^{G}\widehat{\mathbf{p}}_{I_k}
```

그 후, 변환된 포인트 ${G}\mathbf{p}_j$에 대해, 맵에서 가장 가까운 이웃 포인트 5개를 찾아 이들이 형성하는 지역 평면(local plane)과의 수직 거리를 잔차 $\mathbf{z}_j$로 정의한다.1 이 포인트-평면(point-to-plane) 잔차는 다음과 같이 계산된다.

```
\mathbf{z}_j = \mathbf{u}_j^T ({}^{G}\mathbf{p}_j - \mathbf{q}_j)
```

여기서 $\mathbf{u}_j$는 이웃 포인트들로부터 계산된 평면의 법선 벡터(normal vector)이고, $\mathbf{q}_j$는 이웃 포인트들의 중심점(centroid)으로 평면 위의 한 점을 나타낸다. 이 잔차 $\mathbf{z}_j$는 현재 상태 추정치가 얼마나 정확한지를 나타내는 척도가 되며, 이상적으로는 0에 가까워야 한다.






수집된 모든 포인트에 대한 잔차 벡터 $\mathbf{z}$와, 이 잔차를 상태 변수의 오차에 대해 편미분한 자코비안 행렬 $\mathbf{H}$가 구성되면, 표준 칼만 필터 갱신 수식을 사용하여 상태 벡터와 공분산을 갱신한다. IESKF는 이 갱신 과정을 여러 번 반복(iterate)하여 비선형 측정 모델로 인한 오차를 줄인다.13 FAST-LIO 계열의 핵심적인 연산 효율성 비결 중 하나는 칼만 이득(Kalman Gain) 

<code>\mathbf{K}</code>를 계산하는 방식에 있다. 전통적인 칼만 이득 계산은 측정값의 차원(수천 개에 달하는 LiDAR 포인트 수)에 비례하는 행렬의 역행렬을 계산해야 해 매우 비효율적이다. FAST-LIO는 Sherman-Morrison-Woodbury 공식을 활용하여 칼만 이득 계산의 복잡도를 측정값의 차원이 아닌 상태 벡터의 차원(약 20~30)에 의존하도록 변환했다.3 이 덕분에 수많은 LiDAR 포인트를 측정값으로 사용하면서도 실시간성을 유지할 수 있다.

이처럼 Faster-LIO의 상태 추정 알고리즘은 FAST-LIO2의 검증된 프레임워크를 그대로 사용한다. 혁신은 이 프레임워크의 '내부'가 아니라, 프레임워크에 데이터를 '공급'하는 과정에서 일어난다. 즉, 측정 모델을 구성하는 데 필수적인 '최근접 이웃 탐색' 단계를 가속화하는 것이 Faster-LIO의 핵심이며, 이는 다음 장에서 설명할 `iVox` 자료구조를 통해 달성된다. 이러한 모듈식 접근은 검증된 필터의 안정성을 해치지 않으면서 특정 병목 구간만을 집중적으로 개선하는 효율적인 개발 전략의 성공 사례로 볼 수 있다.






Faster-LIO가 FAST-LIO2를 뛰어넘는 속도를 달성할 수 있었던 비결은 전적으로 `iVox`(incremental Voxel)라는 새로운 맵 자료구조에 있다. 이는 단순히 `ikd-Tree`를 대체한 것을 넘어, LIO 문제의 본질에 대한 깊은 통찰을 바탕으로 한 설계 철학의 전환을 의미한다.






FAST-LIO2의 `ikd-Tree`는 동적으로 변화하는 포인트 클라우드 맵에서 k-d 트리의 구조를 효율적으로 유지하는 데 초점을 맞춘 정교한 자료구조다.3 그 목적은 언제나 수학적으로 가장 정확한 최근접 이웃(strict k-NN)을 빠르게 찾는 것이었다. 그러나 Faster-LIO의 개발자들은 LIO라는 특정 응용 분야의 맥락에서 이러한 '엄격함'이 과연 필수적인지에 대해 근본적인 질문을 던졌다.1 LIO 시스템에서는 IMU의 고주파 측정값을 통해 현재 로봇의 위치를 상당히 정확하게 예측할 수 있다. 이는 새로 들어온 LiDAR 스캔의 포인트들이 맵에 투영될 위치가 매우 좁은 영역으로 한정된다는 것을 의미한다. 따라서 전역 맵 전체를 대상으로 완벽한 k-NN 해를 찾는 것은 과도한 연산이며, 예측된 위치 주변에서 '충분히 좋은(good enough)' 이웃들을 '훨씬 빠르게(fast enough)' 찾는 것이 시스템 전체의 실시간성에 더 유리하다는 결론에 도달했다.

이러한 "Good enough, fast enough" 철학은 **근사 k-NN(Approximate k-NN, a-kNN)** 탐색이라는 개념으로 구체화되었다. 즉, 약간의 정확성을 희생하여 수학적인 최적 해가 아닐 수 있는 이웃을 찾더라도, 그 속도를 압도적으로 향상시켜 전체 시스템의 처리량을 극대화하는 것이다. `iVox`는 바로 이 철학을 구현하기 위해 설계된 자료구조다.10






`iVox`는 3차원 공간을 고정된 크기의 정육면체 단위인 복셀(voxel)로 분할하여 포인트 클라우드를 관리한다. 하지만 모든 공간에 대해 복셀을 미리 생성하는 밀집 복셀(dense voxel) 방식이 아니라, 포인트가 실제로 존재하는 위치의 복셀만을 동적으로 생성하고 관리하는 희소 복셀(sparse voxel) 방식을 채택했다. 이는 방대한 3차원 공간을 다루면서도 메모리 사용량을 최소화하기 위함이다.

이러한 동적이고 희소한 복셀들을 효율적으로 관리하기 위해 `iVox`는 내부적으로 해시 맵(hash map)을 사용한다.10 3차원 복셀의 좌표 인덱스 $\mathbf{v} = [v_x, v_y, v_z]$는 다음과 같은 해시 함수를 통해 1차원의 해시 키(key) id로 변환된다.10
$$
id = \text{hash}(\mathbf{v}) = (v_x \cdot n_1) \oplus (v_y \cdot n_2) \oplus (v_z \cdot n_3) \pmod{N}
$$
여기서 $v_x, v_y, v_z$는 포인트의 좌표를 복셀 크기로 나눈 정수 인덱스, $n_1, n_2, n_3$는 해시 충돌을 줄이기 위한 큰 소수(prime numbers), $\oplus$는 비트 단위 XOR 연산, 그리고 $N$은 해시 맵의 크기를 나타낸다. 해시 맵을 사용함으로써 특정 복셀에 포인트를 삽입, 삭제하거나 해당 복셀에 저장된 포인트 리스트를 조회하는 연산은 평균적으로 상수 시간 복잡도, 즉 $O(1)$에 수행될 수 있다.10 이는 트리의 깊이에 따라 탐색 시간이 달라지는 트리 기반 구조에 비해 매우 빠르고 예측 가능한 성능을 제공한다.


Faster-LIO는 한 복셀 내에 여러 개의 포인트가 존재할 경우, 이 포인트들을 관리하는 방식에 따라 두 가지 대안적인 구현을 제공한다. 이는 복셀 내 포인트의 밀도에 따라 최적의 성능을 선택할 수 있도록 유연성을 부여한다.10

1. **Linear iVox**: 가장 간단한 방식으로, 복셀 내에 포함된 포인트들을 단순한 동적 배열이나 연결 리스트(list)로 저장한다. 이 방식은 구조가 매우 단순하고 추가적인 오버헤드가 없어 복셀 당 포인트 수가 적을 때 가장 효율적이다. k-NN 탐색 시에는 해당 복셀 내의 모든 포인트를 순차적으로 검사해야 하므로, 복셀 내 포인트 수를 $n$이라 할 때 시간 복잡도는 $O(n)$이 된다.10
2. **PHC (Pseudo Hilbert Curve) iVox**: 복셀 내 포인트 밀도가 높아져 선형 탐색이 비효율적이 될 경우를 대비한 방식이다. 유사 힐버트 곡선(Pseudo Hilbert Curve, PHC)은 3차원 공간 좌표를 1차원 인덱스로 매핑하면서 공간적 인접성(spatial locality)을 최대한 보존하는 특성을 가진 공간 채움 곡선(space-filling curve)이다. PHC iVox는 복셀 내의 포인트들을 이 PHC 인덱스 순서로 정렬하여 저장한다. 덕분에 3차원 공간에서 가까운 포인트들은 1차원 인덱스 상에서도 서로 가까이 위치하게 된다. 따라서 특정 포인트의 이웃을 찾을 때, 1차원 인덱스 상에서 해당 포인트의 앞뒤로 일정 범위만 탐색하면 되므로, 전체를 탐색하는 것보다 훨씬 효율적인 탐색이 가능하다.10


`iVox`의 진정한 성능은 자료구조 자체의 효율성과 병렬 처리와의 시너지에서 발휘된다. Faster-LIO는 멀티코어 CPU의 성능을 최대한 활용하여 k-NN 탐색 과정을 병렬화한다.10

1. **근사 탐색 범위**: 특정 쿼리 포인트 $P$에 대한 k-NN을 찾기 위해, 시스템은 먼저 $P$가 속한 중심 복셀을 찾는다. 그리고나서, 가장 가까운 이웃이 인접 복셀에 존재할 가능성을 고려하여, 중심 복셀을 둘러싸는 주변의 26개 이웃 복셀(즉, 3x3x3 큐브 형태의 영역)까지 탐색 범위를 확장한다.10 이처럼 고정된 주변 영역만을 탐색하는 것이 바로 '근사(approximate)'의 핵심이다. 이는 수학적으로 가장 가까운 이웃이 이 27개 복셀 바깥에 존재할 가능성을 배제하는 대신, 탐색 속도를 비약적으로 향상시키는 트레이드오프이다.
2. **병렬 처리**: 한 LiDAR 스캔에 포함된 수천 개의 포인트 각각에 대한 k-NN 탐색은 서로 독립적으로 수행될 수 있는 작업이다. `iVox`의 해시 맵 기반 구조는 여러 스레드가 동시에 맵의 다른 영역에 접근할 때 발생하는 잠금 경합(lock contention)이 거의 없어 병렬 처리에 매우 유리하다. Faster-LIO는 Intel TBB(Threading Building Blocks)와 같은 라이브러리를 사용하여 이 탐색 과정을 다수의 스레드에 분배하고, 모든 포인트에 대한 k-NN 탐색을 동시에 병렬적으로 수행한다. 이 병렬화는 전체 스캔 처리 시간을 획기적으로 단축시키는 결정적인 요소이다.

결론적으로, Faster-LIO의 성공은 LIO 문제의 특성(IMU를 통한 지역 탐색)을 정확히 파악하고, 이에 최적화된 자료구조(`iVox`)와 구현 전략(병렬 처리)을 선택한 결과물이다. 이는 범용적인 해결책(strict k-NN)을 고수하기보다, 특정 응용 분야의 제약 조건을 적극적으로 활용하여 특화된 해결책(approximate k-NN)을 설계했을 때 얼마나 큰 성능 향상을 이룰 수 있는지를 보여주는 탁월한 공학적 사례이다.


Faster-LIO는 `iVox`라는 혁신적인 자료구조를 FAST-LIO2의 견고한 상태 추정 프레임워크에 통합한 시스템이다. 이 섹션에서는 데이터가 입력되어 최종 상태 추정치가 출력되기까지의 전체 파이프라인과 시스템을 구성하는 주요 기술적 요소들을 상세히 설명한다.


Faster-LIO의 데이터 처리 파이프라인은 LiDAR 포인트 클라우드와 IMU 데이터를 입력받아 실시간으로 로봇의 6-DOF 포즈를 추정하고 맵을 생성하는 일련의 과정으로 구성된다.

1. **입력 (Input)**: 시스템은 ROS(Robot Operating System) 환경에서 표준 메시지 타입으로 LiDAR 포인트 클라우드(`/livox/lidar`, `/velodyne_points` 등)와 IMU 데이터(`/livox/imu` 등)를 구독(subscribe)한다.
2. **전처리 (Preprocessing)**: 새로운 LiDAR 스캔이 도착하면, 가장 먼저 스캔 내 포인트들의 운동 왜곡을 보정(de-skewing)하는 작업을 수행한다. LiDAR 센서는 한 번에 모든 방향을 측정하는 것이 아니라, 일정 시간 동안 회전하며 순차적으로 포인트를 측정한다. 이 시간 동안 로봇이 움직이면 스캔 데이터가 휘어지는 현상이 발생한다. Faster-LIO는 스캔 시작 시점과 종료 시점 사이의 IMU 측정값들을 적분하여, 각 포인트가 측정된 정확한 시점의 로봇 포즈를 보간(interpolate)하고, 모든 포인트를 스캔 종료 시점 기준으로 재배치함으로써 이 왜곡을 제거한다.
3. **상태 추정 (State Estimation)**: 이 단계는 FAST-LIO2와 동일한 IESKF 프레임워크를 기반으로 하며, 예측과 갱신 두 과정으로 나뉜다.
   - **예측 (Prediction)**: 마지막 상태 추정 이후부터 현재까지 수신된 모든 IMU 데이터를 사용하여 운동 모델에 따라 시스템의 상태(위치, 속도, 자세 등)를 현재 시점으로 전파한다. 이는 LiDAR 측정 없이도 시스템의 상태를 예측하고, 갱신 단계를 위한 사전 정보(prior)를 생성하는 역할을 한다.
   - **갱신 (Update)**: 왜곡이 보정된 LiDAR 스캔의 각 포인트를 예측된 포즈를 이용해 전역 맵 좌표계로 변환한다. 그 후, 각 포인트에 대해 `iVox` 맵을 사용하여 병렬적으로 근사 k-NN을 탐색한다. 찾아낸 이웃 포인트들을 이용해 지역 평면을 구성하고, 포인트와 평면 간의 거리 잔차를 계산한다. 수집된 모든 잔차와 자코비안을 IESKF 갱신 수식에 적용하여 최종적으로 상태 벡터와 공분산을 갱신한다. 이 갱신 과정은 비선형성을 완화하기 위해 여러 번 반복된다.
4. **맵 업데이트 (Map Update)**: IESKF 갱신을 통해 최적화된 현재 포즈가 확정되면, 이 포즈를 사용하여 현재 LiDAR 스캔의 포인트들을 전역 맵에 통합한다. `iVox`의 해시 맵 기반 증분적 삽입 기능 덕분에 이 과정은 매우 효율적으로 수행된다. 맵은 슬라이딩 윈도우 방식으로 관리되어, 현재 로봇 위치에서 너무 멀리 떨어진 오래된 포인트들은 맵에서 제거될 수 있다.


Faster-LIO는 `iVox`의 병렬 탐색 성능을 극대화하기 위해 현대적인 C++ 개발 환경과 고성능 라이브러리들을 적극적으로 활용한다. 이는 단순한 구현 선택을 넘어, 알고리즘의 핵심 성능 목표와 직결된 전략적 결정이라 할 수 있다.11

- **필수 라이브러리**: 시스템을 빌드하고 실행하기 위해서는 다음과 같은 핵심 라이브러리들이 필요하다.11
  - **ROS (Melodic 또는 Noetic)**: 로봇 응용 프로그램을 위한 표준 미들웨어로, 센서 데이터의 입출력과 모듈 간 통신을 담당한다.
  - **PCL (Point Cloud Library)**: 포인트 클라우드 데이터의 처리, 필터링, 정합 등 다양한 알고리즘을 제공하는 핵심 라이브러리이다.
  - **Eigen**: 행렬, 벡터, 선형 대수 연산을 위한 고성능 C++ 템플릿 라이브러리로, SLAM 시스템의 모든 수학적 계산에 필수적이다.
  - **glog, yaml-cpp**: 로깅 및 설정 파일 관리를 위한 유틸리티 라이브러리이다.
- **컴파일러 및 표준 요구사항**: Faster-LIO는 C++17 표준을 기반으로 작성되었다. 이는 최신 언어 기능들을 활용하여 코드의 가독성과 성능을 높이기 위함이다. 따라서 시스템을 컴파일하기 위해서는 g++ 9.0 버전 이상의 최신 컴파일러가 필요하다. Ubuntu 18.04와 같이 기본 컴파일러 버전이 낮은 운영체제에서는 사용자가 직접 컴파일러를 업그레이드해야 한다.11
- **병렬 처리 라이브러리**: `iVox`의 병렬 k-NN 탐색을 효율적으로 구현하기 위해 Intel TBB(Threading Building Blocks) 라이브러리를 사용한다. TBB는 고수준의 병렬 프로그래밍 추상화를 제공하여 멀티코어 아키텍처의 성능을 쉽게 활용할 수 있도록 돕는다. 호환성 문제를 피하기 위해 특정 버전의 TBB 라이브러리가 프로젝트 소스 코드의 `thirdparty` 디렉토리에 포함되어 제공된다.11 이처럼 최신 기술 스택의 채택은 Faster-LIO의 병렬 a-kNN 탐색이라는 핵심 아이디어를 현실에서 최고의 성능으로 구현하기 위한 필수적인 기반이다.


Faster-LIO는 다양한 환경과 시나리오에 대응할 수 있도록 여러 실행 모드와 조정 가능한 파라미터를 제공한다.

- **실행 모드**: 사용자는 두 가지 주요 모드로 시스템을 실행할 수 있다.11
  - **온라인 모드**: ROS 런처(`roslaunch`)를 사용하여 시스템을 실행하고, 실제 센서로부터 들어오는 데이터를 실시간으로 처리한다. `rviz`를 통해 실시간으로 생성되는 맵과 추정된 궤적을 시각화할 수 있다.
  - **오프라인 모드**: 사전에 기록된 `rosbag` 파일을 재생하여 저장된 센서 데이터로 시스템을 테스트하고 평가한다. 이는 알고리즘 개발 및 디버깅, 성능 벤치마킹에 유용하다.
- **핵심 파라미터**: `config` 디렉토리의 YAML 파일에는 시스템의 동작을 제어하는 다양한 파라미터가 정의되어 있다. 그중 가장 중요하고 성능에 민감한 파라미터는 `voxel_size`이다.11
  - `voxel_size`는 `iVox` 맵을 구성하는 복셀의 한 변 길이를 미터 단위로 정의한다.
  - 이 값을 너무 작게 설정하면, 동일한 공간을 표현하는 데 더 많은 복셀이 필요하게 되어 메모리 사용량이 증가하고, 포인트들이 여러 복셀에 걸쳐 분산되어 k-NN 탐색 시 더 많은 이웃 복셀을 확인해야 하는 비효율을 초래할 수 있다.
  - 반대로 너무 크게 설정하면, 하나의 복셀이 너무 넓은 영역을 대표하게 되어 공간 해상도가 낮아지고, 결과적으로 포인트-평면 정합의 정확도가 저하될 수 있다.
  - 저자들은 시스템의 추정 결과가 불안정할 경우, `voxel_size`를 더 크게 설정해 볼 것을 권장한다.11 이는 더 넓은 범위의 포인트들을 하나의 복셀에 포함시켜 지역 평면을 추정할 때 노이즈에 더 강건한(robust) 평면을 계산할 수 있기 때문일 것으로 추정된다. 이 파라미터는 환경의 구조적 특성(예: 넓은 야외 vs. 좁은 실내)에 따라 최적값이 달라지므로, 사용자의 신중한 튜닝이 요구된다.


Faster-LIO의 가치는 처리 속도와 정확도라는 두 가지 핵심 지표를 통해 평가된다. 이 시스템의 설계 목표는 FAST-LIO2의 높은 정확도를 최대한 유지하면서 처리 속도를 획기적으로 개선하는 것이었으며, 공개된 벤치마크 결과는 이 목표가 성공적으로 달성되었음을 보여준다.


Faster-LIO의 가장 두드러진 장점은 압도적인 처리 속도다. 이는 `iVox` 자료구조와 병렬 k-NN 탐색 알고리즘이 효과적으로 작동했음을 증명한다.

- **전체 성능 향상**: 다양한 데이터셋에서 테스트한 결과, Faster-LIO는 원본 FAST-LIO2 대비 약 1.5배에서 2배에 달하는 FPS(Frames Per Second) 성능 향상을 보였다.11 구체적으로, Livox Avia와 같은 Solid-state LiDAR를 사용한 환경에서는 초당 1000~2000 프레임(1k-2k Hz)이라는 경이적인 처리 속도를 달성했으며, Velodyne VLP-32와 같은 32-line Spinning LiDAR 환경에서도 100 Hz 이상의 안정적인 성능을 기록했다.10 이는 실시간 처리가 필수적인 로보틱스 응용 분야에서 매우 큰 이점을 제공한다.
- **모듈별 성능 분석**: 속도 향상의 근원은 IESKF의 갱신 단계, 특히 각 포인트에 대한 최근접 이웃을 찾고 잔차를 계산하는 부분에 있다. 논문에 제시된 실험 결과에 따르면, NCLT와 같은 Spinning LiDAR 데이터셋에서 `iVox`는 `ikd-Tree`를 사용했을 때보다 IEKF 단계의 처리 시간을 40%에서 75%까지 단축시켰다.10 이처럼 특정 병목 구간을 집중적으로 개선한 전략이 전체 시스템의 FPS를 200%에서 320%까지 끌어올리는 결과로 이어진 것이다.

성능 향상 폭은 사용된 센서의 특성에 따라 다르게 나타나는 경향을 보인다. Spinning LiDAR 데이터셋(NCLT, UTBM)에서 `iVox`의 시간 절약 효과가 Solid-state LiDAR 데이터셋(AVIA)에서보다 더 크게 나타났다.10 이는 센서의 스캔 패턴과 포인트 분포의 차이에서 기인하는 것으로 분석된다. Spinning LiDAR는 구조화된 형태의 밀도 높은 포인트 링을 생성하는 반면, Livox와 같은 Solid-state LiDAR는 불규칙하고 반복되지 않는 스캔 패턴을 가진다. 

`ikd-Tree`의 균형 잡힌 트리 구조가 불규칙한 포인트 분포를 상대적으로 잘 처리하는 반면, `iVox`의 복셀 기반 지역 탐색은 구조화된 포인트 군집에서 더 큰 효율성 증대를 보였을 수 있다. 이는 최적의 맵 자료구조 선택이 센서의 물리적 특성과 밀접하게 연관되어 있음을 시사한다.

아래 표는 주요 데이터셋에서 Faster-LIO와 다른 SOTA LIO 시스템들의 처리 속도를 정량적으로 비교한 것이다.

| 시스템 (System) | 데이터셋 (Dataset) | 평균 총 시간 (Avg. Total Time) [ms] | FPS   | 속도 향상 (vs. FastLIO2) |
| --------------- | ------------------ | ----------------------------------- | ----- | ------------------------ |
| **FastLIO2**    | AVIA               | 10.32                               | 96.9  | -                        |
| **Faster-LIO**  | AVIA               | 7.91                                | 126.4 | 1.30x                    |
| **FastLIO2**    | NCLT               | 22.84                               | 43.8  | -                        |
| **Faster-LIO**  | NCLT               | 7.02                                | 142.5 | **3.25x**                |
| **LIO-SAM**     | NCLT               | 79.43                               | 12.6  | -                        |

표의 데이터는 10의 Table I을 기반으로 재구성됨. LIO-SAM의 루프 클로저는 비활성화된 상태로 측정됨.


처리 속도의 비약적인 향상이 정확도의 심각한 저하를 대가로 이루어졌다면 그 가치는 퇴색될 것이다. 따라서 Faster-LIO의 근사 k-NN 탐색 전략이 시스템의 전체 정확도에 미치는 영향을 평가하는 것은 매우 중요하다.

- **결론**: 다양한 데이터셋에서의 평가 결과, Faster-LIO는 FAST-LIO2와 **"비교 가능한 수준의 정확도(comparable accuracy)"**를 유지하는 데 성공했다.10 이는 a-kNN 탐색이 LIO의 포인트-평면 정합에 필요한 '충분히 좋은' 이웃을 제공하는 데 문제가 없었음을 의미한다.
- **구체적 수치**: 절대 궤적 오차(Absolute Pose Error, APE)와 상대 궤적 오차(Relative Pose Error, RPE) 지표에서 Faster-LIO는 FAST-LIO2와 대등한 성능을 보였다. 일반적으로 100미터 주행 당 0.5%에서 1.5% 사이의 이동 드리프트(translational drift)를 기록하여, 높은 정밀도를 요구하는 응용 분야에서도 충분히 사용 가능한 수준의 정확도를 입증했다.10 예를 들어, S-FAST_LIO와 FAST-LIO를 비교한 실험에서도 드리프트율은 각각 0.037%와 0.035%로 거의 차이가 없었는데 16, 이는 FAST-LIO 계열 알고리즘의 프레임워크 자체가 매우 견고함을 뒷받침한다.

이러한 결과는 Faster-LIO의 핵심적인 설계 트레이드오프, 즉 엄격한 정확성을 일부 포기하고 근사적인 탐색을 채택한 것이 성공적이었음을 명확히 보여준다. LIO 문제의 특성상, IMU가 제공하는 강력한 사전 정보 덕분에 정합 문제는 매우 지역적인 탐색으로 한정되며, 이 범위 내에서는 근사적으로 찾은 이웃이라도 최종적인 상태 추정 결과에 미치는 악영향이 미미했던 것이다. 아래 표는 NCLT 데이터셋에서의 정확도를 비교한 결과이다.

| 시스템 (System) | 데이터셋 (Dataset) | APE RMSE [m] | RPE [%] (per 100m) |
| --------------- | ------------------ | ------------ | ------------------ |
| **FastLIO2**    | NCLT               | 1.14         | 0.56               |
| **Faster-LIO**  | NCLT               | 1.19         | 0.55               |
| **LIO-SAM**     | NCLT               | 1.45         | 0.70               |

표의 데이터는 10의 Table II를 기반으로 재구성됨. LIO-SAM의 루프 클로저는 비활성화된 상태로 측정됨.

결론적으로, Faster-LIO는 정확도를 거의 희생하지 않으면서도 처리 속도를 대폭 향상시키는 데 성공함으로써, LIO 기술의 실시간 성능 한계를 한 단계 끌어올린 중요한 성과로 평가할 수 있다.


Faster-LIO는 단순히 하나의 빠른 LIO 알고리즘을 넘어, LIO 연구 커뮤니티와 기술 생태계 전반에 상당한 영향을 미쳤다. 그 기술적 위치와 영향력, 내재적 한계, 그리고 이를 기반으로 파생된 다양한 변형 모델들을 통해 Faster-LIO의 현재와 미래를 조망할 수 있다.


Faster-LIO는 LIO 시스템의 성능 병목이 상태 추정 필터뿐만 아니라 맵 관리 자료구조에 있음을 명확히 보여주며, `ikd-Tree`로 대표되던 트리 기반 구조에서 복셀 기반 구조로의 패러다임 전환을 가속화한 중요한 이정표로 평가된다. `iVox`의 성공은 후속 연구들에게 직접적인 영감을 주었다.

- **FR-LIO**: 이 시스템은 Faster-LIO의 `iVox`가 해시 테이블을 사용하기 때문에 드물게 발생할 수 있는 해시 충돌(hash collision)로 인한 성능 저하 가능성을 지적했다.17 이를 해결하기 위해, 로봇 중심의 고정 크기 3D 배열을 사용하는 `RC-Vox`(Robocentric Voxel Map)를 제안했다.2

  `RC-Vox`는 해시 연산 없이 직접적인 배열 인덱싱을 통해 복셀에 접근하므로 이론적으로 더 빠른 접근 속도를 보장한다. 이는 `iVox`의 아이디어를 계승하면서도 그 단점을 개선하려는 명확한 시도로, Faster-LIO가 후속 연구의 디딤돌 역할을 했음을 보여준다.

- **LIO-PPF**: 이 연구는 k-NN 탐색 후 매번 평면을 피팅하는 과정의 비효율성을 지적하며, 평면 정보를 증분적으로 갱신하는 PPF(Plane Pre-fitting) 파이프라인을 제안했다.1 LIO-PPF 논문은 관련 연구를 소개하며 `Faster-LIO`가 엄격한 k-NN 탐색의 불필요성을 인식하고 `iVox`를 대안으로 제시했다고 명시적으로 언급한다.1 이는 Faster-LIO와 `iVox`가 LIO 커뮤니티 내에서 k-NN 탐색 가속화를 위한 표준적인 대안 중 하나로 널리 인정받고 있음을 의미한다.


뛰어난 성능에도 불구하고 Faster-LIO는 그 자체로 완벽한 시스템은 아니며, 몇 가지 명확한 한계를 가지고 있다.

- **루프 클로저의 부재 (Lack of Loop Closure)**: Faster-LIO는 기본적으로 Odometry, 즉 연속된 스캔 간의 상대적인 움직임을 추정하는 데 집중한 시스템이다. 따라서 장시간, 장거리를 운용할 때 필연적으로 누적되는 드리프트를 보정해 줄 루프 클로저(loop closure) 모듈이 내장되어 있지 않다.10 로봇이 이전에 방문했던 장소를 다시 인식하고 이를 이용해 전체 궤적의 오차를 전역적으로 보정하는 기능이 없기 때문에, 완전한 의미의 SLAM(Simultaneous Localization and Mapping) 시스템이라기보다는 고성능 프론트엔드(front-end) 또는 Odometry 프레임워크로 분류하는 것이 더 정확하다.
- **파라미터 민감성 (Parameter Sensitivity)**: 시스템의 성능과 안정성은 `voxel_size` 파라미터에 상당히 민감하다. 이 값은 환경의 스케일과 구조적 복잡성에 따라 최적값이 달라지므로, 사용자가 환경에 맞춰 수동으로 튜닝해야 하는 번거로움이 있다. 저자들 또한 시스템이 불안정할 경우 이 값을 조정해 볼 것을 권장하고 있어 11, 이는 시스템의 '플러그 앤 플레이' 특성을 저해하는 요인이 될 수 있다.


Faster-LIO의 가장 큰 성공 중 하나는 활발한 오픈소스 커뮤니티를 형성하고 수많은 변형(variant) 및 통합(integration) 프로젝트를 촉발시켰다는 점이다. 이는 원본 시스템의 강력한 장점과 동시에 보완이 필요한 약점을 명확히 보여준다.

- **S-Faster-Lio**: 이 프로젝트는 `S-FAST_LIO`의 단순화된 코드 구조와 `Faster-LIO`의 `iVox`를 결합한 것이다.20

  `S-FAST_LIO`는 복잡한 수치 연산 라이브러리 대신 Sophus와 같은 더 간결한 라이브러리를 사용하여 코드의 가독성과 접근성을 높인 버전이다.16

  `S-Faster-Lio`의 등장은 개발자들이 Faster-LIO의 `iVox`가 제공하는 압도적인 속도는 원하지만, 원본 코드의 복잡성을 부담스러워하며 더 이해하기 쉽고 수정이 용이한 코드 베이스를 선호한다는 점을 시사한다.

- **faster_lio_sam**: 이 프로젝트는 Faster-LIO의 빠른 프론트엔드(`iVox` 기반 IESKF)와 LIO-SAM의 강력한 백엔드(GTSAM 기반 팩터 그래프 최적화)를 결합하려는 시도이다.21 이는 Faster-LIO의 내재적 한계인 루프 클로저의 부재를 LIO-SAM의 검증된 백엔드와 결합하여 해결하려는 자연스러운 접근법이다.

이러한 커뮤니티의 움직임은 Faster-LIO의 진정한 가치가 그 자체로 완결된 제품이 아니라, 다른 SLAM 시스템에 쉽게 통합될 수 있는 **'고성능 프론트엔드 모듈'**로서의 역할에 있음을 보여준다. Faster-LIO가 제공하는 빠르고 정확한 Odometry 추정 능력은 매우 매력적이어서, 다른 연구자들은 이를 하나의 검증된 부품으로 가져와 자신들이 선호하는 백엔드(예: 팩터 그래프)나 추가 기능(예: 루프 클로저, 의미론적 분할)과 결합하는 플랫폼으로 활용하고 있다. 이는 성공적인 오픈소스 로보틱스 프로젝트가 나아갈 방향을 제시한다. 모든 기능을 포함한 거대하고 복잡한 단일 시스템보다, 특정 문제를 매우 효과적으로 해결하는 작고 효율적인 모듈이 커뮤니티에 의해 더 널리 채택되고, 다양한 방식으로 확장되며 생태계를 풍성하게 만들 수 있다는 것이다.



Faster-LIO는 LiDAR-Inertial Odometry 분야에서 실시간 성능의 한계를 한 단계 끌어올린 중요한 기술적 성과이다. 이 시스템의 핵심 기여는 FAST-LIO2의 견고하고 정밀한 IESKF 기반 상태 추정 프레임워크를 계승하면서, 성능의 주요 병목이었던 맵 관리 및 최근접 이웃 탐색 방식을 근본적으로 혁신한 데 있다. 복잡한 트리 구조를 유지해야 하는 `ikd-Tree` 대신, 해시 맵 기반의 희소 증분적 복셀 자료구조인 `iVox`를 도입하고, 이를 병렬 처리와 결합함으로써 k-NN 탐색 과정을 극적으로 가속화했다.

특히, "엄격한 k-NN 탐색은 LIO 문제에서 항상 필수적인 것은 아니다"라는 통찰을 바탕으로, 수학적 최적성을 일부 포기하는 대신 압도적인 속도 향상을 얻는 '근사 k-NN 탐색'이라는 과감한 공학적 트레이드오프를 성공시켰다. 그 결과, Faster-LIO는 FAST-LIO2와 비교하여 정확도에 거의 손실 없이 처리 속도를 1.5배에서 2배 이상 향상시키는 데 성공했으며, 이는 LIO 기술의 적용 범위를 연산 능력이 제한된 소형 로봇이나 고속으로 움직이는 플랫폼으로까지 확장할 수 있는 가능성을 열었다.


Faster-LIO의 기술적 의의는 단순히 빠른 알고리즘을 제시한 것을 넘어, LIO 시스템의 성능이 상태 추정기 자체뿐만 아니라, 이를 뒷받침하는 자료구조와의 상호작용에 의해 크게 좌우된다는 점을 명확히 입증했다는 데 있다. `iVox`의 성공은 FR-LIO의 `RC-Vox`와 같이 복셀 기반 맵 관리 방식에 대한 후속 연구를 촉발시키는 계기가 되었으며, 이는 LIO 분야의 지속적인 발전에 중요한 자양분이 되고 있다.

향후 LIO 기술은 Faster-LIO와 같은 고속 프론트엔드를 LIO-SAM과 같은 강건한 백엔드(루프 클로저, 전역 팩터 그래프 최적화)와 유기적으로 결합하는 방향으로 발전할 것으로 전망된다. 이러한 하이브리드 접근법을 통해, 단기적으로는 빠르고 정확한 지역적 움직임을 추정하고, 장기적으로는 전역적인 일관성을 유지하는, 즉 속도, 정확성, 글로벌 일관성을 모두 갖춘 완전한 SLAM 시스템을 구축할 수 있을 것이다. 이 과정에서 Faster-LIO는 고성능 Odometry 모듈의 표준적인 구현체 중 하나로서, 그리고 더 크고 복잡한 SLAM 시스템을 구축하기 위한 중요한 기술적 초석으로서 그 가치를 지속적으로 인정받을 것이다.


1. LIO-PPF: Fast LiDAR-Inertial Odometry via Incremental Plane Pre-Fitting and Skeleton Tracking, 8월 17, 2025에 액세스, [http://www.jdl.link/doc/2011/20231204_LIO-PPF%20%20Fast%20LiDAR%20Inertial%20Odometry%20via%20Incremental%20Plane%20Pre-Fitting%20and%20Skeleton%20Tracking.pdf](http://www.jdl.link/doc/2011/20231204_LIO-PPF  Fast LiDAR Inertial Odometry via Incremental Plane Pre-Fitting and Skeleton Tracking.pdf)
2. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
3. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/pdf/2107.06829
4. BEV-LIO(LC): BEV Image Assisted LiDAR-Inertial Odometry with Loop Closure - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2502.19242v2
5. Performance Analysis of LIO-SAM - Ronia Peterson, 8월 17, 2025에 액세스, http://www.roniapeterson.com/LIO-SAM.pdf
6. FAST-LIO2: Fast Direct LiDAR-inertial Odometry | Jiarong Lin, 8월 17, 2025에 액세스, https://jiaronglin.com/project/proj_fastlio/
7. FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - YouTube, 8월 17, 2025에 액세스, https://www.youtube.com/watch?v=iYCY6T79oNU
8. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/abs/2107.06829
9. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, 8월 17, 2025에 액세스, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
10. (PDF) Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial ..., 8월 17, 2025에 액세스, https://www.researchgate.net/publication/359662313_Faster-LIO_Lightweight_Tightly_Coupled_Lidar-Inertial_Odometry_Using_Parallel_Sparse_Incremental_Voxels
11. gaoxiang12/faster-lio: Faster-LIO: Lightweight Tightly ... - GitHub, 8월 17, 2025에 액세스, https://github.com/gaoxiang12/faster-lio
12. LiDAR-Inertial Odometry Based on Extended Kalman Filter - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2407.02786v1
13. A Quick Guide for the Iterated Extended Kalman Filter on Manifolds - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/372445074_A_Quick_Guide_for_the_Iterated_Extended_Kalman_Filter_on_Manifolds
14. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry, 8월 17, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2107.06829
15. Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial Odometry Using Parallel Sparse Incremental Voxels - SciSpace, 8월 17, 2025에 액세스, https://scispace.com/papers/faster-lio-lightweight-tightly-coupled-lidar-inertial-1uiy87qh
16. A simplified implementation of FAST_LIO (with Chinese note) - GitHub, 8월 17, 2025에 액세스, https://github.com/zlwang7/S-FAST_LIO
17. Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/pdf/2302.04031
18. [2302.04031] FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/abs/2302.04031
19. [2302.14674] LIO-PPF: Fast LiDAR-Inertial Odometry via Incremental Plane Pre-Fitting and Skeleton Tracking - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/abs/2302.14674
20. ZhangZh3ng/S-Faster-Lio: S-FAST_LIO with ivox map - GitHub, 8월 17, 2025에 액세스, https://github.com/ZhangZh3ng/S-Faster-Lio
21. GDUT-Kyle/faster_lio_sam: FASTER-LIO-SAM: A SLAM system based on iVox and GTSAM., 8월 17, 2025에 액세스, https://github.com/GDUT-Kyle/faster_lio_sam

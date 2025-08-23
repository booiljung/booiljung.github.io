# FAST-LIO


본 섹션에서는 FAST-LIO가 등장하게 된 배경을 설정하고, 이 기술이 해결하고자 했던 특정 기술적 및 알고리즘적 공백을 조명한다. FAST-LIO를 단독적인 발명이 아닌, 해당 분야의 도전 과제에 대한 중요한 응답으로 구성하여 그 맥락을 확립한다.


동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM)은 무인 항공기(UAV), 무인 지상 차량(UGV), 자율 주행차와 같은 자율 시스템의 핵심 기반 기술로 자리 잡았다.1 SLAM은 로봇이 미지의 환경에서 실시간으로 지도를 구축함과 동시에 그 지도 내에서 자신의 위치를 추정하는 과정을 의미하며, 이는 안전한 항법과 경로 계획에 필수적인 정보를 제공한다.1

이러한 상태 추정을 위해 주로 사용되는 센서는 LiDAR(Light Detection and Ranging)와 관성 측정 장치(Inertial Measurement Unit, IMU)이다. 두 센서는 상호 보완적인 특성을 지닌다. LiDAR는 주변 환경에 대한 정밀한 3차원 기하학적 정보를 제공하지만, 상대적으로 낮은 주파수로 동작하며 빠른 움직임에 의한 점군 왜곡(motion distortion)이나 특징이 부족한 환경(feature-poor environments)에서의 성능 저하(degeneracy)에 취약하다.4 반면, IMU는 매우 높은 주파수(수백 Hz)로 가속도와 각속도 데이터를 제공하여 빠른 움직임을 포착할 수 있지만, 시간이 지남에 따라 오차가 누적되어 심각한 드리프트(drift)를 발생시키는 단점이 있다.4 따라서 이 두 센서의 데이터를 효과적으로 융합하여 각 센서가 단독으로 제공할 수 있는 것보다 더 정확하고, 강건하며, 높은 주파수의 상태 추정치를 얻는 것이 LiDAR-관성 주행 거리 측정(LIO) 시스템의 핵심 과제이다.


FAST-LIO가 등장하기 전, LIO 분야는 크게 두 가지 접근법으로 양분되어 있었다: 최적화 기반 방법과 필터 기반 방법이다.


LOAM(LiDAR Odometry and Mapping)은 SLAM 문제를 높은 주파수의 주행 거리 측정(odometry) 스레드와 낮은 주파수의 지도 작성(mapping) 스레드로 분해하는 혁신적인 아이디어를 제시했다.5 이 접근법의 핵심은 점군에서 모서리(edge)와 평면(plane) 같은 기하학적 특징점을 추출하고, 연속된 스캔 간의 특징점 매칭을 통해 로봇의 움직임을 추정하는 것이다.7 LOAM은 높은 정확도를 달성하며 LIO 분야의 표준으로 자리 잡았으나, 종종 배치 최적화(batch optimization) 형태로 동작하여 계산량이 많고, 대규모 환경에서는 드리프트가 누적되는 경향이 있었다.9 LeGO-LOAM과 같은 파생 알고리즘들은 지면 분할(ground segmentation)이나 2단계 최적화 같은 기법을 도입하여 지상 차량에서의 효율성을 개선하고자 했다.10


필터 기반 접근법은 확장 칼만 필터(EKF)와 같은 재귀적 필터를 사용하여 시스템의 상태를 추정한다.5 이 방식은 실시간 처리에 유리하지만, 전통적인 EKF 기반 LIO는 심각한 계산 복잡도 문제에 직면했다. 특히 칼만 필터의 공분산 업데이트 단계는 측정값의 차원에 따라 계산량이 증가하는데, 수천 개의 점으로 구성된 LiDAR 스캔 데이터를 처리할 때 이 계산량은 실시간 운영을 불가능하게 만들었다.14 이러한 한계로 인해 초기 필터 기반 시스템들은 종종 LiDAR와 IMU 데이터를 독립적으로 처리한 후 결과를 합치는 느슨한 결합(loosely-coupled) 방식을 채택할 수밖에 없었고, 이는 정확도 저하로 이어졌다.17


FAST-LIO의 개발 동기는 명확했다: 쿼드콥터와 같이 자원이 제한된 플랫폼에서도 실시간으로 동작할 수 있는 **강결합(tightly-coupled)** 방식의 필터 기반 LIO 시스템을 구현하는 것이었다.2 이 시스템의 목표는 기존 필터 기반 방법의 발목을 잡았던 핵심 문제, 즉 대량의 측정값(LiDAR 점군)으로 인한 과도한 계산 부하를 해결하는 것이었다.14

이 문제의 해결은 단순히 기존 알고리즘을 개선하는 것을 넘어, 오랜 기간 동안 필터 기반 LIO의 실용화를 가로막았던 계산적 장벽을 허무는 것을 의미했다. SLAM 커뮤니티는 오랫동안 '높은 정확도지만 종종 오프라인으로 동작하는 최적화'와 '실시간이지만 측정값 크기에 의해 계산적으로 제한되는 필터링'이라는 두 가지 철학 사이에서 고민해왔다. 강결합 방식이 이론적으로 강건성 측면에서 우수하다는 것은 알려져 있었지만, LiDAR 데이터에 대한 실제 필터 구현은 칼만 이득(Kalman gain) 계산의 병목 현상으로 인해 좌절되었다. 필요한 혁신은 새로운 센서 모델이 아니라, 필터 업데이트 단계의 수학적 재구성이었다. FAST-LIO는 바로 이 지점에서 계산 로봇 공학 분야의 돌파구를 마련했다. 강결합의 강건성과 고주파수 상태 추정에 필요한 계산적 효율성을 결합함으로써, FAST-LIO는 필터 기반 방법이 최적화 기반 방법과 실시간 고밀도 LiDAR SLAM 분야에서 진정으로 경쟁할 수 있는 길을 열었다.


본 섹션에서는 오리지널 FAST-LIO의 수학적 심장부를 해부하며, 이 아키텍처의 두 기둥인 오차 상태 칼만 필터와 혁신적인 칼만 이득 계산법을 집중적으로 분석한다.


FAST-LIO는 **강결합(tightly-coupled)** 융합 방식을 채택했다. 이는 원시 LiDAR 점군과 IMU 측정값을 별개의 주행 거리 측정 시스템에서 처리하는 대신, 단일 필터링 프레임워크 내에서 직접 융합하는 것을 의미한다.9 이 접근법을 통해 시스템은 두 센서 양식 간의 상호 제약 조건을 최대한 활용하여 강건성을 높일 수 있다.

시스템의 상태, 특히 회전(rotation)은 유클리드 공간이 아닌 비선형 매니폴드(SO(3) manifold) 상에 존재한다. 이러한 비선형성을 다루기 위해 FAST-LIO는 **매니폴드 상의 칼만 필터(Kalman Filter on Manifolds)** 개념을 도입했다. 필터는 전체 상태를 직접 추정하는 대신, 국소 접선 공간(local tangent space)에 정의된 "오차 상태(error state)"에 대해 동작한다. 이 접선 공간은 표준 벡터 공간이므로 선형 대수 연산이 간단해진다.19

또한, FAST-LIO는 표준 EKF 대신 **반복 확장 칼만 필터(Iterated EKF, iEKF)**를 사용한다. 표준 EKF는 비선형 시스템을 단일 동작점(operating point)에서 한 번만 선형화하기 때문에, 비선형성이 강한 시스템에서는 상당한 오차를 유발할 수 있다. 반면 iEKF는 업데이트 단계에서 측정 모델을 반복적으로 재선형화하여 상태 추정치를 정교하게 다듬는다. 이 과정은 가우스-뉴턴 최적화(Gauss-Newton optimization)와 유사하며, 상태와 LiDAR 측정값 간의 비선형 관계를 보다 정확하게 처리할 수 있게 해준다.19


FAST-LIO의 필터링 과정은 전파(propagation)와 업데이트(update)의 두 단계로 구성된다.


FAST-LIO에서 사용하는 상태 벡터 `x`는 일반적으로 로봇의 위치(p), 속도(v), 회전(R), 그리고 IMU의 자이로스코프 바이어스(bg)와 가속도계 바이어스(ba)를 포함한다.22

$$ \mathbf{x} = \begin{bmatrix} \mathbf{R} & \mathbf{p} & \mathbf{v} & \mathbf{b}_a & \mathbf{b}_g \end{bmatrix}^T $$

전파 단계에서는 높은 주파수의 IMU 측정값(각속도 ω, 가속도 a)을 사용하여 시간 k-1에서 k까지 상태를 전진 적분(forward propagation)한다. 이 과정을 통해 LiDAR 스캔이 도착하기 전의 사전 상태 추정치(prior state estimate) ˜x를 얻는다.22

#### 2.2.2 업데이트 단계

LiDAR 스캔이 도착하면, 평면 특징점(planar feature points)과 같은 특징점들이 추출된다. 각 특징점에 대해, 측정된 점과 지도 상의 해당 평면 사이의 점-평면 거리(point-to-plane distance)인 잔차(residual)가 계산된다. 이 잔차는 비선형 측정 함수 `h(x)`를 통해 시스템 상태 `x`와 관련된다. iEKF는 사전 상태 오차와 측정 잔차의 가중 합으로 구성된 비용 함수를 반복적으로 최소화하여 사후 상태 추정치(a-posteriori state estimate)를 계산한다.20

### 2.3 핵심 혁신: 계산 효율적인 칼만 이득

FAST-LIO의 가장 중요한 혁신은 칼만 이득(Kalman gain) 계산 방식에 있다. 이 혁신은 필터 기반 LIO의 실용성을 가로막던 계산적 장벽을 무너뜨린 "핵심 기술(enabling technology)"이다.

#### 2.3.1 표준 칼만 이득 공식의 문제점

표준 칼만 이득 K의 공식은 다음과 같다.

K=PHT(HPHT+R)−1

여기서 P는 상태 공분산, H는 측정 자코비안, R은 측정 노이즈 공분산이다. 이 공식의 결정적인 병목 지점은 행렬 역산 항인 (H P H^T + R)^-1이다. LiDAR SLAM의 경우, 측정 벡터 z는 m개의 특징점에 대한 잔차로 구성되며, 따라서 자코비안 H는 m개의 행을 가진다. 결과적으로 역산해야 할 행렬의 크기는 m x m이 된다. FAST-LIO의 실험에서처럼 m이 1,200개 이상으로 커지면 14, 이 역산의 계산 복잡도는 `m`에 대해 3차(`O(m^3)`)로 증가하여 매우 비싸진다.

#### 2.3.2 FAST-LIO의 새로운 공식과 그 영향

FAST-LIO는 우드버리 행렬 항등식(Woodbury matrix identity)을 활용하여 칼만 이득을 수학적으로 동일하지만 계산적으로 훨씬 효율적인 형태로 재구성했다.1 이 접근법의 핵심은 `m x m` 크기의 큰 행렬 역산을 피하고, 대신 상태 벡터의 차원 `n`에 해당하는 `n x n` 크기의 작은 행렬 역산으로 문제를 전환하는 것이다.22

상태 벡터의 차원 `n`은 약 18 정도로 작고 상수인 반면, 측정값의 차원 `m`은 수천에 달하며 가변적이다. 따라서 이 재구성을 통해 필터 업데이트에서 가장 비용이 많이 드는 부분의 계산 복잡도가 `O(m^3)`에서 `O(n^3)`으로 극적으로 감소했다. 이 변화는 FAST-LIO가 수천 개의 점을 수십 밀리초 내에 처리할 수 있게 만든 핵심 요인이었으며 14, 실시간 강결합 필터 기반 LIO를 가능하게 한 결정적인 돌파구였다.

이러한 계산 효율성 증가는 단순히 속도를 높이는 것을 넘어, 알고리즘의 발전을 위한 "계산적 여유 공간(computational headroom)"을 창출했다. 이 여유 덕분에 개발자들은 후속 연구에서 특징점 추출을 완전히 포기하고 모든 원시 점군을 직접 사용하는(FAST-LIO2) 것과 같은 더 복잡하고 야심찬 아이디어를 실시간 필터 프레임워크 내에서 구현할 수 있었다. 즉, FAST-LIO의 효율적인 칼만 이득 계산법은 FAST-LIO2의 직접 정합 및 `ikd-Tree`와 같은 후속 혁신을 위한 필수적인 발판이 되었다.

## 섹션 3: FAST-LIO2로의 진화: 직접 방식과 진보된 지도 관리

본 섹션에서는 FAST-LIO에서 FAST-LIO2로의 중요한 도약을 상세히 다룬다. FAST-LIO2는 전통적인 특징점 추출 방식에서 벗어나 다재다능성과 정확성의 근본적인 문제를 해결했다.

### 3.1 특징점에서 원시 점군으로: 직접 정합으로의 패러다임 전환

#### 3.1.1 특징점 기반 방식의 한계

모서리나 평면과 같은 기하학적 특징을 수작업으로 설계된 규칙에 따라 추출하는 방식은 본질적으로 취약하다. 터널이나 숲과 같이 구조적으로 반복되거나 특징이 부족한 환경에서는 뚜렷한 특징점을 거의 찾을 수 없어 성능이 저하된다.1 또한, 이 방식은 회전형 LiDAR와 고정형(solid-state) LiDAR처럼 스캔 패턴이 다른 센서에 적용할 때마다 특징 추출기를 다시 조정해야 하므로 시스템의 범용성을 저해한다.1

#### 3.1.2 FAST-LIO2의 "직접(Direct)" 방식

이러한 한계를 극복하기 위해 FAST-LIO2는 특징점 추출 모듈을 완전히 제거하는 과감한 선택을 했다. 대신, LiDAR 스캔에서 얻은 원시 점군(raw points)을 지도에 직접 정합(direct registration)한다.1 이 접근법은 시각 SLAM에서 특징점 대신 픽셀의 밝기 값을 직접 사용하는 "직접 방식(direct method)"과 유사하다.

#### 3.1.3 직접 방식의 이점

이 패러다임 전환은 여러 가지 중요한 이점을 가져왔다.

- **정확성 및 강건성 향상:** 특징점 추출 과정에서 버려졌을 미세한 기하학적 정보까지 모두 활용하므로, 더 정확한 상태 추정이 가능하다.1 특히 특징이 부족한 환경에서도 모든 점이 제약 조건으로 작용하여 강건성이 크게 향상된다.25
- **향상된 다재다능성:** 시스템이 특정 센서의 스캔 패턴이나 특징점 정의에 의존하지 않게 되어 "센서에 구애받지 않는(sensor-agnostic)" 특성을 갖게 된다. 이로 인해 회전형, 고정형, 시야각(FoV)이 좁은 LiDAR 등 다양한 센서에 별도의 수정 없이 자연스럽게 적응할 수 있다.1

### 3.2 `ikd-Tree`: 동적 환경을 위한 새로운 증분 K-D 트리

직접 정합 방식은 새로운 도전 과제를 낳았다. 이제 지도는 방대한 양의 원시 점군으로 구성되며, 이 지도는 높은 주파수(예: 최대 100 Hz)로 지속적으로 업데이트되고 검색(최근접 이웃 탐색)되어야 한다.1 정적 K-D 트리나 옥트리(Octree)와 같은 표준적인 자료 구조는 이처럼 빈번한 동적 업데이트(삽입, 삭제)에 비효율적이다.1

이 문제를 해결하기 위해 FAST-LIO2는 `ikd-Tree`라는 새로운 증분 K-D 트리(incremental k-d tree) 자료 구조를 개발했다. `ikd-Tree`는 LIO 애플리케이션의 특정 요구사항에 맞춰 설계된 핵심 혁신이다.

#### 3.2.1 `ikd-Tree`의 아키텍처 세부사항

- **증분 업데이트(Incremental Updates):** 점 단위 및 박스 단위의 삽입과 삭제를 매우 효율적으로 지원하도록 설계되었다. 이는 로봇 주변의 일정 영역만 지도로 유지하는 슬라이딩 윈도우(sliding-window) 방식의 지도 관리에 필수적이다.1
- **지연 삭제 및 동적 재균형(Lazy Deletion & Dynamic Re-balancing):** 점을 삭제할 때 즉시 트리 구조를 변경하는 대신 '삭제됨'으로 표시만 하는 지연 삭제 방식을 사용한다. 트리는 스스로의 균형 상태를 감시하다가 불균형이 심해지면 해당 하위 트리만 재구성한다. 특히, 이 재구성 작업은 별도의 병렬 스레드에서 수행되어 메인 주행 거리 측정 프로세스를 차단하지 않으므로 실시간 성능을 보장한다.1
- **트리 내 다운샘플링(On-Tree Downsampling):** 지도 해상도를 유지하기 위한 다운샘플링 과정이 점 삽입 과정에 통합되어 있다. 새로운 점이 삽입될 때, 해당 복셀 내의 기존 점들과 비교하여 복셀 중심에 가장 가까운 점 하나만 남기고 나머지는 효율적으로 제거한다.1

#### 3.2.2 동적 자료 구조 성능 비교

`ikd-Tree`의 개발이 정당화되는 이유는 기존 자료 구조들이 LIO의 복합적인 요구사항을 만족시키지 못했기 때문이다. 아래 표는 FAST-LIO2 논문에서 제시된 벤치마크 결과를 바탕으로 `ikd-Tree`와 다른 동적 자료 구조들의 성능을 비교한 것이다.1

**표 3.1: LIO 애플리케이션을 위한 동적 자료 구조 성능 비교**

| 자료 구조          | 구축 시간 | 증분 삽입 시간     | 삭제 시간          | K-NN 검색 시간 | 메모리 사용량 |
| ------------------ | --------- | ------------------ | ------------------ | -------------- | ------------- |
| **ikd-Tree**       | **우수**  | **최우수**         | **최우수**         | **우수**       | **우수**      |
| Octree             | 보통      | 보통               | 나쁨               | 보통           | 보통          |
| R*-Tree            | 나쁨      | 나쁨               | 보통               | 나쁨           | 나쁨          |
| nanoflann k-d tree | 최우수    | 나쁨 (재구축 필요) | 나쁨 (재구축 필요) | 최우수         | 최우수        |

이 표는 다른 구조들이 특정 작업(예: nanoflann의 검색 속도)에서는 뛰어날 수 있지만, LIO에서 요구되는 빈번한 삽입, 삭제, 검색의 조합이라는 복합적인 작업 부하 하에서는 `ikd-Tree`가 전반적으로 가장 뛰어난 성능을 제공함을 명확히 보여준다.

결론적으로, FAST-LIO2의 두 가지 핵심 혁신인 직접 정합과 `ikd-Tree`는 서로 깊이 얽혀 있으며 공진화(co-evolutionary)적인 해결책을 나타낸다. 한쪽의 혁신 없이는 다른 쪽의 실용성이 보장될 수 없었다. 강건성과 다재다능성을 향상시키려는 열망이 직접 정합으로의 전환을 이끌었고, 이는 곧바로 원시 점군으로 구성된 거대하고 동적인 지도를 실시간으로 관리해야 하는 새로운 데이터 관리 문제를 야기했다. 기존 자료 구조들이 이 까다로운 사용 사례에 최적이 아님이 벤치마크를 통해 확인되자, 이는 직접 LIO 시스템의 운영 요구사항(빠른 증분 업데이트, 병렬 재균형 등)을 정확히 처리하도록 맞춤 제작된 새로운 자료 구조, 즉 `ikd-Tree`의 발명을 필연적으로 만들었다. 이는 단편적인 개선이 아닌, 전체 시스템을 고려한 총체적인 설계 접근법의 결과물이다.

## 섹션 4: 확장되는 생태계: 특화, 루프 클로저, 그리고 위치 추정

본 섹션에서는 FAST-LIO/FAST-LIO2의 고성능 주행 거리 측정 프론트엔드가 어떻게 방대한 전문 SLAM 시스템 생태계의 기반이 되었는지를 탐구한다.

### 4.1 속도와 강건성의 한계 돌파

FAST-LIO2의 성공은 더 빠르고 더 강건한 시스템을 향한 새로운 연구의 물결을 일으켰다.

- **Faster-LIO:** 이 변종은 계산 속도를 극대화하는 데 초점을 맞춘다. `ikd-Tree`를 해시 테이블 기반의 병렬화된 희소 증분 복셀 구조(iVox)로 대체하여 FAST-LIO2 대비 1.5배에서 2배의 속도 향상을 달성했다고 주장한다.27 이는 계산 자원이 매우 제한적이거나 회전형 LiDAR에서 100 Hz 이상의 초고주파수 업데이트가 필요한 플랫폼에 이상적이다.
- **FR-LIO (Fast and Robust LIO):** 이 변종은 매우 공격적인 움직임(aggressive motion) 중의 강건성을 목표로 한다. 움직임 강도에 따라 전체 스캔을 여러 개의 하위 프레임(sub-frame)으로 적응적으로 분할하고, 하위 프레임의 제약 조건 부족으로 인한 성능 저하를 막기 위해 단순 필터가 아닌 강결합 반복 오차 상태 칼만 **스무더(Smoother, ESKS)**를 사용한다. 또한 효율성을 위해 로봇 중심 복셀 맵(RC-Vox)을 채택했다.27

### 4.2 루프 닫기: 백엔드 포즈 그래프 최적화 통합

핵심 FAST-LIO 프레임워크는 프레임 간의 움직임을 추정하는 *주행 거리 측정(odometry)* 시스템이다. 이는 단기적으로는 매우 정확하지만, 장거리 주행 시에는 필연적으로 오차가 누적된다. 이는 방문했던 장소를 다시 인식하고 전역 지도를 수정하는 루프 클로저(loop closure) 메커니즘이 없기 때문이다.3 이 한계를 극복하기 위해 커뮤니티는 FAST-LIO를 프론트엔드로 활용하는 다양한 완전한 SLAM 시스템을 구축했다.

- **FAST_LIO_LC:** 이 프로젝트는 반경 탐색 기반의 루프 클로저 모듈을 FAST-LIO와 명시적으로 통합한다. 결정적으로, 백엔드 그래프 최적화로부터 얻은 보정된 포즈를 FAST-LIO의 iEKF에 다시 피드백하여 현재 상태와 지도를 모두 수정하는 긴밀한 통합을 구현했다.30
- **FAST-LIO-SAM-QN:** 이는 FAST-LIO2를 주행 거리 측정 프론트엔드로 사용하고 정교한 백엔드를 결합한 복합 SLAM 시스템이다. 루프 클로저 감지를 위해 빠른 전역 정합기인 Quatro를 사용하고, 포즈 그래프 최적화를 위해 GTSAM(Georgia Tech Smoothing and Mapping) 라이브러리를 활용한다.32 이는 FAST-LIO2가 가장 잘하는 작업(프론트엔드)에 집중하고, 최첨단 백엔드 구성 요소와 결합하는 모듈식 접근법을 보여준다.
- **LIO-SAM과의 비교:** LIO-SAM 자체는 LIO를 요인 그래프(factor graph) 문제로 공식화한 선구적인 연구이다.9 많은 FAST-LIO 파생 프로젝트들이 유사한 철학을 채택하며 LIO-SAM의 프론트엔드를 더 효율적인 FAST-LIO2로 대체하는데, 이는 후자가 모듈식 구성 요소로서 얼마나 강력한지를 입증한다.

### 4.3 주행 거리 측정을 넘어: 지도 기반 재위치 추정 프레임워크

FAST-LIO의 강력한 추적 성능은 사전에 구축된 지도 내에서 로봇의 위치를 찾는 재위치 추정(relocalization) 애플리케이션으로도 확장되었다.

- **FAST-LIO-LOCALIZATION:** 이 프로젝트는 FAST-LIO를 확장하여 사전에 구축된 점군 지도 내에서 강건한 위치 추정을 수행한다.34 초기 추정 위치와 ICP(Iterative Closest Point)를 사용하여 지도 내에서 포즈를 초기화한 후, FAST-LIO 프레임워크를 이용해 고주파수 추적을 이어간다.
- **FAST-LIO-Localization-QN:** 유사한 개념이지만, 지도 정합/재위치 추정 단계에서 더 진보된 Quatro와 Nano-GICP를 사용한다. 이는 핵심 프레임워크에 다양한 정합 모듈을 플러그인할 수 있음을 보여준다.35

### 4.4 주요 FAST-LIO 변종의 분류

FAST-LIO 생태계는 방대하고 복잡할 수 있다. 아래 표는 연구자와 개발자가 그 지형을 이해하고 자신의 필요에 맞는 도구를 선택할 수 있도록 명확하고 간결한 가이드를 제공한다.

**표 4.1: 주요 FAST-LIO 변종의 분류**

| 변종 이름                    | 핵심 기여                   | 주요 기술/의존성            | 목표 애플리케이션/해결 문제               |
| ---------------------------- | --------------------------- | --------------------------- | ----------------------------------------- |
| **FAST-LIO** 14              | 효율적인 칼만 이득 계산     | iEKF, 특징점 기반           | 실시간 강결합 LIO의 계산적 실현           |
| **FAST-LIO2** 1              | 직접 점군 정합, `ikd-Tree`  | iEKF, 원시 점군, `ikd-Tree` | 다재다능하고 정확한 고성능 LIO            |
| **Faster-LIO** 28            | 속도 극대화                 | iVox 병렬 복셀 맵           | 자원 제한적 플랫폼, 초고주파 업데이트     |
| **FR-LIO** 29                | 공격적 움직임에 대한 강건성 | ESKS, 적응형 하위 프레임    | 고속 회전 및 급격한 움직임이 있는 UAV/UGV |
| **FAST_LIO_LC** 30           | 루프 클로저 통합            | 반경 탐색, EKF로의 피드백   | 장거리 주행 시 누적 오차 제거             |
| **FAST-LIO-SAM-QN** 32       | 완전한 SLAM 시스템          | Quatro/Nano-GICP, GTSAM     | 대규모 환경의 전역 일관성 있는 지도 작성  |
| **FAST-LIO-LOCALIZATION** 34 | 지도 기반 재위치 추정       | ICP, 사전 구축 지도         | 알려진 환경에서의 항법                    |

이러한 생태계의 확장은 FAST-LIO2 프론트엔드의 모듈성과 고성능이 커뮤니티에 기여한 가장 중요한 부분임을 시사한다. 그 성공은 자체 시스템의 성능뿐만 아니라, 더 복잡하고 유능한 SLAM 시스템에서 "플러그 앤 플레이" 주행 거리 측정 엔진으로 얼마나 자주 채택되는지로 측정된다. FAST-LIO2가 LIO 문제의 한 부분(프론트엔드)을 매우 잘 해결했기 때문에, 다른 연구자들은 백엔드와 같은 다른 부분에 노력을 집중할 수 있었다. 이러한 모듈식 채택은 성공적인 오픈소스 프로젝트와 잘 설계된 소프트웨어의 강력한 지표이다.

## 섹션 5: 이론에서 실습으로: 구현 및 실제 적용 사례

본 섹션에서는 이론적 기반에서 벗어나 FAST-LIO를 사용하는 실용적인 측면으로 전환하여, 배포 가이드를 제공하고 실제 사례 연구를 통해 입증된 성능을 보여준다.

### 5.1 시스템 배포: 의존성, 빌드 프로세스, 및 하드웨어 통합

FAST-LIO 및 그 파생 프로젝트들을 성공적으로 배포하기 위해서는 특정 소프트웨어 의존성을 충족하고 표준화된 빌드 절차를 따라야 한다.

- **핵심 의존성:** 대부분의 FAST-LIO 계열은 우분투(Ubuntu) 운영체제와 ROS(Robot Operating System, Melodic 또는 Noetic 버전) 환경을 기본으로 요구한다. 또한, 점군 처리를 위한 PCL(Point Cloud Library) 버전 1.8 이상과 행렬 연산을 위한 Eigen 라이브러리 버전 3.3.4 이상이 필요하다.34 루프 클로저 기능이 추가된 변종들은 GTSAM 31, 병렬 처리를 강화한 변종들은 tbb 32와 같은 추가적인 라이브러리를 요구할 수 있다.

- **빌드 프로세스:** 빌드는 표준 ROS `catkin_make` 워크플로우를 따른다. 사용자는 GitHub 저장소를 자신의 catkin 작업 공간에 복제하고, 필요한 하위 모듈(submodule)을 초기화한 후 `catkin_make` 명령어를 실행하여 컴파일한다.28

- **하드웨어 지원 및 구성:**

  - **LiDAR:** FAST-LIO의 가장 큰 강점 중 하나는 센서 다재다능성이다. 특징점 추출을 제거한 덕분에 전통적인 회전형 LiDAR(Velodyne, Ouster)뿐만 아니라, 스캔 패턴이 불규칙한 고정형 LiDAR(Livox)도 원활하게 지원한다.1 사용자는 ROS 토픽 이름과 센서의 외부 파라미터(extrinsic parameters)를 정확하게 설정해야 한다.31

  - **IMU:** 시스템은 6축 또는 9축 IMU를 요구한다. 특히 점군 왜곡 보정을 위해 IMU 데이터의 품질과 주파수가 성능에 결정적인 영향을 미친다.38 LiDAR와 IMU 간의 상대적인 위치 및 회전을 나타내는 외부 파라미터(

    `T_imu_lidar`)는 매우 정밀하게 측정되어 제공되어야 한다.34

### 5.2 공개 벤치마크에서의 성능 분석

FAST-LIO2는 다양한 공개 데이터셋에서 그 성능을 입증했다. M2DGR, urbanNav, Newer College, KITTI와 같은 벤치마크 데이터셋을 사용한 실험에서, FAST-LIO2는 다른 최첨단 시스템들과 비교했을 때 훨씬 낮은 계산 부하로 일관되게 더 높은 정확도를 달성했다.1 예를 들어, 일반적인 온보드 컴퓨터에서도 최대 100 Hz의 속도로 스캔을 처리할 수 있는 계산 효율성을 보여주었다.1

### 5.3 사례 연구: 무인 항공기(UAV)의 자율 항법

- **도전 과제:** UAV는 크기, 무게, 전력(SWaP)에 제약이 크며, 종종 빠르고 공격적인 움직임을 보인다. 이러한 플랫폼에서 실시간으로 안정적인 상태 추정을 수행하는 것은 매우 어렵다.
- **FAST-LIO의 역할:** FAST-LIO의 뛰어난 계산 효율성은 DJI Manifold나 NVIDIA Jetson과 같은 UAV용 온보드 컴퓨터에 탑재하기에 이상적이다.2 또한, 초당 1000도에 달하는 빠른 회전에도 강건한 성능을 유지하는 능력은 UAV의 동적인 비행 환경에 필수적이다.1
- **적용 사례:** FAST-LIO는 쿼드콥터의 실시간 위치 추정 및 지도 작성에 성공적으로 사용되어, GPS 신호가 없는 좁은 터널 39이나 복잡한 숲 40과 같은 환경에서의 자율 비행을 가능하게 했다.

### 5.4 사례 연구: 무인 지상 차량(UGV)의 강건한 주행 거리 측정

- **도전 과제:** UGV는 구조화된 실내 환경부터 비구조적인 실외, 특징이 부족한 긴 복도나 넓은 들판, 그리고 동적 객체가 많은 복잡한 시나리오에 이르기까지 매우 다양한 환경에서 운용된다.
- **FAST-LIO의 역할:** 센서와 환경에 대한 다재다능성 덕분에 이러한 다양한 시나리오에 효과적으로 대응할 수 있다. 복잡한 창고 환경 2, 대학 캠퍼스 22, 심지어 특징이 유사한 블루베리 농장 40에서도 그 성능이 검증되었다. ATCrawler 플랫폼은 FAST-LIO를 사용하여 실내외를 넘나드는 지도 작성을 시연하기도 했다.41

이러한 실제 적용 사례들은 FAST-LIO의 이론적 혁신이 어떻게 실용적인 성공으로 이어졌는지를 명확히 보여준다. 예를 들어, UAV 애플리케이션의 성공은 FAST-LIO의 핵심 설계 원칙과 직접적으로 연결된다. UAV는 작고 저전력의 온보드 컴퓨터를 사용하며, 비행 제어를 위해 고주파수의 상태 추정치가 필요하다.2 FAST-LIO의 핵심 설계(효율적인 칼만 이득, 효율적인 

`ikd-Tree`)는 바로 이러한 계산 효율성을 제공하여 해당 하드웨어에서 실시간으로 실행될 수 있게 한다.2 또한, UAV가 구조화된 건물에서 이륙하여 특징이 부족한 개활지로 비행할 때, 특징점 기반 방법은 실패할 수 있다. FAST-LIO2의 직접 정합 방식은 이러한 환경 구조의 변화에 강건하게 대처한다.25 이처럼 추상적인 알고리즘적 특징들이 핵심 응용 분야의 구체적인 요구사항과 직접 연결됨으로써, FAST-LIO가 왜 UAV와 같은 실제 로봇 애플리케이션에서 널리 선택되는지를 설명할 수 있다.

## 섹션 6: 비판적 평가: 한계점 및 향후 연구 방향

본 섹션에서는 FAST-LIO 프레임워크에 대한 균형 잡힌 비평을 제공하며, 그 약점을 인정하고 연구 커뮤니티가 이러한 문제들을 어떻게 해결해 나가고 있는지 논의한다.

### 6.1 내재된 도전 과제 및 한계점

FAST-LIO는 강력한 성능을 자랑하지만, 그 설계 철학에서 비롯된 몇 가지 근본적인 한계점을 가지고 있다.

- **IMU 품질 및 캘리브레이션에 대한 민감도:** 강결합 시스템으로서, FAST-LIO의 성능은 양질의 IMU 데이터와 정확한 외부 파라미터 캘리브레이션에 크게 의존한다. 매우 공격적인 움직임 중에 IMU 센서가 포화(saturation) 상태에 이르면, 잘못된 정보가 상태 추정치에 결합되어 시스템 전체의 실패를 초래할 수 있다.42 또한, 정밀한 하드웨어 시간 동기화(time synchronization)를 달성하는 것 역시 중요하며 도전적인 과제이다.42
- **특징 부족 환경에서의 성능 저하(Degeneracy):** 직접 정합 방식이 많은 부분을 완화했지만, 시스템은 여전히 특징 구분이 거의 불가능한 매우 자기 유사적인 환경(예: 긴 직선 터널)에서는 성능 저하를 겪을 수 있다. 이 경우, 필터의 공분산은 제약되지 않는 방향으로 발산하게 된다.26 한 연구에서는 인공적인 랜드마크가 없는 터널 환경에서 FAST-LIO2가 가장 좋은 성능을 보였지만, 여전히 개선의 여지가 있음을 보여주었다.43 LIO-Livox와 같은 유사 시스템은 개활지에서의 성능 저하를 막기 위해 불규칙한 특징 클래스를 명시적으로 추가하기도 한다.44
- **내재된 백엔드의 부재:** 앞서 논의된 바와 같이, 핵심 FAST-LIO 패키지는 주행 거리 측정 시스템이므로 루프 클로저와 전역 최적화를 위한 백엔드 없이는 장거리 주행 시 오차가 누적된다.30 이는 대규모 지도 작성 애플리케이션에 있어 주요한 한계점이다.
- **동적 객체 처리의 부재:** 기본 FAST-LIO2는 동적 객체를 명시적으로 처리하는 메커니즘을 포함하고 있지 않다. 움직이는 자동차나 보행자는 잘못된 제약 조건을 생성하여 정확도를 저하시킬 수 있다. LIO-Livox와 같은 다른 시스템들은 이를 해결하기 위해 분할 및 동적 객체 제거 단계를 통합했다.44

### 6.2 새로운 해결책과 연구 방향

연구 커뮤니티는 FAST-LIO의 한계를 극복하기 위한 다양한 해결책을 활발히 연구하고 있다.

- **적응형 기술 (Adaptive-LIO):** "Adaptive-LIO"라는 연구는 FAST-LIO2의 여러 한계점을 직접적으로 다룬다.42
  - **적응형 프레임 분할:** 고정된 스캔 주기를 사용하는 대신, 움직임이 격렬할 때 스캔을 더 작은 하위 프레임으로 적응적으로 분할한다. 이는 등속도 가정을 위반하여 발생하는 왜곡을 줄여준다.42 이는 FR-LIO의 아이디어와 유사하다.29
  - **적응형 모션 모델 전환:** IMU의 포화나 고장을 감지하고 LiDAR 단독 주행 거리 측정 모드로 전환할 수 있는 느슨한 결합 프레임워크를 제안하여 시스템의 강건성을 향상시킨다.42
  - **적응형 다중 해상도 지도:** 좁은 복도에서는 고해상도, 넓은 개활지에서는 저해상도 복셀을 사용하는 등, 영역에 따라 지도 해상도를 다르게 적용하여 일관된 지도 밀도와 정확도를 유지한다.42
- **동적 환경에 대한 강건성 향상:** 필터에 입력되기 전에 동적 객체에 속하는 점들을 탐지하고 제거할 수 있는 더 강건한 프론트엔드를 추가하는 연구가 진행 중이다.23
- **다른 센서와의 긴밀한 통합:** 카메라(예: LVI-SAM 45)나 GPS 9와 같은 다른 센서의 데이터를 요인 그래프 내에서 융합하는 것은 장기적인 정확도와 강건성을 향상시키기 위한 핵심 연구 분야로 남아있다.

FAST-LIO의 주요 한계점들은 그 강결합 필터 설계의 핵심 가정에서 직접적으로 비롯된다. 즉, 이 시스템의 강점(강결합으로 인한 속도와 정확성)과 약점(캘리브레이션 및 IMU 품질에 대한 민감성)은 동전의 양면과 같다. FAST-LIO는 LiDAR와 IMU 측정 간의 기하학적 제약 조건을 필터 내에서 엄격하게 강제한다. 센서 데이터와 캘리브레이션이 양호할 때, 이는 매우 높은 정확도로 이어진다. 그러나 IMU 데이터가 나쁘거나(예: 포화) 캘리브레이션이 잘못되면, 필터는 이 잘못된 정보를 강제로 융합하게 되어 상태 추정치의 오염을 초래한다. 반면, 느슨한 결합 시스템은 정확도는 떨어질 수 있지만 단일 센서의 고장에 대해 더 강건할 수 있다. 이러한 트레이드오프는 Adaptive-LIO 42의 제안에서 명확히 드러나는데, 이 연구는 신뢰할 수 없을 때 IMU를 "끌" 수 있는 능력을 얻기 위해 느슨한 결합 프레임워크로의 전환을 제안한다. 이는 SLAM 분야의 근본적인 설계 긴장, 즉 최고 정확도(강결합)와 시스템 수준의 고장 감내 능력(느슨한 결합) 간의 트레이드오프를 보여준다. FAST-LIO는 확고하게 최고 정확도의 편에 서 있으며, 그 한계는 바로 그 선택의 직접적인 결과이다.

## 섹션 7: 결론: FAST-LIO 프레임워크의 지속적인 영향력

본 마지막 섹션에서는 보고서의 핵심 발견 사항들을 종합하여 FAST-LIO의 기여와 로봇 공학 분야에 남긴 지속적인 유산을 요약한다.

### 7.1 핵심 기여 요약

FAST-LIO 프레임워크는 LIO 분야에 세 가지 중요한 기여를 했다.

- **계산적 돌파구:** 효율적인 칼만 이득 계산법의 도입은 가장 중요한 기여이다. 이는 실시간, 필터 기반, 강결합 LIO를 계산적으로 실현 가능한 기술로 만들었으며, 필터 기반 접근법의 오랜 병목 현상을 해결했다.
- **알고리즘적 패러다임 전환:** FAST-LIO2에서 도입된 직접 점군 정합 방식은 센서 다재다능성과 정확성을 크게 향상시켰다. 이와 함께, 그로 인해 발생하는 데이터 부하를 관리하기 위해 `ikd-Tree`를 공동 개발한 것은 전체 시스템을 고려한 총체적 설계의 모범 사례이다.
- **생태계 조성:** 고품질의 오픈소스 프론트엔드를 제공함으로써, FAST-LIO는 더 넓고 복잡하며 유능한 SLAM 시스템을 구축하기 위한 기초적인 빌딩 블록 역할을 했다. 그 영향력은 자체 성능뿐만 아니라, 그것이 가능하게 한 수많은 후속 연구와 응용 프로그램으로 측정된다.

### 7.2 미래 연구를 위한 기반으로서의 FAST-LIO

FAST-LIO는 단순히 하나의 성공적인 알고리즘을 넘어, 새로운 LIO 연구를 위한 벤치마크이자 기준선(baseline)으로서 중요한 역할을 하고 있다. 그 견고한 아키텍처는 새로운 센서 융합 기술, 자료 구조, 그리고 적응형 알고리즘을 실험하기 위한 안정적인 플랫폼을 제공한다.

결론적으로, FAST-LIO 제품군은 최첨단 기술을 크게 발전시켰으며, 연구자와 개발자에게 자율 항법의 도전 과제를 해결하기 위한 강력하고 효율적이며 다재다능한 도구를 제공했다. 그 영향력은 단순히 낮은 드리프트와 빠른 처리 속도를 넘어, 로봇 공학 커뮤니티가 더 복잡한 문제를 해결할 수 있도록 기반을 다지고 새로운 가능성의 문을 연 데에 있다. FAST-LIO는 현대 로봇 공학의 발전에 있어 중요한 이정표로 기억될 것이다.

#### **참고 자료**

1. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 16, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
2. FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter | Request PDF - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/349909094_FAST-LIO_A_Fast_Robust_LiDAR-inertial_Odometry_Package_by_Tightly-Coupled_Iterated_Kalman_Filter
3. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed July 16, 2025, https://www.mathworks.com/discovery/slam.html
4. LVI-Fusion: A Robust Lidar-Visual-Inertial SLAM Scheme - Semantic Scholar, accessed July 16, 2025, https://pdfs.semanticscholar.org/736e/02c261c9b7efdc9519eeb02d757af8b98122.pdf
5. Lidar SLAM in Cognitive Robotics - Number Analytics, accessed July 16, 2025, https://www.numberanalytics.com/blog/lidar-slam-cognitive-robotics
6. LOAM: Lidar Odometry and Mapping in Real-time - Carnegie Mellon University's Robotics Institute, accessed July 16, 2025, https://www.ri.cmu.edu/pub_files/2014/7/Ji_LidarMapping_RSS2014_v8.pdf
7. Introduction to LiDAR SLAM: LOAM and LeGO-LOAM Paper and Code Explanation with ROS 2 Implementation - LearnOpenCV, accessed July 16, 2025, https://learnopencv.com/lidar-slam-with-ros2/
8. An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM and Scan Context for Underground Coalmine - PMC - PubMed Central, accessed July 16, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8778426/
9. Lio-sam: Tightly-coupled lidar inertial odometry via smoothing and mapping - arXiv, accessed July 16, 2025, http://arxiv.org/pdf/2007.00258
10. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain - OpenCV, accessed July 16, 2025, https://opencv.org/blog/lego-loam/
11. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/330592017_LeGO-LOAM_Lightweight_and_Ground-Optimized_Lidar_Odometry_and_Mapping_on_Variable_Terrain
12. Evaluation of LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping, accessed July 16, 2025, https://bpb-us-e1.wpmucdn.com/sites.northeastern.edu/dist/a/1829/files/2022/03/Group-5-Final-Project.pdf
13. The math behind Extended Kalman Filtering | by Sasha Przybylski - Medium, accessed July 16, 2025, https://medium.com/@sasha_przybylski/the-math-behind-extended-kalman-filtering-0df981a87453
14. [2010.08196] FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2010.08196
15. Fast-lio: A fast, robust lidar-inertial odometry package by tightly-coupled iterated Kalman filter - HKU Scholars Hub, accessed July 16, 2025, https://hub.hku.hk/handle/10722/301527
16. FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=iYCY6T79oNU
17. RSS-LIWOM: Rotating Solid-State LiDAR for Robust LiDAR-Inertial-Wheel Odometry and Mapping - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/journal/Remote-Sensing-2072-4292/publication/373155250_RSS-LIWOM_Rotating_Solid-State_LiDAR_for_Robust_LiDAR-Inertial-Wheel_Odometry_and_Mapping/links/657abf7bea5f7f02056e4458/RSS-LIWOM-Rotating-Solid-State-LiDAR-for-Robust-LiDAR-Inertial-Wheel-Odometry-and-Mapping.pdf
18. VOX-LIO: An Effective and Robust LiDAR-Inertial Odometry System Based on Surfel Voxels, accessed July 16, 2025, https://www.mdpi.com/2072-4292/17/13/2214
19. [2307.09237] A Quick Guide for the Iterated Extended Kalman Filter on Manifolds - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2307.09237
20. A Quick Guide for the Iterated Extended Kalman Filter on Manifolds - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/372445074_A_Quick_Guide_for_the_Iterated_Extended_Kalman_Filter_on_Manifolds
21. 6.2.1 The extended Kalman filter and the Iterative extended Kalman filter - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=P0ZVTa9U7XU
22. LIO-EKF: High Frequency LiDAR-Inertial Odometry using Extended Kalman Filters, accessed July 16, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/wu2024icra.pdf
23. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 16, 2025, https://arxiv.org/html/2407.03590v1
24. FAST-LIO2: Fast Direct LiDAR-inertial Odometry, accessed July 16, 2025, https://arxiv.org/abs/2107.06829
25. FAST-LIO2: Fast Direct LiDAR-Inertial Odometry | Request PDF - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/358323678_FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry
26. FAST-LIO - Jiarong Lin, accessed July 16, 2025, https://jiaronglin.com/tag/fast-lio/
27. Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, accessed July 16, 2025, https://arxiv.org/pdf/2302.04031
28. Faster-LIO: Lightweight Tightly Coupled Lidar-inertial Odometry using Parallel Sparse Incremental Voxels - GitHub, accessed July 16, 2025, https://github.com/gaoxiang12/faster-lio
29. [2302.04031] FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2302.04031
30. yanliang-wang/FAST_LIO_LC: The tight integration of FAST-LIO with Radius-Search-based loop closure module. - GitHub, accessed July 16, 2025, https://github.com/yanliang-wang/FAST_LIO_LC
31. FAST_LIO_LC - Autoware Universe Documentation, accessed July 16, 2025, https://autowarefoundation.github.io/autoware-documentation/pr-366/how-to-guides/creating-maps-for-autoware/open-source-slam/fast-lio-lc/
32. engcang/FAST-LIO-SAM-QN: A SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on Quatro and Nano-GICP - GitHub, accessed July 16, 2025, https://github.com/engcang/FAST-LIO-SAM-QN
33. Performance Analysis of LIO-SAM - Ronia Peterson, accessed July 16, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
34. iral-ntua/FAST_LIO_LOCALIZATION: A simple localization system based on FAST_LIO framework - GitHub, accessed July 16, 2025, https://github.com/iral-ntua/FAST_LIO_LOCALIZATION
35. engcang/FAST-LIO-Localization-QN: A Map-based localization implementation combining FAST-LIO2 as an odometry with Quatro + Nano-GICP as a map matching method - GitHub, accessed July 16, 2025, https://github.com/engcang/FAST-LIO-Localization-QN
36. zlwang7/S-FAST_LIO - GitHub, accessed July 16, 2025, https://github.com/zlwang7/S-FAST_LIO
37. hku-mars/FAST_LIO: A computationally efficient and robust LiDAR-inertial odometry (LIO) package - GitHub, accessed July 16, 2025, https://github.com/hku-mars/FAST_LIO
38. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 16, 2025, https://github.com/TixiaoShan/LIO-SAM
39. engcang/SLAM-application: LeGO-LOAM, LIO-SAM, LVI-SAM, FAST-LIO2, Faster-LIO, VoxelMap, R3LIVE, Point-LIO, KISS-ICP, DLO, DLIO, Ada-LIO, PV-LIO, SLAMesh, ImMesh, FAST-LIO-MULTI, M - GitHub, accessed July 16, 2025, https://github.com/engcang/SLAM-application
40. Error comparison of FAST‐LIO2, Point‐LIO‐input, and Point‐LIO, for rotation and position estimations. - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/figure/Error-comparison-of-FAST-LIO2-Point-LIO-input-and-Point-LIO-for-rotation-and-position_fig13_369891453
41. Robust Odometry and Fully Autonomous Driving Robot | ATCrawler x FAST-LIO x Navigation2 - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=0oBHPxRycTk
42. Adaptive-LIO: Enhancing Robustness and Precision through Environmental Adaptation in LiDAR Inertial Odometry - arXiv, accessed July 16, 2025, https://arxiv.org/html/2503.05077v1
43. Lidar SLAM Comparison in a Featureless Tunnel Environment - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/366979766_Lidar_SLAM_Comparison_in_a_Featureless_Tunnel_Environment
44. LIO-Livox/README.md at master - GitHub, accessed July 16, 2025, https://github.com/Livox-SDK/LIO-Livox/blob/master/README.md
45. A Localization and Mapping Algorithm Based on Improved LVI-SAM for Vehicles in Field Environments - MDPI, accessed July 16, 2025, https://www.mdpi.com/1424-8220/23/7/3744


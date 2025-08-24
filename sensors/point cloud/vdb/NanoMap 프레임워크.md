# NanoMap 프레임워크
[VDB](./index.md)



로보틱스 분야, 특히 무인 항공기(UAV) 또는 드론과 같은 자율 이동 로봇이 숲, 창고, 도심과 같이 복잡하고 미지의 동적 환경에서 고속으로 안전하게 비행하는 것은 오랫동안 해결해야 할 핵심적인 난제로 남아있다.1 이러한 임무를 성공적으로 수행하기 위해서는 로봇이 주변 환경을 실시간으로 인식하고, 자신의 위치를 추정하며, 잠재적 충돌을 회피하기 위한 경로를 신속하게 계획하고 실행해야 한다.

이 문제에 대한 전통적인 접근법은 주로 점유 격자 지도(Occupancy Grid Map)나 유클리드 부호 거리 필드(ESDF)와 같은 형태의 지도를 구축하는 것에 의존해왔다.1 이러한 방식들은 깊이 카메라나 LiDAR와 같은 3D 센서로부터 수집된 측정값들을 공통의 전역 좌표계(global world frame)에 점진적으로 융합(fusion)하여 환경에 대한 일관된 단일 모델을 생성하고자 한다.2 그러나 이러한 전역적 융합 기반의 매핑 방식은 고속 자율 비행이라는 특정 과업에 적용될 때 몇 가지 근본적인 한계에 직면한다.

첫째, 상태 추정(state estimation) 오류에 매우 취약하다.2 로봇의 위치나 방향 추정에 작은 오차만 발생해도, 이 오차는 지도 전체로 전파되어 기존에 구축된 지도를 오염시킨다. 이는 지도와 실제 환경 간의 불일치를 야기하며, 결국 로봇이 장애물에 충돌하는 치명적인 결과를 초래할 수 있다. 특히 고속으로 비행할 때는 컴퓨터 비전 알고리즘의 성능이 저하되어 주변 환경을 정확히 파악하기 어렵고, 이로 인해 관성 측정 장치(IMU) 센서의 부정확한 데이터에 더 크게 의존하게 되어 상태 추정 오류가 더욱 빈번하고 심각해진다.4

둘째, 계산 비용이 매우 높다.2 새로운 센서 데이터가 들어올 때마다 이를 전역 지도에 통합하고, 확률적 업데이트를 수행하며, 때로는 지도의 일관성을 유지하기 위해 복잡한 최적화를 수행해야 한다. 이러한 과정은 상당한 계산 자원을 소모하여, 실시간으로 급변하는 상황에 신속하게 대응해야 하는 고속 비행 시나리오에서는 병목 현상을 유발한다.


이러한 기존 접근법의 한계를 극복하기 위한 대안적 프레임워크로서 MIT 컴퓨터 과학 및 인공지능 연구소(CSAIL)에서 NanoMap이 제안되었다.2 NanoMap은 전역적 융합이라는 전통적인 패러다임에서 벗어나, '지연 탐색(Lazy Search)'과 '불확실성 인식(Uncertainty-Awareness)'이라는 두 가지 혁신적인 개념을 핵심 철학으로 제시한다.1 이는 모든 정보를 단일 지도에 완벽하게 통합하려는 시도 대신, 의사결정에 필요한 정보를 필요한 시점에, 그 정보의 불확실성을 명시적으로 고려하여 질의하는 방식으로 문제에 접근한다.

본 보고서는 이 NanoMap 프레임워크에 대한 심층적인 기술적 고찰을 목적으로 한다. 먼저, 2018년 국제 로봇 자동화 학회(ICRA)에서 발표된 원본 NanoMap의 핵심 철학, 데이터 구조, 그리고 불확실성 처리 메커니즘을 상세히 분석할 것이다. 이어서, 2022년에 발표된 GPU 가속 버전의 NanoMap 라이브러리가 어떻게 고성능 컴퓨팅을 통해 매핑 성능을 극대화했는지 살펴보고, 대표적인 3D 매핑 기술인 OctoMap과의 정량적 성능 비교를 통해 그 실용적 가치를 평가한다. 마지막으로, NanoMap의 응용 분야, 시스템 통합의 가능성, 그리고 내재된 한계와 향후 연구 방향을 논함으로써, 이 기술이 로보틱스 분야에 미친 영향과 미래 잠재력을 종합적으로 조망하고자 한다.


NanoMap을 분석하기에 앞서, 동일한 이름으로 불리지만 실제로는 서로 다른 목표와 기술적 초점을 가진 두 개의 시스템이 존재한다는 점을 명확히 인지하는 것이 매우 중요하다. 이를 구분하지 않으면 NanoMap의 본질과 기술적 진보를 오해할 소지가 있다.

첫 번째는 2018년 Peter Florence, Russ Tedrake 등이 발표한 **'개념적 프레임워크'로서의 NanoMap**이다.2 이는 MIT CSAIL의 연구에 기반하며, 주로 CPU에서 동작한다. 이 시스템의 핵심 기여는 '어떻게 로봇의 자세 불확실성을 경로 계획에 효과적으로 통합할 것인가?'라는 질문에 대한 해답으로 '지연 탐색'과 '비융합' 철학을 제시한 데 있다.1 이는 학술적, 개념적 혁신에 중점을 둔다.

두 번째는 2022년 Violet Walker 등이 발표하고 퀸즐랜드 공과대학교(QUT) 연구진이 개발한 **'고성능 라이브러리'로서의 NanoMap**이다.12 이 시스템은 CUDA와 OpenVDB 라이브러리를 활용하여 GPU 가속을 통한 고속 점유 격자 지도 생성에 초점을 맞춘다.12 이 시스템의 핵심 질문은 '어떻게 매핑 과정을 더 빠르게 만들 것인가?'이며, OctoMap과 같은 기존 기술과의 성능 벤치마크를 통해 그 우수성을 입증하는 데 중점을 둔다.

이 두 시스템은 이름은 같지만, 저자, 발표 시기, 기술적 목표, 그리고 핵심 철학에서 명백한 차이를 보인다. 원본 NanoMap이 '불확실성을 다루는 새로운 방법론'을 제시했다면, GPU 버전 NanoMap은 '매핑을 가속하는 새로운 구현 기술'을 제시한 것이다. 따라서 본 보고서는 이 두 시스템을 명확히 구분하여 각각의 기여와 특성을 독립적으로, 그리고 상호 연관적으로 분석함으로써 NanoMap이라는 이름 아래 이루어진 기술적 성과를 다각적으로 이해하고자 한다.


이 장에서는 MIT에서 제안한 원본 NanoMap의 혁신적인 개념과 그 아키텍처를 중심으로, 전통적인 매핑 방식과 근본적으로 다른 접근법을 상세히 분석한다.


전통적인 매핑 방식의 근간은 과거부터 현재까지의 모든 관측 데이터를 하나의 일관된 전역 지도에 통합하는 '융합(fusion)' 과정에 있다. 이와 대조적으로, NanoMap의 가장 핵심적인 철학은 이러한 융합과 이산화(discretization) 과정을 의도적으로 포기하는 것이다.1

대신 NanoMap은 로봇이 이동하면서 얻는 연속적인 정보들을 '지연(lazy)'하여 처리하는 방식을 채택한다. 구체적으로, 로봇의 상대적인 자세 변환(relative pose transforms) 이력과 각 시점에서 획득한 원본(raw) 깊이 센서 측정값을 융합하지 않고 그대로 저장한다.1 이는 마치 "로봇이 세상을 바라보며 촬영한 모든 3D 스냅샷과 그 스냅샷을 찍을 때의 상대적 위치 변화 기록을 테이프처럼 길게 이어 붙여 보관하는 것"과 유사한 개념으로 설명될 수 있다.5

이러한 '지연' 접근 방식은 시스템에 몇 가지 중요한 이점을 제공한다. 가장 직접적인 효과는 데이터 구조의 구축 시간(build time)이 극적으로 감소한다는 점이다. 새로운 센서 데이터가 들어왔을 때, 복잡한 융합 계산 없이 단순히 데이터 목록의 끝에 새로운 측정값과 변환을 추가하기만 하면 되기 때문이다. 이로 인해, 소량의 모션 플래닝 쿼리(실험적으로 약 10,000개 미만)가 발생하는 시나리오에서는, 초기에 방대한 융합 지도를 구축해야 하는 전통적인 방식보다 훨씬 높은 계산 효율성을 보인다.1 이처럼 계산을 필요한 순간까지 미루는 것이 '지연 탐색'의 핵심이다.


NanoMap의 지연 탐색 철학을 효율적으로 구현하기 위해 고안된 핵심 데이터 구조는 '엣지-정점(edge-vertex)' 쌍으로 구성된 체인(chain)이다.2

- **엣지(Edge):** 두 연속된 센서 측정 시점 간의 상대적 자세 변환($T_{S_i S_{i-1}}$)과 그 변환의 불확실성($\Sigma_{T_{S_i S_{i-1}}}$)을 저장한다.
- **정점(Vertex):** 특정 시점에 획득한 원본 3D 포인트 클라우드 데이터와, 이 데이터에 대한 빠른 근접 이웃 탐색을 위해 사전 처리된 k-d 트리(k-dimensional tree)를 포함한다.2

이러한 엣지-정점 쌍의 체인은 **이중 연결 리스트(doubly-linked list)** 형태로 구현된다. 이 자료구조를 선택한 이유는 NanoMap이 고정된 시간 윈도우(time-window) 내의 지역적인 데이터만을 유지하기 때문이다. 새로운 데이터가 추가될 때 리스트의 맨 앞에 새로운 노드를 삽입하고, 가장 오래된 데이터는 리스트의 맨 뒤에서 제거한다. 이중 연결 리스트는 이러한 양 끝단에서의 삽입 및 삭제 연산을 상수 시간 복잡도($O(1)$)로 매우 효율적으로 수행할 수 있게 해준다.2

각 정점에 포함된 k-d 트리는 해당 시점의 포인트 클라우드 데이터를 공간적으로 분할하여 저장하는 자료구조이다. 이는 특정 뷰(view) 내에서 질의 지점(query point)과 가장 가까운 k개의 이웃 점들(k-nearest-neighbors)을 빠르게 찾기 위한 목적으로 사용된다.1 즉, 이중 연결 리스트가 시간적 데이터 관리를 담당한다면, k-d 트리는 각 시간 스냅샷 내에서의 공간적 데이터 검색을 가속화하는 역할을 한다.


모션 플래너가 경로상의 특정 지점(예: 궤적 샘플)이 장애물과 충돌하는지 확인하기 위해 점유 상태를 질의하면, NanoMap은 다음과 같은 독특한 쿼리 프로세스를 수행한다.

1. **시간 역순 탐색 (Reverse Search):** NanoMap은 저장된 데이터 이력을 현재 시점에서부터 과거 방향으로, 즉 시간의 역순으로 탐색한다(reverse searching over time).1 이는 가장 최신의 정보부터 확인하여 가장 신뢰도 높은 데이터를 우선적으로 사용하려는 의도이다.

2. **최소 불확실성 뷰 탐색 (Minimum-Uncertainty View Search):** 탐색의 목표는 단순히 질의 지점을 포함하는 뷰를 찾는 것이 아니라, 현재 로봇의 자세를 기준으로 '최소 불확실성(minimum-uncertainty)'을 갖는 관측 뷰를 찾는 것이다.1 2장에서 상세히 다룰 불확실성 전파 모델에 따라, 현재로부터 시간적으로 가까운 뷰일수록 누적된 자세 오차가 적어 불확실성이 낮다. 따라서 시간 역순 탐색은 자연스럽게 최소 불확실성 뷰를 찾는 과정이 된다.

3. **시야각 내 판별 (IsInFOV Check):** 알고리즘은 질의 지점의 좌표를 이중 연결 리스트를 따라가며 각 과거 센서 프레임($S_i$)의 좌표계로 순차적으로 변환한다. 그리고 각 프레임에서 변환된 질의 지점이 해당 센서의 시야각(Field of View, FoV) 내에 위치하는지를 검사한다. 이 검사 함수가 바로 `IsInFOV()`이다.1`IsInFOV()`는 질의 지점의 투영된 이미지 좌표($u, v$)와 깊이($z$)가 센서의 해상도 및 측정 가능 범위 내에 있는지 확인한다.

4. **최근접 이웃 반환 (K-NN Return):** `IsInFOV()` 조건을 만족하는 첫 번째(즉, 가장 최근의) 뷰를 찾으면 탐색을 중단한다. 그리고 해당 뷰의 정점에 저장된 k-d 트리를 사용하여 질의 지점 주변의 최근접 이웃 점들을 검색하여 반환한다.1 이 이웃 점들이 바로 질의 지점 주변의 장애물 정보를 나타낸다.

이러한 계산의 '지연'과 '비융합' 철학은 단순히 계산을 미루는 소극적 행위가 아니라, 시스템에 세 가지 구체적이고 강력한 이점을 부여하는 능동적인 설계적 선택이다. 첫째, 데이터를 원본 뷰와 변환 그대로 유지하기 때문에, 각 뷰에 연관된 고유의 자세 불확실성을 쿼리 시점에 직접적으로 고려할 수 있다.1 융합된 지도에서는 개별 측정의 불확실성 정보가 손실되거나 평균화되어 버리는 것과 대조적이다.

둘째, SLAM 시스템으로부터 루프 클로저(loop closure) 정보가 수신되어 과거의 특정 자세가 수정되어야 할 때, 융합 기반 지도는 지도 전체 또는 상당 부분을 재계산해야 하는 막대한 비용이 발생한다. 반면 NanoMap은 단순히 이중 연결 리스트 내의 특정 엣지(자세 변환 값) 하나만 수정하면 되므로, 2에서 4 자릿수(orders of magnitude) 더 빠른, 거의 즉각적인 업데이트가 가능하다.2 이는 계산을 미래로 '지연'시켰기 때문에 얻을 수 있는 구조적 유연성의 극적인 예이다.

셋째, 동적 환경에 대한 암묵적인 강인성을 제공한다. 동적 장애물이 나타나거나 사라져도 NanoMap은 지도를 명시적으로 '수정'할 필요가 없다. 가장 최근의 뷰를 우선적으로 찾는 쿼리 메커니즘은 자연스럽게 오래되어 더 이상 유효하지 않은 정보(예: 장애물이 이미 사라진 위치의 관측)를 무시하고 최신 정보만을 사용하게 만든다. 이는 별도의 동적 객체 추적 알고리즘 없이도 시스템에 기본적인 동적 환경 대응 능력을 부여하는 효과를 낳는다.


NanoMap의 핵심적인 기여 중 하나는 로봇의 불확실성을 명시적으로 모델링하고 이를 경로 계획 과정에 직접적으로 활용한다는 점이다. 이 장에서는 NanoMap이 불확실성을 어떻게 수학적으로 정의하고 전파하는지 상세히 분석한다.


NanoMap은 로봇이 자신의 정확한 위치(position)와 방향(orientation)을 완벽하게 알지 못한다는 현실을 인정하고, 이 '자세 불확실성(pose uncertainty)'을 수학적 모델에 포함시킨 최초의 시스템 중 하나로 평가받는다.4

여기서 중요한 설계적 선택이 이루어지는데, NanoMap의 불확실성 모델은 주로 **병진 불확실성(translational uncertainty)**에 초점을 맞추고, **회전 불확실성(rotational uncertainty)**은 모델링에서 명시적으로 제외한다.2 이 결정은 이론적 완결성보다는 실제 로봇 시스템에서의 실용성에 기반을 둔다. IMU와 같은 관성 센서는 중력 벡터를 참조할 수 있기 때문에 롤(roll)과 피치(pitch) 각도에 대한 관측 가능성(observability)이 비교적 양호하다. 반면, 로봇의 위치는 가속도계 측정값을 시간에 대해 이중 적분하여 얻어지므로 오차가 매우 빠르게 누적되는 경향이 있다. 요(yaw) 각도는 자이로스코프의 단일 적분으로 계산되지만, 외부 참조(예: 자력계, GPS) 없이는 드리프트가 발생하기 쉽다. 이러한 특성들을 종합적으로 고려할 때, 단기적인 고속 기동 상황에서는 병진 운동에 대한 불확실성이 회전 불확실성보다 더 지배적인 오차 요인으로 작용할 수 있다는 실용적 판단이 이 모델의 근간을 이룬다.2

또한, NanoMap은 로봇의 자세 추정에서 발생하는 불확실성에 집중하며, 깊이 센서 자체의 측정 노이즈(예: 측정 거리에 따라 증가하는 오차)는 모델링 범위 밖(out of scope)으로 명확히 정의하였다.1 이는 문제의 복잡성을 제어하고 핵심 아이디어에 집중하기 위한 의도적인 단순화이다.


NanoMap에서 경로 계획기가 질의하는 특정 지점(query point)과 로봇의 과거 프레임 간의 각 상대 변환은 3차원 가우시안 확률 분포를 따르는 병진 불확실성으로 모델링된다.2 불확실성이 과거로 전파되는 과정은 다음과 같은 수학적 원리에 의해 기술된다.

현재 로봇의 본체(Body) 좌표계 $B$에서 질의 지점 $x_B$가 평균 위치 벡터 $\mu_B$와 공분산 행렬 $\Sigma_B$를 갖는 정규분포 $x_B \sim N(\mu_B, \Sigma_B)$를 따른다고 가정하자. 이 질의 지점의 불확실성은 데이터 구조에 저장된 이력을 따라 시간의 역순으로, 즉 현재 프레임에서 과거 센서 프레임 $S_0, S_1, \dots, S_N$으로 전파된다.

$i-1$번째 센서 프레임 $S_{i-1}$에서 $i$번째 센서 프레임 $S_i$로의 상대 변환이 $T_{S_i S_{i-1}}$로 주어지고, 이 변환 자체의 병진 불확실성이 공분산 $\Sigma_{T_{S_i S_{i-1}}}$를 갖는다고 할 때, 불확실성 전파는 다음과 같이 재귀적으로 계산된다.

평균(Mean) 전파:

질의 지점의 평균 위치는 표준 동차 변환(homogeneous transformation)을 통해 전파된다.
$$
\mu_{S_i} = T_{S_i S_{i-1}} \mu_{S_{i-1}}
$$
여기서 $\mu_{S_i}$는 $S_i$ 좌표계에서 표현된 질의 지점의 평균 위치 벡터이다. 이 재귀식의 시작은 현재 로봇 본체 프레임 $B$에서 가장 최근의 센서 프레임 $S_0$으로의 변환으로부터 시작된다: $\mu_{S_0} = T_{S_0 B} \mu_B$.

공분산(Covariance) 전파:

질의 지점의 공분산, 즉 불확실성의 크기와 방향은 선형화된 불확실성 전파 법칙에 따라 다음과 같이 계산된다.
$$
\Sigma_{S_i} = R_{S_i S_{i-1}} \Sigma_{S_{i-1}} R_{S_i S_{i-1}}^T + \Sigma_{T_{S_i S_{i-1}}}
$$
이 수식의 각 항은 다음과 같은 의미를 가진다.

- $\Sigma_{S_i}$: $S_i$ 프레임에서 바라본 질의 지점의 누적된 공분산 행렬.
- $\Sigma_{S_{i-1}}$: 이전 프레임 $S_{i-1}$까지 누적된 공분산 행렬.
- $R_{S_i S_{i-1}}$: 변환 행렬 $T_{S_i S_{i-1}}$의 회전 행렬 부분. 이 항은 이전 프레임의 불확실성 타원체를 현재 프레임의 방향으로 회전시키는 역할을 한다.
- $\Sigma_{T_{S_i S_{i-1}}}$: 상대 변환 $T_{S_i S_{i-1}}$ 자체에 내재된 병진 불확실성의 공분산.

이 수식은 각 변환 단계를 거칠 때마다, 이전까지 누적된 불확실성(첫 번째 항)에 해당 변환 자체의 불확실성(두 번째 항)이 더해지는 과정을 명확하게 보여준다. 결과적으로, 현재 시점으로부터 시간적으로 더 멀리 떨어진(즉, $i$가 더 큰) 과거 프레임으로 갈수록 공분산 행렬 $\Sigma_{S_i}$의 값들은 커지게 된다. 이는 해당 과거 관측이 현재 로봇의 위치를 기준으로 할 때 더 큰 불확실성을 가짐을 정량적으로 나타내며, NanoMap이 '최소 불확실성 뷰'를 선택하는 근거가 된다.

이러한 불확실성 모델은 이론적 완결성보다는 실시간 성능과 실용성을 최우선으로 고려한 '의도된 단순화'의 산물로 이해해야 한다. 완벽한 불확실성 모델은 병진, 회전, 센서 고유 노이즈, 시간에 따라 변하는 노이즈 특성 등 모든 요소를 포함해야 하지만, 이는 실시간 시스템에서는 감당하기 어려운 계산 복잡성을 야기한다.16 고속 드론 비행과 같은 응용 분야에서는 수 밀리초의 계산 지연도 치명적인 충돌로 이어질 수 있다.1 연구진은 IMU 데이터의 물리적 특성을 고려하여 단기적으로는 위치 추정 오차가 회전 오차보다 더 지배적인 영향을 미친다고 판단했고 2, 이에 따라 계산 비용이 높은 회전 불확실성과 센서 노이즈 모델링을 과감히 생략했다. 이 실용적인 타협을 통해 계산 복잡성을 크게 줄이면서도, 실제 장애물 회피에 '충분히 좋은(good enough)' 불확실성 정보를 제공할 수 있었다. 이러한 전략적 결정 덕분에 NanoMap은 이론적 정교함을 일부 희생하는 대신, 실제 하드웨어에서 시속 10m/s의 고속 비행을 성공적으로 시연하며 그 실효성을 입증할 수 있었다.2 이는 완벽함이 때로는 실용성의 적이 될 수 있는 공학적 문제에서, 현명한 트레이드오프가 어떻게 성공적인 시스템으로 이어질 수 있는지를 보여주는 대표적인 사례이다.


이 장에서는 2022년에 발표된 GPU 가속 버전의 NanoMap 라이브러리를 중심으로, 3D 환경 표현을 위한 대표적인 기술인 OctoMap과의 정량적 성능을 심층적으로 비교 분석한다. 이 분석은 주로 해당 연구 논문에서 제시된 벤치마크 결과에 기반한다.13


두 기술의 성능 차이를 이해하기 위해서는 먼저 그들의 근본적인 구조와 설계 철학의 차이를 파악해야 한다.

- **OctoMap:** OctoMap은 환경을 3차원 복셀(voxel)로 나누고, 각 복셀의 점유 확률을 **옥트리(octree)**라는 계층적 데이터 구조에 저장한다.13 옥트리는 공간을 재귀적으로 8개의 하위 공간으로 분할하는 트리 구조로, 비어있거나 동일한 상태의 넓은 공간을 단일 노드로 표현할 수 있어 메모리 사용량을 크게 줄일 수 있다.13 또한, 점유(occupied), 비점유(free), 미탐사(unexplored) 영역을 명확하게 구분할 수 있는 장점이 있다.20 그러나 특정 복셀에 접근하거나 업데이트하기 위해서는 트리의 루트부터 리프 노드까지 탐색해야 하므로, 지도의 해상도가 높아져 트리의 깊이가 깊어질수록 연산 속도가 저하될 수 있는 구조적 한계를 가진다.13

- **NanoMap (GPU 버전):** 이 버전의 NanoMap은 시각 효과 산업에서 대용량 희소 데이터셋을 처리하기 위해 널리 사용되어 온 **OpenVDB** 데이터 구조를 기반으로 한다.13 OpenVDB는 B+트리와 유사한 넓고 얕은 트리 구조를 사용하여, 데이터 접근 시 고정된 탐색 깊이를 가지므로 해상도 증가에 따른 접근 속도 저하가 적다.13 더 중요한 것은, 이 라이브러리는 점유 격자 생성의 핵심 연산인 광선 투사(ray-casting)를 **GPU의 대규모 병렬 컴퓨팅**을 통해 처리하도록 설계되었다는 점이다.12 이는 수천 개의 코어를 활용하여 수많은 광선을 동시에 처리함으로써, 특히 밀집된(dense) 포인트 클라우드 데이터 처리에서 CPU 기반 방식에 비해 압도적인 속도 향상을 목표로 한다.


벤치마크는 고성능 랩탑(Asus G14, RTX 2060)과 저전력 임베디드 보드(Nvidia Jetson Nano) 두 플랫폼에서 수행되었으며, 프레임당 평균 처리 시간(ms)을 측정하여 계산 속도를 비교했다.

- **정확도:** 생성된 점유 지도의 기하학적 형태와 복셀의 점유 값 측면에서, GPU NanoMap의 다양한 필터 모드와 OctoMap 간에 유의미한 차이는 발견되지 않았다. 이는 두 시스템이 최종적으로 유사한 수준의 매핑 정확도를 제공함을 시사한다.13

- **메모리 사용량:** GPU NanoMap은 다양한 필터 모드를 제공하여 사용자가 성능과 메모리 사용량 간의 트레이드오프를 조절할 수 있게 한다. 특히 Filter Mode 3는 저전력 GPU를 위해 메모리 비용을 최소화하도록 설계되어, 자원이 제약된 환경에서의 효율성을 보여준다.13

- **계산 속도 (지도 업데이트 시간):**

  Frustum 센서 (RGB-D, Stereo 등 밀집 데이터):

  아래 표는 밀집 포인트 클라우드를 처리할 때의 성능을 보여준다. 이는 GPU 가속의 효과가 가장 극명하게 드러나는 시나리오이다.

  #### Table 3.1: Frustum 센서 성능 비교 (Asus G14, ms, 낮을수록 좋음)

  이 표는 고성능 컴퓨팅 환경에서 GPU 가속이 가져오는 극적인 성능 향상을 명확하게 보여준다. NanoMap의 GPU 가속 모드들은 CPU 기반의 OctoMap 및 NanoMap CPU 버전에 비해 압도적으로 빠른 처리 속도를 기록했다. 이는 고속 처리가 필수적인 로보틱스 응용 분야에서 GPU 활용의 당위성을 정량적으로 입증한다.

| 해상도 | NM Filter 2 (GPU) | NM No Filter (GPU) | NM (CPU) | OctoMap | OctoMap Discretized |
| ------ | ----------------- | ------------------ | -------- | ------- | ------------------- |
| 0.1m   | **2.70**          | 3.12               | 116.25   | 144.51  | 20.25               |
| 0.05m  | **3.96**          | 5.82               | 219.93   | 283.38  | 54.65               |
| 0.025m | **11.52**         | 19.38              | 432.90   | 797.57  | 332.99              |
| 0.01m  | 100.82            | **89.58**          | 1164.55  | 4983.70 | 4559.49             |

*출처: [13] 기반 재구성*

이 표의 결과는 자원이 제한된 임베디드 환경, 즉 실제 로봇에 탑재되는 환경에서의 성능을 보여준다는 점에서 매우 중요하다. Jetson Nano에서의 결과는 NanoMap의 GPU 가속이 단순한 학술적 성능 과시가 아니라, 실제 로봇 플랫폼에서 30Hz와 같은 고속 센서 데이터를 실시간으로 처리하며 안정적인 운영을 가능하게 하는 실용적인 기술임을 증명한다.

| 해상도 | NM Filter 2 (GPU) | NM Filter 3 (GPU) | NM (CPU) | OctoMap | OctoMap Discretized |
| ------ | ----------------- | ----------------- | -------- | ------- | ------------------- |
| 0.1m   | **10.62**         | 11.23             | 533.74   | 498.77  | 58.87               |
| 0.05m  | **28.61**         | 30.15             | 1000.86  | 969.68  | 163.42              |
| 0.025m | 134.00            | **132.50**        | 1955.14  | 2465.05 | 1015.57             |

*출처: [13] 기반 재구성*

**Laser 센서 (LIDAR 등 희소 데이터):**
LIDAR와 같이 상대적으로 희소한(sparse) 데이터를 처리할 때는 GPU의 대규모 병렬 처리 이점이 줄어든다. 아래 표는 이 시나리오에서 CPU만으로 동작했을 때의 성능을 비교한다.

이 표는 GPU 가속의 이점이 적은 희소 데이터 환경에서의 성능을 보여준다. 여기서 주목할 점은, GPU 없이 CPU만으로도 NanoMap이 OctoMap을 능가한다는 사실이다. 이는 NanoMap의 성능 우위가 단순히 GPU 하드웨어에만 의존하는 것이 아니라, 기반 데이터 구조인 OpenVDB 자체의 구조적 효율성에도 크게 기인함을 시사한다. 이는 NanoMap 아키텍처의 근본적인 강점을 보여주는 중요한 증거다.

| 센서 범위 | 해상도 | NM (CPU)  | OctoMap | OctoMap Discretized |
| --------- | ------ | --------- | ------- | ------------------- |
| 5m        | 0.1m   | **7.39**  | 38.39   | 37.41               |
| 5m        | 0.05m  | **16.21** | 184.99  | 206.68              |
| 20m       | 0.1m   | **20.69** | 59.88   | 59.61               |
| 20m       | 0.05m  | **55.06** | 301.99  | 344.24              |

```
*출처: [13] 기반 재구성*
```


- **NanoMap (GPU 버전):**
  - **장점:** 밀집 센서(RGB-D, Stereo) 데이터 처리에 있어 압도적인 속도를 제공하여 실시간 고해상도 매핑을 가능하게 한다. 저전력 임베디드 보드에서도 높은 성능을 발휘하며, CPU 버전조차 희소 데이터(LIDAR) 처리에서 OctoMap보다 우수하다.12
  - **단점:** GPU 가속의 이점을 최대한 활용하기 위해서는 Nvidia CUDA를 지원하는 하드웨어가 필수적이다.12
  - **적용 시나리오:** 실시간 장애물 회피가 생명과 직결되는 고속 드론, 자율주행차, 동적 환경에서의 로봇 네비게이션 등 고성능이 요구되는 모든 분야.
- **OctoMap:**
  - **장점:** 오랫동안 널리 사용되어 안정성과 신뢰성이 검증된 프레임워크이다. GPU 없이 순수 CPU만으로 동작하며, 미탐사 영역을 명시적으로 표현하는 기능과 높은 메모리 효율성은 여전히 유효한 장점이다.13
  - **단점:** 고해상도, 고밀도 데이터 처리 시 심각한 성능 저하가 발생하여 실시간 고속 대응에 명백한 한계를 보인다.13
  - **적용 시나리오:** 정적 환경에서의 전역 지도 구축, 오프라인 지도 생성, 실시간성이 상대적으로 덜 중요한 애플리케이션에 적합하다.

결론적으로, GPU 버전 NanoMap의 성능 우위는 단지 GPU를 사용했다는 단일 요인에서 비롯된 것이 아니다. 이는 점유 격자 지도 생성의 핵심 연산인 '광선 투사'라는 작업이 본질적으로 병렬화에 매우 적합하다는 특성을 간파하고, 이를 GPU라는 하드웨어의 대규모 병렬 처리 능력으로 극대화한 결과이다.13 동시에, 해상도가 높아져도 성능 저하가 적은 OpenVDB라는 효율적인 자료구조를 채택함으로써 알고리즘의 병목 현상을 최소화했다.13 즉, NanoMap의 성공은 '알고리즘(광선 투사)의 병렬성'을 '하드웨어(GPU)'로 증폭시키고, 이 과정에서 발생할 수 있는 데이터 접근 병목을 '자료구조(OpenVDB)'로 해결한, 소프트웨어와 하드웨어, 그리고 자료구조의 이상적인 시너지 결과물이라고 평가할 수 있다.


NanoMap은 그 독특한 철학과 아키텍처 덕분에 특정 로보틱스 응용 분야에서 강력한 성능을 발휘하며, 기존 시스템과 상호 보완적으로 통합될 수 있는 큰 잠재력을 가지고 있다.


NanoMap의 가장 대표적이고 성공적인 응용 분야는 단연코 복잡하고 미지의 환경에서의 드론 고속 자율 비행 및 장애물 회피다.2 연구 결과에 따르면, NanoMap을 탑재한 쿼드콥터는 숲이나 창고와 같은 밀집된 환경에서 최대 시속 10m/s (약 36km/h) 또는 20mph (약 32km/h)의 속도로 안정적인 비행을 수행하는 데 성공했다.4

이러한 성공의 핵심 요인은 자세 불확실성을 명시적으로 모델링하고 경로 계획에 반영한 데 있다. 한 실험에서는 불확실성을 모델링하지 않았을 경우, 로봇의 실제 위치가 예상 위치에서 단 5%만 벗어나도 네 번의 비행 중 한 번 이상(충돌률 25% 이상) 충돌하는 결과를 보였다. 반면, NanoMap을 통해 불확실성을 고려했을 때는 충돌률이 2%로 극적으로 감소하여 비행의 신뢰성과 안전성을 크게 향상시켰다.4

이러한 능력은 단순히 빠른 비행을 넘어, 다양한 실제 응용 분야로의 확장 가능성을 시사한다. 예를 들어, 재난 현장에서의 신속한 수색 및 구조 임무, 적대적 환경에서의 국방 및 정찰, 도심 내 소포 배달, 그리고 박진감 넘치는 영상 촬영을 위한 엔터테인먼트 분야 등에서 활용될 수 있다. 나아가, 그 원리는 드론뿐만 아니라 자율주행차나 지상 로봇과 같은 다른 형태의 자율 이동체에도 동일하게 적용될 수 있다.1


NanoMap은 데이터를 전역 지도에 영구적으로 융합하지 않고, 제한된 시간 윈도우 내의 최신 정보만을 유지하는 구조를 가지고 있다.2 이 설계는 동적 환경에 대한 내재적(implicit) 강인성을 부여하는 예기치 않은 장점을 낳는다.

예를 들어, 사람이 걸어가거나 문이 열리는 등 환경에 변화가 생겼을 때, 융합 기반 지도는 이전의 정적 장애물 정보와 새로운 정보를 구분하고 충돌하는 정보를 해결해야 하는 복잡한 문제를 마주한다. 그러나 NanoMap의 '최소 불확실성 뷰 탐색' 쿼리 메커니즘은 자연스럽게 가장 최신의, 즉 현재 시점에서 유효한 환경 정보만을 우선적으로 사용하게 된다. 과거에 관측된 움직이는 장애물의 이전 위치 정보는 시간적으로 뒤쳐져 있으므로 쿼리 과정에서 선택될 확률이 낮아져 자연스럽게 무시된다.

하지만 여기서 중요한 점은 이것이 **암묵적인(implicit)** 대응 방식이라는 것이다. NanoMap 자체가 움직이는 객체를 명시적으로 탐지(detection), 추적(tracking), 또는 미래 궤적을 예측(prediction)하는 기능은 포함하지 않는다.25 따라서 복잡한 동적 환경에서 보다 정교하고 선제적인 회피 기동을 수행하기 위해서는, DOT(Dynamic Object Tracking) 25나 CC-MPC(Chance-Constrained Model Predictive Control) 3와 같이 동적 객체의 상태를 명시적으로 추정하고 예측하는 별도의 모듈과 통합하는 연구가 필요하다.


NanoMap과 SLAM(Simultaneous Localization and Mapping, 동시적 위치추정 및 지도작성)의 관계를 이해하는 것은 시스템 아키텍처 설계에 있어 매우 중요하다.

- **명확한 역할 구분:** 결론부터 말하자면, NanoMap은 SLAM 시스템이 **아니다**.6 SLAM의 근본적인 목표는 로봇이 미지의 환경을 탐사하면서 "나는 지금 어디에 있는가(Localization)?"와 "주변 지도는 어떻게 생겼는가(Mapping)?"라는 두 가지 질문에 대한 답을 동시에 찾아가는 과정이다.29 이는 전역적이고 일관된 지도와 궤적을 생성하는 데 초점을 맞춘다. 반면, NanoMap은 SLAM 시스템으로부터 받은 '현재 나의 위치(자세)'와 그 불확실성을 입력으로 받아, "이 위치를 기준으로 할 때, 당장 내 앞의 장애물을 어떻게 피할 것인가?"라는 지극히 지역적(local)이고 즉각적인 질문에 대한 해답을 찾는 데 특화된 시스템이다.
- **상호 보완적 파트너 관계:** NanoMap은 SLAM 시스템의 '소비자(consumer)' 역할을 한다. VINS-Mono와 같은 최신 Visual-Inertial SLAM 알고리즘 30은 실시간으로 로봇의 자세와 그 불확실성 정보를 제공하는데, NanoMap은 바로 이 정보를 입력으로 받아 지역적인 불확실성 인식 지도를 구성하고 경로 계획에 활용한다. 즉, SLAM이 제공하는 전역적 위치 정보를 바탕으로 NanoMap이 지역적 안전 기동을 수행하는 상호 보완적인 관계이다.
- **통합의 미래:** NanoMap의 개발자는 장기적으로 SLAM과 NanoMap을 통합하여 두 접근법의 장점을 모두 취하는 것을 향후 목표로 언급한 바 있다.28 이는 NanoMap의 빠른 지역 반응성(local reactivity)과 SLAM의 전역적 일관성(global consistency) 및 루프 클로저를 통한 장기적 오차 보정 능력을 결합하는 것을 의미한다.

이러한 관계를 로보틱스 시스템의 계층적 구조 관점에서 보면, SLAM은 상위 레벨의 '전략/인지(Strategy/Cognition)' 계층에, NanoMap은 하위 레벨의 '전술/실행(Tactic/Execution)' 계층에 속한다고 볼 수 있다. SLAM을 대체하는 것이 아니라, SLAM이 제공하는 전역적 맥락 위에서 로봇이 빠르고 안전하게 움직일 수 있도록 하는 전문화된 '빠른 실행부(Fast Executor)' 파트너인 셈이다. 따라서 "NanoMap 대(vs) SLAM"은 올바른 질문이 아니다. 올바른 질문은 "NanoMap과 SLAM을 어떻게 최적으로 결합하여 강인하고(robust) 민첩한(agile) 자율 시스템을 만들 것인가?"이다. 실제로 `NanoSLAM` 31이라는 이름의 다른 프로젝트가 등장한 것은 이 두 기능을 통합하려는 연구계의 자연스러운 흐름을 보여주지만, 이는 MIT의 NanoMap과는 별개의 접근법이라는 점을 명확히 해야 한다.


NanoMap은 혁신적인 개념과 실용적인 성능을 입증했지만, 동시에 명확한 한계점을 가지고 있으며 이는 향후 연구를 위한 중요한 방향을 제시한다.


- **센서 노이즈 모델의 부재:** 원본 NanoMap의 가장 명시적인 한계 중 하나는 센서 자체의 측정 노이즈를 모델링 범위에서 의도적으로 제외했다는 점이다.1 실제 깊이 카메라나 LiDAR는 측정 거리, 표면의 재질, 입사각 등에 따라 복잡한 노이즈 특성을 보인다.16 이러한 센서 노이즈를 무시하는 것은 모델을 단순화하여 계산 효율을 높이는 데는 기여했지만, 실제 센서 데이터의 통계적 특성을 정확하게 반영하지 못함으로써 쿼리 결과의 신뢰도를 저하시킬 수 있는 잠재적 약점을 안고 있다.36
- **회전 불확실성 미반영:** 2장에서 논의했듯이, 모델은 병진 불확실성에만 집중하고 회전 불확실성은 무시한다.2 이는 단기적인 고속 기동에서는 합리적인 근사일 수 있으나, 로봇이 장시간, 장거리를 항법하는 시나리오에서는 요(yaw) 각도의 드리프트가 누적되어 심각한 오차를 유발할 수 있다. 이러한 상황에서 회전 불확실성을 고려하지 않으면, 과거의 관측 데이터를 현재 좌표계로 변환할 때 큰 오류가 발생하여 지도 전체의 정확도를 저하시킬 수 있다.24
- **지역적 매핑의 한계:** NanoMap은 본질적으로 고정된 시간 윈도우 내의 데이터만을 유지하는 지역(local) 매퍼이다. 시간 윈도우를 벗어나는 오래된 데이터는 시스템에서 폐기되므로, 대규모 환경에 대한 장기적인 전역 지도(long-term global map)를 구축하는 데는 근본적으로 적합하지 않다.1 이는 실시간 반응성을 극대화하기 위한 설계적 선택이지만, 로봇이 이전에 방문했던 장소를 기억하고 재사용하는 등 전역적 맥락 정보가 중요한 애플리케이션에는 명백한 한계로 작용한다.39
- **쿼리 성능 트레이드오프:** NanoMap은 데이터 구조의 구축 시간이 매우 빠르다는 장점이 있지만, 단일 쿼리를 처리하는 시간은 융합 기반 지도(예: OctoMap)에 비해 상대적으로 더 길 수 있다. 융합된 지도는 일단 구축되고 나면 특정 복셀에 대한 접근이 매우 빠른 반면, NanoMap은 쿼리가 들어올 때마다 시간 역순 탐색과 좌표 변환을 수행해야 하기 때문이다. 따라서, 경로 계획 알고리즘이 단위 시간당 매우 많은 양의 쿼리(실험적으로 약 10,000개 이상)를 생성하는 시나리오에서는 오히려 융합 기반 방식이 전체적으로 더 효율적일 수 있다는 성능 트레이드오프가 존재한다.15


위에서 언급된 한계점들은 NanoMap 프레임워크를 더욱 발전시키기 위한 구체적인 연구 방향을 제시한다.

- **정교한 센서 노이즈 모델 통합:** 딥러닝을 이용하여 실제 센서 데이터로부터 직접 노이즈 분포를 학습하는 데이터 주도적 노이즈 모델 16이나, 센서의 물리적 원리를 기반으로 한 정교한 통계적 노이즈 모델 17을 NanoMap의 불확실성 전파 과정에 통합하는 연구가 필요하다. 이는 쿼리 결과의 신뢰도를 한 단계 높여 더욱 안전한 의사결정을 가능하게 할 것이다.
- **회전 불확실성 모델링:** Lie 그룹(Lie Group)과 같은 현대적인 수학적 도구를 활용하여 3차원 공간에서의 회전과 병진을 통합적으로 다루는 불확실성 모델(예: $SE(3)$ 상에서의 불확실성)을 도입할 수 있다. 이를 통해 회전 불확실성을 명시적으로 모델에 포함시켜 장거리 항법 및 장시간 작동 시의 강인성을 크게 향상시킬 수 있다.
- **SLAM과의 긴밀한 결합(Tightly-coupled Integration):** 현재의 느슨한 '생산자-소비자' 관계를 넘어, NanoMap의 지역적 장애물 정보가 SLAM의 자세 추정 정확도를 높이는 데 사용되고, SLAM의 전역적 일관성 정보가 NanoMap의 지역 지도를 보정하는 데 사용되는 양방향 정보 교환 방식의 긴밀한 결합 프레임워크 개발이 유망하다.28 이는 지역적 민첩성과 전역적 일관성을 동시에 달성하는 차세대 자율 항법 시스템의 핵심이 될 수 있다.
- **동적 객체 추적 기능 추가:** 현재의 암묵적인 동적 환경 대응 능력을 넘어, 환경 내에서 움직이는 객체들을 명시적으로 분리하고 그들의 속도와 미래 궤적을 추정하는 모듈을 통합할 수 있다.25 이를 통해 단순한 반응적 회피(reactive avoidance)를 넘어, 동적 장애물의 미래 움직임을 예측하고 선제적으로 안전한 경로를 계획하는 능동적 계획(proactive planning)으로 발전시킬 수 있다.
- **GPU 가속의 발전:** Violet Walker 등의 연구 13는 이미 NanoMap 개념의 중요한 발전 방향, 즉 '고속 처리'를 제시했다. 향후 중요한 연구 과제는 이러한 GPU 가속 기술을 단순히 점유 격자 생성에만 사용하는 것을 넘어, 원본 NanoMap의 핵심인 불확실성 전파 및 최소 불확실성 뷰 탐색 알고리즘 자체에 적용하는 것이다. 이를 통해 '빠른 불확실성 인식 매핑'을 구현함으로써, NanoMap의 두 갈래 발전 방향을 하나로 통합할 수 있을 것이다.19

이러한 연구 방향들을 관통하는 핵심은, NanoMap의 가장 큰 기여가 특정 알고리즘이나 구현체를 넘어선다는 점에 있다. 그것은 로봇 인식 문제에 대한 접근 방식을 '절대적으로 정확한 단일 세계 모델(a single, perfect world model)'을 구축하려는 전통적인 시도에서, '불확실성을 명시적으로 인정하고 관리하며, 필요한 정보를 적시에, 적절한 신뢰도와 함께 질의하는(managing uncertainty and querying information just-in-time)' 방식으로 전환시킨 철학적 변화 그 자체이다.

과거의 로보틱스는 완벽하고 정적인 세계 지도를 만드는 것을 이상으로 삼았지만, 실제 세계는 본질적으로 동적이고 불확실하며, 우리의 센서는 노이즈가 많고 불완전하다. 이러한 이상과 현실의 괴리로 인해 많은 지도 기반 시스템들이 실제 환경에서 예측하지 못한 실패를 겪었다.4 NanoMap은 "우리는 모든 것을 완벽하게 알 수 없다"는 겸손한 사실을 시스템 설계의 전제로 삼는다.4 대신 "안전한 행동을 위해 지금 당장 알아야 할 최소한의 정보는 무엇이며, 그 정보의 신뢰도는 어느 정도인가?"라는 실용적인 질문을 던진다.

이러한 관점의 전환은 회전 불확실성이나 센서 노이즈를 생략하는 등의 '단순화'를 시스템의 '결함'이 아닌, 주어진 목표를 달성하기 위한 합리적인 '전략'으로 재해석하게 한다. 이는 시스템이 자신의 한계를 인지하고 그 안에서 최적의 행동을 찾도록 유도하는, 보다 성숙한 형태의 인공지능이라 할 수 있다. 따라서 NanoMap의 진정한 지적 유산 위에서 이루어질 향후 연구는 단순히 누락된 기능을 추가하는 것을 넘어, '특정 과업(task)을 완수하기 위해 어떤 종류의 불확실성을 어떻게 모델링하고 전파하는 것이 가장 효율적이고 강인한가'를 탐구하는 방향으로 나아가야 할 것이다.


본 보고서는 NanoMap 프레임워크에 대한 심층적인 기술적 고찰을 통해 그 핵심 철학, 아키텍처, 성능, 그리고 미래 발전 가능성을 종합적으로 분석했다. NanoMap은 고속 자율 항법이라는 로보틱스 분야의 오랜 난제를 해결하기 위해, 전통적인 전역 융합 기반 매핑의 한계를 극복하고자 제안된 혁신적인 패러다임이다. 그 핵심은 '불확실성 인식'과 '지연 탐색'이라는 두 기둥 위에 세워져 있다.

분석을 통해 "NanoMap"이라는 이름 아래 두 개의 중요한 기술적 흐름이 존재함을 확인했다. 첫째, MIT CSAIL에서 제안한 원본 NanoMap은 '불확실성 관리'라는 근본적인 문제에 집중했다. 이는 로봇이 자신의 위치를 완벽히 알 수 없다는 현실을 모델에 직접 통합하고, 데이터를 융합하지 않은 채 이력을 저장했다가 필요할 때 최소 불확실성 정보를 탐색하는 독창적인 접근법을 제시했다. 이 철학적 전환은 루프 클로저 정보의 즉각적인 반영, 동적 환경에 대한 암묵적 강인성 등 기존 방식으로는 달성하기 어려운 실용적 이점들을 가져왔다.

둘째, QUT 등에서 개발한 GPU 가속 NanoMap 라이브러리는 '고속 처리'라는 측면에서 NanoMap의 비전을 한 단계 발전시켰다. OpenVDB 자료구조와 GPU 병렬 컴퓨팅을 결합하여, 특히 밀집 센서 데이터로부터 점유 격자 지도를 생성하는 속도를 획기적으로 향상시켰다. 벤치마크 분석 결과, 이 라이브러리는 대표적인 매핑 기술인 OctoMap에 비해 수십 배 빠른 성능을 보였으며, 이는 자원이 제한된 임베디드 플랫폼에서도 실시간 고해상도 매핑을 가능하게 하는 실질적인 성과이다.

전문가적 관점에서 볼 때, NanoMap의 가장 중요한 기여는 '완벽한 전역 지도'라는 이상에서 벗어나, '충분히 좋은 지역 정보'를 실시간으로, 불확실성을 고려하여 활용하는 실용주의적 접근법의 성공 가능성을 입증했다는 데 있다. 이는 로봇이 자신의 인식과 추정 능력의 불완전함을 시스템 설계의 일부로 받아들이고, 그 한계 내에서 강인하게 동작하도록 만드는 중요한 철학적 진일보다.

향후 과제는 이 두 NanoMap의 장점을 결합하는 데 있다. 즉, GPU 가속 기술을 불확실성 전파 및 탐색 알고리즘에 적용하고, 센서 노이즈 및 회전 불확실성과 같은 아직 다루지 않은 불확실성의 영역들을 NanoMap의 철학 안으로 통합하여, 더욱 강인하고 지능적인 차세대 자율 시스템을 구현하는 것이다. NanoMap이 제시한 길은, 완벽을 추구하기보다 불확실성과 함께 살아가는 법을 배우는 것이 더 높은 수준의 자율성을 달성하는 길일 수 있음을 시사한다.


1. NanoMap: Fast, Uncertainty-Aware Proximity Queries with Lazy Search over Local 3D Data - Research, accessed August 4, 2025, https://groups.csail.mit.edu/robotics-center/public_papers/Florence17.pdf
2. NanoMap: Fast, Uncertainty-Aware Proximity Queries with Lazy Search Over Local 3D Data, accessed August 4, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/137099/1802.09076.pdf?sequence=2&isAllowed=y
3. Robust Vision-based Obstacle Avoidance for Micro Aerial Vehicles in Dynamic Environments - Autonomous Multi-Robots Lab, accessed August 4, 2025, https://autonomousrobots.nl/assets/files/publications/20-zhu-icra.pdf
4. Programming drones to fly in the face of uncertainty | MIT News, accessed August 4, 2025, https://news.mit.edu/2018/mit-csail-programming-drones-fly-face-uncertainty-0212
5. Programming drones to fly in the face of uncertainty - Robotics @ MIT, accessed August 4, 2025, https://robotics.mit.edu/programming-drones-fly-face-uncertainty/
6. NanoMap: Because SLAM is too slow for fast drones | Geo Week News, accessed August 4, 2025, https://www.geoweeknews.com/blogs/nanomap-slam-slow-fast-drones
7. [1802.09076] NanoMap: Fast, Uncertainty-Aware Proximity Queries with Lazy Search over Local 3D Data - arXiv, accessed August 4, 2025, https://arxiv.org/abs/1802.09076
8. NanoMap: Fast, Uncertainty-Aware Proximity Queries with Lazy Search Over Local 3D Data, accessed August 4, 2025, https://www.researchgate.net/publication/323411084_NanoMap_Fast_Uncertainty-Aware_Proximity_Queries_with_Lazy_Search_Over_Local_3D_Data
9. Embedded Safe Reactive Navigation for Multirotors Systems using Control Barrier Functions, accessed August 4, 2025, https://arxiv.org/html/2504.15850v1
10. Uncertainty-aware visually-attentive navigation using ... - NTNU Open, accessed August 4, 2025, https://ntnuopen.ntnu.no/ntnu-xmlui/bitstream/handle/11250/3111674/nguyen-et-al-2023-uncertainty-aware-visually-attentive-navigation-using-deep-neural-networks.pdf?sequence=4&isAllowed=y
11. [논문]Imitation Learning of Complex Behaviors for Multiple Drones, accessed August 4, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=NART127670755
12. ViWalkerDev/NanoMap - GitHub, accessed August 4, 2025, https://github.com/ViWalkerDev/NanoMap
13. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents - MDPI, accessed August 4, 2025, https://www.mdpi.com/2072-4292/14/21/5463
14. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents | QUT ePrints, accessed August 4, 2025, https://eprints.qut.edu.au/236544/
15. peteflorence/nanomap_ros: NanoMap: Fast, Uncertainty-Aware Proximity Queries with Lazy Search of Local 3D Data - GitHub, accessed August 4, 2025, https://github.com/peteflorence/nanomap_ros
16. Non-parametric Sensor Noise Modeling and Synthesis - European Computer Vision Association, accessed August 4, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/03438.pdf
17. [2504.19009] An SE(3) Noise Model for Range-Azimuth-Elevation Sensors - arXiv, accessed August 4, 2025, https://arxiv.org/abs/2504.19009
18. 'NanoMap' Programs Drone Navigation in Uncertain Environments - Tech Briefs, accessed August 4, 2025, https://www.techbriefs.com/component/content/article/31895-nanomap-to-program-drone-navigat
19. Remote Sensing | Free Full-Text | NanoMap: A GPU-Accelerated, accessed August 4, 2025, https://www.mdpi.com/2072-4292/14/21/5463/review_report
20. OctoMap: An Efficient Probabilistic 3D Mapping Framework Based on Octrees - Washington, accessed August 4, 2025, https://courses.cs.washington.edu/courses/cse571/16au/slides/hornung13auro.pdf
21. Octomap comparison. / Issue #28 / ethz-asl/voxgraph - GitHub, accessed August 4, 2025, https://github.com/ethz-asl/voxgraph/issues/28
22. Benchmark outputs at 0.1 m grid resolution. (a) OctoMap. (b) Nanomap No... - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/figure/Benchmark-outputs-at-01-m-grid-resolution-a-OctoMap-b-Nanomap-No-Filter-c_fig6_364974244
23. NanoMap: A GPU-Accelerated OpenVDB-Based Mapping and Simulation Package for Robotic Agents - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/364974244_NanoMap_A_GPU-Accelerated_OpenVDB-Based_Mapping_and_Simulation_Package_for_Robotic_Agents
24. Uncertainty helps autonomous drones fly through forests and urban spaces - IMechE, accessed August 4, 2025, https://www.imeche.org/news/news-article/uncertainty-helps-autonomous-drones-fly-through-forests-and-urban-spaces
25. Object-Oriented Grid Mapping in Dynamic Environments | Request PDF - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/384777985_Object-Oriented_Grid_Mapping_in_Dynamic_Environments
26. Safe and Robust Map Updating for Long-Term Operations in Dynamic Environments - PMC, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10346461/
27. Robust Vision-based Obstacle Avoidance for Micro Aerial Vehicles in Dynamic Environments - TU Delft Research Portal, accessed August 4, 2025, https://research.tudelft.nl/files/101618250/Robust_Vision_based_Obstacle_Avoidance_for_Micro_Aerial_Vehicles_in_Dynamic_Environments.pdf
28. Drones Navigate on the Fly, Without Consulting a Map - Smithsonian Magazine, accessed August 4, 2025, https://www.smithsonianmag.com/air-space-magazine/drones-navigate-fly-without-consulting-map-180968136/
29. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed August 4, 2025, https://www.mathworks.com/discovery/slam.html
30. ICRA-2018 - GitHub, accessed August 4, 2025, https://github.com/ICRA-2018
31. NanoSLAM: Enabling Fully Onboard SLAM for Tiny Robots - GitHub, accessed August 4, 2025, https://github.com/ETH-PBL/NanoSLAM
32. NanoSLAM: Enabling Fully Onboard SLAM for Tiny Robots - arXiv, accessed August 4, 2025, https://arxiv.org/html/2309.12008v2
33. NanoMap: Fast, Uncertainty-Aware Proximity Queries with Lazy, accessed August 4, 2025, https://www.researchgate.net/publication/327805191_NanoMap_Fast_Uncertainty-Aware_Proximity_Queries_with_Lazy_Search_Over_Local_3D_Data
34. Robust Cuboid Modeling from Noisy and Incomplete 3D Point, accessed August 4, 2025, https://www.mdpi.com/2072-4292/14/19/5035?type=check_update&version=1
35. Measurement Noise Model for Depth Camera-Based People Tracking - PMC, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8271657/
36. [2008.09370] Learning Camera-Aware Noise Models - arXiv, accessed August 4, 2025, https://arxiv.org/abs/2008.09370
37. Nanoradian-scale precision in light rotation measurement via indefinite quantum dynamics, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11758439/
38. Rotational states of a nanomagnet | Phys. Rev. B - Physical Review Link Manager, accessed August 4, 2025, https://link.aps.org/doi/10.1103/PhysRevB.81.214423
39. [1808.02658] Map Management for Efficient Long-Term Visual Localization in Outdoor Environments - arXiv, accessed August 4, 2025, https://arxiv.org/abs/1808.02658
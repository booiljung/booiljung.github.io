# 라이다 SLAM에서 동적 객체 제거를 위한 시간적 윈도잉(Temporal Windowing) 기술 분석
[SLAM에서 동적 객체 제거 문제](./index.md)



대부분의 고전적이고 영향력 있는 SLAM(Simultaneous Localization and Mapping) 알고리즘은 근본적으로 '정적 세계 가정'에 기반을 두고 있다.1 이는 알고리즘이 동작하는 환경 내의 모든 객체가 움직이지 않고 고정되어 있다는 전제를 의미한다. ICP(Iterative Closest Point)나 NDT(Normal Distributions Transform)와 같은 핵심적인 정합(registration) 알고리즘들은 이 가정이 충족될 때 최적의 성능을 발휘한다. 그러나 도심 주행, 보행자가 많은 실내 공간, 물류 창고 등 실제 응용 환경에서는 이 가정이 빈번하게 위배된다.3

차량, 사람, 로봇과 같은 동적 객체들은 시간에 따라 위치를 바꾸며, 이는 SLAM 시스템에 시계열적으로 일관되지 않은 측정값을 입력하게 만든다. SLAM의 핵심은 현재 센서 데이터와 이전에 구축된 지도 사이의 데이터 연관성(data association)을 정확히 찾는 것인데, 동적 객체는 이 과정에 심각한 오류를 유발한다.6 예를 들어, 이전에 비어 있던 공간에 갑자기 나타난 차량은 지도상의 정적 구조물과 잘못 연관될 수 있으며, 이는 위치 추정 정확도를 급격히 떨어뜨리거나 최악의 경우 SLAM 시스템 전체의 실패로 이어질 수 있다.7


동적 환경에서 맵핑을 수행할 때 발생하는 가장 대표적인 문제점은 '고스팅(ghosting)' 또는 '고스트 트레일(ghost trail)' 현상이다.3 이는 움직이는 객체의 포인트 클라우드가 시간의 흐름에 따라 지도에 누적되면서, 실제로는 존재하지 않는 잔상이나 유령 같은 흔적을 남기는 것을 말한다.12

이러한 고스팅 현상은 단순히 지도를 시각적으로 지저분하게 만드는 것을 넘어, SLAM 시스템의 성능에 직접적인 악영향을 미친다. 첫째, 고스트 포인트들은 도로 경계석, 차선, 건물 벽면과 같은 중요한 정적 특징들을 가리거나 오염시켜, 환경의 구조적 정보를 왜곡한다.9

둘째, 이는 SLAM의 두 가지 핵심 기능인 측위(localization)와 후속 작업(downstream tasks) 모두를 저해한다.

- **측위 정확도에 미치는 영향:** 새로운 라이다 스캔을 고스팅으로 오염된 지도에 정합하려고 할 때, ICP와 같은 알고리즘은 실제 정적 구조물이 아닌 유령 포인트에 잘못 정합될 수 있다. 이는 필연적으로 잘못된 자세(pose) 추정으로 이어지며, 절대 궤적 오차(Absolute Trajectory Error, ATE)와 상대 자세 오차(Relative Pose Error, RPE)를 증가시킨다.12
- **후속 작업에 미치는 영향:** 자율 주행 시스템의 경로 계획 알고리즘은 지도상의 고스트 포인트를 실제 정적 장애물로 오인할 수 있다. 이로 인해 최적의 경로를 생성하지 못하거나, 심지어 실행 불가능한 경로를 계획하게 되어 자율 주행의 안전성에 심각한 위협이 될 수 있다.12

이러한 문제의 근원은 SLAM, 특히 그래프 기반 SLAM의 최적화 과정에서 찾을 수 있다. SLAM 시스템은 로봇의 자세를 노드(node)로, 센서 측정값으로부터 계산된 자세 간의 기하학적 제약을 엣지(edge)로 하는 팩터 그래프(factor graph)를 구축하고 최적화한다. 동적 객체는 현재 스캔과 지도 간의 잘못된 대응점(correspondence)을 유발하고 9, 이는 그래프에 잘못된 제약, 즉 부정확한 엣지를 추가하는 것과 같다. 백엔드 최적화기가 이 오염된 그래프를 풀려고 할 때, 잘못된 제약들은 전체 자세 추정치를 실제 참값(ground truth)으로부터 멀어지게 하여 드리프트(drift)를 누적시킨다. 따라서 동적 객체 제거는 단순히 지도를 깨끗하게 만드는 시각적 후처리 작업이 아니라, SLAM 문제 자체의 수학적 일관성을 유지하기 위한 필수적인 과정이다.


사용자가 제안한 '최신 시간 정보만 모아서 융합하는 방법'을 더 넓은 맥락에서 이해하기 위해, 기존의 동적 객체 제거 전략들을 몇 가지 주요 범주로 분류할 수 있다.3

- **기하학 기반 방법(Geometric-Based Methods):** 이 방법들은 시간에 따른 기하학적 불일치성을 이용한다. 대표적으로 레이캐스팅(ray-casting) 또는 가시성(visibility) 기반 접근법이 있다. 이는 OctoMap과 같은 점유 격자 지도(occupancy grid map)를 사용하여, 이전에 '점유됨(occupied)'으로 관측된 공간이 나중에 '비어있음(free)'으로 관측될 경우, 그 공간에 있던 객체를 동적 객체로 판단하고 제거하는 방식이다.10 또 다른 방식으로는 여러 프레임에 걸쳐 포인트 클러스터의 움직임, 즉 씬 플로우(scene flow)를 추정하여 동적인 부분을 식별하는 방법이 있다.7
- **학습 기반 방법(Learning-Based Methods):** 딥러닝 기술, 특히 의미론적 분할(semantic segmentation)이나 인스턴스 분할(instance segmentation) 네트워크를 활용하는 접근법이다. RangeNet++나 Mask R-CNN과 같은 모델을 사용하여 '자동차', '사람' 등 사전에 동적일 가능성이 높다고 정의된 클래스에 속하는 포인트들을 식별하고, 이들을 지도 생성 과정에서 배제한다.21
- **하이브리드 방법(Hybrid Methods):** 기하학적 단서와 의미론적 정보를 모두 활용하여 더 강건한 동적 객체 탐지를 목표로 하는 방법이다.10 예를 들어, 의미론적으로 '자동차'로 분류된 객체가 기하학적으로도 움직임이 감지될 때만 최종적으로 동적 객체로 판단하는 식이다.

이러한 분류 체계에서 사용자가 제안한 방법은 근본적으로 **시간적(temporal), 기하학 기반 필터링 접근법**에 해당한다. 이 방법은 포인트의 시간적 일관성(temporal consistency) 여부를 지도에 포함할지 결정하는 핵심 기준으로 삼는다.


이 섹션에서는 사용자의 질문에 직접적으로 답하기 위해, 시간적 융합 방법의 핵심 원리를 분석하고, 이 방법의 성능을 좌우하는 가장 중요한 파라미터인 '윈도우 크기'가 미치는 영향을 심층적으로 탐구한다.


이 방법의 핵심 아이디어는 전체 지도를 한 번에 구축하고 유지하는 대신, 제한된 수의 최신 라이다 스캔 프레임으로 구성된 '로컬 맵(local map)' 또는 '서브맵(submap)'을 지속적으로 갱신하며 사용하는 것이다. 이는 개념적으로 스캔 데이터에 대한 슬라이딩 윈도우(sliding window) 또는 순환 버퍼(circular buffer)로 모델링할 수 있다.

시간 t에서 획득한 포인트 클라우드를 Pt라고 하자. 크기가 N인 슬라이딩 윈도우는 시간 t에서의 로컬 맵 $M_{local}$을 시간 t−N+1부터 t까지의 스캔들을 융합하여 정의한다.

수학적으로 로컬 맵 $M_{local}^{(t)}$는 지난 N개의 스캔들을 각각의 추정된 자세(pose) Ti를 이용해 월드 좌표계(world frame)로 변환한 후 모두 합집합(union)한 것으로 표현할 수 있다.28
$$
M_{local}^{(t)} = \bigcup_{i=t-N+1}^{t} T_i P_i
$$
이렇게 융합된 포인트 클라우드 $M_{local}^{(t)}$는 일반적으로 다음 스캔 $P_{t+1}$을 정합하기 위한 타겟으로 사용되기 전에 복셀 그리드(voxel grid) 등을 이용해 관리 가능한 밀도로 다운샘플링된다. 이 과정은 많은 최신 라이다 오도메트리(LiDAR odometry) 시스템의 기본 원리이다.


윈도우 크기 $N$의 선택은 이 방법의 성능을 결정하는 가장 중요한 설계 변수이며, 여러 성능 지표 간의 근본적인 트레이드오프 관계를 내포하고 있다.30

- **동적 객체 제거 성능 (응답성):**

  - **짧은 윈도우 (작은 $N$):** 동적 객체를 매우 신속하게 제거하는 데 효과적이다. 어떤 객체가 움직이면, 그 객체를 구성하던 포인트들은 $N$ 프레임 이후 윈도우에서 완전히 사라진다. 이는 고스팅 현상이 거의 없는 깨끗한 로컬 맵을 만들어내며, 매우 동적인 환경에서의 실시간 주행에 이상적이다.33
  - **긴 윈도우 (큰 $N$):** 동적 객체 제거 속도가 느리다. 고스트 포인트들이 지도에 더 오랜 시간 동안 남아있게 되어, 장시간 동안 측위 정확도를 저해할 수 있다.

- **정적 지도 품질 (밀도 및 완전성):**

  - **짧은 윈도우 (작은 $N$):** 희소한(sparse) 로컬 맵을 생성한다. 이는 긴 복도나 터널과 같이 특징이 부족한 환경에서 문제가 될 수 있다. 작은 윈도우 내에 로봇의 자세를 충분히 구속할 만한 구조적 정보가 부족하여 오도메트리 드리프트가 발생할 수 있다.34 또한, 환경 전체에 대한 완전하고 밀도 높은 지도를 구축하는 데에는 부적합하다.
  - **긴 윈도우 (큰 $N$):** 정적 환경에 대한 더 많은 관측치를 누적함으로써 더 밀도 높고(dense) 완전하며 안정적인 로컬 맵을 생성한다. 이는 특히 희소한 환경에서 스캔 정합을 위한 더 강건한 제약을 제공한다.36

- 일시적으로 정지한 객체 처리:

  이것은 이 방법의 주요 실패 사례 중 하나이다. 일반적으로 동적인 객체(예: 차량)가 잠시 멈추는 경우를 생각해보자.

  - **짧은 윈도우:** 만약 객체가 윈도우 기간 $N$보다 더 오래 정지해 있다면, 이 객체는 로컬 맵에 완전히 통합되어 정적 특징으로 간주된다. 이후 객체가 다시 움직이기 시작하면, 지도에 갑자기 큰 '구멍'을 남기고 새로운 고스트 트레일을 생성하여, 측위 시스템에 급격한 오차(jump)나 실패를 유발할 수 있다.
  - **긴 윈도우:** 짧은 정지 상태에 대해서는 더 강건하다. 객체의 존재가 더 긴 시간에 걸쳐 평균화되기 때문이다. 하지만 장기간 주차된 차량과 같은 문제에는 여전히 취약하며, 이러한 문제는 의미론적 정보나 장기 SLAM(long-term SLAM) 접근법으로 더 잘 해결될 수 있다.10

- **계산 자원 소모:**

  - 윈도우 크기와 계산 비용은 정비례 관계에 있다. 윈도우 크기 $N$이 클수록 더 많은 포인트 클라우드를 저장하기 위한 메모리가 필요하며, 매 프레임마다 이를 융합하고 다운샘플링하는 데 더 많은 처리 능력이 요구된다.39 실시간 성능을 보장하기 위해서는 

    $N$에 대한 현실적인 상한선이 존재한다.

이 방법은 단순히 동적 포인트를 제거하는 이진 필터(binary filter)라기보다는, 시간적 일관성에 기반한 확률적 필터로 이해하는 것이 더 정확하다. 어떤 포인트의 '정적임(static-ness)'은 그 포인트가 윈도우 내에서 얼마나 지속적으로 관측되었는지에 비례한다. 진정한 정적 포인트는 윈도우 내에서 꾸준히 높은 밀도로 존재하지만, 동적 포인트는 일시적으로 나타났다가 사라진다. 이러한 관점은 시간적 윈도우 방법이 점유 격자 지도나 밀도 기반 클러스터링과 같은 더 정교한 방법들의 본질과 맞닿아 있음을 보여준다. 즉, 시간적 윈도우는 이들 방법이 명시적으로 모델링하는 '관측 확률'을 암시적이고 가벼운 방식으로 근사하는 것이다.


KISS-ICP 알고리즘은 "작고 단순하게 유지하라(Keep It Small and Simple)"는 철학을 바탕으로, 별도의 명시적인 동적 객체 제거 모듈 없이도 동적 환경에서 최첨단 성능을 달성하는 대표적인 사례이다.40 이 알고리즘의 강건성은 짧은 시간의 로컬 맵을 사용하는 전략과 다른 핵심 요소들의 시너지 효과에서 비롯된다.

- **로컬 맵 전략:** KISS-ICP는 최대 범위 내의 최신 스캔 포인트들을 저장하는 로컬 복셀 맵을 사용한다. 이는 사실상 공간적으로 제한된 짧은 시간의 시간적 윈도우 역할을 하여, 동적 객체 포인트가 지도에 영구적으로 남는 것을 자연스럽게 방지한다.42
- **시너지를 통한 강건성 확보:** KISS-ICP의 성공은 단순히 짧은 윈도우 맵 때문만이 아니다. 다음과 같은 요소들이 유기적으로 결합하여 강건성을 극대화한다.
  1. **적응형 임계값(Adaptive Thresholding):** ICP의 대응점 탐색 시 고정된 거리 임계값을 사용하는 대신, 시스템의 운동학(kinematics)을 기반으로 임계값을 동적으로 조절한다. 이를 통해 고속 이동 중에는 더 넓은 탐색 범위를 허용하고 저속 이동 중에는 더 엄격한 기준을 적용하여, 동적 객체로부터 발생하는 비정상적인 대응점을 효과적으로 거부한다.42
  2. **강건한 비용 함수(Robust Cost Function):** ICP 최적화 과정에서 이상치(outlier)에 매우 민감한 표준 최소제곱($L_2$) 메트릭 대신, Geman-McClure와 같은 강건한 커널(robust kernel)을 사용한다.46 이 함수는 큰 오차를 가진 포인트(주로 동적 객체 위의 포인트)의 영향력을 크게 줄여, 이들이 전체 자세 추정 결과를 오염시키는 것을 방지한다.44

결론적으로, 시간적 윈도우 방법은 그 자체로도 효과적이지만, KISS-ICP의 사례에서 보듯이 적응형 대응점 탐색 및 강건한 최적화 기법과 결합될 때 그 성능이 극대화된다. 이는 동적 객체를 '제거'하는 것뿐만 아니라, 남아있는 동적 포인트의 '영향을 최소화'하는 이중 방어 체계를 구축하는 것과 같다.


동적 객체 제거 방법의 성능을 객관적으로 평가하기 위해서는 표준화된 지표가 필요하다. 이 섹션에서는 SLAM 성능 평가에 널리 사용되는 궤적 정확도 지표와 지도 품질 지표를 정의하고, 앞서 논의한 트레이드오프를 종합적으로 비교 분석하기 위한 표를 제시한다.


궁극적으로 SLAM 오도메트리의 목표는 정확한 궤적을 추정하는 것이므로, 동적 객체 제거의 효과는 궤적 정확도의 향상으로 직접 측정된다.

- **절대 궤적 오차 (Absolute Trajectory Error, ATE):** 추정된 궤적과 실제 참값 궤적 간의 전역적인 일관성을 평가하는 지표이다. 전체 경로에 걸쳐 추정된 자세와 실제 자세 간의 차이를 직접 계산한다. ATE가 낮을수록 전반적인 정확도가 높고 누적 오차가 적음을 의미하며, 이는 동적 객체 제거가 성공적으로 이루어졌을 때 나타나는 핵심적인 이점이다.13
- **상대 자세 오차 (Relative Pose Error, RPE):** 고정된 시간 또는 거리 간격 동안의 지역적인 일관성, 즉 드리프트를 측정한다. 이는 프레임 간의 회전 및 병진 오차 변화에 민감하다. RPE가 개선되었다는 것은 프레임 대 프레임(frame-to-frame) 정합이 더 안정적으로 이루어졌음을 의미한다.13


궤적 정확도 외에도, 생성된 정적 지도의 품질은 후속 작업(예: 경로 계획)에 매우 중요하다. 벤치마킹 프레임워크들은 지도 자체의 품질을 정량화하기 위한 지표들을 제안했다.12

- **동적 정확도 (Dynamic Accuracy, DA) / 거부율 (Rejection Rate, RR):** 실제 동적 포인트 중에서 올바르게 동적으로 식별되어 제거된 포인트의 비율을 측정한다. 높은 DA/RR은 해당 방법이 지도를 깨끗하게 만드는 데 효과적임을 나타낸다.50
  $$
  DA = \frac{\text{True Negatives}}{\text{True Negatives} + \text{False Positives}}
  $$
  여기서 'Negative'는 동적 포인트를, 'Positive'는 정적 포인트를 의미한다. True Negative는 올바르게 제거된 동적 포인트, False Positive는 잘못 제거된 정적 포인트를 나타낸다.

- **정적 정확도 (Static Accuracy, SA) / 보존율 (Preservation Rate, PR):** 실제 정적 포인트 중에서 올바르게 정적으로 유지된 포인트의 비율을 측정한다. 높은 SA/PR은 매우 중요하다. 지나치게 공격적인 필터링은 정적 환경의 유용한 부분까지 제거하여 지도의 품질을 저하시킬 수 있기 때문이다.12
  $$
  SA = \frac{\text{True Positives}}{\text{True Positives} + \text{False Negatives}}
  $$
  여기서 True Positive는 올바르게 유지된 정적 포인트, False Negative는 제거되지 않고 남은 동적 포인트를 의미한다.

이러한 지표들 사이에는 본질적인 상충 관계가 존재한다. 예를 들어, 매우 공격적인 필터(예: 아주 짧은 윈도우)는 높은 DA를 달성할 수 있지만, 일시적으로 가려지거나 희소하게 관측된 정적 포인트까지 제거하여 낮은 SA를 초래할 수 있다. 반대로, 보수적인 필터(예: 긴 윈도우)는 SA를 우선시하여 정적 포인트를 최대한 보존하지만, 동적 객체의 잔상이 오래 남아 DA가 낮아진다. 따라서 알고리즘의 성능을 온전히 이해하기 위해서는 궤적 오차뿐만 아니라, SA와 DA 지표를 함께 분석하여 균형점을 평가하는 것이 필수적이다.


아래 표는 시간적 윈도우 크기에 따른 각 성능 지표의 변화와 트레이드오프를 종합적으로 요약한 것이다. 이는 특정 응용 분야의 요구사항에 맞춰 최적의 윈도우 전략을 선택하는 데 유용한 가이드라인을 제공한다.

**표 1: 시간적 윈도우 전략이 SLAM 성능에 미치는 영향 비교 분석**

| **전략 / 윈도우 크기**        | **동적 객체 제거 효율성**                                    | **정적 지도 밀도 및 완전성**                                 | **일시적 정지 객체에 대한 강건성**                           | **계산 비용 (ms/frame)**                                     | **궤적 정확도(ATE)에 미치는 영향**                           |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **짧은 윈도우 (예: N=2-5)**   | 높은 응답성, 낮은 고스팅 지연. 높은 DA/RR.                   | 낮은 밀도, 희소한 지도. 잠재적으로 불완전함. 특징 없는 영역에서 낮은 SA/PR. | 낮음. 객체가 잠시 멈추면 빠르게 '정적'으로 간주되어, 다시 움직일 때 문제를 일으킴. | 낮음. 처리할 포인트 수가 적음.                               | 매우 동적인 장면에서 향상됨. 특징이 부족한 장면에서 저하됨.  |
| **중간 윈도우 (예: N=10-20)** | 보통의 응답성. 좋은 균형.                                    | 보통의 밀도. 일반적으로 강건한 측위에 충분함.                | 보통. 객체를 완전히 통합하지 않고 짧은 정지를 처리할 수 있음. | 보통. 실시간 시스템을 위한 일반적인 트레이드오프.            | 다양한 환경에서 전반적으로 가장 균형 잡힌 정확도를 제공함.   |
| **긴 윈도우 (예: N > 50)**    | 낮은 응답성. 고스트가 장시간 지속됨. 낮은 DA/RR.             | 높은 밀도, 매우 완전한 로컬 맵. 높은 SA/PR.                  | 높음. 단기 및 중기 정지에 강건함.                            | 높음. 상당한 하드웨어 없이는 실시간 응용에 부적합할 수 있음. | 지도 오염으로 인해 매우 동적인 장면에서 저하됨. 정적이고 특징이 부족한 장면에서 향상됨. |
| **적응형 윈도우**             | 가변적. 동적 영역에서 축소하여 효율을 높이는 것을 목표로 함. | 가변적. 정적 영역에서 확장하여 밀도를 높이는 것을 목표로 함. | 잠재적으로 높지만, 적응 휴리스틱에 따라 달라짐.              | 가변적이며 구현이 더 복잡함.                                 | 론적으로 최적이지만, 성능은 적응 로직에 크게 의존함.         |


이 섹션에서는 시간적 윈도우 방법을 실제로 구현하기 위한 실용적인 지침을 제공한다. 단일 라이브러리 함수로 이 기능이 제공되지 않음을 인지하고, 필요한 구성 요소와 아키텍처 패턴을 상세히 설명한다.


PCL(Point Cloud Library)이나 Open3D와 같은 주요 3D 데이터 처리 라이브러리를 살펴보면, 이들이 데이터 구조, 필터, 정합 알고리즘 등 근본적인 도구들을 제공하지만, '시간적 동적 객체 제거'와 같은 고수준의 응용별 기능을 직접 제공하지는 않는다는 것을 알 수 있다.20 따라서 구현은 이러한 기본 빌딩 블록들을 ROS(Robot Operating System)와 같은 더 큰 프레임워크 내에서 조합하는 방식으로 이루어진다.54


- **핵심 데이터 구조:** 스캔들의 슬라이딩 윈도우는 `boost::circular_buffer<pcl::PointCloud<PointT>::Ptr>`를 사용하여 효율적으로 관리할 수 있다. 순환 버퍼는 고정된 크기를 가지며 새로운 요소가 추가될 때 가장 오래된 요소를 자동으로 폐기하므로, FIFO(First-In, First-Out) 방식의 시간적 윈도우를 모델링하는 데 이상적이다.55 이 데이터 구조의 선택은 사소한 코딩 문제가 아니라, 실시간 성능을 좌우하는 중요한 아키텍처 결정이다. 단순 `std::vector`를 사용하면 매 프레임마다 $O(N)$의 복잡도를 갖는 원소 이동이 발생하지만, 순환 버퍼는 $O(1)$ 복잡도로 이를 처리하여 실시간성을 보장한다.

- **아키텍처 단계:**

  1. **노드 초기화:** 입력 라이다 토픽(`/points_raw` 등)을 구독하는 ROS 노드를 생성한다.

  2. **순환 버퍼 유지:** 지난 $N$개의 포인트 클라우드 스마트 포인터를 저장할 `boost::circular_buffer`를 멤버 변수로 유지한다.

  3. 융합 루프 (콜백 함수 내): 새로운 프레임이 들어올 때마다 다음을 수행한다.

     a. 새로운 클라우드를 버퍼에 푸시한다.

     b. 로컬 맵을 위한 새로운 빈 pcl::PointCloud를 생성한다.

     c. 버퍼 내의 모든 클라우드를 순회하며, 각 클라우드에 해당하는 변환(SLAM 시스템의 오도메트리 추정치로부터 얻음)을 적용하고 로컬 맵 클라우드에 누적한다.

  4. **다운샘플링:** 융합된 로컬 맵에 `pcl::VoxelGrid` 필터를 적용한다. 이는 포인트 수를 제어하여 실시간 성능을 유지하는 데 매우 중요하다.56 복셀 크기는 핵심 튜닝 파라미터이다.

  5. **발행:** 다운샘플링된 로컬 맵을 새로운 토픽(예: `/local_map`)으로 발행한다. SLAM 시스템의 정합 모듈은 이 토픽을 지도 참조용으로 사용하게 된다.

  6. **암시적 제거:** 동적 객체의 '제거'는 명시적으로 이루어지지 않는다. 동적 객체의 포인트들은 버퍼에 오래 머물지 않으므로, 자연스럽게 융합된 지도에서 사라지게 된다.


- **핵심 데이터 구조:** 파이썬에서는 `collections.deque(maxlen=N)`가 효율적인 고정 크기 큐를 제공하여 순환 버퍼와 동일한 역할을 한다.

- **아키텍처 단계 (PCL과 유사):**

  1. `rospy` 또는 `rclpy`를 사용하여 ROS 구독자를 생성한다.

  2. 지난 $N$개의 Open3D `PointCloud` 객체를 저장할 `deque`를 유지한다.

  3. 콜백 함수 내에서:

     a. 새로운 클라우드를 deque에 추가한다.

     b. deque에 있는 모든 클라우드를 순회하며, 각 클라우드를 해당 자세로 변환하고 + 연산자를 사용하여 하나의 로컬 맵 객체로 합친다.

  4. **다운샘플링:** `local_map.voxel_down_sample(voxel_size=...)`를 호출한다. Open3D의 복셀 다운샘플링은 매우 효율적이다.

  5. 결과 포인트 클라우드를 발행한다.

  - **참고:** Open3D는 `remove_statistical_outlier`나 `remove_radius_outlier`와 같은 명시적인 이상치 제거 함수도 제공한다.52 이는 융합된 지도에 대해 선택적인 후처리 단계로 적용하여, 고스팅의 특징인 희소한 포인트들을 추가로 제거하는 데 사용될 수 있다.


시간적 윈도우 방법과의 비교를 위해, 시간 정보를 활용하는 다른 오픈소스 시스템들을 간략히 소개한다.

- **Removert:** 실시간 오도메트리가 아닌 *오프라인* 후처리 방법이다. 최신 스캔들의 윈도우가 아닌, 사전에 구축된 전역 지도와 각 스캔을 비교한다. 다중 해상도 거리 이미지를 사용하여 정적 포인트를 잘못 제거하는 것을 방지하는 '제거 후 복원(remove-then-revert)' 전략을 사용한다. 더 정확하지만 실시간용은 아니다.59
- **Peopleremover / ERASOR:** 이들은 주로 가시성 기반 방법으로, 시간에 따라 구축된 점유 격자 지도에 레이캐스팅을 수행하여 불일치성을 찾아낸다. 이는 단순 융합 방식보다 점유 상태를 더 명시적으로 모델링하는 방법이다.2
- **GitHub 저장소:** `davutcanakbas/dynamic_object_removal`과 같은 프로젝트는 다른 접근법을 보여준다.54 이는 CenterPoint와 같은 실시간 객체 탐지기를 사용하여 동적 객체의 경계 상자(bounding box)를 얻고, 해당 영역의 포인트들을 직접 잘라내는 학습 기반 방식이다. 이는 시간적 필터링이 아닌 의미론적 필터링으로, 유용한 대조를 제공한다.


이 마지막 섹션에서는 기본 구현을 넘어, 더 정교한 변형과 미래 연구 방향을 탐색하여 전문가적이고 미래 지향적인 관점을 제시한다.


시간적 윈도우 방법은 일시적으로 정지한 객체에 취약하고, 의미론적 방법은 알려지지 않은 동적 객체에 취약하다. 하이브리드 접근법은 이러한 상호 보완적인 약점을 완화할 수 있다.

- **제안 아키텍처:**
  1. 기본적으로 시간적 슬라이딩 윈도우를 사용하여 로컬 맵을 유지한다.
  2. 병렬적으로 경량 3D 객체 탐지기를 실행한다.14
  3. 스캔을 융합할 때 가중치 기법을 적용한다. '자동차'와 같이 잠재적으로 동적인 클래스에 속하는 포인트들은 더 낮은 가중치를 부여받거나, 윈도우 내의 여러 프레임에서 일관되게 관측될 때만 지도에 추가된다. 이를 통해 잠시 멈춘 차량이 건물 벽과 동일한 신뢰도로 처리되는 것을 방지할 수 있다.7


고정된 윈도우 크기 $N$은 최적이 아니다. 이상적인 크기는 환경과 로봇의 상태에 따라 달라지기 때문이다. SLAM 연구 분야에서는 적응형 슬라이딩 윈도우에 대한 탐구가 이루어져 왔다.34

- **적응을 위한 휴리스틱:**
  - **속도 기반:** 저속에서는 $N$을 늘려 더 밀도 높은 지도를 구축하고, 고속에서는 $N$을 줄여 변화에 더 빠르게 반응하고 계산 부하를 줄인다.
  - **환경 기반:** 기하학적 분석을 통해 특징이 부족한 환경(예: 긴 복도)을 감지한다. 이러한 경우, 일시적으로 $N$을 늘려 더 많은 구조 정보를 수집하고 드리프트를 방지한다.34
  - **동적성 기반:** 연속된 스캔 간의 불일치 정도를 측정한다. 높은 동적성이 감지되면 윈도우를 축소하여 오래된 데이터를 신속하게 제거한다.


단순 슬라이딩 윈도우는 FIFO 방식의 '강한 망각(hard forgetting)' 메커니즘을 사용한다. 더 발전된 기법은 '부드러운 망각(soft forgetting)'을 제공하여 강건성을 높일 수 있다. 이는 평생 SLAM(lifelong SLAM) 지도 관리의 핵심 개념이다.38

- **지수적 감쇠(Exponential Decay):** 지난 $N$개 스캔의 단순 합집합 대신, 버퍼 내 각 스캔의 나이에 따라 지수적으로 감쇠하는 가중치를 할당한다. 로컬 맵은 가중치가 적용된 융합 결과가 된다.
  $$
  w_i = e^{-\lambda(t-i)} \quad \text{for } i \in [t-N+1, t]
  $$
  여기서 $w_i$는 스캔 $P_i$에 대한 가중치이고, $\lambda$는 감쇠율이다. 오래된 스캔의 포인트일수록 최종 복셀 그리드의 밀도에 더 적게 기여하게 된다.

- **확률적 점유(Probabilistic Occupancy):** 이는 OctoMap의 개념에 더 가까워지는 접근법이다.10 로컬 맵의 각 복셀은 단순히 포인트를 저장하는 것이 아니라 점유 확률을 저장한다. 새로운 스캔이 들어올 때마다 관측된 복셀에 대해 베이즈 업데이트(Bayesian update)를 수행한다. 이는 불확실성과 망각을 더 원칙적인 방식으로 다루게 해준다. 지속적으로 비어 있는 것으로 관측되는 복셀의 점유 확률은 시간이 지남에 따라 자연스럽게 감소한다.66

이러한 고급 개념들은 시간적 윈도우 방법이 상태 추정 분야의 더 넓은 이론적 틀 안에서 이해될 수 있음을 시사한다. 사용자가 제안한 방법은 사실상 상태 추정에서 사용되는 '고정 지연 스무더(fixed-lag smoother)'의 지도 생성 버전이다.39 고정 지연 스무더에서 논의되는 정보 손실, 최적 윈도우 크기 선택 등의 문제들은 시간적 윈도우 맵핑의 문제(지도 희소성, N$ 선택)와 직접적으로 대응된다. 이는 상태 추정 이론의 고급 기법들, 예를 들어 관측 가능성 제약 필터링(observability-constrained filtering) 39이나 정보 이론 기반 적응형 윈도잉 63 등이 이 맵핑 전략을 개선하는 데 직접 적용될 수 있음을 의미하며, 이는 향후 연구를 위한 풍부한 방향을 제시한다.


본 기술 분석 보고서는 라이다 포인트 클라우드에서 동적 객체를 제거하기 위해 최신 시간 정보만을 융합하는 시간적 슬라이딩 윈도우 방법에 대해 심층적으로 분석했다.

- **분석 결과 요약:** 시간적 슬라이딩 윈도우 방법은 계산적으로 효율적이며 실시간 동적 객체 제거에 효과적인 전략이다. 이 방법의 가장 큰 강점은 단순성과 응답성으로, 온라인 오도메트리 시스템에 매우 적합하다.
- **핵심 트레이드오프 재확인:** 이 방법의 성능은 근본적으로 윈도우 크기에 의해 결정되며, 이는 **동적 요소에 대한 응답성**(짧은 윈도우에 유리)과 **정적 지도의 밀도 및 완전성**(긴 윈도우에 유리) 사이의 중대한 트레이드오프를 야기한다.
- **최종 권장 사항:** 최적의 전략은 응용 분야에 따라 달라진다. 복잡한 도심 환경에서의 고속 자율 주행을 위해서는 KISS-ICP와 같이 짧은 윈도우와 강건한 ICP 변형을 결합하는 것이 우수하다. 반면, 건축 측량과 같이 느리고 상세한 맵핑 작업에는 더 긴 윈도우나 Removert와 같은 오프라인 후처리 방법이 더 적합할 것이다.
- **향후 전망:** 강건한 SLAM의 미래는 환경의 동적 맥락과 의미론적 이해를 바탕으로 시간적 메모리를 조절할 수 있는 하이브리드 및 적응형 시스템에 있을 가능성이 높다. 이러한 시스템은 다양한 시나리오에서 정확성과 실시간성 사이의 최적의 균형을 동적으로 찾아낼 것이다.


1. TRLO: An Efficient LiDAR Odometry with 3D Dynamic Object Tracking and Removal - arXiv, accessed July 24, 2025, https://arxiv.org/html/2410.13240v1
2. Lidar Simultaneous Localization and Mapping Algorithm for Dynamic Scenes - MDPI, accessed July 24, 2025, https://www.mdpi.com/2032-6653/15/12/567
3. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/24/2/645
4. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/377547031_A_Review_of_Dynamic_Object_Filtering_in_SLAM_Based_on_3D_LiDAR
5. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - PMC - PubMed Central, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10821332/
6. Review: Issues and Challenges of Simultaneous Localization and Mapping (SLAM) Technology in Autonomous Robot - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/369646388_Review_Issues_and_Challenges_of_Simultaneous_Localization_and_Mapping_SLAM_Technology_in_Autonomous_Robot
7. Removing Moving Objects without Registration from 3D LiDAR Data Using Range Flow Coupled with IMU Measurements - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/15/13/3390
8. Evolution of SLAM: Toward the Robust- Perception of Autonomy - University of Twente Research Information, accessed July 24, 2025, https://research.utwente.nl/files/302104014/2302.06365.pdf
9. (PDF) Robust Method for Removing Dynamic Objects from Point Clouds - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/346918112_Robust_Method_for_Removing_Dynamic_Objects_from_Point_Clouds
10. Mapping the Static Parts of Dynamic Scenes from 3D LiDAR Point Clouds Exploiting Ground Segmentation, accessed July 24, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/arora2021ecmr.pdf
11. Dynamic objects, highlighted in red, causing ghost trail effect in a point cloud map - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/Dynamic-objects-highlighted-in-red-causing-ghost-trail-effect-in-a-point-cloud-map_fig1_346918112
12. A Dynamic Points Removal Benchmark in Point Cloud Maps, accessed July 24, 2025, https://arxiv.org/pdf/2307.07260
13. Accuracy (RMS-ATE in meters) comparison between RGB-D SLAM [9] and the histogram similarity technique - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/Accuracy-RMS-ATE-in-meters-comparison-between-RGB-D-SLAM-9-and-the-histogram_fig3_320309213
14. No More Potentially Dynamic Objects: Static Point Cloud Map Generation based on 3D Object Detection and Ground Projection - arXiv, accessed July 24, 2025, https://arxiv.org/html/2407.01073v1
15. SLAM for Autonomous Driving: Concept and Analysis | Encyclopedia MDPI, accessed July 24, 2025, https://encyclopedia.pub/entry/history/compare_revision/96146
16. SLAM navigation and how it impacts robotics - Mecalux.com, accessed July 24, 2025, https://www.mecalux.com/blog/slam-navigation-robotics
17. Enhanced Visual SLAM for Collision-Free Driving with Lightweight Autonomous Cars - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11478337/
18. The Peopleremover-Removing Dynamic Objects From 3-D Point Cloud Data by Traversing a Voxel Occupancy Grid - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/322945535_The_Peopleremover-Removing_Dynamic_Objects_From_3-D_Point_Cloud_Data_by_Traversing_a_Voxel_Occupancy_Grid
19. DynaHull: Density-centric Dynamic Point Filtering in Point Clouds - arXiv, accessed July 24, 2025, https://arxiv.org/html/2401.07541v1
20. Removing Moving Objects from Point Cloud Scenes - PhD Alumni ..., accessed July 24, 2025, http://alumni.cs.ucr.edu/~klitomis/files/KrystofFu.pdf
21. Enhancing LiDAR Mapping with YOLO-Based Potential Dynamic Object Removal in Autonomous Driving - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644668/
22. [2104.03657] Dynamic Object Aware LiDAR SLAM based on Automatic Generation of Training Data - arXiv, accessed July 24, 2025, https://arxiv.org/abs/2104.03657
23. MOLO-SLAM: A Semantic SLAM for Accurate Removal of Dynamic Objects in Agricultural Environments - MDPI, accessed July 24, 2025, https://www.mdpi.com/2077-0472/14/6/819
24. SuMa++: Efficient LiDAR-based Semantic SLAM - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/351840542_SuMa_Efficient_LiDAR-based_Semantic_SLAM
25. DynaSLAM: Tracking, Mapping and Inpainting in Dynamic Scenes - arXiv, accessed July 24, 2025, http://arxiv.org/pdf/1806.05620
26. YPR-SLAM: A SLAM System Combining Object Detection and Geometric Constraints for Dynamic Scenes - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11511238/
27. Robust Visual SLAM in Dynamic Environment Based on Motion Detection and Segmentation | J. Auton. Veh. Sys. | ASME Digital Collection, accessed July 24, 2025, https://asmedigitalcollection.asme.org/autonomousvehicles/article/4/1/011001/1201322/Robust-Visual-SLAM-in-Dynamic-Environment-Based-on
28. Lidar-only Odometry based on Multiple Scan-to-Scan Alignments over a Moving Window - arXiv, accessed July 24, 2025, http://www.arxiv.org/pdf/2503.21293
29. Long-Term Localization Method Integrated With Voxel Mapping LiDAR Odometry and Adaptive Updating Map in Diverse Environment | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/388371896_Long-Term_Localization_Method_Integrated_with_Voxel_Mapping_LiDAR_Odometry_and_Adaptive_Updating_Map_in_Diverse_Environment
30. Sliding Window Analysis - osl-dynamics, accessed July 24, 2025, https://osl-dynamics.readthedocs.io/en/v1.4.0/tutorials_build/sliding_window_analysis.html
31. Evaluation of sliding window correlation performance for characterizing dynamic functional connectivity and brain states - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC4889509/
32. (PDF) Quantifying Efficiency of Sliding-Window Based Aggregation Technique by Using Predictive Modeling on Landform Attributes Derived from DEM and NDVI - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/332621765_Quantifying_Efficiency_of_Sliding-Window_Based_Aggregation_Technique_by_Using_Predictive_Modeling_on_Landform_Attributes_Derived_from_DEM_and_NDVI
33. A survey on sliding window sketch for network measurement - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/369177953_A_survey_on_sliding_window_sketch_for_network_measurement
34. AS-LIO: Spatial Overlap Guided Adaptive Sliding Window LiDAR-Inertial Odometry for Aggressive FOV Variation - arXiv, accessed July 24, 2025, https://arxiv.org/html/2408.11426v1
35. LP-ICP: General Localizability-Aware Point Cloud Registration for Robust Localization in Extreme Unstructured Environments - arXiv, accessed July 24, 2025, https://arxiv.org/html/2501.02580v1
36. An Entropy Analysis-Based Window Size Optimization Scheme for Merging LiDAR Data Frames - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9739981/
37. An Entropy Analysis-Based Window Size Optimization Scheme for Merging LiDAR Data Frames - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/22/23/9293
38. Dynamic Pose Graph SLAM: Long-term Mapping in Low Dynamic Environments - CMU School of Computer Science, accessed July 24, 2025, https://www.cs.cmu.edu/~kaess/pub/WalcottBryant12iros.pdf
39. An Observability-Constrained Sliding Window Filter for SLAM, accessed July 24, 2025, https://udel.edu/~ghuang/papers/tr_ocswf.pdf
40. KISS-SLAM: A Simple, Robust, and Accurate 3D LiDAR SLAM System With Enhanced Generalization Capabilities, accessed July 24, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/kiss2025iros.pdf
41. KISS-ICP: In Defense of Point-to-Point ICP – Simple ... - Harmony, accessed July 24, 2025, https://harmony-eu.org/wp-content/uploads/2023/10/vizzo2023kiss.pdf
42. KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way, accessed July 24, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/vizzo2023ral.pdf
43. KISS-ICP - Rerun, accessed July 24, 2025, https://rerun.io/examples/robotics/kiss-icp
44. KISS-ICP: In Defense of Point-to-Point ICP Simple, Accurate, and Robust Registration If Done the Right Way - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/367101986_KISS-ICP_In_Defense_of_Point-to-Point_ICP_Simple_Accurate_and_Robust_Registration_If_Done_the_Right_Way
45. KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way - Bohrium, accessed July 24, 2025, https://www.bohrium.com/paper-details/kiss-icp-in-defense-of-point-to-point-icp-simple-accurate-and-robust-registration-if-done-the-right-way/864973736359493879-2536
46. Jeldrik Axmann - Archiv, accessed July 24, 2025, https://archiv.badw.de/en/050323929/050323929.pdf
47. KISS-ICP: In Defense of Point-to-Point ICP -- Simple, Accurate, and Robust Registration If Done the Right Way - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/364110233_KISS-ICP_In_Defense_of_Point-to-Point_ICP_--_Simple_Accurate_and_Robust_Registration_If_Done_the_Right_Way
48. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PubMed, accessed July 24, 2025, https://pubmed.ncbi.nlm.nih.gov/37299914/
49. (PDF) Dynamic Object Removal for Effective Slam - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/369380319_Dynamic_Object_Removal_for_Effective_Slam
50. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 24, 2025, https://arxiv.org/html/2407.03590v1
51. Removing points from a pcl::PointCloud
52. Point cloud outlier removal - Open3D latest (664eff5) documentation, accessed July 24, 2025, https://www.open3d.org/docs/latest/tutorial/Advanced/pointcloud_outlier_removal.html
53. Remove previously added geometry from the Visualizer / Issue #543 / isl-org/Open3D - GitHub, accessed July 24, 2025, https://github.com/IntelVCL/Open3D/issues/543
54. davutcanakbas/dynamic_object_removal: A ROS-based ... - GitHub, accessed July 24, 2025, https://github.com/davutcanakbas/dynamic_object_removal
55. Creating a Circular Buffer in C and C++ - Embedded Artistry, accessed July 24, 2025, https://embeddedartistry.com/blog/2017/05/17/creating-a-circular-buffer-in-c-and-c/
56. SLAM parameter tuning with Ouster SDK - Software, accessed July 24, 2025, https://community.ouster.com/t/slam-parameter-tuning-with-ouster-sdk/105
57. LiDAR Voxel-Size Optimization for Canopy Gap Estimation - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/14/5/1054
58. Point cloud outlier removal - Open3D 0.9.0 documentation, accessed July 24, 2025, https://www.open3d.org/docs/0.9.0/tutorial/Advanced/pointcloud_outlier_removal.html
59. Removing non-static objects from point clouds | by yodayoda | Map for Robots | Medium, accessed July 24, 2025, https://medium.com/yodayoda/removing-non-static-objects-from-point-clouds-e35000defc3a
60. Remove, Then Revert: Static Point Cloud Map Construction Using Multiresolution Range Images, accessed July 24, 2025, http://ras.papercept.net/images/temp/IROS/files/0855.pdf
61. The peopleremover – removing dynamic objects from 3D point cloud data by traversing a voxel occupancy grid, accessed July 24, 2025, https://robotik.informatik.uni-wuerzburg.de/telematics/download/icra2018.pdf
62. Leveraging Semantic Graphs for Efficient and Robust LiDAR SLAM - arXiv, accessed July 24, 2025, https://arxiv.org/html/2503.11145v2
63. Adaptive Sliding Window for Hierarchical Pose-Graph-Based SLAM | Request PDF, accessed July 24, 2025, https://www.researchgate.net/publication/261092538_Adaptive_Sliding_Window_for_Hierarchical_Pose-Graph-Based_SLAM
64. A life-long SLAM approach using adaptable local maps based on rasterized LIDAR images - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2107.07133
65. (PDF) Long-term 3D map maintenance in dynamic environments - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/262684628_Long-term_3D_map_maintenance_in_dynamic_environments
66. When the object point cloud moves away, the octomap of the original occupied part should be updated to the non-occupied part, but it is very slow to update the non-occupancy part. / Issue #3411 / moveit/moveit - GitHub, accessed July 24, 2025, https://github.com/ros-planning/moveit/issues/3411
67. Consistency Analysis for Sliding-Window Visual Odometry - Department of Electrical and Computer Engineering - University of California, Riverside, accessed July 24, 2025, https://intra.ece.ucr.edu/~mourikis/tech_reports/visual_odometry.pdf
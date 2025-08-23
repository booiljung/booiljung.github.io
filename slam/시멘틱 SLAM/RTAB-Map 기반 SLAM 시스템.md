# RTAB-Map 기반 SLAM 시스템


동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술은 자율 로봇이 미지의 환경을 탐색하고 상호작용하기 위한 핵심적인 인식 능력이다. SLAM의 근본적인 과제는 '닭과 달걀' 문제로 비유되는 위치 추정(Localization)과 지도 작성(Mapping) 간의 상호 의존성에 있다.1 정확한 지도가 존재해야 로봇이 자신의 위치를 추정할 수 있으며, 동시에 로봇의 정확한 위치를 알아야 일관성 있는 지도를 작성할 수 있기 때문이다. 이 순환 종속성은 SLAM 알고리즘이 해결해야 할 본질적인 난제이며, 현대 SLAM 시스템은 두 과업을 확률론적 프레임워크 내에서 동시에 추정함으로써 이 문제를 해결하고자 한다. 초기 SLAM 연구는 확장 칼만 필터(Extended Kalman Filter, EKF)와 같은 필터 기반 접근법을 통해 이 문제를 다루었으나, 환경의 크기가 커짐에 따라 상태 벡터와 공분산 행렬의 차원이 급격히 증가하여 계산 복잡도가 기하급수적으로 높아지는 한계를 드러냈다.3

이러한 계산 복잡성 문제는 대규모 환경에서의 장기 운용(large-scale and long-term operation)이라는 또 다른 중대한 도전 과제로 이어진다. 로봇이 장시간, 넓은 영역을 이동함에 따라 주행 거리계(odometry) 센서에서 발생하는 미세한 측정 오차가 지속적으로 누적되어 '드리프트(drift)'라는 심각한 위치 추정 오차를 유발한다.7 이 누적 오차는 생성된 지도의 전역적 일관성(global consistency)을 파괴하여, 로봇이 이전에 방문했던 장소를 인식하지 못하거나 맵이 뒤틀리는 결과를 초래한다. 또한, 맵의 크기가 방대해짐에 따라 새로운 관측 데이터를 기존의 모든 맵 데이터와 비교하고, 맵 전체의 정합성을 유지하기 위해 최적화하는 데 필요한 계산량이 폭발적으로 증가한다. 이는 결국 실시간(real-time) 처리를 불가능하게 만드는 시스템의 핵심적인 병목 현상으로 작용한다.5

이러한 대규모 및 장기 운용의 문제를 정면으로 해결하기 위해 설계된 정교한 프레임워크가 바로 RTAB-Map(Real-Time Appearance-Based Mapping)이다.12 RTAB-Map은 그래프 기반 SLAM 접근법을 채택하여, 누적 오차와 계산 복잡성이라는 두 가지 난제를 해결하기 위한 독창적인 해법을 제시한다. 그 핵심 철학은 첫째, 로봇의 주행 거리계에만 의존하지 않고 환경의 시각적 '외형(Appearance)' 정보에 기반하여 이전에 방문했던 장소를 강인하게 재인식하는 '루프 폐쇄(loop closure)'를 수행하는 것이다. 둘째, 인간의 기억 메커니즘에서 영감을 받은 정교한 '메모리 관리' 기법을 통해, 실시간 처리에 필요한 계산량을 일정 수준 이하로 제어하여 맵의 크기에 관계없이 시스템의 확장성을 보장하는 것이다.10 본 보고서는 RTAB-Map의 두 가지 핵심 기둥인 외형 기반 루프 폐쇄 탐지 메커니즘과 계층적 메모리 관리 기법을 심층적으로 분석하고자 한다. 이를 위해 Bag-of-Words 모델, 베이즈 필터의 확률적 추론 과정, 그리고 단기/작업/장기 기억(STM/WM/LTM) 간의 상호작용을 포함한 수학적 원리와 실제적 구현 방식을 상세히 기술할 것이다. 이를 통해 현대 SLAM 기술이 장기 자율성을 확보하기 위해 어떠한 아키텍처적 해법을 모색하고 있는지 조망하고, 그 기술적 의의와 미래 발전 가능성을 고찰하는 것을 목적으로 한다.


RTAB-Map의 아키텍처는 대규모 환경에서의 실시간 성능과 확장성을 최우선 목표로 설계되었다. 이는 그래프 기반 SLAM 프레임워크를 근간으로 하며, 프론트엔드와 백엔드를 명확히 분리하고, 시스템의 정체성을 규정하는 외형 정보 중심의 데이터 처리 파이프라인을 채택함으로써 달성된다.


RTAB-Map은 환경 지도와 로봇의 궤적을 하나의 통합된 그래프(Graph) 구조로 표현한다.3 이 그래프에서 각 노드(Node)는 로봇이 특정 시점에 도달한 위치의 포즈(pose, 위치와 방향)와 해당 지점에서 수집한 센서 데이터(예: RGB-D 이미지, 포인트 클라우드)를 저장한다. 노드들을 연결하는 엣지(Edge)는 노드 간의 공간적 제약(spatial constraint)을 나타내며, 이는 로봇의 움직임이나 환경 관측으로부터 얻어진다.14

엣지는 크게 두 가지 유형으로 구분된다.14

1. **주행 거리계 제약 (Odometry Constraint):** 연속된 두 노드(예: $t-1$ 시점과 $t$ 시점) 사이의 상대적인 움직임을 나타낸다. 이는 휠 인코더, IMU, 혹은 시각 주행 거리계(Visual Odometry)와 같은 센서로부터 추정되며, 그래프의 기본적인 뼈대를 형성한다.
2. **루프 폐쇄 제약 (Loop Closure Constraint):** 로봇이 과거에 방문했던 장소를 현재 위치에서 다시 인식했을 때 생성되는 엣지다. 이는 시간적으로 멀리 떨어진 두 노드를 직접 연결하며, 주행 거리계의 누적 오차를 보정하고 지도의 전역적 일관성을 확보하는 데 결정적인 역할을 한다.

이러한 그래프 구조는 새로운 관측이 추가될 때마다 노드와 엣지를 점진적으로 추가할 수 있게 하여 확장성이 뛰어나며, 루프 폐쇄가 발생했을 때 전체 그래프를 최적화함으로써 전역적인 오차를 효과적으로 분배하고 수정할 수 있는 기반을 제공한다.


RTAB-Map은 SLAM 문제를 효율적으로 처리하기 위해 프론트엔드와 백엔드로 역할을 명확히 분리하는 현대적인 SLAM 아키텍처를 따른다.14

- **프론트엔드:** 이 부분은 실시간으로 입력되는 센서 데이터를 직접 처리하는 역할을 담당한다. 주요 과업은 주행 거리계 추정, 즉 연속된 프레임 간의 로봇 움직임을 계산하여 새로운 노드와 주행 거리계 엣지를 생성하는 것이다. 이와 동시에, 프론트엔드는 현재 관측이 과거의 어느 장소와 유사한지를 판단하여 루프 폐쇄 후보를 신속하게 감지한다. RTAB-Map의 프론트엔드는 특히 이미지에서 시각적 특징(visual features)을 추출하고 이를 '시각적 단어(visual words)'로 변환하여, 백엔드가 루프 폐쇄 가설을 검증하는 데 사용할 핵심 정보를 생성하는 데 중점을 둔다.14 모든 연산은 다음 센서 데이터가 들어오기 전에 완료되어야 하므로 속도가 매우 중요하다.
- **백엔드:** 백엔드는 프론트엔드에서 신뢰도 높은 루프 폐쇄 가설이 전달되었을 때 활성화된다. 백엔드의 주된 임무는 새로운 루프 폐쇄 제약 엣지를 그래프에 추가한 뒤, 그래프 최적화(Graph Optimization)를 수행하는 것이다.12 이 최적화 과정은 그래프에 존재하는 모든 제약 조건의 오차를 전역적으로 최소화하도록 모든 노드의 포즈를 재조정한다. 이 과정은 계산 비용이 상대적으로 높기 때문에 실시간으로 매 프레임마다 수행되지 않고, 루프 폐쇄와 같은 중요한 이벤트가 발생했을 때 비동기적으로 실행되어 시스템의 전반적인 실시간성을 해치지 않는다.


많은 SLAM 시스템이 LiDAR 센서 등에서 얻는 정밀한 기하학적 정보에 크게 의존하는 반면, RTAB-Map은 이름(Real-Time **Appearance-Based** Mapping)에서 알 수 있듯이 시각적 '외형' 정보를 시스템의 핵심 축으로 삼는다.12 이는 장기적이고 대규모적인 자율 항법 시나리오에서, 장소의 정밀한 3차원 구조보다 그 장소의 고유한 시각적 특징(예: 벽에 걸린 포스터, 독특한 패턴의 카펫)이 재인식을 위한 더 강인하고 효율적인 단서가 될 수 있다는 통찰에 기반한다.

기하학적 정합(예: 포인트 클라우드 간의 ICP 정합)은 매우 정밀할 수 있지만, 특징이 부족한 복도나 구조적 변화가 있는 환경에서는 실패하기 쉽고 전역적으로 모든 지점과 비교하는 것은 계산 비용이 막대하다. 반면, 외형 기반 인식은 Bag-of-Words와 같은 기법을 통해 대규모 데이터베이스에서도 매우 빠른 검색이 가능하며, 약간의 시점 변화나 조명 변화에 대해 상대적으로 강인한 특성을 보인다.14 RTAB-Map의 파이프라인은 먼저 빠르고 효율적인 외형 정보 비교를 통해 루프 폐쇄 후보를 생성하고, 이후 후보군에 대해서만 정밀한 기하학적 검증을 수행하는 계층적 접근을 취한다. 이러한 설계 철학은 계산 자원을 효율적으로 사용하면서도 대규모 환경에서의 확장성과 실시간 성능을 극대화하는 핵심적인 전략이다.


RTAB-Map의 강인함은 단순히 이미지를 비교하는 것을 넘어, 시간적 일관성을 고려한 확률적 추론을 통해 루프 폐쇄를 결정하는 정교한 메커니즘에 있다. 이 장에서는 Bag-of-Words(BoW) 모델을 이용한 시각적 표현과 베이즈 필터를 통한 가설 검증 과정을 심층적으로 분석한다.


BoW 모델은 원래 문서 분류를 위해 텍스트 처리 분야에서 사용되던 개념으로, 이를 시각 정보에 적용하여 이미지를 소수의 '시각적 단어'들의 집합으로 표현하는 기법이다.


루프 폐쇄 탐지의 첫 단계는 입력 이미지에서 안정적이고 식별력 있는 특징점을 찾는 것이다. RTAB-Map은 기본적으로 SURF(Speeded Up Robust Features)나 SIFT, ORB와 같은 불변 특징점 추출 알고리즘을 사용한다.14 이 알고리즘들은 이미지의 크기 변화, 회전, 조명 변화에도 비교적 안정적으로 동일한 지점을 검출할 수 있다. 검출된 각 특징점은 주변 픽셀의 그래디언트 분포나 밝기 패턴 등을 요약한 고차원 벡터인 '기술자(descriptor)'로 표현된다. 이 기술자는 해당 특징점의 고유한 시각적 '지문' 역할을 한다.


추출된 고차원 기술자들을 직접 비교하는 것은 계산 비용이 매우 높다. BoW 모델은 이 기술자 공간을 양자화(quantize)하여 유한한 개수의 대표 벡터, 즉 '시각적 단어(visual word)'로 구성된 사전을 만든다. 많은 SLAM 시스템이 대규모 이미지 데이터셋으로 미리 훈련된 오프라인 사전을 사용하는 반면, RTAB-Map은 탐색을 진행하면서 마주치는 새로운 특징들을 기반으로 점진적으로 사전을 구축하고 확장하는 '증분적(incremental)' 방식을 채택한다.18 이는 특정 환경(예: 사무실, 창고, 숲)에 동적으로 적응하여 해당 환경에 특화된 시각적 단어들을 생성할 수 있게 하므로, 시스템의 유연성과 성능을 크게 향상시킨다.


시각 사전이 구축되면, 이미지 내의 각 특징점 기술자는 사전에서 가장 가까운 시각적 단어에 할당된다. 이를 통해 이미지는 내부에 포함된 시각적 단어들의 빈도를 나타내는 히스토그램, 즉 BoW 벡터로 간결하게 표현된다.10 이제 두 이미지 간의 시각적 유사도는 그들의 BoW 벡터 간의 유사도(예: 코사인 유사도 또는 L1-정규화된 점수)를 계산함으로써 매우 빠르게 측정할 수 있다. 또한, 각 시각적 단어가 어떤 이미지들에서 나타났는지를 저장하는 '역인덱스(inverted index)' 구조를 사용하면, 현재 이미지에 나타난 특정 단어를 포함하는 과거의 모든 이미지를 데이터베이스에서 즉시 검색할 수 있어 대규모 맵에서도 효율적인 후보군 탐색이 가능하다.14


단일 프레임 간의 높은 BoW 유사도 점수만으로는 루프 폐쇄를 확정하기에 충분하지 않다. 복도나 사무실 칸막이처럼 시각적으로 유사하지만 다른 장소인 '지각적 모호성(perceptual aliasing)' 문제로 인해 잘못된 루프 폐쇄(false positive)가 발생할 위험이 크기 때문이다. RTAB-Map은 이러한 문제를 해결하고 시간적 일관성을 반영하기 위해 이산 베이즈 필터(discrete Bayesian filter)를 사용하여 루프 폐쇄 가설의 신뢰도를 확률적으로 추론한다.14


베이즈 필터는 시간의 흐름에 따라 들어오는 증거(관측)를 바탕으로 시스템의 상태에 대한 믿음(belief)을 갱신하는 재귀적인 확률 추정 도구다. RTAB-Map에서 시간 $t$에서의 상태 변수 $S_t$는 "현재 위치 $L_t$가 이전에 방문했던 어떤 장소와 일치하는가?"에 대한 가설을 나타낸다. 구체적으로, $S_t=i$는 현재 위치 $L_t$가 과거의 위치 $L_i$와 루프를 형성한다는 가설을, $S_t=-1$은 $L_t$가 완전히 새로운 장소라는 가설을 의미한다.21

필터는 이전 시간 단계($t-1$)의 믿음 분포 $p(S_{t-1}|L_{t-1})$를 바탕으로 현재 시간($t$)의 믿음 분포 $p(S_t|L_t)$를 다음과 같은 재귀적 방정식으로 업데이트한다 21:
$$
p(S_t|L_t) = \eta \underbrace{p(L_t|S_t)}_{\text{관측 모델}} \sum_{i} \underbrace{p(S_t|S_{t-1}=i)}_{\text{전이 모델}} \underbrace{p(S_{t-1}=i|L_{t-1})}_{\text{이전 믿음}}
$$
여기서 각 항의 의미는 다음과 같다.

- **이전 믿음 ($p(S_{t-1}=i|L_{t-1})$):** $t-1$ 시점까지의 모든 관측을 고려했을 때, $S_{t-1}=i$ 가설이 참일 확률.
- **전이 모델 ($p(S_t|S_{t-1}=i)$):** $t-1$ 시점에서 $t$ 시점으로 시간이 흐름에 따라 상태가 어떻게 변할지에 대한 확률. 이는 로봇의 움직임에 대한 물리적 제약을 모델링한다. 예를 들어, 로봇이 갑자기 맵의 반대편으로 순간이동할 확률은 매우 낮다. 따라서 $S_{t-1}=j$ 가설이 참이었다면, $S_t$는 $j$의 이웃 노드 중 하나일 확률이 높게 설정된다. 이는 시간적 연속성을 강제하여 단발적인 노이즈에 강인하게 만든다.21
- **관측 모델 ($p(L_t|S_t)$):** 현재 관측(BoW 유사도 점수)이 특정 가설 하에서 얼마나 그럴듯한지를 나타내는 확률. 만약 현재 위치 $L_t$와 과거 위치 $L_j$ 간의 BoW 유사도 점수 $s_j$가 높다면, $p(L_t|S_t=j)$ 값은 높아진다. 이는 현재의 시각적 증거를 믿음에 반영하는 역할을 한다.21
- $\eta$: 확률의 총합을 1로 만들기 위한 정규화 상수.

이러한 베이즈 필터의 적용은 단발적인 높은 유사도 점수에 현혹되지 않고, 여러 프레임에 걸쳐 일관되게 높은 확률을 보이는 가설만을 채택하게 하는 강력한 시간적 필터 역할을 한다. 업데이트된 확률 분포 $p(S_t|L_t)$에서 가장 높은 확률을 가진 가설의 값이 사전에 정의된 임계값 $H$를 초과하면, 해당 루프 폐쇄가 최종적으로 수용된다.14 이 과정을 통해 지각적 모호성 문제에 대한 강인성을 획득하고, 루프 폐쇄의 신뢰도를 극대화할 수 있다.


로봇이 장시간 작동하며 방대한 환경을 탐색하면 맵 데이터의 크기는 무한정 증가한다. 모든 과거 위치를 항상 활성 상태로 유지하고 현재 위치와 비교하는 것은 실시간 SLAM 시스템에서 계산적으로 불가능하다. RTAB-Map은 이러한 확장성 문제를 해결하기 위해 인간의 기억 시스템에서 영감을 받은 독창적인 계층적 메모리 관리 아키텍처를 도입했다.10


RTAB-Map의 메모리는 세 가지 계층으로 나뉘어, 각기 다른 역할과 생명 주기를 가진 데이터를 관리한다.

- **단기 기억 (Short-Term Memory, STM):** 새로 생성된 위치 노드가 가장 먼저 저장되는 고정된 크기의 버퍼다.14 STM의 주된 목적은 시간적으로 매우 인접한 위치들 간의 불필요한 루프 폐쇄 계산을 방지하는 것이다. 로봇이 막 지나온 위치는 현재 위치와 당연히 시각적으로 유사하므로, 이를 루프 폐쇄 후보로 간주하는 것은 계산 낭비이자 잘못된 연결을 유발할 수 있다. 따라서 STM에 있는 노드들은 루프 폐쇄 탐지 대상에서 제외된다.18 STM이 가득 차면, 가장 오래된 노드가 작업 기억(WM)으로 이동하여 본격적인 루프 폐쇄 탐지 후보가 된다.14
- **작업 기억 (Working Memory, WM):** 시스템의 '주의 집중' 영역에 해당하며, 루프 폐쇄 탐지와 그래프 최적화가 활발하게 일어나는 핵심 메모리 공간이다.10 현재 위치는 오직 WM에 있는 노드들과의 유사도만을 비교한다. WM의 크기는 고정되어 있지 않고, 시스템의 실시간 성능을 보장하기 위해 동적으로 관리된다. 즉, WM에서의 연산 시간이 미리 설정된 임계값을 초과하면, WM의 일부 노드가 장기 기억(LTM)으로 옮겨져 WM의 크기와 계산 부하를 일정 수준으로 유지한다.10
- **장기 기억 (Long-Term Memory, LTM):** WM에서 밀려난, 상대적으로 오래되거나 덜 중요하다고 판단된 노드들이 저장되는 영구적인 데이터베이스다 (예: SQLite3 데이터베이스).14 LTM에 저장된 노드들은 직접적인 루프 폐쇄 탐지 대상에서 제외되므로, 시스템의 실시간 계산 부하에 영향을 주지 않는다. 하지만 데이터가 삭제되는 것은 아니므로, 필요할 때 다시 WM으로 '회수(retrieve)'될 수 있다. 이는 방대한 맵 전체를 보존하면서도 실시간 성능을 유지하는 핵심적인 전략이다.10


어떤 노드를 WM에 남기고 어떤 노드를 LTM으로 보낼지 결정하는 것은 메모리 관리 시스템의 효율성을 좌우하는 핵심이다. RTAB-Map은 경험적 가중치(heuristic weight)를 기반으로 이 우선순위를 결정한다.


노드의 중요도를 평가하기 위해 '리허설'이라는 과정이 사용된다.10 로봇이 특정 장소에서 오래 머물거나, 천천히 움직이면서 연속적으로 유사한 이미지를 얻게 되면, RTAB-Map은 매번 새로운 노드를 생성하는 대신 기존 노드에 새로운 관측 정보를 병합하고 해당 노드의 '가중치(weight)'를 1씩 증가시킨다.10 이 과정은 마치 인간이 중요한 정보를 반복적으로 학습하여 기억을 강화하는 것과 유사하다. 결과적으로, 로봇이 자주 방문하거나 중요하다고 판단되는 교차로나 특징적인 공간에 해당하는 노드들은 더 높은 가중치를 갖게 된다.


WM에서 LTM으로의 데이터 전송은 시스템의 실시간 제약 조건이 위협받을 때 촉발된다. 구체적으로, WM에 있는 노드들을 대상으로 루프 폐쇄를 탐지하고 처리하는 데 걸리는 시간이 사전에 설정된 임계 시간(예: `Rtabmap/TimeThr` 파라미터, 700ms)을 초과하거나, WM에 저장된 노드의 총 개수가 임계값(`Rtabmap/MemoryThr`)을 넘어서면 전송 메커니즘이 활성화된다.10

전송될 노드를 선택하는 기준은 명확하다. 시스템은 WM에 있는 모든 노드 중에서 **가장 낮은 가중치를 가진 노드**를 우선적으로 고려한다. 만약 가장 낮은 가중치를 가진 노드가 여러 개 있다면, 그중에서 **가장 오래된(oldest) 노드**가 최종적으로 선택되어 LTM으로 전송된다.10 이 전략은 '덜 중요하고 오래된 기억'부터 비활성화함으로써, 최근에 방문했거나 중요하다고 판단된 장소에 대한 정보를 WM에 최대한 유지하려는 효율적인 휴리스틱이다.


LTM의 진정한 힘은 필요할 때 저장된 정보를 다시 불러올 수 있는 '회수' 기능에 있다. 만약 현재 위치가 WM에 있는 특정 노드 $L_i$와 성공적으로 루프를 형성했다고 판단되면, 이는 로봇이 과거에 탐사했던 영역으로 돌아왔다는 강력한 신호다. 이때 시스템은 LTM 데이터베이스를 조회하여, $L_i$와 과거에 이웃 관계(neighbor link)로 연결되었던 노드들을 찾아낸다. 그리고 이 이웃 노드들을 LTM에서 다시 WM으로 가져온다.25 이 회수 메커니즘 덕분에, 로봇이 한동안 방문하지 않아 LTM으로 옮겨졌던 맵 영역이라도 다시 그 근처로 돌아오면 해당 지역의 맵 정보가 다시 활성화된다. 이는 연속적인 루프 폐쇄 탐지를 가능하게 하고, 맵의 단절을 막으며, 대규모 환경에서의 장기적인 일관성을 유지하는 데 필수적인 역할을 수행한다.


루프 폐쇄가 성공적으로 탐지되고 메모리 관리를 통해 시스템의 실시간성이 보장되었다면, 마지막 단계는 누적된 오차를 수정하여 전역적으로 일관된 지도를 만드는 것이다. 이 과정은 그래프 최적화(Graph Optimization)를 통해 이루어지며, SLAM의 백엔드에서 가장 중요한 역할을 담당한다.


프론트엔드에서 현재 노드 $x_t$와 과거 노드 $x_i$ 간의 루프 폐쇄가 감지되면, 두 노드 사이에 새로운 엣지, 즉 루프 폐쇄 제약이 추가된다. 이 제약은 단순히 두 노드가 연결되어 있다는 정보 이상을 담고 있다. 시스템은 두 노드에 저장된 센서 데이터(예: 포인트 클라우드 또는 RGB-D 이미지)를 서로 정합(registration)하여, $x_i$의 좌표계에서 바라본 $x_t$의 상대적인 변환(relative transformation) $T_{i,t}$를 계산한다. 이 변환은 6자유도(위치 $x, y, z$와 회전 roll, pitch, yaw)로 표현된다. 또한, 이 변환의 불확실성을 나타내는 정보 행렬(information matrix) $\Omega_{i,t}$도 함께 계산된다. 정보 행렬은 공분산 행렬의 역행렬로, 정합의 신뢰도를 나타내며 신뢰도가 높을수록 큰 값을 가진다.12


그래프 최적화의 목표는 주행 거리계 제약과 루프 폐쇄 제약을 포함한 모든 제약 조건을 전역적으로 가장 잘 만족시키는 노드 포즈의 집합 $X = \{x_1,..., x_t\}$를 찾는 것이다. 이는 모든 엣지에서 발생하는 오차의 가중 제곱 합(weighted sum of squared errors)을 최소화하는 비선형 최소제곱 문제로 공식화할 수 있다.29
$$
X^* = \underset{X}{\operatorname{argmin}} \sum_{\langle i,j \rangle \in \mathcal{C}} e(x_i, x_j, z_{ij})^T \Omega_{ij} e(x_i, x_j, z_{ij})
$$
위 식에서 각 항의 의미는 다음과 같다.

- $X^*$: 최적화된 노드 포즈의 집합.
- $\mathcal{C}$: 그래프에 있는 모든 제약 조건(엣지)의 집합.
- $z_{ij}$: 노드 $i$와 $j$ 사이의 측정된 제약(주행 거리계 또는 루프 폐쇄에 의한 상대 변환).
- $x_i, x_j$: 현재 추정된 노드 $i$와 $j$의 전역 포즈.
- $e(x_i, x_j, z_{ij})$: 현재 추정된 포즈 $x_i, x_j$와 측정값 $z_{ij}$ 사이의 오차를 계산하는 함수. 예를 들어, $e(x_i, x_j, z_{ij}) = T_{ij}^{-1} \cdot (T_i^{-1} \cdot T_j)$ 와 같이 정의될 수 있다.
- $\Omega_{ij}$: 제약 $z_{ij}$의 신뢰도를 나타내는 정보 행렬.

이 최적화 문제는 로봇 포즈의 회전 요소로 인해 비선형성을 띠므로, 해석적으로 해를 구하기 어렵다. 따라서 가우스-뉴턴(Gauss-Newton) 또는 레벤버그-마르쿼트(Levenberg-Marquardt)와 같은 반복적인 수치 최적화 기법을 사용하여 해를 근사적으로 찾아야 한다.


RTAB-Map은 복잡한 비선형 최적화 문제를 직접 구현하는 대신, 이 분야에서 검증된 고성능 오픈소스 라이브러리를 백엔드로 활용한다. 대표적으로 g2o(General Graph Optimization)와 GTSAM(Georgia Tech Smoothing and Mapping)이 사용된다.14 이 라이브러리들은 희소 행렬(sparse matrix)의 특성을 활용하여 대규모 그래프에서도 효율적으로 최적화를 수행할 수 있는 정교한 알고리즘을 제공한다. 사용자는 어떤 최적화기를 사용할지 선택할 수 있으며, 이를 통해 다양한 환경과 요구사항에 맞춰 시스템의 성능을 조정할 수 있다.


루프 폐쇄 제약이 추가된 후 그래프 최적화가 실행되면, 시스템은 누적된 주행 오차로 인해 발생한 불일치(inconsistency)를 해소하기 위해 모든 노드의 포즈를 재조정한다. 루프 폐쇄는 강력한 '진실의 닻(truth anchor)' 역할을 한다. 주행 거리계 제약은 국소적이고 점진적으로 오차를 누적시키지만, 루프 폐쇄 제약은 맵의 멀리 떨어진 두 부분을 직접 연결하며 강력한 전역적 정보를 제공한다. 최적화기는 이 새로운 루프 폐쇄 제약의 오차를 줄이기 위해, 해당 루프에 포함된 모든 노드들의 포즈를 조금씩 수정한다. 이 수정은 마치 팽팽하게 당겨진 고무줄을 놓았을 때 전체가 이완되는 것처럼, 루프 폐쇄 지점의 보정 정보가 그래프 전체로 전파되는 효과를 낳는다.14 결과적으로, 맵의 뒤틀림이 펴지고, 분리되었던 맵 영역이 정합되며, 전역적으로 일관되고 정확한 최종 지도가 생성된다.


RTAB-Map은 대규모 장기 운용이라는 특정 문제 영역에 대한 정교한 해법을 제시하며 SLAM 기술 생태계에서 독자적인 위치를 차지한다. 이 장에서는 RTAB-Map의 기술적 장단점을 분석하고, 다른 주요 SLAM 패러다임과 비교하며, SLAM 기술의 미래 발전 방향을 조망한다.


RTAB-Map의 설계 철학은 명확한 장점과 그에 따른 기술적 트레이드오프를 내포하고 있다.

- **장점:**
  - **대규모/장기 운용 능력:** 계층적 메모리 관리 시스템은 맵의 크기가 증가해도 계산 복잡성을 일정하게 유지하여, 장시간 동안 넓은 영역에서 안정적으로 작동할 수 있는 독보적인 능력을 제공한다.12
  - **실시간 성능 보장:** 모든 핵심 연산, 특히 루프 폐쇄 탐지는 실시간 제약 조건 내에서 완료되도록 설계되었다. 이는 자율 항법과 같이 즉각적인 반응이 요구되는 응용에 필수적이다.10
  - **유연한 센서 통합:** RGB-D, 스테레오 카메라, 2D/3D LiDAR 등 다양한 센서 조합을 지원하여 특정 하드웨어나 환경에 국한되지 않는 높은 유연성을 가진다.33
  - **강력한 ROS 통합:** ROS(Robot Operating System)와 긴밀하게 통합되어 있어, 실제 로봇 시스템에 적용하고 다른 ROS 패키지(예: 내비게이션 스택)와 연동하기가 용이하다.34
- **단점 및 트레이드오프:**
  - **루프 폐쇄 실패 가능성:** 메모리 관리의 필연적인 결과로, LTM으로 전송된 노드들은 직접적인 루프 폐쇄 탐지 대상에서 제외된다. 이로 인해 로봇이 LTM에 있는 영역을 다시 방문하더라도, WM에 있는 노드와 먼저 연결고리를 찾지 못하면 루프를 놓칠 수 있다. 이는 '완벽한 정합성'보다 '지속 가능한 실시간성'을 우선시하는 RTAB-Map의 근본적인 트레이드오프다.13
  - **외형 변화에 대한 민감성:** 시스템의 핵심이 외형 정보에 기반하므로, 조명이 극적으로 변하거나(예: 낮과 밤), 계절이 바뀌어 환경의 외관이 크게 달라지는 동적 환경에서는 장소 인식 성능이 저하될 수 있다.13


SLAM 기술 분야에는 단 하나의 '최고' 알고리즘이 존재하는 것이 아니라, 각기 다른 강점을 가진 특화된 솔루션들의 생태계가 형성되어 있다. RTAB-Map의 특징은 다른 주요 SLAM 시스템과 비교할 때 더욱 명확해진다.

- **ORB-SLAM3:** 특징점 기반 Visual SLAM의 표준으로 여겨지며, 매우 정확하고 강인한 실시간 카메라 궤적 추정 능력을 자랑한다. 소수의 안정적인 랜드마크(특징점)로 구성된 희소 맵(sparse map)을 사용하여 계산 효율을 높인다. 하지만 맵이 희소하기 때문에 장애물 회피나 3D 환경 이해와 같은 고수준 작업을 위한 밀집 맵(dense map)을 직접 생성하지는 못하며, RTAB-Map과 같은 정교한 장기 메모리 관리 기능은 상대적으로 부족하다.29
- **Google Cartographer:** LiDAR 기반 SLAM의 산업 표준으로 널리 사용된다. 연속적인 센서 입력을 작은 단위의 '서브맵(Submap)'으로 만들고, 이 서브맵들을 기준으로 지역적, 전역적 최적화를 수행하여 드리프트를 효과적으로 억제한다. 주로 정밀한 기하학적 정보에 의존하며, 높은 맵 정확도와 강인성을 제공한다. RTAB-Map처럼 시각적 외형 정보를 루프 폐쇄의 주된 단서로 적극 활용하지는 않는다.16

아래 표는 이 세 가지 주요 SLAM 시스템의 핵심적인 특징을 비교하여 보여준다.

| **특징 (Feature)** | **RTAB-Map**                                         | **ORB-SLAM3**                                       | **Google Cartographer**                       |
| ------------------ | ---------------------------------------------------- | --------------------------------------------------- | --------------------------------------------- |
| **주요 패러다임**  | 그래프 기반, 외형 기반                               | 특징점 기반, 희소(Sparse)                           | 그래프 기반, 서브맵(Submap) 기반              |
| **주요 센서**      | RGB-D, Stereo, 2D/3D LiDAR (유연)                    | Monocular, Stereo, RGB-D                            | 2D/3D LiDAR, IMU                              |
| **맵 표현 방식**   | 밀집(Dense) 포인트 클라우드, 2D/3D 점유 격자, 그래프 | 희소(Sparse) 랜드마크 맵                            | 서브맵 기반의 2D/3D 점유 격자                 |
| **루프 폐쇄 전략** | BoW + 베이즈 필터 (전역적, 외형 기반)                | BoW + 기하학적 검증 (전역적, 특징점 기반)           | 스캔-서브맵 정합 (지역적/전역적, 기하학 기반) |
| **메모리 관리**    | STM/WM/LTM 계층적 관리 (장기 운용 특화)              | Covisibility Graph, Essential Graph (지역적 최적화) | 서브맵 단위 관리, Pose Graph 가지치기         |
| **강점**           | 대규모/장기 운용, 실시간 성능 보장, 유연한 센서 통합 | 높은 포즈 추정 정확도, 빠른 초기화, 경량화          | 낮은 드리프트, 높은 맵 정확도, 강인성         |
| **약점**           | LTM으로 인한 루프 폐쇄 실패 가능성, 외형 변화에 민감 | 텍스처 부족 환경에 취약, 밀집 맵 생성 어려움        | 특징 없는 긴 복도 등에서 드리프트 발생 가능   |
| **주요 응용 분야** | 장기 서비스 로봇, 지속적인 환경 모니터링             | AR/VR, 드론, 실시간 카메라 추적                     | 실내 로봇 항법, 자율주행 차량 매핑            |

이 비교는 각 시스템이 특정 응용 시나리오에 맞춰 최적화되었음을 보여준다. 단기적인 고정밀 추적이 중요한 AR이나 드론에는 ORB-SLAM3가, 정밀한 기하학적 지도가 필수적인 산업용 로봇에는 Cartographer가, 그리고 메모리와 계산 자원의 제약 속에서 장시간 안정적으로 작동해야 하는 가정용 서비스 로봇이나 보안 로봇에는 RTAB-Map이 가장 적합한 솔루션이 될 수 있다.


SLAM 기술은 여전히 해결해야 할 많은 과제를 안고 있으며, RTAB-Map과 같은 프레임워크는 이러한 미래 기술을 통합할 수 있는 훌륭한 기반을 제공한다.

- **동적 환경 대응:** 실제 세계는 사람, 차량, 움직이는 가구 등 동적인 요소로 가득 차 있다. 전통적인 SLAM은 환경이 정적이라는 가정 하에 작동하므로, 이러한 동적 객체들은 위치 추정의 오차를 유발하는 노이즈로 작용한다. 이 문제를 해결하기 위해 딥러닝 기반의 실시간 객체 탐지(object detection) 및 시맨틱 분할(semantic segmentation) 기술을 SLAM 파이프라인에 통합하여, 동적 객체를 식별하고 맵에서 제거하거나 별도로 추적하는 연구가 활발히 진행되고 있다.38
- **Semantic SLAM:** 기존 SLAM이 '어디에 무엇이 있는지'를 기하학적으로 표현했다면, Semantic SLAM은 '그것이 무엇인지'라는 의미(semantic) 정보를 맵에 부여한다. 예를 들어, 맵에 포인트 클라우드 덩어리가 아닌 '의자', '책상', '문'과 같은 객체 레이블을 기록하는 것이다. 이는 로봇이 "주방으로 가서 컵을 가져와"와 같은 고차원적인 명령을 이해하고 수행하며, 인간과 보다 자연스럽게 상호작용하기 위한 필수적인 기술이다.41
- **Neural SLAM (NeRF/3DGS):** 최근 컴퓨터 비전 분야에서 각광받는 NeRF(Neural Radiance Fields)나 3D Gaussian Splatting과 같은 신경망 기반 장면 표현 기술을 SLAM에 도입하려는 시도가 급증하고 있다. 이러한 기술들은 희소한 이미지 입력으로부터 매우 사실적인 3D 장면을 렌더링할 수 있는 잠재력을 가지고 있어, 기존의 포인트 클라우드나 복셀 맵을 뛰어넘는 고품질의 밀집 맵 생성을 가능하게 한다. 하지만 현재로서는 방대한 계산량으로 인해 실시간 성능과 대규모 환경으로의 확장성에 큰 어려움을 겪고 있어, 이를 경량화하고 SLAM 프레임워크에 효율적으로 통합하는 것이 중요한 연구 과제로 남아있다.41


RTAB-Map은 SLAM 기술이 직면한 근본적인 도전 과제인 누적 오차와 계산 복잡성, 특히 대규모 환경에서의 장기 운용 문제에 대한 실용적이고 확장 가능한 해법을 제시했다는 점에서 중요한 기여를 한다. 이 시스템의 핵심은 단순히 정밀한 기하학적 계산에만 의존하는 것이 아니라, 강인한 외형 기반 장소 인식과 인간의 인지 과정을 모사한 효율적인 메모리 관리라는 두 가지 전략을 유기적으로 결합한 데에 있다. Bag-of-Words 모델과 베이즈 필터를 통한 확률적 루프 폐쇄 탐지는 시각적 모호성이 존재하는 실제 환경에서의 강인성을 확보해주었고, STM-WM-LTM으로 이어지는 계층적 메모리 구조는 시스템 자원의 한계 내에서 '지속 가능한' 맵핑을 가능하게 했다.

RTAB-Map의 성공은 미래 SLAM 기술의 발전 방향에 중요한 시사점을 던진다. 자율 시스템이 점차 복잡하고 동적인 실제 세계로 나아감에 따라, SLAM은 더 이상 정적인 환경의 기하학적 구조를 한 번 재구성하는 단발성 과업이 될 수 없다. 대신, 끊임없이 변화하는 정보를 효율적으로 관리하고, 환경 내 객체들의 의미를 이해하며, 장기간에 걸쳐 지속적으로 학습하고 지도를 갱신하는 능력이 요구된다. RTAB-Map이 보여준 정보 관리의 중요성은 Semantic SLAM과 Neural SLAM과 같은 차세대 기술들이 실용화되기 위해 반드시 해결해야 할 과제이기도 하다. 결국, RTAB-Map과 같은 정교한 SLAM 시스템은 지능형 로봇이 실제 세계를 이해하고 그 안에서 진정한 자율성을 획득하기 위한 필수적인 인식의 토대를 구축하는 핵심 기술이라 할 수 있다.


1. Exploring SLAM 3D Mapping Technology | How It Works - LidarTechPros.com, accessed July 31, 2025, https://www.lidartechpros.com/exploring-slam-mapping-technology-works/
2. Simultaneous Localization And Mapping (SLAM) using RTAB-Map - arXiv, accessed July 31, 2025, http://arxiv.org/pdf/1809.02989
3. Simultaneous localization and mapping (SLAM) | Robotics and Bioinspired Systems Class Notes | Fiveable, accessed July 31, 2025, https://library.fiveable.me/robotics-bioinspired-systems/unit-11/simultaneous-localization-mapping-slam/study-guide/PtQW9FKUtK8FAuiA
4. SLAM Algorithms In Dynamic Environments - TUM Neuroscientific System Theory (NST), accessed July 31, 2025, https://tum.neurocomputing.systems/fileadmin/w00bqs/www/publications/as/SS2016_AS_SLAM_DynamicEnv.pdf
5. (PDF) Computational cost analysis of extended Kalman filter in simultaneous localization and mapping (EKF-SLAM) problem for autonomous vehicle - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/283824088_Computational_cost_analysis_of_extended_Kalman_filter_in_simultaneous_localization_and_mapping_EKF-SLAM_problem_for_autonomous_vehicle
6. Power-SLAM: a linear-complexity, anytime algorithm for SLAM - SciSpace, accessed July 31, 2025, https://scispace.com/pdf/power-slam-a-linear-complexity-anytime-algorithm-for-slam-2geaerupyt.pdf
7. GLC-SLAM: Gaussian Splatting SLAM with Efficient Loop Closure - arXiv, accessed July 31, 2025, https://arxiv.org/html/2409.10982v1
8. Understanding Drift in Simultaneous Localization and Mapping (SLAM), accessed July 31, 2025, https://robotics.stackexchange.com/questions/9318/understanding-drift-in-simultaneous-localization-and-mapping-slam
9. A Probabilistic-based Drift Correction Module for Visual Inertial SLAMs, accessed July 31, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-2-2024/297/2024/isprs-archives-XLVIII-2-2024-297-2024.pdf
10. Memory Management for Real-Time Appearance-Based Loop Closure Detection - arXiv, accessed July 31, 2025, https://arxiv.org/html/2407.15890v1
11. A Low-Cost 3D SLAM System Integration of Autonomous Exploration Based on Fast-ICP Enhanced LiDAR-Inertial Odometry - MDPI, accessed July 31, 2025, https://www.mdpi.com/2072-4292/16/11/1979
12. RTAB-Map | Real-Time Appearance-Based Mapping - introlab.github.io, accessed July 31, 2025, http://introlab.github.io/rtabmap/
13. RTAB-Map as an Open-Source Lidar and Visual SLAM Library for Large-Scale and Long-Term Online Operation - arXiv, accessed July 31, 2025, https://arxiv.org/html/2403.06341v1
14. Introduction to 3D SLAM with RTAB-Map | by Shiva Chandrachary ..., accessed July 31, 2025, https://shivachandrachary.medium.com/introduction-to-3d-slam-with-rtab-map-8df39da2d293
15. Simultaneous Localization And Mapping (SLAM) using RTAB-Map - Overleaf, accessed July 31, 2025, https://www.overleaf.com/articles/simultaneous-localization-and-mapping-slam-using-rtab-map/hxynyjskyhsr
16. The Complete Guide to SLAM: Origin, Applications, and Comparison of 5 systems - dtLabs, accessed July 31, 2025, https://dt-labs.ai/blog/the-complete-guide-to-slam/
17. rtabmap - ROS Wiki, accessed July 31, 2025, http://wiki.ros.org/rtabmap
18. Memory Management for Real-Time Appearance-Based Loop Closure Detection - arXiv, accessed July 31, 2025, https://arxiv.org/pdf/2407.15890
19. RTAB-Map 3D Mapping Navigation - Yahboom, accessed July 31, 2025, [https://www.yahboom.net/public/upload/upload-html/1733885422/RTAB-Map%203D%20Mapping%20Navigation.html](https://www.yahboom.net/public/upload/upload-html/1733885422/RTAB-Map 3D Mapping Navigation.html)
20. Ensemble of Bayesian Filters for Loop Closure Detection, accessed July 31, 2025, https://www.jaist.ac.jp/jaist-sast2015/images/pdf/IS_EntertainmentTechnology_Azizi-ABDULLAH_Ensemble-of-Bayesian-Filters-for-Loop-Closure-Detection.pdf
21. (PDF) Appearance-Based Loop Closure Detection for Online Large ..., accessed July 31, 2025, https://www.researchgate.net/publication/260634918_Appearance-Based_Loop_Closure_Detection_for_Online_Large-Scale_and_Long-Term_Operation
22. Appearance-Based Loop Closure Detection in Real-Time for Large-Scale and Long-Term Operation, accessed July 31, 2025, https://cs.gmu.edu/~kosecka/12-0202_01_MS.pdf
23. Analysis of a RGB-D SLAM system using Real-Time Appearance-Based Mapping on the Kbot robot - ADDI, accessed July 31, 2025, https://addi.ehu.es/bitstream/handle/10810/58107/TFG_JonAnder_Ruiz.pdf?sequence=1
24. FAQ / introlab/rtabmap Wiki - GitHub, accessed July 31, 2025, https://github.com/introlab/rtabmap/wiki/FAQ
25. Appearance-Based Loop Closure Detection for Online Large-Scale and Long-Term Operation - arXiv, accessed July 31, 2025, https://arxiv.org/html/2407.15304v1
26. rtabmap: Parameters.h Source File - ROS documentation, accessed July 31, 2025, http://docs.ros.org/lunar/api/rtabmap/html/Parameters_8h_source.html
27. MemoryThr - Official RTAB-Map Forum, accessed July 31, 2025, http://official-rtab-map-forum.206.s1.nabble.com/MemoryThr-td9766.html
28. Memory management in localization mode - Official RTAB-Map Forum - Nabble, accessed July 31, 2025, http://official-rtab-map-forum.206.s1.nabble.com/Memory-management-in-localization-mode-td9886.html
29. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed July 31, 2025, https://www.mathworks.com/discovery/slam.html
30. Robust Graph Optimization / introlab/rtabmap Wiki - GitHub, accessed July 31, 2025, https://github.com/introlab/rtabmap/wiki/Robust-Graph-Optimization
31. rtabmap_ros/Tutorials/Advanced Parameter Tuning - ROS Wiki, accessed July 31, 2025, [http://wiki.ros.org/rtabmap_ros/Tutorials/Advanced%20Parameter%20Tuning](http://wiki.ros.org/rtabmap_ros/Tutorials/Advanced Parameter Tuning)
32. (PDF) Real-Time Localization for an AMR Based on RTAB-MAP - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/389428095_Real-Time_Localization_for_an_AMR_Based_on_RTAB-MAP
33. Comparison of SLAM algorithms on omnidirectional four wheel mobile robot, accessed July 31, 2025, https://www.etran.rs/2022/zbornik/ICETRAN-22_radovi/082-ROI1.5.pdf
34. rtabmap_ros - ROS Wiki, accessed July 31, 2025, http://wiki.ros.org/rtabmap_ros
35. Dense Mapping From an Accurate Tracking SLAM, accessed July 31, 2025, https://dds.sciengine.com/cfs/files/pdfs/view/2329-9266/CE9E014F63994A3DB03A68B9AC19F68C.pdf
36. An evaluation of ROS-compatible stereo visual SLAM methods on a nVidia Jetson TX2, accessed July 31, 2025, https://www.researchgate.net/publication/332216227_An_evaluation_of_ROS-compatible_stereo_visual_SLAM_methods_on_a_nVidia_Jetson_TX2
37. Comparative Study on Simulated Outdoor Navigation for Agricultural Robots - MDPI, accessed July 31, 2025, https://www.mdpi.com/1424-8220/24/8/2487
38. Improving SLAM Techniques with Integrated Multi-Sensor Fusion for 3D Reconstruction - PMC - PubMed Central, accessed July 31, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014387/
39. Robotics Vision: Simultaneous Localization and Mapping (SLAM) in Dynamic Environments, accessed July 31, 2025, https://eureka.patsnap.com/article/robotics-vision-simultaneous-localization-and-mapping-slam-in-dynamic-environments
40. Optimization of visual SLAM in dynamic environments using object ..., accessed July 31, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13486/134860J/Optimization-of-visual-SLAM-in-dynamic-environments-using-object-detection/10.1117/12.3055716.full
41. Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey - arXiv, accessed July 31, 2025, https://arxiv.org/html/2505.12384v1
42. [Literature Review] Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey - Moonlight | AI Colleague for Research Papers, accessed July 31, 2025, https://www.themoonlight.io/en/review/is-semantic-slam-ready-for-embedded-systems-a-comparative-survey
43. Semantic SLAM - velog, accessed July 31, 2025, https://velog.io/@thkweon/Semantic-SLAM
44. [2001.01028] Visual Semantic SLAM with Landmarks for Large-Scale Outdoor Environment, accessed July 31, 2025, https://arxiv.org/abs/2001.01028
45. NeRF: Neural Radiance Field in 3D Vision: A Comprehensive Review - arXiv, accessed July 31, 2025, https://arxiv.org/html/2210.00379v6
46. How NeRFs and 3D Gaussian Splatting are Reshaping SLAM: a Survey - arXiv, accessed July 31, 2025, https://arxiv.org/html/2402.13255v1
47. GS-SLAM: Dense Visual SLAM with 3D Gaussian Splatting - CVF Open Access, accessed July 31, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Yan_GS-SLAM_Dense_Visual_SLAM_with_3D_Gaussian_Splatting_CVPR_2024_paper.pdf


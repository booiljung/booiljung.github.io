# 3D 커버리지 경로 계획(CPP)
[경로 계획 (Path Planning)](./index.md)


3D 커버리지 경로 계획(Coverage Path Planning, CPP)은 로봇 공학의 핵심적인 문제 중 하나로, 주어진 3차원 작업 공간 내의 모든 지점이나 표면을 로봇의 센서 또는 작업 도구가 누락 없이, 그리고 가능한 한 중복을 최소화하며 방문하기 위한 최적의 경로를 생성하는 과업으로 정의된다.1 이는 단순히 지정된 시작점에서 목표점까지의 경로를 찾는 점대점(point-to-point) 경로 계획과는 근본적으로 구별된다. 점대점 계획이 '어떻게 도달할 것인가'에 초점을 맞춘다면, 커버리지 계획은 '어떻게 전체를 훑을 것인가'라는, '영역' 또는 '체적' 전체를 대상으로 하는 보다 포괄적인 질문에 답해야 한다.4 이 본질적인 차이로 인해 CPP는 훨씬 더 복잡한 제약 조건과 최적화 목표를 다루게 된다.

3D CPP의 성공 여부는 여러 핵심 목표와 평가 지표를 통해 측정된다. 가장 기본적인 요구사항은 **완전성(Completeness)**으로, 목표 영역의 100% 커버리지를 보장해야 한다.2 다음으로 **효율성(Efficiency)**은 총 경로 길이, 작업 완료 시간, 그리고 특히 배터리로 구동되는 이동 로봇에게 치명적인 에너지 소비량을 최소화하는 것을 목표로 한다.6 최근 연구에서는 단순히 경로 길이를 줄이는 것을 넘어, 가속과 감속이 빈번하게 발생하는 경로의 회전(turn) 횟수를 줄이는 것이 에너지 효율성에 지대한 영향을 미친다는 점이 밝혀지면서 핵심적인 최적화 지표로 부상하고 있다.8 마지막으로 **최적성(Optimality)**은 주어진 제약 조건 하에서 정의된 비용 함수(예: 경로 길이, 에너지, 시간의 가중 합)를 최소화하는 경로를 찾는 것을 의미한다.10 하지만 CPP 문제는 조합 최적화 문제의 대표적인 난제인 외판원 문제(Traveling Salesman Problem, TSP)를 일반화한 형태로, 본질적으로 NP-hard 문제로 알려져 있다.4 따라서 대규모의 복잡한 환경에서는 전역 최적해(globally optimal solution)를 찾는 것이 현실적으로 불가능하며, 대신 만족스러운 준최적해(near-optimal solution)를 효율적으로 찾는 휴리스틱 또는 학습 기반 접근법이 널리 연구되고 있다.

이러한 3D CPP 기술의 중요성은 광범위한 응용 분야에서 명확하게 드러난다. **무인 항공기(Unmanned Aerial Vehicles, UAVs)**는 정밀 농업에서의 작황 모니터링 및 방제, 재난 지역 감시, 실종자 수색 및 구조, 3D 지도 제작 및 도시 매핑, 통신 중계기 역할 등 거의 모든 항공 임무의 기반 기술로 CPP를 활용한다.11 **자율 수중 탐사체(Autonomous Underwater Vehicles, AUVs)**는 인간의 접근이 극히 어려운 심해 환경에서 해저 지형 매핑, 기뢰 탐색, 해양 생태계 조사, 수중 구조물 검사 등을 수행하는 데 필수적인 기술이다.8 한편, **지상 로봇(Ground Robots)**은 대규모 건설 현장의 진행 상황을 3D 모델과 비교하며 모니터링하거나, 건물 내부를 자율적으로 청소 및 소독하고, 물류 창고 자동화와 같은 다양한 지상 임무에 CPP 기술을 적용하고 있다.2

본 보고서는 이처럼 다방면에 걸쳐 핵심적인 역할을 수행하는 3D CPP 기술을 체계적으로 조망하고 심층적으로 분석하는 것을 목표로 한다. 보고서는 총 4부로 구성된다. 제1부에서는 CPP 연구의 근간을 이루는 셀 분해, 샘플링, 그래프 기반의 고전적 알고리즘들을 탐구하고, 이들이 어떻게 현대적 문제 해결의 이론적 토대를 마련했는지 분석한다. 제2부에서는 최근 가장 활발하게 연구되는 강화학습(Reinforcement Learning)을 중심으로 한 현대적 패러다임을 심층적으로 다루며, 문제 정의 방식부터 최신 알고리즘 동향까지를 고찰한다. 제3부에서는 모든 CPP 시스템이 직면하는 공통적인 과제인 에너지 효율성 최적화와 다중 로봇 시스템 운용 문제를 심도 있게 분석한다. 마지막으로 제4부에서는 앞서 언급된 주요 응용 분야별(정밀 농업, 수중 탐사, 산업 자동화) 특수한 환경과 요구사항에 맞춰 CPP 기술이 어떻게 변형되고 발전하고 있는지를 구체적인 사례를 통해 살펴본다. 이러한 체계적인 분석을 통해 본 보고서는 3D CPP 기술의 과거와 현재를 종합적으로 이해하고, 미래 연구가 나아가야 할 방향을 제시하고자 한다.


이 장에서는 3D 커버리지 경로 계획의 근간을 이루는 고전적인 알고리즘들을 분석한다. 이 방법론들은 수십 년간 연구되며 발전해왔으며, 후대의 하이브리드 및 학습 기반 접근법에 중요한 이론적 토대와 문제 해결의 단초를 제공했다는 점에서 그 중요성이 크다. 이들은 크게 셀 분해, 샘플링, 그래프 기반, 그리고 휴리스틱 접근법으로 나눌 수 있다.


셀 분해(Cellular Decomposition) 기반 접근법은 복잡한 형태의 자유 공간(free space)을 '셀(cell)'이라고 불리는, 기하학적으로 단순하고 서로 겹치지 않는 영역들의 집합으로 분해하는 것에서 시작한다. 각 셀은 구조가 단순하기 때문에 왕복 운동(back-and-forth motion)과 같은 간단한 패턴으로 쉽게 커버할 수 있다. 모든 셀을 커버한 뒤, 이 셀들을 효율적인 순서로 방문하면 전체 영역에 대한 커버리지가 완료된다.2 이는 복잡한 문제를 여러 개의 작은 문제로 나누어 해결하는 '분할 정복(divide and conquer)' 전략의 전형적인 예시다.


부스트로피돈 분해는 '소가 밭을 갈듯이' 왕복하며 움직이는 모습에서 유래한 이름으로, 셀 분해 방식 중 가장 널리 알려진 기법 중 하나이다.20 이 알고리즘의 핵심 원리는 가상의 수직선인 스캔 라인(sweep line)을 작업 공간의 한쪽 끝에서 다른 쪽 끝으로 점진적으로 이동시키는 것이다. 이동 과정에서 스캔 라인이 장애물과 만나거나 떨어지면서 스캔 라인과 장애물의 교차점 개수가 변하는 지점이 발생하는데, 이를 '임계점(critical point)'이라고 정의한다.20

이러한 임계점들은 셀의 생성과 소멸을 유발하는 '이벤트'를 발생시킨다. 예를 들어, 스캔 라인이 오목한 장애물로 진입하여 하나의 연결된 자유 공간이 두 개로 나뉘는 지점은 `IN` 이벤트로, 반대로 장애물을 빠져나와 두 개의 자유 공간이 하나로 합쳐지는 지점은 `OUT` 이벤트로 정의된다.20 이러한 이벤트를 기준으로 공간을 분할하면, 각 셀 내부에서는 스캔 라인의 연결성이 동일하게 유지된다. 초기에는 이러한 이벤트를 기반으로 공간을 사다리꼴 형태의 셀들로 나누는 '사다리꼴 분해'가 제안되었으나, 부스트로피돈 분해는 

`IN` 이벤트와 `OUT` 이벤트 사이의 여러 사다리꼴 셀들을 하나의 큰 셀로 통합함으로써 불필요한 셀 생성을 억제하고 전체 셀의 개수를 줄여 효율성을 크게 향상시켰다.20

초기의 부스트로피돈 분해는 2D 평면에 국한되었지만, 연구가 발전하면서 3D 곡면으로 그 원리가 확장되었다.23 더 나아가 최근 연구는 단순히 공간을 분해하는 것을 넘어, 생성된 각 셀을 최적으로 커버하고 연결하는 문제로 초점을 옮기고 있다. 예를 들어, 하나의 셀에 대해서도 가로 방향, 세로 방향 등 여러 가능한 스윕 패턴(sweep pattern)을 생성할 수 있다. 이 경우, 문제는 "각 셀에 대해 어떤 스윕 패턴을 선택하고, 어떤 순서로 셀들을 방문해야 전체 경로 비용이 최소화되는가?"로 발전한다. 이는 각 셀(클러스터)에서 정확히 하나의 스윕 패턴(노드)을 선택하여 모든 셀을 방문하는 최단 경로를 찾는 문제로, **일반화된 외판원 문제(Generalized Traveling Salesman Problem, GTSP)**로 공식화하여 해결할 수 있다.25

이러한 접근 방식의 변화는 셀 분해 기법이 초기의 순수한 기하학적 분할 도구에서, 복잡한 조합 최적화 문제를 풀기 위한 '전처리 단계'로 진화하고 있음을 명확히 보여준다. 즉, CPP 문제를 '공간 분할' 문제에서 '경로 순서 및 패턴 최적화'라는 더 고차원적인 문제로 재정의하는 패러다임의 전환이 일어난 것이다. 초기 알고리즘에서 최적화의 대상이 '셀의 수'였다면, 이제는 분해된 셀들을 기반으로 구성된 훨씬 더 복잡한 의사결정 공간에서의 '최적의 조합'을 찾는 것으로 문제가 심화되었다. 이는 CPP가 단순한 기하학적 알고리즘을 넘어, 운영 연구(Operations Research) 및 최적화 이론과 깊이 융합되고 있음을 시사한다. 분해는 더 이상 해답 그 자체가 아니라, 더 정교한 최적화 알고리즘을 적용하기 위한 문제의 구조화 과정이 되었다.


웨이브프론트 알고리즘은 격자(grid)로 표현된 환경에서 최단 경로를 찾는 고전적인 방법으로, 셀 분해의 한 형태로 볼 수 있다. 이 알고리즘은 목표 지점(Goal)에서 시작하여 마치 물에 돌을 던졌을 때 퍼져나가는 파동(wave)처럼 주변 셀로 전파하며 각 셀에 목표까지의 거리(비용) 값을 할당한다.10 이 과정은 그래프 탐색 기법 중 너비 우선 탐색(Breadth-First Search, BFS)과 동일한 원리로 작동한다. 모든 도달 가능한 셀에 거리 값이 할당되고 나면, 시작 지점(Start)에서 현재 위치 주변의 셀들 중 가장 낮은 값(가장 가파른 경사)을 따라 이동하는 과정을 반복하면 자연스럽게 목표 지점까지의 최단 경로가 생성된다.

이 단순하고 직관적인 원리는 2D 환경뿐만 아니라 3D 격자 환경으로도 자연스럽게 확장될 수 있다. 최근 연구에서는 3D 실내 공간 탐색 27이나, 복잡한 환경의 구조적 특징을 효과적으로 포착하기 위한 위상학적 그래프, 즉 스켈레톤(skeleton)을 추출하는 기반 기술로 웨이브프론트 알고리즘이 활용되고 있다.28 하지만 이 알고리즘은 환경에 대한 전체 정보가 사전에 주어져야 하는 오프라인(off-line) 방식이며, 장애물이 정적인 환경에서 가장 효과적이다. 만약 장애물이 움직이는 동적 환경에 적용할 경우, 환경이 변할 때마다 파동 전파 과정을 처음부터 다시 계산해야 하므로 비효율적일 수 있다는 명확한 한계를 가진다.29


샘플링 기반 경로 계획(Sampling-Based Path Planning)은 로봇의 설정 공간(Configuration Space, C-space)이 고차원이고 장애물 등으로 인해 형태가 복잡하여 명시적으로 탐색하기 어려울 때 매우 효과적인 방법론이다. 이 접근법은 전체 공간을 결정론적으로 분해하거나 탐색하는 대신, 설정 공간 내에서 무작위로 점들을 샘플링하고 이들을 연결하여 확률적으로 경로를 찾아 나간다.29


샘플링 기반 방법론의 대표적인 알고리즘으로는 RRT(Rapidly-exploring Random Tree)가 있다. RRT는 시작점에서부터 하나의 트리(tree)를 점진적으로 확장해 나가는 방식으로 작동한다. 매 단계마다 설정 공간에서 무작위로 점을 샘플링하고, 트리에 있는 노드 중 샘플링된 점과 가장 가까운 노드를 찾아 그 방향으로 일정 거리만큼 새로운 가지를 뻗어 트리를 성장시킨다. 이 과정을 통해 트리는 미지의 공간을 향해 빠르게 뻗어 나가며 효율적으로 공간을 탐색한다.31

RRT의 수렴 속도를 높이기 위해 RRT-Connect와 같은 변형 알고리즘도 제안되었다. RRT-Connect는 시작점과 목표점에서 동시에 두 개의 트리를 확장하여 중간에서 만나게 함으로써 단일 트리를 사용하는 것보다 훨씬 빠르게 경로를 찾는 경향이 있다.31 한편, RRT와 RRT-Connect는 경로를 찾는 즉시 탐색을 종료하기 때문에 찾아낸 경로의 품질(예: 경로 길이)이 최적이 아닐 수 있다. 이러한 단점을 보완하기 위해 RRT* 알고리즘이 개발되었다. RRT*는 경로가 발견된 후에도 샘플링을 계속하며, '재연결(rewiring)'이라는 과정을 통해 주변 노드들을 더 짧은 경로로 연결하는 작업을 반복한다. 이 과정을 통해 RRT*는 샘플 수가 무한대에 가까워짐에 따라 점근적으로 최적 경로(asymptotically optimal path)로 수렴하는 특성을 가진다.30


RRT와 그 변형 알고리즘들은 본래 점대점 경로 탐색을 위해 설계되었기 때문에, '영역'을 커버하는 CPP 문제에 직접 적용하기는 어렵다. 이 근본적인 차이를 극복하기 위해, 연구자들은 CPP 문제를 두 개의 하위 문제로 재구성하는 체계적인 프레임워크를 제안했다.30 이는 샘플링 기반 방법론을 CPP에 성공적으로 적용한 핵심적인 전환으로, 해결하기 어려운 '영역 커버' 문제를 잘 알려진 '다중 목표점 순회' 문제로 변환하는 전략이다.

1. **1단계: 커버리지 샘플링 문제 (Coverage Sampling Problem, CSP):** 첫 번째 단계는 '영역 커버'라는 연속적인 문제를 이산적인 문제로 변환하는 것이다. 로봇의 센서가 특정 위치와 방향에 있을 때 감지할 수 있는 영역(footprint)을 고려하여, 목표 구조물의 모든 부분을 100% 커버할 수 있는 유한한 개수의 로봇 설정(configuration) 집합을 무작위 샘플링을 통해 찾는다. 이 설정들은 커버리지 경로를 구성하는 '중간 목표점(intermediate goals)'이 된다.
2. **2단계: 다중 목표 계획 문제 (Multi-goal Planning Problem, MPP):** CSP 단계에서 찾은 목표점 집합이 주어지면, 문제는 "이 목표점들을 모두 방문하는 가장 효율적인 경로(tour)는 무엇인가?"로 바뀐다. 이는 전형적인 다중 목표점 순회 문제이며, 외판원 문제(TSP)와 매우 유사하다. 이 단계에서는 RRT나 PRM(Probabilistic Roadmap)과 같은 샘플링 기반 플래너를 사용하여 목표점들 간의 충돌 없는 연결 경로를 생성하고, TSP 솔버 등을 이용해 최적의 방문 순서를 결정한다.

이러한 문제 재구성 전략은 복잡한 CPP 문제를 이미 해결책이 존재하는 더 작은 문제들로 분할함으로써 전체 문제의 해결 가능성과 효율성을 높인다. 또한, 각 단계(CSP, MPP)를 독립적으로 개선할 수 있어 전체 시스템의 모듈성과 확장성을 크게 향상시킨다. 예를 들어, `30`에서는 RRT*를 변형한 `RRT*||` 알고리즘을 제안하여, 이미 생성된 커버리지 경로를 국소적으로 개선(local improvement)하는 데 사용한다. 이는 기존의 목표점을 더 짧은 경로를 생성하는 새로운 목표점으로 대체하는 방식으로 작동하며, MPP 단계에서 경로의 질을 점진적으로 향상시키는 역할을 한다.


그래프 기반 경로 계획은 작업 환경을 노드(node)와 엣지(edge)로 구성된 그래프로 추상화하고, 잘 알려진 그래프 탐색 알고리즘을 이용해 경로를 찾는 방식이다. 이 접근법은 앞서 설명한 셀 분해 방식과 밀접한 관련이 있다. 예를 들어, 환경을 격자 셀로 분해한 뒤 각 셀의 중심을 노드로, 인접한 셀 간의 이동 가능성을 엣지로 표현하여 그래프를 구축할 수 있다.32


신장 트리 커버리지(STC)는 그래프 기반 CPP의 대표적인 알고리즘이다. STC의 기본 원리는 다음과 같다. 먼저, 작업 환경을 일정한 크기의 격자(grid)로 분해한다. 그런 다음, 장애물이 없는 자유 공간 셀들을 노드로 간주하여 그래프를 생성한다. 이 그래프의 모든 노드를 연결하면서 사이클(cycle)이 없는 부분 그래프인 신장 트리(Spanning Tree), 특히 엣지 가중치의 합이 최소가 되는 최소 신장 트리(Minimum Spanning Tree, MST)를 찾는다. 마지막으로, 로봇이 이 MST의 외곽을 마치 '벽을 따라가듯이(wall-following)' 이동하면, 모든 셀을 중복 방문 없이 한 번씩 지나가는 완전한 커버리지 경로가 생성된다.34

STC의 가장 큰 장점은 알고리즘이 매우 단순하고 계산 효율성이 뛰어나다는 점이다. 전체 계산 복잡도는 환경을 구성하는 셀의 개수 $N$에 대해 선형 시간($O(N)$)에 비례하므로, 대규모 환경에서도 매우 빠르게 경로를 생성할 수 있으며, 완전한 커버리지를 수학적으로 보장한다.34

그러나 이러한 단순함과 효율성은 종종 경로의 질(quality)을 희생하는 대가로 얻어진다. STC로 생성된 경로의 효율성은 전적으로 MST의 형태에 따라 결정된다. 만약 무작위로 생성된 MST가 길고 가는 가지를 많이 포함하는 '나쁜' 형태라면, 로봇은 이 가지들을 따라 계속해서 들어갔다 나오는 왕복 운동을 반복해야 하며, 이는 수많은 불필요한 회전과 비효율적인 경로를 유발한다.9 반면, MST가 길고 곧은 '벽돌(brick)'과 같은 구조들로 주로 구성된 '좋은' 형태라면, 로봇은 훨씬 부드럽고 효율적인 경로로 이동할 수 있다.

이러한 관찰은 "모든 MST가 동일한 품질의 경로를 생성하지 않는다"는 중요한 사실을 시사한다. 따라서 연구의 초점은 단순히 '아무 MST나 찾기'에서 '경로 비용(특히 회전 수)을 최소화하는 이상적인 형태의 MST를 어떻게 구축할 것인가'라는 더 어려운 최적화 문제로 이동하고 있다. 예를 들어, TMSTC*(Turn-minimizing Multirobot Spanning Tree Coverage Star)와 같은 개선된 알고리즘은 '벽돌'이라는 구조적 요소를 먼저 식별하고, 이들을 탐욕 알고리즘(greedy algorithm)으로 연결하여 회전 수를 최소화하는 신장 트리를 구성하는 전략을 제안한다.9 이는 STC의 단순성과 효율성을 유지하면서도 생성된 경로의 질을 개선하려는 노력으로, 고전적 알고리즘 내에서도 심화된 최적화가 활발히 이루어지고 있음을 보여주는 좋은 예다.

STC는 또한 다중 로봇 커버리지(Multi-robot Coverage Path Planning, MCPP)로 쉽게 확장될 수 있다. 생성된 단일 신장 트리를 여러 로봇에게 적절히 분할하여 할당하거나 9, 각 로봇이 자신의 커버리지 영역을 점진적으로 확장하며 분산된 방식으로 자신만의 신장 트리를 구축하는 방법(예: AWSTC) 등이 연구되고 있다.36


3D CPP는 본질적으로 NP-hard 문제이기 때문에, 문제의 규모가 커지면 고전적인 결정론적 알고리즘으로 최적해를 찾는 것이 계산적으로 불가능에 가까워진다. 이러한 상황에서 휴리스틱(Heuristic) 및 생체 모방(Bio-inspired) 알고리즘은 엄밀한 최적성을 보장하는 대신, 자연 현상이나 생물의 집단행동에서 영감을 얻어 짧은 시간 안에 만족스러운 수준의 준최적해(near-optimal solution)를 찾는 실용적인 대안을 제공한다.14

`12`과 `12`의 체계적인 분류에 따르면, UAV 경로 계획에 사용되는 휴리스틱 알고리즘은 다음과 같이 나눌 수 있다.

- **생체 모방 알고리즘 (Bio-inspired Algorithms):** 이들은 자연계의 진화, 집단 지능, 면역 체계 등에서 영감을 얻은 최적화 기법들이다.
  - **유전 알고리즘 (Genetic Algorithm, GA):** 생물의 진화 과정을 모방하여, 해(경로)를 유전자로 표현하고 선택, 교차, 돌연변이 연산을 통해 더 좋은 해를 탐색한다. 주로 경로점들의 방문 순서를 최적화하는 TSP 문제 해결에 널리 사용된다.38
  - **입자 군집 최적화 (Particle Swarm Optimization, PSO):** 새 떼나 물고기 떼의 움직임을 모방한다. 각 입자(해)는 자신의 경험과 집단 전체의 경험을 바탕으로 해 공간을 탐색하며 최적해를 찾아간다.12
  - **개미 군집 최적화 (Ant Colony Optimization, ACO):** 개미들이 페로몬을 통해 최단 경로를 찾는 원리를 이용한다. 개미(에이전트)들이 경로를 탐색하며 페로몬을 남기고, 다른 개미들은 페로몬 농도가 높은 경로를 선택할 확률이 높아지면서 점차 최적 경로가 강화된다.12
  - **기타:** 회색 늑대 최적화(Gray Wolf Optimizer, GWO), 생물지리학 기반 최적화(Biogeography-based Optimization, BBO) 등 다양한 알고리즘들이 제안되었다.12
- **기타 휴리스틱 알고리즘:** A*나 Dijkstra와 같은 그래프 탐색 알고리즘은 최단 경로를 보장하지만, 휴리스틱 함수를 사용하여 탐색을 안내한다는 점에서 넓은 의미의 휴리스틱으로 분류될 수 있다. 인공 포텐셜 필드(Artificial Potential Field, APF)는 목표점은 인력(attractive force)을, 장애물은 척력(repulsive force)을 발생시키는 가상의 필드를 생성하여 로봇을 안내하는 기법이다.12

이러한 다양한 휴리스틱 알고리즘의 존재는 "모든 문제에 대해 항상 최고의 성능을 내는 단일 최적화 알고리즘은 없다"는 '공짜 점심 없음(No-Free-Lunch)' 정리를 실제로 보여준다. 각 알고리즘은 해 공간의 넓은 영역을 탐색하는 능력(exploration)과 이미 찾은 좋은 해 주변을 정밀하게 파고드는 능력(exploitation) 사이의 트레이드오프에서 서로 다른 강점과 약점을 가진다.

이러한 한계를 극복하기 위해 최근에는 여러 알고리즘을 지능적으로 결합하는 **하이브리드(Hybrid) 접근법**이 부상하고 있다. 이는 단순히 두 알고리즘을 섞는 것을 넘어, 각 알고리즘의 장점이 가장 잘 발휘될 수 있는 단계에 선택적으로 적용함으로써 시너지를 창출하는, 한 단계 더 높은 수준의 최적화 전략이다. 대표적인 예로 `[39]`에서 제안된 **적응형 하이브리드 PSO-ACO 플래너**가 있다. 이 플래너는 문제 해결 과정을 동적으로 분석하여 알고리즘을 전환한다.

1. **초기 탐색 단계:** 해 공간에 대한 정보가 없는 초기에는, 전역 탐색(global exploration) 능력이 뛰어나고 수렴 속도가 빠른 PSO를 사용하여 유망한 해가 존재할 가능성이 높은 영역으로 신속하게 이동한다.
2. **후기 정제 단계:** 탐색이 어느 정도 진행되어 유망한 영역이 특정되면, 지역 탐색(local exploitation) 능력이 뛰어난 ACO로 전환한다. ACO는 페로몬 메커니즘을 통해 해당 영역 내에서 해를 정교하게 다듬고 지역 최적점(local minima)에서 탈출하는 데 도움을 준다.

이처럼 문제 해결의 상태에 따라 가장 적합한 도구를 동적으로 선택하는 하이브리드 방식은 단일 알고리즘의 한계를 극복하고 더 강건하고 효율적인 해를 찾는 강력한 전략으로 자리 잡고 있다.


| 패러다임 (Paradigm)        | 핵심 원리 (Core Principle)             | 완전성 보장 (Completeness)               | 최적성 (Optimality)                              | 계산 복잡도 (Complexity)       | 주요 장점 (Advantages)                    | 주요 단점 (Disadvantages)                                 |
| -------------------------- | -------------------------------------- | ---------------------------------------- | ------------------------------------------------ | ------------------------------ | ----------------------------------------- | --------------------------------------------------------- |
| **셀 분해 (Cellular)**     | 공간을 단순한 셀로 분할 후 순회        | 결정론적/완전 (Deterministic/Complete)   | 보장 안 됨 (경로 순서 별도 최적화 필요)          | 환경 복잡도에 의존 (다항 시간) | 구조화된 커버리지, 복잡한 경계 처리 용이  | 비효율적 경로(많은 회전) 생성 가능성                      |
| **샘플링 기반 (Sampling)** | 설정 공간의 무작위 샘플링 및 연결      | 확률적 완전 (Probabilistically Complete) | 점근적 최적 (Asymptotically Optimal, e.g., RRT*) | 샘플 수에 비례                 | 고차원/복잡한 공간 처리 용이, 장애물 회피 | 최적성 보장에 많은 샘플 필요, 경로가 불규칙함             |
| **그래프 기반 (Graph)**    | 환경을 그래프로 변환 후 신장 트리 순회 | 결정론적/완전 (Deterministic/Complete)   | 보장 안 됨 (트리 형태에 따라 비효율적)           | 선형 시간 ($O(N)$)             | 계산 속도가 매우 빠름, 완전 커버리지 보장 | 경로 품질이 낮을 수 있음, 회전 최소화 별도 고려 필요      |
| **생체 모방 휴리스틱**     | 자연계의 최적화 과정 모방              | 보장 안 됨 (Heuristic)                   | 보장 안 됨 (준최적 해)                           | 인구 크기 및 세대 수에 비례    | 복잡한 최적화 목표 처리, 유연성 높음      | 수렴 속도 느림, 파라미터 튜닝 필요, 전역 최적해 보장 불가 |


고전적 알고리즘들이 명시적인 규칙과 모델에 기반하여 경로를 생성하는 반면, 현대적인 접근법은 데이터와 경험으로부터 최적의 행동 정책을 학습하는 방향으로 진화하고 있다. 그 중심에는 **강화학습(Reinforcement Learning, RL)**이 있다. RL은 로봇(에이전트)이 환경과 상호작용하며 얻는 보상(reward)을 최대화하는 방향으로 시행착오를 통해 스스로 행동 전략을 학습하는 패러다임이다.4 이는 사전에 모든 환경 정보를 알 수 없거나, 환경이 동적으로 변하거나, 최적화 목표가 복잡하여 수학적으로 모델링하기 어려운 CPP 문제에 강력한 해결책을 제시한다.


강화학습을 CPP에 적용하기 위해서는 먼저 문제를 **순차적 마르코프 결정 과정(Markov Decision Process, MDP)**으로 공식화해야 한다. MDP는 $(S, A, P, R, γ)$의 튜플로 정의된다.4

- **상태 공간 (State Space, S):** 에이전트가 관찰할 수 있는 환경의 모든 정보를 포함한다. 3D CPP 문제에서 상태 $s_t$는 일반적으로 다음 요소들을 포함한다 40:
  - UAV의 3차원 위치 $p_t = (x_t, y_t, z_t)$
  - 카메라의 방향 및 줌 설정 $θ_t = (pan, tilt, zoom)$
  - 정규화된 배터리 잔량 $b_t$
  - 커버리지 맵(어느 영역이 커버되었는지) 정보
- **행동 공간 (Action Space, A):** 에이전트가 각 상태에서 취할 수 있는 모든 행동의 집합이다. 일반적으로 이산적인 행동 공간이 사용되며, 다음과 같이 구성될 수 있다 40:
  - 이동 행동 `Amove`: 상, 하, 좌, 우, 전, 후 등 단위 이동
  - 카메라 제어 행동 `Acam`: 팬, 틸트, 줌 조절
- **전이 함수 (Transition Function, P):** 특정 상태 $s_t$에서 특정 행동 $a_t$를 취했을 때, 다음 상태 $s_{t+1}$로 전이될 확률 분포 $P(s_{t+1} | s_t, a_t)$를 정의한다. 실제 로봇 시스템에서는 이 함수를 명시적으로 알지 못하며, 에이전트는 환경과의 상호작용을 통해 이를 암묵적으로 학습한다.
- **보상 함수 (Reward Function, R):** 강화학습의 핵심으로, 에이전트의 행동을 유도하는 신호 역할을 한다. 상태 $s_t$에서 행동 $a_t$를 취해 상태 $s_{t+1}$로 전이했을 때 받는 즉각적인 보상 $r_{t+1}$을 정의한다. CPP 문제의 복잡한 목표를 달성하기 위해 보상 함수는 여러 요소의 가중 합으로 설계되는 경우가 많다.3
  - **커버리지 보상:** 새로운 영역을 커버했을 때 양의 보상 ($λ_c / ∆C_t$)
  - **효율성 패널티:** 시간 경과나 에너지 소모에 대한 음의 보상 ($λ_b / f_{battery}(b_{t+1})$)
  - **중복 방문 패널티:** 이미 커버한 영역을 다시 방문했을 때 음의 보상
  - **충돌 패널티:** 장애물과 충돌했을 때 큰 음의 보상
  - **탐험 보상 (Curiosity):** 방문 빈도가 낮은 지역을 방문하도록 유도하는 보상
- **할인율 (Discount Factor, γ):** $γ \in $ 사이의 값으로, 미래에 받을 보상의 현재 가치를 결정한다. $γ$가 0에 가까우면 에이전트는 즉각적인 보상에 집중하고, 1에 가까우면 장기적인 보상을 더 중요하게 고려한다.

강화학습의 목표는 누적 할인 보상(cumulative discounted reward)의 기댓값, 즉 다음을 최대화하는 최적의 정책(policy) $π(a|s)$를 찾는 것이다. 정책은 주어진 상태에서 어떤 행동을 선택할지에 대한 전략이다.
$$
J(\pi) = E_{\pi}[\sum_{t=0}^{\infty} \gamma^t r_{t+1}]
$$


CPP 문제 해결을 위해 다양한 RL 알고리즘이 연구되고 있으며, 특히 심층 강화학습(Deep Reinforcement Learning, DRL)이 주목받고 있다. DRL은 심층 신경망(Deep Neural Network, DNN)을 사용하여 정책이나 가치 함수를 근사함으로써 고차원의 상태 및 행동 공간을 처리할 수 있다.

- **PPO (Proximal Policy Optimization):** 현재 가장 널리 사용되는 정책 경사(Policy Gradient) 계열 알고리즘 중 하나다. PPO는 정책을 업데이트할 때 이전 정책과 새로운 정책의 차이가 너무 커지지 않도록 '클리핑(clipping)'이라는 기법을 사용하여 학습 안정성을 크게 높인다.40 이는 복잡한 CPP 환경에서 안정적인 학습을 가능하게 하는 중요한 특징이다.
- **SAC (Soft Actor-Critic):** 연속적인 행동 공간(continuous action space)을 다루는 데 효과적인 액터-크리틱(Actor-Critic) 계열 알고리즘이다. SAC의 핵심 특징은 '최대 엔트로피(maximum entropy)' 최적화다. 이는 보상을 최대화하는 동시에 정책의 엔트로피, 즉 무작위성을 최대한 유지하도록 장려한다. 이를 통해 에이전트는 더 활발하게 탐험(exploration)을 수행하고, 다양한 해결책을 찾을 가능성을 높이며, 지역 최적점에 빠지는 것을 방지한다.4


최근 RL 기반 CPP 연구는 단순한 경로 생성을 넘어, 더 복잡한 제약 조건과 현실적인 문제를 해결하는 방향으로 빠르게 발전하고 있다.


가장 혁신적인 연구 방향 중 하나는 대규모 언어 모델(Large Language Model, LLM)의 추론 능력을 강화학습에 접목하는 것이다. **프롬프트 기반 강화학습(Prompt-Informed Reinforcement Learning, PIRL)**은 이러한 시도의 대표적인 예다.40 기존 RL의 보상 함수는 특정 환경과 작업에 고정되어 의미론적 적응성이 부족했다. PIRL은 이 문제를 해결하기 위해 LLM(예: GPT-3.5)을 '소프트 태스크 플래너(soft-task planner)'로 활용한다.

PIRL의 작동 방식은 다음과 같다:

1. **구조화된 프롬프트 생성:** 매 학습 단계에서 현재 UAV의 상태(위치, 카메라, 배터리, 커버리지 현황 등)와 작업 목표를 포함하는 구조화된 텍스트 프롬프트를 생성한다.
2. **LLM의 제안 생성:** 이 프롬프트를 LLM에 입력하면, LLM은 자신의 방대한 사전 지식과 문맥 이해 능력을 바탕으로 "다음 단계에서 UAV가 어디로 이동하고 카메라를 어떻게 조절하면 좋을지"에 대한 상식적이고 전략적인 제안을 생성한다.
3. **보상 신호 조절 (Reward Shaping):** **프롬프트 적응형 보상 엔진(Prompt-Adaptive Reward Engine, PARE)**이라는 모듈이 RL 에이전트의 실제 행동과 LLM의 제안 사이의 편차를 계산한다. 에이전트의 행동이 LLM의 제안과 유사할수록 추가적인 양의 보상을, 벗어날수록 음의 보상을 부여한다.
4. **정책 학습:** 이 추가적인 보상 신호($f_{LLM}$)가 기존의 RL 보상 함수에 더해져 PPO와 같은 알고리즘으로 정책을 학습시킨다.

이러한 방식은 LLM이 직접 행동을 지시하는 것이 아니라, 보상 신호를 통해 에이전트의 학습 방향을 '부드럽게 안내'한다는 점에서 독창적이다. 이를 통해 에이전트는 정책의 유연성을 유지하면서도, LLM의 의미론적, 전략적 지도를 받아 더 효율적이고 지능적인 커버리지 경로를 학습할 수 있다. 실험 결과, PIRL은 기존 RL 방식에 비해 커버리지, 배터리 효율성, 중복 방문 감소 등 모든 지표에서 월등한 성능을 보였다.40


실제 로봇은 이산적인 격자 위를 움직이는 것이 아니라, 물리 법칙에 따라 부드러운 연속적인 궤적을 따라 움직인다. 이러한 현실을 반영하기 위해, 이산적인 행동 공간을 넘어 연속 공간에서의 CPP를 다루는 연구가 활발히 진행되고 있다.4

이러한 접근법에서는 환경을 축-정렬 직사각형들의 집합으로, UAV의 움직임을 **곡률 제약이 있는 베지에 곡선(curvature-constrained Bézier curves)**으로 모델링한다. 베지에 곡선은 위치, 속도, 곡률의 연속성을 보장하여 부드럽고 동역학적으로 실현 가능한 경로를 생성하는 데 적합하다.

이 문제를 해결하기 위해 **AM-SAC(Action-Mapping-based Soft Actor-Critic)** 알고리즘이 제안되었다.4 AM-SAC은 문제를 두 단계로 나누어 접근한다. 먼저, 주어진 행동이 동역학적 제약(예: 최대 회전 곡률)을 만족하는지 판별하는 '실현 가능성 모델(feasibility model)'을 사전 학습한다. 그런 다음, 본 학습 단계에서는 SAC 에이전트가 이 모델을 통해 '실현 가능한' 행동들 중에서만 최적의 행동을 선택하도록 학습한다. 이는 탐색 공간을 효과적으로 줄여 학습 효율과 안정성을 크게 향상시킨다. 또한, **자기 적응형 커리큘럼(self-adaptive curriculum)**을 도입하여, 에이전트가 처음에는 쉬운 문제(작은 맵)부터 학습하고 성능이 향상됨에 따라 점차 어려운 문제(큰 맵)로 나아가도록 하여 학습 과정을 안정화시킨다.


UAV의 가장 큰 현실적 제약은 제한된 배터리 용량이다. 대규모 영역을 커버해야 할 경우, 한 번의 비행으로 임무를 완수하는 것은 불가능하다. 따라서 커버리지 경로 중간에 재충전소로 돌아가는 여정을 전략적으로 통합하는 것이 필수적이다. 이는 단순히 경로를 계획하는 것을 넘어, "언제, 어떤 상태에서 재충전하러 가야 하는가?"라는 장기적인 전략적 의사결정 문제를 포함한다.41

강화학습은 이러한 장기적 의사결정 문제에 매우 적합하다. RL 에이전트는 현재 배터리 잔량, 목표 영역까지의 거리, 재충전소까지의 거리 등을 종합적으로 고려하여 '지금 커버리지를 계속할 것인가' 아니면 '재충전소로 복귀할 것인가'를 학습할 수 있다. `41`의 연구에서는 PPO 기반의 DRL 접근법을 제안하며, 재충전으로 인해 동일한 상태를 반복적으로 방문할 수 있는 문제(state loops)를 해결하기 위해 에이전트에게 자신의 과거 위치 기록을 제공하고, 장기적인 보상을 효과적으로 학습시키기 위해 할인율을 동적으로 조절하는 기법(discount factor scheduling) 등을 도입하여 이 복잡한 문제를 해결한다.


| 접근법 (Approach)        | 핵심 아이디어 (Core Idea)                | 상태 공간 (State Space)              | 행동 공간 (Action Space)           | 보상 함수 특징 (Reward Features)                        | 해결 과제 (Target Problem)                        |
| ------------------------ | ---------------------------------------- | ------------------------------------ | ---------------------------------- | ------------------------------------------------------- | ------------------------------------------------- |
| **Standard PPO/DDQN** 3  | 심층 신경망을 이용한 정책/가치 함수 근사 | 위치, 커버리지 맵, 환경 정보         | 이산적 이동 (Discrete movement)    | 커버리지 증가, 에너지 소모, 충돌 등 다중 목표의 가중 합 | 일반적인 3D CPP, 대규모 환경으로의 확장성         |
| **PIRL (PPO + LLM)** 40  | LLM의 의미론적 추론을 보상 신호에 통합   | 위치, 카메라, 배터리, 커버리지 맵    | 이산적 이동 및 카메라 제어         | 기본 보상 + LLM 제안과의 편차에 기반한 동적 보상 조절   | 시각적 커버리지 최적화, 의미론적/전략적 경로 계획 |
| **AM-SAC** 4             | 실현 가능한 행동 공간 내에서만 탐색      | 위치, 속도, 곡률, 환경 객체(NFZ, TZ) | 연속적 (베지에 곡선 제어점)        | 에너지(전력) 소모 최소화, 제약 위반 시 패널티           | 연속 공간, 동역학 제약(곡률) 만족 경로 계획       |
| **PPO with Recharge** 41 | 재충전 여정을 커버리지 경로에 통합       | 위치, 커버리지 맵, 배터리, 위치 기록 | 이산적 이동 (커버리지 또는 재충전) | 커버리지 보상 + 재충전 결정에 대한 전략적 보상          | 배터리 제약 하에서의 장기적 임무 계획             |


3D CPP 알고리즘을 현실 세계에 성공적으로 적용하기 위해서는 이론적인 경로 생성을 넘어, 실제 로봇 시스템이 직면하는 물리적, 운영적 제약을 해결해야 한다. 이 장에서는 모든 CPP 시스템에 공통적으로 적용되는 두 가지 핵심 최적화 과제, 즉 **에너지 효율성**과 **다중 로봇 시스템 운용**에 대해 심층적으로 분석한다.


특히 배터리로 구동되는 UAV나 AUV와 같은 이동 로봇에게 에너지 효율성은 임무의 성공 여부를 결정짓는 가장 중요한 요소 중 하나다. 제한된 배터리 용량은 로봇의 작동 시간과 커버할 수 있는 영역의 크기를 직접적으로 제한하기 때문이다.6 따라서 단순히 경로를 찾는 것을 넘어 '가장 적은 에너지로 커버하는 경로'를 찾는 것이 필수적이다.

에너지 소비에 영향을 미치는 주요 요인은 다음과 같다:

- **경로 길이:** 이동 거리가 길수록 더 많은 에너지가 소모된다. 이는 가장 기본적인 최적화 목표다.
- **회전(Turns):** 직선 비행에 비해 회전 시에는 감속, 선회, 재가속 과정에서 훨씬 더 많은 에너지가 소모된다. 따라서 경로 길이는 조금 길어지더라도 회전 수를 최소화하는 것이 전체 에너지 소비 측면에서 더 유리할 수 있다.6
- **속도 및 가속도:** 높은 속도나 급격한 가속/감속은 에너지 소비를 증가시킨다. 따라서 부드러운 속도 프로파일을 유지하는 것이 중요하다.
- **외부 환경 요인:** 바람(UAV의 경우)이나 조류(AUV의 경우)는 로봇의 움직임에 저항으로 작용하여 에너지 소비를 크게 증가시킬 수 있다.

이러한 요인들을 고려하여 에너지 효율적인 CPP를 달성하기 위한 다양한 전략이 연구되고 있다.

1. **정확한 에너지 모델링:** 최적화를 위해서는 먼저 에너지 소비를 정확하게 예측할 수 있는 모델이 필요하다. 초기 모델들은 단순히 이동 거리에 비례한다고 가정했지만, 최근 연구들은 가속, 감속, 회전 기동, 심지어 장애물과의 거리까지 고려하는 정교한 에너지 소비 모델을 제안하고 있다.14 이러한 모델은 시뮬레이션 환경에서 알고리즘의 에너지 효율성을 평가하고 최적화하는 데 기반이 된다.
2. **경로 패턴 및 회전 수 최소화:** 전통적인 왕복(back-and-forth) 패턴은 영역의 경계에서 급격한 180도 회전을 유발하여 에너지 비효율을 초래한다. 이를 해결하기 위해, 다각형 영역의 가장 긴 변에 평행하게 스윕 방향을 설정하여 회전 수를 줄이거나 43, 나선형(spiral) 패턴 또는 이 둘을 혼합한 패턴을 사용하는 연구가 진행되고 있다.44 또한, 신장 트리 커버리지(STC)에서는 회전을 최소화하는 형태의 트리를 생성하는 것이 핵심 연구 주제다.9
3. **최적 비행 속도 결정:** 모든 비행 속도에서 에너지 효율이 동일하지는 않다. 일반적으로 특정 속도에서 단위 거리당 에너지 소비가 최소가 되는 '최적 비행 속도'가 존재한다. `43`의 연구에서는 경로 계획 단계에서 이 최적 비행 속도를 계산하고, 이를 고려하여 경로의 총 에너지 소비를 직접적으로 추정하고 최적화하는 새로운 프레임워크를 제안한다. 이는 단순히 경로 길이를 최소화하는 기존 접근법보다 훨씬 더 현실적인 에너지 최적화를 가능하게 한다.
4. **통합 최적화 프레임워크:** 최신 연구들은 개별적인 최적화를 넘어, 여러 요소를 통합적으로 고려하는 방향으로 나아가고 있다. 예를 들어, **SSDP(Shrinking-rings and Segmentation with Dynamic Programming)** 알고리즘은 볼록 및 비볼록 영역이 혼합된 복잡한 환경에 대해 적용된다.44 이 방법은 영역을 분해하는 대신, 각 영역의 경계에서부터 안쪽으로 축소되는 '링(ring)'들을 생성하고 이들을 순차적으로 연결하여 회전이 적고 부드러운 경로를 만든다. 그런 다음, 동적 프로그래밍(Dynamic Programming)을 사용하여 이 전체 경로를 각 UAV의 에너지 제약에 맞게 최적으로 분할한다. 이는 경로 생성과 에너지 제약 만족을 동시에 고려하는 통합적인 접근법이다.


단일 로봇으로는 커버하기 어려운 대규모 영역이나 복잡한 임무를 효율적으로 수행하기 위해 다중 로봇 시스템(Multi-Robot System)을 활용하는 것은 자연스러운 해결책이다.6 여러 대의 로봇이 협력하여 작업을 분담하면 전체 임무 완료 시간을 크게 단축하고, 한 로봇이 고장 나더라도 다른 로봇이 임무를 이어받을 수 있어 시스템의 강건성(robustness)을 높일 수 있다.5 그러나 다중 로봇 운용은 새로운 차원의 복잡성을 야기하며, 이를 해결하기 위한 **다중 로봇 커버리지 경로 계획(Multi-robot CPP, MCPP)** 기술이 요구된다.

MCPP의 핵심 과제는 크게 두 가지로 나눌 수 있다.

1. **작업 할당 (Task Allocation):** 전체 작업 영역을 여러 개의 하위 영역으로 분할하고, 각 하위 영역을 어떤 로봇에게 할당할 것인지 결정하는 문제다.
2. **개별 경로 계획 (Individual Path Planning):** 각 로봇은 할당받은 자신의 하위 영역에 대해 최적의 커버리지 경로를 계획해야 한다.

이러한 과제를 해결하기 위한 접근법은 크게 중앙 집중형과 분산형으로 나뉜다.

- **중앙 집중형 접근법 (Centralized Approach):** 하나의 중앙 서버나 리더 로봇이 전체 환경 정보와 모든 로봇의 상태를 파악하여, 전역적으로 최적의 작업 분할 및 경로 계획을 수행한 뒤 각 로봇에게 명령을 전달하는 방식이다.
  - **DARP (Divide Areas for Optimal Multi-Robot Coverage Path Planning):** 대표적인 중앙 집중형 알고리즘으로, 전체 영역을 로봇의 수만큼의 하위 다각형으로 분할한다. 이때 분할의 목표는 모든 로봇의 작업 완료 시간이 거의 동일하도록, 즉 가장 오래 걸리는 로봇의 작업 시간을 최소화하도록 하는 것이다. 최근에는 이를 3D 환경으로 확장한 DARP-3D가 제안되어, 2D로 생성된 경로를 3D 지형 모델에 맞춰 고도를 조정하는 방식으로 3D 커버리지를 수행한다.45
- **분산형 접근법 (Distributed Approach):** 각 로봇이 독립적으로 자신의 주변 환경 정보만을 이용하여 의사결정을 내리고, 이웃 로봇과의 통신을 통해 협력하는 방식이다. 이는 중앙 서버의 고장 위험이 없고, 시스템의 확장성이 뛰어나다는 장점이 있다.
  - **AWSTC (Artificially Weighted Spanning Tree Coverage):** 분산형 MCPP를 위한 알고리즘으로, 환경을 셀로 분해한 뒤 각 로봇이 번갈아 가며 자신의 커버리지 영역에 새로운 셀을 추가하는 방식으로 작동한다.36 이때 각 로봇은 다른 로봇들이 이미 탐색한 셀은 피하고 아직 탐색되지 않은 셀을 우선적으로 선택하도록 설계된 점수 공식을 사용하여, 충돌 없이 영역을 분할하고 커버리지를 수행한다.

MCPP에서 중요한 최적화 목표는 **작업 부하 균형(workload balancing)**이다. 전체 임무 완료 시간은 가장 마지막에 작업을 마치는 로봇에 의해 결정되므로, 모든 로봇의 작업 시간(또는 경로 길이, 에너지 소비)을 최대한 균등하게 분배하는 것이 중요하다.8 이를 위해 유전 알고리즘과 같은 최적화 기법을 사용하여 경로들을 로봇들에게 할당함으로써 에너지 소비와 작업 시간을 균등하게 만드는 연구도 진행되고 있다.17 이 접근법은 작업 공간을 사전에 분할할 필요 없이, 생성된 경로 자체를 유전자처럼 취급하여 로봇 간에 '경로 이전(path migration)'을 수행함으로써 최적의 할당을 찾아낸다.


3D CPP 기술은 이론적인 프레임워크를 넘어, 특정 응용 분야의 고유한 환경과 요구사항에 맞춰 특화된 형태로 발전하고 있다. 이 장에서는 정밀 농업, 수중 환경, 산업 자동화라는 세 가지 대표적인 분야를 중심으로, 각 분야가 제기하는 독특한 도전 과제와 이를 해결하기 위한 CPP 기술의 진화 과정을 살펴본다.


정밀 농업(Precision Agriculture)에서 UAV는 작물의 건강 상태 모니터링, 잡초 탐지, 비료 및 농약 살포 등 다양한 임무를 수행하며 생산성 향상에 기여하고 있다.46 그러나 농경지, 특히 과수원이나 구릉지는 평평하지 않고 지형의 기복이 심하며, 작물의 높이도 일정하지 않다. 이러한 환경은 UAV의 CPP에 다음과 같은 특수한 과제를 제기한다.

- **지형 추종 (Terrain Following):** 효과적인 데이터 수집이나 균일한 약제 살포를 위해서는 UAV가 지면 또는 작물 캐노피로부터 일정한 고도를 유지하며 비행해야 한다.46 이는 고정된 고도로 비행하는 일반적인 CPP와 달리, 경로상의 모든 지점에서 지형에 따라 고도를 동적으로 조절해야 함을 의미한다.
- **3D 모델 기반 계획:** 정확한 지형 추종을 위해서는 작업 영역에 대한 정밀한 3D 모델이 필수적이다. 이 모델은 주로 항공 사진 측량(Photogrammetry) 기법인 SfM(Structure-from-Motion)이나 LiDAR 센서를 통해 사전에 구축된다.33

이러한 과제를 해결하기 위해 정밀 농업 분야의 3D CPP는 주로 2단계 접근법을 채택한다.

1. **2D 경로 생성:** 먼저, 3D 지형 모델을 2D 평면에 투영하고, 부스트로피돈 분해와 같은 고전적인 알고리즘을 사용하여 2D 커버리지 경로를 생성한다.46
2. **3D 경로로의 확장:** 다음으로, 생성된 2D 경로의 각 웨이포인트(waypoint)에 대해 3D 모델에서 해당 위치의 지형 고도 값을 가져온다. 여기에 요구되는 작전 고도(예: 작물 위 2m)를 더하여 각 웨이포인트의 최종 3D 좌표($x, y, z$)를 결정한다. 이를 통해 2D 경로를 지형을 따라 부드럽게 기동하는 3D 경로로 확장할 수 있다.45

에너지 효율성 또한 중요한 고려사항이다. 구릉지에서는 오르막 비행 시 더 많은 에너지가 소모된다. `46`의 연구에서는 UAV의 진행 방향(heading angle)에 따라 총 에너지 소비가 크게 달라짐을 보였다. 이 연구는 1도부터 180도까지 모든 가능한 진행 방향에 대해 3D 경로를 생성하고 각각의 에너지 소비를 시뮬레이션하여, 가장 적은 에너지로 임무를 완수할 수 있는 최적의 진행 방향을 찾아내는 접근법을 제시했다. 이는 지형 정보를 에너지 최적화에 적극적으로 활용하는 진일보한 방식이다. 또한, 딥러닝 기반의 영상 분석을 통해 잡초가 무성한 지역만 식별하고, 해당 지역(볼록 다각형으로 표현)에 대해서만 커버리지 경로를 생성하여 농약 살포의 효율을 극대화하는 연구도 활발히 진행되고 있다.38


수중 환경은 빛이 거의 없고 통신이 어려우며 압력이 높은 극한 환경으로, AUV는 해저 지형 매핑, 해양 자원 탐사, 수중 구조물 검사 등에서 대체 불가능한 역할을 수행한다.17 그러나 수중 환경의 물리적 특성은 AUV의 CPP에 다음과 같은 독특하고 어려운 과제를 안겨준다.

- **가변적인 소나 성능:** AUV는 주로 음파를 이용하는 소나(Sonar) 센서를 사용하여 주변을 탐지한다. 그러나 소나의 탐지 성능은 수온, 염분, 수심에 따른 음속 변화와 해저 지형의 형태에 매우 민감하게 반응한다.16 평평한 해저에서는 소나의 탐지 범위가 부채꼴이나 원형에 가깝지만, 경사진 지형이나 복잡한 지형에서는 탐지 범위가 불규칙한 타원형으로 왜곡된다. 기존의 많은 CPP 연구들이 소나의 탐지 반경을 상수로 가정했지만, 이는 심각한 커버리지 공백(gap)을 유발할 수 있다.48
- **해류의 영향:** 강한 해류는 AUV의 동역학에 직접적인 영향을 미친다. 해류를 거슬러 올라갈 때는 속도가 느려지고 에너지 소비가 급증하는 반면, 해류를 따라 이동할 때는 그 반대 현상이 나타난다. 따라서 두 지점 간의 최단 경로는 직선이 아닐 수 있으며, 이동 시간과 에너지 소비는 이동 방향에 따라 비대칭적이 된다.16

이러한 수중 환경의 특수성을 극복하기 위해 제안된 `16`와 `16`의 연구는 다음과 같은 혁신적인 접근법을 제시한다.

1. **현실적인 샘플링 포인트 결정:** 먼저, 음향 시뮬레이션 소프트웨어(예: RAM)와 소나 방정식을 결합하여 특정 위치와 방향에서의 실제 소나 탐지 범위를 예측한다. 이를 통해 지형 변화를 고려한 가변적인 탐지 범위를 계산하고, 이를 바탕으로 전체 영역을 완전히 커버하는 데 필요한 최소한의 샘플링 포인트들을 결정한다. 이는 고정된 탐지 반경을 가정하는 기존 방식보다 훨씬 정밀한 커버리지를 보장한다.
2. **해류를 고려한 최단 시간 경로 계산:** 다음으로, 결정된 샘플링 포인트들 사이를 이동하는 데 걸리는 최단 시간을 계산해야 한다. 이때 해류의 영향을 고려하여 이동 시간을 계산하며, 수많은 샘플링 포인트 쌍에 대해 이를 효율적으로 계산하기 위해 기존 Dijkstra 알고리즘의 탐색 전략을 수정한 '개선된 Dijkstra 알고리즘'을 사용한다.
3. **다중 AUV 작업 할당 및 경로 최적화:** 마지막으로, 다중 AUV 시스템의 경우, PSO 알고리즘을 사용하여 각 AUV에게 샘플링 포인트들을 균등하게 할당(작업 할당)하고, 각 AUV는 할당된 포인트들을 최단 시간에 방문하는 경로를 ELKAI 솔버(TSP 문제 해결기)를 이용해 찾는다. 이 과정은 전체 임무 완료 시간을 최소화하도록 반복적으로 최적화된다.

이처럼 수중 CPP는 센서 물리학, 해양학, 최적화 이론이 복합적으로 융합된 분야로, 환경과의 상호작용을 정밀하게 모델링하는 것이 문제 해결의 핵심임을 보여준다.


산업 현장에서 6자유도(6-DOF) 이상의 다관절 로봇은 자동차 차체, 가구, 항공기 부품 등의 표면에 페인트를 도포하는 스프레이 페인팅(Spray Painting)과 같은 정밀한 커버리지 작업을 수행한다.21 이 작업의 주된 목표는 제품 표면에 **균일한 두께의 도막(paint film)**을 형성하는 것이다. 이는 다음과 같은 고유한 과제를 수반한다.

- **도구 경로 생성(Tool Path Generation):** 단순히 표면을 훑는 것을 넘어, 스프레이 건(로봇의 엔드 이펙터)의 위치와 방향(자세)을 동시에 정밀하게 제어해야 한다. 일반적으로 스프레이 건은 표면에 항상 수직 방향을 유지해야 하며, 일정한 거리를 유지해야 최적의 도포 품질을 얻을 수 있다.50
- **복잡한 자유 곡면 처리:** 자동차 차체와 같이 복잡한 3D 자유 곡면을 다룰 때, 균일한 커버리지를 위한 경로를 생성하는 것은 매우 어렵다.
- **도포 모델링:** 스프레이 건에서 분사되는 페인트 입자의 분포는 종종 가우시안(Gaussian)과 같은 형태로 모델링된다. 경로 계획 시에는 인접한 경로 간의 간격(pass interval)을 조절하여 페인트가 겹치는 부분(overlap)에서 도막 두께가 균일하게 유지되도록 해야 한다.
- **로봇 동역학 제약:** 로봇의 각 관절은 속도 및 가속도 제한을 가진다. 경로를 따라 이동할 때, 특히 급격한 방향 전환이 일어나는 구간에서 이러한 동역학적 제약을 위반하지 않도록 속도 프로파일을 계획(trajectory planning)해야 한다.50

이러한 과제를 해결하기 위해 산업계와 학계에서는 다양한 접근법이 사용되고 있다.50

- **수동 교시 (Manual Teaching):** 숙련된 작업자가 직접 로봇을 움직여 경로를 기록하는 방식으로, 시간이 많이 걸리고 품질이 작업자의 숙련도에 크게 의존한다.
- **오프라인 자동 경로 생성 (Offline Automatic Path Planning):** 제품의 CAD 모델을 입력받아 컴퓨터가 자동으로 경로를 생성하는 방식이다.
  - **오프셋 곡선 방식 (Offset Curve Method):** 파라메트릭 곡면(parametric surface)에 대해 기준이 되는 '시드 커브(seed curve)'를 하나 생성한 뒤, 이를 표면을 따라 일정한 간격으로 오프셋(offset)하여 전체 경로 패턴(지그재그, 래스터 등)을 생성한다.
  - **표면 분할 (Surface Segmentation):** 복잡한 자유 곡면을 다루기 위해, 전체 표면을 기하학적으로 더 단순한 여러 개의 패치(patch)로 분할한다. 그런 다음 각 패치에 대해 독립적으로 경로를 생성하고, 이들을 부드럽게 연결한다.
  - **절단 방식 (Cutting Method):** 가상의 평면들로 제품 모델을 잘라내어 단면 곡선들을 생성하고, 이 곡선들을 따라 이동하는 경로를 만든다.

최근에는 RGB-D 카메라와 같은 비전 시스템을 사용하여 실시간으로 제품의 위치나 형태 변화를 감지하고, 이에 맞춰 경로를 즉석에서 수정하는 온라인(online) 경로 계획 기술도 연구되고 있으나, 페인트 분진으로 인한 카메라 오염 문제 등 해결해야 할 과제가 남아있다.50


본 보고서는 3D 커버리지 경로 계획(CPP) 기술의 다층적인 스펙트럼을 고전적 접근법부터 현대적 패러다임, 핵심 최적화 과제, 그리고 주요 응용 분야에 이르기까지 체계적으로 분석하고 심층적으로 고찰했다. 분석을 통해 3D CPP 기술이 지난 수십 년간 어떻게 진화해왔으며, 현재 어떤 방향으로 나아가고 있는지에 대한 명확한 그림을 그릴 수 있었다.

기술의 진화는 **'기하학적 해법'에서 '학습 기반 최적화'로의 패러다임 전환**으로 요약될 수 있다. 초기의 셀 분해나 그래프 기반 알고리즘은 환경을 명시적으로 모델링하고 결정론적인 규칙에 따라 경로를 생성하는 데 중점을 두었다. 이들은 계산적으로 효율적이고 완전한 커버리지를 보장하는 강력한 이론적 토대를 마련했다. 그러나 이들은 종종 생성된 경로의 품질이 낮거나, 복잡하고 동적인 환경에 유연하게 대처하기 어렵다는 한계를 보였다. 샘플링 기반 방법론은 문제 자체를 재구성(reformulation)하는 영리한 전략을 통해 고차원의 복잡한 공간을 다루는 돌파구를 마련했으며, 점근적 최적성이라는 개념을 도입했다.

현대에 이르러, 강화학습(RL)의 등장은 CPP 분야에 또 한 번의 혁신을 가져왔다. RL은 명시적인 모델 없이, 환경과의 상호작용과 시행착오를 통해 복잡한 다중 목표(커버리지, 에너지, 안전성)를 동시에 만족시키는 최적의 행동 정책을 스스로 학습할 수 있는 가능성을 열었다. 특히, 대규모 언어 모델(LLM)을 '소프트 태스크 플래너'로 활용하여 RL 에이전트의 학습을 안내하는 PIRL과 같은 최신 연구는, 기계가 인간의 상식적, 전략적 추론 능력을 모방하여 더 지능적인 경로를 생성할 수 있음을 시사한다. 또한, 베지에 곡선을 이용한 연속 공간에서의 경로 계획이나 재충전과 같은 현실적인 제약을 통합하는 연구들은 RL 기반 CPP가 실제 현장에 적용될 수 있는 수준으로 성숙하고 있음을 보여준다.

응용 분야별 분석을 통해, CPP 기술이 더 이상 범용적인 단일 해법에 머무르지 않고, 각 분야의 고유한 물리적, 환경적 제약에 맞춰 고도로 **전문화(specialization)**되고 있음을 확인했다. 정밀 농업에서는 지형 추종을 위해 3D 모델을 활용하고, 수중 탐사에서는 소나의 물리적 특성과 해류의 영향을 동역학 모델에 통합하며, 산업 현장에서는 균일한 도포 품질을 위해 로봇의 자세와 속도를 정밀하게 제어한다. 이는 미래의 CPP 기술이 특정 도메인 지식(domain knowledge)과 더욱 깊이 융합될 것임을 예고한다.

이러한 분석을 바탕으로, 3D CPP 기술의 미래 연구는 다음과 같은 방향으로 나아갈 것으로 전망된다.

1. **인식과 계획의 통합 (Tighter Integration of Perception and Planning):** 현재 많은 시스템이 '인식(3D 모델링) 후 계획'이라는 분리된 파이프라인을 따르고 있다. 미래에는 로봇이 환경을 탐색하며 실시간으로 얻는 정보를 즉시 경로 계획에 반영하여, 불확실하고 동적인 환경에 더욱 강건하게 대처하는 통합 프레임워크가 중요해질 것이다.
2. **일반화 가능한 학습 기반 플래너 (Generalizable Learning-based Planners):** 현재의 RL 기반 플래너들은 특정 작업이나 환경에 과적합(overfitting)되는 경향이 있다. 메타 학습(meta-learning)이나 전이 학습(transfer learning) 기법을 적용하여, 한 번 학습된 정책이 새로운 환경이나 작업에 최소한의 추가 학습만으로 빠르게 적응할 수 있는 일반화 능력을 갖추는 것이 핵심 과제가 될 것이다.
3. **Sim-to-Real의 고도화:** 시뮬레이션에서 학습된 RL 정책을 실제 로봇에 성공적으로 이전하는 Sim-to-Real 문제는 여전히 큰 장벽이다. 시뮬레이션 환경을 현실과 더욱 가깝게 만드는 물리 엔진의 발전과 함께, 실제 환경과의 차이를 극복하는 도메인 적응(domain adaptation) 기술에 대한 연구가 더욱 활발해질 것이다.
4. **인간-로봇 협력 커버리지 (Human-Robot Collaborative Coverage):** 완전 자율 시스템을 넘어, 인간 작업자가 고수준의 목표(예: "이 구역을 더 정밀하게 탐색해줘")를 제시하면 로봇이 이를 해석하여 자율적으로 세부 커버리지 경로를 생성하고 실행하는, 상호작용이 가능한 협력적 CPP 프레임워크가 새로운 연구 분야로 부상할 것이다.

결론적으로, 3D 커버리지 경로 계획은 로봇 자율성의 핵심을 이루는 기술로서, 고전적인 알고리즘의 견고한 기반 위에 인공지능, 특히 강화학습이라는 강력한 도구를 장착하며 빠르게 발전하고 있다. 앞으로도 이론적 깊이와 실제적 적용 가능성을 동시에 추구하며, 로봇이 우리 삶의 더 많은 영역에서 지능적인 작업을 수행할 수 있도록 하는 데 결정적인 역할을 할 것이다.


1. Survey on Coverage Path Planning with Unmanned Aerial Vehicles - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2504-446X/3/1/4
2. A survey on coverage path planning for robotics - SciSpace, 8월 9, 2025에 액세스, https://scispace.com/pdf/a-survey-on-coverage-path-planning-for-robotics-584p6o3a2k.pdf
3. Coverage Path Planning for Unmanned Aerial Vehicles in Complex 3D Environments with Deep Reinforcement Learning - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/367262283_Coverage_Path_Planning_for_Unmanned_Aerial_Vehicles_in_Complex_3D_Environments_with_Deep_Reinforcement_Learning
4. Continuous World Coverage Path Planning for Fixed-Wing ... - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/pdf/2505.08382
5. On Complete Coverage Path Planning Algorithms for Non-holonomic Mobile Robots: Survey and Challenges, 8월 9, 2025에 액세스, [https://www.uhb.edu.sa/Lists/Books%20and%20Articals/Attachments/1125/BookFile-%202017-Amna-On%20complete%20coverage%20path%20planning%20of%20non%20holonomic%20mobile%20robots.pdf](https://www.uhb.edu.sa/Lists/Books and Articals/Attachments/1125/BookFile- 2017-Amna-On complete coverage path planning of non holonomic mobile robots.pdf)
6. (PDF) Energy-Efficient UAVs Coverage Path Planning Approach - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/369121898_Energy-Efficient_UAVs_Coverage_Path_Planning_Approach
7. Coverage Path Planning Methods Focusing on Energy Efficient and Cooperative Strategies for Unmanned Aerial Vehicles - PubMed Central, 8월 9, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8839296/
8. A Multi-Robot Coverage Path Planning Method for Maritime Search and Rescue Using Multiple AUVs - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2072-4292/15/1/93
9. TMSTC*: A Turn-minimizing Algorithm For Multi-robot Coverage Path Planning - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/pdf/2212.02231
10. Global Path Planning Method Based on a Modification of the Wavefront Algorithm for Ground Mobile Robots - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2218-6581/12/1/25
11. A Review of UAV Path-Planning Algorithms and Obstacle Avoidance Methods for Remote Sensing Applications - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2072-4292/16/21/4019
12. 3D Path Planning Algorithms in UAV-Enabled Communications ..., 8월 9, 2025에 액세스, https://www.mdpi.com/1999-5903/15/9/289
13. A Review on Viewpoints and Path-planning for UAV-based 3D Reconstruction - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2205.03716
14. Energy optimal UAV path planning for full area coverage: A Review - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/392148441_Energy_optimal_UAV_path_planning_for_full_area_coverage_A_Review
15. A Survey on Multi-UAV Path Planning: Classification, Algorithms, Open Research Problems, and Future Directions - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2504-446X/9/4/263
16. Coverage path planning for multi-AUV considering ocean ... - Frontiers, 8월 9, 2025에 액세스, https://www.frontiersin.org/journals/marine-science/articles/10.3389/fmars.2024.1483122/full
17. Three-Dimensional Coverage Path Planning for Cooperative Autonomous Underwater Vehicles: A Swarm Migration Genetic Algorithm Approach - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2077-1312/12/8/1366
18. Multi-AUV coverage path planning algorithm using side-scan sonar for maritime search | Request PDF - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/380251502_Multi-AUV_coverage_path_planning_algorithm_using_side-scan_sonar_for_maritime_search
19. [2302.00968] 3D Coverage Path Planning for Efficient Construction Progress Monitoring - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2302.00968
20. Coverage Path Planning: The Boustrophedon Cellular Decomposition, 8월 9, 2025에 액세스, https://www.ri.cmu.edu/pub_files/pub4/choset_howie_1997_3/choset_howie_1997_3.pdf
21. (PDF) Path Planning for Special Robotic Operations - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/364103981_Path_Planning_for_Special_Robotic_Operations
22. Efficient Boustrophedon Multi-Robot Coverage: an algorithmic approach - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/220642845_Efficient_Boustrophedon_Multi-Robot_Coverage_an_algorithmic_approach
23. A survey on coverage path planning for robotics - sci-hub.red, 8월 9, 2025에 액세스, https://dacemirror.sci-hub.red/journal-article/858fba7bd11dcfc753536574d26af483/galceran2013.pdf
24. Geometric Trajectory Planning for Robot Motion Over a 3D Surface, 8월 9, 2025에 액세스, https://www.asmedigitalcollection.asme.org/DSCC/proceedings-pdf/DSCC2019/59162/6455745/v003t19a006-dscc2019-9063.pdf
25. Revisiting boustrophedon coverage path planning as a generalized ..., 8월 9, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/20.500.11850/395090/1/Baehnemann2019revisiting.pdf
26. Comp150-07: Intelligent Robotics: Wavefront Planner, 8월 9, 2025에 액세스, https://www.cs.tufts.edu/comp/150IR/labs/wavefront.html
27. A Simple Robot Selection Criteria After Path Planning Using Wavefront Algorithm, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/372784966_A_Simple_Robot_Selection_Criteria_After_Path_Planning_Using_Wavefront_Algorithm
28. A Skeleton-Based Topological Planner for Exploration in Complex Unknown Environments, 8월 9, 2025에 액세스, https://arxiv.org/html/2412.13664v1
29. On-the-Go Path Planning and Repair in Static and Dynamic Scenarios - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2411.12014v1
30. MIT Open Access Articles Sampling-Based ... - DSpace@MIT, 8월 9, 2025에 액세스, [https://dspace.mit.edu/bitstream/handle/1721.1/87729/Brendan%20Englot%20ICAPS%20final%20version%20submitted%20March%202012.pdf;sequence=1](https://dspace.mit.edu/bitstream/handle/1721.1/87729/Brendan Englot ICAPS final version submitted March 2012.pdf;sequence=1)
31. Improved RRT-Connect Algorithm Based on Triangular Inequality for Robot Path Planning, 8월 9, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC7825297/
32. (PDF) 3D path planning, routing algorithms and routing protocols for unmanned air vehicles: a review - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/333764825_3D_path_planning_routing_algorithms_and_routing_protocols_for_unmanned_air_vehicles_a_review
33. Spanning Tree Coverage Algorithm -Key steps. - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/figure/Spanning-Tree-Coverage-Algorithm-Key-steps_fig1_347154520
34. Coverage Path Planning in Dynamic Environment through Guided Reinforcement Learning - UC Berkeley EECS, 8월 9, 2025에 액세스, https://www2.eecs.berkeley.edu/Pubs/TechRpts/2022/EECS-2022-123.pdf
35. Spanning tree covering - Steven M. LaValle, 8월 9, 2025에 액세스, https://lavalle.pl/planning/node353.html
36. An Improved Spanning Tree-Based Algorithm for Coverage of Large Areas Using Multi-UAV Systems - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2504-446X/7/1/9
37. (PDF) UAV Path Planning Techniques: A Survey - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/379299233_UAV_Path_Planning_Techniques_A_Survey
38. Optimization of Coverage Path Planning for Agricultural Drones in Weed-Infested Fields Using Semantic Segmentation - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2077-0472/15/12/1262
39. Adaptive Particle Swarm and Ant Colony Optimization Path Planning for Autonomous Robot Navigation - Journal UMY, 8월 9, 2025에 액세스, https://journal.umy.ac.id/index.php/jrc/article/view/26853
40. Prompt Informed Reinforcement Learning for Visual ... - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2507.10284
41. [2309.03157] Learning to Recharge: UAV Coverage Path Planning through Deep Reinforcement Learning - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2309.03157
42. Energy-Efficient UAVs Coverage Path Planning Approach, 8월 9, 2025에 액세스, https://pure.kfupm.edu.sa/en/publications/energy-efficient-uavs-coverage-path-planning-approach
43. [Literature Review] Energy-aware Multi-UAV Coverage Mission Planning with Optimal Speed of Flight - Moonlight, 8월 9, 2025에 액세스, https://www.themoonlight.io/en/review/energy-aware-multi-uav-coverage-mission-planning-with-optimal-speed-of-flight
44. Coverage Path Planning for UAVs: An Energy-Efficient Method in Convex and Non-Convex Mixed Regions - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2504-446X/8/12/776
45. Terrain-Aware Adaptation for Two-Dimensional UAV Path Planners - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2507.17519v2
46. A Three-Dimensional Full-Coverage Operation Path Planning Method for Plant Protection Unmanned Aerial Vehicles Based on Energy Consumption Modeling - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2079-9292/12/19/4051
47. Coverage path planning with unmanned aerial vehicles for 3D terrain reconstruction | Request PDF - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/296691159_Coverage_path_planning_with_unmanned_aerial_vehicles_for_3D_terrain_reconstruction
48. Coverage Path Planning With Track Spacing Adaptation for Autonomous Underwater Vehicles | Request PDF - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/342324144_Coverage_Path_Planning_With_Track_Spacing_Adaptation_for_Autonomous_Underwater_Vehicles
49. Robot Design: From Theory to Service Applications 3031111273, 9783031111273 - EBIN.PUB, 8월 9, 2025에 액세스, https://ebin.pub/robot-design-from-theory-to-service-applications-3031111273-9783031111273.html
50. Università degli studi di Udine, 8월 9, 2025에 액세스, [https://air.uniud.it/retrieve/0e9e3224-6fc0-479c-9569-814ad1948c66/CHAPTER_PATH_PLANNING%20-%20final.pdf](https://air.uniud.it/retrieve/0e9e3224-6fc0-479c-9569-814ad1948c66/CHAPTER_PATH_PLANNING - final.pdf)


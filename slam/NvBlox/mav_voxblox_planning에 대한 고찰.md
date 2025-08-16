# mav_voxblox_planning에 대한 고찰

## 1. 서론 (Introduction)

### 0.1 자율 비행 MAV와 실시간 경로 계획의 도전 과제

소형 무인 항공기(Micro Aerial Vehicle, MAV)는 민첩한 기동성과 작은 크기 덕분에 정찰, 탐사, 구조 등 다양한 분야에서 각광받는 로보틱스 플랫폼으로 자리 잡았다. 그러나 MAV는 탑재할 수 있는 페이로드와 전력 예산이 매우 제한적이며, 빠른 동역학적 특성을 가지므로 경량화되고 신속하게 작동하는 알고리즘을 필요로 한다.1 특히, 사전에 알려지지 않은 비정형 환경(unstructured, unexplored environments)에서의 자율 비행은 실시간으로 주변 환경을 인지하여 3차원 지도를 작성(mapping)하고, 동시에 안전하고 실행 가능한 비행 경로를 계획(planning)해야 하는 매우 어려운 문제이다. 본 보고서는 이러한 실시간 경로 계획의 핵심 도전 과제를 해결하기 위해 제안된 `mav_voxblox_planning` 프레임워크를 심층적으로 분석하고자 한다.

### 0.2 환경 표현의 중요성: 왜 Signed Distance Field인가?

효율적인 경로 계획을 위해서는 로봇이 주변 환경을 어떻게 인식하고 표현하는지가 매우 중요하다. 다양한 맵 표현 방식 중, 특히 궤적 최적화(trajectory optimization)에 기반한 경로 계획 기법들은 장애물과의 충돌을 회피하기 위해 현재 위치에서 가장 가까운 장애물까지의 거리 정보와 그 거리의 변화율, 즉 기울기(gradient) 정보를 필수적으로 요구한다.1 유클리드 부호 거리 필드(Euclidean Signed Distance Field, ESDF)는 이러한 요구사항을 완벽하게 충족시키는 맵 표현 방식이다. ESDF는 3차원 공간을 일정한 크기의 복셀(voxel) 그리드로 나누고, 각 복셀에 가장 가까운 장애물까지의 최단 유클리드 거리를 저장한다. 이때 거리는 장애물 바깥의 자유 공간(free space)에서는 양수 값을, 장애물 내부에서는 음수 값을 가져 '부호(signed)' 거리 필드라 불린다. 이처럼 ESDF는 맵 상의 모든 지점에서 장애물까지의 거리와 기울기 정보를 연속적으로 제공함으로써, MAV가 동역학적 제약을 만족시키면서도 부드럽고 안전한 비행 궤적을 생성하는 데 이상적인 환경 표현 방법으로 평가받는다.3

### 0.3 `mav_voxblox_planning`의 소개 및 보고서의 목적

`mav_voxblox_planning`은 스위스 연방 공과대학교 취리히(ETH Zurich)의 자율 시스템 연구소(Autonomous Systems Lab, ASL)에서 개발한 오픈소스 소프트웨어 패키지이다. 이 프레임워크의 가장 큰 특징은 **Voxblox**라는 맵핑 라이브러리를 지도 표현의 근간으로 삼아 MAV를 위한 다양한 경로 계획 도구를 통합적으로 제공한다는 점이다.5 이 시스템은 당시 로보틱스 연구의 최전선에 있던 기술들, 즉 증분적으로 ESDF를 생성하는 Voxblox와 연속 시간 궤적 최적화(continuous-time trajectory optimization) 기법을 결합하여, 실제 MAV에 탑재하고 실시간으로 구동되는 것을 성공적으로 검증했다는 점에서 큰 학술적, 실용적 의의를 가진다.6 본 보고서의 목적은 `mav_voxblox_planning` 프레임워크를 구성하는 핵심 기술의 수학적 원리부터 소프트웨어 아키텍처, 실제 구현을 위한 가이드, 그리고 학술적 맥락에서의 위치와 한계점을 분석하고, FIESTA, EGO-Planner와 같은 주요 후속 연구들과의 비교를 통해 MAV 자율 비행 기술의 발전 동향을 포괄적으로 조망하는 것이다. 이를 통해 독자에게 해당 기술에 대한 깊이 있는 통찰을 제공하고자 한다.

## 1.  핵심 기반 기술: Voxblox의 수학적 원리 (Core Foundational Technology: Mathematical Principles of Voxblox)

`mav_voxblox_planning`의 성능과 특징을 이해하기 위해서는 그 근간을 이루는 맵 표현 방식인 Voxblox를 먼저 이해해야 한다. Voxblox의 핵심은 TSDF(Truncated Signed Distance Field)와 ESDF(Euclidean Signed Distance Field)라는 두 가지 거리 필드를 효율적으로 연계하여 관리하는 데 있다.

### 1.1  체적 맵핑과 Signed Distance Fields (Volumetric Mapping and Signed Distance Fields)

Voxblox는 3차원 공간을 복셀 단위로 표현하는 체적 맵핑(volumetric mapping) 라이브러리이다. 이는 단순히 표면 정보만 저장하는 것이 아니라 공간 전체의 부피에 대한 정보를 담을 수 있어 로봇 경로 계획에 매우 적합하다.8

- **TSDF (Truncated Signed Distance Field):** TSDF는 주로 3D 재구성 및 시각화 분야에서 널리 사용되는 암시적 표면 표현(implicit surface representation) 방식이다. 각 복셀은 센서 원점으로부터 뻗어 나가는 광선(ray)을 따라 측정된 표면까지의 거리를 저장하는데, 이를 '투영 거리(projective distance)'라고 한다. 모든 거리를 저장하는 대신, 표면 주변의 일정 절단 거리($\delta$, truncation distance) 내에 있는 값만 유효하게 저장하고 그 밖의 값은 최대값으로 처리하여 계산 효율성을 높인다.6 TSDF는 여러 번의 센서 측정값을 자연스럽게 융합하여 노이즈를 평활화(smoothing)하고, 고품질의 3D 메쉬(mesh)를 생성하는 데 강점을 가진다.10
- **ESDF (Euclidean Signed Distance Field):** 반면, ESDF는 경로 계획을 위해 특화된 표현 방식이다. 각 복셀은 센서 광선 방향과 무관하게, 공간상에서 가장 가까운 장애물 표면까지의 '최단 유클리드 거리'를 저장한다.1 이 정보는 궤적 최적화 알고리즘이 충돌 비용과 그 기울기를 계산하는 데 직접적으로 사용되어, 안전하고 효율적인 경로를 찾는 데 결정적인 역할을 한다.1

이 두 가지 거리 필드의 관계에서 Voxblox의 핵심 설계 철학이 드러난다. Voxblox는 TSDF의 장점인 빠른 증분적 업데이트, 센서 노이즈에 대한 강인함, 그리고 시각적으로 우수한 메쉬 생성 능력을 활용하면서, 동시에 경로 계획에 필수적인 ESDF를 효율적으로 생성하고자 한다. 즉, 먼저 빠르고 쉽게 TSDF 맵을 구축한 다음, 이로부터 ESDF를 파생시키는 2단계 접근법을 취한다. 하지만 이러한 설계는 근본적인 트레이드오프를 내포한다. TSDF가 사용하는 '투영 거리'는 항상 실제 '유클리드 거리'보다 크거나 같기 때문에, TSDF로부터 생성된 ESDF는 필연적으로 오차를 포함하게 된다. 이 오차는 Voxblox의 내재적 한계이며, FIESTA와 같은 후속 연구들이 TSDF를 거치지 않고 점유 격자 지도(occupancy map)에서 직접 ESDF를 생성하는 방식으로 발전하게 된 직접적인 동기가 되었다.11 결국 TSDF를 중간 단계로 사용하는 것은 Voxblox의 장점(메싱)과 단점(정확도)을 동시에 낳는 핵심적인 아키텍처 결정이라 할 수 있다.

### 1.2  TSDF로부터 ESDF 증분적 생성 알고리즘 (The Incremental ESDF Generation Algorithm from TSDF)

Voxblox는 새로운 센서 데이터가 들어올 때마다 맵 전체를 다시 계산하는 대신, 변경된 부분만 효율적으로 업데이트하는 증분적(incremental) 방식을 채택한다. 이 과정은 크게 TSDF 통합과 ESDF 전파의 두 단계로 나뉜다.

- **TSDF 통합 과정:** 깊이 카메라 등으로부터 새로운 포인트 클라우드 데이터가 입력되면, 각 포인트에 대해 센서 원점($s$)부터 해당 포인트($p$)까지 가상의 광선을 투사한다. 이 광선이 통과하는 경로상의 각 복셀($x$)에 대해 투영 거리($d$)를 계산하고, 이를 기존 복셀 값에 융합한다.

  - **가중치 함수(Weighting Function):** 여러 측정값을 평균 내어 융합할 때, 각 측정의 신뢰도를 반영하기 위해 가중치를 사용한다. Voxblox는 단순한 상수 가중치($w_{const}$)뿐만 아니라, 깊이 센서의 노이즈 모델을 근사하여 거리가 멀어질수록 가중치를 낮추는 2차 가중치($w_{quad}$) 방식을 제안하여 정확도를 개선했다.1
    $$
    d(x, p, s) = ||p - x|| \text{ sign}((p - x) \cdot (p - s))
    $$
    새로운 측정값(거리 $d$, 가중치 $w$)을 기존 복셀 값(거리 $D_i$, 가중치 $W_i$)에 통합하는 수식은 다음과 같다:
    $$
    D_{i+1}(x) = \frac{W_i(x)D_i(x) + w(x, p)d(x, p, s)}{W_i(x) + w(x, p)}
    $$
    
    $$
    W_{i+1}(x) = \min(W_i(x) + w(x, p), W_{max})
    $$

  - **Wavefront Propagation:** ESDF 업데이트는 이 과정의 핵심이다. TSDF 값이 변경된 복셀들을 시작점으로 하여, 마치 파동이 퍼져나가듯 거리 정보를 주변으로 전파시킨다. 이 알고리즘은 두 개의 우선순위 큐(priority queue)를 사용하여 효율적으로 관리된다.1

  - **`raise` 큐:** 이 큐는 기존에 장애물로 인식되었던 영역이 비어있게 되거나, 장애물이 멀어져서 기존 ESDF 값이 더 이상 유효하지 않게 된 복셀들을 처리한다. `raise` 큐에 들어간 복셀과, 그 복셀의 거리 정보에 의존하던 주변 '자식' 복셀들의 거리 값은 무한대에 가까운 값으로 초기화되어, 새로운 거리 정보가 전파될 수 있도록 준비한다.
  - **`lower` 큐:** 이 큐는 새로운 장애물이 관측되거나 기존 장애물이 더 가까워져서 더 작은(더 정확한) 거리 값이 생긴 복셀들을 처리한다. `lower` 큐의 복셀로부터 시작하여 26개의 이웃 복셀들을 순회하며, 현재 복셀을 거쳐가는 경로가 기존에 알려진 거리보다 더 짧은지를 확인하고, 만약 그렇다면 이웃 복셀의 거리 값을 갱신하고 큐에 추가하여 전파를 계속한다.

- **동적 맵 확장:** 로봇이 미지의 환경을 탐사함에 따라 맵은 계속해서 커져야 한다. Voxblox는 이를 위해 복셀 해싱(Voxel Hashing) 기법을 사용한다. 이는 맵의 크기가 동적으로 확장될 수 있게 하면서도, 특정 복셀에 접근하는 시간을 평균적으로 상수 시간($O(1)$)으로 보장한다. 이는 맵이 커질수록 접근 시간이 느려지는 Octree 구조에 비해 속도 면에서 큰 이점을 가진다.1

### 1.3  오차 분석 및 정확도 (Error Analysis and Accuracy)

Voxblox는 실시간 성능을 위해 몇 가지 근사를 사용하며, 이는 필연적으로 오차를 발생시킨다. 주목할 점은, Voxblox 연구자들이 이러한 오차를 무시하지 않고, 그 원인을 수학적으로 분석하고 정량화하여 시스템의 한계를 명확히 밝혔다는 것이다. 이는 단순한 알고리즘 제안을 넘어, 시스템의 내재적 한계를 인정하고 이를 극복하기 위한 실용적인 해결책까지 함께 제시하는 성숙한 공학 연구의 태도를 보여준다.

- **투영 거리 오차 (Projective Distance Error):** TSDF 생성 시 사용되는 투영 거리는 실제 최단 유클리드 거리보다 항상 크거나 같을 수밖에 없다. 입사각($\theta$)과 측정된 거리($d$)에 따른 오차의 기댓값은 다음과 같이 계산된다.1
  $$
  E[r_p(\theta)] = -0.3014d
  $$
  이 음수 값은 시스템이 실제보다 장애물이 더 멀리 있다고(즉, 더 안전하다고) 판단하는 경향이 있음을 의미한다.

- **준-유클리드 거리 오차 (Quasi-Euclidean Distance Error):** ESDF 전파 과정에서, 복셀 그리드 상의 이웃까지의 거리를 계산할 때 실제 유클리드 거리가 아닌 수평, 수직, 대각선 방향의 정해진 값(1, $\sqrt{2}$, $\sqrt{3}$)을 사용한다. 이로 인해 발생하는 오차를 준-유클리드 거리 오차라 하며, 그 기댓값은 다음과 같다.1
  $$
  E[r_q(\phi)] = -0.0548d
  $$

이러한 오차 분석을 바탕으로, 연구자들은 경로 계획 시 안전을 보장하기 위한 실용적인 방안을 제시한다. 즉, 로봇의 실제 크기보다 약간 더 큰 가상의 경계 상자(bounding box)를 사용하여 경로를 계획하라는 것이다. 논문에서는 두 오차를 종합적으로 고려하여 로봇의 크기를 **8.25%**만큼 부풀려서(inflate) 사용하는 것을 권장한다.1 이는 시스템의 이론적 한계를 실제 적용 시의 안전 마진으로 연결시킨 매우 중요한 공학적 결론이다.

## 2.  `mav_voxblox_planning` 패키지 아키텍처 분석 (Analysis of the `mav_voxblox_planning` Package Architecture)

`mav_voxblox_planning`은 단일 프로그램이 아니라, 특정 목적을 수행하는 여러 개의 ROS(Robot Operating System) 패키지들이 유기적으로 결합된 프레임워크이다. 이러한 모듈식 설계는 유연성과 확장성을 높여준다.

### 2.1  시스템 개요 및 구성 요소 (System Overview and Components)

`mav_voxblox_planning` 프레임워크는 전역 경로 계획(global planning)과 지역 경로 계획(local planning)을 위한 다양한 도구들로 구성되어 있다. 각 기능은 독립적인 ROS 패키지로 구현되어 필요에 따라 조합하여 사용할 수 있다.5

- 주요 패키지 목록 5:
  - `voxblox`: 프레임워크의 심장부. TSDF와 ESDF 맵을 생성하고 관리하는 핵심 맵핑 라이브러리이다.9
  - `voxblox_rrt_planner`: OMPL(Open Motion Planning Library)과의 인터페이스를 제공한다. RRT*, BIT* 등 다양한 샘플링 기반 알고리즘을 사용하여 전역 경로를 계획한다.
  - `voxblox_skeleton_planner`: ESDF 맵에서 공간의 위상(topology)을 나타내는 스켈레톤 그래프를 추출하고, 이 그래프 상에서 최단 경로를 탐색하여 전역 경로를 생성한다.
  - `mav_local_planner`: 지역 경로 계획을 위한 고수준 관리자 역할을 한다. 전역 경로를 입력받아 부드럽게 추종하거나, 비행 중 새로운 장애물을 감지하면 `voxblox_loco_planner`를 호출하여 실시간으로 경로를 재계획(replan)한다.13
  - `voxblox_loco_planner`: ESDF 맵의 거리 및 기울기 정보를 활용하여 궤적 최적화 기반의 저수준 지역 경로를 생성하는 핵심 실행기이다.
  - `mav_planning_rviz`: 로봇 시각화 도구인 RViz를 위한 플러그인. 사용자가 GUI를 통해 인터랙티브 마커로 출발점과 목표점을 지정하고, 다양한 플래너의 작동을 시각적으로 확인하며 제어할 수 있게 해준다.

### 2.2  전역 경로 계획기: 샘플링 및 토폴로지 기반 접근법 (Global Planners: Sampling and Topology-Based Approaches)

전역 경로 계획은 맵 전체를 고려하여 출발점에서 목표점까지의 대략적인 경로를 찾는 과정이다. `mav_voxblox_planning`은 두 가지 주요한 전역 계획 방식을 제공한다.

- **RRT/OMPL 기반 플래너 (`voxblox_rrt_planner`):** 이 플래너는 미지의 넓은 공간을 탐색하는 데 효과적인 샘플링 기반(sampling-based) 접근법을 사용한다. OMPL이라는 검증된 라이브러리를 활용하여 RRT*(Rapidly-exploring Random Tree*), BIT*(Batch Informed Trees*), PRM*(Probabilistic Roadmap*) 등 학계에서 널리 사용되는 다양한 알고리즘을 손쉽게 적용할 수 있다.5 이 과정에서 ESDF 맵은 무작위로 샘플링된 점이나 두 점을 잇는 경로가 장애물과 충돌하는지 여부를 빠르고 정확하게 판단하는 데 사용된다.
- **스켈레톤 기반 플래너 (`voxblox_skeleton_planner`):** 이 방식은 특히 복도나 동굴처럼 좁고 복잡한 구조의 환경에서 매우 효율적이다. 먼저 ESDF 맵으로부터 공간의 중심선을 따라 이어지는 희소 그래프(sparse graph), 즉 '스켈레톤'을 추출한다. 이 스켈레톤은 환경의 전체적인 연결 구조(topology)를 간결하게 표현한다. 일단 스켈레톤이 생성되면, A*와 같은 고전적인 그래프 탐색 알고리즘을 사용하여 이 그래프 상에서 매우 빠르게 최적의 전역 경로를 찾을 수 있다.5

### 2.3  지역 경로 계획기: 온라인 재계획 (Local Planners: Online Replanning)

지역 경로 계획은 로봇의 현재 상태와 주변 환경만을 고려하여, 짧은 시간 범위(short horizon) 내에서 즉시 실행 가능한 구체적인 궤적을 생성하는 과정이다. 이 프레임워크의 핵심적인 강점은 바로 실시간 재계획 능력에 있다.

`mav_local_planner`는 고수준의 제어기 역할을 수행하며, 전역 경로를 따라 비행하도록 `voxblox_loco_planner`에 명령을 내린다.5 비행 중 MAV의 센서가 이전에 발견하지 못했던 새로운 장애물을 감지하여 `voxblox` 맵이 업데이트되면, `mav_local_planner`는 즉시 이 변화를 인지한다. 그리고 현재의 전역 경로가 더 이상 안전하지 않다고 판단되면, 즉시 `voxblox_loco_planner`를 다시 호출하여 현재 위치에서부터 장애물을 안전하게 우회하는 새로운 지역 궤적을 생성하도록 지시한다. 이 전체 재계획 과정은 약 4Hz의 빠른 속도로 반복적으로 수행될 수 있어, MAV가 동적인 환경 변화에 민첩하게 대응할 수 있도록 한다.5

`mav_voxblox_planning`의 아키텍처는 이처럼 전역 계획과 지역 계획을 나누는 고전적인 계층 구조를 따른다. 하지만 그 이면에는 매우 중요한 설계적 결정이 숨어있다. 바로 **전역 계획기(충돌 체크용)와 지역 계획기(궤적 최적화용)가 ESDF라는 단일한 맵 표현을 공유**한다는 점이다. 전역 계획기는 ESDF의 거리 값을 이용해 샘플링된 노드의 유효성을 이산적으로(충돌/비충돌) 판단하고, 지역 계획기는 동일한 ESDF의 거리 값과 그 기울기를 이용해 궤적 최적화를 위한 연속적인 비용 함수를 계산한다. 이는 맵 표현 방식이 이원화될 경우 발생할 수 있는 데이터 불일치 문제나 복잡한 데이터 변환 과정의 필요성을 원천적으로 제거한다. 이처럼 ESDF가 서로 다른 두 가지 목적(이산적 충돌 체크, 연속적 최적화)에 모두 효과적으로 사용될 수 있다는 사실은, ESDF가 얼마나 강력하고 다재다능한 맵 표현 방식인지를 보여주는 대표적인 사례라 할 수 있다.

## 3.  ESDF 기반 궤적 최적화: Loco Planner (ESDF-Based Trajectory Optimization: The Loco Planner)

`voxblox_loco_planner`는 `mav_voxblox_planning` 프레임워크에서 실제로 MAV가 따라갈 부드럽고 안전한 궤적을 생성하는 핵심 엔진이다. 이 플래너는 "Continuous-Time Trajectory Optimization for Online UAV Replanning" 2 논문에서 제안된 궤적 최적화 기법을 구현한 것이다.

### 3.1  연속 시간 궤적 최적화 문제 정식화 (Formulation of the Continuous-Time Trajectory Optimization Problem)

로봇의 궤적을 이산적인 점들의 나열이 아닌, 시간에 대한 연속적인 함수로 표현하는 것이 이 접근법의 핵심이다. 이를 위해 궤적을 여러 개의 N차 다항식 조각(polynomial segments)들을 부드럽게 이어붙인 스플라인(spline) 형태로 표현한다.2 이러한 표현 방식은 몇 가지 중요한 장점을 가진다. 첫째, 궤적 전체를 소수의 다항식 계수만으로 간결하게 표현할 수 있다. 둘째, 임의의 시간 $t$에서의 위치, 속도, 가속도 등 동역학적 상태를 미분을 통해 쉽게 계산할 수 있다.

$K$차원 공간에서 $N$차 다항식으로 표현되는 한 조각의 궤적은 다음과 같다:
$$
f_k(t) = a_0 + a_1t + a_2t^2 + \dots + a_Nt^N
$$
여기서 $p_k = [a_0, a_1, \dots, a_N]^T$는 다항식의 계수 벡터이다. 궤적 최적화 문제의 목표는 결국 MAV가 따라가기에 가장 '좋은' 궤적을 만드는 최적의 계수 집합 $p^*$를 찾는 것이다.2

### 3.2  목적 함수: 평활도와 충돌 비용 (The Objective Function: Smoothness and Collision Cost)

'좋은' 궤적이란 무엇인가? Loco Planner는 이를 수학적인 목적 함수(objective function) $J$로 정의하고, 이 함수 값을 최소화하는 것을 목표로 삼는다. 목적 함수는 일반적으로 평활도 비용($J_{smooth}$)과 충돌 비용($J_{collision}$)이라는 두 가지 상충하는 요소의 가중합으로 구성된다.2
$$
J = J_{smooth} + \lambda J_{collision}
$$

- **평활도 비용 ($J_{smooth}$):** 이 비용은 궤적이 얼마나 '부드러운지'를 정량화한다. 물리적으로 이는 MAV가 불필요하게 급가속, 급정지하는 것을 막아 에너지 소모를 줄이고 안정적인 비행을 유도하는 역할을 한다. 수학적으로는 주로 가속도나 저크(jerk, 가속도의 변화율)와 같은 고차 미분값의 제곱을 궤적 전체 시간에 대해 적분하여 계산한다.2

- **충돌 비용 ($J_{collision}$):** 이 비용은 궤적이 장애물로부터 얼마나 안전하게 떨어져 있는지를 나타낸다. 바로 이 지점에서 ESDF 맵이 결정적인 역할을 한다. 궤적 상의 임의의 점 $x$에서의 충돌 비용 $c(x)$는 해당 위치의 ESDF 값 $d(x)$와 사용자가 설정한 안전 마진 $\epsilon$을 이용하여 계산된다. Oleynikova 등의 논문에서 제안된 비용 함수는 다음과 같다 4:
  $$
  c(x) = \begin{cases} -d(x) + \frac{1}{2}\epsilon & \text{if } d(x) < 0 \\ \frac{1}{2\epsilon}(d(x)-\epsilon)^2 & \text{if } 0 \le d(x) \le \epsilon \\ 0 & \text{otherwise} \end{cases}
  $$
  이 함수는 매우 영리하게 설계되었다. 장애물 내부($d(x) < 0$)에서는 거리에 비례하여 비용이 선형적으로 증가하고, 안전 마진 내의 자유 공간($0 \le d(x) \le \epsilon$)에서는 거리가 0에 가까워질수록 비용이 2차 함수 형태로 부드럽게 증가한다. 그리고 안전 마진 밖에서는 비용이 0이 된다. 이처럼 비용 함수가 모든 구간에서 연속적이고 미분 가능하기 때문에, 경사 하강법(gradient-based optimization)을 원활하게 적용할 수 있다.

### 3.3  최적화 기법 및 실시간 적용 (Optimization Technique and Real-Time Application)

정의된 목적 함수 $J$는 일반적으로 비선형(nonlinear)이고 여러 개의 지역 최저점(local minima)을 갖는 비볼록(non-convex) 함수이다. 따라서 전역 최적해(global optimum)를 찾는 것은 매우 어렵고 시간이 많이 걸린다. 실시간 재계획이 목표인 Loco Planner는 대신, 현재 궤적에서 시작하여 비용을 낮추는 방향으로 빠르게 개선해 나가는 지역 최적화(local optimization) 기법을 사용한다. 대표적으로 경사 하강법(gradient descent)이나 이보다 수렴 속도가 빠른 준-뉴턴 방법(quasi-Newton method)인 BFGS 등이 사용된다.4

이 과정에서 ESDF의 진정한 위력이 발휘된다. 최적화 알고리즘은 비용 함수 $J$의 기울기, 즉 그래디언트($\nabla J$)를 계산하여 궤적의 계수들을 어느 방향으로 수정해야 비용이 가장 빠르게 감소하는지를 알아내야 한다. 평활도 비용의 그래디언트는 다항식의 형태로 쉽게 계산할 수 있으며, 충돌 비용의 그래디언트($\nabla c(x)$)는 ESDF 맵의 그래디언트로부터 직접 얻을 수 있다. ESDF의 그래디언트는 물리적으로 각 지점에서 가장 가까운 장애물 표면을 향하는 방향 벡터를 나타내므로, 이는 궤적을 장애물로부터 밀어내는 '힘'과 같은 역할을 하여 최적화 과정을 효율적으로 이끈다.

이러한 최적화 과정을 통해, 전체 계획 사이클은 50ms 이내에 완료될 수 있으며, 이는 MAV가 4Hz 이상의 빠른 속도로 주변 환경 변화에 대응하며 온라인으로 궤적을 수정하는 것을 가능하게 한다.2

## 4.  시스템 구축 및 실제 적용 (System Implementation and Practical Application)

`mav_voxblox_planning`은 학술 연구뿐만 아니라 실제 로봇 시스템에 적용하는 것을 목표로 개발되었으므로, 구체적인 하드웨어 구성과 소프트웨어 설치, 그리고 시각화 도구 사용법이 잘 문서화되어 있다.

### 4.1  표준 하드웨어 구성 (Standard Hardware Configuration)

이 프레임워크를 실제 MAV에 탑재하여 구동하기 위한 일반적인 하드웨어 구성은 다음과 같다.

- **카메라:** Intel RealSense D400 시리즈 깊이 카메라, 특히 D435 또는 D435i 모델이 널리 사용된다. 이 카메라들은 스테레오 비전 기술을 이용하여 실시간으로 깊이 정보(포인트 클라우드)를 생성하며, `librealsense`라는 공식 SDK를 통해 ROS 환경에서 쉽게 데이터를 얻을 수 있다.5
- **컴패니언 컴퓨터 (Companion Computer):** MAV의 주 비행 컨트롤러(Flight Controller)는 비행 안정화에 집중하고, 복잡한 연산이 필요한 맵핑 및 경로 계획 알고리즘은 별도의 고성능 소형 컴퓨터에서 처리한다. 이를 컴패니언 컴퓨터라 하며, UP Squared 보드(Celeron 또는 ATOM CPU 버전)가 공식적으로 지원되고 권장된다.16
- **연결:** 카메라는 고속 데이터 전송을 위해 USB 3.0 포트를 통해 컴패니언 컴퓨터에 연결된다. 컴패니언 컴퓨터는 계산된 최종 비행 명령(궤적)을 비행 컨트롤러(예: Pixhawk)에 전달해야 하는데, 이는 주로 직렬 포트(Serial Port)를 통해 MAVLink라는 표준 통신 프로토콜을 사용하여 이루어진다. 컴패니언 컴퓨터의 시리얼 포트와 비행 컨트롤러의 TELEM 포트를 연결하는 것이 일반적이다.16

### 4.2  소프트웨어 스택 및 설치 과정 (Software Stack and Installation Process)

소프트웨어 환경 구축은 전형적인 ROS 기반 프로젝트의 절차를 따른다.

- **운영체제 및 ROS:** Ubuntu 16.04 운영체제와 ROS Kinetic 배포판, 또는 그 이상의 버전이 요구된다.5
- **의존성 라이브러리:** `mav_voxblox_planning`과 그 구성 요소들을 컴파일하고 실행하기 위해서는 여러 외부 라이브러리가 필요하다. ROS의 `catkin` 빌드 시스템, Intel RealSense 카메라를 위한 `librealsense2` SDK, 포인트 클라우드 처리를 위한 PCL(Point Cloud Library), 행렬 및 벡터 연산을 위한 Eigen 라이브러리 등이 대표적이다.5
- **빌드 과정:** GitHub 리포지토리에서 소스 코드를 클론한 후, `catkin_make` 또는 `catkin build` 명령어를 사용하여 전체 ROS 워크스페이스를 컴파일한다. 이 과정에서 모든 의존성이 올바르게 설치되어 있어야 한다.

### 4.3  데모 실행 및 시각화 (Running Demos and Visualization)

`mav_voxblox_planning`은 다양한 데모 시나리오를 제공하여 사용자가 각 플래너의 기능을 쉽게 테스트해볼 수 있도록 한다. 데모는 주로 `roslaunch` 파일을 통해 실행된다. 예를 들어, `rrt_saved_map.launch` 파일은 미리 생성되어 저장된 ESDF 맵 파일을 불러온 후, 그 위에서 RRT 플래너를 실행하는 데모이다.5

실행된 알고리즘의 작동을 실시간으로 확인하고 제어하는 데에는 **RViz**가 핵심적인 역할을 한다.

- **`PlanningPanel`:** `mav_voxblox_planning`이 제공하는 커스텀 RViz 패널이다. 사용자는 이 패널을 통해 사용하고자 하는 플래너의 ROS 서비스 이름(예: `voxblox_rrt_planner`)을 입력하고, 'Plan' 버튼을 눌러 경로 계획을 요청하거나 중지할 수 있다.5
- **`InteractiveMarkers`:** 3D 시각화 창에 나타나는 상호작용 가능한 화살표 마커이다. 사용자는 이 마커를 마우스로 직접 드래그하여 로봇의 출발점(start)과 목표점(goal)의 위치 및 방향을 직관적으로 설정할 수 있다.5
- **`VoxbloxMesh` / `MarkerArray`:** Voxblox가 생성하는 3D 환경 메쉬, ESDF 맵의 단면, 그리고 플래너가 계산한 최종 경로 등을 시각화하는 데 사용되는 RViz 디스플레이 타입이다. 이를 통해 사용자는 맵이 어떻게 구축되고, 로봇이 어떤 경로를 따라 비행할 계획인지 명확하게 확인할 수 있다.5

## 5.  학술적 맥락과 후속 연구 동향 (Academic Context and Follow-up Research Trends)

`mav_voxblox_planning`은 ESDF 기반 실시간 경로 계획 분야에서 중요한 이정표를 세웠지만, 동시에 후속 연구들이 해결해야 할 명확한 과제들을 남겼다. 이 시스템의 한계를 분석하고, 이를 극복하기 위해 등장한 FIESTA와 EGO-Planner 같은 대표적인 후속 연구들의 흐름을 살펴보는 것은 MAV 자율 비행 기술의 발전 과정을 이해하는 데 매우 중요하다.

### 5.1  Voxblox 접근법의 내재적 한계 (Inherent Limitations of the Voxblox Approach)

Voxblox 기반의 계획 방식은 선구적이었으나, 두 가지 주요한 내재적 한계를 가지고 있었다.

- **과도한 계산 비용:** ESDF 맵을 유지하기 위해서는 맵의 일부가 변경될 때마다 파동 전파 알고리즘을 통해 주변 복셀들을 업데이트해야 한다. 이는 상당한 계산 자원을 소모하는 작업이다. 특히, 궤적 최적화 과정은 실제로는 맵의 아주 제한된 일부 영역에 대한 정보만을 필요로 함에도 불구하고, Voxblox는 궤적과 전혀 상관없는 영역의 ESDF 값까지 유지하고 업데이트하는 비효율성을 안고 있었다.18
- **정확도 문제:** 앞서 2.3절에서 상세히 분석했듯이, Voxblox의 ESDF는 두 가지 주요 오차 원인을 가진다. 첫째는 TSDF를 생성할 때 사용하는 '투영 거리'가 실제 유클리드 거리가 아니라는 점, 둘째는 거리 전파 시 '준-유클리드' 방식을 사용하여 거리를 근사한다는 점이다. 이러한 오차는 MAV가 장애물에 매우 근접하여 비행해야 하는 공격적인(aggressive) 기동 상황에서 안전성을 심각하게 저해할 수 있는 잠재적 위험 요소가 된다.11

### 5.2  성능 및 정확도 개선 연구: FIESTA (Research on Improving Performance and Accuracy: FIESTA)

FIESTA(Fast Incremental Euclidean DiSTAnce Fields)는 이름에서 알 수 있듯이, Voxblox의 ESDF 생성 속도와 정확도 문제를 직접적으로 겨냥하여 개발된 시스템이다.11

- **TSDF 의존성 탈피:** FIESTA의 가장 큰 구조적 차이점은 TSDF라는 중간 단계를 완전히 제거한 것이다. 대신, 더 간단한 맵 표현 방식인 점유 격자 지도(occupancy grid map)로부터 직접 ESDF를 증분적으로 계산한다. 이를 통해 TSDF의 투영 거리에서 비롯되는 고질적인 오차 문제를 원천적으로 해결했다.11
- **효율적인 업데이트 메커니즘:** FIESTA는 ESDF 업데이트를 위해 장애물이 새로 추가되는 경우(`insert` 큐)와 기존 장애물이 제거되는 경우(`delete` 큐)를 처리하는 두 개의 독립적인 큐를 도입했다. 이 구조는 너비 우선 탐색(BFS) 기반의 업데이트가 필요한 최소한의 노드에만 적용되도록 하여 계산 효율을 극대화한다.11
- **정확도 향상:** 거리 전파 시 Voxblox의 준-유클리드 근사 방식 대신, 실제 유클리드 거리를 계산하여 ESDF의 정확도를 크게 향상시켰다.8 이러한 개선들을 통해 FIESTA는 Voxblox 대비 ESDF 계산 속도를 최대 4배까지 향상시키는 성과를 거두었다.8

### 5.3  ESDF 의존성 탈피 패러다임: EGO-Planner (The Paradigm Shift Away from ESDF-Dependency: EGO-Planner)

EGO-Planner는 한 걸음 더 나아가, "지역 경로 계획을 위해 과연 ESDF 맵이 필수적인가?"라는 더욱 근본적인 질문을 던진다. ESDF 맵을 생성하고 유지하는 것 자체가 전체 경로 계획 과정의 병목(bottleneck)이라는 문제의식에서 출발하여, **ESDF 맵 없이** 경사 하강법 기반의 궤적 최적화를 수행하는 혁신적인 프레임워크를 제안했다.18

- **새로운 충돌 비용 공식:** EGO-Planner는 ESDF의 거리 값에 의존하는 대신, 현재의 (충돌 위험이 있는) 궤적과 충돌하지 않는 안전한 '가이드 경로(guiding path)'를 비교하는 방식으로 충돌 비용과 그래디언트를 '추정'한다. 이는 사실상 원시 센서 데이터인 포인트 클라우드로부터 직접 장애물 표면을 인지하고, 궤적을 장애물로부터 밀어내는 가상의 힘을 계산하는 것과 유사하다.18
- **압도적인 성능 향상:** ESDF 맵을 유지보수하는 데 소요되던 막대한 계산 비용(일부 연구에서는 전체 계획 시간의 약 70%를 차지한다고 분석됨)을 완전히 제거함으로써, EGO-Planner는 기존 ESDF 기반 방법들보다 한 자릿수 이상(order of magnitude) 빠른 계산 속도를 달성했다.19

Voxblox에서 FIESTA, 그리고 EGO-Planner로 이어지는 이 연구의 흐름은 로보틱스 모션 플래닝 분야에서 문제가 해결되고 기술이 발전해 나가는 전형적인 단계를 명확하게 보여준다. 첫 번째 단계는 **개념 증명(Proof of Concept)**으로, Voxblox는 ESDF 기반 실시간 궤적 최적화라는 아이디어의 실현 가능성을 열었다. 두 번째 단계는 **최적화(Optimization)**로, FIESTA는 기존 방식의 명확한 병목(ESDF 생성 속도 및 정확도)을 집중적으로 개선했다. 마지막 세 번째 단계는 **패러다임 전환(Paradigm Shift)**으로, EGO-Planner는 문제의 원인(ESDF 의존성) 자체를 제거하는 새로운 접근법을 제시했다. 이러한 발전 과정은 `mav_voxblox_planning`이 단순히 과거의 유산이 아니라, 후속 혁신을 이끌어낸 매우 중요한 학술적 발판이었음을 명백히 증명한다.

| 특징 (Feature)    | **Voxblox (mav_voxblox_planning)**                           | **FIESTA**                                                   | **EGO-Planner**                                              |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 아이디어** | TSDF로부터 ESDF를 증분적으로 생성하여 궤적 최적화에 활용     | 점유 격자 지도에서 직접 ESDF를 더 빠르고 정확하게 생성       | ESDF 맵 생성 자체를 생략하고 충돌 그래디언트를 직접 추정     |
| **맵 표현**       | TSDF -> ESDF                                                 | Occupancy Grid -> ESDF                                       | ESDF 없음. 원시 센서 데이터(포인트 클라우드) 사용            |
| **장점**          | - 선구적인 통합 시스템 - 우수한 시각화 메쉬 - 검증된 오픈소스 | - Voxblox 대비 높은 속도 - 높은 정확도 (실제 유클리드 거리) - 투영 거리 오차 없음 | - 압도적으로 빠른 계산 속도 - ESDF 유지보수 비용 없음 - 경량화된 시스템 |
| **단점**          | - ESDF 생성의 계산 비용 - 정확도 한계 (Projective, Quasi-Euclidean) | - 여전히 ESDF 맵 유지보수 필요 - 시각화용 메쉬 품질은 TSDF 기반보다 떨어질 수 있음 | - 그래디언트가 '추정치'이므로 지역 최저점에 빠질 가능성 - 전역적인 맵 정보 활용에 한계 |

## 6.  종합 고찰 및 결론 (Overall Assessment and Conclusion)

### 6.1  `mav_voxblox_planning`의 학술적 의의와 공헌

`mav_voxblox_planning`은 MAV 자율 비행 연구 분야에서 중요한 학술적 공헌을 남겼다. 이 시스템은 ESDF 기반의 실시간 궤적 최적화라는, 당시로서는 선도적인 개념을 실제 MAV 하드웨어 플랫폼 상에서 성공적으로 구현하고 통합한 최초의 시스템 중 하나였다. 더 중요한 것은, 이 모든 성과를 연구 커뮤니티가 자유롭게 사용하고 발전시킬 수 있도록 완전한 오픈소스 형태로 공개했다는 점이다.5

이러한 공개는 후속 연구의 발전을 크게 촉진하는 기폭제가 되었다. `mav_voxblox_planning`은 후발 주자들이 해결해야 할 명확한 문제(ESDF 생성의 계산 비용, 정확도 한계)를 정의해주었고, 새로운 알고리즘의 성능을 비교하고 평가할 수 있는 구체적인 기준점(benchmark) 역할을 수행했다. FIESTA와 EGO-Planner 같은 혁신적인 연구들이 Voxblox의 한계를 명시적으로 언급하며 자신들의 장점을 부각시킬 수 있었던 것도, 바로 `mav_voxblox_planning`이라는 견고한 선행 연구가 있었기 때문이다.19 따라서 이 시스템은 단순히 하나의 성공적인 프로젝트를 넘어, 해당 연구 분야 전체의 발전을 이끌어낸 중요한 학술적 발판으로 평가받아야 한다.

### 6.2  현재 개발 상태 및 실용적 고려사항

`mav_voxblox_planning`의 GitHub 리포지토리 README 파일에는 "이 패키지는 활발하게 개발 중입니다(THIS PACKAGE IS UNDER ACTIVE DEVELOPMENT!)"라고 명시되어 있다.5 하지만 객관적인 데이터를 살펴보면 다른 결론에 이르게 된다. 리포지토리의 활동 기록(Activity)을 확인하면, 최근 몇 년간 의미 있는 코드 변경이나 주요 업데이트가 거의 이루어지지 않았음을 알 수 있다.23

문서상의 주장과 실제 개발 활동 사이의 이러한 명백한 불일치는 이 프로젝트의 현재 상태에 대한 중요한 정보를 시사한다. 즉, `mav_voxblox_planning`은 더 이상 활발히 개발되는 프로젝트가 아니며, 사실상 '유지보수 모드'에 있거나 '레거시(legacy)' 프로젝트로 전환되었을 가능성이 매우 높다.

따라서, 새로운 연구 프로젝트나 상용 제품 개발의 기반으로 `mav_voxblox_planning`을 채택하는 것은 매우 신중하게 결정해야 할 문제이다. 최신 버전의 ROS(예: ROS 2)와의 호환성 문제, 잠재적인 버그 수정의 어려움, 커뮤니티로부터의 기술 지원 부재 등 다양한 실용적인 문제에 직면할 수 있다. 기술의 선구적인 가치와는 별개로, 프로젝트의 지속적인 유지보수와 커뮤니티 지원은 기술 채택에 있어 결정적인 요소이다. 현재 시점에서는 FIESTA나 EGO-Planner와 같이 보다 활발하게 개발이 이루어지고 있으며 성능적으로도 우수하다고 입증된 대안들을 우선적으로 고려하는 것이 현명한 선택일 것이다.

### 6.3  미래 연구 방향 제언

`mav_voxblox_planning`과 그 후속 연구들이 제시한 아이디어들은 여전히 미래 연구를 위한 풍부한 영감을 제공한다.

- **하이브리드 접근법:** EGO-Planner의 압도적인 속도와 경량성, 그리고 Voxblox나 FIESTA가 제공하는 풍부한 전역 맵 정보(ESDF)의 장점을 결합하는 하이브리드 시스템이 유망해 보인다. 예를 들어, 짧은 시간 범위의 지역 충돌 회피는 EGO-Planner와 같이 ESDF 없이 빠르게 처리하되, 장기적인 자율 탐사(exploration)나 고수준의 의미론적(semantic) 경로 계획이 필요할 때는 전역 ESDF 맵을 활용하는 방식으로 역할을 분담하는 아키텍처를 구상할 수 있다.
- **학습 기반 방법과의 통합:** 최근 로보틱스 분야의 가장 큰 화두인 심층 학습(Deep Learning) 기술을 접목하는 연구 또한 활발히 진행될 수 있다. 예를 들어, 센서가 관측하지 못한 미관측 영역(unobserved space)에 대해 주변 환경 정보를 바탕으로 ESDF 값을 예측하여 채워 넣거나 24, 더 나아가 실제 주행 데이터를 학습하여 특정 환경에 최적화된 충돌 비용 그래디언트를 생성하는 신경망 모델을 개발하는 방향으로 발전시킬 수 있다. 이러한 접근법들은 기존의 기하학적, 최적화 기반 방법들의 한계를 보완하고 MAV 자율 비행 기술을 한 단계 더 높은 수준으로 끌어올릴 잠재력을 가지고 있다.

#### 6.3.1 Works cited

1. Voxblox: Incremental 3D Euclidean Signed ... - Helen Oleynikova, accessed August 8, 2025, https://helenol.github.io/publications/iros_2017_voxblox.pdf
2. Continuous-Time Trajectory Optimization for ... - Helen Oleynikova, accessed August 8, 2025, https://helenol.github.io/publications/iros_2016_replanning.pdf
3. Mapping for online path planning and 3D reconstruction, accessed August 8, 2025, https://ethz.ch/content/dam/ethz/special-interest/baug/igp/photogrammetry-remote-sensing-dam/documents/pdf/Student_Theses/MA_YuePan.pdf
4. (PDF) Continuous-Time Trajectory Optimization for Online UAV Replanning - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/309291325_Continuous-Time_Trajectory_Optimization_for_Online_UAV_Replanning
5. ethz-asl/mav_voxblox_planning: MAV planning tools using ... - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/mav_voxblox_planning
6. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, accessed August 8, 2025, https://arxiv.org/abs/1611.03631
7. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, accessed August 8, 2025, https://www.youtube.com/watch?v=ZGvnGFnTVR8
8. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 8, 2025, https://arxiv.org/html/2311.00626v2
9. ethz-asl/voxblox: A library for flexible voxel-based mapping, mainly focusing on truncated and Euclidean signed distance fields. - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/voxblox
10. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, accessed August 8, 2025, https://huggingface.co/papers/1611.03631
11. [1903.02144] FIESTA: Fast Incremental Euclidean Distance Fields for Online Motion Planning of Aerial Robots - ar5iv, accessed August 8, 2025, https://ar5iv.labs.arxiv.org/html/1903.02144
12. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 8, 2025, https://arxiv.org/html/2407.09649v1
13. mav_voxblox_planning/mav_local_planner/include/mav_local_planner/mav_local_planner.h at master · ethz-asl/mav_voxblox_planning - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/mav_voxblox_planning/blob/master/mav_local_planner/include/mav_local_planner/mav_local_planner.h
14. Continuous-time trajectory optimization for online UAV replanning - Research Collection - ETH Zürich, accessed August 8, 2025, https://www.research-collection.ethz.ch/bitstream/20.500.11850/124980/1/eth-50476-01.pdf
15. IntelRealSense/librealsense: Intel® RealSense™ SDK - GitHub, accessed August 8, 2025, https://github.com/IntelRealSense/librealsense
16. Intel RealSense Depth Camera — Copter documentation - ArduPilot, accessed August 8, 2025, https://ardupilot.org/copter/docs/common-realsense-depth-camera.html
17. HKUST-Aerial-Robotics/FIESTA: Fast Incremental Euclidean Distance Fields for Online Motion Planning of Aerial Robots - GitHub, accessed August 8, 2025, https://github.com/HKUST-Aerial-Robotics/FIESTA
18. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/343786586_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
19. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/348028324_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
20. EGO-Planner: An ESDF-Free Gradient-Based Local Planner for Quadrotors, accessed August 8, 2025, https://www.semanticscholar.org/paper/EGO-Planner%3A-An-ESDF-Free-Gradient-Based-Local-for-Zhou-Wang/6fd8ff3bfee9ba2b57eea07780da868964a47486
21. FIESTA: Fast Incremental Euclidean Distance Fields for Online Motion Planning of Aerial Robots - arXiv, accessed August 8, 2025, https://arxiv.org/abs/1903.02144
22. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 8, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
23. Activity · ethz-asl/mav_voxblox_planning - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/mav_voxblox_planning/activity
24. Predicting Unobserved Space for Planning via Depth Map Augmentation - Semantic Scholar, accessed August 8, 2025, https://www.semanticscholar.org/paper/Predicting-Unobserved-Space-for-Planning-via-Depth-Fehr-Taubner/d60bc885e062cf60f25eb2a5ce4de071b9c69c7d
25. Informed Sampling Exploration Path Planner for 3D Reconstruction of Large Scenes, accessed August 8, 2025, https://www.semanticscholar.org/paper/Informed-Sampling-Exploration-Path-Planner-for-3D-Kompis-Bartolomei/2a48511e56323f8be6473f16a8893b1b48aae9e5


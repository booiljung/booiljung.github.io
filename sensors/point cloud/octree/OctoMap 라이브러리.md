# OctoMap 라이브러리

## 1. 서론

### 0.1 A. 로보틱스 분야 3차원 환경 인식의 패러다임

자율 주행 자동차, 무인 항공기(UAV), 로봇 팔 조작 등 현대 로보틱스 응용 분야의 성공은 주변 환경을 얼마나 정확하고 신속하게 인식하고 이해하는지에 달려있다. 특히, 3차원 공간에 대한 완전한 부피적(volumetric) 표현은 로봇이 지능적인 결정을 내리는 데 있어 필수적인 전제 조건이다.1 이는 단순히 눈앞의 장애물을 감지하는 수준을 넘어, 안전하고 효율적인 경로를 계획하고, 로봇 팔이 도달 가능한 작업 공간을 분석하며, 미지의 환경을 자율적으로 탐사하는 등 고차원적인 지능적 행동의 기반이 되기 때문이다.4 로봇이 자신이 차지하는 공간 외에도 비어있는 공간(free space)과 아직 탐사하지 않은 미지의 공간(unknown space)을 명확히 구분할 수 있을 때, 비로소 진정한 의미의 자율성이 실현될 수 있다.

### 0.2 B. 기존 3D 맵핑 기술의 근본적 한계

OctoMap이 등장하기 이전, 3차원 환경을 모델링하려는 여러 시도가 있었으나 각각 명확한 한계를 가지고 있었다.

첫째, **고정 복셀 그리드(Rigid Voxel Grids)** 방식은 공간을 일정한 크기의 정육면체(복셀)로 나누어 표현한다. 이 방식은 직관적이고 구현이 용이하지만, 치명적인 단점을 내포한다. 가장 큰 문제는 막대한 메모리 요구량이다.1 맵의 전체 경계를 포함하도록 미리 메모리를 할당해야 하므로, 실제 장애물이나 구조물이 거의 없는 넓은 공간이라 할지라도 모든 복셀에 대한 메모리를 점유한다. 이로 인해 대규모 실외 환경이나 고해상도 맵핑이 필요한 경우 메모리 사용량이 감당할 수 없는 수준으로 증가하며, 맵의 크기를 확장해야 할 때마다 발생하는 비싼 데이터 복사 비용 또한 실시간 응용에 큰 부담이 되었다.1

둘째, **포인트 클라우드(Point Clouds)** 방식은 레이저 스캐너나 깊이 카메라와 같은 센서가 측정한 3차원 점들의 집합을 그대로 저장한다. 이 방식은 원시 데이터(raw data)를 보존한다는 장점이 있지만, 두 가지 근본적인 문제를 해결하지 못한다. 가장 치명적인 단점은 '자유 공간'과 '미탐사 공간'을 구분할 수 없다는 점이다.1 센서 광선이 도달한 지점은 장애물로 간주되지만, 센서와 그 지점 사이의 공간이 비어있는지, 아니면 단지 관측되지 않았을 뿐인지 알 수 없다. 이는 안전한 경로 계획에 필수적인 정보를 제공하지 못함을 의미한다. 또한, 센서 노이즈에 취약하며, 측정 횟수가 늘어날수록 데이터가 무한정 누적되어 메모리 사용량에 상한선이 없다는 문제도 있다.6

셋째, **2.5D 맵(Elevation Maps)** 방식은 2차원 그리드의 각 셀에 높이 값을 저장하는 형태이다. 이는 지면의 고저차를 표현하는 데 효과적이어서 야외 지형 주행 등 특정 응용 분야에서는 충분히 유용하다.6 하지만 나무 가지, 다리 밑, 건물의 처마와 같이 수직적으로 복잡한 구조물을 표현할 수 없다는 명백한 한계를 가진다.1 이는 로봇이 활동할 수 있는 공간을 완전한 3차원으로 이해해야 하는 대부분의 현대 로보틱스 응용에는 부적합하다.

### 0.3 C. OctoMap의 등장: 패러다임의 전환

OctoMap은 2010년 처음 소개된 이래, 앞서 언급된 기존 기술들의 단점들을 극복하기 위해 설계된 혁신적인 통합 프레임워크다.1 OctoMap의 핵심 목표는 명확했다. 첫째, **확률적 점유 추정(Probabilistic Occupancy Estimation)**을 통해 센서 측정에 내재된 불확실성과 노이즈에 강건하게 대응한다. 둘째, 

**옥트리(Octree)** 자료구조를 기반으로 3차원 공간을 효율적으로 표현하여 메모리 사용량을 최소화한다. 셋째, 이 두 가지를 결합하여 실시간으로 업데이트 가능하며, 유연하고(flexible), 컴팩트한(compact) 3D 환경 모델을 제공하는 것이다.2

기존 기술들이 '메모리', '공간 표현력', '불확실성 처리' 중 하나 이상의 약점을 가졌던 반면, OctoMap은 이 문제들을 개별적으로 접근하지 않았다. 대신, 옥트리라는 효율적인 공간 분할 구조와 베이즈 필터 기반의 확률 모델을 하나의 프레임워크 안에서 유기적으로 결합함으로써, 3D 맵핑의 핵심 요구사항들을 종합적으로 해결하는 시스템 아키텍처의 성공적인 사례를 제시했다. 이는 단편적인 기술 개선을 넘어, 문제 해결을 위한 종합적 설계의 결과물이라 할 수 있다.

더욱 중요한 것은 OctoMap이 오픈소스로 공개되었다는 점이다. 당시 로보틱스 연구 분야에서는 신뢰성 있고 효율적인 3D 맵핑 구현체가 부족하여 많은 연구팀이 "기초적인 소프트웨어 구성 요소를 재창조"하는 데 시간과 노력을 낭비하고 있었고, 이는 연구 발전의 병목 현상으로 지적되었다.1 OctoMap이 관대한 BSD 라이선스를 따르는 C++ 라이브러리로 공개되면서 1, 연구자들은 '바퀴를 재발명'하는 대신, 안정적인 3D 맵을 기반으로 경로 계획, 자율 탐사, 객체 인식 등 더 고차원적인 문제에 집중할 수 있게 되었다. 이처럼 OctoMap은 하나의 뛰어난 알고리즘을 넘어, 로보틱스 연구 생태계 전체를 활성화하고 그 발전을 가속화하는 중요한 '공공재(public good)'로서의 역할을 수행했다.8

## 1.  OctoMap의 이론적 기반 및 핵심 원리

OctoMap의 효율성과 강건함은 두 가지 핵심 이론, 즉 공간 표현을 위한 '옥트리 자료구조'와 불확실성 처리를 위한 '확률적 점유 모델링'의 정교한 결합에서 비롯된다.

### 1.1 A. 옥트리(Octree) 자료구조: 공간의 계층적 표현

옥트리는 3차원 공간을 효율적으로 표현하기 위한 계층적 자료구조이다. OctoMap은 이 구조를 채택하여 메모리 문제를 근본적으로 해결한다.

#### 1.1.1 공간 분할 원리

옥트리의 기본 아이디어는 3차원 공간 전체를 나타내는 하나의 큰 정육면체(루트 노드)에서 시작하여, 공간을 재귀적으로 8개의 동일한 크기의 하위 정육면체(자식 노드, 또는 Octant)로 분할하는 것이다.10 각 정육면체 노드는 '복셀(Voxel)'이라 불린다. 이 재귀적 분할 과정은 사용자가 미리 정의한 최소 복셀 크기, 즉 맵의 최종 해상도(resolution)에 도달할 때까지 계속된다.11 이 계층적 구조 덕분에, 필요에 따라 특정 깊이에서 트리를 잘라내어 다양한 해상도의 맵을 유연하게 얻을 수 있다.

#### 1.1.2 메모리 효율성 달성 원리

옥트리가 고정 그리드 방식에 비해 압도적인 메모리 효율성을 보이는 이유는 두 가지 핵심적인 최적화 기법 덕분이다.

첫째, **프루닝(Pruning)** 기법이다. 만약 어떤 노드의 8개 자식 노드가 모두 동일한 점유 상태(예: 모두 '점유' 또는 모두 '비점유')를 가질 경우, 이 8개의 자식 노드를 메모리에서 제거하고 부모 노드 하나로 상태를 통합하여 표현한다.1 예를 들어, 넓은 빈 공간이나 거대한 빌딩 벽면은 수많은 작은 복셀들로 채워지는 대신, 소수의 큰 복셀 노드로 압축되어 표현될 수 있다. 이는 특히 장애물이 드문드문 있거나 넓은 개활지가 포함된 환경에서 극적인 메모리 절감 효과를 가져온다.

둘째, **동적 확장(Dynamic Expansion)** 방식이다. 고정 그리드와 달리, OctoMap은 맵의 전체 범위를 미리 가정하고 메모리를 할당하지 않는다.4 대신, 로봇의 센서 데이터가 들어오는 영역에 대해서만 동적으로 노드를 생성하고 트리를 확장해 나간다. 이는 탐사되지 않은 미지의 공간에 대해서는 메모리를 전혀 사용하지 않음을 의미하며, 맵의 크기나 형태를 미리 알 수 없는 자율 탐사 시나리오에 매우 적합한 특성이다.5

### 1.2 B. 확률적 점유 격자 모델링 (Probabilistic Occupancy Grid Modeling)

실세계의 로봇 센서는 완벽하지 않다. LiDAR나 RGB-D 카메라로부터 얻는 측정값에는 항상 노이즈, 반사로 인한 오류, 움직이는 물체로 인한 오측정 등 다양한 불확실성이 포함된다.1 OctoMap은 이러한 불확실성을 다루기 위해 각 복셀의 상태를 '점유(occupied)' 또는 '비점유(free)'라는 이진(binary) 값으로 결정하지 않고, 해당 복셀이 점유되었을 확률 값으로 표현한다.4

#### 1.2.1 베이즈 필터 기반 업데이트

OctoMap은 새로운 센서 측정값(zt)이 주어질 때마다 각 복셀(n)의 점유 확률을 갱신하기 위해 베이즈 필터(Bayes filter)를 사용한다. 특정 복셀 n이 시간 t까지의 모든 측정값 $z_{1:t}$가 주어졌을 때 점유될 확률 $P(n | z_{1:t})$는 다음과 같은 재귀적인 식으로 업데이트된다.1
$$
P(n | z_{1:t}) = \left[ 1 + \frac{1-P(n|z_t)}{P(n|z_t)} \frac{1-P(n|z_{1:t-1})}{P(n|z_{1:t-1})} \frac{P(n)}{1-P(n)} \right]^{-1}
$$
여기서 $P(n | z_{1:t-1})$은 이전 시간까지의 믿음(prior belief), $P(n|z_t)$는 현재 측정값 zt에 대한 센서 모델, 그리고 $P(n)$은 초기 사전 확률(보통 0.5로 가정)을 의미한다.

#### 1.2.2 로그 오즈(Log-Odds) 표현

위의 확률 업데이트 식은 여러 개의 곱셈 연산을 포함하고 있어 계산 비용이 높고, 확률 값이 0 또는 1에 가까워질수록 수치적으로 불안정해질 수 있다. OctoMap은 이 문제를 해결하기 위해 확률을 직접 다루는 대신 **로그 오즈(log-odds)** 표현을 사용한다. 확률 p는 다음과 같이 로그 오즈 L로 변환된다.
$$
L(n) = \log\left(\frac{P(n)}{1-P(n)}\right)
$$
로그 오즈 표현의 가장 큰 장점은 확률 업데이트가 매우 간단한 덧셈 연산으로 바뀐다는 점이다.1
$$
L(n | z_{1:t}) = L(n | z_{1:t-1}) + L(n | z_t)
$$
이 변환 덕분에 각 복셀의 업데이트는 단 한 번의 덧셈으로 완료되어 계산이 매우 효율적이고 빨라진다. 각 노드는 확률 값 대신 이 로그 오즈 값을 저장하며, 필요할 때 언제든지 다시 확률 값으로 변환할 수 있다.

이처럼 OctoMap의 효율성은 단순히 하나의 기술에 기인하는 것이 아니다. 옥트리 구조는 '어디를 업데이트할 것인가'의 문제를 효율적으로 해결한다. 즉, 넓은 빈 공간과 같이 변화가 없는 영역은 상위 레벨 노드에서 한 번에 처리함으로써 불필요한 계산을 대폭 줄여준다. 동시에, 로그 오즈 업데이트 방식은 '어떻게 업데이트할 것인가'의 문제를 효율적으로 해결한다. 복잡한 확률 곱셈을 단순 덧셈으로 치환하여 각 노드 업데이트 자체의 계산 비용을 최소화한다. 이 두 가지 핵심 원리가 결합되어, OctoMap은 '불필요한 영역에 대한 계산은 생략'하고 '필요한 영역의 계산은 빠르게' 수행하는 이중의 최적화를 달성하는 것이다.

#### 1.2.3 클램핑 (Clamping)

OctoMap은 **클램핑**이라는 실용적인 기법을 도입하여 맵의 안정성과 강건성을 높인다. 각 복셀의 로그 오즈 값이 특정 상한 및 하한 임계값(clamping thresholds)에 도달하면, 그 노드는 '안정된(stable)' 상태로 간주되어 더 이상 업데이트되지 않는다.1 이 임계값은 ROS 환경에서 `sensor_model/[min|max]` 파라미터를 통해 조절할 수 있다.12

클램핑은 단순한 최적화를 넘어, 맵의 실용성을 보장하는 핵심 철학을 담고 있다. 이론적으로 베이즈 필터는 무한한 데이터를 통해 확률을 0 또는 1에 수렴시킬 수 있지만, 현실 세계는 동적 객체나 센서 오류와 같은 예측 불가능한 요소로 가득하다. 만약 클램핑이 없다면, 한때 벽으로 확신했던 공간(점유 확률이 1에 매우 가까움)을 사람이 지나갈 때, 그 공간이 '자유' 상태로 바뀌는 데 매우 오랜 시간이 걸릴 수 있다. 클램핑은 확률을 일정 범위 내로 제한함으로써, 맵이 새로운 정보에 적절히 반응할 수 있는 '유연성'을 부여한다. 이는 이론적 완벽성보다 실제 환경에서의 강건함(robustness)을 우선시하는 실용적인 설계 결정이다. 더 나아가, 이렇게 안정화된(clamped) 노드들은 앞서 설명한 프루닝(pruning)의 대상이 되어 메모리 압축을 더욱 촉진하는 직접적인 연관성을 가진다.1

## 2.  OctoMap 라이브러리 활용 및 생태계

OctoMap은 이론적 우수성뿐만 아니라, 개발자들이 쉽게 접근하고 활용할 수 있는 잘 구축된 소프트웨어 생태계를 통해 로보틱스 커뮤니티의 표준 도구로 자리 잡았다. 이 생태계는 독립적인 C++ 라이브러리, ROS와의 깊은 통합, 그리고 시각화 도구로 구성된다.

### 2.1 A. 핵심 C++ 라이브러리

OctoMap의 핵심은 플랫폼에 독립적인 C++ 라이브러리다. 이는 ROS와 같은 특정 프레임워크에 의존하지 않고도 사용할 수 있음을 의미한다.

- **설치 및 의존성:** 라이브러리를 빌드하기 위해서는 `cmake`와 `gcc`와 같은 표준 C++ 개발 환경만 필요하다.13 이는 최소한의 의존성으로 다양한 시스템에 쉽게 설치할 수 있게 한다. 3D 시각화 도구인 `octovis`를 함께 사용하고자 할 경우에만 `Qt`와 `QGLViewer` 라이브러리가 추가적으로 요구된다.13

- **프로젝트 통합:** OctoMap은 다른 프로젝트와의 통합을 매우 용이하게 설계했다. 라이브러리를 빌드하면 `octomap-config.cmake` 파일이 생성되는데, 이를 통해 다른 `CMake` 기반 프로젝트에서는 단 몇 줄의 코드로 OctoMap 라이브러리를 쉽게 찾고 링크할 수 있다.13

  ```CMake
  find_package(octomap REQUIRED)
  include_directories(${OCTOMAP_INCLUDE_DIRS})
  target_link_libraries(your_target ${OCTOMAP_LIBRARIES})
  ```

  이러한 간편한 통합 방식은 개발자가 자신의 프로젝트에 3D 맵핑 기능을 빠르게 추가할 수 있도록 돕는다.

- **주요 API:** 라이브러리의 핵심 클래스는 `octomap::OcTree`이다. 개발자는 이 클래스의 인스턴스를 생성하여 3D 맵을 관리한다. 주요 기능으로는 `insertPointCloud()` 함수를 통해 센서로부터 얻은 포인트 클라우드 데이터를 맵에 통합하는 것, `castRay()` 함수를 통해 특정 광선이 통과하는 복셀들을 계산하는 것, 그리고 `leaf_iterator`와 같은 반복자를 사용하여 트리의 모든 리프 노드를 효율적으로 순회하는 것 등이 있다.5

### 2.2 B. ROS(Robot Operating System) 통합

OctoMap의 진정한 잠재력은 세계 최대의 로봇 소프트웨어 플랫폼인 ROS와의 깊은 통합에서 발현된다.

- **핵심 패키지:** ROS 생태계 내에서 OctoMap은 여러 패키지로 모듈화되어 제공된다.

  - `octomap`: 핵심 C++ 라이브러리를 포함하는 ROS 패키지.9
  - `octomap_msgs`: `octomap_msgs/Octomap`과 같은 ROS 메시지 타입을 정의한다. 이를 통해 OctoMap 데이터를 노드 간에 토픽(topic) 형태로 쉽게 주고받을 수 있다.11
  - `octomap_server`: OctoMap을 ROS에서 사용하는 가장 일반적인 방법으로, 핵심적인 노드를 제공한다. 이 노드는 포인트 클라우드 토픽(`sensor_msgs/PointCloud2`)을 구독하여 점진적으로 3D 맵을 구축하고, 그 결과를 `octomap_msgs` 형태의 토픽으로 발행한다.12
  - `octomap_ros`: OctoMap의 C++ 자료구조와 ROS의 메시지 타입 간의 변환을 돕는 유틸리티 함수들을 제공한다.11

- **SLAM과의 연동:** 중요한 점은 `octomap_server`가 그 자체로 SLAM(Simultaneous Localization and Mapping)을 수행하지 않는다는 것이다. 즉, 로봇의 위치를 스스로 추정하지 않는다. 따라서 `octomap_server`는 `gmapping`, `Cartographer`, `RTAB-Map`과 같은 외부 SLAM 시스템으로부터 로봇의 정확한 위치 및 자세 정보(pose)를 `tf` 변환(transform) 형태로 지속적으로 제공받아야 한다.11

  `octomap_server`는 `cloud_in` 토픽으로 들어온 포인트 클라우드 데이터를 `frame_id` 파라미터에 지정된 전역 좌표계(보통 `/map`)로 변환한 후, 맵에 통합한다.12

이러한 `octomap_server`와 외부 SLAM 시스템 간의 '느슨한 결합(Loose Coupling)'은 높은 유연성을 제공한다. 개발자는 자신이 선호하는 어떤 SLAM 알고리즘과도 `octomap_server`를 조합하여 사용할 수 있다. 하지만 이는 잠재적인 문제의 원인이 되기도 한다. 예를 들어, SLAM 시스템이 루프 클로저(loop closure)를 감지하여 과거의 전체 경로를 대대적으로 수정했을 때, `octomap_server`는 이미 잘못된 위치 정보로 맵에 기록된 '유령(ghost)' 장애물들을 자동으로 수정하지 못한다.20 이는 맵의 일관성을 해치는 주요 원인이 될 수 있다. 반면, 

`RTAB-Map`과 같이 SLAM과 3D 맵핑이 긴밀하게 통합된 시스템은 포즈 그래프 최적화가 발생하면 맵 자체를 수정하여 이러한 문제를 해결한다. 이 차이는 소프트웨어 아키텍처 설계에서 '모듈성'과 '통합성' 사이의 고전적인 트레이드오프를 명확히 보여준다.

아래 표는 `octomap_server`를 ROS 환경에서 사용하고자 하는 개발자에게 필수적인 정보를 요약한 것이다.

#### 2.2.1 표 1: `octomap_server` 주요 토픽, 서비스 및 파라미터

| 구분              | 이름/경로                  | 타입                             | 설명                                                         |
| ----------------- | -------------------------- | -------------------------------- | ------------------------------------------------------------ |
| **구독 토픽**     | `cloud_in`                 | `sensor_msgs/PointCloud2`        | 3D 맵 생성을 위해 입력받는 포인트 클라우드 데이터. 12        |
| **발행 토픽**     | `octomap_binary`           | `octomap_msgs/Octomap`           | 점유/비점유 정보만 포함하는 압축된 이진 맵. 파일 크기가 작다. 12 |
|                   | `octomap_full`             | `octomap_msgs/Octomap`           | 모든 노드의 완전한 확률 정보를 포함하는 맵. 12               |
|                   | `occupied_cells_vis_array` | `visualization_msgs/MarkerArray` | RViz에서 점유된 복셀을 시각화하기 위한 마커 배열. 12         |
|                   | `map`                      | `nav_msgs/OccupancyGrid`         | 3D 맵을 2D 평면에 투영한 2D 점유 격자 지도. 12               |
| **서비스**        | `octomap_binary`           | `octomap_msgs/GetOctomap`        | 현재 맵을 이진 형태로 요청하고 응답받는다. 12                |
|                   | `clear_bbx`                | `octomap_msgs/BoundingBoxQuery`  | 지정된 경계 상자(Bounding Box) 내의 모든 복셀을 '자유' 상태로 초기화한다. 12 |
|                   | `reset`                    | `std_srvs/Empty`                 | 전체 맵을 초기화한다. 12                                     |
| **주요 파라미터** | `resolution`               | `float`                          | 맵의 해상도(최소 복셀 크기, 미터 단위). 12                   |
|                   | `frame_id`                 | `string`                         | 맵이 생성될 기준 좌표계 (예: `/map`). 12                     |
|                   | `sensor_model/max_range`   | `float`                          | 센서 데이터 통합 시 고려할 최대 거리. 이 값을 넘는 측정값은 무시된다. 12 |
|                   | `sensor_model/hit`         | `float`                          | 측정된 지점이 점유되었을 확률. 12                            |
|                   | `sensor_model/miss`        | `float`                          | 광선이 통과한 지점이 비어있을 확률. 12                       |
|                   | `filter_ground`            | `bool`                           | 지면으로 판단되는 평면을 필터링하여 맵에서 제외할지 여부. 12 |

### 2.3 C. 시각화 및 데이터 관리

- **`octovis`:** OctoMap 라이브러리와 함께 제공되는 독립 실행형 3D 뷰어다.5 Qt 라이브러리를 기반으로 제작되었으며, 저장된 맵 파일(`.bt`, `.ot`)을 직접 열어 맵의 구조, 해상도, 점유 상태 등을 상세히 분석하는 데 유용하다.23
- **RViz:** ROS 환경의 표준 3D 시각화 도구로, `octomap_server`와 완벽하게 연동된다. `octomap_server`가 발행하는 `occupied_cells_vis_array` 토픽을 `MarkerArray` 디스플레이 타입으로 추가하면, 실시간으로 생성되는 3D 맵을 로봇의 다른 상태 정보와 함께 확인할 수 있다.11
- **파일 포맷:** OctoMap은 두 가지 주요 파일 형식을 지원한다.
  - `.ot` (OctoMap Tree): 맵의 모든 노드에 대한 완전한 확률 정보를 저장하는 **무손실(lossless)** 포맷이다.17 이 형식으로 저장된 맵은 나중에 다시 불러와서 추가적인 센서 데이터로 업데이트를 계속할 수 있다.
  - `.bt` (Binary Tree): 각 노드의 상태를 점유 확률 임계값(기본값 0.5)을 기준으로 '점유' 또는 '비점유'의 이진 상태로만 저장하는 **손실(lossy)** 압축 포맷이다.11 확률 정보는 사라지지만 파일 크기가 훨씬 작아져 저장 및 네트워크 전송에 유리하다.26

이 두 가지 파일 포맷은 '정확성/확장성'과 '효율성/이식성' 사이의 명확한 트레이드오프를 사용자에게 제공한다. `.ot`는 맵핑 작업을 계속 이어가야 하는 연구 개발 단계에 적합하며, `.bt`는 맵핑이 완료된 후 경로 계획이나 다중 로봇 간 맵 공유와 같이 최종 결과물만 필요한 응용 단계에 적합하다. 라이브러리가 이 두 가지 옵션을 모두 제공한다는 것은 개발자가 응용 프로그램의 특정 요구에 맞게 저장 전략을 선택할 수 있도록 배려한 성숙한 설계임을 보여준다.

## 3.  성능 분석, 한계 및 대안 기술과의 비교

OctoMap은 3D 맵핑 분야에서 큰 성공을 거두었지만, 만능 해결책은 아니다. 그 성능 특성과 내재적 한계를 정확히 이해하고, 최신 대안 기술과 비교하여 장단점을 파악하는 것은 기술을 올바르게 선택하고 적용하기 위해 필수적이다.

### 3.1 A. 성능 특성: 연산 비용 및 메모리 효율성

- **업데이트 및 쿼리 속도:** OctoMap은 실시간 맵 업데이트가 가능하도록 설계되었다. Armin Hornung의 개선 작업을 통해 스캔 데이터 삽입 속도가 두 배 향상되는 등 지속적인 최적화가 이루어졌다.27 하지만 옥트리의 계층적 구조로 인해 특정 노드를 찾기 위한 탐색 시간은 맵에 포함된 노드의 수(n)에 대해 로그 시간 복잡도, 즉 $O(\log n)$을 가진다.28 이는 맵이 매우 커지거나 복잡해질 경우, 특히 빈번한 쿼리가 필요한 응용에서는 성능 저하의 요인이 될 수 있다.
- **메모리 사용량:** 옥트리의 프루닝(pruning) 기법 덕분에, OctoMap은 동일한 공간을 표현할 때 고정 복셀 그리드 방식에 비해 월등한 메모리 효율성을 자랑한다.1 그러나 이 효율성은 맵의 해상도에 매우 민감하다. 맵의 해상도를 두 배 높이면(즉, 최소 복셀 크기를 절반으로 줄이면), 이론적으로 필요한 노드의 수는 최대 8배까지 증가할 수 있어 메모리 요구량이 기하급수적으로 늘어날 수 있다.30
- **하드웨어 가속 연구:** OctoMap의 연산 집약적인 특성은 특히 자원이 제한된 엣지 디바이스(예: 드론)에서 실시간 처리에 부담이 된다. 한 연구에서는 3D 맵 생성이 전체 자율 주행 시간의 70% 이상을 차지할 수 있다고 보고했다.31 이러한 병목 현상을 해결하기 위해, OctoMap 연산을 위한 전용 하드웨어 가속기(OMU: Occupancy Mapping Accelerator)에 대한 연구가 진행되었다. 이 가속기는 Nvidia Jetson TX2 플랫폼의 ARM CPU 대비 최대 62배의 성능 향상과 708배의 에너지 효율 향상을 달성하며, 실시간 3D 맵핑의 높은 연산 요구량과 하드웨어 가의 필요성을 명확히 보여준다.31

### 3.2 B. 내재적 한계와 극복 방안

- **동적 환경:** OctoMap의 기본 모델은 환경이 정적(static)이라는 가정하에 설계되었다. 이로 인해 사람, 차량 등 움직이는 객체가 존재하는 실제 환경에서는 해당 객체들이 지나간 자리에 '유령(ghost)' 흔적(artifact)이 남게 된다.29 이는 로봇의 경로 계획을 방해하는 심각한 문제다. 이 문제를 해결하기 위해, 특정 시간 동안 업데이트되지 않은 복셀의 점유 확률을 점차 낮추어 결국 삭제하는 시간 기반 소멸(time-based decay) 방식 29이나, 현재 센서 측정값과 기존 맵 사이의 기하학적 불일치를 분석하여 동적 객체를 능동적으로 탐지하고 제거하는 연구들이 활발히 진행되고 있다.32
- **센서 모델의 한계:** OctoMap의 기본 확률 센서 모델은 측정 거리에 따른 오차 변화가 적은 레이저 스캐너에 비교적 잘 맞는다. 하지만 저가형 RGB-D 카메라나 스테레오 비전과 같이 거리가 멀어질수록 오차(불확실성)가 크게 증가하는 시각 기반 센서의 복잡한 에러 모델을 정확히 반영하지 못하는 한계가 있다.34
- **SLAM 의존성:** 앞서 언급했듯이, OctoMap으로 생성되는 맵의 품질은 전적으로 외부 SLAM 시스템이 제공하는 위치 추정의 정확성에 의존한다. SLAM 시스템의 작은 오차나 드리프트(drift)는 맵 전체의 왜곡으로 직결되며, 이는 OctoMap 자체적으로는 해결할 수 없는 문제다.20

### 3.3 C. 대안 기술과의 비교: TSDF/ESDF

최근 로보틱스, 특히 고속 자율 비행 및 최적화 기반 경로 계획 분야에서는 OctoMap의 대안으로 Signed Distance Fields (SDF) 계열의 맵 표현 방식이 주목받고 있다.

- **TSDF/ESDF 개념:**
  - **TSDF (Truncated Signed Distance Field):** 공간상의 각 지점에서 가장 가까운 물체 표면까지의 거리를 저장하되, 센서 광선 방향으로의 '투영(projective)' 거리를 사용한다. 또한, 계산을 표면 근처의 좁은 영역(truncation radius)으로 제한하여 효율성을 높인다. 주로 3D 재구성(reconstruction) 및 렌더링에 널리 사용된다.28
  - **ESDF (Euclidean Signed Distance Field):** 공간상의 모든 지점에서 가장 가까운 장애물까지의 실제 '유클리드(Euclidean)' 거리를 저장한다. 이 정보는 로봇이 장애물로부터 얼마나 떨어져 있는지 직접 알려주므로 경로 계획에 매우 유용하다.28
- **심층 분석 및 비교:** OctoMap과 SDF 계열 기술의 근본적인 차이는 '어떤 정보를 제공하는가'에 있다. OctoMap은 "이곳을 지나갈 수 있는가, 없는가?"라는 이산적인 질문에 확률적으로 답한다. 반면, ESDF는 "장애물에서 얼마나 떨어져 있으며, 가장 안전한 방향(경사도)은 어디인가?"라는 연속적인 질문에 답한다.

이 차이는 특히 최신 경로 계획 알고리즘에서 중요해진다. CHOMP나 TrajOpt와 같은 최적화 기반 플래너는 경로의 안전성, 평활도(smoothness) 등을 정량화한 비용 함수(cost function)를 정의하고, 경사도 하강법(gradient descent)과 같은 수치 최적화 기법을 사용하여 비용을 최소화하는 경로를 찾는다. 이 과정에서 비용 함수와 그 경사도(gradient)를 계산하기 위해서는 ESDF가 제공하는 연속적인 거리와 방향 정보가 필수적이다.34 OctoMap은 이러한 정보를 직접 제공하지 못하므로, 이러한 최신 플래너와의 통합에 근본적인 한계가 있다.

기술의 '우수성'은 절대적인 기준이 아니라 시대적 요구에 따라 상대적으로 정의된다. OctoMap은 '확률적 로보틱스' 패러다임이 주류였던 시대의 산물로, 불확실한 환경에서 강건한 지도를 구축하는 데 중점을 두었다. 당시의 A*나 RRT와 같은 경로 계획 알고리즘은 점유/비점유 정보만으로도 충분히 동작했다. 그러나 최근 로보틱스 분야의 패러다임이 '최적화 기반 제어'로 이동하면서, 로봇의 움직임을 더욱 정교하고 동역학적으로 제어하려는 요구가 커졌다. 이 새로운 요구에 TSDF/ESDF가 더 적합한 해답을 제공하게 된 것이다. 즉, OctoMap의 한계는 기술 자체의 결함이라기보다, 로보틱스 패러다임이 진화함에 따라 발생하는 '요구사항과의 불일치'로 해석하는 것이 타당하다.

역설적이게도, OctoMap의 이러한 성능 및 기능적 한계는 오히려 새로운 연구를 촉진하는 계기가 되었다. 실시간 처리의 병목 현상은 OMU와 같은 하드웨어 가속 연구를 촉발했고 31, 정적 환경 가정의 한계는 동적 OctoMap 29, 순수 기하학적 표현의 한계는 시맨틱 OctoMap 36이라는 새로운 연구 분야를 낳았다. 이는 OctoMap이 '완성된 기술'이 아니라, 커뮤니티에 의해 지속적으로 확장되고 개선되는 '살아있는 플랫폼'임을 보여준다. 그 한계점들이 오히려 후속 연구자들에게 명확한 도전 과제와 연구 목표를 제시하는 역할을 한 것이다.

아래 표는 세 가지 3D 맵 표현 방식의 핵심적인 특징을 비교하여 기술 선택에 도움을 주고자 한다.

#### 3.3.1 표 2: 3D 맵 표현 방식 비교 (OctoMap vs. TSDF vs. ESDF)

| 특징              | OctoMap                                                      | TSDF (Truncated Signed Distance Field)                       | ESDF (Euclidean Signed Distance Field)                       |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 데이터**   | 점유 확률 (Log-Odds)                                         | 표면까지의 투영 거리 (Projective Distance)                   | 가장 가까운 장애물까지의 유클리드 거리 (Euclidean Distance)  |
| **공간 표현**     | 점유/비점유/미탐사                                           | 표면 근처의 연속적인 거리 필드                               | 전체 공간의 연속적인 거리 필드                               |
| **메모리 효율성** | 높음 (넓은 빈/미탐사 공간 압축)                              | 중간 (표면 근처만 저장, Voxel Hashing으로 최적화)            | 낮음 (전체 공간에 대한 거리 정보 저장 필요)                  |
| **쿼리 속도**     | $O(\log n)$ (트리 탐색)                                      | $O(1)$ (Voxel Hashing 사용 시)                               | $O(1)$ (그리드 직접 접근)                                    |
| **주요 장점**     | - 불확실성 처리 능력 - 미탐사 영역 명시적 표현 - 성숙한 오픈소스 생태계 | - 빠른 업데이트 속도 - 서브 복셀 정밀도의 표면 재구성 - ESDF 생성의 효율적인 중간 단계 | - 경로 최적화에 필수적인 거리 및 경사도 정보 제공 - 충돌 검사 비용 계산에 용이 |
| **주요 단점**     | - 최적화 기반 플래너와 부적합 - 동적 환경에 취약 - 시각 센서 모델 부정확 | - 전체 공간에 대한 거리 정보 부재 - 경로 계획에 직접 사용하기 어려움 | - 생성 및 업데이트 비용이 높음 - 미탐사 영역을 직접 표현하지 않음 |
| **주요 응용**     | 자율 탐사, 기본 장애물 회피, 조작                            | 고품질 3D 재구성, Visual SLAM                                | 고속 주행, 최적화 기반 경로 계획 (예: CHOMP, TrajOpt)        |

## 4.  응용 분야 및 최신 연구 동향

OctoMap은 그 견고함과 효율성을 바탕으로 다양한 로보틱스 분야에서 성공적으로 활용되어 왔으며, 그 한계를 극복하기 위한 최신 연구들을 통해 끊임없이 진화하고 있다.

### 4.1 A. 주요 응용 분야

- **무인 항공기(UAV) 및 지상 로봇(UGV):** OctoMap의 가장 대표적인 응용 분야는 자율 로봇의 내비게이션이다. 점유, 비점유, 미탐사 공간을 명확히 구분하는 능력은 로봇이 미지의 환경을 안전하게 탐사하고, 장애물을 회피하며, 목표 지점까지 경로를 계획하는 데 필수적인 정보를 제공한다.2
- **로봇 팔 조작(Manipulation):** 산업용 로봇 팔이나 서비스 로봇이 작업을 수행할 때, 작업 공간 내의 장애물을 3차원으로 정확히 인식하고 충돌 없는 움직임을 계획하는 데 OctoMap이 활용된다.1
- **물류 및 재고 관리:** 최근에는 드론을 활용한 자동화된 물류 창고 재고 조사 시스템이 주목받고 있다. 이 시스템에서 OctoMap은 창고 내부의 복잡한 선반 구조와 재고품들을 3D 맵으로 표현하여 드론의 정밀한 위치 추정 및 경로 계획을 지원한다.39
- **수색 및 구조:** 지진이나 건물 붕괴와 같은 재난 현장은 인간이 접근하기 위험하고 구조를 파악하기 어렵다. 이러한 환경에 로봇을 투입하여 3D 맵을 생성하고 생존자를 탐색하는 임무에 OctoMap이 중요한 역할을 한다.3

### 4.2 B. 동적 환경 대응 연구 (Dynamic OctoMap)

실제 세계는 정적이지 않다. 사람, 자동차, 다른 로봇 등 수많은 동적 객체들이 존재하며, 이는 정적 맵핑의 가정을 무너뜨린다.32 이러한 한계를 극복하기 위해 'Dynamic OctoMap' 또는 'Time-aware OctoMap'과 같은 연구들이 활발하게 진행되고 있다.

- **접근법:**
  - **시간 기반 소멸(Time-based Fading):** 가장 간단한 접근법 중 하나로, 특정 시간 동안 센서에 의해 관측되지 않은 복셀의 점유 확률을 점진적으로 감소시켜 결국 '미탐사' 상태로 되돌리거나 삭제하는 방식이다. 이를 통해 오래전에 관측되었지만 현재는 사라졌을 수 있는 동적 객체의 흔적을 자연스럽게 지울 수 있다.29
  - **기하학적 불일치 탐지:** 보다 능동적인 방식으로, 현재 센서 측정값과 기존에 구축된 맵 사이의 불일치를 탐지한다. 예를 들어, 이전에 '자유 공간'으로 알려졌던 곳에서 센서 측정값이 감지된다면, 이는 새로운 동적 객체의 출현으로 간주할 수 있다. 반대로, 이전에 '점유 공간'이었던 곳을 현재 센서 광선이 통과한다면, 이는 기존에 있던 객체가 사라졌음을 의미한다. 이러한 기하학적 불일치를 분석하여 동적 객체에 해당하는 포인트들을 식별하고 맵에서 제거한다.32
  - **데이터 프레임 기반 분석:** 특히 모바일 레이저 스캐닝(MLS)과 같이 연속적인 데이터를 얻는 경우, 전체 데이터를 시간 순서에 따라 여러 데이터 프레임으로 분할한다. 그리고 각 프레임과 그 주변 프레임들을 비교하여 일관되지 않은 측정값을 동적 객체로 판단하고 제거하는 연구가 제안되었다. 이 방법은 기존 OctoMap 기반 방식보다 평균 18.27% 더 효율적인 성능을 보였다고 보고되었다.33

### 4.3 C. 시맨틱 맵핑으로의 확장 (Semantic OctoMap)

로봇 지능의 다음 단계는 단순히 공간의 기하학적 구조를 이해하는 것을 넘어, 그 공간을 구성하는 객체들의 '의미(semantics)'를 이해하는 것이다. "이곳에 장애물이 있다"에서 한 걸음 더 나아가, "이것은 '의자'이고, 저것은 '테이블'이며, 저 사람은 '동료 작업자'이다"와 같이 맵에 의미론적 정보를 부여하려는 연구가 바로 시맨틱 맵핑이다. 이는 로봇이 환경을 더 깊이 이해하고, 인간과 자연스럽게 상호작용하며, 고차원적인 임무를 수행하는 데 필수적이다.37

- **방법론:** 시맨틱 맵핑은 주로 딥러닝 기반의 컴퓨터 비전 기술과 SLAM을 결합하여 구현된다. 로봇의 카메라로부터 얻은 RGB 이미지에 객체 탐지(Object Detection) 또는 시맨틱 분할(Semantic Segmentation) 알고리즘을 적용하여 이미지 내의 각 픽셀 또는 객체에 '사람', '자동차', '문' 등과 같은 시맨틱 레이블을 할당한다. 그 후, 깊이 정보를 이용하여 이 레이블을 3D 공간상의 해당 포인트들에 투영하고, 이를 OctoMap의 각 복셀에 점유 확률과 함께 저장한다.36
- **구현:** Semantic OctoMap은 기존 OctoMap의 노드 구조를 확장하여, 각 복셀이 점유 확률뿐만 아니라 시맨틱 레이블에 대한 확률 분포를 함께 저장하도록 한다. 여러 번의 관측을 통해 동일한 복셀에 대해 다른 레이블이 할당될 경우, 베이즈 퓨전(Bayesian fusion)이나 최대값 퓨전(max fusion)과 같은 방식을 사용하여 가장 가능성 있는 레이블로 통합한다.37
- **효과:** 시맨틱 맵은 로봇의 능력을 획기적으로 향상시킨다. 단순한 장애물 회피를 넘어 "거실로 가서 소파 옆에 있는 컵을 가져와"와 같은 인간의 언어로 표현된 고차원적인 임무 수행을 가능하게 한다. 또한, '사람'이나 '자동차'와 같이 동적일 가능성이 높은 객체와 '벽'이나 '건물'과 같이 정적일 가능성이 높은 객체를 시맨틱 레이블을 통해 구분할 수 있다. 이는 동적 환경에서의 맵핑 강건성을 높이는 데 크게 기여할 수 있다.36

OctoMap의 연구 동향은 로봇 지능의 발달 과정을 명확하게 보여준다. 초기 OctoMap은 공간의 **기하학(Geometry)**을 표현하는 데 집중했다. Dynamic OctoMap 연구는 여기에 **시간(Time)** 차원을 더하여 맵이 과거에 얽매이지 않고 현재를 정확히 반영하도록 만들었다. 그리고 Semantic OctoMap 연구는 **의미(Semantics)** 차원을 추가하여 맵을 단순한 공간 모델에서 환경에 대한 '지식 베이스(knowledge base)'로 격상시켰다.

더 나아가, 시맨틱 맵핑은 SLAM의 정확도를 역으로 향상시키는 선순환 구조를 만들 잠재력을 가진다. 한 연구에서는 시맨틱 맵을 기반으로 위치 추정 정확도를 향상시켰다고 보고했다.36 이는 매우 중요한 시사점을 가진다. 기존에는 정확한 SLAM이 좋은 맵핑의 전제 조건이었지만, 시맨틱 맵핑은 이 관계를 역전시킬 수 있다. 예를 들어, 로봇이 이전에 봤던 '특정한 포스터가 붙은 벽'이나 '독특한 모양의 소파'를 다시 인식한다면, 이는 수백 개의 평범한 기하학적 특징점보다 훨씬 더 강력하고 신뢰성 있는 루프 클로저의 단서가 될 수 있다. 즉, '의미'가 '위치'를 보정하는 것이다. 이는 SLAM과 고수준 장면 이해(scene understanding) 기술이 깊이 융합되는 미래 로보틱스 연구의 방향을 제시한다.

## 5.  결론 및 제언

OctoMap은 지난 10여 년간 로보틱스 3D 맵핑 분야에서 표준적인 도구로 자리매김하며 수많은 연구와 응용에 기여해왔다. 본 보고서는 OctoMap의 이론적 기반부터 실제 활용, 한계, 그리고 최신 연구 동향까지 다각적으로 심층 분석하였다. 이를 바탕으로 종합적인 평가와 함께, 실제 응용을 위한 기술 선택 가이드라인을 제언하고자 한다.

### 5.1 A. OctoMap 종합 평가

- **핵심 강점:** OctoMap의 가장 큰 강점은 **탁월한 메모리 효율성**이다. 옥트리 자료구조를 통해 넓은 공간을 매우 적은 메모리로 표현할 수 있는 능력은 자원이 제한된 로봇 시스템에서 특히 빛을 발한다.1 또한, 

  **확률 모델을 통한 불확실성 처리 능력**은 실제 환경의 노이즈와 센서 오차에 강건한 맵을 구축하게 해주며, **ROS를 중심으로 한 강력하고 성숙한 오픈소스 생태계**는 개발자들이 쉽게 기술을 도입하고 활용할 수 있는 기반을 제공한다.4

- **명확한 약점:** 반면, OctoMap은 몇 가지 명확한 약점을 가지고 있다. **정적 환경 가정**으로 인해 동적 객체가 많은 실제 환경에서는 맵의 신뢰성이 저하될 수 있다.29 또한, 점유 여부 정보만 제공하므로 경로의 안전성이나 평활도를 연속적으로 최적화하는 **최신 최적화 기반 플래너와의 부조화** 문제가 있다.34 마지막으로, 맵의 품질이 전적으로 **외부 SLAM 시스템의 정확성에 의존**한다는 점은 구조적인 한계로 작용한다.20

### 5.2 B. 기술 선택을 위한 전문가 제언

어떤 기술이든 모든 상황에 완벽한 해결책은 될 수 없다. 따라서 응용 프로그램의 구체적인 요구사항에 따라 적절한 3D 맵핑 기술을 선택하는 것이 중요하다.

- **OctoMap이 적합한 경우:**
  - **탐사 및 커버리지(Exploration & Coverage):** 로봇이 미지의 환경을 탐사하며 '미탐사 영역'을 식별하고, 점유/비점유 영역을 구분하여 맵을 완성해나가는 임무에 매우 효과적이다.
  - **자원 제약적 환경:** 드론이나 소형 모바일 로봇과 같이 메모리와 연산 능력이 제한된 임베디드 시스템에서 3D 맵을 운용해야 할 때 가장 현실적인 선택지이다.
  - **전통적인 경로 계획:** A*나 RRT/RRT*와 같이 점유 격자 지도를 기반으로 동작하는 경로 계획 알고리즘을 사용하는 경우, OctoMap은 필요한 정보를 충실히 제공한다.
- **TSDF/ESDF 기반 솔루션(예: Voxblox)을 고려해야 할 경우:**
  - **고속 동적 환경:** 경주용 드론과 같이 빠른 속도로 움직이는 로봇이 실시간으로 장애물을 회피해야 할 때, ESDF가 제공하는 즉각적인 거리 정보는 필수적이다.
  - **최적화 기반 경로 계획:** 경로의 안전성, 평활도, 에너지 효율 등을 종합적으로 고려하여 최적의 궤적을 생성하는 고급 플래너(CHOMP, TrajOpt 등)를 사용하고자 할 때, ESDF는 필수적인 입력 데이터를 제공한다.
  - **고품질 3D 재구성:** 환경의 정밀한 표면 모델이나 사실적인 메시(mesh)를 생성하여 시각화하거나 분석해야 하는 응용에서는 TSDF 기반의 재구성 파이프라인이 더 적합하다.

### 5.3 C. 향후 전망

OctoMap은 앞으로도 중요한 역할을 계속 수행하겠지만, 그 역할은 점차 변화할 것이다. 독립적인 맵핑 솔루션으로서의 역할보다는, 더 크고 복잡한 시스템의 **'기반 레이어(foundational layer)'**로서의 역할이 더욱 강화될 것으로 전망된다.

미래의 지능형 로봇은 단순히 현재 환경을 기하학적으로 표현하는 것을 넘어, 보이지 않는 부분을 추론하여 완성하고(3D scene completion), 동적 객체의 미래 움직임을 예측하며(prediction), 객체와 환경의 의미를 이해하는(semantic understanding) 능력을 갖추어야 한다. OctoMap의 유연하고 확장 가능한 노드 구조는 이러한 풍부한 정보들을 저장하기 위한 훌륭한 백본(backbone)을 제공할 수 있다. 점유 확률뿐만 아니라, 시맨틱 레이블, 객체 ID, 예측 궤적, 통행 가능성 비용 등 다양한 데이터를 복셀에 통합하는 '월드 모델(world model)'로 진화하는 데 있어 OctoMap의 기본 철학과 구조는 여전히 유효할 것이다.2 결국 OctoMap은 그 자체의 한계를 극복하는 연구들과 함께, 차세대 로봇 지능을 뒷받침하는 핵심 데이터 구조로서 그 생명력을 이어갈 것이다.

#### **참고 자료**

1. OctoMap: An Efficient Probabilistic 3D Mapping Framework Based on Octrees - Washington, accessed July 31, 2025, https://courses.cs.washington.edu/courses/cse571/16au/slides/hornung13auro.pdf
2. OctoMap: An efficient probabilistic 3D mapping framework based on octrees - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/257523133_OctoMap_An_efficient_probabilistic_3D_mapping_framework_based_on_octrees
3. Autonomous Multi-Robot Exploration Strategies for 3D Environments with Fire Detection Capabilities - arXiv, accessed July 31, 2025, https://arxiv.org/html/2411.15953v1
4. OctoMap - 3D occupancy mapping, accessed July 31, 2025, https://octomap.github.io/
5. Introduction - OctoMap, accessed July 31, 2025, https://octomap.github.io/octomap/doc/
6. OctoMap: A Probabilistic, Flexible, and Compact 3D Map Representation for Robotic Systems - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/235008236_OctoMap_A_Probabilistic_Flexible_and_Compact_3D_Map_Representation_for_Robotic_Systems
7. Benchmarking Occupancy Mapping libraries - Nicolò Valigi, accessed July 31, 2025, https://nicolovaligi.com/articles/occupancy-mapping-benchmarks/
8. octomap Documentation - the official ROS docs, accessed July 31, 2025, https://docs.ros.org/diamondback/api/octomap/html/index.html
9. octomap - ROS Wiki, accessed July 31, 2025, http://wiki.ros.org/octomap
10. 3DVFH+: Real-Time Three-Dimensional Obstacle Avoidance Using an Octomap - CEUR-WS.org, accessed July 31, 2025, https://ceur-ws.org/Vol-1319/morse14_paper_08.pdf
11. tejalbarnwal/octomap_tutorial: Practicing using octomap - GitHub, accessed July 31, 2025, https://github.com/tejalbarnwal/octomap_tutorial
12. 9. ORB_SLAM2_Octomap - Yahboom, accessed July 31, 2025, [http://www.yahboom.net/public/upload/upload-html/1743490362/9.%20ORB_SLAM2_Octomap.html](http://www.yahboom.net/public/upload/upload-html/1743490362/9. ORB_SLAM2_Octomap.html)
13. octomap/octomap/README.md at devel / OctoMap/octomap / GitHub, accessed July 31, 2025, https://github.com/OctoMap/octomap/blob/devel/octomap/README.md
14. ROS Package: octomap, accessed July 31, 2025, https://index.ros.org/p/octomap/
15. octomap/octovis/README.md at devel - GitHub, accessed July 31, 2025, https://github.com/OctoMap/octomap/blob/devel/octovis/README.md
16. Octomap - A probabilistic, flexible, and compact 3D mapping library for robotic systems, accessed July 31, 2025, https://octomap.github.io/octomap/doc/md_README.html
17. octomap_msgs/Octomap Message - ROS documentation, accessed July 31, 2025, http://docs.ros.org/en/noetic/api/octomap_msgs/html/msg/Octomap.html
18. ROS stack for mapping with OctoMap, contains octomap_server package - GitHub, accessed July 31, 2025, https://github.com/OctoMap/octomap_mapping
19. octomap_server - ROS Wiki, accessed July 31, 2025, http://wiki.ros.org/octomap_server
20. octomap_ros on ros 2 humble - Robotics Stack Exchange, accessed July 31, 2025, https://robotics.stackexchange.com/questions/103089/octomap-ros-on-ros-2-humble
21. 3D Mapping with OctoMap - ArminHornung.de, accessed July 31, 2025, https://www.arminhornung.de/Research/pub/hornung13roscon.pdf
22. how to improve maps from octomap? - ros - Robotics Stack Exchange, accessed July 31, 2025, https://robotics.stackexchange.com/questions/65945/how-to-improve-maps-from-octomap
23. octovis - ROS Wiki, accessed July 31, 2025, http://wiki.ros.org/octovis
24. Robots/TIAGo/Tutorials/MoveIt/Planning_Octomap - ROS Wiki, accessed July 31, 2025, http://wiki.ros.org/Robots/TIAGo/Tutorials/MoveIt/Planning_Octomap
25. octomap/octomap/src/graph2tree.cpp at devel - GitHub, accessed July 31, 2025, https://github.com/OctoMap/octomap/blob/devel/octomap/src/graph2tree.cpp
26. Accessing 3D map data saved in bt file/formats to save octomap - Robotics Stack Exchange, accessed July 31, 2025, https://robotics.stackexchange.com/questions/41712/accessing-3d-map-data-saved-in-bt-file-formats-to-save-octomap
27. Improvements To 3D Navigation Using Octomap And ROS | RoboticsTomorrow, accessed July 31, 2025, https://www.roboticstomorrow.com/story/2011/09/improvements-to-3d-navigation-using-octomap-and-ros/309/
28. Voxblox: Incremental 3D Euclidean Signed ... - Helen Oleynikova, accessed July 31, 2025, https://helenol.github.io/publications/iros_2017_voxblox.pdf
29. Timebased local OctoMaps - TU Chemnitz, accessed July 31, 2025, https://www.tu-chemnitz.de/etit/proaut/en/research/octomap.html
30. Comparison of the Octomap representation for two different voxel sizes... - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/figure/Comparison-of-the-Octomap-representation-for-two-different-voxel-sizes-05-m-and-02-m_fig3_330426624
31. OMU: A Probabilistic 3D Occupancy Mapping Accelerator for Real-time OctoMap at the Edge - arXiv, accessed July 31, 2025, https://arxiv.org/pdf/2205.03325
32. ERASOR2: Instance-Aware Robust 3D Mapping of the Static World in Dynamic Scenes - Robotics, accessed July 31, 2025, https://www.roboticsproceedings.org/rss19/p067.pdf
33. (PDF) Data frame aware optimized Octomap-based dynamic object detection and removal in Mobile Laser Scanning data - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/371978146_Data_frame_aware_optimized_Octomap-based_dynamic_object_detection_and_removal_in_Mobile_Laser_Scanning_data
34. Signed Distance Field - Dibyendu Biswas - Medium, accessed July 31, 2025, https://dibyendu-biswas.medium.com/signed-distance-field-14d829d8103e
35. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Research Collection, accessed July 31, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/128029/eth-50477-01.pdf
36. Semantic SLAM based on Object Detection and Improved Octomap - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/328114559_Semantic_SLAM_based_on_Object_Detection_and_Improved_Octomap
37. Continuous Online Semantic Implicit Representation for ... - MDPI, accessed July 31, 2025, https://www.mdpi.com/2218-6581/13/7/108
38. Example of a UAV updating its own OctoMap when entering communication... | Download Scientific Diagram - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/figure/Example-of-a-UAV-updating-its-own-OctoMap-when-entering-communication-range-with-other_fig3_385484744
39. Autonomous Full 3D Coverage Using an Aerial Vehicle, Performing ..., accessed July 31, 2025, https://www.mdpi.com/2218-6581/13/6/83
40. A Comparison of Segmentation Methods for Semantic OctoMap Generation - MDPI, accessed July 31, 2025, https://www.mdpi.com/2076-3417/15/13/7285
41. 3D Perception for Mobile Manipulation with OctoMap - MoveIt, accessed July 31, 2025, https://moveit.ai/assets/pdfs/2013/icra2013tutorial/octomap_planning.pdf


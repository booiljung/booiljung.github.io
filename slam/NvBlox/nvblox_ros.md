---
layout: page
title: nvblox_ros에 대한 고찰
permalink: /slam/NvBlox/nvblox_ros
---



현대 로보틱스 기술의 발전은 자율 이동 로봇(Autonomous Mobile Robots, AMR), 협동 로봇 팔(Collaborative Robot Arms) 등 다양한 형태의 로봇 시스템이 산업 현장과 일상 공간에 통합되는 것을 가속화하고 있다. 이러한 로봇들이 주어진 임무를 안전하고 효율적으로 수행하기 위한 가장 근본적인 전제 조건은 주변 환경에 대한 정확하고 실시간적인 3차원(3D) 인식 능력이다.1 로봇은 3D 맵을 통해 자신의 위치를 파악하고, 장애물을 인지하여 충돌을 회피하며, 최적의 경로를 계획하고, 나아가 객체를 조작하는 등 복잡한 작업을 수행할 수 있다.3

그러나 로봇이 실시간으로 고품질의 3D 환경 모델을 구축하는 것은 상당한 기술적 난제를 동반한다. 기존의 대표적인 3D 매핑 시스템, 예를 들어 `Voxblox`나 `OctoMap`과 같은 프레임워크들은 주로 중앙 처리 장치(CPU)의 연산 능력에 의존해왔다.5 CPU 기반의 접근 방식은 알고리즘적으로는 견고하지만, 3D 공간을 복셀(Voxel) 단위로 나누고 각 복셀의 상태를 지속적으로 업데이트하는 과정에서 발생하는 막대한 계산 부하로 인해 몇 가지 본질적인 한계에 직면했다. 특히, 로봇에 탑재되는 임베디드 시스템과 같이 계산 자원이 제한된 환경에서는 실시간성을 보장하기 위해 맵의 해상도를 낮추거나 처리 가능한 영역을 축소해야만 했다.3 이는 결국 맵의 정밀도를 저하시켜 로봇이 정교한 경로 계획이나 섬세한 조작 작업을 수행하는 데 제약을 가하는 요인으로 작용했다.


이러한 기술적 한계를 극복하기 위해 NVIDIA는 자사의 핵심 역량인 GPU(Graphics Processing Unit) 병렬 컴퓨팅 기술과 CUDA(Compute Unified Device Architecture) 아키텍처를 로보틱스 매핑 분야에 접목한 `nvblox` 라이브러리를 선보였다.8

`nvblox`는 기존의 CPU 중심적 패러다임에서 벗어나, 3D 재구성 과정의 거의 모든 단계를 GPU에서 병렬로 처리하도록 처음부터 새롭게 설계된 고성능 라이브러리이다.

`nvblox`의 핵심 목표는 명확하다. 바로 계산 자원이 제한적인 임베디드 GPU 플랫폼, 특히 NVIDIA Jetson 시리즈에서도 실시간으로 고해상도의 3D 환경 모델을 구축하고 유지하는 것이다.1 이를 위해 `nvblox`는 두 가지 핵심적인 결과물을 동시에 생성한다. 첫째는 환경의 표면을 정밀하게 표현하는 TSDF(Truncated Signed Distance Function) 맵이고, 둘째는 로봇의 경로 계획과 충돌 회피에 결정적인 정보를 제공하는 ESDF(Euclidean Signed Distance Field) 맵이다.1 이 두 가지를 GPU 가속을 통해 동시에, 그리고 매우 빠르게 생성함으로써 로봇의 '인식'과 '행동' 사이의 지연을 최소화하는 것을 목표로 한다.

`nvblox_ros`는 이러한 강력한 C++/CUDA 기반의 `nvblox` 코어 라이브러리를 로봇 운영체제인 ROS 2(Robot Operating System 2) 생태계에 완벽하게 통합하는 래퍼(wrapper) 패키지이다.10

`nvblox_ros`는 ROS 2의 표준 메시지 타입과 통신 방식을 준수하여 `isaac_ros_visual_slam`(위치 추정), `isaac_ros_unet`(의미론적 분할) 등 다른 Isaac ROS 패키지들과 유기적으로 연동된다. 이를 통해 개발자는 복잡한 GPU 프로그래밍 없이도 고성능 3D 인식 파이프라인을 손쉽게 구축하고, 기존의 ROS 2 기반 로봇 시스템에 통합하여 시너지를 창출할 수 있다.12

이러한 `nvblox`의 등장은 단순히 기존 기술의 성능을 개선하는 것을 넘어, '실시간 고품질 3D 인식의 대중화'라는 패러다임의 전환을 이끌고 있다. 과거 고성능 데스크톱이나 연구실 환경에서만 가능했던 정밀한 3D 재구성을 저전력 임베디드 보드에서 실시간으로 구현할 수 있게 되면서, AMR, 드론, 로봇 팔 등 더 광범위한 로봇 애플리케이션에서 한 차원 높은 수준의 자율성을 달성할 수 있는 기술적 토대가 마련된 것이다.6 이는 곧 기술 접근성의 민주화를 의미하며, 더 많은 개발자와 연구자들이 정교한 3D 기반 로봇 애플리케이션을 개발하고 배포할 수 있는 길을 열어주고 있다.


본 보고서는 `nvblox_ros`가 제시하는 새로운 패러다임을 심도 있게 이해하고, 그 기술적 잠재력과 한계를 명확히 파악하는 것을 목표로 한다. 이를 위해 단순한 기능 소개를 넘어, `nvblox_ros`의 근간을 이루는 핵심 알고리즘의 원리부터 시스템 아키텍처, 데이터 흐름, 성능 특성, 그리고 실제 응용 사례에 이르기까지 다각적이고 심층적인 분석을 제공하고자 한다. 또한, `Voxblox`, `OctoMap` 등 기존의 주요 매핑 프레임워크 및 NeRF와 같은 최신 기술과의 비교 고찰을 통해 `nvblox_ros`의 기술적 위상을 객관적으로 조명할 것이다.

본 보고서는 다음과 같은 순서로 구성된다.

1. **서론**: `nvblox_ros`의 등장 배경과 기술적 목표를 제시한다.
2. **핵심 기술**: TSDF와 ESDF 알고리즘의 원리를 수학적 모델과 함께 심층적으로 분석한다.
3. **아키텍처 및 데이터 흐름**: `nvblox_ros`의 내부 구조와 Isaac ROS 생태계 내에서의 데이터 처리 파이프라인을 해부한다.
4. **동적 환경 대응 전략**: 의미론적 분할과 감쇠 모델을 통해 동적 객체를 처리하는 방식을 탐구한다.
5. **성능 벤치마킹 및 분석**: 다양한 하드웨어 플랫폼과 센서 입력에 따른 성능을 정량적으로 분석하고, 주요 파라미터 튜닝 가이드를 제공한다.
6. **타 매핑 프레임워크와의 비교 고찰**: `Voxblox`, `OctoMap`, `VDBFusion`, NeRF 등과의 비교를 통해 `nvblox`의 장단점을 명확히 한다.
7. **응용 및 확장성 탐구**: 로봇 팔 조작, 장기 매핑, 의미론적 맵으로의 확장 가능성을 논한다.
8. **결론 및 제언**: `nvblox_ros`의 핵심 가치를 요약하고, 실무자를 위한 제언과 미래 연구 방향을 제시한다.


`nvblox`의 성능과 기능은 두 가지 핵심적인 3D 공간 표현 방식인 TSDF(Truncated Signed Distance Function)와 ESDF(Euclidean Signed Distance Field)에 기반한다. 이 두 기술은 각각 환경의 '표면 재구성'과 '자유 공간 탐색'이라는 로봇 공학의 근본적인 요구사항을 해결하기 위해 설계되었다. `nvblox`는 이 두 표현을 GPU 상에서 매우 효율적으로 계산하고 유지하는 알고리즘을 구현함으로써 그 가치를 증명한다.


TSDF는 3D 환경을 복셀 그리드(voxel grid)로 표현하는 기법 중 하나로, 특히 표면을 매끄럽고 연속적으로 재구성하는 데 탁월한 성능을 보인다. 전통적인 점유 격자(occupancy grid)가 각 복셀을 '점유됨', '비어있음', '알 수 없음'의 이산적인 상태로 표현하는 것과 달리, TSDF는 보다 풍부한 정보를 담고 있다.


TSDF의 핵심 아이디어는 복셀 그리드 내의 각 복셀이 가장 가까운 물리적 표면(surface)까지의 '부호화된 거리(signed distance)'를 저장하는 것이다.1 여기서 '부호'는 복셀이 표면을 기준으로 안쪽에 있는지 바깥쪽에 있는지를 나타낸다. 일반적으로 표면 바깥(센서와 표면 사이의 자유 공간)은 양수 값을, 표면 안쪽(표면 뒤의 관측되지 않은 공간)은 음수 값을 가진다. 따라서, 거리 값이 정확히 0이 되는 지점들의 집합, 즉 영점 집합(zero-level set)이 바로 환경의 실제 표면을 나타내게 된다.14 이 연속적인 거리 값 표현 덕분에 TSDF는 센서 노이즈에 강인하며, 점유 격자 방식에 비해 훨씬 더 고품질의 매끄러운 표면 모델을 생성할 수 있다.


`nvblox`는 센서로부터 수신된 새로운 깊이 정보를 기존의 TSDF 맵에 통합(integration)하는 과정을 통해 맵을 점진적으로 구축한다. 이 과정은 주로 '투영적(projective)' 방식을 따르는데, 이는 각 센서 측정값(예: 깊이 카메라의 픽셀)을 3D 공간으로 투영하여 관련 복셀들을 업데이트하는 것을 의미한다.16

구체적인 과정은 다음과 같다. 깊이 카메라의 특정 픽셀 u에서 측정된 깊이 값이 $d(u)$라고 하자. 이 픽셀에 해당하는 3D 공간상의 광선(ray) 위에 놓인 임의의 복셀 중심점 p를 고려한다. 이 복셀 p의 현재 TSDF 값을 업데이트하기 위해, 먼저 p를 카메라 좌표계로 변환하여 깊이 값 z를 계산한다. 그 다음, 측정된 깊이 $d(u)$와 복셀의 깊이 z의 차이를 계산하여 부호화된 거리 함수(Signed Distance Function, SDF) 값을 구한다.
$$
SDF(p) = d(u) - z
$$
이 SDF 값은 표면으로부터의 거리를 나타내며, 양수이면 복셀이 표면보다 멀리(자유 공간), 음수이면 표면보다 가까이(표면 내부) 있음을 의미한다.

'Truncated(절단된)'라는 이름에서 알 수 있듯이, TSDF는 이 SDF 값을 특정 거리, 즉 절단 거리(truncation distance) \tau 내에서만 유효한 값으로 간주한다. \tau보다 멀리 떨어진 복셀의 SDF 값은 각각 최대값(+1) 또는 최소값(−1)으로 고정(clamp)된다. 이는 계산 효율성을 높이고, 센서 측정의 불확실성이 큰 먼 거리의 정보가 맵에 과도한 영향을 미치는 것을 방지하기 위함이다. 최종적으로 복셀에 저장될 TSDF 값 $F(p)$는 다음과 같이 정규화된다.
$$
F(p) = \max(-1, \min(1, \frac{SDF(p)}{\tau}))
$$


실제 센서 데이터에는 상당한 노이즈가 포함되어 있다. 단일 측정값만으로 맵을 구성하면 노이즈가 그대로 맵에 반영되어 표면이 울퉁불퉁해지거나 부정확해질 수 있다. `nvblox`는 이러한 문제를 해결하기 위해 여러 프레임에 걸쳐 동일한 복셀에 대한 측정값을 통계적으로 융합하는 방식을 사용한다. 각 TSDF 복셀은 거리 값 $F(p)$와 함께 가중치(weight) $W(p)$를 저장한다.

새로운 측정값 $F_{update}(p)$와 그에 해당하는 가중치 $w_{update}$가 주어지면, 복셀의 기존 값 $F_{old}(p)$와 가중치 $W_{old}(p)$는 다음과 같은 가중 평균(weighted average) 공식을 통해 갱신된다.16
$$
W_{new}(p) = \min(W_{max}, W_{old}(p) + w_{update})
$$

$$
F_{new}(p) = \frac{W_{old}(p)F_{old}(p) + w_{update}F_{update}(p)}{W_{old}(p) + w_{update}}
$$

여기서 $W_{max}$는 가중치의 최대값으로, 맵이 너무 경직되어 새로운 변화에 둔감해지는 것을 방지하는 역할을 한다. 이 과정을 통해 일시적인 센서 노이즈는 상쇄되고, 여러 번 일관되게 관측된 표면 정보만이 맵에 안정적으로 누적된다.


앞서 설명한 TSDF 통합 과정, 즉 각 복셀에 대한 SDF 계산, 절단, 가중 평균 업데이트는 본질적으로 다른 복셀의 계산 과정과 독립적이다. 수백만 개의 복셀에 대한 업데이트 계산을 동시에 수행할 수 있다는 의미이며, 이는 GPU의 대규모 병렬 처리 아키텍처에 완벽하게 부합한다. `nvblox`는 CUDA 커널을 통해 이 모든 계산을 GPU 상에서 병렬로 처리함으로써, CPU 기반 시스템으로는 상상하기 어려운 속도로 TSDF 맵을 실시간으로 통합하고 업데이트할 수 있다.2


TSDF가 환경의 표면을 정밀하게 재구성하는 데 중점을 둔다면, ESDF는 로봇의 자율 주행, 즉 경로 계획(path planning)에 직접적으로 활용되는 정보를 제공하는 데 특화되어 있다.


ESDF는 각 복셀에 가장 가까운 '점유된(occupied)' 복셀까지의 실제 유클리드 거리(Euclidean distance)를 저장하는 필드이다.1 TSDF가 센서 광선을 따라 측정된 '투영적' 거리를 사용하는 것과 달리, ESDF는 방향에 무관한 최단 거리를 제공한다.

이러한 특성 때문에 ESDF는 경로 계획 알고리즘에 매우 유용하다. 예를 들어, 로봇이 특정 지점 $(x, y, z)$로 이동하는 것을 고려할 때, 해당 위치의 ESDF 값만 조회하면 가장 가까운 장애물까지의 거리를 즉시 알 수 있다. 이 값이 로봇의 반경보다 크면 안전한 위치이고, 작으면 충돌 위험이 있는 위치이다.23 또한, ESDF의 그래디언트(gradient)는 장애물로부터 가장 빠르게 멀어지는 방향을 가리키므로, 최적화 기반의 경로 계획 알고리즘(예: CHOMP)이 충돌을 회피하는 경로를 생성하는 데 결정적인 정보를 제공한다.24


ESDF를 생성하는 전통적인 CPU 기반 방식 중 하나는 `Voxblox`에서 사용된 웨이브 전파(wavefront propagation) 알고리즘이다.23 이 방식은 TSDF 맵에서 새롭게 점유 상태로 변경된 복셀(sites)들을 시작점으로 하여, 마치 물결이 퍼져나가듯 주변의 비어있는 복셀들로 거리 값을 전파시킨다. 너비 우선 탐색(Breadth-First Search, BFS)과 유사한 방식으로 동작하며, 큐(queue)를 사용하여 업데이트할 복셀들을 관리한다. 이 방법은 증분적 업데이트(incremental update)에 효율적이지만, 큐를 사용하고 한 복셀의 값이 결정되어야 그 이웃을 처리할 수 있다는 점에서 본질적으로 순차적인(sequential) 특성을 가지며, GPU의 대규모 병렬 처리에 적합하지 않다.5


`nvblox`는 ESDF 생성을 GPU에서 효율적으로 수행하기 위해 기존의 순차적 알고리즘을 근본적으로 재설계했다. 그 결과물은 `Voxblox`의 증분적 접근법과 PBA(Parallel Banding Algorithm)의 아이디어를 결합한 새로운 하이브리드 병렬 알고리즘이다.5 이 알고리즘은 ESDF 계산 과정을 여러 단계로 나누어 각 단계를 GPU에서 대규모로 병렬 처리함으로써 CPU 방식 대비 수십 배에 달하는 극적인 성능 향상을 달성했다.2

`nvblox`의 ESDF 생성 파이프라인은 다음과 같이 구성된다.

1. **Allocate & Mark Sites**: TSDF 맵이 업데이트되면, 변경된 TSDF 복셀 블록에 해당하는 ESDF 복셀 블록을 메모리에 할당한다. 그 후, TSDF 값의 부호가 바뀌는 지점, 즉 표면 경계에 해당하는 복셀들을 'site'로 표시한다. 이 site들이 거리 계산의 기준점이 된다.2
2. **Clear Invalid & Lower ESDF**: 이전에 site였던 복셀이 더 이상 site가 아니게 되면, 그 site를 가장 가까운 장애물로 참조하고 있던 다른 복셀들의 거리 값은 무효(invalid)가 된다. 'Clear Invalid' 단계에서는 이러한 무효화된 복셀들을 찾아내어 거리 값을 최대치로 초기화한다. 그 후 'Lower ESDF' 단계에서 본격적인 거리 계산이 이루어진다. 이 단계에서는 각 복셀 블록 내에서 6개의 축 방향(+X,−X,+Y,−Y,+Z,−Z)으로 스위핑(sweeping)을 수행한다. 각 복셀은 자신의 이웃 복셀이 참조하는 site를 통해 자신에게 도달하는 거리가 현재 자신의 거리 값보다 짧은 경우, 그 값으로 자신의 거리 값을 갱신한다. 이 스위핑 과정은 복셀 블록 단위로 GPU에서 완전히 병렬 처리된다. 이 전체 과정을 여러 번 반복하면, 마치 파도가 퍼지듯 거리 정보가 블록의 경계를 넘어 전체 맵으로 전파된다.2

이러한 `nvblox`의 ESDF 생성 알고리즘은 단순한 코드의 GPU 포팅을 넘어선, '증분적 병렬화'라는 알고리즘적 혁신이라 할 수 있다. 이는 문제의 공간적 지역성(spatial locality)을 활용하여 대부분의 계산을 독립적인 블록 단위로 병렬 처리하고, 블록 간의 정보 전파만 증분적으로 수행하는 영리한 접근법이다. `Voxblox`의 증분적 업데이트라는 장점과 GPU의 병렬 처리 능력을 모두 취한 이 설계는 `nvblox`의 압도적인 ESDF 계산 성능의 비결이다.


고해상도 3D 맵을 구축할 때 발생하는 가장 큰 문제 중 하나는 메모리 사용량이다. 거대한 3D 배열을 미리 할당하는 방식은 로봇이 탐사하는 공간이 넓어질수록 비현실적이 된다. `nvblox`는 이 문제를 해결하기 위해 '복셀 해싱(voxel hashing)'이라는 기법을 사용한다.2

이 기법은 전체 3D 공간을 미리 할당하는 대신, 로봇이 실제로 관측한 영역에 대해서만 고정된 크기의 복셀 덩어리, 즉 '복셀 블록(VoxelBlock)'을 동적으로 메모리에 할당한다.6 그리고 3D 공간 좌표를 해시 함수(hash function)를 통해 고유한 해시 키(hash key)로 변환한 뒤, 이 키를 사용하여 해시 테이블에서 해당 복셀 블록이 저장된 메모리 주소를 찾는다. 이 방식은 해시 테이블의 특성상 평균적으로 상수 시간(

O(1)) 복잡도로 데이터에 접근할 수 있게 해주면서도, 필요한 메모리 양을 관측된 영역의 부피에 비례하도록 유지한다. 덕분에 `nvblox`는 메모리를 매우 효율적으로 사용하며, 이론적으로는 무한한 크기의 공간으로 맵을 확장할 수 있다.7

결론적으로, `nvblox`의 TSDF와 ESDF라는 이중 표현 방식은 '재구성'과 '계획'이라는 로봇 공학의 두 가지 핵심 요구사항을 하나의 프레임워크에서 동시에, 그리고 효율적으로 해결하기 위한 전략적 선택이다. TSDF는 고품질의 시각화와 정밀한 표면 모델링을 가능하게 하고, ESDF는 이를 기반으로 고속의 경로 계획과 충돌 회피를 지원한다. `nvblox`는 TSDF로부터 ESDF를 GPU 가속을 통해 매우 빠르게 파생시키는 파이프라인을 구축함으로써, 시스템의 복잡성을 줄이고 데이터의 일관성을 보장하며, 로봇이 주변 환경을 깊이 '이해'하고 그에 맞춰 '행동'하는 과정을 긴밀하게 연결하는 통합 솔루션을 제공한다.


`nvblox`의 강력한 성능은 단순히 뛰어난 알고리즘뿐만 아니라, 이를 효율적으로 실행하고 ROS 2 생태계와 유기적으로 결합하는 잘 설계된 소프트웨어 아키텍처에 기인한다. `nvblox`의 아키텍처는 프레임워크에 독립적인 코어 라이브러리와 이를 ROS 2 환경에 맞게 감싸는 래퍼(wrapper)로 명확하게 구분되어 있으며, Isaac ROS 생태계 내에서 표준화된 데이터 파이프라인을 형성한다.


`nvblox` 시스템은 두 개의 주요 구성 요소로 나눌 수 있다.


코어 라이브러리는 `nvblox`의 모든 핵심 연산 기능을 담고 있는 순수 C++ 및 CUDA로 작성된 라이브러리이다.8 이 라이브러리는 ROS 2와 같은 특정 미들웨어에 대한 의존성이 전혀 없도록 설계되어, 다양한 로보틱스 프로젝트에 독립적으로 통합될 수 있다. 코어 라이브러리의 주요 구성 요소는 다음과 같다.

- **레이어(Layers):** TSDF, ESDF, Color, Mesh 등 각기 다른 종류의 복셀 데이터를 저장하는 자료 구조이다. 이 모든 레이어를 묶어서 관리하는 `LayerCake`라는 개념이 사용된다.26

- **통합기(Integrators):** 센서 데이터를 입력받아 특정 레이어를 업데이트하는 역할을 수행하는 클래스들이다. 예를 들어, `ProjectiveTsdfIntegrator`는 깊이 이미지를 TSDF 레이어에 통합하고, `EsdfIntegrator`는 TSDF 레이어를 기반으로 ESDF 레이어를 계산한다.26

- **매퍼(Mapper):** 여러 종류의 레이어와 통합기들을 총괄하여 전체 맵핑 프로세스를 관리하는 상위 레벨의 클래스이다. `Mapper`는 정적 환경을, `MultiMapper`는 정적 맵과 동적 맵을 함께 관리하는 역할을 한다.26

  

  이 코어 라이브러리는 C++ API와 Python 바인딩을 직접 제공하여, 개발자가 ROS 2 환경이 아니더라도 nvblox의 GPU 가속 3D 재구성 기능을 활용할 수 있도록 한다.8


`nvblox_ros`는 `nvblox` 코어 라이브러리의 강력한 기능들을 ROS 2 생태계에서 손쉽게 사용할 수 있도록 만들어주는 패키지이다.10 이 패키지의 중심에는 `rclcpp::Node`를 상속받아 구현된 `NvbloxNode` 클래스가 있다.27

`NvbloxNode`는 ROS 2 노드로서 다음과 같은 역할을 수행한다.

- **ROS 인터페이스 관리:** ROS 파라미터 서버로부터 `voxel_size`, `global_frame` 등과 같은 설정을 동적으로 읽어온다. 또한, 센서 데이터 토픽을 구독(subscribe)하고, 재구성된 맵 데이터(메시, ESDF 슬라이스 등)를 토픽으로 발행(publish)하며, 맵 저장/로드와 같은 기능을 서비스(service) 형태로 제공한다.27
- **데이터 흐름 제어:** 외부 ROS 노드로부터 수신한 센서 데이터를 직접 처리하지 않고, 내부의 스레드 안전(thread-safe) 큐(queue)에 저장한다. 별도의 타이머가 주기적으로 이 큐를 확인하여 데이터를 코어 라이브러리의 `MultiMapper` 객체로 전달하고 GPU 연산을 트리거한다. 이는 ROS 콜백 스레드가 장시간 점유되는 것을 방지하여 시스템 전체의 반응성을 높인다.27
- **좌표 변환 관리:** `tf2` 라이브러리를 사용하여 로봇의 각 센서 프레임과 맵의 글로벌 프레임 간의 좌표 변환(transform) 관계를 실시간으로 조회하고, 이를 맵 통합 과정에 정확하게 반영한다.27


`nvblox_ros`는 단일 노드로서의 가치도 높지만, 그 진정한 힘은 NVIDIA Isaac ROS 생태계의 다른 'GEM(Isaac ROS 용어로 개별 기능을 수행하는 패키지를 의미)'들과 결합하여 고성능 인식 파이프라인을 구축할 때 발휘된다.11 NVIDIA는 

`nvblox`를 중심으로 한 표준화된 참조 아키텍처를 제시함으로써, 개발자들이 복잡한 통합 과정 없이도 검증된 고성능 시스템을 신속하게 구축할 수 있도록 지원한다.


일반적인 `nvblox` 기반의 자율 주행 파이프라인은 다음과 같은 노드들로 구성된 계산 그래프를 형성한다.11

- **센서 드라이버 노드 (예: `realsense_ros`):** 물리적인 깊이 카메라로부터 이미지 데이터를 받아 ROS 토픽으로 발행한다.
- **`isaac_ros_visual_slam` (자세 추정):** 스테레오 카메라의 이미지 스트림을 입력받아 로봇의 6-DoF 자세(pose)를 실시간으로 추정한다. 이 자세 정보는 `/tf` 토픽을 통해 다른 노드들이 사용할 수 있도록 브로드캐스팅된다. `nvblox`는 이 자세 정보를 기준으로 각 시점의 센서 데이터를 정합하여 일관된 3D 맵을 구축한다.3
- **`isaac_ros_unet` (의미론적 분할):** 동적 객체(특히 사람)가 존재하는 환경에서는, 컬러 이미지가 이 노드로 전달된다. `unet` 노드는 `PeopleSemSegNet`과 같은 사전 훈련된 DNN 모델을 사용하여 이미지 내에서 사람에 해당하는 영역을 픽셀 단위로 식별하고, 그 결과를 분할 마스크(segmentation mask) 이미지로 출력한다.14
- **`nvblox_ros` (`NvbloxNode`):** 위 노드들로부터 깊이 이미지, 자세, 그리고 분할 마스크를 입력받아 GPU에서 3D 맵(TSDF, ESDF 등)을 생성한다.
- **`nav2_stack` (내비게이션):** `nvblox`가 생성한 2D ESDF 슬라이스를 `nvblox_nav2` 플러그인을 통해 로컬 코스트맵(local costmap)으로 입력받아, 동적 장애물을 회피하는 경로를 실시간으로 계획하고 로봇을 제어한다.
- **`rviz2` (시각화):** `nvblox`가 발행하는 메시(mesh) 토픽을 구독하여, `nvblox_rviz_plugin`을 통해 재구성된 3D 환경을 시각적으로 확인한다.


이러한 Isaac ROS GEM들 간의 데이터 전송은 NITROS라는 고성능 전송 계층을 통해 최적화된다.29 전통적인 ROS 통신은 데이터를 CPU 메모리로 복사한 후 직렬화하여 전송하고, 수신 측에서 다시 역직렬화하여 CPU 메모리에 올린 뒤, 필요하다면 다시 GPU 메모리로 복사하는 과정을 거친다. 이는 상당한 지연과 오버헤드를 유발한다. 반면, NITROS는 '타입 변환(type adaptation)'과 '협상(negotiation)'을 통해, GPU 가속 노드 간에는 데이터가 GPU 메모리에 머무른 채 포인터만 전달되도록 한다. 이로써 불필요한 메모리 복사와 CPU 개입을 최소화하여 전체 파이프라인의 처리량과 지연 시간을 극적으로 개선한다.29


`nvblox_node` 내부의 데이터 처리 흐름은 실시간성과 일관성을 모두 달성하기 위해 '이벤트 기반 비동기 처리'와 '주기적 동기화'의 하이브리드 모델을 채택하고 있다.

1. **입력 단계 (Input Stage):** `NvbloxNode`는 다양한 센서 데이터 토픽을 비동기적으로 구독한다. 주요 입력 토픽은 다음과 같다.33
   - `depth/image`, `depth/camera_info`: 맵 재구성을 위한 핵심 데이터인 깊이 정보.
   - `color/image`, `color/camera_info`: 재구성된 메시에 색상을 입히기 위한 선택적 데이터.
   - `pointcloud`: 깊이 카메라 대신 3D LiDAR를 사용할 경우의 입력.
   - `mask/image`: 동적 객체 분리를 위한 의미론적 분할 마스크.
   - `/tf`, `/tf_static`: 각 센서 데이터가 수집된 시점의 정확한 자세 정보.
2. **큐잉 단계 (Queuing Stage):** 각 토픽에 대한 콜백 함수가 트리거되면, 수신된 메시지에 타임스탬프를 찍어 해당 데이터 타입의 내부 큐(queue)에 즉시 저장하고 콜백을 종료한다.27 이 방식은 ROS 통신 스레드의 점유 시간을 최소화하여 높은 빈도의 센서 데이터를 누락 없이 수신할 수 있게 한다.
3. **처리 단계 (Processing Stage):** `NvbloxNode` 내부의 메인 타이머가 `tick_period_ms` 파라미터에 설정된 주기마다 `tick()` 함수를 호출한다.27
   - `tick()` 함수는 각 입력 큐를 순서대로 확인한다.
   - 큐에 처리할 데이터가 있고, 해당 데이터의 타임스탬프에 대한 유효한 `tf` 변환이 존재하면, 데이터를 큐에서 꺼내 GPU 메모리로 전송한다.
   - `MultiMapper` 객체의 적절한 통합 함수(예: `integrateDepth()`, `integrateLidar()`)를 호출하여 GPU에서 TSDF, Color 등의 레이어를 업데이트하는 CUDA 커널을 실행시킨다.27
4. **출력 단계 (Output Stage):** 별도의 타이머들이 `update_mesh_rate_hz`, `update_esdf_rate_hz` 등의 파라미터에 설정된 주기에 따라 각각의 출력 생성 및 발행 작업을 수행한다.34
   - **ESDF 업데이트 및 슬라이스:** ESDF 통합기가 실행되어 최신 TSDF를 기반으로 ESDF 맵을 업데이트한다. 그 후, `esdf_slice_height`에 지정된 높이에서 2D 평면으로 맵을 절단하여 `nvblox_msgs/DistanceMapSlice` 타입의 메시지를 생성하고 `~/map_slice` 토픽으로 발행한다. 이 데이터는 Nav2 스택으로 전달되어 로컬 코스트맵으로 활용된다.12
   - **메시 생성 및 발행:** Marching Cubes 알고리즘이 TSDF 레이어의 영점 집합을 찾아 3D 삼각형 메시를 생성한다. 이 메시는 `nvblox_msgs/Mesh` 타입으로 `~/mesh` 토픽에 발행되어 Rviz에서 시각화된다.33
   - **기타 디버그 정보 발행:** `~/static_occupancy` (점유 격자 포인트 클라우드), `~/human_map_slice` (사람 포함 ESDF 슬라이스) 등 다양한 디버깅 및 시각화용 토픽도 각각의 주기에 맞춰 발행된다.33

이러한 아키텍처는 입력 데이터 수신(비동기 이벤트)과 핵심 연산(주기적 처리)을 분리함으로써 시스템의 반응성을 극대화하고, 연산 부하가 큰 작업들을 중요도에 따라 다른 주기로 조절하여 전체 시스템의 실시간성을 안정적으로 보장하는 정교한 설계의 결과물이다.


로봇이 실제 환경에서 마주하는 가장 큰 어려움 중 하나는 사람, 다른 로봇, 움직이는 가구 등 예측 불가능한 동적 객체들의 존재이다. 전통적인 3D 매핑 알고리즘은 대부분 환경이 정적(static)이라는 가정 하에 설계되었기 때문에, 동적 객체는 맵의 품질을 심각하게 저하시키는 주된 요인이 된다. `nvblox`는 이러한 문제를 해결하기 위해 의미론적 분할(semantic segmentation)과 점유 확률 감쇠(occupancy decay)라는 두 가지 핵심 전략을 채택하여 동적 환경에 효과적으로 대응한다.



기본적인 TSDF 통합 알고리즘은 모든 센서 측정값이 고정된 환경으로부터 온다고 가정한다. 만약 사람이 로봇 앞을 지나가면, 알고리즘은 사람이 있던 공간을 '표면'으로 인식하고 TSDF 맵에 통합한다. 사람이 지나간 후에도 이 정보는 맵에 그대로 남아 '유령(ghosting)' 현상을 일으킨다. 이렇게 오염된 맵은 로봇의 경로 계획기가 실제로는 비어있는 공간을 장애물로 인식하여 비효율적이거나 불가능한 경로를 생성하게 만드는 원인이 된다.14


`nvblox`는 이러한 문제를 근본적으로 해결하기 위해, 동적 객체와 정적 배경을 처음부터 분리하여 처리하는 '사람 재구성' 모드를 제공한다. 이 모드는 외부의 의미론적 분할 모듈과의 협업을 통해 이루어진다.

1. **의미론적 정보 획득:** 먼저, 로봇의 컬러 카메-라에서 나온 이미지는 `isaac_ros_unet`과 같은 딥러닝 기반 분할 노드로 전달된다. 이 노드는 `PeopleSemSegNet`과 같은 사전 훈련된 DNN 모델을 사용하여 이미지 내에서 '사람'으로 분류되는 모든 픽셀을 식별하고, 그 결과를 이진 마스크(binary mask) 형태로 출력한다. 이 마스크에서 사람에 해당하는 픽셀은 1, 배경은 0의 값을 가진다.11
2. **데이터 스트림 분리:** `nvblox_node`는 이 분할 마스크를 입력받아, 동일한 타임스탬프를 가진 깊이 이미지에 적용한다. 마스크를 통해 깊이 이미지의 각 픽셀이 '정적 배경'에 속하는지, 아니면 '사람'에 속하는지를 판단하고, 이를 기반으로 원본 깊이 이미지를 '정적 배경 깊이 이미지'와 '사람 깊이 이미지'라는 두 개의 독립적인 데이터 스트림으로 분리한다.14
3. **분리된 맵 통합:** 분리된 두 데이터 스트림은 서로 다른 맵 레이어에 통합된다.
   - **정적 배경 깊이 이미지:** 이 데이터는 기존의 방식과 동일하게 TSDF 레이어에 통합된다. 이를 통해 사람이나 다른 동적 객체에 의해 오염되지 않은, 깨끗하고 영구적인 정적 환경 맵이 구축된다.
   - **사람 깊이 이미지:** 이 데이터는 TSDF 레이어가 아닌, 별도로 존재하는 '동적 점유 격자(dynamic occupancy grid)' 레이어에 통합된다. 이 레이어는 오직 사람과 같이 동적으로 분류된 객체들의 위치 정보만을 일시적으로 저장하는 역할을 한다.14

이러한 '분류 후 분리(Classify and Separate)' 전략은 `nvblox`의 동적 객체 처리 철학을 명확하게 보여준다. `nvblox`는 모든 객체의 복잡한 움직임을 추적하고 모델링하는 완전한 동적 SLAM(Simultaneous Localization and Mapping)을 지향하기보다는, 가장 일반적이고 내비게이션에 문제를 일으키는 동적 장애물(사람)을 선제적으로 '분리'함으로써 정적 맵의 무결성과 견고성을 확보하는 데 집중한다. 이는 전체 문제의 복잡성을 낮추면서도, 실제 로봇 애플리케이션에서 겪는 대부분의 동적 환경 문제를 해결하는 매우 실용적이고 효율적인 절충안이라 할 수 있다. 이 접근법은 사람을 위한 별도의 코스트맵 생성을 가능하게 하여, 로봇이 단순히 충돌을 피하는 것을 넘어 사람의 개인 공간을 존중하며 우회하는 등 사회적으로 수용 가능한 경로 계획(social navigation)을 구현할 수 있는 중요한 기술적 기반을 제공한다.14


의미론적 분할을 통해 사람을 별도의 동적 점유 격자에 성공적으로 분리했다 하더라도, 이 정보가 영구적으로 남아서는 안 된다. 사람이 특정 위치를 지나가면 그 공간은 점유되지만, 잠시 후에는 다시 비어있는 공간이 되기 때문이다. `nvblox`는 이러한 일시적인 점유 상태를 처리하기 위해 '감쇠(decay)'라는 메커니즘을 사용한다.


`nvblox`는 `decay_dynamic_occupancy_rate_hz` 파라미터에 설정된 고정된 주기로 동적 점유 격자 전체를 스캔한다. 그리고 각 복셀에 저장된 점유 확률 값을 '알 수 없음(unknown)' 상태에 해당하는 확률 값인 0.5를 향해 일정한 비율로 감소시킨다.14

이 감쇠 메커니즘은 시간이 지남에 따라 새로운 관측 정보가 들어오지 않는 복셀의 정보가 점차 희미해지다가 결국 사라지게 만드는 효과를 낳는다. 즉, 로봇의 시야에서 사라진 사람의 흔적은 맵에서 점진적으로 지워져, 로봇이 그 공간을 다시 자유롭게 통과할 수 있도록 길을 열어준다.

이 감쇠 모델은 객체의 물리적 움직임을 명시적으로 모델링하는 것이 아니라, "오래된 정보는 덜 신뢰할 수 있다"는 정보 이론적 원칙을 확률 모델에 적용한 것이다. 이는 베이즈적 관점에서 시간이 흐를수록 이전 관측 정보의 신뢰도가 떨어지고, 따라서 '점유됨' 또는 '비어있음'이라는 확신이 줄어들어 '알 수 없음' 상태로 회귀해야 한다는 개념을 단순화하여 구현한 것이다. 이 방식은 매우 간단하면서도 대부분의 시나리오에서 효과적으로 동작하는 일반적인 해결책을 제공한다.


유사한 감쇠 개념이 정적 TSDF 레이어에도 적용될 수 있다. `decay_tsdf_rate_hz` 파라미터를 통해 TSDF 복셀의 가중치(weight)를 시간이 지남에 따라 점진적으로 감소시킬 수 있다.36 이는 환경의 미세한 변화(예: 가구가 약간 옮겨짐)나 SLAM 시스템의 누적된 오차(drift)에 맵이 점진적으로 적응하도록 돕는 역할을 한다. 가중치가 높은 복셀은 여러 번 관측된 신뢰도 높은 정보이므로 천천히 감쇠되는 반면, 가중치가 낮은 복셀은 더 빨리 사라지게 된다.37

그러나 이러한 시간 기반 감쇠 모델은 명백한 한계를 가진다. 이 모델은 모든 동적 객체가 동일한 '망각' 속도를 갖는다고 가정하며, 객체의 실제 속도나 가속도와 같은 동역학적 특성을 전혀 고려하지 않는다. 따라서 객체가 잠시 멈춰있으면 정적 장애물로 오인될 수 있고, 반대로 매우 빠르게 움직이는 객체의 흔적은 감쇠 속도가 따라가지 못해 맵에 잔상이 남을 수 있다. 그럼에도 불구하고, `nvblox`의 동적 환경 대응 전략은 완벽한 해결책보다는 실시간 로봇 내비게이션이라는 구체적인 목표를 달성하기 위한 실용적이고 계산적으로 효율적인 접근법으로서 그 가치를 가진다.


`nvblox`의 가장 큰 특징은 GPU 가속을 통한 압도적인 성능이다. 이 성능은 사용되는 하드웨어 플랫폼, 센서의 종류, 그리고 각종 파라미터 설정에 따라 달라진다. `nvblox`의 성능을 올바르게 이해하고 최적으로 활용하기 위해서는 이러한 요소들에 대한 정량적인 분석과 이해가 필수적이다.


`nvblox`는 다양한 NVIDIA GPU 플랫폼에서 동작하도록 설계되었으며, 플랫폼의 성능에 따라 처리 능력에 상당한 차이를 보인다. Isaac ROS 문서에 제공된 벤치마크 결과는 Replica 데이터셋과 0.05m(5cm) 복셀 크기를 기준으로 각 플랫폼의 성능을 명확하게 보여준다.11

| 측정 항목 (Component)      | x86_64 w/ RTX 3090 (Desktop) | x86_64 w/ RTX A3000 (Laptop) | Jetson AGX Orin | Jetson Orin Nano |
| -------------------------- | ---------------------------- | ---------------------------- | --------------- | ---------------- |
| **TSDF Integration (ms)**  | 0.5                          | 0.3                          | 0.8             | 2.1              |
| **Color Integration (ms)** | 0.7                          | 0.7                          | 1.1             | 3.6              |
| **Meshing (ms)**           | 0.7                          | 1.3                          | 2.3             | 13.0             |
| **ESDF Integration (ms)**  | 0.8                          | 1.2                          | 1.7             | 6.2              |
| **Dynamics Handling (ms)** | 1.7                          | 1.4                          | 2.0             | N/A              |

**Table 1: 플랫폼별 성능 벤치마크** 11

위 표는 몇 가지 중요한 사실을 시사한다.

- **데스크톱 GPU (RTX 3090, A3000):** 데스크톱 및 고성능 워크스테이션 환경에서는 모든 처리 단계가 1-2ms 이내에 완료될 정도로 압도적인 성능을 보여준다. 이는 고해상도, 다중 카메라 설정에서도 실시간 성능을 여유롭게 보장할 수 있음을 의미한다.
- **임베디드 GPU (Jetson Orin Series):** `nvblox`의 주력 타겟인 임베디드 플랫폼에서도 뛰어난 성능을 유지한다. Jetson AGX Orin은 TSDF와 ESDF 통합에 각각 0.8ms, 1.7ms밖에 소요되지 않아 데스크톱 성능에 근접한다. 이는 AGX Orin이 복잡한 로봇 애플리케이션을 위한 강력한 온보드 AI 컴퓨터임을 증명한다.1 반면, 엔트리 레벨인 Jetson Orin Nano는 성능 저하가 눈에 띄게 나타나지만(TSDF 2.1ms, ESDF 6.2ms), 30Hz 이상의 센서 데이터를 처리하기에는 여전히 충분한 성능을 보여준다.
- **다중 카메라 지원:** `nvblox`는 다중 카메라 입력을 동시에 처리하여 단일 카메라의 시야각 한계와 폐색(occlusion) 문제를 해결할 수 있다. Jetson AGX Orin 64GB 플랫폼의 경우, 정적 재구성 모드에서는 최대 4대의 RealSense 카메라를, 사람 분할 모드에서는 최대 3대의 카메라를 각각 30Hz로 동시에 처리할 수 있는 것으로 보고되었다.1 이는 로봇 주변의 360도 환경을 실시간으로 인식하는 데 필수적인 능력이다.


`nvblox`는 깊이 카메라와 3D LiDAR라는 두 가지 주요 3D 센서 입력을 지원하며, 각각의 처리 방식과 성능 특성이 다르다.

- **깊이 카메라 (Depth Camera):** `nvblox`는 깊이 카메라에서 생성되는 조밀한(dense) 깊이 이미지를 주 입력으로 가정하고 설계되었다.11 깊이 이미지는 각 픽셀마다 거리 정보를 가지고 있어, TSDF의 투영적 통합(projective integration) 방식에 매우 이상적인 데이터 형태이다. Intel RealSense나 Stereolabs ZED와 같은 상용 깊이 카메라와 원활하게 연동된다.

- **3D LiDAR:** `nvblox`는 `sensor_msgs/PointCloud2` 형태의 3D LiDAR 포인트 클라우드 입력 또한 지원한다.1 그러나 

  `nvblox`의 LiDAR 처리 방식은 '실용성을 위한 추상화' 전략을 채택하고 있다. 이는 LiDAR 데이터를 위한 별도의 통합기를 구현하는 대신, 수신된 포인트 클라우드를 가상의 카메라 파라미터(`lidar_width`, `lidar_height`, `lidar_vertical_fov_rad`)를 이용해 가상의 깊이 이미지로 변환한 후, 이미 고도로 최적화된 깊이 이미지 통합 파이프라인을 재활용하는 방식이다.33

  이 접근법은 코드 재사용성과 개발 효율성 측면에서 영리하지만, 성능과 품질 면에서 트레이드오프를 내포한다. LiDAR 데이터는 본질적으로 희소(sparse)하기 때문에, 변환된 깊이 이미지에는 스캔 라인 사이에 정보가 없는 빈 픽셀이 많이 존재하게 된다. 이는 재구성된 표면에 인공적인 구멍이나 부정확성을 유발할 수 있다. 또한, 포인트 클라우드를 이미지로 변환하는 과정 자체가 추가적인 계산 오버헤드를 발생시킬 수 있다. 따라서 nvblox의 LiDAR 지원은 기능적으로는 가능하지만, 희소 포인트 클라우드 처리에 처음부터 특화된 다른 LiDAR SLAM 알고리즘과 동일한 수준의 품질과 성능을 기대하기는 어려울 수 있다. 실제로 유효하지 않은 포인트(NaN 또는 0 값)가 포함된 LiDAR 데이터가 파이프라인 전체를 중단시키는 문제가 보고되기도 했다.40


`nvblox`의 성능과 결과물의 품질은 다양한 ROS 파라미터에 의해 크게 좌우된다. 사용자는 자신의 애플리케이션 요구사항과 하드웨어 사양에 맞춰 이 파라미터들을 신중하게 튜닝해야 한다. 파라미터는 C++ 소스 코드가 아닌, `.yaml` 설정 파일을 통해 관리되며, `nvblox_examples_bringup` 패키지에 제공되는 `nvblox_base.yaml` 파일이 기본적인 설정을 담고 있다.34

| 파라미터 (ROS Parameter)                           | 타입   | 기본값        | 설명 및 튜닝 가이드                                          |
| -------------------------------------------------- | ------ | ------------- | ------------------------------------------------------------ |
| `voxel_size`                                       | float  | 0.05          | 맵의 해상도(미터 단위). 작을수록 정밀하지만 메모리/계산량 급증. AMR 내비게이션에는 5cm, 로봇 팔 조작에는 1-2cm가 일반적. 34 |
| `mapping_type`                                     | string | "static_tsdf" | 매핑 타입. "static_tsdf", "static_occupancy", "dynamic" 등을 선택. 동적 객체 처리가 필요하면 "dynamic"으로 설정. 34 |
| `esdf_mode`                                        | string | "2d"          | ESDF 계산 방식. "2d" 또는 "3d". 지상 로봇(AMR)은 주로 "2d"를 사용하여 계산량을 줄임. 드론이나 로봇 팔은 "3d"가 필요. 34 |
| `map_clearing_radius_m`                            | float  | 7.0           | 로봇 주변 지정된 반경(미터) 밖의 맵을 삭제. 0 이하이면 비활성화. 장기 실행 시 메모리 증가를 막기 위한 핵심 파라미터. 34 |
| `clear_map_outside_radius_rate_hz`                 | float  | 1.0           | 맵 삭제 작업의 실행 주기(Hz). 34                             |
| `decay_tsdf_rate_hz`                               | float  | 5.0           | TSDF 가중치 감쇠 주기(Hz). 0이면 비활성화. 환경 변화나 SLAM 오차에 점진적으로 적응하게 함. 34 |
| `decay_dynamic_occupancy_rate_hz`                  | float  | 10.0          | 동적 점유 격자 감쇠 주기(Hz). 높을수록 동적 장애물 흔적이 빨리 사라짐. 34 |
| `projective_integrator_max_integration_distance_m` | float  | 5.0           | 센서로부터 이 거리(미터) 내의 데이터만 맵 통합에 사용. 센서 노이즈가 심한 장거리를 필터링하는 데 유용. 45 |
| `projective_integrator_truncation_distance_vox`    | float  | 4.0           | TSDF 절단 거리(복셀 단위). 표면의 두께와 관련. 값이 크면 더 두꺼운 표면이 생성되나 계산량이 증가. 45 |
| `projective_integrator_max_weight`                 | float  | 5.0           | TSDF 복셀의 최대 가중치. 높을수록 맵이 안정적이나 동적 변화에 둔감해짐. 동적 환경에서는 낮추는 것이 유리. 45 |

**Table 2: 주요 ROS 2 파라미터 튜닝 가이드**

특히 `map_clearing_radius_m`과 `decay_tsdf_rate_hz` 파라미터는 `nvblox`의 기술적 정체성을 명확히 보여준다. `nvblox`는 전 세계를 영구적으로 기록하는 전역 지도 제작 도구가 아니라, 로봇의 '단기 기억'처럼 현재 주변 환경을 매우 상세하고 동적으로 표현하고, 관심 영역에서 벗어나거나 오래된 정보는 과감히 버리는 '로컬 동적 맵퍼(Local Dynamic Mapper)'로서의 역할에 충실하도록 설계되었다. 이는 `nvblox`가 SLAM 시스템의 루프 클로저로 인한 과거 자세 수정에 대응하는 메커니즘이 없다는 한계를 회피하고, Nav2의 로컬 코스트맵과 같은 역할에 집중하기 위한 의도적인 설계 철학의 반영이다.44 따라서 사용자는 `nvblox`를 전역 일관성이 보장되는 SLAM 시스템과 함께 사용하여 각 모듈의 역할을 명확히 분담하는 아키텍처를 구성해야 한다.


`nvblox`의 기술적 위상과 특징을 명확히 이해하기 위해서는, 로보틱스 분야에서 사용되는 다른 주요 매핑 프레임워크들과의 비교 분석이 필수적이다. 각 프레임워크는 서로 다른 설계 철학과 목표를 가지고 있으며, 이는 데이터 구조, 계산 플랫폼, 그리고 최종 결과물의 특성에서 뚜렷한 차이로 나타난다.


`Voxblox`는 `nvblox`의 직접적인 전신(predecessor)이라고 할 수 있으며, 많은 핵심 개념을 공유한다.

- **공통점:** 두 프레임워크 모두 TSDF를 이용한 고품질 표면 재구성과 ESDF를 이용한 경로 계획용 맵 생성을 목표로 한다. 또한, 메모리 효율성을 위해 복셀 해싱 기반의 동적 맵 할당 구조를 채택하고 있다.23

- **차이점 (플랫폼 및 성능):** 가장 결정적인 차이는 계산 플랫폼이다. `Voxblox`는 CPU에서 싱글 또는 멀티 스레드로 동작하도록 설계된 반면, `nvblox`는 NVIDIA GPU의 대규모 병렬 처리 능력을 최대한 활용하도록 처음부터 CUDA로 재설계되었다.5 이 아키텍처의 차이는 압도적인 성능 격차로 이어진다. 벤치마크에 따르면 

  `nvblox`는 `Voxblox` 대비 TSDF 통합에서 최대 177배, ESDF 계산에서 최대 31배 빠른 성능을 보여준다.2

- **차이점 (알고리즘):** 성능 향상을 위해 `nvblox`는 알고리즘 수준에서도 개선을 이루었다. 특히 ESDF 생성 시 `Voxblox`가 사용하는 순차적인 웨이브 전파 방식 대신, 병렬 처리에 훨씬 유리한 하이브리드 스위핑 알고리즘을 도입했다. 또한, `Voxblox`가 계산 편의를 위해 사용했던 준-유클리드(quasi-Euclidean) 거리 근사를 완전한 유클리드 거리 계산으로 대체하여 ESDF 맵의 정확도를 높였다.2


`OctoMap`은 확률적 점유 격자 맵핑 분야에서 가장 널리 사용되는 표준적인 프레임워크 중 하나이다.

- **데이터 구조:** `nvblox`가 해시 테이블을 이용해 복셀 블록에 O(1) 시간 복잡도로 접근하는 평면적인 구조를 가지는 반면, `OctoMap`은 3D 공간을 재귀적으로 8개의 자식 노드로 분할하는 옥트리(Octree)라는 계층적 트리 구조를 사용한다.50 옥트리는 공간의 비어있는 영역을 상위 노드 하나로 압축하여 표현할 수 있어 메모리 효율성이 높다.
- **맵 표현:** `nvblox`의 핵심이 표면까지의 '거리'를 저장하는 SDF라면, `OctoMap`의 핵심은 각 복셀이 장애물에 의해 점유되었을 '확률'을 로그 오즈(log-odds) 형태로 저장하는 것이다.12 이는 베이즈 필터 이론에 기반하여 센서의 불확실성을 통계적으로 모델링하고, 여러 측정값을 융합하여 신뢰도를 높이는 데 강점을 가진다.
- **성능:** `Voxblox`에 대한 연구에서 이미 TSDF 통합 방식이 `OctoMap`보다 빠르다고 보고된 바 있으며 6, 이를 GPU로 극단적으로 가속한 `nvblox`는 `OctoMap`과 비교할 수 없을 정도로 빠른 맵 업데이트 속도를 제공한다.


`VDBFusion`은 `nvblox`와 마찬가지로 TSDF 기반의 볼류메트릭 맵핑을 수행하지만, 매우 다른 데이터 구조와 플랫폼 철학을 가지고 있다.

- **데이터 구조:** `VDBFusion`은 영화 및 VFX 산업에서 널리 사용되는 OpenVDB 라이브러리를 기반으로 한다.53 OpenVDB는 B+ 트리와 유사한, 넓고 얕은(wide and shallow) 계층적 트리 구조를 사용하여 희소한 볼류메트릭 데이터를 매우 효율적으로 저장하고 접근한다.55 이는 

  `nvblox`의 평면적인 복셀 해싱 구조와 뚜렷하게 대조된다.

- **플랫폼 및 철학:** `nvblox`가 GPU라는 특정 하드웨어에 대한 최적화를 통해 성능을 극대화하는 '하드웨어 특화 가속' 철학을 따른다면, `VDBFusion`은 OpenVDB라는 고도로 최적화된 데이터 구조 자체의 효율성을 통해 성능을 확보하는 '데이터 구조 최적화' 철학을 따른다. 이 덕분에 `VDBFusion`은 GPU 없이 범용 CPU만으로도 64채널 LiDAR 데이터를 실시간으로 처리하는 인상적인 성능을 보여준다.53 이는 GPU 자원이 부족하거나 다른 AI 연산에 할당되어야 하는 시스템에서 `nvblox`의 강력한 대안이 될 수 있음을 시사한다.


NeRF는 최근 컴퓨터 비전과 로보틱스 분야에서 각광받고 있는 새로운 3D 장면 표현 방식으로, 전통적인 명시적(explicit) 표현 방식인 TSDF와 근본적인 차이를 보인다.

- **표현 방식:** TSDF는 3D 공간을 복셀이라는 이산적인 단위로 명시적으로 분할하고 각 위치에 거리 값을 저장한다.60 반면, NeRF는 3D 장면을 하나의 연속적인 함수로 간주하고, 이를 심층 신경망(MLP)의 가중치(weights)로 암시적(implicit)하게 표현한다. 이 MLP는 5D 입력(3D 위치 

  `(x, y, z)`와 2D 시선 방향 `(θ, φ)`)에 대해 해당 지점의 색상(RGB)과 밀도(density, `σ`)를 출력하는 함수를 학습한다.61

- **장단점 및 로보틱스 적용:**

  - **TSDF (`nvblox`):** 가장 큰 장점은 실시간으로 센서 데이터를 맵에 통합할 수 있는 빠른 속도와, ESDF와 같은 기하학적 쿼리(예: "가장 가까운 장애물까지의 거리는?")를 매우 효율적으로 처리할 수 있다는 점이다. 이는 경로 계획이나 충돌 회피와 같이 로봇의 '행동'에 직접적으로 필요한 정보를 제공하는 데 매우 유리하다.25 단점으로는 복셀 해상도에 따라 메모리 사용량이 증가하고, 이산화로 인한 계단 현상(aliasing)이 발생할 수 있다는 점이 있다.60
  - **NeRF:** 장점은 신경망 가중치라는 매우 압축된 형태로 극도로 사실적인(photorealistic) 3D 장면을 표현할 수 있다는 것이다. 이는 고품질 시각화나 시뮬레이션-현실 격차(sim-to-real gap)를 줄이는 데 강력한 잠재력을 가진다.61 그러나 단점으로는 모델 학습과 새로운 시점 렌더링에 막대한 계산량이 필요하여 실시간 SLAM이나 경로 계획에 직접 적용하기 어렵다는 점이 있다.63 또한, NeRF가 학습하는 '밀도' 필드에서 물리적인 '점유' 상태나 '장애물까지의 거리'와 같은 명확한 기하학적 정보를 추출하는 것은 아직 연구가 진행 중인 어려운 문제이다.65

이러한 비교는 로보틱스에서 '기하학적 정확성'과 '시각적 충실도' 중 무엇을 우선시할 것인가에 대한 근본적인 질문을 제기한다. 현재로서는 로봇의 실시간 행동 제어를 위해서는 `nvblox`의 TSDF/ESDF와 같은 명시적 기하학 표현이 더 직접적이고 실용적이다. 반면, NeRF는 환경에 대한 깊은 '이해'나 고품질 시뮬레이션을 위한 표현에 더 가깝다. 미래의 로보틱스 시스템은 이 두 가지 접근법의 장점을 결합하는 하이브리드 형태로 발전할 가능성이 높다.

| 비교 항목 (Attribute) | nvblox                  | Voxblox                 | OctoMap          | VDBFusion         | NeRF-based       |
| --------------------- | ----------------------- | ----------------------- | ---------------- | ----------------- | ---------------- |
| **핵심 데이터 구조**  | 복셀 해싱               | 복셀 해싱               | 옥트리           | VDB (B+트리 유사) | MLP (신경망)     |
| **주 계산 플랫폼**    | GPU (CUDA)              | CPU                     | CPU              | CPU               | GPU (학습/추론)  |
| **맵 표현 방식**      | TSDF + ESDF             | TSDF + ESDF             | 확률적 점유 격자 | TSDF              | 밀도 + 색상 필드 |
| **동적 환경 지원**    | 의미론적 분할 + 감쇠    | -                       | 확률적 업데이트  | -                 | 동적 NeRF 변형   |
| **ESDF 생성 방식**    | 병렬 스위핑             | 웨이브 전파 (순차)      | 별도 EDT 필요    | -                 | 직접 추출 어려움 |
| **주 용도**           | 실시간 경로 계획/재구성 | 실시간 경로 계획/재구성 | 확률적 3D 맵핑   | 효율적 TSDF 통합  | 고품질 뷰 합성   |

**Table 3: 볼류메트릭 매핑 프레임워크 비교 분석**


`nvblox`의 강력한 실시간 3D 인식 능력은 전통적인 자율 주행을 넘어, 로봇 팔 조작, 장기적인 대규모 매핑, 그리고 의미론적 환경 이해와 같은 고도화된 로보틱스 애플리케이션으로의 확장을 가능하게 한다. 그러나 동시에 `nvblox`의 설계 철학에서 비롯된 명확한 한계점들도 존재하며, 이를 이해하고 올바른 아키텍처 내에서 활용하는 것이 중요하다.


`nvblox`는 특히 동적인 환경에서 로봇 팔이 안전하고 효율적으로 작업을 수행하는 데 핵심적인 역할을 한다. 이는 NVIDIA의 `cuMotion` 라이브러리 및 `MoveIt2` 프레임워크와의 긴밀한 통합을 통해 이루어진다.

- **`cuMotion` 및 `MoveIt2`와의 연동:** `isaac_ros_cumotion` 패키지는 `nvblox`를 핵심적인 환경 인식 모듈로 활용한다.67 로봇 팔 주변에 설치된 깊이 카메라로부터 실시간으로 3D 환경을 재구성하여, 작업 공간 내에 예기치 않게 나타나는 장애물(예: 사람의 손, 다른 물체)을 즉각적으로 감지한다.

- **ESDF의 활용:** `cuMotion`은 최적화 기반의 고속 모션 플래너이다. 이 플래너는 `nvblox`가 실시간으로 생성하고 업데이트하는 ESDF 맵을 직접적인 입력으로 사용한다.67 ESDF는 작업 공간 내 모든 지점에서 가장 가까운 장애물까지의 거리와 그 방향(그래디언트) 정보를 제공한다. 

  `cuMotion`은 이 정보를 충돌 비용 함수(collision cost function)로 활용하여, GPU 상에서 수많은 후보 궤적들을 병렬로 평가하고 최적화함으로써 충돌 없이 가장 빠르고 부드러운 경로를 수십 밀리초 내에 생성해낸다.9

- **로봇 자체 필터링:** 로봇 팔이 자기 자신의 일부를 장애물로 오인하는 문제를 방지하기 위해, `cuMotion`은 로봇의 현재 관절 각도(kinematics) 정보와 URDF 모델을 바탕으로 깊이 이미지에서 로봇 팔에 해당하는 픽셀들을 마스킹하여 제거하는 기능을 제공한다. 이를 통해 `nvblox`는 순수하게 외부 환경의 장애물만을 재구성할 수 있다.67

이러한 통합은 전통적인 '계획 후 실행(Plan-then-Execute)' 패러다임을 넘어서는 중요한 진전을 의미한다. 기존 방식에서는 정적인 환경을 가정하고 경로를 한 번 계획한 뒤, 실행 중에 예상치 못한 장애물이 나타나면 전체 계획을 중단하고 처음부터 다시 수립해야 했다. 반면, `nvblox`와 `cuMotion`의 결합은 로봇이 움직이는 동안에도 주변 환경의 변화를 ESDF에 실시간으로 반영하고, 이를 즉시 궤적 최적화에 사용하여 경로를 '지속적으로' 수정해나가는 '반응형 최적화(Reactive Optimization)' 패러다임을 가능하게 한다. 여기서 `nvblox`는 실시간 '충돌 비용장(Collision Field)' 생성기로서의 핵심적인 역할을 수행하는 것이다.


`nvblox`의 아키텍처는 실시간 로컬 매핑에 최적화되어 있으며, 이는 장기적인 대규모 매핑(long-term mapping)에는 명확한 한계를 가진다.

- **`nvblox`의 한계:** `nvblox`는 `isaac_ros_visual_slam`과 같은 외부 SLAM 시스템이 제공하는 자세(pose) 정보를 입력으로 받아 맵을 누적한다. 그러나 SLAM 시스템이 넓은 영역을 탐사하다가 이전에 방문했던 장소를 다시 인식하여 '루프 클로저(loop closure)'를 수행하면, 누적된 오차를 보정하기 위해 과거의 전체 궤적을 수정하게 된다. `nvblox`는 이렇게 소급하여 수정된 과거의 자세 정보를 반영하여 이미 생성된 맵을 업데이트하는 메커니즘을 내장하고 있지 않다.44
- **결과:** 이로 인해 루프 클로저가 발생하면, SLAM 시스템이 보고하는 로봇의 현재 위치는 지도상에서 크게 점프하지만, `nvblox`가 생성한 3D 메시는 이전의 잘못된 궤적을 따라 그대로 남아있게 되어 심각한 불일치와 맵의 파손을 야기한다.47
- **해결 방안 (아키텍처 패턴):** 이 문제는 `nvblox`의 기술적 결함이라기보다는, 현대 SLAM 시스템에서 '프론트엔드'와 '백엔드'의 역할을 명확히 구분하는 설계 철학의 반영으로 이해해야 한다. `nvblox`는 프론트엔드로부터 최신 자세를 받아 주변 환경을 상세하게 그리는 '고밀도 로컬 맵퍼'의 역할에 집중한다. 전역 일관성을 보장하는 것은 백엔드(예: VSLAM의 포즈 그래프 최적화기)의 책임이다. 따라서 `nvblox`를 올바르게 사용하는 아키텍처 패턴은 다음과 같다.
  1. **로컬 맵으로 사용 (권장 방식):** `nvblox`의 `global_frame`을 `map`이 아닌 `odom`으로 설정하고, `map_clearing_radius_m` 파라미터를 활성화하여 로봇 주변의 일정 반경 내 맵만 동적으로 생성하고 유지한다. 이는 Nav2의 로컬 코스트맵과 유사한 역할을 수행하며, 루프 클로저의 영향을 받지 않는다. 이것이 NVIDIA가 공식적으로 권장하는 사용 방식이다.44
  2. **전역 맵과의 연동 (오프라인 처리):** Kudan VSLAM과 같은 외부 SLAM 시스템과의 통합 사례에서 볼 수 있듯이, 먼저 SLAM 시스템을 이용해 전체 영역을 탐사하며 루프 클로저가 포함된 최적화된 전역 궤적을 생성한다. 그 후, 이 '정제된' 궤적을 입력으로 사용하여 저장된 센서 데이터를 `nvblox`로 재처리(re-process)함으로써, 전역적으로 일관된 고품질 3D 맵을 오프라인으로 구축할 수 있다.70


순수한 기하학적 정보만을 담은 미터법 맵(metric map)을 넘어, 각 공간이나 객체에 '의자', '테이블', '바닥'과 같은 의미론적(semantic) 레이블을 부여하는 것은 로봇이 인간과 유사한 수준에서 환경을 이해하고 상호작용하기 위해 필수적이다.71

`nvblox`의 유연한 레이어 기반 아키텍처는 이러한 의미론적 맵으로의 확장을 위한 훌륭한 기반을 제공하며, 관련 연구들이 활발히 진행되고 있다.

- **2D 분할 정보 융합:** 가장 일반적인 접근법은 2D 이미지에서 동작하는 의미론적 분할 네트워크의 출력을 3D 맵에 융합하는 것이다. 각 이미지 프레임에 대해 픽셀 단위로 의미론적 레이블을 예측한 후, 각 픽셀을 카메라 자세와 깊이 정보를 이용해 3D 공간으로 역투영(back-project)한다. 그리고 해당 픽셀이 투영되는 TSDF 복셀에 의미론적 정보를 누적시킨다.18

- **확률적 업데이트:** 단일 관측은 노이즈나 오분류에 취약하므로, 여러 각도와 시점에서 관측된 레이블 정보를 확률적으로 융합하는 것이 중요하다. 예를 들어, 각 복셀이 모든 가능한 클래스에 대한 확률 분포를 가지도록 하고, 새로운 관측이 들어올 때마다 베이즈 필터(Bayesian filter)를 사용하여 이 확률 분포를 업데이트할 수 있다. 이를 통해 시간이 지남에 따라 가장 일관되게 관측된 레이블이 해당 복셀의 최종 의미론적 정보로 수렴하게 된다.18

  `nvblox`의 `LayerCake` 구조는 기존의 TSDF, ESDF 레이어와 나란히 이러한 'Semantic' 레이어를 추가하여 관리하도록 쉽게 확장될 수 있다.

이러한 의미론적 맵 확장은 로봇이 "주방으로 가서 테이블 위의 컵을 가져와"와 같은 고수준의 자연어 명령을 이해하고 수행하는 미래의 서비스 로봇 기술을 구현하는 데 있어 핵심적인 단계가 될 것이다.


본 보고서는 NVIDIA Isaac ROS `nvblox_ros` 패키지에 대한 심층적인 기술적 고찰을 수행했다. `nvblox`는 로보틱스 분야의 실시간 3D 환경 인식에 있어 중요한 이정표를 제시하는 기술로, 그 핵심 가치와 강점, 그리고 명확한 한계점을 종합적으로 이해하는 것은 이 기술을 효과적으로 활용하고자 하는 모든 개발자와 연구자에게 필수적이다.


`nvblox_ros`의 핵심 가치는 세 가지로 요약할 수 있다.

1. **GPU 가속을 통한 실시간 고해상도 3D 인식의 대중화:** `nvblox`는 NVIDIA GPU 아키텍처와 CUDA 병렬 컴퓨팅을 극단적으로 활용하여, 이전에는 고성능 데스크톱에서나 가능했던 고품질 3D 재구성을 Jetson과 같은 저전력 임베디드 시스템에서도 실시간으로 가능하게 만들었다. 이는 AMR, 드론, 로봇 팔 등 더 넓은 범위의 로봇 플랫폼에서 정교한 3D 기반 자율성을 구현할 수 있는 문을 열었다.
2. **'재구성'과 '계획'의 통합:** 고품질 표면 재구성에 유리한 TSDF와 경로 계획에 최적화된 ESDF를 하나의 파이프라인에서 동시에, 그리고 매우 빠르게 생성함으로써 로봇의 '인식(perception)'과 '행동(planning)' 사이의 간극을 효과적으로 메웠다. 이는 로봇이 동적으로 변화하는 환경에 더 빠르고 안전하게 반응할 수 있도록 하는 핵심 요소이다.
3. **표준화된 고성능 인식 파이프라인 제공:** `nvblox_ros`는 Isaac ROS 생태계 내에서 VSLAM, 의미론적 분할 등 다른 GPU 가속 모듈들과 긴밀하게 통합된다. NITROS를 통한 효율적인 데이터 전송을 기반으로 하는 이 통합은 개발자들이 복잡한 파이프라인 구축의 부담 없이, 검증된 고성능 인식 시스템을 신속하게 개발하고 배포할 수 있는 표준 참조 아키텍처를 제공한다.


이러한 핵심 가치를 바탕으로 `nvblox_ros`의 강점과 약점을 정리하면 다음과 같다.

- **강점:**
  - **압도적인 성능:** CPU 기반 솔루션 대비 수십에서 수백 배 빠른 처리 속도.
  - **고품질/고해상도 맵:** TSDF를 통한 매끄러운 표면 재구성과 작은 복셀 크기 지원.
  - **경로 계획 친화성:** 실시간 ESDF 생성을 통해 최적화 기반 플래너와 직접 연동 가능.
  - **동적 객체 분리:** 의미론적 분할을 이용해 사람과 같은 동적 장애물을 정적 맵에서 분리하여 맵의 오염을 방지.
  - **강력한 생태계:** Isaac ROS 및 Nav2와의 긴밀한 통합.
- **약점:**
  - **GPU 의존성:** NVIDIA GPU, 특히 CUDA 지원이 필수적이므로 범용성이 제한된다.
  - **로컬 매핑으로의 역할 제한:** 루프 클로저와 같은 전역 맵 보정 메커니즘의 부재로 인해, 단독으로는 대규모 환경의 전역적으로 일관된 맵을 생성할 수 없다.
  - **제한적인 동적 객체 처리:** 현재는 주로 '사람' 클래스에 대한 분리에 초점을 맞추고 있으며, 일반적인 동적 객체(움직이는 가구, 다른 로봇 등)를 처리하는 능력은 제한적이다.
  - **희소 LiDAR 처리의 한계:** LiDAR 포인트 클라우드를 내부적으로 깊이 이미지로 변환하는 방식은 희소한 데이터의 특성을 완벽하게 활용하지 못할 수 있다.


`nvblox_ros`를 실제 로봇 시스템에 적용하고자 하는 개발자를 위한 제언은 다음과 같다.

- **적용 분야 선정:** 실시간 충돌 회피가 시스템의 성패를 좌우하는 애플리케이션에 가장 적합하다. 구체적으로는 물류 창고의 AMR, 사람과 협업하는 로봇 팔, 복잡한 환경을 비행하는 드론 등이 이상적인 적용 분야이다.
- **시스템 아키텍처 구성:** `nvblox_ros`를 단독 SLAM 시스템으로 사용해서는 안 된다. 반드시 `isaac_ros_visual_slam`과 같이 루프 클로저 기능이 포함된 강력한 외부 SLAM 시스템과 함께 구성해야 한다. 이 때, `nvblox`는 `odom` 프레임 기반의 로컬 맵을 생성하고, 전역 일관성은 외부 SLAM 시스템이 `map` 프레임을 통해 관리하는 역할 분담이 필수적이다.
- **하드웨어 선택:** 애플리케이션의 요구사항에 따라 신중한 하드웨어 선택이 필요하다. 단일 카메라와 중간 해상도의 맵핑은 Jetson Orin Nano로도 가능하지만, 다중 카메라 입력이나 고해상도(예: 1-2cm 복셀)가 필요한 로봇 팔 조작과 같은 작업에는 Jetson AGX Orin 이상의 성능을 가진 플랫폼을 권장한다.


`nvblox`는 강력한 기반 기술이지만, 여전히 개선과 확장을 위한 많은 연구 기회가 존재한다.

- **전역 일관성 확보:** SLAM 백엔드의 포즈 그래프 최적화 결과를 효율적으로 `nvblox` 맵에 반영하여, 루프 클로저 발생 시 맵을 소급하여 수정하는 메커니즘에 대한 연구가 필요하다. 이는 `nvblox`를 진정한 의미의 전역 맵퍼로 확장하는 핵심 과제가 될 것이다.
- **일반 동적 객체 처리 고도화:** 현재의 의미론적 분할 기반 접근을 넘어, 광학 흐름(optical flow)이나 장면 흐름(scene flow)과 같은 모션 기반 단서를 활용하여 클래스에 구애받지 않는 일반적인 동적 객체를 탐지하고 그 궤적을 추적하는 기능의 통합이 요구된다.72
- **의미론적 SDF의 심화:** 단순한 레이블 융합을 넘어, 각 복셀이 다중 클래스에 대한 확률 분포와 함께 객체 인스턴스 ID를 저장하는 'Panoptic SDF'로의 발전이 필요하다. 이는 로봇이 "저기 있는 세 번째 의자로 가라"와 같은 복잡한 명령을 이해하고 수행하는 데 필수적인 기술이다.
- **NeRF와의 하이브리드 접근:** `nvblox`의 명시적이고 빠른 기하학적 표현 능력과 NeRF의 고품질 시각적 표현 및 장면 이해 능력을 결합하는 하이브리드 맵 표현 방식은 차세대 로봇 인식 기술의 유력한 후보이다. 전역적인 컨텍스트 이해는 NeRF가, 로봇 주변의 실시간 상호작용은 `nvblox`의 ESDF가 담당하는 상호 보완적인 아키텍처에 대한 연구가 기대된다.

결론적으로, `nvblox_ros`는 로보틱스 3D 인식 분야에 GPU 가속이라는 강력한 도구를 제공함으로써 새로운 가능성의 시대를 열었다. 그 한계를 명확히 인지하고 올바른 아키텍처 내에서 활용한다면, 로봇이 더 복잡하고 동적인 현실 세계와 안전하게 상호작용하는 미래를 앞당기는 핵심 기술이 될 것이다.


1. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
2. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2311.00626v2
3. AMR navigation using Isaac ROS VSLAM and Nvblox with Intel Realsense camera, 8월 9, 2025에 액세스, https://www.einfochips.com/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
4. Motion Planning with Nvidia cuRobo and ROS2 | by Gaurav Gupta | Black Coffee Robotics, 8월 9, 2025에 액세스, https://medium.com/black-coffee-robotics/motion-planning-with-nvidia-curobo-and-ros2-4b16fa6b27a9
5. The logics about the ESDF update / Issue #8 / nvidia-isaac/nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/issues/8
6. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/iros_2017_voxblox.pdf
7. VoxelCache: Accelerating Online Mapping in Robotics and 3D Reconstruction Tasks - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/pdf/2210.08729
8. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox
9. nvblox/README.md at public / nvidia-isaac/nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/blob/public/README.md?plain=1
10. nvblox_ros - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/nvblox_ros/index.html
11. Isaac ROS Nvblox - isaac_ros_docs documentation, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
12. NVIDIA-ISAAC-ROS/isaac_ros_nvblox: NVIDIA-accelerated ... - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox
13. GPU Accelerated Robust Scene Reconstruction - CMU School of Computer Science, 8월 9, 2025에 액세스, https://www.cs.cmu.edu/~kaess/pub/Dong19iros.pdf
14. Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox
15. leggedrobotics/glimpse_nvblox_ros1: Static fork from Nvidia's Nvblox repo. Converted to ROS-1 and adapted for 3d-LiDAR generated depth images - GitHub, 8월 9, 2025에 액세스, https://github.com/leggedrobotics/glimpse_nvblox_ros1
16. TSDF of KinectFusion: detailed description - GitHub Gist, 8월 9, 2025에 액세스, https://gist.github.com/savuor/407fdc1807f9d5836d68aebfee726ef7
17. GLSL truncated signed distance representation (TSDF) implementation - Stack Overflow, 8월 9, 2025에 액세스, https://stackoverflow.com/questions/40560801/glsl-truncated-signed-distance-representation-tsdf-implementation
18. Online Metric-Semantic Mapping for Autonomous ... - GitHub Pages, 8월 9, 2025에 액세스, https://mit-spark.github.io/robotRepresentations-RSS2023/assets/papers/15.pdf
19. Truncated Signed Distance Function Volume Integration Based on Voxel-Level Optimization for 3D Reconstruction - IS&T | Library, 8월 9, 2025에 액세스, https://library.imaging.org/admin/apis/public/api/ist/website/downloadArticle/ei/28/21/art00006
20. Open3D: How does volumetric integration work? | by Aashutosh Pyakurel - Medium, 8월 9, 2025에 액세스, https://medium.com/ekbana/open3d-how-does-volumetric-integration-work-b41d3c14d4e6
21. Feature-Based Mobile Mapping with 3D Lidars on Embedded GPUs, 8월 9, 2025에 액세스, https://kbs.informatik.uos.de/thesis/jgaal/
22. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/rss_2016_workshop.pdf
23. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, 8월 9, 2025에 액세스, https://huggingface.co/papers/1611.03631
24. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Research Collection, 8월 9, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/128029/eth-50477-01.pdf
25. iSDF: Real-Time Neural Signed Distance Fields for Robot Perception, 8월 9, 2025에 액세스, https://www.roboticsproceedings.org/rss18/p012.pdf
26. nvblox::Mapper Class Reference, 8월 9, 2025에 액세스, https://nvblox.readthedocs.io/en/public/classnvblox_1_1Mapper.html
27. nvblox_node.hpp - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_ros/include/nvblox_ros/nvblox_node.hpp
28. isaac_ros_nvblox/nvblox_ros/src/lib/nvblox_node.cpp at main / NVIDIA-ISAAC-ROS ... - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_ros/src/lib/nvblox_node.cpp
29. Isaac ROS (Robot Operating System) - NVIDIA Developer, 8월 9, 2025에 액세스, https://developer.nvidia.com/isaac/ros
30. AMR Navigation Using Isaac ROS VSLAM and Nvblox with Intel Realsense Camera, 8월 9, 2025에 액세스, https://www.einfochips.com/blog/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
31. People Avoidance and Following in Isaac Sim Using Isaac ROS U-NET Segmentation, 8월 9, 2025에 액세스, https://medium.com/@kabilankb2003/people-avoidance-and-following-in-isaac-sim-using-isaac-ros-u-net-segmentation-a4960ad39309
32. Build High Performance Robotic Applications with NVIDIA Isaac ROS Developer Preview 3, 8월 9, 2025에 액세스, https://developer.nvidia.com/blog/build-high-performance-robotic-applications-with-nvidia-isaac-ros-developer-preview-3/
33. nvblox_ros1/docs/topics-and-services.md at main / ethz-asl ... - GitHub, 8월 9, 2025에 액세스, https://github.com/ethz-asl/nvblox_ros1/blob/main/docs/topics-and-services.md
34. isaac_ros_nvblox/nvblox_examples/nvblox_examples_bringup/config/nvblox/nvblox_base.yaml at main - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_examples/nvblox_examples_bringup/config/nvblox/nvblox_base.yaml
35. Isaac ROS Office Hours: Isaac Perceptor - YouTube, 8월 9, 2025에 액세스, https://www.youtube.com/watch?v=ZHivDfuEGmE
36. isaac_manipulator/isaac_manipulator_bringup/config/nvblox/nvblox_manipulator_base.yaml at main - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_manipulator/blob/main/isaac_manipulator_bringup/config/nvblox/nvblox_manipulator_base.yaml
37. Nvblox decay rate - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/nvblox-decay-rate/304609
38. Evaluation of 3D Nvblox map with Real World Map - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/evaluation-of-3d-nvblox-map-with-real-world-map/317787
39. NVIDIA-Isaac-ROS-Nvblox/docs/topics-and-services.md at main - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/topics-and-services.md
40. LiDAR intrinsics are inconsistent with the received pointcloud. Failing integration. / Issue #107 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/107
41. Isaac ROS VSLAM and Nvblox - how to change internal parameters, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/isaac-ros-vslam-and-nvblox-how-to-change-internal-parameters/306635
42. Changing launch parameters for nvblox realsense example - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/changing-launch-parameters-for-nvblox-realsense-example/336623
43. Nvidia Isaac ROS Nvblox, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/nvidia-isaac-ros-nvblox/311053
44. What's the realistic maximum size this can run on an Orin? / Issue #96 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/96
45. parameters.md - Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/parameters.md
46. Nvblox with realsense or any depth camera on Jetson Xavier AGX / Issue #23 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/23
47. Does nvblox support loop closure? - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/does-nvblox-support-loop-closure/275738
48. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2410.21149v1
49. Voxblox - voxblox documentation, 8월 9, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/
50. Benchmarking Occupancy Mapping libraries - Nicolò Valigi, 8월 9, 2025에 액세스, https://nicolovaligi.com/articles/occupancy-mapping-benchmarks/
51. OctoMap - 3D occupancy mapping, 8월 9, 2025에 액세스, https://octomap.github.io/
52. OctoMap: An Efficient Probabilistic 3D Mapping Framework Based on Octrees - Washington, 8월 9, 2025에 액세스, https://courses.cs.washington.edu/courses/cse571/16au/slides/hornung13auro.pdf
53. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/358523933_VDBFusion_Flexible_and_Efficient_TSDF_Integration_of_Range_Sensor_Data
54. The VDB data structure [15] compared to octrees [52]. A conventional... - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/figure/The-VDB-data-structure-15-compared-to-octrees-52-A-conventional-octree-subdivides_fig2_358523933
55. Insight: VDB, a deep dive - JangaFX, 8월 9, 2025에 액세스, https://jangafx.com/insights/vdb-a-deep-dive
56. OpenVDB Overview, 8월 9, 2025에 액세스, https://www.openvdb.org/documentation/doxygen/overview.html
57. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - Frontiers, 8월 9, 2025에 액세스, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2020.600387/full
58. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor ..., 8월 9, 2025에 액세스, https://www.mdpi.com/1424-8220/22/3/1296
59. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - PubMed, 8월 9, 2025에 액세스, https://pubmed.ncbi.nlm.nih.gov/35162040/
60. (PDF) Neural Fields in Robotics: A Survey - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/385318171_Neural_Fields_in_Robotics_A_Survey
61. NeRFs in Robotics: A Survey - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2405.01333v2
62. Neural Fields in Robotics: A Survey - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2410.20220v1
63. NeRF in Robotics: A Survey - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2405.01333v1
64. An Effective Scheme to Accelerate NeRF for Web Applications Using Hash-based Caching and Precomputed Features | Journal of Web Engineering - River Publishers, 8월 9, 2025에 액세스, https://journals.riverpublishers.com/index.php/JWE/article/view/27355
65. Neural Radiance Fields of Robots for Planning with Contact - Stanford University, 8월 9, 2025에 액세스, https://web.stanford.edu/class/cs231a/prev_projects_2022/mCS231A_Final_Project_Report.pdf
66. Vision-Only Robot Navigation in a Neural Radiance World - Michal Adamkiewicz, 8월 9, 2025에 액세스, https://mikh3x4.github.io/nerf-navigation/assets/NeRF_Navigation.pdf
67. NVIDIA-ISAAC-ROS/isaac_ros_cumotion: NVIDIA-accelerated packages for arm motion planning and control - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_cumotion
68. Help Integrating NVBLOX with cuMotion for Real-time Obstacle Avoidance with Franka FR3 Arm - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/help-integrating-nvblox-with-cumotion-for-real-time-obstacle-avoidance-with-franka-fr3-arm/341178
69. cuRobo, 8월 9, 2025에 액세스, https://curobo.org/
70. Towards Next-Gen Autonomous Mobile Robotics: A Technical Deep Dive into Visual-Data-Driven AMRs Powered by Kudan Visual SLAM and NVIDIA Isaac Perceptor, 8월 9, 2025에 액세스, https://www.kudan.io/blog/a-technical-deep-dive-into-visual-data-driven-amrs-powered-by-kdvisual-and-nvidia-isaac-perceptor/
71. Semantic Mapping for Navigation - Jianhao Jiao, 8월 9, 2025에 액세스, https://gogojjh.github.io/projects/2024_semantic_mapping/
72. STORM - Jiawei Yang, 8월 9, 2025에 액세스, https://jiawei-yang.github.io/STORM/
73. IOFusion: instance segmentation and optical-flow guided 3D ..., 8월 9, 2025에 액세스, https://www.researchgate.net/publication/380128610_IOFusion_instance_segmentation_and_optical-flow_guided_3D_reconstruction_in_dynamic_scenes


[NvBLox](./index.md)
# Nvblox B



현대 로보틱스 기술의 발전은 자율이동로봇(AMR)이나 로봇 팔(Manipulator)과 같은 지능형 시스템이 복잡하고 역동적인 실제 환경과 상호작용하는 능력을 요구한다.1 과거의 로봇 시스템이 정적이고 통제된 환경에서 반복적인 작업을 수행하는 데 중점을 두었다면, 현대의 로봇은 물류창고, 공장, 병원, 그리고 일상 공간과 같이 예측 불가능한 요소로 가득 찬 환경에서 안전하고 효율적으로 임무를 수행해야 한다. 이를 위한 핵심 전제 조건은 주변 환경에 대한 정확하고 실시간적인 3차원 공간 인식 능력이다.

전통적인 2D LiDAR 기반의 평면적 장애물 회피 기술은 로봇의 기본적인 이동성을 보장할 수 있었으나, 3차원 공간의 복잡성을 온전히 표현하지 못하는 명백한 한계를 가진다.3 예를 들어, 로봇 팔의 작업 반경 내에 있는 장애물, 공중에 매달린 물체, 또는 지면의 높낮이 변화와 같은 요소들은 2D 맵으로 표현하기 어렵다. 따라서, 로봇이 보다 정교한 작업을 수행하고 예측 불가능한 상황에 효과적으로 대응하기 위해서는 환경을 3차원 체적 맵(Volumetric Map)으로 표현하고 이해하는 기술이 필수적으로 요구된다.4


이러한 배경 속에서 NVIDIA는 Isaac 플랫폼을 통해 로보틱스 애플리케이션 개발의 패러다임을 전환하고 있다. NVIDIA Isaac은 단순히 개별 소프트웨어 구성 요소를 넘어, 시뮬레이션(Isaac Sim), 개발 프레임워크(Isaac ROS), 그리고 고수준 응용 워크플로우(Isaac Perceptor, Isaac Manipulator)를 아우르는 포괄적인 GPU 가속 로보틱스 솔루션이다.5 Isaac 플랫폼의 핵심 철학은 그래픽 처리 장치(GPU)의 막대한 병렬 연산 능력을 활용하여, 기존의 CPU 중심적 접근 방식으로는 실시간 처리가 어려웠던 복잡한 인식 및 계획 알고리즘을 가속하는 데 있다.

특히 Isaac ROS는 널리 사용되는 오픈소스 로봇 운영체제인 ROS 2(Robot Operating System 2)를 기반으로, NVIDIA의 고성능 CUDA 가속 라이브러리와 최적화된 AI 모델을 긴밀하게 통합한다.6 이를 통해 ROS 개발자 커뮤니티가 NVIDIA의 하드웨어 가속 기술에 쉽게 접근하고, 이를 활용하여 고성능 로봇 인식, 내비게이션, 조작 파이프라인을 신속하게 구축할 수 있도록 지원한다.


Nvblox는 NVIDIA Isaac ROS 생태계의 핵심 구성 요소 중 하나로, RGB-D 카메라나 3D LiDAR와 같은 센서로부터 입력되는 데이터를 실시간으로 처리하여 주변 환경의 3차원 장면을 재구성하는 GPU 가속 라이브러리이다.1 Nvblox의 주요 출력물은 두 가지 핵심적인 3D 표현 방식, 즉 고품질의 표면 재구성을 위한 TSDF(Truncated Signed Distance Function)와 충돌 없는 경로 계획을 위한 ESDF(Euclidean Signed Distance Function)이다.1 이 두 가지 표현을 하나의 통합된 파이프라인에서 실시간으로 생성함으로써, Nvblox는 로보틱스의 근본적인 두 축인 '인식(Perception)'과 '계획(Planning)' 사이의 간극을 효과적으로 메운다.

본 보고서는 Nvblox의 기술적 근간을 이루는 수학적 원리부터 시작하여, Isaac 생태계 내에서의 역할과 구조, 실제 ROS 2 기반 응용 워크플로우, 하드웨어별 성능 특성, 그리고 현재 기술이 가지는 명확한 한계점까지 심층적으로 고찰하는 것을 목적으로 한다. 이를 통해 로보틱스 연구자 및 개발자들이 Nvblox를 자신의 시스템에 도입하고 활용하는 데 필요한 포괄적이고 깊이 있는 기술적 분석을 제공하고자 한다.


Nvblox의 강력한 성능과 기능은 Signed Distance Function(SDF)이라는 수학적 개념에 깊이 뿌리내리고 있다. 이 장에서는 SDF의 기본 원리부터 시작하여, 이를 응용한 TSDF와 ESDF가 각각 어떻게 3D 환경의 표면을 재구성하고 로봇의 경로 계획을 지원하는지 그 원리를 상세히 분석한다.


Signed Distance Function (또는 Signed Distance Field, SDF)은 특정 공간상의 한 점 $\mathbf{x}$에서 가장 가까운 경계면(surface) $\partial\Omega$까지의 부호(sign)를 가진 유클리드 거리를 나타내는 스칼라 함수 $f(\mathbf{x})$이다.10 함수의 부호는 점 $ \mathbf{x} $가 경계면으로 둘러싸인 집합 $\Omega$의 내부에 있는지 외부에 있는지에 따라 결정된다. 일반적으로 점이 집합의 내부에 있으면 음수, 외부에 있으면 양수 값을 가지며, 경계면 바로 위에서는 0이 된다. (단, 이 부호 규칙은 관례에 따라 반대로 정의되기도 한다.)

수학적으로 SDF는 다음과 같이 정의된다.10

$$
f(\mathbf{x}) = \begin{cases} d(\mathbf{x}, \partial\Omega) & \text{if } \mathbf{x} \in \Omega \\ 0 & \text{if } \mathbf{x} \in \partial\Omega \\ -d(\mathbf{x}, \partial\Omega) & \text{if } \mathbf{x} \in \Omega^c \end{cases}
$$
여기서 $d(\mathbf{x}, \partial\Omega) = \inf_{\mathbf{y} \in \partial\Omega} \|\mathbf{x} - \mathbf{y}\|_2$는 점 $\mathbf{x}$와 경계면 ‘∂Ω‘ 위의 모든 점들 사이의 유클리드 거리 중 가장 작은 값(infimum)을 의미한다.

SDF의 가장 중요한 특징은 환경의 기하학적 구조를 암시적으로(implicitly) 표현한다는 점이다. 환경의 표면은 SDF 값이 0이 되는 지점들의 집합, 즉 영점 집합(zero-level set)으로 정확하게 정의된다. 이 속성은 로봇 경로 계획 시 매우 유용하게 사용되는데, 로봇의 특정 위치가 장애물과 충돌하는지 여부를 단순히 해당 위치의 SDF 값을 확인하는 것만으로 즉각적으로 판단할 수 있기 때문이다.1


SDF는 이론적으로 강력하지만, 무한한 공간의 모든 점에 대한 거리 값을 저장하는 것은 현실적으로 불가능하다. Truncated Signed Distance Function(TSDF)은 이러한 문제를 해결하기 위해 등장한 실용적인 표현 방식이다.

**개념의 기원과 발전**: TSDF 개념은 Microsoft의 KinectFusion 프로젝트에서 처음으로 대중화되었다.11 KinectFusion은 저렴한 소비자용 깊이 카메라(Kinect)를 사용하여, 사용자가 카메라를 들고 움직이는 동안 실시간으로 주변 환경의 고품질 3D 모델을 생성하는 시스템을 선보였다.2 이 시스템의 핵심이 바로 여러 깊이 이미지 프레임을 하나의 일관된 3D 볼륨으로 융합하는 TSDF 표현이었다. Nvblox는 KinectFusion에서 제시된 이 아이디어를 계승하고, 현대적인 GPU 아키텍처에 맞게 최적화하여 성능을 극대화한 결과물이다.16

**TSDF의 정의**: TSDF의 핵심 아이디어는 '절단(truncation)'이다. 표면 재구성에 있어 표면에서 매우 멀리 떨어진 지점의 정확한 거리 값은 중요하지 않다는 관점에서 출발한다. 따라서 TSDF는 표면 주변의 특정 거리(truncation distance, ‘t‘) 내에 있는 지점들의 SDF 값만 저장하고, 그 거리 밖에 있는 지점들의 값은 최대값(‘+t‘) 또는 최소값(‘−t‘)으로 절단하여 저장한다.17 이를 통해 전체 공간이 아닌 표면 근처의 정보만을 효율적으로 저장하여 메모리 사용량을 크게 줄일 수 있다.

**깊이 데이터 통합 및 복셀 업데이트**: Nvblox는 3D 공간을 복셀(voxel)이라는 작은 정육면체 격자로 나누어 TSDF 값을 저장한다. 새로운 깊이 이미지 $D_i$와 그에 해당하는 카메라 자세가 주어지면, 각 복셀의 TSDF 값은 가중 평균(weighted average) 방식을 통해 점진적으로 업데이트된다. 이 과정은 센서 노이즈를 완화하고 여러 시점의 정보를 통합하여 더 정확하고 일관된 표면을 만들어낸다.21

i번째 프레임이 들어왔을 때, 복셀 $\mathbf{x}$의 업데이트된 TSDF 값 $V_i(\mathbf{x})$와 누적 가중치 $W_i(\mathbf{x})$는 다음과 같은 공식으로 계산된다.21

$$
V_i(\mathbf{x}) = \frac{W_{i-1}(\mathbf{x})V_{i-1}(\mathbf{x}) + w_i(\mathbf{x})v_i(\mathbf{x})}{W_{i-1}(\mathbf{x}) + w_i(\mathbf{x})}
$$

$$
W_i(\mathbf{x}) = W_{i-1}(\mathbf{x}) + w_i(\mathbf{x})
$$

여기서 $V_{i-1}(\mathbf{x})$와 $W_{i-1}(\mathbf{x})$는 이전 프레임까지 누적된 TSDF 값과 가중치이다. $v_i(\mathbf{x})$는 현재 프레임의 깊이 데이터를 기반으로 계산된 새로운 SDF 값이며, $w_i(\mathbf{x})$는 해당 측정의 신뢰도를 나타내는 가중치이다. 일반적으로 가중치는 센서 모델에 따라 거리가 멀거나 비스듬한 각도에서 측정된 값에 대해 낮게 설정된다.19 이 과정을 모든 관련 복셀에 대해 반복하면, 깊이 정보가 TSDF 볼륨에 점진적으로 융합된다.


TSDF는 표면을 정밀하게 표현하는 데는 뛰어나지만, 절단 거리 밖의 영역에 대한 정보가 없다는 한계가 있다. 이는 로봇이 표면에서 멀리 떨어진 자유 공간(free space)을 탐색할 때, 가장 가까운 장애물까지의 거리를 알 수 없어 효율적인 경로 계획이 어렵다는 것을 의미한다. Euclidean Signed Distance Function(ESDF)은 이러한 한계를 극복하기 위해 사용된다.1

**ESDF의 필요성 및 생성**: ESDF는 맵 상의 모든 복셀에 대해 가장 가까운 점유(occupied) 복셀까지의 완전한 유클리드 거리를 저장하는 필드이다. Nvblox는 먼저 TSDF를 업데이트한 후, 이로부터 ESDF를 생성한다. 구체적으로, TSDF에서 부호가 양에서 음으로 바뀌는 경계면에 위치한 복셀들을 '사이트(site)' 복셀로 식별한다. 그 후, GPU의 병렬 처리 능력을 활용하여 병렬 스위핑(parallel sweeping)과 같은 효율적인 알고리즘을 통해 맵 상의 모든 복셀에서 가장 가까운 사이트 복셀까지의 거리를 계산한다.2

**충돌 감지 및 경로 최적화**: ESDF 맵이 구축되면 로봇의 경로 계획은 매우 효율적으로 이루어진다. 로봇의 미래 위치 $\mathbf{p}$가 주어졌을 때, 해당 위치가 장애물과 충돌하는지 여부는 단순히 ESDF 맵에서 해당 위치의 값 $ESDF(\mathbf{p})$를 조회하고, 이 값이 안전 여유(safety margin)보다 작은지 확인하는 것으로 매우 빠르게 판단할 수 있다. 더욱 중요한 것은, ESDF의 그래디언트(gradient) $\nabla ESDF(\mathbf{p})$가 해당 지점에서 장애물로부터 가장 빠르게 멀어지는 방향을 가리킨다는 점이다. 이 그래디언트 정보는 CHOMP(Covariant Hamiltonian Optimization for Motion Planning)나 TrajOpt와 같은 최적화 기반의 경로 계획 알고리즘에서 충돌 회피 비용 함수로 직접 사용되어, 부드럽고 안전한 경로를 생성하는 데 핵심적인 역할을 한다.1

이처럼 Nvblox는 하나의 통합된 GPU 가속 파이프라인 내에서 표면 재구성을 위한 TSDF와 경로 계획을 위한 ESDF를 모두 생성한다. 이 두 기능의 긴밀한 결합은 로봇이 환경을 정밀하게 '인식'하고, 그 인식 결과를 바탕으로 즉각적으로 '계획'을 수립하는 연속적인 과정을 실시간으로 가능하게 만든다. 이는 Nvblox가 단순한 3D 매핑 도구를 넘어, 실시간 로봇 행동 생성을 직접적으로 지원하도록 설계된 고성능 라이브러리임을 명확히 보여준다.


Nvblox는 독립적인 라이브러리로도 존재하지만, 그 진정한 가치는 NVIDIA Isaac ROS 생태계 내에서 다른 구성 요소들과 유기적으로 결합될 때 발휘된다. Nvblox는 단순히 3D 지도를 생성하는 것을 넘어, NVIDIA가 제시하는 엔드-투-엔드 로보틱스 솔루션의 핵심적인 중간 다리 역할을 수행한다.


Isaac ROS는 로보틱스의 다양한 기능을 모듈화된 패키지, 즉 'GEM(GPU-accelerated ROS Packages)' 형태로 제공한다.6 Nvblox는 이 GEM들 중에서 3D 장면 재구성 및 환경 표현을 담당하는 핵심적인 인식(Perception) GEM으로 자리매김하고 있다. Nvblox의 근간은 C++와 CUDA로 작성된 고성능 라이브러리이며, `isaac_ros_nvblox` 패키지는 이 핵심 라이브러리를 ROS 2 개발 환경에서 쉽게 사용할 수 있도록 래핑(wrapping)하는 역할을 한다.8 이를 통해 개발자는 복잡한 CUDA 프로그래밍 없이도 ROS 2의 표준 메시지 인터페이스를 통해 Nvblox의 강력한 GPU 가속 기능을 활용할 수 있다.


Nvblox는 더 큰 규모의 응용 프로그램을 위한 참조 워크플로우의 기반 기술로 적극 활용된다. 이는 Nvblox가 NVIDIA의 로보틱스 전략에서 얼마나 중요한 위치를 차지하는지를 보여준다.

- **Isaac Perceptor**: 자율이동로봇(AMR) 개발을 위해 사전 통합된 참조 워크플로우이다. Isaac Perceptor는 최대 8대의 카메라로부터 시각 정보를 받아 360도 서라운드 비전(surround vision)을 구축하는데, 여기서 Nvblox의 역할은 결정적이다. 여러 카메라에서 들어오는 깊이 정보를 실시간으로 융합하여 로봇 주변의 3D 점유 그리드(occupancy grid)와 내비게이션을 위한 2D 코스트맵을 생성한다. NVIDIA는 이 과정이 CPU 중심의 기존 방식보다 최대 100배 빠른 성능을 제공한다고 주장하며, 이를 통해 AMR이 복잡한 창고나 공장 환경에서 장애물을 정밀하게 인식하고 회피할 수 있도록 지원한다.4
- **Isaac Manipulator**: AI 기반 로봇 팔(manipulator) 개발을 위한 참조 워크플로우이다. 로봇 팔이 정교한 조작 작업을 수행하기 위해서는 자신의 작업 공간 내에 있는 장애물을 정확히 인식하고 충돌 없이 움직여야 한다. Isaac Manipulator에서 Nvblox는 작업 공간에 대한 실시간 3D ESDF 맵을 생성하는 역할을 담당한다. 이 ESDF 맵은 `cuMotion`과 같은 고속 모션 플래너에 입력으로 제공되어, 플래너가 장애물을 고려한 충돌 없는 경로를 매우 빠르게 계획할 수 있도록 돕는다.5


Nvblox가 3D 맵을 정확하게 생성하기 위해서는 두 가지 핵심 정보, 즉 센서 데이터(깊이, 컬러)와 해당 데이터가 측정된 시점의 로봇의 정확한 **자세(pose)** 정보가 필요하다.9 센서 데이터만으로는 각기 다른 시점에서 측정된 3D 포인트를 하나의 일관된 좌표계로 정렬할 수 없기 때문이다.

이러한 자세 추정의 역할을 담당하는 것이 바로 `isaac_ros_visual_slam` 패키지이다. 이 패키지는 NVIDIA의 고성능 VSLAM(Visual Simultaneous Localization and Mapping) 알고리즘인 `cuVSLAM`을 ROS 2 노드로 구현한 것이다.5

`cuVSLAM`은 스테레오 또는 RGB-D 카메라의 시각 정보와 IMU(Inertial Measurement Unit) 데이터를 융합하여, 로봇의 위치와 방향을 실시간으로 정밀하게 추정한다.

따라서, Nvblox를 사용하는 일반적인 파이프라인은 다음과 같은 데이터 흐름을 가진다 9:

1. 카메라 및 IMU 드라이버가 센서 데이터를 발행한다.
2. `isaac_ros_visual_slam` 노드가 이 데이터를 구독하여 로봇의 자세를 계산하고, 이를 TF(Transform) 트리나 `geometry_msgs/PoseStamped` 토픽으로 발행한다.
3. `nvblox_node`는 센서 데이터와 `isaac_ros_visual_slam`이 발행한 자세 정보를 함께 구독하여, 이를 기반으로 일관된 3D 맵을 생성한다.

이러한 구조는 Nvblox가 단독으로 동작하는 것이 아니라, NVIDIA가 제공하는 고성능 인식 파이프라인의 한 부분으로서 기능하도록 설계되었음을 명확히 보여준다. 개발자가 Nvblox를 도입한다는 것은 단순히 하나의 매핑 라이브러리를 추가하는 것을 넘어, 센서 입력 처리(`cuVSLAM`)부터 환경 인식(`Nvblox`), 그리고 행동 생성(`Nav2`, `cuMotion`)에 이르기까지 NVIDIA의 GPU 가속 로보틱스 생태계 전체를 채택하는 것을 의미할 수 있다. 이는 한편으로는 강력한 성능과 개발 편의성을 제공하지만, 다른 한편으로는 특정 하드웨어 및 소프트웨어 스택에 대한 종속성을 높이는 양면성을 가진다.


Nvblox의 강력한 기능을 실제 로봇 시스템에 통합하기 위해서는 ROS 2 인터페이스에 대한 정확한 이해가 필수적이다. 이 장에서는 `isaac_ros_nvblox` 패키지의 구조를 분석하고, 핵심 노드, 토픽, 서비스 및 주요 파라미터에 대해 상세히 설명하여 개발자가 Nvblox를 효과적으로 활용할 수 있는 실질적인 가이드를 제공한다.


`isaac_ros_nvblox` 저장소는 여러 개의 개별 ROS 2 패키지로 구성되어 있으며, 각 패키지는 명확한 역할을 수행한다 9:

- `nvblox_ros`: 핵심 C++ 라이브러리를 ROS 2 노드로 래핑하여, ROS 메시지를 통해 라이브러리의 기능을 호출하고 결과를 발행하는 역할을 담당한다.
- `nvblox_msgs`: Nvblox에서만 사용되는 커스텀 ROS 메시지 타입을 정의한다. 대표적으로 재구성된 3D 메시를 담는 `nvblox_msgs/Mesh`와 Nav2 연동을 위한 2D ESDF 슬라이스 정보를 담는 `nvblox_msgs/DistanceMapSlice`가 있다.
- `nvblox_nav2`: ROS 2 내비게이션 스택(Nav2)과 연동하기 위한 Costmap 레이어 플러그인을 제공한다.
- `nvblox_rviz_plugin`: ROS 2의 표준 시각화 도구인 RViz2에서 `nvblox_msgs/Mesh` 메시지를 효율적으로 렌더링하기 위한 플러그인을 포함한다.
- `nvblox_examples_bringup`: Isaac Sim 시뮬레이션 환경 및 Intel RealSense, Stereolabs ZED와 같은 실제 센서를 사용하여 Nvblox를 실행하는 다양한 예제 launch 파일을 제공하여, 개발자가 빠르게 시작할 수 있도록 돕는다.


Nvblox는 주로 두 가지 실행 노드를 제공하며, 사용 사례에 따라 선택하여 사용한다.23

- `nvblox_node`: 정적 환경을 재구성하거나, 'Dynablox' 알고리즘에 기반한 일반적인 동적 객체를 처리하는 기능을 수행하는 기본 노드이다.
- `nvblox_human_node`: `nvblox_node`의 기능을 모두 포함하면서, 추가적으로 사람 분할(human segmentation) 마스크를 입력받아 사람을 배경과 분리하여 별도의 동적 레이어로 관리하는 기능이 특화된 노드이다.


Nvblox 노드는 ROS 2의 표준적인 토픽 및 서비스 인터페이스를 통해 다른 노드들과 상호작용한다. 개발자가 Nvblox를 자신의 시스템에 통합하기 위해 알아야 할 핵심적인 입출력 인터페이스는 다음 표와 같다. 이 표는 여러 문서9에 분산된 API 정보를 체계적으로 통합하여 제공한다.

**Table 4.1: Nvblox ROS API 명세**

| 구분                | 토픽/서비스 이름        | 메시지 타입                                                  | 설명                                                         | 관련 노드           |
| ------------------- | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------- |
| **구독 (Input)**    | `depth/image`           | `sensor_msgs/Image`                                          | 핵심 입력. 미터(float) 또는 밀리미터(uint16) 단위의 깊이 이미지. | 모두                |
|                     | `depth/camera_info`     | `sensor_msgs/CameraInfo`                                     | 깊이 카메라의 내부 파라미터. `depth/image`와 쌍으로 필수.    | 모두                |
|                     | `color/image`           | `sensor_msgs/Image`                                          | (선택) 재구성된 메쉬에 색상을 입히기 위한 컬러 이미지.       | 모두                |
|                     | `pose` 또는 `transform` | `geometry_msgs/PoseStamped`, `geometry_msgs/TransformStamped` | 로봇의 자세 정보. TF를 사용하지 않을 경우 토픽으로 수신.     | 모두                |
|                     | `mask/image`            | `sensor_msgs/Image`                                          | 사람 분할 마스크 (사람=1, 배경=0).                           | `nvblox_human_node` |
| **발행 (Output)**   | `~/mesh`                | `nvblox_msgs/Mesh`                                           | 재구성된 3D 메시. `nvblox_rviz_plugin`으로 시각화.           | 모두                |
|                     | `~/map_slice`           | `nvblox_msgs/DistanceMapSlice`                               | Nav2 Costmap 플러그인을 위한 2D ESDF 슬라이스.               | 모두                |
|                     | `~/combined_map_slice`  | `nvblox_msgs/DistanceMapSlice`                               | 정적+동적 장애물을 모두 포함한 2D ESDF 슬라이스.             | `nvblox_human_node` |
|                     | `~/esdf_pointcloud`     | `sensor_msgs/PointCloud2`                                    | ESDF를 시각화하기 위한 포인트 클라우드.                      | 모두                |
|                     | `~/human_occupancy`     | `sensor_msgs/PointCloud2`                                    | 분리된 사람 레이어의 점유 복셀을 시각화.                     | `nvblox_human_node` |
| **서비스 (Action)** | `~/save_map`            | `nvblox_msgs/srv/FilePath`                                   | 현재 맵(TSDF, ESDF 등)을 `.nvblx` 파일로 저장.               | 모두                |
|                     | `~/load_map`            | `nvblox_msgs/srv/FilePath`                                   | 파일에서 맵을 불러오기.                                      | 모두                |
|                     | `~/save_ply`            | `nvblox_msgs/srv/FilePath`                                   | 현재 메시를 `.ply` 파일로 저장.                              | 모두                |


Nvblox의 동작은 다양한 ROS 파라미터를 통해 세밀하게 조정될 수 있다. 성능과 재구성 품질에 큰 영향을 미치는 주요 파라미터는 다음과 같다.23

- `voxel_size` (단위: 미터): 맵을 구성하는 복셀의 크기를 결정한다. 이 값이 작을수록 맵의 해상도가 높아져 더 정밀한 표면 표현이 가능하지만, 동일한 공간을 표현하는 데 더 많은 메모리와 연산량을 필요로 한다.
- `mapper.projective_integrator_truncation_distance_vox` (단위: 복셀): TSDF의 절단 거리를 복셀 단위로 설정한다. 이 값이 크면 더 두꺼운 표면이 재구성되고 노이즈에 강해지지만, 얇은 구조물을 표현하기 어려울 수 있다. 일반적으로 2~5 사이의 값이 사용된다.
- `esdf_update_rate_hz` (단위: Hz): ESDF 맵을 계산하고 발행하는 주기. 이 값이 높을수록 동적 장애물에 대한 반응성이 좋아지지만, GPU 연산 부하가 증가한다.
- `use_static_occupancy_layer` (bool): `true`로 설정하면, 정적 환경에 대해 TSDF 대신 확률적 점유 그리드(probabilistic occupancy grid)를 사용한다. 이는 TSDF보다 메모리 사용량이 적지만, 표면 재구성 품질은 낮아질 수 있다.
- `mapper.tsdf_integrator_max_weight`: TSDF 융합 시 한 복셀이 가질 수 있는 최대 가중치를 설정한다. 이 값이 높으면 맵이 한번 생성된 후 잘 변하지 않아 정적인 환경에서 높은 품질을 보이지만, 동적인 환경 변화나 오측정에 대한 수정이 느려진다.

이러한 파라미터들은 로봇의 응용 분야, 사용 가능한 하드웨어 자원, 그리고 환경의 특성을 고려하여 신중하게 튜닝되어야 한다.


로봇이 활동하는 실제 환경은 정적인 공간이 아니다. 사람, 다른 로봇, 움직이는 장비 등 다양한 동적 객체들이 존재한다. 이러한 동적 요소를 정적 장애물로 오인하면, 로봇은 일시적인 장애물 때문에 경로를 생성하지 못하거나, 이미 사라진 장애물을 영원히 피해 가는 비효율적인 움직임을 보이게 된다.25 Nvblox는 이러한 문제를 해결하기 위해 정적 배경과 동적 객체를 분리하여 처리하는 정교한 메커니즘을 제공한다.


가장 흔한 동적 객체인 사람을 처리하기 위해, Nvblox는 딥러닝 기반의 시맨틱 분할(Semantic Segmentation) 기술을 활용한다.

- **원리**: 이 모드에서는 Isaac ROS의 `isaac_ros_image_segmentation` 패키지가 함께 사용된다. 이 패키지의 `unet` 노드는 `PeopleSemSegNet`이라는 사전 훈련된 DNN 모델을 사용하여, 입력된 컬러 이미지에서 픽셀 단위로 사람 영역을 식별하고, 그 결과를 마스크 이미지(사람 영역은 1, 배경은 0으로 표시)로 생성한다.9
- **데이터 흐름 및 레이어 분리**: 생성된 마스크 이미지는 `nvblox_human_node`의 `mask/image` 토픽으로 전달된다. `nvblox_human_node`는 이 마스크 정보를 이용하여, 동일한 타임스탬프의 깊이 이미지에서 사람에 해당하는 깊이 값과 정적 배경에 해당하는 깊이 값을 분리한다.9 분리된 데이터는 각기 다른 레이어에 통합된다. 정적 배경의 깊이 정보는 기존과 같이 TSDF 레이어에 누적되어 정밀한 고정 지도를 만드는 데 사용된다. 반면, 사람에게서 측정된 깊이 정보는 별도의 'human' 점유 그리드(occupancy grid) 레이어에 통합된다.23
- **점유 확률 감소(Occupancy Decay)**: 동적 객체를 위한 레이어는 정적 레이어와 다르게 처리된다. 'human' 점유 그리드 레이어의 복셀들은 시간이 지남에 따라 점유 확률이 점차 0.5(unknown 상태)를 향해 감소(decay)한다. `decay_dynamic_occupancy_rate_hz` 파라미터로 이 감소 속도를 조절할 수 있다. 이 메커니즘 덕분에, 사람이 특정 위치에서 사라지고 나면 해당 영역의 점유 확률이 점차 낮아져 결국 다시 '이동 가능한 공간'으로 인식된다. 이는 로봇이 일시적인 장애물에 의해 영구적으로 경로가 막히는 문제를 방지한다.23


사람 외에 예측할 수 없는 다양한 종류의 동적 객체를 처리하기 위해, Nvblox는 별도의 DNN 모델 없이 순수 기하학적 정보만을 이용하는 일반 동적 재구성 모드를 제공한다.

- **기반 기술**: 이 기능은 Lukas Schmid 등의 "Dynablox: Real-time Detection of Diverse Dynamic Objects in Complex Environments"라는 연구 논문의 알고리즘에 기반을 두고 있다.27
- **원리**: 이 방법의 핵심은 높은 신뢰도를 가진 '자유 공간(freespace)' 레이어를 별도로 유지하고 활용하는 것이다. 로봇이 특정 영역을 관측했을 때, 해당 영역이 장애물 없이 비어있다고 높은 신뢰도로 판단되면 이 정보가 freespace 레이어에 기록된다. 이후, 로봇이 다시 이 영역을 관측했을 때 이전에 비어있던 freespace 레이어 위치에서 새로운 깊이 측정값이 감지되면, Nvblox는 이를 '정적인 배경의 변화'가 아닌 '동적 객체의 등장'으로 간주한다.27
- **장점 및 한계**: 이 방식의 가장 큰 장점은 DNN 모델이 필요 없으므로, 특정 클래스(예: 사람)나 카테고리에 국한되지 않고 모든 종류의 움직이는 객체를 원리적으로 탐지할 수 있다는 것이다.27 하지만 이 방법의 성능은 로봇의 자세 추정(odometry) 정확도에 크게 의존한다는 명확한 한계를 가진다. 만약 로봇의 자세 추정 오차(drift)가 동적 객체의 실제 움직임보다 크거나 비슷하다면, 시스템은 객체의 움직임을 로봇 자신의 오차로 오인하여 동적 객체를 탐지하지 못할 수 있다.27


정적 레이어와 동적 레이어가 분리되어 관리되더라도, 최종적으로 로봇의 경로 계획에는 이 두 정보가 모두 반영되어야 한다. Nvblox는 각 레이어로부터 개별적인 ESDF를 생성한 후, 이를 하나로 통합한다. 예를 들어, 정적 TSDF 레이어로부터 '정적 ESDF'를 계산하고, 사람/동적 점유 그리드로부터 '동적 ESDF'를 계산한다. 그 후, 각 복셀 위치에서 두 ESDF 값 중 더 작은 값(즉, 더 가까운 장애물까지의 거리)을 선택하는 방식으로 '통합 ESDF'를 생성한다. 이 최종 결과물은 `~/combined_map_slice`나 `~/combined_esdf_pointcloud`와 같은 토픽으로 발행되어, Nav2와 같은 경로 플래너가 정적 및 동적 장애물을 모두 고려하여 안전한 경로를 계획할 수 있도록 한다.24

이러한 모듈식 레이어 구조는 Nvblox 아키텍처의 중요한 유연성을 보여준다. 정적 배경처럼 시간에 따라 변하지 않고 누적을 통해 정밀도를 높여야 하는 정보는 TSDF 융합 방식을 사용하고, 동적 객체처럼 최신 정보가 중요하고 과거 정보는 잊혀져야 하는 정보는 점유 그리드와 decay 메커니즘을 사용하는 등, 각 정보의 특성에 맞는 최적의 처리 방식을 적용할 수 있다. 이는 향후 '차량', '카트' 등 특정 클래스에 대한 전문화된 동적 레이어를 추가하거나 사용자가 정의한 커스텀 동적 레이어를 통합하는 방향으로 시스템이 확장될 수 있는 가능성을 시사한다.


Nvblox의 가장 큰 특징은 GPU 가속을 통한 압도적인 실시간 성능이다. 이 장에서는 Nvblox의 성능을 구체적인 벤치마크 데이터를 통해 분석하고, 전통적인 CPU 기반의 매핑 라이브러리들과 비교하여 그 기술적 우위를 명확히 한다.


Nvblox는 CPU 기반의 대표적인 체적 매핑 라이브러리인 Voxblox의 핵심 아이디어를 계승하면서도, 모든 연산 과정을 NVIDIA GPU의 병렬 처리 아키텍처에 맞게 재설계하고 최적화했다.16 그 결과, 기존 방식으로는 달성하기 어려웠던 수준의 실시간 성능을 확보했다.

NVIDIA가 발표한 논문2에 따르면, Nvblox는 최신 CPU 기반 방법론과 비교했을 때, 표면 재구성(TSDF 통합) 속도에서 최대 **177배**, 그리고 경로 계획에 필수적인 거리 필드 계산(ESDF 생성) 속도에서 최대 **31배** 빠른 성능을 달성했다고 보고되었다. 이러한 성능 향상은 로봇이 더 높은 해상도의 맵을 실시간으로 구축하고, 더 빠르게 동적 환경 변화에 대응할 수 있게 함을 의미한다.


Nvblox는 고성능 데스크탑 GPU뿐만 아니라, 로봇에 직접 탑재되는 임베디드 플랫폼인 NVIDIA Jetson Orin 시리즈에서도 실시간 성능을 발휘하도록 설계되었다.1 NVIDIA가 공개한 공식 벤치마크 데이터9는 각 하드웨어 플랫폼에서 기대할 수 있는 성능 수준을 구체적으로 보여준다.

**Table 6.1: 하드웨어 플랫폼별 Nvblox 연산 시간 (Replica 데이터셋, 5cm 복셀 기준)**

| 연산 요소 | x86_64 w/ RTX 3090 (Desktop) | Jetson AGX Orin (Embedded) | Jetson Orin Nano (Embedded) |
| --------- | ---------------------------- | -------------------------- | --------------------------- |
| TSDF 통합 | 0.5 ms                       | 0.8 ms                     | 2.1 ms                      |
| 컬러 통합 | 0.7 ms                       | 1.1 ms                     | 3.6 ms                      |
| 메시 생성 | 0.7 ms                       | 2.3 ms                     | 13.0 ms                     |
| ESDF 계산 | 0.8 ms                       | 1.7 ms                     | 6.2 ms                      |

이 표는 Nvblox의 핵심 연산들이 데스크탑 GPU 환경에서는 물론, 전력 및 크기 제약이 있는 Jetson AGX Orin과 같은 임베디드 환경에서도 수 밀리초(ms) 내에 처리됨을 보여준다. 이는 로봇 온보드 컴퓨터만으로 고해상도 3D 매핑과 경로 계획을 실시간으로 수행할 수 있음을 의미하며, Nvblox의 실용적 가치를 입증하는 중요한 데이터이다. 반면, 보급형 모델인 Orin Nano에서는 연산 시간이 상당히 증가하여, 고성능을 요구하는 애플리케이션에는 제약이 따를 수 있음을 시사한다.


Nvblox의 특징은 기존의 대표적인 3D 매핑 라이브러리들과 비교할 때 더욱 명확해진다.

- **vs. Voxblox**: Voxblox는 Nvblox의 직접적인 전신 격인 라이브러리로, ETH 취리히에서 개발되었다. Voxblox는 CPU 기반으로 동작하며, 코드의 가독성과 학술적 연구를 위한 확장성을 우선시하여 설계되었다.31 Nvblox는 Voxblox의 핵심 아이디어(해시 기반 복셀 구조, TSDF에서 ESDF 생성 등)를 채택하되, 목표를 '최고 성능'으로 설정하고 모든 것을 GPU에 맞게 재작성한 결과물이다.16
- **vs. OctoMap**: OctoMap은 옥트리(Octree)라는 계층적 자료 구조를 사용하여 3D 공간을 표현하는 대표적인 점유 그리드 매핑 라이브러리이다.3 옥트리는 비어있거나 동일한 상태의 넓은 공간을 하나의 큰 노드로 압축하여 표현하므로 메모리 효율성이 뛰어나다. 반면, Nvblox는 주로 해시맵(hash map) 기반의 복셀 블록 구조를 사용하여 특정 위치의 복셀에 대한 접근 및 업데이트 속도를 높이는 데 중점을 둔다.32 또한, OctoMap이 각 복셀의 '점유 확률'만을 저장하는 데 비해, Nvblox는 '표면까지의 거리' 정보를 저장하는 TSDF/ESDF를 사용하므로, 더 높은 품질의 표면 메시를 추출하고 경로 계획에 더 유용한 정보를 제공할 수 있다.1


Nvblox의 등장은 후속 연구들에게 새로운 기준점이자 기반 기술이 되고 있다. coVoxSLAM과 같은 최신 연구는 Nvblox를 기반으로 하거나 주요 비교 대상으로 삼는다.16 coVoxSLAM은 Nvblox가 다루지 않는 SLAM의 백엔드(Backend) 영역, 즉 루프 클로저 감지 및 포즈 그래프 최적화(Pose Graph Optimization)까지 모두 GPU로 가속화하여 완전한 GPU 기반 SLAM 시스템을 제안한다. 이 연구에서는 자신들의 TSDF 통합 방식이 Nvblox보다 1.5배에서 2배가량 더 빠르다고 주장하며, 이는 Nvblox가 개척한 GPU 가속 매핑 분야가 계속해서 발전하고 있음을 보여준다.29

Nvblox와 Voxblox의 성능을 비교할 때 흥미로운 점은, 맵의 해상도가 높아질수록(즉, 복셀 크기가 작아질수록) Nvblox의 상대적인 성능 향상 폭이 감소하는 경향이 있다는 것이다.16 이는 Nvblox의 연산 방식과 관련이 있다. Nvblox는 개별 복셀이 아닌, 여러 복셀을 묶은 '복셀 블록(VoxelBlock)' 단위로 GPU 연산을 수행한다. 센서 데이터를 맵에 통합할 때, 센서의 시야각(FOV)에 걸쳐 있는 모든 복셀 블록이 처리 대상이 된다. 해상도가 높아지면 동일한 공간을 표현하는 데 더 많은 복셀이 필요하고, 결과적으로 더 많은 복셀 블록이 활성화된다. 이 과정에서 센서 광선이 실제로 통과하지 않는 블록 내의 불필요한 복셀까지 함께 처리되는 오버헤드가 발생하며, 이 비효율성이 해상도가 높아질수록 상대적으로 커져 전체적인 성능 향상률을 둔화시키는 요인으로 작용한다. 이는 개발자가 무조건적으로 해상도를 높이기보다는, 자신의 애플리케이션이 요구하는 정밀도와 하드웨어의 실시간 처리 능력 사이에서 최적의 복셀 크기를 선택하는 엔지니어링적 트레이드오프가 필요함을 시사한다.


Nvblox의 진가는 실제 로봇 애플리케이션에 통합되어 구체적인 문제를 해결할 때 드러난다. 이 장에서는 가장 대표적인 두 가지 응용 사례, 즉 자율이동로봇(AMR)의 내비게이션과 로봇 팔(Manipulator)의 충돌 회피 워크플로우를 통해 Nvblox가 어떻게 활용되는지 상세히 분석한다.


AMR이 동적인 환경에서 안전하고 효율적으로 주행하기 위해서는 주변 환경을 실시간으로 인식하고 경로 계획에 반영하는 것이 필수적이다. Nvblox는 ROS 2의 표준 내비게이션 스택인 Nav2와 긴밀하게 통합되어 이 문제를 해결한다.

- **워크플로우 개요**: 전체적인 데이터 흐름은 다음과 같다. 먼저 `isaac_ros_visual_slam` 노드가 카메라와 IMU 센서 데이터로부터 로봇의 현재 자세를 실시간으로 추정한다. `nvblox_node`는 이 자세 정보와 깊이 카메라의 데이터를 입력받아 3D TSDF/ESDF 맵을 생성한다. 마지막으로, 이 3D 맵에서 로봇이 주행하는 평면에 해당하는 2D 슬라이스를 추출하여 Nav2 스택에 코스트맵(Costmap)으로 제공한다. Nav2는 이 코스트맵을 기반으로 장애물을 피해 목적지까지의 경로를 계획하고 로봇을 제어한다.9
- **핵심 컴포넌트: `nvblox_nav2` 플러그인**: Nvblox와 Nav2의 통합은 `nvblox_nav2` 패키지에 포함된 Costmap 레이어 플러그인을 통해 매우 효율적으로 이루어진다. 이 플러그인은 Nav2의 `global_costmap` 또는 `local_costmap`의 일부로 동작하며, Nvblox 노드가 발행하는 `~/map_slice` 또는 `~/combined_map_slice` 토픽(메시지 타입: `nvblox_msgs/DistanceMapSlice`)을 직접 구독한다. 이 방식은 맵 데이터를 ROS 파라미터 서버나 별도의 파일로 저장하고 읽어오는 전통적인 방식에 비해 오버헤드가 적어, Nvblox가 생성하는 실시간 환경 변화를 거의 지연 없이 Nav2의 경로 계획에 반영할 수 있게 한다.9
- **통합 과정**: 실제 로봇에 Nvblox와 Nav2를 통합하는 과정은 다음과 같이 요약할 수 있다.34
  1. `isaac_ros_nvblox`, `isaac_ros_visual_slam`, `nav2_bringup` 등 필요한 모든 패키지를 포함하는 ROS 2 launch 파일을 작성한다.
  2. Nav2의 파라미터 설정 파일(`nav2_params.yaml`)을 수정하여, `global_costmap`과 `local_costmap`의 `plugins` 리스트에 `nvblox_layer`를 추가하고, 해당 플러그인의 `map_slice_topic` 파라미터를 Nvblox 노드가 발행하는 토픽 이름으로 정확히 지정한다.
  3. 모든 노드를 실행하면, 로봇이 움직이며 주변을 탐색함에 따라 Nvblox가 실시간으로 맵을 구축하고, 이 정보가 Nav2 코스트맵에 동적으로 업데이트되어 장애물 회피 주행이 가능해진다.


로봇 팔이 복잡한 작업 공간에서 물체를 집거나 조립하는 등의 작업을 수행할 때, 주변의 고정된 장애물이나 예기치 않게 나타나는 동적 장애물과의 충돌을 피하는 것은 매우 중요하다. Nvblox는 고속 모션 플래닝 라이브러리와 결합하여 이 문제를 해결한다.

- **워크플로우 개요**: 로봇 팔의 작업 공간을 비추는 깊이 카메라로부터 실시간으로 3D ESDF 맵이 Nvblox에 의해 생성된다. 로봇 팔의 모션 계획 프레임워크인 MoveIt 2 내에서, NVIDIA의 고속 모션 플래너인 `cuMotion`이 이 ESDF 맵을 직접 참조하여, 장애물과 충돌하지 않으면서도 부드럽고 시간 최적화된 궤적(trajectory)을 실시간으로 생성한다.22

- **핵심 컴포넌트: `cuMotion`과 `cuRobo`**: `cuMotion`은 NVIDIA가 MoveIt 2와 통합하여 사용할 수 있도록 제공하는 고성능 모션 플래닝 라이브러리이다.6

  `cuMotion`의 내부에는 `cuRobo`라는 GPU 가속 로보틱스 라이브러리가 백엔드로 동작하고 있다. `cuRobo`는 Nvblox로부터 생성된 3D ESDF 맵을 직접 입력받아, GPU의 병렬 처리 능력을 이용해 로봇 팔의 수많은 지점과 주변 환경 간의 충돌 거리를 동시에 계산한다. 이 정보를 바탕으로 충돌을 회피하는 최적의 경로를 매우 빠르게 찾아낸다.38

- **데이터 흐름 및 아키텍처**: 전체적인 아키텍처는 Isaac Manipulator 참조 워크플로우에 잘 나타나 있으며, 데이터 흐름은 다음과 같다.40

  1. 깊이 카메라가 작업 공간의 깊이 이미지를 제공한다. (선택적으로, `cuMotion`의 로봇 분할 기능을 사용하여 깊이 이미지에서 로봇 팔 자신의 모습을 제거할 수 있다 22).
  2. Nvblox는 이 깊이 이미지와 로봇 팔의 현재 상태(kinematics)를 기반으로 3D ESDF 맵을 실시간으로 업데이트한다.
  3. 사용자가 MoveIt 2를 통해 목표 자세를 지정하면, `isaac_ros_cumotion_moveit` 플러그인이 `cuMotion` 플래너를 호출한다.
  4. `cuMotion`은 Nvblox로부터 최신 ESDF 맵을 받아 `cuRobo`의 충돌 검사기에 전달하고, `cuRobo`는 GPU 가속 최적화를 통해 충돌 없는 경로를 계산하여 반환한다.38

이 두 가지 핵심 응용 사례는 Nvblox가 생성하는 **ESDF** 표현의 범용성과 강력함을 명확히 보여준다. AMR 내비게이션에서는 3D ESDF를 2D 평면으로 잘라낸 `map_slice`가 사용되고, 로봇 팔 충돌 회피에서는 완전한 3D ESDF가 사용된다. 서로 다른 로봇과 다른 차원의 계획 문제임에도 불구하고, '가장 가까운 장애물까지의 거리'라는 ESDF의 본질적인 정보는 두 경우 모두에서 효과적으로 활용된다. 이는 Nvblox가 단순히 시각화를 위한 3D 모델을 만드는 것을 넘어, 로봇이 직접 해석하고 행동 계획에 사용할 수 있는 **'기계-판독 가능(machine-readable)' 월드 모델**을 생성하는 핵심 기술임을 시사한다. NVIDIA는 이 일관된 데이터 표현을 중심으로 각 도메인에 맞는 플러그인과 인터페이스를 제공함으로써, Nvblox라는 단일 기술을 전체 로보틱스 생태계에 걸쳐 효과적으로 재사용하고 있다.


Nvblox는 실시간 3D 인식 분야에서 혁신적인 성능을 제공하지만, 모든 문제를 해결하는 만능 솔루션은 아니다. 그 설계 철학과 현재 기술 수준에서 비롯되는 명확한 한계점들이 존재한다. 개발자는 이러한 한계를 정확히 인지하고 시스템을 설계해야 Nvblox의 잠재력을 최대한 활용할 수 있다.


Nvblox의 가장 근본적인 한계는 SLAM(Simultaneous Localization and Mapping) 시스템의 전체 파이프라인 중 프론트엔드(자세 추정)와 매핑 부분만을 다룬다는 점이다. 즉, 장시간 탐사 후 이전에 방문했던 장소로 돌아왔을 때 발생하는 오차(drift)를 보정하는 루프 클로저(loop closure)나, 전체 경로와 지도를 최적화하는 백엔드(backend) 기능이 Nvblox 자체에는 포함되어 있지 않다.16

이는 상위의 VSLAM 시스템(예: `isaac_ros_visual_slam`)에서 루프 클로저가 발생하여 과거의 로봇 자세 정보가 수정되더라도, 그 수정된 정보가 이미 Nvblox에 의해 생성된 맵에 소급하여 적용되지 않음을 의미한다. 결과적으로, 로봇이 넓은 영역을 장시간 탐사할 경우 맵에 드리프트가 누적되어 뒤틀리거나, 같은 장소가 중복으로 생성되는 등의 비일관성 문제가 발생할 수 있다.41


Nvblox는 관측된 모든 공간에 대한 정보를 복셀 단위로 저장하므로, 탐사 영역이 넓어질수록 GPU 메모리 사용량이 계속해서 증가하는 문제를 안고 있다. 한 기술 포럼의 논의에 따르면, 5cm 해상도의 복셀을 사용하는 3D TSDF는 1 입방미터(‘m3‘)당 약 64KB의 메모리를 소모한다. 만약 64GB의 메모리를 가진 Jetson AGX Orin에서 16GB를 맵에 할당한다고 가정하면, 이는 약 250,000 $m^3$의 공간을 표현할 수 있다. 천장 높이를 3m로 가정할 경우 약 80,000 $m^2$의 바닥 면적에 해당하는데, 이는 대규모 물류창고 전체를 하나의 맵에 담기에는 부족할 수 있는 크기이다.41

이러한 한계는 Nvblox가 본질적으로 **로컬 매핑(local mapping)**, 즉 로봇 주변의 환경을 실시간으로 인식하는 데 초점을 맞춰 설계되었음을 시사한다. 이 문제를 완화하기 위해 Nvblox는 다음과 같은 메모리 관리 기능을 제공하거나 계획하고 있다 41:

- **Voxel Decay**: 로봇의 시야에서 오랫동안 벗어나 관측되지 않는 복셀의 가중치를 점차 감소시켜, 결국 맵에서 해당 복셀 정보를 제거하는 기능이다.
- **Map Cropping/Clearing**: `map_clearing_radius_m` 파라미터를 통해 설정된 로봇 주변의 특정 반경 바깥 영역의 맵 데이터를 주기적으로 삭제하여, 맵이 무한정 커지는 것을 방지한다.


현재 Nvblox가 Nav2 연동을 위해 제공하는 2D 코스트맵(`map_slice`) 생성 기능은 지면이 대체로 평평하다는 가정(flat ground assumption) 하에 동작한다.42 이는 3D ESDF 맵을 특정 높이 범위에서 잘라내어 2D로 투영하는 방식으로 구현되기 때문이다.

따라서 경사로나 언덕과 같이 지면의 높이가 변하는 비평탄 지형에서 지상 로봇을 운용할 경우, Nvblox는 경사면을 장애물로 오인하여 코스트맵에 표시할 수 있다. 이는 로봇이 충분히 주행 가능한 경사로를 통과하지 못하고 경로 계획에 실패하는 결과로 이어질 수 있다.42 이 문제를 근본적으로 해결하기 위해서는 지면의 형태를 고려하는 3D 경로 플래너가 필요하지만, 이는 현재 Nav2의 표준 기능이 아니며 Nvblox 개발 로드맵에서도 높은 우선순위를 차지하고 있지는 않은 것으로 보인다.42


- **다중 카메라 지원 한계**: 공식 문서에 따르면, 지원되는 카메라의 수와 종류에 제한이 있다. 예를 들어, Intel RealSense 카메라의 경우 정적 재구성 모드에서 최대 4대까지 지원되지만, ZED 카메라는 현재 단일 카메라만 지원되는 등 센서 및 플랫폼별로 제약이 존재한다.1
- **플랫폼 종속성**: Nvblox는 ROS 2 Humble, NVIDIA GPU(Ampere 아키텍처 이상), 그리고 NVIDIA Jetson 플랫폼에 맞춰 집중적으로 개발 및 테스트되었다. 따라서 다른 ROS 2 버전(예: Rolling, Iron)이나 타사의 GPU, 또는 CPU-only 환경에서의 호환성 및 성능은 보장되지 않는다.9

이러한 한계점들은 Nvblox의 단점이라기보다는, 그 설계 철학의 자연스러운 결과물로 해석될 수 있다. Nvblox의 목표는 '영구적이고 전역적으로 완벽한 지도를 만드는 것'이 아니라, '로봇이 **지금 당장** 마주한 주변 환경을 정확히 인식하고 **실시간**으로 안전하게 행동하는 데 필요한 정보를 제공하는 것'에 있다. 실시간성을 최우선으로 확보하기 위해 과거 데이터의 전역적 일관성을 일부 희생하고, 자원 사용량을 제어하기 위해 로봇과 멀리 떨어진 데이터는 과감히 버리는 전략을 채택한 것이다. 따라서 개발자는 Nvblox를 '만능 SLAM 솔루션'으로 기대하기보다는, **'고성능 실시간 로컬 인식 센서'**로 간주하고 시스템을 설계해야 한다. 만약 전역 지도가 반드시 필요한 애플리케이션이라면, Nvblox를 로컬 플래너용으로 사용하고, 이와는 별개로 2D LiDAR로 만든 정적 지도를 기반으로 하는 AMCL과 같은 전역 측위 시스템을 함께 사용하는 하이브리드 접근 방식이 효과적인 대안이 될 수 있다.4



NVIDIA Isaac Nvblox는 현대 로보틱스가 직면한 실시간 3D 환경 인식이라는 핵심 과제에 대한 강력한 해답을 제시한다. Nvblox는 NVIDIA의 GPU 하드웨어와 CUDA 병렬 프로그래밍 모델의 이점을 극대화하여, 기존의 CPU 기반 방식으로는 실시간 처리가 어려웠던 고해상도 3D 표면 재구성(TSDF)과 경로 계획을 위한 거리장 계산(ESDF)을 가능하게 했다.

Nvblox의 가장 중요한 기술적 기여는 단순히 속도 향상에 그치지 않는다. TSDF와 ESDF라는 두 가지 핵심 데이터 표현을 하나의 통합된 GPU 가속 파이프라인에서 동시에 생성함으로써, 로보틱스의 근본적인 두 축인 '인식(Perception)'과 '계획(Planning)'을 긴밀하고 효율적으로 연결하는 강력한 가교 역할을 수행한다. 이를 통해 로봇은 주변 환경을 정밀하게 모델링하고, 그 모델을 기반으로 즉각적으로 안전하고 효율적인 행동 계획을 수립할 수 있게 되었다.


Nvblox는 강력한 도구이지만, 그 특성을 정확히 이해하고 활용할 때 최고의 가치를 발휘한다. 로보틱스 개발자가 Nvblox 도입을 고려할 때 다음 사항들을 염두에 두어야 한다.

- **적합한 응용 분야**: 사람이나 다른 로봇과 같이 예측 불가능한 동적 장애물이 많고, 실시간 반응성이 시스템의 성패를 좌우하는 실내 AMR 내비게이션이나 로봇 팔의 동적 충돌 회피와 같은 작업에 매우 효과적이다.
- **도입 시 고려사항**: Nvblox의 압도적인 성능은 NVIDIA GPU 및 Jetson 플랫폼 위에서 발휘되도록 최적화되어 있다. 따라서 Nvblox의 성능을 최대한 활용하기 위해서는 `isaac_ros_visual_slam`(cuVSLAM)과 같은 Isaac ROS 생태계 내의 다른 구성 요소들과의 통합을 적극적으로 고려하는 것이 바람직하다. 이는 특정 하드웨어 및 소프트웨어 생태계에 대한 의존성을 수반하는 결정임을 인지해야 한다.
- **한계 인지 및 보완 전략**: Nvblox를 전역적이고 영구적인 지도를 생성하는 만능 SLAM 솔루션으로 기대해서는 안 된다. Nvblox는 본질적으로 '실시간 로컬 인식 센서'에 가깝다. 만약 애플리케이션이 대규모 환경에 대한 전역 지도를 필요로 한다면, Nvblox는 로컬 경로 계획 및 동적 장애물 회피용으로 사용하고, 이를 보완할 별도의 전역 지도 생성 및 측위 시스템(예: 2D LiDAR 기반의 지도와 AMCL)을 함께 설계하는 하이브리드 접근 방식을 취하는 것이 현명하다.


Nvblox는 NVIDIA의 적극적인 지원 하에 지속적으로 발전할 것으로 기대된다. 향후에는 더 다양한 종류의 동적 객체를 분리하여 처리하는 기능, 비평탄 지형에서의 내비게이션 지원 강화, 그리고 대규모 환경에서의 메모리 관리 효율성 향상 등이 이루어질 것이다.

궁극적으로 Nvblox와 같이 GPU 가속에 기반한 고성능 인식 라이브러리의 등장은 로보틱스 기술의 발전을 한 단계 끌어올리는 중요한 동력이 될 것이다. 로봇이 더욱 복잡하고 예측 불가능한 환경 속에서 인간과 안전하게 공존하고 상호작용하는 미래를 앞당기는 데 있어, Nvblox는 핵심적인 기반 기술 중 하나로 자리매김할 것이다.


1. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 5, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
2. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 5, 2025, https://arxiv.org/html/2311.00626v2
3. Outdoor Navigation in ROS: Current State & Future Direction - Open Robotics Discourse, accessed August 5, 2025, https://discourse.ros.org/t/outdoor-navigation-in-ros-current-state-future-direction/33599
4. NVIDIA Isaac Perceptor - NVIDIA Developer, accessed August 5, 2025, https://developer.nvidia.com/isaac/perceptor
5. NVIDIA Isaac Summary - Tutorials, accessed August 5, 2025, https://tutorial.j3soon.com/robotics/nvidia-isaac-summary/
6. Isaac ROS (Robot Operating System) - NVIDIA Developer, accessed August 5, 2025, https://developer.nvidia.com/isaac/ros
7. Getting Started - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 5, 2025, https://nvidia-isaac-ros.github.io/getting_started/index.html
8. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, accessed August 5, 2025, https://github.com/nvidia-isaac/nvblox
9. Isaac ROS Nvblox - isaac_ros_docs documentation, accessed August 5, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
10. Signed distance function - Wikipedia, accessed August 5, 2025, https://en.wikipedia.org/wiki/Signed_distance_function
11. KinectFusion: Real-Time Dense Surface Mapping and ... - Microsoft, accessed August 5, 2025, https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/ismar2011.pdf
12. (PDF) KinectFusion: Real-time dynamic 3D surface reconstruction and interaction, accessed August 5, 2025, https://www.researchgate.net/publication/220721971_KinectFusion_Real-time_dynamic_3D_surface_reconstruction_and_interaction
13. KinectFusion: Real-Time Dense Surface Mapping and Tracking - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/221221368_KinectFusion_Real-Time_Dense_Surface_Mapping_and_Tracking
14. Brief Analysis of KinectFusion: Real-Time Dense Surface Mapping and Tracking - Computer Vision Group - Technische Universität München, accessed August 5, 2025, https://cvg.cit.tum.de/_media/teaching/ss2019/seminar_3dcv/material/02_report_jin_tian.pdf
15. Moving Volume KinectFusion - Northeastern University, accessed August 5, 2025, https://www2.ccs.neu.edu/research/gpc/publications/Roth_Vona__2012__Moving_Volume_KinectFusion.pdf
16. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping | Request PDF, accessed August 5, 2025, https://www.researchgate.net/publication/382991572_nvblox_GPU-Accelerated_Incremental_Signed_Distance_Field_Mapping
17. RGBTSDF: An Efficient and Simple Method for Color Truncated Signed Distance Field (TSDF) Volume Fusion Based on RGB-D Images - MDPI, accessed August 5, 2025, https://www.mdpi.com/2072-4292/16/17/3188
18. Truncated Signed Distance Fields Applied To Robotics - DiVA portal, accessed August 5, 2025, https://www.diva-portal.org/smash/get/diva2:1136113/FULLTEXT01.pdf
19. CS 184: Final Project - Ryan Koh, accessed August 5, 2025, https://kaipinryankoh.github.io/3d_reconstruction_app/
20. Truncated Signed Distance Function | by Simsangcheol - Medium, accessed August 5, 2025, https://medium.com/@sim30217/truncated-signed-distance-function-f765a0f1d432
21. DFusion: Denoised TSDF Fusion of Multiple Depth Maps with ..., accessed August 5, 2025, https://www.mdpi.com/1424-8220/22/4/1631
22. NVIDIA-ISAAC-ROS/isaac_ros_cumotion: NVIDIA ... - GitHub, accessed August 5, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_cumotion
23. parameters.md - Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, accessed August 5, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/parameters.md
24. NVIDIA-Isaac-ROS-Nvblox/docs/topics-and-services.md at main - GitHub, accessed August 5, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/topics-and-services.md
25. NVBlox voxel artifacts appear in low-light environment - NVIDIA Developer Forums, accessed August 5, 2025, https://forums.developer.nvidia.com/t/nvblox-voxel-artifacts-appear-in-low-light-environment/319333
26. nvBLOX: Compute Dynamic Layer Only - Isaac ROS - NVIDIA Developer Forums, accessed August 5, 2025, https://forums.developer.nvidia.com/t/nvblox-compute-dynamic-layer-only/338613
27. Technical Details - isaac_ros_docs documentation, accessed August 5, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
28. Isaac ROS March update, vision based navigation - Open Robotics Discourse, accessed August 5, 2025, https://discourse.ros.org/t/isaac-ros-march-update-vision-based-navigation/24816
29. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, accessed August 5, 2025, https://arxiv.org/html/2410.21149v1
30. nvblox/README.md at public / nvidia-isaac/nvblox - GitHub, accessed August 5, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/README.md?plain=1
31. Performance - voxblox documentation - Read the Docs, accessed August 5, 2025, https://voxblox.readthedocs.io/en/latest/pages/Performance.html
32. Octomap comparison. / Issue #28 / ethz-asl/voxgraph - GitHub, accessed August 5, 2025, https://github.com/ethz-asl/voxgraph/issues/28
33. Fast Collision Mapping For Industrial Manipulators - DiVA portal, accessed August 5, 2025, http://www.diva-portal.org/smash/get/diva2:1897677/FULLTEXT01.pdf
34. Get Map From NVBLOX , Want to run Nav2 with this map and vslam ..., accessed August 5, 2025, https://forums.developer.nvidia.com/t/get-map-from-nvblox-want-to-run-nav2-with-this-map-and-vslam/312177
35. ROS2 Navigation - Isaac Sim Documentation - NVIDIA, accessed August 5, 2025, https://docs.isaacsim.omniverse.nvidia.com/latest/ros2_tutorials/tutorial_ros2_navigation.html
36. Isaac ROS VSLAM. NVBLOX usage with NAV2 - NVIDIA Developer Forums, accessed August 5, 2025, https://forums.developer.nvidia.com/t/isaac-ros-vslam-nvblox-usage-with-nav2/303717
37. Help Integrating NVBLOX with cuMotion for Real-time Obstacle Avoidance with Franka FR3 Arm - Isaac ROS - NVIDIA Developer Forums, accessed August 5, 2025, https://forums.developer.nvidia.com/t/help-integrating-nvblox-with-cumotion-for-real-time-obstacle-avoidance-with-franka-fr3-arm/341178
38. Using with Depth Camera - cuRobo, accessed August 5, 2025, https://curobo.org/get_started/2d_nvblox_demo.html
39. cuRobo and cuMotion - Isaac Sim Documentation - NVIDIA, accessed August 5, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/manipulators/manipulators_curobo.html
40. Isaac Manipulator Reference Architecture - isaac_ros_docs documentation, accessed August 5, 2025, https://nvidia-isaac-ros.github.io/reference_workflows/isaac_manipulator/reference_architecture.html
41. What's the realistic maximum size this can run on an Orin? / Issue #96 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, accessed August 5, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/96
42. Use nvblox in a non-flat environment / Issue #61 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox, accessed August 5, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/61
43. Running NVblox with more than 4 camera - Isaac Sim - NVIDIA Developer Forums, accessed August 5, 2025, https://forums.developer.nvidia.com/t/running-nvblox-with-more-than-4-camera/332296
44. Use Nvblox with Nav2 Rolling - Isaac ROS - NVIDIA Developer Forums, accessed August 5, 2025, https://forums.developer.nvidia.com/t/use-nvblox-with-nav2-rolling/271374


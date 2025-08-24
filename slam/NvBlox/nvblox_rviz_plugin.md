# nvblox_rviz_plugin
[NvBLox](./index.md)


현대 로보틱스 기술은 자율 이동 로봇(AMR)과 로봇 팔 조작(manipulation) 분야에서 실시간으로 주변 환경을 고밀도로 3차원 인식하는 능력을 핵심 요구사항으로 삼고 있다. 이러한 요구는 복잡하고 예측 불가능한 동적 환경에서 로봇이 안전하고 효율적으로 임무를 수행하기 위한 전제 조건이다. 과거에는 중앙 처리 장치(CPU)를 기반으로 한 희소(sparse) 맵핑 기법이 주를 이루었으나, 이는 로봇의 반응성과 안전성을 확보하는 데 명백한 계산적 한계를 드러냈다.1 포인트 클라우드나 특징점 기반의 맵은 환경의 전체적인 볼륨 정보를 표현하지 못해 경로 계획 시 충돌 가능성을 정확히 판단하기 어려웠기 때문이다.

이러한 계산적 제약을 극복하기 위한 돌파구로 NVIDIA의 CUDA(Compute Unified Device Architecture) 기술을 활용한 GPU(Graphics Processing Unit) 병렬 컴퓨팅이 부상하였다.4

`nvblox`는 이러한 패러다임 전환을 선도하는 라이브러리로서, GPU 하드웨어의 대규모 병렬 처리 능력을 최대한 활용하여 실시간으로 고해상도 체적 맵(volumetric map)을 생성하고 유지하도록 설계되었다.1

`nvblox`는 단순히 환경의 표면을 재구성하는 것을 넘어, 로봇의 경로 계획에 필수적인 거리 정보를 포함하는 맵을 제공함으로써 인식과 계획 사이의 간극을 효과적으로 메운다.

그러나 `nvblox`가 생성하는 데이터는 그 자체로 매우 복잡하고 다층적이다. 실시간으로 갱신되는 3D 메시, 정적 환경과 동적 객체를 분리한 다중 맵 레이어 등은 단순한 수치나 로그만으로는 그 상태를 직관적으로 파악하기 어렵다. 여기서 `nvblox_rviz_plugin`의 본질적인 역할이 정의된다. 이 플러그인은 ROS 2(Robot Operating System 2)의 표준 시각화 도구인 RViz2 내에서 `nvblox`의 복잡한 출력을 효율적으로 렌더링하는 특화된 교량 역할을 수행한다.6 이는 단순히 재구성된 3D 모델을 보여주는 것을 넘어, 시스템의 내부 동작을 투명하게 드러내고, 알고리즘의 정확성을 검증하며, 성능 병목을 식별하는 핵심적인 디버깅 및 분석 수단으로 기능한다. 본 보고서는 먼저 `nvblox`의 근간을 이루는 핵심 기술 원리를 심도 있게 분석하고, 이를 바탕으로 `nvblox_rviz_plugin`이 제공하는 시각화 기능의 세부 사항과 그 활용 방안을 체계적으로 고찰하고자 한다.


`nvblox`의 혁신은 단순히 기존 알고리즘을 GPU로 이식한 것이 아니라, 로봇의 경로 계획이라는 명확한 목적 아래 체적 맵핑 파이프라인 전체를 GPU 병렬 아키텍처에 맞춰 재설계한 데 있다. 이는 고해상도 표면 재구성과 실시간 충돌 비용 계산이라는 두 가지 상충될 수 있는 요구사항을 동시에, 그리고 압도적인 성능으로 만족시키는 결과를 낳았다.


TSDF는 3D 공간을 이산적인 복셀(voxel) 그리드로 나누고, 각 복셀의 중심점에서 가장 가까운 표면까지의 부호화된 거리(signed distance)를 저장하는 스칼라 필드(scalar field)다.5 이 거리 값은 복셀이 표면 외부에 있으면 양수, 내부에 있으면 음수, 표면에 정확히 위치하면 0의 값을 가진다. 

`nvblox`에서 이 거리는 계산 효율성을 위해 센서로부터의 광선 방향으로 측정된 투영 거리(projective distance)로 근사된다.9 또한, 'Truncated'라는 이름에서 알 수 있듯이, 이 거리 값은 표면 주변의 좁은 영역(truncation distance, 절단 거리) 내에서만 유효한 값을 가지며, 그 밖의 영역은 최대/최소값으로 고정된다. 이는 메모리 사용량을 줄이고, 센서의 측정 범위 밖이나 폐색(occlusion) 영역에서 발생하는 오류의 영향을 제한하는 효과를 가져온다.8 환경의 실제 표면은 이 SDF 스칼라 필드에서 값이 0이 되는 지점들의 집합, 즉 등위 집합(zero-level set)으로 암묵적으로(implicitly) 정의된다.10

새로운 깊이 측정값(depth map)이 입력되면, `nvblox`는 해당 측정 시점의 센서 자세(pose) 정보를 이용해 깊이 이미지의 각 픽셀을 전역 좌표계의 복셀 그리드로 역투영(back-project)한다. 그 후, 각 복셀에 대해 이전에 저장된 TSDF 값과 새로 계산된 TSDF 값을 가중 평균하여 점진적으로 갱신한다.11 이 통합(integration) 과정은 베이즈 필터와 유사하게 동작하여, 여러 시점에서 얻은 측정값을 융합함으로써 센서 노이즈를 효과적으로 상쇄시키고, 시간이 지남에 따라 더욱 정확하고 매끄러운 표면을 재구성하게 한다.12

`nvblox`의 핵심은 이 모든 복잡한 투영 및 통합 연산을 수많은 복셀에 대해 GPU에서 병렬로 처리하여 실시간 성능을 보장한다는 점이다.4


TSDF가 환경의 표면을 정밀하게 표현하는 데 중점을 둔다면, ESDF는 로봇의 자율 주행, 특히 경로 계획(path planning)에 직접적으로 활용되는 정보를 제공한다. ESDF는 3D 공간의 각 복셀에서 가장 가까운 장애물(점유된 공간)까지의 실제 유클리드 거리(Euclidean distance)를 저장하는 필드다.9 로봇의 경로 계획기는 이 ESDF 맵을 참조하여 특정 경로상의 각 지점이 장애물로부터 얼마나 안전하게 떨어져 있는지를 즉시 알 수 있으며, 이를 충돌 비용(collision cost)으로 변환하여 최적의 경로를 신속하게 계산할 수 있다.1 즉, ESDF는 '장애물로부터 얼마나 멀리 떨어져 있는가'라는, 로봇의 안전 운행에 직결되는 질문에 대한 답을 제공한다.

`nvblox`는 `Voxblox`와 같은 선행 연구에서 영감을 받아, 이미 구축된 TSDF 맵으로부터 ESDF를 효율적으로 파생시키는 알고리즘을 구현한다.1 먼저, TSDF 맵에서 SDF 값이 0에 가까운, 즉 표면 경계에 위치한 복셀들을 '사이트(sites)'로 식별한다. 이 '사이트'들은 거리 전파의 시작점이 된다. 그 후, GPU의 병렬 처리 능력을 활용하여 이 '사이트'들로부터 주변의 빈 공간(free space)으로 거리를 동시에 전파시키는 파면 전파(wavefront propagation)와 유사한 알고리즘을 수행한다.1 이 과정은 여러 단계에 걸쳐 진행되며, 각 복셀은 이웃 복셀로부터 거리 정보를 받아 자신의 ESDF 값을 갱신한다. 이 모든 과정이 GPU 상에서 대규모 병렬로 이루어지기 때문에, 전체 맵에 대한 ESDF 계산이 매우 빠른 시간 안에 완료될 수 있다.


`nvblox`의 가장 두드러진 특징은 C++과 CUDA를 기반으로 설계되어, TSDF 통합, ESDF 계산, 메시 생성 등 모든 핵심 연산 파이프라인이 GPU에서 병렬로 수행된다는 점이다.4 이는 CPU 기반으로 순차적 처리에 의존하는 `Voxblox`와 같은 기존의 방식들과 근본적인 아키텍처 차이를 만든다.2 이러한 아키텍처의 우수성은 정량적인 성능 벤치마크를 통해 명확히 입증된다.

2024년 ICRA(International Conference on Robotics and Automation)에서 발표된 `nvblox` 논문에 따르면, `nvblox`는 CPU 기반의 최신 기술인 `Voxblox`와 비교했을 때, 표면 재구성(TSDF 통합) 속도에서 최대 **177배**, 거리 필드(ESDF) 계산 속도에서 최대 **31배**의 놀라운 성능 향상을 달성했다.1 특히 주목할 점은, 맵의 해상도가 높아질수록 성능 격차가 기하급수적으로 벌어진다는 것이다. 

`nvblox`는 1cm의 매우 정밀한 복셀 크기로 맵을 구성하면서도 실시간 성능을 유지하는 반면, `Voxblox`는 동일한 작업을 수행하는 데 수십 배 이상의 시간이 소요된다. 한 벤치마크에서는 `nvblox`를 사용한 1cm 해상도의 맵핑이 `Voxblox`를 사용한 10cm 해상도의 맵핑보다 더 빠른 성능을 보이는 결과가 나타났다.18 이는 `nvblox`가 단순히 빠른 것이 아니라, 이전에는 실시간으로 불가능했던 수준의 정밀한 3D 환경 인식을 가능하게 했음을 의미한다.

아래 표는 `nvblox`와 `Voxblox`의 성능을 주요 지표에 따라 비교한 것이다.

| 지표 (Metric)                              | Voxblox (CPU)           | nvblox (GPU)                 | 성능 향상 (Speed-up) |
| ------------------------------------------ | ----------------------- | ---------------------------- | -------------------- |
| **TSDF 통합 시간 (TSDF Integration Time)** | 수백 ms ~ 수 초         | 수 ms                        | 최대 `$177 \times$`  |
| **ESDF 계산 시간 (ESDF Computation Time)** | 수십 ms ~ 수백 ms       | 수 ms                        | 최대 `$31 \times$`   |
| **플랫폼 (Platform)**                      | Multi-core CPU          | NVIDIA GPU (Jetson, Desktop) | -                    |
| **주요 언어 (Languages)**                  | C++                     | C++, CUDA                    | -                    |
| **설계 철학 (Design Philosophy)**          | CPU 기반 순차/병렬 처리 | GPU 기반 대규모 병렬 처리    | -                    |

주: 위 표의 수치는 ICRA 2024 논문 1 및 관련 벤치마크 자료 18를 기반으로 재구성되었음.

이러한 압도적인 성능 차이는 `nvblox`가 실시간 고해상도 맵핑이 필수적인 자율 주행 드론, 물류 로봇, 협동 로봇 등의 애플리케이션에서 왜 표준적인 선택지로 부상하고 있는지를 명확하게 설명하는 기술적 근거가 된다.


실제 환경은 정적인 구조물뿐만 아니라 사람, 다른 로봇 등 움직이는 객체들로 가득 차 있다. 이러한 동적 객체들을 정적 맵의 일부로 잘못 통합하면, 로봇의 경로 계획에 심각한 오류를 유발하는 '유령' 장애물이 생성될 수 있다. `nvblox`는 이러한 문제를 해결하기 위해 다중 레이어(multi-layer) 아키텍처를 도입했다. 환경을 정적(static), 사람(people), 그리고 일반 동적(dynamic) 객체를 위한 세 가지 독립적인 레이어로 분리하여 관리함으로써, 동적 요소가 정적 환경 맵을 오염시키는 것을 원천적으로 방지하고 각 요소의 특성에 맞는 차별화된 맵핑 전략을 적용한다.6

**사람 재구성 (People Reconstruction)** 모드에서는 Isaac ROS Image Segmentation과 같은 외부의 딥러닝(DNN) 모델을 활용한다.6 컬러 이미지에서 사람의 영역을 픽셀 단위로 분할(segmentation)한 마스크 이미지를 입력받아, 이 정보를 깊이 이미지에 투영한다. 이를 통해 깊이 측정값 중 사람에 해당하는 부분과 배경에 해당하는 부분을 분리할 수 있다. 배경 부분은 정적 TSDF 레이어에 통합되고, 사람에 해당하는 부분은 별도의 'people' 레이어에 점유 그리드(occupancy grid) 형태로 저장된다. 이 'people' 레이어에는 시간이 지남에 따라 점유 확률이 점차 감소하는 '소멸(decay)' 메커니즘이 적용된다. 덕분에 사람이 시야에서 사라지면 그 흔적이 맵에서 서서히 사라져, 로봇이 더 이상 존재하지 않는 장애물을 회피하려는 비효율적인 행동을 하지 않게 된다.10

**일반 동적 재구성 (General Dynamic Reconstruction)** 모드는 특정 클래스(예: 사람)에 국한되지 않고 다양한 종류의 동적 객체를 처리하기 위한 기법으로, "Dynablox" 논문의 아이디어를 기반으로 한다.10 이 방식은 DNN에 의존하는 대신, 높은 신뢰도를 가진 빈 공간(freespace) 정보를 별도의 레이어에 유지 관리한다. 만약 이전에 확실히 비어있다고 관측되었던 공간에 새로운 깊이 측정값이 나타나면, 이는 정적 환경의 일부가 아닌 동적 객체일 가능성이 높다고 판단한다. 이렇게 탐지된 객체들은 별도의 'dynamic' 점유 그리드 레이어에 통합된다. 이 접근법은 DNN 모델이 학습하지 않은 예측 불가능한 동적 객체(예: 갑자기 나타나는 무인 운반차)에 대해서도 강건하게 대응할 수 있는 능력을 제공한다.10


`nvblox`의 강력한 맵핑 능력은 ROS 2 생태계 내에서 다른 핵심 모듈들과 유기적으로 연동될 때 비로소 그 가치를 완전히 발휘한다. `isaac_ros_nvblox` 패키지는 `nvblox` C++ 라이브러리를 ROS 2 노드로 감싸고, 표준 메시 인터페이스를 통해 SLAM, 경로 계획 등 다른 시스템과 데이터를 교환하는 역할을 수행한다. 이 통합 아키텍처는 `nvblox`를 단순한 맵핑 라이브러리가 아닌, VSLAM(자세 추정), `nvblox`(맵핑), Nav2(경로 계획)가 긴밀하게 결합된 GPU 가속 '인식-계획(Perception-Planning)' 파이프라인의 핵심 허브로 기능하게 한다.


`isaac_ros_nvblox` 패키지의 중심에는 `nvblox_node`가 있다.6 이 노드는 로봇 시스템의 다양한 센서와 알고리즘으로부터 데이터를 입력받아 처리한 후, 그 결과를 시각화 및 경로 계획 모듈로 전달하는 데이터 흐름의 중심에 위치한다.

**입력 데이터:** `nvblox_node`의 정상적인 동작을 위해서는 다음과 같은 세 가지 종류의 데이터가 지속적으로 공급되어야 한다.

1. **센서 데이터:** 가장 핵심적인 입력은 깊이 센서(예: Intel RealSense, ZED)로부터 수신하는 깊이 이미지(`sensor_msgs/Image`)와 해당 카메라의 내부 파라미터 정보(`sensor_msgs/CameraInfo`)다. 선택적으로, 3D 재구성에 색상을 입히기 위한 컬러 이미지나, 깊이 카메라를 보완하기 위한 3D LiDAR 포인트 클라우드(`sensor_msgs/PointCloud2`)도 입력으로 사용할 수 있다.6
2. **자세 데이터 (Pose):** `nvblox`는 각 센서 데이터가 어느 위치와 방향에서 측정되었는지를 알아야 맵을 정확하게 구축할 수 있다. 이 자세 정보는 일반적으로 Isaac ROS Visual SLAM과 같은 외부 시각 SLAM(VSLAM) 노드로부터 `tf2` 변환 정보(`/tf`, `/tf_static`) 또는 `geometry_msgs/PoseStamped` 메시 형태로 공급받는다.6 VSLAM의 정확도와 안정성은 `nvblox` 맵 품질에 직접적인 영향을 미치는 매우 중요한 요소다.
3. **시맨틱 데이터:** 동적 객체 처리 모드를 사용할 경우, 사람 분할 마스크(`sensor_msgs/Image`)와 같은 시맨틱 정보를 추가로 입력받아 정적/동적 레이어를 분리하는 데 사용한다.6

**출력 데이터:** `nvblox_node`는 내부적으로 계산한 다양한 맵 레이어를 각각의 목적에 맞는 ROS 토픽으로 발행한다. 이 출력 데이터는 주로 RViz에서의 시각화와 Nav2에서의 경로 계획에 사용된다. 주요 출력으로는 3D 메시(`visualization_msgs/MarkerArray`), 2D/3D 거리 맵(`nvblox_msgs/DistanceMap`, `nav_msgs/OccupancyGrid`), 그리고 점유 복셀을 나타내는 포인트 클라우드(`sensor_msgs/PointCloud2`) 등이 있다.6

이러한 데이터 흐름은 하나의 완전한 파이프라인을 구성한다. 예를 들어, VSLAM의 성능 저하(예: 빠른 회전으로 인한 추적 실패)는 부정확한 자세 정보를 `nvblox`에 전달하게 되고, 이는 곧바로 맵의 품질 저하(예: 벽이 이중으로 생기는 현상)로 이어진다. 또한, ARPLab의 보고서에 따르면, NVIDIA Jetson Xavier NX와 같은 임베디드 플랫폼에서는 VSLAM과 `nvblox`를 포함한 전체 파이프라인의 계산 부하 및 ROS 2 통신 오버헤드가 시스템의 한계를 초과하여 전체가 멈추는 현상이 발생할 수 있다.21 이는 `nvblox`를 성공적으로 운용하기 위해서는 `nvblox` 노드 자체의 최적화를 넘어, VSLAM의 안정성, ROS 2의 통신 설정(DDS), 그리고 하드웨어 사양까지 고려하는 시스템 수준의 접근이 필수적임을 시사한다.


`nvblox_node`는 표준 ROS 2 인터페이스(토픽, 서비스)를 통해 외부와 상호작용한다. 개발자가 `nvblox`를 자신의 로봇 시스템에 통합하기 위해서는 이 API 명세를 정확히 이해해야 한다. 아래 표는 `isaac_ros_nvblox` 패키지가 제공하는 주요 ROS API를 정리한 것이다.

| 종류 (Type)   | 이름 (Name)                     | 메시지 타입 (Message Type)       | 설명 (Description)                                       |
| ------------- | ------------------------------- | -------------------------------- | -------------------------------------------------------- |
| **구독 토픽** | `/tf`, `/tf_static`             | `tf2_msgs/TFMessage`             | VSLAM 등으로부터 로봇의 자세(transform) 정보를 수신한다. |
|               | `/camera/depth/image`           | `sensor_msgs/Image`              | 깊이 카메라로부터 깊이 이미지를 수신한다.                |
|               | `/camera/depth/camera_info`     | `sensor_msgs/CameraInfo`         | 깊이 카메라의 캘리브레이션 정보를 수신한다.              |
|               | `/localization_pose`            | `geometry_msgs/PoseStamped`      | 로봇의 현재 위치 및 방향 정보를 수신한다.                |
|               | `/pointcloud`                   | `sensor_msgs/PointCloud2`        | 3D LiDAR 등으로부터 포인트 클라우드 데이터를 수신한다.   |
|               | `/mask`                         | `sensor_msgs/Image`              | 사람 분할 마스크 이미지를 수신한다 (동적 모드).          |
| **발행 토픽** | `/nvblox_node/static_mesh`      | `visualization_msgs/MarkerArray` | 정적 환경의 3D 메시를 발행한다 (RViz 시각화용).          |
|               | `/nvblox_node/people_mesh`      | `visualization_msgs/MarkerArray` | 분리된 사람의 3D 메시를 발행한다 (RViz 시각화용).        |
|               | `/nvblox_node/esdf_map_slice`   | `nav_msgs/OccupancyGrid`         | 2D ESDF 맵 슬라이스를 발행한다 (Nav2 코스트맵용).        |
|               | `/nvblox_node/static_occupancy` | `sensor_msgs/PointCloud2`        | 점유된 복셀의 중심점을 포인트 클라우드로 발행한다.       |
|               | `/nvblox_node/static_map_slice` | `nav_msgs/OccupancyGrid`         | 2D 정적 점유 그리드 슬라이스를 발행한다.                 |
| **서비스**    | `/nvblox_node/save_map`         | `nvblox_msgs/SaveMap`            | 현재 `nvblox` 맵을 파일로 저장하는 서비스를 제공한다.    |
|               | `/nvblox_node/load_map`         | `nvblox_msgs/LoadMap`            | 파일로부터 `nvblox` 맵을 불러오는 서비스를 제공한다.     |
|               | `/nvblox_node/clear_static_map` | `std_srvs/Empty`                 | 정적 맵 레이어를 초기화하는 서비스를 제공한다.           |

주: 위 표는 `isaac_ros_nvblox` 공식 문서 6 및 관련 자료 6를 바탕으로 핵심 API를 선별하여 정리하였음.

이 표는 `nvblox`를 시스템에 통합하려는 개발자에게 필수적인 청사진을 제공한다. 어떤 데이터를 입력으로 준비해야 하는지, 그리고 어떤 출력을 활용하여 시각화하거나 경로 계획에 연결할 수 있는지를 명확하게 보여준다.


`nvblox`의 가장 중요한 응용 중 하나는 ROS 2의 표준 내비게이션 프레임워크인 Nav2와의 통합이다. `isaac_ros_nvblox`는 `nvblox_nav2`라는 이름의 Nav2 Costmap2D 플러그인을 제공하여 이러한 연동을 원활하게 한다.6 이 플러그인은 Nav2의 로컬 코스트맵(local_costmap) 또는 글로벌 코스트맵(global_costmap)의 레이어로 동작하도록 설정할 수 있다.

전통적인 Nav2 설정에서는 3D 센서(LiDAR, 깊이 카메라)로부터 들어온 포인트 클라우드 데이터를 2D 점유 그리드로 변환하고, 이를 기반으로 장애물 레이어(ObstacleLayer)가 코스트맵을 갱신한다. 이 과정은 여러 단계의 데이터 변환을 거치며 CPU에서 수행되므로 지연 시간이 발생할 수 있다.

반면, `nvblox_nav2` 플러그인은 이러한 중간 과정을 완전히 생략한다. 이 플러그인은 `nvblox_node`가 GPU에서 직접 계산하여 발행하는 2D ESDF 맵 슬라이스(`/nvblox_node/esdf_map_slice`) 토픽을 직접 구독한다.6 이 토픽의 `nav_msgs/OccupancyGrid` 메시지는 각 셀에 점유 확률 대신 장애물까지의 거리가 인코딩되어 있다. 플러그인은 이 거리 정보를 Nav2가 이해할 수 있는 비용(cost) 값으로 변환하여 코스트맵을 직접 갱신한다. 이 방식은 `nvblox`가 GPU에서 이미 계산을 마친, 경로 계획에 최적화된 정보를 최소한의 지연으로 Nav2에 전달하므로 매우 효율적이다.6 그 결과, 로봇은 주변 환경의 변화, 특히 동적 장애물의 움직임에 대해 훨씬 더 신속하고 반응적으로 회피 기동을 수행할 수 있게 된다.


`nvblox`가 생성하는 풍부하고 동적인 3D 환경 정보는 그 자체로는 추상적인 데이터의 집합에 불과하다. `nvblox_rviz_plugin`은 이러한 추상적인 데이터 계층들-TSDF 기반의 표면, ESDF 기반의 계획 가능 공간, 동적 객체 레이어-을 하나의 통합된 시각적 공간에 투영함으로써, 개발자가 시스템의 내부 상태를 직관적으로 이해하고 문제를 진단할 수 있게 하는 '실시간 디버깅 콘솔'의 역할을 수행한다.


`nvblox`는 재구성된 3D 환경을 메시(mesh) 형태로 시각화 데이터를 발행한다. 만약 맵 전체를 단일 메시로 매 프레임마다 전송한다면, 이는 엄청난 데이터 대역폭을 요구하여 실시간성을 저해할 것이다. 이 문제를 해결하기 위해 `nvblox`는 더 효율적인 방식을 채택한다. 즉, 전체 맵을 한 번에 보내는 대신, 이전 프레임과 비교하여 변경이 발생한 복셀 블록(voxel block)에 대한 메시 업데이트 정보만을 스트림 형태로 전송한다.6

`nvblox_rviz_plugin`은 바로 이러한 증분 업데이트(incremental updates) 스트림을 효율적으로 수신하고 렌더링하도록 특별히 설계된 RViz의 커스텀 'Display' 플러그인이다.6 RViz의 표준 메시 디스플레이(Marker Display 등)는 이러한 스트리밍 방식의 업데이트를 처리하는 데 최적화되어 있지 않다. 

`nvblox_rviz_plugin`은 내부에 메시 블록들을 캐싱하고, 새로운 업데이트가 들어올 때마다 해당 부분만 교체하거나 추가함으로써, 최소한의 렌더링 부하로 실시간에 가까운 맵 시각화를 가능하게 한다. 또한, 이 플러그인은 3D 메시뿐만 아니라 2D 맵 슬라이스, 포인트 클라우드 등 `nvblox`가 발행하는 다양한 데이터 유형을 하나의 디스플레이 패널에서 통합적으로 관리하고 시각화하는 기능을 제공하여 사용 편의성을 높인다.


`nvblox_rviz_plugin`을 RViz에 추가하면, 사용자는 `nvblox`가 발행하는 다양한 토픽을 선택하여 시각화할 수 있다. 각 데이터 유형은 로봇의 인식 상태에 대한 서로 다른 측면의 정보를 제공한다.

3D 메시 (Meshes):

nvblox_node는 정적 환경, 사람, 일반 동적 객체에 대한 메시를 각각 별도의 토픽으로 발행한다. (/nvblox_node/static_mesh, /nvblox_node/people_mesh, /nvblox_node/dynamic_mesh).6

`nvblox_rviz_plugin`은 이 토픽들을 구독하여 3D 환경을 렌더링한다. 사용자는 RViz의 디스플레이 옵션에서 각 레이어(static, people, dynamic)를 개별적으로 켜고 끌 수 있다. 이를 통해, 예를 들어 정적 환경 맵 위에 움직이는 사람의 메시가 어떻게 분리되어 표현되는지를 명확하게 확인할 수 있다. 만약 `nvblox_node`에 컬러 이미지가 입력으로 제공된다면, 재구성된 메시에 실제 환경의 색상과 질감이 텍스처로 입혀져 훨씬 더 사실적이고 직관적인 시각화가 가능하다.6

2D 맵 슬라이스 (Map Slices):

nvblox의 가장 중요한 출력 중 하나인 ESDF 맵은 3차원 데이터이지만, 평면 위를 움직이는 AMR의 경우 특정 높이의 2D 슬라이스만으로도 충분한 정보를 얻을 수 있다. nvblox_node는 /nvblox_node/esdf_map_slice와 같은 토픽을 통해 nav_msgs/OccupancyGrid 메시지 형태로 이 2D 맵 슬라이스를 발행한다. nvblox_rviz_plugin은 이 2D 맵을 3D 뷰 공간 내에 사용자가 지정한 특정 높이(Z축)의 평면으로 오버레이하여 보여준다.26 각 셀의 색상은 해당 지점의 ESDF 값(장애물까지의 거리)을 나타내도록 설정할 수 있다. 예를 들어, 장애물에 가까울수록 붉은색, 멀수록 푸른색으로 표현할 수 있다. 이를 통해 개발자는 3D 구조물(메시)과 Nav2가 경로 계획에 실제로 사용할 2D 코스트맵 간의 관계를 한눈에 파악하고, ESDF 계산이 올바르게 이루어지고 있는지를 검증할 수 있다.

포인트 클라우드 (Point Clouds):

메시가 생성되기 전의 더 원시적인 맵 상태를 확인하고 싶을 때, 점유된 복셀의 중심점을 나타내는 포인트 클라우드를 시각화할 수 있다. /nvblox_node/static_occupancy (sensor_msgs/PointCloud2) 토픽이 이러한 정보를 제공한다.21 이는 메시 생성 알고리즘(marching cubes)에 문제가 있거나, 매우 희소한 환경에서 맵이 어떻게 구성되는지를 디버깅하는 데 유용하다.


`nvblox_rviz_plugin`에 대한 공식 문서는 상대적으로 부족하지만 4, 플러그인이 처리하는 데이터 유형과 일반적인 RViz 플러그인의 구조를 바탕으로 다음과 같은 핵심 설정 옵션들을 추론할 수 있다.

**주요 설정 옵션 (추론 기반):**

- **Topic Selection:** 시각화할 각 데이터(static mesh, people mesh, esdf slice 등)에 해당하는 ROS 토픽 이름을 사용자가 직접 지정하는 기능.
- **Layer Visibility:** 각 맵 레이어(static, people, dynamic)의 표시 여부를 개별적으로 제어하는 체크박스.
- **Coloring Scheme:** 메시의 색상을 결정하는 방식. 예를 들어, 고정된 단일 색상, 복셀의 높이(Z값)에 따른 색상 그라데이션, 또는 발행된 텍스처 정보 사용 등을 선택할 수 있다.
- **Slice Height & Alpha:** 2D 맵 슬라이스를 3D 뷰에 표시할 Z축 높이를 조절하는 슬라이더와 투명도(alpha)를 조절하는 기능.

이러한 시각화 기능은 다양한 실제 개발 및 디버깅 시나리오에서 강력한 도구로 활용된다.

**활용 시나리오:**

1. **SLAM 성능 검증:** 로봇을 움직이면서 VSLAM의 성능을 실시간으로 검증할 수 있다. 만약 VSLAM에서 드리프트가 발생하여 자세 추정값이 실제와 어긋나기 시작하면, `nvblox_rviz_plugin`이 렌더링하는 맵에서 이전에 생성된 벽과 현재 생성되는 벽이 서로 겹치거나 어긋나는 현상이 시각적으로 즉시 나타난다.
2. **동적 장애물 회피 디버깅:** 복잡한 환경에서 로봇이 사람이나 다른 동적 장애물을 올바르게 인식하고 회피하는지 검증한다. `people_mesh` 또는 `dynamic_mesh` 레이어를 활성화하여 `nvblox`가 동적 객체를 정확히 분리하고 있는지 확인한다. 동시에, ESDF 슬라이스를 오버레이하여 Nav2의 경로 계획기가 이 동적 장애물 주변에 높은 비용 영역을 형성하고 안전하게 우회하는 경로를 생성하는지를 함께 검증할 수 있다.
3. **센서 데이터 품질 분석:** 깊이 센서의 측정 오류나 특정 재질(예: 검은색, 반사체)에 대한 노이즈가 맵에 어떤 영향을 미치는지 실시간으로 모니터링한다. RViz 화면에 나타나는 불필요한 점이나 면(아티팩트), 또는 존재하지 않는 '유령' 장애물은 센서 데이터의 품질 문제를 직접적으로 시사한다.22
4. **파라미터 튜닝:** `nvblox`는 `voxel_size`(맵의 해상도), `map_clearing_radius_m`(로봇 주변에서 맵을 지우는 반경) 등 성능과 맵 품질에 큰 영향을 미치는 여러 파라미터를 가지고 있다.22 개발자는 이러한 파라미터 값을 변경하면서 RViz 화면에 렌더링되는 맵의 정밀도, 노이즈 수준, 그리고 실시간 업데이트 속도가 어떻게 변하는지를 즉각적으로 확인하며 최적의 값을 찾아낼 수 있다.

이처럼 `nvblox_rviz_plugin`은 '인식 결과(TSDF mesh)'와 '계획 입력(ESDF slice)'을 동일한 시각적 공간에 통합하여 보여줌으로써, 인식과 계획이라는 두 시스템 사이의 정보 변환 과정에서 발생할 수 있는 문제를 명확히 드러낸다. 이는 개발자가 시스템 전체의 동작을 종합적으로 이해하고 디버깅할 수 있도록 돕는 이 플러그인의 핵심적인 가치다.


`nvblox`는 GPU의 대규모 병렬 처리 능력을 활용하여 로봇의 실시간 3D 환경 인식 능력을 이전에는 불가능했던 수준으로 끌어올렸다. 특히 고해상도의 정밀한 맵을 실시간으로 구축하고, 사람과 같은 동적 객체가 존재하는 복잡한 환경에 강건하게 대응하는 능력은 자율 이동 로봇 기술의 발전에 크게 기여했다. 그러나 이러한 강력한 성능과 복잡한 내부 데이터 구조는 개발자가 시스템을 이해하고 디버깅하는 데 새로운 어려움을 야기했다.

`nvblox_rviz_plugin`은 바로 이 지점에서 필수적인 역할을 수행한다. 이 플러그인은 단순한 3D 뷰어를 넘어, `nvblox`의 다층적인 내부 상태를 투명하게 시각화하는 강력한 분석 및 디버깅 도구다. TSDF로 재구성된 정적 환경, 분리된 동적 객체, 그리고 Nav2가 경로 계획에 사용할 ESDF 거리 필드를 하나의 통합된 뷰에서 보여줌으로써, 개발자는 복잡한 인식-계획 파이프라인 전체의 동작을 직관적으로 파악하고 문제의 원인을 신속하게 진단할 수 있다. 결과적으로 `nvblox_rviz_plugin`은 `nvblox`의 잠재력을 개발자가 실제로 최대한 활용할 수 있도록 만드는 핵심적인 인터페이스이며, 전체 로봇 시스템의 개발 효율성과 안정성을 극대화하는 데 기여한다.

향후 `nvblox`와 그 시각화 도구는 다음과 같은 방향으로 더욱 발전할 것으로 전망된다.

- **시맨틱 정보 시각화 강화:** 현재는 '사람'과 같은 제한된 클래스만 분리하여 시각화하지만, 향후 실시간 시맨틱 분할(semantic segmentation) 기술과 결합하여 '의자', '테이블', '도로' 등 다양한 객체와 환경 요소를 인식하고, 이를 각기 다른 색상이나 형태로 RViz에 시각화하는 기능이 추가될 수 있다. 이는 로봇이 환경을 기하학적으로만 이해하는 것을 넘어, 의미론적으로 이해하고 더 지능적인 작업을 수행하는 데 기틀을 마련할 것이다.
- **인터랙티브 기능 추가:** 현재의 플러그인은 수동적인 시각화에 머물러 있지만, 미래에는 사용자와 상호작용하는 기능이 추가될 수 있다. 예를 들어, 개발자가 RViz 화면에서 마우스 클릭으로 맵의 특정 영역을 강제로 지우거나, 가상의 장애물을 추가하여 경로 계획기가 이에 어떻게 반응하는지를 즉시 테스트하는 등의 인터랙티브 디버깅 기능이 개발될 수 있다.
- **성능 프로파일링 시각화:** `nvblox` 시스템의 성능 최적화는 매우 중요한 과제다. 향후 `nvblox_rviz_plugin` 내에 `nvblox_node`의 각 처리 단계(예: TSDF 통합, ESDF 계산, 메시 생성)에 소요되는 시간을 실시간 그래프나 색상 코드로 표시하는 기능이 추가될 수 있다. 이를 통해 개발자는 어느 부분에서 계산 병목이 발생하는지를 시각적으로 즉시 식별하고, 시스템 최적화 작업을 훨씬 더 효율적으로 수행할 수 있을 것이다.


1. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2311.00626v2
2. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/abs/1611.03631
3. Intelligent Robot Design and Implementation, 8월 17, 2025에 액세스, https://wp2023.cs.hku.hk/fyp23005/wp-content/uploads/sites/6/final_presentation_slides_fyp23005.pdf
4. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, 8월 17, 2025에 액세스, https://github.com/nvidia-isaac/nvblox
5. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 17, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
6. Isaac ROS Nvblox - isaac_ros_docs documentation, 8월 17, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
7. ethz-asl/nvblox_ros1: ROS1 wrappers for GPU-acceleration volumetric mapping with nvblox., 8월 17, 2025에 액세스, https://github.com/ethz-asl/nvblox_ros1
8. Truncated Signed Distance Function | by Simsangcheol - Medium, 8월 17, 2025에 액세스, https://medium.com/@sim30217/truncated-signed-distance-function-f765a0f1d432
9. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Helen Oleynikova, 8월 17, 2025에 액세스, https://helenol.github.io/publications/rss_2016_workshop.pdf
10. Technical Details - isaac_ros_docs documentation, 8월 17, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
11. Truncated Signed Distance Function Volume Integration Based on Voxel-Level Optimization for 3D Reconstruction - IS&T | Library, 8월 17, 2025에 액세스, https://library.imaging.org/admin/apis/public/api/ist/website/downloadArticle/ei/28/21/art00006
12. TSDF Integration - Open3D 0.14.1 documentation, 8월 17, 2025에 액세스, https://www.open3d.org/docs/0.14.1/tutorial/t_reconstruction_system/integration.html
13. Truncated Signed Distance Fields Applied To Robotics - DiVA portal, 8월 17, 2025에 액세스, https://www.diva-portal.org/smash/get/diva2:1136113/FULLTEXT01.pdf
14. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/html/2410.21149v1
15. Performance - voxblox documentation - Read the Docs, 8월 17, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/pages/Performance.html
16. [2311.00626] nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 17, 2025에 액세스, https://arxiv.org/abs/2311.00626
17. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - Semantic search for arXiv papers with AI, 8월 17, 2025에 액세스, https://axi.lims.ac.uk/paper/2311.00626
18. nvblox: GPU-Accelerated Incremental Signed Distance Field, 8월 17, 2025에 액세스, https://www.alphaxiv.org/overview/2311.00626v2
19. nvblox/docs/pages/technical.md at public - GitHub, 8월 17, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/blob/public/docs/pages/technical.md
20. AMR navigation using Isaac ROS VSLAM and Nvblox with Intel Realsense camera, 8월 17, 2025에 액세스, https://www.einfochips.com/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
21. Repo for the modified NvBlox package for mapping in ROS 2. - GitHub, 8월 17, 2025에 액세스, https://github.com/arplaboratory/nvblox
22. Nvidia Isaac ROS Nvblox, 8월 17, 2025에 액세스, https://forums.developer.nvidia.com/t/nvidia-isaac-ros-nvblox/311053
23. Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, 8월 17, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox
24. Nvblox Nav2 Implementation - Isaac ROS - NVIDIA Developer Forums, 8월 17, 2025에 액세스, https://forums.developer.nvidia.com/t/nvblox-nav2-implementation/272598
25. Isaac ROS (Robot Operating System) - NVIDIA Developer, 8월 17, 2025에 액세스, https://developer.nvidia.com/isaac/ros
26. isaac_ros_nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 17, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/isaac_ros_nvblox/index.html
27. For mapping and navigation, what approach should I take - Isaac ROS, 8월 17, 2025에 액세스, https://forums.developer.nvidia.com/t/for-mapping-and-navigation-what-approach-should-i-take/282915
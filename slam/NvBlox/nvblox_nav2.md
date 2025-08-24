[NvBLox](./index.md)
# nvblox_nav2에 대한 심층 고찰



자율 이동 로봇(AMR)의 핵심 기술인 내비게이션은 지난 수십 년간 2D LiDAR(Light Detection and Ranging) 센서에 크게 의존해왔다. 2D LiDAR는 수평면상의 장애물 정보를 정확하고 빠르게 제공하여 2차원 평면에서의 경로 계획과 장애물 회피에 효과적이었다. 그러나 현실 세계는 3차원이며, 테이블 모서리, 선반, 계단, 경사로와 같이 2D 평면 스캔만으로는 감지하기 어려운 복잡한 구조물로 가득 차 있다. 이러한 3차원적 장애물은 2D LiDAR 기반 로봇에게 심각한 항해 실패나 충돌의 원인이 될 수 있다. 이와 같은 한계를 극복하기 위해 깊이 카메라(Depth Camera)나 3D LiDAR와 같은 센서를 활용하여 환경을 3차원으로 인식하고, 이를 기반으로 내비게이션을 수행하려는 연구가 활발히 진행되어 왔다.1

3차원 환경 인식은 데이터의 양과 처리의 복잡성 때문에 막대한 계산 자원을 요구한다. 전통적인 CPU 기반 시스템으로는 실시간으로 고밀도의 3D 맵을 생성하고 경로를 계획하는 데 명백한 한계가 존재했다.2 바로 이 지점에서 GPU(Graphics Processing Unit) 기술의 발전이 로봇 공학, 특히 인식(perception) 및 맵핑(mapping) 분야에 혁신적인 변화를 가져왔다. 대규모 병렬 처리에 특화된 GPU는 방대한 양의 3D 센서 데이터를 실시간으로 처리하여, 과거에는 불가능했던 복잡한 환경에서의 강건하고 안전한 자율 주행을 가능하게 하는 핵심 동력이 되었다.3


이러한 기술적 흐름의 중심에 NVIDIA가 있다. NVIDIA는 자사의 강력한 GPU 하드웨어와 CUDA 병렬 컴퓨팅 플랫폼을 기반으로 로봇 개발을 가속화하기 위한 포괄적인 소프트웨어 개발 키트인 Isaac ROS(Robot Operating System)를 구축했다.5 Isaac ROS는 고성능, 저지연의 ROS 2 패키지 및 AI 모델의 집합체로서, 로봇 개발자들이 복잡한 AI 기반 인식 알고리즘을 보다 쉽게 자신의 로봇에 통합할 수 있도록 지원한다.6

Isaac ROS 생태계 내에서 `nvblox`는 실시간 3D 장면 재구성 및 동적 장애물 회피를 위한 핵심 구성요소, 즉 'GEM(Isaac ROS Hardware-Accelerated Package)'으로서 중요한 위치를 차지한다.5

`nvblox`는 깊이 센서로부터 입력받은 데이터를 GPU 가속을 통해 실시간으로 3D 맵으로 재구성하고, 이를 로봇 내비게이션 스택인 Nav2와 연동하여 동적인 환경에서도 안전하게 경로를 계획하고 주행할 수 있도록 지원한다.


`nvblox`와 이를 Nav2에 통합하는 `nvblox_nav2` 패키지에 대한 정보는 공식 문서, 학술 논문, 개발자 포럼, 소스 코드 등 다양한 곳에 흩어져 있다. 이로 인해 개발자나 연구자가 전체 시스템을 체계적으로 이해하고 실제 프로젝트에 적용하는 데 어려움을 겪는 경우가 많다.

따라서 본 보고서는 이러한 단편적인 정보들을 종합하고, 기술적 원리부터 실제 적용까지의 전 과정을 심층적으로 분석하여 하나의 통일되고 포괄적인 기술 문서를 제공하는 것을 목표로 한다. 본 보고서를 통해 독자는 `nvblox_nav2`의 아키텍처, 핵심 알고리즘인 TSDF(Truncated Signed Distance Field)와 ESDF(Euclidean Signed Distance Field)의 원리, 시스템 구축 방법, 하드웨어별 성능 특성, 그리고 OctoMap과 같은 다른 기술과의 비교 분석을 통해 명확하고 실용적인 이해를 얻게 될 것이다.



`nvblox`는 근본적으로 로봇 애플리케이션, 특히 경로 계획을 위해 설계된 실시간 3D 재구성 및 맵핑 라이브러리이다.3 이 라이브러리의 핵심 기능은 깊이 카메라나 3D LiDAR로부터 들어오는 센서 데이터 스트림과 로봇의 위치 추정값(pose)을 입력받아, 주변 환경을 복셀(voxel) 기반의 3D 맵으로 구축하는 것이다.8

`nvblox`는 단순히 3D 모델을 만드는 것을 넘어, 두 가지 중요한 결과물을 실시간으로 출력한다. 첫째는 RViz와 같은 시각화 도구에서 확인할 수 있는 3D 메시(mesh)이고, 둘째는 로봇의 경로 계획기가 장애물 회피에 직접 사용할 수 있는 2D 코스트맵(cost map)이다.10

`nvblox`의 가장 큰 특징은 이 모든 복잡한 연산 과정을 NVIDIA GPU 상에서 CUDA를 활용하여 병렬로 처리한다는 점이다. 이를 통해 고성능 데스크톱 GPU뿐만 아니라 NVIDIA Jetson과 같은 임베디드 시스템에서도 실시간 성능을 보장하며, 이는 자율 이동 로봇에 탑재되어 현장에서 동작해야 하는 실제적인 요구사항을 충족시킨다.2


`nvblox`의 아키텍처는 의도적으로 모듈화되어 있어 유연성과 확장성을 극대화한다. 시스템은 크게 두 부분으로 구성된다.4

1. **`nvblox` 코어 라이브러리:** 순수한 C++로 작성된 프레임워크 독립적인 라이브러리이다. 여기에는 TSDF/ESDF 맵핑, 메시 생성 등 핵심 알고리즘이 모두 구현되어 있다. ROS에 대한 의존성이 없기 때문에, 개발자는 이 코어 라이브러리만 가져와 ROS가 아닌 다른 로봇 프레임워크나 독자적인 시스템에 통합할 수 있다.3
2. **`isaac_ros_nvblox` ROS 2 래퍼:** `nvblox` 코어 라이브러리를 ROS 2 환경에서 쉽게 사용할 수 있도록 감싼 메타패키지이다. 이 패키지는 ROS의 표준 통신 방식인 토픽(topic)과 서비스(service)를 통해 `nvblox`의 기능을 노출시킨다. `isaac_ros_nvblox`는 다시 여러 개의 하위 패키지로 나뉜다 11:
   - `nvblox_ros`: 코어 라이브러리를 ROS 2 노드(`nvblox_node`, `nvblox_human_node`)로 만드는 핵심 래퍼.
   - `nvblox_nav2`: `nvblox`가 생성한 맵을 Nav2 내비게이션 스택의 코스트맵 레이어로 통합하는 플러그인.
   - `nvblox_msgs`: `DistanceMapSlice`, `Mesh` 등 `nvblox`에서 사용하는 커스텀 ROS 메시지를 정의.
   - `nvblox_rviz_plugin`: RViz에서 `nvblox`의 메시와 맵 슬라이스를 효율적으로 시각화하기 위한 플러그인.

이러한 다층적 구조는 각 구성요소의 역할을 명확히 분리하여 유연성을 확보하려는 설계 철학을 반영한다. 사용자는 필요에 따라 전체 스택을 사용하거나, 특정 부분만 선택하여 자신의 시스템에 맞게 수정 및 확장할 수 있다.


`nvblox`가 동작하는 전체 데이터 처리 파이프라인은 다음과 같은 단계로 이루어진다.

- **입력 (Input):**

  - **센서 데이터:** `nvblox`는 주로 깊이 카메라로부터 `sensor_msgs/Image` 타입의 깊이 이미지와 `sensor_msgs/CameraInfo` 타입의 카메라 내부 파라미터 정보를 입력으로 받는다. 선택적으로, 재구성된 메시에 색상을 입히기 위해 컬러 이미지(`sensor_msgs/Image`)를 함께 사용할 수 있다. 3D LiDAR의 경우 `sensor_msgs/PointCloud2` 데이터를 입력받아 내부적으로 깊이 이미지 형태로 변환하여 사용한다.11

  - **포즈 (Pose) 데이터:** 센서 데이터가 어느 위치와 방향에서 관측되었는지를 알려주는 포즈 정보는 필수적이다. `nvblox`는 이 정보를 두 가지 방식으로 받을 수 있다. 첫 번째는 ROS의 표준 좌표 변환 시스템인 `tf2`를 통해 실시간으로 로봇의 `odom` 또는 `map` 프레임과 카메라 프레임 간의 변환을 조회하는 것이다 (`use_tf_transforms: true`). 두 번째는 외부 노드로부터 `geometry_msgs/TransformStamped` 또는 `geometry_msgs/PoseStamped` 메시지를 직접 토픽으로 구독하는 방식이다.14 일반적으로 이 포즈 정보는 

    `isaac_ros_visual_slam`과 같은 별도의 SLAM(Simultaneous Localization and Mapping) 시스템이 실시간으로 추정하여 제공한다.4

- **처리 (Processing):**

  - `nvblox_node`는 입력된 깊이 이미지와 포즈 정보를 GPU로 전송한다.
  - GPU 상에서 각 깊이 픽셀은 3D 공간상의 점으로 역투영(back-projected)된다.
  - 이 3D 점들을 기반으로 TSDF 융합 알고리즘을 통해 3D 복셀 그리드 맵을 점진적으로 업데이트하고 재구성한다.11 이 과정에서 여러 프레임의 정보가 누적되면서 센서 노이즈가 완화되고 정밀한 표면이 형성된다.

- **출력 (Output):**

  - **Nav2용 코스트맵:** `nvblox`는 경로 계획을 위해 재구성된 3D 맵을 2D 코스트맵으로 변환한다. 구체적으로는 3D ESDF 맵을 생성한 뒤, 로봇의 현재 높이를 기준으로 얇게 잘라낸 2D 슬라이스(slice)를 만든다. 이 2D ESDF 슬라이스는 `nvblox_msgs/DistanceMapSlice`라는 커스텀 메시지 형태로 발행되며, `nvblox_nav2` 플러그인이 이를 구독하여 Nav2의 로컬 코스트맵으로 주입한다.11

  - **3D 메시:** 3D 맵의 표면을 삼각형 메시 형태로 추출하여 `nvblox_msgs/Mesh` 타입으로 발행한다. 이는 RViz와 같은 시각화 도구에서 로봇이 환경을 어떻게 인식하고 있는지를 직관적으로 확인하는 데 사용된다.11

    `nvblox`는 전체 메시를 한 번에 보내는 대신, 변경된 부분만 스트리밍하여 실시간 시각화가 가능하도록 최적화되어 있다.11


`nvblox`를 이용한 일반적인 내비게이션 시스템은 여러 ROS 노드들이 유기적으로 연동하여 구성된다.

- **카메라 노드 (예: `realsense-ros`):** 물리적인 깊이 카메라(예: Intel RealSense)로부터 이미지 데이터를 받아 ROS 토픽으로 발행하는 역할을 한다.4
- **SLAM 노드 (예: `isaac_ros_visual_slam`):** 카메라 노드가 발행한 스테레오 또는 RGB-D 이미지와 IMU 데이터를 입력받아 로봇의 실시간 위치와 자세(pose)를 추정하고, 이를 `tf2` 변환 정보로 발행한다.4
- **`nvblox_node` / `nvblox_human_node`:** `nvblox`의 핵심 처리 노드이다. SLAM 노드로부터 포즈 정보를, 카메라 노드로부터 깊이/컬러 이미지를 구독하여 3D 맵 재구성과 2D 코스트맵 슬라이스 생성을 수행한다.13 동적인 사람을 특별히 처리해야 할 경우, `unet`과 `PeopleSemSegNet` DNN 모델을 사용하여 사람 영역을 분리하고 별도의 동적 레이어로 관리하는 `nvblox_human_node`를 사용한다.11
- **`nvblox_nav2` Costmap Plugin:** 이 구성요소는 독립적인 노드가 아니라, Nav2의 로컬 코스트맵 서버에 로드되는 플러그인 라이브러리이다. `nvblox_node`가 발행하는 `~/map_slice` 토픽을 구독하여 그 정보를 Nav2의 내부 코스트맵 데이터 구조에 실시간으로 업데이트하는 '어댑터(adapter)' 역할을 수행한다.15
- **Nav2 스택:** `nvblox_nav2` 플러그인을 통해 업데이트된 로컬 코스트맵을 참조하여, 전역 경로를 따라가면서 실시간으로 지역 경로를 계획하고 로봇의 모터에 속도 명령(`cmd_vel`)을 내린다.4
- **RViz:** 전체 시스템의 동작을 시각화하고 디버깅하는 데 사용된다. 카메라 이미지, 로봇 모델, TF 트리, `nvblox`가 생성한 메시와 코스트맵 슬라이스, Nav2가 계획한 경로 등을 한 화면에서 확인할 수 있다.13

이처럼 `nvblox_nav2`는 단일 패키지가 아닌, 센서 드라이버, SLAM, 맵핑, 경로 계획 등 여러 모듈이 긴밀하게 연결된 복잡한 시스템의 일부로 동작한다. 각 구성 요소 간의 데이터 흐름과 의존 관계를 명확히 이해하는 것이 시스템 구축과 문제 해결의 핵심이다.

**Table 1: Nvblox 주요 ROS 노드 및 토픽/서비스**

| 구분 (Category) | 이름 (Name)            | 메시지/서비스 타입 (Type)      | 설명 (Description)                                           | 출처 (Source) |
| --------------- | ---------------------- | ------------------------------ | ------------------------------------------------------------ | ------------- |
| **구독 토픽**   | `depth/image`          | `sensor_msgs/Image`            | 통합할 깊이 이미지. 픽셀 값은 미터 단위의 부동소수점 또는 밀리미터 단위의 uint16 형식 지원. | 14            |
|                 | `depth/camera_info`    | `sensor_msgs/CameraInfo`       | 깊이 카메라의 내부 파라미터. `depth/image`와 쌍으로 필요.    | 14            |
|                 | `color/image`          | `sensor_msgs/Image`            | (선택) 메쉬에 색상을 입히기 위한 컬러 이미지.                | 14            |
|                 | `color/camera_info`    | `sensor_msgs/CameraInfo`       | (선택) 컬러 카메라의 내부 파라미터.                          | 14            |
|                 | `pointcloud`           | `sensor_msgs/PointCloud2`      | 3D LiDAR 포인트 클라우드 입력. 내부적으로 깊이 이미지로 변환됨. | 14            |
|                 | `pose`                 | `geometry_msgs/PoseStamped`    | 로봇의 오도메트리 정보. `use_tf_transforms`가 false일 때 사용. | 14            |
| **발행 토픽**   | `~/map_slice`          | `nvblox_msgs/DistanceMapSlice` | 정적 ESDF의 2D 슬라이스. `nvblox_nav2` 플러그인이 Nav2 코스트맵 생성을 위해 소비. | 15            |
|                 | `~/human_map_slice`    | `nvblox_msgs/DistanceMapSlice` | (`nvblox_human_node` 전용) 사람 ESDF의 2D 슬라이스.          | 15            |
|                 | `~/combined_map_slice` | `nvblox_msgs/DistanceMapSlice` | (`nvblox_human_node` 전용) 정적 맵과 사람 맵을 결합한 2D 슬라이스. | 15            |
|                 | `~/mesh`               | `nvblox_msgs/Mesh`             | 시각화를 위한 3D 메시. 실시간으로 업데이트되는 부분만 스트리밍. | 11            |
|                 | `~/static_occupancy`   | `sensor_msgs/PointCloud2`      | 정적 점유 격자 맵을 포인트 클라우드 형태로 발행.             | 18            |
| **제공 서비스** | `~/save_ply`           | `nvblox_msgs/srv/FilePath`     | 현재 재구성된 메시를 PLY 파일 형식으로 저장.                 | 15            |
|                 | `~/save_map`           | `nvblox_msgs/srv/FilePath`     | `nvblox`의 내부 맵 레이어(TSDF, ESDF 등)를 파일로 직렬화하여 저장. | 15            |
|                 | `~/load_map`           | `nvblox_msgs/srv/FilePath`     | 저장된 맵 파일을 불러와 현재 맵 상태를 복원.                 | 15            |


`nvblox`의 성능과 기능은 두 가지 핵심적인 3D 공간 표현 방식, 즉 Truncated Signed Distance Field (TSDF)와 Euclidean Signed Distance Field (ESDF)에 기반한다. 이 두 알고리즘의 원리를 이해하는 것은 `nvblox`를 깊이 있게 파악하는 데 필수적이다.


SDF는 특정 표면을 기준으로 공간상의 모든 점의 위치 관계를 표현하는 수학적 함수이다. 공간상의 한 점 $x$에서 어떤 집합 $\Omega$(예: 3D 객체)의 경계 $\partial\Omega$까지의 부호 있는 최단 거리를 값으로 갖는다.19

일반적으로 사용되는 관례에 따르면, 점 $x$가 객체 내부($\Omega$)에 있으면 양수, 외부에 있으면 음수, 그리고 경계면에 정확히 위치하면 0의 값을 가진다.19 (반대의 부호 관례를 사용하기도 한다.) 이를 수학적으로 표현하면 다음과 같다.
$$
f(x) = \begin{cases} d(x, \partial\Omega) & \text{if } x \in \Omega \\ 0 & \text{if } x \in \partial\Omega \\ -d(x, \partial\Omega) & \text{if } x \notin \Omega \end{cases}
$$
여기서 $d(x, \partial\Omega)$는 점 $x$에서 경계 $\partial\Omega$까지의 최단 유클리드 거리를 의미한다. SDF는 표면의 위치뿐만 아니라 표면으로부터의 거리와 내/외부 정보를 모두 포함하므로, 단순한 점유/비점유 정보만 갖는 점유 격자(occupancy grid)보다 훨씬 풍부한 정보를 제공한다.


TSDF는 SDF의 개념을 실제 센서 데이터 기반의 3D 재구성에 적용하기 위해 실용적으로 변형한 것이다.

- **개념:** 실제 로봇 응용에서는 전체 공간에 대한 거리 값을 모두 계산하고 저장하는 것이 비효율적이다. TSDF는 표면 근처의 좁은 영역에 대해서만 SDF 값을 정확히 계산하고, 이 영역을 벗어나는 먼 곳의 값은 특정 값으로 '절단(truncate)'하여 처리한다. 이 절단 거리(truncation distance, $\tau$) 내의 복셀들만 유효한 거리 값을 가지며, 그 밖의 영역은 보통 $+1$ (표면 앞) 또는 $-1$ (표면 뒤)로 정규화된 값으로 설정된다.20 이 방식은 계산량과 메모리 사용량을 크게 줄여준다.

- **수학적 표현 및 점진적 융합 (Incremental Fusion):** `nvblox`는 깊이 카메라로부터 연속적으로 들어오는 깊이 이미지들을 융합하여 하나의 일관된 3D 모델을 만든다. 이 과정은 가중 평균(weighted average)을 통해 점진적으로 이루어진다.22

  1. 새로운 깊이 이미지 프레임 $i$가 들어오면, 각 픽셀은 카메라의 포즈와 내부 파라미터를 이용해 3D 공간상의 복셀 그리드로 투영된다.
  2. 각 복셀 $\mathbf{x}$에 대해, 센서 광선(ray)을 따라 측정된 표면까지의 부호 있는 거리인 $v_i(\mathbf{x})$와 이 측정의 신뢰도를 나타내는 가중치 $w_i(\mathbf{x})$가 계산된다.
  3. 이 새로운 측정값은 이전 프레임까지 누적된 TSDF 값 $V_{i-1}(\mathbf{x})$와 누적 가중치 $W_{i-1}(\mathbf{x})$와 결합하여 다음과 같이 업데이트된다.22

  $$
  V_i(\mathbf{x}) = \frac{W_{i-1}(\mathbf{x})V_{i-1}(\mathbf{x}) + w_i(\mathbf{x})v_i(\mathbf{x})}{W_{i-1}(\mathbf{x}) + w_i(\mathbf{x})}
  $$

  $$
  W_i(\mathbf{x}) = W_{i-1}(\mathbf{x}) + w_i(\mathbf{x})
  $$

- **장점:**

  - **센서 노이즈 완화:** 위와 같은 가중 평균 방식은 여러 프레임에 걸쳐 측정된 값을 자연스럽게 평균화하는 효과를 가져온다. 이는 깊이 센서가 가진 고유의 무작위 노이즈(speckles 등)를 효과적으로 필터링하여 매우 부드럽고 안정적인 표면을 재구성하게 한다.23
  - **고품질 표면 재구성:** TSDF에서 환경의 표면은 SDF 값이 0이 되는 지점, 즉 양수 값과 음수 값이 교차하는 '제로-크로싱(zero-crossing)'으로 정의된다. 이 제로-크로싱 지점을 보간(interpolation)하여 찾기 때문에, 복셀 크기보다 더 정밀한 서브-복셀(sub-voxel) 수준의 표면 위치를 얻을 수 있다. 이는 Marching Cubes와 같은 알고리즘을 통해 매우 매끄럽고 고품질의 3D 메시를 생성하는 기반이 된다.15
  - **메모리 효율성:** `nvblox`는 관측된 영역에 대해서만 복셀 블록을 동적으로 할당하는 해싱(hashing) 기반의 자료구조를 사용한다.24 이로 인해 로봇이 넓은 공간을 탐험하더라도 메모리 사용량이 효율적으로 관리된다.


TSDF가 환경의 '표면'을 정밀하게 표현하는 데 중점을 둔다면, ESDF는 그 표면을 기반으로 '안전한 공간'을 탐색하는 데 최적화된 표현 방식이다.

- **개념 및 필요성:** ESDF는 공간상의 모든 자유 공간(free-space) 복셀에 대해, 가장 가까운 장애물(occupied) 복셀까지의 실제 유클리드 거리를 저장한다. 이 '안전 거리' 정보는 경로 계획기에게 매우 유용하다. 예를 들어, 경로 계획기는 ESDF 값을 통해 특정 경로가 장애물과 얼마나 떨어져 있는지 즉시 알 수 있으며, 이 정보를 비용 함수로 사용하여 더 안전하고 부드러운 경로를 생성할 수 있다. 특히 CHOMP와 같은 최적화 기반 경로 계획기는 ESDF의 그래디언트(gradient)를 따라 경로를 '밀어내어' 충돌을 회피하는데, 이 때문에 ESDF는 현대 로봇 내비게이션에서 필수적인 요소로 간주된다.10

- **TSDF로부터 ESDF 생성 알고리즘 (voxblox 기반 원리):** `nvblox`는 이미 구축된 TSDF 맵으로부터 ESDF 맵을 효율적으로 파생시킨다. 이 과정은 CPU 기반의 선행 연구인 `voxblox`에서 제안된 점진적 업데이트 알고리즘에 기반하며, `nvblox`는 이를 GPU에서 병렬로 수행하여 극적인 성능 향상을 이룬다.2

  1. **초기화 및 시드(Seed) 생성:** TSDF와 ESDF는 별개의 레이어로 관리된다. TSDF 맵이 업데이트되면, 변경 사항이 ESDF 레이어로 전파된다. 먼저, TSDF 표면 주변의 절단 거리 내에 있는 복셀들, 즉 유효한 거리 값을 가진 복셀들이 ESDF 계산의 '시드'가 된다. 이 영역의 ESDF 복셀들은 TSDF의 거리 값을 직접 상속받고 '고정(fixed)'된 것으로 표시된다.24

  2. **파면 전파 (Wavefront Propagation):** 이 고정된 시드 복셀들로부터 파면(wavefront)이 시작된다. 알고리즘은 두 개의 우선순위 큐, 즉 `raise` 큐와 `lower` 큐를 유지한다.

     - `raise` 큐: 장애물이 제거되어 기존의 ESDF 값이 더 이상 유효하지 않게 된 복셀들이 들어간다. 이 큐의 복셀들은 자신을 부모로 삼던 주변 복셀들에게 '거리가 멀어졌다'는 정보를 전파한다.

     - lower 큐: 새로운 장애물이 추가되거나 더 가까운 장애물이 발견되어 ESDF 값이 줄어들어야 하는 복셀들이 들어간다. 이 큐의 복셀들은 주변 이웃들에게 '거리가 가까워졌다'는 정보를 전파하며, 유클리드 거리에 따라 이웃들의 거리 값을 업데이트한다.

       이 파면 전파 과정이 큐가 빌 때까지 반복되면, 맵 전체에 걸쳐 모든 복셀이 가장 가까운 장애물까지의 정확한 유클리드 거리를 갖게 된다.24

       `nvblox`는 이 모든 과정을 수많은 GPU 스레드를 통해 동시에 처리함으로써, CPU 코어에서 순차적으로 처리하던 `voxblox` 대비 수십 배의 속도 향상을 달성한다.2

- **2D ESDF 슬라이스:** Nav2와 같은 대부분의 이동 로봇 경로 계획기는 2D 평면에서 동작한다. `nvblox`는 이러한 시스템과의 호환성을 위해, 완성된 3D ESDF 맵의 특정 높이(`esdf_slice_height` 파라미터로 지정)를 중심으로 얇은 2D 단면을 추출한다. 이 2D 슬라이스가 바로 `nvblox_nav2` 플러그인으로 전달되는 `map_slice` 토픽의 내용물이며, Nav2의 로컬 코스트맵 역할을 수행한다.4

이처럼 TSDF와 ESDF는 '표면 재구성'과 '경로 계획'이라는 상호 보완적인 두 가지 목적을 위해 설계된 이중적 표현(dual representation) 전략이다. TSDF가 센서 데이터를 융합하여 '무엇이 어디에 있는가'에 대한 가장 정확한 모델을 만드는 데 최적화되어 있다면, ESDF는 이 모델을 기반으로 '어떻게 안전하게 움직일 것인가'에 대한 탐색 정보를 가공하는 역할을 한다. `nvblox`는 이 2단계 프로세스 전체를 GPU로 가속하여 실시간으로 만들었다는 점에서 기술적 혁신을 이루었다.


`nvblox_nav2`를 성공적으로 구동하기 위해서는 올바른 하드웨어 및 소프트웨어 환경을 구축하고, 애플리케이션에 맞게 주요 파라미터를 설정하는 과정이 필수적이다.


`nvblox`는 특정 버전의 소프트웨어 스택과 고성능 하드웨어에 강하게 의존한다.

- **ROS 2 버전:** 공식적으로 **ROS 2 Humble Hawksbill**이 지원 및 테스트되었다. `nvblox`는 Humble 릴리즈부터 도입된 특정 ROS 2 구현 기능에 의존하기 때문에, Foxy 등 이전 버전은 지원되지 않는다.11 Iron Irwini와 같은 최신 버전에 대한 호환성은 보장되지 않으며, RViz2 플러그인 충돌과 같은 예기치 않은 문제가 발생할 수 있다.30
- **운영체제 및 하드웨어:**
  - **x86_64 플랫폼:** Ubuntu 20.04 또는 22.04 환경이 권장된다. 핵심 요구사항은 NVIDIA GPU이며, 최상의 성능을 위해 Ampere 아키텍처 이상의 그래픽 카드가 권장된다.11
  - **Jetson 임베디드 플랫폼:** NVIDIA Jetson Orin 또는 Xavier 시리즈가 지원된다. JetPack 버전은 5.1.1 이상이 필요하다.15 Jetson Orin Nano 4GB 모델은 메모리 부족 문제로 인해 대부분의 Isaac ROS 패키지 구동에 부적합하여 권장되지 않는다.11
- **소프트웨어 스택:**
  - **CUDA:** CUDA 11.8 이상 버전이 필요하다.15
  - **CMake:** CMake 3.22 이상 버전이 요구된다.18 구버전 CMake 사용 시 빌드 오류가 발생할 수 있다.
  - **NVIDIA 드라이버:** 호스트 시스템에 설치된 NVIDIA 드라이버 버전과 컨테이너 내 CUDA 버전 간의 호환성이 중요하며, 버전 불일치는 종종 `libnvidia-compute` 관련 의존성 문제를 야기한다.31


`nvblox`를 설치하는 방법은 크게 두 가지가 있으며, NVIDIA는 Docker 기반 설치를 강력히 권장한다.

- **Docker 기반 설치 (권장 방식):**
  - **이유:** `nvblox`와 Isaac ROS 생태계는 복잡하고 민감한 의존성 관계를 가지고 있다. Docker를 사용하면 이러한 모든 의존성(특정 버전의 CUDA, cuDNN, TensorRT 등)이 사전 구성된 격리된 환경에서 개발을 진행할 수 있어, "내 컴퓨터에서는 안 되는데"와 같은 일반적인 환경 설정 문제를 근본적으로 방지할 수 있다.15
  - **절차:**
    1. 호스트 시스템에 NVIDIA Container Toolkit을 설치한다.34
    2. `isaac_ros_common` 저장소를 클론하고, 내부에 포함된 `run_dev.sh` 스크립트를 사용하여 개발용 Docker 컨테이너를 실행한다.35
    3. 실행된 컨테이너 내부에서 `sudo apt-get install ros-humble-isaac-ros-nvblox` 명령어로 데비안 패키지를 설치하거나, 소스 코드를 클론한 후 `rosdep`으로 의존성을 해결하고 `colcon build`로 직접 빌드한다.35
- **소스 코드 직접 빌드 (Native Install):**
  - Docker를 사용하지 않고 호스트 시스템에 직접 `nvblox`를 설치하는 방식이다.
  - 앞서 언급된 모든 의존성(ROS 2 Humble, 특정 버전의 CUDA, CMake, NVIDIA 드라이버 등)을 사용자가 직접 정확하게 설치하고 환경 변수를 설정해야 한다.18
  - 이 과정은 매우 복잡하고, 버전 불일치로 인한 빌드 오류(예: CUDA 컴파일러 경로 문제, CMake 버전 문제)에 직면할 가능성이 매우 높다.18 전문가가 아닌 이상 권장되지 않는 방법이다.


`nvblox`의 동작은 ROS 파라미터를 통해 세밀하게 제어된다. 이 파라미터들은 주로 `nvblox_examples_bringup/config/` 디렉토리 내의 YAML 파일(`nvblox_base.yaml` 및 `specializations/*.yaml`)을 통해 설정된다.37 주요 파라미터의 의미와 역할은 다음과 같다.

- **`global_frame` (string):** 맵이 생성되고 유지되는 기준 좌표계. 일반적으로 로봇의 주행 기록(odometry)이 누적되는 "odom" 프레임이나, SLAM에 의해 전역적으로 위치가 보정되는 "map" 프레임을 사용한다.37
- **`voxel_size` (float):** 맵의 해상도를 결정하는 가장 중요한 파라미터 중 하나. 복셀의 한 변 길이를 미터 단위로 지정한다. 값이 작을수록 더 정밀하고 상세한 맵을 만들 수 있지만, 메모리 사용량과 계산량이 기하급수적으로 증가한다. 반대로 값이 크면 성능은 향상되지만 작은 장애물을 놓치거나 맵의 정확도가 저하된다. 5cm(`0.05`)가 일반적인 기본값이다.37
- **`mapping_type` (string):** 맵핑 방식을 선택한다. 기본값은 "static_tsdf"이며, "static_occupancy"로 설정하면 TSDF 대신 전통적인 점유 격자 방식으로 맵을 생성한다.40
- **`esdf_mode` (int):** ESDF를 계산할 방식을 결정한다. 3D 공간 전체에 대해 계산하려면 `0`으로, Nav2와 같이 2D 평면 내비게이션을 위해 2D 슬라이스만 필요할 경우 `1`로 설정한다. `nvblox_nav2` 연동 시에는 반드시 `1`로 설정해야 한다.39
- **`esdf_slice_height` (float):** `esdf_mode`가 `1`(2D)일 때, 2D 코스트맵을 추출할 기준 높이를 로봇의 `base_link` 프레임 기준으로 지정한다. 로봇의 주행 환경과 센서 높이에 맞게 적절히 설정해야 한다.
- **각종 업데이트 주기 (`\*_update_rate_hz`):** `depth_update_rate_hz`, `esdf_update_rate_hz`, `mesh_update_rate_hz` 등 각 레이어의 업데이트 주기를 Hz 단위로 설정한다. 시스템의 실시간성과 계산 부하 사이의 균형을 맞추는 데 사용된다. 예를 들어, 동적 장애물 회피가 매우 중요하다면 `esdf_update_rate_hz`를 높이고, 시각화의 중요도가 낮다면 `mesh_update_rate_hz`를 낮출 수 있다.37
- **`map_clearing_radius_m` (float):** 로봇이 무한정 주행할 때 맵이 계속 커져 메모리가 고갈되는 것을 방지하기 위한 기능이다. 로봇의 현재 위치(`map_clearing_frame_id`로 지정된 프레임, 보통 `base_link`)를 중심으로 이 파라미터에 지정된 반경 밖의 맵 데이터를 주기적으로 삭제한다. `0.0` 이하의 값으로 설정하면 이 기능이 비활성화된다.37 로컬 내비게이션에만 집중할 경우 유용한 옵션이다.

이 외에도 수십 개의 파라미터가 존재하며, 이를 통해 센서 모델, 통합 가중치, 동적 객체 처리 방식 등을 미세하게 조정할 수 있다. 성공적인 `nvblox` 적용을 위해서는 이러한 파라미터들의 의미를 정확히 이해하고, 자신의 로봇과 주행 환경에 맞게 튜닝하는 과정이 필수적이다.

**Table 2: 주요 YAML 설정 파라미터 및 역할**

| 파라미터 이름 (Parameter Name)                     | 타입 (Type) | 기본값 (Default) | 역할 및 튜닝 가이드 (Role & Tuning Guidance)                 | 출처 (Source) |
| -------------------------------------------------- | ----------- | ---------------- | ------------------------------------------------------------ | ------------- |
| `voxel_size`                                       | `float`     | `0.05`           | 맵의 해상도(m). 작을수록 정밀하지만 메모리/연산량 증가. 환경의 복잡도와 시스템 성능 간의 트레이드오프를 통해 결정. | 37            |
| `global_frame`                                     | `string`    | `"odom"`         | 맵의 기준이 되는 TF 프레임. 전역 SLAM과 연동 시 `"map"`으로 변경. | 37            |
| `use_tf_transforms`                                | `bool`      | `true`           | `true`이면 TF 트리에서 포즈를 조회하고, `false`이면 토픽으로 포즈를 구독. | 37            |
| `esdf_mode`                                        | `int`       | `1`              | ESDF 계산 모드. `0`은 3D, `1`은 2D. Nav2 연동 시 `1`로 설정. | 39            |
| `esdf_slice_height`                                | `float`     | `0.0`            | 2D ESDF 슬라이스를 추출할 높이(m). 로봇의 주행 평면에 맞게 설정. | -             |
| `max_depth_update_hz`                              | `float`     | `10.0`           | 깊이 이미지를 통합하는 최대 주기(Hz). 0은 제한 없음. 시스템 부하를 줄이려면 낮춤. | 37            |
| `update_esdf_rate_hz`                              | `float`     | -                | ESDF 맵을 업데이트하는 주기(Hz). 동적 장애물 반응성을 높이려면 값을 높임. | 40            |
| `map_clearing_radius_m`                            | `float`     | `-1.0`           | 로봇 주변 일정 반경(m) 밖의 맵을 삭제. 양수 값으로 활성화. 로컬 맵핑에 유용. | 37            |
| `map_clearing_frame_id`                            | `string`    | `"base_link"`    | 맵 클리어링의 중심이 되는 TF 프레임.                         | 37            |
| `use_static_occupancy_layer`                       | `bool`      | `false`          | `true`로 설정 시 TSDF 대신 점유 격자(Occupancy) 맵을 사용.   | 37            |
| `projective_integrator.max_integration_distance_m` | `float`     | `7.0`            | 센서로부터 이 거리(m)를 초과하는 측정값은 무시. 센서의 유효 거리에 맞춰 조정. | -             |
| `projective_integrator.truncation_distance_vox`    | `float`     | `4.0`            | TSDF 절단 거리를 복셀 단위로 지정. 값이 크면 더 두꺼운 표면이 생성되나 노이즈에 취약해질 수 있음. | -             |


`nvblox`의 가장 큰 가치는 그 성능에 있으며, 이는 NVIDIA GPU의 병렬 처리 능력을 극대화한 결과이다. 이 섹션에서는 `nvblox`의 성능을 정량적으로 분석하고, 다양한 환경과 요구사항에 맞춰 성능을 최적화하는 전략을 논한다.


`nvblox`는 스위스 취리히 연방 공과대학교(ETH Zurich)에서 개발한 CPU 기반의 선구적인 실시간 3D 맵핑 라이브러리인 `voxblox`의 알고리즘적 유산을 계승한다.2

`voxblox`는 TSDF와 ESDF를 점진적으로 구축하는 효율적인 방법을 제시했지만, 모든 연산을 CPU에서 순차적으로 처리했기 때문에 맵의 해상도나 업데이트 주기에 명백한 한계가 있었다.2

`nvblox`는 `voxblox`의 핵심 아이디어를 가져와 모든 계산 파이프라인을 GPU에서 병렬로 실행하도록 재설계했다.3 그 결과는 극적인 성능 향상으로 나타났다. 학술 논문[2311.00626]에 발표된 벤치마크 결과에 따르면, 

`nvblox`는 CPU 기반의 최신 방법들과 비교했을 때, 표면 재구성(TSDF 통합)에서 최대 177배, 그리고 경로 계획에 필수적인 거리장 계산(ESDF 생성)에서 최대 31배의 속도 향상을 달성했다.2

이러한 압도적인 성능 향상은 단순한 속도 개선 이상의 의미를 가진다. 이는 CPU 기반 시스템에서는 실시간 처리가 불가능했던 높은 해상도(예: 5cm 이하)의 맵을 실시간으로 구축하고 업데이트하는 것을 가능하게 하여, 로봇이 더 정밀하게 환경을 인식하고 더 안전하게 항해할 수 있는 길을 열었다.2


`nvblox`의 성능은 사용되는 GPU의 사양에 직접적으로 비례한다. NVIDIA는 다양한 하드웨어 플랫폼에서의 성능 측정 결과를 제공하여, 개발자가 자신의 시스템에서 기대할 수 있는 성능 수준을 가늠할 수 있도록 돕는다.11

예를 들어, Replica 데이터셋과 5cm 복셀 크기를 기준으로 한 벤치마크에서 각 처리 단계별 소요 시간은 다음과 같이 측정되었다 11:

- **TSDF 통합:** 데스크톱(RTX 3090)에서 0.5ms, AGX Orin에서 0.8ms, Orin Nano에서 2.1ms.
- **ESDF 생성:** 데스크톱에서 0.8ms, AGX Orin에서 1.7ms, Orin Nano에서 6.2ms.

이 결과는 하이엔드 데스크톱 GPU와 임베디드 GPU 간의 명확한 성능 차이를 보여주며, 애플리케이션의 실시간 요구사항과 예산에 맞춰 적절한 하드웨어를 선택하는 것이 중요함을 시사한다.

또한, `isaac_ros_nvblox` 패키지에는 `nvblox_performance_measurement`라는 성능 측정 도구가 포함되어 있다. 이 도구를 사용하면 특정 ROS Bag 파일을 재생하면서 `nvblox` 노드의 처리율(Hz), 메시지 지연 시간, 그리고 CPU 및 GPU 점유율과 같은 핵심 성능 지표(KPI)를 직접 측정하고 분석할 수 있다.41


실제 환경에는 정적인 장애물뿐만 아니라 사람, 다른 로봇과 같은 동적 장애물이 존재한다. `nvblox`는 이러한 동적 환경에 대응하기 위해 여러 가지 작동 모드를 제공한다.

- **정적 모드 (Static Mode):** 모든 환경 요소가 변하지 않는다고 가정하는 가장 기본적인 모드이다.11
- **사람 재구성 (People Reconstruction):** 이 모드에서는 `unet`과 같은 딥러닝 기반의 시맨틱 분할(semantic segmentation) 모델을 사용하여 카메라 이미지에서 사람을 먼저 식별한다.11 식별된 사람 영역의 깊이 정보는 별도의 동적 점유 격자 맵(dynamic occupancy grid map)으로 처리되고, 나머지 배경은 정적인 TSDF 맵으로 재구성된다. 이 방식은 맵에 사람이 영구적인 '유령' 장애물로 남는 것을 방지하고, 사람과 일반 장애물을 구분하여 회피 전략을 세울 수 있게 한다.15
- **일반 동적 재구성 (Dynamic Reconstruction):** 사람 외의 일반적인 동적 객체를 처리하기 위한 모드이다. 이 모드에서는 점유 격자의 확률 값을 시간에 따라 점차 0.5(미확인 상태)로 감소시키는 '소멸(decay)' 메커니즘을 사용한다.15 이로 인해 한때 관측되었던 동적 장애물이 시야에서 사라지면, 해당 영역의 점유 확률이 점차 낮아져 결국 맵에서 사라지게 된다. 이는 로봇이 일시적인 장애물에 의해 경로가 영구적으로 막히는 것을 방지해준다.15


`nvblox`의 성능을 특정 애플리케이션에 맞게 최적화하기 위한 몇 가지 전략이 있다.

- **파라미터 튜닝:** YAML 설정 파일의 파라미터를 조정하는 것이 가장 기본적인 최적화 방법이다. `voxel_size`를 키우면 성능이 향상되지만 정밀도가 떨어지고, 각종 `*_update_rate_hz`를 낮추면 시스템 부하가 줄어들지만 실시간 반응성이 저하된다. `map_clearing_radius_m`을 활성화하면 장기적인 메모리 사용량을 관리할 수 있다. 이러한 파라미터들을 시스템의 요구사항(실시간성, 맵 품질, 메모리 제약)에 맞춰 신중하게 튜닝해야 한다.39
- **최신 릴리즈 활용:** NVIDIA는 `nvblox`의 성능을 지속적으로 개선하고 있다. 2024년 12월 릴리즈 노트에 따르면, 복셀의 대량 초기화를 통한 실행 오버헤드 감소, GPU에서 CPU로의 데이터 스트리밍 파이프라인 최적화, 병렬 커널 실행 극대화, 비동기 라이브러리 호출을 통한 동기화 지점 제거 등 다양한 런타임 최적화가 이루어졌다.43 항상 최신 버전을 사용하는 것이 최고의 성능을 얻는 길이다.
- **적절한 하드웨어 선택:** 목표 성능을 달성하기 위해서는 애플리케이션에 맞는 적절한 사양의 GPU를 선택하는 것이 근본적으로 중요하다. 고해상도, 다중 카메라, 빠른 업데이트가 필요하다면 Jetson AGX Orin이나 데스크톱 GPU가 필요하며, 비교적 단순한 환경에서는 Jetson Orin Nano로도 구동이 가능하다.11

**Table 3: 플랫폼별 Nvblox 성능 벤치마크 (Replica 데이터셋, 5cm 복셀 기준)**

| 처리 단계 (Component) | x86_64 w/ RTX 3090 (Desktop) | x86_64 w/ RTX A3000 (Laptop) | AGX Orin (Jetson) | Orin Nano (Jetson) |
| --------------------- | ---------------------------- | ---------------------------- | ----------------- | ------------------ |
| **TSDF 통합**         | 0.5 ms                       | 0.3 ms                       | 0.8 ms            | 2.1 ms             |
| **컬러 통합**         | 0.7 ms                       | 0.7 ms                       | 1.1 ms            | 3.6 ms             |
| **메시 생성**         | 0.7 ms                       | 1.3 ms                       | 2.3 ms            | 13.0 ms            |
| **ESDF 생성**         | 0.8 ms                       | 1.2 ms                       | 1.7 ms            | 6.2 ms             |
| **동적 객체 처리**    | 1.7 ms                       | 1.4 ms                       | 2.0 ms            | N/A                |

주: 위 표는 NVIDIA에서 제공한 벤치마크 결과를 재구성한 것임.11 처리 시간은 평균값이며, 실제 성능은 장면에 따라 달라질 수 있음. Orin Nano는 동적 객체 처리 모드를 지원하지 않을 수 있음.


`nvblox`의 기술적 가치를 명확히 이해하기 위해서는 기존의 내비게이션 및 3D 맵핑 기술들과의 비교가 필수적이다. 이 섹션에서는 `nvblox`를 전통적인 2D 내비게이션, 널리 사용되는 3D 맵핑 라이브러리인 OctoMap, 그리고 `nvblox`와 함께 사용되는 SLAM 시스템과 비교 분석한다.


- **Nvblox의 장점:** 전통적인 2D LiDAR 기반 내비게이션은 세상을 수평 단면으로만 인식한다. 이로 인해 로봇의 주행 경로 상에 있지만 LiDAR 스캔 평면보다 높거나 낮은 장애물, 예를 들어 돌출된 테이블 모서리, 벽에 걸린 선반, 낮은 포크리프트의 발, 또는 경사로나 계단 같은 3차원 지형을 전혀 인식하지 못하는 근본적인 한계가 있다. `nvblox`는 깊이 카메라를 통해 3D 공간 전체를 인식하므로 이러한 모든 종류의 장애물을 감지하고 코스트맵에 반영할 수 있다.1 이는 로봇의 주행 안전성과 환경에 대한 강건성을 비약적으로 향상시킨다.
- **Nvblox의 단점:** 3차원 인식의 장점은 높은 비용을 수반한다. `nvblox`는 강력한 NVIDIA GPU를 필수적으로 요구하며, 시스템 설정은 2D 내비게이션 스택에 비해 훨씬 복잡하다. 또한, 3D 복셀 맵은 2D 점유 격자 맵에 비해 훨씬 많은 메모리를 차지하며, 처리해야 할 데이터의 양도 방대하다.44 따라서 단순하고 평평한 환경에서는 2D 내비게이션이 여전히 비용 효율적인 선택일 수 있다.


`nvblox`와 OctoMap은 3D 환경을 복셀 기반으로 표현한다는 공통점이 있지만, 그 철학과 구현 방식에는 근본적인 차이가 있다.

- **기술적 차이:**
  - **데이터 표현:** `nvblox`의 핵심은 TSDF, 즉 표면까지의 '부호 있는 거리'를 저장하는 것이다. 반면, OctoMap은 각 복셀이 장애물에 의해 '점유되었을 확률'을 0과 1 사이의 값으로 저장한다.26
  - **자료 구조:** `nvblox`는 관측된 복셀 블록들을 해시 테이블(hash table)을 이용해 관리하는 플랫한 구조를 사용한다. 이는 특정 복셀에 대한 접근이 O(1)의 시간 복잡도를 가져 매우 빠르다.24 반면, OctoMap은 공간을 8개의 자식 노드로 재귀적으로 분할하는 계층적인 옥트리(octree) 구조를 사용한다. 이는 넓은 빈 공간이나 점유된 공간을 하나의 큰 노드로 압축하여 표현할 수 있어 메모리 효율성이 높다.28
  - **주 처리 장치:** `nvblox`는 모든 핵심 연산을 GPU에서 병렬로 처리하도록 설계되었다. OctoMap은 전통적으로 CPU 기반으로 동작한다.24
- **비교 분석:**
  - **맵 품질:** `nvblox`의 TSDF는 표면을 연속적인 함수(SDF)의 제로-크로싱으로 표현하므로, 서브-복셀 수준의 정밀도로 매우 부드럽고 실제와 유사한 메시를 생성할 수 있다.24 반면, OctoMap은 이산적인 복셀 단위로 점유 여부만 표현하므로 생성된 맵은 계단 현상(aliasing)이 두드러지고 표면의 디테일이 떨어진다.
  - **성능:** `nvblox`는 GPU 가속 덕분에 CPU 기반의 OctoMap보다 훨씬 빠른 속도로 센서 데이터를 맵에 통합할 수 있다.24 이는 로봇이 빠르게 움직이거나 환경이 급격하게 변하는 상황에서 실시간성을 유지하는 데 결정적인 장점이다.
  - **내비게이션 활용성:** 이 지점에서 두 기술의 차이가 가장 극명하게 드러난다. `nvblox`가 생성하는 ESDF는 모든 지점에서 가장 가까운 장애물까지의 거리와 방향(그래디언트) 정보를 직접 제공한다. 이는 최적화 기반의 고급 경로 계획 알고리즘이 충돌을 회피하고 경로의 품질을 향상시키는 데 매우 유용한 정보이다.27 반면, OctoMap은 점유/비점유라는 이진 정보만 제공하므로, 이러한 거리 기반 계획 기법에 직접적으로 활용하기 어렵다.

결론적으로, `nvblox`와 OctoMap의 비교는 단순히 두 라이브러리의 성능 비교를 넘어선다. 이는 로봇 맵핑에 대한 두 가지 다른 철학의 대립으로 볼 수 있다. OctoMap이 '센서 측정의 불확실성을 확률적으로 어떻게 모델링할 것인가'에 집중한다면, `nvblox`는 '압도적인 계산 능력을 바탕으로 경로 계획에 더 유용한 표현(거리장)을 어떻게 실시간으로 만들 것인가'에 집중한다. 이 선택은 로봇 시스템 전체의 아키텍처와 성능에 근본적인 영향을 미친다.


`nvblox`는 맵핑 라이브러리일 뿐, 스스로 로봇의 위치를 추정하지 않는다. 따라서 `isaac_ros_visual_slam`이나 ORB-SLAM3과 같은 외부 SLAM 시스템으로부터 정확한 포즈 정보를 실시간으로 공급받는 것이 필수적이다.

- **`isaac_ros_visual_slam` (cuVSLAM):** NVIDIA가 `nvblox`와 함께 Isaac ROS 생태계 내에서 사용하도록 개발한 V-SLAM(Visual SLAM) 시스템이다. 모든 핵심 알고리즘이 GPU 가속에 최적화되어 있어 매우 빠른 처리 속도를 자랑한다. KITTI 벤치마크에서 유명한 오픈소스인 ORB-SLAM2보다 낮은 오차율과 8배 이상 빠른 런타임을 기록했다.47 스테레오 카메라와 IMU 입력을 융합하여 시각 정보가 부족한 환경(예: 특징 없는 벽)에서도 강건하게 위치를 추정할 수 있다.48

  `nvblox`와의 긴밀한 통합을 염두에 두고 설계되어, Isaac ROS 사용자에게 가장 자연스러운 선택지이다.4

- **ORB-SLAM3:** 학계에서 널리 인정받는 가장 대표적인 오픈소스 V-SLAM 라이브러리 중 하나이다. 단안, 스테레오, RGB-D, 관성 센서 등 다양한 센서 조합을 지원하며, 특히 이전에 방문했던 장소를 재인식하여 맵을 합치고 오차를 보정하는 강력한 루프 클로저(loop closure) 및 다중 맵(multi-map) 관리 기능이 최대 강점이다.49

  `nvblox`와 달리 주로 CPU 기반으로 동작하며, `isaac_ros_visual_slam`에 비해 처리 속도는 느릴 수 있으나, 장기적인 맵의 일관성과 정확도 측면에서는 더 우수한 성능을 보일 수 있다.47

`nvblox`와 어떤 SLAM 시스템을 조합할 것인가는 애플리케이션의 요구사항에 따라 결정된다. 최고의 실시간 성능과 NVIDIA 생태계 내에서의 완벽한 통합을 원한다면 `isaac_ros_visual_slam`이, 장시간에 걸친 전역적 맵 정확성과 학술적 검증이 더 중요하다면 ORB-SLAM3이 더 적합한 선택이 될 수 있다.

**Table 4: Nvblox vs. OctoMap 기술 비교**

| 특징 (Feature)       | Nvblox                                                       | OctoMap                                          |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| **데이터 표현**      | 부호 있는 거리 필드 (Signed Distance Field)                  | 점유 확률 (Occupancy Probability)                |
| **자료 구조**        | 해시 기반 복셀 블록 (Hashed Voxel Blocks)                    | 옥트리 (Octree)                                  |
| **주 처리 장치**     | GPU (NVIDIA)                                                 | CPU                                              |
| **맵 품질**          | 서브-복셀 정밀도의 연속적인 표면, 고품질 메시 생성           | 이산적인 복셀, 계단 현상 발생                    |
| **경로 계획 적합성** | ESDF를 통해 장애물까지의 거리/그래디언트 직접 제공, 최적화 기반 계획에 매우 유리 | 점유/비점유 정보만 제공, 거리 기반 계획에 부적합 |
| **동적 환경 처리**   | 시맨틱 분할(사람) 또는 확률 소멸(decay) 방식 지원            | 확률적 업데이트를 통해 자연스럽게 처리           |
| **메모리 효율성**    | 관측된 영역만 할당하여 효율적                                | 넓은 빈 공간/점유 공간을 압축하는 데 유리        |
| **성능**             | GPU 가속으로 매우 빠른 실시간 맵 업데이트 가능               | CPU 기반으로 상대적으로 느림                     |


`nvblox`는 강력한 성능과 기능을 제공하지만, 실제 로봇에 성공적으로 적용하기 위해서는 몇 가지 시나리오와 기술적 과제, 그리고 내재적 한계를 이해해야 한다.


NVIDIA는 개발자들이 `nvblox`를 쉽게 시작하고 테스트할 수 있도록 다양한 예제와 튜토리얼을 제공한다.

- **Isaac Sim 시뮬레이션:** `nvblox`를 처음 접하고 개발하는 가장 쉽고 효율적인 방법은 NVIDIA의 로봇 시뮬레이터인 Isaac Sim을 활용하는 것이다. Isaac Sim은 물리적으로 정확한 환경과 실제와 유사한 센서(깊이 카메라, LiDAR 등) 데이터를 생성하며, 내장된 ROS 2 브릿지를 통해 시뮬레이션 환경과 `nvblox` 노드를 직접 연결할 수 있다.16

  `carter_warehouse.py`와 같은 예제 스크립트는 시뮬레이션 내에서 `nvblox`를 이용한 내비게이션 전체 스택을 구동하는 방법을 보여준다.16

- **실제 로봇 적용 (RealSense/ZED):** Intel RealSense D435i, Stereolabs ZED 등 널리 사용되는 깊이 카메라를 장착한 실제 로봇에 `nvblox`를 적용하는 상세한 튜토리얼이 제공된다.4 이 튜토리얼들은 카메라 드라이버 설치부터 

  `isaac_ros_visual_slam`과의 연동, `nvblox` 설정, Nav2 통합까지 전 과정을 안내한다.

- **데이터 기록 및 재생 (ROS Bag):** `ros2 bag record` 명령어를 사용하여 실제 로봇이나 시뮬레이터에서 수집된 센서 데이터(이미지, 포즈, TF 등)를 파일로 기록할 수 있다. 이후, `nvblox` 런치 파일에 `from_bag:=True` 옵션을 주고 이 파일을 재생하면, 실제 로봇 없이도 동일한 조건에서 `nvblox`의 동작을 반복적으로 테스트하고 파라미터를 튜닝하며 디버깅할 수 있다.52


`nvblox`를 실제 시스템에 통합하는 과정에서 개발자들은 몇 가지 공통적인 문제에 직면할 수 있다.

- **TF 트리 문제:** `nvblox`가 "Could not get transform from odom to camera_link"와 같은 오류를 내며 동작하지 않는 경우가 가장 흔한 문제 중 하나이다. 이는 `nvblox`가 로봇의 포즈 정보를 얻지 못해 발생하는 것으로, 근본 원인은 `map` -->> `odom` -->> `base_link` -->> `camera_link`로 이어지는 좌표 변환 체인(TF tree) 어딘가가 끊어졌기 때문이다.53 이 문제는 

  `nvblox` 자체의 결함이라기보다는, 연동된 SLAM 시스템(`isaac_ros_visual_slam` 등)이 포즈를 제대로 발행하지 않거나, 로봇의 URDF 설정이 잘못되었거나, 휠 오도메트리 발행 노드가 비정상적인 경우에 발생한다. `robot_localization` 패키지를 사용하여 휠 오도메트리, IMU, V-SLAM 출력을 융합하여 안정적인 `odom` 프레임을 생성하는 것이 일반적인 해결책이다.40

- **센서 데이터 문제:** 특히 Intel RealSense 카메라의 경우, 최신 리눅스 커널 버전과의 호환성 문제로 인해 커널 모듈이 제대로 로드되지 않을 수 있다. 이 경우, 이미지가 ROS 토픽으로 전혀 발행되지 않거나, 발행되더라도 VSLAM과 `nvblox`가 요구하는 프로젝터 상태 메타데이터가 누락될 수 있다. 이 문제 해결을 위해 `librealsense-dkms` 패키지를 설치하여 커널을 패치하거나, `librealsense` 라이브러리를 CUDA 지원 없이 소스에서 직접 빌드해야 할 수 있다.54

- **네트워크 부하:** `nvblox`가 시각화를 위해 발행하는 메시(특히 `~/mesh`나 `~/static_occupancy` 포인트클라우드)는 데이터 크기가 매우 크다. 로봇과 개발용 PC가 Wi-Fi로 연결된 경우, 이 대용량 데이터를 전송하는 데 병목이 발생하여 RViz에서 심각한 지연(lag)이나 메시지 유실이 발생할 수 있다.18 디버깅 시에는 메시 발행 주기(

  `update_mesh_rate_hz`)를 낮추거나, 복셀 크기(`voxel_size`)를 키워 전체 메시의 데이터 양을 줄이는 것이 임시적인 해결책이 될 수 있다.


`nvblox`는 매우 강력한 도구이지만, 모든 문제를 해결하는 만능 솔루션은 아니며 몇 가지 명확한 한계를 가지고 있다.

- **지역 맵핑(Local Mapping)에 국한:** `nvblox`의 가장 큰 구조적 한계는 전역적인 맵 일관성(global consistency)을 보장하는 메커니즘이 없다는 점이다. `nvblox`는 현재 시점의 센서 입력과 포즈를 기반으로 지역적인(local) 맵을 생성하고 업데이트하는 데 초점을 맞춘다. 만약 SLAM 시스템이 루프 클로저를 통해 과거의 경로를 수정하여 로봇의 포즈 그래프 전체를 업데이트하더라도, `nvblox`는 이 과거의 변화를 소급하여 이미 생성된 3D 맵에 반영하지 않는다.44 따라서 

  `nvblox`가 생성한 맵은 장시간, 넓은 영역을 주행할 경우 SLAM의 드리프트가 누적되어 뒤틀릴 수 있으며, 전역적으로 일관된 맵을 요구하는 애플리케이션에는 적합하지 않다. 현재로서는 주로 Nav2의 로컬 코스트맵과 같이 로봇 주변의 동적 장애물 회피를 위한 지역 맵 생성에 사용하는 것이 주된 용도이다.

- **NVIDIA 하드웨어 종속성:** `nvblox`의 핵심적인 성능은 전적으로 NVIDIA GPU와 CUDA 생태계에 의존한다. 이는 AMD나 Intel의 GPU, 또는 GPU가 없는 시스템에서는 `nvblox`를 사용할 수 없음을 의미한다. 이는 기술적인 종속성을 야기하며, 사용자의 하드웨어 선택의 폭을 제한하는 명백한 단점이다.10

- **복잡한 의존성 관리:** `nvblox`는 특정 버전의 ROS, CUDA, NVIDIA 드라이버, JetPack 등에 강하게 의존한다. 이러한 복잡한 의존성 관계는 개발 환경 설정을 매우 어렵게 만들며, 버전 불일치는 해결하기 어려운 빌드 및 런타임 오류의 주된 원인이 된다.31 NVIDIA가 Docker 사용을 강력히 권장하는 이유도 바로 이 때문이다.



`nvblox`와 이를 Nav2에 통합하는 `nvblox_nav2` 패키지는 현대 로봇 내비게이션이 직면한 핵심적인 도전 과제, 즉 복잡하고 동적인 3차원 환경에서의 실시간 인식 및 경로 계획에 대한 NVIDIA의 강력한 해답이다. 이 기술은 NVIDIA GPU의 압도적인 병렬 처리 능력을 활용하여, 기존의 CPU 기반 시스템으로는 불가능했던 수준의 성능과 정밀도를 달성했다.

`nvblox_nav2`의 핵심적인 기술적 기여는 두 가지로 요약할 수 있다. 첫째, 고품질의 3D 표면 재구성을 위한 TSDF와 경로 계획에 즉시 사용 가능한 ESDF를 실시간으로 동시에 생성함으로써, 로봇의 '인식(Perception)'과 '계획(Planning)' 사이의 간극을 효과적으로 메웠다. 둘째, 시맨틱 분할과 확률 소멸 기법을 도입하여 정적인 환경뿐만 아니라 사람이 오가는 동적인 환경에서도 강건하게 동작할 수 있는 기반을 마련했다.


본 보고서에서 분석한 내용을 종합하면 `nvblox_nav2`의 장단점은 다음과 같다.

- **장점:**
  - **압도적인 실시간 성능:** GPU 가속을 통해 고해상도 3D 맵을 실시간으로 구축 및 업데이트한다.
  - **고품질 3D 맵:** TSDF 기반으로 노이즈가 적고 실제와 유사한 정밀한 3D 표면을 재구성한다.
  - **동적 환경 대응 능력:** 사람을 포함한 동적 장애물을 효과적으로 처리하여 맵 오염을 방지한다.
  - **뛰어난 내비게이션 활용성:** ESDF를 통해 최적화 기반 경로 계획기에 필수적인 거리 정보를 제공한다.
  - **Nav2와의 긴밀한 통합:** `nvblox_nav2` 플러그인을 통해 ROS 2의 표준 내비게이션 스택인 Nav2와 원활하게 연동된다.
- **단점:**
  - **NVIDIA 하드웨어 강제성:** NVIDIA GPU와 CUDA 생태계에 대한 강한 기술적 종속성을 가진다.
  - **복잡한 환경 설정:** 특정 버전의 ROS, 드라이버, 라이브러리에 대한 의존성이 높아 환경 구축이 복잡하다.
  - **전역 맵 일관성 부재:** 루프 클로저에 의한 과거 포즈 업데이트를 맵에 반영하지 못해, 전역 맵핑보다는 지역 맵핑에 적합하다.

이러한 특성을 고려할 때, `nvblox_nav2`는 다음과 같은 분야에 가장 효과적으로 적용될 수 있다.

- **물류 창고, 공장 등 동적 요소가 많은 실내 환경을 주행하는 자율 이동 로봇(AMR):** 실시간으로 변화하는 장애물(사람, 지게차 등)을 회피하며 안전하게 임무를 수행해야 하는 경우.
- **정밀한 충돌 회피가 요구되는 로봇 팔(Manipulator)의 작업 공간 맵핑:** 로봇 팔이 주변 환경이나 작업 대상과 충돌하지 않도록 작업 공간을 정밀하게 3D로 맵핑하고 실시간으로 업데이트해야 하는 경우.
- **3차원 지형 인식이 중요한 실외 로봇:** 2D LiDAR로는 한계가 있는 복잡한 지형이나 구조물을 인식하며 주행해야 하는 경우(단, 전역적 일관성보다는 지역적 장애물 회피가 더 중요할 때).


`nvblox`는 정체된 프로젝트가 아니며, NVIDIA의 로보틱스 전략의 핵심 축으로서 지속적으로 발전하고 있다.

- **성능 지속 개선:** NVIDIA는 릴리즈를 거듭하며 `nvblox`의 런타임 성능을 꾸준히 최적화하고 있다. 이를 통해 더 낮은 사양의 하드웨어(예: Jetson Nano)에서도 구동이 가능해지고, 동일 하드웨어에서는 더 높은 해상도(예: 1cm 복셀)의 맵을 실시간으로 처리할 수 있게 될 것이다.43

- **기능 확장:** 현재는 깊이 카메라 입력이 중심이지만, 향후 3D LiDAR 데이터 지원이 강화되고, 나아가 2D LiDAR 데이터와의 융합도 고려될 수 있다.56 또한, 다중 카메라 입력을 보다 효율적으로 처리하고 동적 객체 탐지 및 분리 능력을 고도화하는 방향으로 개발이 진행 중이다.43

- **Isaac Perceptor 및 Manipulator와의 통합:** `nvblox`는 단순한 개별 ROS 패키지를 넘어, NVIDIA가 제시하는 차세대 로보틱스 애플리케이션 프레임워크인 Isaac Perceptor(AMR용)와 Isaac Manipulator(로봇 팔용)의 핵심 인식 기술로 깊숙이 통합되고 있다.5 이는 

  `nvblox`의 발전이 NVIDIA의 전체 로보틱스 비전과 함께 간다는 것을 의미하며, 향후 더 고도화된 기능들이 이들 플랫폼을 통해 제공될 것임을 시사한다.

- **생태계 확장:** NVIDIA는 Isaac ROS 블로그, ROS Discourse 포럼, 웨비나 등 다양한 채널을 통해 개발자 커뮤니티와 활발하게 소통하며 업데이트를 공유하고 있다.57 이는 

  `nvblox`를 포함한 Isaac ROS 생태계가 앞으로도 계속해서 진화하고, 사용자 피드백을 반영하며 성장해 나갈 것임을 보여준다.

결론적으로 `nvblox_nav2`는 로봇 내비게이션 기술의 패러다임을 2D에서 3D로, CPU에서 GPU로 전환시키는 중요한 이정표이며, NVIDIA의 지속적인 투자와 개발 로드맵을 통해 그 영향력은 앞으로 더욱 확대될 것으로 전망된다.


1. Towards Next-Gen Autonomous Mobile Robotics: A Technical Deep Dive into Visual-Data-Driven AMRs Powered by Kudan Visual SLAM and NVIDIA Isaac Perceptor, 8월 9, 2025에 액세스, https://www.kudan.io/blog/a-technical-deep-dive-into-visual-data-driven-amrs-powered-by-kdvisual-and-nvidia-isaac-perceptor/
2. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2311.00626v2
3. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox
4. AMR navigation using Isaac ROS VSLAM and Nvblox with Intel Realsense camera, 8월 9, 2025에 액세스, https://www.einfochips.com/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
5. Isaac ROS (Robot Operating System) - NVIDIA Developer, 8월 9, 2025에 액세스, https://developer.nvidia.com/isaac/ros
6. NVIDIA ROS 2 Projects - ROS 2 Documentation: Humble documentation, 8월 9, 2025에 액세스, https://ftp.udx.icscoe.jp/ros/ros_docs_mirror/en/humble/Related-Projects/Nvidia-ROS2-Projects.html
7. NVIDIA ROS 2 Projects - ROS 2 Documentation: Humble documentation, 8월 9, 2025에 액세스, https://docs.ros.org/en/humble/Related-Projects/Nvidia-ROS2-Projects.html
8. nvidia-isaac-ros.github.io, 8월 9, 2025에 액세스, [https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html#:~:text=Nvblox%20is%20a%20library%20for,for%20use%20in%20path%20planning.](https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html#:~:text=Nvblox is a library for,for use in path planning.)
9. Nvblox - SoftwareOne Marketplace, 8월 9, 2025에 액세스, https://platform.softwareone.com/product/nvblox/PCP-2427-1474
10. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
11. Isaac ROS Nvblox - isaac_ros_docs documentation, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
12. [2311.00626] nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2311.00626
13. AMR Navigation Using Isaac ROS VSLAM and Nvblox with Intel ..., 8월 9, 2025에 액세스, https://www.einfochips.com/blog/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
14. NVIDIA-Isaac-ROS-Nvblox/docs/topics-and-services.md at main - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/topics-and-services.md
15. Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox
16. How to run nvblox sample with isaac_sim-2022.2.0? - Isaac ROS, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/how-to-run-nvblox-sample-with-isaac-sim-2022-2-0/237806
17. Isaac ROS March update, vision based navigation - ROS Discourse - Open Robotics, 8월 9, 2025에 액세스, https://discourse.openrobotics.org/t/isaac-ros-march-update-vision-based-navigation/24816
18. Repo for the modified NvBlox package for mapping in ROS 2. - GitHub, 8월 9, 2025에 액세스, https://github.com/arplaboratory/nvblox
19. Signed distance function - Wikipedia, 8월 9, 2025에 액세스, https://en.wikipedia.org/wiki/Signed_distance_function
20. Truncated Signed Distance Function | by Simsangcheol | Medium, 8월 9, 2025에 액세스, https://medium.com/@sim30217/truncated-signed-distance-function-f765a0f1d432
21. Understanding Real Time 3D Reconstruction and KinectFusion - ITNEXT, 8월 9, 2025에 액세스, https://itnext.io/understanding-real-time-3d-reconstruction-and-kinectfusion-33d61d1cd402
22. DFusion: Denoised TSDF Fusion of Multiple Depth Maps with Sensor Pose Noises - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/1424-8220/22/4/1631
23. Truncated Signed Distance Fields Applied To Robotics - DiVA portal, 8월 9, 2025에 액세스, https://www.diva-portal.org/smash/get/diva2:1136113/FULLTEXT01.pdf
24. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/iros_2017_voxblox.pdf
25. When creating a TSDF from point cloud data, how do you determine the nearest "surface"?, 8월 9, 2025에 액세스, https://www.reddit.com/r/computervision/comments/1bcim0h/when_creating_a_tsdf_from_point_cloud_data_how_do/
26. Technical Details - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
27. Signed Distance Fields: A Natural ... - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/rss_2016_workshop.pdf
28. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Research Collection, 8월 9, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/128029/eth-50477-01.pdf
29. How Does ESDF Generation Work? - voxblox documentation - Read the Docs, 8월 9, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/pages/How-Does-ESDF-Generation-Work.html
30. Use Nvblox with Nav2 Rolling - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/use-nvblox-with-nav2-rolling/271374
31. Ros-humble-isaac-ros-nvblox and isaac-ros-nvblox packages have **hard** dependency on Nvidia driver 560? - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/ros-humble-isaac-ros-nvblox-and-isaac-ros-nvblox-packages-have-hard-dependency-on-nvidia-driver-560/340693
32. README.md - nvidia-isaac/nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/blob/public/README.md
33. Can not build 'nvblox ros 'issac ros nvblox - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/can-not-build-nvblox-ros-issac-ros-nvblox/300565
34. Learning NVidia Isaac ROS2 - Part 1 - Dev Setup | by RoboFoundry - Medium, 8월 9, 2025에 액세스, https://robofoundry.medium.com/learning-nvidia-isaac-ros2-part-1-dev-setup-0e54a3325082
35. isaac_ros_nvblox - isaac_ros_docs documentation, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/isaac_ros_nvblox/index.html
36. Install the Isaac ROS | Seeed Studio Wiki, 8월 9, 2025에 액세스, https://wiki.seeedstudio.com/install_isaacros/
37. NVIDIA-Isaac-ROS-Nvblox/docs/parameters.md at main / Tinker ..., 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/parameters.md
38. Changing launch parameters for nvblox realsense example - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/changing-launch-parameters-for-nvblox-realsense-example/336623
39. Nvidia Isaac ROS Nvblox, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/nvidia-isaac-ros-nvblox/311053
40. Using vSLAM + NVBLOX + Robot Localization - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/using-vslam-nvblox-robot-localization/336509
41. NVIDIA-Isaac-ROS-Nvblox/nvblox_performance_measurement/README.md at main, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/nvblox_performance_measurement/README.md
42. Isaac ROS VSLAM and Nvblox - how to change internal parameters, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/isaac-ros-vslam-and-nvblox-how-to-change-internal-parameters/306635
43. nvblox/release-notes.md at public - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/blob/public/release-notes.md
44. What's the realistic maximum size this can run on an Orin? / Issue #96 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/96
45. Octomap comparison. / Issue #28 / ethz-asl/voxgraph - GitHub, 8월 9, 2025에 액세스, https://github.com/ethz-asl/voxgraph/issues/28
46. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/1424-8220/22/3/1296
47. Isaac ROS Visual SLAM - isaac_ros_docs documentation, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_visual_slam/index.html
48. Difference between vslam launched using isaac_ros_examples v/s isaac_ros_visual_slam - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/difference-between-vslam-launched-using-isaac-ros-examples-v-s-isaac-ros-visual-slam/333663
49. [2007.11898] ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2007.11898
50. UZ-SLAMLab/ORB_SLAM3: ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - GitHub, 8월 9, 2025에 액세스, https://github.com/UZ-SLAMLab/ORB_SLAM3
51. Does ORB-Slam3 perform better than ORB-Slam2? : r/robotics - Reddit, 8월 9, 2025에 액세스, https://www.reddit.com/r/robotics/comments/18ne4f5/does_orbslam3_perform_better_than_orbslam2/
52. NVIDIA-Isaac-ROS-Nvblox/docs/tutorial-realsense-record.md at main - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/tutorial-realsense-record.md
53. Discussion - Nav2 Introduction (Making a Mobile Robot Pt 16), 8월 9, 2025에 액세스, https://discourse.articulatedrobotics.xyz/t/discussion-nav2-introduction-making-a-mobile-robot-pt-16/199
54. NVIDIA-Isaac-ROS-Nvblox/docs/troubleshooting-nvblox-realsense.md at main - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/troubleshooting-nvblox-realsense.md
55. Isaac ROS release-3.1: Dependency Issues with nvblox After Restarting Docker Container, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/isaac-ros-release-3-1-dependency-issues-with-nvblox-after-restarting-docker-container/316644
56. Isaac ROS VSLAM. NVBLOX usage with NAV2 - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/isaac-ros-vslam-nvblox-usage-with-nav2/303717
57. Tag: Isaac ROS | NVIDIA Technical Blog, 8월 9, 2025에 액세스, https://developer.nvidia.com/blog/tag/isaac-ros/
58. CES 2025 - Isaac ROS 3.2 and platform updates - General, 8월 9, 2025에 액세스, https://discourse.ros.org/t/ces-2025-isaac-ros-3-2-and-platform-updates/41444
59. Blogs - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/blog/index.html


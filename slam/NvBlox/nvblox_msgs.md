# nvblox_msgs에 대한 기술적 고찰

## 1. GPU 가속 3D 재구성과 nvblox_msgs의 등장

### 1.1 로봇 인식 기술의 패러다임 전환

자율 로봇 기술이 발전함에 따라, 로봇이 주변 환경을 정확하고 신속하게 인식하고 이해하는 능력은 핵심적인 요구사항이 되었다. 특히 자율이동로봇(AMR)이나 로봇 팔(manipulator)과 같은 시스템이 복잡하고 동적인 환경에서 안전하고 효율적으로 임무를 수행하기 위해서는, 주변 공간에 대한 고밀도의 3차원 체적 맵(volumetric map)이 필수적이다.1 전통적인 3D 매핑 기술들은 주로 중앙 처리 장치(CPU)의 연산 능력에 의존해왔다. 이러한 CPU 기반 시스템들은 계산량의 한계로 인해 실시간으로 처리할 수 있는 맵의 해상도나 규모에 제약을 가질 수밖에 없었다.1 이는 로봇이 빠르게 변화하는 환경에 즉각적으로 대응하는 것을 어렵게 만드는 근본적인 한계로 작용했다.

이러한 기술적 간극을 극복하기 위해 그래픽 처리 장치(GPU)의 병렬 처리 능력을 활용하려는 시도가 이어졌고, 이는 로봇 인식 기술의 패러다임 전환을 가져왔다. GPU는 수천 개의 코어를 활용하여 방대한 양의 데이터를 동시에 처리할 수 있으므로, 3D 센서로부터 들어오는 깊이 정보를 실시간으로 융합하고 복잡한 3차원 구조를 재구성하는 데 이상적인 하드웨어이다.

### 1.2 NVIDIA nvblox의 역할

이러한 패러다임 전환의 중심에 NVIDIA의 `nvblox` 라이브러리가 있다. `nvblox`는 로봇 응용, 특히 경로 계획을 목표로 설계된 GPU 가속 체적 매핑 라이브러리다.1

`nvblox`의 핵심적인 기여는 기존 CPU 기반 시스템에서 병목 현상을 일으켰던 두 가지 핵심 연산, 즉 TSDF(Truncated Signed Distance Field) 기반 표면 재구성과 ESDF(Euclidean Signed Distance Field) 기반 거리장 계산을 CUDA를 통해 극적으로 가속화한 데에 있다.1

TSDF는 센서로부터 측정된 깊이 정보를 3차원 복셀 그리드에 점진적으로 융합하여 노이즈에 강하고 정밀한 표면을 표현하는 방식이다.6 ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 유클리드 거리를 계산한 거리장으로, 로봇이 충돌 없이 안전한 경로를 계획하는 데 결정적인 정보를 제공한다.1

`nvblox`는 대표적인 CPU 기반 선행 연구인 `Voxblox`와 비교했을 때, TSDF 재구성에서 최대 177배, ESDF 계산에서 최대 31배의 압도적인 성능 향상을 달성했다.1 이러한 성능 향상은 로봇이 더 높은 해상도의 맵을 실시간으로 생성하고 유지하며, 더 빠르고 정확하게 환경 변화에 대응할 수 있음을 의미한다.

### 1.3 `nvblox_msgs`의 핵심적 역할

`nvblox` 라이브러리가 GPU 상에서 아무리 빠른 속도로 정교한 3D 맵을 생성하더라도, 그 결과물이 로봇 운영 체제(ROS) 생태계의 다른 구성 요소들과 효율적으로 통신할 수 없다면 그 가치는 제한될 수밖에 없다. 예를 들어, `nvblox`가 생성한 ESDF 맵은 경로 계획기(예: Nav2)로 전달되어야 하고, 재구성된 3D 메시는 시각화 도구(예: RViz)에 표시되어야 한다. 이처럼 서로 다른 프로세스 간의 데이터 교환을 위해 표준화된 인터페이스가 필요하며, 바로 이 지점에서 `nvblox_msgs` 패키지가 핵심적인 역할을 수행한다.

`nvblox_msgs`는 `nvblox`의 강력한 GPU 연산 결과를 ROS 2 생태계의 다른 노드들과 주고받기 위해 특별히 설계된 메시지(`msg`) 및 서비스(`srv`) 정의의 집합이다.8 이는 단순히 데이터를 담는 컨테이너를 넘어, 고성능 연산을 수행하는 `nvblox` 코어 라이브러리와 ROS라는 응용 계층을 명확하게 분리하는 'API 계약(API Contract)'으로서 기능한다.

`nvblox`의 핵심 데이터 구조, 예를 들어 복셀 그리드나 메시 데이터는 최적의 성능을 위해 GPU 메모리 상에 존재한다.1 반면, ROS 2의 표준 메시징 시스템은 기본적으로 CPU 메모리 간의 데이터 직렬화 및 복사를 전제로 동작한다.10 따라서 `nvblox`의 연산 결과를 ROS 토픽으로 발행하기 위해서는 GPU 메모리에 있는 데이터를 CPU 메모리로 효율적으로 전송하고, ROS가 이해할 수 있는 표준화된 형식으로 재구성하는 과정이 필수적이다. `nvblox_msgs`는 바로 이 재구성된 데이터의 최종 '형태'를 정의한다. 예를 들어, `Mesh.msg`는 GPU에서 병렬 처리로 생성된 수많은 정점(vertex)과 삼각형(triangle) 정보를 ROS가 처리할 수 있는 배열 형태로 변환한 결과물의 구조를 명시한다. 이처럼 `nvblox_msgs`의 설계는 GPU-to-CPU 데이터 전송의 오버헤드와 ROS 생태계의 표준 메시징 방식 사이에서 최적의 균형점을 찾으려는 노력의 산물이며, 이는 전체 로봇 시스템의 실시간 성능에 직접적인 영향을 미치는 중요한 공학적 결정이라 할 수 있다.

## 2.  nvblox 아키텍처와 데이터 표현의 원리

`nvblox_msgs`의 구조를 이해하기 위해서는 먼저 그 메시지가 표현하고자 하는 대상, 즉 `nvblox` 라이브러리가 생성하는 데이터의 원리를 파악해야 한다. `nvblox`는 크게 TSDF 통합, ESDF 생성, 그리고 이들을 ROS 노드 내에서 관리하는 데이터 흐름의 세 가지 핵심 요소로 구성된다.

### 2.1  TSDF (Truncated Signed Distance Field) 통합

TSDF는 `nvblox` 맵 표현의 근간을 이룬다. 그 핵심 아이디어는 3차원 공간을 작은 정육면체 단위인 복셀(voxel)로 나누고, 각 복셀에 가장 가까운 표면까지의 부호화된 거리(signed distance) 정보를 저장하는 것이다.6 여기서 '부호화된'이란, 복셀이 표면보다 카메라에 가까우면 양수(+), 표면 뒤에 있으면 음수(-) 값을 가지는 것을 의미한다. 표면 자체는 거리 값이 0이 되는 지점들의 집합, 즉 영점 집합(zero-level set)으로 정의된다.6 '절단된(Truncated)'이라는 용어는 표면에서 일정 거리(`$\tau$`) 이상 떨어진 복셀들에는 최대/최소값(+1 또는 −1)을 할당하여 계산량을 줄이는 기법을 의미한다.

`nvblox`의 TSDF 통합 과정은 점진적 갱신(incremental update) 방식으로 이루어진다. 로봇의 RGB-D 카메라나 LiDAR로부터 새로운 깊이 프레임이 입력될 때마다, `nvblox`는 각 픽셀(또는 포인트)을 3차원 공간으로 투영한다. 그리고 이 광선(ray)이 통과하는 경로에 있는 복셀들의 TSDF 값을 갱신한다. 이때, 기존에 저장된 TSDF 값과 새로 측정된 값을 단순히 덮어쓰는 것이 아니라, 가중 평균(weighted average) 방식을 사용하여 융합한다.11

수식으로 표현하면, n번째 프레임까지 누적된 복셀의 TSDF 값 `$D_{n-1}(v)$`와 가중치 `$W_{n-1}(v)$`가 있을 때, 새로운 측정값 `$d_n(v)$`와 그에 해당하는 가중치 `$w_n(v)$`가 주어지면, 갱신된 TSDF 값 `$D_n(v)$`와 가중치 `$W_n(v)$`는 다음과 같이 계산된다:
$$
W_n(v) = W_{n-1}(v) + w_n(v)
$$

$$
D_n(v) = \frac{W_{n-1}(v)D_{n-1}(v) + w_n(v)d_n(v)}{W_{n-1}(v) + w_n(v)}
$$

이 가중 평균 방식은 여러 프레임에 걸쳐 일관되게 측정되는 표면에 대해서는 가중치를 높여 신뢰도를 증가시키고, 일시적으로 나타나는 센서 노이즈와 같은 이상치(outlier)의 영향은 자연스럽게 감소시키는 강력한 필터링 효과를 제공한다.13

여기서 새로운 측정의 가중치 `$w_n(v)$`는 센서의 불확실성 모델에 기반하여 결정된다. 일반적으로 깊이 센서는 측정 거리가 멀어질수록, 그리고 센서의 광선이 표면에 비스듬하게 입사할수록(즉, 입사각이 클수록) 측정 오차가 커지는 경향이 있다. `nvblox`는 이러한 센서의 물리적 특성을 모델링하여, 신뢰도가 낮은 측정값에는 낮은 가중치를 부여함으로써 재구성된 맵의 전반적인 품질과 강인함(robustness)을 향상시킨다.12

### 2.2  ESDF (Euclidean Signed Distance Field) 생성

TSDF가 정밀한 표면 재구성에 탁월한 반면, 로봇의 경로 계획에는 다소 부적합하다. TSDF 값은 표면 근처의 제한된 영역에서만 유효한 거리 정보를 담고 있기 때문이다. 경로 계획기는 맵 상의 임의의 지점에서 가장 가까운 장애물까지의 실제 유클리드 거리를 알아야 충돌 없이 안전하고 효율적인 경로를 생성할 수 있다. 이 역할을 수행하는 것이 바로 ESDF다.1

`nvblox`는 TSDF 맵으로부터 ESDF 맵을 생성하는 과정을 GPU 상에서 병렬로 처리하여 극적인 성능 향상을 이루었다. 이 과정은 CPU 기반의 선행 연구인 `Voxblox`의 아이디어를 계승하지만, 구현 방식에서 근본적인 차이를 보인다. `Voxblox`는 두 개의 큐(queue), 즉 'raise' 큐와 'lower' 큐를 사용하여 변경 사항을 파면(wavefront)처럼 순차적으로 전파하는 방식을 사용한다.16 반면, `nvblox`는 이 과정을 대규모 병렬 처리에 적합하도록 재설계했다.

`nvblox`의 ESDF 생성 알고리즘은 다음과 같은 단계로 구성된다 1:

1. **블록 할당 (Allocate Blocks)**: TSDF 맵에서 변경이 발생한 복셀 블록들을 식별한다. ESDF 맵에도 이에 대응하는 블록들을 할당한다.
2. **사이트 마킹 (Mark Sites)**: TSDF 복셀들 중에서 표면 경계에 위치한 복셀들, 즉 부호가 바뀌는 지점(zero-crossing)에 있는 복셀들을 '사이트(site)'로 지정한다. 이 사이트들이 거리 전파의 시작점이 된다.
3. **무효 영역 정리 (Clear Invalid)**: 이전에 사이트였던 복셀이 더 이상 사이트가 아니게 된 경우(예: 장애물이 사라진 경우), 해당 사이트를 부모로 삼고 있던 다른 복셀들의 거리 정보를 '무효화'한다. 이 복셀들은 거리를 재계산해야 한다.
4. **거리 전파 (Propagate Distances)**: 사이트로부터 주변 복셀들로 유클리드 거리를 병렬적으로 전파한다. 각 복셀은 자신에게 거리를 전파해 준 가장 가까운 사이트를 자신의 '부모'로 기록한다. 이 과정은 모든 복셀이 가장 가까운 사이트까지의 거리를 가질 때까지 반복적으로 수행된다.

이 모든 과정이 CUDA 커널을 통해 수많은 복셀에 대해 동시에 수행되므로, `Voxblox`의 순차적인 큐 처리 방식과는 비교할 수 없는 속도를 달성할 수 있다.1

특히 `nvblox`의 ESDF 생성 알고리즘은 '점진적(incremental)' 계산에 고도로 최적화되어 있다. 로봇이 움직이며 새로운 센서 데이터를 획득하면, 전체 맵의 ESDF를 처음부터 다시 계산하는 것이 아니라, TSDF가 변경된 영역과 그 주변부만 효율적으로 업데이트한다.1

`nvblox`는 TSDF가 업데이트된 복셀 블록의 목록을 내부적으로 추적하고 1, ESDF 업데이트 시 이 블록 목록을 기반으로 영향을 받는 영역만을 재계산 대상으로 삼는다. 이러한 점진적 업데이트 방식 덕분에 `nvblox`는 실시간으로 맵을 유지할 수 있으며, `nvblox_msgs`를 통해 발행되는 `map_slice`나 `mesh` 같은 토픽들은 전체 맵의 스냅샷이 아닌, 이전 상태로부터 변경된 부분에 대한 스트리밍 업데이트의 형태를 띠게 된다. 이는 `nvblox_rviz_plugin`과 같은 구독자 노드가 이러한 스트리밍 데이터를 효율적으로 처리하도록 설계된 이유를 설명해준다.8

### 2.3  `nvblox_node`의 데이터 흐름

`nvblox_node`는 `nvblox` C++ 라이브러리를 ROS 2 노드로 감싸, ROS 생태계와의 상호작용을 담당하는 핵심 실행 파일이다. 이 노드의 데이터 흐름은 크게 입력(구독), 내부 처리, 출력(발행)의 세 단계로 나눌 수 있다.

- **입력 (Subscriptions)**: `nvblox_node`는 로봇의 센서 및 위치 추정 시스템으로부터 데이터를 입력받는다. 주요 구독 토픽은 다음과 같다 20:

  - `depth/image` (`sensor_msgs/Image`): 깊이 카메라로부터의 깊이 이미지.
  - `depth/camera_info` (`sensor_msgs/CameraInfo`): 깊이 카메라의 내부 파라미터(intrinsic parameters).
  - `color/image` (`sensor_msgs/Image`): (선택 사항) 재구성된 메시를 채색하기 위한 컬러 이미지.
  - `pose` (`geometry_msgs/PoseStamped`) 또는 TF: 로봇의 위치와 자세. 이는 VSLAM(예: `isaac_ros_visual_slam`)과 같은 외부 위치 추정 노드로부터 제공된다.8
  - `mask/image` (`sensor_msgs/Image`): (동적 객체 처리 모드 시) 사람이나 다른 동적 객체를 분리하기 위한 시맨틱 세그멘테이션 마스크.

- **센서 데이터 동기화**: `nvblox`는 정확한 3D 재구성을 위해 서로 다른 센서(깊이, 컬러, 마스크 등)로부터 들어오는 데이터의 타임스탬프를 정밀하게 동기화해야 한다. 이를 위해 `nvblox_node`는 내부적으로 ROS의 `message_filters` 라이브러리를 사용하여 근사 시간 동기화(approximate time synchronization) 정책을 적용한다.21 NVIDIA Isaac ROS 프레임워크에서는 이 과정을 NITROS(NVIDIA Isaac Transport for ROS)라는 최적화된 전송 계층을 통해 더욱 가속화하여 데이터 복사를 최소화하고 처리 지연을 줄인다.10

- **내부 처리**: 동기화된 센서 데이터와 포즈 정보는 `nvblox` 코어 라이브러리의 `Mapper` 클래스 인스턴스로 전달된다.24

  `Mapper` 클래스는 GPU 상에서 TSDF 통합, ESDF 업데이트, 메시 생성 등 앞서 설명한 모든 핵심 연산을 수행하는 역할을 담당한다.

- **출력 (Publications)**: `Mapper` 클래스에 의해 GPU 메모리 상에서 계산된 결과물들은 CPU 메모리로 복사된 후, `nvblox_msgs`에 정의된 메시지 타입으로 변환(직렬화)되어 ROS 토픽으로 발행된다. 이를 통해 다른 ROS 노드들이 `nvblox`의 결과물을 활용할 수 있게 된다. 대표적인 발행 토픽은 다음과 같다 20:

  - `~/mesh` (`nvblox_msgs/Mesh`): RViz 시각화를 위한 3D 메시.
  - `~/map_slice` (`nvblox_msgs/DistanceMapSlice`): Nav2 경로 계획을 위한 2D ESDF 슬라이스.
  - `~/esdf_pointcloud` (`sensor_msgs/PointCloud2`): ESDF를 포인트 클라우드 형태로 시각화하기 위한 토픽.

이러한 데이터 흐름을 통해 `nvblox_node`는 고성능 GPU 연산과 표준화된 ROS 2 통신을 유기적으로 결합하여, 실시간 3D 환경 인식이라는 복잡한 작업을 안정적으로 수행한다.

## 3.  핵심 메시지 타입 상세 분석

`nvblox_msgs` 패키지는 `nvblox`의 다양한 결과물을 ROS 생태계에 전달하기 위한 여러 메시지 타입을 정의한다. 이 장에서는 그 중 가장 핵심적인 `Mesh`와 `DistanceMapSlice` 메시지를 중심으로 그 구조와 역할을 심층적으로 분석한다.

**표 1: `nvblox_msgs` 핵심 인터페이스 요약**

| 인터페이스 이름    | 타입  | 주요 데이터 필드                                     | 주 사용 목적                                      | 관련 토픽/서비스                         |
| ------------------ | ----- | ---------------------------------------------------- | ------------------------------------------------- | ---------------------------------------- |
| `Mesh`             | `msg` | `block_size`, `blocks` (vertices, triangles, colors) | RViz에서의 3D 메시 시각화                         | `~/mesh`                                 |
| `DistanceMapSlice` | `msg` | `origin`, `resolution`, `width`, `height`, `data`    | Nav2를 위한 2D 비용 지도(costmap) 제공            | `~/map_slice`, `~/human_map_slice`       |
| `FilePath`         | `srv` | `file_path` (request), `success` (response)          | 생성된 맵의 저장 및 로딩 (영속성)                 | `~/save_map`, `~/load_map`, `~/save_ply` |
| `EsdfAndGradients` | `srv` | `aabb_min_m`, `aabb_size_m` (request)                | 특정 영역에 대한 ESDF 및 그래디언트 온디맨드 쿼리 | `~/get_esdf_and_gradients`               |

### 3.1  `nvblox_msgs/Mesh`: 3D 시각화를 위한 메시지 구조

`nvblox_msgs/Mesh` 메시지는 `nvblox`가 TSDF로부터 추출한 3D 표면 메시를 표현하기 위한 커스텀 메시지 타입이다. 이 메시지는 주로 RViz에서의 실시간 3D 시각화를 위해 사용된다.6

- **목적 및 필요성**: `nvblox`가 생성하는 메시는 수십만 개 이상의 정점과 면으로 구성될 수 있어 데이터 양이 매우 크다. 표준 메시지 타입인 `visualization_msgs/Marker` (특히 `TRIANGLE_LIST` 타입)를 사용할 수도 있지만, 대규모 메시 데이터를 효율적으로 스트리밍하고, 특히 `nvblox`의 블록 기반 증분 업데이트(incremental update)를 직접적으로 반영하기에는 한계가 있다. 따라서 `nvblox`는 자체적인 `Mesh` 메시지 타입을 정의하고, 이를 전문적으로 렌더링하는 `nvblox_rviz_plugin`을 함께 제공하는 방식을 택했다.8

- **추정 데이터 구조**: `isaac_ros_nvblox` 리포지토리의 소스 코드와 사용 예를 종합해볼 때 20, 

  `Mesh.msg`의 구조는 다음과 같이 유추할 수 있다. 이는 `nvblox`의 내부 데이터 구조인 복셀 블록(VoxelBlock)을 직접적으로 반영한다.

  ```
  # nvblox_msgs/msg/Mesh.msg
  
  std_msgs/Header header
  float32 block_size          # 각 메시 블록의 한 변 길이 (미터)
  nvblox_msgs/MeshBlock blocks # 업데이트된 메시 블록들의 배열
  ```

  여기서 `MeshBlock.msg`는 개별 블록의 메시 정보를 담으며, 다음과 같은 구조를 가질 것으로 예상된다.

  ```
  # nvblox_msgs/msg/MeshBlock.msg
  
  # 블록의 원점 좌표 (블록 인덱스 * block_size)
  # block_index * block_size (implicitly defined by its position in some spatial hash or array)
  # 혹은 명시적으로 geometry_msgs/Point origin
  
  geometry_msgs/Point vertices  # 정점 좌표 배열
  int32 triangles               # 삼각형을 구성하는 정점 인덱스 배열 (3개씩 묶음)
  std_msgs/ColorRGBA colors     # (선택 사항) 각 정점의 색상 정보
  geometry_msgs/Point normals   # (선택 사항) 각 정점의 법선 벡터
  ```

- **데이터 특징 및 `nvblox_rviz_plugin`과의 연동**: 이 구조의 가장 큰 특징은 전체 메시를 하나의 거대한 배열로 보내는 대신, `blocks`라는 배열을 통해 변경되거나 새로 생성된 블록 단위로 메시 정보를 전송한다는 점이다. 이는 데이터 전송량을 최소화하고 실시간성을 극대화하기 위한 핵심적인 설계 결정이다.8

  `nvblox_rviz_plugin`은 이 `Mesh` 메시지를 구독하여, 각 `MeshBlock`을 독립적인 렌더링 단위로 처리한다. 이 플러그인은 GPU의 Geometry Shader와 같은 고급 그래픽스 기능을 활용하여 수많은 블록을 효율적으로 렌더링하도록 설계되었으며, 해당 기능이 지원되지 않는 환경에서는 성능이 저하될 수 있는 폴백(fallback) 렌더링 방식을 사용한다.28

### 3.2  `nvblox_msgs/DistanceMapSlice`: 2D 내비게이션을 위한 ESDF 슬라이스

`nvblox_msgs/DistanceMapSlice` 메시지는 `nvblox`의 핵심 출력 중 하나로, 3차원 ESDF 공간의 특정 높이 구간을 잘라내어 2차원 거리 지도로 만든 것이다. 이 메시지는 주로 ROS 2의 표준 내비게이션 스택인 Nav2와 연동하여 실시간 로컬 비용 지도(local costmap)를 제공하는 데 사용된다.2

- **목적 및 필요성**: 대부분의 지상 이동 로봇(AMR)은 2D 평면상에서의 내비게이션을 기본으로 한다. 따라서 완전한 3D ESDF 맵 전체를 경로 계획기에 제공하는 것은 불필요한 계산 부하를 유발할 수 있다. `DistanceMapSlice`는 로봇의 주행에 직접적인 영향을 미치는 높이 구간(예: 바닥으로부터 0.1m ~ 1.5m)의 장애물 정보만을 추출하여 2D 그리드 맵 형태로 제공함으로써, Nav2와 같은 2D 경로 계획기가 효율적으로 사용할 수 있도록 한다.

- **추정 데이터 구조**: 이 메시지는 표준 `nav_msgs/OccupancyGrid`와 유사한 구조를 가지지만, 셀의 값이 점유 확률이 아닌 '거리'라는 점에서 차이가 있다. 관련 문서들을 바탕으로 20 구조를 유추하면 다음과 같다.

  ```
  # nvblox_msgs/msg/DistanceMapSlice.msg
  
  std_msgs/Header header
  geometry_msgs/Pose origin     # 2D 맵 원점의 3D 공간상 위치 및 방향
  float32 resolution            # 맵의 해상도 (미터/셀)
  uint32 width                  # 맵의 너비 (셀 개수)
  uint32 height                 # 맵의 높이 (셀 개수)
  float32 data                # 거리 값 배열 (row-major order)
  ```

- **데이터 레이아웃 및 `nvblox_nav2` 플러그인**: `data` 필드는 1차원 배열로 직렬화된 거리 값들을 담고 있다. 이 배열의 i번째 요소는 2D 그리드 상의 좌표 (x,y) = $(i \pmod{width}, i / width)$에 해당한다. 각 셀의 값은 해당 위치에서 가장 가까운 장애물까지의 유클리드 거리(미터 단위)를 나타낸다. 장애물 내부는 음수, 자유 공간은 양수 값을 가진다. 아직 관측되지 않은 영역의 셀들은 `-1000.0`과 같은 특정한 값으로 초기화된다.30

  `nvblox_nav2` 패키지에 포함된 커스텀 비용 지도 플러그인은 이 `DistanceMapSlice` 메시지를 구독하여, 거리 값을 Nav2가 이해할 수 있는 비용(cost) 값으로 변환한다. 예를 들어, 거리가 0에 가까울수록(장애물에 가까울수록) 높은 비용을, 거리가 멀수록 낮은 비용을 할당하는 방식이다. 이를 통해 로봇은 `nvblox`가 실시간으로 감지하는 장애물을 효과적으로 회피하며 주행할 수 있다.2

### 3.3  동적 객체 처리를 위한 메시지 확장

`nvblox`는 정적 환경뿐만 아니라 사람과 같이 움직이는 동적 객체가 있는 환경에서도 강인한 성능을 발휘하도록 설계되었다.6 이를 위해 외부의 DNN 기반 시맨틱 세그멘테이션 노드(예: `isaac_ros_unet`)로부터 `mask/image` 토픽을 입력받아, 깊이 이미지에서 사람에 해당하는 영역을 분리해낸다.20

분리된 동적 객체 정보는 별도의 맵 레이어에서 관리되며, 이로부터 생성된 ESDF 슬라이스는 특화된 토픽으로 발행된다.

- `~/human_map_slice`: 오직 사람(동적 객체)만을 장애물로 간주하여 생성한 ESDF 슬라이스.
- `~/combined_map_slice`: 정적 환경 맵과 동적 객체 맵을 결합하여, 두 종류의 장애물까지의 최소 거리를 나타내는 ESDF 슬라이스.

여기서 주목할 점은, 이들 토픽이 새로운 메시지 타입을 정의하지 않고 기존의 `nvblox_msgs/DistanceMapSlice` 타입을 그대로 재사용한다는 것이다.20 이는 매우 효율적이고 확장성 있는 설계 방식으로, '데이터의 구조'와 '데이터의 의미'를 분리하는 ROS의 설계 철학을 잘 보여준다. 

`DistanceMapSlice.msg`는 '2D 그리드 형태의 거리 값 배열'이라는 데이터의 구조(structure)를 정의하는 역할을 한다. 그리고 이 구조에 담긴 데이터가 '정적 장애물에 대한 것인지', '사람에 대한 것인지', 아니면 '둘을 합친 것인지'와 같은 의미(semantics)는 메시지를 발행하는 토픽의 이름으로 구분된다. 만약 각기 다른 의미를 위해 `StaticMapSlice.msg`, `HumanMapSlice.msg`와 같이 별도의 메시지 타입을 정의했다면, 이는 불필요한 코드 중복을 야기하고 패키지 간의 의존성을 복잡하게 만들었을 것이다. 이처럼 메시지 타입을 재사용하는 방식은 `nvblox_msgs`가 간결하면서도 다양한 활용 사례에 유연하게 대응할 수 있도록 하는 핵심적인 설계 원리이다.

## 4.  핵심 서비스 타입 상세 분석

`nvblox_msgs`는 지속적으로 스트리밍되는 토픽 외에도, 클라이언트의 요청에 따라 특정 작업을 수행하고 응답하는 서비스(service) 인터페이스를 제공한다. 이는 맵 데이터의 영속성을 관리하거나, 특정 영역의 정보를 온디맨드(on-demand) 방식으로 쿼리하는 데 사용된다.

### 4.1  `nvblox_msgs/srv/FilePath`: 맵 데이터의 영속성 관리

`FilePath` 서비스는 `nvblox` 노드의 메모리 상에 구축된 3D 맵 데이터를 파일 시스템에 저장하거나, 저장된 파일로부터 맵을 다시 불러오는 기능을 제공하는 핵심적인 인터페이스다.33 이는 로봇이 작업을 종료한 후에도 맵 정보를 보존하거나, 사전에 구축된 맵을 불러와 로컬라이제이션 및 내비게이션을 수행하는 등 다양한 시나리오에서 필수적이다.

- **목적 및 역할**: 로봇이 넓은 환경을 탐사하며 생성한 맵은 일회성으로 사용되고 버려지기에는 매우 가치 있는 자산이다. `FilePath` 서비스는 `save_map`, `save_ply`, `load_map`이라는 세 가지 구체적인 서비스 서버를 통해 이러한 맵 데이터의 영속성을 보장한다.

- **서비스 정의**: ROS의 서비스는 요청(request)과 응답(response) 부분으로 나뉘며, `---`로 구분된다. `FilePath.srv`의 정의는 매우 간결하며, 파일 경로를 요청으로 받아 작업의 성공 여부를 응답으로 반환하는 구조를 가진다.34

  ```
  # nvblox_msgs/srv/FilePath.srv
  
  # Request
  string file_path  # 저장하거나 불러올 파일의 절대 경로
  
  ---
  
  # Response
  bool success      # 작업 성공 여부
  ```

- **사용법 및 저장 포맷**: 이 서비스는 `ros2 service call` 커맨드라인 도구를 통해 쉽게 호출할 수 있다. 이때 `file_path` 인자에는 반드시 상대 경로가 아닌 절대 경로를 지정해야 한다.36

  - `~/save_ply`: `nvblox_node`가 생성한 3D 메시를 PLY(Polygon File Format) 형식으로 저장한다. `.ply` 파일은 MeshLab이나 CloudCompare와 같은 표준 3D 뷰어에서 쉽게 열어볼 수 있어, 재구성된 맵의 품질을 시각적으로 확인하는 데 유용하다.33
  - `~/save_map`: `nvblox`의 모든 내부 상태를 포함하는 완전한 맵 데이터를 `.nvblx`라는 바이너리 파일로 직렬화하여 저장한다. 이 파일에는 TSDF, ESDF, Color, Mesh 등 모든 레이어의 정보가 포함되어 있어, 맵의 상태를 완벽하게 보존하고 복원할 수 있다.33
  - `~/load_map`: `.nvblx` 파일로부터 맵 데이터를 불러와 현재 `nvblox_node`의 맵 상태를 덮어쓴다. 이를 통해 이전에 저장된 맵 환경에서 작업을 재개할 수 있다.

- **한계점**: `nvblox`는 순수한 매핑(mapping) 라이브러리이며, SLAM(Simultaneous Localization and Mapping)의 위치 추정(localization) 기능은 포함하지 않는다. 따라서 `load_map` 서비스를 통해 기존 맵을 성공적으로 불러왔다 하더라도, 로봇이 그 맵 안에서 자신의 현재 위치를 자동으로 인식(relocalization)하지는 못한다. 이 기능은 `isaac_ros_visual_slam`과 같은 별도의 SLAM 시스템이 담당해야 할 역할이다.37

### 4.2  `nvblox_msgs/srv/EsdfAndGradients`: 동적 경로 계획을 위한 온디맨드 쿼리

`EsdfAndGradients` 서비스는 `nvblox`의 활용 범위를 단순한 2D 내비게이션을 넘어, 보다 정교한 3D 모션 플래닝으로 확장시키는 중요한 인터페이스다. 이 서비스는 클라이언트가 지정한 특정 관심 영역(Axis-Aligned Bounding Box, AABB)에 대한 ESDF 값과 그래디언트(gradient) 정보를 온디맨드 방식으로 요청하고 응답받을 수 있게 한다.38

- **목적 및 역할**: 로봇 팔과 같은 매니퓰레이터가 복잡한 환경에서 작업을 수행할 때, 전체 맵 정보를 실시간으로 스트리밍받는 것은 비효율적이다. 경로 계획기는 주로 팔의 작업 공간(workspace)이나 현재 계획 중인 경로 주변의 장애물 정보에만 관심이 있다. `EsdfAndGradients` 서비스는 이러한 요구에 부응하여, 필요한 영역의 정보만을 정밀하게 쿼리할 수 있는 수단을 제공한다.

- **서비스 정의 (주요 부분)**: 이 서비스는 요청 부분에 질의할 영역의 경계 상자(AABB)와 좌표계(frame_id) 등을 포함하며, 응답으로 해당 영역을 포함하는 복셀 그리드의 원점, 해상도, 그리고 ESDF 및 그래디언트 데이터를 반환한다.38

  ````
  # nvblox_msgs/srv/EsdfAndGradients.srv
  
  # Request
  bool update_esdf             # 서비스 호출 시 ESDF를 먼저 업데이트할지 여부
  bool use_aabb                # 아래의 AABB를 사용하여 영역을 제한할지 여부
  string frame_id              # AABB가 정의된 좌표계
  geometry_msgs/Point aabb_min_m # AABB의 최소 코너 좌표 (미터)
  geometry_msgs/Vector3 aabb_size_m # AABB의 크기 (미터)
  
  ---
  
  # Response
  bool success                 # 쿼리 성공 여부
  geometry_msgs/Point origin_m # 반환된 그리드의 원점 좌표
  float32 voxel_size_m         # 복셀 크기 (미터)
  std_msgs/Float32MultiArray esdf_and_gradients # ESDF 및 그래디언트 데이터
  ```
  ````

- **활용 사례**: 응답으로 받은 ESDF 값은 충돌 검사에 직접 사용될 수 있다. 특히 그래디언트 정보는 장애물로부터 멀어지는 방향을 나타내므로, CHOMP(Covariant Hamiltonian Optimization for Motion Planning)와 같은 최적화 기반의 경로 계획 알고리즘에서 충돌 회피를 위한 비용 함수(cost function)를 계산하는 데 매우 유용하게 활용될 수 있다.3

`FilePath` 서비스와 `EsdfAndGradients` 서비스는 `nvblox`의 두 가지 상이한 사용 패러다임을 명확하게 보여준다. `FilePath` 서비스는 AMR이 넓은 환경을 탐사하고 지도를 구축한 뒤, 이를 저장하고 나중에 다시 불러와 활용하는 '맵 구축 및 정적 활용(Build-then-Navigate)' 워크플로우에 적합하다.40 이 경우 데이터 통신은 비주기적이며 대용량의 데이터를 한 번에 처리한다. 반면, 

`EsdfAndGradients` 서비스는 매니퓰레이터가 실시간으로 변화하는 작업 공간 내에서 충돌을 회피하며 정밀한 작업을 수행해야 하는 '실시간 동적 쿼리(Real-time Dynamic Query)' 시나리오에 최적화되어 있다.42 전자가 서버(

`nvblox_node`)가 일방적으로 데이터를 발행(push)하는 토픽 방식과 유사한 패러다임의 연장선에 있다면, 후자는 클라이언트(경로 계획기)가 필요한 정보를 능동적으로 요청(pull)하는 방식으로, 각기 다른 애플리케이션의 데이터 소비 패턴에 유연하게 대응할 수 있도록 `nvblox_msgs`가 설계되었음을 보여준다.

## 제4장: 타 매핑 메시지 패키지와의 비교 고찰

`nvblox_msgs`의 설계 철학과 그 독창성을 깊이 이해하기 위해서는 ROS 생태계에서 널리 사용되는 다른 매핑 관련 메시지 패키지들과의 비교 분석이 필수적이다. 이를 통해 각 기술이 어떤 문제 해결에 중점을 두고 있으며, 어떤 장단점을 갖는지 명확히 파악할 수 있다. 본 장에서는 `octomap_msgs`와 `grid_map_msgs`를 중심으로 `nvblox_msgs`와의 비교 고찰을 수행한다.

**표 2: 매핑 메시지 패키지 비교**

| 기준            | `nvblox_msgs`                                                | `octomap_msgs`                                           | `grid_map_msgs`                                      |
| --------------- | ------------------------------------------------------------ | -------------------------------------------------------- | ---------------------------------------------------- |
| **데이터 표현** | 밀집 거리장 (Dense Distance Field - TSDF/ESDF)               | 희소 점유 격자 (Sparse Occupancy Grid)                   | 다중 레이어 2.5D 그리드 (Multi-layered 2.5D Grid)    |
| **핵심 메시지** | `Mesh`, `DistanceMapSlice`                                   | `Octomap`                                                | `GridMap`                                            |
| **주 사용처**   | 실시간 3D 재구성, 충돌 회피, 2D/3D 경로 계획                 | 대규모 환경 표현, 3D 탐사, 메모리 효율적 매핑            | 지형 분석, 실외 주행, 보행 로봇 발 디딤 계획         |
| **장점**        | GPU 가속, 고품질 표면, 경로 계획에 직접 활용 가능한 거리 정보 | 높은 메모리 효율성, 넓은 공간 표현 용이, 확률적 업데이트 | 다중 정보(고도, 법선 등) 표현, 지형 정보 처리에 특화 |
| **단점**        | 높은 메모리/GPU 자원 요구, 희소 환경 표현에 비효율적         | 표면 디테일 부족, 경로 계획 시 추가 계산 필요            | 완전 3D 구조 표현 불가, 수직 구조물 표현에 한계      |

### 4.1. `nvblox_msgs` vs. `octomap_msgs`

`nvblox_msgs`와 `octomap_msgs`는 3차원 공간을 표현한다는 공통점이 있지만, 그 근본적인 철학과 데이터 구조에서 큰 차이를 보인다.

- **데이터 표현 방식의 차이**: `nvblox_msgs`는 TSDF와 ESDF를 기반으로 하는 밀집 거리장(dense distance field)을 표현한다.6 이는 공간을 균일한 복셀로 나누고 각 복셀에 표면까지의 '거리'라는 연속적인 메트릭 정보를 저장하는 방식이다. 반면, 

  `octomap_msgs`는 옥트리(Octree) 자료구조를 사용하여 공간을 효율적으로 분할하고, 각 말단 노드(leaf node)에 '점유 확률(occupancy probability)'이라는 이산적인 상태 정보를 저장하는 희소 점유 격자(sparse occupancy grid)를 표현한다.44 옥트리는 비어있거나 동일한 상태의 넓은 공간을 단일 노드로 압축하여 표현하므로 메모리 효율성이 매우 뛰어나다.

- **정보의 종류와 활용**: `nvblox_msgs`가 전달하는 ESDF 정보는 각 지점의 안전 거리(clearance)를 직접적으로 나타내므로, 경로 계획기가 충돌 비용을 계산하는 데 즉시 활용될 수 있다. 이는 로봇이 장애물로부터 안전한 거리를 유지하며 부드러운 경로를 생성하는 데 유리하다. 반면, `octomap_msgs`가 전달하는 정보는 '점유/비점유/알수없음'의 세 가지 상태다. 경로 계획을 위해서는 이 정보를 바탕으로 별도의 거리 계산(distance transform)을 수행하거나, 단순히 점유된 셀을 통과하지 못하도록 하는 제약 조건으로 사용해야 한다.

- **철학적 차이**: 이러한 차이는 두 기술의 지향점에서 비롯된다. `nvblox_msgs`는 NVIDIA GPU의 막강한 병렬 처리 능력을 활용하여, 다소 많은 메모리를 사용하더라도 실시간으로 고품질의 밀집 3D 맵을 생성하고 경로 계획에 유용한 정보를 직접 제공하는 데 초점을 맞춘다. 반면, `octomap_msgs`는 CPU 환경에서도 효율적으로 동작하도록 설계되었으며, 메모리 사용량을 최소화하면서 넓은 공간을 표현하고 확률적으로 맵을 갱신하는 데 중점을 둔다.

### 4.2. `nvblox_msgs` vs. `grid_map_msgs`

`nvblox_msgs`와 `grid_map_msgs`는 로봇 내비게이션에 사용되는 맵 정보를 다룬다는 점에서 유사하지만, 표현하는 공간의 차원과 주 사용처에서 명확한 차이를 보인다.

- **차원의 차이**: `nvblox_msgs`는 근본적으로 3차원 볼류메트릭 데이터를 다룬다. `DistanceMapSlice` 메시지는 이 3D 데이터로부터 특정 높이 구간을 추출한 2D 투영(projection)에 해당한다.6 반면, 

  `grid_map_msgs`는 본질적으로 2.5D 맵을 표현하도록 설계되었다.47 이는 2D 그리드(

  x,y)의 각 셀이 고도(z), 표면 법선 벡터(normal vector), 색상, 마찰 계수 등 여러 '레이어(layer)'의 값을 동시에 가질 수 있음을 의미한다.

- **주 사용처의 차이**: 이러한 구조적 차이로 인해 두 메시지 패키지는 서로 다른 응용 분야에 강점을 보인다. `nvblox_msgs`는 벽, 가구, 사람 등 수직 구조물이 많은 실내 환경의 3D 구조를 정밀하게 재구성하고 장애물을 회피하는 데 주로 사용된다. `grid_map_msgs`는 지면의 고도 변화나 지형의 특성이 중요한 실외 주행 환경이나, 다리 달린 로봇이 발을 디딜 위치를 계획하는(foothold planning) 등의 시나리오에 더 적합하다.

- **상호 보완성**: 두 패키지는 경쟁 관계라기보다는 상호 보완적인 관계에 가깝다. 예를 들어, `nvblox`를 사용하여 실외 환경의 완전한 3D 메시를 생성한 뒤, 이 메시에서 지면에 해당하는 부분만을 추출하여 `grid_map`으로 변환하고, 이를 지형 분석 및 주행 가능 영역 판단에 활용하는 복합적인 파이프라인을 구축할 수 있다. 이는 각 기술의 장점을 결합하여 보다 정교한 로봇 인식 시스템을 구현하는 한 가지 방법이 될 수 있다.

결론적으로, `nvblox_msgs`는 GPU 가속을 기반으로 한 실시간 밀집 3D 거리장 표현에 특화된, 비교적 최신의 전문화된 메시지 패키지라 할 수 있다. 이는 기존의 범용적인 매핑 메시지들과는 명확히 구분되는 독자적인 영역을 구축하고 있으며, 고성능 3D 인식 및 경로 계획을 요구하는 현대 로보틱스 애플리케이션에 새로운 가능성을 제시한다.

## 결론: nvblox_msgs의 의의와 미래 전망

`nvblox_msgs` 패키지는 NVIDIA의 고성능 GPU 가속 3D 인식 기술과 방대한 ROS 2 로보틱스 생태계를 연결하는 핵심적인 교량 역할을 수행한다. 이는 단순히 몇 가지 메시지와 서비스를 정의하는 것을 넘어, 고성능 컴퓨팅과 표준 로봇 소프트웨어 플랫폼 간의 데이터 흐름을 정의하는 표준 프로토콜로서 깊은 의의를 지닌다. `nvblox_msgs`와 `nvblox_nav2`, `nvblox_rviz_plugin`과 같은 관련 플러그인들의 존재는 ROS 2 개발자들이 복잡한 CUDA 커널 프로그래밍이나 GPU 아키텍처에 대한 깊은 이해 없이도, 실시간 3D 매핑과 동적 장애물 회피가 가능한 고성능 내비게이션 스택을 비교적 손쉽게 구축할 수 있는 길을 열었다.2 이는 로보틱스 애플리케이션 개발의 진입 장벽을 낮추고, 더 많은 연구자와 개발자가 GPU 가속의 이점을 누릴 수 있게 함으로써 생태계 전체의 발전에 기여한다.

`nvblox_msgs`의 미래는 `nvblox` 라이브러리와 NVIDIA Isaac 플랫폼의 발전 방향과 밀접하게 연관되어 있다. 몇 가지 주요 전망을 예측해볼 수 있다.

첫째, **3D 경로 계획과의 통합 강화**이다. 현재 `DistanceMapSlice` 메시지는 주로 2D 내비게이션에 초점이 맞춰져 있다. 하지만 드론이나 매니퓰레이터와 같이 3차원 공간 전체를 활용하는 로봇 시스템에서는 완전한 3D ESDF 맵이 필수적이다. 현재는 `EsdfAndGradients` 서비스를 통해 제한적인 3D 쿼리가 가능하지만, 향후에는 대규모 3D ESDF 맵 자체를 효율적으로 전송하고 부분적으로 업데이트할 수 있는 새로운 메시지 타입이나 서비스가 등장하여, 3D 모션 플래닝과의 통합이 더욱 긴밀해질 것으로 예상된다.50

둘째, **시맨틱 정보의 풍부한 통합**이다. 현재 `nvblox`는 동적 객체 분리를 위해 이진(binary) 세그멘테이션 마스크를 입력으로 받는다. 미래에는 이를 넘어, '바닥', '벽', '테이블', '의자' 등 더 다양한 시맨틱 레이블을 포함하는 정보를 처리하고, 이를 메시지 형태로 출력하는 방향으로 발전할 수 있다.18 예를 들어, 

`SemanticMesh`나 `SemanticVoxelLayer`와 같은 새로운 메시지 타입은 각 기하학적 요소에 시맨틱 정보를 결합하여, 로봇이 "의자 옆으로 이동하라"와 같은 고수준의 명령을 이해하고 수행하는 데 필요한 풍부한 환경 정보를 제공할 수 있을 것이다.

마지막으로, **NVIDIA Isaac 플랫폼과의 연계 심화**이다. `nvblox`는 Isaac Sim, Isaac Manipulator(cuMotion), Isaac Perceptor 등 NVIDIA의 통합 로보틱스 플랫폼의 핵심 구성 요소이다.51 이들 플랫폼이 더욱 정교해지고 기능이 확장됨에 따라, 

`nvblox_msgs`는 이들 간의 원활한 데이터 교환을 위한 중추적인 역할을 담당하며 함께 진화할 것이다. 특히, 실시간 3D 맵 정보를 활용한 동적 충돌 회피 경로 계획을 위해 `nvblox`의 ESDF 맵을 `cuMotion` 모션 플래너와 직접 연동하는 것은 현재 로보틱스 커뮤니티에서 높은 관심을 받는 주제이며 42, 이를 위한 표준화된 메시지 및 서비스 인터페이스의 발전이 기대된다.

요컨대, `nvblox_msgs`는 GPU 기반 고속 3D 인식의 시대를 여는 중요한 기술적 이정표이며, 앞으로 로봇이 더욱 지능적으로 환경과 상호작용하는 미래를 구현하는 데 있어 그 역할과 중요성이 더욱 커질 것으로 전망된다.

#### **참고 자료**

1. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2311.00626v2
2. Towards Next-Gen Autonomous Mobile Robotics: A Technical Deep Dive into Visual-Data-Driven AMRs Powered by Kudan Visual SLAM and NVIDIA Isaac Perceptor, 8월 9, 2025에 액세스, https://www.kudan.io/blog/a-technical-deep-dive-into-visual-data-driven-amrs-powered-by-kdvisual-and-nvidia-isaac-perceptor/
3. Making Industrial Robots More Nimble With NVIDIA Isaac Manipulator and Vention MachineMotion AI, 8월 9, 2025에 액세스, https://developer.nvidia.com/blog/making-industrial-robots-more-nimble-with-nvidia-isaac-manipulator-and-vention-machinemotion-ai/
4. Performance - voxblox documentation - Read the Docs, 8월 9, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/pages/Performance.html
5. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox
6. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
7. nvblox/README.md at public / nvidia-isaac/nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/blob/public/README.md?plain=1
8. NVIDIA-ISAAC-ROS/isaac_ros_nvblox: NVIDIA-accelerated 3D scene reconstruction and Nav2 local costmap provider using nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox
9. Isaac ROS Nvblox - isaac_ros_docs documentation, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
10. CUDA with NITROS - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/concepts/nitros/cuda_with_nitros.html
11. Need help understanding TSDF algorithm from KinectFusion paper. : r/computervision, 8월 9, 2025에 액세스, https://www.reddit.com/r/computervision/comments/6f0oln/need_help_understanding_tsdf_algorithm_from/
12. PSDF Fusion: Probabilistic Signed Distance Function for On-the-fly 3D Data Fusion and Scene Reconstruction - Wei Dong, 8월 9, 2025에 액세스, https://dongwei.info/assets/pdf/psdf.pdf
13. A New Volumetric Fusion Strategy with Adaptive Weight Field for RGB-D Reconstruction, 8월 9, 2025에 액세스, https://www.mdpi.com/1424-8220/20/15/4330
14. Rendering the Directional TSDF for Tracking and Multi-Sensor Registration with Point-To-Plane Scale ICP, 8월 9, 2025에 액세스, https://www.ais.uni-bonn.de/papers/RAS_2023_Splietker.pdf
15. Improving 3D Reconstruction Through RGB-D Sensor Noise Modeling - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/1424-8220/25/3/950
16. How Does ESDF Generation Work? - voxblox documentation - Read the Docs, 8월 9, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/pages/How-Does-ESDF-Generation-Work.html
17. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/iros_2017_voxblox.pdf
18. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping | Request PDF, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/382991572_nvblox_GPU-Accelerated_Incremental_Signed_Distance_Field_Mapping
19. AMR navigation using Isaac ROS VSLAM and Nvblox with Intel Realsense camera, 8월 9, 2025에 액세스, https://www.einfochips.com/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
20. NVIDIA-Isaac-ROS-Nvblox/docs/topics-and-services.md at main - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/topics-and-services.md
21. isaac_ros_nvblox/nvblox_ros/include/nvblox_ros/nvblox_node.hpp at main / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_ros/include/nvblox_ros/nvblox_node.hpp
22. message_filters - ROS Wiki, 8월 9, 2025에 액세스, http://wiki.ros.org/message_filters
23. NVIDIA-ISAAC-ROS/isaac_ros_nova: ROS 2 support packages for Nova. - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nova
24. nvblox::Mapper Class Reference, 8월 9, 2025에 액세스, https://nvblox.readthedocs.io/en/public/classnvblox_1_1Mapper.html
25. nvblox/docs/pages/technical.md at public - GitHub, 8월 9, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/blob/public/docs/pages/technical.md
26. leggedrobotics/glimpse_nvblox_ros1: Static fork from Nvidia's Nvblox repo. Converted to ROS-1 and adapted for 3d-LiDAR generated depth images - GitHub, 8월 9, 2025에 액세스, https://github.com/leggedrobotics/glimpse_nvblox_ros1
27. nvblox_rviz_plugin - isaac_ros_docs documentation - NVIDIA Isaac ROS, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/nvblox_rviz_plugin/index.html
28. Nvblox Mesh Visualization Problem - Release 3.2 - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/nvblox-mesh-visualization-problem-release-3-2/321992
29. Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox
30. Nvidia Isaac ROS Nvblox, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/nvidia-isaac-ros-nvblox/311053
31. /nvblox_node/static_map_slice Unavailable - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/nvblox-node-static-map-slice-unavailable/302431
32. Isaac ROS Nvblox Costmap Plugin For Nav2 Implementation Problems, 8월 9, 2025에 액세스, https://robotics.stackexchange.com/questions/103816/isaac-ros-nvblox-costmap-plugin-for-nav2-implementation-problems
33. How build and save map 3D reconstruction to use with nvblox nav2 - Isaac ROS, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/how-build-and-save-map-3d-reconstruction-to-use-with-nvblox-nav2/308972
34. srv - ROS Wiki, 8월 9, 2025에 액세스, http://wiki.ros.org/srv
35. Creating custom ROS 2 msg and srv files, 8월 9, 2025에 액세스, https://docs.ros.org/en/crystal/Tutorials/Custom-ROS2-Interfaces.html
36. Assistance Requested with Isaac ROS nvblox Service Error - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/assistance-requested-with-isaac-ros-nvblox-service-error/285929
37. How could the robot localize after loading a saved map? / Issue #78 / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/78
38. isaac_ros_nvblox/nvblox_msgs/srv/EsdfAndGradients.srv at main - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/blob/main/nvblox_msgs/srv/EsdfAndGradients.srv
39. Manipulator Planning - MATLAB & Simulink - MathWorks, 8월 9, 2025에 액세스, https://www.mathworks.com/help/robotics/manipulator-planning.html
40. Isaac ROS VSLAM. NVBLOX usage with NAV2 - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/isaac-ros-vslam-nvblox-usage-with-nav2/303717
41. Evaluation of 3D Nvblox map with Real World Map - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/evaluation-of-3d-nvblox-map-with-real-world-map/317787
42. Help Integrating NVBLOX with cuMotion for Real-time Obstacle Avoidance with Franka FR3 Arm - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/help-integrating-nvblox-with-cumotion-for-real-time-obstacle-avoidance-with-franka-fr3-arm/341178
43. NVIDIA-ISAAC-ROS/isaac_ros_cumotion: NVIDIA-accelerated packages for arm motion planning and control - GitHub, 8월 9, 2025에 액세스, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_cumotion
44. OctoMap/octomap_msgs: ROS package to provide messages and serializations / conversion for the OctoMap library - GitHub, 8월 9, 2025에 액세스, https://github.com/OctoMap/octomap_msgs
45. octomap_msgs - ROS Wiki, 8월 9, 2025에 액세스, http://wiki.ros.org/octomap_msgs
46. octomap - ROS Wiki, 8월 9, 2025에 액세스, http://wiki.ros.org/octomap
47. ANYbotics/grid_map: Universal grid map library for mobile robotic mapping - GitHub, 8월 9, 2025에 액세스, https://github.com/ANYbotics/grid_map
48. ROS Package: grid_map_msgs, 8월 9, 2025에 액세스, https://index.ros.org/p/grid_map_msgs/
49. AMR Navigation Using Isaac ROS VSLAM and Nvblox with Intel Realsense Camera, 8월 9, 2025에 액세스, https://www.einfochips.com/blog/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
50. Multiple cost map with Isaac ROS Nvblox - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/multiple-cost-map-with-isaac-ros-nvblox/294846
51. Isaac ROS (Robot Operating System) - NVIDIA Developer, 8월 9, 2025에 액세스, https://developer.nvidia.com/isaac/ros
52. NVIDIA Isaac Perceptor - NVIDIA Developer, 8월 9, 2025에 액세스, https://developer.nvidia.com/isaac/perceptor
53. Create, Design, and Deploy Robotics Applications Using New NVIDIA Isaac Foundation Models and Workflows, 8월 9, 2025에 액세스, https://developer.nvidia.com/blog/create-design-and-deploy-robotics-applications-using-new-nvidia-isaac-foundation-models-and-workflows/
54. How to integrate Nvblox with Cumotion? - Isaac ROS - NVIDIA Developer Forums, 8월 9, 2025에 액세스, https://forums.developer.nvidia.com/t/how-to-integrate-nvblox-with-cumotion/340130
55. Isaac ROS cuMotion - isaac_ros_docs documentation, 8월 9, 2025에 액세스, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_cumotion/index.html
# isaac_ros_nvblox

## 1. Isaac ROS 생태계 속 nvblox의 이해

### 1.1  isaac_ros_nvblox의 정의: 로봇 인식의 새로운 패러다임

`isaac_ros_nvblox`는 로봇 운영 체제(ROS 2)를 위한 패키지 모음으로, 자율 로봇의 항법(navigation)에 필수적인 3차원 환경 재구성(3D reconstruction)과 비용 지도(cost map) 생성을 GPU 가속을 통해 실시간으로 수행하는 기술이다.1 이 패키지의 핵심은 프레임워크에 독립적인 C++ 및 CUDA 기반의 코어 라이브러리인 `nvblox`에 있으며, `isaac_ros_nvblox`는 이 코어 라이브러리에 ROS 2 인터페이스를 제공하는 래퍼(wrapper) 역할을 한다.1

`nvblox`의 근본적인 개발 목표는 고해상도의 정밀한 3차원 지도를 생성하는 작업과, 로봇 경로 계획(path planning)이 요구하는 실시간 성능 제약 사이의 간극을 메우는 데 있다. 특히 NVIDIA Jetson 시리즈와 같은 임베디드 하드웨어 상에서 이 문제를 해결하는 것에 중점을 둔다.4 기존의 CPU 기반 시스템들은 맵의 해상도나 규모에 한계가 있었고, 일부 GPU 기반 시스템들은 경로 계획에 필수적인 유클리드 부호 거리 필드(ESDF)와 같은 핵심 기능을 누락하는 경우가 많았다.5

`nvblox`는 이러한 한계를 극복하여, 과거 인식 파이프라인의 병목 현상을 유발했던 고밀도 체적 매핑(dense volumetric mapping)을 저지연(low-latency) 핵심 구성 요소로 전환시키는 것을 목표로 한다.5

### 1.2  NVIDIA 로보틱스 스택 내 전략적 위치

`nvblox`는 단순히 독립된 도구가 아니라, Isaac ROS 생태계 내에서 핵심적인 역할을 수행하는 기본 패키지, 즉 "GEM(Isaac ROS package)" 중 하나로 자리매김하고 있다.8 이는 `nvblox`가 더 높은 수준의 NVIDIA 로보틱스 애플리케이션을 가능하게 하는 기반 기술임을 시사한다. 실제로 자율이동로봇(AMR)을 위한 Isaac Perceptor나 로봇 팔을 위한 Isaac Manipulator와 같은 NVIDIA의 주력 워크플로우에서 핵심 환경 인식 모듈로 사용된다.8 이는 `nvblox`가 자율 시스템에 대한 NVIDIA의 비전에서 중심적인 역할을 하고 있음을 명확히 보여준다.

성능 극대화를 위해 `nvblox`는 NITROS(NVIDIA Isaac Transport for ROS) 기술을 적극적으로 활용한다.8 NITROS는 하드웨어 가속에 친화적인 메시지 전송 방식을 통해 노드 간 통신 오버헤드를 최소화하며, 이는 실시간 성능 확보에 결정적이다. 실제로 `nvblox`의 다중 카메라 지원 및 성능 향상은 NITROS 통합과 직접적으로 연결되어 있다.1

### 1.3  아키텍처 선택: ROS 비종속적 코어

`nvblox`의 설계에서 가장 주목할 만한 점 중 하나는 ROS에 독립적인 코어 C++/CUDA 라이브러리와 이를 감싸는 ROS 2 인터페이스 패키지 `isaac_ros_nvblox`로 분리된 모듈식 구조이다.1 이러한 설계는 의도적인 전략의 결과로 볼 수 있다. 만약 `nvblox`가 ROS와 긴밀하게 결합된 단일 패키지로 개발되었다면, 그 사용은 ROS 커뮤니티로 제한되었을 것이다. 하지만 핵심 기능을 프레임워크 비종속적인 라이브러리로 제공함으로써, NVIDIA는 자사의 고성능 매핑 기술을 소프트웨어 프레임워크에 구애받지 않는 전체 로보틱스 산업에 제공할 수 있게 되었다.

이러한 접근 방식은 기술의 전체 시장 규모(Total Addressable Market)를 확장시키고, GPU 가속 매핑의 사실상 표준(de facto standard)으로 자리 잡도록 유도한다. 결과적으로, 이는 라이브러리가 최적화된 기반 하드웨어, 즉 NVIDIA Jetson 및 dGPU의 채택을 촉진하는 효과를 낳는다. 강력하고 모듈화된 소프트웨어 구성 요소가 하드웨어 플랫폼의 보급을 견인하는 전형적인 생태계 전략인 것이다.

이러한 모듈성은 비-ROS C++ 프로젝트에서의 직접적인 사용을 가능하게 하며 6, 나아가 `nvblox_torch`와 같은 파이썬 인터페이스 개발을 통해 연구 및 프로토타이핑 커뮤니티로의 확장을 용이하게 한다.3 이러한 설계는 다른 로보틱스 알고리즘들이 ROS 자료 구조에 깊이 얽혀 발생하는 문제점들과 대조되며, 성숙한 소프트웨어 아키텍처의 모범 사례로 평가받을 수 있다.11

더불어, `nvblox`의 지속적이고 빠른 업데이트 주기는 이 프로젝트가 정적인 도구가 아니라, 공격적인 개발 로드맵을 가진 진화하는 플랫폼임을 보여준다.1 초기 버전의 버그 수정 및 ROS 2 Humble 지원에서 시작하여 3D LiDAR 지원, 맵 직렬화, 동적 객체 및 사람 분리 재구성, 다중 카메라 지원, 1cm 수준의 고해상도 복셀 최적화에 이르기까지, 그 기능 발전은 매우 체계적이다. 이는 NVIDIA가 `nvblox`를 자사 로보틱스 전략의 핵심적이고 장기적인 기둥으로 간주하고 있음을 시사한다. `nvblox`는 자율 시스템의 진화하는 요구에 맞춰 더 높은 정밀도와 지능을 향해 적극적으로 발전하고 있다.

## 2. 장: 체적 재구성의 기초 개념

### 2.1  점유 격자에서 부호 거리 필드(SDF)까지

로봇 환경을 표현하는 전통적인 방식 중 하나인 점유 격자 지도(occupancy grid map)는 공간을 작은 복셀(voxel)로 나누고 각 복셀이 장애물에 의해 점유될 확률을 저장한다.4 OctoMap과 같은 시스템은 옥트리(octree) 구조를 사용하여 이 개념을 효율적으로 구현한 대표적인 예이다.14 그러나 점유 격자 지도는 본질적으로 각진 형태의 저품질 표면을 생성하며, 장애물까지의 거리 정보를 직접 제공하지 않는다는 한계를 가진다.

반면, 부호 거리 필드(Signed Distance Field, SDF)는 공간상의 모든 지점에서 가장 가까운 표면까지의 거리를 저장한다. 이때 부호는 해당 지점이 객체의 내부에 있는지(-) 혹은 외부에 있는지(+)를 나타낸다.16 이 표현 방식은 단순히 점유 여부만을 나타내는 것보다 훨씬 풍부한 기하학적 정보를 담고 있다.

### 2.2  절단 부호 거리 함수(TSDF)

절단 부호 거리 함수(Truncated Signed Distance Function, TSDF)는 SDF의 실용적인 구현체로, 표면 주변의 일정 거리($μ$) 내에서만 거리 값을 저장하고 그 밖의 값은 최대값으로 "절단(truncate)"한다.16 이 방식은 표면에서 멀리 떨어진 공간에 대한 정보를 저장하지 않음으로써 메모리 사용량과 계산량을 크게 줄인다. TSDF에서 표면은 거리 값이 0이 되는 지점들의 집합, 즉 영점 교차(zero-crossing)로 암시적으로 정의된다.4

TSDF 통합의 핵심은 새로운 깊이 측정값($d_k$)이 주어졌을 때, 해당 복셀($v$)의 기존 TSDF 값($D_{old}(v)$)과 가중치($W_{old}(v)$)를 새로운 측정값과 가중치($w_k$)를 사용하여 갱신하는 가중 평균 방식에 있다. 이 과정은 여러 프레임에 걸쳐 센서 노이즈를 평균화하여 부드럽고 안정적인 표면을 재구성하는 효과를 낳는다.19 핵심적인 갱신 수식은 다음과 같다.

새로운 TSDF 값:
$$
D_{new}(v) = \frac{W_{old}(v)D_{old}(v) + w_k d_k(v)}{W_{old}(v) + w_k}
$$
새로운 가중치:
$$
W_{new}(v) = \min(W_{old}(v) + w_k, W_{max})
$$
여기서 $d_k(v)$는 복셀 $v$에 대한 새로운 깊이 측정값의 부호 거리이며, $w_k$는 해당 측정의 가중치, $W_{max}$는 최대 누적 가중치이다. `nvblox`는 깊이 카메라나 LiDAR로부터 얻은 깊이 정보($\mathcal{D}:\Omega\to d$)를 이와 같은 방식으로 3D 복셀 그리드에 저장된 TSDF에 통합한다.4

### 2.3  유클리드 부호 거리 필드(ESDF)

TSDF는 표면 *근처*에서는 정확한 거리 정보를 제공하지만, 로봇 경로 계획기는 자유 공간(free space) 내의 *모든* 지점에서 가장 가까운 장애물까지의 실제 유클리드 거리를 필요로 한다. 이 역할을 수행하는 것이 바로 유클리드 부호 거리 필드(Euclidean Signed Distance Field, ESDF)이다.4

`nvblox`는 생성된 TSDF 맵으로부터 완전한 비절단(non-truncated) ESDF를 계산한다.4 이 과정은 표면 경계에 위치한 "사이트(site)" 복셀들이 자신의 거리 정보를 자유 공간으로 전파시키는 파면 전파(wavefront propagation) 알고리즘을 통해 이루어진다.5 ESDF는 로봇의 잠재적 미래 위치가 환경과 충돌하는지 즉시 확인할 수 있는 수단을 제공하며, 최적화 기반 경로 계획기를 위한 경사도(gradient) 정보까지 제공할 수 있어 항법에 있어 핵심적인 출력이 된다.20

TSDF를 사용하는 것은 단순히 점유 격자보다 나은 품질을 위한 선택을 넘어선다. NVIDIA는 자사의 GPU 하드웨어를 통해 TSDF의 계산적 부담을 극복할 수 있다고 판단했고, 이를 통해 과거 CPU 기반 시스템들이 성능 문제로 포기해야 했던 고품질 재구성을 실시간으로 제공한다.

`nvblox` 접근 방식의 핵심은 거리 필드의 "이중 활용성(dual utility)"에 있다.4 즉, 하나의 통합된 데이터 파이프라인(센서 데이터 -->> TSDF -->> 메시 / ESDF -->> 경로 계획기)을 통해 재구성(reconstruction)과 경로 계획(planning)이라는 두 가지 다른 목적을 동시에 만족시킨다. TSDF는 고품질 메시 생성을 위한 기반이 되고 2, 동시에 경로 계획기에 이상적인 입력인 ESDF를 생성하는 데 사용된다.4 이처럼 하나의 핵심 표현 방식이 여러 다운스트림 작업을 지원하는 것은 매우 효율적이고 우아한 엔지니어링 솔루션의 특징이다.

## 3. 장: nvblox 아키텍처: 알고리즘과 자료 구조

### 3.1  시스템 입력 및 전처리

`nvblox`는 다양한 센서 입력에 대응할 수 있도록 유연하게 설계되었다. ZED나 Hawk와 같은 스테레오 카메라, RealSense나 Kinect와 같은 RGB-D 카메라에서 얻은 깊이 이미지, 그리고 3D LiDAR 포인트 클라우드를 모두 입력으로 받을 수 있다.1

시스템의 필수 입력 데이터는 깊이 정보와 센서의 위치 및 자세(pose)를 나타내는 주행 거리 측정(odometry) 정보다.20 선택적으로 메시(mesh)에 색상을 입히기 위한 컬러 이미지와 20, 깊이 및 컬러 센서의 내부 파라미터(intrinsics)를 입력으로 사용할 수 있다.22 3D LiDAR 입력을 처리할 경우, `nvblox`는 `lidar_height`, `lidar_width`, `lidar_vertical_fov_rad`와 같은 파라미터를 사용하여 3D 포인트 클라우드를 원통형 깊이 이미지로 재투영(re-project)하여 처리한다.20

### 3.2  메모리 관리: 복셀 해싱 자료 구조

고해상도의 3D 복셀 그리드를 조밀하게(dense) 저장하는 것은 조금만 규모가 커져도 메모리 측면에서 비현실적이다.23 이 문제를 해결하기 위해 `nvblox`는 Voxblox 5 및 Nießner 등의 선구적인 연구 24에서 제안된 복셀 해싱(voxel hashing) 기법을 채택했다. 이 방식은 해시 테이블을 사용하여 3D 공간 좌표를 "복셀 블록(voxel block)"(예: 8x8x8 크기의 복셀 덩어리)에 대한 포인터에 매핑한다. 메모리는 센서에 의해 관측된 영역의 블록에 대해서만 동적으로 할당된다.23

이러한 접근법 덕분에 메모리 소비량은 환경의 전체 *부피*가 아닌 관측된 *표면적*에 비례하게 되어 대규모 환경의 재구성이 가능해진다.23 또한, 해시 테이블의 특성상 데이터의 갱신 및 조회에 평균적으로 O(1)의 효율적인 접근 시간을 제공한다.24

`nvblox`의 혁신은 복셀 해싱이라는 자료 구조 자체에 있는 것이 아니라, 이 자료 구조 위에서 수행되는 모든 연산(통합, 메시 생성, ESDF 계산 등)을 CUDA 프로그래밍 모델을 통해 대규모로 병렬화한 데 있다. 즉, 메모리 효율성을 위해 이미 검증된 해결책을 채택하고, 그 위에 NVIDIA의 핵심 역량인 GPU 컴퓨팅을 적용하여 처리 속도를 극대화한 것이다.

### 3.3  핵심 알고리즘 1: GPU 가속 TSDF 통합

`nvblox`의 핵심 매핑 프로세스는 새로운 깊이 프레임을 전역 TSDF 볼륨에 통합하는 것이다. `nvblox`의 ICRA 2024 논문 5에 따르면, 이는 투영 기반(projection-based) 접근 방식을 사용하는 것으로 보인다. 카메라의 절두체(frustum) 내에 있는 각 복셀 블록에 대해, 블록 내의 복셀들을 깊이 이미지로 투영하여 대응하는 깊이 측정값을 찾는다. 이 측정값을 사용하여 부호 거리를 계산하고, 이를 2.2절에서 설명한 가중 평균 공식을 통해 해당 복셀의 TSDF 값에 융합(fuse)한다.

이 모든 과정은 GPU에서 대규모로 병렬 처리된다. 각 복셀 또는 복셀 블록은 독립적인 CUDA 스레드에 의해 처리될 수 있어, CPU 기반 방식에 비해 극적인 속도 향상을 가져온다.3

### 3.4  핵심 알고리즘 2: GPU 가속 ESDF 계산

ESDF는 갱신된 TSDF 레이어로부터 생성된다. 이 과정은 본질적으로 파면 전파 또는 거리 변환(distance transform) 알고리즘의 병렬화된 버전이다.

알고리즘의 주요 단계는 다음과 같다 5:

1. **블록 할당 및 사이트 마킹**: TSDF 레이어에 새로운 복셀 블록이 추가되면 ESDF 레이어에도 동일하게 블록을 할당한다. TSDF 표면 경계(영점 교차)에 있는 복셀들은 거리 전파의 시작점인 "사이트"로 표시된다.
2. **거리 전파**: 사이트로부터 바깥쪽으로 거리를 전파하는 병렬 반복 프로세스가 시작된다. 각 반복 단계에서 복셀들은 이웃 복셀들의 거리 값을 기반으로 자신의 거리 값을 갱신하며, 이 과정이 전체 그리드에 걸쳐 병렬로 수행된다.
3. **갱신 및 무효화 처리**: TSDF가 갱신되어 기존의 장애물이 사라지면, 해당 ESDF 복셀들과 그 복셀을 가장 가까운 장애물로 참조하던 "자식" 복셀들을 모두 무효화하고 재계산해야 한다. 이 또한 맵의 넓은 영역에 걸쳐 자신의 부모 사이트가 더 이상 유효하지 않은 복셀들을 병렬로 탐색하여 처리된다.

이러한 GPU 병렬 접근 방식은 Voxblox와 같은 CPU 기반의 순차적 방법에 비해 월등히 빠르며, 최대 31배의 속도 향상이 보고되었다.5

### 3.5  출력 레이어

`nvblox`는 TSDF, ESDF, 색상(Color), 메시(Mesh) 등 여러 개의 개별적인 데이터 "레이어"를 유지한다.13 이러한 모듈성은 다운스트림 애플리케이션이 필요한 데이터만 선택적으로 소비할 수 있게 해준다. 최종적으로 사용자에게 제공되는 주요 출력물은 시각화를 위한 3D 메시(`~/mesh`)와 경로 계획을 위한 2D ESDF 슬라이스(`~/map_slice`)이다.2

한편, TSDF 통합 전략에는 미묘하지만 중요한 기술적 논쟁이 존재한다. `nvblox`는 복셀을 이미지에 투영하는 **투영 매핑** 방식을 사용하는 것으로 보이며, 이는 업데이트할 복셀 수에 의해 계산량이 제한된다. 반면, `coVoxSLAM`과 같은 대안적 연구 26는 센서에서 이미지 픽셀을 통해 광선을 쏘는 **레이캐스팅(raycasting)** 방식을 주장한다. 이 방식은 깊이 이미지의 픽셀 수에 의해 계산량이 제한된다. `coVoxSLAM`의 저자들은 자신들의 GPU 기반 레이캐스팅 구현이 동일한 데이터셋에서 `nvblox`보다 1.5배에서 2배 더 빠르다고 주장하며 26, 특정 유형의 센서 노이즈에 덜 민감하다고 보고한다.28 이는 `nvblox`가 매우 빠르기는 하지만, 그 핵심 통합 알고리즘이 성능 면에서 절대적인 최첨단이 아닐 수 있음을 시사한다. NVIDIA의 투영 매핑 선택은 아키텍처의 단순성, 하드웨어에 대한 특정 최적화, 또는 희소한 LiDAR 데이터 처리의 용이성을 위한 설계적 타협일 수 있으며, 이는 향후 개선의 여지가 있는 유효한 연구 영역임을 나타낸다.

## 4. 장: 성능 분석 및 하드웨어 가속

### 4.1  GPU 가속의 필연성

Voxblox와 같은 CPU 기반 시스템은 본질적으로 직렬 처리 또는 제한된 스레드 수에 의해 성능이 제약된다.29 이로 인해 실시간으로 구축할 수 있는 지도의 해상도, 규모, 또는 갱신 빈도가 제한될 수밖에 없다.5 실제로 Voxblox는 성능보다는 코드의 가독성과 확장성, 그리고 GPU가 없는 시스템에서의 작동을 우선순위로 두고 설계되었다.30

반면, 수천 개의 코어를 가진 NVIDIA GPU는 동일한 명령을 다수의 데이터에 동시에 적용하는(SIMD) 대규모 병렬 작업에 최적화되어 있다. 동일한 갱신 규칙을 수백만 개의 복셀에 적용하는 체적 매핑은 이러한 아키텍처에 완벽하게 부합하는 작업이다.3

`nvblox`는 이러한 병렬성을 최대한 활용하기 위해 처음부터 C++와 CUDA로 작성되었다.2

### 4.2  정량적 성능 벤치마크

`nvblox`의 성능은 다양한 하드웨어 플랫폼에서 측정되었으며, 그 결과는 개발자가 시스템 설계 시 중요한 참고 자료로 활용할 수 있다. 아래 표는 주요 하드웨어에서의 처리 시간을 요약한 것이다.

**표 1: NVIDIA 하드웨어 기반 `nvblox` 성능 벤치마크** 1

| 플랫폼                 | 복셀 크기 (m) | 구성 요소 | 데이터셋 | 처리 시간 (ms) | 대략적 FPS |
| ---------------------- | ------------- | --------- | -------- | -------------- | ---------- |
| **x86_64 w/ RTX 3090** | 0.05          | TSDF      | Replica  | 0.5 ms         | 2000       |
|                        | 0.05          | Color     | Replica  | 0.7 ms         | 1428       |
|                        | 0.05          | Meshing   | Replica  | 0.7 ms         | 1428       |
|                        | 0.05          | ESDF      | Replica  | 0.8 ms         | 1250       |
| **Jetson AGX Orin**    | 0.05          | TSDF      | Replica  | 0.8 ms         | 1250       |
|                        | 0.05          | Color     | Replica  | 1.1 ms         | 909        |
|                        | 0.05          | Meshing   | Replica  | 2.3 ms         | 435        |
|                        | 0.05          | ESDF      | Replica  | 1.7 ms         | 588        |
| **Jetson Orin Nano**   | 0.05          | TSDF      | Replica  | 2.1 ms         | 476        |
|                        | 0.05          | Color     | Replica  | 3.6 ms         | 277        |
|                        | 0.05          | Meshing   | Replica  | 13.0 ms        | 77         |
|                        | 0.05          | ESDF      | Replica  | 6.2 ms         | 161        |

이 데이터는 임베디드 플랫폼인 Jetson AGX Orin에서도 전체 매핑 파이프라인(TSDF+Color+Mesh+ESDF) 단계가 약 5.9 ms 내에 완료될 수 있음을 보여준다. 이는 100 Hz가 넘는 갱신율을 가능하게 하며, 일반적인 깊이 카메라의 입력 속도인 30 Hz를 훨씬 상회한다. 이러한 성능적 여유는 매핑이 전체 시스템의 병목이 되지 않도록 보장하는 핵심적인 특징이다.

`nvblox`의 성능은 단순히 '더 빠르다'는 것을 넘어, 로보틱스에서 가능한 것의 범위를 근본적으로 바꾼다. 이전에는 CPU 기반 방법론을 사용하는 개발자들이 낮은 해상도, 낮은 갱신 빈도, 또는 좁은 매핑 영역이라는 고통스러운 타협을 해야만 했다. `nvblox`의 밀리초 단위 처리 시간은 이러한 타협을 불필요하게 만든다. 개발자들은 이제 넓은 영역에 걸쳐 고해상도(예: 1-2cm 복셀) 1 지도를 센서의 최대 프레임 속도(30Hz 이상)로 갱신할 수 있게 되었다. 이는 더 정밀한 항법, 조작, 그리고 환경과의 상호작용을 가능하게 하는 질적인 도약이다.

### 4.3  성능 비교: `nvblox` 대 `Voxblox`

`nvblox` 논문은 Voxblox와 같은 최신 CPU 기반 방법에 비해 표면 재구성(TSDF)에서 최대 **177배**, 거리 필드 계산(ESDF)에서 최대 **31배**의 경이적인 성능 향상을 보고한다.5

이러한 속도 향상은 단순히 "GPU가 CPU보다 빠르다"는 사실 때문만은 아니다. 이는 병렬 실행을 위해 처음부터 다시 설계한 결과이다. Voxblox가 많은 연산에서 단일 스레드로 동작하는 반면 29, `nvblox`는 메모리 할당, 센서 데이터 통합, ESDF 전파 등 모든 과정을 수천 개의 CUDA 코어에 걸쳐 병렬화한다.

다만 한 가지 미묘한 점은, 맵의 해상도가 증가할수록 `nvblox`의 Voxblox 대비 속도 우위가 감소하는 경향이 있다는 것이다.31 이는 고해상도에서는 복셀 블록이 관측된 표면을 효율적으로 감싸지 못하고, 각 블록 내에서 낭비되는 계산이 늘어나기 때문이다.

그럼에도 불구하고 `nvblox` 성능의 가장 심오한 함의는 Jetson Orin 시리즈와 같은 임베디드 플랫폼에서의 실행 가능성이다. 고성능의 조밀한 매핑은 전통적으로 고사양 GPU를 갖춘 강력한 데스크톱 워크스테이션의 영역이었다. AGX Orin과 저전력 Orin Nano에서의 벤치마크 결과 1는 이러한 기능이 이제 이동 로봇에 적합한 작고 전력 효율적인 모듈에서도 사용 가능함을 보여준다. 이는 고품질 3D 인식 기술에 대한 접근성을 민주화하며, 과거 고가의 하드웨어를 갖춘 연구실에서나 가능했던 복잡한 자율 행동이 실제 상용 제품에 배포될 수 있는 길을 열어준다. 

`nvblox`는 엣지 AI 및 로보틱스 시장으로 진출하려는 NVIDIA의 하드웨어 전략을 뒷받침하는 핵심적인 소프트웨어 조력자이다.

## 5. 장: 실제 환경을 위한 고급 기능

### 5.1  동적 장면 매핑: 다중 모드 접근법

`nvblox`는 순수하게 정적이지 않은 환경을 처리하기 위해 여러 작동 모드를 제공한다.1 이러한 유연성은 사람이나 다른 객체가 움직이는 실제 애플리케이션에서 매우 중요하다.

### 5.2  의미론적 분할을 통한 인간 중심 매핑

이 기능은 동적 개체를 처리하는 가장 정교한 모드로, `PeopleSemSegNet`과 같은 의미론적 분할(semantic segmentation)을 위한 심층 신경망(DNN)에 의존한다.1

데이터 흐름은 다음과 같다: 컬러 이미지가 DNN에 의해 처리되어 분할 마스크를 생성한다. 사람에 해당하는 픽셀이 레이블링된 이 마스크는 `mask/image` 토픽을 통해 `nvblox`로 전달된다.22

`nvblox`는 이 마스크를 사용하여 깊이 이미지를 분리한다. 정적인 배경에 해당하는 깊이 데이터는 주 TSDF 맵에 통합되고, 사람에 해당하는 깊이 데이터는 *별도의* 사람 전용 맵 레이어에 통합된다.1

이 접근 방식의 장점은 움직이는 사람이 정적 지도에 "유령"이나 "잔상" 같은 인공물을 남기며 지도를 오염시키는 것을 방지한다는 것이다. 또한, 사람의 위치에 대한 전용 지도를 제공하여, 경로 계획기가 사회적으로 인지적인 항법(socially-aware navigation)을 수행하는 데 활용될 수 있다.33

### 5.3  점유 감쇠를 통한 일반 동적 재구성

분할 모델을 사용할 수 없는 동적 객체를 위해, `nvblox`는 보다 일반적인 동적 재구성 모드를 제공한다.1 이 모드는 시간적 감쇠(temporal decay) 메커니즘을 사용한다. 고정된 빈도로, 동적 맵에 있는 모든 복셀의 점유 확률은 "알 수 없음" 상태(확률 0.5)를 향해 점차 감소한다.13

그 결과, 한 번 관측된 후 사라진 객체는 "유령" 장애물로 계속 남아있는 대신 지도에서 서서히 사라지게 된다. 이 감쇠율은 사용자가 조정할 수 있는 파라미터이다.35

### 5.4  다중 센서 융합

`nvblox`는 여러 대의 카메라에서 들어오는 데이터 스트림을 동시에 통합하는 것을 지원한다.1 이를 통해 로봇 주변의 사각지대를 줄이고 360도에 가까운 포괄적인 시야를 확보하여, 복잡한 창고 환경의 AMR과 같은 애플리케이션에 필수적인 완전하고 강건한 환경 모델을 생성할 수 있다.4 지원되는 카메라의 수는 플랫폼 및 작동 모드에 따라 달라진다.4

`nvblox`가 사람을 처리하는 방식은 고전적인 기하학 파이프라인에 AI를 통합하는 매우 실용적이고 효과적인 접근법을 보여준다. 순수 기하학 시스템은 정적인 상자와 움직일 수 있는 사람을 구별하기 어렵다. 반면, 순수 AI 기반 매핑 시스템은 기하학적 정밀도가 부족하거나 해석이 어려울 수 있다. `nvblox`는 하이브리드 접근법을 취한다: 특정 의미론적 클래스(사람)를 인식하는 데 가장 뛰어난 전문 AI 구성 요소(분할 DNN)를 사용한다. 그런 다음, 이 AI 모듈의 출력(마스크)을 견고하고 잘 이해된 기하학 파이프라인에 간단하고 명확한 입력으로 사용하여 데이터를 다른 레이어로 보낸다. 이는 딥러닝의 의미론적 능력과 TSDF의 기하학적 정밀성을 결합하여, 어느 한쪽만으로는 달성할 수 없는 더 나은 결과를 만들어낸다.

이러한 동적 및 의미론적 처리 기능의 포함은 "매핑"의 정의에 근본적인 변화가 일어나고 있음을 의미한다. 전통적인 매핑은 세계의 정적인 기하학적 모델을 구축하는 것이었다. 그러나 `nvblox`의 기능들은 현대적인 지도가 시간적으로 변하며 의미론적으로 인지하는 표현이어야 함을 보여준다. 지도는 정적 요소와 동적 요소를 구별해야 하며, 일부 요소가 *무엇*인지(예: 사람) 이해해야 한다. 이는 지도를 단순한 "청사진"에서 지능적인 행동의 기반이 되는 풍부하고 동적인 세계 모델로 전환시킨다. `nvblox`는 이러한 전환을 가능하게 하는 핵심 기술이다.

## 6. 장: isaac_ros_nvblox를 통한 실용적 통합

### 6.1  패키지 생태계

`isaac_ros_nvblox` 저장소는 함께 작동하도록 설계된 여러 패키지로 구성되어 있다.1

- `nvblox_ros`: 주 ROS 2 래퍼 노드를 포함한다.
- `nvblox_msgs`: `Mesh` 및 `DistanceMapSlice`와 같은 커스텀 ROS 메시지를 정의한다.
- `nvblox_rviz_plugin`: `nvblox_msgs/Mesh` 출력을 시각화하기 위한 RViz 플러그인이다.
- `nvblox_nav2`: Nav2 스택과 연동하기 위한 비용 지도 플러그인을 제공한다.
- `nvblox_examples_bringup`: 예제 실행 파일을 포함한다.

### 6.2  ROS 2 애플리케이션 프로그래밍 인터페이스(API)

주요 노드는 표준 재구성을 위한 `nvblox_node`와, `nvblox_node`를 상속받아 인간 분할 기능을 추가한 `nvblox_human_node`이다.22 개발자가 `nvblox`를 통합하고자 할 때 가장 중요한 실용적인 참조 자료는 통신 인터페이스를 요약한 아래의 표이다.

**표 2: `isaac_ros_nvblox` ROS 2 API**

| 분류                   | 토픽/서비스 이름        | 타입                                                | 설명 및 목적                                                 |
| ---------------------- | ----------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| **구독(Subscribed)**   | `depth/image`           | `sensor_msgs/Image`                                 | **필수.** 입력 깊이 이미지. float(미터) 또는 uint16(밀리미터) 지원. 22 |
|                        | `depth/camera_info`     | `sensor_msgs/CameraInfo`                            | **필수.** 깊이 카메라의 내부 파라미터. 22                    |
|                        | `pose` 또는 `transform` | `geometry_msgs/PoseStamped` 또는 `TransformStamped` | **필수.** SLAM/VIO 시스템으로부터의 센서 포즈(주행 거리). `use_tf_transforms`가 false일 때 사용. 22 |
|                        | `color/image`           | `sensor_msgs/Image`                                 | *선택.* 입력 컬러 이미지. 출력 메시의 색상 부여에만 사용됨. 22 |
|                        | `color/camera_info`     | `sensor_msgs/CameraInfo`                            | *선택.* 컬러 카메라의 내부 파라미터. 22                      |
|                        | `pointcloud`            | `sensor_msgs/PointCloud2`                           | *선택.* 3D LiDAR로부터의 입력. 깊이 이미지의 대안. 22        |
|                        | `mask/image`            | `sensor_msgs/Image`                                 | **(Human Node 전용)**. DNN으로부터의 입력 분할 마스크. 비-인간=0, 인간=1. 22 |
|                        | `mask/camera_info`      | `sensor_msgs/CameraInfo`                            | **(Human Node 전용)**. 마스크 이미지의 내부 파라미터. 22     |
| **발행(Published)**    | `~/mesh`                | `nvblox_msgs/Mesh`                                  | RViz에서 시각화를 위한 정적 재구성의 3D 메시. 22             |
|                        | `~/map_slice`           | `nvblox_msgs/DistanceMapSlice`                      | **항법에 결정적.** `nvblox_nav2` 비용 지도 플러그인이 소비하는 정적 ESDF의 2D 슬라이스. 22 |
|                        | `~/esdf_pointcloud`     | `sensor_msgs/PointCloud2`                           | 시각화를 위한 2D ESDF의 포인트 클라우드 표현. 22             |
|                        | `~/occupancy`           | `sensor_msgs/PointCloud2`                           | 시각화를 위한 점유 지도의 포인트 클라우드. 22                |
|                        | `~/human_map_slice`     | `nvblox_msgs/DistanceMapSlice`                      | **(Human Node 전용)**. 인간 전용 ESDF의 2D 슬라이스. 22      |
|                        | `~/combined_map_slice`  | `nvblox_msgs/DistanceMapSlice`                      | **(Human Node 전용)**. Nav2를 위한 정적 및 인간 지도의 결합된 2D 슬라이스. 22 |
| **서비스(Advertised)** | `~/save_map`            | `nvblox_msgs/FilePath`                              | 전체 `nvblox` 맵(TSDF, ESDF 등)을 파일로 직렬화하여 저장하는 서비스. 22 |
|                        | `~/load_map`            | `nvblox_msgs/FilePath`                              | 이전에 저장된 `nvblox` 맵을 불러와 현재 맵을 덮어쓰는 서비스. 22 |
|                        | `~/save_ply`            | `nvblox_msgs/FilePath`                              | 재구성된 메시를 외부 소프트웨어에서 볼 수 있도록 `.ply` 파일로 저장하는 서비스. 22 |

### 6.3  일반적인 애플리케이션 그래프

일반적인 데이터 흐름은 다음과 같이 구성된다 1:

1. **센서** (예: RealSense D455)가 `depth/image`, `color/image`, `infra/image` 등을 발행한다.
2. **`isaac_ros_visual_slam`**이 `infra/image`와 IMU 데이터를 소비하여 로봇의 포즈를 `/tf` 트리에 발행한다.
3. **(선택) `isaac_ros_image_segmentation`**이 `color/image`를 소비하여 `mask/image`를 발행한다.
4. **`isaac_ros_nvblox`** (`nvblox_node` 또는 `nvblox_human_node`)가 `depth/image`, `color/image`, `mask/image`를 구독하고 `/tf`에서 포즈를 조회한다.
5. `nvblox`는 `~/map_slice`와 `~/mesh`를 발행한다.
6. **`Nav2`** (`nvblox_nav2` 플러그인 포함)가 `~/map_slice`를 경로 계획을 위한 비용 지도 레이어로 소비한다.
7. **`RViz`** (`nvblox_rviz_plugin` 포함)가 `~/mesh`를 시각화를 위해 소비한다.

실용적인 항법에서 가장 중요한 출력은 `nvblox_msgs/DistanceMapSlice` 토픽이다. `nvblox`는 풍부한 3D 세계에서 작동하여 완전한 3D ESDF를 생성하지만, 지상 로봇 항법의 지배적인 스택인 Nav2는 2D 비용 지도 위에서 작동한다. 이 차원적 불일치를 해결하기 위해 `map_slice`가 존재한다. 이는 단순히 3D 맵을 2D 그리드로 투영하여 정보를 잃는 순진한 접근법이 아니다. `map_slice`는 로봇에 관련된 높이에서 잘라낸, 3D 세계의 *거리 정보*를 여전히 포함하는 2D 표현이다. 이는 2D 경로 계획기가 우수한 3D 인식의 이점을 누릴 수 있도록 하는 "번역 계층" 역할을 하며, `nvblox`를 기존 ROS 생태계를 위한 플러그 앤 플레이 비용 지도 제공자로 만든다.

또한, API가 외부 포즈 입력(`pose`, `transform`, 또는 TF)을 요구한다는 점은 NVIDIA의 "자신만의 SLAM을 가져오라(Bring Your Own SLAM)"는 설계 철학을 명확히 보여준다. `nvblox`는 SLAM 문제의 일부인 지역화(localization)를 직접 수행하지 않고, 외부에서 제공된 포즈를 단순히 소비한다. 이러한 문제의 분리(disaggregation)는 `nvblox`가 GPU 가속 매핑이라는 한 가지 일에만 집중하여 탁월한 성능을 발휘하게 한다. 동시에 사용자에게는 자신의 애플리케이션에 가장 적합한 지역화 프론트엔드(예: 시각적 SLAM, LiDAR SLAM, GPS/IMU 등)를 선택하고, 이를 동급 최고의 매핑 백엔드인 `nvblox`와 결합할 수 있는 유연성을 제공한다. 이러한 모듈성은 성숙하고 유연한 소프트웨어 생태계의 특징이다.

## 7. 장: 비교 분석 및 현재의 한계

### 7.1  현대 3D 매핑 기술 속 nvblox의 위치

`nvblox`를 그 주요 선행 기술 및 동시대 기술들과 비교함으로써, 각 매핑 시스템 선택에 따르는 장단점을 명확히 할 수 있다.

**표 3: 3D 매핑 시스템 비교 분석**

| 특징                        | `nvblox`                    | `Voxblox` 29                  | `OctoMap` 14             | `Kimera-SLAM` 39               |
| --------------------------- | --------------------------- | ----------------------------- | ------------------------ | ------------------------------ |
| **핵심 표현 방식**          | TSDF & ESDF                 | TSDF & ESDF                   | 확률적 점유              | 미터법-의미론적 메시           |
| **주요 연산 장치**          | **GPU (CUDA)**              | CPU                           | CPU                      | CPU                            |
| **메모리 구조**             | 복셀 해싱                   | 복셀 해싱                     | 옥트리                   | 복셀 해싱 (Voxblox 기반)       |
| **핵심 강점**               | **고해상도 실시간 속도**    | 유연성, CPU 기반, 점진적 ESDF | 메모리 효율성, 확률 기반 | **완전한 SLAM 시스템, 의미론** |
| **동적 객체 처리**          | 가능 (감쇠 & 분할)          | 불가 (기본적으로)             | 확률적 감쇠              | 가능 (동적 객체 추적)          |
| **완전한 SLAM 시스템 여부** | **아니오 (외부 포즈 필요)** | 아니오 (외부 포즈 필요)       | 아니오 (매핑만)          | **예 (VIO + 포즈 그래프)**     |

### 7.2. 핵심 강점 요약

`nvblox`의 가장 큰 강점은 임베디드 GPU에서도 고해상도의 조밀한 3D 맵과 ESDF를 실시간으로 생성할 수 있는 전례 없는 성능이다.5 또한, TSDF 표현 방식은 시각화 및 상호작용에 적합한 고품질의 표면 메시를 생성한다.4 마지막으로, NVIDIA 하드웨어(Jetson, GPU) 및 소프트웨어(Isaac ROS, NITROS, Nav2)와의 긴밀한 생태계 통합은 강력한 수직 통합 솔루션을 제공한다.1

### 7.2  알려진 한계 및 실용적 과제

`nvblox`의 가장 중요한 한계는 순수한 매핑 라이브러리라는 점이다. 즉, 지역화, 포즈 그래프 최적화, 또는 루프 폐쇄(loop closure)를 수행하지 않는다.26 따라서 상위 오도메트리 소스의 드리프트(drift)에 취약하다. 이는 Kimera 39나 ORB-SLAM과 같은 완전한 SLAM 패키지와 대조되는 지점이다.

또한, 성능은 호환되는 NVIDIA GPU와 CUDA 지원 여부에 본질적으로 묶여 있어, 비-NVIDIA 하드웨어나 GPU가 없는 플랫폼을 위한 범용 솔루션은 아니다.1

실제 사용 시에는 구성 및 의존성 문제의 복잡성이라는 현실적인 과제에 직면할 수 있다. 사용자 포럼과 GitHub 이슈들은 다음과 같은 공통적인 문제들을 보고하고 있다 42:

- **센서 문제**: Intel RealSense 카메라와 관련된 문제가 빈번히 보고되며, 이는 주로 리눅스 커널 버전 비호환성 및 특정 DKMS 패치 필요성에서 비롯된다.42
- **빌드/설치 문제**: 사용자들은 `stdgpu`와 같은 의존성 문제, 컴파일 오류, 그리고 Docker 환경의 복잡한 설정 과정에 대한 어려움을 토로한다.45
- **성능 튜닝**: 특히 다중 카메라 환경에서 실시간 성능을 달성하는 것은 어려울 수 있으며, 올바르게 구성되지 않을 경우 상당한 지연이 발생한다는 보고와 함께 신중한 파라미터 튜닝을 요구한다.47

`nvblox`는 로보틱스 소프트웨어 분야에서 "플러그인 가능한 고성능 매핑 백엔드"라는 새롭고 중요한 틈새 시장을 성공적으로 개척했다. SLAM 분야는 역사적으로 지역화와 매핑을 모두 수행하는 단일 시스템에 의해 지배되어 왔다. `nvblox`가 SLAM을 수행하지 않는다는 "한계"는 역설적으로 그 핵심 가치 제안이 된다. 이는 문제를 분리하여 개발자가 "동급 최강(best-of-breed)" 시스템을 구축할 수 있게 해준다. 즉, 프론트엔드로는 최첨단 VIO를 사용하고, 이를 최첨단 매핑 백엔드인 `nvblox`에 연결하는 것이다. 이러한 모듈성은 복잡한 로보틱스 시스템에서 매우 바람직하며, 올인원 솔루션에서 벗어나 더 유연하고 구성 가능한 아키텍처로 나아가는 분야의 성숙을 나타낸다.

그러나 공식 문서와 논문이 제시하는 강력하고 원활한 시스템의 모습과, 커뮤니티 피드백이 보여주는 현실 사이에는 간극이 존재한다. 특정 하드웨어, 커널 버전, 센서 모델이라는 "황금 경로"를 벗어날 경우 시스템이 불안정하고 구성하기 어려울 수 있다는 점은, 핵심 기술은 견고하지만 이를 둘러싼 설치 스크립트, 의존성 관리, 대체 설정에 대한 문서화 등 "연결 조직"이 덜 성숙했을 수 있음을 시사한다. 이는 고도로 최적화된 하드웨어 종속 소프트웨어에서 흔히 발견되는 과제이며, 잠재적 도입자에게는 상당한 통합 및 문제 해결 노력을 위한 시간 예산을 책정해야 한다는 중요한 현실적 고려 사항을 제시한다.

## 8. 장: 결론 및 미래 전망

### 8.1. nvblox의 기여 요약

`nvblox`는 조밀한 체적 매핑을 계산적으로 불가능에 가까웠던 작업에서, 임베디드 로봇 시스템에서도 실시간 저지연으로 수행 가능한 구성 요소로 전환시키는 패러다임의 변화를 대표한다. 이는 NVIDIA 생태계의 힘을 빌어 로보틱스 커뮤니티에 고성능 3D 재구성 기술을 성공적으로 상품화(commoditize)한 사례이다.

### 8.1  앞으로의 길: 진정한 의미론적 SLAM을 향하여

`nvblox`의 미래는 명백히 의미론적(semantic)이다. `nvblox_torch`의 개발은 이러한 방향성을 보여주는 가장 명확한 증거다.12

`nvblox_torch`는 비전 파운데이션 모델(VFM) 및 비전-언어 모델(VLM)에서 추출한 고차원 특징 벡터를 3D 복셀 그리드에 직접 융합하는 것을 가능하게 한다. 이를 통해 지도는 단순히 기하학(거리)과 외형(색상)의 저장소를 넘어, 풍부하고 학습된 의미론적 정보의 저장소로 거듭난다.12

이는 지도를 정적인 청사진에서 동적이고 질의 가능한(queryable) 공간 데이터베이스로 변환시킨다. 로봇은 "주행 가능한 모든 표면을 보여줘" 또는 "가장 가까운 의자는 어디에 있는가?"와 같은 의미론적 질의를 3D 지도에 직접 수행할 수 있게 될 것이다.

### 8.2  실시간 3D 매핑의 미래: 체화된 AI의 중추

미래의 추세는 `nvblox`가 중심적인 통합 표현 역할을 하는 여러 첨단 인식 기술의 융합이다. Kudan Visual SLAM과 같은 시스템은 매핑 프로세스를 위한 견고한 지역화를 제공하기 위해 `nvblox`와 통합되고 있다.52 FoundationPose 12와 같은 포즈 추정 및 FoundationStereo 12와 같은 깊이 추정을 위한 파운데이션 모델들은 더 풍부한 입력을 제공할 것이다.

`nvblox_torch`의 개발은 단순한 편의성 제공을 넘어, 고전 로보틱스(C++, ROS, 기하학)의 세계와 현대 AI 연구(Python, PyTorch, 파운데이션 모델)의 세계를 잇는 전략적인 다리 역할을 한다. AI 연구자들은 이제 CUDA/C++ 전문가가 되지 않고도 새로운 신경망 특징을 3D 지도에 융합하는 프로토타입을 쉽게 만들고 실험할 수 있다. 이는 AI 연구자들이 로보틱스 문제에 참여하는 진입 장벽을 극적으로 낮추고, 로봇 공학자들이 최신 AI 모델을 활용할 수 있게 하여 의미론적 매핑 분야의 혁신 속도를 가속화할 것이다.

궁극적으로 `nvblox`의 발전 방향은 체화된 AI(embodied AI)를 위한 공간 메모리 중추가 되는 것이다. `nvblox`가 생성하는 풍부한 의미론적 3D 지도는 장기적인 작업 계획, 인간-로봇 상호작용, 그리고 복잡한 환경 추론을 가능하게 하는 이상적인 자료 구조이다. 이는 로봇이 자신의 세계에 대한 지속적이고 맥락적인 이해를 구축하게 하여, 원시적인 인식과 지능적인 행동 사이의 간극을 메우는 핵심 요소이다. `nvblox`의 진화는 범용 로봇 구축을 위한 세계 모델을 제공하려는 NVIDIA의 최종 목표를 시사한다. 거대한 사전 훈련된 파운데이션 모델의 특징을 지속적인 3D 지도에 융합할 수 있게 함으로써, `nvblox`는 그러한 세계 모델을 구축하기 위한 핵심 구성 요소를 제공하고 있다. 이는 NVIDIA의 기술 스택을 오늘날의 AMR과 로봇 팔을 위한 솔루션일 뿐만 아니라, 차세대 진정으로 지능적인 자율 시스템을 위한 필수 플랫폼으로 자리매김하게 한다. `nvblox`는 AI가 물리적 세계를 이해하는 내용을 그려나갈 캔버스인 셈이다.

#### 참조 문서

1. Isaac ROS Nvblox - isaac_ros_docs documentation, accessed August 9, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
2. NVIDIA-ISAAC-ROS/isaac_ros_nvblox: NVIDIA-accelerated 3D scene reconstruction and Nav2 local costmap provider using nvblox - GitHub, accessed August 9, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox
3. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, accessed August 9, 2025, https://github.com/nvidia-isaac/nvblox
4. Nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 9, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
5. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 9, 2025, https://arxiv.org/html/2311.00626v2
6. nvblox/README.md at public / nvidia-isaac/nvblox - GitHub, accessed August 9, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/README.md?plain=1
7. [2311.00626] nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 9, 2025, https://arxiv.org/abs/2311.00626
8. Isaac ROS (Robot Operating System) - NVIDIA Developer, accessed August 9, 2025, https://developer.nvidia.com/isaac/ros
9. Release Notes - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 9, 2025, https://nvidia-isaac-ros.github.io/releases/index.html
10. Isaac Manipulator Reference Architecture - isaac_ros_docs documentation, accessed August 9, 2025, https://nvidia-isaac-ros.github.io/reference_workflows/isaac_manipulator/reference_architecture.html
11. While it's used in industry sometimes, ROS is really one of those by-and-for-aca... | Hacker News, accessed August 9, 2025, https://news.ycombinator.com/item?id=42759854
12. R²D²: Building AI-based 3D Robot Perception and Mapping with NVIDIA Research, accessed August 9, 2025, https://developer.nvidia.com/blog/r2d2-building-ai-based-3d-robot-perception-and-mapping-with-nvidia-research/
13. Technical Details - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 9, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
14. OctoMap: A Probabilistic, Flexible, and Compact 3D Map Representation for Robotic Systems - ResearchGate, accessed August 9, 2025, https://www.researchgate.net/publication/235008236_OctoMap_A_Probabilistic_Flexible_and_Compact_3D_Map_Representation_for_Robotic_Systems
15. OctoMap - 3D occupancy mapping, accessed August 9, 2025, https://octomap.github.io/
16. Truncated Signed Distance Function | by Simsangcheol - Medium, accessed August 9, 2025, https://medium.com/@sim30217/truncated-signed-distance-function-f765a0f1d432
17. Truncated signed distance function (TSDF) [4] | Download Scientific Diagram, accessed August 9, 2025, https://www.researchgate.net/figure/Truncated-signed-distance-function-TSDF-4_fig2_368009629
18. Motion Segmentation of Truncated Signed Distance Function based Volumetric Surfaces - Department of Computing, accessed August 9, 2025, http://www.doc.ic.ac.uk/~bglocker/pdfs/perera2015wacv.pdf
19. Truncated Signed Distance Function Volume Integration Based on Voxel-Level Optimization for 3D Reconstruction - IS&T | Library, accessed August 9, 2025, https://library.imaging.org/admin/apis/public/api/ist/website/downloadArticle/ei/28/21/art00006
20. nvblox/docs/pages/technical.md at public - GitHub, accessed August 9, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/docs/pages/technical.md
21. (Open Access) Voxblox: Building 3D Signed Distance Fields for Planning (2016) | Helen Oleynikova | 29 Citations - SciSpace, accessed August 9, 2025, https://scispace.com/papers/voxblox-building-3d-signed-distance-fields-for-planning-20bu5qcvaw
22. NVIDIA-Isaac-ROS-Nvblox/docs/topics-and-services.md at main - GitHub, accessed August 9, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/topics-and-services.md
23. VoxelCache: Accelerating Online Mapping in Robotics and 3D Reconstruction Tasks - arXiv, accessed August 9, 2025, https://arxiv.org/pdf/2210.08729
24. Real-time 3D Reconstruction at Scale using Voxel Hashing | Request PDF - ResearchGate, accessed August 9, 2025, https://www.researchgate.net/publication/262166855_Real-time_3D_Reconstruction_at_Scale_using_Voxel_Hashing
25. Real-time 3D Reconstruction at Scale using Voxel Hashing - Matthias Niessner, accessed August 9, 2025, https://niessnerlab.org/papers/2013/4hashing/niessner2013hashing.pdf
26. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, accessed August 9, 2025, https://arxiv.org/html/2410.21149v1
27. (PDF) coVoxSLAM: GPU Accelerated Globally Consistent Dense SLAM - ResearchGate, accessed August 9, 2025, https://www.researchgate.net/publication/385318409_coVoxSLAM_GPU_Accelerated_Globally_Consistent_Dense_SLAM
28. Chisel: Real Time Large Scale 3D Reconstruction Onboard a Mobile Device using Spatially Hashed Signed Distance Fields - ResearchGate, accessed August 9, 2025, https://www.researchgate.net/publication/314582222_Chisel_Real_Time_Large_Scale_3D_Reconstruction_Onboard_a_Mobile_Device_using_Spatially_Hashed_Signed_Distance_Fields
29. ethz-asl/voxblox: A library for flexible voxel-based mapping, mainly focusing on truncated and Euclidean signed distance fields. - GitHub, accessed August 9, 2025, https://github.com/ethz-asl/voxblox
30. Performance - voxblox documentation - Read the Docs, accessed August 9, 2025, https://voxblox.readthedocs.io/en/latest/pages/Performance.html
31. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping | Request PDF, accessed August 9, 2025, https://www.researchgate.net/publication/382991572_nvblox_GPU-Accelerated_Incremental_Signed_Distance_Field_Mapping
32. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - Semantic search for arXiv papers with AI, accessed August 9, 2025, https://axi.lims.ac.uk/paper/2311.00626
33. Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox - GitHub, accessed August 9, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox
34. NVIDIA Isaac ROS Developer Preview 3 Enables Developers to Build High-Performance Robotics Applications - Robotics 24/7, accessed August 9, 2025, https://www.robotics247.com/article/nvidia_isaac_ros_developer_preview_3_enables_developers_to_build_high_performance_robotics_applications
35. Nvblox decay rate - Isaac ROS - NVIDIA Developer Forums, accessed August 9, 2025, https://forums.developer.nvidia.com/t/nvblox-decay-rate/304609
36. How build and save map 3D reconstruction to use with nvblox nav2 - Isaac ROS, accessed August 9, 2025, https://forums.developer.nvidia.com/t/how-build-and-save-map-3d-reconstruction-to-use-with-nvblox-nav2/308972
37. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, accessed August 9, 2025, https://arxiv.org/abs/1611.03631
38. OctoMap: An Efficient Probabilistic 3D Mapping Framework Based on Octrees - Washington, accessed August 9, 2025, https://courses.cs.washington.edu/courses/cse571/16au/slides/hornung13auro.pdf
39. Kimera: an Open-Source Library for Real-Time Metric-Semantic Localization and Mapping - MIT, accessed August 9, 2025, https://www.mit.edu/~arosinol/papers/Rosinol20icra-Kimera.pdf
40. MIT-SPARK/Kimera: Index repo for Kimera code - GitHub, accessed August 9, 2025, https://github.com/MIT-SPARK/Kimera
41. Kimera-Multi: Robust, Distributed, Dense Metric-Semantic SLAM for Multi-Robot Systems, accessed August 9, 2025, https://mit.edu/sparklab/2023/08/25/Kimera-Multi__Robust_Distributed_Dense_Metric-Semantic_SLAM_for_Multi-Robot-Systems.html
42. Problems with Isaac ROS + nvblox - NVIDIA Developer Forums, accessed August 9, 2025, https://forums.developer.nvidia.com/t/problems-with-isaac-ros-nvblox/237630
43. Nvblox Nav2 Implementation - Isaac ROS - NVIDIA Developer Forums, accessed August 9, 2025, https://forums.developer.nvidia.com/t/nvblox-nav2-implementation/272598
44. Issues / nvidia-isaac/nvblox - GitHub, accessed August 9, 2025, https://github.com/nvidia-isaac/nvblox/issues
45. Issues / NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, accessed August 9, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues
46. NVIDIA-Isaac-ROS-Nvblox/docs/troubleshooting-nvblox-realsense.md at main - GitHub, accessed August 9, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/troubleshooting-nvblox-realsense.md
47. Nvblox not working in realtime - Isaac ROS - NVIDIA Developer Forums, accessed August 9, 2025, https://forums.developer.nvidia.com/t/nvblox-not-working-in-realtime/323644
48. nvblox issue with isaac_sim_4.0.0 docker / Issue #348 / NVlabs/curobo - GitHub, accessed August 9, 2025, https://github.com/NVlabs/curobo/issues/348
49. isaac_ros_nvblox - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 9, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/isaac_ros_nvblox/index.html
50. Why do we need to run run_dev.sh when installing isaac_ros_nvblox? - Isaac ROS, accessed August 9, 2025, https://forums.developer.nvidia.com/t/why-do-we-need-to-run-run-dev-sh-when-installing-isaac-ros-nvblox/309511
51. how to use with stereolabs zed camera ? / Issue #6 / nvidia-isaac/nvblox - GitHub, accessed August 9, 2025, https://github.com/nvidia-isaac/nvblox/issues/6
52. Towards Next-Gen Autonomous Mobile Robotics: A Technical Deep Dive into Visual-Data-Driven AMRs Powered by Kudan Visual SLAM and NVIDIA Isaac Perceptor, accessed August 9, 2025, https://www.kudan.io/blog/a-technical-deep-dive-into-visual-data-driven-amrs-powered-by-kdvisual-and-nvidia-isaac-perceptor/


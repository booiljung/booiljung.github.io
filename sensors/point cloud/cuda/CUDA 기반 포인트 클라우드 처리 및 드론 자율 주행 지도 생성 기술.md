# CUDA 기반 포인트 클라우드 처리 및 드론 자율 주행 지도 생성 기술

## 1. 서론

자율 주행 드론 기술은 물류, 감시, 농업, 재난 구조 등 다양한 산업 분야에서 혁신을 주도하고 있다.1 이러한 드론이 복잡하고 동적인 환경에서 안전하게 임무를 수행하기 위해서는 주변 환경을 3차원으로 정밀하게 인식하고 자신의 위치를 실시간으로 파악하는 능력이 필수적이다. LiDAR, RGB-D 카메라 등에서 생성되는 포인트 클라우드는 환경의 기하학적 정보를 가장 풍부하게 담고 있는 핵심 데이터 형태이다.4

그러나 포인트 클라우드는 수십만에서 수백만 개의 3D 좌표로 구성된 대용량 데이터로, 이를 실시간으로 처리하는 것은 상당한 계산 부하를 유발한다.6 특히, 전력과 탑재 중량에 제약이 있는 드론의 임베디드 시스템에서는 이 문제가 더욱 심화된다.7 무선 통신을 통해 지상 시스템에서 처리하는 방법도 있으나, 이는 통신 가능 범위 내에서만 작동하는 한계를 가진다.7

이러한 계산적 한계를 극복하기 위한 해결책으로 NVIDIA의 CUDA(Compute Unified Device Architecture)가 부상하였다. CUDA는 GPU의 대규모 병렬 처리 능력을 범용 계산에 활용할 수 있게 하는 플랫폼으로, 포인트 클라우드 데이터의 고유한 병렬성을 활용하여 처리 속도를 획기적으로 향상시킬 수 있는 핵심 기술이다.7

본 보고서는 CUDA를 기반으로 한 포인트 클라우드 처리 기술의 기본 원리부터 시작하여, 주요 라이브러리, SLAM(동시적 위치추정 및 지도작성) 알고리즘, 딥러닝 기반 인식 모델, 그리고 이를 실제 드론에 탑재하기 위한 하드웨어 플랫폼까지 포괄적으로 고찰한다. 각 기술의 핵심 개념, CUDA 가속 원리, 성능 벤치마크, 장단점을 심층적으로 분석하여 드론 자율 주행 시스템 개발자 및 연구자에게 실질적인 통찰과 방향성을 제공하는 것을 목표로 한다.

## 1.  CUDA 기반 포인트 클라우드 처리의 기본 원리

### 1.1  포인트 클라우드의 병렬 처리 적합성

포인트 클라우드 데이터는 본질적으로 병렬 처리에 매우 적합한 구조를 가진다. 이는 수많은 독립적인 점(point)의 집합으로 구성되어 있기 때문이다. 각 점에 대한 변환, 필터링, 특징 계산 등의 연산은 다른 점의 상태에 영향을 받지 않고 독립적으로 수행될 수 있다. 이러한 특성은 CUDA의 SIMD(Single Instruction, Multiple Data) 실행 모델과 완벽하게 부합한다.10 SIMD 모델은 수천 개의 GPU 코어에 각각의 포인트 또는 포인트 그룹을 할당하고, 모든 코어가 동일한 연산(커널 함수)을 각기 다른 데이터에 대해 동시에 실행하는 방식이다. 이를 통해 CPU의 순차 처리 방식으로는 상상하기 어려운 막대한 처리량(throughput)을 달성할 수 있다.9

### 1.2  CUDA 아키텍처와 메모리 최적화

CUDA 프로그래밍 모델은 커널(GPU에서 실행되는 함수), 스레드(연산의 최소 단위), 블록(스레드의 집합), 그리드(블록의 집합)라는 계층적 구조를 가진다.7 포인트 클라우드 처리에서는 이러한 구조를 효율적으로 매핑하는 것이 성능의 관건이다. 예를 들어, 각 스레드가 하나의 포인트를 처리하도록 설계하거나, 하나의 스레드 블록이 포인트 클라우드의 특정 공간 영역, 가령 복셀(voxel) 하나를 담당하도록 구성할 수 있다.

성능에 결정적인 영향을 미치는 또 다른 요소는 GPU 메모리 계층 구조의 최적화다. GPU는 상대적으로 느린 전역 메모리(Global Memory)와 매우 빠른 공유 메모리(Shared Memory), 그리고 특정 접근 패턴에 유리한 텍스처 메모리(Texture Memory) 등을 갖추고 있다. 특히, k-최근접 이웃(k-NN) 탐색과 같이 인접한 포인트 데이터에 대한 반복적인 접근이 필요한 연산에서는, 관련 데이터를 공유 메모리에 미리 올려놓고 사용함으로써 전역 메모리 접근을 최소화하여 성능을 크게 향상시킬 수 있다. 또한, 불규칙한 메모리 접근 패턴으로 인해 발생하는 성능 저하를 완화하기 위해 텍스처 메모리를 활용하는 기법도 SLAM 알고리즘의 입자 가중치 계산 등에서 효과적인 것으로 나타났다.11 만약 메모리 접근이 병합(coalesced)되지 않고 분산되어 발생하면 GPU의 성능이 급격히 저하될 수 있으므로, 데이터 구조 설계 시 이를 반드시 고려해야 한다.12

이러한 CUDA의 특성은 단순히 기존 CPU 알고리즘을 가속하는 것을 넘어, 알고리즘 설계 패러다임 자체를 변화시키고 있다. 초기에는 CPU에서 개발된 순차적 알고리즘을 CUDA로 이식하는 시도가 많았으나, 복잡한 제어 흐름이나 불규칙한 메모리 접근 패턴은 GPU의 장점을 상쇄시켰다. 이로 인해 최근 연구들은 문제 정의 단계부터 대규모 병렬 처리를 염두에 두고 알고리즘을 재구성하는 방향으로 나아가고 있다. 예를 들어, 3D 공간을 직접 탐색하는 대신 포인트 클라우드를 2D 이미지로 투영하여 고도로 최적화된 2D CNN을 적용하거나 6, 포인트를 복셀(voxel) 14 또는 필라(pillar) 4와 같은 정형화된 데이터 구조로 변환하여 처리하는 방식이 대표적이다. 이는 알고리즘 개발의 무게 중심이 순차적 논리에서 데이터 구조와 병렬 패턴 설계로 이동하고 있음을 보여준다.

### 1.3  NVIDIA DriveWorks SDK: 저수준 모듈의 역할과 구현

NVIDIA가 자율주행 차량용으로 제공하는 DriveWorks SDK는 포인트 클라우드 처리를 위한 최적화된 CUDA 기반 저수준 모듈들을 포함하며, 이는 드론 애플리케이션에도 적용 가능한 핵심 기반 기술을 제시한다.15 주요 모듈의 기능은 다음과 같다.15

- **Accumulation:** 여러 LiDAR 패킷을 순차적으로 수집하여 하나의 완전한 포인트 클라우드를 생성한다. 이 과정에서 드론의 움직임으로 인해 발생하는 모션 왜곡(motion distortion)을 최적으로 보정하는 기능이 포함되어 있다. 또한, 생성된 포인트 클라우드를 2D 그리드 형태로 구성하여 후속 처리 단계의 효율을 높인다.15
- **Stitching:** 다수의 LiDAR 센서로부터 수집된 여러 포인트 클라우드를 하나의 공통 좌표계로 정합(stitching)한다. 센서 간의 시간적, 공간적 오차를 보정하여 일관성 있는 통합 지도를 생성하는 데 필수적이다.15
- **Iterative Closest Point (ICP):** 두 포인트 클라우드 간의 상대적인 변환(회전 및 이동)을 정밀하게 계산하는 알고리즘이다. 이는 SLAM의 주행 거리 측정(odometry) 단계에서 핵심적인 역할을 한다. DriveWorks는 KD-Tree 기반과 Depth Map 기반의 두 가지 ICP 변형을 제공하여 사용 사례에 맞는 최적의 선택지를 제공한다.15
- **Plane Extractor:** RANSAC(Random Sample Consensus) 알고리즘을 사용하여 포인트 클라우드 내에서 지면이나 벽과 같은 평면을 추출한다. 이는 지면 제거(ground removal)나 환경의 구조적 특징을 파악하는 데 유용하게 사용된다.15

이러한 모듈들은 NVIDIA의 하드웨어(Jetson)부터 소프트웨어(CUDA, SDK)에 이르는 수직 통합 생태계의 일부이다. 개발자 입장에서는 최적화된 솔루션을 통해 개발 시간을 단축하고 높은 성능을 쉽게 확보할 수 있다는 장점이 있다. 하지만 이는 동시에 NVIDIA 플랫폼에 대한 강한 기술적 종속성을 야기하는 양면성을 지닌다. 과거 NVIDIA가 CUDA-PCL 라이브러리를 소스 코드 없이 바이너리로만 배포하여 커뮤니티에서 논란이 되었던 사례는 이러한 종속성의 단면을 보여준다.16 개발자는 내부 구현을 알 수 없어 디버깅이나 확장이 어렵고, 특정 플랫폼에 묶이게 될 위험이 있다. 따라서 개발자는 NVIDIA 생태계가 제공하는 편의성과 특정 벤더에 대한 종속성 사이에서 전략적인 선택을 해야 하며, PCL이나 Open3D와 같은 오픈소스 대안과의 장단점을 신중히 비교해야 한다.

## 2.  주요 포인트 클라우드 라이브러리의 CUDA 가속

### 2.1  Point Cloud Library (PCL)의 GPU 모듈 (`pcl::gpu`)

PCL은 포인트 클라우드 처리 분야에서 오랫동안 사실상의 표준으로 자리 잡은 오픈소스 라이브러리다. 초기부터 GPU 가속을 위한 `pcl::gpu` 모듈을 포함하고 있었으나, 이 모듈은 오랫동안 불안정하고 제한적인 기능만을 제공했다.18 워프(warp) 수준의 저수준 CUDA 프로그래밍으로 구현되어 있어 유지보수가 어렵고 커뮤니티의 지원도 상대적으로 부족했다.18

핵심 클래스인 `pcl::gpu::Octree`는 Octree 자료구조를 GPU 상에서 병렬로 구축하고, 다수의 쿼리를 동시에 처리하는 배치 탐색(batch search)을 지원한다.19 반경 탐색(Radius Search)과 근사 최근접 이웃 탐색(Approximate Nearest Search)을 GPU에서 병렬로 수행할 수 있지만, 정확한 K-최근접 이웃(k-NN) 탐색은 k=1인 경우에만 지원하는 등 기능적 한계가 명확하다.19 최근 Google Summer of Code(GSoC) 등을 통해 오래된 CUDA 코드를 현대화하고 기능을 개선하려는 노력이 이루어지고는 있으나 18, 여전히 특정 버전의 CUDA 툴킷, 컴파일러, 드라이버에 대한 복잡한 의존성 문제로 인해 초보자가 사용하기에는 진입 장벽이 높다.20

이러한 상황에서 NVIDIA는 자체적으로 PCL의 핵심 기능을 CUDA로 가속화한 `CUDA-PCL` 라이브러리를 별도로 배포했다.16 이 라이브러리는 ICP, Segmentation, Filter 모듈을 제공하며, 기존 PCL의 CPU 구현 대비 최대 10배 이상의 성능 향상을 보여주었다.16 하지만 이 라이브러리는 소스 코드 없이 바이너리로만 제공되어 디버깅이나 커스터마이징이 어렵고, 지원하는 포인트 타입이 불분명하다는 비판을 받으며 커뮤니티와의 완전한 통합에는 실패했다.17

### 2.2  Open3D의 텐서 기반 GPU 아키텍처

Open3D는 3D 데이터 처리를 위해 등장한 현대적인 오픈소스 라이브러리로, 처음부터 딥러닝 프레임워크와의 유기적인 통합을 목표로 설계되었다.14 Open3D의 가장 큰 특징은 레거시 API와 텐서 기반 API를 모두 제공한다는 점이다. 초기 `open3d.geometry.PointCloud`는 NumPy 배열과 유사하게 CPU에서 주로 동작했지만 22, 최신 `open3d.t.geometry.PointCloud`는 PyTorch 텐서와 유사한 개념을 도입하여 데이터가 상주할 디바이스(CPU 또는 CUDA)를 명시적으로 지정할 수 있게 했다.24

CUDA를 활용하는 방법은 매우 직관적이다. `o3d.t.geometry.PointCloud(device="cuda:0")`와 같이 객체를 생성할 때 CUDA 디바이스를 지정하기만 하면 된다. 이렇게 하면 포인트 클라우드의 모든 데이터(위치, 색상, 법선 등)가 GPU 메모리에 직접 생성되며, 이후에 수행되는 모든 연산, 예를 들어 복셀 다운샘플링, 법선 추정, 클러스터링 등은 자동으로 GPU에서 가속화된다.24

이러한 텐서 기반 구조는 PyTorch나 TensorFlow와 같은 딥러닝 프레임워크와의 데이터 공유를 극도로 효율적으로 만든다. GPU 메모리에 있는 포인트 클라우드 데이터를 CPU로 복사하는 오버헤드 없이(zero-copy), 딥러닝 모델의 입출력으로 직접 사용할 수 있어 엔드투엔드 GPU 파이프라인 구축에 이상적이다.25 실제로 Open3D는 PointNet++와 같은 모델의 커스텀 TensorFlow 연산자(op)를 가속화하는 데 사용되기도 했다.26

### 2.3  라이브러리 비교 분석

각 라이브러리의 발전 과정과 아키텍처 철학은 그들이 지향하는 목표가 다르다는 것을 보여준다. PCL은 전통적인 컴퓨터 비전 알고리즘의 구현체 집합에 가깝다. `pcl::gpu` 역시 이러한 알고리즘들을 GPU로 이식하는 데 초점을 맞췄다. 반면, Open3D의 `o3d.t.geometry`는 알고리즘 자체보다는 '데이터 컨테이너'로서의 `PointCloud` 텐서에 집중한다.24 이 텐서는 CPU나 GPU에 상주하며 다양한 속성을 가질 수 있고, 그 자체로 딥러닝 프레임워크의 일부처럼 동작한다. 이는 포인트 클라우드 라이브러리의 발전 방향이 '미리 정의된 알고리즘의 집합'에서 '유연한 데이터 구조와 기본 연산자를 제공하는 딥러닝 백엔드'로 진화하고 있음을 시사한다. 개발자는 이 백엔드를 기반으로 자신만의 복잡한 모델과 파이프라인을 자유롭게 구축할 수 있다.

아래 표는 각 라이브러리의 특징을 요약하여 개발자가 프로젝트 요구사항에 맞는 최적의 도구를 선택하는 데 도움을 준다.

**표 1: CUDA 가속 포인트 클라우드 라이브러리 비교**

| 구분               | PCL (`pcl::gpu`)                     | CUDA-PCL (NVIDIA)                        | Open3D (`o3d.t.geometry`)                |
| ------------------ | ------------------------------------ | ---------------------------------------- | ---------------------------------------- |
| **아키텍처 철학**  | 전통적 C++ 객체 지향                 | PCL 기능의 직접 가속                     | 텐서 기반, 딥러닝 친화적                 |
| **주요 가속 기능** | Octree 탐색, 일부 필터링 19          | ICP, Segmentation, PassThrough Filter 16 | 거의 모든 기하 연산, 딥러닝 op 22        |
| **딥러닝 통합성**  | 낮음 (데이터 변환 필요)              | 낮음 (데이터 변환 필요)                  | 매우 높음 (PyTorch/TF와 제로카피) 25     |
| **개발 편의성**    | 복잡함 (설치 및 사용) 20             | 중간 (API는 간단하나 블랙박스) 17        | 높음 (Pythonic, 직관적 API) 14           |
| **소스 공개**      | 공개 (Open Source)                   | 비공개 (Binary only) 17                  | 공개 (Open Source)                       |
| **적합한 용도**    | 레거시 PCL 시스템의 부분적 성능 개선 | Jetson 환경에서 특정 PCL 기능의 고속화   | 신규 R&D, 딥러닝 기반 3D 비전 파이프라인 |

## 3.  자율 주행 드론을 위한 CUDA 가속 SLAM 기술

SLAM(Simultaneous Localization and Mapping)은 드론이 미지의 환경에서 자신의 위치를 추정(Localization)하면서 동시에 지도를 작성(Mapping)하는 기술의 총칭이다.27 SLAM 시스템은 일반적으로 실시간 센서 데이터를 처리하여 단기적인 움직임을 추정하는 **프론트엔드(front-end)**와, 지금까지의 전체 궤적과 지도의 일관성을 유지하기 위해 전역 최적화를 수행하는 **백엔드(back-end)**로 구성된다.30 CUDA는 이 두 단계의 계산 집약적인 부분을 가속화하여 드론의 실시간 자율 주행을 가능하게 하는 데 결정적인 역할을 한다.

### 3.1  Visual SLAM (VSLAM) 및 Visual-Inertial Odometry (VIO)

카메라를 주 센서로 사용하는 VSLAM과 VIO는 저렴하고 가볍다는 장점 때문에 드론에 널리 적용된다.

- **cuVSLAM:** NVIDIA가 직접 개발한 VSLAM 라이브러리로, 전체 파이프라인이 CUDA 가속에 극도로 최적화되어 있다.30 프론트엔드에서는 Lucas-Kanade 알고리즘을 변형한 특징점 추적, "Good Features to Track" 기반의 특징점 선택, 스테레오 매칭 등이 모두 GPU에서 병렬로 처리된다. 백엔드에서는 자세 그래프 최적화(Pose Graph Optimization)와 번들 조정(Bundle Adjustment)과 같은 무거운 전역 최적화 과정 또한 CUDA를 통해 가속화된다. 그 결과, NVIDIA Jetson과 같은 임베디드 플랫폼에서도 실시간 프레임 처리가 가능하며, KITTI, EuRoC와 같은 표준 데이터셋에서 낮은 오차율과 높은 성능을 입증했다.30

- **ORB-SLAM:** 가장 널리 사용되는 오픈소스 VSLAM 알고리즘 중 하나로, ORB 특징점을 사용한다.7 ORB 특징점 추출 과정은 다수의 FAST 코너 검출과 BRIEF 디스크립터 계산으로 구성되는데, 이 과정은 이미지 패치 단위로 병렬화가 가능하여 CUDA 가속에 매우 적합하다.9 OpenCV의 

  `GpuMat`을 사용하여 이미지 피라미드를 생성하고 특징점을 추출하는 등, 커뮤니티를 중심으로 CUDA를 활용한 병렬화 연구가 활발히 진행되고 있다.7

- **VINS-Mono:** 관성 측정 장치(IMU)와 단안 카메라를 강하게 결합(Tightly-coupled)한 VIO 프레임워크로, 두 센서의 장점을 상호 보완한다.32 IMU는 높은 빈도로 출력되어 단기적인 움직임 예측에 강하고, 카메라는 드리프트 누적을 보정해준다.34 VINS-Mono의 핵심인 비선형 최적화 과정은 계산량이 많아 CUDA 가속의 잠재력이 크며, 특히 특징점 추적(LK optical flow)과 백엔드 최적화 부분을 GPU로 오프로딩하여 임베디드 장치에서의 실시간 성능을 확보하려는 연구가 진행되고 있다.32

### 3.2  LiDAR SLAM 및 LiDAR-Inertial Odometry (LIO)

LiDAR는 카메라보다 정밀한 거리 정보를 제공하여 더 정확하고 강건한 SLAM을 가능하게 한다.

- **LIO-SAM:** LiDAR, IMU, GPS 데이터를 팩터 그래프(Factor Graph) 기반으로 강하게 결합하는 최신 LIO 알고리즘이다.36 핵심 최적화 엔진으로 iSAM2(incremental Smoothing and Mapping)를 사용하여, 새로운 측정값이 들어올 때마다 전체 그래프를 재계산하는 대신 영향을 받는 부분만 효율적으로 업데이트한다.36 LIO-SAM의 원본 구현은 CPU 기반이지만, 포인트 클라우드 전처리(다운샘플링, 특징 추출), 지면 추출(CSF 필터), 스캔 매칭 등의 과정은 GPU 가속화의 대상이 된다. 한 연구에서는 CSF 필터를 GPU로 이식하여 반복 속도를 8배 향상시켰으나, CPU-GPU 간 데이터 전송 오버헤드로 인해 전체적인 실시간 성능 향상은 미미했다고 보고했다.38 이는 단순히 병목 구간만 가속하는 것의 한계를 보여주며, 알고리즘 전체를 GPU 친화적으로 재설계하는 것의 중요성을 시사한다.
- **NVIDIA Isaac Nvblox:** SLAM 자체라기보다는 로봇 경로 계획을 위해 특별히 설계된 GPU 가속 3D 매핑 라이브러리다.39 SLAM 알고리즘으로부터 추정된 드론의 위치(pose)를 입력받아, 주변 환경의 고밀도 3D 지도를 실시간으로 생성한다. Nvblox는 기존의 CPU 기반 방식(예: voxblox) 대비 TSDF(Truncated Signed Distance Field) 표면 재구성에서 최대 177배, ESDF(Euclidean Signed Distance Field) 계산에서 최대 31배의 압도적인 성능 향상을 달성했다.39 TSDF는 센서 데이터를 3D 볼륨에 통합하여 표면을 표현하고, ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 거리를 저장하여 충돌 없는 경로 계획에 필수적이다. Nvblox는 드론이 실시간으로 고밀도의 3D 장애물 지도를 생성하고 40, 이를 기반으로 안전한 경로를 계획하는 데 직접적으로 활용될 수 있다.39

### 3.3  SLAM 기술 비교

다양한 SLAM 기법들은 사용하는 센서, 핵심 원리, 그리고 CUDA 활용 방식에서 차이를 보인다. VSLAM은 저렴하지만 조명 변화나 텍스처가 부족한 환경에 취약하고, LIO는 비싸지만 정밀하고 환경 변화에 강건하다. CUDA의 활용 방식 또한 '엔드투엔드 GPU 설계(cuVSLAM)', '병목 구간 가속(ORB-SLAM)', 'CPU 최적화 기반(LIO-SAM)' 등 각기 다른 접근 방식을 취하고 있다. 아래 표는 이러한 차이점을 비교하여 특정 애플리케이션에 적합한 기술을 선택하는 데 도움을 준다.

**표 2: 주요 SLAM/VIO/LIO 알고리즘의 CUDA 활용 분석**

| 알고리즘     | 주요 센서                 | 핵심 원리                  | CUDA 활용 방식 및 범위                                       | 장점                                       | 단점                                      |
| ------------ | ------------------------- | -------------------------- | ------------------------------------------------------------ | ------------------------------------------ | ----------------------------------------- |
| **cuVSLAM**  | 스테레오 카메라, IMU      | 특징점 기반 VIO            | **엔드투엔드 가속:** 특징점 추적, 백엔드 최적화 등 전체 파이프라인 30 | Jetson에서 최고의 실시간 성능, 높은 정확도 | NVIDIA 플랫폼 종속성                      |
| **ORB-SLAM** | 단안/스테레오 카메라, IMU | ORB 특징점 기반 VSLAM      | **부분 가속:** ORB 특징점 추출 병렬화 7                      | 높은 범용성, 강력한 커뮤니티               | 전체 파이프라인 가속은 별도 구현 필요     |
| **LIO-SAM**  | LiDAR, IMU, GPS           | 팩터 그래프 최적화 (iSAM2) | **부분 가속 가능성:** 포인트 클라우드 전처리, 특징 추출 38   | 다중 센서 융합으로 높은 강건성             | 핵심 최적화 엔진은 CPU 기반, GPU화 어려움 |
| **Nvblox**   | RGB-D/LiDAR + Pose        | TSDF/ESDF 융합             | **엔드투엔드 가속:** TSDF/ESDF 생성 및 업데이트 39           | 초고속 고밀도 맵 생성, 경로 계획에 최적화  | SLAM 기능 부재, 외부 Pose 입력 필요       |

## 4.  포인트 클라우드 인식을 위한 딥러닝 모델과 CUDA

### 4.1  딥러닝 기반 인식의 패러다임 전환

전통적인 포인트 클라우드 인식은 RANSAC, ICP 등 수작업으로 설계된 특징(hand-crafted features)에 크게 의존했다. 그러나 딥러닝의 발전은 데이터로부터 직접 특징을 학습하는 엔드투엔드 방식으로 패러다임을 전환시켰다.41 포인트 클라우드는 이미지와 달리 비정형적(unstructured), 비순서적(unordered), 희소(sparse)한 특성을 가져 2D CNN을 직접 적용하기 어렵다는 근본적인 문제를 가지고 있다.4 이를 해결하기 위해 다양한 딥러닝 아키텍처가 제안되었으며, 이들의 효율적인 구현은 CUDA의 병렬 처리 능력에 크게 의존한다.

### 4.2  핵심 딥러닝 아키텍처 분석 (CUDA 기반 구현)

- PointNet/PointNet++:

  포인트 클라우드를 직접 입력으로 받는 선구적인 모델이다.43 PointNet은 각 포인트를 독립적인 MLP(Multi-Layer Perceptron)로 처리한 후, 순서에 무관한 대칭 함수(symmetric function)인 최대 풀링(max-pooling)을 통해 전체 포인트 클라우드를 대표하는 전역 특징(global feature)을 추출한다.44

  PointNet++는 이를 계층적으로 확장하여, Set Abstraction Layer라는 핵심 모듈을 통해 지역적 특징(local feature)을 학습한다.44 이 레이어는 **샘플링(Sampling) -> 그룹핑(Grouping) -> PointNet**의 3단계로 구성된다. 샘플링 단계에서는 전체 포인트 중 일부를 중심점(centroid)으로 선택하고(Farthest Point Sampling, FPS), 그룹핑 단계에서는 각 중심점 주변의 이웃 포인트들을 묶어 로컬 영역을 형성한다(Ball Query). 마지막으로 각 로컬 영역을 작은 PointNet에 통과시켜 지역 특징을 추출한다. 이 과정들은 모두 CUDA 커널로 효율적으로 구현될 수 있으며, 특히 FPS와 Ball Query 같은 연산은 병렬화의 핵심 대상이다.26

- RandLA-Net:

  수백만 개의 점으로 구성된 대규모 포인트 클라우드를 실시간으로 처리하기 위해 설계된 경량 네트워크다.42 이 모델의 핵심 아이디어는 계산 비용이 높은 FPS 대신 극도로 효율적인 **무작위 샘플링(Random Sampling)**을 사용하는 것이다.42 무작위 샘플링으로 인해 발생할 수 있는 중요한 정보의 손실은 정교하게 설계된 **로컬 특징 집계(Local Feature Aggregation)** 모듈을 통해 보완된다. 이 모듈은 k-NN을 사용하여 주변 점들의 위치 정보와 특징을 효율적으로 집계하며, 이 과정 역시 GPU에서 병렬로 처리되어 높은 효율을 보인다. PyTorch, Open3D-ML 등 다양한 프레임워크에서 구현되어 있으며, NVIDIA Jetson 플랫폼에서도 구동 사례가 보고되었다.48

- PointPillars:

  자율주행 분야에서 널리 사용되는 실시간 3D 객체 탐지 모델이다.4 포인트 클라우드를 3D 공간에서 2D 그리드로 투영한 뒤, 각 그리드 셀을 수직 기둥(Pillar)으로 간주한다. 각 필라 내부에 포함된 포인트들을 작은 신경망으로 인코딩하여 하나의 특징 벡터를 생성하고, 이를 통해 2D 의사 이미지(pseudo-image)인 

  **BEV(Bird's-Eye-View) 특징 맵**을 만든다. 이후의 객체 탐지 과정은 고도로 최적화된 2D CNN을 통해 처리되므로 매우 효율적이다.4 NVIDIA는 TensorRT에 최적화된 

  **CUDA-PointPillars**를 제공하여, 기존 구현의 병목(많은 작은 연산, TensorRT 미지원 연산자)을 해결하고 Jetson과 같은 임베디드 장치에서의 성능을 극대화했다.4

이러한 모델들의 발전 과정은 포인트 클라우드 처리의 핵심 딜레마, 즉 '어떻게 수백만 개의 점을 효율적으로 처리하면서도 중요한 기하학적 정보를 잃지 않을 것인가?'에 대한 서로 다른 해법을 보여준다. PointNet++의 FPS는 표현력을 위해 효율성을 일부 희생했고, RandLA-Net의 무작위 샘플링은 효율성을 극대화하는 대신 정교한 집계 모듈로 표현력을 보강했다. 이는 '샘플링 효율성'과 '로컬 특징 표현력' 사이의 끊임없는 기술적 투쟁의 역사라 할 수 있다.

### 4.3  희소 텐서(Sparse Tensor)와 Minkowski Engine

3D 공간을 복셀(voxel)로 나누어 처리하는 방식은 정형화된 데이터 구조를 만들어 CNN 적용을 용이하게 하지만, 대부분의 복셀이 비어있게 되어 조밀한(dense) 텐서로 표현할 경우 극심한 메모리 및 연산 낭비를 초래한다.50 이 문제를 해결하기 위해 **희소 텐서(Sparse Tensor)** 개념이 도입되었다. 희소 텐서는 0이 아닌 값의 좌표(Coordinate)와 특징(Feature)만을 리스트 형태로 저장하는 COO(Coordinate list) 형식을 사용하여 데이터를 효율적으로 표현한다.50

**Minkowski Engine**은 PyTorch를 기반으로 하는 희소 텐서용 자동 미분 라이브러리로, 고차원 희소 컨볼루션(Sparse Convolution)을 비롯한 다양한 신경망 레이어를 제공한다.52 희소 컨볼루션은 입력 희소 텐서에 존재하는 좌표에 대해서만 출력을 계산하므로, 연산량을 획기적으로 줄일 수 있다.50 Minkowski Engine은 이 과정을 고도로 최적화된 CUDA 커널로 구현하여 CPU 대비 100배 이상의 가속을 제공하며 52, 이를 통해 3D U-Net과 같은 깊은 3D CNN 모델을 고해상도 포인트 클라우드에 적용하는 것을 가능하게 했다.54

### 4.4  성능 벤치마크

딥러닝 모델의 성능은 주로 정확도 지표인 mIoU(mean Intersection over Union)와 속도 지표인 FPS(Frames Per Second)로 평가된다. 아래 표는 주요 모델들의 표준 벤치마크 데이터셋에서의 성능을 요약한 것이다. 이를 통해 연구자와 개발자는 특정 작업(예: 실시간 세분화, 고정밀 오프라인 분석)에 적합한 모델을 선택할 때, 정확도와 속도 간의 트레이드오프를 고려하여 객관적인 판단을 내릴 수 있다.

**표 3: 포인트 클라우드 딥러닝 모델 성능 벤치마크 (SemanticKITTI, ScanNet 기준)**

| 모델                | 기반 기술               | 주요 데이터셋 | mIoU (%)              | FPS / 처리 시간               | 특징                            | 소스 |
| ------------------- | ----------------------- | ------------- | --------------------- | ----------------------------- | ------------------------------- | ---- |
| **PointNet++**      | 포인트 기반 계층적 학습 | Semantic3D    | -                     | 35.82ms (RTX 3090, 변형 모델) | 선구적 모델, 로컬 특징 학습     | 6    |
| **RandLA-Net**      | 랜덤 샘플링 + 로컬 집계 | SemanticKITTI | 57.9                  | ~12 FPS (RangeNet++ 비교)     | 대규모 포인트 클라우드에 효율적 | 56   |
| **Minkowski U-Net** | 희소 컨볼루션           | ScanNet       | 76.4 AP (Person Det.) | 22.6 (Jetson AGX)             | 고해상도, 메모리 효율적         | 57   |
| **FRNet**           | 2D 투영 기반            | SemanticKITTI | 73.3                  | SOTA 대비 5배 빠름            | 2D CNN 활용, 고속               | 58   |
| **LFNet**           | 희소 텐서 + Attention   | SemanticKITTI | (기존 대비 +6.4%)     | 실시간 가능                   | 경량화, 실시간성 강조           | 55   |

## 5.  하드웨어 플랫폼: NVIDIA Jetson 및 FPGA와의 비교 분석

### 5.1  NVIDIA Jetson: 드론을 위한 엣지 AI 플랫폼

NVIDIA Jetson 시리즈는 드론과 같은 모바일 로봇에 탑재되어 실시간 AI 연산을 수행하기 위해 설계된 소형 단일 보드 컴퓨터(SBC)다.8 컴팩트한 크기와 저전력 설계에도 불구하고 강력한 GPU를 내장하고 있어, 엣지 컴퓨팅의 대표적인 플랫폼으로 자리 잡았다.

플랫폼은 성능과 가격에 따라 다양한 라인업으로 구성된다. **Jetson Nano**는 가장 기본적인 모델로, 간단한 딥러닝 모델 실행은 가능하지만 메모리(4GB) 부족으로 복잡한 3D 객체 탐지 모델 실행에는 명확한 한계를 보인다.8

**Jetson TX2**는 Nano보다 향상된 성능으로 로보틱스 애플리케이션에 널리 사용되었으나, 여전히 최신 3D 모델 구동에는 어려움이 있다.8

**Jetson Xavier NX**는 강력한 성능과 작은 폼팩터를 겸비하여 대규모 딥러닝 모델 배포에 유리하며, **Jetson AGX Xavier**는 시리즈 중 가장 강력한 하드웨어로 여러 소프트웨어 기능을 동시에 요구하는 산업용 드론에 적합하다.8 최근에 출시된 **Jetson Orin 시리즈(Nano, AGX)**는 최신 Ampere 아키텍처 GPU를 탑재하여 이전 세대 대비 AI 성능을 수십 배 향상시켰으며, 특히 Orin Nano는 기존 Nano 대비 최대 80배의 성능을 제공하며 엔트리급 엣지 AI의 새로운 표준을 제시하고 있다.61

한 벤치마크 연구에 따르면, 3D 객체 탐지 시 모든 Jetson 보드에서 GPU 자원의 80% 이상이 소모될 정도로 계산 집약적이다.8 PointPillar와 같은 모델은 AGX Xavier에서 약 10 FPS, NX에서 약 6 FPS를 기록하여 정확도와 속도 간의 현실적인 균형점을 보여주었다.8

이러한 임베디드 환경에서 성능을 극대화하기 위한 핵심 기술은 **TensorRT**이다. TensorRT는 NVIDIA의 추론 최적화 라이브러리로, 신경망의 레이어를 융합하고, 정밀도를 낮추며(FP16/INT8), 타겟 하드웨어에 최적화된 커널을 생성하여 추론 속도를 획기적으로 향상시킨다. CenterPoint 모델을 이용한 실험에서, TensorRT를 적용하자 AGX Xavier에서의 추론 속도가 4.41 FPS에서 18.4 FPS로 약 4배 향상되었고, CPU 및 메모리 사용량은 절반으로 감소했다.8 이는 제한된 자원의 임베디드 시스템에서 3D 인식과 같은 무거운 작업을 수행하면서도 경로 계획, 제어 등 다른 필수 기능을 함께 실행할 수 있는 중요한 여유 자원을 확보해준다는 것을 의미한다.

**표 4: NVIDIA Jetson 플랫폼별 3D 객체 탐지 성능 비교 (CenterPoint 모델 기준)**

| 플랫폼                | 모델               | 실행 가능 여부         | 추론 속도 (FPS)    | CPU 사용량 (%)  | 메모리 사용량 (%) |      |
| --------------------- | ------------------ | ---------------------- | ------------------ | --------------- | ----------------- | ---- |
| **Jetson Nano**       | CenterPoint (원본) | **실패 (메모리 부족)** | -                  | -               | -                 |      |
|                       | CenterPoint (TRT)  | 성공                   | (데이터 없음)      | (데이터 없음)   | (데이터 없음)     |      |
| **Jetson TX2**        | CenterPoint (원본) | **실패 (메모리 부족)** | -                  | -               | -                 |      |
|                       | CenterPoint (TRT)  | 성공                   | (데이터 없음)      | (데이터 없음)   | (데이터 없음)     |      |
| **Jetson Xavier NX**  | CenterPoint (원본) | **실패 (메모리 부족)** | -                  | -               | -                 |      |
|                       | CenterPoint (TRT)  | 성공                   | ~9.2 (추정)        | ~27%            | (데이터 없음)     |      |
| **Jetson AGX Xavier** | CenterPoint (원본) | 성공                   | 4.41               | ~60%            | ~45%              |      |
|                       | CenterPoint (TRT)  | 성공                   | **18.4 (4.17배↑)** | **~18% (감소)** | **~25% (감소)**   |      |

데이터 소스:.8 일부 값은 논문 내용 기반 추정치.

### 5.2  GPU (CUDA) vs. FPGA: 아키텍처 및 성능 비교

드론의 온보드 프로세서로 GPU 외에 FPGA(Field-Programmable Gate Array)도 고려될 수 있다. 둘은 근본적으로 다른 아키텍처와 성능 특성을 가진다.

- **아키텍처:** GPU는 수천 개의 동일한 코어가 SIMD 방식으로 동작하는 대규모 병렬 프로세서로, 소프트웨어(CUDA C++)를 통해 프로그래밍된다.10 반면, FPGA는 프로그래밍 가능한 논리 게이트의 집합으로, 하드웨어 기술 언어(HDL)나 고수준 합성(HLS)을 통해 하드웨어 회로 자체를 설계하는 방식이다. 데이터가 준비되는 즉시 파이프라인을 따라 연산이 수행되는 데이터플로우(Dataflow) 아키텍처를 가진다.10
- **성능 특성:**
  - **지연 시간(Latency):** FPGA는 특정 작업에 최적화된 파이프라인을 하드웨어로 직접 구성하므로, 매우 낮고 예측 가능한(deterministic) 지연 시간을 제공한다. 이는 1ms의 지연도 치명적일 수 있는 실시간 제어 시스템에 매우 유리하다.12 GPU는 높은 처리량(throughput)에 최적화되어 있지만, 운영체제 스케줄링이나 메모리 접근 패턴 등으로 인해 지연 시간이 변동적일 수 있다.34
  - **전력 효율성:** FPGA는 필요한 만큼의 논리 회로만 활성화하여 사용하므로, 동일한 작업을 수행할 때 범용 코어를 사용하는 GPU보다 전력 효율성이 월등히 높다.62 이는 배터리로 동작하는 드론에 매우 중요한 장점이다.
  - **개발 복잡성:** GPU(CUDA)는 C++에 익숙한 소프트웨어 개발자에게 접근성이 높다.10 반면, FPGA는 하드웨어 설계에 대한 깊은 이해가 필요하며, 컴파일(합성 및 배치/라우팅)에 수 시간이 소요될 수 있어 개발 주기가 길고 복잡하다.12

일부 연구에서는 경량 CNN 모델을 FPGA에 구현하여 GPU(RTX 3090) 대비 2.74배의 속도 향상과 46배의 전력 효율성 개선을 달성하기도 했다.6 이는 FPGA가 특정, 고정된 연산을 반복하는 추론 작업에 매우 효율적임을 보여준다. 그러나 복잡하고 알고리즘이 계속 변하는 딥러닝 모델을 빠르게 프로토타이핑하고 테스트하는 데는 유연성이 높은 GPU가 여전히 우세하다.12

GPU와 FPGA의 경쟁은 단순히 '어느 것이 더 빠른가'의 문제가 아니라, '시스템 아키텍처 철학'의 차이로 이해해야 한다. GPU는 '소프트웨어 정의(Software-defined)' 하드웨어다. 강력한 범용 병렬 프로세서를 만들어 놓고, 그 위에서 소프트웨어를 통해 유연하게 다양한 문제를 푼다. 반면 FPGA는 '하드웨어 정의(Hardware-defined)' 솔루션이다. 풀고자 하는 문제에 맞춰 하드웨어 회로 자체를 '조각'하여 특정 작업에 대한 극도의 효율성을 추구한다. 따라서 미래의 드론 시스템은 이 둘이 각자의 장점을 살려 공존하는 이기종 컴퓨팅(heterogeneous computing) 아키텍처로 발전할 가능성이 높다. 예를 들어, 모델이 계속 바뀌고 복잡한 연산이 필요한 **인식(Perception)** 파트는 유연한 GPU가 담당하고, 매우 빠르고 예측 가능한 응답이 필요한 **제어(Control)** 파트는 저지연/저전력의 FPGA가 담당하는 형태가 될 수 있다.

## 6.  핵심 연산 성능 벤치마크 분석

### 6.1  정합(Registration) 알고리즘: Iterative Closest Point (ICP)

ICP는 연속적인 LiDAR 스캔이나 포인트 클라우드 지도를 정렬하여 로봇의 움직임을 추정하는 SLAM의 핵심 연산이다.65 하지만 대응점 탐색과 변환 행렬 계산을 반복적으로 수행해야 하므로 계산 비용이 매우 높다.16 CUDA 가속은 이 병목을 효과적으로 해결한다. NVIDIA의 CUDA-PCL 벤치마크에 따르면, 7,000개의 포인트로 구성된 클라우드를 20회 반복하여 ICP를 수행했을 때, CPU 기반 PCL 구현은 523.1ms가 걸린 반면, CUDA-ICP는 **55.1ms** 만에 완료하여 약 **9.5배**의 속도 향상을 보였다.16 또 다른 연구에서는 CPU에서 33ms 걸리던 연산이 GPU에서 3ms로 단축되었다고 보고했다.66 이 극적인 성능 향상은 ICP의 핵심 병목인 '대응점 탐색(correspondence search)' 과정을 GPU의 수천 개 코어에서 대규모로 병렬 처리함으로써 가능해진다. 이는 드론이 더 빠른 속도로 움직이면서도 더 조밀한 포인트 클라우드를 사용하여 자신의 위치를 더 빠르고 정확하게 갱신할 수 있음을 의미한다.

### 6.2  특징점 추출: ORB (Oriented FAST and Rotated BRIEF)

ORB는 VSLAM에서 계산 효율성과 준수한 성능으로 널리 사용되는 특징점 기술이다.9 하지만 실시간 영상 스트림에서 프레임마다 수백 개의 특징점을 추출하고 매칭하는 것은 CPU에 상당한 부담을 준다. ORB 추출의 각 단계(FAST 코너 검출, Harris 코너 점수 계산, rBRIEF 디스크립터 계산)는 픽셀 또는 이미지 패치 단위로 독립적으로 수행될 수 있어 CUDA 병렬화에 이상적이다.68

`cuda-efficient-features`라는 오픈소스 프로젝트의 벤치마크 결과, FHD(1920x1080) 이미지에서 특징점을 검출(detect)하는 데 Jetson Xavier에서 **5.6ms**, RTX 3060 Ti에서 **1.6ms**가 소요되었다.69 검출과 디스크립터 계산을 모두 포함한 `detectAndCompute` 연산은 40,000개의 키포인트에 대해 Jetson Xavier에서 약 40-50ms, RTX 3060 Ti에서 약 7-9ms가 소요되었다.69 가속화된 ORB는 드론이 더 높은 해상도의 영상을 사용하거나 더 빠른 속도로 움직이면서도 안정적으로 시각적 특징을 추적할 수 있게 해준다.

### 6.3  최적화: Pose Graph Optimization (PGO)

PGO는 SLAM 백엔드의 핵심으로, 누적된 오차를 최소화하기 위해 전체 궤적(자세 그래프)을 전역적으로 최적화하는 과정이다.70 이는 결국 거대한 비선형 최소제곱 문제(non-linear least squares problem)를 푸는 것으로 귀결되며, 대규모 희소 선형 시스템을 반복적으로 풀어야 하는 계산 집약적인 작업이다. 전통적인 PGO 솔버인 g2o, GTSAM 등은 C++ 기반으로 CPU에서 동작했다.70 하지만 최근 PyPose, Theseus와 같은 딥러닝 기반 최적화 라이브러리들은 이 희소 선형 시스템을 푸는 과정을 CUDA를 통해 가속화한다.72 예를 들어, Facebook AI Research(FAIR)에서 개발한 Theseus 라이브러리는 맞춤형 CUDA 백엔드를 통해 희소 선형 솔버를 GPU에서 가속하며, 이를 통해 CPU 솔버(CHOLMOD) 대비 수십 배의 성능 향상을 보여준다.73 PGO의 CUDA 가속은 드론이 더 크고 복잡한 환경을 탐사하더라도, 루프 클로징(loop closure)이 발생했을 때 지연 없이 전역 맵을 실시간으로 보정할 수 있게 해준다. 이는 장시간 임무 수행 시 맵의 일관성을 유지하는 데 결정적이다.

## 7.  결론 및 향후 전망

본 보고서는 CUDA가 포인트 클라우드 처리와 드론 자율 주행 지도 생성의 전 영역에 걸쳐 필수적인 기술로 자리 잡았음을 다각도로 입증했다. 저수준 라이브러리(PCL, Open3D)의 가속부터 고수준 애플리케이션인 SLAM(cuVSLAM, Nvblox) 및 딥러닝(PointNet++, Minkowski Engine)에 이르기까지, CUDA는 임베디드 환경의 계산적 한계를 극복하고 실시간 3D 인식 및 매핑을 가능하게 하는 핵심 동력이다.

기술적 관점에서 현재 시점의 최적 스택을 제안하자면, **NVIDIA Jetson Orin 시리즈** 61를 하드웨어 플랫폼으로, **Open3D** 14를 포인트 클라우드 처리 및 딥러닝 파이프라인의 백엔드로,  **Minkowski Engine** 50을 이용한 희소 3D CNN을 인식 모델로, 그리고  **Nvblox** 39를 실시간 고밀도 매핑 및 경로 계획에 활용하는 조합이 가장 현대적이고 강력한 성능을 제공할 것으로 판단된다. 그러나 개발자는 항상 성능, 전력 소모, 개발 용이성 간의 트레이드오프를 인지해야 한다. 최고의 성능을 위해서는 NVIDIA의 최적화된 솔루션이 매력적이지만, 유연성과 확장성을 위해서는 오픈소스 기반의 접근이 유리할 수 있으며, 극도의 저지연과 저전력이 요구되는 특정 제어 루프에는 FPGA의 적용을 고려해볼 가치가 있다.13

향후 기술은 다음과 같은 방향으로 발전할 것으로 전망된다.

첫째, **새로운 3D 표현 방식의 부상**이다. 포인트 클라우드를 넘어, NeRF(Neural Radiance Fields)와 3D 가우시안 스플래팅(Gaussian Splatting)과 같은 새로운 방식이 고품질 3D 장면 재구성과 렌더링에서 두각을 나타내고 있다.74 이러한 기술들은 본질적으로 신경망 기반이므로, 이들의 학습과 추론을 가속화하기 위한 CUDA의 역할은 더욱 중요해질 것이다. NVIDIA는 이미 InstantSplat과 같은 관련 도구를 발표하며 이 분야를 선도하고 있다.74

둘째, **AI 기반 SLAM의 심화**이다. 전통적인 기하학 기반 SLAM에 딥러닝이 더욱 깊숙이 통합될 것이다. 예를 들어, 딥러닝으로 특징점을 더 강건하게 추출하거나 33, 장면의 의미론적 정보(semantic information)를 활용하여 루프 클로징의 정확도를 높이는 연구가 활발하다.38 이는 결국 더 많은 AI 연산을 요구하며, 엣지 디바이스에서의 CUDA 성능을 더욱 중요하게 만들 것이다.

셋째, **이기종 컴퓨팅의 보편화**이다. 미래의 드론 시스템은 단일 프로세서가 아닌, 다양한 작업을 분담하는 이기종 컴퓨팅 아키텍처로 발전할 것이다. 인식과 고수준 계획은 강력한 GPU가, 실시간 제어와 센서 데이터의 저수준 처리는 저지연 DSP나 FPGA가 담당하는 구조가 보편화될 것이다. 이러한 복잡한 시스템을 효율적으로 프로그래밍하고 조율하는 것이 미래의 핵심 과제가 될 것이다.

#### 참고자료

1. CubeDN: Real-time Drone Detection in 3D Space from Dual mmWave Radar Cubes - UCL Discovery, accessed August 4, 2025, https://discovery.ucl.ac.uk/10204066/1/ICRA25_1026_Submitted.pdf
2. 포인트 클라우드, 로봇 혁신을 이끄는 활용 사례 TOP 5 - Everything is NORMAL - 티스토리, accessed August 4, 2025, https://23min.tistory.com/11
3. Real-Time Semantic Segmentation for UAV Perspectives on Embedded Platforms, accessed August 4, 2025, https://www.researchgate.net/publication/393979445_Real-Time_Semantic_Segmentation_for_UAV_Perspectives_on_Embedded_Platforms
4. Detecting Objects in Point Clouds with NVIDIA CUDA-Pointpillars | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/detecting-objects-in-point-clouds-with-cuda-pointpillars/
5. [논문]LiDAR 기반 포인트 클라우드 획득 및 전처리 - 한국과학기술정보연구원, accessed August 4, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO202116553486877
6. Real-Time LiDAR Point-Cloud Moving Object Segmentation for Autonomous Driving - MDPI, accessed August 4, 2025, https://www.mdpi.com/1424-8220/23/1/547
7. Compuer Unified Device Architecture (CUDA)- Accelerated Visual SLAM for UAVs - IJARSE, accessed August 4, 2025, http://www.ijarse.com/images/fullpdf/1516450748_VCET335IJARSE.pdf
8. Run Your 3D Object Detector on NVIDIA Jetson Platforms:A ..., accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10144830/
9. Accelerating Feature Extraction and Image Stitching Algorithm Using Nvidia CUDA, accessed August 4, 2025, https://neumanncondition.com/articles/accelerating-feature-extraction-and-image-stitching-algorithm/
10. What are the key differences between CUDA and FPGA-specific programming models for high-performance computing?, accessed August 4, 2025, [https://massedcompute.com/faq-answers/?question=What%20are%20the%20key%20differences%20between%20CUDA%20and%20FPGA-specific%20programming%20models%20for%20high-performance%20computing?](https://massedcompute.com/faq-answers/?question=What+are+the+key+differences+between+CUDA+and+FPGA-specific+programming+models+for+high-performance+computing?)
11. CUDA accelerated robot localization and mapping - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/261019482_CUDA_accelerated_robot_localization_and_mapping
12. CUDA or FPGA for special purpose 3D graphics computations? [closed] - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/317731/cuda-or-fpga-for-special-purpose-3d-graphics-computations
13. Real-Time LiDAR Point Cloud Semantic Segmentation for Autonomous Driving - MDPI, accessed August 4, 2025, https://www.mdpi.com/2079-9292/11/1/11
14. Point Cloud Processing with Open3D and Python - Sigmoidal, accessed August 4, 2025, https://sigmoidal.ai/en/point-cloud-processing-with-open3d-and-python/
15. Point Cloud Processing with NVIDIA DriveWorks SDK | NVIDIA ..., accessed August 4, 2025, https://developer.nvidia.com/blog/point-cloud-processing-nvidia-driveworks-sdk/
16. Accelerating Lidar for Robotics with NVIDIA CUDA-based PCL, accessed August 4, 2025, https://developer.nvidia.com/blog/accelerating-lidar-for-robotics-with-cuda-based-pcl/
17. 10x acceleration for point cloud processing with CUDA PCL on Jetson - ROS Discourse, accessed August 4, 2025, https://discourse.openrobotics.org/t/10x-acceleration-for-point-cloud-processing-with-cuda-pcl-on-jetson/18800
18. Google Summer of Code 2020 - Refactoring, Modernisation & Feature Addition with Emphasis on GPU Module | Point Cloud Library, accessed August 4, 2025, https://pointclouds.org/gsoc-2020/gpu/
19. pcl::gpu::Octree Class Reference - Point Cloud Library (PCL), accessed August 4, 2025, https://pointclouds.org/documentation/classpcl_1_1gpu_1_1_octree.html
20. Configuring your PC to use your Nvidia GPU with PCL - Point Cloud Library 0.0 documentation, accessed August 4, 2025, https://pcl.readthedocs.io/projects/tutorials/en/pcl-1.11.0/gpu_install.html
21. Configuring your PC to use your Nvidia GPU with PCL - Point Cloud Library, accessed August 4, 2025, https://pointclouds.org/documentation/tutorials/gpu_install.html
22. Point Cloud - Open3D latest (664eff5) documentation, accessed August 4, 2025, https://www.open3d.org/docs/latest/tutorial/Basic/pointcloud.html
23. Point cloud - Open3D primary (unknown) documentation, accessed August 4, 2025, https://www.open3d.org/docs/latest/tutorial/geometry/pointcloud.html
24. PointCloud - Open3D primary (unknown) documentation, accessed August 4, 2025, https://www.open3d.org/docs/latest/tutorial/t_geometry/pointcloud.html
25. Added functionality for point cloud with pytorch tensors and PCA analysis, all working on CUDA / isl-org Open3D / Discussion #6965 - GitHub, accessed August 4, 2025, https://github.com/isl-org/Open3D/discussions/6965
26. On point clouds Semantic Segmentation - Open3D, accessed August 4, 2025, https://www.open3d.org/2019/01/16/on-point-clouds-semantic-segmentation/
27. The Future of Drone Mapping with SLAM Technology, accessed August 4, 2025, https://www.thedroneu.com/blog/slam-technology/
28. Visual SLAM for Unmanned Aerial Vehicles: Localization and Perception - PMC, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11126069/
29. [스마트 건설 백과사전 VoI.06] 로봇이 스스로 움직일 수 있는 이유, SLAM, accessed August 4, 2025, [https://www.hdec.kr/kr/newsroom/news_view.aspx?NewsSeq=781&NewsType=BRAND&NewsListType=](https://www.hdec.kr/kr/newsroom/news_view.aspx?NewsSeq=781&NewsType=BRAND&NewsListType)
30. cuVSLAM: CUDA accelerated visual odometry and mapping - arXiv, accessed August 4, 2025, https://arxiv.org/html/2506.04359v2
31. cuVSLAM: CUDA accelerated visual odometry and mapping - arXiv, accessed August 4, 2025, https://arxiv.org/html/2506.04359v1
32. Accelerated Reconstruction of Scenes Using CUDA-Based Parallel Computing, accessed August 4, 2025, https://www.researchgate.net/publication/387454784_Accelerated_Reconstruction_of_Scenes_Using_CUDA-Based_Parallel_Computing
33. SuperVINS: A visual-inertial SLAM framework integrated deep learning features - arXiv, accessed August 4, 2025, https://arxiv.org/html/2407.21348v1
34. Monocular Visual-Inertial SLAM, accessed August 4, 2025, [https://slides.games-cn.org/pdf/GAMES201840%E6%B2%88%E5%8A%AD%E5%8A%BC.pdf](https://slides.games-cn.org/pdf/GAMES201840沈劭劼.pdf)
35. Improving Monocular Visual-Inertial Initialization with Structureless Visual-Inertial Bundle Adjustment - arXiv, accessed August 4, 2025, https://arxiv.org/html/2502.16598v1
36. LB-LIOSAM: an improved mapping and localization method with loop detection | Emerald Insight, accessed August 4, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-07-2024-0314/full/html
37. Development of a GPU-Accelerated NDT Localization Algorithm for GNSS-Denied Urban Areas - PMC - PubMed Central, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8914899/
38. LIO-GC: LiDAR Inertial Odometry with Adaptive Ground Constraints - MDPI, accessed August 4, 2025, https://www.mdpi.com/2072-4292/17/14/2376
39. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 4, 2025, https://arxiv.org/html/2311.00626v2
40. For mapping and navigation, what approach should I take - Isaac ROS, accessed August 4, 2025, https://forums.developer.nvidia.com/t/for-mapping-and-navigation-what-approach-should-i-take/282915
41. Enhanced Semantic Segmentation for Large-Scale and Imbalanced Point Clouds - arXiv, accessed August 4, 2025, https://arxiv.org/html/2409.13983v1
42. [CVPR2020/PaperSummary]RandLA-Net: Efficient Semantic Segmentation of Large-Scale Point Clouds | by abhigoku10, accessed August 4, 2025, https://abhigoku10.medium.com/cvpr2020-papersummary-randla-net-efficient-semantic-segmentation-of-large-scale-point-clouds-a9529c993a46
43. Effective Training and Inference Strategies for Point Classification in LiDAR Scenes - MDPI, accessed August 4, 2025, https://www.mdpi.com/2072-4292/16/12/2153
44. PointNet++: Deep hierarchical feature learning on point sets in a metric space - arXiv, accessed August 4, 2025, http://arxiv.org/pdf/1706.02413
45. PointNet++: Deep Hierarchical Feature Learning on Point Sets in a Metric Space - ar5iv, accessed August 4, 2025, https://ar5iv.labs.arxiv.org/html/1706.02413
46. (PDF) Building Extraction from LiDAR Point Clouds Based on Revised RandLA-Net, accessed August 4, 2025, https://www.researchgate.net/publication/380641981_Building_Extraction_from_LiDAR_Point_Clouds_Based_on_Revised_RandLA-Net
47. Semantic Segmentation Method for Road Intersection Point Clouds Based on Lightweight LiDAR - MDPI, accessed August 4, 2025, https://www.mdpi.com/2076-3417/14/11/4816
48. 3D semantic segmentation on Jetson - NVIDIA Developer Forums, accessed August 4, 2025, https://forums.developer.nvidia.com/t/3d-semantic-segmentation-on-jetson/260162
49. Semantic Segmentation with Open3D-ML, PyTorch Backend, and a Custom Dataset | by Carlos Argueta | Medium, accessed August 4, 2025, https://soulhackerslabs.com/semantic-segmentation-with-open3d-ml-pytorch-backend-and-a-custom-dataset-b82e324ebe95
50. 4D Spatio-Temporal ConvNets: Minkowski Convolutional Neural Networks - CVF Open Access, accessed August 4, 2025, https://openaccess.thecvf.com/content_CVPR_2019/papers/Choy_4D_Spatio-Temporal_ConvNets_Minkowski_Convolutional_Neural_Networks_CVPR_2019_paper.pdf
51. SparseTensor and TensorField - MinkowskiEngine 0.5.3 documentation, accessed August 4, 2025, https://nvidia.github.io/MinkowskiEngine/sparse_tensor.html
52. NVIDIA Research: Tensors Are the Future of Deep Learning, accessed August 4, 2025, https://developer.nvidia.com/blog/nvidia-research-tensors-are-the-future-of-deep-learning/
53. Minkowski Engine - MinkowskiEngine 0.5.3 documentation, accessed August 4, 2025, https://nvidia.github.io/MinkowskiEngine/overview.html
54. Towards 3D scene reconstruction using Wi-Fi, accessed August 4, 2025, [https://www.spiedigitallibrary.org/proceedings/Download?urlId=10.1117%2F12.3012357&downloadType=proceedings%20article&isResultClick=True](https://www.spiedigitallibrary.org/proceedings/Download?urlId=10.1117/12.3012357&downloadType=proceedings+article&isResultClick=True)
55. Real-Time Semantic Segmentation of Point Clouds Based on an Attention Mechanism and a Sparse Tensor - MDPI, accessed August 4, 2025, https://www.mdpi.com/2076-3417/13/5/3256
56. Improved semantic segmentation network using normal vector guidance for LiDAR point clouds | Journal of Computational Design and Engineering | Oxford Academic, accessed August 4, 2025, https://academic.oup.com/jcde/article/10/6/2332/7419878
57. M.Sc. Dan Jia - Computer Vision - RWTH Aachen, accessed August 4, 2025, https://www.vision.rwth-aachen.de/person/229/
58. Daily Papers - Hugging Face, accessed August 4, 2025, [https://huggingface.co/papers?q=LiDAR%20points](https://huggingface.co/papers?q=LiDAR+points)
59. Design and Flight Demonstration of a Quadrotor for Urban Mapping and Target Tracking Research - arXiv, accessed August 4, 2025, https://arxiv.org/html/2402.13195v1
60. Edge Computing-Driven Real-Time Drone Detection Using YOLOv9 and NVIDIA Jetson Nano - MDPI, accessed August 4, 2025, https://www.mdpi.com/2504-446X/8/11/680
61. Solving Entry-Level Edge AI Challenges with NVIDIA Jetson Orin Nano, accessed August 4, 2025, https://developer.nvidia.com/blog/solving-entry-level-edge-ai-challenges-with-nvidia-jetson-orin-nano/
62. FPGA vs. GPU vs. CPU – hardware options for AI applications | Avnet Silica, accessed August 4, 2025, https://my.avnet.com/silica/resources/article/fpga-vs-gpu-vs-cpu-hardware-options-for-ai-applications/
63. FPGA vs. GPU for Deep Learning Applications | IBM, accessed August 4, 2025, https://www.ibm.com/think/topics/fpga-vs-gpu
64. OpenCL for GPU vs. FPGA - cuda - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/12332609/opencl-for-gpu-vs-fpga
65. Accurate Global Point Cloud Registration Using GPU-Based Parallel Angular Radon Spectrum - MDPI, accessed August 4, 2025, https://www.mdpi.com/1424-8220/23/20/8628
66. GPU Accelerated Parallel Occupancy Voxel Based ICP for Position Tracking, accessed August 4, 2025, https://www.araa.asn.au/acra/acra2013/papers/pap140s1-file1.pdf
67. Comprehensive empirical evaluation of feature extractors in computer vision - PMC - PubMed Central, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11623105/
68. VPI - Vision Programming Interface: ORB feature detector - NVIDIA Docs Hub, accessed August 4, 2025, https://docs.nvidia.com/vpi/algo_orb_feature_detector.html
69. fixstars/cuda-efficient-features: A CUDA implementation of keypoint detection and descriptor extraction - GitHub, accessed August 4, 2025, https://github.com/fixstars/cuda-efficient-features
70. miniSAM: A Flexible Factor Graph Non-linear Least Squares Optimization Framework, accessed August 4, 2025, https://project.inria.fr/ppniv19/files/2019/11/PPNIV19-paper_Dong.pdf
71. An Indoor Wayfinding System based on Geometric Features Aided Graph SLAM for the Visually Impaired - PubMed Central, accessed August 4, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5659309/
72. Pose Graph Optimization Tutorial - PyPose, accessed August 4, 2025, https://pypose.org/tutorials/pgo/pgo_tutorial.html
73. Theseus: A Library for Differentiable Nonlinear Optimization - arXiv, accessed August 4, 2025, https://arxiv.org/pdf/2207.09442
74. Accelerating Reality Capture Workflows with AI and NVIDIA RTX GPUs, accessed August 4, 2025, https://developer.nvidia.com/blog/accelerating-reality-capture-workflows-with-ai-and-nvidia-rtx-gpus/


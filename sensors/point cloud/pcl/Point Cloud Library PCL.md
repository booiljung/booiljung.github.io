# Point Cloud Library (PCL)에 대한 고찰



Point Cloud Library(PCL)는 2D/3D 이미지 및 포인트 클라우드 처리를 위한 대규모의 독립형 오픈소스 C++ 라이브러리 프로젝트다.1 이 라이브러리는 3차원 컴퓨터 비전 분야에서 발생하는 다양한 문제를 해결하기 위해 설계되었으며, 필터링(filtering), 특징 추출(feature estimation), 표면 재구성(surface reconstruction), 정합(registration), 모델 피팅(model fitting), 분할(segmentation) 등 수많은 최첨단 알고리즘을 포괄적으로 제공한다.1

PCL의 가장 중요한 특징 중 하나는 라이선스 정책이다. PCL은 BSD 라이선스 하에 배포되므로, 상업적 및 연구용 애플리케이션에서 아무런 제약 없이 자유롭게 사용할 수 있다.2 이러한 개방적인 라이선스 정책은 PCL이 학계와 산업계 양쪽에서 광범위하게 채택되고, 거대한 사용자 및 개발자 커뮤니티를 형성하는 데 결정적인 역할을 했다.


PCL의 개발은 2010년 3월, 로봇 공학 연구의 선구자였던 Willow Garage에서 시작되었다.4 이 배경은 PCL의 설계 철학과 기능 구성에 깊은 영향을 미쳤다. PCL은 처음부터 로봇 운영체제(Robot Operating System, ROS)와 함께 성장했으며, 로봇이 주변 환경을 3D로 인식하고 상호작용하는 데 필요한 핵심 기술을 제공하는 것을 목표로 했다.5

이러한 기원으로 인해 PCL은 ROS와 완벽하게 통합되어 있다. `pcl_ros` 패키지를 통해 ROS의 메시지 타입(`sensor_msgs/PointCloud2`)과 PCL의 네이티브 데이터 구조(`pcl::PointCloud<T>`) 간의 변환이 용이하며, PCL 알고리즘을 ROS 노드(node) 또는 노드렛(nodelet) 형태로 쉽게 실행할 수 있다.4 이는 로봇 인식 파이프라인을 구축하는 데 있어 PCL을 사실상의 표준으로 만들었다.

또한, PCL은 뛰어난 이식성을 자랑한다. Linux, Windows, macOS, Android 등 대부분의 주요 운영체제에서 원활하게 동작하는 크로스플랫폼을 지원한다.4 이는 PCL이 특정 플랫폼에 종속되지 않고, 데스크톱 연구 환경부터 모바일 기기, 임베디드 시스템에 이르기까지 다양한 환경에서 활용될 수 있는 기반을 제공한다.


PCL의 아키텍처는 두 가지 핵심 철학, 즉 모듈성과 병렬 처리에 기반을 두고 있다.

첫째, PCL은 거대한 단일 라이브러리가 아니라, 기능별로 잘게 나뉜 여러 개의 작은 모듈식 라이브러리의 집합으로 구성되어 있다.2 예를 들어, 필터링 알고리즘은 `libpcl_filters`에, 특징 추출 알고리즘은 `libpcl_features`에, 시각화 관련 기능은 `libpcl_visualization`에 포함되는 식이다.4 이러한 모듈식 구조는 개발자에게 높은 유연성을 제공한다. 사용자는 자신의 프로젝트에 필요한 특정 모듈만 선택적으로 빌드하고 링크함으로써, 의존성을 최소화하고 최종 실행 파일의 크기를 최적화할 수 있다.

둘째, PCL은 대용량 3D 데이터의 고속 처리를 위해 설계 초기부터 병렬 처리를 염두에 두었다. 멀티코어 CPU의 성능을 최대한 활용하기 위해 OpenMP와 Intel Threading Building Blocks(TBB)를 기본적으로 지원한다.4

`FPFHEstimationOMP` 11나 `GreedyProjectionTriangulation`의 OpenMP 버전 13처럼, 계산 집약적인 여러 알고리즘에 대해 병렬화된 구현이 제공된다. 사용자는 클래스 이름만 변경하는 것만으로 간단하게 싱글 스레드 버전에서 멀티 스레드 버전으로 전환하여 성능을 극대화할 수 있다. 더 나아가, Intel oneAPI Base Toolkit을 활용한 PCL 최적화 버전은 SYCL 표준을 통해 CPU뿐만 아니라 Intel 내장 및 외장 GPU를 활용한 이기종 컴퓨팅까지 지원하여, 현대 컴퓨팅 하드웨어의 발전에 발맞춰 진화하고 있다.14


PCL의 강력하고 다양한 기능은 여러 고품질 서드파티 라이브러리 위에 구축되어 있다. 이러한 의존성을 이해하는 것은 PCL을 효과적으로 사용하고 빌드하는 데 필수적이다.


- **Boost:** PCL의 가장 근간이 되는 라이브러리 중 하나다. PCL 개발 초기(C++11 표준 이전)에 표준 라이브러리가 제공하지 못했던 스마트 포인터(`shared_ptr`), 스레딩, 파일 시스템, MPL(Metaprogramming Library) 등 핵심적인 기능을 제공하기 위해 광범위하게 사용되었다.3 PCL의 역사는 C++ 표준의 발전과 궤를 같이하며, 초기에 Boost에 의존했던 많은 기능들이 점차 현대 C++ 표준 라이브러리의 기능으로 대체되고 있다. 예를 들어, `boost::filesystem`을 `std::filesystem`으로 전환하려는 노력이 진행 중이지만, C++14와의 호환성을 유지해야 하는 제약으로 인해 점진적으로 이루어지고 있다.18 이러한 기술적 부채는 PCL의 성숙도를 보여주는 동시에, 현대화 과정에서 겪는 어려움을 시사한다.
- **Eigen:** PCL의 모든 수학적 연산의 심장과 같은 역할을 하는 고성능 C++ 템플릿 라이브러리다.3 행렬, 벡터, 기하학적 변환, 수치 해석 솔버 등 선형대수와 관련된 모든 계산을 담당한다. Eigen은 표현식 템플릿(expression templates)을 사용하여 컴파일 타임에 연산을 최적화하고, SSE(Streaming SIMD Extensions) 명령어셋을 활용하여 매우 높은 계산 효율을 보장한다.19 또한, 헤더 전용(header-only) 라이브러리이므로 별도의 빌드나 링크 과정 없이 헤더 파일만 포함하면 바로 사용할 수 있어 프로젝트 통합이 매우 간편하다.5
- **FLANN (Fast Library for Approximate Nearest Neighbors):** PCL의 수많은 알고리즘 성능에 결정적인 영향을 미치는 라이브러리다. 고차원 공간에서 빠른 근사 최근접 이웃(Approximate Nearest Neighbor, ANN) 탐색을 수행하는 데 사용된다.3 PCL의 Kd-tree 구현은 내부적으로 FLANN을 활용하며, 특징 매칭, 정합, 군집화 등 이웃 탐색이 필요한 거의 모든 과정에서 FLANN의 빠른 검색 속도에 의존한다.22


- **VTK (Visualization Toolkit):** PCL에 강력한 3D 시각화 기능을 부여하는 핵심 라이브러리다. `pcl_visualization` 모듈은 VTK를 백엔드로 사용하여 포인트 클라우드, 표면 메시, 기하학적 형상 등을 렌더링한다.3 VTK가 설치되어 있지 않으면 PCL의 시각화 관련 기능을 사용할 수 없다.

- **QHULL:** 볼록 헐(convex hull) 및 오목 헐(concave hull) 계산과 같은 계산 기하학 알고리즘에 사용된다.4 주로 

  `pcl_surface` 모듈에서 활용된다.

- **OpenNI (Open Natural Interaction):** Microsoft Kinect, Asus Xtion Pro와 같은 OpenNI 호환 3D 센서로부터 직접 포인트 클라우드 데이터를 실시간으로 취득(`grabbing`)하는 데 사용된다.3

- **Qt:** PCL을 사용하여 그래픽 사용자 인터페이스(GUI) 애플리케이션을 개발할 때 필요한 크로스플랫폼 프레임워크다.3



PCL에서 모든 작업의 기본이 되는 데이터 구조는 `pcl::PointCloud<PointT>` 템플릿 클래스다.6 이는 포인트 클라우드를 저장하고 관리하기 위한 핵심 컨테이너 역할을 한다. 

`PointT` 템플릿 매개변수에는 저장할 포인트의 종류(예: `pcl::PointXYZ`, `pcl::PointXYZRGB`)가 지정된다.

이 클래스는 다음과 같은 주요 멤버 변수를 포함한다 28:

- `width` (int): 포인트 클라우드의 너비를 나타낸다. 데이터셋이 이미지처럼 격자 구조를 가지는 '정규(organized)' 클라우드의 경우, 한 행에 포함된 포인트의 수를 의미한다. 격자 구조가 없는 '비정규(unorganized)' 클라우드의 경우, 전체 포인트의 수를 의미한다.
- `height` (int): 포인트 클라우드의 높이를 나타낸다. 정규 클라우드의 경우 행의 수를 의미하며, 비정규 클라우드의 경우 항상 1로 설정된다. 따라서 `height` 값이 1보다 큰지 여부는 해당 클라우드가 정규 클라우드인지 판별하는 중요한 기준이 된다.
- `points` (`std::vector<PointT>`): 실제 포인트 데이터가 저장되는 `std::vector` 컨테이너다. 각 요소는 `PointT` 타입의 객체다.
- `is_dense` (bool): `points` 벡터 내의 모든 데이터가 유한한 값(Inf/NaN 값을 포함하지 않음)인지 여부를 나타낸다. `true`이면 모든 포인트가 유효함을 의미한다.
- `sensor_origin_` (Eigen::Vector4f): 데이터를 수집한 센서의 원점(위치)을 저장하는 선택적 필드다.
- `sensor_orientation_` (Eigen::Quaternionf): 데이터를 수집한 센서의 방향(회전)을 저장하는 선택적 필드다.

PCL 프로그래밍에서는 메모리 관리를 용이하게 하기 위해 `PointCloud` 객체를 직접 다루기보다는 스마트 포인터(주로 `boost::shared_ptr` 또는 현대 C++의 `std::shared_ptr`)를 사용하는 것이 일반적인 관례다. 이를 위해 `PointCloud<PointT>::Ptr`과 같은 타입 별칭(type alias)이 널리 사용된다.6


PCL은 포인트 클라우드를 정규(organized)와 비정규(unorganized) 두 가지 유형으로 구분하며, 이 구분은 단순한 데이터의 형태를 넘어 알고리즘의 선택과 성능에 지대한 영향을 미친다.

- **정규 클라우드:** `height`가 1보다 큰 값으로, 2D 이미지와 유사한 행렬 구조를 가진다.29 이러한 데이터는 주로 스테레오 카메라나 ToF(Time-of-Flight) 카메라와 같이 고정된 격자 형태의 센서에서 생성된다. 정규 클라우드의 가장 큰 장점은 특정 포인트의 이웃을 찾을 때 복잡한 공간 탐색 알고리즘 없이 2D 배열의 인덱싱처럼 O(1) 시간 복잡도로 매우 빠르게 찾을 수 있다는 점이다. PCL의 `search` 모듈에는 이러한 정규 데이터셋을 위한 특화된 탐색 기법이 포함되어 있어, 이를 활용하는 알고리즘의 계산 비용을 극적으로 낮출 수 있다.23
- **비정규 클라우드:** `height`가 1인 데이터로, 포인트들 간의 이웃 관계가 데이터 구조에 내재되어 있지 않다. 회전형 LiDAR 스캔이나 여러 개의 스캔을 정합하여 얻은 데이터가 대표적인 예다. 이 경우, 이웃을 찾기 위해서는 반드시 Kd-tree나 Octree와 같은 공간 분할 자료구조를 사용한 탐색이 필요하며, 이는 O(log N) 또는 그 이상의 계산 비용을 요구한다.

따라서 개발자는 센서 데이터가 정규 구조를 가지고 있다면, 처리 파이프라인 전반에 걸쳐 가능한 한 이 구조를 유지하도록 노력해야 한다. 많은 필터링이나 변환 작업이 이 구조를 깨뜨릴 수 있으므로, 알고리즘 선택에 신중을 기하는 것이 전체 시스템의 성능을 좌우하는 핵심 요소가 된다.


`PointT`는 `PointCloud`에 저장될 개별 포인트의 데이터 구조를 정의한다. PCL은 다양한 시나리오에 맞춰 미리 정의된 여러 `PointT` 타입을 제공하며, 사용자는 데이터의 특성에 따라 가장 적합한 타입을 선택해야 한다.33 필요하다면 사용자가 직접 새로운 `PointT` 타입을 정의하여 확장하는 것도 가능하다.34

다음 표는 가장 일반적으로 사용되는 `PointT` 타입들을 정리한 것이다.

| Point Type (`pcl::...`) | 구성 필드 (Component Fields)                    | 주요 사용처 (Primary Use Case)                               |
| ----------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `PointXYZ`              | `float x, y, z`                                 | 가장 기본적인 3차원 좌표 정보만 필요할 때 사용.34            |
| `PointXYZI`             | `float x, y, z, intensity`                      | LiDAR 스캔 데이터와 같이 각 점의 강도(intensity) 정보가 포함될 때 사용.6 |
| `PointXYZRGB`           | `float x, y, z, rgb`                            | RGB-D 카메라 데이터와 같이 각 점의 색상(RGB) 정보가 포함될 때 사용.6 |
| `PointXYZRGBA`          | `float x, y, z, uint32_t rgba`                  | 색상 정보에 투명도(alpha) 채널까지 포함될 때 사용.34         |
| `Normal`                | `float normal_x, normal_y, normal_z, curvature` | 점의 위치 정보 없이 표면 법선 벡터와 곡률 정보만 저장할 때 사용.34 |
| `PointNormal`           | `float x, y, z`, `float normal, curvature`      | 3차원 좌표와 해당 지점의 법선 및 곡률 정보를 함께 저장할 때 사용.34 |
| `FPFHSignature33`       | `float histogram`                               | FPFH 특징 디스크립터를 저장하기 위한 타입.12                 |
| `VFHSignature308`       | `float histogram`                               | VFH 특징 디스크립터를 저장하기 위한 타입.36                  |



필터링은 포인트 클라우드 처리 파이프라인의 가장 첫 단계에서 수행되는 핵심적인 전처리 과정이다. 센서로부터 취득된 원본 데이터에는 노이즈, 이상점(outlier), 불필요한 영역 등이 포함되어 있을 수 있으며, 이를 제거하거나 데이터를 간소화하여 후속 처리의 정확성과 효율성을 높이는 것이 필터링의 주된 목적이다.4

- **`PassThrough` 필터:** 가장 간단하면서도 효과적인 필터 중 하나로, 특정 좌표 축(x, y, z 등)을 기준으로 사용자가 지정한 값 범위 내에 있는 점들만 통과시키거나, 혹은 그 범위 밖에 있는 점들만 통과시키는 역할을 한다.39 예를 들어, 로봇의 작업 공간에서 바닥이나 천장에 해당하는 영역을 제거하거나, 특정 거리 내에 있는 객체에만 집중하고자 할 때 유용하게 사용된다.38 `setFilterFieldName()`으로 축을 지정하고 `setFilterLimits()`로 범위를 설정한다.41
- **`VoxelGrid` 다운샘플링:** 대용량 포인트 클라우드의 데이터 수를 줄여 처리 속도를 향상시키기 위해 널리 사용되는 기법이다.38 3차원 공간을 작은 정육면체 모양의 복셀(voxel)로 나누고, 각 복셀 내에 포함된 모든 포인트들을 그들의 무게중심(centroid)점 하나로 근사한다.43 이 방식은 포인트 수를 효과적으로 줄이면서도 전체적인 표면의 형태를 보존하는 장점이 있다. 다운샘플링의 정도는 `setLeafSize()` 메서드를 통해 설정하는 복셀의 크기에 의해 결정된다.43
- **`StatisticalOutlierRemoval` 필터:** 측정 오류 등으로 인해 발생하는 희소한 이상점(sparse outlier)을 제거하는 데 효과적인 통계 기반 필터다.4 이 알고리즘은 각 포인트에 대해 지정된 수(`k`)의 최근접 이웃까지의 평균 거리를 계산한다. 그 후, 전체 포인트들의 평균 거리 분포가 가우시안 분포를 따른다고 가정하고, 전체 평균으로부터 특정 표준편차 배수 이상 벗어나는 평균 거리를 가진 포인트를 이상점으로 간주하여 제거한다.44 `setMeanK()`로 이웃의 수 `k`를, `setStddevMulThresh()`로 표준편차의 배수를 설정하여 필터링의 강도를 조절할 수 있다.46
- **기타 주요 필터:**
  - `RadiusOutlierRemoval`: 지정된 반경 내에 특정 개수 미만의 이웃 포인트를 가진 점들을 이상점으로 간주하여 제거한다.4
  - `ConditionalRemoval`: 사용자가 직접 정의한 복잡한 조건을 만족하는 포인트들을 제거하거나 남길 수 있는 유연한 필터다.48
  - `ProjectInliers`: 평면이나 구와 같은 기하학적 모델을 정의하고, 클라우드의 포인트들을 해당 모델 위로 투영시킨다.49


PCL의 거의 모든 고수준 알고리즘은 내부적으로 특정 포인트의 이웃을 찾는 '탐색' 작업에 의존한다. `pcl_search` 모듈과 이를 기반으로 하는 `pcl_kdtree`, `pcl_octree` 모듈은 이러한 탐색 작업을 효율적으로 수행하기 위한 핵심 자료구조와 인터페이스를 제공한다. 이 탐색 기능의 성능이 전체 PCL 파이프라인의 성능을 좌우한다고 해도 과언이 아니다. 법선 추정, 특징 기술, 군집화, 정합 등 거의 모든 과정이 이웃 포인트들의 정보를 필요로 하기 때문이다.32 PCL은 이러한 의존성을 `pcl::search::Search`라는 추상 базовый 클래스(base class)를 통해 관리하며, 사용자는 실제 구현체(Kd-tree 또는 Octree)를 선택하여 주입하는 '플러그인' 방식으로 탐색 방법을 교체할 수 있다.

- **`KdTree` (`pcl::search::KdTree`):** k-차원 트리(k-dimensional tree)는 공간을 재귀적으로 분할하여 포인트를 저장하는 자료구조로, 특히 3차원과 같은 저차원 공간에서의 최근접 이웃(Nearest Neighbor, NN) 탐색에 매우 효율적이다.53 PCL의 Kd-tree 구현은 내부적으로 FLANN 라이브러리를 사용하여 최적의 성능을 제공한다.53
  - `nearestKSearch(point, k, indices, distances)`: 주어진 `point`로부터 가장 가까운 `k`개의 이웃을 찾아 그 인덱스와 거리(제곱)를 반환한다.55
  - `radiusSearch(point, radius, indices, distances)`: 주어진 `point`로부터 특정 `radius` 반경 내에 있는 모든 이웃을 찾아 반환한다.55
- **`Octree` (`pcl::octree::OctreePointCloudSearch`):** 옥트리는 3차원 공간을 정육면체로 보고, 이를 재귀적으로 8개의 작은 정육면체(자식 노드)로 분할해 나가는 계층적 트리 구조다.56 이러한 구조는 다양한 해상도(resolution)에서 공간을 분석하는 데 유용하다.
  - 옥트리는 NN 탐색뿐만 아니라, 공간 점유 여부 확인, 밀도 기반 다운샘플링, 공간 변화 감지 등 다양한 응용에 활용될 수 있다.32
  - Kd-tree와 마찬가지로 `K-NN search`, `radius search`와 같은 표준적인 탐색 인터페이스를 제공하며, 추가적으로 `voxelSearch`와 같이 특정 복셀 내의 모든 포인트를 찾는 기능도 제공한다.59


특징점 기술(Feature Description)은 포인트 클라우드의 각 점이나 특정 영역이 가진 고유한 기하학적 특성을 정량적인 수치 벡터, 즉 '디스크립터(descriptor)'로 표현하는 과정이다. 잘 설계된 디스크립터는 시점이나 조명 변화, 노이즈 등에 강인해야 하며, 이를 통해 서로 다른 포인트 클라우드 간의 대응점을 찾거나(정합), 특정 객체를 식별하는(인식) 등의 고수준 작업을 수행할 수 있다.4

- **법선 벡터 및 곡률 추정 (`NormalEstimation`):** 가장 기본적이면서도 중요한 지역적(local) 특징이다. 한 점의 법선 벡터는 그 점 주변의 국소적인 표면 방향을 나타낸다. PCL에서는 주성분 분석(Principal Component Analysis, PCA)을 통해 법선을 추정하는 것이 일반적이다. 즉, 특정 점의 k-최근접 이웃들을 하나의 집합으로 보고, 이 집합의 공분산 행렬을 계산한다. 이 행렬의 고유값 분해(eigendecomposition)를 수행했을 때, 가장 작은 고유값에 해당하는 고유벡터가 바로 해당 표면의 법선 벡터가 된다.32 또한, 고유값들의 상대적인 크기 관계를 통해 표면의 곡률(curvature)도 함께 추정할 수 있다.32 추정된 법선의 방향은 모호할 수 있는데, `setViewPoint()` 함수를 사용하여 시점을 기준으로 방향을 일관되게 정렬할 수 있다.61

- **FPFH (Fast Point Feature Histogram):** PFH(Point Feature Histogram)의 계산 복잡도를 개선하여 실시간 처리가 가능하도록 만든 강력한 지역 디스크립터다.12 PFH는 한 점과 그 이웃 내의 모든 점 쌍 간의 기하학적 관계(각도, 거리 등)를 히스토그램으로 표현하여 계산 복잡도가 $O(nk^2)$ 에 달하는 반면, FPFH는 두 단계 접근법을 통해 이를 $O(nk)$ 로 줄였다.12

  1. 먼저, 각 점 `p`에 대해 자신의 이웃들과의 관계만을 계산하여 단순화된 히스토그램인 SPFH(Simplified PFH)를 생성한다.
  2. 그 다음, 다시 `p`의 이웃들을 찾아, 각 이웃의 SPFH를 `p`와의 거리에 따라 가중치를 주어 합산함으로써 최종 FPFH를 완성한다.12

  이 과정을 통해 계산량을 줄이면서도 2차 이웃의 정보까지 간접적으로 반영하여 풍부한 표현력을 유지한다. PCL에서는 FPFHSignature33이라는 33차원 float 벡터로 표현된다.12

- **VFH (Viewpoint Feature Histogram):** FPFH와 같은 지역 디스크립터와 달리, 하나의 객체 클러스터 전체에 대해 단일 디스크립터를 계산하는 전역(global) 또는 준-전역(meta-local) 디스크립터다.65 이는 개별 점이 아닌, 객체 전체를 인식하고 6-DoF(자유도) 자세를 추정하는 데 특화되어 있다. VFH는 두 가지 주요 구성 요소로 이루어진다 37:

  1. **시점 방향 성분:** 클러스터의 중심점과 시점(카메라 위치)을 연결하는 벡터, 그리고 클러스터 내 각 점의 법선 벡터 간의 각도 분포를 히스토그램으로 나타낸다.
  2. 확장된 FPFH 성분: 클러스터의 중심점에서 계산된 FPFH와, 클러스터 내 모든 점과 중심점 간의 거리 및 법선 관계를 추가로 인코딩한다.

  이 두 정보를 결합하여 시점에 따라 객체가 어떻게 보이는지에 대한 정보를 포함하는 강력한 디스크립터를 생성한다. PCL에서는 VFHSignature308이라는 308차원 float 벡터로 표현된다.37


키포인트(또는 관심점, interest points) 검출은 전체 포인트 클라우드에서 후속 처리(예: 특징 매칭, 정합)의 기준점이 될 소수의 중요하고 안정적인 점들을 추출하는 과정이다.32 전체 포인트에 대해 특징 디스크립터를 계산하는 대신, 키포인트에 대해서만 계산함으로써 전체 처리 속도를 크게 향상시킬 수 있다.

- **NARF (Normal Aligned Radial Feature):** 거리 이미지(range image)를 입력으로 받아 키포인트를 추출하는 알고리즘이다.68 거리 이미지는 3D 포인트를 2D 이미지 형태로 표현한 것으로, 이웃 관계를 빠르게 파악할 수 있다. NARF는 객체의 경계(border)가 명확하고 주변 표면이 안정적인 지점을 키포인트로 선호한다. 즉, 표면의 변화가 급격한 곳과 안정적인 곳의 중간 지점을 찾아내어, 반복적인 관측에도 안정적으로 검출될 수 있는 위치를 선택한다.69
- **SIFT (Scale-Invariant Feature Transform) Keypoint:** 2D 이미지 처리 분야에서 매우 유명한 SIFT 알고리즘을 3D 포인트 클라우드에 적용한 것이다.70 이 알고리즘은 각 포인트의 강도(intensity) 값을 필요로 하며, 스케일 공간(scale-space)에서 가우시안 차분(Difference of Gaussians, DoG)을 계산하여 스케일 변화에 강인한 키포인트를 찾는다. `setScales()`를 통해 탐색할 스케일 범위를, `setMinimumContrast()`를 통해 키포인트의 최소 대비를 설정할 수 있다.70
- **HarrisKeypoint3D:** 2D 이미지의 Harris 코너 검출 기법을 3D 공간으로 확장한 것이다. 2D에서 이미지 그래디언트를 사용하는 대신, 3D에서는 각 점의 표면 법선 벡터의 변화량을 분석하여 코너, 즉 여러 방향으로 표면이 꺾이는 지점을 검출한다.67
- **기타 키포인트 검출기:** PCL은 이 외에도 `ISSKeypoint3D`(Intrinsic Shape Signatures), `SUSANKeypoint`, 2D 이미지 기반의 `AGASTKeypoint2D` 등 다양한 키포인트 검출 알고리즘을 제공한다.67


정합은 서로 다른 좌표계에서 측정된 두 개 이상의 포인트 클라우드를 하나의 공통 좌표계로 정렬하는 과정이다. 이는 3D 모델링, 지도 작성(mapping), 로봇 위치 추정(localization) 등 다양한 분야의 핵심 기술이다.32

- **ICP (Iterative Closest Point):** 가장 대표적이고 널리 사용되는 정합 알고리즘이다.73 이름에서 알 수 있듯이, 두 포인트 클라우드(소스와 타겟)를 반복적으로 가장 가까운 점들을 기준으로 맞춰나가는 방식으로 동작한다. ICP의 기본 단계는 다음과 같다 74:

  1. **대응점 탐색 (Correspondence Search):** 소스 클라우드의 각 점에 대해 타겟 클라우드에서 가장 가까운 점을 찾는다. 이 과정은 보통 Kd-tree를 사용하여 효율적으로 수행된다.

  2. **변환 추정 (Transform Estimation):** 찾아낸 대응점 쌍들 간의 거리 오차(주로 유클리드 거리의 제곱합)를 최소화하는 최적의 강체 변환(회전 행렬 `R`과 이동 벡터 `t`)을 계산한다. 이 최적화 문제는 보통 특이값 분해(SVD)를 통해 해석적으로 풀 수 있다.

  3. **변환 적용 (Transform Application):** 계산된 변환을 소스 클라우드에 적용하여 위치를 갱신한다.

  4. 반복: 위 1~3단계를 오차가 특정 임계값 이하로 줄어들거나, 최대 반복 횟수에 도달할 때까지 반복한다.

     ICP가 최소화하려는 목적 함수는 다음과 같이 표현된다 76:

  $$
  E(R, \mathbf{t}) = \sum_{i=1}^{N} \| (\mathbf{R}\mathbf{p}_i + \mathbf{t}) - \mathbf{q}_i \|^2
  $$

  여기서 $`\mathbf{p}_i`$는 소스 클라우드의 점, $`\mathbf{q}_i`$는 그에 대응하는 타겟 클라우드의 가장 가까운 점이다. PCL의 `pcl::IterativeClosestPoint` 클래스는 `setMaximumIterations` (최대 반복 횟수), `setTransformationEpsilon` (연속된 변환 간의 차이 임계값), `setMaxCorrespondenceDistance` (대응점으로 간주할 최대 거리) 등 다양한 파라미터를 제공하여 정합 과정을 세밀하게 제어할 수 있다.51

- **NDT (Normal Distributions Transform):** ICP와는 다른 접근 방식을 사용하는 정합 알고리즘이다. NDT는 개별 포인트들을 직접 비교하는 대신, 타겟 포인트 클라우드가 차지하는 공간을 일정한 크기의 복셀(voxel)로 나누고, 각 복셀에 포함된 포인트들의 분포를 다변량 정규분포(가우시안)로 모델링한다.78 즉, 타겟 클라우드를 연속적인 확률 밀도 함수(PDF)로 표현하는 것이다. 정합 과정은 소스 클라우드의 각 점을 특정 변환으로 옮겼을 때, 이 점들이 타겟 클라우드의 PDF 상에서 나타날 확률(likelihood)의 총합을 최대화하는 변환을 찾는 최적화 문제로 정의된다.78 NDT는 미분 가능한 목적 함수를 가지므로 뉴턴 방법과 같은 경사도 기반 최적화 기법을 사용할 수 있다. 이 방법은 포인트 수가 매우 많은 대규모 클라우드에 대해 ICP보다 빠르고, 특정 상황에서는 더 강건할 수 있다. 하지만 초기 추정치가 좋지 않으면 지역 최적해(local minimum)에 빠지기 쉬운 단점이 있다.78 PCL의 

  `pcl::NormalDistributionsTransform` 클래스는 `setResolution` (복셀 크기) 파라미터가 성능에 가장 큰 영향을 미친다.80


분할은 전체 포인트 클라우드를 특정 기준에 따라 의미 있는 여러 부분 집합(클러스터 또는 세그먼트)으로 나누는 과정이다. 예를 들어, 실내 환경 스캔에서 바닥, 벽, 테이블, 의자 등을 각각의 객체로 분리하는 작업이 이에 해당한다.6

- **RANSAC 기반 모델 분할 (`pcl::SACSegmentation`):** RANSAC(Random Sample Consensus)은 이상점(outlier)이 많이 포함된 데이터에서 특정 수학적 모델(예: 평면, 원통, 구)의 파라미터를 강건하게(robustly) 추정하는 반복적인 방법이다. PCL의 `SACSegmentation` 클래스는 이를 이용해 포인트 클라우드에서 원하는 기하학적 모델에 해당하는 포인트 집합(inliers)을 찾아낸다.82
  - `setModelType()`: `SACMODEL_PLANE`, `SACMODEL_CYLINDER` 등 찾고자 하는 모델의 종류를 지정한다.83
  - `setMethodType()`: `SAC_RANSAC`과 같은 강건한 추정 방법을 선택한다.83
  - `setDistanceThreshold()`: 모델로부터 특정 거리 이내에 있는 포인트를 인라이어(inlier)로 간주할지를 결정하는 임계값을 설정한다. 이 값이 분할의 정밀도를 결정하는 중요한 파라미터다.83 분할이 성공하면, 찾은 모델의 파라미터(예: 평면의 방정식 계수)와 해당 모델에 속하는 인라이어 포인트들의 인덱스 리스트를 반환한다.82
- **유클리드 군집화 (`pcl::EuclideanClusterExtraction`):** 기하학적 모델을 가정하지 않고, 단순히 유클리드 공간상에서 가까이 모여 있는 포인트들을 하나의 군집(cluster)으로 묶는 방법이다.52 이는 테이블 위의 여러 물체들을 각각 분리하는 것과 같은 작업에 효과적이다.
  - 알고리즘은 Kd-tree를 사용하여 임의의 한 점에서 시작하여, 지정된 거리(`tolerance`) 내에 있는 모든 이웃을 찾고, 다시 그 이웃들의 이웃을 재귀적으로 찾아나가는 방식으로 동작한다. 마치 물감이 번져나가듯(flood-fill) 연결된 모든 포인트를 하나의 클러스터로 묶는다.52
  - `setClusterTolerance()`: 같은 클러스터로 간주될 포인트 간의 최대 거리를 설정한다. 이 값이 너무 작으면 하나의 객체가 여러 조각으로 나뉠 수 있고, 너무 크면 서로 다른 객체가 하나로 합쳐질 수 있어 신중한 설정이 필요하다.52
  - `setMinClusterSize()`와 `setMaxClusterSize()`: 결과로 나올 클러스터가 포함해야 할 최소 및 최대 포인트 수를 지정하여, 너무 작거나 큰 노이즈성 클러스터를 걸러낼 수 있다.52
- **Conditional Euclidean Clustering:** 기본적인 유클리드 군집화에 사용자 정의 조건을 추가한 확장된 버전이다. 두 포인트가 단순히 가까운 것뿐만 아니라, 예를 들어 그들의 법선 벡터 방향이 유사하거나 색상이 비슷한 경우에만 같은 클러스터로 묶도록 조건을 걸 수 있다. 이를 통해 더 정교하고 의미론적인 분할이 가능하다.85


표면 재구성은 흩어져 있는 이산적인 3D 포인트들로부터 원래 객체의 연속적인 표면을 복원하는 과정이며, 결과물은 보통 삼각형 메시(triangle mesh) 형태로 표현된다.86 PCL은 이 문제를 해결하기 위해 근본적으로 다른 두 가지 철학에 기반한 알고리즘들을 제공한다: 명시적(explicit) 방법과 암시적(implicit) 방법이다.

- 명시적 방법: GreedyProjectionTriangulation

  이 방법은 입력된 포인트들을 직접 메시의 정점(vertex)으로 사용하여 "점들을 연결하는" 방식으로 표면을 만든다. GreedyProjectionTriangulation은 각 점의 국소적인 이웃들을 2D 평면에 투영한 뒤, 탐욕 알고리즘(greedy algorithm)에 따라 가장 적절한 삼각형을 만들어 나가는 방식으로 동작한다.88 이 방식은 매우 빠르고 원본 데이터의 정밀도를 그대로 유지하는 장점이 있지만, 데이터에 노이즈가 많거나 구멍(hole)이 있는 경우 이를 처리하지 못하고 불완전한 메시를 생성하는 단점이 있다.88

- 암시적 방법: Poisson 재구성

  이 방법은 입력 포인트를 직접 사용하지 않는다. 대신, 포인트들의 위치와 법선 벡터 정보를 이용하여 3D 공간 전체에 대한 암시적인 함수(implicit function)를 정의한다. 구체적으로, 모델의 내부와 외부를 구분하는 지시 함수(indicator function)를 정의하고, 이 함수의 그래디언트가 표면 법선 벡터 필드와 같아지도록 하는 푸아송(Poisson) 방정식을 푼다.90 최종 표면은 이 암시적 함수의 등위면(isosurface)을 추출하여 얻는다. 이 과정은 전역 최적화(global optimization)에 해당하므로 노이즈에 매우 강건하고, 데이터가 없는 구멍을 자연스럽게 메워 물이 새지 않는(watertight) 부드러운 메시를 생성하는 강력한 장점이 있다.90 하지만 입력으로 반드시 정확한 법선 벡터가 필요하며, 계산 비용이 명시적 방법에 비해 훨씬 높다.

- 평활화 및 전처리: MovingLeastSquares (MLS)

  MLS는 엄밀히 말해 표면 재구성 알고리즘이라기보다는, 노이즈가 많은 데이터를 평활화(smoothing)하고 더 정확한 법선을 추정하기 위한 전처리 기법이다.93 각 점의 주변 이웃들을 이용하여 국소적인 다항식 표면을 피팅(fitting)하고, 원래 점을 이 표면 위로 투영하여 새로운 위치를 계산한다. 이를 통해 노이즈를 효과적으로 제거하고 고품질의 법선을 얻을 수 있어, 

  `Poisson` 재구성과 같은 후속 처리의 성능을 향상시키는 데 자주 사용된다.

다음 표는 PCL의 주요 표면 재구성 기법들의 특징을 비교한 것이다.

| 기법 (Technique)                | 기본 원리 (Principle)                  | 장점 (Pros)                           | 단점 (Cons)                              | 주요 파라미터 (Key Parameters)             |
| ------------------------------- | -------------------------------------- | ------------------------------------- | ---------------------------------------- | ------------------------------------------ |
| `GreedyProjectionTriangulation` | 명시적, 지역적 투영 기반 삼각화        | 빠름, 원본 데이터 정점 보존           | 노이즈/홀에 취약, 법선 필요              | `setSearchRadius`, `setMu` 95              |
| `Poisson`                       | 암시적, 푸아송 방정식 기반 전역 최적화 | 노이즈에 강함, 홀 채움, 부드러운 결과 | 법선 필수, 계산 비용 높음                | `setDepth` 96                              |
| `MovingLeastSquares`            | 평활화, 국소적 다항식 피팅             | 노이즈 제거, 고품질 법선 생성         | 표면 재구성이 아닌 전처리                | `setPolynomialOrder`, `setSearchRadius` 93 |
| `OrganizedFastMesh`             | 명시적, 정규 클라우드 이웃 관계 활용   | 매우 빠름                             | 정규(organized) 클라우드에서만 사용 가능 | 파라미터 거의 없음 86                      |


알고리즘의 결과를 확인하고 디버깅하기 위해 3D 데이터를 시각화하는 것은 필수적이다. `pcl_visualization` 라이브러리는 VTK(Visualization Toolkit)를 백엔드로 사용하여 강력하고 유연한 시각화 기능을 제공한다.24

- **`PCLVisualizer`:** 모든 기능을 갖춘 핵심 시각화 클래스다.98 이 클래스를 통해 하나 이상의 포인트 클라우드, 기하학적 형상, 텍스트 등을 3D 뷰어 창에 렌더링할 수 있다.
  - `addPointCloud()`: 포인트 클라우드를 뷰어에 추가한다. 각 클라우드에 고유한 ID 문자열을 부여하여 개별적으로 관리(업데이트, 제거 등)할 수 있다.99 색상 정보를 담은 `PointXYZRGB` 타입은 자동으로 색상이 표현되며, `PointCloudColorHandlerCustom` 등을 사용하여 임의의 색상을 지정할 수도 있다.
  - `addPointCloudNormals()`: 계산된 법선 벡터를 각 포인트에서 뻗어 나오는 작은 선분 형태로 시각화하여 법선의 방향과 일관성을 직관적으로 확인할 수 있다.101
  - `addShape()` 계열 함수: `addLine()`, `addSphere()`, `addCube()`, `addPlane()`, `addCylinder()` 등 다양한 기본 기하학적 형상을 뷰어에 직접 그릴 수 있다.98 이는 분할 알고리즘이 찾은 모델을 시각화하는 데 유용하다.
  - `setPointCloudRenderingProperties()`: 렌더링되는 포인트의 크기(`PCL_VISUALIZER_POINT_SIZE`)나 투명도(`PCL_VISUALIZER_OPACITY`)와 같은 시각적 속성을 동적으로 변경할 수 있다.104
  - **고급 기능:** 하나의 창을 여러 개의 뷰포트(viewport)로 나누어 서로 다른 시점이나 데이터를 동시에 보여주거나, 키보드 및 마우스 이벤트에 대한 콜백(callback) 함수를 등록하여 사용자와의 상호작용을 구현하는 등 다양한 고급 기능을 지원한다.98
- **`CloudViewer`:** `PCLVisualizer`의 복잡한 설정 없이, 단 몇 줄의 코드로 간단하게 포인트 클라우드를 화면에 띄우고 싶을 때 사용하는 간소화된 클래스다.97 빠른 프로토타이핑이나 간단한 결과 확인에 매우 유용하다.
- **`pcl_viewer` 실행 파일:** PCL을 설치하면 제공되는 커맨드 라인 유틸리티로, 소스 코드 작성 없이 PCD(Point Cloud Data) 파일을 직접 열어볼 수 있다. 배경색(`-bc`), 포인트 크기(`-ps`), 법선 표시(`-normals`) 등 다양한 옵션을 통해 렌더링 방식을 제어할 수 있다.97



PCL은 크로스플랫폼 빌드 시스템인 CMake를 표준으로 채택하고 있다.4 따라서 PCL을 사용하는 모든 프로젝트는 `CMakeLists.txt` 파일을 통해 빌드 설정을 관리해야 한다. PCL의 모듈식 구조는 CMake와 긴밀하게 연동되어, '파이프라인을 코드로 정의(Pipeline as Code)'하는 개념을 가능하게 한다. 즉, 프로젝트의 `CMakeLists.txt` 파일은 단순한 빌드 스크립트를 넘어, 해당 프로젝트가 어떤 처리 단계를 포함하는지를 명시하는 선언적인 문서 역할을 한다.

사용자 프로젝트의 `CMakeLists.txt`에서 PCL을 연동하는 표준적인 방법은 다음과 같다 108:

1. **`find_package(PCL <version> REQUIRED)`**: 시스템에 설치된 PCL 라이브러리를 찾는다. `REQUIRED` 키워드는 PCL을 찾지 못하면 빌드 과정을 중단시킨다. `COMPONENTS` 키워드 뒤에 `io`, `filters`, `segmentation`과 같이 필요한 모듈의 이름을 명시하면, 해당 모듈과 그 의존성만 프로젝트에 연결된다. 이는 불필요한 라이브러리 링크를 방지하고 빌드 시간을 단축시킨다. 이 `COMPONENTS` 목록 자체가 프로젝트가 수행하는 작업의 종류(입출력, 필터링, 분할 등)를 고수준에서 설명해준다.
2. **`target_link_libraries(your_executable ${PCL_LIBRARIES})`**: `find_package`를 통해 찾아낸 PCL 라이브러리들(`PCL_LIBRARIES` 변수에 저장됨)을 실제 실행 파일에 링크한다. 최신 PCL 버전에서는 이 한 줄만으로 헤더 경로(`PCL_INCLUDE_DIRS`)와 컴파일러 정의(`PCL_DEFINITIONS`)까지 자동으로 처리된다.108

`find_package` 명령이 성공적으로 실행되면, `PCL_VERSION`, `PCL_INCLUDE_DIRS`, `PCL_LIBRARIES` 등 PCL 관련 정보가 담긴 여러 CMake 변수들이 설정되어 프로젝트에서 사용할 수 있게 된다.108


PCL을 성공적으로 설치하고 사용하는 것은 그 의존성들을 올바르게 관리하는 것에서 시작된다. 설치 방법은 크게 패키지 매니저를 이용하는 방법과 소스 코드를 직접 컴파일하는 방법으로 나뉜다.9

- **패키지 매니저를 통한 설치 (권장):** 대부분의 사용자에게 권장되는 가장 간편한 방법이다. 각 운영체제에 맞는 패키지 매니저를 사용하면 PCL뿐만 아니라 Boost, Eigen, FLANN, VTK 등 복잡한 의존성까지 한 번에 자동으로 설치해준다.9
  - **Windows:** Microsoft에서 개발한 크로스플랫폼 패키지 매니저인 `vcpkg`를 사용하는 것이 공식적으로 권장된다. `.\vcpkg install pcl` 명령어로 간단히 설치할 수 있다.9
  - **macOS:** `Homebrew`를 사용하는 것이 가장 일반적이다. `brew install pcl` 명령어로 설치한다.9
  - **Linux (Ubuntu/Debian 기준):** `apt`를 사용하여 `libpcl-dev` 패키지를 설치한다. `sudo apt install libpcl-dev` 명령어를 사용한다.9
- **소스 코드 컴파일:** 최신 개발 버전을 사용하거나, 특정 컴파일 옵션을 적용해야 하는 고급 사용자를 위한 방법이다. 이 경우, PCL의 모든 필수 및 선택적 의존성 라이브러리들을 먼저 수동으로 설치하거나 소스 컴파일해야 한다. 그 후, 각 라이브러리의 경로를 CMake에 정확히 알려주는 복잡한 설정 과정이 필요할 수 있다.16


PCL은 3D 포인트 클라우드 처리를 위한 사실상의 산업 표준 라이브러리로서, 지난 10여 년간 학계와 산업계의 수많은 요구사항을 반영하며 방대하고 성숙한 기능 집합을 구축했다. Willow Garage와 ROS 생태계에서 태동한 배경 덕분에 로봇 공학의 실제 문제 해결에 매우 실용적인 도구들을 제공하며, BSD 라이선스 정책은 그 확산을 가속화했다.

PCL의 핵심적인 특징은 **모듈성**, **병렬 처리 지원**, 그리고 **강력한 서드파티 라이브러리와의 통합**으로 요약할 수 있다. 기능별로 분리된 모듈식 아키텍처와 CMake 기반의 유연한 빌드 시스템은 개발자가 필요한 기능만을 선택적으로 사용하여 효율적이고 유지보수하기 쉬운 애플리케이션을 만들 수 있게 한다. OpenMP와 oneAPI를 통한 병렬 및 이기종 컴퓨팅 지원은 대용량 데이터를 실시간으로 처리해야 하는 현대적 요구에 부응한다. 또한 Eigen, FLANN, VTK와 같은 검증된 라이브러리들을 기반으로 하여, 수학적 연산, 근접 이웃 탐색, 시각화 등 핵심 기능의 성능과 안정성을 보장한다.

PCL은 필터링, 특징 추출, 정합, 분할, 표면 재구성 등 포인트 클라우드 처리의 전 과정을 아우르는 포괄적인 알고리즘 파이프라인을 제공한다. `PointCloud<PointT>`라는 핵심 데이터 구조를 중심으로, 각 처리 단계에 맞는 다양한 알고리즘들이 유기적으로 연결되어 동작한다. 특히 `KdTree`와 `Octree`를 기반으로 하는 효율적인 탐색 메커니즘은 PCL의 거의 모든 기능의 근간을 이루는 중요한 요소다.

물론 C++11 이전 시대에 설계되어 Boost 라이브러리에 대한 의존성이 깊다는 점과 같은 기술적 유산도 존재하지만, 커뮤니티는 `std` 라이브러리를 활용하는 방향으로 꾸준히 현대화를 진행하고 있다. PCL은 3D 인식 분야에 입문하는 개발자에게는 포괄적인 학습 도구를, 숙련된 연구자에게는 강력하고 유연한 실험 플랫폼을 제공하는 대체 불가능한 프레임워크로 확고히 자리매김하고 있다.


1. PCL Tutorial: - The Point Cloud Library By Example - Jeff Delmerico, accessed July 24, 2025, http://www.jeffdelmerico.com/wp-content/uploads/2014/03/pcl_tutorial.pdf
2. pcl/Handbook - ROS Wiki, accessed July 24, 2025, http://wiki.ros.org/pcl/Handbook
3. Point Cloud Library - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Point_Cloud_Library
4. 3D is here: Point cloud library (PCL) - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/221068443_3D_is_here_Point_cloud_library_PCL
5. Navigating Common Pitfalls in Point Cloud Analysis - Mindkosh, accessed July 24, 2025, https://mindkosh.com/blog/navigating-common-pitfalls-in-point-cloud-analysis/
6. 11x Performance Gain: Latest Connext DDS, ROS 2 Performance Benchmarks - RTI, accessed July 24, 2025, https://www.rti.com/blog/latest-connext-dds-ros-2-performance-benchmarks
7. sensor_msgs/Reviews/2010-03-01 PointCloud2_API_Review - ROS Wiki, accessed July 24, 2025, [http://wiki.ros.org/sensor_msgs/Reviews/2010-03-01%20PointCloud2_API_Review](http://wiki.ros.org/sensor_msgs/Reviews/2010-03-01 PointCloud2_API_Review)
8. Adding your own custom PointT type - Point Cloud Library 0.0 documentation, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/latest/adding_custom_ptype.html
9. Case study: convert ROS message to PCL - CPP Optimizations diary, accessed July 24, 2025, https://cpp-optimizations.netlify.app/pcl_fromros/
10. Optimized ROS->PCL conversion - Open Robotics Discourse, accessed July 24, 2025, https://discourse.ros.org/t/optimized-ros-pcl-conversion/25833
11. pcl::PointCloud< PointT > Class Template Reference - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/diamondback/api/pcl/html/classpcl_1_1PointCloud.html
12. pcl::PointCloud< PointT > Class Template Reference - Point Cloud Library, accessed July 24, 2025, http://pointclouds.org/documentation/classpcl_1_1_point_cloud.html
13. Module common - Point Cloud Library (PCL), accessed July 24, 2025, http://pointclouds.org/documentation/group__common.html
14. [conversions] toPCLPointCloud2 or toROSMsg
15. ros2_cookbook/rclcpp/pcl.md at main - GitHub, accessed July 24, 2025, https://github.com/mikeferguson/ros2_cookbook/blob/main/rclcpp/pcl.md
16. sensor_msgs/msg/PointCloud2 Message - ROS Documentation, accessed July 24, 2025, https://docs.ros2.org/foxy/api/sensor_msgs/msg/PointCloud2.html
17. PointCloud2 - sensor_msgs: Humble 4.9.0 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/ros2_packages/humble/api/sensor_msgs/msg/PointCloud2.html
18. Built-in Message Type | Introduction to ROS2 and Robotics, accessed July 24, 2025, https://www.learnros2.com/ros/ros2-building-blocks/built-in-types/built-in-message-type
19. PointCloud2 and PointField - pcl - Robotics Stack Exchange, accessed July 24, 2025, https://robotics.stackexchange.com/questions/73981/pointcloud2-and-pointfield
20. Using sensor_msgs/PointCloud2 natively - Robotics Stack Exchange, accessed July 24, 2025, https://robotics.stackexchange.com/questions/70072/using-sensor-msgs-pointcloud2-natively
21. In-Place Modification of PointCloud2 message? - ROS Answers archive, accessed July 24, 2025, https://answers.ros.org/question/324666/
22. ROS2 pointcloud : r/ROS - Reddit, accessed July 24, 2025, https://www.reddit.com/r/ROS/comments/gkxyqk/ros2_pointcloud/
23. GitRepJo/pcl_example: A template ROS2 C++ node to test a PCL Pointcloud2 processing, accessed July 24, 2025, https://github.com/GitRepJo/pcl_example
24. Conversion from sensor_msgs::PointCloud2 to pcl::PointCloud
25. home/travis/build/PointCloudLibrary/pcl/common/include/pcl/ros/conversions.h Source File, accessed July 24, 2025, https://pointclouds.org/documentation/ros_2conversions_8h_source.html
26. ROS 2 Performance Benchmarking - ROS General - Open Robotics Discourse, accessed July 24, 2025, https://discourse.ros.org/t/ros-2-performance-benchmarking/44382
27. NVIDIA-ISAAC-ROS/ros2_benchmark: Benchmark the performance of your ROS 2 graphs, accessed July 24, 2025, https://github.com/NVIDIA-ISAAC-ROS/ros2_benchmark
28. ROS 2 benchmark open source release, accessed July 24, 2025, https://discourse.ros.org/t/ros-2-benchmark-open-source-release/30753
29. irobot-ros/ros2-performance: Framework to evaluate peformance of ROS 2 - GitHub, accessed July 24, 2025, https://github.com/irobot-ros/ros2-performance
30. PillarGen: Enhancing Radar Point Cloud Density and Quality via Pillar-based Point Generation Network - arXiv, accessed July 24, 2025, https://arxiv.org/html/2403.01663v2
31. How Can I Estimate the LiDAR Point Density for My Flight? - Knowledge Base - Inertial Labs, accessed July 24, 2025, https://resources.inertiallabs.com/en-us/knowledge-base/how-can-i-estimate-the-point-density-for-my-flight
32. Influence of point cloud density on the results of automated Object-Based building extraction from ALS data - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/262673446_Influence_of_point_cloud_density_on_the_results_of_automated_Object-Based_building_extraction_from_ALS_data
33. Frequency of depth and pointclouds got much slower when we subscribed compressedDepth / Issue #1672 / IntelRealSense/realsense-ros - GitHub, accessed July 24, 2025, https://github.com/IntelRealSense/realsense-ros/issues/1672


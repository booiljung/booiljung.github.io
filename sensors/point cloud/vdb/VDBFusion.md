# VDBFusion
[VDB](./index.md)


3차원 볼류메트릭 재구성(3D Volumetric Reconstruction)은 로보틱스, 자율주행, 가상 및 증강 현실(VR/AR) 등 현대 기술의 여러 핵심 분야에서 필수불가결한 기술로 자리 잡았다. 로봇은 주변 환경에 대한 정확한 지도를 통해 자신의 위치를 파악하고, 안전한 경로를 계획하며, 주어진 임무를 수행할 수 있다.1 특히 동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM) 기술은 자율 이동 로봇 시스템의 근간을 이루는 핵심 기술이다.2

초기 볼류메트릭 융합 연구의 대표주자인 KinectFusion과 같은 접근법들은 그래픽 처리 장치(GPU)의 강력한 병렬 처리 능력에 크게 의존하였다. 그러나 이러한 방식은 대규모 환경으로 확장할 때 메모리 사용량과 연산 비용이 기하급수적으로 증가하는 근본적인 한계를 안고 있었다.3 이 문제는 특히 배터리 전력과 컴퓨팅 자원이 제한적인 모바일 로봇 플랫폼에서 더욱 심각한 제약으로 작용했다.1

2022년, Vizzo 연구팀은 이러한 한계를 극복하기 위한 실용적인 대안으로 VDBFusion을 제안하였다.1 VDBFusion의 핵심 아이디어는 3D 재구성 분야에서 널리 검증된 절단 부호 거리 함수(Truncated Signed Distance Function, TSDF) 통합 기법을, 영화 및 시각 효과(VFX) 산업에서 기원하여 그 효율성이 입증된 희소 볼류메트릭 데이터 구조인 OpenVDB와 결합하는 것이었다.8 VDBFusion의 개발 목표는 명확했다. 특정 센서(예: RGB-D 카메라, LiDAR)나 환경의 크기에 대한 가정을 배제하고 1, 고가의 GPU 없이 단일 코어 중앙 처리 장치(CPU)만으로도 대규모 포인트 클라우드 데이터를 효율적으로 통합할 수 있는 유연하고 강력한 시스템을 제공하는 것이었다.1

이러한 접근은 단순한 기술적 개선을 넘어, 3D 맵핑 기술의 패러다임을 전환하려는 중요한 시도로 해석될 수 있다. 기존 연구가 GPU 중심의 고성능 컴퓨팅 환경에 집중했다면, VDBFusion은 자원 제약적인 실제 로보틱스 환경으로 실시간 3D 맵핑 기술의 실용성을 확장하려는 의도적인 방향 전환을 보여준다. 연구팀은 이 문제를 해결하기 위해 "GPU 요구 사항은 오늘날 로보틱스에서 여전히 한계"라고 명시적으로 지적하며 1, 문제 해결을 위해 컴퓨터 그래픽스 분야에서 이미 성숙 단계에 접어든 오픈소스 라이브러리 OpenVDB를 현명하게 채택했다.8 이 결정은 복잡한 데이터 구조를 처음부터 개발하는 수고를 덜고, TSDF '융합 알고리즘' 자체의 효율화에 집중할 수 있게 만들었다.8 그 결과, 64채널 라이다 데이터를 초당 20프레임(FPS)으로 단일 코어 CPU에서 처리 가능한 시스템이 탄생했으며 1, 이는 3D 맵핑 기술의 접근성을 크게 높이고 더 넓은 범위의 로봇 플랫폼에 적용할 수 있는 길을 열었다. 즉, VDBFusion의 가장 큰 기여는 완전히 새로운 알고리즘의 발명이라기보다는, 이종(異種) 분야의 성숙한 기술을 차용하여 기존 패러다임의 실용적 한계를 돌파한 탁월한 공학적 성취에 있다.

본 보고서는 VDBFusion의 핵심 원리인 TSDF 통합과 OpenVDB 데이터 구조를 시작으로, 기술적 구현, 성능 분석, 그리고 다양한 응용 분야를 순차적으로 심도 있게 분석한다. 나아가, NeRF, 3DGS와 같은 최신 3D 표현 기술과의 비교를 통해 VDBFusion의 기술적 위상을 객관적으로 평가하고, 그 한계와 미래 발전 방향을 딥러닝과의 융합 및 NeuralVDB의 등장과 같은 최신 기술 동향 속에서 종합적으로 조망하고자 한다.


VDBFusion의 효율성과 유연성은 세 가지 핵심 원리의 유기적인 결합에 기반한다: (A) 절단 부호 거리 함수(TSDF)를 이용한 점진적 표면 정보 융합, (B) OpenVDB라는 고효율 희소 데이터 구조를 통한 공간 표현, 그리고 (C) 이 둘을 결합한 실용적인 융합 파이프라인.


TSDF는 3D 공간을 모델링하는 강력하고 널리 사용되는 방법이다. 그 기본 개념은 3차원 공간을 일정한 크기의 복셀(voxel) 그리드로 나눈 뒤, 각 복셀의 중심점에서 가장 가까운 실제 표면(surface)까지의 거리를 저장하는 것이다. 이 거리는 부호를 가지는데, 표면을 기준으로 센서 쪽에 있는 공간(보이는 공간)은 양수, 표면 뒤쪽에 있는 공간(가려진 공간)은 음수 값을 가지며, 표면 자체는 정확히 0의 값을 가진다. 모든 공간에 대해 거리를 계산하는 것은 비효율적이므로, 연산 효율성을 위해 표면 주변의 특정 거리(truncation distance), 즉 $t$를 벗어나는 영역의 값은 최대값(+t) 또는 최소값(−t)으로 절단(truncate)한다. 이것이 '절단된(Truncated)' 부호 거리 함수라고 불리는 이유이다.1

VDBFusion은 이러한 TSDF를 점진적 융합(incremental fusion) 방식으로 사용하여 맵을 구축한다. 새로운 센서 데이터(예: 라이다 포인트 클라우드)가 들어올 때마다, 각 복셀의 TSDF 값과 해당 값의 신뢰도를 나타내는 가중치(weight)를 갱신한다. 이 과정은 가중 평균(weighted average) 방식을 통해 이루어지며, 여러 시점에서 관측된 정보를 통합함으로써 센서 측정에 포함된 노이즈를 점진적으로 줄여나가고, 데이터가 누적됨에 따라 더 정확하고 완전한 표면을 재구성하게 된다.11

수학적으로, $k-1$번째 프레임까지 통합된 복셀 $\mathbf{x}$의 TSDF 값 $D_{k-1}(\mathbf{x})$와 가중치 $W_{k-1}(\mathbf{x})$가 주어졌을 때, $k$번째 새로운 프레임에서 계산된 SDF 값 $d_k(\mathbf{x})$와 그 가중치 $w_k(\mathbf{x})$를 사용하여 다음과 같이 갱신된다.
$$
D_k(\mathbf{x}) = \frac{W_{k-1}(\mathbf{x})D_{k-1}(\mathbf{x}) + w_k(\mathbf{x})d_k(\mathbf{x})}{W_{k-1}(\mathbf{x}) + w_k(\mathbf{x})}$$
$$W_k(\mathbf{x}) = W_{k-1}(\mathbf{x}) + w_k(\mathbf{x})
$$
이 과정을 통해, 더 많은 관측이 쌓인 복셀일수록 더 높은 가중치를 갖게 되어 새로운 측정값의 영향을 덜 받게 되며, 이는 맵의 안정성과 정확성을 보장하는 핵심 메커니즘으로 작용한다.

### B. OpenVDB: 희소 볼류메트릭 데이터 구조

VDBFusion의 뛰어난 효율성은 핵심 데이터 구조로 채택한 OpenVDB에서 비롯된다. OpenVDB는 원래 영화 산업에서 복잡한 연기, 구름, 불 등의 시각 효과를 효율적으로 처리하기 위해 개발된 기술로, B+ 트리와 유사한 동적 계층 트리 구조를 사용하여 방대한 3차원 공간을 표현한다.8 이 구조는 '얕고 넓은(shallow and wide)' 형태를 가지며, 최하위 리프 노드(leaf node)는 일반적으로 8x8x8 크기의 복셀 블록으로 구성된다.8

이러한 VDB 구조는 전통적인 희소 데이터 구조인 옥트리(Octree)와 비교했을 때 몇 가지 뚜렷한 장점을 가진다.

- **접근 속도**: 옥트리는 공간을 8개의 자식 노드로 재귀적으로 분할하기 때문에 깊이가 가변적이며, 특정 복셀에 접근하기 위해서는 루트 노드부터 시작하는 순회(traversal) 과정이 필요하다. 반면, VDB는 고정된 깊이(fixed depth)의 트리 구조를 가지므로, 해시맵과 유사하게 거의 상수 시간($O(1)$)에 가까운 빠른 랜덤 접근이 가능하다.8 이는 수많은 복셀에 대해 반복적으로 읽고 쓰는 TSDF 융합 과정에서 성능을 극적으로 향상시킨다.
- **메모리 구조 친화성**: VDB의 계층 구조는 현대 CPU의 L1, L2 캐시와 같은 다층 캐시 메모리 아키텍처를 모방하여 설계되었다. 이는 데이터 지역성(data locality)을 극대화하여 캐시 효율을 높이고, 결과적으로 데이터 접근 속도를 더욱 가속화한다.8
- **무한 공간 표현**: 옥트리는 단일 루트 노드에서 시작하여 유한한 공간을 분할하지만, VDB는 여러 개의 루트 노드를 가질 수 있어 이론적으로 무한한(unbounded) 3D 인덱스 공간을 표현할 수 있다.9 이는 맵의 크기가 사전에 정해지지 않은 대규모 탐사 임무에 매우 유리하다.

또한, OpenVDB는 아카데미 과학기술상을 수상한 산업 표준 라이브러리로서, 시각화, 필터링, 압축, 레벨셋 연산 등 풍부하고 최적화된 도구 세트를 함께 제공하며, 활발한 커뮤니티에 의해 지속적으로 유지보수되고 있다. VDBFusion은 이러한 성숙한 생태계의 이점을 그대로 활용하여, 안정적이고 강력한 기반 위에서 개발될 수 있었다.8

### C. 융합 파이프라인

VDBFusion의 전체 파이프라인은 앞서 설명한 두 핵심 원리를 바탕으로 간결하고 효율적으로 구성된다.

1. **입력**: 시스템은 오직 두 종류의 데이터, 즉 3D 포인트 클라우드($P_i$)와 해당 데이터를 취득한 센서의 전역 포즈(위치와 방향, $T_i$)만을 입력으로 받는다.8 이는 특정 센서 모델에 대한 의존성을 제거하여 시스템의 유연성을 높인다.
2. **통합 (Integration)**: 핵심 로직은 `VDBVolume` 객체의 `integrate` 함수 내에 구현되어 있다. 이 함수는 입력된 포인트 클라우드를 센서 포즈를 이용해 월드 좌표계로 변환한 뒤, 각 포인트에 대해 영향을 받는 복셀 영역(주로 포인트와 센서 원점을 잇는 광선을 따라)을 식별한다. 이후, 해당 복셀들에 대해 TSDF 값과 가중치를 앞서 설명한 가중 평균 공식에 따라 계산하고 갱신한다.16
3. **주요 파라미터**: 사용자는 몇 가지 주요 파라미터를 통해 재구성 과정을 제어할 수 있다.
   - `voxel_size`: 재구성될 맵의 해상도를 결정한다. 값이 작을수록 더 세밀한 표현이 가능하지만, 메모리 사용량과 연산량이 증가한다.
   - `sdf_trunc`: TSDF의 절단 거리. 표면 주변의 어느 범위까지 유효한 거리 정보를 저장할지를 결정하며, 재구성될 표면의 두께에 영향을 미친다.
   - `space_carving`: 센서 원점과 측정된 포인트 사이의 공간을 '비어있는 공간(free space)'으로 명시적으로 처리할지 여부를 결정하는 옵션이다. 이 옵션을 활성화하면 관측되지 않은 공간(unseen)과 비어있다고 확인된 공간(empty)을 구분할 수 있어 경로 계획 등에 유용하지만, 추가적인 광선 투사(ray casting) 연산으로 인해 통합 속도가 저하된다.1
4. **출력**: 모든 데이터 프레임의 통합이 완료되면, 최종적으로 생성된 TSDF 볼륨으로부터 널리 알려진 Marching Cubes 알고리즘을 사용하여 표면을 나타내는 삼각형 메시(triangular mesh)를 추출할 수 있다.4 이 메시는 시각화, 시뮬레이션, 추가 분석 등 다양한 후속 작업에 사용된다.

VDBFusion의 이러한 설계 철학은 '추상화'와 '모듈화'에 깊이 뿌리내리고 있다. 개발자는 OpenVDB의 복잡한 트리 구조나 메모리 관리 방식의 내부 구현을 알 필요 없이, 제공되는 고수준 API를 통해 마치 균일하고 밀집된 3차원 배열을 다루는 것처럼 간단하게 볼류메트릭 맵을 생성하고 조작할 수 있다. 원본 논문과 GitHub 저장소는 "애플리케이션 개발자는 기본 데이터 구조 구현을 안전하게 무시할 수 있다"고 명시적으로 강조한다.8

`VDBVolume.integrate` 함수 하나가 복잡한 VDB 트리 순회, 노드 할당, TSDF 업데이트 과정을 모두 추상화하여 단일 호출로 처리해준다.16 이러한 높은 수준의 추상화는 기술의 사용 장벽을 크게 낮추고, 연구자나 개발자가 SLAM, 경로 계획 등 상위 레벨의 응용 프로그램 개발에 더 집중할 수 있도록 돕는다. 이는 VDBFusion이 단순한 알고리즘 구현을 넘어, 다른 시스템에 쉽게 통합될 수 있는 재사용 가능한 '소프트웨어 라이브러리' 또는 '컴포넌트'로서의 가치를 갖게 하는 핵심적인 이유이다.

## 3. 기술적 구현 및 활용

VDBFusion은 학술적 아이디어를 실제 개발자들이 쉽게 사용하고 통합할 수 있는 실용적인 소프트웨어로 구현하는 데 중점을 두었다. 이는 C++와 Python API 제공, 그리고 로봇 운영체제(ROS)와의 긴밀한 통합을 통해 달성되었다.

### A. C++ 및 Python API

VDBFusion의 핵심 로직은 성능을 극대화하기 위해 C++로 구현되었으며, 동시에 사용 편의성과 빠른 프로토타이핑을 위해 Python 바인딩을 제공한다.16 소스 코드의 언어 비중을 보면 C++가 약 57.5%, Python이 약 18.0%를 차지하여, 성능과 접근성의 균형을 맞추려는 의도를 엿볼 수 있다.16

특히 Python API는 사용 편의성에 초점을 맞춰 설계되었다. 입력 데이터(포인트 클라우드, 센서 원점)와 출력 데이터(메시 정점 및 삼각형)를 모두 표준적인 NumPy 배열 형태로 처리하도록 만들어졌다.1 이는 기존의 Python 기반 로보틱스나 컴퓨터 비전 파이프라인을 사용하는 개발자들이 복잡한 데이터 타입 변환 과정 없이 VDBFusion을 자신의 프로젝트에 손쉽게 '플러그인' 할 수 있게 한다. 이러한 설계는 프로토타이핑 속도를 비약적으로 향상시킨다.

라이브러리의 중심에는 `VDBVolume` 클래스가 있다. 사용자는 이 클래스의 객체를 생성하고, `integrate(points, origin)` 함수를 반복적으로 호출하여 순차적으로 들어오는 포인트 클라우드 데이터를 볼륨에 통합한다. 모든 데이터가 통합된 후에는 `extract_triangle_mesh()` 함수를 호출하여 최종 3D 메시를 추출할 수 있다.16 추출된 메시는 Open3D와 같은 널리 사용되는 외부 3D 라이브러리를 통해 간단하게 시각화할 수 있다.16

접근성을 높이기 위한 노력은 설치 과정에서도 드러난다. 대부분의 사용자는 터미널에서 `pip install vdbfusion`이라는 단 한 줄의 명령어로 라이브러리를 설치할 수 있다.16 물론, C++ API를 직접 사용하거나 소스 코드를 수정하려는 고급 사용자는 Conda와 같은 환경에서 직접 소스를 빌드해야 하는 옵션도 제공된다.17

### B. 로봇 운영체제(ROS) 통합

로보틱스 연구 및 개발 커뮤니티의 표준 플랫폼인 ROS(Robot Operating System)와의 통합은 VDBFusion의 실용성을 보여주는 중요한 부분이다. 이를 위해 `vdbfusion_ros`라는 공식 ROS 1 래퍼 패키지가 제공되어, ROS 기반 로봇 시스템에 VDBFusion의 맵핑 기능을 원활하게 통합할 수 있도록 지원한다.16

이 ROS 래퍼는 표준 ROS 메시 타입을 사용하여 다른 노드들과 통신한다. 주로 `sensor_msgs/PointCloud2` 토픽을 통해 포인트 클라우드 데이터를 수신하고, `tf2_msgs/TFMessage` 또는 `geometry_msgs/TransformStamped` 토픽을 통해 로봇의 포즈(TF 변환) 정보를 받아 맵을 생성하고 갱신한다.19 이러한 표준화된 인터페이스는 VDBFusion 노드를 기존의 SLAM 시스템이나 네비게이션 스택에 쉽게 연결할 수 있게 한다.

개발자들의 편의를 위해, 실제 데이터셋을 이용한 예제도 충실히 제공된다. 예를 들어, 자율주행 연구에서 널리 사용되는 KITTI 데이터셋을 `rosbag` 파일 형식으로 변환하여 `vdbfusion_ros` 노드를 실행하는 샘플 설정 파일이 포함되어 있어, 사용자들이 실제와 유사한 환경에서 라이브러리를 테스트하고 활용법을 익힐 수 있도록 돕는다.19 또한, 커뮤니티에서는 VDBFusion의 원본 래퍼를 기반으로 색상(color) 정보를 추가로 통합하거나 특정 좌표계 변환 문제를 해결하기 위해 파생된 `vdbfusion_mapping`과 같은 변형 버전들도 등장하였다.19

VDBFusion의 이러한 구현 및 배포 전략은 '학술적 재현성'과 '실용적 접근성' 사이에서 성공적인 균형을 이룬 모범 사례로 평가할 수 있다. 많은 학술 연구가 아이디어의 참신함에도 불구하고, 복잡한 설치 과정이나 불분명한 사용법 때문에 다른 연구자들이 재현하거나 활용하기 어려운 장벽에 부딪히곤 한다. VDBFusion 팀은 이 문제를 해결하기 위해 의도적으로 노력했다. `pip`을 통한 간편한 설치, NumPy를 통한 직관적인 데이터 교환, 명확한 예제 코드, 그리고 사용자 케이스 스터디 문서 제공에 이르기까지, 개발자의 입장에서 접근성을 높이기 위한 세심한 배려가 돋보인다.1 특히 로보틱스 커뮤니티의 핵심 플랫폼인 ROS 래퍼를 공식적으로 제공한 것은 이 기술을 단순한 '논문 속 아이디어'가 아닌, 현장에서 '사용 가능한 도구'로 포지셔닝하려는 강한 의지를 보여준다.19 이러한 노력의 결과, VDBFusion은 SHINE-Mapping, Occ3D, RGBTSDF 등 수많은 후속 연구에서 성능 비교를 위한 표준 베이스라인으로 널리 채택되었다.5 이는 VDBFusion의 기술적 우수성뿐만 아니라, 그 높은 접근성과 사용 편의성이 학술 커뮤니티로부터 신뢰를 얻었음을 방증한다.

## 4. 성능 분석 및 확장성

VDBFusion의 핵심 가치는 연산 효율성과 메모리 확장성에 있으며, 이는 자원이 제한된 로봇 플랫폼에서의 실시간 운영 가능성을 결정하는 중요한 요소이다.

### A. 연산 효율성 및 실시간성

VDBFusion이 기존 연구들과 차별화되는 가장 큰 특징 중 하나는 고가의 GPU 없이도 실시간에 가까운 성능을 달성했다는 점이다. 공개된 성능 측정 결과에 따르면, VDBFusion은 단일 코어 CPU 환경에서 64채널 LiDAR 센서로부터 초당 10회 입력되는 포인트 클라우드 데이터를 초당 20프레임 이상 처리할 수 있는 성능을 보여준다.1 이는 센서 데이터가 발생하는 속도보다 빠르게 맵을 갱신할 수 있음을 의미하며, 실시간 맵핑이 가능함을 입증한다. 이러한 높은 성능은 근본적으로 OpenVDB 데이터 구조의 효율적인 상수 시간($O(1)$) 접근 방식 덕분이다.8

자율주행 분야의 표준 벤치마크인 KITTI Odometry 데이터셋(Sequence 07)을 이용한 실험 결과는 VDBFusion의 우수성을 객관적으로 보여준다. 이 실험에서 VDBFusion은 약 45 FPS의 통합 속도를 기록하여, 대표적인 경쟁 기술인 Voxblox(약 15 FPS)나 Octomap(약 20 FPS)보다 2배에서 3배가량 빠른 성능을 보였다.1 흥미로운 점은 C++로 구현된 핵심 로직과 Python API 간의 성능 차이가 거의 없다는 것이다. Python API 역시 약 44 FPS의 대등한 성능을 기록했는데, 이는 Python의 느린 속도를 C++ 백엔드와 효율적인 데이터 연동(NumPy)으로 극복했음을 의미하며, 사용자가 프로토타이핑 단계에서부터 실제 운영 성능에 가까운 속도를 경험할 수 있게 한다.1

다만, `space_carving` 옵션의 사용 여부에 따라 성능은 크게 달라진다. 이 옵션은 센서와 측정된 표면 사이의 공간을 '비어있음(empty)'으로 명시적으로 처리하여, 관측되지 않은 공간(unseen)과 구분할 수 있게 해주는 유용한 기능이다. 그러나 이 과정에 추가적인 연산이 필요하므로 통합 속도는 현저히 저하된다. 따라서 응용 프로그램의 요구사항에 따라, 맵 정보의 풍부함과 처리 속도 사이에서 실용적인 트레이드오프를 선택할 수 있다.1

### B. 메모리 확장성

대규모 환경을 맵핑할 때 가장 큰 병목 중 하나는 메모리 사용량이다. VDBFusion은 OpenVDB의 희소(sparse) 데이터 구조를 채택하여 이 문제를 효과적으로 해결한다. 전체 3D 공간을 밀집(dense) 복셀 그리드로 표현하는 대신, 실제 표면이 존재하는 주변 영역의 복셀만 동적으로 할당하고 대부분의 비어있는 공간은 메모리에 저장하지 않는다. 이 방식은 밀집 그리드 방식에 비해 메모리 사용량을 극적으로 줄여준다.11

이러한 메모리 효율성 덕분에 VDBFusion은 사전에 환경의 크기나 범위를 가정할 필요 없이, 작은 실내 공간부터 넓은 실외 환경까지 유연하게 맵핑을 수행할 수 있다.1 특히, VDBFusion에서 활성화된 복셀의 수는 입력되는 데이터 프레임의 수에 비례하여 무한정 증가하는 것이 아니라, 실제 환경(scene)의 표면적 크기에 수렴하는 경향을 보인다. 이는 장시간 동안 로봇이 운용되어도 메모리 사용량이 안정적으로 유지될 수 있음을 의미한다.3

전통적인 TSDF 구현체들은 종종 메모리 효율성 한계로 인해 대규모 맵핑에 어려움을 겪는다.3 옥트리 기반 방법들 역시 메모리를 절약하는 좋은 대안이지만 23, VDB의 얕고 넓은 트리 구조와 효율적인 압축 방식은 더 나은 확장성을 제공할 잠재력을 가진다.8 그러나 VDBFusion이 메모리 효율성 측면에서 최종적인 해답은 아니다. 예를 들어, RGBTSDF라는 후속 연구는 ICL-NUIM 및 TUM 데이터셋을 이용한 실험에서 VDBFusion보다 1.5배에서 2배 더 나은 메모리 효율성을 달성했다고 보고했다.5 이는 VDBFusion의 접근법이 매우 효율적임에도 불구하고, 데이터 구조나 인덱싱 방식의 추가적인 개선을 통해 더 최적화될 여지가 남아있음을 시사한다.

다음 표는 주요 TSDF 융합 방식들의 성능을 표준 데이터셋에서 비교한 결과를 요약한 것이다. 이를 통해 각 기술의 런타임 및 메모리 효율성 트레이드오프를 한눈에 파악하고 VDBFusion의 상대적 위치를 명확히 이해할 수 있다.

| 방법 (Method)      | 기반 데이터 구조     | 주요 플랫폼 | 데이터셋    | 런타임 (FPS, 높을수록 좋음) | 메모리 (MB, 낮을수록 좋음) | 출처 |
| ------------------ | -------------------- | ----------- | ----------- | --------------------------- | -------------------------- | ---- |
| **VDBFusion**      | **VDB (B+ Tree)**    | **CPU**     | KITTI 07    | **~45**                     | N/A                        | 1    |
| VDBFusion (Python) | VDB (B+ Tree)        | CPU         | KITTI 07    | ~44                         | N/A                        | 1    |
| Voxblox            | Hashing / Voxel Grid | CPU/GPU     | KITTI 07    | ~15                         | N/A                        | 1    |
| Octomap            | Octree               | CPU         | KITTI 07    | ~20                         | N/A                        | 1    |
| **VDBFusion**      | **VDB (B+ Tree)**    | **CPU**     | ICL-NUIM    | 15.2                        | 102.3                      | 5    |
| Open3D             | Hash Grid            | CPU         | ICL-NUIM    | 14.5                        | 821.5                      | 5    |
| RGBTSDF            | Grid Octree          | CPU         | ICL-NUIM    | **34.1**                    | **67.3**                   | 5    |
| **VDBFusion**      | **VDB (B+ Tree)**    | **CPU**     | TUM fr1_xyz | 4.1                         | 189.2                      | 5    |
| Open3D             | Hash Grid            | CPU         | TUM fr1_xyz | 11.5                        | 1273.1                     | 5    |
| RGBTSDF            | Grid Octree          | CPU         | TUM fr1_xyz | **23.1**                    | **93.5**                   | 5    |

## 5. 주요 응용 분야

VDBFusion의 높은 효율성과 유연성은 다양한 첨단 기술 분야에서 그 활용 가능성을 입증하고 있다. 특히 로보틱스, 자율주행, 그리고 컴퓨터 그래픽스 분야에서 핵심적인 역할을 수행하거나 잠재력을 보이고 있다.

### A. 로보틱스 및 SLAM

VDBFusion의 가장 직접적이고 중요한 응용 분야는 로보틱스, 그중에서도 SLAM 시스템이다. SLAM은 로봇이 미지의 환경에서 자신의 위치를 추정(Localization)함과 동시에 주변 환경의 지도를 작성(Mapping)하는 기술이다.2 VDBFusion은 이 과정에서 맵핑(mapping) 부분을 담당하는 강력한 백엔드(back-end) 도구로 사용될 수 있다.1 VDBFusion이 실시간으로 생성하는 밀집된 3D 볼류메트릭 맵은 환경의 기하학적 구조를 정확하게 표현한다.

이렇게 생성된 맵은 로봇의 자율적인 행동을 위한 핵심 정보를 제공한다. 맵에 표현된 점유(occupied) 공간과 자유(free) 공간에 대한 명확한 정보는 로봇이 충돌 없이 안전하게 이동할 경로를 계획하고, 예기치 않은 장애물을 회피하며, 미지의 영역을 탐색(exploration)하는 데 직접적으로 활용된다.1 또한, VDBFusion은 RGB-D 카메라, 3D LiDAR 등 다양한 종류의 거리 측정 센서와 원활하게 연동될 수 있는 유연성을 갖추고 있어 1, 다양한 크기와 목적을 가진 로봇 플랫폼에 폭넓게 적용될 수 있다.

### B. 자율주행

자율주행 시스템에서 주변 환경을 정밀하게 인식하는 것은 안전과 직결되는 가장 중요한 과제이다. VDBFusion은 자율주행 차량에 탑재된 LiDAR 센서 데이터를 실시간으로 통합하여, 차량 주변 환경에 대한 3D 점유 격자(occupancy grid) 맵을 생성하는 데 효과적으로 사용될 수 있다.26 이러한 밀집 맵은 기존의 3D 바운딩 박스(bounding box) 표현 방식으로는 포착하기 어려운 비정형 장애물이나 복잡한 지형의 형태를 세밀하게 모델링할 수 있다는 장점이 있다.26

흥미롭게도, VDBFusion은 딥러닝 기반의 차세대 자율주행 인식 기술 개발에도 기여하고 있다. 3D 점유 예측(3D occupancy prediction)과 같은 최신 딥러닝 모델을 학습시키기 위해서는 방대한 양의 고품질 정답 데이터(ground truth)가 필요한데, VDBFusion이 이 데이터를 생성하는 파이프라인의 일부로 활용되는 것이다. 예를 들어, Occ3D 벤치마크는 실제 주행에서 얻은 희소한 LiDAR 포인트 클라우드를 VDBFusion을 이용해 조밀한 3D 메시로 재구성하고, 이를 바탕으로 각 복셀의 점유 상태를 레이블링하여 딥러닝 모델의 학습 데이터를 생성한다.21 이는 VDBFusion이 동적 객체 자체를 처리하지는 못하지만, 4D 재구성이나 동적 객체 분리와 같은 더 복잡한 연구에서 시공간적 일관성을 확보하기 위한 기반 맵 표현으로 사용될 수 있는 가능성을 보여준다.3

### C. 컴퓨터 그래픽스 및 가상/증강 현실

VDBFusion은 VR/AR 콘텐츠 제작을 위한 실세계 캡처(real-world capture) 도구로서의 잠재력도 가지고 있다. 현실 세계의 공간이나 객체를 3D 모델로 정밀하게 스캔하고 재구성하는 과정에 VDBFusion을 적용할 수 있다. 특히 CPU 기반의 효율적인 처리는 고가의 장비 없이도 현장에서 실시간으로 스캔 결과를 확인하고 피드백을 받을 수 있게 하여, 전체 작업의 효율성을 크게 향상시킬 수 있다. 이렇게 생성된 고품질 3D 메시는 VR/AR 환경의 가상 배경으로 사용되거나, 현실과 가상을 연결하는 디지털 트윈(Digital Twin)을 구축하는 기초 데이터로 활용될 수 있다.27 실제로 유럽의 ODIN 프로젝트에서는 VDBFusion으로 생성된 맵이 AR 애플리케이션에 활용된 사례가 언급되기도 했다.29

한편, 'VDBFusion'이라는 이름은 컴퓨터 그래픽스 분야에서 혼동을 일으킬 수 있다. Houdini, Cinema 4D와 같은 3D 그래픽 소프트웨어에서 사용되는 절차적 모델링(procedural modeling) 툴킷 중에도 동명의 'VDB Fusion'이 존재하기 때문이다.30 이 툴킷은 본 보고서에서 다루는 TSDF 통합 라이브러리와는 완전히 다른 도구이다. 그러나 이러한 이름의 혼동은 역설적으로 VDBFusion의 기반 기술인 OpenVDB가 VFX 및 컴퓨터 그래픽스 분야에서 얼마나 핵심적이고 보편적인 기술인지를 보여주는 증거이기도 하다.

이러한 응용 사례들을 종합해 보면, VDBFusion은 단순한 '재구성(Reconstruction)' 도구를 넘어 '인식(Perception)' 기술과의 중요한 다리 역할을 수행하고 있음을 알 수 있다. 순수한 기하학적 재구성 도구로 시작했지만, 그 결과물은 딥러닝 기반 인식 모델의 학습 데이터가 되거나, 더 복잡한 4D 시공간 인식 시스템의 기초가 되는 등 점차 '인식'의 영역으로 그 영향력을 확장하고 있다. 자율주행 분야의 연구자들이 VDBFusion으로 재구성된 모델을 단순히 시각화하는 데 그치지 않고, 이를 딥러닝 모델이 '인식'해야 할 대상의 정답지로 활용한 것이 대표적인 예이다.21 즉, 재구성의 결과물이 인식의 입력(학습 데이터)이 된 것이다. 이는 VDBFusion과 같은 강력한 명시적(explicit) 재구성 도구가, 딥러닝 기반의 암시적(implicit) 인식 모델의 성능과 데이터 효율성을 보완하고 향상시키는 데 핵심적인 역할을 할 수 있음을 시사한다. 두 기술은 서로를 대체하는 관계가 아니라, 상호 보완적인 공생 관계를 형성하며 발전해 나가고 있다.

## 6. 최신 3D 표현 기술과의 비교 분석

VDBFusion의 기술적 위상을 정확히 평가하기 위해서는, 동시대의 다른 3D 표현 기술들과의 비교가 필수적이다. 비교 대상은 크게 전통적인 명시적 표현 방식과 최근 각광받는 신경망 기반의 암시적 표현 방식으로 나눌 수 있다.

### A. VDBFusion 대 다른 명시적 표현

- **옥트리(Octree) 기반 TSDF**: 옥트리는 메모리 효율성을 높이기 위해 3D 공간을 계층적으로 분할하는 고전적인 희소 표현 방식이다.11 Octomap이나 Voxblox와 같은 시스템들이 이 방식을 활용한다. 그러나 VDBFusion의 기반인 VDB 데이터 구조는 고정된 트리 깊이로 인한 상수 시간 접근 속도와 캐시 친화적인 메모리 구조 덕분에, 특히 대규모 데이터에 대한 반복적인 랜덤 접근이 많은 TSDF 융합 작업에서 옥트리보다 더 높은 성능을 보인다.8
- **해싱(Hashing) 기반 TSDF**: Voxel Hashing과 같은 기법은 복셀 데이터를 해시 테이블에 저장하여 GPU 상에서 매우 빠른 접근 속도를 구현한다.11 이는 특히 GPU 가속을 적극적으로 활용하는 시스템에서 강력한 성능을 발휘한다. 하지만 해시 충돌(collision)의 가능성이 잠재하며, 데이터 지역성(data locality) 측면에서 VDB만큼 체계적으로 최적화되어 있지 않을 수 있다. VDBFusion은 이와 달리 CPU 중심의 설계를 통해 다른 종류의 효율성을 추구한 접근법이다.

### B. VDBFusion 대 신경망 기반 암시적 표현

#### 1. 뉴럴 래디언스 필드 (Neural Radiance Fields, NeRF)

- **표현 방식**: 두 기술의 가장 근본적인 차이는 표현 방식에 있다. VDBFusion은 3D 공간을 복셀 그리드로 나누고 각 복셀에 기하학적 정보(거리 값)를 명시적으로 저장하는 이산적인(discrete) 표현이다.33 반면, NeRF는 씬의 색상(radiance)과 밀도(density)를 다층 퍼셉트론(MLP)이라는 신경망의 가중치 안에 암시적으로(implicitly) 인코딩하는 연속적인(continuous) 함수이다.35
- **재구성 및 렌더링 속도**: VDBFusion은 입력되는 포인트 클라우드를 실시간으로 통합하여 기하학적 구조(메시)를 빠르게 생성하는 데 초점을 맞춘다.1 NeRF는 하나의 씬을 표현하기 위해 수 시간에서 수일에 걸친 긴 학습(최적화) 과정을 거쳐야 하며, 새로운 시점의 이미지를 렌더링하는 과정 역시 볼류메트릭 광선 추적(ray marching) 방식으로 인해 매우 느리다.35 FastNeRF, Instant-NGP와 같은 수많은 변형 기술들이 속도 개선을 이루었지만, 여전히 실시간 상호작용에는 한계가 있으며 근본적인 속도-품질 트레이드오프가 존재한다.41
- **품질 및 정확도**: NeRF의 가장 큰 강점은 시점 의존적 효과(specular reflection 등)와 미세한 투과 효과까지 표현하여 극도로 사실적인(photorealistic) 이미지를 생성하는 능력이다.39 VDBFusion은 텍스처나 광학 효과를 표현하지 못하는 대신, 다중 시점의 측정치를 기하학적으로 일관되게 융합하여 높은 정확도의 표면을 재구성하는 데 집중한다. 반대로 NeRF는 밀도 필드로부터 기하학을 추출하기 때문에, 표면이 불분명하거나 노이즈가 포함된 형태로 나타날 수 있다.44

#### 2. 3D 가우시안 스플래팅 (3D Gaussian Splatting, 3DGS)

- **표현 방식**: 3DGS는 씬을 수많은 3D 가우시안(Gaussian) 집합으로 명시적으로 표현한다. 각 가우시안은 위치, 크기(공분산), 방향, 색상, 불투명도와 같은 속성을 가지는 프리미티브(primitive)이다.36 이는 VDBFusion의 이산적인 복셀과 NeRF의 연속적인 MLP 사이의 하이브리드적인 특성을 가진다.
- **속도**: 3DGS는 속도 면에서 혁신을 이루었다. NeRF보다 훨씬 빠른 수십 분 내의 학습 시간과, 신경망 없이 고속 래스터화(rasterization) 파이프라인을 통해 렌더링하므로 초당 100프레임 이상의 극도로 빠른 실시간 렌더링이 가능하다.35 VDBFusion은 렌더링이 아닌 기하학 '통합'에 초점을 맞추므로 직접적인 속도 비교는 어렵지만, 최종 결과물을 실시간으로 시각화하는 능력에 있어서는 3DGS가 압도적인 우위를 보인다.
- **메모리 및 편집**: 3DGS는 NeRF보다 메모리 효율적일 수 있으며 45, 가우시안이라는 명시적인 프리미티브로 구성되어 있어 NeRF보다는 편집이 상대적으로 용이하다. 그러나 수십만에서 수백만 개의 가우시안으로 씬이 표현되기 때문에, 객체 단위의 직관적인 조작은 여전히 어려운 과제이다.43 반면 VDBFusion은 최종적으로 삼각형 메시를 추출하므로, 전통적인 컴퓨터 그래픽스 파이프라인에서 매우 쉽게 편집하고 활용할 수 있다.

아래 표는 VDBFusion, NeRF, 3DGS 세 가지 기술을 다양한 핵심 지표에 따라 종합적으로 비교하여, 각 기술의 장단점과 적합한 응용 분야를 명확히 보여준다. 이 표는 3D 표현 기술의 전체 스펙트럼에서 VDBFusion이 차지하는 위치를 이해하는 데 도움을 줄 것이다.

| 구분 (Criteria)    | **VDBFusion**                                                | **Neural Radiance Fields (NeRF)**                            | **3D Gaussian Splatting (3DGS)**                             |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **표현 방식**      | 명시적(Explicit), 이산적(Discrete) 볼류메트릭 TSDF 그리드    | 암시적(Implicit), 연속적(Continuous) 래디언스/밀도 필드 (MLP) | 명시적(Explicit), 프리미티브 기반 (3D 가우시안 집합)         |
| **핵심 강점**      | - **CPU 기반 실시간 통합** 1<br />- 기하학적 정확성 및 일관성- 대규모 환경 확장성 1 | - **최고 수준의 사실적 렌더링** 43<br />- 복잡한 광학 효과(반사, 투과) 표현 - 연속적이고 매끄러운 시점 변환 | - **극도로 빠른 실시간 렌더링** 35<br />- NeRF 대비 매우 빠른 학습 시간 35<br />- 우수한 시각적 품질 |
|                    |                                                              |                                                              |                                                              |
| **핵심 약점**      | - 동적 환경 처리 불가 49<br />- 텍스처/외형 정보 부재- 희소 입력에 대한 품질 저하 50 | - **매우 느린 학습 및 렌더링 속도** 35<br />- 큰 메모리 및 연산 자원 요구- 명시적 기하 구조 부재, 편집 어려움 36<br />- 복잡한 광학 효과 표현 한계 43 | - 개별 객체 단위 편집/조작의 어려움 43<br />- 흐릿한 입력 이미지에 취약 51 |
| **주요 응용 분야** | 로보틱스 SLAM, 자율주행 맵핑, 실시간 3D 스캐닝               | 고품질 Novel View Synthesis, VFX, 디지털 아카이빙            | 실시간 렌더링, VR/AR, 동적 장면 캡처, 게임                   |

## 7. 한계 및 과제

VDBFusion은 그 효율성과 실용성에도 불구하고, 명시적, 규칙 기반 방법론이 가지는 본질적인 한계들을 내포하고 있다. 이러한 한계는 동적 환경 처리의 어려움과 세밀한 디테일 표현의 제약으로 요약될 수 있다.

### A. 동적 환경 처리

VDBFusion을 포함한 대부분의 전통적인 TSDF 융합 기법은 근본적으로 '세상은 정적이다(static world assumption)'라는 가정하에 설계되었다.49 이 가정은 제어된 환경에서는 유효하지만, 사람이나 차량이 끊임없이 움직이는 실제 환경에서는 심각한 문제를 야기한다.

동적 객체가 존재하는 환경에서 VDBFusion을 그대로 사용하면, 움직이는 객체의 과거 위치와 현재 위치가 맵에 모두 중첩되어 기록된다. 이로 인해 맵 상에 잔상이 남는 '고스팅(ghosting)' 현상이 발생하여, 재구성된 맵의 기하학적 정확성과 신뢰도를 심각하게 훼손한다.25 예를 들어, 움직이는 보행자는 길게 늘어진 유령 같은 형태로 맵에 남게 된다.

이러한 문제를 해결하기 위해, VDBFusion 파이프라인의 전후에 동적 객체를 탐지하고 포인트 클라우드에서 제거하는 별도의 모듈을 추가하는 연구들이 진행되고 있다.52 하지만 이는 VDBFusion 자체의 기능이 아닌, 외부적인 처리를 통해 한계를 보완하는 방식이다. 반면, DynamicFusion 49 이나 최근의 4D-SLAM 52 같은 연구들은 시간 축을 포함하여 동적인 장면 자체를 모델링하려는 보다 근본적인 접근법을 제시한다. 이러한 맥락에서 VDBFusion은 동적 처리 능력을 갖춘 최신 시스템들과 비교 평가될 때, '정적 환경'을 다루는 강력한 베이스라인으로서의 역할을 수행한다.

### B. 세밀한 디테일 표현 및 희소 데이터 의존성

VDBFusion의 재구성 품질은 입력되는 포인트 클라우드의 품질과 밀도에 직접적으로 의존하는 한계를 가진다. 특히 360도 회전형 LiDAR 센서처럼 측정 거리가 멀어질수록 포인트 간 간격이 넓어져 데이터가 희소해지는 경우, 이 한계는 더욱 두드러진다. 희소한 입력 데이터는 표면의 세밀한 디테일을 복원하는 것을 어렵게 만들고, 데이터가 전혀 없는 영역은 맵 상에 그대로 구멍(hole)으로 남게 된다.3

또한, TSDF의 근간이 되는 복셀화(voxelization) 과정 자체도 부작용을 낳는다. 연속적인 3D 공간을 이산적인 복셀 단위로 나누기 때문에, 재구성된 메시 표면에는 미세한 계단 현상(aliasing)이나 각진 형태의 '블로킹 효과(blocking effect)'가 나타날 수 있다.5 이 현상은 특히 낮은 해상도(큰 복셀 크기)로 맵을 생성할 때 더욱 명확하게 관찰된다.

TSDF의 가중 평균 융합 방식 역시 양날의 검이다. 이 과정은 센서 노이즈를 효과적으로 제거하고 표면을 부드럽게 만드는 장점이 있지만, 동시에 건물의 날카로운 모서리나 조각상의 정교한 문양과 같은 고주파(high-frequency)의 기하학적 디테일까지 함께 뭉개버리는 평활화(smoothing) 효과를 유발할 수 있다.5

이러한 VDBFusion의 한계들은 '표현력의 근본적인 트레이드오프'를 명확히 보여준다. VDBFusion은 CPU 기반의 효율성과 대규모 확장성을 확보하기 위해 '측정된 거리 값을 가중 평균한다'는 단순화된 물리적 모델을 채택했다. 이로 인해 발생하는 동적 처리 불가, 희소 데이터에 대한 취약성, 디테일 손실 등의 한계는 필연적인 결과이다. 바로 이 지점에서 후속 연구들이 왜 '학습(learning)' 기반의 접근법으로 이동했는지가 명확해진다. 신경망 기반의 표현 방식은 데이터로부터 씬의 '사전 지식(prior)'을 학습하여, 관측되지 않은 부분을 자연스럽게 채우거나 33, 시간의 흐름에 따른 복잡한 변화를 모델링할 수 있는 52 잠재력을 제공하기 때문이다. 따라서 VDBFusion의 한계는 기술 자체의 결함이라기보다는, 명시적이고 규칙에 기반한 모델링 방식이 가진 본질적인 한계를 드러내는 것이며, 이는 3D 비전 분야의 연구가 왜 더 복잡하고 데이터 주도적인 '학습' 기반 모델로 자연스럽게 진화했는지를 설명하는 중요한 기술적 이정표 역할을 한다. VDBFusion은 '고전적 방법론의 정점'인 동시에 '새로운 방법론의 필요성'을 역설적으로 보여주는 기술인 셈이다.

## 8. 미래 연구 동향 및 발전 방향

VDBFusion이 제시한 효율적인 볼류메트릭 융합의 개념은 그 자체로 완결된 것이 아니라, 딥러닝과 같은 최신 기술과 결합하며 새로운 가능성을 열어가고 있다. VDBFusion의 미래는 독립적인 기술로서의 생존보다는, 더 큰 시스템의 핵심 구성 요소로서 다른 기술들과 전략적으로 융합하는 방향으로 진화하고 있다.

### A. 딥러닝과의 융합

VDBFusion의 한계, 특히 표현력의 한계를 극복하기 위해 VDB의 효율적인 공간 관리 능력과 딥러닝의 강력한 표현력을 결합하는 하이브리드 방식이 활발히 연구되고 있다.

- **SHINE-Mapping**: VDBFusion을 직접적인 비교 대상으로 삼는 이 연구는, VDB 데이터 구조 대신 옥트리 기반의 계층 구조를 사용하되, 각 복셀에 고정된 TSDF 값을 저장하는 대신 학습 가능한 신경망 특징(learnable neural features)을 저장한다.20 이 특징들을 작은 신경망에 통과시켜 SDF 값을 추론함으로써, VDBFusion보다 더 적은 메모리를 사용하면서도 더 완벽하고 부드러운 표면 재구성이 가능함을 보여주었다.33
- **VDB-GPDF**: 이 연구는 VDB 구조에 가우시안 프로세스(Gaussian Process)라는 확률 모델을 결합한다. 이를 통해 각 지점의 거리 값에 대한 불확실성을 명시적으로 모델링하고, 데이터가 없는 지점의 거리 값을 확률적으로 추론(생성)하는 능력을 부여한다. 이는 VDBFusion이 가지지 못했던 씬 완성(scene completion) 능력을 일부 제공하여, 희소한 데이터로부터 더 완전한 맵을 생성할 수 있게 한다.58
- **딥러닝 파이프라인의 모듈**: VDBFusion은 그 자체로 딥러닝 기반의 복잡한 파이프라인에서 하나의 핵심 모듈로 활용되기도 한다. 예를 들어, 라이다와 카메라 센서 데이터를 융합하여 의미론적 분할(semantic segmentation)을 수행하기 위한 전처리 단계에서, 4D 시공간 재구성을 위해 VDBFusion이 사용되는 사례가 있다.4

### B. NeuralVDB: 차세대 VDB의 등장

NVIDIA에서 개발한 NeuralVDB는 VDB 기반 기술의 미래에 큰 영향을 미칠 수 있는 혁신적인 기술이다. NeuralVDB는 OpenVDB 데이터 구조 자체를 인공지능, 즉 신경망을 이용해 압축하는 기술이다.59 그 핵심은 VDB 트리의 하위 노드 구조(topology)와 각 복셀에 저장된 값(value)들을 작고 효율적인 다층 퍼셉트론(MLP)으로 인코딩하여 저장하는 것이다.61

이 기술의 효과는 놀랍다. 기존의 고도로 압축된 OpenVDB 파일 대비 메모리 사용량을 최대 100배까지 줄일 수 있다.59 이는 극도로 높은 해상도의 대규모 볼류메트릭 데이터를 개인용 워크스테이션이나 심지어 랩탑에서도 원활하게 다룰 수 있게 됨을 의미한다.

NeuralVDB 기술이 VDBFusion과 같은 시스템에 미치는 함의는 명확하다. 이 기술이 성숙하고 보편화된다면, VDBFusion 기반의 맵핑 시스템은 지금보다 훨씬 적은 메모리 자원으로 훨씬 더 크고 상세한 3D 맵을 실시간으로 생성할 수 있게 될 것이다. 이는 VDB 기반 방법론의 경쟁력과 수명을 한 단계 더 끌어올릴 수 있는 중요한 잠재력이며, 명시적 표현 방식의 한계를 극복하는 데 기여할 수 있다.

### C. 동적 장면 및 실시간 상호작용으로의 확장

VDBFusion의 정적인 한계를 넘어서, 시간 축을 포함하는 4D 시공간을 모델링하는 연구가 3D 비전 분야의 주요 화두로 떠오르고 있다. 이는 VDB와 같은 효율적인 데이터 구조를 동적인 변화를 표현할 수 있도록 확장하는 연구를 포함한다.9 VDBFusion의 후속 연구들은 실시간으로 동적 객체를 필터링하여 정적인 배경 맵의 일관성을 유지하거나 57, 더 나아가 움직이는 객체들을 개별적으로 추적하고 모델링하여 전체 동적 씬을 지속적으로 업데이트하는 방향으로 나아가고 있다.4 이러한 발전은 로봇이 단순히 정적인 지도를 보고 움직이는 것을 넘어, 실시간으로 변화하는 동적인 환경까지 완벽하게 이해하고 상호작용하는 미래를 가능하게 할 것이다.

이러한 미래 연구 동향들은 VDBFusion의 아이디어, 특히 '효율적인 VDB 구조 위에서 3D 정보를 통합한다'는 핵심 개념이 여전히 유효하며, 다른 첨단 기술과 결합될 때 더 큰 시너지를 낼 수 있음을 보여준다. VDBFusion은 미래 기술에 의해 완전히 대체되는 것이 아니라, 그 핵심 가치인 '효율적인 희소 공간 관리' 능력을 바탕으로 미래 기술의 일부로 흡수되고 진화하는 '플랫폼' 또는 '구성 요소'로서의 역할을 수행하고 있다.

## 9. 결론

VDBFusion은 로보틱스와 3D 컴퓨터 비전 분야에서 중요한 기술적 이정표를 제시했다. 이 기술은 널리 검증된 TSDF 융합 기법과 영화 산업에서 수십 년간 단련된 고효율 희소 데이터 구조 OpenVDB를 독창적으로 결합함으로써, 고가의 GPU 장비 없이 범용 CPU만으로 대규모 3D 환경을 실시간에 가깝게 재구성할 수 있는 실용적인 길을 열었다. 이는 특히 컴퓨팅 및 전력 자원이 제한적인 모바일 로보틱스 분야에서 고품질 3D 맵핑 기술의 적용 문턱을 크게 낮춘 의미 있는 공학적 성취라 할 수 있다.

기술적 위상 측면에서 VDBFusion은 명시적이고 규칙에 기반한 전통적인 볼류메트릭 재구성 방법론의 정점에 위치한 기술로 평가할 수 있다. 그 탁월한 효율성과 기하학적 정확성은 수많은 후속 연구에서 성능 비교를 위한 표준 베이스라인으로 채택될 만큼 강력함을 입증했다. 그러나 동시에, 동적 환경 처리의 부재와 희소한 입력 데이터로부터 세밀한 디테일을 표현하는 데 따르는 명백한 한계는, 3D 비전 분야의 연구가 NeRF, 3DGS와 같은 학습 기반의 암시적 또는 하이브리드 표현 방식으로 나아가게 된 자연스러운 기술적 흐름을 명확히 보여주는 이정표 역할을 한다.

VDBFusion의 미래는 단독 기술로서의 발전보다는, 더 크고 지능적인 시스템의 핵심 '구성 요소(component)'로서 그 가치를 이어갈 것으로 전망된다. VDB의 효율적인 공간 관리 능력은 신경망의 강력한 표현력과 결합하여(예: SHINE-Mapping), 기존의 한계를 뛰어넘는 더 정확하고 완성도 높은 하이브리드 맵핑 시스템을 탄생시키는 데 기여할 것이다. 또한, NVIDIA의 NeuralVDB와 같은 차세대 압축 기술은 VDB 기반 시스템의 메모리 효율성을 극한으로 끌어올려, 미래의 초고해상도, 초대규모 3D 인식 시스템에서 VDBFusion의 유산이 계속해서 중요한 역할을 할 것임을 시사한다.

결론적으로, VDBFusion은 '고전적 방법론의 완성'인 동시에 '차세대 지능형 맵핑의 초석'으로서 그 역사적, 기술적 가치를 인정받을 것이다. 이는 효율성, 실용성, 그리고 개방성을 통해 학계와 산업계 모두에 실질적인 기여를 한 성공적인 연구 사례로 기억될 것이다.

#### 참고자료

1. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - MDPI, accessed August 5, 2025, https://www.mdpi.com/1424-8220/22/3/1296
2. Improving SLAM Techniques with Integrated Multi-Sensor Fusion for 3D Reconstruction - PMC - PubMed Central, accessed August 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014387/
3. The VDB data structure [15] compared to octrees [52]. A conventional... - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/The-VDB-data-structure-15-compared-to-octrees-52-A-conventional-octree-subdivides_fig2_358523933
4. High-level overview of VDBFusion. Our system only takes as input point... - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/High-level-overview-of-VDBFusion-Our-system-only-takes-as-input-point-clouds-Pi-with_fig3_358523933
5. RGBTSDF: An Efficient and Simple Method for Color Truncated Signed Distance Field (TSDF) Volume Fusion Based on RGB-D Images - MDPI, accessed August 5, 2025, https://www.mdpi.com/2072-4292/16/17/3188
6. ‪Ignacio Vizzo‬ - ‪Google Scholar‬, accessed August 5, 2025, https://scholar.google.com/citations?user=nTjF-kkAAAAJ&hl=en
7. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - PubMed, accessed August 5, 2025, https://pubmed.ncbi.nlm.nih.gov/35162040/
8. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/358523933_VDBFusion_Flexible_and_Efficient_TSDF_Integration_of_Range_Sensor_Data
9. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - Frontiers, accessed August 5, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2020.600387/full
10. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - PMC, accessed August 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7935507/
11. 3D-SLNR: A Super Lightweight Neural Representation for Large-scale 3D Mapping - CVPR, accessed August 5, 2025, https://cvpr.thecvf.com/virtual/2025/poster/33040
12. OctNetFusion: Learning Depth Fusion from Data - Andreas Geiger, accessed August 5, 2025, https://www.cvlibs.net/publications/Riegler2017THREEDV.pdf
13. VDB: High-Resolution Sparse Volumes with Dynamic Topology - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/259288658_VDB_High-Resolution_Sparse_Volumes_with_Dynamic_Topology
14. About OpenVDB, accessed August 5, 2025, https://www.openvdb.org/about/
15. What is OpenVDB and why should you care? [thinkingParticles Documentation], accessed August 5, 2025, https://cebas.com/manual/LifeLicenser/doku.php?id=reference_guide:thinkingparticles_nodes:operator_nodes:openvdb:what_is_openvdb
16. PRBonn/vdbfusion: C++/Python Sparse Volumetric TSDF Fusion - GitHub, accessed August 5, 2025, https://github.com/PRBonn/vdbfusion
17. vdbfusion/INSTALL.md at main - GitHub, accessed August 5, 2025, https://github.com/PRBonn/vdbfusion/blob/master/INSTALL.md
18. vdbfusion user case study, accessed August 5, 2025, https://www.ipb.uni-bonn.de/html/software/vdbfusion/vdbfusion_user_case_study.pdf
19. PRBonn/vdbfusion_ros: ROS1 Wrapper for VDBFusion https://github.com/PRBonn/vdbfusion - GitHub, accessed August 5, 2025, https://github.com/PRBonn/vdbfusion_ros
20. README.md - PRBonn/SHINE_mapping - GitHub, accessed August 5, 2025, https://github.com/PRBonn/SHINE_mapping/blob/master/README.md
21. CVPR Poster Accurate Training Data for Occupancy Map Prediction in Automated Driving Using Evidence Theory, accessed August 5, 2025, https://cvpr.thecvf.com/virtual/2024/poster/29493
22. Disaggregated Design for GPU-Based Volumetric Data Structures - arXiv, accessed August 5, 2025, https://arxiv.org/html/2503.07898v1
23. Volumetric 3D Mapping in Real-Time on a CPU - Computer Vision Group, accessed August 5, 2025, https://cvg.cit.tum.de/_media/spezial/bib/steinbruecker_etal_icra2014.pdf
24. A review of visual SLAM for robotics: evolution, properties, and future applications - Frontiers, accessed August 5, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2024.1347985/full
25. Real-Time Spatial Reasoning by Mobile Robots for Reconstruction and Navigation in Dynamic LiDAR Scenes - arXiv, accessed August 5, 2025, https://arxiv.org/html/2505.12267v1
26. Vision-based 3D occupancy prediction in autonomous driving: a review and outlook - arXiv, accessed August 5, 2025, https://arxiv.org/html/2405.02595v1
27. How do we execute | VR, AR, MR Development Process | FusionVR - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=Jh3Cbh3iJAQ
28. Towards Large-Scale Incremental Dense Mapping using Robot-centric Implicit Neural Representation - arXiv, accessed August 5, 2025, https://arxiv.org/html/2306.10472v3
29. Open-Digital-Industrial and Networking pilot lines using ... - ODIN, accessed August 5, 2025, https://odin-h2020.eu/ODIN_D5.5_Final_Version.pdf
30. VDBFusion - Orbolt, accessed August 5, 2025, https://www.orbolt.com/asset/TimVanHelsdingen::VDBFusion
31. HVOFusion: Incremental Mesh Reconstruction Using Hybrid Voxel Octree - arXiv, accessed August 5, 2025, https://arxiv.org/html/2404.17974v1
32. Efficient Octree-Based Volumetric SLAM Supporting Signed-Distance and Occupancy Mapping - Department of Computing, accessed August 5, 2025, https://www.doc.ic.ac.uk/~sleutene/publications/EVespaRAL_final.pdf
33. 3QFP: Efficient neural implicit surface reconstruction using Tri-Quadtrees and Fourier feature Positional encoding - arXiv, accessed August 5, 2025, https://arxiv.org/html/2401.07164v1
34. Efficient neural implicit surface reconstruction using Tri-Quadtrees and Fourier feature Positional encoding - DiVA, accessed August 5, 2025, http://oru.diva-portal.org/smash/get/diva2:1909147/FULLTEXT01.pdf
35. Novel view synthesis, NeRF vs Gaussian splatting : r/computervision - Reddit, accessed August 5, 2025, https://www.reddit.com/r/computervision/comments/1if2w7f/novel_view_synthesis_nerf_vs_gaussian_splatting/
36. GauStudio: A Modular Framework for 3D Gaussian Splatting and Beyond - arXiv, accessed August 5, 2025, https://arxiv.org/html/2403.19632v1
37. Gaussian Splatting with NeRF-based Color and Opacity - arXiv, accessed August 5, 2025, https://arxiv.org/html/2312.13729v5
38. A Probabilistic Formulation of LiDAR Mapping with Neural Radiance Fields - arXiv, accessed August 5, 2025, https://arxiv.org/html/2411.01725v1
39. NeRFusion: Fusing Radiance Fields for Large-Scale Scene Reconstruction - CVF Open Access, accessed August 5, 2025, https://openaccess.thecvf.com/content/CVPR2022/papers/Zhang_NeRFusion_Fusing_Radiance_Fields_for_Large-Scale_Scene_Reconstruction_CVPR_2022_paper.pdf
40. Evaluating 3D Reconstruction: A Side-by-Side Comparison of NeRF and Gaussian Splatting Under Various Filming Techniques - Preprints.org, accessed August 5, 2025, https://www.preprints.org/manuscript/202504.1068/v1
41. Instant-NGP: 3D Reconstruction in Seconds with NERF Optimized - Reddit, accessed August 5, 2025, https://www.reddit.com/r/computervision/comments/1ifggqs/instantngp_3d_reconstruction_in_seconds_with_nerf/
42. FastNeRF: High-Fidelity Neural Rendering at 200FPS [Extended] - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=mi5b142WEmw
43. Gaussian splatting vs. photogrammetry vs. NeRFs – pros and cons - Teleport by Varjo, accessed August 5, 2025, https://teleport.varjo.com/blog/photogrammetry-vs-nerfs-gaussian-splatting-pros-and-cons
44. Neural Surface Reconstruction and Rendering for LiDAR-Visual Systems - arXiv, accessed August 5, 2025, https://arxiv.org/html/2409.05310v1
45. The Battle For Realism in 3D Rendering: A Brief Overview of NeRFs vs. Gaussian Splatting | by Aahana | Antaeus AR | Medium, accessed August 5, 2025, https://medium.com/antaeus-ar/the-battle-for-realism-in-3d-rendering-a-brief-overview-of-nerfs-vs-gaussian-splatting-580cff4d8801
46. Gaussian Splatting: A Technique for Efficient Real-Time Rendering - Generative AI, accessed August 5, 2025, https://generativeai.pub/gaussian-splatting-a-technique-for-efficient-real-time-rendering-9e029383d206
47. 4D Gaussian Splatting for Real-Time Dynamic Scene Rendering - CVPR 2024 Open Access Repository - The Computer Vision Foundation, accessed August 5, 2025, https://openaccess.thecvf.com/content/CVPR2024/html/Wu_4D_Gaussian_Splatting_for_Real-Time_Dynamic_Scene_Rendering_CVPR_2024_paper.html
48. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=T_kXY43VZnk
49. DynamicFusion: Reconstruction and Tracking of Non-rigid Scenes in Real-Time - UW Robotics and State Estimation Lab, accessed August 5, 2025, https://rse-lab.cs.washington.edu/papers/dynamic-fusion-cvpr-2015.pdf
50. Qualitative comparison between Voxblox and VDBFusion, where we... - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/Qualitative-comparison-between-Voxblox-and-VDBFusion-where-we-extracted-for-both_fig11_358523933
51. ECCV Poster Deblurring 3D Gaussian Splatting, accessed August 5, 2025, https://eccv.ecva.net/virtual/2024/poster/905
52. 3D LiDAR Mapping in Dynamic Environments using a 4D Implicit Neural Representation, accessed August 5, 2025, https://cvpr.thecvf.com/virtual/2024/poster/31240
53. The mapping results of SLAMesh and VDBFusion with dynamic removal on KITTI Seq 07. We enlarged the area shown in the red box to compare the details. - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/The-mapping-results-of-SLAMesh-and-VDBFusion-with-dynamic-removal-on-KITTI-Seq-07-We_fig5_385352580
54. CVPR Poster 4DTAM: Non-Rigid Tracking and Mapping via Dynamic Surface Gaussians, accessed August 5, 2025, https://cvpr.thecvf.com/virtual/2025/poster/32594
55. 3D LiDAR Mapping in Dynamic Environments using a 4D Implicit Neural Representation - CVF Open Access, accessed August 5, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Zhong_3D_LiDAR_Mapping_in_Dynamic_Environments_using_a_4D_Implicit_CVPR_2024_paper.pdf
56. Neural Implicit Representation for Highly Dynamic LiDAR Mapping and Odometry - arXiv, accessed August 5, 2025, https://arxiv.org/html/2409.17729v1
57. CAD-Mesher, accessed August 5, 2025, https://yaepiii.github.io/CAD-Mesher/
58. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 5, 2025, https://arxiv.org/html/2407.09649v1
59. Upping the Standard: NVIDIA Introduces NeuralVDB, Bringing AI and GPU Optimization to Award-Winning OpenVDB, accessed August 5, 2025, https://blogs.nvidia.com/blog/neuralvdb-ai/
60. NVIDIA NeuralVDB - NVIDIA Developer, accessed August 5, 2025, https://developer.nvidia.com/rendering-technologies/neuralvdb
61. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks - Research at NVIDIA, accessed August 5, 2025, https://research.nvidia.com/labs/prl/neuralvdb/neuralvdb2024.pdf
62. Optimizing Large-Scale Sparse Volumetric Data with NVIDIA NeuralVDB Early Access, accessed August 5, 2025, https://developer.nvidia.com/blog/optimizing-large-scale-sparse-volumetric-data-with-nvidia-neuralvdb-early-access/
63. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks - Research at NVIDIA, accessed August 5, 2025, https://research.nvidia.com/labs/prl/publication/neuralvdb/
64. [2503.08166] Dynamic Scene Reconstruction: Recent Advance in Real-time Rendering and Streaming - arXiv, accessed August 5, 2025, https://arxiv.org/abs/2503.08166
65. (PDF) Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/349471767_Hierarchical_Dense_and_Dynamic_3D_Reconstruction_Based_on_VDB_Data_Structure_for_Robotic_Manipulation_Tasks
66. Hierarchical, Dense and Dynamic 3D Reconstruction Based on VDB Data Structure for Robotic Manipulation Tasks - DOAJ, accessed August 5, 2025, https://doaj.org/article/032fee85d7a84c75a17baab02d160ee8


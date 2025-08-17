# VDBFusion과 OpenVDB에 대한 비교

## 서론

현대 컴퓨터 그래픽스(CG), 컴퓨터 비전, 로보틱스 분야에서 3차원 공간을 효율적으로 표현하고 처리하는 것은 핵심적인 과제이다. 특히 연기, 구름과 같은 비정형 볼륨 데이터나, 자율주행 로봇이 실시간으로 구축하는 대규모 환경 맵은 막대한 양의 데이터를 수반한다.1 이러한 희소(sparse)하고 동적인(dynamic) 3D 데이터를 어떻게 메모리 효율적으로 저장하고, 실시간으로 접근 및 조작할 것인가라는 근본적인 문제에 대한 해답으로 OpenVDB와 이를 활용한 VDBFusion이 등장했다.

본 보고서는 이 두 기술을 심층적으로 비교 고찰한다. 핵심 주장은 OpenVDB를 시각 효과(VFX) 산업의 요구에서 탄생한 범용적이고 확장 가능한 희소 볼류메트릭 데이터 구조 라이브러리로, VDBFusion을 로보틱스 분야의 특정 문제, 즉 실시간 3D 표면 재구성을 해결하기 위해 OpenVDB의 아키텍처를 창의적으로 활용한 응용 알고리즘으로 정의하는 것이다. 이 둘은 경쟁 관계가 아닌, 강력한 기반 기술과 그 위에 구축된 혁신적인 솔루션의 관계에 있다.4

보고서의 구성은 다음과 같다. 제1장에서는 OpenVDB의 탄생 배경, 핵심 데이터 구조, 그리고 인공지능(AI) 기술과 융합하며 진화하는 기술 생태계를 분석한다. 제2장에서는 VDBFusion의 목표와 TSDF(절단 부호 거리 함수) 통합 알고리즘의 원리를 OpenVDB 활용 전략과 연계하여 심층적으로 탐구한다. 제3장에서는 두 기술의 철학, 성능, 적용 분야를 다각적으로 비교하고, 마지막으로 제4장에서는 VDB 기반 기술의 미래와 3D 가우시안 스플래팅과 같은 새로운 패러다임과의 관계를 조망하며 결론을 맺는다.

## 1.  OpenVDB - 희소 볼륨 데이터 표현의 표준

### 1.1  개발 배경 및 철학: VFX 산업의 요구와 DreamWorks의 혁신

OpenVDB는 시각 효과(VFX) 산업의 실용적인 필요성에서 탄생했다. VFX 파이프라인에서 연기, 불, 구름, 물과 같은 유체(fluid) 및 무정형 재료(amorphous materials)를 컴퓨터 그래픽으로 표현하는 것은 매우 중요한 작업이다.2 과거에는 이러한 볼류메트릭 데이터를 표현하기 위해 주로 밀집 그리드(dense grid) 방식을 사용했다. 이 방식은 3차원 공간을 균일한 복셀(voxel)로 나누고 각 복셀에 밀도, 온도, 속도 등의 속성 값을 저장하는 직관적인 구조를 가졌지만, 치명적인 단점이 있었다. 연기나 구름과 같은 대상은 3차원 공간의 극히 일부만을 차지함에도 불구하고, 전체 경계 상자(bounding box)에 해당하는 모든 복셀에 메모리를 할당해야 했다. 이는 해상도가 높아질수록 메모리 요구량이 기하급수적으로 증가하는 문제로 이어져, 고품질 VFX 제작에 큰 걸림돌이 되었다.9

이러한 한계를 극복하기 위해 DreamWorks Animation의 Ken Museth를 필두로 한 연구팀이 VDB(Volumetric Dynamic B+tree)를 개발했다.10 VDB는 데이터가 실제로 존재하는 '활성(active)' 영역에만 메모리를 할당하는 희소(sparse) 데이터 구조를 채택하여 메모리 효율성을 극대화했다. 이 기술은 '장화신은 고양이 (Puss in Boots, 2011)'와 '가디언즈 (Rise of the Guardians, 2012)'와 같은 장편 애니메이션 제작에 처음으로 성공적으로 적용되어 그 실용성과 효율성을 입증했다.1

DreamWorks는 2012년 8월, VDB를 OpenVDB라는 이름으로 오픈소스화하여 VFX 커뮤니티에 공개했다.1 이는 특정 스튜디오의 독점 기술을 넘어 업계 전반의 표준으로 나아가는 중요한 전환점이었다. 현재 OpenVDB는 영화 산업의 오픈소스 소프트웨어를 지원하기 위해 설립된 Academy Software Foundation (ASWF)에 의해 유지보수되고 있으며, 이는 특정 기업에 종속되지 않고 여러 스튜디오와 개발자 커뮤니티의 협력을 통해 지속적으로 발전하는 건강한 생태계를 구축하는 데 크게 기여했다.1

### 1.2  핵심 데이터 구조 분석: B+트리 기반의 계층적 그리드

OpenVDB의 핵심 경쟁력은 B+트리(B+tree)와 유사한 구조를 3차원으로 확장한 독창적인 동적 계층적 데이터 구조에 있다. 이 구조는 세 가지 핵심적인 장점을 제공한다: 사실상 무한한 3D 인덱스 공간, 압축적인 저장소, 그리고 빠른 데이터 접근 속도이다.1

OpenVDB의 트리 구조는 일반적으로 루트 노드(Root Node), 내부 노드(Internal Node), 그리고 리프 노드(Leaf Node)로 구성된 고정된 깊이를 가진다. 리프 노드는 실제 복셀 데이터 값을 저장하는 데이터 블록이다. 이러한 고정 깊이 구조는 OpenVDB가 옥트리(Octree)와 같은 다른 계층적 공간 분할 구조와 구별되는 중요한 특징이다. 옥트리는 데이터의 분포에 따라 트리의 깊이가 가변적이어서, 깊은 곳에 위치한 노드에 접근하기 위해 재귀적인 순회(traversal)가 필요하며 이는 접근 시간의 변동성을 야기한다. 반면, OpenVDB는 해시 테이블과 유사한 방식으로 각 레벨의 노드에 접근하므로 특정 복셀 좌표에 대한 임의 접근(random access) 시간이 트리의 크기와 무관하게 거의 상수 시간(`$O(1)$`)에 가깝다. 이는 실시간 상호작용이나 빠른 데이터 처리가 요구되는 응용 프로그램에서 결정적인 이점으로 작용한다.4

또한, OpenVDB는 데이터 접근 속도를 높이기 위해 CPU 캐시 아키텍처를 고려하여 설계되었다. 데이터가 메모리에 연속적으로 배치되도록 하여 캐시 효율성을 높이고, 계층적 비트마스크(hierarchical bitmask)를 사용하여 활성 복셀 영역을 빠르게 건너뛰며 순차 접근(sequential access)할 수 있는 이터레이터(iterator)를 제공한다. 이를 통해 볼륨 전체를 스캔하는 작업의 속도를 크게 향상시킨다.10

### 1.3  주요 기능 및 도구 모음: 데이터 구조를 넘어선 종합 솔루션

OpenVDB는 단순히 효율적인 데이터 저장 형식을 제공하는 데 그치지 않고, 볼류메트릭 데이터를 생성, 조작, 분석하기 위한 방대하고 강력한 C++ 라이브러리 및 도구 모음을 포함하는 종합 솔루션이다.2

- **변환 (Conversion):** OpenVDB의 가장 강력한 기능 중 하나는 다른 3D 데이터 표현과의 상호 변환 능력이다. 폴리곤 메시(mesh)나 파티클(particle) 데이터로부터 VDB 볼륨(예: Signed Distance Field)을 견고하고 효율적으로 생성할 수 있으며, 반대로 VDB 볼륨을 적응형 메시(adaptive mesh)나 포인트 클라우드로 변환할 수도 있다.10 이는 기존의 VFX 및 시뮬레이션 파이프라인에 OpenVDB를 쉽게 통합할 수 있게 해준다.
- **레벨 셋 (Level Set):** OpenVDB는 레벨 셋 기법을 위한 풍부한 연산자들을 제공한다. 레벨 셋은 표면을 암시적으로 표현하는 방법으로, 유체 시뮬레이션이나 형상 모델링에서 널리 사용된다. OpenVDB는 다중 스레드를 지원하는 이류(advection), 평활화(smoothing), 필터링, 표면 추적, 그리고 거의 실시간으로 수행되는 부울 연산(합집합, 교집합, 차집합) 등을 포함하여 복잡한 동적 표면 변화를 효과적으로 처리할 수 있다.10
- **필터 및 연산:** 가우시안 필터와 같은 평활화 필터, 볼륨에 절차적 디테일을 추가하기 위한 노이즈 적용 필터는 DreamWorks의 영화 속 구름 모델링 도구의 기반을 형성했다. 또한, 팽창(dilation) 및 침식(erosion)과 같은 형태학적 연산은 동적 볼륨 처리 시 필수적이다. 이 외에도 기울기(gradient), 라플라시안(Laplacian), 컬(curl), 발산(divergence) 등 벡터 미적분 연산을 지원하여 물리 기반 시뮬레이션 및 분석에 활용된다.10
- **포인트 데이터 저장:** `PointDataGrid`는 OpenVDB의 또 다른 혁신적인 기능이다. 이는 포인트 클라우드 데이터를 VDB의 계층 구조 내에 직접 저장할 수 있게 해준다. 포인트들은 공간적으로 VDB 복셀에 구성되어 저장되므로, 선형 배열(linear array)로 저장하는 방식에 비해 특정 영역의 포인트에 더 빠르게 접근할 수 있으며, 공간적 중복성을 활용하여 더 높은 데이터 압축률을 달성할 수 있다.10
- **렌더링 및 시각화 지원:** OpenVDB는 자체적으로 독립 실행형 OpenGL 기반 볼륨 뷰어를 제공하며, 가속화된 광선 순회(ray traversal)를 활용하는 간단한 명령줄 레이 트레이서도 포함하고 있다. 더 중요한 것은, Houdini, Blender, Cinema 4D, RenderMan, Arnold 등 거의 모든 주요 DCC(Digital Content Creation) 툴과 렌더러에서 기본적으로 지원된다는 점이다. 이는 OpenVDB가 VFX 파이프라인의 사실상 표준 데이터 교환 포맷으로 자리 잡게 된 핵심적인 이유다.1

### 1.4  기술 생태계의 확장: GPU와 AI 시대로의 진화

OpenVDB는 초기 VFX 산업의 요구를 넘어, 하드웨어 및 알고리즘의 발전에 발맞추어 지속적으로 진화하며 그 생태계를 확장하고 있다. 특히 GPU 가속과 인공지능 기술의 접목은 OpenVDB의 활용 범위를 혁신적으로 넓히고 있다.

- **NanoVDB:** 2020년 NVIDIA가 기여한 NanoVDB는 OpenVDB 생태계의 중요한 변곡점이었다. NanoVDB는 OpenVDB의 핵심적인 희소 트리 구조를 유지하면서도, GPU에서의 실시간 처리에 최적화된 경량화된 정적 토폴로지(static-topology) 구현체이다.9 이는 기존의 CPU 기반 처리 방식에서 벗어나 대규모 병렬 처리가 가능한 GPU의 이점을 최대한 활용할 수 있게 해주었다. 결과적으로 NanoVDB는 VFX를 넘어 게임 엔진, 실시간 시각화, 인터랙티브 시뮬레이션 등 고성능이 요구되는 분야로 OpenVDB의 영향력을 확장하는 결정적인 계기가 되었다.
- **NeuralVDB:** 2022년 발표된 NeuralVDB는 NanoVDB의 GPU 가속을 기반으로 머신러닝을 접목한 차세대 기술이다. 이 기술의 핵심은 VDB로 표현된 볼륨 데이터를 콤팩트한 신경망 표현(neural representation)으로 변환하여 압축하는 것이다. NeuralVDB는 VDB 트리의 위상 정보(topology)와 값 정보(value)를 각각 신경 분류기(neural classifier)와 회귀기(regressor)를 사용하여 인코딩한다. 이를 통해 기존의 VDB 데이터 대비 메모리 사용량을 최대 100배까지 줄일 수 있다.20 이러한 압도적인 압축률은 매우 크고 복잡한 데이터셋을 제한된 메모리를 가진 개인용 워크스테이션이나 노트북에서도 다룰 수 있게 하며, 클라우드를 통한 데이터 전송 및 공유를 훨씬 효율적으로 만든다.
- **fVDB:** 가장 최근의 진화는 fVDB로, 이는 OpenVDB/NanoVDB 구조 위에 희소 컨볼루션(sparse convolution), 풀링(pooling), 어텐션(attention)과 같은 미분 가능한(differentiable) AI 연산자들을 구축한 딥러닝 프레임워크이다.34 fVDB는 VDB를 단순한 데이터 저장 구조를 넘어, 3D 공간 데이터를 이해하고 생성하는 AI 모델을 위한 핵심 인프라로 격상시킨다. 이를 통해 현실 세계의 대규모 디지털 트윈(digital twin) 구축, NeRF(Neural Radiance Fields) 스케일업, 3D 생성형 AI, 물리 AI(physical AI) 시뮬레이션 등 이전에는 불가능했던 규모의 공간 지능(spatial intelligence) 연구 및 응용이 가능해진다.

이러한 발전 과정을 종합해 보면, OpenVDB의 진화는 단순히 데이터를 '저장'하는 것에서 시작하여, GPU를 통해 '실시간으로 상호작용'하는 단계를 거쳐, 이제는 AI가 직접 데이터를 '연산'하고 '이해'하는 플랫폼으로 나아가는 뚜렷한 패러다임 전환을 보여준다. 이는 OpenVDB가 미래의 3D 데이터 처리 및 공간 AI 분야에서 계속해서 핵심적인 역할을 수행할 것임을 시사한다.

## 2.  VDBFusion - OpenVDB 기반의 실시간 3D 재구성

### 2.1  문제 정의: 로보틱스를 위한 대규모 TSDF 통합

로보틱스, 특히 SLAM(동시적 위치추정 및 지도작성)과 자율주행 분야에서, 로봇은 주변 환경을 인식하고 안전하게 탐색하기 위해 센서 데이터를 기반으로 3D 지도를 실시간으로 생성해야 한다.3 이 분야의 선구적인 연구 중 하나인 KinectFusion은 2011년에 발표되어, 저가의 RGB-D 센서(Kinect)를 사용하여 고품질의 3D 표면을 실시간으로 재구성하는 방법을 제시하며 큰 반향을 일으켰다.38 KinectFusion은 TSDF(절단 부호 거리 함수, Truncated Signed Distance Function)라는 데이터 표현을 사용하여, 센서로부터 들어오는 깊이 정보를 연속적으로 통합하고 부드러운 표면 모델을 생성했다.

그러나 KinectFusion은 명확한 한계를 가지고 있었다. 바로 고정된 크기의 밀집 복셀 그리드(dense voxel grid)를 사용한다는 점이다. 이는 재구성할 수 있는 공간의 크기가 GPU 메모리에 의해 엄격하게 제한됨을 의미했다. 따라서 방 하나 정도의 작은 실내 공간에서는 효과적이었지만, 더 넓은 공간이나 대규모 실외 환경으로 확장하는 것은 거의 불가능했다.42

VDBFusion은 바로 이 확장성 문제를 해결하기 위해 등장했다. VDBFusion의 핵심 아이디어는 KinectFusion의 강력한 TSDF 통합 알고리즘을 그대로 유지하되, 그 기반이 되는 데이터 구조를 밀집 그리드에서 OpenVDB의 희소 계층적 그리드로 교체하는 것이었다. 이를 통해 표면이 존재하는 주변의 좁은 영역(narrow band)에만 데이터를 저장하고, 나머지 방대한 빈 공간은 메모리를 차지하지 않도록 함으로써, 환경의 크기나 사용되는 센서의 종류(RGB-D 카메라, LiDAR 등)에 제약을 받지 않는 유연하고 효율적인 대규모 3D 매핑 시스템을 구축하는 것을 목표로 삼았다.4

### 2.2  알고리즘 심층 분석: 가중 평균 업데이트와 공간 조각 기법

DBFusion의 알고리즘적 핵심은 Curless와 Levoy의 연구에서 제안된 전통적인 TSDF 통합 방식을 충실히 따르는 것이다.4 TSDF 그리드의 각 복셀 x는 가장 가까운 표면까지의 부호가 있는 거리(signed distance) 값을 저장한다. 이 거리는 센서의 측정 한계를 고려하여 특정 값(truncation distance)으로 절단된다. 새로운 깊이 측정값이 들어오면, 각 복셀의 TSDF 값은 가중 평균(weighted average) 방식을 통해 점진적으로 갱신된다.

시간 t에서의 복셀 x의 TSDF 값 $D_t(x)$와 누적 가중치 $W_t(x)$는 이전 시간 t−1의 값들과 새로운 측정값 dt(x) 및 그 가중치 $w_t(x)$를 사용하여 다음과 같이 업데이트된다 4:
$$
D_t(x) = \frac{W_{t-1}(x) \cdot D_{t-1}(x) + w_t(x) \cdot d_t(x)}{W_{t-1}(x) + w_t(x)}$$
$$W_t(x) = W_{t-1}(x) + w_t(x)
$$
여기서 $d_t(x)$는 현재 측정된 포인트와 복셀 중심 사이의 투영된 부호 거리(projective signed distance)이며, $w_t(x)$는 측정의 신뢰도를 반영하는 가중치 함수이다. 이 과정을 통해 노이즈가 있는 센서 측정값들이 점차 평균화되어 부드럽고 일관된 표면이 형성된다.

데이터 통합 과정은 센서 원점(origin)에서 측정된 포인트 클라우드의 각 점까지 가상의 광선(ray)을 쏘는 방식으로 이루어진다. 이 광선이 통과하는 경로상의 복셀들이 업데이트 대상이 된다. 특히, 측정된 표면 주변의 절단 거리 내에 있는 복셀들만 TSDF 값 갱신이 이루어진다.4

VDBFusion은 여기에 `space_carving`이라는 옵션을 추가로 제공한다.5 이 옵션이 활성화되면, 센서 원점과 측정된 표면 사이의 공간, 즉 명백히 비어있는 공간(free space)에 해당하는 복셀들의 TSDF 값을 양수(+)로 갱신한다. 이는 관측되지 않은 영역 뒤에 잘못 생성될 수 있는 허상 표면(hallucinated surfaces)을 효과적으로 제거(carving)하는 역할을 한다. 이 개념은 여러 시점의 이미지로부터 물체의 3차원 형태를 조각해내는 고전적인 컴퓨터 비전 기법인 'Space Carving'에서 유래했다.44

### 2.3. OpenVDB 활용 전략: 희소성을 통한 메모리 및 연산 효율 극대화

VDBFusion의 진정한 혁신은 TSDF 통합 알고리즘 자체가 아니라, 이 알고리즘을 OpenVDB라는 고도로 최적화된 희소 데이터 구조 위에서 구현했다는 점에 있다. OpenVDB를 활용함으로써 VDBFusion은 이전의 TSDF 기반 시스템들이 가졌던 근본적인 메모리 및 확장성 한계를 극복할 수 있었다.

가장 중요한 전략은 OpenVDB의 희소성을 이용하여 표면 주변의 좁은 영역(narrow band)에만 복셀을 할당하고, 나머지 대부분의 비어 있거나 관측되지 않은 공간은 메모리에서 완전히 배제하는 것이다.4 이는 재구성하는 공간의 크기가 커지더라도 메모리 사용량이 표면의 면적에 비례하여 완만하게 증가하도록 만든다.

이러한 접근 방식 덕분에 VDBFusion은 놀라운 연산 효율성을 달성했다. 논문에 따르면, 64채널 LiDAR 센서로부터 초당 20프레임으로 수신되는 방대한 양의 포인트 클라우드 데이터를 단일 CPU 코어만으로 실시간 처리하는 것이 가능하다.4 이는 고가의 GPU 가속에 의존했던 기존의 많은 실시간 3D 재구성 시스템들과 VDBFusion을 차별화하는 중요한 지점이다.47

또한, OpenVDB의 빠른 임의 접근 및 순차 접근 성능은 실시간으로 들어오는 센서 데이터를 지연 없이 그리드에 통합하는 데 필수적인 기반을 제공한다. 데이터 통합이 완료된 후에는, 최종적으로 저장된 희소 TSDF 그리드로부터 Marching Cubes 알고리즘을 적용하여 효율적으로 삼각형 메시(triangular mesh)를 추출하고, 이를 시각화하거나 추가적인 로봇 경로 계획 등에 활용할 수 있다.5

### 2.4. 주요 응용 분야 및 구현

VDBFusion의 주요 응용 분야는 OpenVDB의 그것과는 명확히 구분된다. VDBFusion은 실시간성과 대규모 공간 처리 능력이 동시에 요구되는 로보틱스 분야에 집중되어 있다. 구체적인 응용 사례는 다음과 같다.

- **로봇 환경 매핑 및 SLAM:** 이동 로봇이 미지의 환경을 탐색하며 자신의 위치를 추정하고 동시에 지도를 작성하는 데 사용된다. VDBFusion이 생성한 3D 지도는 로봇의 경로 계획(path planning)과 장애물 회피(obstacle avoidance)에 직접적으로 활용될 수 있다.3
- **자율주행:** 자율주행 차량이 LiDAR나 카메라 센서를 이용해 주변 환경을 정밀하게 3D로 인식하고, 동적인 객체와 정적인 구조물을 구분하는 데 기반 기술로 사용될 수 있다.
- **증강현실 (AR):** AR 애플리케이션에서 현실 공간을 실시간으로 스캔하여 가상의 객체를 자연스럽게 배치하고 상호작용하게 하는 데 VDBFusion의 재구성 기술이 활용될 수 있다.50

VDBFusion은 학계와 산업계 개발자들의 접근성을 높이기 위해 잘 설계된 소프트웨어 패키지를 제공한다. 핵심 로직은 C++로 구현되어 높은 성능을 보장하며, 동시에 풍부한 기능을 갖춘 Python API를 제공하여 연구자들이나 개발자들이 복잡한 C++ 빌드 과정 없이도 NumPy 배열과 같은 표준 데이터 형식을 이용해 빠르게 프로토타이핑하고 기존 시스템에 통합할 수 있도록 했다. 특히 `pip install vdbfusion`이라는 단일 명령어로 설치가 가능하도록 배포하여 사용 편의성을 극대화했다.5

더 나아가, 로보틱스 개발의 표준 플랫폼인 로봇 운영체제(ROS)와의 손쉬운 통합을 위한 ROS 래퍼(wrapper)도 커뮤니티를 통해 제공되고 있다. 이를 통해 ROS의 메시지 통신 시스템을 이용하여 센서 데이터(예: `sensor_msgs/PointCloud2`)와 로봇의 위치 정보(예: `nav_msgs/Odometry`)를 직접 입력받아 3D 지도를 생성하는 노드를 쉽게 구성할 수 있다.5

## 제3장: 심층 비교 분석

### 3.1. 목적 및 설계 철학: 범용 라이브러리 vs. 특정 목적 솔루션

OpenVDB와 VDBFusion의 가장 근본적인 차이는 그들의 목적과 설계 철학에서 비롯된다. 이 둘의 관계는 '도구'와 '도구를 사용해 만든 제품'에 비유할 수 있다.

**OpenVDB**는 특정 응용 분야에 국한되지 않는 **범용 라이브러리(General-Purpose Library)**를 지향한다. 그 핵심 목표는 희소 볼류메트릭 데이터를 다루는 모든 분야-VFX, 과학 시뮬레이션, 의료 영상, 로보틱스 등-에서 공통적으로 사용할 수 있는 효율적인 데이터 구조와 기본적인 연산 도구의 집합을 제공하는 것이다.1 OpenVDB는 "어떻게 희소 볼륨을 효율적으로 저장하고 접근할 것인가?"라는 근본적인 질문에 대한 해답을 제공하며, 개발자들이 이 기반 위에서 각자의 특정 문제를 해결할 수 있도록 지원한다.

반면, **VDBFusion**은 로보틱스 분야의 **특정 목적 솔루션(Specific-Purpose Solution)**이다. 그 목표는 '센서 데이터를 이용해 대규모 3D 환경을 실시간으로 재구성'하는 명확하고 구체적인 문제를 해결하는 것이다.4 VDBFusion은 OpenVDB라는 범용 도구를 창의적으로 활용하여, TSDF 통합이라는 특정 알고리즘을 구현하고 최적화한 성공적인 사례이다. 즉, VDBFusion의 설계는 범용성이 아닌 특정 문제 해결의 효율성과 효과성에 초점이 맞춰져 있다.

**<표 1> OpenVDB와 VDBFusion 핵심 특징 비교**

| 항목             | OpenVDB                                           | VDBFusion                               |
| ---------------- | ------------------------------------------------- | --------------------------------------- |
| **유형**         | C++ 라이브러리 및 도구 모음                       | C++/Python 라이브러리 (알고리즘 구현체) |
| **주요 목표**    | 범용 희소 볼류메트릭 데이터의 효율적 저장 및 조작 | 실시간 대규모 3D 표면 재구성            |
| **핵심 기술**    | 계층적 B+트리 기반 희소 데이터 구조               | OpenVDB 위에서의 TSDF 통합 알고리즘     |
| **데이터 표현**  | 범용 볼류메트릭 그리드 (밀도, SDF, 벡터 필드 등)  | 희소 TSDF(절단 부호 거리 함수) 볼륨     |
| **주 응용 분야** | VFX, 렌더링, 물리 시뮬레이션, 의료 영상           | 로보틱스, SLAM, 자율주행, 3D 매핑       |
| **생태계**       | ASWF, DCC 툴 (Houdini, Blender), NVIDIA (GPU, AI) | ROS, 로보틱스 연구 커뮤니티, Open3D     |

### 3.2. 성능 및 효율성: VDB 기반 구조와 옥트리 기반 구조의 비교

VDBFusion의 성능을 객관적으로 평가하기 위해서는 로보틱스 매핑 분야에서 널리 사용되는 다른 희소 데이터 구조, 특히 옥트리(Octree) 기반의 OctoMap과 비교하는 것이 유용하다.55

- **접근 속도:** OpenVDB의 고정 깊이 트리 구조는 특정 복셀에 대한 접근 시간 복잡도가 거의 상수 시간(`$O(1)$`)에 가깝다. 반면, 옥트리는 데이터 분포에 따라 깊이가 달라지므로 로그 시간(`$O(\log N)$`)의 복잡도를 가진다. 이는 수많은 복셀에 대한 무작위 접근이 빈번하게 발생하는 실시간 데이터 통합 과정에서 OpenVDB 기반의 VDBFusion이 더 예측 가능하고 안정적인 성능을 보일 수 있음을 시사한다.4
- **메모리 효율성:** 두 구조 모두 희소 데이터를 효율적으로 저장하지만, 데이터의 공간적 분포 특성에 따라 상대적인 효율성이 달라질 수 있다. 옥트리는 매우 희소하거나 밀집도가 불균일한 데이터에 대해 적응적으로 노드를 분할하므로 메모리 효율이 높을 수 있다. 반면 OpenVDB는 리프 노드 단위의 데이터 블록과 내부적인 압축 기법을 통해 높은 메모리 효율을 달성한다. 실제 벤치마크에서 VDBFusion은 유사한 작업을 수행하는 다른 구현체들보다 더 적은 메모리를 사용하는 것으로 보고되기도 했다.55
- **병렬성:** OpenVDB의 데이터 구조는 독립적인 블록 단위로 연산을 수행하기 용이하여 병렬 처리에 유리한 측면이 있다. 이는 멀티코어 CPU나 GPU를 활용하여 성능을 극대화하는 데 도움이 된다. 반면, 옥트리는 트리 구조의 상하위 노드 간 의존성 때문에 병렬적인 업데이트 및 수정이 더 복잡할 수 있다.55

**<표 2> 3D 매핑 데이터 구조 성능 비교**

| 항목                    | VDBFusion (OpenVDB 기반)      | OctoMap (Octree 기반)              | KinectFusion (Dense Grid 기반) |
| ----------------------- | ----------------------------- | ---------------------------------- | ------------------------------ |
| **데이터 구조**         | 고정 깊이 트리 (B+트리 유사)  | 가변 깊이 트리                     | 다차원 배열                    |
| **접근 시간 복잡도**    | 상수 시간 (`$O(1)$`)          | 로그 시간 (`$O(\log N)$`)          | 상수 시간 (`$O(1)$`)           |
| **업데이트(삽입) 속도** | 높음 (빠른 순회 및 접근)      | 상대적으로 낮음 (재귀적 순회)      | 매우 높음 (직접 인덱싱)        |
| **메모리 사용량**       | 희소성에 따라 가변적 (효율적) | 희소성에 따라 가변적 (매우 효율적) | 매우 높음 (비효율적)           |
| **병렬 처리 용이성**    | 높음                          | 상대적으로 낮음                    | 매우 높음                      |

### 3.3. 적용 분야의 분화와 융합: VFX와 로보틱스 생태계

OpenVDB와 VDBFusion은 각자의 설계 철학에 따라 서로 다른 산업 생태계에서 발전해왔다.

- **분화:** OpenVDB는 Houdini, Blender, Cinema 4D와 같은 DCC 툴과 RenderMan, Arnold 같은 렌더러에 깊숙이 통합되어 VFX 및 애니메이션 파이프라인의 핵심 구성 요소로 자리 잡았다.9 아티스트와 TD(Technical Director)들은 OpenVDB를 사용하여 정교한 시각 효과를 제작하고 데이터를 교환한다. 반면, VDBFusion은 로봇 운영체제(ROS) 생태계를 중심으로 로봇 공학자들과 컴퓨터 비전 연구자들에게 채택되어, 실제 로봇 시스템의 인식 및 자율 항법 능력 향상에 기여하고 있다.54
- **융합:** 최근 디지털 트윈과 산업 메타버스의 부상은 이 두 생태계의 융합을 촉진하고 있다. NVIDIA의 Omniverse 플랫폼이 대표적인 예이다. Omniverse는 Pixar의 OpenUSD(Universal Scene Description)를 기반으로 다양한 소스로부터 생성된 3D 데이터를 통합하고 협업할 수 있는 환경을 제공한다. 여기에 NVIDIA는 fVDB NIMs(NVIDIA Inference Microservices)와 같은 기술을 도입하여, OpenVDB 기반의 물리 시뮬레이션 및 AI 연산 결과를 OpenUSD 씬(scene)에 통합할 수 있도록 지원한다.35 이는 VFX 분야에서 제작된 고품질 볼류메트릭 에셋이 로보틱스 시뮬레이션 환경에서 물리적으로 상호작용하거나, 로봇이 VDBFusion으로 스캔한 실제 환경 데이터가 가상 프로덕션의 배경으로 활용되는 등, 두 세계가 하나의 플랫폼에서 원활하게 결합하는 미래를 제시한다.

### 3.4. 한계점 및 실패 사례 분석

모든 기술과 마찬가지로 OpenVDB와 VDBFusion 역시 한계점을 가지고 있다.

- **VDBFusion의 한계:**
  - **입력 데이터 품질 의존성:** VDBFusion의 재구성 품질은 입력되는 포인트 클라우드의 밀도와 정확성에 크게 의존한다. 특히 LiDAR 데이터가 매우 희소(sparse)할 경우, 표면 사이에 구멍이 생기거나 기하학적 구조가 불완전하게 재구성될 수 있다.47
  - **TSDF의 본질적 한계:** TSDF는 표면까지의 거리를 근사적으로 표현하며, 참된 유클리드 거리 필드(ESDF, Euclidean Signed Distance Field)를 직접 제공하지 않는다. ESDF는 장애물로부터의 최단 거리를 제공하므로 로봇의 경로 계획(path planning)에 더 직접적이고 유용하지만, TSDF로부터 ESDF를 계산하기 위해서는 추가적인 변환 과정이 필요하다.51
  - **동적 환경 처리의 어려움:** 기본적인 VDBFusion 알고리즘은 정적인 환경을 가정한다. 따라서 사람이나 다른 로봇과 같이 움직이는 객체가 있는 환경에서는, 이들의 움직임이 지도에 잔상으로 남아 재구성에 오류를 유발할 수 있다.60
- **OpenVDB 생태계의 한계:**
  - **NeuralVDB의 손실 압축:** NeuralVDB는 높은 압축률을 제공하지만, 이는 신경망을 이용한 손실 압축(lossy compression) 방식이다. 따라서 원본 데이터와의 완벽한 일치를 보장하지 않으며, 미세한 디테일이 손실될 수 있다. 또한, 데이터를 신경망으로 인코딩(학습)하고 디코딩(추론)하는 과정에서 추가적인 연산 오버헤드가 발생한다.22
  - **NanoVDB의 정적 토폴로지:** NanoVDB는 GPU에서의 빠른 렌더링을 위해 정적 토폴로지(static topology)에 최적화되어 있다. 이는 한 번 생성된 볼륨의 구조가 변하지 않는 경우에 가장 효율적이며, 시뮬레이션 도중 복셀이 동적으로 생성되거나 삭제되는 복잡한 유체 시뮬레이션과 같은 작업에는 적합하지 않을 수 있다.12
  - **구현 및 설치의 복잡성:** VDBFusion을 포함한 OpenVDB 기반 프로젝트들은 다양한 외부 라이브러리(예: Boost, TBB)에 대한 의존성을 가진다. 이로 인해 사용자의 시스템 환경(OS, 컴파일러, Python 버전 등)에 따라 빌드 및 설치 과정에서 의존성 충돌이나 버전 불일치 문제가 발생할 수 있다.54

## 제4장: 미래 전망 및 결론

### 4.1. VDB 기반 기술의 발전 방향: VDB-GPDF와 AI 기반 월드 모델

VDBFusion이 보여준 가능성은 VDB 데이터 구조를 활용한 로보틱스 매핑 기술의 발전을 촉진했다. 그 자연스러운 진화의 방향은 VDBFusion의 한계, 특히 TSDF의 근본적인 제약을 극복하는 데 맞춰져 있다. 그 대표적인 예가 VDB-GPDF(Gaussian Process Distance Field)이다.49

VDB-GPDF는 TSDF 대신 가우시안 프로세스(GP)라는 확률론적 모델을 사용하여 공간의 거리 필드를 표현한다. 이 접근법은 다음과 같은 중요한 개선점을 제공한다.

1. **정확한 유클리드 거리:** TSDF가 표면까지의 투영 거리를 근사하는 반면, VDB-GPDF는 보다 정확한 유클리드 거리 필드(ESDF)를 직접 모델링하여 로봇의 경로 계획에 더 유용한 정보를 제공한다.50
2. **불확실성 모델링:** GP의 핵심적인 장점은 예측에 대한 불확실성(uncertainty)을 정량적으로 표현할 수 있다는 것이다. VDB-GPDF는 이를 활용하여 각 거리 측정값의 신뢰도를 모델링하고, 새로운 데이터를 통합할 때 이 불확실성을 고려하는 '확률적 융합(probabilistic fusion)'을 수행한다. 이는 노이즈가 많거나 불완전한 센서 데이터에 대해 훨씬 더 강건한(robust) 맵을 생성하게 한다.49
3. **연속성 및 생성 모델:** VDB-GPDF는 불연속적인 복셀 그리드를 넘어 연속적인(continuous) 거리 필드를 표현한다. 이는 GP가 생성 모델(generative model)의 일종이기 때문에 가능하며, 데이터가 관측되지 않은 영역에 대해서도 합리적인 거리 값을 추론(inference)할 수 있는 능력을 부여한다.50

더 나아가, fVDB와 같은 프레임워크의 등장은 VDB 기반 기술이 단순한 3D 재구성을 넘어 AI 기반 월드 모델(world model)의 핵심 구성 요소로 발전하고 있음을 시사한다. 월드 모델은 자율 에이전트가 주변 환경을 내재적으로 이해하고, 행동의 결과를 시뮬레이션하며, 미래를 예측하여 최적의 계획을 수립하는 데 사용되는 내부적인 세계관이다.67 미래의 로봇은 VDBFusion이나 VDB-GPDF와 같은 기술로 현실 세계를 3D로 재구성하고, fVDB와 같은 기술을 이용해 그 위에서 물리 법칙을 시뮬레이션하며 행동을 학습하는 통합된 시스템으로 발전할 가능성이 크다.

### 4.2. 대안 기술과의 경쟁 및 공존: 3D 가우시안 스플래팅의 부상

VDB 기반 기술이 진화하는 동안, 3D 장면 표현 분야에서는 새로운 패러다임이 등장했다. 2023년 SIGGRAPH에서 발표된 3D 가우시안 스플래팅(3DGS)은 NeRF(Neural Radiance Fields)가 보여준 높은 시각적 품질과 실시간 렌더링 속도를 동시에 달성하며 학계와 산업계에 큰 충격을 주었다.71

VDB와 3DGS는 근본적으로 다른 철학을 가진다. VDB가 공간을 복셀 그리드로 이산화하고 각 복셀에 값을 할당하여 표면을 '암시적(implicit)'으로 표현하는 반면, 3DGS는 수백만 개의 위치, 크기, 방향, 색상, 불투명도를 가진 3D 가우시안이라는 '명시적(explicit)' 원시 객체(primitive)의 집합으로 장면을 직접 표현한다.78

**<표 3> 3D 장면 표현 기술 패러다임 비교**

| 항목                       | VDB (OpenVDB)                       | 3DGS (Gaussian Splatting)        | Mesh (전통적)               |
| -------------------------- | ----------------------------------- | -------------------------------- | --------------------------- |
| **기본 단위**              | 복셀 (Voxel)                        | 3D 가우시안 (Gaussian)           | 삼각형 (Triangle)           |
| **표현 방식**              | 암시적 (Implicit) 볼류메트릭 그리드 | 명시적 (Explicit) 원시 객체 집합 | 명시적 (Explicit) 표면      |
| **렌더링 방식**            | 레이마칭 / 볼륨 렌더링              | 스플래팅 / 래스터화              | 래스터화 / 레이 트레이싱    |
| **물리 시뮬레이션 적합성** | 높음 (볼륨 속성 저장)               | 낮음 (표면 표현에 가까움)        | 중간 (표면 기반 시뮬레이션) |
| **실시간 렌더링 성능**     | 중간~높음 (NanoVDB 활용 시)         | 매우 높음                        | 매우 높음                   |
| **메모리 사용량**          | 희소성에 따라 가변적                | 높음 (최적화 필요)               | 낮음                        |

궁극적으로 미래의 3D 월드 모델은 단일 표현 방식에 의존하기보다는, 목적에 따라 VDB와 3DGS, 그리고 전통적인 메시와 같은 여러 표현이 유기적으로 결합되고 상호 변환되는 '하이브리드' 또는 '다중 표현(multi-representation)' 시스템이 될 것이다.

### 4.3. 산업계 통합 동향: 디지털 트윈과 메타버스

VDB와 3DGS와 같은 첨단 3D 기술은 산업 전반에 걸쳐 디지털 전환을 가속화하고 있다.

- **디지털 트윈:** OpenVDB와 그 파생 기술들, 특히 fVDB는 공장, 도시, 발전소와 같은 현실 세계의 복잡한 시스템을 가상 공간에 정밀하게 복제하는 산업용 디지털 트윈 구축의 핵심 기술로 부상하고 있다. VDB의 효율적인 데이터 구조는 방대한 시뮬레이션 데이터를 처리하는 데 이상적이며, 이를 통해 가상 환경에서 다양한 시나리오를 테스트하고, 시스템을 최적화하며, 운영자를 훈련하는 데 활용된다.10
- **메타버스 및 AEC:** 3DGS는 현실 세계를 빠르고 사실적으로 캡처하여 가상 환경 및 메타버스 에셋을 제작하는 데 매우 강력한 도구로 주목받고 있다.91 스마트폰 카메라로 촬영한 영상만으로도 고품질의 3D 모델을 생성할 수 있어, 3D 콘텐츠 제작의 장벽을 크게 낮추고 있다. 건축, 엔지니어링, 건설(AEC) 분야에서도 3DGS는 건설 현장의 진행 상황을 시각화하거나, 완공 후의 모습을 가상으로 체험하는 등 다양한 용도로 활용되기 시작했다.95

이러한 산업 동향은 VDB와 3DGS가 각자의 강점을 바탕으로 서로 다른 시장을 개척하면서도, 결국에는 Omniverse와 같은 통합 플랫폼을 통해 상호 보완적으로 사용될 것임을 시사한다.

### 4.4. 종합 결론

본 보고서는 OpenVDB와 VDBFusion을 기반 기술과 응용 기술의 관점에서 심층적으로 비교 분석했다.

**OpenVDB**는 VFX 산업의 특수한 문제를 해결하기 위해 탄생했지만, 그 혁신적인 희소 데이터 구조는 시뮬레이션, 로보틱스, AI 등 다양한 분야로 확장되어 3D 공간 데이터 처리의 패러다임을 바꾼 핵심 기반 기술이다. 이는 잘 설계된 범용 기술이 특정 분야를 넘어 얼마나 큰 파급력을 가질 수 있는지를 보여주는 대표적인 사례이다.

**VDBFusion**은 OpenVDB라는 강력한 도구를 로보틱스 분야의 오랜 난제였던 대규모 실시간 3D 재구성에 성공적으로 적용한 탁월한 응용 사례이다. 이는 새로운 알고리즘의 발명만이 혁신이 아니라, 기존 기술들의 창의적인 융합을 통해서도 중요한 문제 해결과 기술적 진보를 이룰 수 있음을 명확히 보여준다.

OpenVDB와 VDBFusion에서 시작된 기술적 흐름은 이제 AI와 결합하여 더욱 정교하고 지능적인 월드 모델을 구축하는 방향으로 나아가고 있다. VDB는 더 이상 단순한 데이터 포맷이 아니라, 물리 세계를 이해하고 시뮬레이션하며 예측하는 '공간 AI'의 핵심 인프라로 자리매김하고 있다. 이는 미래의 자율 시스템, 디지털 트윈, 그리고 메타버스 기술의 발전에 지속적으로 기여하며 그 가치를 더욱 높여갈 것이다.

#### **Works cited**

1. OpenVDB - Wikipedia, accessed August 5, 2025, https://en.wikipedia.org/wiki/OpenVDB
2. VDB가 뭔데? : r/vfx - Reddit, accessed August 5, 2025, https://www.reddit.com/r/vfx/comments/l7uhaq/what_is_vdb/?tl=ko
3. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - PubMed, accessed August 5, 2025, https://pubmed.ncbi.nlm.nih.gov/35162040/
4. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/358523933_VDBFusion_Flexible_and_Efficient_TSDF_Integration_of_Range_Sensor_Data
5. PRBonn/vdbfusion: C++/Python Sparse Volumetric TSDF Fusion - GitHub, accessed August 5, 2025, https://github.com/PRBonn/vdbfusion
6. Plug in - UV Layout 설치법 - CG study blog. - 티스토리, accessed August 5, 2025, https://houdinifpahs2223.tistory.com/999
7. www.numberanalytics.com, accessed August 5, 2025, [https://www.numberanalytics.com/blog/vdb-in-visual-effects-deep-dive#:~:text=At%20its%20core%2C%20OpenVDB%20is,clouds%2C%20water%2C%20and%20fire.](https://www.numberanalytics.com/blog/vdb-in-visual-effects-deep-dive#:~:text=At its core%2C OpenVDB is,clouds%2C water%2C and fire.)
8. What is OpenVDB and why should you care? [thinkingParticles Documentation], accessed August 5, 2025, https://cebas.com/manual/LifeLicenser/doku.php?id=reference_guide:thinkingparticles_nodes:operator_nodes:openvdb:what_is_openvdb
9. ZibraVDB – a new solution, bringing groundbreaking OpenVDB format to game development - Zibra AI, accessed August 5, 2025, https://www.zibra.ai/blog-posts/zibravdb-a-new-solution-by-zibraai
10. About OpenVDB, accessed August 5, 2025, https://www.openvdb.org/about/
11. 드림웍스 '가디언즈' 제작기술 오픈소스로 공개 - 지디넷코리아, accessed August 5, 2025, https://zdnet.co.kr/view/?no=20121123183206
12. OpenVDB, accessed August 5, 2025, https://www.openvdb.org/
13. VDB-EDT: An Efficient Euclidean Distance Transform ... - alphaXiv, accessed August 5, 2025, https://www.alphaxiv.org/ko/overview/2105.04419v1
14. VDB 복셀, accessed August 5, 2025, https://learn.foundry.com/ko/modo/content/help/pages/rendering/open_vdb.html
15. Download - OpenVDB, accessed August 5, 2025, https://www.openvdb.org/download/
16. Houdini VDB to Unreal Engine 5.5 (Preview) | visualnoobs - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=Cm9q96hrAMw
17. OpenVDB: Houdini to Blender export | Forums - SideFX, accessed August 5, 2025, https://www.sidefx.com/forum/topic/43547/
18. This neat free plugin lets you import OpenVDBs into UE5 - CG Channel, accessed August 5, 2025, https://www.cgchannel.com/2022/07/free-plugin-unrealvdb-lets-you-import-openvdbs-into-ue5/
19. VDB into Blender, or render in simulation software and composite? : r/vfx - Reddit, accessed August 5, 2025, https://www.reddit.com/r/vfx/comments/129roty/vdb_into_blender_or_render_in_simulation_software/
20. 과학과 산업의 판도를 바꿀 NVIDIA NeuralVDB 발표, accessed August 5, 2025, https://blogs.nvidia.co.kr/blog/neuralvdb-ai/?nv_excludes=9203,9206
21. NanoVDB - NVIDIA Developer, accessed August 5, 2025, https://developer.nvidia.com/nanovdb
22. NVIDIA NeuralVDB - NVIDIA Developer, accessed August 5, 2025, https://developer.nvidia.com/rendering-technologies/neuralvdb
23. Optimizing Large-Scale Sparse Volumetric Data with NVIDIA NeuralVDB Early Access, accessed August 5, 2025, https://developer.nvidia.com/blog/optimizing-large-scale-sparse-volumetric-data-with-nvidia-neuralvdb-early-access/
24. NVIDIA Introduces NeuralVDB at SIGGRAPH - HPCwire, accessed August 5, 2025, https://www.hpcwire.com/off-the-wire/nvidia-introduces-neuralvdb-at-siggraph/
25. NeuralVDB: AI-Powered Version of OpenVDB Revealed - 80 Level, accessed August 5, 2025, https://80.lv/articles/neuralvdb-ai-powered-version-of-openvdb-revealed
26. NVIDIA NeuralVDB Brings AI and GPU Optimization to OpenVDB - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=uAs8X5es1DE
27. NeuralVDB: High-Resolution Sparse Volume Representation Using Hierarchical Neural Networks | SC2022 2022 | NVIDIA On-Demand, accessed August 5, 2025, https://www.nvidia.com/en-us/on-demand/session/SC2022-T-11/
28. [2208.04448] NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks - arXiv, accessed August 5, 2025, https://arxiv.org/abs/2208.04448
29. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks - Consensus, accessed August 5, 2025, https://consensus.app/papers/neuralvdb-highresolution-sparse-volume-representation-lee-museth/7eae0cfabe3c544da973305e7d559581/
30. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/362591437_NeuralVDB_High-resolution_Sparse_Volume_Representation_using_Hierarchical_Neural_Networks
31. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks | Request PDF - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/377642706_NeuralVDB_High-resolution_Sparse_Volume_Representation_using_Hierarchical_Neural_Networks
32. Neural-network-based Volume Compression - ERK, accessed August 5, 2025, https://erk.fe.uni-lj.si/2024/papers/kristan(neural_network_based_volume).pdf
33. NeuralVDB: High-resolution Sparse Volume Representation using Hierarchical Neural Networks | NVIDIA - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=IfgeduMv_ns
34. 대규모 디지털 모델 구축 위한 'fVDB', 현실을 재구성하다 - NVIDIA Blog Korea, accessed August 5, 2025, https://blogs.nvidia.co.kr/blog/fvdb-bigger-digital-models/
35. fVDB Deep Learning Framework - NVIDIA Developer, accessed August 5, 2025, https://developer.nvidia.com/fvdb
36. Reality Reimagined: NVIDIA Introduces fVDB to Build Bigger Digital Models of the World, accessed August 5, 2025, https://blogs.nvidia.com/blog/fvdb-bigger-digital-models/
37. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - MDPI, accessed August 5, 2025, https://www.mdpi.com/1424-8220/22/3/1296
38. KinectFusion: real-time dynamic 3D surface reconstruction and interaction - AIT Lab, accessed August 5, 2025, https://ait.ethz.ch/kinectfusion
39. KinectFusion: Real-Time Dense Surface Mapping and Tracking - Microsoft, accessed August 5, 2025, https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/ismar2011.pdf
40. (PDF) KinectFusion: Real-time dynamic 3D surface reconstruction and interaction, accessed August 5, 2025, https://www.researchgate.net/publication/220721971_KinectFusion_Real-time_dynamic_3D_surface_reconstruction_and_interaction
41. KinectFusion: Real-Time Dense Surface Mapping and Tracking - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/221221368_KinectFusion_Real-Time_Dense_Surface_Mapping_and_Tracking
42. KinectFusion: Real-Time Dense Surface Mapping and Tracking - Azmarie Wang, accessed August 5, 2025, https://azmarie.github.io/paper-review/KinectFusion/
43. Real-time 3D Reconstruction at Scale using Voxel Hashing - Matthias Niessner, accessed August 5, 2025, https://niessnerlab.org/papers/2013/4hashing/niessner2013hashing.pdf
44. A Theory of Shape by Space Carving - Computer Science, accessed August 5, 2025, https://www.cs.toronto.edu/~kyros/pubs/00.ijcv.carve.pdf
45. Space Carving - Computerphile - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=cGs90KF4oTc
46. 3QFP: Efficient neural implicit surface reconstruction using Tri-Quadtrees and Fourier feature Positional encoding - arXiv, accessed August 5, 2025, https://arxiv.org/html/2401.07164v1
47. Qualitative comparison between Octomap and VDBFusion. While Octomap... | Download Scientific Diagram - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/Qualitative-comparison-between-Octomap-and-VDBFusion-While-Octomap-clearly-models-the_fig10_358523933
48. High-level overview of VDBFusion. Our system only takes as input point... - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/figure/High-level-overview-of-VDBFusion-Our-system-only-takes-as-input-point-clouds-Pi-with_fig3_358523933
49. [논문 리뷰] VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure, accessed August 5, 2025, https://www.themoonlight.io/ko/review/vdb-gpdf-online-gaussian-process-distance-field-with-vdb-structure
50. (PDF) VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/382271448_VDB-GPDF_Online_Gaussian_Process_Distance_Field_with_VDB_Structure
51. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 5, 2025, https://arxiv.org/html/2407.09649v1
52. The Fusion of Robotics, AI and AR/VR: A 2024 Revolution in Manufacturing, accessed August 5, 2025, https://www.automation.com/en-us/articles/march-2024/fusion-robotics-ai-ar-vr-2024-manufacturing
53. vdbfusion - PyPI, accessed August 5, 2025, https://pypi.org/project/vdbfusion/0.1.2/
54. Kin-Zhang/vdbfusion_mapping: vdbfusion ros version but with refactor compared to official one and work great also. - GitHub, accessed August 5, 2025, https://github.com/Kin-Zhang/vdbfusion_mapping
55. RGBTSDF: An Efficient and Simple Method for Color Truncated Signed Distance Field (TSDF) Volume Fusion Based on RGB-D Images - MDPI, accessed August 5, 2025, https://www.mdpi.com/2072-4292/16/17/3188
56. Benchmarking Occupancy Mapping libraries - Nicolò Valigi, accessed August 5, 2025, https://nicolovaligi.com/articles/occupancy-mapping-benchmarks/
57. Outdoor Navigation in ROS: Current State & Future Direction - Open Robotics Discourse, accessed August 5, 2025, https://discourse.ros.org/t/outdoor-navigation-in-ros-current-state-future-direction/33599
58. Rapidly Create Real-Time Physics Digital Twins with NVIDIA Omniverse Blueprints, accessed August 5, 2025, https://developer.nvidia.com/blog/rapidly-create-real-time-physics-digital-twins-with-nvidia-omniverse-blueprints/
59. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 5, 2025, https://arxiv.org/html/2407.09649v2
60. Real-Time Spatial Reasoning by Mobile Robots for Reconstruction and Navigation in Dynamic LiDAR Scenes - arXiv, accessed August 5, 2025, https://arxiv.org/html/2505.12267v1
61. Could not find a version that satisfies the requirement
62. Papers by Cedric Le Gentil - AIModels.fyi, accessed August 5, 2025, https://www.aimodels.fyi/author-profile/cedric-le-gentil-c3f57647-2692-4587-9a07-2e5164d8eb67
63. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure | Request PDF, accessed August 5, 2025, https://www.researchgate.net/publication/386122570_VDB-GPDF_Online_Gaussian_Process_Distance_Field_with_VDB_Structure
64. UTS-RI/VDB_GPDF - GitHub, accessed August 5, 2025, https://github.com/UTS-RI/VDB_GPDF
65. [2407.09649] VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure, accessed August 5, 2025, https://arxiv.org/abs/2407.09649
66. PINGS: Gaussian Splatting Meets Distance Fields within a Point-Based Implicit Neural Map, accessed August 5, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/pan2025rss.pdf
67. Revolutionizing Robotics with AI-Driven 3D Reconstruction - Number Analytics, accessed August 5, 2025, https://www.numberanalytics.com/blog/revolutionizing-robotics-with-ai-driven-3d-reconstruction
68. R²D²: Building AI-based 3D Robot Perception and Mapping with NVIDIA Research, accessed August 5, 2025, https://developer.nvidia.com/blog/r2d2-building-ai-based-3d-robot-perception-and-mapping-with-nvidia-research/
69. First Open-Source AI for Full 3D World Generation: Hunyuan3D 1.0 - Parametric Architecture, accessed August 5, 2025, https://parametric-architecture.com/3d-world-generation-hunyuan3d-1-0/
70. Tech leaders eye world models as link to smarter AI - IBM, accessed August 5, 2025, https://www.ibm.com/think/news/world-models-smarter-ai
71. Gaussian Splatting: From Research to Reality | by Paula Ramos, PhD. | Voxel51 | Medium, accessed August 5, 2025, https://medium.com/voxel51/gaussian-splatting-from-research-to-reality-54bac4242e6d
72. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - Inria, accessed August 5, 2025, https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/
73. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - Semantic Scholar, accessed August 5, 2025, https://www.semanticscholar.org/paper/3D-Gaussian-Splatting-for-Real-Time-Radiance-Field-Kerbl-Kopanas/2cc1d857e86d5152ba7fe6a8355c2a0150cc280a
74. 3D Gaussian Splatting for Real Time Radiance Field Rendering Using Insta360 Camera: - CORE, accessed August 5, 2025, https://core.ac.uk/download/617974924.pdf
75. 3D Gaussian Splatting for Real-Time Radiance Field Rendering - ResearchGate, accessed August 5, 2025, https://www.researchgate.net/publication/372989904_3D_Gaussian_Splatting_for_Real-Time_Radiance_Field_Rendering
76. Original reference implementation of "3D Gaussian Splatting for Real-Time Radiance Field Rendering" - GitHub, accessed August 5, 2025, https://github.com/graphdeco-inria/gaussian-splatting
77. 3D Gaussian Splatting for Real-Time Radiance Field Rendering | Qiang Zhang, accessed August 5, 2025, https://zhangtemplar.github.io/3d-gaussian-splatting/
78. LI-GS: Gaussian Splatting with LiDAR Incorporated for Accurate Large-Scale Reconstruction, accessed August 5, 2025, https://arxiv.org/html/2409.12899v1
79. Gaussian Splatting vs NeRF Models - What's the Difference and Which is created by Lumalabs? : r/GaussianSplatting - Reddit, accessed August 5, 2025, https://www.reddit.com/r/GaussianSplatting/comments/1ckmboa/gaussian_splatting_vs_nerf_models_whats_the/
80. ELMGS: Enhancing Memory and Computation Scalability through Compression for 3D Gaussian Splatting - CVF Open Access, accessed August 5, 2025, https://openaccess.thecvf.com/content/WACV2025/papers/Ali_ELMGS_Enhancing_Memory_and_Computation_Scalability_through_Compression_for_3D_WACV_2025_paper.pdf
81. Novel view synthesis, NeRF vs Gaussian splatting : r/computervision - Reddit, accessed August 5, 2025, https://www.reddit.com/r/computervision/comments/1if2w7f/novel_view_synthesis_nerf_vs_gaussian_splatting/
82. HDRSplat: Gaussian Splatting for High Dynamic Range 3D Scene Reconstruction from Raw Images - BMVA Archive, accessed August 5, 2025, https://bmva-archive.org.uk/bmvc/2024/papers/Paper_22/paper.pdf
83. From NeRFs to Gaussian Splats, and Back - arXiv, accessed August 5, 2025, https://arxiv.org/html/2405.09717v2
84. Does 3D Gaussian Splatting Need Accurate Volumetric Rendering? - arXiv, accessed August 5, 2025, https://arxiv.org/html/2502.19318v1
85. WaterSplatting: Fast Underwater 3D Scene Reconstruction using Gaussian Splatting, accessed August 5, 2025, https://water-splatting.github.io/
86. Hybrid Mesh-Gaussian Representation for Efficient Indoor Scene Reconstruction - arXiv, accessed August 5, 2025, https://arxiv.org/html/2506.06988v1
87. Anttwo/SuGaR: [CVPR 2024] Official PyTorch implementation of SuGaR: Surface-Aligned Gaussian Splatting for Efficient 3D Mesh Reconstruction and High-Quality Mesh Rendering - GitHub, accessed August 5, 2025, https://github.com/Anttwo/SuGaR
88. HybridNeRF: Efficient Neural Rendering via Adaptive Volumetric Surfaces - arXiv, accessed August 5, 2025, https://arxiv.org/html/2312.03160v1
89. Volumetric Surfaces: Representing Fuzzy Geometries with Layered Meshes - arXiv, accessed August 5, 2025, https://arxiv.org/html/2409.02482v2
90. LinPrim: Linear Primitives for Differentiable Volumetric Rendering - arXiv, accessed August 5, 2025, https://arxiv.org/html/2501.16312v2
91. 3D Gaussian Splatting: Performant 3D Scene Reconstruction at Scale - AWS, accessed August 5, 2025, https://aws.amazon.com/blogs/spatial/3d-gaussian-splatting-performant-3d-scene-reconstruction-at-scale/
92. Creating Virtual Environments with 3D Gaussian Splatting: A Comparative Study - arXiv, accessed August 5, 2025, https://arxiv.org/abs/2501.09302
93. vid2scene - a free, end-to-end video to gaussian splat web platform : r/GaussianSplatting - Reddit, accessed August 5, 2025, https://www.reddit.com/r/GaussianSplatting/comments/1i7ixsh/vid2scene_a_free_endtoend_video_to_gaussian_splat/
94. DreamGaussian: Generative Gaussian Splatting for Efficient 3D Content Creation - arXiv, accessed August 5, 2025, https://arxiv.org/html/2309.16653v2
95. Is Gaussian Splatting Ready for Standardization? - Geo Week News, accessed August 5, 2025, https://www.geoweeknews.com/blogs/gaussian-splats-3d-rendering-aec-visualization-standardization-standards
96. Gaussian Splatting in Adobe Substance 3D Viewer - YouTube, accessed August 5, 2025, https://www.youtube.com/watch?v=7bVgNeCzDgU
97. 3D Gaussian Splatting aided Localization for Large and Complex Indoor-Environments, accessed August 5, 2025, https://arxiv.org/html/2502.13803v1
98. 3D Gaussian Splatting for Construction Sites - Computer Graphics Group – der TH Köln, accessed August 5, 2025, https://cg.web.th-koeln.de/3d-gaussian-splatting-for-construction-sites/
99. AI turns Gaussian Splats into Revit models - AEC Magazine, accessed August 5, 2025, https://aecmag.com/reality-capture-modelling/ai-tool-transforms-gaussian-splats-into-revit-models-lcc-for-revit/


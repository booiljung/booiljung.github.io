# nvBlox와 coVoxSLAM의 비교 고찰

## I. 서론

자율 이동 로봇(AMR), 무인 항공기(드론), 로봇 팔 등 현대 로보틱스 시스템의 자율성은 주변 환경을 3차원으로 정밀하게 인식하고 자신의 위치를 추정하는 능력에 깊이 의존한다.1 특히, 장애물의 위치뿐만 아니라 비어있는 공간까지 표현하는 고밀도의 체적 맵(volumetric map)은 로봇이 안전하게 경로를 계획하고, 환경과 효과적으로 상호작용하기 위한 필수적인 정보를 제공한다.1

그러나 이러한 고밀도 3D 맵을 실시간으로, 특히 NVIDIA Jetson과 같이 계산 자원이 제한된 임베디드 플랫폼에서 생성하고 유지하는 것은 상당한 기술적 과제이다.1 이 문제를 해결하기 위해, 대규모 병렬 처리가 가능한 GPU(Graphics Processing Unit)의 잠재력을 최대한 활용하려는 연구가 활발히 진행되고 있다.2

본 보고서는 GPU 가속 3D 매핑 분야를 선도하는 두 가지 대표적인 최신 기술, 즉 NVIDIA의 `nvBlox`와 아르헨티나 부에노스아이레스 대학 LRSE 연구실의 `coVoxSLAM`을 심층적으로 비교 분석한다. `nvBlox`는 NVIDIA Isaac ROS(Robot Operating System) 생태계의 핵심 구성 요소로, GPU 가속을 통해 실시간 3D 환경 재구성 및 경로 계획을 위한 거리 필드(distance field) 계산에 특화된 라이브러리이다.6 반면, `coVoxSLAM`은 위치 추정부터 지도 작성, 전역 최적화에 이르는 전체 SLAM(Simultaneous Localization and Mapping) 파이프라인을 GPU 상에서 처리하여, 대규모 환경에서도 전역적으로 일관된(globally consistent) 맵을 구축하는 것을 목표로 하는 통합 SLAM 시스템이다.2

본 보고서는 두 시스템의 근본적인 설계 철학, 아키텍처, 핵심 알고리즘의 구현 방식, 성능 벤치마크, 생태계 및 명백한 한계점까지 다각적으로 비교 분석한다. 이를 통해 로보틱스 개발자와 연구자가 자신의 응용 목적과 기술적 요구사항에 가장 부합하는 솔루션을 선택할 수 있도록 전문가 수준의 통찰과 구체적인 가이드라인을 제공하는 것을 목표로 한다.

## 1. II. 시스템 철학 및 목표 범위 비교: 라이브러리 대 통합 시스템

`nvBlox`와 `coVoxSLAM`의 가장 근본적인 차이는 그들의 정체성에서 비롯된다. `nvBlox`는 특정 기능을 수행하는 '라이브러리'이고, `coVoxSLAM`은 완전한 기능을 갖춘 '통합 시스템'이다. 이 차이는 두 기술의 설계 철학, 목표 범위, 그리고 궁극적으로 사용 방식에 결정적인 영향을 미친다.

### 1.1 `nvBlox`: 모듈화된 고성능 매핑 컴포넌트

`nvBlox`는 완전한 SLAM 시스템이 아니라, 3D 재구성 및 경로 계획용 비용 맵(cost map) 생성이라는 특정 작업에 고도로 최적화된 C++, Python, ROS2 인터페이스를 제공하는 라이브러리이다.6 그 핵심 기능은 RGB-D 카메라나 3D LiDAR와 같은 센서로부터 얻은 깊이 데이터와 **외부에서 제공된 센서의 위치 및 자세(pose)**를 입력받아, GPU의 병렬 처리 능력을 활용하여 TSDF(Truncated Signed Distance Function) 및 ESDF(Euclidean Signed Distance Function) 기반의 3D 맵을 실시간으로 생성하는 것이다.7

`nvBlox`의 주된 목표는 로보틱스 개발자들이 자신의 시스템에 고성능 3D 인식 및 매핑 기능을 손쉽게 통합할 수 있도록, 잘 정의된 API를 갖춘 모듈식 컴포넌트를 제공하는 것이다. 이는 NVIDIA Isaac ROS 생태계의 전반적인 설계 철학과 일맥상통한다.14 즉, 각 분야(SLAM, 매핑, 인식, 경로 계획)에 최적화된 개별 모듈(GEMs, Group of Emerging Technologies)을 제공하고, 개발자는 이를 레고 블록처럼 조합하여 자신의 애플리케이션을 구축한다.

이러한 모듈적 접근 방식은 필연적으로 기능적 범위를 제한한다. `nvBlox` 자체는 센서의 위치를 추정(localization)하거나, 시간이 지남에 따라 누적되는 주행 거리계(odometry) 오차를 보정하기 위한 루프 폐쇄(loop closure) 및 전역 최적화(global optimization) 기능을 포함하지 않는다.17 이러한 핵심적인 SLAM 기능은 `cuVSLAM`과 같은 외부 SLAM 시스템에 전적으로 의존해야 한다.14

### 1.2 `coVoxSLAM`: 전역 일관성을 지향하는 완전한 SLAM 시스템

`coVoxSLAM`은 `nvBlox`와 달리 위치 추정과 지도 작성을 동시에 수행하는 완전한(end-to-end) SLAM 시스템이다.2 이 시스템은 센서 데이터 입력부터 TSDF/ESDF 맵 생성(프론트엔드), 그리고 포즈 그래프 최적화 및 루프 폐쇄를 통한 전역 일관성 확보(백엔드)까지, SLAM의 모든 핵심 과정을 단일 시스템 내에서 통합하여 처리한다.2

`coVoxSLAM`의 핵심 연구 목표는 CPU와 GPU 간의 데이터 전송으로 인한 병목 현상을 최소화하고 GPU의 병렬 처리 능력을 극대화하기 위해, 프론트엔드와 백엔드 전체를 GPU 상에서 가속하는 새로운 아키텍처를 제시하고 그 성능을 입증하는 것이다. 이를 통해 대규모 환경에서도 실시간으로 전역적으로 일관된 맵을 생성하는 것을 궁극적인 목표로 삼는다.2 결과적으로 `coVoxSLAM`은 `nvBlox`가 외부 시스템에 의존해야 하는 위치 추정의 누적 오차 문제와 맵의 전역 일관성 문제를 시스템 내부에서 직접 해결한다.2

이러한 근본적인 정체성의 차이는 두 기술의 개발 배경과 지향점에서 비롯된다. `nvBlox`는 NVIDIA라는 거대 기술 기업이 자사의 로보틱스 플랫폼인 Isaac ROS의 핵심 '부품'으로 개발하고 배포하는 기술이다.7 따라서 안정성, 상세한 문서, `cuVSLAM`이나 `Nav2`와 같은 다른 Isaac 모듈과의 완벽한 호환성 및 통합 용이성에 중점을 둔다.14 이는 개발자가 즉시 현장에 적용할 수 있는 잘 다듬어진 '상용 제품'의 성격을 강하게 띤다.

반면, `coVoxSLAM`은 대학 연구실에서 발표한 학술 논문을 기반으로 하며, 기존 기술(`nvBlox` 포함)이 가진 한계, 즉 백엔드 최적화의 CPU 의존성을 극복하는 새로운 아키텍처(완전한 GPU 가속)를 제안하고 그 학문적, 기술적 우수성을 입증하는 데 초점을 맞춘다.2 이는 최첨단 기술을 탐구하는 '연구 프로토타입'의 성격이 짙다. 

`nvBlox`의 모듈성은 개발자에게 유연성을 제공하고 생태계를 확장하려는 NVIDIA의 플랫폼 전략에서 기인한 반면, `coVoxSLAM`의 통합성은 '세계 최초의 완전 GPU 가속 SLAM 시스템'이라는 연구 목표를 달성하기 위한 필연적인 선택이었다.

이러한 차이는 사용자의 기술 선택에 중대한 시사점을 제공한다. 상용 수준의 안정성, 풍부한 예제와 문서, 체계적인 기술 지원을 바탕으로 신속하게 애플리케이션을 개발하고자 하는 엔지니어에게는 `nvBlox`가 합리적인 선택이다. 반면, SLAM의 근본적인 성능 한계를 돌파하려는 연구자나, 특정 응용을 위해 시스템 내부까지 깊숙이 수정해야 하는 전문가는 `coVoxSLAM`의 선도적인 아키텍처와 공개된 소스 코드에서 더 큰 가치를 발견할 것이다.9 결국 두 기술의 비교는 단순한 기능의 우열을 가리는 것을 넘어, '안정적인 개발 플랫폼'과 '첨단 연구 플랫폼' 사이의 선택 문제로 귀결된다.

------

**표 1: 핵심 특징 및 철학 비교**

| 항목                         | nvBlox                                                      | coVoxSLAM                                                    |
| ---------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **시스템 유형**              | 3D 재구성 및 매핑 **라이브러리** 6                          | 완전한(End-to-End) **SLAM 시스템** 2                         |
| **주요 목표**                | 로보틱스 응용을 위한 실시간, 고성능 3D 맵 및 비용 맵 생성 7 | 대규모 환경을 위한 전역적으로 일관된(globally consistent) 맵 실시간 구축 2 |
| **위치 추정 (Localization)** | **외부** SLAM 시스템에 의존 (예: `cuVSLAM`) 14              | **내장** (프론트엔드에서 ICP 기반 포즈 추정) 2               |
| **루프 폐쇄 및 전역 최적화** | **미지원** (지역 매핑에 초점) 18                            | **내장** (GPU 가속 포즈 그래프 최적화 백엔드) 2              |
| **설계 철학**                | **모듈성 및 생태계 통합** (Isaac ROS의 구성 요소) 16        | **통합성 및 성능 극대화** (완전한 GPU 파이프라인) 2          |



## 2. III. 아키텍처 및 기술적 구현 심층 분석

두 시스템의 내부 작동 방식을 구성 요소별로 분해하여, GPU 가속 기술이 구체적으로 어떻게 적용되었는지, 그리고 그 구현 방식의 차이가 성능과 기능에 어떠한 영향을 미치는지 기술적으로 상세히 분석한다.

### 2.1  데이터 처리 파이프라인 및 의존성

`nvBlox`는 선형적이고 외부 의존적인 데이터 처리 파이프라인을 가진다. 먼저 외부 시스템(예: `cuVSLAM`)으로부터 깊이/컬러 이미지(`Image`), 카메라 내부 파라미터(`Camera`), 그리고 가장 중요하게는 센서의 포즈(`Transform`)를 입력받는다.13 그 후, 입력된 포즈를 기준으로 깊이 데이터를 3D 공간의 TSDF 복셀 그리드에 통합하는 처리 단계를 거친다.7 최종적으로 경로 계획을 위한 2D ESDF 맵 슬라이스(`EsdfLayer`)와 시각화를 위한 3D 메시(`MeshLayer`)를 출력한다.13 이 구조에서 전체 시스템의 정확도는 전적으로 입력되는 포즈의 품질에 의존하며, `isaac_ros_visual_slam` (`cuVSLAM`)과 같은 외부 VSLAM 노드가 포즈를 제공하는 것이 일반적인 구성이다.14

반면, `coVoxSLAM`은 프론트엔드와 백엔드가 유기적으로 결합된 통합적 구조를 가진다. 시스템의 프론트엔드는 센서 데이터(깊이/컬러 이미지)를 직접 입력받아 TSDF 볼륨에 통합하고, 이를 기반으로 Fast ICP(Iterative Closest Point) 알고리즘을 사용하여 **자체적으로 포즈를 추정**한다.2 동시에 업데이트된 복셀 정보를 ESDF 서브맵(submap)으로 전파한다.2 시스템의 백엔드는 프론트엔드에서 생성된 여러 서브맵들을 관리하며, 주행 거리계(odometry), 루프 폐쇄, 서브맵 간 정합(registration)이라는 세 가지 제약조건을 사용하여 포즈 그래프를 **GPU 상에서 직접 최적화**한다.2 이 최적화된 포즈 정보는 다시 프론트엔드의 맵 일관성을 향상시키는 데 사용될 수 있는 순환적 피드백 구조의 잠재력을 내포한다.

### 2.2  핵심 알고리즘 비교: TSDF 및 ESDF

두 시스템 모두 3D 환경을 표현하기 위해 TSDF와 ESDF를 핵심적인 데이터 표현 방식으로 사용하지만, 이를 계산하고 통합하는 방식에서 중요한 차이를 보인다.

#### 2.2.1 TSDF (Truncated Signed Distance Function) 통합

TSDF는 3D 공간을 복셀 그리드로 나누고 각 복셀에 가장 가까운 표면까지의 부호화된 거리(signed distance)를 저장하는 방식이다. 이 값은 일반적으로 절단(truncation) 거리 $\delta$ 내로 정규화된다. 특정 복셀 $x∈\mathbb R^3$의 TSDF 값 $F(x)$는 다음과 같이 표현될 수 있다.
$$
F(x) = \max(-1, \min(1, \frac{\text{sdf}(x)}{\delta}))
$$
새로운 깊이 측정값 $\delta$가 주어지면, 이전에 저장된 값과 새로운 측정값을 가중 평균하여 점진적으로 업데이트한다.
$$
F_{k+1}(x) = \frac{W_k(x)F_k(x) + w_{k+1}f_{k+1}(x)}{W_k(x) + w_{k+1}}
$$
여기서 $W$와 $w$는 각 측정의 가중치를 의미한다. 이 통합 과정에서 `nvBlox`와 `coVoxSLAM`은 서로 다른 전략을 취한다.

- **`nvBlox`의 투영 기반 방식 (Projection-based):** 3D 공간의 복셀들을 2D 이미지 평면에 투영(project)하고, 해당 픽셀의 깊이 값과 비교하여 TSDF 값을 업데이트하는 방식을 사용한다. 이 방식은 `coVoxSLAM` 논문에서 'projection mapping'으로 지칭되었다.2 구조가 비교적 간단하고 병렬화에 용이하지만, 맵의 해상도가 높아질수록 하나의 픽셀이 여러 복셀에 영향을 주거나 그 반대의 경우가 발생하여 비효율적일 수 있다.22
- **`coVoxSLAM`의 레이캐스팅 기반 방식 (Raycasting-based):** 센서의 각 픽셀에서부터 3D 공간으로 광선(ray)을 쏘아(cast), 그 광선이 통과하는 복셀들을 순차적으로 식별하고 업데이트한다. 이 방식은 복셀을 평가할 때 센서의 모든 포인트를 고려하므로, 가려짐(occlusion) 처리나 센서 노이즈 모델링 등에서 더 정교한 가중치 기법을 적용하기 용이하다. `coVoxSLAM` 논문은 이 방식이 `nvBlox`의 투영 방식보다 우수하며, 자신들의 고도로 최적화된 GPU 구현 덕분에 더 빠르다고 주장한다.2 이 주장은 단순히 알고리즘 선택의 차이를 넘어, GPU 커널 수준의 최적화 기술력(예: 효율적인 메모리 접근 패턴 설계)에서 비롯된 결과일 가능성을 시사한다.

#### 2.2.2 ESDF (Euclidean Signed Distance Function) 계산

ESDF는 각 복셀에서 가장 가까운 '점유된' 복셀까지의 실제 유클리드 거리를 저장하는 필드로, 충돌 검사 및 경로 계획에 매우 유용하다.

- **`nvBlox`의 병렬 알고리즘:** CPU 기반의 선구적 연구인 `Voxblox` 1의 아이디어를 계승하여 GPU에 맞게 최적화했다. 그 과정은 여러 단계로 구성된다.8 먼저 TSDF 맵에서 표면 경계에 있는 복셀들을 '사이트(sites)'로 식별한다. 만약 이전에 사이트였던 복셀이 더 이상 사이트가 아니게 되면(예: 동적 장애물이 사라짐), 해당 사이트를 '부모'로 참조하던 모든 '자식' 복셀들의 거리 값을 무효화(invalidate)한다. 마지막으로, 병렬 스위핑(parallel sweeping) 방식을 사용하여 거리를 전파한다. 이는 각 복셀 블록 내부에서 6개의 축 방향(X+, X-, Y+, Y-, Z+, Z-)으로 스캔을 병렬적으로 수행하며 이웃 복셀로부터 더 짧은 거리 값을 업데이트하는 과정이다. 이 방식은 `PBA (Parallel Banding Algorithm)`와 유사한 원리로 작동한다.8
- **`coVoxSLAM`의 ESDF 계산:** `coVoxSLAM` 역시 `Voxblox`의 아이디어를 계승하며, TSDF 서브맵에서 ESDF 서브맵으로 업데이트된 복셀 정보를 전파하는 방식으로 작동한다.2 `coVoxSLAM` 논문은 자신들의 ESDF 통합 방식이 `Voxblox` 대비 10배에서 50배 빠르다고 주장하는데, 이는 `nvBlox`가 달성한 최대 31배의 성능 향상폭과 비교해볼 만한 수치이다.4

### 2.3  전역 일관성 및 루프 폐쇄

`nvBlox`는 설계상 루프 폐쇄로 인한 포즈 그래프의 급격한 변화를 처리하는 메커니즘이 내장되어 있지 않다.18 사용자가 `cuVSLAM`과 같은 외부 SLAM 시스템과 연동하여 루프 폐쇄를 수행하면, 로봇의 포즈(`map` 프레임 기준)는 올바르게 수정되지만 이미 생성된 `nvBlox`의 메시는 그 변화를 따라가지 못하고 이전의 잘못된 위치에 남아있게 되어 맵이 틀어지는 현상이 발생한다.19 NVIDIA는 이에 대한 해결책으로 `nvBlox`를 전역 매핑(global mapping)이 아닌 지역 매핑(local mapping) 용도로 사용하고, `map_clearing_radius_m` 파라미터를 이용해 로봇 주변 일정 반경 외의 맵을 지속적으로 삭제하여 누적 오차 문제를 근본적으로 회피하는 방식을 권장한다.18

반면, `coVoxSLAM`의 가장 핵심적인 차별점은 GPU에서 완전히 가속되는 백엔드 최적화 기능을 내장하고 있다는 점이다.2 백엔드는 주행 거리계, 루프 폐쇄, 서브맵 정합이라는 세 가지 제약 조건을 최소화하는 비선형 최소제곱 문제를 푼다.11 이를 위해 `Ceres`와 같은 기존의 CPU 기반 최적화 라이브러리를 사용하는 대신, **Levenberg-Marquardt(LM) 알고리즘**과 **공액기울기법(Conjugate Gradient)**을 경량화하여 GPU에 직접 구현했다.4 자코비안 행렬과 잔차(residual) 벡터를 GPU 메모리에 미리 계산하고 할당함으로써 백엔드 실행 시간을 극적으로 단축시킨다.4 이러한 완전한 GPU 기반 최적화는 최근 SLAM 연구의 중요한 흐름이기도 하다.2

### 2.4  데이터 구조: `coVoxSLAM`의 SoA 최적화

`coVoxSLAM`은 성능 극대화를 위해 데이터 구조 수준의 최적화를 도입했다. 바로 **SoA(Structure of Arrays)** 데이터 레이아웃이다.2 전통적인 프로그래밍에서 흔히 사용되는 AoS(Array of Structs) 방식은 `Voxel`이라는 구조체의 배열, 즉 `Voxel[N]` 형태로 데이터를 저장한다. 이 경우 메모리에는 `[Voxel1(x,y,z,w), Voxel2(x,y,z,w),...]` 와 같이 서로 다른 타입의 데이터가 뒤섞여 배치된다.

GPU가 특정 연산, 예를 들어 모든 복셀의 가중치(`w`)만을 업데이트하는 연산을 수행할 때, AoS 구조에서는 각 복셀의 `w`에 접근하기 위해 불필요한 `x,y,z` 데이터를 함께 메모리에서 읽어와야 한다(strided memory access). 이는 GPU의 넓은 메모리 버스를 비효율적으로 사용하게 만들어 대역폭을 낭비하고 캐시 효율성을 저하시킨다.

`coVoxSLAM`이 도입한 SoA 방식은 데이터 구조를 `float x[N], float y[N], float z[N], float w[N]` 와 같이 각 멤버 변수별 배열로 분리한다. 이렇게 하면 특정 연산에 필요한 데이터(예: `w` 배열)가 메모리에 연속적으로 모여있게 된다(coalesced memory access). GPU의 수많은 스레드가 `w` 배열의 각 요소에 동시에 접근할 때, 메모리 트랜잭션 한 번으로 필요한 모든 데이터를 가져올 수 있어 메모리 대역폭을 최적으로 활용하게 된다. 이는 단순한 코드 포팅을 넘어, 알고리즘의 데이터 구조 자체를 GPU 아키텍처에 맞게 근본적으로 재설계했음을 의미하며, 'GPU-aware'를 넘어 'GPU-centric' 설계 철학으로의 전환을 보여주는 중요한 사례이다. `coVoxSLAM`이 SoA를 명시적으로 내세우는 것은 자신들의 성능 우위가 이러한 근본적인 설계 차이에서 비롯됨을 강조하는 것이다.

------

**표 2: 아키텍처 및 알고리즘 상세 비교**

| 구성 요소            | nvBlox                                                       | coVoxSLAM                                                    |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **맵 표현**          | TSDF, ESDF, 색상, 동적 객체 레이어 등 13                     | TSDF, ESDF 서브맵 2                                          |
| **TSDF 통합 방식**   | 투영 기반 매핑 (Projection-based Mapping) 2                  | 레이캐스팅 기반 통합 (Raycasting-based Integration) 2        |
| **ESDF 계산 방식**   | 병렬 스위핑(Parallel Sweeping) 기반의 증분적 업데이트 8      | TSDF 서브맵으로부터 병렬 파면(wavefront) 전파 방식 4         |
| **백엔드 최적화**    | **없음**. 외부 SLAM 시스템의 포즈 보정에 수동적으로 대응해야 함 18 | **GPU 가속 포즈 그래프 최적화**. LM 알고리즘과 공액기울기법 내장 2 |
| **핵심 데이터 구조** | 해시 기반 복셀 블록 관리 (Voxel Block Hashing) 8             | **SoA (Structure of Arrays)** 데이터 레이아웃을 통한 캐시 및 메모리 효율 극대화 2 |
| **동적 환경 처리**   | 사람 분리, 동적 객체 분리 등 별도 레이어 제공, 점유율 감소(decay) 모델 7 | 주된 초점은 아니나, 서브맵 기반 아키텍처가 잠재적으로 동적 환경에 대응할 가능성 내포 |
| **주요 의존성**      | 외부 위치 추정 시스템 (예: `cuVSLAM`), ROS2 (권장) 12        | CUDA 12 (GPU 버전), Ceres (CPU 버전) 11                      |

## 3. IV. 성능 벤치마크 및 자원 활용도

두 시스템의 성능을 정량적으로 비교하는 것은 기술 선택에 있어 매우 중요하다. 각 시스템의 논문과 공식 문서에서 제시된 벤치마크 결과를 통해 그들의 계산 효율성과 실시간성을 분석할 수 있다.

### 3.1  성능 비교: `nvBlox` vs. CPU 기반 방법 (Voxblox)

`nvBlox`는 GPU 가속을 통해 기존의 CPU 기반 접근 방식에 비해 압도적인 성능 향상을 달성했음을 강조한다. 대표적인 CPU 기반 TSDF/ESDF 라이브러리인 `Voxblox`와 비교했을 때, `nvBlox` 논문은 다음과 같은 성능 향상을 보고했다.1

- **표면 재구성 (TSDF 통합):** 최대 **177배**의 속도 향상
- **거리 필드 계산 (ESDF 계산):** 최대 **31배**의 속도 향상

NVIDIA Isaac ROS 문서에 따르면, Replica 데이터셋과 5cm 복셀 크기, NVIDIA 3090 데스크톱 GPU 환경에서 `nvBlox`의 각 컴포넌트 처리 시간은 밀리초(ms) 단위로 측정되어 실시간성이 충분히 확보됨을 보여준다.14 예를 들어, TSDF 통합에 0.5 ms, ESDF 계산에 0.8 ms가 소요된다.

### 3.2  성능 비교: `coVoxSLAM` vs. `nvBlox` 및 `Voxblox`

`coVoxSLAM`은 여기서 한 걸음 더 나아가, 자신들의 시스템이 `nvBlox`보다도 더 뛰어난 성능을 보인다고 주장한다. `coVoxSLAM` 논문은 `nvBlox`를 직접적인 비교 대상으로 삼아 다음과 같은 결과를 제시했다.2

- **TSDF 통합:** `nvBlox` 대비 **1.5배에서 2배**의 속도 향상을 달성했다. 이는 `nvBlox`가 `Voxblox` 대비 보였던 25~38배의 성능 향상을, `coVoxSLAM`은 50배 이상으로 끌어올렸다는 것을 의미한다.2
- **ESDF 통합:** `Voxblox` 대비 **10배에서 50배**의 속도 향상을 보였다.4
- **백엔드 최적화:** CPU 기반의 `Ceres` 솔버 대비 **2배에서 14배**의 속도 향상을 달성했다.4

`coVoxSLAM` 연구팀은 이러한 성능 우위가 앞서 분석한 아키텍처 및 알고리즘의 차이, 즉 ▲레이캐스팅 기반 TSDF 통합 방식의 우수성 ▲SoA 데이터 구조를 통한 메모리 접근 효율 극대화 ▲프론트엔드와 백엔드 전체를 GPU에서 처리하여 CPU-GPU 간 데이터 전송 오버헤드를 제거한 것에서 비롯된다고 분석한다.2

### 3.3  임베디드 플랫폼에서의 성능

두 시스템 모두 로봇에 탑재되는 임베디드 플랫폼에서의 활용을 염두에 두고 설계되었다.

- **`nvBlox`:** NVIDIA Jetson 플랫폼을 주요 타겟으로 하며, 공식 문서에 AGX Orin 및 Orin Nano에서의 성능 데이터가 구체적으로 명시되어 있다.6 예를 들어 AGX Orin 환경(Replica 데이터셋, 5cm 복셀)에서 TSDF 통합에 0.8 ms, ESDF 계산에 1.7 ms가 소요되어 충분한 실시간성을 보여준다. 다만, Orin Nano 4GB 모델은 메모리 부족으로 많은 Isaac ROS 패키지 실행에 부적합할 수 있으며 14, 다중 카메라(특히 ZED 카메라) 사용 시 성능 저하가 보고된 바 있어 시스템 구성 시 주의가 필요하다.29
- **`coVoxSLAM`:** 논문에서 이산(discrete) GPU뿐만 아니라 임베디드 GPU에서도 배포 및 테스트되었으며, 소형 모바일 로봇에 탑재 가능한 임베디드 장치에서도 실시간 성능을 달성할 수 있다고 주장한다.4 하지만 제공된 자료 내에서는 특정 Jetson 플랫폼에서의 구체적인 벤치마크 수치는 명시되어 있지 않다.

이러한 성능 주장의 비대칭성은 기술 발전의 자연스러운 과정을 반영한다. 후발 주자인 `coVoxSLAM`은 기존의 최첨단 기술인 `nvBlox`를 넘어서야 자신의 가치를 입증할 수 있으므로, `nvBlox`를 직접적인 벤치마크 대상으로 삼는 것은 필연적이다. 반면 `nvBlox`가 발표될 당시에는 `coVoxSLAM`이 존재하지 않았으므로, 당시의 SOTA(State-of-the-art)였던 CPU 기반 방법론과의 비교에 집중했다. 현재 `nvBlox` 측에서 `coVoxSLAM`의 성능 주장에 대한 공식적인 반박이나 재검증 자료를 내놓지 않은 상황에서, 모든 성능 평가는 각 연구 그룹의 '자체 보고' 수치에 의존할 수밖에 없다. 따라서 사용자는 이 수치들을 해석할 때, 벤치마크에 사용된 데이터셋, 하드웨어, 측정 방식의 미묘한 차이가 결과에 영향을 미칠 수 있음을 인지해야 한다. 진정한 성능 우위는 향후 TUM-RGBD, EuRoC, KITTI와 같은 표준 벤치마크 데이터셋에 대해 제3자에 의해 수행되는 독립적이고 공정한 비교 평가를 통해 가려질 것이다.30 현재로서는 

`coVoxSLAM`이 아키텍처적으로 더 진보했고 더 빠를 '잠재력'이 높다고 해석하는 것이 타당하다.

------

**표 3: 성능 벤치마크 요약 (논문 주장 기반)**

| 측정 항목              | 비교 대상         | nvBlox                | coVoxSLAM                |
| ---------------------- | ----------------- | --------------------- | ------------------------ |
| **TSDF 통합 속도**     | **Voxblox (CPU)** | 최대 **177배** 빠름 5 | 평균 **50~100배** 빠름 2 |
|                        | **nvBlox**        | -                     | **1.5 ~ 2배** 빠름 2     |
| **ESDF 계산 속도**     | **Voxblox (CPU)** | 최대 **31배** 빠름 5  | **10 ~ 50배** 빠름 4     |
|                        | **nvBlox**        | -                     | 명시적 비교 데이터 없음  |
| **백엔드 최적화 속도** | **Ceres (CPU)**   | 해당 기능 없음        | **2 ~ 14배** 빠름 4      |



## 4. V. 적용 사례 및 생태계

기술의 가치는 실험실에서의 성능뿐만 아니라 실제 세계에서의 활용도와 개발자 생태계의 지원 수준에 따라 결정된다. `nvBlox`와 `coVoxSLAM`은 이 측면에서 뚜렷한 차이를 보인다.

### 4.1  `nvBlox`: NVIDIA Isaac 생태계 기반의 산업 및 개발자 중심 활용

`nvBlox`는 NVIDIA Isaac ROS라는 강력한 생태계의 핵심 구성 요소(GEM)이다.14 이는 NVIDIA의 GPU, Jetson 임베디드 플랫폼, Isaac Sim 시뮬레이터와 매우 긴밀하게 통합되어 있음을 의미하며, 개발자에게 일관되고 강력한 개발 환경을 제공한다.7

주요 적용 사례는 다음과 같다:

- **자율 이동 로봇(AMR) 내비게이션:** `cuVSLAM`으로 포즈를 추정하고 `nvBlox`로 실시간 비용 맵을 생성하여 ROS2의 표준 내비게이션 스택인 `Nav2`에 전달, 동적 장애물을 포함한 환경에서 안전하게 주행하는 것이 가장 대표적인 활용 사례이다.14

- **로봇 팔 모션 생성:** `nvBlox`가 생성한 3D ESDF 맵은 로봇 팔이 주변 환경 및 장애물과 충돌 없이 작업을 수행하기 위한 경로를 생성하는 데 직접적으로 활용된다.7 특히 NVIDIA의 GPU 가속 로봇 모션 생성 라이브러리인 

  `cuRobo`와의 연동은 이 분야의 강력한 솔루션으로 자리 잡고 있다.33

- **동적/의미론적 매핑:** `nvBlox`는 `UNet`과 같은 딥러닝 기반의 의미론적 분할(semantic segmentation) 모델과 연동하여, 입력 이미지에서 사람이나 특정 동적 객체를 식별하고 이를 별도의 맵 레이어에 분리하여 재구성하는 기능을 제공한다.14 이는 움직이는 장애물이 많은 복잡한 환경에서 로봇이 강건하게 작동하도록 돕는다.

NVIDIA는 상세한 공식 문서, 다양한 센서(RealSense, ZED 등)와 시뮬레이터를 활용한 튜토리얼 및 예제 코드를 풍부하게 제공하며 7, 개발자 포럼을 통해 발생하는 문제에 대해 적극적으로 기술을 지원하여 개발자들의 진입 장벽을 낮추고 있다.17

### 4.2  `coVoxSLAM`: 학술 연구 중심의 대규모 전역 매핑 응용

`coVoxSLAM`은 특정 상용 플랫폼에 종속되지 않는 독립적인 오픈소스 프로젝트로, GitHub를 통해 소스 코드가 공개되어 있다.9 이는 연구자들에게 높은 자유도와 유연성을 제공하지만, NVIDIA와 같은 대규모 기업의 체계적인 지원은 기대하기 어렵다.

주요 적용 분야는 다음과 같다:

- **대규모 환경의 전역적 일관성을 갖춘 맵 생성:** `coVoxSLAM`의 핵심 목표이자 가장 적합한 적용 분야이다. 수백 미터에 달하는 드론 비행 데이터나 넓은 실내외 환경에서 누적 오차 없이 정확한 단일 3D 모델을 구축하는 데 강력한 성능을 발휘할 수 있다.2
- **차세대 SLAM 아키텍처 연구:** 프론트엔드와 백엔드 전체를 GPU에서 가속하는 `coVoxSLAM`의 선구적인 아키텍처는 그 자체로 중요한 연구 대상이다. 다른 연구자들이 이를 기반으로 새로운 루프 폐쇄 기법, 다중 로봇 협력 SLAM, 의미론적 SLAM 알고리즘 등을 개발하고 테스트하는 이상적인 플랫폼으로 활용될 수 있다.

개발자 지원은 주로 공식 논문과 GitHub의 README 파일에 의존하며, `nvBlox`에 비해 상세한 튜토리얼이나 활발한 커뮤니티 포럼은 상대적으로 부족할 가능성이 높다.2

`nvBlox`의 의미론적 매핑 기능은 현재 단계에서 진정한 '의미론적 융합(semantic fusion)'이라기보다는 '의미론적 필터링'에 가깝다는 점을 주목할 필요가 있다. `nvBlox`는 외부 세그멘테이션 마스크를 이용해 사람과 같은 특정 클래스를 정적 맵에서 '제거'하거나 '분리'하여 별도의 레이어로 관리한다.14 이는 동적 장애물 회피라는 실용적인 문제를 해결하는 데 매우 효과적이지만, 로봇이 환경을 깊이 이해하는 것과는 거리가 있다. 진정한 의미론적 융합은 `Kimera` 39나 `SemanticInstanceFusion` 40과 같은 연구에서처럼, 각 복셀에 '벽', '바닥', '의자'와 같은 의미론적 클래스를 직접 할당하고 융합하여 '의자'라는 3D 인스턴스를 재구성하는 것을 목표로 한다. 

`nvBlox`의 현재 접근 방식은 실용성에 초점을 맞춘 초기 단계이며, 향후 모듈식 구조를 통해 이러한 고급 의미론적 융합 기능이 추가될 수 있는 확장성을 가지고 있다.

## 5. VI. 한계점 및 향후 발전 방향

두 시스템 모두 현재 기술 수준에서 명확한 한계점을 가지고 있으며, 이는 향후 기술 발전의 방향을 제시한다.

### 5.1  `nvBlox`의 한계점

- **전역 일관성 부재:** 가장 명확하고 근본적인 한계로, 루프 폐쇄나 전역 최적화 기능이 없어 시간이 지남에 따라 누적되는 오차를 자체적으로 보정할 수 없다.18 이로 인해 대규모 환경의 단일 통합 맵을 생성하는 데는 부적합하며, 그 사용이 '지역 매핑'에 국한된다.18

- **비평탄 지형 처리의 제약:** 2D ESDF 슬라이스 모드는 지면이 평평하다는 가정을 기반으로 하므로, 경사로나 요철이 있는 지형에서는 이를 장애물로 잘못 인식하는 문제가 있다.38 3D ESDF 모드를 사용할 수 있지만, 이는 지상 로봇 내비게이션에 즉시 적용하기에는 계산적으로 비효율적일 수 있다.41

- **메모리 및 성능 제약:** 맵의 크기가 커질수록 TSDF 복셀 저장을 위한 메모리 사용량이 선형적으로 증가하며, 이는 무한정 큰 맵을 유지하는 데 한계로 작용한다.18

  `nvBlox` 개발자는 업데이트 계산 비용은 맵 크기에 비례하지 않는다고 주장하지만, 저장 비용은 비례한다. 또한, 다중 카메라 입력이나 고해상도 처리 시 실시간성이 저하될 수 있다는 사용자 보고가 있다.29

### 5.2  `coVoxSLAM`의 한계점

- **생태계 및 지원 부족:** 최신 학술 연구 결과물로서, NVIDIA Isaac과 같은 성숙한 상용 생태계가 제공하는 풍부한 문서, 튜토리얼, 예제, 커뮤니티 지원이 부족할 가능성이 높다. 이는 실제 프로젝트에 적용할 때 높은 진입 장벽으로 작용할 수 있다.11
- **검증 및 강건성 부족:** 논문에서 제시된 특정 데이터셋에서는 우수한 성능을 보였지만, `nvBlox`처럼 다양한 실제 로봇과 예측 불가능한 환경에서 장기간 테스트되며 검증된 강건성은 아직 입증되지 않았다. 실제 환경의 심한 센서 노이즈나 급격한 움직임에 대한 대처 능력이 상대적으로 부족할 수 있다.
- **협업 매핑 기능의 부재:** `coVoxSLAM`은 단일 로봇을 위한 SLAM 시스템으로 설계되었다. 여러 로봇이 협력하여 하나의 맵을 만드는 협업 SLAM(Collaborative SLAM) 기능은 현재 아키텍처에 포함되어 있지 않다.42

`nvBlox`가 전역 매핑을 지원하지 않는 것은 단순한 기술적 한계를 넘어 의도된 '설계적 선택'으로 해석될 수 있다. NVIDIA는 이미 `cuVSLAM`을 통해 강력한 루프 폐쇄 및 포즈 그래프 최적화 기술을 보유하고 있다.46

`nvBlox`에 이 기능을 통합하지 않은 것은, `nvBlox`의 역할을 '전역 맵' 생성이 아닌, 로봇의 '즉각적인 주변 환경'을 가장 빠르고 정확하게 인식하는 것에 집중시키기 위함일 수 있다. 이는 사전에 구축된 정적인 '전역 맵'(예: 건물의 HD 맵) 위에서 로봇이 자신의 위치를 추정하고, `nvBlox`를 이용해 자신의 주변에서 발생하는 '동적인 변화'(예: 움직이는 사람, 새로 놓인 상자)를 실시간으로 감지하여 지역 경로를 수정하는 '분산 지능' 패러다임에 부합한다. 이 관점에서 `nvBlox`의 한계는 단일 로봇의 전역 매핑 관점에서의 한계일 뿐, '사전 맵 기반의 동적 환경 내비게이션'이라는 더 큰 패러다임에서는 핵심적인 강점이 된다. 이는 `coVoxSLAM`이 추구하는 '단일 로봇의 자수성가형 전역 매핑'과는 근본적으로 다른 접근 방식이다.

### 5.3  향후 발전 방향 및 제언

이상적인 미래의 3D 매핑 시스템은 `nvBlox`의 안정성과 생태계 통합 능력, 그리고 `coVoxSLAM`의 완전한 GPU 가속 아키텍처와 전역 일관성 확보 능력을 결합한 형태일 것이다. 예를 들어, `nvBlox`가 `cuVSLAM`으로부터 루프 폐쇄 신호를 받아 맵을 효율적으로 변형하고 재통합하는 기능을 추가하거나, `coVoxSLAM`의 백엔드 최적화 로직을 별도의 ROS 노드로 구현하여 `nvBlox`와 연동하는 방식이 가능하다. 또한, 현재의 기하학적 매핑을 넘어, 환경 내 객체들의 종류와 인스턴스를 인식하고 3D로 재구성하는 진정한 의미론적 SLAM으로의 확장이 중요한 연구 방향이 될 것이다.

## 6. VII. 결론 및 제언

본 보고서는 GPU 가속 3D 매핑 기술의 두 대표주자인 `nvBlox`와 `coVoxSLAM`을 다각도로 심층 분석했다. 분석 결과를 바탕으로 핵심 차이점을 요약하고, 사용자의 목표에 따른 기술 선택 가이드라인을 제시한다.

### 7.1 핵심 차이점 요약

- **정체성:** `nvBlox`는 잘 정의된 API를 가진 고성능 **매핑 라이브러리**이며, NVIDIA Isaac이라는 강력한 산업 생태계의 일부이다. 반면, `coVoxSLAM`은 전역 일관성을 목표로 하는 독립적인 **통합 SLAM 시스템**이며, 학술 연구 결과물에 가깝다.
- **아키텍처:** `nvBlox`는 외부의 포즈 추정기에 의존하는 **모듈식 구조**를 가진다. `coVoxSLAM`은 프론트엔드와 백엔드 모두 GPU에서 처리하는 **완전 통합형 구조**를 채택하여 성능을 극대화한다.
- **핵심 기능:** `nvBlox`는 루프 폐쇄 및 전역 최적화 기능이 **없어** 지역 매핑에 국한된다. `coVoxSLAM`은 GPU 가속 백엔드를 통해 이를 **내장**하고 있어 전역적으로 일관된 맵 생성이 가능하다.
- **성능:** `nvBlox`는 기존 CPU 기반 방법에 비해 압도적으로 빠른 성능을 제공한다. `coVoxSLAM`은 여기서 더 나아가, `nvBlox`보다도 빠른 TSDF 통합 속도와 독자적인 GPU 기반 백엔드 최적화 성능을 **주장한다**.
- **생태계:** `nvBlox`는 **산업계 중심의 강력한 생태계**와 풍부한 문서, 기술 지원을 제공한다. `coVoxSLAM`은 **학술 연구 중심의 오픈소스 프로젝트**로, 유연성은 높지만 지원은 상대적으로 부족하다.

### 6.1  사용자 목표 기반 기술 선택 가이드라인

**다음과 같은 경우 `nvBlox`를 선택하는 것이 바람직하다:**

- 이미 `cuVSLAM`, `RTK-GPS`, Lidar SLAM 등 안정적인 위치 추정 시스템을 보유하고 있으며, 여기에 고성능 3D 재구성 및 실시간 충돌 회피 기능을 추가하고자 할 때.
- ROS/ROS2 기반으로 로봇 애플리케이션을 개발 중이며, NVIDIA Jetson 또는 데스크톱 GPU 플랫폼에서 검증된 성능과 손쉬운 통합을 원할 때.
- 주된 관심사가 로봇 주변의 '지역 맵(local map)'을 활용한 실시간 내비게이션이나 로봇 팔 조작일 때.
- 동적인 사람이나 물체를 감지하고 이를 경로 계획에 실시간으로 반영해야 하는 동적 환경에서 작동해야 할 때.

**다음과 같은 경우 `coVoxSLAM`을 선택하는 것이 바람직하다:**

- 센서 입력만으로 대규모 환경의 전역적으로 일관된 3D 맵을 처음부터 생성하는 단일 통합 시스템이 필요할 때.
- GPU 가속 SLAM, 특히 완전한 GPU 기반 백엔드 최적화나 SoA와 같은 저수준 데이터 구조 최적화에 대한 심도 있는 연구를 수행하고자 할 때.
- 특정 상용 플랫폼에 종속되지 않는 오픈소스 코드를 기반으로 시스템을 깊이 있게 수정하고 확장하고 싶을 때.
- 최고 수준의 처리 속도를 달성하는 것이 프로젝트의 가장 중요한 목표이며, 상대적으로 부족한 문서와 커뮤니티 지원을 감수할 수 있을 때.

### 6.2  최종 제언

`nvBlox`와 `coVoxSLAM`은 직접적인 경쟁 관계라기보다는, 서로 다른 문제 정의와 목표를 가진 상호 보완적인 기술로 이해하는 것이 타당하다. `nvBlox`는 '어떻게 로봇이 주변을 보고 안전하게 움직일 것인가'라는 실용적인 엔지니어링 문제에 대한 강력하고 안정적인 솔루션을 제공한다. 반면, `coVoxSLAM`은 '어떻게 로봇이 자신이 지나온 모든 길을 오차 없이 기억하여 하나의 완벽한 지도로 만들 것인가'라는 근본적인 SLAM 문제에 대한 최신 학술적 답변을 제시한다.

따라서 기술 선택의 첫걸음은, 자신의 프로젝트가 해결하고자 하는 문제가 전자에 가까운지 후자에 가까운지를 명확히 정의하는 것에서부터 시작되어야 한다. 이를 통해 개발자와 연구자는 각 기술의 장점을 최대한 활용하고 단점을 효과적으로 보완하는 최적의 시스템을 구축할 수 있을 것이다.

#### 참고자료

1. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - Semantic search for arXiv papers with AI, accessed August 6, 2025, https://axi.lims.ac.uk/paper/2311.00626
2. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/html/2410.21149v1
3. R²D²: Building AI-based 3D Robot Perception and Mapping with NVIDIA Research, accessed August 6, 2025, https://developer.nvidia.com/blog/r2d2-building-ai-based-3d-robot-perception-and-mapping-with-nvidia-research/
4. (PDF) coVoxSLAM: GPU Accelerated Globally Consistent Dense SLAM - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/385318409_coVoxSLAM_GPU_Accelerated_Globally_Consistent_Dense_SLAM
5. [2311.00626] nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2311.00626
6. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, accessed August 6, 2025, https://github.com/nvidia-isaac/nvblox
7. Nvblox — isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 6, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/index.html
8. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, accessed August 6, 2025, https://arxiv.org/html/2311.00626v2
9. coVoxSLAM: GPU Accelerated Globally Consistent Dense SLAM - Cool Papers, accessed August 6, 2025, https://papers.cool/arxiv/2410.21149
10. [2410.21149] coVoxSLAM: GPU Accelerated Globally Consistent Dense SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2410.21149
11. lrse-uba/coVoxSLAM - GitHub, accessed August 6, 2025, https://github.com/lrse-uba/covoxslam
12. nvblox/README.md at public · nvidia-isaac/nvblox - GitHub, accessed August 6, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/README.md?plain=1
13. nvblox/docs/pages/technical.md at public · nvidia-isaac/nvblox ..., accessed August 6, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/docs/pages/technical.md
14. Isaac ROS Nvblox — isaac_ros_docs documentation, accessed August 6, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nvblox/index.html
15. Technical Details — isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 6, 2025, https://nvidia-isaac-ros.github.io/concepts/scene_reconstruction/nvblox/technical_details.html
16. Isaac ROS (Robot Operating System) - NVIDIA Developer, accessed August 6, 2025, https://developer.nvidia.com/isaac/ros
17. Nvidia Isaac ROS Nvblox, accessed August 6, 2025, https://forums.developer.nvidia.com/t/nvidia-isaac-ros-nvblox/311053
18. What's the realistic maximum size this can run on an Orin? · Issue #96 · NVIDIA-ISAAC-ROS/isaac_ros_nvblox - GitHub, accessed August 6, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/96
19. Does nvblox support loop closure? - Isaac ROS - NVIDIA Developer Forums, accessed August 6, 2025, https://forums.developer.nvidia.com/t/does-nvblox-support-loop-closure/275738
20. AMR navigation using Isaac ROS VSLAM and Nvblox with Intel Realsense camera, accessed August 6, 2025, https://www.einfochips.com/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
21. AMR Navigation Using Isaac ROS VSLAM and Nvblox with Intel Realsense Camera, accessed August 6, 2025, https://www.einfochips.com/blog/amr-navigation-using-isaac-ros-vslam-and-nvblox-with-intel-realsense-camera/
22. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping | Request PDF, accessed August 6, 2025, https://www.researchgate.net/publication/382991572_nvblox_GPU-Accelerated_Incremental_Signed_Distance_Field_Mapping
23. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, accessed August 6, 2025, https://arxiv.org/abs/1611.03631
24. Parallel Banding Algorithm to Compute Exact Distance Transform with the GPU - NUS Computing - National University of Singapore, accessed August 6, 2025, https://www.comp.nus.edu.sg/~tants/pba_files/pba-old.pdf
25. GPU-Accelerated Numerical Differentiation for Loop Closure in Visual SLAM - SFU Summit, accessed August 6, 2025, https://summit.sfu.ca/_flysystem/fedora/2024-06/etd23063.pdf
26. Globally Consistent 3D LiDAR Mapping with GPU-accelerated GICP Matching Cost Factors, accessed August 6, 2025, https://staff.aist.go.jp/shuji.oishi/assets/papers/preprint/GICP_RA-L2021.pdf
27. GPU Implementation of Levenberg-Marquardt Optimization for T1 Mapping - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/323665784_GPU_Implementation_of_Levenberg-Marquardt_Optimization_for_T1_Mapping
28. Hybrid Camera Pose Estimation with Online Partitioning for SLAM - Stony Brook Computer Science, accessed August 6, 2025, https://www3.cs.stonybrook.edu/~hling/publication/slam-icra20.pdf
29. Nvblox not working in realtime - Isaac ROS - NVIDIA Developer Forums, accessed August 6, 2025, https://forums.developer.nvidia.com/t/nvblox-not-working-in-realtime/323644
30. The KITTI Vision Benchmark Suite - Andreas Geiger, accessed August 6, 2025, https://www.cvlibs.net/datasets/kitti/
31. RGB-D SLAM Dataset and Benchmark - Computer Vision Group - Technische Universität München, accessed August 6, 2025, https://cvg.cit.tum.de/data/datasets/rgbd-dataset
32. Physical AI and Robotics Conference Sessions | NVIDIA GTC 2025, accessed August 6, 2025, https://www.nvidia.com/gtc/sessions/physical-ai-and-robotics/
33. nvblox Documentation — nvblox_torch 0.0.8 - GitHub Pages, accessed August 6, 2025, https://nvidia-isaac.github.io/nvblox/
34. cuRobo, accessed August 6, 2025, https://curobo.org/
35. Using with Depth Camera - cuRobo, accessed August 6, 2025, https://curobo.org/get_started/2d_nvblox_demo.html
36. Towards Next-Gen Autonomous Mobile Robotics: A Technical Deep Dive into Visual-Data-Driven AMRs Powered by Kudan Visual SLAM and NVIDIA Isaac Perceptor, accessed August 6, 2025, https://www.kudan.io/blog/a-technical-deep-dive-into-visual-data-driven-amrs-powered-by-kdvisual-and-nvidia-isaac-perceptor/
37. NVIDIA-Isaac-ROS-Nvblox/docs/topics-and-services.md at main - GitHub, accessed August 6, 2025, https://github.com/Tinker-Twins/NVIDIA-Isaac-ROS-Nvblox/blob/main/docs/topics-and-services.md
38. Isaac_ros_nvblox: How to use in Slope Environment - Isaac ROS, accessed August 6, 2025, https://forums.developer.nvidia.com/t/isaac-ros-nvblox-how-to-use-in-slope-environment/333413
39. Efficient Semantic-Aware TSDF Mapping with Adaptive Resolutions | Request PDF, accessed August 6, 2025, https://www.researchgate.net/publication/382610639_Efficient_Semantic-Aware_TSDF_Mapping_with_Adaptive_Resolutions
40. renezurbruegg/SemanticInstanceFusion: Semantic Instance Fusion for 3D reconstruction of RGB-D indoor images in python - GitHub, accessed August 6, 2025, https://github.com/renezurbruegg/SemanticInstanceFusion
41. Use nvblox in a non-flat environment · Issue #61 · NVIDIA-ISAAC-ROS/isaac_ros_nvblox, accessed August 6, 2025, https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_nvblox/issues/61
42. A Collaborative Mapping System for a Brain-Inspired SLAM - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/390648809_A_Collaborative_Mapping_System_for_A_Brain-Inspired_SLAM
43. maplab 2.0 – A Modular and Multi-Modal Mapping Framework | Request PDF - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/366131403_maplab_20_-_A_Modular_and_Multi-Modal_Mapping_Framework
44. Collaborative SLAM for Facilitating Radiological Search and Mapping on a Multi-agent Aerial Platform - Aerospace Controls Laboratory, accessed August 6, 2025, https://acl.mit.edu/projects/cslam-rad-search
45. Epsilon8854/awesome-Collaborative-SLAM - GitHub, accessed August 6, 2025, https://github.com/Epsilon8854/awesome-Collaborative-SLAM
46. cuVSLAM: CUDA accelerated visual odometry and mapping - arXiv, accessed August 6, 2025, https://arxiv.org/html/2506.04359v2
47. cuVSLAM: CUDA accelerated visual odometry and mapping - arXiv, accessed August 6, 2025, https://arxiv.org/html/2506.04359v1
48. cuVSLAM — isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 6, 2025, https://nvidia-isaac-ros.github.io/concepts/visual_slam/cuvslam/index.html
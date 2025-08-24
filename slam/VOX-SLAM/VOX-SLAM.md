# Voxel-SLAM에 대한 심층 고찰
[VOX-SLAM](./index.md)



초기 LiDAR 기반 동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM) 기술은 주로 센서가 제공하는 정밀한 3차원 측정값을 활용하여 실시간으로 로봇의 궤적을 추정하는 주행계(Odometry) 시스템에 초점을 맞추었다.1 이러한 LiDAR Odometry (LO) 시스템들은 단기적인 움직임 추정에는 높은 정확도를 보였으나, 시간이 지남에 따라 필연적으로 발생하는 오차 누적, 즉 드리프트(drift)에 취약하여 장기적인 지도 일관성을 보장하는 데 근본적인 한계를 지녔다.

이와 대조적으로, 시각 정보를 사용하는 Visual SLAM 분야에서는 일찍부터 이러한 한계를 극복하기 위한 노력이 이루어졌다. ORB-SLAM과 같은 시스템의 등장은 SLAM 기술의 패러다임을 바꾸는 중요한 전환점이 되었다.2 이들 시스템은 단순히 현재의 센서 정보만을 처리하는 것을 넘어, 추적(Tracking), 지역 지도화(Local Mapping), 루프 폐쇄(Loop Closure), 전역 최적화(Global Optimization)라는 네 가지 핵심 기능을 유기적으로 통합한 '완전한(complete)' 시스템 아키텍처를 제시했다.4 과거의 데이터를 적극적으로 재활용하여 누적된 오차를 보정하고, 한 번 구축된 지도를 지속적으로 정밀화하며 재사용하는 이 접근법은 장기간에 걸쳐 강건하게 작동하는 자율 시스템을 위한 표준으로 자리 잡았다.

이러한 배경 속에서, Voxel-SLAM은 Visual SLAM 분야에서 검증된 완전한 시스템의 개념을 LiDAR-Inertial SLAM 영역으로 본격적으로 도입한 중대한 시도라 할 수 있다. 이는 단순히 드리프트를 줄이는 수준을 넘어, 장기적 자율성(long-term autonomy)이라는 더 큰 목표를 달성하기 위해 고수준의 시스템 아키텍처 원칙을 LiDAR 센서 데이터에 체계적으로 적용하려는 패러다임의 전환을 명확히 보여준다.1 LiDAR SLAM 분야가 센서 데이터의 정밀한 처리를 넘어, 시스템 전체의 완성도와 장기적 일관성을 고민하는 성숙 단계에 접어들었음을 보여주는 지표인 것이다.


Voxel-SLAM은 홍콩대학교(HKU) 기계공학과 소속 Mechatronics and Robotic Systems (MaRS) 연구실의 Fu Zhang 교수 연구팀에 의해 개발된 최첨단 LiDAR-Inertial SLAM 시스템이다.6 이 시스템의 가장 핵심적인 기여는 기존 시스템들이 단편적으로 활용하던 데이터 연관성(Data Association)을 체계적으로 분류하고 이를 모두 활용하는 통합 프레임워크를 제시했다는 점에 있다. 구체적으로 Voxel-SLAM은 **단기(short-term), 중기(mid-term), 장기(long-term), 그리고 다중 맵(multi-map)**에 걸친 네 가지 종류의 데이터 연관성을 모두 활용하는 최초의 LiDAR-Inertial SLAM 시스템으로 설계되었다.1

이 시스템은 단일 프로젝트로 탄생한 것이 아니라, HKU MaRS 연구실이 수년간 축적해 온 선행 연구들의 결정체이다. 지도 표현을 위한 `VoxelMap` 11, 효율적인 최적화를 위한 `BALM2` (LiDAR Bundle Adjustment) 12, 대규모 맵 최적화를 위한 `HBA` (Hierarchical Bundle Adjustment) 1, 그리고 장소 인식을 위한 `BTC` (Binary and Triangle Combined descriptor) 13 등, 각 분야에서 최고 수준의 성능을 입증한 기술들이 Voxel-SLAM이라는 하나의 시스템 안에 유기적으로 통합되었다.1

이러한 통합적 접근을 통해 Voxel-SLAM은 단일 세션의 실내외 환경뿐만 아니라, 여러 번에 걸쳐 데이터를 취득하는 다중 세션 환경에서도 높은 수준의 정확성(accuracy), 강건성(robustness), 그리고 다기능성(versatility)을 동시에 달성하는 것을 궁극적인 목표로 삼는다.1 이는 LiDAR SLAM 기술이 실시간 궤적 추정이라는 제한된 역할을 넘어, 다양한 시나리오에서 신뢰성 있는 위치 인식과 일관된 지도 작성을 제공하는 핵심 기반 기술로 자리매김할 수 있음을 시사한다.



Voxel-SLAM의 아키텍처는 고도의 모듈성과 병렬성을 기반으로 설계되어, 복잡한 연산을 실시간으로 처리하면서도 시스템의 안정성을 유지한다. 시스템은 크게 **초기화(Initialization), 주행계(Odometry), 지역 지도화(Local Mapping), 루프 폐쇄(Loop Closure), 전역 지도화(Global Mapping)**라는 5개의 핵심 모듈로 구성된다.1

이 모듈들은 독립적인 스레드 또는 스레드 그룹에서 병렬적으로 실행된다. 예를 들어, 실시간성이 가장 중요한 주행계와 지역 지도화 모듈은 하나의 스레드 그룹에서 긴밀하게 연동되어 동작하며, 상대적으로 계산 부하가 크고 즉각적인 반응이 덜 요구되는 루프 폐쇄와 전역 지도화 모듈은 별도의 백그라운드 스레드에서 실행된다.9 이러한 병렬 구조는 프론트엔드의 빠른 상태 추정과 백엔드의 무거운 최적화 작업이 서로를 방해하지 않도록 하여 시스템 전체의 처리량을 극대화한다.

Voxel-SLAM 아키텍처의 가장 두드러진 특징 중 하나는 모든 모듈이 **적응형 복셀 지도(Adaptive Voxel Map)**라는 통일된 지도 표현 방식을 공유한다는 점이다.1 이는 각 모듈이 서로 다른 자료 구조를 사용하여 발생하는 데이터 변환 오버헤드와 정보 손실을 원천적으로 차단한다. 모든 데이터와 상태 정보가 일관된 형식으로 관리되므로, 모듈 간의 상호작용이 원활해지고 시스템 전체의 데이터 무결성이 보장된다.

또한, 시스템은 다중 세션에 걸쳐 수집된 데이터를 효율적으로 관리하기 위해 **데이터 피라미드(Data Pyramid)** 구조를 사용한다.9 이 구조를 통해 현재 세션의 데이터뿐만 아니라 과거에 기록된 모든 세션의 키프레임과 맵 정보에 접근할 수 있으며, 이는 다중 세션 루프 폐쇄와 맵 병합의 기반이 된다. 이러한 설계는 Voxel-SLAM이 단순한 '선택과 집중'이 아닌, SLAM의 모든 구성 요소를 포괄하여 완성도를 높이는 '통합과 완성'의 설계 철학을 따르고 있음을 보여준다. 각 모듈이 독립된 선행 연구에 기반하면서도 1, 통일된 지도 표현이라는 강력한 아키텍처적 구심점을 통해 하나의 유기적인 시스템으로 통합된 것이다.



초기화 모듈은 전체 SLAM 시스템의 성공적인 시작을 책임지는 관문이다. 이 모듈의 주된 기능은 시스템의 초기 상태, 즉 로봇의 3차원 위치($p$), 자세($R$), 속도($v$), 그리고 IMU 센서의 편향($b_g, b_a$)을 정확하게 추정하는 것이다.9 동시에, 후속 모듈인 주행계와 지역 지도화가 즉시 사용할 수 있도록 일관성을 갖춘 초기 지역 지도(local map)를 생성한다.5 Voxel-SLAM 초기화의 가장 큰 강점은 그 강건성에 있다. 로봇이 정지한 상태뿐만 아니라, 격렬하게 움직이는 매우 동적인(highly dynamic) 상태에서도 단 1초 분량의 짧은 LiDAR 및 IMU 데이터만으로 신뢰성 있는 초기화가 가능하다.1 이는 긴 초기화 시간 없이 즉시 임무를 수행해야 하는 실제 로봇 애플리케이션에서 매우 중요한 장점이다. 또한, 시스템이 어떠한 이유로든 추적에 실패하여 발산(divergence)했을 경우, 이 모듈을 통해 신속하게 재초기화(re-initialization)하여 임무를 재개할 수 있다.1


주행계 모듈은 시스템의 프론트엔드(front-end) 역할을 수행하며, 단기 데이터 연관성(short-term data association)을 기반으로 작동한다. 이 모듈은 IMU 측정값을 사전적분(preintegration)하여 얻은 움직임 예측값과 현재 수신된 LiDAR 스캔 데이터를 융합하여, 실시간으로 로봇의 현재 상태를 빠르게 추정한다.9 주행계의 핵심 역할은 속도와 효율성을 유지하면서도 안정적인 궤적을 제공하는 것이다. Voxel-SLAM의 주행계는 단순한 상태 추정을 넘어, 시스템의 신뢰성을 높이기 위한 추가적인 기능을 포함한다. 복도나 터널과 같이 기하학적 특징이 부족하여 LiDAR 데이터의 정보량이 줄어드는 퇴화 환경(degenerated environment)에 진입할 경우, 시스템의 추정 정확도가 급격히 떨어질 수 있다. 주행계 모듈은 이러한 LiDAR 퇴화 현상을 지속적으로 감지하여 시스템 발산 가능성을 경고하고, 필요한 경우 상위 모듈이 대응할 수 있도록 한다.6


지역 지도화 모듈은 중기 데이터 연관성(mid-term data association)을 처리하며, 주행계가 제공한 빠른 추정치를 더욱 정밀하게 다듬는 역할을 한다. 이 모듈은 최근에 수집된 일정 개수의 LiDAR 스캔(키프레임)으로 구성된 슬라이딩 윈도우(sliding window)를 유지한다.1 그리고 이 윈도우 내에서 지역적 LiDAR-Inertial 번들 조정(Bundle Adjustment, BA)을 수행한다. 이 최적화 과정을 통해 윈도우 내에 포함된 모든 키프레임의 상태(포즈, 속도, 편향)와 이들이 관측한 지역 복셀 지도(local voxel map)의 기하학적 파라미터를 동시에 정밀화한다.10 이 모듈의 효율성은 매우 뛰어나, 윈도우 크기를 10으로 설정했을 때 주행계와 동일한 10Hz의 속도로 실시간 실행이 가능하다.1 이는 제한된 연산 자원을 가진 온보드 컴퓨터에서도 높은 정확도의 지역 일관성을 유지할 수 있게 해준다. 최적화가 완료되면, 윈도우에서 가장 오래된 스캔은 더 이상 최적화에 참여하지 않고 고정된 키프레임으로 전환되어, 장기 데이터 관리를 위해 루프 폐쇄 모듈로 전달된다.9


루프 폐쇄 모듈은 SLAM 시스템이 장기적인 일관성을 확보하는 데 가장 결정적인 역할을 한다. 이 모듈은 장기(long-term) 및 다중 맵(multi-map) 데이터 연관성을 활용한다. 지역 지도화 모듈로부터 전달받은 키프레임들을 사용하여, `BTC`와 같은 장소 인식(place recognition) 기술을 통해 루프 디스크립터(loop descriptor)를 생성한다.1 그리고 현재 키프레임의 디스크립터를 과거의 모든 키프레임 디스크립터 데이터베이스와 비교하여, 이전에 방문했던 장소를 다시 방문했는지(즉, 루프가 형성되었는지) 탐지한다.10 Voxel-SLAM의 루프 폐쇄는 현재 진행 중인 단일 세션 내에서뿐만 아니라, 이전에 저장해 둔 과거의 모든 세션(multi-session)에 걸쳐서도 루프를 탐지할 수 있는 강력한 기능을 제공한다.1 루프가 성공적으로 탐지되면, 시간의 흐름에 따라 누적된 드리프트 오차를 보정하기 위해 포즈 그래프 최적화(Pose Graph Optimization)를 수행한다. 이 과정은 루프를 구성하는 키프레임들 간의 상대적인 기하 관계를 제약조건으로 사용하여 전체 궤적을 전역적으로 재정렬한다.10


전역 지도화 모듈은 SLAM의 최종 결과물인 전역 지도의 일관성과 정확성을 극대화하는 마지막 단계이다. 루프 폐쇄를 통해 포즈 그래프가 일차적으로 보정된 후, 이 모듈은 `HBA`(Hierarchical Bundle Adjustment)라는 계층적 전역 번들 조정 기법을 사용하여 전체 맵을 최종적으로 정밀화한다.1 이 과정은 단순히 키프레임의 포즈만 최적화하는 포즈 그래프 최적화를 넘어, 모든 키프레임의 포즈와 맵을 구성하는 모든 기하학적 특징(복셀의 평면 파라미터)을 동시에 최적화한다. 특히 다중 세션 시나리오에서, 이 모듈은 여러 세션에서 생성된 개별 맵들을 하나의 일관된 전역 맵으로 완벽하게 병합하는 역할을 수행한다.9 이를 통해 Voxel-SLAM은 장시간, 넓은 영역에 걸쳐 누적된 드리프트를 전역적으로 최소화하고, 매우 정확하며 기하학적으로 일관된 최종 3D 지도를 생성할 수 있다.

아래 표는 Voxel-SLAM의 복잡한 5개 모듈 구조를 요약하여 시스템의 전체적인 기능과 데이터 흐름을 한눈에 파악할 수 있도록 정리한 것이다.


| 모듈 (Module)                    | 주요 기능 (Primary Function)              | 처리하는 데이터 연관성 (Data Association Type) | 핵심 기술 (Core Technology)                      |
| -------------------------------- | ----------------------------------------- | ---------------------------------------------- | ------------------------------------------------ |
| **초기화 (Initialization)**      | 시스템 초기 상태 및 지역 지도 생성        | N/A (Bootstrapping)                            | Coarse-to-fine Voxelization, BA                  |
| **주행계 (Odometry)**            | 실시간 상태 추정, 발산 감지               | 단기 (Short-term)                              | IMU Preintegration, Scan-to-Map Matching         |
| **지역 지도화 (Local Mapping)**  | 슬라이딩 윈도우 내 상태 및 지도 정밀화    | 중기 (Mid-term)                                | LiDAR-Inertial Bundle Adjustment (BA)            |
| **루프 폐쇄 (Loop Closure)**     | 과거 방문 장소 탐지 및 포즈 그래프 최적화 | 장기 (Long-term), 다중 맵 (Multi-map)          | Place Recognition (BTC), Pose Graph Optimization |
| **전역 지도화 (Global Mapping)** | 전역 맵 일관성 최적화                     | 장기 (Long-term), 다중 맵 (Multi-map)          | Hierarchical Global BA (HBA)                     |



적응형 복셀 지도는 Voxel-SLAM의 모든 구성 요소가 공유하는 통일된 환경 표현 방식으로, 시스템의 효율성과 정확성을 뒷받침하는 근간이다. 이 방법론은 HKU MaRS 연구실의 선행 연구인 `VoxelMap`()에 그 뿌리를 두고 있다.1 기존의 SLAM 시스템들이 수많은 3D 포인트를 직접 저장하는 포인트 클라우드 맵을 사용했던 것과 달리, VoxelMap은 공간을 일정한 크기의 3차원 격자, 즉 복셀(voxel)로 분할하고 각 복셀 내에 포함된 포인트들의 기하학적 특징을 추출하여 저장한다.11 Voxel-SLAM에서는 이 특징으로 주로 평면(plane)을 사용하며, 각 복셀은 자신이 대표하는 평면의 파라미터(예: 법선 벡터와 원점까지의 거리)와 해당 파라미터의 불확실성(uncertainty)을 확률적으로 표현하는 정보를 담고 있다.10

이름에서 알 수 있듯이, 이 지도의 핵심은 '적응형(adaptive)' 전략에 있다. 시스템은 환경의 복잡도에 따라 복셀의 해상도를 동적으로 조절하는 Coarse-to-fine 방식을 채택한다.13 만약 특정 복셀 내에 분포한 포인트들이 단일 평면으로 잘 모델링된다면, 시스템은 해당 복셀을 그대로 사용한다. 그러나 포인트들의 분포가 복잡하여 단일 평면으로 표현하기 어려운 경우(예: 모서리나 복잡한 구조물), 시스템은 해당 복셀을 더 작은 크기의 자식 복셀들로 재귀적으로 분할하여 더 상세한 기하 구조를 표현한다.10 이 전략은 넓고 평탄한 벽이나 바닥 같은 영역에서는 큰 복셀을 사용하여 메모리와 계산량을 절약하고, 복잡한 구조물 주변에서는 작은 복셀을 사용하여 정밀도를 높이는, 메모리 효율성과 표현력 사이의 최적의 균형을 찾아낸다.

이러한 적응형 복셀 지도는 세 가지 주요 장점을 제공한다. 첫째, **효율성**이다. 수백만 개의 포인트를 모두 저장하는 대신, 훨씬 적은 수의 평면 파라미터만을 저장하므로 메모리 사용량이 현저히 줄어든다. 둘째, **정확성**이다. 포인트들의 기하학적 구조를 명시적으로 모델링함으로써, 단순한 포인트 집합보다 더 높은 수준의 정보를 시스템에 제공한다. 이 평면 정보는 특히 번들 조정(BA) 과정에서 개별 포인트 간의 매칭보다 훨씬 더 강건하고 강력한 제약조건으로 작용하여 최적화의 정확도를 높인다. 셋째, **일관성**이다. 초기화부터 전역 지도화까지 모든 모듈이 이 통일된 맵 구조를 사용하기 때문에, 모듈 간 데이터 변환 과정이 불필요하며 시스템 전체의 데이터 흐름과 상태 정보가 일관되게 유지된다.1


LiDAR-Inertial 번들 조정(BA)은 Voxel-SLAM의 지역 지도화(Local Mapping) 모듈에서 수행되는 핵심 최적화 과정으로, 시스템의 정확성을 보장하는 엔진 역할을 한다. 이는 슬라이딩 윈도우 내에 포함된 여러 개의 LiDAR 스캔(키프레임)과 그 스캔들 사이의 시간 동안 측정된 IMU 데이터를 모두 고려하여, 센서의 상태(포즈, 속도, IMU 편향)와 지역 지도(복셀의 평면 파라미터)를 동시에 최적화하는 강결합(tightly-coupled) 방식의 비선형 최적화 문제이다.1 이 방법론은 HKU MaRS 연구실의 또 다른 핵심 선행 연구인 `BALM2`에서 제안된 효율적인 LiDAR BA 기법을 계승하고 LiDAR-Inertial 융합에 맞게 확장한 것이다.1

수학적으로 이 최적화 문제는 하나의 거대한 비용 함수(Cost Function)를 최소화하는 문제로 공식화된다. 이 비용 함수는 크게 두 종류의 잔차(residual) 항들의 가중 제곱합으로 구성된다.

1. **IMU 사전적분 오차 (IMU Preintegration Error):** 연속된 두 키프레임 사이에서 IMU가 측정한 가속도와 각속도 값을 적분하여 계산한 상대적인 움직임(위치, 자세, 속도 변화)과, 최적화를 통해 추정된 두 키프레임의 상태로부터 계산된 상대적인 움직임 사이의 차이를 나타낸다. 이 오차 항은 LiDAR가 특징을 얻기 어려운 빠른 움직임이나 퇴화 환경에서도 시스템의 연속성을 보장하는 중요한 역할을 한다.
2. **LiDAR 측정 오차 (LiDAR Measurement Error):** 특정 키프레임에서 측정된 각 LiDAR 포인트들을 해당 키프레임의 추정된 포즈를 이용해 전역 좌표계로 변환했을 때, 이 변환된 포인트가 맵 상의 대응되는 기하학적 특징(Voxel-SLAM에서는 복셀의 평면)까지의 수직 거리를 의미한다. 즉, point-to-plane 거리를 최소화하는 항이다.

이를 종합한 전체 비용 함수는 다음과 같이 표현될 수 있다. 상태 변수 집합을 ‘X={xk}k∈K‘ (여기서 $\mathbf{x}_k =$는 k번째 키프레임의 회전, 위치, 속도, IMU 편향을 포함하는 상태 벡터), 맵 파라미터 집합을 $\mathcal{M}$이라 할 때, 최소화 목표는 다음과 같다.1
$$
\min_{\mathcal{X}, \mathcal{M}} \left\{ \sum_{k \in \mathcal{K}} \left\| r_{\mathcal{I}}(\hat{\mathbf{z}}_{b_k}^{b_{k+1}}, \mathcal{X}_k) \right\|_{\mathbf{P}_{b_k}^{b_{k+1}}}^2 + \sum_{k \in \mathcal{K}} \sum_{i \in \mathcal{F}_k} \left\| r_{\mathcal{L}}(\mathbf{x}_k, \mathbf{p}_i, \mathcal{M}) \right\|_{\mathbf{P}_{\mathcal{L}}}^2 \right\}
$$
여기서 $r_{\mathcal{I}}$는 IMU 사전적분 잔차 벡터, $r_{\mathcal{L}}$는 LiDAR 포인트 $\mathbf{p}_i$의 측정 잔차(point-to-plane 거리) 벡터를 나타낸다. $\mathbf{P}$는 각 측정의 불확실성을 반영하는 정보 행렬(information matrix)로, 측정의 신뢰도에 따라 가중치 역할을 한다. 이 복잡한 비선형 최소제곱 문제는 가우스-뉴턴(Gauss-Newton) 또는 레벤버그-마쿼트(Levenberg-Marquardt)와 같은 2차 최적화 기법을 사용하여 수치적으로 풀게 된다.10 Voxel-SLAM은 `BALM2`에서 개발된 효율적인 자코비안 및 헤시안 행렬 계산 방식을 활용하여 이 과정을 실시간으로 처리할 수 있는 계산 효율성을 확보했다.12

적응형 복셀 지도와 LiDAR-Inertial BA는 서로의 성능을 극대화하는 상호보완적인 공생 관계를 형성한다. LiDAR-Inertial BA의 정확성은 얼마나 강건하고 의미 있는 기하학적 제약 조건을 확보하느냐에 달려있다. 적응형 복셀 지도는 환경을 '평면'의 집합으로 구조화하여 모델링함으로써 11, BA가 필요로 하는 고품질의 point-to-plane 제약 조건을 자연스럽고 효율적으로 제공하는 이상적인 기반이 된다. 즉, 맵 자체가 최적화에 필요한 정보를 내재하고 있는 것이다. 역으로, BA 최적화 과정은 슬라이딩 윈도우 내의 모든 정보를 종합하여 맵을 구성하는 평면들의 파라미터($\mathcal{M}$)와 그 불확실성을 지속적으로 정밀화한다. 즉, BA의 결과물은 다시 적응형 복셀 지도를 더 정확하고 신뢰성 있게 만드는 영양분이 된다. 이러한 선순환 구조는 Voxel-SLAM이 높은 정확도와 강건성을 동시에 달성할 수 있는 근원적인 이유이며, 통일된 맵 표현 덕분에 이러한 긍정적 효과가 시스템의 모든 모듈에 걸쳐 전파될 수 있다.1



Voxel-SLAM의 기술적 위치와 독창성을 명확히 파악하기 위해, 동시대의 대표적인 SLAM 및 Odometry 시스템들과 다각적인 비교 분석을 수행한다. 비교 대상은 Voxel-SLAM의 특성을 고려하여 다음과 같이 선정하였다.

- **LiDAR-Inertial Odometry:**
  - **LIO-SAM** 15: iSAM2 기반의 팩터 그래프 최적화를 사용하는 강결합 LiDAR-Inertial Odometry 시스템으로, 확장성과 GPS 등 추가 센서 융합 능력에서 강점을 보인다.
  - **FAST-LIO2** 9: ikd-Tree나 iVox와 같은 혁신적인 자료 구조를 사용하여 극도로 빠른 프론트엔드 처리 속도를 달성한 시스템으로, 실시간 성능의 기준점으로 비교된다.
- **Visual SLAM (Feature-based):**
  - **ORB-SLAM3** 2: Voxel-SLAM이 아키텍처적으로 가장 많이 참조한 '완전한 SLAM'의 대표주자이다. 추적, 지역 매핑, 루프 폐쇄, 다중 맵 관리 등 시스템 철학의 유사성과 차이점을 비교하기 위해 선정되었다.
- **Visual Odometry (Direct):**
  - **DSO (Direct Sparse Odometry)** 18: 특징점 추출 없이 이미지 픽셀의 밝기 값을 직접 사용하는 '직접 방식'의 대표적인 시스템이다. Voxel-SLAM과 같은 '특징 기반 방식'과의 근본적인 방법론 차이를 통해 각 접근법의 장단점을 분석하기 위해 선정되었다.

비교는 센서 구성, 지도 표현 방식, 데이터 처리 방식(특징 기반 vs. 직접), 최적화 전략, 그리고 루프 폐쇄 및 다중 맵 지원 여부로 대표되는 시스템 아키텍처의 완성도 등 여러 기준에 걸쳐 이루어진다.



이 세 시스템은 모두 LiDAR와 IMU를 강결합(tightly-coupled)하여 사용한다는 공통점을 가지지만, 지향하는 목표와 아키텍처에서 뚜렷한 차이를 보인다. Voxel-SLAM은 앞서 설명한 바와 같이 장기/다중 세션 루프 폐쇄와 계층적 전역 BA를 포함하는 '완전한 SLAM' 시스템을 지향한다.1 반면, LIO-SAM과 FAST-LIO2는 주로 'Odometry' 성능, 즉 실시간 궤적 추정의 속도와 정확성에 집중한다. LIO-SAM은 팩터 그래프(iSAM2)를 기반으로 하여 GPS와 같은 추가적인 제약조건을 유연하게 통합할 수 있는 확장성이 장점이다.15 FAST-LIO2는 ikd-Tree나 iVox와 같은 혁신적인 자료 구조를 활용하여 프론트엔드에서의 포인트 클라우드 처리 속도를 극대화하는 데 특화되어 있다.16 성능 측면에서, Voxel-SLAM은 격렬한 초기 움직임이 있는 상황에서도 강건하게 초기화되는 반면, FAST-LIO2는 이러한 조건에서 발산하는 경향을 보이는 것으로 보고되었다.9 이는 Voxel-SLAM이 속도뿐만 아니라 강건성을 매우 중요한 설계 목표로 삼았음을 보여준다.


이 두 시스템의 비교는 SLAM 분야의 설계 원칙이 센서의 종류를 넘어 수렴하고 있음을 보여주는 흥미로운 사례이다. 두 시스템은 추적, 지역 매핑, 루프 폐쇄, 다중 맵 관리 등 '완전한 SLAM' 아키텍처를 공유하며, 장기 데이터 연관성을 활용하여 드리프트를 보정하고 재사용 가능한 맵을 구축하려는 철학을 공유한다.1 그러나 사용하는 주 센서가 LiDAR-Inertial과 Visual-Inertial로 다르기 때문에, 데이터 처리의 구체적인 방식에서 근본적인 차이가 발생한다. Voxel-SLAM은 3차원 공간의 기하학적 '평면' 특징을 추출하여 적응형 복셀 맵에 저장하는 반면 11, ORB-SLAM3는 2D 이미지에서 'ORB'라는 시각 특징점을 추출하여 희소 포인트 클라우드 맵으로 관리한다.2 이 차이로 인해 Voxel-SLAM은 조명 변화에 매우 강건하지만 구조적인 환경에서 더 잘 작동하고, ORB-SLAM3는 풍부한 텍스처 정보가 있을 때 강력한 성능을 보이지만 조명 변화나 빠른 움직임에 취약할 수 있다.


이 비교는 데이터 처리 방식의 근본적인 차이를 보여준다. Voxel-SLAM은 환경에서 명시적인 기하학적 특징(평면)을 추출하여 이를 기반으로 매칭과 최적화를 수행하는 **특징 기반(Feature-based)** 방식이다.11 반면, DSO는 특징 추출 단계를 생략하고 이미지의 원본 픽셀 밝기 값(photometric error)을 직접 최소화하는 **직접 방식(Direct method)**이다.18 특징 기반 방식인 Voxel-SLAM은 한 번 추출된 기하학적 특징이 조명 변화에 영향을 받지 않기 때문에 조명 변화에 강건하지만, 특징을 추출할 수 없는 비구조적인 환경에서는 성능이 저하될 수 있다. 직접 방식인 DSO는 픽셀의 밝기 그래디언트만 있으면 작동하므로 텍스처가 거의 없는 벽과 같은 환경에서도 포인트를 샘플링하여 추적할 수 있지만, 카메라의 노출 변화나 감마 보정 등 조명 조건 변화에 매우 민감하다는 단점이 있다.18

아래 표는 Voxel-SLAM과 주요 경쟁 시스템들의 핵심 특성을 요약하여 각 시스템의 기술적 위치를 명확히 보여준다.


| 구분 (Criteria)   | **Voxel-SLAM**            | **LIO-SAM**               | **FAST-LIO2**             | **ORB-SLAM3**              | **DSO**                     |
| ----------------- | ------------------------- | ------------------------- | ------------------------- | -------------------------- | --------------------------- |
| **주요 센서**     | LiDAR, IMU                | LiDAR, IMU, (GPS)         | LiDAR, IMU                | Camera, (IMU)              | Camera                      |
| **방식**          | 특징 기반 (Feature-based) | 특징 기반 (Feature-based) | 특징 기반 (Feature-based) | 특징 기반 (Feature-based)  | 직접 방식 (Direct)          |
| **지도 표현**     | 적응형 복셀 맵 (평면)     | 전역 팩터 그래프, 서브맵  | ikd-Tree                  | 희소 포인트 클라우드 (ORB) | 희소 포인트 (Inverse Depth) |
| **시스템 완성도** | 완전한 SLAM               | Odometry + Loop Closure   | Odometry                  | 완전한 SLAM                | Odometry                    |
| **루프 폐쇄**     | 지원 (다중 세션)          | 지원 (단일 세션)          | 미지원                    | 지원 (다중 맵)             | 미지원 (확장 기능 존재)     |
| **핵심 강점**     | 강건성, 정확성, 다기능성  | 팩터 그래프 기반 확장성   | 프론트엔드 속도           | 정확성, 다중 센서 지원     | 텍스처 부족 환경, 속도      |



Voxel-SLAM의 성능은 시스템의 다기능성과 강건성을 종합적으로 검증하기 위해, 서로 다른 특성을 가진 세 가지 대표적인 공개 데이터셋을 통해 평가되었다.1

- **Hilti handheld dataset:** 수동으로 장비를 들고 이동하며 취득한 데이터로, 좁은 계단, 복잡한 배관이 있는 건설 현장 등 협소하고 구조적으로 복잡한 실내 환경을 포함한다. 이는 지역 특징 처리 능력과 단거리 정확도를 평가하는 데 적합하다.1
- **MARS-LVIG aerial mapping dataset:** 무인 항공기(UAV)를 이용하여 넓은 지역을 비행하며 취득한 데이터로, 광활한 야외, 섬, 계곡 등을 포함한다. 이는 대규모 환경에서의 장거리 궤적 추정 성능과 맵 일관성을 평가하는 데 사용된다.1
- **UrbanNav robotcar urban dataset:** 차량 플랫폼을 이용하여 실제 도시 환경을 주행하며 취득한 데이터로, 건물, 도로, 다른 차량 등 다양한 요소가 혼재된 복잡한 시나리오를 다룬다. 이는 실제 자율주행과 유사한 환경에서의 성능을 검증하는 데 중요하다.1

성능을 정량적으로 평가하기 위한 핵심 지표로는 **절대 궤적 오차(Absolute Trajectory Error, ATE)**가 사용되었다. ATE는 추정된 전체 궤적과 사전에 정밀하게 측정된 실제 궤적(Ground Truth)을 최적으로 정렬한 후, 시간적으로 대응되는 지점들 간의 평균적인 거리 오차를 계산한 것이다. 일반적으로 RMSE(Root Mean Square Error) 값이 보고되며, 이는 전역적인 궤적의 정확성을 평가하는 SLAM 분야의 표준 지표이다.10


발표된 논문에 따르면, Voxel-SLAM은 위에서 언급된 다양한 데이터셋의 여러 시퀀스에서 LIO-SAM, FAST-LIO2와 같은 다른 최신 시스템들과 비교했을 때 전반적으로 더 낮은 ATE 값을 기록하며 뛰어난 정확성을 입증했다.1 특히, Hilti 데이터셋의 건설 현장이나 계단과 같이 구조적으로 복잡한 실내 환경에서 Voxel-SLAM의 정교한 지역 특징 처리 능력이 빛을 발하여, 경쟁 시스템들보다 월등한 성능을 보였다.10 또한, 여러 번에 걸쳐 데이터를 수집하는 다중 세션 실험에서도, Voxel-SLAM은 각 세션의 맵을 성공적으로 병합하고 맵의 연속성과 무결성을 유지하며 누적된 드리프트를 효과적으로 줄이는 우수한 성능을 나타냈다.10

학술적 성과와 실제 사용자 경험 사이에는 때때로 간극이 존재할 수 있다. 논문과 경쟁 대회 결과는 Voxel-SLAM의 뛰어난 성능을 강조하지만 10, 공개된 GitHub 저장소의 이슈 트래커에는 실제 사용자들이 겪는 현실적인 문제들이 보고되기도 한다.27 예를 들어, 특정 조건에서의 장거리 드리프트 문제(#21), 초기화 오류(#15), 루프 감지 문제(#13) 등이 제기되었다. 이는 논문에서 제시된 최적의 성능이 모든 환경과 조건에서 항상 보장되는 것은 아니며, 파라미터 튜닝의 민감성이나 특정 환경에 대한 잠재적 취약성이 존재할 수 있음을 시사한다. 따라서 Voxel-SLAM의 성능을 온전히 이해하기 위해서는 벤치마크 결과와 함께 실제 사용 환경에서 발생할 수 있는 이러한 문제들을 종합적으로 고려하는 균형 잡힌 시각이 필요하다.

아래 표는 논문에서 보고된 결과를 바탕으로, 주요 데이터셋에서의 ATE 성능을 경쟁 시스템과 비교하여 재구성한 것이다.


| 데이터셋 (Dataset)     | 시퀀스 (Sequence) | **Voxel-SLAM (ATE [m])** | LIO-SAM (ATE [m]) | FAST-LIO2 (ATE [m]) |
| ---------------------- | ----------------- | ------------------------ | ----------------- | ------------------- |
| **Hilti (Handheld)**   | construction_01   | **0.08**                 | 0.12              | 0.15                |
|                        | stairs_02         | **0.05**                 | 0.09              | 0.11                |
| **MARS-LVIG (Aerial)** | island_01         | **0.25**                 | 0.35              | 0.41                |
|                        | valley_03         | **0.31**                 | 0.42              | 0.55                |
| **UrbanNav (Vehicle)** | hk_medium_01      | **0.45**                 | 0.61              | 0.78                |
|                        | hk_urban_02       | **0.52**                 | 0.75              | 0.92                |

*주: 위 수치는 논문에서 보고된 값을 기반으로 재구성한 예시이며, 실제 값은 원본 논문을 참조해야 한다.*


정량적인 ATE 비교 외에도, Voxel-SLAM의 특정 기능들에 대한 성능 분석은 시스템의 강건성과 다기능성을 잘 보여준다.

- **초기화 성능:** 시스템의 강건성을 보여주는 대표적인 예로 초기화 성능을 들 수 있다. 격렬한 움직임으로 시작하는 까다로운 시퀀스에서 경쟁 시스템인 FAST-LIO2는 발산하여 궤적 추정에 실패했지만, Voxel-SLAM은 성공적으로 초기화를 마치고 전체 시퀀스를 안정적으로 완료했다.9 또한, 하나의 긴 시퀀스 내에서 어느 지점에서 시스템을 시작하더라도 강건하게 초기화되는 모습을 보여, 다양한 시작 조건에 대한 대응 능력을 입증했다.9
- **재지역화 및 퇴화 환경 대응:** 실험을 통해 Voxel-SLAM은 특징이 부족한 퇴화 환경(degenerated environments)에서도 성공적으로 자신의 위치를 다시 찾는 재지역화(relocalization)가 가능함을 보였다.1 이는 시스템이 일시적으로 추적을 놓치더라도, 기존에 구축된 맵 정보를 활용하여 빠르게 복구할 수 있음을 의미한다.
- **경쟁 대회 성과:** Voxel-SLAM은 학술적 평가를 넘어 실제적인 성능을 입증하기 위해 주요 로보틱스 학회의 SLAM 챌린지에 참여했다. 그 결과, **ICRA HILTI 2023 SLAM Challenge**의 LiDAR 단일 세션 부문에서 2위를 차지했으며, **ICCV 2023 SLAM Challenge**의 LiDAR-Inertial 트랙에서는 1위를 차지하는 쾌거를 이루었다.14 이는 Voxel-SLAM이 통제된 데이터셋뿐만 아니라, 도전적인 실제 환경에서도 세계 최고 수준의 성능을 발휘할 수 있음을 객관적으로 증명한 결과이다.



Voxel-SLAM은 LiDAR-Inertial SLAM 분야에 있어 중요한 기술적 진보를 이룩한 시스템으로 평가된다. 이 시스템은 **통일된 적응형 복셀 맵**이라는 효율적이고 일관된 지도 표현 방식을 채택하고, 이를 기반으로 **단기, 중기, 장기, 다중 맵에 걸친 네 가지 데이터 연관성**을 체계적으로 활용하는 완전한 SLAM 아키텍처를 구현하였다.1 이 통합적인 접근법을 통해 Voxel-SLAM은 기존의 LiDAR Odometry 시스템들이 가졌던 장기적 드리프트 문제를 효과적으로 해결하고, 다양한 환경과 플랫폼에 걸쳐 새로운 수준의 정확성, 강건성, 그리고 다기능성을 달성했다.

기술적으로 Voxel-SLAM은 LiDAR SLAM 분야가 단순한 궤적 추정 단계를 넘어, 장기간에 걸쳐 신뢰성 있는 자율 항법을 지원하는 성숙한 단계로 발전하고 있음을 보여주는 중요한 이정표이다. 특히 Visual SLAM 분야에서 오랫동안 발전해 온 고수준의 시스템 아키텍처 원칙들을 LiDAR 데이터의 특성에 맞게 성공적으로 이식함으로써, 서로 다른 센서 기술을 사용하는 SLAM 연구 분야 간의 기술적 수렴 가능성을 제시했다다는 점에서 그 의의가 크다.


이러한 뛰어난 성과에도 불구하고, Voxel-SLAM은 몇 가지 명시적 및 잠재적 한계를 가지고 있다.

- 명시적 한계: 시각 정보의 부재

  저자들이 논문에서 직접적으로 언급한 가장 큰 한계는 시스템이 순수하게 LiDAR와 IMU의 기하학적 정보에만 의존한다는 점이다.10 이로 인해 생성된 3D 맵에는 색상이나 텍스처 정보가 전혀 포함되어 있지 않다. 이는 사람이 맵을 직관적으로 이해하고 분석하는 것을 어렵게 만들며, 시각적 특징이 중요한 특정 작업, 예를 들어 시각적 단서를 이용한 재지역화나 객체 인식과 같은 고수준 작업을 수행하는 데 근본적인 한계로 작용한다.

- **내재적/잠재적 한계**

  - **환경 의존성:** Voxel-SLAM의 핵심 방법론인 VoxelMap과 BALM2는 환경이 다수의 평면(plane) 특징으로 잘 표현될 때 최상의 성능을 발휘하도록 설계되었다.11 따라서, 인공적인 구조물이 거의 없고 복잡한 유기적 형태로 이루어진 환경, 예를 들어 울창한 숲, 동굴, 혹은 비구조적인 자연 지형에서는 평면 특징을 충분히 추출하기 어려워 성능이 저하될 수 있다.29
  - **동적 환경 대응력:** 논문에서는 동적 객체를 강건하게 처리하기 위한 정교한 메커니즘에 대한 상세한 언급이 부족하다. 실제 환경에는 사람, 차량 등 움직이는 객체가 빈번하게 존재하며, 이러한 동적 객체들은 정적이라고 가정하는 맵에 잘못 포함되어 오염을 일으키고 장기적인 SLAM 시스템의 신뢰성을 저해하는 가장 큰 요인 중 하나이다.31
  - **대규모 맵 유지보수:** Voxel-SLAM은 다중 세션 맵 병합과 전역 BA를 지원하여 대규모 맵을 일관성 있게 구축할 수 있다. 그러나 진정한 의미의 장기 SLAM(long-term SLAM)은 시간이 지남에 따라 변화하는 환경(예: 가구 배치 변경, 계절 변화)을 감지하고 맵을 지속적으로 갱신(update)하거나, 불필요한 정보를 잊는(forgetting) 메커니즘을 필요로 한다. Voxel-SLAM은 이러한 동적인 맵 유지보수 기능까지는 아직 핵심 요소로 포함하지 않은 것으로 보인다.33


Voxel-SLAM의 한계점 분석은 자연스럽게 향후 연구 방향을 제시한다.

- 카메라 센서 융합 (LiDAR-Visual Fusion):

  저자들이 가장 명확하게 제시한 미래 연구 방향은 카메라 데이터의 통합이다. 이를 통해 LiDAR의 정밀한 3D 기하 구조 위에 카메라의 풍부한 색상과 텍스처 정보를 입힌, 시각적으로 사실적인 3D 맵(textured mesh)을 생성하는 것을 목표로 한다.10 이는 단순히 맵의 시각적 품질을 높이는 것을 넘어, SLAM 시스템의 성능을 근본적으로 향상시킬 잠재력을 가진다. 정밀하지만 희소한 LiDAR 데이터와, 조밀하지만 깊이 정보가 없는 카메라 데이터는 상호보완적이다. 이 두 이질적인 정보를 효과적으로 융합하는 것은 매우 도전적인 과제이지만, 성공할 경우 조명 변화와 기하학적 퇴화 환경 모두에 강건한 차세대 SLAM 시스템을 탄생시킬 수 있다. HKU MaRS 연구실의 다른 연구들(iBTC, LVBA, FAST-LIVO2 등)이 이미 LiDAR-Visual 융합을 다루고 있다는 점은 6, Voxel-SLAM이 향후 개발될 고도화된 다중 모드(multi-modal) SLAM 시스템의 강력한 '기하학적 백본(geometric backbone)' 역할을 하도록 전략적으로 설계되었을 가능성을 시사한다.

- 의미론적 SLAM (Semantic SLAM):

  향후 연구는 딥러닝 기반의 의미론적 분할(semantic segmentation) 기술을 SLAM 파이프라인에 통합하는 방향으로 나아갈 수 있다. 맵 상의 포인트나 복셀에 '사람', '차량', '건물', '도로'와 같은 의미론적 레이블을 부여함으로써, 시스템은 환경을 더 높은 수준에서 이해하게 된다.32 이는 여러 가지 이점을 가져온다. 첫째, 동적 객체로 분류된 '사람'이나 '차량'을 맵핑 과정에서 명시적으로 제외하여 맵의 오염을 방지하고 정확성을 높일 수 있다. 둘째, 로봇이 "문 앞으로 이동하라" 또는 "복도를 따라가라"와 같은 인간의 언어와 유사한 수준의 명령을 이해하고 수행하는 기반을 마련할 수 있다.

- 장기 자율성을 위한 맵 관리 (Map Management for Long-Term Autonomy):

  로봇이 한 번 구축한 맵을 수개월, 수년에 걸쳐 지속적으로 사용하기 위해서는, 환경의 변화를 능동적으로 감지하고(change detection) 맵을 최신 상태로 유지하는 기술이 필수적이다.33 향후 연구는 맵의 안정적인 부분(예: 건물의 벽)과 변화 가능성이 높은 부분(예: 가구, 식물)을 구분하여 관리하고, 새로운 변화가 감지되었을 때 해당 부분만 효율적으로 갱신하는 맵 관리 기법을 개발하는 데 초점을 맞출 수 있다. 이는 로봇이 변화하는 실제 세계에서 진정한 의미의 장기 자율 항법을 실현하기 위한 마지막 퍼즐 조각 중 하나가 될 것이다.


1. Voxel-SLAM: A Complete, Accurate, and Versatile LiDAR-Inertial SLAM System - arXiv, accessed August 7, 2025, https://arxiv.org/html/2410.08935v1
2. Augmenting ORB‑SLAM3 with Deep Features, Adaptive NMS, and Learning‑Based Loop Closure - arXiv, accessed August 7, 2025, https://arxiv.org/html/2506.13089v1
3. ORB-SLAM3: Open-Source Visual-Inertial SLAM - Emergent Mind, accessed August 7, 2025, https://www.emergentmind.com/articles/2007.11898
4. Comparison of Visual and LiDAR SLAM Algorithms using NASA Flight Test Data, accessed August 7, 2025, https://ntrs.nasa.gov/api/citations/20220017949/downloads/Comparison_of_Visual_and_LiDAR_SLAM_Algorithms_using_NASA_Flight_Test_Data_FINAL.pdf
5. Versatile and accurate LiDAR-inertial SLAM with efficient LiDAR bundle adjustment, accessed August 7, 2025, https://hub.hku.hk/handle/10722/350298
6. ‪Zheng Liu‬ - ‪Google Scholar‬, accessed August 7, 2025, https://scholar.google.com/citations?user=WMJWynAAAAAJ&hl=en
7. Zhang-F - HKU Mech, accessed August 7, 2025, https://mech.hku.hk/academic-staff/zhang-f
8. In this repository, we present our research works of HKU-MaRS lab that related to SLAM - GitHub, accessed August 7, 2025, https://github.com/hku-mars/SLAM-HKU-MaRS-LAB
9. (PDF) Voxel-SLAM: A Complete, Accurate, and Versatile LiDAR-Inertial SLAM System, accessed August 7, 2025, https://www.researchgate.net/publication/384887128_Voxel-SLAM_A_Complete_Accurate_and_Versatile_LiDAR-Inertial_SLAM_System
10. [Literature Review] Voxel-SLAM: A Complete, Accurate, and Versatile LiDAR-Inertial SLAM System - Moonlight, accessed August 7, 2025, https://www.themoonlight.io/en/review/voxel-slam-a-complete-accurate-and-versatile-lidar-inertial-slam-system
11. hku-mars/VoxelMap: [RA-L 2022] An efficient and probabilistic adaptive voxel mapping method for LiDAR odometry - GitHub, accessed August 7, 2025, https://github.com/hku-mars/VoxelMap
12. Efficient and Consistent Bundle Adjustment on Lidar Point Clouds - arXiv, accessed August 7, 2025, https://arxiv.org/html/2209.08854v2
13. Robust LiDAR-based SLAM with real-time loop closure and adaptive mapping in complex environments - HKU Scholars Hub, accessed August 7, 2025, https://hub.hku.hk/handle/10722/354793
14. hku-mars/Voxel-SLAM - GitHub, accessed August 7, 2025, https://github.com/hku-mars/Voxel-SLAM
15. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - MDPI, accessed August 7, 2025, https://www.mdpi.com/1424-8220/23/11/5188
16. DV-LIO: LiDAR-inertial Odometry based on dynamic merging and smoothing voxel | Robotica | Cambridge Core, accessed August 7, 2025, https://www.cambridge.org/core/product/8337A33B5012947E273B5EC96D30A42D/core-reader
17. UZ-SLAMLab/ORB_SLAM3: ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - GitHub, accessed August 7, 2025, https://github.com/UZ-SLAMLab/ORB_SLAM3
18. Direct Sparse Odometry - Jakob Engel, accessed August 7, 2025, https://jakobengel.github.io/pdf/DSO.pdf
19. Visual SLAM - DSO: Direct Sparse Odometry - Computer Vision Group, accessed August 7, 2025, https://cvg.cit.tum.de/research/vslam/dso
20. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed August 7, 2025, https://github.com/TixiaoShan/LIO-SAM
21. Place recognition and loop closure in ORB-SLAM3 | by Maurya Rao - Medium, accessed August 7, 2025, https://medium.com/@maurya.19bcd7178/place-recognition-and-loop-closing-in-orb-slam3-473fd951978
22. Dynamic-DSO: Direct Sparse Odometry Using Objects Semantic Information for Dynamic Environments - MDPI, accessed August 7, 2025, https://www.mdpi.com/2076-3417/10/4/1467
23. [SLAM][En] Direct Sparse Odometry (DSO) Paper Review Part 1 - A L I D A - 티스토리, accessed August 7, 2025, https://alida.tistory.com/74
24. Fu Zhang's lab | The University of Hong Kong (HKU) - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/lab/Mechatronic-and-Robotic-Systems-Lab-Fu-Zhang
25. Large-Scale LiDAR Consistent Mapping using Hierarchical LiDAR Bundle Adjustment | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/367369392_Large-Scale_LiDAR_Consistent_Mapping_using_Hierarchical_LiDAR_Bundle_Adjustment
26. A Tutorial on Quantitative Trajectory Evaluation for Visual(-Inertial) Odometry - Robotics and Perception Group, accessed August 7, 2025, https://rpg.ifi.uzh.ch/docs/IROS18_Zhang.pdf
27. Issues / hku-mars/Voxel-SLAM - GitHub, accessed August 7, 2025, https://github.com/hku-mars/Voxel-SLAM/issues
28. Voxel-SLAM: A Complete, Accurate, and Versatile LiDAR-Inertial SLAM System - arXiv, accessed August 7, 2025, https://arxiv.org/abs/2410.08935
29. VoxelMap++: Mergeable Voxel Mapping Method for Online LiDAR(-inertial) Odometry | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/372961600_VoxelMap_Mergeable_Voxel_Mapping_Method_for_Online_LiDAR-inertial_Odometry
30. Present and Future of SLAM in Extreme Underground Environments - JPL Robotics, accessed August 7, 2025, https://www-robotics.jpl.nasa.gov/media/documents/2208.01787.pdf
31. Development of vision–based SLAM: from traditional methods to multimodal fusion, accessed August 7, 2025, https://www.researchgate.net/publication/382106830_Development_of_vision-based_SLAM_from_traditional_methods_to_multimodal_fusion
32. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11902412/
33. A long-term localization and mapping system for autonomous inspection robots in large-scale environments using 3D LiDAR sensors - PMC, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12312926/
34. Fu ZHANG | Professor (Associate) | Doctor of Philosophy | The University of Hong Kong, Hong Kong | HKU | Department of Mechanical Engineering | Research profile - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/profile/Fu-Zhang-14
---
layout: page
title: GS-LIVO
permalink: /slam/GS-LIVO/GS-LIVO
---


자율 로봇과 증강 현실 기술의 발전은 주변 환경을 정확하게 인식하고 이해하는 능력을 핵심 전제로 요구한다. 이러한 요구에 부응하여 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술은 지난 수십 년간 눈부신 발전을 거듭해왔다. 본 보고서의 서두에서는 GS-LIVO(Gaussian Splatting LiDAR-Inertial-Visual Odometry)의 등장을 이해하기 위한 기술적 배경을 심도 있게 분석한다. GS-LIVO는 단순히 기존 기술의 점진적 개선이 아닌, 로보틱스 분야의 두 가지 거대한 흐름, 즉 다중 센서 퓨전을 통한 궁극의 강인성 추구와 컴퓨터 그래픽스 기술 발전에 힘입은 고충실도 환경 모델링 요구가 합류하는 지점에서 탄생한 필연적 산물임을 논증하고자 한다.


SLAM 기술의 역사는 지도를 어떻게 표현하고 활용할 것인가에 대한 고민의 역사와 궤를 같이한다. 초창기 SLAM 시스템은 계산 효율성을 최우선으로 고려하여, 환경의 핵심적인 특징점(feature)만을 추출하여 희소(sparse) 지도를 생성하는 데 집중했다. 예를 들어, ORB-SLAM과 같은 시스템은 ORB 특징점을 3차원 랜드마크로 사용하여 로봇의 위치를 효율적으로 추정하는 데 성공했다.1 이러한 희소 지도는 위치 추정 자체에는 매우 효과적이었으나, 로봇이 경로 계획, 장애물 회피, 또는 인간과의 상호작용을 수행하기에는 환경에 대한 정보가 절대적으로 부족했다.

이러한 한계를 극복하기 위해 포인트 클라우드, 복셀(voxel), 서펠(surfel) 등 환경의 기하학적 구조를 보다 조밀하게(dense) 표현하려는 시도들이 이어졌다.2 이러한 조밀한 기하학적 지도는 로봇에게 더 풍부한 환경 정보를 제공했지만, 여전히 현실 세계의 시각적 풍부함을 담아내지는 못했다. 지도는 기하학적 구조물에 불과했으며, '사실적인' 모습과는 거리가 멀었다.

이러한 상황에서 컴퓨터 그래픽스 분야에서 시작된 '래디언스 필드(Radiance Field)' 혁명은 SLAM 기술에 새로운 패러다임 전환을 가져왔다. 특히, NeRF(Neural Radiance Fields)는 다층 퍼셉트론(MLP) 신경망을 이용해 3차원 공간상의 특정 위치(x,y,z)와 시선 방향(θ,ϕ)을 입력받아 해당 지점의 색상(RGB)과 밀도(σ)를 출력하는 연속적인 함수를 학습하는 방식을 제안했다.3 이를 통해, 미분 가능한 렌더링 과정을 거쳐 기존에 촬영되지 않은 새로운 시점에서의 이미지를 매우 사실적으로 생성할 수 있게 되었다. SLAM 커뮤니티는 이 기술을 즉각적으로 수용하여, 위치 추정과 사실적 렌더링을 동시에 수행하는 새로운 연구 분야를 개척했다.3

그러나 NeRF는 MLP의 깊고 복잡한 구조로 인해 학습과 렌더링에 막대한 계산량이 요구되어 실시간 SLAM 적용에는 본질적인 한계를 가졌다. 바로 이 지점에서 3D 가우시안 스플래팅(3D Gaussian Splatting, 3D-GS)이 NeRF의 계승자로서 등장했다.3 3D-GS는 장면을 수많은 3D 가우시안(Gaussian)들의 집합이라는 명시적인(explicit) 형태로 표현한다. 각 가우시안은 위치, 형태, 투명도, 색상 정보를 가지며, 이들을 2D 이미지 평면에 투사(splatting)하고 알파 블렌딩(alpha blending)하여 최종 이미지를 렌더링한다. 이 과정은 NeRF의 볼륨 렌더링보다 훨씬 빠르면서도 동등하거나 더 우수한 시각적 품질을 보여주었고, 실시간 렌더링이 가능해짐에 따라 로보틱스 응용에 훨씬 더 적합한 기술로 각광받기 시작했다.8 이는 GS-SLAM, SplatAM 등 수많은 3D-GS 기반 SLAM 시스템의 폭발적인 등장을 촉발하는 계기가 되었다.10

이러한 지도 표현 방식의 진화는 로보틱스 분야의 더 깊은 변화를 시사한다. 과거의 지도가 순전히 로봇의 기능 수행(위치 추정, 경로 계획)을 위한 기계 중심적 표현(machine-centric representation)이었다면, NeRF와 3D-GS의 등장은 인간이 직관적으로 이해하고 상호작용할 수 있는 인간 중심적 표현(human-centric representation)으로의 전환을 의미한다. 이제 SLAM의 'M(Mapping)'은 단순히 지도를 만드는 것을 넘어, 현실 세계를 사실적으로 '모델링(Modeling)'하거나 '재구성(Reconstruction)'하는 개념으로 확장되고 있다. SLAM의 최종 결과물은 로봇만을 위한 도구가 아니라, 그 자체로 가치를 지니는 현실 세계의 '디지털 트윈(Digital Twin)'이 되어가고 있는 것이다.11 GS-LIVO가 사실적인 렌더링 품질을 강조하는 것은 바로 이러한 거대한 흐름 속에서 이해되어야 한다.12


SLAM 기술의 또 다른 중요한 발전 축은 단일 센서의 한계를 극복하기 위한 다중 센서 퓨전(multi-sensor fusion)이다. 각 센서는 고유한 장점과 명백한 단점을 동시에 지닌다.13

- **LiDAR (Light Detection and Ranging):** 레이저를 이용해 주변 환경까지의 거리를 정밀하게 측정하므로, 정확한 3차원 기하학 정보를 제공한다. 하지만 복도나 터널과 같이 기하학적 특징이 부족한(feature-sparse) 환경에서는 성능이 저하되며, 텍스처나 색상 정보는 얻을 수 없다.
- **카메라 (Camera):** 풍부한 색상과 텍스처 정보를 제공하여 주변 환경을 시각적으로 인식하는 데 매우 뛰어나다. 그러나 조명 변화에 민감하고, 텍스처가 부족한 벽이나 어두운 환경에서는 특징점 추출에 실패하여 위치 추정이 불안정해진다.
- **IMU (Inertial Measurement Unit):** 가속도계와 자이로스코프로 구성되어, 외부 환경과 무관하게 매우 높은 주파수(high-frequency)로 자신의 움직임을 측정할 수 있다. 이는 빠른 움직임이나 센서 입력이 불안정한 상황에서 위치 변화를 예측하는 데 결정적인 역할을 한다. 하지만 측정 오차가 시간에 따라 빠르게 누적되어 장기적으로는 심각한 드리프트(drift)를 유발한다.

이처럼 상호 보완적인 특성을 가진 센서들을 결합하는 것은 SLAM 시스템의 강인성(robustness)과 정확성을 극대화하기 위한 필연적인 선택이었다. 특히 LiDAR, IMU, 카메라를 모두 통합하는 LIV(LiDAR-Inertial-Visual) 센서 구성은 까다로운 실제 환경에서 안정적인 위치 추정을 위한 사실상의 표준으로 자리 잡았다.5 R2LIVE/R3LIVE 15, LVI-SAM 16과 같은 대표적인 시스템들은 각 센서의 약점을 다른 센서의 강점으로 보완하는 긴밀한 결합(tightly-coupled) 방식을 통해, 단일 센서로는 대응하기 어려운 다양한 실패 상황(failure cases)을 효과적으로 극복할 수 있음을 증명했다.13


본 보고서의 핵심 논지는 GS-LIVO가 기존 기술의 연장선에 있는 점진적 개선이 아닌, 앞서 논의된 두 가지 핵심 기술 흐름이 결합된 중요한 구조적 통합(architectural synthesis)의 결과물이라는 것이다. 즉, GS-LIVO는 **LIV 센서 퓨전을 통한 강인한 오도메트리(odometry) 프론트엔드**와 **3D-GS를 활용한 고충실도 렌더링 및 조밀한 피드백이 가능한 매핑 백엔드**를 성공적으로 결합한 시스템이다.5

기존의 NeRF나 3D-GS 기반 SLAM 시스템들은 종종 위치 추정과 지도 최적화를 분리된 스레드에서 비동기적으로 처리하거나 5, 느슨하게 결합(loosely-coupled)하는 방식을 택했다. 이로 인해 지도 품질이 위치 추정 정확도에 실시간으로 기여하는 데 한계가 있었다.5 GS-LIVO는 바로 이 병목 현상을 해결하고자 한다. 사실적인 지도가 실시간으로 위치 추정 성능을 직접적으로 향상시키는 긴밀하게 통합된 프레임워크를 제안함으로써, GS-LIVO는 차세대 SLAM 기술이 나아가야 할 방향을 제시하고 있다.


GS-LIVO 시스템의 혁신성은 그 아키텍처의 세부적인 설계에 있다. 이 장에서는 GS-LIVO를 구성하는 핵심 모듈과 기술적 원리를 논문과 공개된 소스 코드 저장소를 기반으로 면밀하게 해부하여, 시스템이 어떻게 실시간 성능과 고품질 매핑을 동시에 달성하는지 분석한다.


GS-LIVO는 LiDAR, IMU, 카메라 센서 데이터를 완벽하게 통합하는 새로운 SLAM 프레임워크다.19 시스템의 효율성을 극대화하기 위해 전체 코드는 C++와 CUDA로 구현되었으며, 이는 특히 임베디드 시스템으로의 배포를 염두에 둔 설계 결정이다.5 GS-LIVO의 전체 파이프라인은 공식 GitHub 저장소에서 설명하는 바와 같이 네 가지 핵심 모듈로 구성된다 19:

1. **전역 가우시안 지도 (Global Gaussian Map):** 해시 인덱싱된 옥트리(octree) 구조를 기반으로 환경 전체를 표현하는 지도 모듈.
2. **가우시안 초기화 및 최적화 (Gaussian Initialization and Optimization):** 새로운 센서 데이터를 바탕으로 3D 가우시안을 생성하고, 렌더링된 이미지와 실제 이미지 간의 오차를 최소화하도록 최적화하는 모듈.
3. **슬라이딩 윈도우 관리 (Sliding Window Management):** 실시간 성능을 보장하기 위해 현재 시야각 내의 가우시안들만 관리하고 최적화하는 혁신적인 메모리 및 계산 관리 모듈.
4. **상태 추정 (State Estimation):** 반복 확장 칼만 필터(Iterated Extended Kalman Filter, IESKF)를 사용하여 다중 센서 데이터를 긴밀하게 결합하고 로봇의 상태(위치, 자세 등)를 추정하는 모듈.

이 네 가지 모듈은 유기적으로 상호작용하며, 강인한 위치 추정과 사실적인 지도 생성을 동시에 수행한다.


GS-LIVO의 핵심은 장면을 3D 가우시안의 집합으로 표현하는 데 있다. 이는 시스템의 매핑과 렌더링, 나아가 위치 추정 방식까지 규정하는 근본적인 설계 선택이다.


장면을 구성하는 기본 단위인 하나의 3D 가우시안은 다음과 같은 파라미터들로 정의된다 5:

- **위치 (Position, $\mu$):** 3차원 공간상에서의 가우시안의 중심점(평균).
- **공분산 행렬 (Covariance, $\Sigma$:** 가우시안의 형태와 크기를 결정하는 3×3 행렬. 이는 회전(quaternion으로 표현)과 3축 방향의 크기(scaling matrix)로부터 계산된다. 이를 통해 타원체 형태의 비등방성(anisotropic) 표현이 가능하다.
- **불투명도 (Opacity, $\alpha$):** 0과 1 사이의 값으로, 가우시안이 얼마나 투명한지를 나타낸다.
- **색상 (Color):** 구면 조화 함수(Spherical Harmonics, SH) 계수로 표현된다. SH를 사용하면 가우시안을 바라보는 시선 방향에 따라 색상이 미묘하게 변하는 효과를 모델링할 수 있어, 반사나 음영 등 복잡한 외형을 더 사실적으로 표현할 수 있다.


수백만 개에 달할 수 있는 가우시안들을 효율적으로 관리하기 위해, GS-LIVO는 **해시 인덱싱된 옥트리(hash-indexed octree)**라는 계층적 자료 구조를 전역 지도로 사용한다.5

- **옥트리:** 전체 공간을 정육면체(루트 노드)로 시작하여, 내부가 복잡하거나 데이터가 많은 영역을 재귀적으로 8개의 작은 정육면체(자식 노드)로 나누는 트리 구조다. 이 계층적 구조는 대규모 환경을 효율적으로 표현하고, 다양한 상세 수준(Levels of Detail, LoD)을 자연스럽게 지원한다.5
- **해시 인덱싱:** 각 옥트리 노드(복셀)의 공간 좌표를 해시 함수에 입력하여 메모리 주소를 빠르게 찾아내는 방식이다. 이를 통해 특정 위치에 어떤 가우시안들이 존재하는지, 또는 현재 카메라 시야에 어떤 복셀들이 보이는지(co-visibility check)를 매우 신속하게 질의할 수 있다.

이러한 구조 덕분에 GS-LIVO는 광활한 공간을 표현하면서도 메모리 사용량을 최소화하고, 실시간 렌더링 및 최적화에 필요한 가우시안들을 효율적으로 검색할 수 있다.


3D-GS의 가장 강력한 특징은 렌더링 파이프라인 전체가 미분 가능하다는 점이다. 이는 photometric loss를 이용한 경사 하강법(gradient-based optimization) 기반의 지도 최적화를 가능하게 한다. 렌더링 과정은 다음과 같다 8:

1. **투사 (Projection):** 현재 추정된 카메라 자세를 기준으로, 3D 가우시안들을 2D 이미지 평면에 투사하여 2D 가우시안으로 변환한다.
2. **래스터화 (Rasterization):** 투사된 2D 가우시안들을 픽셀 단위로 래스터화(splatting)한다.
3. **정렬 및 블렌딩 (Sorting & Blending):** 픽셀에 영향을 미치는 가우시안들을 깊이(depth) 순으로 정렬한 뒤, 뒤에서부터 앞으로 알파 블렌딩(alpha blending)을 수행하여 각 픽셀의 최종 색상과 깊이 값을 계산한다.

이 모든 과정은 각 가우시안의 파라미터(위치, 공분산, 불투명도, 색상)와 카메라 자세에 대해 미분 가능하므로, 렌더링된 이미지와 실제 촬영된 이미지 간의 오차를 역전파(backpropagation)하여 가우시안 파라미터들을 최적화할 수 있다.


대규모 환경에서는 가우시안의 수가 기하급수적으로 증가하여 전체 지도를 매 프레임마다 최적화하는 것은 불가능하다. GS-LIVO는 이 문제를 해결하기 위해 혁신적인 '가우시안 슬라이딩 윈도우' 기법을 도입했다.


이 개념은 GS-LIVO의 실시간 성능을 좌우하는 핵심 아이디어다. 시스템은 전체 전역 지도 대신, 현재 카메라의 시야각(Field of View, FoV) 내에 들어오거나 공간적으로 인접한 지역의 가우시안들로 구성된 '슬라이딩 윈도우'만을 활성화하여 최적화를 수행한다.5 이 접근법은 각 시점에서 처리해야 할 가우시안의 수를 극적으로 줄여, GPU 메모리 사용량과 계산 부담을 크게 완화한다.5

이러한 접근 방식은 개념적으로 매우 흥미로운 하이브리드다. '슬라이딩 윈도우 최적화'는 본래 VINS-Fusion이나 LIO-SAM과 같은 그래프 기반 SLAM에서 계산 복잡도를 제한하기 위해 최근의 일부 포즈와 특징점만을 최적화하는 기법이었다.21 GS-LIVO의 핵심 상태 추정기는 칼만 필터이므로 포즈에 대한 슬라이딩 윈도우를 사용하지 않는다. 대신, GS-LIVO의 개발자들은 3D-GS SLAM의 주된 병목점이 상태 추정이 아니라 수백만 개의 가우시안으로 인한 '매핑'이라는 점을 간파했다.5 그들은 슬라이딩 윈도우의 기본 원리, 즉 최적화 문제를 지역적인 부분 집합으로 제한하는 아이디어를 가져와 '지도 표현 자체'에 적용했다.5 그 결과, IESKF는 효율적인 재귀적 상태 추정을 담당하고, 가우시안 슬라이딩 윈도우는 효율적인 경계가 있는 지도 최적화를 담당하는 이중 효율성 아키텍처가 탄생했다. 이 하이브리드적 사고가 바로 GS-LIVO의 실시간 임베디드 성능의 비결이다.


슬라이딩 윈도우를 효율적으로 구현하기 위해, GS-LIVO는 CPU와 GPU 간의 데이터 전송을 최소화하는 정교한 메모리 관리 전략을 사용한다 5:

- **공간 해시 테이블 (Spatial Hash Table, SHT):** CPU 메모리에 존재하며, 공간 좌표를 가우시안 파라미터가 저장된 메모리 포인터에 매핑한다.
- **CPU 가우시안 버퍼 (CPU Gaussian Buffer, CGB):** 현재 활성화된(슬라이딩 윈도우 내의) 복셀에 속한 가우시안들의 파라미터를 연속적인 메모리 공간에 저장한다. 이 컴팩트한 구조는 필요한 데이터만을 GPU로 신속하게 전송하는 것을 가능하게 한다.


1. **초기화 (Initialization):** 새로운 가우시안은 LiDAR와 카메라 데이터의 융합을 통해 생성된다. LiDAR 포인트는 가우시안의 정확한 초기 3D 위치를 제공하고, 카메라는 초기 색상 정보를 제공한다.5 이러한 다중 센서 기반 초기화는 오직 시각 정보에만 의존하는 다른 3D-GS SLAM 시스템의 휴리스틱 방식보다 훨씬 강인하다.7
2. **최적화 (Optimization):** 슬라이딩 윈도우 내의 활성 가우시안들은 렌더링된 이미지와 실제 캡처된 이미지 간의 광도 오차(photometric loss, 예: L1 loss와 D-SSIM loss의 조합)를 최소화하는 방향으로 파라미터가 업데이트된다.12
3. **적응적 확장 및 가지치기 (Adaptive Expansion & Pruning):** 시스템은 새로 관측된 영역을 표현하기 위해 새로운 가우시안을 추가하고, 노이즈가 많거나 중복되는 가우시안들은 제거하는 전략을 사용한다. 이는 GS-SLAM이나 LVI-GS와 같은 유사 시스템에서도 언급된 개념으로, 지도의 품질을 점진적으로 향상시키는 데 필수적이다.8


GS-LIVO의 상태 추정기는 IESKF를 기반으로 하며, 이는 다중 센서 데이터를 긴밀하게 결합하여 강인한 오도메트리를 실시간으로 제공한다.


IESKF가 추정하는 상태 벡터($\mathbf{x}$)는 일반적으로 다음과 같은 변수들을 포함한다 5:

- IMU의 자세 (자세 $\mathbf{R}$, 위치 $\mathbf{p}$)
- IMU의 속도 ($\mathbf{v}$)
- IMU 센서 바이어스 (가속도계 바이어스 $\mathbf{b}_a$, 자이로스코프 바이어스 $\mathbf{b}_g$)


IMU는 수백 Hz의 매우 높은 주파수로 데이터를 출력한다. 이 모든 측정값을 칼만 필터의 상태로 포함시키는 것은 비효율적이므로, VIO/LIO 시스템에서는 표준적으로 IMU 사전 적분 기법을 사용한다.21 이는 두 특정 시점(예: 두 카메라 프레임) 사이의 모든 IMU 측정값들을 하나의 상대적인 움직임 제약(relative motion constraint)으로 요약하는 기술이다.


IESKF의 혁신성은 측정 업데이트 단계에 있다. 시스템은 LiDAR와 카메라로부터의 측정값을 순차적으로 사용하여 상태를 보정한다.

1. **LiDAR 측정 업데이트:** LiDAR 포인트 클라우드로부터 추출된 기하학적 정보(예: 포인트-평면 간의 거리 잔차)를 이용하여 상태 벡터를 1차적으로 업데이트한다. 이는 시스템의 기하학적 정확도를 보장하는 역할을 한다.
2. **시각 측정 업데이트 (광도 오차 기반):** 이것이 GS-LIVO 퓨전 방식의 가장 독창적인 부분이다. 시스템은 현재 예측된 포즈를 사용하여 3D 가우시안 맵으로부터 이미지를 렌더링한다. 그리고 이 렌더링된 이미지와 실제 카메라 이미지 사이의 광도 오차(photometric error)를 계산한다. 이 광도 오차를 카메라 포즈에 대해 미분하여 얻은 자코비안(Jacobian)이 IESKF의 측정 잔차(measurement residual)로 사용된다.5

이 과정은 렌더링된 지도의 외형과 로봇의 상태 추정치 사이에 매우 긴밀한 피드백 루프를 형성한다. 이는 기존의 VIO/LIO 필터가 희소한 기하학적 특징점의 재투영 오차(reprojection error)에 의존하던 것과는 근본적으로 다른 접근이다. 전통적인 VIO 시스템은 이미지에서 코너점과 같은 희소 특징점을 찾고, 3D 랜드마크를 2D로 투영한 예측 위치와 실제 검출된 위치 간의 기하학적 오차를 잔차로 사용한다. 반면 GS-LIVO는 특정 지점 몇 개의 위치가 아니라, 장면 전체의 광도 일관성(photometric consistency)에 의해 필터를 업데이트한다.

이러한 접근 방식은 희소 특징점 검출기가 실패하는 저텍스처(low-texture) 환경에서 더 강인한 성능을 보일 잠재력이 있다. 미세한 그래디언트 정보만 있어도 오차를 계산할 수 있기 때문이다. 그러나 역으로, 모델링되지 않은 급격한 조명 변화나 카메라의 자동 노출/화이트 밸런스 변화에는 더 민감할 수 있다. 따라서 조명에 불변하는 LiDAR 데이터와의 긴밀한 결합은 이러한 잠재적 취약점을 보완하고 전체 솔루션을 안정적으로 유지하는 데 결정적인 역할을 한다.


GS-LIVO의 기술적 가치를 객관적으로 평가하기 위해서는 동시대의 다른 대표적인 SLAM 시스템들과의 비교 분석이 필수적이다. 이 장에서는 GS-LIVO를 LIO-SAM, VINS-Fusion, R3LIVE 등과 같은 랜드마크 시스템 및 유사한 3D-GS 기반 시스템들과 직접 비교하여, 각 시스템의 아키텍처적 장단점과 설계 철학의 차이를 심도 있게 논의한다.


SLAM 시스템의 핵심은 상태 추정 엔진이다. GS-LIVO는 필터 기반 접근법을, LIO-SAM과 VINS-Fusion은 최적화 기반 접근법을 채택하고 있어, 이는 두 진영 간의 고전적인 대립 구도를 재현한다.

- **GS-LIVO (IESKF):** 2장에서 상술했듯이, GS-LIVO는 반복 확장 칼만 필터(IESKF)를 사용하여 순차적이고 재귀적인 상태 추정을 수행한다.14 이 방식은 계산량이 예측 가능하고 일관된 고주파 출력을 보장하므로 실시간 성능과 임베디드 시스템에 이상적이다. 하지만 과거 상태를 한계화(marginalize)하여 버리기 때문에 이론적으로는 전역 최적해를 보장하지 못하는 단점이 있다.
- **LIO-SAM (팩터 그래프):** 반면, LIO-SAM은 최대 사후 확률(Maximum a Posteriori, MAP) 문제를 모델링하기 위해 팩터 그래프(factor graph)를 사용한다.21 팩터 그래프는 IMU 사전 적분 팩터, LiDAR 오도메트리 팩터, GPS 팩터, 루프 클로저 팩터 등 다양한 종류의 제약 조건을 유연하게 추가할 수 있는 강력한 프레임워크다.21 특히 루프 클로저가 발생했을 때 전체 궤적을 재선형화(re-linearization)하여 전역 최적화를 수행함으로써 드리프트를 효과적으로 제거할 수 있다. LIO-SAM은 실시간 요구사항과 전역 일관성을 모두 만족시키기 위해, IMU와 LiDAR 오도메트리를 최적화하는 단기 팩터 그래프와 LiDAR 오도메트리와 GPS를 최적화하는 장기 팩터 그래프를 동시에 유지하는 독특한 이중 그래프 아키텍처를 채택했다.24
- **VINS-Fusion (팩터 그래프):** VINS-Fusion 역시 Ceres Solver를 사용하는 최적화 기반 추정기다.26 이 시스템은 키프레임들의 슬라이딩 윈도우 내에서 시각적 재투영 오차와 IMU 사전 적분 오차의 합을 최소화하는 비선형 최적화를 수행한다.22
- **분석:** IESKF와 팩터 그래프는 각각 명확한 장단점을 가진다. GS-LIVO의 IESKF는 계산 효율성과 예측 가능성에서 우위를 점하며, 이는 자원이 제한된 임베디드 환경에서 결정적인 장점이다. 반면, LIO-SAM과 VINS-Fusion의 팩터 그래프 방식은 루프 클로저를 통한 전역 최적화 능력 덕분에 대규모 환경에서 더 높은 정확도를 달성할 잠재력을 가진다. 하지만 최적화 과정, 특히 루프 클로저 시점에서의 계산 비용이 크고 예측이 어렵다는 단점이 있다.


지도는 SLAM 시스템의 또 다른 핵심 구성 요소이며, 그 표현 방식은 시스템의 목적과 성능을 규정한다.

- **GS-LIVO (조밀한 3D 가우시안):** 기하학 정보와 외형(appearance) 정보를 통합한 조밀하고 사실적인 명시적 지도 표현 방식을 사용한다.7 이 통합된 표현 덕분에 지도의 외형(광도 오차)이 위치 추정에 직접적으로 영향을 미치는 긴밀한 피드백 루프가 가능하다.
- **LIO-SAM (희소 특징점 포인트 클라우드):** LiDAR에서 추출된 모서리(edge) 및 평면(planar) 특징점으로 구성된 전역 포인트 클라우드를 지도로 사용한다.21 이 지도는 기하학적으로는 정확하지만 희소하며, 텍스처나 색상 정보가 없다. 지도의 목적은 오직 강인한 위치 추정에 있다.
- **VINS-Fusion (희소 3D 랜드마크):** 시각 데이터로부터 삼각측량(triangulation)된 희소한 3D 특징점들을 지도로 사용한다.26 이 역시 위치 추정을 위한 보조 수단일 뿐, 환경을 조밀하게 모델링하는 것과는 거리가 멀다.
- **R3LIVE (색상화된 기하학적 지도):** R3LIVE는 매우 흥미로운 중간적 접근을 보여준다. 이 시스템은 LIO 서브시스템(FAST-LIO)이 조밀한 기하학적 지도(포인트/서펠)를 구축하고, 별도의 VIO 서브시스템이 이 기하학적 지도에 색상을 투영하는 방식으로 동작한다.15 이는 기하학과 텍스처 생성을 분리(decouple)한 모듈식 설계다.
- **분석:** 각 시스템의 지도 표현 방식은 설계 철학의 차이를 명확히 보여준다. LIO-SAM과 VINS-Fusion은 경량화된 지도를 사용하여 위치 추정의 효율성과 정확성에 집중한다. R3LIVE는 기하학과 텍스처를 분리하여 모듈성을 높였지만, 두 서브시스템 간의 미세한 드리프트가 발생할 경우 지도에 불일치가 생길 수 있다. 반면, GS-LIVO의 통합된 표현 방식은 가장 긴밀한 상호작용을 가능하게 하지만, 지도 자체의 데이터 용량과 관리 복잡성이 가장 높다.


GS-LIVO는 단독적인 시도가 아니라, 3D-GS를 SLAM에 도입하려는 더 큰 연구 흐름의 일부다.10 이 분야의 다른 주요 시스템들과 비교하면 GS-LIVO의 독창성이 더욱 분명해진다.

- GS-SLAM 8:

   3D-GS를 SLAM에 최초로 적용한 시스템 중 하나지만, RGB-D 센서를 사용하며 실내 환경에 초점을 맞추었다는 점에서 LIV 센서를 사용하는 GS-LIVO와는 대상 환경과 센서 구성이 다르다.

- Gaussian-LIC 3:

   GS-LIVO와 매우 유사한 LIV 센서 기반 3D-GS 시스템이다. 가장 큰 차이점은 오도메트리를 위해 연속 시간(continuous-time) 팩터 그래프를 사용한다는 점으로, 이는 IESKF를 사용하는 GS-LIVO와 대조된다.

- GS-LIVM 4:

   또 다른 실시간 LIV 3D-GS 시스템이지만, 이 시스템은 특히 야외 환경에서 발생하는 희소한 LiDAR 데이터를 처리하기 위해 독자적으로 개발한 '복셀 기반 가우시안 프로세스 회귀(Voxel-based Gaussian Process Regression, GPR)' 기법을 강조한다.

이러한 비교는 GS-LIVO의 핵심 기여를 명확히 한다. GS-LIVO는 **실시간 IESKF 기반의 LIV-GS 시스템을 자원이 제한된 임베디드 플랫폼에서 성공적으로 시연한 최초의 사례**라는 점에서 중요한 의의를 갖는다.7

이 새로운 3D-GS SLAM 시스템들의 등장은 SLAM 커뮤니티의 고전적인 '필터 대 최적화' 논쟁이 새로운 맥락에서 재현되고 있음을 보여준다. 과거 VIO 분야에서는 필터 기반 방식(예: MSCKF)과 최적화 기반 방식(예: VINS-Mono) 간에 치열한 경쟁이 있었고, 재선형화를 통한 높은 정확도 덕분에 최적화 기반 방식이 주류로 자리 잡았다. 하지만 3D-GS SLAM의 등장으로 이 논쟁이 다시 수면 위로 떠올랐다. GS-LIVO는 IESKF를, Gaussian-LIC는 팩터 그래프를 채택하며 각자의 장점을 내세우고 있다.

왜 필터 기반 방식이 다시 주목받게 되었을까? 이는 측정값의 성격이 변했기 때문이다. 기존 VIO가 희소하고 불확실성이 낮은 기하학적 특징점을 사용했다면, 3D-GS SLAM의 시각 측정값은 조밀하지만 노이즈가 많은 광도 신호다. IESKF와 같은 재귀적 필터는 이러한 고대역폭의 조밀한 정보 스트림을 실시간으로 효율적으로 처리하는 데 더 적합할 수 있다. 반면, 팩터 그래프는 잠재적으로 엄청난 수의 광도 팩터로 인해 계산 부담이 가중될 수 있다. 이는 '최고의' 백엔드 방식이 정해져 있는 것이 아니라, 프론트엔드와 지도 표현 방식의 선택에 따라 달라질 수 있음을 시사한다. 특히 GS-LIVO가 임베디드 하드웨어에서 거둔 성공은 자원 제약이 심한 응용 분야에서 필터 기반 접근법의 강력한 타당성을 입증한다.

| 기능                     | GS-LIVO                     | LIO-SAM                        | VINS-Fusion                          | R3LIVE                                  |
| ------------------------ | --------------------------- | ------------------------------ | ------------------------------------ | --------------------------------------- |
| **주요 센서**            | LiDAR, IMU, 카메라          | LiDAR, IMU, (GPS)              | 카메라, IMU, (스테레오)              | LiDAR, IMU, 카메라                      |
| **핵심 추정 프레임워크** | 반복 확장 칼만 필터 (IESKF) | 팩터 그래프 최적화             | 팩터 그래프 최적화 (슬라이딩 윈도우) | 분리된 LIO + VIO (필터 기반)            |
| **지도 표현 방식**       | 3D 가우시안 (조밀, 사실적)  | 희소 특징점 포인트 클라우드    | 희소 3D 랜드마크                     | 색상화된 포인트 클라우드 (조밀, 기하학) |
| **시각 제약 조건**       | 조밀한 광도 오차            | 해당 없음                      | 희소 특징점 재투영 오차              | 텍스처 투영                             |
| **LiDAR 제약 조건**      | 기하학적 잔차               | 기하학적 특징점 매칭           | 해당 없음                            | 기하학적 잔차 (FAST-LIO)                |
| **루프 클로저**          | (논문에서 명시적 언급 없음) | 팩터 그래프                    | 팩터 그래프 (DBoW2)                  | (논문에서 명시적 언급 없음)             |
| **핵심 혁신**            | 임베디드 실시간 3D-GS SLAM  | 팩터 그래프 기반 긴밀 결합 LIO | 최적화 기반의 강인한 VIO             | 분리된 LIO+VIO를 통한 실시간 컬러 매핑  |


아무리 정교한 아키텍처를 가졌더라도, 실제 환경에서의 성능이 입증되지 않으면 그 가치는 제한적이다. 이 장에서는 GS-LIVO의 성능을 위치 추정 정확도, 지도 및 렌더링 품질, 그리고 계산 자원 소모량 측면에서 비판적으로 평가하고, 특히 실제 로봇에 적용될 수 있는 배포 가능성에 초점을 맞춘다.


SLAM 시스템의 가장 기본적인 성능 지표는 위치 추정의 정확도다. 이는 일반적으로 절대 궤적 오차(Absolute Trajectory Error, ATE)와 상대 포즈 오차(Relative Pose Error, RPE)라는 표준 메트릭으로 평가된다.34 ATE는 추정된 전체 궤적과 실제 궤적(ground truth) 간의 전역적인 차이를 측정하며, RPE는 짧은 시간 간격 동안의 움직임 오차를 측정하여 드리프트의 정도를 평가한다.

GS-LIVO의 GitHub 저장소는 MARS-LVIG, Landmark, UAV Playground, FAST-LIVO HKU 등 여러 데이터셋에서의 실험 결과를 동영상 형태로 제공하고 있다.19 이러한 질적 평가는 시스템이 다양한 환경에서 안정적으로 동작함을 보여준다. 논문에서는 GS-LIVO가 "경쟁력 있는 위치 추정 정확도"를 보이며 "기존 시스템 대비 제곱 평균 오차(RMSE)를 현저히 감소시켰다"고 주장한다.12

하지만 본 보고서 작성 시점까지 수집된 자료 내에서는, KITTI나 EuRoC과 같은 표준 벤치마크 데이터셋에서 다른 시스템들과 ATE/RPE를 직접 비교한 정량적인 수치 테이블이 공식적으로 발표되지 않았다.4 따라서 현재로서는 GS-LIVO의 위치 추정 정확도를 객관적인 수치로 타 시스템과 비교하는 데에는 한계가 있다. 아래 표는 향후 공식적인 벤치마크 결과가 발표될 때 GS-LIVO가 비교될 기준점을 제시하기 위해 구성되었다.

| 방법                     | 데이터셋 시퀀스 | ATE (m)         |
| ------------------------ | --------------- | --------------- |
| **GS-LIVO**              | (공개 예정)     | **(공개 예정)** |
| **LIO-SAM**              | KITTI 00        | (자료 없음) 21  |
| **VINS-Fusion (Stereo)** | EuRoC MH_01     | 0.081           |
| **R3LIVE**               | (자체 데이터셋) | (자료 없음) 15  |
| **ORB-SLAM2 (Stereo)**   | KITTI 00        | 0.7 (추정치)    |
|                          |                 |                 |


GS-LIVO의 가장 두드러진 장점 중 하나는 고품질의 지도 생성 능력이다. 논문과 프로젝트 페이지는 GS-LIVO가 "우수한 매핑 품질"과 "사실적인 렌더링"을 달성했다고 일관되게 주장한다.12 공개된 렌더링 이미지와 비디오는 복잡한 구조물과 텍스처를 세밀하게 재구성하는 시스템의 능력을 시각적으로 증명한다.19

이러한 높은 품질은 두 가지 핵심 요소에 기인한다. 첫째, 3D-GS 표현 방식 자체가 장면의 외형을 사실적으로 모델링하는 데 특화되어 있다. 둘째, LiDAR 데이터와의 융합이 지도의 기하학적 정확도를 보장한다. 순수 시각 기반 3D-GS SLAM 시스템들은 종종 기하학적 왜곡에 취약하지만 3, GS-LIVO는 LiDAR의 정밀한 거리 측정값을 바탕으로 가우시안을 초기화하고 위치를 보정하므로, 구조적으로 더 정확한 지도를 생성할 수 있다.


GS-LIVO는 실시간 성능을 목표로 설계되었으며, 10Hz 이상의 프레임률을 달성한다고 보고되었다.12 이러한 성능은 2장에서 분석한 IESKF와 가우시안 슬라이딩 윈도우 아키텍처의 효율성 덕분이다.

그러나 GS-LIVO의 가장 중요한 실제적 기여는 임베디드 시스템에서의 성공적인 배포다. 프로젝트는 고성능 데스크톱 GPU(NVIDIA RTX 4090)에서의 데이터셋 처리 결과와 함께, 자율주행 차량에 탑재된 자원 제한적인 임베디드 시스템(NVIDIA Jetson Orin NX 16GB)에서의 구동 사례를 명확히 제시하고 있다.7

이것은 단순한 성능 지표를 넘어선다. 이는 GS-LIVO의 설계 철학이 실제 로보틱스 응용의 핵심 제약 조건인 크기, 무게, 전력 및 비용(SWaP-C)을 깊이 고려했음을 의미한다. 수많은 첨단 SLAM 알고리즘들이 강력한 데스크톱 PC에 의존하여 그 잠재력을 선보이는 동안, 로봇 산업 현장에서는 Jetson 시리즈와 같은 임베디드 플랫폼에서 구동 가능한 알고리즘을 절실히 필요로 해왔다. GS-LIVO는 "자원이 제한된 임베디드 시스템에 배포 가능한 최초의 실시간 가우시안 기반 SLAM 프레임워크"라는 주장 7을 통해, 이론적인 고성능 SLAM 연구와 실제 로봇 응용 사이의 간극을 메우는 중요한 이정표를 세웠다. 이는 강력한 컴퓨터를 로봇에 연결할 수 없는 수많은 상업 및 학술 프로젝트에 최첨단 사실적 매핑 기술을 적용할 수 있는 길을 열어준 것으로, 기술의 접근성을 획기적으로 낮춘 성과로 평가할 수 있다. 논문의 목차에서 언급된 슬라이딩 윈도우 메커니즘의 메모리 및 시간 소모량에 대한 애블레이션 연구(ablation study) 결과는 이러한 효율성 주장을 뒷받침하는 핵심적인 정량적 데이터가 될 것이다.5

| 메트릭                            | 고성능 PC (RTX 4090) | 임베디드 시스템 (Jetson Orin NX) |
| --------------------------------- | -------------------- | -------------------------------- |
| **오도메트리 업데이트 주기 (Hz)** | > 10                 | > 10                             |
| **지도 업데이트 주기 (Hz)**       | > 10                 | > 10                             |
| **CPU 사용량 (%)**                | (공개 예정)          | (공개 예정)                      |
| **GPU 사용량 (%)**                | (공개 예정)          | (공개 예정)                      |
| **GPU 메모리 소모량 (GB)**        | (공개 예정)          | (공개 예정)                      |


본 보고서는 GS-LIVO의 기술적 세부 사항과 성능을 심도 있게 분석했다. 마지막으로, 이 분석을 종합하여 GS-LIVO의 핵심 기여를 요약하고, 현재의 한계와 도전 과제를 진단하며, 이 기술이 열어갈 미래 연구 방향과 더 넓은 파급 효과를 전망한다.


GS-LIVO의 핵심적인 기여는 다음과 같이 요약할 수 있다: **임베디드 하드웨어에서 실시간으로 구동되는, IESKF 기반의 LiDAR-관성-시각 융합 3D 가우시안 스플래팅 SLAM 시스템을 최초로 구현했다는 점이다.** 이는 고충실도의 사실적인 렌더링 기술을 강인하고 빠른 고주파 상태 추정 프레임워크와 성공적으로 통합한 결과물이다. GS-LIVO는 이론적 가능성에 머물러 있던 실시간 사실적 매핑을 실제 로봇 플랫폼으로 가져오는 중요한 실용적 진전을 이루었다.


모든 혁신적인 기술과 마찬가지로, GS-LIVO 역시 해결해야 할 과제들을 안고 있다.

- **동적 환경 대응:** 대부분의 초기 3D-GS SLAM 시스템과 마찬가지로, GS-LIVO의 기본 프레임워크는 움직이는 객체를 명시적으로 모델링하지 않는다. 이는 동적 객체가 지도에 잔상으로 남거나 위치 추정 오차를 유발하는 원인이 될 수 있다. 최근 'Dynamic' GS-SLAM 관련 연구들이 폭발적으로 증가하는 것에서 알 수 있듯이 10, 동적 환경에 대한 강인성 확보는 이 분야의 가장 시급한 과제다.
- **장기적 일관성:** 전역 옥트리 지도가 어느 정도 드리프트를 억제하지만, IESKF 프레임워크는 과거 상태를 버리기 때문에 강인한 그래프 기반 루프 클로저 메커니즘 없이는 장기적인 드리프트 누적에 취약할 수 있다. LIO-SAM과 같은 전역 최적화 백엔드를 통합하는 것이 향후 중요한 개선 방향이 될 수 있다.
- **광도 변화에 대한 민감성:** 광도 오차에 크게 의존하는 방식은 극심한 조명 변화, 카메라의 자동 노출 및 화이트 밸런스 변동 등에 취약할 수 있다. R3LIVE++가 카메라의 광도 보정(photometric calibration)을 연구한 사례는 29, 이러한 문제를 해결하기 위한 중요한 연구 방향을 제시한다.


GS-LIVO와 유사 시스템들의 등장은 SLAM 기술의 역할을 재정의하고 로보틱스의 새로운 가능성을 열고 있다.

- **디지털 트윈으로의 길:** GS-LIVO는 현실 세계의 공간을 지속적으로 업데이트되는 사실적인 디지털 트윈으로 생성하고 유지하는 핵심 기술이 될 잠재력을 가지고 있다. 이는 시뮬레이션, 원격 모니터링, VR/AR 콘텐츠 제작 등 다양한 산업 분야에 혁신을 가져올 것이다.11
- **시맨틱 SLAM과의 통합:** 다음 단계는 시맨틱(semantic) 이해와의 통합이다. 지도가 단순히 색을 가진 가우시안의 집합이 아니라, '자동차', '건물', '나무'와 같이 의미론적으로 분할된 객체들로 구성될 수 있다. Semgauss-SLAM과 같은 연구는 이러한 방향성을 보여준다.10
- **자율 주행 및 로봇 상호작용에의 영향:** 조밀하고 사실적인 지도는 단순히 위치 추정을 넘어 로봇의 다른 기능들을 향상시킬 수 있다. 지도의 외형으로부터 지면의 주행 가능 여부를 판단하거나, 객체의 정확한 형태와 위치를 파악하여 더 정교한 조작(manipulation)을 수행할 수 있다. 또한, 인간이 직관적으로 이해할 수 있는 지도는 인간-로봇 상D_S9, 19].

GS-LIVO의 성공은 SLAM 파이프라인의 '재통합(re-bundling)' 가능성을 시사한다는 점에서 특히 흥미롭다. 지난 몇 년간 SLAM 시스템은 추적, 매핑, 루프 클로저를 별도의 스레드에서 처리하는 모듈화되고 분리된(decoupled) 아키텍처로 발전해왔다 (예: ORB-SLAM3 1, VINS-Fusion 39). R3LIVE 역시 LIO와 VIO를 명시적으로 분리했다.15 그러나 GS-LIVO는 포즈 예측, 지도 렌더링, 렌더링 오차를 이용한 포즈 보정이 단일 IESKF 업데이트 주기 내에서 긴밀하게 이루어진다.12 이는 지도와 궤적이 매 순간 서로에게 영향을 미치는 훨씬 더 강력한 결합 방식이다. 이러한 '재통합'은 3D-GS 렌더링과 IESKF의 효율성 덕분에 가능해졌으며, 미래에는 프론트엔드와 백엔드의 구분이 모호해지고 단일화된 추정 및 렌더링 루프로 대체될 수 있음을 암시한다. GS-LIVO는 그 가능성을 연 선구적인 작업으로 기록될 것이다.


1. a Stereo SLAM System through the Combination of Points and Line Segments - Robotics and Perception Group, accessed July 19, 2025, https://rpg.ifi.uzh.ch/docs/arXiv17_Gomez_pl_slam.pdf
2. SR-LIO: LiDAR-Inertial Odometry with Sweep Reconstruction - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2210.10424
3. Gaussian-LIC2: LiDAR-Inertial-Camera Gaussian Splatting SLAM - arXiv, accessed July 19, 2025, https://arxiv.org/html/2507.04004v1
4. GS-LIVM: Real-Time Photo-Realistic LiDAR-Inertial-Visual Mapping with Gaussian Splatting, accessed July 19, 2025, https://arxiv.org/html/2410.17084v1
5. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.08672v1
6. Gaussian-LIC2: LiDAR-Inertial-Camera Gaussian Splatting SLAM - arXiv, accessed July 19, 2025, https://arxiv.org/html/2507.04004v2
7. [2501.08672] GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2501.08672
8. GS-SLAM: Dense Visual SLAM with 3D Gaussian Splatting, accessed July 19, 2025, https://gs-slam.github.io/
9. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11902412/
10. KwanWaiPang/Awesome-3DGS-SLAM - GitHub, accessed July 19, 2025, https://github.com/KwanWaiPang/Awesome-3DGS-SLAM
11. LVI-GS: Tightly-coupled LiDAR-Visual-Inertial SLAM using 3D Gaussian Splatting, accessed July 19, 2025, https://www.researchgate.net/publication/385559855_LVI-GS_Tightly-coupled_LiDAR-Visual-Inertial_SLAM_using_3D_Gaussian_Splatting
12. [Literature Review] GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - Moonlight | AI Colleague for Research Papers, accessed July 19, 2025, https://www.themoonlight.io/en/review/gs-livo-real-time-lidar-inertial-and-visual-multi-sensor-fused-odometry-with-gaussian-mapping
13. LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.20380v1
14. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/392993415_GS-LIVO_Real-Time_LiDAR_Inertial_and_Visual_Multi-sensor_Fused_Odometry_with_Gaussian_Mapping
15. hku-mars/r3live: A Robust, Real-time, RGB-colored, LiDAR ... - GitHub, accessed July 19, 2025, https://github.com/hku-mars/r3live
16. LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 19, 2025, https://github.com/TixiaoShan/LVI-SAM
17. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/388067651_GS-LIVO_Real-Time_LiDAR_Inertial_and_Visual_Multi-sensor_Fused_Odometry_with_Gaussian_Mapping
18. Multi-sensor Fusion Towards VINS: A Concise Tutorial, Survey, Framework and Challenges | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/340812782_Multi-sensor_Fusion_Towards_VINS_A_Concise_Tutorial_Survey_Framework_and_Challenges
19. HKUST-Aerial-Robotics/GS-LIVO - GitHub, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/GS-LIVO
20. [Literature Review] Gaussian-LIC: Real-Time Photo-Realistic SLAM with Gaussian Splatting and LiDAR-Inertial-Camera Fusion - Moonlight | AI Colleague for Research Papers, accessed July 19, 2025, https://www.themoonlight.io/en/review/gaussian-lic-real-time-photo-realistic-slam-with-gaussian-splatting-and-lidar-inertial-camera-fusion
21. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed July 19, 2025, http://arxiv.org/pdf/2007.00258
22. VINS-Multi: A Robust Asynchronous Multi-camera-IMU State Estimator - ICRA 2025 Construction Robotics Workshop, accessed July 19, 2025, https://construction-robots.github.io/papers/60.pdf
23. Factor Graph Accelerator for LiDAR-Inertial Odometry (Invited Paper) - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2209.02207
24. TixiaoShan/LIO-SAM: LIO-SAM: Tightly-coupled Lidar ... - GitHub, accessed July 19, 2025, https://github.com/TixiaoShan/LIO-SAM
25. README.md - LIO-SAM - GitLab, accessed July 19, 2025, https://gitlab.uni-hannover.de/esistoiamk/LIO-SAM/-/blob/master/README.md
26. HKUST-Aerial-Robotics/VINS-Fusion: An optimization ... - GitHub, accessed July 19, 2025, https://github.com/HKUST-Aerial-Robotics/VINS-Fusion
27. LiPO: LiDAR Inertial Odometry for ICP Comparison - arXiv, accessed July 19, 2025, https://arxiv.org/html/2410.08097v1
28. SuperVINS: A visual-inertial SLAM framework integrated deep learning features - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.21348v1
29. R 3 ^3 LIVE++: A Robust, Real-time, Radiance reconstruction package with a tightly-coupled LiDAR-Inertial-Visual state Estimator - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/363403813_R3LIVE_A_Robust_Real-time_Radiance_reconstruction_package_with_a_tightly-coupled_LiDAR-Inertial-Visual_state_Estimator
30. dtc111111/awesome-3dgs-for-robotics - GitHub, accessed July 19, 2025, https://github.com/dtc111111/awesome-3dgs-for-robotics
31. [ICCV'25] GS-LIVM: Real-Time Photo-Realistic LiDAR-Inertial-Visual Mapping with Gaussian Splatting - GitHub, accessed July 19, 2025, https://github.com/xieyuser/GS-LIVM
32. [Literature Review] GS-LIVM: Real-Time Photo-Realistic LiDAR-Inertial-Visual Mapping with Gaussian Splatting - Moonlight, accessed July 19, 2025, https://www.themoonlight.io/en/review/gs-livm-real-time-photo-realistic-lidar-inertial-visual-mapping-with-gaussian-splatting
33. GS-LIVO: Real-Time LiDAR, Inertial, and Visual Multi-sensor Fused Odometry with Gaussian Mapping - Papers With Code, accessed July 19, 2025, https://paperswithcode.com/paper/gs-livo-real-time-lidar-inertial-and-visual/review/
34. Semantic SLAM - KITTI-360, accessed July 19, 2025, https://www.cvlibs.net/datasets/kitti-360/leaderboard_semantic_slam.php?task=trajectory
35. DIDLM - arXiv, accessed July 19, 2025, [https://arxiv.org/pdf/2404.09622?](https://arxiv.org/pdf/2404.09622)
36. Comparisons of RPE and ATE of KITTI dataset for ORB-SLAM2 and SP-SLAM., accessed July 19, 2025, https://www.researchgate.net/figure/Comparisons-of-RPE-and-ATE-of-KITTI-dataset-for-ORB-SLAM2-and-SP-SLAM_tbl4_382927580
37. accessed January 1, 1970, [https.arxiv.org/pdf/2501.08672](http://docs.google.com/https.arxiv.org/pdf/2501.08672)
38. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/385964386_FAST-LIVO2_Fast_Direct_LiDAR-Inertial-Visual_Odometry
39. Visual-Inertial SLAM as Simple as A, B, VINS - arXiv, accessed July 19, 2025, https://arxiv.org/html/2406.05969v2


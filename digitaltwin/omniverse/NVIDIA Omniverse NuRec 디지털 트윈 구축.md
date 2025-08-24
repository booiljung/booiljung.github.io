[NVidia Omniverse](./index.md)
# NVIDIA Omniverse NuRec를 활용한 물리 기반 고충실도 디지털 트윈 구축


현대 산업의 디지털 전환은 물리적 세계와 가상 세계의 경계를 허물고 있습니다. 이 혁신의 중심에는 '디지털 트윈' 기술이 있으며, 이는 단순한 3D 모델을 넘어 실제 객체, 프로세스, 또는 시스템과 실시간으로 동기화되는 살아있는 가상 복제본입니다.1 고충실도(High-Fidelity) 디지털 트윈은 물리 법칙에 기반한 정밀한 시뮬레이션과 사실적인 시각화를 통해 현실 세계의 복잡한 현상을 예측하고 최적화하는 것을 목표로 합니다.1 본 보고서는 NVIDIA Omniverse 플랫폼, 특히 신경 재구성 기술인 NuRec(Neural Reconstruction)을 활용하여 이러한 고충실도 디지털 트윈을 구축하는 전 과정을 이론적 기반부터 실용적 워크플로우, 그리고 산업 적용 사례에 이르기까지 심층적으로 탐구합니다.


NVIDIA Omniverse는 단일 애플리케이션이 아닌, 산업 디지털화를 위한 운영 체제에 가까운 개방형 플랫폼입니다.4 이는 다양한 3D 설계 및 시뮬레이션 도구들을 연결하고, 분산된 팀이 동일한 가상 공간에서 실시간으로 협업할 수 있는 환경을 제공합니다.6 Omniverse의 힘은 그 핵심 아키텍처에서 비롯됩니다.


Omniverse 플랫폼은 세 가지 핵심 기술을 기반으로 구축되어 상호 유기적으로 작동합니다.

- **OpenUSD (Universal Scene Description):** Pixar Animation Studios에서 개발하고 오픈소스로 공개한 OpenUSD는 단순한 3D 파일 형식을 넘어, 복잡한 3D 장면을 구성하고 편집하기 위한 강력한 프레임워크입니다.7 OpenUSD의 가장 큰 특징은 '레이어(Layer)' 개념을 통한 비파괴적(Non-destructive) 편집 워크플로우입니다. 이를 통해 여러 아티스트나 엔지니어가 각자의 작업(모델링, 레이아웃, 애니메이션, 조명 등)을 별도의 레이어 파일로 관리하면서 하나의 통합된 장면으로 조합할 수 있습니다. 이는 데이터 중복을 피하고, 원본 데이터의 수정을 최소화하며, 복잡한 프로젝트의 협업 효율성을 극대화합니다.9 Omniverse 생태계에서 OpenUSD는 모든 데이터와 애플리케이션을 연결하는 '결합 조직(connective tissue)' 역할을 수행합니다.8
- **Nucleus:** Nucleus는 OpenUSD 데이터를 실시간으로 관리하고 동기화하는 데이터베이스 및 협업 엔진입니다.6 사용자들이 Omniverse Nucleus 서버에 3D 에셋과 장면 파일을 저장하면, Nucleus는 파일의 변경 사항을 원자적 단위(atomic updates)로 추적하고 구독(subscribe)하고 있는 모든 연결된 클라이언트(사용자 또는 애플리케이션)에게 실시간으로 전송합니다. 이 '라이브 싱크(Live Sync)' 기능 덕분에, 한 사용자가 Maya에서 모델을 수정하면 그 변경 사항이 즉시 다른 사용자의 Unreal Engine이나 Omniverse USD Composer 뷰포트에 반영되는 동시적 협업이 가능해집니다.6
- **Kit SDK:** Kit SDK는 Omniverse를 위한 애플리케이션, 확장(Extension), 커넥터(Connector)를 개발하기 위한 모듈형 툴킷입니다.8 파이썬 기반의 유연한 아키텍처를 통해 개발자들은 특정 산업 워크플로우에 필요한 맞춤형 도구를 제작하거나 기존 기능을 확장할 수 있습니다. 예를 들어, 특정 CAD 파일 형식을 가져오는 변환기를 만들거나, 시뮬레이션 데이터를 분석하는 독자적인 UI 패널을 구축하는 등의 작업이 가능합니다.9


산업 현장에서는 설계, 엔지니어링, 시뮬레이션 등 각기 다른 단계에서 다양한 소프트웨어가 사용되며, 이는 심각한 데이터 사일로(silo) 문제를 야기합니다. Omniverse는 OpenUSD와 '커넥터' 생태계를 통해 이 문제를 해결합니다.8 커넥터는 Autodesk 3ds Max, Maya, Revit, Trimble SketchUp, Epic Games Unreal Engine 등 주요 디지털 콘텐츠 제작(DCC) 도구에 설치되는 플러그인으로, 해당 애플리케이션과 Nucleus 서버 간의 실시간 데이터 동기화를 가능하게 합니다.8

데이터 통합 방식은 크게 세 가지로 나뉩니다.

1. **네이티브 커넥터:** DCC 도구와 Nucleus 간의 양방향 라이브 싱크를 지원합니다.
2. **에셋 임포터(Asset Importer):** FBX, OBJ, glTF와 같은 일반적인 3D 형식을 OpenUSD로 변환하여 Omniverse 환경으로 가져옵니다.13
3. **CAD 변환기(CAD Converter):** CATIA, SolidWorks, STEP 등 정밀한 엔지니어링 CAD 데이터를 시각화 및 시뮬레이션에 적합한 USD 폴리곤 메시로 변환합니다.14

이러한 통합 방식을 통해 Omniverse는 다양한 소스로부터 생성된 데이터를 단일한 '진실의 원천(Single Source of Truth)'으로 통합하여 일관성 있는 디지털 트윈을 구축할 수 있는 기반을 제공합니다.


Omniverse는 시뮬레이션에서 인공지능(AI)의 역할을 근본적으로 재정의합니다. 과거의 AI가 자동화된 스크립트나 제한된 분석 도구에 머물렀다면, Omniverse 환경에서의 AI는 복잡한 시뮬레이션 세계를 감각하고, 조율하며, 결론을 내리는 핵심 주체로 부상합니다.4

전통적인 디지털 트윈은 주로 인간 운영자가 시스템을 모니터링하고 분석하기 위한 도구였습니다.17 그러나 디지털 트윈의 규모가 BMW의 전체 공장이나 삼성전자의 반도체 팹과 같이 거대해지면서, 상호 작용하는 변수와 데이터의 양은 인간의 인지 능력을 초월하게 되었습니다.4 수천 개의 로봇, 수만 개의 센서 데이터, 다양한 생산 시나리오가 얽힌 상태 공간(state space)을 인간이 실시간으로 이해하고 최적의 결정을 내리는 것은 불가능에 가깝습니다.

이러한 복잡성을 관리하기 위해 Omniverse는 AI를 새로운 '관측 주체(observing subject)'로 내세웁니다.4 AI는 인간을 대신하여 수천 개의 시나리오를 병렬적으로 평가하고, 다양한 조건 변화에 따른 결과를 예측하며, 최적의 운영 방안을 탐색합니다. 이는 인간과 AI의 역할을 역전시키는 패러다임의 전환입니다. 인간의 역할은 더 이상 미시적인 데이터 분석이 아니라, AI가 도출한 거시적인 결론을 해석하고, AI 에이전트의 행동을 위한 전략적 목표와 윤리적 경계를 설정하며, 시뮬레이션 결과를 통해 새로운 통찰을 얻는 고차원적인 지적 활동으로 이동합니다.

이러한 비전의 핵심에는 '에이전트 AI(Agentic AI)' 개념이 있습니다. 이는 단순히 데이터를 처리하는 것을 넘어, 주어진 환경 내에서 합리적으로 추론하고, 계획하며, 행동하도록 설계된 AI 에이전트를 의미합니다.16 Omniverse의 물리 기반 시뮬레이션 환경은 이러한 에이전트 AI를 훈련하고 검증하기 위한 이상적인 가상 테스트베드를 제공하며, 이는 자율 로봇 및 자율주행차 개발의 핵심 요소가 됩니다.


'고충실도'와 '물리 기반'이라는 용어는 마케팅 수사가 아닌, 엄격한 수학적 모델과 물리 법칙에 근거합니다. 디지털 트윈이 현실 세계를 정확하게 모사하기 위해서는 그 근간이 되는 운동, 기하학, 빛의 상호작용을 수학적으로 올바르게 표현해야 합니다.


물리 기반 시뮬레이션의 핵심은 고전 역학, 특히 강체(Rigid Body)의 운동을 지배하는 법칙들을 수치적으로 계산하는 것입니다. Omniverse는 NVIDIA PhysX 물리 엔진을 통해 이러한 계산을 실시간으로 수행합니다.24

- **병진 운동 (Translational Motion):** 물체의 무게 중심이 외부 힘에 반응하여 어떻게 움직이는지는 뉴턴의 제2법칙으로 기술됩니다. 이 법칙은 물체에 가해진 모든 외부 힘의 합이 물체의 질량과 무게 중심 가속도의 곱과 같다고 명시합니다.27
  $$
  \sum \vec{F}{ext} = \frac{d\vec{P}}{dt} = m\vec{a}{cg}
  $$
  

  여기서 $\sum \vec{F}_{ext}$는 외부 힘의 벡터 합, $\vec{P}$는 선형 운동량, $m$은 질량, $\vec{a}_{cg}$는 무게 중심의 가속도입니다.

- **회전 운동 (Rotational Motion):** 물체의 회전 운동은 오일러 운동방정식(Euler's equations of motion)으로 설명됩니다. 이 방정식은 물체에 가해진 토크(torque)가 각속도의 변화를 어떻게 유발하는지를 나타냅니다. 이는 로봇 팔, 차량, 기계 부품 등 회전하는 모든 시스템을 시뮬레이션하는 데 필수적입니다.30
  $$
  \mathbf{N} = \mathbf{I}\dot{\boldsymbol{\omega}} + \boldsymbol{\omega} \times (\mathbf{I}\boldsymbol{\omega})
  $$
  이 식에서 $\mathbf{N}$은 외부 토크의 벡터 합, $\mathbf{I}$는 관성 텐서(물체의 질량 분포와 형태를 나타내는 3x3 행렬), $\boldsymbol{\omega}$는 각속도 벡터, $\dot{\boldsymbol{\omega}}$는 각가속도 벡터입니다. 이 방정식은 물체의 고유 좌표계에서 계산되며, 관성 텐서의 복잡한 항 때문에 비선형적인 특징을 가집니다.


3D 가상 공간 내에서 객체의 상태(위치, 방향, 크기)를 표현하고 조작하기 위해 행렬 연산이 사용됩니다. 특히 동차 좌표계(Homogeneous Coordinates)를 사용하면 이동, 회전, 크기 조절과 같은 기하학적 변환을 단일 4x4 행렬의 곱셈으로 통합하여 표현할 수 있습니다. 이는 현대 그래픽 하드웨어(GPU)의 병렬 처리 아키텍처에 매우 효율적입니다.32

3D 공간의 한 점 $\mathbf{p} = [x, y, z, 1]^T$를 새로운 점 $\mathbf{p'}$로 변환하는 과정은 변환 행렬의 순차적인 곱으로 나타낼 수 있습니다. 변환의 순서가 결과에 영향을 미치므로 매우 중요합니다. 일반적인 순서는 크기 조절($\mathbf{S}$), 회전($\mathbf{R}$), 이동($\mathbf{T}$) 순입니다.32
$$
\mathbf{p}′=\mathbf{T} \cdot \mathbf{R} \cdot  \mathbf{S} \cdot  \mathbf{p}
$$
각 변환 행렬은 다음과 같이 정의됩니다.

- 이동 행렬 (Translation Matrix):
  $$
  \mathbf{T}(t_x, t_y, t_z) = \begin{bmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix}
  $$

- 크기 조절 행렬 (Scaling Matrix):
  $$
  \mathbf{S}(s_x, s_y, s_z) = \begin{bmatrix} s_x & 0 & 0 & 0 \\ 0 & s_y & 0 & 0 \\ 0 & 0 & s_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}
  $$

- 회전 행렬 (Rotation Matrix, 예: Y축 기준 $\theta$ 회전):
  $$
  \mathbf{R}_y(\theta) = \begin{bmatrix} \cos\theta & 0 & \sin\theta & 0 \\ 0 & 1 & 0 & 0 \\ -\sin\theta & 0 & \cos\theta & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}
  $$


디지털 트윈의 시각적 충실도는 NVIDIA RTX 기술에 의해 구동되는 물리 기반 렌더링(Physically Based Rendering, PBR)에 의해 결정됩니다.36 PBR의 이론적 기반은 1986년 제임스 카지야(James Kajiya)에 의해 정립된 렌더링 방정식(Rendering Equation)입니다.
$$
L_o(x, \omega_o) = L_e(x, \omega_o) + \int_{\Omega} f_r(x, \omega_i, \omega_o) L_i(x, \omega_i) (\omega_i \cdot \vec{n}) d\omega_i
$$
이 적분 방정식은 장면 내 빛 에너지의 평형 상태를 설명하며, 각 항은 다음과 같은 의미를 가집니다 37:

- $L_o(x, \omega_o)$: 특정 지점 $x$에서 특정 방향 $\omega_o$으로 나가는 빛의 양(Radiance). 우리가 최종적으로 계산하고자 하는 값입니다.
- $L_e(x, \omega_o)$: 지점 $x$ 자체에서 방출되는 빛의 양 (예: 광원).
- $\int_{\Omega}$: 해당 지점을 중심으로 하는 반구(hemisphere) $\Omega$ 내의 모든 방향에 대한 적분.
- $f_r(x, \omega_i, \omega_o)$: 양방향 반사 분포 함수(Bidirectional Reflectance Distribution Function, BRDF). 들어오는 빛 $\omega_i$가 나가는 방향 $\omega_o$으로 얼마나 반사되는지를 결정하는 재질(material)의 고유 특성입니다.
- $L_i(x, \omega_i)$: 방향 $\omega_i$에서 지점 $x$로 들어오는 빛의 양.
- $(\omega_i \cdot \vec{n})$: 들어오는 빛의 각도에 따른 감쇠를 나타내는 기하학적 항 ($\vec{n}$은 표면의 법선 벡터).

이 방정식은 해석적으로 풀기 매우 어렵기 때문에, 패스 트레이싱(Path Tracing)과 같은 몬테카를로(Monte Carlo) 기반의 수치 해석 기법이 사용됩니다. 패스 트레이싱은 카메라에서부터 빛의 경로를 역추적하여 광원에 도달할 때까지의 상호작용을 시뮬레이션하는 방식입니다. NVIDIA RTX GPU는 이 과정, 특히 광선과 물체의 교차점 계산을 하드웨어적으로 가속하여, 과거에는 수 시간이 걸렸던 물리적으로 정확한 렌더링을 실시간으로 가능하게 합니다. 이를 통해 간접 조명(Global Illumination), 부드러운 그림자, 사실적인 반사와 굴절 효과를 구현하여 디지털 트윈의 시각적 사실감을 극대화합니다.36

**표 1: 강체 동역학의 핵심 방정식**

이 표는 고충실도 물리 시뮬레이션의 근간을 이루는 핵심 방정식을 요약하여 제공합니다. 이 방정식들은 디지털 트윈 내 객체들의 움직임과 상호작용을 지배하는 물리 법칙의 수학적 표현입니다.

| 방정식 명칭          | 수학 공식                                                    | 설명                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 뉴턴 제2법칙 (병진)  | $\sum \vec{F}_{ext} = m\vec{a}_{cg}$                         | 물체에 가해진 순수 외부 힘은 물체의 질량과 무게 중심 가속도의 곱과 같습니다. |
| 오일러 방정식 (회전) | $\mathbf{N} = \mathbf{I}\dot{\boldsymbol{\omega}} + \boldsymbol{\omega} \times (\mathbf{I}\boldsymbol{\omega})$ | 물체에 가해진 순수 외부 토크는 관성 텐서, 각속도, 각가속도와 관련됩니다. |
| 기하학적 변환        | $\mathbf{p'} = \mathbf{T} \mathbf{R} \mathbf{S} \mathbf{p}$  | 점 $\mathbf{p}$는 크기 조절, 회전, 이동 행렬을 순차적으로 곱하여 새로운 점 $\mathbf{p'}$로 변환됩니다. |
| 렌더링 방정식        | $L_o = L_e + \int_{\Omega} f_r L_i (\omega_i \cdot \vec{n}) d\omega_i$ | 한 지점에서 나가는 빛은 자체 발광하는 빛과 모든 방향에서 들어와 반사되는 빛의 합입니다. |


이 파트에서는 이론적 배경을 바탕으로, 실제 세계의 한 장면을 포착하여 상호작용이 가능한 고충실도 디지털 트윈으로 변환하는 구체적이고 단계적인 워크플로우를 제시합니다. 이 과정의 핵심에는 NVIDIA Omniverse NuRec 신경 재구성 라이브러리가 있습니다.


Omniverse NuRec는 여러 장의 2D 이미지나 센서 데이터로부터 사실적인 3D 환경을 신속하게 재구성하는 기술 모음입니다.38 이 기술은 로보틱스, 자율주행 등 시뮬레이션 기반 AI 모델 개발에서 현실과 가상 환경 간의 차이, 즉 '심투리얼 갭(Sim-to-Real Gap)'을 줄이는 데 결정적인 역할을 합니다.24


고품질의 신경 재구성 결과물은 양질의 입력 데이터에서 시작됩니다. SfM(Structure-from-Motion) 파이프라인이 이미지 간의 특징점을 효과적으로 매칭할 수 있도록 데이터를 수집하는 것이 중요합니다.38

- **이미지 수량 및 중첩:** 약 100장 내외의 사진을 촬영하는 것이 권장됩니다. 각 이미지는 인접한 이미지와 충분한 영역(약 60-80%)이 중첩되도록 촬영하여 특징점 추적의 연속성을 보장해야 합니다.
- **촬영 경로:** 재구성하려는 대상이나 공간의 모든 각도에서 촬영해야 합니다. 일반적으로 대상 주위를 여러 높이에서 원을 그리며 촬영하는 'Orbit' 방식과 공간 내부를 이동하며 촬영하는 'Walkthrough' 방식이 사용됩니다.
- **카메라 설정:** 조리개(f-stop)는 f/8 이상으로 조여 심도를 깊게 하고, 셔터 속도는 1/100초 이상으로 확보하여 흔들림 없는 선명한 이미지를 얻는 것이 좋습니다. 초점 거리는 18mm와 같은 광각 렌즈가 더 넓은 영역을 한 번에 담아 특징점 매칭에 유리할 수 있습니다.38
- **조명:** 부드럽고 균일한 조명 환경이 가장 이상적입니다. 강한 직사광선으로 인한 짙은 그림자나 하이라이트는 특징점 추출을 방해할 수 있습니다.


수집된 이미지 세트는 오픈소스 SfM 및 MVS(Multi-View Stereo) 파이프라인인 COLMAP을 사용하여 처리됩니다.38 이 단계의 목표는 이미지들로부터 3D 공간상의 희소한 점들(sparse point cloud)과 각 이미지가 촬영될 당시의 카메라 위치 및 방향(pose), 그리고 렌즈 왜곡과 같은 내부 파라미터(intrinsics)를 추정하는 것입니다.

COLMAP의 처리 과정은 다음과 같습니다.

1. **특징점 추출(Feature Extraction):** 각 이미지에서 SIFT와 같은 알고리즘을 사용하여 고유한 특징점(코너, 엣지 등)을 수백 개에서 수천 개씩 추출합니다.
2. **특징점 매칭(Feature Matching):** 이미지 쌍 간에 동일한 3D 지점에서 비롯된 것으로 보이는 특징점들을 매칭합니다.
3. **희소 재구성(Sparse Reconstruction):** 매칭된 특징점 정보를 기반으로, 광속 조정(Bundle Adjustment)이라는 최적화 기법을 사용하여 모든 카메라의 포즈와 3D 특징점의 위치를 동시에 계산합니다. 그 결과물은 3D 공간에 흩어져 있는 점들의 집합인 희소 포인트 클라우드와 각 카메라의 정밀한 파라미터입니다.


이 단계는 NuRec 기술의 핵심으로, COLMAP에서 얻은 희소 포인트 클라우드와 카메라 파라미터를 입력으로 받아 장면을 조밀하고 사실적인 신경 표현(neural representation)으로 변환합니다.

- **3D Gaussian Splatting (3DGS):** 3DGS는 장면을 메시나 복셀이 아닌, 수백만 개의 3D 가우시안(Gaussian)으로 표현하는 혁신적인 렌더링 기법입니다.39 각 3D 가우시안은 다음과 같은 속성을 가집니다.

  - **위치(Position):** 3D 공간상의 중심점 $(x, y, z)$.

  - **공분산(Covariance):** 3D 타원체의 형태와 방향을 결정하는 3x3 행렬. 이는 가우시안의 크기와 회전을 나타냅니다.

  - **색상(Color):** RGB 값.

  - 불투명도(Opacity): 알파(Alpha) 값.

    이 가우시안들은 미분 가능한 래스터화 파이프라인, 즉 '스플래팅(splatting)'을 통해 2D 이미지 평면에 투영됩니다. 이 과정은 훈련을 통해 원본 이미지와 렌더링된 이미지 간의 오차를 최소화하도록 가우시안들의 모든 속성을 최적화합니다. 이 명시적인 표현 방식 덕분에 3DGS는 NeRF(Neural Radiance Fields)와 같은 암시적(implicit) 표현 방식보다 훨씬 빠른 훈련과 실시간 렌더링이 가능합니다.42

- **3D Gaussian Unscented Transforms (3DGUT):** 3DGUT는 NVIDIA Research에서 개발한 3DGS의 확장 기술입니다.38 기존의 3DGS가 주로 이상적인 핀홀(pinhole) 카메라 모델을 가정하는 반면, 3DGUT는 무향 변환(Unscented Transform)을 사용하여 어안 렌즈나 롤링 셔터 왜곡과 같은 복잡하고 비선형적인 카메라 모델을 통해 3D 가우시안을 더 정확하게 투영합니다. 이를 통해 실제 산업 현장에서 사용되는 다양한 센서 데이터로부터 더 높은 충실도의 재구성을 가능하게 합니다.39

이 단계의 결과물은 단순한 점들의 집합이 아닌, 장면의 형상과 외형을 매우 상세하게 담고 있는 고품질의 체적 표현(volumetric representation)입니다.


신경 재구성 모델의 훈련이 완료되면, 그 결과물은 Omniverse 생태계에서 표준으로 사용되는 OpenUSD 형식(일반적으로 `.usdz` 파일)으로 내보내집니다.38 이 USD 에셋은 Omniverse USD Composer와 같은 애플리케이션으로 직접 로드할 수 있으며, 다른 3D 모델과 마찬가지로 장면 내에서 이동, 회전, 크기 조절이 가능합니다. 이로써, 현실 세계의 한 단면이 디지털 트윈을 구성하는 기본 배경 요소로 준비됩니다.

이 NuRec 워크플로우는 로보틱스와 자율 시스템 개발의 근본적인 난제인 '심투리얼 갭'을 해소하는 촉매제 역할을 합니다. 기존에는 시뮬레이션 환경을 수작업으로 제작하는 데 막대한 시간과 비용이 소요되어, 훈련 시나리오의 규모와 다양성에 한계가 있었습니다. NuRec는 이 과정을 자동화하고 극적으로 가속화함으로써, 개발자가 로봇이 실제 배치될 환경 자체를 빠르고 정확하게 포착하여 사실적인 시뮬레이션 공간으로 전환할 수 있게 합니다. 이는 '배포 → 데이터 캡처 → 디지털 트윈 업데이트 → 시뮬레이션 기반 재훈련 → 재배포'로 이어지는 강력하고 신속한 피드백 루프를 형성하며, 이를 통해 AI 모델의 성능을 지속적으로 개선하고 현실 세계에서의 강건함(robustness)을 확보할 수 있습니다.


NuRec으로 재구성된 배경은 시작에 불과합니다. 진정한 고충실도 디지털 트윈은 시각적 복제를 넘어, 엔지니어링 데이터와 물리 법칙을 통합하여 상호작용이 가능한 동적인 환경이 되어야 합니다.


디지털 트윈은 종종 특정 기계, 로봇, 또는 제품과 상호작용하는 환경을 모사합니다. 이러한 대상들은 대부분 CAD 소프트웨어로 정밀하게 설계됩니다. Omniverse는 강력한 CAD 변환기 확장 기능을 통해 이러한 엔지니어링 데이터를 원활하게 통합할 수 있는 경로를 제공합니다.9

- **CAD 변환기 활용:** Omniverse는 JT, STEP, IGES, SolidWorks(`.SLDPRT`, `.SLDASM`), CATIA 등 광범위한 산업 표준 CAD 형식을 지원하는 `omni.kit.converter.cad` 확장을 제공합니다.15 사용자는 

  `파일 > 가져오기(File > Import)` 메뉴를 통해 CAD 파일을 직접 스테이지로 가져오거나, 콘텐츠 브라우저에서 파일을 우클릭하여 USD로 변환할 수 있습니다.

- **가져오기 옵션:** 두 가지 주요 워크플로우가 있습니다. '스테이지로 가져오기(Import to Stage)'는 CAD 데이터를 현재 열려 있는 USD 장면에 직접 변환하여 삽입하므로, 데이터를 즉시 편집하고 최적화하는 데 유리합니다. 반면, '참조로 가져오기(Import as Reference)'는 CAD 파일을 별도의 USD 파일로 변환한 후, 현재 장면에 참조(reference) 형태로 포함시킵니다. 이는 원본 CAD 데이터의 변경 사항이 있을 때 쉽게 업데이트할 수 있는 비파괴적 워크플로우에 더 적합합니다.14

- **정렬 및 배치:** 가져온 CAD 에셋(예: 로봇 모델)은 NuRec으로 생성된 배경 씬 내에서 실제 위치와 방향에 맞게 정확하게 배치되어야 합니다. 이는 Omniverse 조작 도구를 사용하여 수동으로 수행하거나, 실제 세계의 좌표계를 기준으로 자동 정렬할 수 있습니다.

**표 2: Omniverse 지원 CAD 및 3D 형식**

고충실도 디지털 트윈은 다양한 소스의 데이터를 통합하는 능력에 크게 의존합니다. 이 표는 Omniverse의 주요 가져오기 및 변환 확장에서 지원하는 파일 형식들을 정리하여 플랫폼의 광범위한 상호 운용성을 보여줍니다.

| 확장 기능                | 지원 형식                                                    | 설명                                                         |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **CAD 변환기**           | CATIA, SolidWorks, NX, Creo, STEP, IGES, JT, Parasolid, Revit, DWG, DXF 등 15 | 정밀한 경계 표현(B-rep) 방식의 엔지니어링 데이터를 시뮬레이션에 사용하기 위한 폴리곤 메시 형태의 USD로 변환합니다. |
| **에셋 임포터**          | FBX, OBJ, glTF, DAE 13                                       | 미디어, 엔터테인먼트, 시각화 분야에서 널리 사용되는 일반적인 3D 모델 및 장면 형식을 가져옵니다. |
| **포인트 클라우드 통합** | PLY (Kaolin과 같은 라이브러리를 통한 사용자 정의 워크플로우) 45, Prevu3D의 RealityConnect와 같은 서드파티 플러그인 46 | 3D 스캐너 및 LiDAR 데이터를 통합합니다. 네이티브 지원이 확장되고 있으나, 현재는 중간 라이브러리나 플러그인을 활용하는 경우가 많습니다. |


시각적으로만 존재하는 디지털 트윈은 제한적입니다. 객체들이 서로 충돌하고, 중력의 영향을 받으며, 마찰력을 갖게 하려면 물리적 속성을 부여해야 합니다. NuRec으로 생성된 신경 렌더링 장면과 물리 엔진을 결합하는 과정은 독창적인 접근 방식을 필요로 합니다.

NuRec 장면 자체는 명시적인 표면 지오메트리가 없는 체적 렌더링 결과물이기 때문에, 전통적인 물리 엔진이 충돌을 계산하기 어렵습니다. 이 문제를 해결하기 위해 '프록시 메시(proxy mesh)'라는 개념이 도입됩니다.41 프록시 메시는 NuRec 장면의 형상을 대략적으로 따라가는 보이지 않는 단순한 지오메트리입니다. NVIDIA PhysX 엔진은 모든 물리 계산(충돌 감지, 강체 동역학 등)을 이 효율적인 프록시 메시와 상호작용하여 수행하고, 렌더링은 고품질의 NuRec 볼륨을 통해 이루어집니다. 이처럼 렌더링과 물리 계산을 분리하는 것은 시각적 충실도를 극대화하면서도 물리 시뮬레이션의 계산 효율성을 유지하기 위한 핵심적인 최적화 전략입니다.

실제 설정 과정은 다음과 같습니다.

1. NuRec 장면의 바닥, 벽 등 충돌이 필요한 영역에 해당하는 간단한 프록시 메시(예: 평면, 큐브)를 생성하고 배치합니다.
2. NuRec 프리미티브에 적용된 `OmniNuRecVolumeAPI` 스키마에서 'Proxy' 관계를 설정하여 이 프록시 메시를 연결합니다.41
3. 가져온 CAD 에셋(로봇)과 프록시 메시에 각각 물리 속성을 부여합니다. 여기에는 'Rigid Body' 컴포넌트를 추가하여 질량, 관성 등의 동적 속성을 설정하고, 'Collider' 컴포넌트를 추가하여 충돌 형태를 정의하며, 'Physics Material'을 통해 마찰 계수($\mu$), 반발 계수(restitution) 등을 지정하는 작업이 포함됩니다.


물리적 상호작용이 설정된 장면에 최종적인 시각적 사실감을 불어넣는 단계입니다.

- **물리 기반 조명:** Omniverse의 RTX 기반 패스 트레이서는 물리적으로 정확한 조명 시뮬레이션을 제공합니다.36 장면의 전반적인 분위기를 결정하는 간접 조명을 위해 돔 라이트(Dome Light)에 HDRI(High-Dynamic Range Image) 텍스처를 적용할 수 있습니다. 또한, CAD로 가져온 로봇과 같은 동적 객체가 NuRec 장면에 사실적인 그림자를 드리우게 하려면, 프록시 메시의 재질 설정에서 '매트 객체(Matte Object)' 옵션을 활성화해야 합니다. 이 옵션은 프록시 메시 자체는 렌더링하지 않으면서 그 위에 드리워진 그림자나 반사만을 NuRec 렌더링 결과 위에 합성해줍니다.41
- **MDL 재질:** CAD 에셋이 빛과 사실적으로 상호작용하도록 만들기 위해 NVIDIA의 재질 정의 언어(Material Definition Language, MDL)를 사용합니다.44 Omniverse는 금속, 플라스틱, 유리 등 다양한 물리 기반 재질 라이브러리를 제공하며, 사용자는 각 부품의 특성(색상, 거칠기, 금속성 등)에 맞는 MDL 재질을 적용하여 높은 수준의 사실감을 구현할 수 있습니다.

이 모든 과정을 거치면, 현실 세계의 스냅샷은 이제 물리 법칙을 따르며, 다른 디지털 에셋과 상호작용하고, 빛에 사실적으로 반응하는 살아있는 고충실도 디지털 트윈으로 완성됩니다.


완성된 고충실도 디지털 트윈은 단순한 시각화 모델을 넘어, AI 개발, 분석, 의사결정을 위한 강력한 도구로 활성화됩니다. 이 파트에서는 구축된 디지털 트윈을 실제 산업 문제 해결에 활용하는 고급 애플리케이션과 사례를 탐구합니다.


산업용 디지털 트윈의 궁극적인 목표 중 하나는 AI 기반 자율 시스템을 개발하고 테스트하기 위한 안전하고, 확장 가능하며, 비용 효율적인 환경을 제공하는 것입니다.


NVIDIA Isaac Sim은 Omniverse 플랫폼을 기반으로 구축된 로보틱스 시뮬레이션 애플리케이션입니다.24 Part II에서 구축한 디지털 트윈은 Isaac Sim 내에서 로봇이 작동할 가상 환경으로 직접 사용될 수 있습니다.

예를 들어, 공장 자동화를 위한 '픽 앤 플레이스(pick-and-place)' 로봇의 지능을 개발하는 시나리오를 생각해 볼 수 있습니다. NuRec으로 재구성된 사실적인 작업대와 부품 컨테이너 환경 속에서 로봇 팔 모델을 배치합니다. 그런 다음, Omniverse Replicator를 사용하여 훈련 데이터를 생성합니다. Replicator는 조명, 부품의 위치와 방향, 카메라 각도 등 장면의 속성을 절차적으로 무작위화(domain randomization)하여 수천, 수만 개의 다양한 학습용 합성 이미지를 생성할 수 있습니다.24 로봇의 AI 모델(예: 강화학습 에이전트)은 이 데이터를 사용하여 시뮬레이션 환경 내에서 수많은 시도를 통해 물체를 성공적으로 집어 옮기는 정책(policy)을 학습합니다. NuRec으로 생성된 환경의 높은 시각적 충실도는 시뮬레이션에서 학습된 정책이 실제 로봇에 배포되었을 때 성공적으로 작동할 확률을 높여, 심투리얼 갭을 최소화하는 데 결정적인 기여를 합니다.4


Omniverse NuRec는 오픈소스 자율주행 시뮬레이터인 CARLA와 통합되어, 자율주행차(AV) 개발 워크플로우를 가속화합니다.38 개발자들은 테스트 차량 플릿(fleet)에서 수집한 실제 주행 데이터(비디오, LiDAR 등)를 사용하여 복잡하고 위험한 실제 도로 환경을 NuRec으로 재구성할 수 있습니다.

이렇게 재구성된 디지털 트윈 환경은 CARLA 시뮬레이터로 로드됩니다. 개발자는 이 환경에서 실제 주행 시나리오를 재현하면서, 가상의 다른 차량이나 보행자를 추가로 삽입하여 AV의 인식 및 계획 스택이 위험한 엣지 케이스(edge case)에 어떻게 반응하는지 테스트할 수 있습니다.39 또한, 원본 데이터에는 없었던 새로운 시점(예: 다른 차선에 있는 차량의 시점)에서 센서 데이터를 렌더링하여 데이터 증강(data augmentation)에 활용할 수도 있습니다. 이는 실제 도로 테스트의 위험과 비용 없이, AV 시스템의 안전성과 강건함을 검증할 수 있는 강력한 방법을 제공합니다.


디지털 트윈은 AI뿐만 아니라 인간과의 상호작용을 통해서도 그 가치를 발휘합니다. Omniverse는 고충실도 디지털 트윈을 Apple Vision Pro와 같은 XR 디바이스로 스트리밍하여, 사용자가 가상 환경에 몰입하여 직관적인 상호작용을 할 수 있도록 지원합니다.


매우 복잡하고 사실적인 디지털 트윈을 경량 XR 헤드셋에서 직접 렌더링하는 것은 불가능합니다. Omniverse는 '하이브리드 렌더링(hybrid rendering)' 접근 방식을 통해 이 문제를 해결합니다.11 강력한 클라우드 GPU나 로컬 워크스테이션에서 모든 복잡한 렌더링 연산을 수행하고, 그 결과 비디오 스트림을 네트워크를 통해 XR 디바이스로 전송합니다. XR 디바이스는 이 스트림을 수신하여 사용자의 머리 움직임에 맞게 표시해주기만 하면 됩니다.

이 워크플로우를 설정하는 주요 단계는 다음과 같습니다 11:

1. **Omniverse Kit 애플리케이션 생성:** 스트리밍 서버 역할을 할 Omniverse 애플리케이션을 생성합니다.
2. **XR 확장 기능 추가:** 공간 스트리밍에 필요한 모든 구성 요소를 포함하는 XR 확장 번들을 활성화합니다.
3. **클라이언트-서버 연결:** XR 디바이스의 클라이언트 애플리케이션이 Omniverse 서버에 연결되도록 설정합니다. 통신은 Omniverse의 ActionGraph 로직을 통해 관리됩니다.


이 기술의 실제 활용 사례로, 공장 설계팀이 Apple Vision Pro를 착용하고 건설 전인 공장의 1:1 스케일 디지털 트윈 내부를 가상으로 걸어 다니는 모습을 상상할 수 있습니다. 팀원들은 새로 설치될 기계 장비의 배치가 적절한지, 로봇의 작업 반경이 다른 설비와 충돌하지 않는지, 작업자의 동선이 효율적이고 안전한지 등을 직관적으로 검토할 수 있습니다.11 이러한 몰입형 경험은 2D 도면이나 화면상 3D 모델로는 파악하기 어려운 공간적 문제들을 조기에 발견하고 해결하여, 실제 건설 단계에서 발생할 수 있는 값비싼 재작업을 방지하는 데 큰 도움이 됩니다.


Omniverse 기반 디지털 트윈은 이미 여러 글로벌 선도 기업들에 의해 실제 산업 현장에 적용되어 가시적인 성과를 내고 있으며, 생성형 AI와의 융합을 통해 그 가능성을 더욱 확장하고 있습니다.


- **BMW 그룹의 가상 공장:** BMW는 Omniverse를 사용하여 실제 공장을 건설하기 수년 전부터 가상 공간에서 전체 공장을 설계, 시뮬레이션, 최적화합니다. 전 세계에 흩어져 있는 설계, 로봇, 물류 팀이 하나의 가상 공장 디지털 트윈에 접속하여 협업함으로써, 공장 레이아웃과 생산 흐름의 효율성을 극대화합니다. BMW는 이 방식을 통해 계획 효율성을 30% 향상시켰다고 보고했으며, 이는 물리적 프로토타입 제작과 현장 변경 작업에 드는 막대한 비용과 시간을 절감하는 효과를 보여줍니다.6
- **삼성전자의 팹 디지털 트윈:** 삼성전자는 2030년까지 완전 자동화된 반도체 팹(Fab)을 구축하는 것을 목표로, Omniverse 기반 디지털 트윈 플랫폼을 개발하고 있습니다.20 이 디지털 트윈은 수백만 개의 부품으로 구성된 극도로 복잡한 팹 환경을 가상으로 복제하여, 장비 배치, 웨이퍼 물류, 공정 수율 등을 시뮬레이션을 통해 최적화하는 데 사용됩니다. 삼성전자는 이 거대한 디지털 트윈 모델의 로딩 시간을 5분 이내로 단축하여 실시간 분석 및 의사결정을 지원하는 것을 핵심 기술 목표로 삼고 있으며, 이는 대규모 산업 디지털 트윈의 실용성을 결정하는 중요한 과제임을 시사합니다.20


지금까지의 디지털 트윈은 주로 '현실을 복제'하는 데 중점을 두었습니다. 그러나 디지털 트윈의 미래는 생성형 AI와의 결합을 통해 '현실을 창조하고 확장'하는 방향으로 나아가고 있습니다.

이 비전의 중심에는 **NVIDIA Cosmos**와 같은 월드 파운데이션 모델(World Foundation Model, WFM)이 있습니다.40 Cosmos는 텍스트 프롬프트나 기타 입력 데이터를 기반으로, 물리적으로 그럴듯하고 다양한 3D 환경, 객체, 시나리오를 생성할 수 있는 거대 AI 모델입니다.

미래의 개발자들은 NuRec으로 현실 세계를 캡처하는 것을 넘어, Cosmos를 사용하여 기존 환경을 변형하거나(예: "이 장면에 비를 내리게 하라"), 완전히 새로운 테스트 시나리오를 생성할 수 있게 될 것입니다(예: "안개가 낀 새벽의 교차로를 생성하라"). 이는 AI 훈련과 검증에 필요한 데이터를 거의 무한에 가깝게 생성할 수 있음을 의미하며, 물리 AI(Physical AI) 모델의 능력을 기하급수적으로 발전시킬 잠재력을 가지고 있습니다.38

결론적으로, NVIDIA Omniverse와 NuRec으로 대표되는 고충실도 디지털 트윈 기술은 현실 세계를 정밀하게 복제하고 시뮬레이션하는 단계를 넘어, AI와의 깊은 융합을 통해 산업의 설계, 운영, 최적화 방식을 근본적으로 혁신하고 있습니다. 이는 과거의 현실을 담는 거울에서 미래의 가능성을 생성하는 엔진으로 진화하는 디지털 트윈의 새로운 여정을 예고합니다.


1. 시대 상황과 디지털 트윈(Digital Twin)의 반영 한계: 올바른 이해 및 활용 방향, 8월 20, 2025에 액세스, https://www.unipress.co.kr/news/articleView.html?idxno=5300
2. '디지털 트윈' 알아보기 (2) | NVIDIA Blog, 8월 20, 2025에 액세스, https://blogs.nvidia.co.kr/blog/what-is-a-digital-twin-2/
3. 시대 상황과 디지털 트윈의 반영 한계 : 올바른 이해 및 활용 방향 - KISTI Institutional Repository - 한국과학기술정보연구원, 8월 20, 2025에 액세스, [https://repository.kisti.re.kr/bitstream/10580/16943/1/KISTI%20%EC%9D%B4%EC%8A%88%EB%B8%8C%EB%A6%AC%ED%94%84%20%EC%A0%9C39%ED%98%B8.pdf](https://repository.kisti.re.kr/bitstream/10580/16943/1/KISTI 이슈브리프 제39호.pdf)
4. Omniverse - Pollux AI, 8월 20, 2025에 액세스, https://pollux.ai/ko/physical_ai/omniverse
5. NVIDIA Omniverse Cloud 확장으로 한 걸음 더 가까워진 산업 디지털화, 8월 20, 2025에 액세스, https://blogs.nvidia.co.kr/blog/nvidia-expands-omniverse-cloud-to-power-industrial-digitalization/
6. 메타버스란 무엇인가? | NVIDIA Blog, 8월 20, 2025에 액세스, https://blogs.nvidia.co.kr/blog/what-is-the-metaverse/?nv_excludes=6957,6963
7. en.wikipedia.org, 8월 20, 2025에 액세스, https://en.wikipedia.org/wiki/Nvidia_Omniverse
8. 3D의 미래를 만들어 가는 NVIDIA Omniverse 소개 - Render Farm, 8월 20, 2025에 액세스, https://garagefarm.net/ko-blog/how-the-nvidia-omniverse-is-shaping-the-future-of-3d
9. Building CAD to USD Workflows with NVIDIA Omniverse, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/building-cad-to-usd-workflows-with-nvidia-omniverse/
10. Third Party Connectors — Omniverse Connect, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/connect/latest/3rd-party-connectors.html
11. NVIDIA Omniverse 공간 스트리밍으로 XR에서 디지털 트윈을 경험하세요., 8월 20, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/experience-digital-twins-in-xr-with-nvidia-omniverse-spatial-streaming/
12. Connecting to the NVIDIA Omniverse collaboration platform - Vectorworks, 8월 20, 2025에 액세스, https://app-help.vectorworks.net/2025/eng/VW2025_Guide/Rendering2/Connecting_to_the_NVIDIA_Omniverse_collaboration_platform.htm
13. Asset Importer — Omniverse Extensions, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/extensions/latest/ext_asset-importer.html
14. Data Exchange — Data Aggregation and Navigation Guide - NVIDIA Omniverse, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/dang/latest/guide/connect.html
15. User Manual - CAD Converter - NVIDIA Omniverse, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/extensions/latest/ext_cad-converter/manual.html
16. NVIDIA Omniverse Cloud, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/omniverse/cloud/
17. 디지털 트윈 기술이란 무엇인가요? - AWS, 8월 20, 2025에 액세스, https://aws.amazon.com/ko/what-is/digital-twin/
18. 에머슨, 디지털트윈을 정의하는 7가지 필수 단계를 말하다 - 인더스트리뉴스, 8월 20, 2025에 액세스, https://www.industrynews.co.kr/news/articleView.html?idxno=34689
19. 메타버스=옴니버스... 젠슨 황 "기적같은 플랫폼" - 더밀크 | The Miilk, 8월 20, 2025에 액세스, https://www.themiilk.com/articles/acabd5c97
20. 삼성전자의 Omniverse 기반 Fab 디지털 트윈 플랫폼: 반도체 산업 혁신, 8월 20, 2025에 액세스, https://www.pollux.ai/ko/industry/news/samsung-electronics-fab-digital-twin-platform-based-on-omniverse-semiconductor-industry
21. 산업용 메타버스를 위한 디지털 트윈 구축 - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/omniverse/digital-twins/siemens/
22. 산업 시설 디지털 트윈 - 사용 사례 - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/use-cases/industrial-facility-digital-twins/
23. 디지털 트윈 시작을 위한 5단계 E-BOOK - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/omniverse/solutions/digital-twins/5-steps-to-get-started-ebook/
24. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 20, 2025에 액세스, https://developer.nvidia.com/isaac/sim
25. [PhysX] 5화 Material - Keep it simple, Stupid !!! - 티스토리, 8월 20, 2025에 액세스, [https://mocamilk.tistory.com/entry/PhysX-5%ED%99%94-Material](https://mocamilk.tistory.com/entry/PhysX-5화-Material)
26. 물리 엔진에 관한 고찰 : 실시간 물리 기술을 중심으로※, 8월 20, 2025에 액세스, https://www.nl.go.kr/NL/onlineFileIdDownload.do?fileId=FILE-00010379568
27. 강체의 운동방정식, 8월 20, 2025에 액세스, https://nisl.kau.ac.kr/Lec_Flt_Dyns_1.pdf
28. 강체의 운동방정식 - 1 - DeepCampus - 티스토리, 8월 20, 2025에 액세스, https://pasus.tistory.com/189
29. 파티클 동역학 - 컴공생 일상 블로그, 8월 20, 2025에 액세스, https://doraeul19.tistory.com/259
30. 강체의 운동방정식 - 4 - DeepCampus - 티스토리, 8월 20, 2025에 액세스, https://pasus.tistory.com/192
31. 강체 역학 - 나무위키, 8월 20, 2025에 액세스, [https://namu.wiki/w/%EA%B0%95%EC%B2%B4%20%EC%97%AD%ED%95%99](https://namu.wiki/w/강체 역학)
32. [3D 수학] 3D 변환 행렬 - HIT - 티스토리, 8월 20, 2025에 액세스, https://showmiso.tistory.com/48
33. 3D 강체 변환(Rigid Body Transformation) 개념 정리 - A L I D A - 티스토리, 8월 20, 2025에 액세스, https://alida.tistory.com/78
34. (게임수학) 3차원 회전 변환 행렬 (유도하는 방법) - 달리는 개발자, 8월 20, 2025에 액세스, https://dev-sbee.tistory.com/30
35. [영상 Geometry #5] 3D 변환 - 다크 프로그래머 - 티스토리, 8월 20, 2025에 액세스, https://darkpgmr.tistory.com/81
36. OpenUSD용 Omniverse 플랫폼 | NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/omniverse/
37. 렌더링 방정식 - noriwiki, 8월 20, 2025에 액세스, [https://junhoahn.kr/noriwiki/index.php/%EB%A0%8C%EB%8D%94%EB%A7%81_%EB%B0%A9%EC%A0%95%EC%8B%9D](https://junhoahn.kr/noriwiki/index.php/렌더링_방정식)
38. How to Instantly Render Real-World Scenes in Interactive ..., 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/how-to-instantly-render-real-world-scenes-in-interactive-simulation/
39. Accelerating AV Simulation with Neural Reconstruction and World ..., 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/accelerating-av-simulation-with-neural-reconstruction-and-world-foundation-models/
40. NVIDIA Opens Portals to World of Robotics With New Omniverse Libraries, Cosmos Physical AI Models and AI Computing Infrastructure, 8월 20, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-opens-portals-to-world-of-robotics-with-new-omniverse-libraries-cosmos-physical-ai-models-and-ai-computing-infrastructure
41. Neural (NuRec) Rendering - NVIDIA Omniverse, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/materials-and-rendering/latest/neural-rendering.html
42. What is 3D Reconstruction? | NVIDIA Glossary, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/glossary/3d-reconstruction/
43. Usage — omni.services.convert.cad - NVIDIA Omniverse, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/kit/docs/omni.services.convert.cad/latest/Usage.html
44. Formats — Omniverse USD, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/usd/latest/common/formats.html
45. Point Cloud에서 Kaolin를 활용한 Omniverse 시각화 - Pollux AI, 8월 20, 2025에 액세스, https://www.pollux.ai/ko/tech-blog/visualizing-point-cloud-data-in-omniverse-using-kaolin
46. RealityConnect™ Plugins - Prevu3D, 8월 20, 2025에 액세스, https://www.prevu3d.com/solutions/realityconnect/
47. 자율주행 자동차 개발 위한 NVIDIA Drive Sim의 새로운 AI 도구, 8월 20, 2025에 액세스, https://blogs.nvidia.co.kr/blog/drive-sim-omniverse-neural-ai-digital-twin/
48. NVIDIA의 디지털 트윈 Omniverse - 현실 세계를 시뮬레이션 하는 3D 환경 - TILNOTE - 틸노트, 8월 20, 2025에 액세스, https://tilnote.io/pages/65f97ecac9a979a33d1a0c32
49. NVIDIA Omniverse로 계속되는 BMW 그룹의 혁신, 8월 20, 2025에 액세스, https://blogs.nvidia.co.kr/blog/bmw-group-nvidia-omniverse_1/
50. Neural Reconstruction for Robotics and Autonomous Vehicles - YouTube, 8월 20, 2025에 액세스, https://www.youtube.com/watch?v=e3dfXj6ATA0
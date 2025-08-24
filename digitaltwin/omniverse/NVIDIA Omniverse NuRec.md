[NVidia Omniverse](./index.md)
# NVIDIA Omniverse NuRec 현실과 시뮬레이션의 융합



인공지능(AI) 기술은 가상 세계의 데이터를 처리하고 이해하는 단계를 넘어, 물리 법칙이 지배하는 현실 세계와 직접 상호작용하는 '피지컬 AI(Physical AI)'의 시대로 진입하고 있다.1 피지컬 AI는 현대 로보틱스, 자율주행차, 스마트 팩토리 및 지능형 인프라의 핵심 엔진으로, AI가 주변 환경을 인식하고, 이해하며, 그 안에서 행동하는 능력을 갖추는 패러다임의 전환을 의미한다.2 이러한 지능형 시스템은 현실 세계에서 직접 시행착오를 겪으며 학습하기에는 막대한 비용과 예측 불가능한 위험이 수반된다. 따라서 AI가 물리 법칙을 배우고 다양한 시나리오에 대한 대응 능력을 안전하게 연마할 수 있는 '평행 우주', 즉 물리적으로 정확하고 극도로 사실적인 가상 환경의 필요성이 절대적으로 대두된다.1 이 가상 환경, 즉 시뮬레이션은 더 이상 개발 후반 단계의 단순한 테스트 베드가 아니라, AI 모델의 성능, 강건성, 그리고 안전성을 결정하는 핵심적인 훈련장(training ground)으로 자리매김하고 있다.

이러한 배경 속에서 컴퓨터 그래픽스와 AI 기술의 융합은 로보틱스와 자율 시스템 분야를 근본적으로 변화시키는 원동력이 되고 있다. 확장 가능하고 물리적으로 정확한 시뮬레이션과 AI의 추론 능력을 결합함으로써, 개발자들은 이전에는 불가능했던 복잡성과 규모를 갖춘 차세대 로봇 및 자율주행차를 구축할 수 있게 되었다.3 그러나 이 비전을 실현하는 데 있어 가장 큰 병목 현상 중 하나는 바로 방대한 시뮬레이션 수요를 충족시킬 수 있는 고품질 3D 환경을 신속하게 제작하는 것이었다.5 전통적인 수작업 기반의 3D 모델링 방식은 시간과 비용 측면에서 한계가 명확하며, 이는 피지컬 AI 개발의 속도를 저해하는 심각한 제약 요인으로 작용해왔다. NVIDIA는 이러한 문제를 해결하고 피지컬 AI로의 전환을 가속화하기 위한 전략적 필연성의 산물로서 NuRec 기술을 제시한다. 이는 단순한 그래픽스 기술의 발전을 넘어, NVIDIA의 거시적인 AI 전략을 뒷받침하는 핵심 인프라 기술로서의 중요성을 가진다.


NVIDIA Omniverse는 3D 워크플로우의 근본적인 혁신을 목표로 설계된 개발 플랫폼이다.7 그 핵심에는 픽사(Pixar)에서 개발한 오픈 소스 프레임워크인 USD(Universal Scene Description)와 NVIDIA의 실시간 레이 트레이싱 기술인 RTX가 자리 잡고 있다.9 Omniverse의 철학은 세 가지 핵심 원칙에 기반한다. 첫째, USD를 통한 완벽한 데이터 상호 운용성이다. 서로 다른 3D 소프트웨어 도구와 애플리케이션 간의 데이터 사일로를 허물고, 파괴적인 데이터 변환 과정 없이 원활한 협업을 가능하게 한다.10 둘째, 실시간 동시 협업이다. Nucleus와 같은 서비스를 통해 지리적으로 분산된 팀원들이 여러 애플리케이션에서 동일한 3D 씬을 동시에 편집하고 검토할 수 있다.9 셋째, 물리적으로 정확한 대규모 시뮬레이션이다. RTX 렌더러와 PhysX 물리 엔진을 통해 실제 세계와 동일한 법칙이 적용되는 가상 세계를 구축하고, 이를 산업용 디지털 트윈, 제품 개발, 공정 최적화, 운영 효율성 혁신에 활용한다.8


NVIDIA Omniverse NuRec(Neural Reconstruction)은 Omniverse 플랫폼의 비전을 현실로 만드는 데 있어 가장 중요한 기술적 돌파구 중 하나다. NuRec은 다중 센서 데이터, 즉 여러 장의 사진이나 차량에 장착된 카메라 영상을 입력받아, 신경망을 통해 현실 세계를 극도로 사실적인 3D 가상 환경으로 재구성하는 기술 라이브러리 및 API의 집합이다.5 NuRec의 가장 큰 의의는 속도에 있다. 전통적으로 숙련된 아티스트가 수일에서 수주에 걸쳐 수행하던 3D 환경 제작 작업을 거의 즉각적으로(instantly) 완료할 수 있도록 워크플로우를 혁신했다.5

NuRec은 단순한 3D 스캐닝이나 모델링 도구를 넘어선다. 이는 시뮬레이션의 사실성을 극대화하여 시뮬레이션 환경과 현실 세계 간의 차이, 즉 'Sim-to-Real' 격차를 해소하는 데 핵심적인 역할을 한다. AI 모델이 NuRec으로 생성된 가상 환경에서 충분히 학습하면, 그 학습 결과가 현실 세계에서도 높은 성능으로 발현될 가능성이 커진다.3 결국 NuRec은 현실 세계를 Omniverse라는 디지털 우주로 흡수하는 빠르고 효율적인 포털(portal) 역할을 수행하며, 피지컬 AI와 산업 디지털화의 미래를 앞당기는 핵심 동력이라 할 수 있다.



NVIDIA Omniverse NuRec은 신경 재구성(Neural Reconstruction) 및 뉴럴 렌더링(Neural Rendering)을 위해 특별히 설계된 API와 소프트웨어 도구의 집합으로 정의된다.18 이 기술의 핵심 목표는 개발자가 이미 보유하고 있는 센서 데이터, 예를 들어 자율주행 차량 플릿(fleet)에서 수집된 카메라 영상이나 간단한 사진 촬영물을 활용하여, 상호작용이 가능한 고충실도의 디지털 트윈을 신속하게 생성하는 것이다. NuRec은 개발자가 재구성된 가상 환경 내에서 물리적으로는 불가능하거나 위험한 새로운 이벤트를 시뮬레이션하고, 가상의 카메라를 배치하여 새로운 시점(novel points of view)에서 센서 데이터를 렌더링할 수 있도록 지원한다.18

NuRec의 기능적 파이프라인은 크게 세 단계로 요약할 수 있다. 첫째, **데이터 준비 및 처리** 단계로, 다양한 소스로부터 입력된 센서 데이터를 재구성에 적합한 형태로 정제하고 준비한다. 둘째, **3D 재구성** 단계로, 처리된 데이터를 기반으로 장면의 기하학과 외형을 표현하는 3D 볼륨 표현을 생성한다. 셋째, **가우시안 기반 렌더링** 단계로, 생성된 3D 표현을 Omniverse의 렌더링 파이프라인과 연결하여 시뮬레이션 환경 내에서 시각화하고 상호작용할 수 있도록 만든다.18


NuRec은 Omniverse 생태계 내에서 현실 세계와 가상 세계를 잇는 핵심적인 가교 역할을 수행한다. 그 중심에는 OpenUSD가 있다. NuRec 워크플로우의 최종 산출물은 항상 USD 또는 USDZ 포맷의 3D 에셋이다.5 이 USD 에셋은 Omniverse 생태계의 표준 데이터 형식으로서, 다양한 애플리케이션과 도구 간의 완벽한 상호 운용성을 보장하는 '디지털 청사진'과 같다.7 NuRec은 이질적인 현실 세계의 센서 데이터를 입력받아, Omniverse 생태계가 이해하고 활용할 수 있는 표준화된 USD 에셋으로 '번역'하는 역할을 수행한다. 따라서 NuRec의 아키텍처적 역할은 단순히 3D 모델을 만드는 것을 넘어, 현실 세계와 Omniverse 가상 세계 사이의 데이터 흐름을 자동화하고 표준화하는 파이프라인 그 자체에 있다. 이는 Omniverse 생태계의 확장성과 실용성을 극대화하는 매우 중요한 전략적 포지셔닝이다.

일단 USD 에셋으로 변환되면, 이 재구성된 환경은 Omniverse의 핵심 시뮬레이션 애플리케이션으로 원활하게 통합된다. 로보틱스 개발 및 검증을 위한 **NVIDIA Isaac Sim**과 자율주행 연구 및 개발을 위한 오픈소스 시뮬레이터인 **CARLA**가 대표적인 예다.3 개발자는 NuRec으로 생성된 USD 씬을 Isaac Sim이나 CARLA에 직접 로드하여, 그 안에서 로봇이나 자율주행 차량을 즉시 테스트하고 검증할 수 있다.3 이처럼 NuRec은 Omniverse 플랫폼을 구성하는 여러 SDK, API, 마이크로서비스 중 하나로서, 개발자가 3D 애플리케이션을 구축하는 데 사용하는 핵심 빌딩 블록(building block) 역할을 한다.7


NuRec 시스템은 기능적으로 세 가지 핵심 구성 요소로 나눌 수 있다.

1. **데이터 전처리 및 수집 라이브러리:** 성공적인 3D 재구성은 양질의 입력 데이터에서 시작된다. NuRec은 이 과정을 용이하게 하기 위해 Voxel51의 FiftyOne과 같은 외부 데이터 엔진과의 통합을 지원한다.3 이를 통해 개발자는 자체적으로 수집한 대규모 데이터셋을 효율적으로 관리하고, 재구성에 적합한 데이터를 선별하며, 재구성 결과물의 품질을 정량적으로 평가할 수 있다. 이 라이브러리들은 다운스트림 시뮬레이션 작업에 사용될 고품질 디지털 트윈을 생성하기 위한 첫 단계를 담당한다.18

2. **3D 재구성 엔진:** NuRec의 심장부로, 2D 이미지나 센서 데이터로부터 3D 장면을 표현하는 볼륨 데이터를 생성한다. 이 엔진은 3D 가우시안 스플래팅(3DGS)과 이를 NVIDIA가 확장한 3DGUT와 같은 최첨단 알고리즘을 기반으로 한다. 재구성된 볼륨 데이터는 `.nurec`과 같은 특수 포맷으로 저장될 수 있으며, 이는 장면의 기하학적 구조와 외형 정보를 압축적으로 담고 있다.22

3. **뉴럴 렌더링 API (NuRec Rendering):** 재구성된 3D 볼륨 데이터를 시각화하는 역할을 담당한다. 이 API는 Omniverse의 핵심 렌더러인 RTX 렌더러 내에 깊숙이 통합되어 있다. 이를 통해 NeRF, 3DGS, 3DGUT와 같은 다양한 신경 표현(neural primitives)을 기존의 메시(mesh) 기반 3D 에셋과 동일한 씬 안에서 함께 렌더링할 수 있다.23 이 기능은 Omniverse Kit의 확장(extension) 형태로 제공되며, 개발자는 USD의 

   `OmniNuRecVolumeAPI` 스키마를 통해 렌더링 경계, 프록시 지오메트리, 색상 보정 등 다양한 렌더링 속성을 정밀하게 제어할 수 있다.23



NuRec의 기술적 기반을 이해하기 위해서는 먼저 뉴럴 렌더링과 역 렌더링의 개념을 파악해야 한다. 전통적인 컴퓨터 그래픽스의 핵심 과제는 **순방향 렌더링(Forward Rendering)**으로, 이는 잘 정의된 3D 모델(기하학, 재질, 조명 정보 포함)로부터 2D 이미지를 생성하는 과정이다.1 반면, **역 렌더링(Inverse Rendering)**은 그 반대 방향의 문제로, 한 장 또는 여러 장의 2D 이미지로부터 3D 장면을 구성하는 내재적 속성들, 즉 기하학(geometry), 재질(materials), 조명(illumination)을 추론하는 훨씬 더 어렵고 비정형적인(ill-posed) 문제다.1 NuRec은 본질적으로 이 역 렌더링 문제에 대한 실용적이고 효율적인 해법을 제시한다.

**뉴럴 렌더링**은 이 과정을 딥러닝 모델, 즉 신경망을 사용하여 해결하는 접근 방식이다. 대표적인 초기 기술인 NeRF(Neural Radiance Fields)는 3D 장면을 연속적인 함수로 모델링한다. 이 함수는 3D 공간 좌표 $(x, y, z)$와 2D 시선 방향 $(\theta, \phi)$을 입력받아, 해당 지점의 색상(RGB)과 밀도(density) 값을 출력하는 다층 퍼셉트론(MLP)으로 구현된다.25 볼륨 렌더링 기법을 통해 이 함수를 적분함으로써, 어떤 새로운 시점에서든 사실적인 이미지를 합성할 수 있다. NuRec은 NeRF의 아이디어를 계승하면서도, 실시간 성능과 편집 용이성을 위해 다른 표현 방식을 채택했다.


3D 가우시안 스플래팅(3DGS)은 NuRec의 핵심 알고리즘으로, NeRF와 달리 장면을 명시적(explicit) 프리미티브의 집합으로 표현하여 혁신적인 성능을 달성했다.27

- **3D 가우시안 표현:** 3DGS는 장면을 수백만 개의 3D 가우시안(Gaussian ellipsoids)으로 모델링한다.29 각 가우시안은 다음과 같은 파라미터로 수학적으로 정의된다.

  - **위치(Position/Mean) $\mu \in \mathbb{R}^3$**: 3D 공간에서의 가우시안 중심 좌표(XYZ)이다.30
  - **공분산(Covariance) $\Sigma \in \mathbb{R}^{3 \times 3}$**: 3D 타원체의 크기(scaling)와 방향(rotation)을 결정하는 3x3 대칭 양의 정부호 행렬(symmetric positive definite matrix)이다. 이 행렬을 통해 가우시안은 등방성(isotropic, 구 형태)뿐만 아니라 이방성(anisotropic, 타원체 형태)을 가질 수 있어, 평평한 표면이나 긴 구조물을 더 적은 수의 프리미티브로 효율적으로 표현할 수 있다.29
  - **색상(Color) $c$**: 가우시안의 기본 색상 정보이다. 시선 방향에 따라 색상이 변하는 반사 효과 등을 표현하기 위해, 단순 RGB 값이 아닌 구면 조화 함수(Spherical Harmonics, SH)의 계수를 저장한다.31
  - **불투명도(Opacity) $\alpha \in $**: 가우시안의 투명도를 나타내는 스칼라 값이다.30

- **미분 가능한 래스터화(Differentiable Rasterization):** 최적화를 위해, 3D 가우시안을 2D 이미지 평면에 투영("splatting")하고 렌더링하는 전 과정이 미분 가능해야 한다. 이 과정은 다음과 같다.

  1. 월드 좌표계의 3D 공분산 행렬 $\Sigma$를 카메라 뷰 행렬 $W$와 투영 행렬 $P$를 이용해 2D 이미지 평면의 공분산 행렬 $\Sigma'$로 변환한다.

  2. 장면의 모든 가우시안을 깊이(depth) 순으로 정렬한다.

  3. 각 픽셀에 대해, 정렬된 가우시안들을 앞에서부터 순서대로 순회하며 색상을 혼합(alpha blending)한다. 픽셀 $p$의 최종 색상 $C(p)$는 다음 식으로 계산된다.
     $$
     C(p) = \sum_{i \in N} c_i \alpha_i' \prod_{j=1}^{i-1} (1 - \alpha_j')
     $$
     여기서 $N$은 정렬된 가우시안들의 집합, $c_i$는 $i$번째 가우시안의 색상, $\alpha_i'$는 2D 평면에 투영된 가우시안이 픽셀 $p$에 기여하는 계산된 불투명도이다.29

- **최적화 및 적응형 밀도 제어:** 렌더링된 이미지와 실제 촬영된 이미지(ground truth) 간의 차이를 손실 함수로 정의하고, 이를 최소화하기 위해 확률적 경사 하강법(SGD)을 사용하여 모든 가우시안의 파라미터($\mu, \Sigma, c, \alpha$)를 업데이트한다.29 이 과정에서 **적응형 밀도 제어(adaptive density control)**가 핵심적인 역할을 한다. 특정 주기마다 모델은 장면을 분석하여, 재구성이 부족한 영역(under-reconstructed)에는 가우시안을 복제(clone)하거나 분할(split)하여 디테일을 추가하고, 불필요한 영역(e.g., 투명도가 매우 낮은 가우시안)은 제거(prune)하여 효율성을 유지한다.30


3DGS의 최적화 과정은 렌더링된 이미지 $\hat{I}$와 실제 이미지 $I$ 간의 시각적 차이를 정량화하는 손실 함수 $\mathcal{L}$을 최소화하는 방향으로 진행된다. 이 손실 함수는 두 가지 주요 구성 요소의 가중 합으로 이루어져 있다.29
$$
\mathcal{L} = (1 - \lambda) \mathcal{L}_{L1} + \lambda \mathcal{L}_{D-SSIM}
$$

- **$\mathcal{L}_{L1}$ (L1 손실):** 이는 픽셀 단위의 절대 오차 합으로, $\mathcal{L}_{L1} = \sum_{p} |\hat{I}(p) - I(p)|$로 계산된다. L1 손실은 이상치(outlier)에 덜 민감하며, 렌더링된 이미지의 전반적인 색상과 밝기가 실제 이미지와 유사해지도록 유도하는 역할을 한다.34
- **$\mathcal{L}_{D-SSIM}$ (D-SSIM 손실):** SSIM(Structural Similarity Index Measure)은 인간의 시각 시스템이 이미지 품질을 평가하는 방식을 모방하여 밝기(luminance), 대비(contrast), 구조(structure)의 세 가지 요소를 비교하는 지표다. D-SSIM은 $(1 - \text{SSIM}(\hat{I}, I))/2$로 정의되며, 이 값을 최소화하는 것은 두 이미지의 구조적 유사성을 최대화하는 것과 같다. L1이나 L2(MSE) 손실이 픽셀 값의 차이에만 집중하여 이미지를 흐릿하게 만드는 경향이 있는 반면, D-SSIM 손실은 질감, 경계선 등 고주파 디테일을 보존하고 사실적인 구조를 유지하는 데 매우 중요한 역할을 한다.32
- **$\lambda$**: 두 손실 항의 상대적 중요도를 조절하는 하이퍼파라미터로, 원본 논문에서는 0.2로 설정되었다. 이 조합을 통해 3DGS는 전반적인 색상 정확도와 미세한 구조적 디테일을 모두 효과적으로 학습할 수 있다.


NVIDIA는 표준 3DGS 기술을 자사의 플랫폼과 목표 애플리케이션에 최적화하기 위해 몇 가지 핵심적인 확장 기술을 개발했다.

- **3DGUT (3D Gaussian Unscented Transform):** 표준 3DGS는 이상적인 핀홀(pinhole) 카메라 모델을 가정하지만, 자율주행 차량이나 로봇에 사용되는 카메라는 종종 넓은 화각을 위한 어안 렌즈(fisyehe)나 빠른 움직임으로 인한 롤링 셔터(rolling shutter)와 같은 심각한 비선형 왜곡을 포함한다. 3DGUT는 제어 이론에서 사용되는 **무향 변환(Unscented Transform)**을 적용하여, 이러한 복잡하고 비선형적인 카메라 모델을 통해 3D 가우시안을 2D 이미지 평면에 더 정확하게 투영한다.5 이는 특히 광각 카메라 데이터로부터 고품질의 재구성을 달성하는 데 필수적이다.

- **3DGRT (3D Gaussian Ray Tracing):** 3DGS의 실시간 래스터화 성능과 광선 추적(Ray Tracing)의 물리적 정확도를 결합한 하이브리드 렌더링 기술이다. 이 접근 방식에서 1차 광선(primary rays, 카메라에서 직접 보이는 표면)은 빠른 래스터화를 통해 렌더링하고, 반사, 굴절, 정교한 그림자와 같은 2차 광선 효과(secondary effects)는 NVIDIA RTX 하드웨어의 RT 코어를 활용한 광선 추적으로 시뮬레이션한다.27 이를 통해 래스터화만으로는 표현하기 어려운 복잡한 광학 현상을 물리 기반으로 정확하게 렌더링하여 시뮬레이션의 사실성을 극대화할 수 있다. NVIDIA는 3DGRT와 3DGUT를 통합한 오픈소스 프로젝트인 

  **3DGRUT**를 공개하여 연구 및 개발 커뮤니티의 접근성을 높였다.37

NVIDIA가 NeRF 대신 3DGS를 NuRec의 핵심 기술로 채택한 것은 기술적 완벽성보다 산업 현장에서의 실용성을 우선시한 전략적 결정이다. NeRF는 종종 뷰 합성 품질 면에서 최고 수준으로 평가받지만, 느린 학습과 렌더링 속도는 실시간 디지털 트윈과 같은 산업용 애플리케이션에 치명적인 단점이다.27 반면 3DGS는 NeRF에 필적하는 시각적 품질을 유지하면서 실시간 렌더링이 가능하며, 가우시안이라는 명시적 프리미티브 덕분에 장면 편집이 용이하다.28 이러한 특성은 시뮬레이션 환경에서 가상 객체를 추가하거나 시나리오를 동적으로 변경해야 하는 산업 워크플로우의 요구사항과 완벽하게 부합한다. 이는 연구실의 기술을 산업 현장으로 이전하는 과정에서 발생하는 전형적인 실용주의적 트레이드오프를 보여주는 사례다.


| 특징 (Feature)                                        | **NuRec (3DGS/3DGUT 기반)**                                  | **NeRF (신경 방사 필드)**                                   | **전통적 사진측량 (Traditional Photogrammetry)**        |
| ----------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------- |
| **렌더링 속도 (Rendering Speed)**                     | 실시간 (Real-time) 28                                        | 느림 (Slow), 최적화 필요 27                                 | 실시간 (Real-time, 메시/텍스처 생성 후) 41              |
| **학습/처리 시간 (Training/Processing Time)**         | 빠름 (Fast) 28                                               | 매우 느림 (Very Slow) 27                                    | 중간~느림 (Medium to Slow) 41                           |
| **시각적 품질 (Visual Quality)**                      | 매우 높음, 사실적 (Very High, Photorealistic) 28             | 매우 높음, SOTA (Very High, State-of-the-art) 27            | 높음, 텍스처 품질에 의존 (High, Texture-dependent) 41   |
| **메모리/저장 공간 (Memory/Storage)**                 | 큼 (Large) 30                                                | 상대적으로 작음 (Relatively Small)                          | 중간 (Medium)                                           |
| **동적 장면 처리 (Dynamic Scene Handling)**           | 어려움, 별도 연구 필요 (Difficult, requires separate research) 29 | 어려움, 별도 연구 필요                                      | 불가능 (Impossible)                                     |
| **편집 용이성 (Editability)**                         | 높음 (명시적 표현) (High, explicit representation) 40        | 매우 낮음 (암시적 표현) (Very Low, implicit representation) | 높음 (메시 기반) (High, mesh-based)                     |
| **반사/투명 재질 (Reflective/Transparent Materials)** | 어려움 (Difficult)                                           | 상대적으로 우수 (Relatively better) 25                      | 매우 어려움 (Very Difficult) 25                         |
| **핵심 기술 (Core Technology)**                       | 미분 가능한 래스터화 (Differentiable Rasterization)          | 신경망, 볼륨 렌더링 (Neural Network, Volume Rendering)      | 특징점 매칭, 삼각측량 (Feature Matching, Triangulation) |


NVIDIA Omniverse NuRec은 현실 세계의 센서 데이터로부터 상호작용 가능한 시뮬레이션 환경을 생성하기까지 체계적인 워크플로우를 제공한다. 이 과정은 데이터 수집부터 최종 배포까지 여러 단계로 구성되며, 오픈소스 도구와 NVIDIA의 독자 기술이 유기적으로 결합되어 있다.


워크플로우의 첫 단계는 현실 세계를 디지털 정보로 변환하는 데이터 수집이다.

- **데이터 수집:** 일반적인 환경 재구성을 위해서는, 대상 장면의 모든 각도에서 충분한 시각적 중첩(overlap)을 가지도록 약 100장 내외의 고품질 사진을 촬영하는 것이 권장된다. 이때 적절한 조리개 값(e.g., f/8), 셔터 속도(e.g., 1/100s 이상), 초점 거리(e.g., 18mm) 등을 사용하여 선명하고 조명이 균일한 이미지를 확보하는 것이 중요하다.5 자율주행 시나리오의 경우, 차량에 장착된 다중 카메라 시스템의 데이터를 활용한다. 예를 들어, NVIDIA Physical AI 데이터셋의 재구성에 사용된 데이터는 전방 광각(120도), 전방 망원(30도), 좌우측 교차(120도), 후방 좌우측(70도) 등 총 6개의 카메라로부터 동시에 수집된 영상이다.22
- **희소 재구성:** 수집된 이미지 세트는 오픈소스 SfM(Structure-from-Motion) 및 MVS(Multi-View Stereo) 파이프라인인 **COLMAP**의 입력으로 사용된다.5 COLMAP은 이미지들 간의 특징점을 매칭하고 삼각측량 원리를 이용하여 각 이미지의 카메라 위치와 방향(포즈, poses)을 추정하고, 동시에 장면의 3차원 구조를 나타내는 희소 포인트 클라우드(sparse point cloud)를 생성한다. 이 결과물은 후속 3DGS 학습 단계에서 가우시안의 초기 위치와 카메라 파라미터로 사용되는 매우 중요한 초기값을 제공한다.


희소 재구성 단계에서 얻은 초기 데이터를 바탕으로, 장면을 고밀도의 사실적인 3D 표현으로 변환하는 학습 과정이 진행된다.

- **모델 학습:** COLMAP으로부터 얻은 카메라 포즈와 희소 포인트 클라우드를 초기값으로 사용하여 3DGUT 모델의 학습을 시작한다.5 이 과정에서 수백만 개의 3D 가우시안 파라미터(위치, 공분산, 색상, 불투명도)가 앞서 설명한 L1 + D-SSIM 손실 함수를 최소화하는 방향으로 반복적으로 최적화된다. NVIDIA는 이 학습 과정을 수행할 수 있는 `train.py` 스크립트를 `nv-tlabs/3dgrut` GitHub 리포지토리를 통해 제공하고 있다.5


학습이 완료된 3D 가우시안 씬은 Omniverse 생태계에서 표준 데이터 형식으로 활용될 수 있도록 USD 포맷으로 변환된다.

- **USDZ 변환:** `3dgrut` 리포지토리는 학습 과정 중에 `export_usdz.enabled=true` 옵션을 설정하여, 학습이 완료된 씬을 Omniverse에서 직접 로드하고 렌더링할 수 있는 USDZ 파일로 자동 변환하는 기능을 베타 버전으로 제공한다.23 이 USDZ 파일은 3D 가우시안 볼륨 데이터와 렌더링에 필요한 메타데이터를 포함하는 패키지다.
- **기존 에셋 변환:** 만약 이미 다른 도구를 통해 학습된 표준 3DGS `.ply` 모델을 보유하고 있다면, `3dgrut` 리포지토리에 포함된 `ply_to_usd.py` 유틸리티 스크립트를 사용하여 이를 Omniverse 호환 USDZ 포맷으로 변환할 수 있다.23


표준화된 USDZ 에셋은 Omniverse의 다양한 애플리케이션으로 손쉽게 배포되어 즉시 활용될 수 있다.

- **Isaac Sim / Omniverse USD Composer:** 생성된 USDZ 에셋은 다른 3D 모델 파일과 마찬가지로, Isaac Sim이나 USD Composer와 같은 Omniverse 애플리케이션의 뷰포트로 파일을 드래그 앤 드롭하거나, 'File > Import' 메뉴를 통해 간단하게 로드할 수 있다. 로드가 완료되면, 재구성된 씬은 즉시 렌더링되며, 개발자는 이 환경 안에 로봇이나 다른 가상 객체를 배치하여 시뮬레이션을 시작할 수 있다.5
- **CARLA Simulator:** 자율주행 시뮬레이터인 CARLA와의 통합을 위해서는 추가적인 설정이 필요하다. CARLA 디렉토리 내에서 제공되는 `install_nurec.sh` 셸 스크립트를 실행하여 NuRec 렌더링 기능을 활성화한다. 이후, 파이썬 API 예제 스크립트(`example_replay_recording.py`)를 사용하여 특정 USDZ 시나리오 파일을 로드하고 재구성된 주행 장면을 재생(replay)할 수 있다.5


3D 재구성 기술은 입력 데이터가 부족한 영역이나 복잡한 기하학 구조에서 시각적 결함, 즉 아티팩트(artifact)나 갭(gap)이 발생하는 내재적 한계를 가진다.18 NVIDIA는 이 문제를 해결하기 위해 AI 기반 후처리 솔루션인 

**NuRec Fixer**를 제공한다.

- **기반 기술 (Difix3D+):** NuRec Fixer는 NVIDIA Research가 CVPR 2025에서 발표한 **Difix3D+** 논문에 기반한 트랜스포머 아키텍처 모델이다.18 Difix3D+는 사전 학습된 2D 이미지 생성 확산 모델(diffusion model)을 단일 단계(single-step)로 미세 조정하여, 3D 재구성 모델이 렌더링한 이미지에 포함된 아티팩트를 제거하고 품질을 향상시키는 신경 인핸서(neural enhancer) 역할을 한다.46
- **작동 원리:** NuRec Fixer는 두 가지 방식으로 작동한다. 첫째, 재구성 학습 과정에서 생성된 가상의 학습 뷰(pseudo-training views)를 정제하여 3D 표현 자체의 품질을 근본적으로 향상시킨다. 둘째, 최종 렌더링(추론) 시점에 실시간 후처리 필터처럼 작동하여, 남아있는 미세한 아티팩트를 제거하고 전반적인 시각적 충실도를 높인다.46 이 기술 덕분에, 재구성된 장면에 대한 새로운 뷰 합성이 개방 루프 및 폐쇄 루프 시뮬레이션 워크플로우에 실용적으로 사용될 수 있을 만큼 신뢰성이 높아진다.18


NVIDIA는 개발자들이 NuRec 워크플로우를 신속하게 이해하고 활용할 수 있도록, 사전 재구성된 장면들을 포함하는 **NVIDIA Physical AI 데이터셋**을 Hugging Face를 통해 공개적으로 제공한다.18

- **구조 및 내용:** 이 데이터셋은 자율주행 시나리오를 중심으로 구성되어 있으며, 각 장면은 약 20초 길이의 주행 데이터를 재구성한 것이다. 데이터는 USDZ 파일 형태로 제공되며, 각 파일은 단순한 3D 모델이 아닌 시뮬레이션에 필요한 모든 정보를 담고 있는 포괄적인 패키지다. 여기에는 3D 가우시안 볼륨 데이터(`volume.nurec`), 학습된 신경망 가중치(`checkpoint.ckpt`), 도로망 정보가 담긴 OpenDRIVE 맵 파일(`map.xodr`), 지오메트리 메시(`mesh.ply`), 차량 및 센서의 궤적 데이터(`rig_trajectories.json`) 등이 포함된다.22
- **활용 방안:** 이 데이터셋은 개발자들에게 두 가지 중요한 가치를 제공한다. 첫째, NuRec 워크플로우의 최종 결과물이 어떤 형태와 구조를 가지는지 직접 확인할 수 있는 구체적인 예시가 된다. 둘째, 자체 데이터를 수집하고 처리하기 전에, 이 데이터셋을 사용하여 Isaac Sim이나 CARLA에서의 시뮬레이션 파이프라인을 미리 구축하고 테스트하는 테스트베드 역할을 한다.

이처럼 NuRec 워크플로우는 오픈소스 생태계와의 공생과 AI 기반 후처리라는 두 가지 핵심 전략을 통해 완성도를 높인다. COLMAP과 CARLA 같은 검증된 오픈소스 도구를 파이프라인에 적극적으로 통합함으로써 개발 속도를 높이고 광범위한 커뮤니티의 접근성을 확보했다. 동시에, 3D 재구성 기술의 내재적 한계를 NuRec Fixer라는 AI 기반 후처리 기술로 보완함으로써, 단일 알고리즘의 완벽성에 의존하기보다는 각 단계에 최적화된 도구들을 조합하는 유연하고 실용적인 엔지니어링 전략을 보여준다.


NVIDIA Omniverse NuRec은 현실 세계를 빠르고 사실적으로 디지털화하는 능력을 바탕으로 다양한 산업 분야에서 혁신적인 애플리케이션을 가능하게 한다. 특히 자율주행, 로보틱스, 산업용 디지털 트윈, 합성 데이터 생성 분야에서 그 가치가 두드러진다.


자율주행 시스템의 안전성과 신뢰성을 확보하기 위해서는 수십억 마일에 달하는 주행 테스트가 필요하며, 이를 현실 세계에서 모두 수행하는 것은 불가능하다. NuRec은 이러한 과제를 해결하는 데 핵심적인 역할을 한다.

- **엣지 케이스 테스트:** 실제 주행 중 수집된 데이터를 NuRec으로 재구성함으로써, 물리적으로 재현하기 어렵거나 위험한 엣지 케이스(edge cases)를 가상 환경에서 안전하고 반복적으로 테스트할 수 있다. 예를 들어, 갑자기 보행자가 튀어나오는 교차로나 복잡한 교통 상황에서의 취약 도로 사용자(VRU, Vulnerable Road Users) 시나리오 등을 시뮬레이션하여 AV 시스템의 대응 능력을 검증할 수 있다.18
- **산업 생태계 통합:** 자율주행 검증 및 유효성 검사(V&V) 툴체인의 선두주자인 Foretellix는 자사의 플랫폼에 NuRec, Omniverse Sensor RTX(물리적으로 정확한 센서 시뮬레이션), 그리고 Cosmos Transfer(생성형 AI 기반 시나리오 변형)를 통합하고 있다. 이를 통해 대규모 합성 데이터 생성 파이프라인의 물리적 정확도와 시나리오 다양성을 획기적으로 향상시키고 있다.3
- **도메인 무작위화(Domain Randomization):** NuRec으로 재구성된 기본 장면에 NVIDIA Cosmos Transfer와 같은 생성형 AI 모델을 적용하여, 조명 조건(낮, 밤, 황혼), 날씨(맑음, 비, 눈, 안개), 주변 객체의 종류와 배치 등을 다양하게 변화시킬 수 있다.5 이렇게 생성된 무한에 가까운 시나리오 변형을 통해 AV 시스템이 다양한 환경 변화에 강건하게 대처할 수 있도록 훈련하고 검증한다.


로봇이 복잡하고 예측 불가능한 현실 환경에서 효과적으로 작동하기 위해서는 시뮬레이션에서의 충분한 학습과 테스트가 필수적이다. NuRec은 이 과정의 효율성과 정확성을 극대화한다.

- **현실 기반 학습 환경 구축:** 공장, 창고, 병원, 가정 등 로봇이 실제로 작동할 환경을 NuRec을 이용해 빠르고 정밀하게 스캔하여 디지털 트윈을 생성한다.27 이는 로봇에게 현실과 거의 동일한 가상 훈련장을 제공하는 것과 같다.
- **Sim-to-Real 전이성 향상:** 로봇은 이 가상 복제본 내에서 탐색(navigation), 물체 조작(manipulation), 인간과의 상호작용 등 다양한 작업을 안전하고 비용 효율적으로 수없이 반복 연습할 수 있다. 시뮬레이션 환경이 현실과 매우 유사하기 때문에, 가상 환경에서 습득한 기술이 현실 세계에서도 높은 성공률로 발현될 가능성, 즉 'sim-to-real transfer'가 크게 향상된다.5
- **플랫폼 통합 및 산업 채택:** NVIDIA의 로보틱스 시뮬레이션 애플리케이션인 Isaac Sim 최신 버전(5.0)은 NuRec 뉴럴 렌더링 기능을 기본적으로 포함하여, 개발자들이 시뮬레이션과 현실 간의 간극을 더욱 쉽게 줄일 수 있도록 지원한다.3 Boston Dynamics, Figure AI, Hexagon 등 세계 유수의 로봇 기업들이 로봇 AI 개발을 가속화하기 위해 Isaac Sim과 Omniverse 라이브러리를 적극적으로 채택하고 있다.3


NuRec은 개별 제품이나 환경을 넘어, 전체 생산 라인이나 도시와 같은 복잡한 시스템의 디지털 트윈을 구축하고 운영하는 데 핵심적인 기술이다.

- **사례 연구: Amazon의 'Zero-Touch Manufacturing'**: Amazon Devices의 한 제조 시설에서는 NuRec과 같은 기술을 활용한 '시뮬레이션 우선(simulation-first)' 접근 방식을 구현했다. 이 시스템에서는 먼저 제품의 CAD 모델로부터 수만 개의 다양한 합성 이미지를 생성하여 로봇 팔이 제품의 미세한 결함을 감지하도록 훈련시킨다. 그런 다음, 실제 생산 라인의 디지털 트윈 내에서 로봇의 동작을 시뮬레이션하고 최적화한다. 이 접근 방식의 가장 큰 장점은 새로운 제품을 생산 라인에 도입할 때 값비싼 물리적 하드웨어 변경 없이, 오직 소프트웨어와 AI 모델 업데이트만으로 전환이 가능하다는 점이다. 이는 생산 라인의 유연성을 극대화하고 제품 출시 기간을 단축하는 'Zero-Touch Manufacturing' 비전을 실현한다.19
- **사례 연구: Texas Children's Hospital의 차세대 병실 설계**: Texas Children's Hospital은 새로운 분만실을 설계하고 건설하는 과정에서 Omniverse 기반의 디지털 트윈을 구축했다. 이 프로젝트에서는 Dell의 고성능 워크스테이션과 NVIDIA RTX GPU를 사용하여 실제와 동일한 가상 병실을 만들었다. 의사, 간호사, 건축가, 건설 인력 등 다양한 이해관계자들이 이 가상 공간에 접속하여 원격으로 협업하며, 의료 장비의 배치를 최적화하고 의료진의 동선을 시뮬레이션했다. 이를 통해 물리적인 건설이 시작되기 전에 잠재적인 문제점을 발견하고 설계를 개선함으로써, 최종적으로 건설 비용을 절감하고 환자와 의료진 모두의 만족도를 높이는 결과를 얻었다. 이 사례는 NuRec 기술이 AEC(건축, 엔지니어링, 건설) 및 헬스케어 분야로 확장될 수 있는 무한한 잠재력을 보여준다.51


고성능 AI 모델을 훈련시키기 위해서는 방대하고 다양한, 그리고 정확하게 레이블링된 데이터가 필수적이다. 그러나 현실 세계에서 이러한 데이터를 수집하는 것은 프라이버시 문제, 희귀 사건의 부재, 막대한 레이블링 비용 등 여러 제약에 부딪힌다.52

- **디지털 무대(Digital Stage) 제작:** 합성 데이터 생성(SDG) 파이프라인에서 NuRec의 역할은 현실 세계를 사실적으로 복제한 3D 환경, 즉 '디지털 무대'를 만드는 것이다.8 이 무대는 합성 데이터가 생성될 배경이 된다.
- **절차적 콘텐츠 생성:** 이 디지털 무대 위에 NVIDIA NIM(NVIDIA Inference Microservices)이나 Cosmos WFM(World Foundation Models)과 같은 생성형 AI 도구를 사용하여 다양한 가상 객체(자동차, 사람, 가구 등)를 절차적으로 생성하고 배치한다. 또한 조명, 날씨, 시간대와 같은 환경 조건을 무작위로 변경하여 데이터의 다양성을 극대화한다.52
- **자동 레이블링:** 이 모든 과정은 컴퓨터 시뮬레이션 내에서 이루어지기 때문에, 생성된 데이터에 대한 정확한 레이블(e.g., 시맨틱 분할 맵, 깊이 정보, 경계 상자)을 자동으로, 그리고 완벽하게 생성할 수 있다. 이렇게 생성된 방대한 양의 고품질 합성 데이터는 AI 모델의 일반화 성능과 정확도를 크게 향상시키는 데 기여한다.

이러한 응용 사례들은 NuRec이 단순한 3D 재구성 기술을 넘어, '데이터 플라이휠(Data Flywheel)'을 구동하는 핵심 엔진임을 보여준다. 이 플라이휠은 (1) 현실 데이터 수집 → (2) NuRec을 통한 시뮬레이션 환경 구축 → (3) 생성형 AI를 통한 합성 데이터 대량 생성 → (4) AI 모델 훈련 및 성능 향상 → (5) 향상된 AI를 통한 더 정교한 현실 데이터 수집 → (1)로 이어지는 선순환 구조다.18 이 구조에서 NuRec은 현실의 원시 데이터를 시뮬레이션 가능한 고품질 디지털 자산으로 변환하여 플라이휠을 처음으로 돌리는, 대체 불가능한 역할을 수행한다.


NVIDIA Omniverse NuRec은 3D 재구성 분야에서 혁신적인 성과를 이루었지만, 기술이 성숙해감에 따라 몇 가지 내재적 한계와 해결해야 할 도전 과제들이 존재한다. 이러한 한계점들은 역설적으로 NVIDIA와 관련 연구 커뮤니티의 미래 기술 개발 로드맵을 명확하게 보여주는 지표가 된다.


- **동적 객체 및 장면 표현:** 현재 NuRec의 기반 기술인 3DGS는 기본적으로 정적인(static) 장면에 최적화되어 있다. 움직이는 사람, 차량, 또는 시간에 따라 변화하는 환경과 같은 동적 요소를 포함하는 4D 장면을 실시간으로 고품질 재구성하는 것은 여전히 기술적으로 매우 어려운 과제이며, 활발한 연구가 진행 중인 분야다.29
- **재질 및 물리적 속성 표현:** NuRec은 주로 장면의 기하학적 형태와 시각적 외형(appearance)을 매우 사실적으로 재구성한다. 그러나 시뮬레이션의 물리적 정확성을 한 단계 더 높이기 위해서는 눈에 보이지 않는 속성들, 즉 재료의 반사율(reflectance), 거칠기(roughness), 투과율(transmittance)과 같은 물리 기반 재질(PBR) 속성과 객체의 질량(mass), 마찰 계수(friction coefficient)와 같은 물리적 속성을 정확하게 추정해야 한다. 이러한 속성들을 단지 2D 이미지로부터 추론하는 것은 매우 어려운 역 렌더링 문제이며, 현재 NuRec의 주된 초점은 아니다.1
- **구조적 안정성 및 물리적 타당성:** 2D 이미지로부터 재구성된 3D 모델은 시각적으로는 완벽해 보일 수 있지만, 물리 시뮬레이션 엔진에 적용했을 때 구조적으로 불안정할 수 있다. 예를 들어, AI가 시각적 추정만으로 생성한 의자 모델은 다리의 비율이 미세하게 비대칭이거나 연결 부위가 약해서 물리 시뮬레이션에서 하중을 가하면 쉽게 무너질 수 있다.1 재구성된 지오메트리가 물리 법칙을 준수하도록 보장하는 것은 중요한 과제다.


NuRec은 Omniverse 플랫폼에 깊숙이 통합되어 있지만, 현재 구현에는 몇 가지 구체적인 제약사항이 존재한다.

- **카메라 모델 지원:** NuRec 렌더링은 표준적인 핀홀(pinhole) 카메라 모델과 OpenCV 기반의 어안 렌즈 왜곡 모델 등 특정 카메라 프로젝션 모델만을 지원한다. 이 외의 다른 복잡한 렌즈 모델을 사용하려고 시도하면 오류가 발생할 수 있다.23
- **프록시 메시(Proxy Mesh) 의존성:** 현재 NuRec으로 재구성된 볼륨 표현 자체는 물리적 충돌 계산이나 정확한 그림자 및 반사를 직접 생성하지 못한다. 대신, 이러한 효과를 시뮬레이션하기 위해 보이지 않는 단순화된 지오메트리, 즉 **프록시 메시**를 대리인으로 사용한다. 따라서 반사의 품질은 프록시 메시의 정밀도에 의해 결정되며, NuRec 볼륨과 프록시 메시의 형태가 정확히 일치하지 않으면 시각적 아티팩트가 발생할 수 있다.23
- **합성 데이터 AOV(Arbitrary Output Variables) 미지원:** 시맨틱 분할(semantic segmentation) ID나 인스턴스 ID와 같은 합성 데이터 생성에 필수적인 정보들은 NuRec 프리미티브 자체에서 렌더링되지 않는다. 이러한 데이터는 반드시 프록시 메시에 할당되어야 하므로, 데이터 생성 워크플로우가 이원화되는 복잡성이 있다.23
- **기타 제약:** 이 외에도 다중 GPU 환경에서 실시간 모드로 렌더링할 때 뷰포트가 검게 변하는 문제나, 고해상도 이미지 출력을 위한 타일 기반 렌더링(tiled rendering)이 지원되지 않는 등의 기술적 제약이 보고되었다.23


- **대규모 환경의 실시간 재구성:** 현재 NuRec의 워크플로우는 수집된 데이터를 오프라인에서 처리하는 것을 기반으로 한다. 도시 전체와 같은 광범위한 대규모 환경을 실시간으로 캡처하고, 재구성하며, 이를 원격 사용자에게 스트리밍하는 것은 현재의 계산 능력과 네트워크 대역폭으로는 상당한 도전 과제다.
- **계산 비용 및 접근성:** 수백만 개의 3D 가우시안을 학습하고 렌더링하기 위해서는 여전히 고사양의 NVIDIA GPU와 상당한 양의 VRAM 및 저장 공간이 필요하다.30 모델 압축, 양자화, 렌더링 알고리즘 최적화 등을 통해 이를 경량화하고 더 넓은 범위의 하드웨어에서 실행 가능하게 만드는 것은 기술의 대중화를 위한 중요한 과제다.


NuRec의 현재 한계점들은 NVIDIA의 미래 기술 개발 방향을 명확히 제시한다.

- **생성형 AI와의 심층적 융합:** 미래의 NuRec은 단순히 현실을 '재구성'하는 것을 넘어, 생성형 AI와 결합하여 가상 세계를 '창조'하는 방향으로 발전할 것이다. NuRec으로 구축된 정적 장면에 NVIDIA Cosmos WFM(World Foundation Models)과 같은 기술을 적용하여, "이 거리를 비 오는 저녁 시간으로 바꾸고, 보행자 10명을 추가해줘"와 같은 자연어 프롬프트를 통해 동적이고 다채로운 시나리오를 자동으로 생성하게 될 것이다.3
- **물리 기반 렌더링(PBR)과의 심화된 통합:** 시각적 외형뿐만 아니라 물리적으로 정확한 재질 속성을 이미지로부터 추론하는 역 렌더링 연구(e.g., PBR-NeRF)가 NuRec에 통합될 것이다.26 이는 시뮬레이션의 사실성을 한 차원 더 높여, 빛과 재질의 상호작용을 더욱 정밀하게 예측하고, 센서 시뮬레이션의 정확도를 극대화할 것이다.
- **실시간 캡처 및 스트리밍:** 이벤트 카메라(event camera)와 같이 시간 해상도가 매우 높은 새로운 센서 기술과 5G/6G 네트워크, 엣지 컴퓨팅의 발전은 현실 세계를 실시간으로 캡처하여 원격지의 Omniverse 디지털 트윈으로 스트리밍하는 것을 가능하게 할 것이다.55 이는 원격 현장 모니터링, AR/VR 기반의 원격 협업, 실시간 재난 대응 시뮬레이션 등 새로운 응용 분야를 열 것이다.
- **다중 모달리티(Multi-modality) 통합:** 미래의 재구성 기술은 시각 데이터(카메라)뿐만 아니라 LiDAR의 정밀한 거리 정보, 레이더의 악천후 투과 능력, 마이크의 공간 음향 정보 등 다양한 센서 데이터를 융합(sensor fusion)하여, 단일 센서로는 얻을 수 없는 더 풍부하고 강건한 3D 세계 모델을 구축하는 방향으로 발전할 것이다.57

이처럼 NuRec의 현재 한계점들은 미해결 과제가 아니라, NVIDIA가 Cosmos WFM, PBR 연구, 차세대 Blackwell GPU, DGX Cloud와 같은 제품 및 연구 파이프라인을 통해 이미 해결책을 개발하고 있는 '차기 기술 로드맵'을 가리키는 이정표 역할을 한다. 이는 NVIDIA가 단기적인 제품 출시와 장기적인 연구 개발을 긴밀하게 연계하여 기술 생태계를 체계적으로 확장하고 있음을 보여준다.



NVIDIA Omniverse NuRec은 3D 가우시안 스플래팅이라는 최첨단 뉴럴 렌더링 기술을 산업 현장에 성공적으로 적용한 사례다. 이는 현실 세계를 고충실도의 디지털 트윈으로 전환하는 데 필요한 시간과 비용을 기존의 수 주에서 수 시간 단위로 획기적으로 단축시키는 기술적 변곡점을 만들었다. NuRec은 단순한 시각적 복제를 넘어, 로보틱스와 자율주행 시스템이 현실과 거의 구별 불가능한 가상 환경에서 학습하고 검증받을 수 있는 길을 열었다. 이를 통해 AI 개발의 가장 큰 난제 중 하나였던 'Sim-to-Real' 문제, 즉 시뮬레이션에서 얻은 학습 결과를 현실 세계로 효과적으로 이전하는 문제를 해결하는 데 결정적인 기여를 하고 있다.


NuRec은 피지컬 AI 개발의 핵심 병목이었던 '사실적인 대규모 가상 세계의 부족' 문제를 정면으로 해결함으로써, AI가 물리 세계와 상호작용하는 능력을 기하급수적으로 발전시킬 거대한 잠재력을 가지고 있다. 로봇과 자율주행차는 이제 NuRec이 생성한 무한에 가까운 가상 세계에서 안전하게 실패하고, 배우고, 진화할 수 있게 되었다.

더 나아가, NuRec은 공장, 도시, 인프라, 심지어 인체에 이르기까지 현실 세계의 모든 것을 디지털 트윈으로 만들고 시뮬레이션하려는 '산업 메타버스(Industrial Metaverse)' 비전을 현실로 만드는 근간 기술이다. 이는 제품의 설계, 제조, 운영, 유지보수에 이르는 산업 전 과정의 패러다임을 바꿀 것이다. 가상 환경에서 모든 것을 미리 시뮬레이션하고 최적화함으로써 기업은 비용을 절감하고, 혁신을 가속화하며, 지속 가능성을 높일 수 있다.


NVIDIA Omniverse NuRec은 현실과 가상의 경계를 허물고 두 세계를 유기적으로 연결하는 중요한 기술적 이정표다. 그러나 현재의 성공은 거대한 변화의 시작에 불과하다. 향후 연구는 현재의 정적이고 시각적인 재구성을 넘어, 동적인 상호작용, 물리적으로 타당한 재질 속성, 그리고 장면의 의미론적 이해(semantic understanding)까지 포함하는 총체적인 세계 모델(holistic world model)을 구축하는 방향으로 나아가야 한다.

궁극적으로 NuRec과 Omniverse 플랫폼은 생성형 AI와의 완전한 융합을 통해, 단순히 현실을 '재구성(reconstruction)'하는 단계를 넘어, 사용자의 의도에 따라 가상 세계를 능동적으로 '창조(creation)'하고 변형하는 영역으로 확장될 것이다. 현실 데이터로부터 학습하되, 그 한계를 뛰어넘어 무한한 가능성을 시뮬레이션하는 것, 이것이 NuRec과 Omniverse가 열어갈 피지컬 AI 시대의 진정한 미래가 될 것이다.


1. NVIDIA Research Shapes Physical AI, 8월 19, 2025에 액세스, https://blogs.nvidia.com/blog/physical-ai-research-siggraph-2025/
2. Nvidia pushes "Physical AI" with new Blackwell hardware and AI models - The Decoder, 8월 19, 2025에 액세스, https://the-decoder.com/nvidia-pushes-physical-ai-with-new-blackwell-hardware-and-ai-models/
3. NVIDIA Opens Portals to World of Robotics With New Omniverse Libraries, Cosmos Physical AI Models and AI Computing Infrastructure, 8월 19, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-opens-portals-to-world-of-robotics-with-new-omniverse-libraries-cosmos-physical-ai-models-and-ai-computing-infrastructure
4. Nvidia Unveils Cosmos Reason AI to Give Robots Human-Like Thinking and Planning Skills, 8월 19, 2025에 액세스, https://www.thehansindia.com/tech/nvidia-unveils-cosmos-reason-ai-to-give-robots-human-like-thinking-and-planning-skills-996242
5. How to Instantly Render Real-World Scenes in Interactive ..., 8월 19, 2025에 액세스, https://developer.nvidia.com/blog/how-to-instantly-render-real-world-scenes-in-interactive-simulation/
6. NVIDIA Opens Portals to World of Robotics With New Omniverse Libraries, Cosmos Physical AI Models and AI Computing Infrastructure - Stock Titan, 8월 19, 2025에 액세스, https://www.stocktitan.net/news/NVDA/nvidia-opens-portals-to-world-of-robotics-with-new-omniverse-m3l0104hz8jn.html
7. developer.nvidia.com, 8월 19, 2025에 액세스, [https://developer.nvidia.com/omniverse#:~:text=NVIDIA%20Omniverse%E2%84%A2%20is%20a,OpenUSD)%20and%20NVIDIA%20RTX%E2%84%A2.](https://developer.nvidia.com/omniverse#:~:text=NVIDIA Omniverse™ is a,OpenUSD) and NVIDIA RTX™.)
8. Develop on NVIDIA Omniverse Platform, 8월 19, 2025에 액세스, https://developer.nvidia.com/omniverse
9. NVIDIA Omniverse, 8월 19, 2025에 액세스, https://docs.nvidia.com/omniverse/index.html
10. NVIDIA Omniverse Enterprise | Dell USA, 8월 19, 2025에 액세스, https://www.dell.com/en-us/lp/nvidia-omniverse
11. NVIDIA Omniverse Foundational Technology Montage I GTC Spring 2025 Edition - YouTube, 8월 19, 2025에 액세스, https://www.youtube.com/watch?v=u5TNCt45Q90
12. Nvidia Omniverse - Wikipedia, 8월 19, 2025에 액세스, https://en.wikipedia.org/wiki/Nvidia_Omniverse
13. NVIDIA Omniverse, could someone explain its use and what it does in Layman's term : r/VisionPro - Reddit, 8월 19, 2025에 액세스, https://www.reddit.com/r/VisionPro/comments/1biiu52/nvidia_omniverse_could_someone_explain_its_use/
14. Neural Concept Demonstrates Engineering AI Platform Integrated with NVIDIA Omniverse Blueprint at NVIDIA GTC, Accelerating Product Development, 8월 19, 2025에 액세스, https://www.neuralconcept.com/post/neural-concept-demonstrates-engineering-ai-platform-integrated-with-nvidia-omniverse-blueprint-at-nvidia-gtc-accelerating-product-development
15. Neural Reconstruction for Robotics and Autonomous Vehicles - YouTube, 8월 19, 2025에 액세스, https://www.youtube.com/watch?v=e3dfXj6ATA0
16. NVIDIA announces new Omniverse libraries and Cosmos WFMs - Engineering.com, 8월 19, 2025에 액세스, https://www.engineering.com/nvidia-announces-new-omniverse-libraries-and-cosmos-wfms/
17. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 19, 2025에 액세스, https://developer.nvidia.com/isaac/sim
18. Accelerating AV Simulation with Neural Reconstruction and World Foundation Models, 8월 19, 2025에 액세스, https://developer.nvidia.com/blog/accelerating-av-simulation-with-neural-reconstruction-and-world-foundation-models/
19. NVIDIA's Omniverse NuRec Revolutionizes Real-World Scene ..., 8월 19, 2025에 액세스, https://blockchain.news/news/nvidia-omniverse-nurec-revolutionizes-real-world-scene-simulation
20. Neural Volume Rendering - Isaac Sim Documentation, 8월 19, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/latest/assets/usd_assets_nurec.html
21. NVIDIA Announces NuRec Gaussian Splatting Libraries at SIGGRAPH - Radiance Fields, 8월 19, 2025에 액세스, https://radiancefields.com/nvidia-announces-nurec-libraries-at-siggraph
22. nvidia/PhysicalAI-Autonomous-Vehicles-NuRec · Datasets at ..., 8월 19, 2025에 액세스, https://huggingface.co/datasets/nvidia/PhysicalAI-Autonomous-Vehicles-NuRec
23. Neural (NuRec) Rendering — Omniverse Materials and Rendering, 8월 19, 2025에 액세스, https://docs.omniverse.nvidia.com/materials-and-rendering/latest/neural-rendering.html
24. Omniverse Materials and Rendering, 8월 19, 2025에 액세스, https://docs.omniverse.nvidia.com/materials-and-rendering/latest/materials.html
25. What Are Neural Radiance Fields - KIRI Engine, 8월 19, 2025에 액세스, https://www.kiriengine.app/blog/explained/what-are-neural-radiance-fields
26. PBR-NeRF: Inverse Rendering with Physics-Based Neural Fields - CVF Open Access, 8월 19, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2025/papers/Wu_PBR-NeRF_Inverse_Rendering_with_Physics-Based_Neural_Fields_CVPR_2025_paper.pdf
27. What is 3D Reconstruction? | NVIDIA Glossary, 8월 19, 2025에 액세스, https://www.nvidia.com/en-us/glossary/3d-reconstruction/
28. Radiance Fields (Gaussian Splatting and NeRFs), 8월 19, 2025에 액세스, https://radiancefields.com/
29. Gaussian splatting - Wikipedia, 8월 19, 2025에 액세스, https://en.wikipedia.org/wiki/Gaussian_splatting
30. Introduction to 3D Gaussian Splatting - Hugging Face, 8월 19, 2025에 액세스, https://huggingface.co/blog/gaussian-splatting
31. Understanding and Exploring 3D Gaussian Splatting: A Comprehensive Overview - Medium, 8월 19, 2025에 액세스, https://medium.com/@logessiva/understanding-and-exploring-3d-gaussian-splatting-a-comprehensive-overview-b4004f28ef1c
32. 3D Gaussian Splatting Introduction – Paper Explanation & Training on Custom Datasets with NeRF Studio Gsplats - LearnOpenCV, 8월 19, 2025에 액세스, https://learnopencv.com/3d-gaussian-splatting/
33. Gaussian Splatting: A Deep Dive into Transforming 3D Data for Real-Time Visualization, 8월 19, 2025에 액세스, https://karthick.ai/blog/2024/Gaussian-Splatting/
34. Introduction to Loss Functions | DataRobot Blog, 8월 19, 2025에 액세스, https://www.datarobot.com/blog/introduction-to-loss-functions/
35. PyTorch Loss Functions: The Ultimate Guide - Neptune.ai, 8월 19, 2025에 액세스, https://neptune.ai/blog/pytorch-loss-functions
36. Loss Functions in Machine Learning Explained - DataCamp, 8월 19, 2025에 액세스, https://www.datacamp.com/tutorial/loss-function-in-machine-learning
37. nv-tlabs/3dgrut: Ray tracing and hybrid rasterization of ... - GitHub, 8월 19, 2025에 액세스, https://github.com/nv-tlabs/3dgrut
38. The Biggest Breakthrough in Gaussian Splatting - A 3DGRUT First Impressions and Tutorial, 8월 19, 2025에 액세스, https://www.youtube.com/watch?v=FI5JluMoBDE
39. Why Neural Radiance Fields (NeRF) is the Future of 3D Rendering and Visualization | by Thiriloganathan Manimohan | Medium, 8월 19, 2025에 액세스, https://medium.com/@manimohan517/neural-radiance-fields-nerf-7fe13120086a
40. [2403.11134] Recent Advances in 3D Gaussian Splatting - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/abs/2403.11134
41. Did a comparison test between photogrammetry and Artec3d Space Spider : r/3DScanning, 8월 19, 2025에 액세스, https://www.reddit.com/r/3DScanning/comments/1m3lhq3/did_a_comparison_test_between_photogrammetry_and/
42. Photogrammetry vs 3D scanning for creating a 3D model - Artec 3D, 8월 19, 2025에 액세스, https://www.artec3d.com/learning-center/photogrammetry-vs-3d-scanning
43. What is a NeRF? - Mapillary, 8월 19, 2025에 액세스, https://help.mapillary.com/hc/en-us/articles/17262288782108-What-is-a-NeRF
44. LiDAR vs Photogrammetry: Key Differences & Use Cases - Matterport, 8월 19, 2025에 액세스, https://matterport.com/blog/lidar-vs-photogrammetry
45. [2503.01774] Difix3D+: Improving 3D Reconstructions with Single-Step Diffusion Models - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/abs/2503.01774
46. Difix3D+: Improving 3D Reconstructions with Single-Step Diffusion Models, 8월 19, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/difix3d
47. nv-tlabs/Difix3D: [CVPR 2025 Oral & Award Candidate] Difix3D+: Improving 3D Reconstructions with Single-Step Diffusion Models - GitHub, 8월 19, 2025에 액세스, https://github.com/nv-tlabs/Difix3D
48. Difix3D+: Improving 3D Reconstructions with Single-Step Diffusion Models - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/html/2503.01774v1
49. [Literature Review] Difix3D+: Improving 3D Reconstructions with Single-Step Diffusion Models - Moonlight, 8월 19, 2025에 액세스, https://www.themoonlight.io/en/review/difix3d-improving-3d-reconstructions-with-single-step-diffusion-models
50. Revolutionizing Interactive Simulation with NVIDIA Omniverse NuRec and 3DGUT - AInvest, 8월 19, 2025에 액세스, https://www.ainvest.com/news/revolutionizing-interactive-simulation-nvidia-omniverse-nurec-3dgut-2508/
51. Delivering digital twins with Precision: a modern approach for a new kind of healthcare - Dell, 8월 19, 2025에 액세스, https://www.delltechnologies.com/asset/sv-se/products/workstations/customer-stories-case-studies/mark-iii-systems-case-study.pdf
52. Synthetic Data for AI & 3D Simulation Workflows | Use Case - NVIDIA, 8월 19, 2025에 액세스, https://www.nvidia.com/en-us/use-cases/synthetic-data/
53. PBR-NeRF: Inverse Rendering with Physics-Based Neural Fields - Bohrium, 8월 19, 2025에 액세스, https://www.bohrium.com/paper-details/pbr-nerf-inverse-rendering-with-physics-based-neural-fields/1075661722464813075-108597
54. PBR-NeRF, 8월 19, 2025에 액세스, https://s3anwu.github.io/pbrnerf/
55. [2505.08438] A Survey of 3D Reconstruction with Event Cameras - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/abs/2505.08438
56. A Survey of 3D Reconstruction with Event Cameras: From Event-based Geometry to Neural 3D Rendering - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/html/2505.08438v1
57. A Comprehensive Review of Vision-Based 3D Reconstruction Methods - PMC, 8월 19, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014007/
58. The Future of Robotics: 3D Reconstruction Techniques - Number Analytics, 8월 19, 2025에 액세스, https://www.numberanalytics.com/blog/future-robotics-3d-reconstruction-techniques


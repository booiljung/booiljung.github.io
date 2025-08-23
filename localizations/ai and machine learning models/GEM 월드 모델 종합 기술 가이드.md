# GEM 월드 모델 종합 기술 가이드


자율 시스템, 특히 비전 기반 항법 분야에서 'Gem'이라는 용어는 다양한 맥락에서 사용되어 혼동을 야기할 수 있다. 본 보고서의 핵심 주제는 CVPR 2025에서 발표된 **GEM(Generalizable Ego-vision Multimodal) 월드 모델**로, 이는 기존 항법 기술과는 근본적으로 다른 접근 방식을 제시한다.1 이 모델은 NVIDIA의 Isaac ROS GEMs(로봇 운영 체제용 소프트웨어 모듈) 2, GEM-SLAM(SLAM의 한 변형) 4, 또는 기타 상업용 제품 5과는 명확히 구분되는 독자적인 연구 성과이다. 이러한 구분을 명확히 하는 것은 GEM의 혁신성을 정확히 이해하기 위한 첫걸음이다.

본 보고서는 GEM이 단순한 기술 개선이 아닌, 자율 시스템의 환경 인식 및 계획 수립 방식에 있어 근본적인 패러다임 전환을 의미한다는 점을 논증하고자 한다. 기존의 주류 기술인 SLAM(동시적 위치 추정 및 지도 작성)이나 VO(시각 주행 거리 측정)는 본질적으로 **서술적(descriptive)**이고 **회고적(retrospective)**이다.7 이 기술들의 핵심 목표는 환경의 기하학적 지도를 구축하고, 그 안에서 에이전트의 과거 또는 현재 위치를 파악하는 것, 즉 "나는 어디에 있는가?"라는 질문에 답하는 것이다.9

반면, GEM과 같은 월드 모델은 **생성적(generative)**이고 **예측적(predictive)**이다.11 이 모델의 주된 기능은 세계의 물리 법칙과 동역학에 대한 내재적 모델을 학습하여, 주어진 행동이나 조건에 따라 발생 가능한 미래 상태를 시뮬레이션하는 것이다.12 이는 핵심 질문을 "만약 ~하면 어떻게 될 것인가?"로 전환시키며, 에이전트가 단순히 현재 위치를 파악하는 것을 넘어 미래를 예측하고 계획을 수립하는 새로운 차원의 능력을 부여한다.

따라서 본 보고서는 GEM 월드 모델에 대한 결정적이고 다각적인 전문가 수준의 분석을 제공하는 것을 목표로 한다. 기초 개념과 아키텍처 심층 분석부터 시작하여 성능 평가, 핵심 도전 과제, 그리고 이를 둘러싼 광범위한 기술 및 윤리적 생태계까지 체계적으로 탐구할 것이다.


이 장에서는 월드 모델로의 전환이 왜 일어나고 있으며, 이들이 해결하고자 하는 근본적인 문제가 무엇인지에 대한 이론적 토대를 마련한다.


전통적인 항법 기술과 월드 모델은 환경을 이해하고 상호작용하는 방식에서 근본적인 차이를 보인다. SLAM과 VO는 환경의 기하학적 구조를 복원하는 데 중점을 두는 반면, 월드 모델은 환경의 동적 규칙을 학습하여 미래를 시뮬레이션하는 데 초점을 맞춘다.

SLAM/VO는 기하학적 추정 문제를 해결하는 것을 목표로 한다. 카메라 이미지나 LiDAR 스캔과 같은 연속적인 센서 데이터를 입력받아 에이전트의 궤적을 계산하고 환경 지도를 구축한다.4 여기서 지도는 점, 선, 면 등으로 구성된 기하학적 표현이며, 위치 추정은 에이전트를 이 지도와 연관시키는 과정이다.7

반면, 월드 모델은 세계의 압축된 잠재 표현(latent representation)과 그 전이 동역학(transition dynamics)을 학습한다.12 이 모델의 주된 출력물은 정적인 지도가 아니라, 일련의 행동이 주어졌을 때 미래의 센서 데이터(예: 비디오 프레임)를 생성하도록 쿼리할 수 있는 **학습된 시뮬레이터(learned simulator)**이다.1 이 능력은 종종 '상상력(imagination)'으로 묘사되며, 에이전트가 미래의 결과를 '꿈꾸면서' 계획을 세울 수 있게 한다.14

이러한 차이는 추상화 수준의 전환을 의미한다. SLAM/VO가 제공하는 것은 점, 선, 면으로 이루어진 **기하학적 추상화**이며, 에이전트는 이 기하학적 공간 내에서 경로를 계획한다. 반면 월드 모델은 현재 상태와 행동을 미래 상태로 매핑하는 함수, 즉 **동적, 예측적 추상화**를 제공한다. 에이전트는 이 학습된 시뮬레이터 내에서 '가상 시나리오'를 실행하며 계획을 수립한다. 이는 기하학을 넘어 인과관계에 가까운 더 높은 수준의 추상화이다. 이러한 전환은 인간이 추론과 계획을 위해 정신 모델을 사용하는 방식과 유사하며 12, 보다 일반적인 지능을 구축하는 데 근본적인 중요성을 갖는다. 결과적으로, 자율 시스템 개발의 공학적 초점은 모든 가능한 기하학적 시나리오에 대한 복잡한 의사결정 규칙을 수작업으로 만드는 것에서, 방대한 데이터셋을 관리하고 월드 모델 자체를 설계, 훈련, 검증하는 도전 과제로 이동하게 될 것이다.


월드 모델은 SLAM과 같은 전통적인 방식이 직면한 몇 가지 근본적인 한계를 해결할 잠재력을 가지고 있다.

- **동적 환경의 문제**: 전통적인 SLAM 시스템은 대부분 정적인 세계를 가정한다. 따라서 교통량이 많거나 보행자가 붐비는 것과 같이 역동성이 높은 환경에서는 성능이 크게 저하된다. 움직이는 객체들은 정적 세계 가정을 위반하여 추적 실패와 지도 손상을 유발하기 때문이다.16 반면, 월드 모델은 본질적으로 이러한 동역학을 학습하고 예측하도록 설계되었다.1
- **확장성 및 장기 일관성**: SLAM 시스템은 긴 궤적에 걸쳐 오차가 누적되는 드리프트(drift) 문제에 시달린다. 루프 폐쇄(loop closure)와 같은 기술로 이를 보정할 수 있지만, 항상 신뢰할 수 있는 것은 아니며 시각적으로 반복적인 대규모 환경에서는 실패할 수 있다.18 또한, 크고 전역적으로 일관된 지도를 메모리에 유지하는 것은 계산 비용이 많이 들고 확장성에 심각한 제약을 가한다.21 압축된 잠재 공간에서 작동하는 월드 모델은 장기적인 계획 수립에 더 확장 가능한 솔루션을 제공할 수 있다.
- **롱테일 문제 (The Long-Tail Problem)**: 모든 자율 시스템의 치명적인 실패 지점은 드물지만 치명적인 사건들, 즉 '롱테일' 문제다.23 예를 들어 특이한 교통사고나 기이한 보행자 행동과 같은 모든 가능성을 포괄하는 실제 데이터를 수집하는 것은 불가능하다. GEM과 같이 제어 가능한 월드 모델은 이러한 희귀 시나리오에 대한 방대한 양의 합성 데이터를 생성함으로써 이 문제를 해결할 경로를 제공하며, 이를 통해 계획 및 제어 모듈의 더 강력한 훈련과 테스트를 가능하게 한다.23


월드 모델은 현대 자율 시스템 아키텍처 내에서 여러 핵심적인 역할을 수행하며, 특히 학습 및 계획 능력의 혁신을 주도한다.

- **모델 기반 강화학습(MBRL)의 구현**: 월드 모델은 MBRL의 핵심 구성 요소이다.12 에이전트는 느리고 비용이 많이 드는 실제 세계의 시행착오 대신, 빠르고 병렬화 가능하며 미분 가능한 월드 모델의 '상상' 환경 내에서 훈련함으로써 훨씬 더 샘플 효율적으로 정책을 학습할 수 있다.29
- **강력한 계획 및 의사결정의 기반**: 월드 모델은 잠재적인 행동 순서의 결과를 예측함으로써, 계획자가 더 먼 미래를 내다보고 근시안적인 결정을 피하며 더 나은 장기적 결과를 가져오는 행동을 선택할 수 있게 한다.31 이러한 예측 능력은 안전과 효율성의 초석으로 간주된다.14
- **데이터 증강 및 시뮬레이션을 위한 소스**: 앞서 언급했듯이, 월드 모델은 현실적이고 다양하며 제어 가능한 시나리오를 생성하는 강력한 데이터 생성기 역할을 할 수 있다. 이는 인지부터 제어에 이르는 전체 자율 주행 스택을 훈련하고 검증하는 데 사용될 수 있으며, 주요 산업체들이 적극적으로 탐구하고 있는 핵심 응용 분야이다.33


이 장에서는 CVPR 2025 논문과 관련 자료를 바탕으로 GEM 모델 자체의 세부적인 기술적 구조를 분석한다.


GEM은 명시적으로 **확산 기반(diffusion-based)** 월드 모델로 정의된다.35 이는 중요한 아키텍처 선택이다. 확산 모델은 훈련 중에 데이터에 점진적으로 노이즈를 추가한 다음, 순수한 노이즈에서 시작하여 이 과정을 역으로 되돌려 새로운 데이터 샘플을 생성하는 모델을 학습하는 방식으로 작동한다.

GitHub 저장소에서 확인된 바와 같이, 모델 아키텍처는 세 가지 주요 구성 요소로 이루어져 있다 36:

1. **자율 이동 인코더(Ego-Motion Encoder)**: 에이전트 자신의 의도된 이동 시퀀스를 처리한다.
2. **객체 역학 모듈(Object Dynamics Module)**: 장면 내 다른 개체들의 상호작용과 움직임을 모델링한다.
3. **장면 디코더(Scene Decoder)**: 최종적으로 사실적인 비디오 프레임(RGB 및 깊이 모두)을 재구성하고 생성한다.

비디오 생성의 핵심 과제 중 하나는 많은 프레임에 걸쳐 시간적 일관성을 유지하는 것이다. GEM은 **자기회귀 노이즈 스케줄(autoregressive noise schedules)**을 도입하여 이 문제를 해결한다.1 이 기술은 모든 프레임을 한 번에 생성하는 대신 순차적으로 생성하며, 각 새 프레임의 노이즈 스케줄은 이전에 생성된 프레임에 의존한다. 이는 프레임 간의 인과 관계를 강화하고 생성된 비디오가 시간이 지남에 따라 비일관적으로 발산하는 것을 방지한다.1


GEM의 가장 중요한 기여는 생성된 현실에 대한 미세하고 다각적인 제어 능력이다.1 이 제어 기능은 GEM을 단순한 비디오 생성기에서 강력하고 상호작용적인 시뮬레이터로 격상시킨다.

- **1. 자율 이동 경로를 통한 자율 이동 제어**: 모델은 `ego-trajectory`를 제어 입력으로 받아 에이전트의 미래 경로를 정밀하게 지정할 수 있다.11 기술적으로는 이 궤적 정보를 확산 모델의 UNet 백본에 교차-어텐션 레이어(cross-attention layers)를 사용하여 통합하며, LoRA(Low-Rank Adaptation) 모듈을 통해 효율적인 미세 조정을 가능하게 한다.1

- **2. 희소 특징을 통한 객체 역학 및 장면 구성 제어**: 이는 아마도 가장 혁신적인 제어 측면일 것이다. GEM은 **DINOv2 인코더**에 의해 추출된 희소한 시각적 토큰(sparse visual tokens)을 사용하여 장면 내 다른 객체의 동역학을 제어한다.1 이 과정은 다음과 같이 설명될 수 있다:

  - 사용자는 이러한 희소 특징점들의 미래 위치를 지정할 수 있다.
  - 그러면 모델은 미래 장면을 "인페인팅(inpainting)"하여 해당 특징과 관련된 객체가 그럴듯한 동역학으로 지정된 위치로 이동하도록 보장한다.11
  - 결정적으로, 이 메커니즘은 해당 객체의 DINOv2 특징을 제공함으로써 장면에 **완전히 새로운 객체를 삽입**하는 것도 지원한다.11

- **3. 자세 골격을 통한 미세한 인간 동작 제어**: 보행자의 복잡하고 관절이 있는 움직임을 처리하기 위해 GEM은 세 번째 제어 신호인 인간 자세 골격(human pose skeletons)을 통합한다.1

  **PoseNet**이라는 작은 컨볼루션 신경망(CNN)이 이 골격 이미지를 처리하고, 결과 특징은 확산 모델의 U-Net에 주입되어 특정 자세와 움직임을 가진 인간 형상을 생성하도록 유도한다.1


- **쌍을 이루는 RGB 및 깊이 생성**: GEM은 RGB 비디오와 해당 깊이 맵(depth map) 시퀀스를 동시에 생성한다.1 깊이 출력은 항법, 장애물 회피, 기하학적 추론에 필수적인 명시적인 3D 공간 이해를 제공하므로, 이는 RGB 전용 모델에 비해 상당한 이점이다.11
- **교차 도메인 일반화**: 모델은 이름(Generalizable)에서 알 수 있듯이 명시적으로 "일반화 가능"하도록 설계되었다. 방대하고 다양한 데이터셋으로 훈련되었으며, 주요 초점인 자율 주행 외에도 **자기중심적 인간 활동(egocentric human activities)** 및 **드론 항법(drone navigation)**과 같은 여러 자율 시각 도메인에 성공적으로 미세 조정되었다.1 이는 기초 AI 모델의 핵심 목표인 높은 수준의 적응성을 보여준다.

이러한 정밀한 제어 능력은 GEM의 가장 큰 강점이지만, 동시에 잠재적인 약점이 될 수도 있다. 생성 모델은 시각적으로는 그럴듯하지만 물리적으로나 논리적으로는 불가능한 결과물, 즉 '환각(hallucination)'을 만들어내는 것으로 알려져 있다.38 GEM이 제공하는 제어 수단이 많을수록, 사용자가 물리적으로 일관되지 않거나 비상식적인 세계를 생성하도록 모델을 강제하는 행동 조합(예: 자동차가 단단한 벽을 통과하거나 보행자가 공중에 떠 있도록 명령)을 지정하기가 더 쉬워진다. 이는 제어 가능한 월드 모델 설계에 있어 근본적인 긴장 관계를 드러낸다. 이 기술의 미래는 단순히 더 많은 제어 기능을 추가하는 데 있는 것이 아니라, 그 제어를 학습된 물리 법칙의 범위 내로 *제한*하는 메커니즘을 개발하는 데 있다. 이는 잠재 공간에서 제안된 행동을 생성 *전에* 검증하여 해롭거나 오해의 소지가 있는 시뮬레이션 생성을 방지할 수 있는 '안전 필터' 40, 인과 추론 엔진 41, 또는 정형 검증 기술 43과 월드 모델을 통합해야 할 필요성을 시사한다.


이 장에서는 이론에서 실습으로 전환하여, GitHub 저장소와 논문의 방법론 섹션을 기반으로 GEM이 어떻게 구축되고 훈련되며 연구 커뮤니티에 제공되는지 자세히 설명한다.


GEM의 기반은 총 **4000시간 이상**의 비디오 데이터로 구성된 방대한 훈련 데이터셋이다.1 이 코퍼스의 구성은 다음과 같다: BDD100K와 같은 오픈 소스 데이터셋에서 가져온 약 3200시간의 주행 비디오, 약 1000시간의 자기중심적 인간 활동 데이터, 그리고 약 27시간의 드론 영상.1 이 코퍼스의 다양성은 모델의 일반화 능력의 핵심이다.


GEM과 같은 멀티모달 모델을 훈련시키는 데 있어 주요한 실질적 과제는, 특히 이렇게 크고 다양한 데이터셋 전반에 걸쳐 깊이, 궤적, 인간 자세와 같은 양식에 대한 실제 레이블이 부족하다는 점이다.1

GEM 개발팀은 **의사 레이블링(pseudo-labeling) 파이프라인**을 구현하여 이 문제를 극복했다.1 이 과정에 사용된 구체적인 최첨단 모델들은 다음과 같다:

- **깊이 맵**: **Depth Anything V2**를 사용하여 생성.
- **자율 이동 궤적**: **GeoCalib**과 **DroidSLAM**의 조합을 사용하여 생성.
- **인간 자세 주석**: **DWPose**를 사용하여 생성.

의사 레이블링에 대한 의존성은 잠재적인 '인식론적 루프(epistemic loop)' 또는 '현실과의 괴리(reality gap)'를 만들어낸다. GEM의 3D 공간, 움직임, 인간 형태에 대한 이해는 객관적으로 측정된 현실에 직접 기반하는 것이 아니라, DroidSLAM, Depth Anything 등 *다른 AI 모델*의 출력에 기반한다. 이는 해당 상위 모델에 존재하는 체계적인 편향, 오류 또는 실패 모드가 필연적으로 GEM 자체의 월드 모델에 상속되고 내재화됨을 의미한다. 예를 들어, DroidSLAM이 특정 유형의 환경에서 일관되게 드리프트 현상을 보인다면, GEM은 이 드리프트를 세계의 '특성'으로 학습하게 될 것이다. 이는 안전성 검증 및 인증에 심오한 도전 과제를 제기한다. 안전이 중요한 응용 분야에서 GEM을 신뢰하려면, GEM을 단독으로 검증하는 것만으로는 불충분하며, 훈련 데이터를 생성하는 데 사용된 전체 모델 체인을 검증해야 한다. 이는 월드 모델에서 '자기 기반 확보(self-grounding)' 또는 '현실 고정(reality-anchoring)'을 위한 방법론 개발이라는 중요한 미래 연구 분야를 시사하며, 아마도 희소하지만 매우 정확한 실제 데이터(예: 고정밀 RTK-GPS/IMU)를 사용하여 방대한 의사 레이블링 데이터를 보정하고 정규화하는 방식이 될 수 있다.


이 하위 섹션은 GEM의 오픈 소스 저장소를 기반으로, 이를 사용하거나 복제하고자 하는 연구자들을 위한 실용적인 가이드 역할을 한다.36

- **환경 설정**: 재현성을 위해 Docker 사용이 권장된다. 이미지 빌드 및 컨테이너 실행을 위한 명령어(`docker build`, `docker run`)가 제공된다.36
- **데이터 처리 파이프라인**: 사용자 정의 데이터셋을 준비하는 단계별 프로세스가 설명된다: 비디오를 프레임으로 변환(`fast_ffmpeg.py`), 프레임을 HDF5 파일로 패킹(`convert_to_h5.py`), 의사 레이블 생성, 메타데이터 준비(`prepare_data.py`).36
- **훈련 및 미세 조정**: `train_config.yaml` 파일을 참조하여 훈련 작업을 구성하고 실행하는 과정이 설명된다. `data_dict`(양식 지정), `batch_size`, 분산 훈련(`torchrun`)과 같은 주요 구성이 강조된다.36
- **샘플링 및 추론**: `sample.py`를 사용하여 조건부 및 비조건부 샘플을 생성하는 명령어가 제공되며, 다양한 `condition_type` 플래그(`object_manipulation`, `skeleton_manipulation`, `ego_motion`)가 상세히 설명된다.36

| 파라미터 그룹                                                | 파라미터 이름    | 설명                                               | 예시 값/옵션                                                 |      |
| ------------------------------------------------------------ | ---------------- | -------------------------------------------------- | ------------------------------------------------------------ | ---- |
| **데이터 (Data)**                                            | `data_dict`      | 훈련에 사용할 입력 양식 및 제어 신호를 지정합니다. | - depth_img- trajectory- rendered_poses                      |      |
|                                                              | `batch_size`     | GPU 메모리에 따라 조절되는 훈련 배치 크기입니다.   | 4                                                            |      |
| **모델 (Model)**                                             | `target`         | 사용할 모델의 클래스 경로입니다.                   | `gem.models.diffusion.GEMv1`                                 |      |
|                                                              | `params`         | 모델 초기화에 사용되는 파라미터입니다.             | `unet_config`, `scale_factor` 등                             |      |
| **훈련 (Training)**                                          | `learning_rate`  | 모델 훈련을 위한 학습률입니다.                     | `1.0e-4`                                                     |      |
|                                                              | `nproc_per_node` | 분산 훈련 시 노드당 사용할 GPU 수입니다.           | 4                                                            |      |
| **제어 (Control)**                                           | `condition_type` | 샘플링 시 사용할 조건부 제어 유형입니다.           | unconditionalobject_manipulationskeleton_manipulationego_motion |      |
|                                                              |                  |                                                    |                                                              |      |
| 표 1: GEM 구현 및 훈련 구성. 이 표는 GitHub 저장소의 `train_config.yaml` 파일과 문서에서 언급된 주요 구성 옵션을 요약하여 연구자들이 모델을 실용적으로 활용하는 데 필요한 구체적인 정보를 제공합니다.36 |                  |                                                    |                                                              |      |


이 장에서는 표준 생성 품질 지표와 GEM의 고유한 제어 능력을 평가하기 위해 도입된 새로운 지표를 모두 사용하여 GEM의 성능을 비판적으로 평가한다.


GEM의 성능은 표준 생성 모델 평가 지표를 통해 분석된다.

- **프레셰 인셉션 거리 (FID, Fréchet Inception Distance)**: 이미지 품질과 현실성을 측정한다.
- **프레셰 비디오 거리 (FVD, Fréchet Video Distance)**: 비디오 품질과 시간적 일관성을 측정한다.

GEM의 성능은 구글의 Vista 월드 모델과 같은 강력한 기준 모델과 비교하여 평가된다. 연구 결과에 따르면, GEM은 특히 안정적인 장기 생성에서 우수한 FVD 점수를 달성하여 더 나은 시간적 일관성을 입증했다.1


이 하위 섹션은 GEM의 세 가지 제어 메커니즘에 대한 정량적 평가에 초점을 맞추며, 이는 이 연구의 핵심 기여이다.

- **자율 이동 제어 (ADE)**: **평균 변위 오차(Average Displacement Error, ADE)**는 명령된 자율 이동 궤적과 생성된 비디오에서 에이전트의 실제 궤적 사이의 편차를 측정하는 데 사용된다. 결과에 따르면 조건부 생성(궤적 사용)이 비조건부 생성보다 18% 더 정확하여 제어의 효과를 입증했다.1
- **인간 자세 제어 (AP)**: 객체 감지에서 차용한 **평균 정밀도(Average Precision, AP)** 지표는 생성된 인간 형상이 입력된 자세 골격과 얼마나 잘 일치하는지 평가하는 데 사용된다. 이 제어는 특히 크고 명확한 인간 형상에 대해 효과적인 것으로 나타났다.1
- **객체 이동 제어 및 COM 지표**: 가장 혁신적인 평가는 객체 이동에 대한 것이다. 논문은 이를 구체적으로 측정하기 위해 새로운 지표인 **객체 조작 제어(Control of Object Manipulation, COM)**를 도입했다.1
  - COM 지표는 조작된 객체의 명령된 최종 위치와 생성된 비디오에서의 실제 위치 사이의 픽셀 수준 변위 오차를 측정한다.
  - 결과는 놀랍다. 이 제어를 사용한 조건부 생성은 비조건부 생성보다 **nuScenes 데이터셋에서 68.8%, OpenDV 데이터셋에서 79%** 더 나은 성능을 보였다.1 이는 모델의 정밀한 객체 수준 제어에 대한 강력한 정량적 증거를 제공한다.


수치 외에도 생성된 비디오의 시각적 품질과 현실성은 매우 중요하다. 본 보고서는 참가자들이 GEM 및 다른 모델(예: Vista)이 생성한 비디오를 비교하도록 요청받은 **인간 평가 연구**의 결과를 논의한다.

결과는 특히 15초 길이의 긴 비디오에서 현실감, 시각적 품질, 시간적 일관성 측면에서 GEM의 결과물에 대한 강력한 인간 선호도를 보여준다.1 이 주관적인 데이터는 정량적 지표를 보완하고 모델의 최첨단 상태를 강화한다.

| 평가 지표                                                    | 모델/조건              | 성능 값           | 출처 |      |
| ------------------------------------------------------------ | ---------------------- | ----------------- | ---- | ---- |
| **FVD (↓ 낮을수록 좋음)**                                    | GEM (15초)             | **~350**          | 1    |      |
|                                                              | Vista (15초)           | ~450              | 1    |      |
| **FID (↓ 낮을수록 좋음)**                                    | GEM (15초)             | **~35**           | 1    |      |
|                                                              | Vista (15초)           | ~40               | 1    |      |
| **자율 이동 제어 (ADE, ↓ 낮을수록 좋음)**                    | GEM (조건부)           | **18% 향상**      | 1    |      |
|                                                              | GEM (비조건부)         | 기준선            | 1    |      |
| **객체 이동 제어 (COM, ↑ 높을수록 좋음)**                    | GEM (조건부, nuScenes) | **68.8% 향상**    | 1    |      |
|                                                              | GEM (조건부, OpenDV)   | **79% 향상**      | 1    |      |
| **인간 자세 제어 (AP, ↑ 높을수록 좋음)**                     | GEM (조건부, 큰 자세)  | **유의미한 향상** | 1    |      |
|                                                              | GEM (비조건부)         | 기준선            | 1    |      |
|                                                              |                        |                   |      |      |
| 표 2: GEM 성능 벤치마크 (비교 분석). 이 표는 주요 지표 전반에 걸쳐 GEM의 성능을 명확하고 통합된 시각으로 제공하며, 기준 모델 대비 성능 우위를 정량적으로 보여줍니다.1 |                        |                   |      |      |


이 장에서는 GEM과 더 넓은 월드 모델 패러다임의 한계에 대한 비판적 분석으로 넘어가, 이를 극복하기 위해 필요한 주요 연구 방향을 탐구한다.


생성 모델의 가장 중요한 과제는 **환각(hallucination)**이다. 즉, 시각적으로는 그럴듯하지만 의미론적으로나 물리적으로는 비상식적인 콘텐츠를 생성하는 것이다.38 여기에는 객체가 사라지거나, 서로를 통과하거나, 중력을 거스르는 현상이 포함된다.38

이는 치명적인 안전 문제이다. 이러한 물리적 위반을 허용하는 월드 모델에서 훈련된 계획자는 위험하거나 비효율적인 정책을 학습할 수 있다.45

환각을 완화하기 위한 연구는 여러 방향으로 진행되고 있다.

- **개선된 평가**: **VideoHallu**와 같이 생성된 비디오에서 상식 및 물리 법칙 위반을 테스트하기 위해 특별히 설계된 벤치마크를 도입하는 것이다.39

- **추론 및 자기 수정**: 몬테카를로 트리 탐색(MCTS)이나 검색 증강 생성(RAG)과 같은 기술을 사용하여 모델이 자신의 출력에 대해 추론하고 스스로 수정하여 결함 있는 콘텐츠를 생성할 가능성을 줄이는 것이다.48

  이는 제2장에서 논의된 '제어와 환각의 이중성' 문제와 직접적으로 연결된다. 환각을 완화해야 할 필요성은 물리적 제약에 대한 명시적인 이해를 갖춘 월드 모델 개발을 촉진할 것이다.


GEM이 도메인 전반에 걸쳐 인상적인 일반화 능력을 보여주지만, 모든 데이터 기반 모델은 **분포 외(Out-of-Distribution, OOD)** 시나리오, 즉 훈련 데이터와 근본적으로 다른 상황(예: 극심한 날씨, 본 적 없는 유형의 차량 또는 도로 구성)에 어려움을 겪는다.50

OOD 일반화를 개선하기 위한 현재의 접근 방식은 다음과 같다.

- **대조 학습(Contrastive Learning)**: '스타일'(예: 질감, 조명)에는 불변하지만 '내용'(예: 기하학, 인과 구조)에는 민감한 표현을 학습하도록 모델을 훈련시키는 것이다.51
- **도메인 무작위화(Domain Randomization)**: 특히 시뮬레이션-현실 전환에 강력한 기술로, 훈련 환경의 매개변수(예: 조명, 질감, 물리)를 의도적으로 광범위하게 변화시킨다. 이는 모델이 이러한 변화에 강건한 정책을 학습하고, 이전에 보지 못한 또 다른 변형인 실제 세계에 일반화할 수 있도록 강제한다.54
- **생성적 OOD 시나리오**: LLM 및 기타 생성 모델을 사용하여 롱테일 및 OOD 시나리오에 대한 텍스트 및 시각적 설명을 명시적으로 생성하고, 이를 훈련 데이터 증강이나 표적 테스트에 사용하는 것이다.50


현재의 월드 모델은 상관관계의 대가이다. 특정 픽셀 패턴 다음에 다른 픽셀 패턴이 온다는 것을 학습한다. 그러나 세계에 대한 깊은 **인과적 이해(causal understanding)**는 부족하다.41 이것이 모델이 취약하고 환각을 일으키는 경향이 있는 주요 원인이다.

**인과 강화학습(Causal Reinforcement Learning, CRL)**은 인과적 지식을 RL 에이전트에 내장하는 것을 목표로 하는 새로운 분야이다.57 인과적 월드 모델은 다음에 

*무엇이* 일어날지 예측할 뿐만 아니라 *왜* 일어나는지 이해할 것이다. 인과적 월드 모델은 개입과 분포 변화에 더 강건하고, 새로운 환경에 더 잘 일반화하며, 결정에 대해 더 해석 가능한 설명을 제공할 것이다.41 이는 진정으로 지능적이고 신뢰할 수 있는 체화된 에이전트를 구축하는 데 중요한 단계로 간주된다.42 이를 위해서는 인과 구조 추론 및 반사실적 예측을 테스트하는 

**CausalVLBench**와 같은 새로운 벤치마크와 인과 발견을 위한 새로운 지표가 필요하다.62


자동차와 같은 안전이 중요한 시스템에 배포하기 위해 GEM과 같은 생성 모델이 안전하다는 것을 어떻게 *증명*할 수 있는가? 이는 거대하고 아직 해결되지 않은 과제이다.66

- **기존 표준의 한계**: **ISO 26262**(기능 안전) 및 **ISO 21448**(SOTIF - 의도된 기능의 안전)과 같은 전통적인 자동차 안전 표준은 딥러닝 시스템의 비결정론적이고 '블랙박스'적인 특성을 위해 설계되지 않았다.66
- **AI 관련 표준의 부상**: **ISO/PAS 8800**("도로 차량 - 안전과 인공지능")과 같은 새로운 표준이 개발되고 있으며, 이는 AI 기반 시스템의 안전 보증을 위한 프레임워크를 제공하기 위해 특별히 설계되었다. 여기에는 데이터 품질, 일반화, 투명성과 같은 AI 관련 문제를 다루는 내용이 포함된다.68
- **정형 기법의 역할**: 유망하지만 도전적인 연구 방향은 신경망을 검증하기 위해 **정형 기법(formal methods)**을 적용하는 것이다.43 이는 수학적 논리를 사용하여 주어진 입력 집합에 대해 모델의 출력이 항상 안전한 범위 내에 머무를 것임을 증명하는 것을 포함한다. 현재 기술은 GEM만큼 큰 모델로 확장하는 데 어려움을 겪고 있지만 70, 순전히 통계적인 평가를 넘어 엄격하고 감사 가능한 안전 보증을 제공할 수 있는 경로를 제시한다.71 또한 생성된 비디오를 정형 시간 논리 명세에 대해 검사할 수 있는 새로운 신경-기호 검증 기술도 등장하고 있다.74


이 마지막 장에서는 GEM을 가능하게 하는 더 넓은 기술적 맥락과 그 배포가 수반하는 사회적 책임 내에 GEM을 위치시킨다.


- **대규모 컴퓨팅의 필요성**: 4000시간 이상의 비디오 데이터로 GEM과 같은 모델을 훈련하는 것은 계산적으로 엄청난 작업이며, **하드웨어 가속**, 특히 GPU의 발전 덕분에 가능해졌다.75 NVIDIA의 아키텍처(예: Hopper, Blackwell)와 플랫폼(DGX)이 이러한 대규모 생성 모델의 훈련을 가능하게 하는 역할을 한다.76
- **배포의 과제: 엣지 컴퓨팅**: 훈련은 클라우드/데이터 센터에서 이루어지지만, 자율 주행과 같은 실시간 애플리케이션을 위한 추론은 차량 내부, 즉 **엣지(edge)**에서 이루어져야 한다.79 이는 지연 시간, 전력 소비, 대역폭과 같은 다른 종류의 과제를 제기한다.79
- **하이브리드 엣지-클라우드 아키텍처**: 미래는 하이브리드 접근 방식을 포함할 가능성이 높다. 일상적인 결정은 엣지 장치(예: NVIDIA Jetson 플랫폼)의 더 작고 효율적인 모델이 처리하고 82, 더 복잡하거나 새로운 시나리오는 분석을 위해 강력한 클라우드 모델로 데이터를 업로드하여 그 결과를 다시 차량군에 전송하는 방식이다.84

월드 모델에 필요한 기술 스택-데이터 큐레이션 도구(NVIDIA NeMo Curator) 86, 시뮬레이션 플랫폼(Omniverse) 82, 훈련 하드웨어(DGX) 76, 배포 하드웨어(Jetson) 83-은 매우 복잡하다. NVIDIA와 같은 기업들은 이 전체 생태계를 구축하고 제공하고 있다. 이는 혁신을 위한 강력한 선순환 구조를 만들지만 동시에 상당한 진입 장벽을 형성한다. 이러한 맥락에서, 학계 및 연구 기여자들이 GEM을 오픈소스로 공개한 것 36은 이 기술을 민주화하려는 전략적 움직임으로 볼 수 있다. 이는 대기업이 개발하는 독점적이고 폐쇄적인 월드 모델에 대한 강력하고 최첨단인 대안을 제공한다. 따라서 GEM을 단순한 기술적 성과가 아니라 개방형 연구 생태계에 대한 중요한 기여로 조명하는 것이 중요하다.


- **대규모 데이터 수집과 개인정보 보호**: GEM에 필요한 수천 시간의 실제 주행 비디오 수집은 심각한 윤리 및 개인정보 보호 문제를 제기한다.87 이 데이터는 필연적으로 식별 가능한 사람들의 이미지, 번호판, 민감한 위치 정보를 포함한다.89
- **법률 및 규제 프레임워크**: **GDPR**과 같은 규제의 적용 가능성을 논의해야 한다. GDPR은 개인 데이터가 필요할 때만 처리되어야 하며 가능한 경우 익명화를 사용해야 한다고 규정한다.89 또한 보행자와 같은 비사용자로부터 의미 있는 동의를 얻는 것의 어려움도 다루어야 한다.89
- **감시의 위협**: **전자 프런티어 재단(EFF)**과 같은 단체의 분석을 바탕으로, 이러한 방대한 데이터셋이 기업이나 법 집행 기관의 영장을 통해 감시에 사용될 위험을 탐구해야 한다.91 이 데이터의 집계는 공공 생활의 상세하고 위축적인 초상을 만들어낼 수 있다.94
- **편향과 공정성**: 이 모델들을 훈련시키는 데 사용된 데이터는 내재된 편향(예: 특정 지리적 위치나 날씨 조건에서 주로 훈련됨)을 포함할 수 있으며, 이는 모델이 소외된 그룹이나 시나리오에 대해 성능이 저하되거나 불공정하게 작동하게 할 수 있다.46
- **완화 및 책임**: 장치 내 데이터 익명화(예: 얼굴 및 번호판 흐림 처리) 89, 연합 학습과 같은 개인정보 보호 학습 기술 95, 그리고 유출 방지를 위한 데이터 투명성 및 강력한 보안의 중요성 등 기술적, 정책적 완화 전략을 논의해야 한다.91



GEM은 최첨단 기술의 제어 가능하고 다중 모드를 지원하는 월드 모델로서 그 위치를 확고히 했다. 이 모델의 핵심 혁신은 삼중 제어 메커니즘, 다중 모드 및 다중 도메인 역량, 그리고 강력한 학습 기반 시뮬레이터로서의 기능이다. 이러한 특징들은 GEM을 단순한 데이터 생성기를 넘어, 자율 시스템의 계획 및 의사결정 능력을 근본적으로 향상시키는 도구로 만든다.


본 보고서는 월드 모델이 자율성의 핵심 과제를 기하학적 위치 추정과 규칙 기반 계획에서 데이터 기반 예측 시뮬레이션으로 전환시키는 근본적인 패러다임 변화를 대표한다는 점을 거듭 강조한다. 이는 물리적 세계에 대한 일종의 '상식' 또는 '직관'을 가진 에이전트를 구축하려는 움직임이다.12 이 새로운 패러다임은 더 높은 수준의 일반화, 강건성, 그리고 궁극적으로는 더 안전하고 유능한 자율 시스템을 약속한다.


자율 시스템 분야의 연구자 및 실무자들을 위해 다음과 같은 미래 지향적인 권고안을 제시한다.

1. **안전 및 검증 연구 우선순위화**: 생성형 월드 모델의 실제 배포에 있어 가장 큰 단일 장벽은 안전 인증을 위한 실행 가능한 경로의 부재이다. 정형 검증, 강력한 OOD 테스트, 그리고 ISO/PAS 8800과 같은 AI 관련 안전 표준에 대한 연구가 최우선 과제가 되어야 한다.
2. **인과적 혁명의 수용**: 상관관계 학습의 한계와 환각 문제를 극복하기 위해, 이 분야는 인과 추론을 월드 모델에 통합하는 데 집중적으로 투자해야 한다. 이는 진정으로 강건하고 일반화 가능한 AI를 구축하는 가장 유망한 경로이다.
3. **윤리적 데이터 거버넌스 옹호**: 이 모델들의 힘은 데이터의 규모와 직결된다. 커뮤니티는 대중의 신뢰를 유지하고 오용을 방지하기 위해 데이터 수집, 주석 달기, 사용에 대한 엄격한 윤리 지침과 개인정보 보호 기술을 적극적으로 개발하고 채택해야 한다.
4. **개방적이고 협력적인 생태계 조성**: GEM과 같은 모델의 오픈소싱은 매우 중요하다. 모델, 데이터셋, 벤치마크의 지속적인 협력과 공유는 발전을 가속화하고 폐쇄적인 독점 시스템에 대한 민주적인 대안을 제공하는 데 필수적이다.


1. GEM: A Generalizable Ego-Vision Multimodal ... - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2025/papers/Hassan_GEM_A_Generalizable_Ego-Vision_Multimodal_World_Model_for_Fine-Grained_Ego-Motion_CVPR_2025_paper.pdf
2. NVIDIA Isaac ROS Delivers AI Perception to ROS Developers | NVIDIA Technical Blog, accessed July 1, 2025, https://developer.nvidia.com/blog/nvidia-isaac-ros-delivers-ai-perception-to-ros-developers/
3. NVIDIA Isaac ROS Delivers AI Perception to ROS Developers - Silicon Valley Robotics, accessed July 1, 2025, https://svrobo.org/nvidia-isaac-ros-delivers-ai-perception-to-ros-developers/
4. Advances in Visual Simultaneous Localisation and Mapping Techniques for Autonomous Vehicles: A Review, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9694639/
5. Essential's Project GEM Previews a "Radically Different" Smartphone Design - Design Milk, accessed July 1, 2025, https://design-milk.com/essentials-project-gem-previews-a-radically-different-smartphone-design/
6. Stainless Steel Slam Latch Kits For Boats - Gemlux, accessed July 1, 2025, https://gemlux.com/collections/slam-latches
7. VISUAL POSITIONING, SLAM and NAVIGATION - Intermodalics, accessed July 1, 2025, https://www.intermodalics.ai/vision-guided-navigation
8. Understanding SLAM in Robotics and Autonomous Vehicles - Flyability, accessed July 1, 2025, https://www.flyability.com/blog/simultaneous-localization-and-mapping
9. Simultaneous localization and mapping - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping
10. Visual odometry - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Visual_odometry
11. GEM: A Generalizable Ego-Vision Multimodal World Model for Fine-Grained Ego-Motion, Object Dynamics, and Scene Composition Control - arXiv, accessed July 1, 2025, https://arxiv.org/html/2412.11198v1
12. (PDF) World Models for Cognitive Agents: Transforming Edge Intelligence in Future Networks - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/392334658_World_Models_for_Cognitive_Agents_Transforming_Edge_Intelligence_in_Future_Networks
13. Accurate localization of indoor high similarity scenes using visual slam combined with loop closure detection algorithm | PLOS One, accessed July 1, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0312358
14. World Models for Autonomous Driving: An Initial Survey (2403.02622v3) - Emergent Mind, accessed July 1, 2025, https://www.emergentmind.com/articles/2403.02622
15. arXiv:2411.14499v1 [cs.CL] 21 Nov 2024, accessed July 1, 2025, https://fi.ee.tsinghua.edu.cn/~dingjingtao/papers/WorldModel.pdf
16. Autonomous Navigation without HD Prior Maps Teddy Ort - DSpace@MIT, accessed July 1, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/147214/Ort-teddy-PhD-EECS-2022-thesis.pdf?sequence=1&isAllowed=y
17. CVPR Poster ZeroVO: Visual Odometry with Minimal Assumptions, accessed July 1, 2025, https://cvpr.thecvf.com/virtual/2025/poster/32648
18. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed July 1, 2025, https://www.mathworks.com/discovery/slam.html
19. Enhancing Real-Time Visual SLAM with Distant Landmarks in Large-Scale Environments, accessed July 1, 2025, https://www.mdpi.com/2504-446X/8/10/586
20. Scale Drift-Aware Large Scale Monocular SLAM - Robotics, accessed July 1, 2025, https://www.roboticsproceedings.org/rss06/p10.pdf
21. (PDF) Towards Visual SLAM with Memory Management for Large-Scale Environments, accessed July 1, 2025, https://www.researchgate.net/publication/325057681_Towards_Visual_SLAM_with_Memory_Management_for_Large-Scale_Environments
22. Slamcore's visual SLAM can improve safety and throughput - Robotics 24/7, accessed July 1, 2025, https://www.robotics247.com/article/slamcores_visual_slam_can_improve_safety_and_throughput
23. LiDARsim: Realistic LiDAR Simulation by Leveraging the Real World - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content_CVPR_2020/papers/Manivasagam_LiDARsim_Realistic_LiDAR_Simulation_by_Leveraging_the_Real_World_CVPR_2020_paper.pdf
24. V2X-REALM: Vision-Language Model-Based Robust End-to-End Cooperative Autonomous Driving with Adaptive Long-Tail Modeling - arXiv, accessed July 1, 2025, https://arxiv.org/html/2506.21041
25. The Future of Autonomous Driving: An Introduction to AV2.0 | Dell Technologies Info Hub, accessed July 1, 2025, https://infohub.delltechnologies.com/nl-nl/p/the-future-of-autonomous-driving-an-introduction-to-av2-0/
26. Cosmos-Drive-Dreams: Scalable Synthetic Driving Data Generation with World Foundation Models | alphaXiv, accessed July 1, 2025, https://www.alphaxiv.org/overview/2506.09042v2
27. Application of foundation models for autonomous driving: a survey of data synthesis - SPIE Digital Library, accessed July 1, 2025, [https://www.spiedigitallibrary.org/proceedings/Download?urlId=10.1117%2F12.3054764&downloadType=proceedings%20article&isResultClick=True](https://www.spiedigitallibrary.org/proceedings/Download?urlId=10.1117/12.3054764&downloadType=proceedings+article&isResultClick=True)
28. A Review of Deep Reinforcement Learning Approaches for Smart Manufacturing in Industry 4.0 and 5.0 Framework - MDPI, accessed July 1, 2025, [https://www.mdpi.com/2076-3417/12/23/12377?type=check_updateversion=1](https://www.mdpi.com/2076-3417/12/23/12377?type=check_updateversion%3D1)
29. Full article: World models and predictive coding for cognitive and developmental robotics: frontiers and challenges - Taylor & Francis Online, accessed July 1, 2025, https://www.tandfonline.com/doi/full/10.1080/01691864.2023.2225232
30. Mobile Robot Learning - UC Berkeley EECS, accessed July 1, 2025, https://www2.eecs.berkeley.edu/Pubs/TechRpts/2020/EECS-2020-203.pdf
31. [Literature Review] Generative AI for Autonomous Driving: A Review, accessed July 1, 2025, https://www.themoonlight.io/en/review/generative-ai-for-autonomous-driving-a-review
32. World Models for Autonomous Driving: An Initial Survey - OpenReview, accessed July 1, 2025, https://openreview.net/forum?id=XsOGjkKrvy
33. Autonomous Vehicle Research - Our Latest Publications - Waymo, accessed July 1, 2025, https://waymo.com/research/
34. Vista: A Generalizable Driving World Model with High Fidelity and Versatile Controllability - NIPS, accessed July 1, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/a6a066fb44f2fe0d36cf740c873b8890-Paper-Conference.pdf
35. CVPR Poster GEM: A Generalizable Ego-Vision Multimodal World Model for Fine-Grained Ego-Motion, Object Dynamics, and Scene Composition Control - CVPR 2025, accessed July 1, 2025, https://cvpr.thecvf.com/virtual/2025/poster/33119
36. vita-epfl/GEM: Official Github Repo for GEM - GitHub, accessed July 1, 2025, https://github.com/vita-epfl/GEM
37. GEM: A Generalizable Ego-Vision Multimodal World Model for Fine-Grained Ego-Motion, Object Dynamics, and Scene Composition Control - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/387105101_GEM_A_Generalizable_Ego-Vision_Multimodal_World_Model_for_Fine-Grained_Ego-Motion_Object_Dynamics_and_Scene_Composition_Control
38. 1X World Model, accessed July 1, 2025, https://www.1x.tech/discover/1x-world-model
39. VideoHallu: Evaluating and Mitigating Multi-modal Hallucinations for Synthetic Videos, accessed July 1, 2025, https://www.researchgate.net/publication/391461446_VideoHallu_Evaluating_and_Mitigating_Multi-modal_Hallucinations_for_Synthetic_Videos
40. Human–AI Safety: A Descendant of Generative AI and Control Systems Safety - arXiv, accessed July 1, 2025, https://arxiv.org/html/2405.09794v1
41. Learning Local Causal World Models with State Space Models and Attention - arXiv, accessed July 1, 2025, https://arxiv.org/html/2505.02074v1
42. The Essential Role of Causality in Foundation World Models for Embodied AI - arXiv, accessed July 1, 2025, https://arxiv.org/html/2402.06665v1
43. Formal Methods and Verification Techniques for Secure and Reliable AI - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/389097700_Formal_Methods_and_Verification_Techniques_for_Secure_and_Reliable_AI
44. Model checking deep neural networks: opportunities and challenges - Frontiers, accessed July 1, 2025, https://www.frontiersin.org/journals/computer-science/articles/10.3389/fcomp.2025.1557977/full
45. openreview.net, accessed July 1, 2025, [https://openreview.net/forum?id=3RSLW9YSgk#:~:text=Current%20world%20models%20typically%20cannot,for%20real%2Dworld%20robotics%20applications.](https://openreview.net/forum?id=3RSLW9YSgk#:~:text=Current world models typically cannot,for real-world robotics applications.)
46. A ChatGPT Moment Is Coming for Robotics. AI World Models Could Help Make It Happen., accessed July 1, 2025, https://singularityhub.com/2025/01/13/a-chatgpt-moment-is-coming-for-robotics-ai-world-models-could-help-make-it-happen/
47. VideoHallu: Evaluating and Mitigating Multi-modal Hallucinations for Synthetic Videos, accessed July 1, 2025, https://arxiv.org/html/2505.01481v1
48. Prompt-Based Monte Carlo Tree Search for Mitigating Hallucinations in Large Models - arXiv, accessed July 1, 2025, [https://arxiv.org/pdf/2501.13942?](https://arxiv.org/pdf/2501.13942)
49. Addressing ResNetVLLM's Multi-Modal Hallucinations University of Windsor. - arXiv, accessed July 1, 2025, https://arxiv.org/html/2504.14429v1
50. Generating Out-Of-Distribution Scenarios Using Language Models - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/386111197_Generating_Out-Of-Distribution_Scenarios_Using_Language_Models
51. ReCoRe: Regularized Contrastive Representation Learning of World Model - arXiv, accessed July 1, 2025, https://arxiv.org/html/2312.09056v1
52. Contrastive Unsupervised Learning of World Model with Invariant Causal Features, accessed July 1, 2025, https://openreview.net/forum?id=7DtgxVZGj-y
53. World Models for Embodied Agents - Toshiba Europe, accessed July 1, 2025, https://www.toshiba.eu/pages/eu/Cambridge-Research-Laboratory/world_models
54. Using Simulations and Domain Randomization for Autonomous Driving - OPUS, accessed July 1, 2025, https://opus4.kobv.de/opus4-hs-kempten/files/1658/Using_Simulations_and_Domain_Randomization_2.pdf
55. UNDERSTANDING DOMAIN RANDOMIZATION FOR SIM-TO-REAL TRANSFER - OpenReview, accessed July 1, 2025, https://openreview.net/pdf?id=T8vZHIRTrY
56. Domain Randomization for Sim2Real Transfer | Lil'Log, accessed July 1, 2025, https://lilianweng.github.io/posts/2019-05-05-domain-randomization/
57. libo-huang/Awesome-Causal-Reinforcement-Learning - GitHub, accessed July 1, 2025, https://github.com/libo-huang/Awesome-Causal-Reinforcement-Learning
58. A Survey on Causal Reinforcement Learning - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2302.05209
59. 9 Reinforcement Learning - Applied Causal Inference, accessed July 1, 2025, https://appliedcausalinference.github.io/aci_book/11-reinforcement-learning.html
60. Generating Causal Explanations of Vehicular Agent Behavioural Interactions with Learnt Reward Profiles - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2503.14557
61. Towards Causal Representation Learning - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2102.11107
62. CausalVLBench: Benchmarking Visual Causal Reasoning in Large Vision-Language Models - arXiv, accessed July 1, 2025, https://www.arxiv.org/pdf/2506.11034
63. CausalVLBench: Benchmarking Visual Causal Reasoning in Large Vision-Language Models - arXiv, accessed July 1, 2025, https://arxiv.org/html/2506.11034v1
64. Causal AI: Current State-of-the-Art & Future Directions | by Alex G. Lee | Medium, accessed July 1, 2025, https://medium.com/@alexglee/causal-ai-current-state-of-the-art-future-directions-c17ad57ff879
65. Tree search in DAG space with model-based reinforcement learning for causal discovery | Proceedings of the Royal Society A: Mathematical, Physical and Engineering Sciences, accessed July 1, 2025, https://royalsocietypublishing.org/doi/10.1098/rspa.2024.0450
66. A review on AI Safety in highly automated driving - Frontiers, accessed July 1, 2025, https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2022.952773/full
67. arXiv:2107.12045v3 [cs.LG] 1 Dec 2021, accessed July 1, 2025, https://arxiv.org/pdf/2107.12045
68. Ensuring AI Safety in Autonomous Vehicles: A Framework Based on ..., accessed July 1, 2025, https://www.ijisrt.com/assets/upload/files/IJISRT25APR1584.pdf
69. Functional Safety and AI for Autonomous Driving Systems - MulticoreWare, accessed July 1, 2025, https://multicorewareinc.com/functional-safety-and-ai-for-autonomous-driving-systems/
70. arXiv:2307.11784v1 [cs.LG] 20 Jul 2023, accessed July 1, 2025, https://arxiv.org/pdf/2307.11784
71. 7 Verifying the Safety of Autonomous Systems with Neural Network Controllers - James Weimer, accessed July 1, 2025, https://jamesweimer.net/pdf/2021-verisig-journal.pdf
72. Efficient Formal Safety Analysis of Neural Networks, accessed July 1, 2025, http://papers.neurips.cc/paper/7873-efficient-formal-safety-analysis-of-neural-networks.pdf
73. Formal Verification of Neural Networks for Safety-Critical Tasks in Deep Reinforcement Learning, accessed July 1, 2025, https://proceedings.mlr.press/v161/corsi21a/corsi21a.pdf
74. Neuro-Symbolic Evaluation of Text-to-Video Models using Formal Verification - CVPR 2025, accessed July 1, 2025, https://cvpr.thecvf.com/virtual/2025/poster/34285
75. AI's Leap from Information to Atoms - Viewpoints - FOV Ventures, accessed July 1, 2025, https://viewpoints.fov.ventures/p/ai-s-leap-from-information-to-atoms
76. What are World Foundation Models? | NVIDIA Glossary, accessed July 1, 2025, https://www.nvidia.com/en-us/glossary/world-models/
77. AI for Robotics | NVIDIA, accessed July 1, 2025, https://www.nvidia.com/en-us/industries/robotics/
78. The Engine Behind AI Factories | NVIDIA Blackwell Architecture, accessed July 1, 2025, https://www.nvidia.com/en-us/data-center/technologies/blackwell-architecture/
79. Edge AI for Autonomous Vehicles - XenonStack, accessed July 1, 2025, https://www.xenonstack.com/blog/edge-ai-for-autonomous-vehicles
80. Edge Computing Solutions For Enterprise - NVIDIA, accessed July 1, 2025, https://www.nvidia.com/en-us/edge-computing/
81. Leveraging Edge Computing for Generative AI - Wevolver, accessed July 1, 2025, https://www.wevolver.com/article/chapter-i-leveraging-edge-computing-for-generative-ai
82. NVIDIA Partners Showcase Cutting-Edge Robotic and Industrial AI Solutions at Automate 2025, accessed July 1, 2025, https://blogs.nvidia.com/blog/robotics-industrial-ai-automate/
83. Jetson AGX Orin for Next-Gen Robotics - NVIDIA, accessed July 1, 2025, https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/
84. Driving On the Edge - The Ways Edge Computing Will Power Autonomous Driving Solutions, accessed July 1, 2025, https://www.thefastmode.com/expert-opinion/36658-driving-on-the-edge-the-ways-edge-computing-will-power-autonomous-driving-solutions
85. Edge-Cloud Collaborative Motion Planning for Autonomous Driving with Large Language Models - arXiv, accessed July 1, 2025, https://arxiv.org/html/2408.09972v1
86. World Foundation Models: 10 Use Cases & Examples [2025] - Research AIMultiple, accessed July 1, 2025, https://research.aimultiple.com/world-foundation-model/
87. A Literature Review of Data Ethics in Autonomous Vehicles | NHSJS, accessed July 1, 2025, https://nhsjs.com/wp-content/uploads/2024/05/A-Literature-Review-of-Data-Ethics-in-Autonomous-Vehicles.pdf
88. A Literature Review of Data Ethics in Autonomous Vehicles - NHSJS, accessed July 1, 2025, https://nhsjs.com/2024/a-literature-review-of-data-ethics-in-autonomous-vehicles/
89. Data Privacy in Autonomous Vehicles - Gallio PRO, accessed July 1, 2025, https://gallio.pro/blog/data-privacy-in-autonomous-vehicles/
90. Self-Driving Cars and Data Collection: Privacy Perceptions of Networked Autonomous Vehicles - USENIX, accessed July 1, 2025, https://www.usenix.org/system/files/conference/soups2017/soups2017-bloom.pdf
91. New ALPR Vulnerabilities Prove Mass Surveillance Is a Public Safety Threat, accessed July 1, 2025, https://www.eff.org/deeplinks/2024/06/new-alpr-vulnerabilities-prove-mass-surveillance-public-safety-threat
92. The Impending Privacy Threat of Self-Driving Cars | Electronic Frontier Foundation, accessed July 1, 2025, https://www.eff.org/deeplinks/2023/08/impending-privacy-threat-self-driving-cars
93. Automated License Plate Readers - Street Level Surveillance, accessed July 1, 2025, https://sls.eff.org/technologies/automated-license-plate-readers-alprs
94. You Really Do Have Some Expectation of Privacy in Public | Electronic Frontier Foundation, accessed July 1, 2025, https://www.eff.org/deeplinks/2024/09/you-really-do-have-some-expectation-privacy-public
95. Privacy-Preserved Federated Learning for Autonomous Driving | Request PDF - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/352467826_Privacy-Preserved_Federated_Learning_for_Autonomous_Driving
96. Data Security in Autonomous Driving: Multifaceted Challenges of Technology, Law, and Social Ethics - MDPI, accessed July 1, 2025, https://www.mdpi.com/2032-6653/16/1/6
97. World Models for Autonomous Driving: An Initial Survey - arXiv, accessed July 1, 2025, https://arxiv.org/html/2403.02622


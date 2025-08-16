# 3D 표면 생성을 위한 트랜스포머 기반 확산 모델 DiffSurf에 대한 기술적 고찰

## 1. D 생성 모델의 진화하는 지형

3D 생성 모델링 분야는 최근 몇 년간 표현 방식, 모델 아키텍처, 학습 패러다임 전반에 걸쳐 근본적인 변화를 겪어왔습니다. DiffSurf와 같은 최신 모델의 혁신을 완전히 이해하기 위해서는 이러한 기술적 진화의 맥락을 파악하는 것이 필수적입니다. 이 섹션에서는 3D 표현 방식의 변화, 확산 모델의 부상, 그리고 3D 비전 분야에서 트랜스포머의 결정적 역할에 대해 고찰합니다.

### 1.1 D 표현 방식의 패러다임 전환: 명시적, 암시적, 그리고 다시 명시적으로

초기 3D 딥러닝 모델은 2D 이미지 처리에서 성공을 거둔 컨볼루션 신경망(CNN)을 3차원으로 직접 확장한 **복셀 그리드(Voxel Grids)**와 같은 명시적이고 구조화된 표현에 의존했습니다.1 복셀은 규칙적인 그리드 구조 덕분에 CNN을 적용하기 용이했지만, 해상도가 증가함에 따라 메모리와 연산량이 세제곱으로 증가하는 심각한 확장성 문제를 겪었습니다. 이로 인해 고해상도 모델의 학습이 실질적으로 불가능했고, 생성된 결과물은 종종 낮은 품질에 머물렀습니다.

이러한 한계를 극복하기 위해 등장한 것이 **포인트 클라우드(Point Clouds)**를 직접 처리하는 방식입니다. PointNet과 그 후속 연구들(PointNet++, DGCNN)은 순서가 없는 3D 포인트 집합에서 직접 특징을 학습할 수 있는 새로운 길을 열었습니다.4 이는 구조화되지 않은 원시 센서 데이터(예: LiDAR)를 직접 활용할 수 있다는 점에서 큰 진전이었으나, 포인트 간의 국소적 기하 구조와 객체 전체의 전역적 맥락을 동시에 효과적으로 포착하는 데에는 여전히 어려움이 있었습니다.

이후 3D 생성 모델링 분야는 **암시적 표현(Implicit Representations)**의 등장으로 혁명을 맞이했습니다. 부호화 거리 함수(Signed Distance Fields, SDFs)나 신경망 복사 조도장(Neural Radiance Fields, NeRFs)과 같은 암시적 함수는 3D 형상을 연속적인 함수로 표현합니다.1 이 방식은 메모리 사용량을 공간 해상도로부터 분리하여, 임의의 토폴로지를 가진 고품질의 연속적인 표면을 생성할 수 있게 했습니다.1 암시적 표현은 고품질 3D 형상 생성의 지배적인 패러다임으로 자리 잡았습니다.

그러나 DiffSurf의 접근 방식은 이러한 흐름 속에서 **명시적 표현으로의 전략적 회귀**를 보여줍니다. 암시적 표현이 강력하기는 하지만, 게임, 증강현실(AR), 가상현실(VR)과 같은 실제 산업 응용 분야에서는 여전히 **표면 메시(Surface Mesh)**가 표준적이고 효율적인 표현 방식으로 사용됩니다.8 DiffSurf의 핵심 목표는 마칭 큐브(Marching Cubes)와 같은 비용이 많이 드는 변환 과정 없이, 업계 표준 형식인 메시에 직접 최첨단 생성 기술을 적용하는 것입니다.8

### 1.2 생성 AI에서 확산 모델의 부상

잡음 제거 확산 모델(Denoising Diffusion Models)은 점진적인 잡음 추가 과정을 역으로 학습하여 데이터 분포를 모델링하는 생성 모델의 한 종류입니다.10 이 모델들은 생성적 적대 신경망(GANs)에 비해 학습이 안정적이고 고품질의 다양한 샘플을 생성할 수 있어 많은 영역에서 GAN을 능가하는 성능을 보여주었습니다.8

이러한 성공에 힘입어 확산 모델은 포인트 클라우드, 암시적 필드, 메시 등 다양한 3D 데이터 생성에 빠르게 채택되었습니다.7 3D 확산 모델 설계의 핵심 과제는 적합한 3D 표현을 선택하고 확산 과정을 어떻게 적용할지 결정하는 것입니다.7 DiffSurf는 3D 좌표에 직접 확산 과정을 적용하므로, '3D 공간 확산(3D space diffusion)' 범주에 속합니다.13

### 1.3 D 비전에서의 트랜스포머: 비구조적 데이터의 전역적 의존성 포착

기존의 CNN은 구조화된 그리드 데이터를 요구하기 때문에 순서가 없는 포인트 클라우드에는 부적합합니다. 그래프 기반 신경망(GNN)은 국소적 이웃 관계를 포착할 수는 있지만, 종종 장거리 의존성을 모델링하는 데 한계를 보입니다.4

반면, **트랜스포머(Transformers)**의 셀프 어텐션(self-attention) 메커니즘은 본질적으로 순서에 무관하며(permutation-invariant) 토큰 집합 내의 장거리 의존성을 모델링하는 데 탁월한 성능을 보입니다.15 이는 포인트 클라우드 처리에 이상적인 아키텍처입니다. 예를 들어, 캐릭터의 왼손과 오른발 사이의 관계는 그럴듯한 포즈 생성에 매우 중요하지만 공간적으로는 멀리 떨어져 있습니다. 트랜스포머는 이러한 관계를 효과적으로 학습할 수 있습니다.

DiffSurf의 아키텍처는 이러한 기술들의 의도적이고 시너지 효과를 내는 융합체입니다. 확산 프레임워크는 강력하고 안정적인 생성 과정을 제공하고, 명시적 포인트 표현은 실제 3D 그래픽스 파이프라인과 부합합니다. 그리고 트랜스포머 아키텍처는 형상의 모든 정점 간의 복잡하고 비국소적인 상호 의존성을 효과적으로 모델링함으로써, 명시적 포인트 집합에 대한 고품질 확산 모델링을 가능하게 하는 '접착제' 역할을 합니다.17 이 조합은 복셀 그리드의 확장성 문제, 초기 포인트 기반 네트워크의 국소적 한계, 암시적 표현의 간접성이라는 이전 방법들의 약점을 직접적으로 해결합니다. 즉, 고품질 명시적 메시 생성의 주된 병목 현상은 표현 방식 자체가 아니라, 그 복잡한 전역 분포를 학습할 만큼 강력한 아키텍처(트랜스포머)의 부재였음을 시사합니다.

## 2. DiffSurf: 구조적 및 수학적 기초

DiffSurf는 Yusuke Yoshiyasu와 Leyuan Sun이 ECCV 2024에서 발표한 논문에서 제안된 모델로, 명시적 3D 표면 표현에 직접 확산 모델을 적용한 혁신적인 아키텍처입니다.18 이 섹션에서는 DiffSurf의 핵심 철학, 확산 트랜스포머의 상세 구조, 확산 과정의 수학적 공식, 그리고 최종 메시 생성 파이프라인을 심층적으로 분석합니다.

### 2.1 핵심 철학: 명시적 표면 표현에 대한 직접 확산

DiffSurf의 중심 아이디어는 3D 표면의 정점(vertices)과, 선택적으로 인체 관절(joints)의 3D 좌표에 가해진 잡음을 예측하도록 학습하는 잡음 제거 확산 모델입니다.10 저자들이 밝힌 주요 기여는 다음과 같습니다: (1) 다양한 신체 형태와 포즈의 3D 표면을 생성할 수 있는 잡음 제거 확산 트랜스포머 모델, (2) 포인트-노멀(point-normal) 표현을 활용하여 인체, 동물, 인공물 등 다양한 객체 유형의 3D 형상을 생성하는 능력, (3) 점수 증류 샘플링(Score Distillation Sampling, SDS) 및 분류기 없는 안내(Classifier-Free Guidance, CFG)를 기반으로 사전 학습된 모델을 활용하여 다양한 다운스트림 작업을 처리하는 방법론 제공.10

### 2.2 확산 트랜스포머의 구조적 심층 분석

#### 2.2.1 입력 표현 및 토큰화

DiffSurf는 여러 유형의 입력을 개별 토큰으로 처리하는 다중 모드(multi-modal) 아키텍처를 채택합니다.

- **다중 모드 입력:** 사람이나 동물과 같이 관절로 움직이는 객체의 경우, 잡음이 섞인 표면 정점 좌표 $x_t \in R^{N \times 3}$와 신체 관절 좌표 $y_t \in R^{J \times 3}$를 입력으로 받습니다.11
- **포인트-노멀 표현:** 날카로운 특징을 가진 인공물의 경우, 품질 향상을 위해 잡음이 섞인 정점 좌표와 해당 표면의 법선 벡터(normal vector) $n_t \in R^{N \times 3}$를 결합하여 $N \times 6$ 차원의 행렬을 입력으로 사용합니다. 이는 모델에 중요한 국소적 기하 정보를 제공합니다.10
- **타임스텝 임베딩:** 확산 과정의 타임스텝 $t$ 또한 임베딩으로 변환되어 트랜스포머에 입력됩니다. 이를 통해 모델은 현재의 잡음 수준에 따라 잡음 예측을 조절할 수 있습니다.20

#### 2.2.2 트랜스포머 블록 구조

모델의 핵심은 256 채널의 은닉 차원을 가진 7개의 트랜스포머 블록 스택으로 구성됩니다.11 각 블록은 정점, 관절, 타임스텝 토큰의 조합을 함께 처리하며, 셀프 어텐션 메커니즘이 모드 간의 의존성(예: 관절 위치가 특정 정점 집합에 미치는 영향)을 학습하도록 합니다.

#### 2.2.3 출력 레이어: 잡음 예측 함수

최종 트랜스포머 블록에서 나온 특징들은 다층 퍼셉트론(MLP)을 통과하여, 원본 정점 및 관절 좌표에 추가되었던 잡음 벡터 $\epsilon^x_\theta$와 $\epsilon^y_\theta$를 예측합니다.20 이것이 모델의 핵심 예측 목표입니다.

### 2.3 확산 과정: 수학적 공식화

#### 2.3.1 순방향 과정 (잡음 추가)

순방향 과정은 초기 데이터 $x_0$에 $T$ 타임스텝에 걸쳐 점진적으로 가우시안 잡음을 추가하는 고정된 마르코프 체인입니다. 이 과정은 분산 스케줄 $\beta_t$에 따라 다음과 같이 정의됩니다 20:

$$
q({\bf x}_{1:T} | {\bf x}_0) = \prod _{t=1}^T q({\bf x}_{t} | {\bf x}_{t-1})
$$

$$
q({\bf x}_{t} | {\bf x}_{t-1}) = \mathcal {N} ({\bf x}_{t} ; \sqrt {1 - \beta _t} {\bf x}_{t-1}, \beta _t {\bf I})
$$

#### 2.3.2 역방향 과정 (잡음 제거)

역방향 과정의 목표는 실제 역방향 분포 $q(x_{t-1} | x_t)$를 근사하는 신경망 $p_\theta$를 학습하는 것입니다. 이는 평균이 트랜스포머 네트워크 $\epsilon_\theta$에 의해 예측되는 가우시안 분포로 매개변수화됩니다 11:

$$
p_\theta ({\bf x}_{t-1} | {\bf x}_{t}) = \mathcal {N} ({\bf x}_{t-1} ; \mu _{\theta }({\bf x}_{t}, t), \sigma ^2 {\bf I})
$$
여기서 평균 $\mu_\theta$는 예측된 잡음 $\epsilon_\theta$의 함수로 다음과 같이 표현됩니다:

$$
\mu _{\theta }({\bf x}_{t}, t) = \frac {1}{\sqrt {\alpha }_t} \left ({\bf x}_t - \frac {1- \alpha _t} {\sqrt {1 - \bar {\alpha }_t}} \epsilon _\theta ({\bf x}_t,t) \right )
$$
이때 $\alpha_t = 1 - \beta_t$이고 $\bar{\alpha}_t = \prod^t_{i=1} \alpha_i$입니다.

#### 2.3.3 단순화된 학습 목적 함수

복잡한 변분 하한(variational lower bound) 대신, 학습은 임의의 타임스텝 $t$에서 실제 추가된 잡음 $\epsilon$과 네트워크가 예측한 잡음 $\epsilon_\theta$ 사이의 평균 제곱 오차(MSE) 손실로 단순화됩니다 20:

$$
L_{simple} = \mathbb {E}_{t,{\bf x}_0,\epsilon } |

| \epsilon - \epsilon _\theta (\sqrt{\bar{\alpha}_t}{\bf x}_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon, t) ||^2_2
$$

### 2.4 거친 포인트에서 조밀한 메시로: 업샘플링 파이프라인

DiffSurf는 모든 3D 객체가 동일한 토폴로지 특성을 갖지 않는다는 점을 인식하고, 객체의 특성에 따라 업샘플링 메커니즘을 특화하는 실용적인 설계 선택을 했습니다.

- **고정된 토폴로지 (인체/동물):** 인체와 같이 메시 연결성이 일정한 객체의 경우, MLP 기반 업샘플러를 사용하여 확산 트랜스포머가 예측한 거친 정점들로부터 조밀한 메시를 생성합니다. 이 방식은 알려진 메시 구조를 보존합니다.11
- **가변 토폴로지 (인공물):** 토폴로지가 다양할 수 있는 인공물의 경우, 더 유연한 접근법을 사용합니다. 개선된 PointNet++ 모델이 포인트 클라우드를 정제하고 업샘플링한 후, Shape-As-Points(SAP)와 같은 학습 기반 표면 재구성 기술을 사용하여 최종 메시를 생성합니다.11 이는 임의의 토폴로지를 가진 객체 생성을 가능하게 합니다.

## 3. 성능 및 기능 분석

이 섹션에서는 DiffSurf의 성능을 정량적 데이터와 정성적 결과를 통합하여 비판적으로 평가하고, 다양한 다운스트림 작업에서의 활용 가능성을 분석합니다.

### 3.1 다양한 범주에 걸친 무조건적 생성

정성적 평가 결과, DiffSurf는 다양한 포즈의 인체, 여러 가지 제스처(엄지척, 브이)의 손, 동물(개, 포유류), 그리고 다양한 인공물 등 광범위한 종류의 그럴듯한 3D 표면을 성공적으로 생성하는 능력을 보여주었습니다.10 이는 모델이 여러 데이터 분포에 걸쳐 일반화될 수 있음을 입증합니다.

### 3.2 표준 벤치마크에서의 정량적 평가

#### 3.2.1 인체 생성 품질

DiffSurf의 핵심 생성 능력은 복잡하고 관절화된 클래스인 인체에 대한 평가를 통해 검증됩니다. RA-1-NNA 메트릭은 생성된 포즈의 현실성과 다양성을 평가하며, 점수가 낮을수록 우수합니다. 아래 표는 SURREAL 및 DFAUST 데이터셋에서 DiffSurf를 다른 기준 모델들과 비교한 결과입니다.

**표 1: 무조건적 인체 생성에 대한 정량적 비교 (RA-1-NNA 메트릭 [%])**

| 모델                 | SURREAL  | DFAUST   |
| -------------------- | -------- | -------- |
| Random SMPL (SD×0.2) | 81.1     | 92.8     |
| VPoser               | 60.7     | 70.2     |
| Parametric Diffusion | 59.6     | 76.2     |
| LIMP                 | 81.3     | 93.3     |
| **DiffSurf SURREAL** | **54.4** | **69.6** |
| **DiffSurf AMASS**   | **54.0** | **69.5** |

출처: 20

이 표는 DiffSurf가 매개변수 기반 모델(VPoser, Parametric Diffusion)과 비매개변수 모델(LIMP) 모두를 능가하는 성능을 보임을 명확히 보여줍니다. DiffSurf는 가장 낮은 점수를 기록하며, 정점에 대한 직접적인 확산 접근 방식이 이전 생성 모델들보다 더 높은 품질과 다양성을 가진 인체 형상을 생성한다는 것을 입증합니다.20

#### 3.2.2 D 인체 메시 복원 (HMR)

단일 이미지로부터 3D 메시를 복원하는 HMR 작업은 매우 어려운 문제이며, 강력한 생성적 사전 지식(generative prior)은 해를 더 그럴듯한 형태로 정규화하는 데 도움을 줄 수 있습니다. 아래 표는 3DPW(실외) 및 Human3.6M(실내) 데이터셋에서 DiffSurf를 최신 HMR 방법들과 비교한 결과입니다. MPVE, PA-MPJPE, MPJPE와 같은 메트릭은 낮을수록 좋습니다.

**표 2: 3D 인체 메시 복원에 대한 정량적 비교**

| 모델             | 데이터셋      | MPVE ↓    | PA-MPJPE ↓ | MPJPE ↓  |
| ---------------- | ------------- | --------- | ---------- | -------- |
| SPIN             | 3DPW          | 116.4     | 59.2       | -        |
| METRO            | 3DPW          | 119.1     | 63.0       | -        |
| GATOR            | 3DPW          | 104.5     | 56.8       | -        |
| **DiffSurf**     | **3DPW**      | **108.0** | **53.7**   | -        |
| **DiffSurf-EFT** | **3DPW**      | **102.6** | **52.6**   | -        |
| HMDiff           | Human3.6M     | -         | 32.4       | 49.3     |
| **DiffSurf**     | **Human3.6M** | -         | **36.1**   | **48.9** |

출처: 20

DiffSurf는 특히 어려운 3DPW 데이터셋에서 PA-MPJPE 53.7, Human3.6M에서 MPJPE 48.9라는 최첨단에 필적하거나 더 나은 결과를 거의 실시간 속도로 달성했습니다.18 이는 DiffSurf가 재구성 작업을 위한 효과적인 사전 지식 모델로서의 가치를 입증하는 것입니다.

### 3.3 다운스트림 작업의 숙달

DiffSurf의 진정한 힘은 단순 생성을 넘어선 다양한 응용 가능성에 있습니다. 이러한 기능들은 확산 프레임워크가 학습한 부드럽고 연속적인 잠재 공간(가우시안 잡음 공간) 덕분에 가능해진 결과물입니다.

- **포즈 조건부 생성 및 형상 변형:** 역방향 확산 과정을 3D 관절 위치에 조건부로 적용함으로써, DiffSurf는 특정 포즈에 맞는 메시를 생성할 수 있습니다. 동일한 스켈레톤 조건을 유지하면서 초기 잡음 $z$를 변경하면, 같은 포즈에 대해 다양한 신체 형상을 생성할 수 있습니다.20
- **매끄러운 모핑:** 두 형상 간의 보간은 정점 좌표 공간이 아닌 가우시안 잡음 공간에서 수행됩니다. 두 형상의 잡음 벡터를 선형 보간한 후 역방향 확산 과정을 실행하면, 인공물 없이 부드럽고 자연스러운 전환이 이루어집니다.20
- **고품질 메시 피팅 및 정제:** DiffSurf는 점수 증류 샘플링(SDS) 기술을 활용하여 최적화 기반 작업을 수행합니다.10 사전 학습된 확산 모델의 그래디언트를 사용하여 메시를 2D 키포인트와 같은 목표에 맞추도록 최적화하면서, 결과 형상이 그럴듯한 형태를 유지하도록 합니다.

**표 3: 2D 키포인트에 대한 메시 피팅 정량적 비교 (3DPW)**

| 방법                       | MPVE ↓   | PA-MPJPE ↓ |
| -------------------------- | -------- | ---------- |
| SMPLify                    | -        | 106.1      |
| LearnedGD                  | -        | 55.9       |
| HMR-EFT + SDS fitting      | 98.6     | 52.1       |
| **DiffSurf + SDS fitting** | **93.7** | **48.7**   |

출처: 20

SDS를 이용한 DiffSurf 피팅은 PA-MPJPE 48.7이라는 최첨단 성능을 달성하여, 다른 최적화 기반 피팅 방법들을 크게 능가했습니다. 이는 DiffSurf가 학습한 형상 다양체(shape manifold)가 강력한 정규화(regularizer) 역할을 수행함을 증명합니다.20

## 4. 비교 및 비판적 관점

이 섹션에서는 DiffSurf의 결과를 넘어서, 3D AI의 더 넓은 맥락 속에서 모델을 비판적으로 분석하고 잠재적인 약점을 식별합니다.

### 4.1 DiffSurf 대 암시적 표현 모델: 근본적인 트레이드오프

DiffSurf의 접근 방식을 대규모 재구성 모델(Large Reconstruction Model, LRM)과 같은 암시적 표현 기반 모델과 비교하는 것은 중요합니다.22 LRM은 Objaverse, MVImgNet과 같은 대규모 데이터셋으로 학습되어 단일 이미지로부터 암시적인 NeRF 표현을 예측합니다.

- **트레이드오프 분석:**
  - **DiffSurf (명시적):** 표준 파이프라인에서 렌더링 및 편집이 효율적인, 직접 사용 가능한 메시를 생성합니다.8 특히 인체와 같이 알려진 토폴로지를 가진 관절 객체에 적합합니다. 그러나 Objaverse와 같은 대규모 데이터셋에 존재하는 매우 다양한 객체 토폴로지에 대한 확장성은 떨어질 수 있습니다.24
  - **LRM (암시적):** 일반화 능력이 매우 뛰어나며 실제 환경의 이미지로부터 임의의 객체를 재구성할 수 있습니다.22 하지만 결과물은 NeRF이므로, 메시로 변환하기 위해 마칭 큐브와 같은 후처리 단계가 필요하며, 암시적 필드를 직접 편집하는 것은 직관적이지 않습니다.1

고품질의 명시적 생성 모델(DiffSurf)과 암시적 생성 모델(LRM)의 동시 등장은 어느 한쪽이 우월하다는 것을 의미하지 않습니다. 오히려 이는 3D 생성 분야의 전문화가 진행되고 있음을 시사합니다. DiffSurf는 일관된 구조를 가진 고품질의 애니메이션 가능한 에셋 생성에 탁월하며, LRM은 2D 이미지로부터의 범용 재구성에 뛰어납니다.

### 4.2 식별된 한계 및 개선 영역

- **미세한 기하학적 디테일의 충실도:** DiffSurf가 매우 미세한 기하학적 디테일을 포착하거나 극단적이고 비현실적인 변형을 처리하는 데 어려움을 겪을 수 있다는 비판적 분석이 있습니다.17 이는 생성 모델의 일반적인 과제이며, 저자들이 향후 연구 과제로 표현력이 풍부한 얼굴과 손가락 생성을 언급한 것과도 일맥상통합니다.20
- **계산 복잡성 및 추론 시간:** 논문은 계산 비용이나 추론 시간에 대한 심층적인 분석을 제공하지 않습니다.17 HMR 작업에서 "거의 실시간"이라고 설명되었지만 18, 확산 샘플링의 반복적인 특성은 본질적으로 단일 패스 회귀 모델보다 계산 집약적입니다. 더 높은 품질과 잠재적으로 더 높은 지연 시간 사이의 상세한 트레이드오프 분석이 필요합니다.
- **데이터 의존성 및 확장성:** DiffSurf의 성능은 특정 범주(인체, 손, 동물)의 데이터셋에서 입증되었습니다. 저자들이 "기초 3D 생성 모델"을 만들겠다는 목표를 밝혔듯이, Objaverse와 같은 대규모의 다양한 다중 범주 데이터셋으로 확장할 수 있는지 여부는 아직 해결되지 않은 과제이자 향후 연구의 핵심 방향입니다.20
- **코드 가용성:** 연구 시점에서 공개된 코드 구현이 없어 26, 연구 커뮤니티의 광범위한 채택과 검증에 일시적인 장벽이 될 수 있습니다.

## 5. 결론 및 향후 전망

이 보고서는 DiffSurf 모델에 대한 심층적인 기술적 분석을 제공했습니다. 마지막으로, 핵심적인 기여를 요약하고 이와 유사한 아키텍처의 미래 영향에 대해 전망합니다.

### DiffSurf의 핵심 기여 요약

DiffSurf는 확산 모델의 생성 능력과 트랜스포머의 구조적 강점을 성공적으로 결합하여 고품질의 명시적 3D 표면 메시를 직접 생성하고 재구성하는 데 성공했습니다. 무조건적 생성에서 최첨단 성능을 입증했으며, 인체 메시 복원 및 피팅과 같은 다운스트림 작업에서 강력한 사전 지식 모델 역할을 하여 그 접근 방식의 타당성을 검증했습니다. 또한, 특화된 입력 표현과 업샘플링 파이프라인을 통해 관절 객체와 고정 객체를 모두 처리할 수 있는 다재다능한 프레임워크를 제시했습니다.

### 5.1 D 표면을 위한 기초 모델을 향한 길

저자들은 DiffSurf의 용량을 늘리고 더 큰 규모의 데이터로 학습시켜 3D 생성을 위한 기초 모델(foundation model)로 확장하려는 포부를 밝혔습니다.20 이 길은 4장에서 식별된 한계, 특히 다양한 토폴로지로의 확장과 미세 디테일 향상 문제를 해결해야 합니다. 또한, 현재 3D 데이터셋이 종종 객체 중심적이거나 물리적 속성이 부족한 한계가 있으므로 28, 더 크고 고품질의 메시 데이터셋을 구축하거나 활용해야 할 것입니다.

### 5.2 새로운 연구 방향

- **하이브리드 표현:** 향후 연구는 명시적 메시의 직접적인 편집 가능성과 암시적 필드의 토폴로지 유연성을 결합한 하이브리드 모델을 탐색할 수 있습니다.
- **향상된 제어 및 편집 가능성:** 연구는 포즈와 형태를 넘어 텍스처, 재질 속성, 국소적 편집 등 생성 과정에 대한 더 세분화된 제어에 초점을 맞출 가능성이 높습니다.
- **물리 기반 생성:** 물리 시뮬레이터를 생성 루프에 통합하여 시각적으로 그럴듯할 뿐만 아니라 물리적으로 현실적이고 상호작용 가능한 객체를 생성하는 것은 중요한 차세대 연구 분야입니다.30 DiffSurf의 명시적 메시 출력은 이러한 통합에 매우 적합한 후보입니다.

#### **참고 자료**

1. (PDF) 3D Shape Generation: A Survey - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/publication/393183418_3D_Shape_Generation_A_Survey
2. Dig into Detailed Structures: Key Context Encoding and Semantic-based Decoding for Point Cloud Completion - OpenReview, accessed July 20, 2025, https://openreview.net/pdf/08e54eb6abdd51b81d0c2b88edf8025a7f218a45.pdf
3. Instance Segmentation of Point Clouds: from City Roads to Forests - Research Collection, accessed July 20, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/652038/Dissertation_Binbin_Xiang---0112.pdf?sequence=6&isAllowed=y
4. Vicinity-Based Abstraction: VA-DGCNN Architecture for Noisy 3D Indoor Object Classification - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/publication/352296186_Vicinity-Based_Abstraction_VA-DGCNN_Architecture_for_Noisy_3D_Indoor_Object_Classification
5. Graph Neural Networks in Point Clouds: A Survey - MDPI, accessed July 20, 2025, https://www.mdpi.com/2072-4292/16/14/2518
6. Review: Deep Learning on 3D Point Clouds - MDPI, accessed July 20, 2025, https://www.mdpi.com/2072-4292/12/11/1729
7. Diffusion models for 3D generation: A survey - SciOpen, accessed July 20, 2025, https://www.sciopen.com/article/10.26599/CVM.2025.9450452?issn=2096-0433
8. DiffSurf: A Transformer-based Diffusion Model for Generating and Reconstructing 3D Surfaces in Pose - arXiv, accessed July 20, 2025, https://arxiv.org/html/2408.14860
9. Yzmblog/SurfD: Surf-D: Generating High-Quality Surfaces of Arbitrary Topologies Using Diffusion Models (ECCV 2024) - GitHub, accessed July 20, 2025, https://github.com/Yzmblog/SurfD
10. DiffSurf: A Transformer-based Diffusion Model for Generating and Reconstructing 3D Surfaces in Pose - arXiv, accessed July 20, 2025, https://arxiv.org/html/2408.14860v1
11. [Literature Review] DiffSurf: A Transformer-based Diffusion Model for Generating and Reconstructing 3D Surfaces in Pose - Moonlight | AI Colleague for Research Papers, accessed July 20, 2025, https://www.themoonlight.io/en/review/diffsurf-a-transformer-based-diffusion-model-for-generating-and-reconstructing-3d-surfaces-in-pose
12. Diffusion Models in 3D Vision: A Survey - arXiv, accessed July 20, 2025, https://arxiv.org/html/2410.04738v2
13. Diffusion models for 3D generation: A survey - SciOpen, accessed July 20, 2025, https://www.sciopen.com/article/10.26599/CVM.2025.9450452
14. Diffusion Models in 3D Vision: A Survey - arXiv, accessed July 20, 2025, https://arxiv.org/html/2410.04738v1
15. 3D Vision with Transformers: A Survey - arXiv, accessed July 20, 2025, https://arxiv.org/pdf/2208.04309
16. PTA-Det: Point Transformer Associating Point Cloud and Image for 3D Object Detection, accessed July 20, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10052646/
17. DiffSurf: A Transformer-based Diffusion Model for Generating and ..., accessed July 20, 2025, https://www.aimodels.fyi/papers/arxiv/diffsurf-transformer-based-diffusion-model-generating-reconstructing
18. [2408.14860] DiffSurf: A Transformer-based Diffusion Model for Generating and Reconstructing 3D Surfaces in Pose - arXiv, accessed July 20, 2025, https://arxiv.org/abs/2408.14860
19. PROGRAMME GUIDE - EventHosts, accessed July 20, 2025, https://media.eventhosts.cc/Conferences/ECCV2024/ConferenceProgram.pdf
20. DiffSurf: A Transformer-based Diffusion Model for Generating and ..., accessed July 20, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/10778.pdf
21. DiffSurf addresses the unconditional generation of 3D surfaces in... - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/figure/DiffSurf-addresses-the-unconditional-generation-of-3D-surfaces-in-diverse-poses-It-can_fig1_383461091
22. lrm: large reconstruction model for single image to 3d - arXiv, accessed July 20, 2025, http://arxiv.org/pdf/2311.04400
23. LRM: Large Reconstruction Model for Single Image to 3D - Yicong Hong, accessed July 20, 2025, https://yiconghong.me/LRM/
24. MegaScenes: Scene-Level View Synthesis at Scale - European Computer Vision Association, accessed July 20, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/04207.pdf
25. MVImgNet2.0: A Larger-scale Dataset of Multi-view Images - arXiv, accessed July 20, 2025, https://arxiv.org/html/2412.01430v1
26. Papers with Code - Leyuan Sun, accessed July 20, 2025, https://paperswithcode.com/author/leyuan-sun
27. accessed January 1, 1970, https://github.com/yusukey03012/DiffSurf
28. 人工智能2024_6_7 - arXiv每日学术速递, accessed July 20, 2025, https://www.arxivdaily.com/thread/56191
29. Daily Papers - Hugging Face, accessed July 20, 2025, [https://huggingface.co/papers?q=object-centric%20images](https://huggingface.co/papers?q=object-centric+images)
30. PhysID: Physics-based Interactive Dynamics from a Single-view Image - arXiv, accessed July 20, 2025, https://arxiv.org/html/2506.17746v1
31. PhysID: Physics-based Interactive Dynamics from a Single-view Image - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/publication/392942023_PhysID_Physics-based_Interactive_Dynamics_from_a_Single-view_Image

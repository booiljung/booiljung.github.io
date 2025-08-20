# NVIDIA Grasp Data Generation 기술

## I. 서론: 로봇 파지(Grasping)의 패러다임 전환과 데이터 중심 접근법

### 0.1  로봇 파지의 근본적 도전 과제

로봇 파지(Robotic Grasping)는 로봇이 물리적 환경과 상호작용하는 데 있어 가장 근본적이면서도 핵심적인 기술이다. 이는 산업 현장의 부품 피킹(picking) 작업부터 물류 창고 자동화, 나아가 서비스 로봇이나 휴머노이드가 인간의 일상 환경에서 복잡한 임무를 수행하기까지, 거의 모든 로봇 매니퓰레이션(manipulation) 작업의 초석이 된다.1 성공적인 파지는 단순히 물체를 들어 올리는 행위를 넘어, 물체의 기하학적 형태, 질량 분포, 표면 마찰, 그리고 주변 환경과의 상호작용 등 수많은 변수를 고려해야 하는 고차원적인 문제이다.3

이 문제의 복잡성은 특히 6자유도(6-DOF, Degrees of Freedom) 파지 생성에서 극명하게 드러난다. 6-DOF 파지는 3차원 공간상의 위치($x, y, z$)와 3차원 방향(roll, pitch, yaw)을 모두 결정해야 하므로, 로봇 그리퍼가 취할 수 있는 잠재적 자세의 탐색 공간은 사실상 무한대에 가깝다.3 여기에 더해, 실제 환경에서는 다음과 같은 추가적인 도전 과제들이 존재한다:

- **미지의 객체(Unknown Objects):** 사전에 모델링되지 않은 새로운 물체에 대해서도 안정적인 파지를 생성할 수 있는 일반화 능력이 요구된다.1
- **다양한 그리퍼(Diverse Grippers):** 산업용 평행 그리퍼(parallel-jaw gripper)부터 다관절 로봇 손(dexterous hand), 흡착 그리퍼(suction gripper)에 이르기까지 다양한 말단장치(end-effector)에 적용 가능해야 한다.1
- **부분적 관측(Partial Observation):** 단일 시점의 카메라나 센서로부터 얻어지는 부분적인 포인트 클라우드(point cloud) 정보만으로도 전체 객체의 형태를 추론하고 파지를 계획해야 한다.1
- **혼잡 환경(Cluttered Environments):** 여러 물체가 서로 겹쳐 있거나 쌓여 있는 복잡한 환경에서는 목표물뿐만 아니라 주변 장애물과의 충돌을 회피하며 파지를 성공시켜야 한다.4

이러한 복합적인 요인들로 인해, 강인하고(robust) 일반화 가능한(generalizable) 파지 정책(grasp policy)을 개발하는 것은 로보틱스 분야의 오랜 난제로 남아있다.

### 0.2  분석적 접근법에서 데이터 중심 접근법으로의 전환

초기 로봇 파지 연구는 주로 분석적(analytical) 방법론에 기반을 두었다. 이 접근법은 물체와 그리퍼의 기하학적 모델, 그리고 접촉점에서의 물리 법칙을 바탕으로 힘 폐쇄(Force Closure)와 같은 안정성 조건을 수학적으로 계산하여 최적의 파지를 찾으려 시도했다.5 힘 폐쇄는 그리퍼가 물체에 가하는 힘을 통해 어떠한 외부 방해(외력 및 토크)에도 저항할 수 있는 상태를 의미하며, 파지 안정성의 이론적 기반을 제공했다.

그러나 이러한 분석적 접근법은 몇 가지 근본적인 한계에 부딪혔다. 첫째, 현실 세계의 복잡한 접촉 동역학, 특히 마찰이나 연체(soft body)의 변형과 같은 비선형적 요소를 완벽하게 모델링하는 것은 거의 불가능에 가깝다.8 둘째, 센서 노이즈나 불완전한 객체 모델로 인해 입력 정보 자체가 불확실성을 내포하고 있어, 결정론적인(deterministic) 분석 모델이 실패할 확률이 높았다. 결과적으로, 분석적 방법은 통제된 환경의 특정 객체에 대해서는 유효했지만, 미지의 객체와 비정형적 환경에 대한 일반화 성능은 매우 저조했다.

이러한 한계를 극복하기 위해, 2010년대 이후 딥러닝 기술이 부상하면서 로봇 파지 연구의 패러다임은 데이터 중심(data-driven) 접근법으로 급격히 전환되었다.3 이 새로운 패러다임은 파지 문제를 명시적인 물리 모델링이 아닌, 대규모 데이터셋으로부터 파지 성공 여부를 예측하는 패턴 인식 문제로 재정의했다. 수백만 개의 파지 예시(성공 및 실패 사례)를 학습한 심층 신경망(Deep Neural Network)은 복잡한 물리적 상호작용을 암시적으로 학습하여, 불완전한 관측 정보만으로도 미지의 객체에 대해 높은 성공률을 보이는 파지를 생성할 수 있게 되었다.6

이러한 패러다임의 전환은 단순히 기술적 변화를 넘어, 연구의 철학적 기반을 바꾸어 놓았다. 물리적 상호작용의 복잡성은 제1원리(first principles)로부터 완벽하게 모델링하기에는 너무나 어렵다는 점이 암묵적으로 받아들여졌다. 그 대신, 방대한 양의 경험 데이터(실제 또는 시뮬레이션)가 이러한 복잡한 동역학을 수작업 모델보다 더 효과적으로 근사할 수 있다는 믿음이 자리 잡게 되었다. 이제 '알고리즘'은 단순히 모델 아키텍처(예: CNN, Transformer)만을 의미하는 것이 아니라, 모델과 데이터셋의 결합체를 의미하게 되었다. 데이터셋의 규모와 품질이 모델의 설계만큼, 혹은 그 이상으로 중요해진 것이다. 이는 로보틱스 분야에서 "데이터가 곧 새로운 알고리즘"이라는 경향이 명확히 나타나는 현상이다.

### 0.3  합성 데이터(Synthetic Data)의 부상과 NVIDIA의 역할

데이터 중심 접근법의 성공은 필연적으로 '데이터 병목 현상(data bottleneck)'이라는 새로운 문제를 야기했다. 실제 로봇을 이용해 대규모 파지 데이터셋을 구축하는 것은 막대한 비용과 시간이 소요되며, 반복적인 실험 과정에서 로봇이나 물체가 손상될 위험도 크다.3 예를 들어, 수백만 개의 파지 데이터를 수집하기 위해서는 여러 대의 로봇이 수개월 동안 쉬지 않고 작업을 수행해야 할 수도 있다.

이러한 현실적인 제약을 극복하기 위한 가장 유망한 대안으로 물리 기반 시뮬레이션을 통한 합성 데이터(synthetic data) 생성이 부상했다.9 시뮬레이션 환경에서는 시간과 비용의 제약 없이 거의 무한에 가까운 양의 레이블링된 데이터를 자동으로 생성할 수 있다.11 또한, 현실에서는 구현하기 어려운 다양한 환경(조명, 재질, 객체 배치 등)을 가상으로 만들어냄으로써, 학습된 모델의 강인성과 일반화 성능을 극대화할 수 있다.

이 지점에서 NVIDIA는 자사의 핵심 역량을 바탕으로 로보틱스 분야의 데이터 문제를 해결하는 데 전략적으로 집중하기 시작했다. 세계 최고 수준의 GPU 기반 가속 컴퓨팅 기술과 사실적인 물리 엔진(PhysX)을 결합하여, NVIDIA는 로보틱스를 위한 고충실도 디지털 트윈(Digital Twin) 플랫폼인 Isaac Sim을 개발했다.2 Isaac Sim은 단순한 시각화 도구를 넘어, 물리적으로 정확한 환경에서 로봇의 행동과 센서 데이터를 시뮬레이션하고, 이를 통해 대규모 합성 데이터를 생성하는 강력한 도구로 자리매김했다.

이러한 배경 속에서, 본 보고서는 NVIDIA의 파지 데이터 생성 기술, 특히 최신 프레임워크인 GraspGen과 그 기술적 기반이 되는 Isaac Sim 플랫폼을 심층적으로 분석하고 고찰하고자 한다. GraspGen의 아키텍처적 혁신, Isaac Sim을 통한 데이터 생성 파이프라인, 그리고 그 결과물인 대규모 GraspGen 데이터셋의 특징을 상세히 다룰 것이다. 또한, 이를 DexGraspNet과 같은 동시대의 주요 연구와 비교 분석하고, 파지 안정성 평가의 이론적 기반을 검토하며, 궁극적으로 'Sim-to-Real'이라는 근본적인 도전 과제 속에서 NVIDIA의 기술이 갖는 의미와 미래 전망을 제시할 것이다.

## 1.  NVIDIA GraspGen 프레임워크 심층 분석

NVIDIA GraspGen은 로봇 파지 생성 문제에 대한 최신 데이터 중심 접근법을 집대성한 프레임워크로, 확장성, 유연성, 그리고 성능을 핵심 가치로 삼는다. 이는 단순히 하나의 모델을 넘어, 데이터 생성부터 모델 학습, 평가에 이르는 통합된 시스템을 지향한다.

### 1.1  GraspGen의 목표와 핵심 철학

GraspGen의 궁극적인 목표는 다양한 조건에 걸쳐 강인하게 작동하는 범용 6-DOF 파지 생성 시스템을 구축하는 것이다. 프레임워크 설계의 핵심 철학은 '확장성(scalability)'으로, 이는 세 가지 차원에서 구체화된다 4:

1. **다양한 구현체(Embodiments):** 특정 그리퍼에 종속되지 않고, Franka Panda와 같은 연구용 평행 그리퍼, Robotiq-2F-140과 같은 산업용 그리퍼, 그리고 흡착 그리퍼 등 여러 종류의 말단장치에 유연하게 적용될 수 있도록 설계되었다.
2. **다양한 관측 조건(Observability):** 객체의 전체 형상을 담은 완전한 포인트 클라우드뿐만 아니라, 단일 시점에서 얻은 부분적인 포인트 클라우드에서도 강인하게 작동하는 것을 목표로 한다.
3. **다양한 복잡성(Complexity):** 정돈된 환경의 단일 객체 파지 문제에서부터, 여러 객체가 뒤섞인 혼잡한 장면(cluttered scene)에서의 파지 문제까지 포괄적으로 다룬다.

이러한 확장성을 달성하기 위해 GraspGen은 값비싼 실제 로봇 데이터 수집을 지양하고, 대신 NVIDIA Isaac Sim을 활용한 대규모 합성 데이터 생성에 전적으로 의존하는 철학을 채택했다.1 이는 'Sim-to-Real' 전이 문제를 정면으로 다루겠다는 의지를 보여주며, 시뮬레이션의 충실도와 데이터 생성 파이프라인의 효율성을 극대화하는 방향으로 기술 개발이 이루어졌음을 시사한다.

### 1.2  생성 모델 아키텍처: Diffusion-Transformer

GraspGen의 핵심에는 파지 포즈 생성을 담당하는 생성 모델이 있으며, 이는 최신 확산 모델(Diffusion Model)과 트랜스포머(Transformer) 아키텍처를 결합한 'Diffusion-Transformer' 구조를 채택했다.4

파지 생성 과정을 반복적인 노이즈 제거 과정으로 모델링하는 확산 모델의 채택은 중요한 의미를 가진다. 하나의 객체에는 무수히 많은 성공적인 파지 자세가 존재할 수 있으며, 이는 수학적으로 다중 모드 분포(multi-modal distribution)를 형성한다. 기존의 회귀(regression) 기반 모델들은 이러한 분포의 평균적인 해를 예측하려는 경향이 있어 다양한 파지를 생성하는 데 한계가 있었다. 반면, 확산 모델은 노이즈로부터 점진적으로 데이터를 생성하는 방식을 통해 복잡한 다중 모드 분포를 자연스럽게 학습하고, 이를 통해 공간적으로 다양한(spatially-diverse) 파지 후보군을 생성할 수 있다.1 이러한 다양성은 혼잡 환경에서 특히 중요하다. 많은 성공적인 파지들이 주변 장애물과의 충돌이나 로봇 팔의 도달 불가능성 때문에 실행이 불가능할 수 있으므로, 최대한 많은 대안을 생성하여 모션 플래너가 최적의 해를 선택할 수 있도록 해야 하기 때문이다.4

GraspGen의 구체적인 아키텍처는 다음과 같은 두 단계로 구성된다 1:

1. **인코딩(Encoding):** 강력한 3D 포인트 클라우드 처리 백본인 PointTransformerV3 (PTv3)를 사용하여 입력된 객체 중심 포인트 클라우드를 고차원의 잠재 표현(latent representation)으로 인코딩한다. 이는 기존의 PointNet++ 기반 구조보다 더 높은 품질과 계산 효율성을 제공한다.
2. **생성(Generation):** 인코딩된 잠재 표현을 조건(condition)으로 하여, 트랜스포머 기반의 확산 모델이 파지 포즈 공간에서 반복적으로 노이즈를 예측하고 제거해 나간다. 최종적으로 노이즈가 완전히 제거되면 하나의 완전한 6-DOF 파지 포즈가 생성된다.

이 과정에서 파지 포즈는 특수 유클리드 군 SE(3)의 원소, 즉 3차원 평행이동(translation)과 3차원 회전(rotation)의 조합으로 표현된다. 모델의 안정적이고 정확한 학습을 위해 평행이동 벡터는 데이터셋 전체의 통계치를 이용해 정규화되며, 회전은 리 대수(Lie algebra)나 6D 연속 표현(6D continuous representation)과 같은 기법을 통해 표현된다.1

### 1.3  방법론적 혁신: On-Generator Discriminator Training

GraspGen 프레임워크의 또 다른 한 축은 생성된 파지들의 품질을 평가하고 순위를 매기는 판별자(Discriminator) 또는 스코어러(Scorer)이다.4 생성 모델이 아무리 다양한 파지를 만들어내도, 그중에서 성공 확률이 높은 '좋은' 파지를 선별해내지 못하면 시스템 전체의 성능은 저하될 수밖에 없다. GraspGen은 이 판별자를 학습시키는 방식에서 독창적인 혁신을 이루었다.

전통적인 판별자 학습은 사전에 구축된 정적인 오프라인 데이터셋(성공한 파지 vs. 실패한 파지)을 사용한다. 그러나 이 방식은 생성 모델이 학습 과정에서 저지르는 특유의 미묘한 오류들을 포착하기 어렵다는 한계를 가진다. 예를 들어, 시뮬레이션 데이터셋의 실패 사례는 대부분 물체와 완전히 동떨어진 명백한 실패인 경우가 많다. 하지만 학습이 진행된 생성 모델이 만들어내는 실패 사례는 '거의 성공할 뻔했지만 물체 표면과 미세하게 충돌하는' 경우처럼 훨씬 더 판별하기 어려운 형태일 가능성이 높다.1

GraspGen은 이 문제를 해결하기 위해 'On-Generator' 학습 레시피라는 새로운 패러다임을 제안했다.1 이는 판별자를 오프라인 데이터셋이 아닌, 학습 과정 중에 생성 모델(Generator)이 실시간으로 만들어내는 샘플들을 이용해 학습시키는 방식이다. 이 접근법은 단순한 학습 기법의 개선을 넘어, 생성자와 판별자 간의 공생적(symbiotic)이며 적대적(adversarial-like)인 학습 루프를 형성한다.

이 과정은 다음과 같이 해석될 수 있다. 먼저, 생성자는 현재 자신의 능력 수준에서 가장 판별하기 어려운 '까다로운 오답'들을 판별자에게 문제로 제출한다. 판별자는 이 문제들을 풀면서 자신의 결정 경계(decision boundary)를 더욱 정교하게 다듬는다. 이렇게 더 똑똑해진 판별자는 다시 생성자에게 더 정확하고 유용한 피드백(기울기 신호)을 제공하여, 생성자가 실패 모드로부터 더 효과적으로 벗어나도록 돕는다. 즉, 생성자의 발전이 판별자의 학습을 위한 더 좋은 커리큘럼을 제공하고, 판별자의 발전이 다시 생성자의 학습을 가속화하는 선순환 구조가 만들어지는 것이다. 이는 학생(생성자)과 교사(판별자)가 함께 성장하는 역동적인 교육 과정에 비유할 수 있으며, 학생의 취약점을 정확히 파악하여 맞춤형 교육을 제공하는 것과 같다.

이러한 On-Generator 학습 방식은 판별자가 생성 모델의 편향이나 시스템적인 오류 패턴에 직접적으로 노출되게 함으로써, 추론 시에 발생할 수 있는 거짓 양성(false positives)을 획기적으로 줄인다.1 이는 특히 Sim-to-Real 문제 해결에 중요한 기여를 할 수 있다. 시뮬레이션과 현실의 차이는 종종 시뮬레이션이 현실의 미묘한 실패 모드를 제대로 모델링하지 못하는 데서 발생한다. On-Generator 학습은 판별자가 '일반적으로 나쁜 파지'가 아니라 '자신의 시뮬레이션 기반 생성 모델이 만들어낼 법한 특정한 종류의 나쁜 파지'를 걸러내는 데 특화되도록 만든다. 그 결과, 실제 로봇 환경에서 더 높은 정밀도(precision)와 성공률을 달성하는 핵심 요인이 된다.

### 1.4  성능 평가 및 벤치마킹

GraspGen은 시뮬레이션과 실제 로봇 환경 모두에서 그 성능을 입증했다. 다양한 그리퍼를 사용한 단일 객체 파지 시뮬레이션에서 기존의 SOTA(State-of-the-Art) 방법들을 능가하는 성능을 보였다.1 특히, 로봇 파지 분야의 표준 벤치마크 중 하나인 FetchBench에서 SOTA 성능을 달성하며 17%의 성능 향상을 기록했다.4

더욱 주목할 만한 점은 Sim-to-Real 전이 성능이다. GraspGen은 100% 시뮬레이션 데이터로만 학습되었음에도 불구하고, 실제 로봇 플랫폼에 적용되었을 때 강력한 제로샷(zero-shot) 성능을 나타냈다.4 이는 세분화(segmentation) 오류나 센서 노이즈를 모방한 데이터 증강 기법과 더불어, On-Generator 학습을 통해 시뮬레이션 아티팩트(artifact)에 강인한 판별자를 학습시킨 결과로 분석된다.1 이러한 결과는 대규모 고품질 합성 데이터와 지능적인 학습 전략이 결합될 때, 값비싼 실제 데이터 없이도 현실 세계에서 작동하는 로봇 기술을 개발할 수 있다는 가능성을 명확히 보여준다.

## 2.  대규모 파지 데이터셋 생성의 핵심: NVIDIA Isaac Sim

NVIDIA GraspGen 프레임워크의 성공은 대규모 고품질 합성 데이터를 생성할 수 있는 강력한 기반 기술 없이는 불가능하다. 그 핵심에는 NVIDIA Isaac Sim이 자리 잡고 있다. Isaac Sim은 단순한 3D 렌더링 도구를 넘어, 로보틱스 개발의 전 과정을 지원하는 물리 기반 디지털 트윈 플랫폼이다.

### 2.1  로보틱스를 위한 디지털 트윈 플랫폼

NVIDIA Isaac Sim은 물리적으로 정확한 가상 환경을 구축하기 위한 참조 애플리케이션(reference application)으로, NVIDIA의 3D 협업 및 시뮬레이션 플랫폼인 Omniverse를 기반으로 한다.11 이는 로봇 개발자에게 세 가지 핵심 워크플로우를 제공함으로써 개발 주기를 획기적으로 단축시킨다 11:

1. **합성 데이터 생성:** 로봇 파운데이션 모델 학습을 위한 지각(perception), 이동(mobility), 물리 기반 파지(physics-based grasp) 데이터를 대규모로 생성한다.
2. **Software-in-the-Loop (SIL) 테스팅:** 개발된 로봇 제어 소프트웨어 스택을 실제 하드웨어 없이 가상 환경에서 안전하고 반복적으로 테스트한다.
3. **로봇 학습:** Isaac Lab과 같은 도구를 통해 강화학습(Reinforcement Learning) 등 다양한 학습 패러다임을 시뮬레이션 내에서 수행한다.

GraspGen의 데이터 생성은 이 중 첫 번째 워크플로우의 대표적인 사례로, Isaac Sim의 강력한 시뮬레이션 및 데이터 생성 능력을 십분 활용한다.

### 2.2  물리적 정확성의 근간: NVIDIA PhysX

합성 파지 데이터의 품질은 시뮬레이션의 물리적 정확성에 의해 좌우된다. 파지의 성공 여부는 물체와 그리퍼 간의 미세한 접촉, 마찰, 그리고 중력 하에서의 동역학에 의해 결정되기 때문이다. Isaac Sim은 NVIDIA의 고성능 물리 엔진인 PhysX를 내장하여 이러한 물리 현상을 높은 충실도(high-fidelity)로 시뮬레이션한다.11

GraspGen 데이터셋에서 각 파지에 '성공' 또는 '실패' 레이블을 부여하는 과정은 전적으로 이 물리 시뮬레이션에 의존한다. 예를 들어, 특정 파지 자세로 그리퍼를 닫아 물체를 들어 올린 후, 가상의 힘을 가하거나 흔드는 테스트(shaking tests)를 시뮬레이션 내에서 수행한다.1 이 과정에서 물체가 그리퍼에서 떨어지지 않고 안정적으로 유지되면 해당 파지는 '성공'으로 레이블링된다. 이러한 물리 기반 평가는 데이터셋의 신뢰도를 높이고, Sim-to-Real 전이 성공률을 향상시키는 데 결정적인 역할을 한다.

### 2.3  자동화된 데이터 생성 파이프라인

Isaac Sim은 대규모 데이터 생성을 자동화하기 위한 정교한 도구들을 제공한다. 이 도구들은 마치 고수준의 목표(예: "이 물체에 대한 안정적인 파지 데이터 생성")를 저수준의 기계 학습용 데이터로 변환하는 '컴파일러'처럼 작동한다. 이러한 접근 방식은 복잡한 시뮬레이션 코드를 직접 작성해야 하는 부담을 줄여주고, 연구자들이 파지 문제의 본질에 더 집중할 수 있도록 돕는다.

- **Omniverse Replicator:** Isaac Sim의 핵심 데이터 생성 엔진이다.11 Replicator는 '도메인 무작위화(Domain Randomization)'를 체계적으로 수행하는 역할을 한다. 조명, 카메라 위치, 객체의 재질 및 색상, 배경 등 씬(scene)의 다양한 속성을 절차적으로(procedurally) 무작위화하여 수천, 수만 가지의 고유한 학습 환경을 생성한다. 이렇게 생성된 데이터의 다양성은 학습된 모델이 특정 시뮬레이션 환경에 과적합(overfitting)되는 것을 방지하고, 실제 환경의 예측 불가능한 변화에 대한 일반화 성능을 크게 향상시킨다.15
- **Grasping SDG Extension:** GraspGen과 같은 파지 데이터셋 생성을 위해 특별히 설계된 확장 기능(Synthetic Data Generation Extension)이다.14 이 도구는 UI를 통해 파지 데이터 생성의 전 과정을 자동화하며, 그 과정은 다음과 같은 단계로 구성된다:
  1. **설정(Configuration):** 사용할 그리퍼의 URDF 파일, 대상 객체의 3D 모델, 그리고 그리퍼의 관절 상태(예: 열렸을 때와 닫혔을 때의 관절 값) 등을 정의한다.
  2. **파지 포즈 샘플링(Grasp Pose Sampling):** 대척 파지 샘플러(antipodal grasp sampler)와 같은 내장 알고리즘을 사용하여 객체 주위에 수백, 수천 개의 잠재적인 파지 후보 자세를 자동으로 생성한다. 이 단계에서는 그리퍼의 접근 방향, 손가락 너비 등 다양한 파라미터를 조절할 수 있다.
  3. **파지 실행 단계 정의(Grasp Execution Phases):** 실제 로봇의 동작과 유사하게, 하나의 파지 시도를 여러 단계의 시퀀스로 정의한다. 예를 들어, `Phase 1: Pre-grasp 위치로 이동`, `Phase 2: 그리퍼 닫기`, `Phase 3: 객체 들어올리기`, `Phase 4: 객체 흔들기`와 같이 복잡한 다단계 행동을 설계하고 시뮬레이션할 수 있다.
  4. **물리 기반 평가 및 저장(Physics-Based Evaluation & Saving):** 정의된 실행 단계를 각 후보 포즈에 대해 순차적으로 시뮬레이션한다. 각 단계의 성공 여부, 최종적인 객체의 안정성, 접촉력 등의 물리적 메트릭을 기록하고, 이 결과를 구조화된 형식(예: YAML)으로 저장하여 최종 데이터셋을 구축한다.
- **Grasp Editor:** 자동화된 파이프라인을 보완하는 수동 저작 도구이다.16 사용자는 3D 뷰포트에서 직접 그리퍼와 객체를 조작하여 원하는 파지 자세를 시각적으로 만든 후, 'Simulate' 버튼을 눌러 물리 엔진을 통해 그 안정성을 즉시 확인할 수 있다. 이 기능은 특정 까다로운 객체에 대한 파지를 미세 조정하거나, 자동 샘플링 알고리즘이 찾지 못하는 독특한 파지를 데이터셋에 추가하는 데 유용하다.

이처럼 Isaac Sim은 파지 데이터 생성을 위한 고수준의 인터페이스를 제공함으로써, 복잡한 물리 시뮬레이션과 데이터 로깅 과정을 추상화한다. 이는 로보틱스 연구의 민주화와 표준화에 기여한다. 서로 다른 연구팀이 동일한 플랫폼(Isaac Sim) 위에서 데이터를 생성함으로써, 결과물(데이터셋)의 비교 가능성과 재현성이 높아진다. 이는 마치 표준화된 컴파일러(예: GCC)가 소프트웨어 개발 생태계의 발전을 가속화하는 것과 같은 효과를 낳으며, NVIDIA 플랫폼의 중심적 역할을 더욱 공고히 한다.

### 2.4  Sim-to-Real 격차 해소를 위한 노력

NVIDIA는 Isaac Sim 플랫폼을 통해 Sim-to-Real 격차를 줄이기 위한 다각적인 노력을 기울이고 있다. 물리 엔진의 정확성을 높이는 것 외에도, 시각적 사실감을 극대화하는 것이 중요한 전략이다. Isaac Sim은 사실적인 렌더링(photorealistic rendering) 기술을 통해 시뮬레이션 이미지가 실제 카메라 이미지와 매우 유사하도록 만든다.11

더 나아가, NVIDIA Omniverse NuRec과 같은 최신 신경망 렌더링 기술은 실제 환경에서 촬영한 센서 데이터를 3D 가우시안 스플래팅(3D Gaussian Splatting)과 같은 기법을 사용해 상호작용 가능한 시뮬레이션 씬으로 재구성한다.11 이는 실제 세계의 복잡한 조명과 반사 특성을 시뮬레이션으로 그대로 가져와, 시각적 현실감을 전례 없는 수준으로 끌어올린다. 이와 함께 앞서 언급한 도메인 무작위화 기법을 병행하여, 모델이 특정 시각적 특징에 의존하지 않고 본질적인 기하학적, 물리적 단서를 학습하도록 유도한다. 이러한 노력들은 시뮬레이션에서 학습된 모델이 별도의 미세 조정 없이도 실제 환경에서 성공적으로 작동할 가능성을 높이는 핵심 요소이다.

## 3.  GraspGen 데이터셋의 구조와 특징

NVIDIA Isaac Sim을 통해 생성된 GraspGen 데이터셋은 그 규모, 다양성, 그리고 접근성 측면에서 기존의 파지 데이터셋들과 차별화된다. 이는 데이터 중심 로보틱스 연구를 가속화하기 위한 전략적 산물이다.

### 3.1  데이터셋의 규모와 다양성

GraspGen 데이터셋의 가장 큰 특징 중 하나는 압도적인 규모이다. 이 데이터셋은 총 5,700만 개 이상의 파지 데이터를 포함하고 있어, 딥러닝 모델이 다양한 파지 패턴을 학습하기에 충분한 양을 제공한다.12

데이터의 양뿐만 아니라 질적 다양성 또한 뛰어나다. 데이터셋은 Objaverse XL (LVIS) 컬렉션에서 선별된 8,515개의 고유한 3D 객체 메쉬를 기반으로 생성되었다.1 Objaverse는 일상용품부터 공구, 음식, 장난감에 이르기까지 광범위한 카테고리의 객체들을 포함하고 있어, 모델이 특정 객체 도메인에 국한되지 않고 범용적인 파지 능력을 학습할 수 있도록 돕는다. 이는 기존 데이터셋들이 제한된 수의 객체만을 다루어 현실 세계 적용에 한계를 보였던 문제를 해결하려는 시도이다.19

또한, 데이터셋에 사용된 모든 객체 모델은 상업적 활용이 가능한 Creative Commons 라이선스를 따르고 있어, 학계 연구자뿐만 아니라 산업계 개발자들도 자유롭게 데이터셋을 활용하여 상용 솔루션을 개발할 수 있는 길을 열어준다.1

### 3.2  다양한 그리퍼 지원

GraspGen은 특정 로봇 하드웨어에 대한 의존성을 줄이고, 다양한 '구현체(embodiments)'에 걸쳐 파지 기술이 일반화될 수 있도록 설계되었다.4 이를 위해 데이터셋은 현재 로보틱스 분야에서 널리 사용되는 세 가지 대표적인 그리퍼 유형에 대한 파지 데이터를 각각 제공한다 1:

1. **Franka Panda Gripper:** 7자유도 로봇 팔에 장착된 2지 평행 그리퍼로, 학계 연구에서 표준 플랫폼으로 널리 사용된다.
2. **Robotiq-2F-140 Gripper:** 산업 자동화 현장에서 흔히 볼 수 있는 견고한 2지 평행 그리퍼이다.
3. **Suction Gripper:** 반경 30mm의 단일 접촉점을 가진 흡착 그리퍼로, 물류 및 포장 공정에서 널리 활용된다.

이처럼 여러 그리퍼에 대한 데이터를 제공함으로써, 개발자들은 자신의 로봇 시스템에 맞는 데이터를 선택하여 모델을 학습시키거나, 혹은 여러 그리퍼 데이터를 동시에 사용하여 그리퍼의 형태에 무관하게 작동하는 보다 일반화된 파지 모델을 개발할 수 있다.

### 3.3  데이터 포맷 및 접근성

NVIDIA는 GraspGen 데이터셋을 단순한 연구 결과물로 취급하는 것을 넘어, 커뮤니티가 쉽게 접근하고 활용할 수 있는 '제품'으로 만들고자 했다. 이러한 철학은 데이터셋의 포맷, 배포 방식, 그리고 함께 제공되는 도구들에서 명확히 드러난다.

데이터셋은 대규모 데이터의 효율적인 처리를 위해 설계된 **WebDataset** 형식으로 배포된다.17 이는 데이터를 여러 개의 `.tar` 아카이브(shard)로 분할하여 저장하는 방식으로, 전체 데이터셋을 메모리에 올리지 않고도 데이터를 스트리밍 방식으로 읽어올 수 있어 학습 효율성을 높여준다. 데이터셋의 폴더 구조는 그리퍼 유형별로 명확하게 분리되어 있으며 (`franka/`, `robotiq2f140/`, `suction/`), 각 그리퍼에 대한 훈련(train) 및 검증(validation) 데이터 분할 정보가 별도의 JSON 파일로 제공되어 연구의 재현성을 보장한다.17

각 데이터 샘플에는 파지 학습에 필요한 핵심 정보들이 포함되어 있다 17:

- `'object'/'scale'`: 객체 메쉬의 스케일링 팩터 (실수).
- `'grasps'/'object_in_gripper'`: 각 파지의 성공 여부를 나타내는 불리언 마스크 (크기: `[파지 개수 X 1]`).
- `'grasps'/'transforms'`: 객체 좌표계 기준으로 표현된 그리퍼의 6-DOF 포즈. 4x4 동차 변환 행렬(Homogeneous Transformation Matrix) 형태로 제공된다 (크기: `[파지 개수 X 4 X 4]`).

데이터셋의 접근성을 극대화하기 위해, NVIDIA는 머신러닝 분야의 허브인 **Hugging Face**를 통해 GraspGen 데이터셋을 공개했다.12 이를 통해 전 세계 누구나 몇 줄의 코드로 데이터셋을 쉽게 다운로드하고 사용할 수 있다. 또한, 데이터셋을 시각화하고 이해하는 데 도움을 주기 위한 파이썬 스크립트(

`visualize_dataset.py`)와 Objaverse 객체 데이터를 다운로드하기 위한 헬퍼 스크립트(`download_objaverse.py`)를 함께 제공하여 사용자의 편의성을 높였다.17

이러한 전략은 GraspGen 데이터셋을 6-DOF 평행 그리퍼 파지 연구의 사실상 표준(de facto standard) 벤치마크로 만들려는 의도로 해석될 수 있다. 연구자들이 GraspGen 데이터셋을 널리 사용하게 되면, 이는 자연스럽게 데이터셋 생성에 사용된 Isaac Sim 시뮬레이터의 우수성을 간접적으로 입증하는 효과를 낳는다. 즉, 데이터셋의 성공이 Isaac Sim의 성공으로 이어지고, 이는 다시 더 많은 개발자들이 NVIDIA의 로보틱스 생태계로 유입되도록 만드는 강력한 선순환 구조(flywheel)를 구축하는 것이다.

#### 3.3.1 표 1: NVIDIA GraspGen 데이터셋 명세

| 특징 (Feature)                        | 명세 (Specification)                                         | 출처 (Source Snippets) |
| ------------------------------------- | ------------------------------------------------------------ | ---------------------- |
| **총 파지 수 (Total Grasps)**         | > 5,700만                                                    | 12                     |
| **총 객체 수 (Total Objects)**        | 8,515                                                        | 1                      |
| **객체 출처 (Object Source)**         | Objaverse XL (LVIS subset)                                   | 1                      |
| **객체 라이선스 (Object License)**    | 허용적 크리에이티브 커먼즈 (Permissive Creative Commons)     | 1                      |
| **그리퍼 종류 (Gripper Types)**       | 1. Franka Panda (평행 그리퍼)  2. Robotiq-2F-140 (산업용 그리퍼)  3. Suction Gripper (반경 30mm) | 1                      |
| **데이터 포맷 (Data Format)**         | WebDataset (`.tar` shards)                                   | 17                     |
| **주요 어노테이션 (Key Annotations)** | - 그리퍼 포즈 (4x4 동차 행렬)  - 파지 성공 레이블 (불리언)  - 객체 스케일 (실수) | 17                     |
| **접근성 (Accessibility)**            | Hugging Face 데이터셋 허브                                   | 12                     |
| **생성 도구 (Generation Tool)**       | NVIDIA Isaac Sim                                             | 1                      |

## 4.  비교 고찰: GraspGen 대 DexGraspNet

NVIDIA의 GraspGen은 로봇 파지 데이터 생성 분야에서 중요한 이정표이지만, 이 분야의 유일한 플레이어는 아니다. 특히, 인간형 손재주(dexterity)를 목표로 하는 DexGraspNet 프로젝트는 GraspGen과 흥미로운 대조를 이루며, 로봇 매니퓰레이션 연구의 두 가지 다른 방향성을 보여준다. 이 두 프로젝트를 비교 분석하는 것은 현재 기술의 현황과 미래를 이해하는 데 중요한 통찰을 제공한다.

### 4.1  목표의 분기점: 산업용 파지 vs. 인간형 손재주

두 프로젝트의 가장 근본적인 차이는 그들이 해결하고자 하는 문제의 본질에 있다.

- **GraspGen:** 이 프로젝트의 초점은 '넓이(breadth)'에 있다. 즉, 현재 산업 및 물류 현장에서 가장 널리 사용되는 비교적 단순한 구조의 평행 그리퍼와 흡착 그리퍼를 대상으로, 수많은 종류의 객체에 대해 안정적이고 확장 가능한 파지 솔루션을 제공하는 것을 목표로 한다.1 이는 당장 현실에 적용 가능한 실용적인 기술을 개발하려는 의도가 강하다.
- **DexGraspNet:** 이 프로젝트의 초점은 '깊이(depth)'에 있다. ShadowHand, LEAP Hand와 같이 20개 이상의 자유도를 가진 고도의 다관절 로봇 손을 대상으로, 인간과 같이 정교하고 다양한 방식으로 물체를 파지하는 능력을 구현하고자 한다.20 이는 단순히 물체를 집는 것을 넘어, 후속 조작(in-hand manipulation)까지 염두에 둔 인간 수준의 손재주(human-like dexterity)를 향한 장기적인 연구 목표를 담고 있다.

이처럼 GraspGen과 DexGraspNet은 로봇 매니퓰레이션이라는 거대한 과제에 대해 서로 다른 두 갈래의 공격로를 개척하고 있다. 하나는 현재의 산업용 하드웨어를 가지고 오늘날 가능한 것의 신뢰성과 범용성을 극한으로 밀어붙이는 방향이고, 다른 하나는 미래의 인간형 하드웨어를 가지고 내일 가능해질 정교한 조작 기술의 기반을 닦는 방향이다. 로보틱스 분야 전체의 발전을 위해서는 이 두 방향의 연구가 모두 필수적이다.

### 4.2  데이터 생성 방법론 비교

목표의 차이는 데이터 생성 방법론의 차이로 이어진다.

- **GraspGen:** 주로 기하학적 휴리스틱(예: 대척 파지 샘플링)을 통해 대량의 후보 파지를 생성하고, 이를 Isaac Sim의 물리 엔진을 통해 검증하는 방식을 사용한다.3 이 방식은 계산적으로 효율적이며, 대규모 데이터셋을 빠르게 구축하는 데 적합하다. 최종적으로는 이 데이터를 기반으로 학습된 확산 모델이 새로운 파지를 생성하는 생성적 접근법을 취한다.
- **DexGraspNet:** 고차원의 관절 공간을 탐색해야 하는 다관절 손의 특성상, 단순 샘플링 방식으로는 효율적인 파지를 찾기 어렵다. 따라서 이 프로젝트는 '미분 가능한 힘 폐쇄 추정기(differentiable force closure estimator)'라는 정교한 최적화 기반의 합성 방법을 사용한다.20 이는 파지의 안정성(힘 폐쇄)을 직접적으로 최적화 목표로 삼아, 안정적이면서도 다양한 파지를 효율적으로 찾아낸다. 생성된 모든 파지는 최종적으로 Isaac Gym 시뮬레이터에서 동역학적 안정성 검증을 거친다.20

### 4.3  데이터셋 철학의 차이: 단일 객체 vs. 혼잡한 씬

데이터셋이 담고 있는 환경의 복잡성에서도 두 프로젝트는 다른 진화 경로를 보여준다.

- **GraspGen:** 초기 버전은 주로 단일 객체(singulated objects)에 대한 파지 데이터 생성에 집중했다.4 이는 파지 생성 모델 자체의 핵심 성능, 즉 객체의 형상으로부터 좋은 파지를 추론하는 능력을 먼저 확보하려는 전략으로 볼 수 있다.
- **DexGraspNet:** 초기 버전(1.0)은 단일 객체에 집중했지만, DexGraspNet 2.0부터는 여러 객체가 무작위로 쌓여 있는 복잡한 혼잡 씬(cluttered scenes)에서의 파지 데이터 생성으로 초점을 옮겼다.23 이는 실제 환경의 복잡성을 데이터셋 단계에서부터 직접적으로 다루려는 시도이다. 혼잡 씬에서는 목표 객체 파지뿐만 아니라 주변 객체와의 충돌 회피가 매우 중요한 제약 조건으로 추가되므로, 훨씬 더 어려운 문제를 다루게 된다.

### 4.4  공통 기반 기술로서의 Isaac Sim

이러한 차이점들에도 불구하고, 두 프로젝트는 매우 중요한 공통점을 공유한다. 바로 데이터 생성과 검증의 핵심 도구로 NVIDIA의 시뮬레이션 플랫폼(Isaac Sim 및 Isaac Gym)을 사용한다는 점이다.1 DexGraspNet은 NVIDIA 외부의 학술 기관(북경대학교 등)에서 주도한 프로젝트임에도 불구하고, 자신들의 야심 찬 목표를 달성하기 위한 가장 적합한 도구로 NVIDIA의 시뮬레이터를 선택했다.

이는 Isaac Sim이 특정 기업의 내부용 툴을 넘어, 로보틱스 연구 커뮤니티 전반에서 고충실도 합성 데이터 생성을 위한 사실상의 표준(de facto standard) 플랫폼으로 인정받고 있음을 보여주는 강력한 증거이다. NVIDIA의 플랫폼 전략은 이처럼 자사의 연구뿐만 아니라 외부의 혁신적인 연구까지도 가능하게 함으로써, 전체 생태계의 발전을 가속화하고 그 중심에서 기술적 리더십을 공고히 하는 효과를 낳고 있다. 즉, 파지 문제의 '넓이'를 공략하는 GraspGen과 '깊이'를 공략하는 DexGraspNet 모두가 동일한 시뮬레이션 기반 위에서 성공적으로 수행되고 있다는 사실 자체가, 그 기반 기술의 강력함과 범용성을 입증하는 셈이다.

#### 4.4.1 표 2: 주요 파지 데이터셋 비교 분석

| 특징 (Feature)                        | **NVIDIA GraspGen**                     | **DexGraspNet 1.0**                | **DexGraspNet 2.0**                 | **ACRONYM**                 |
| ------------------------------------- | --------------------------------------- | ---------------------------------- | ----------------------------------- | --------------------------- |
| **주요 초점 (Primary Focus)**         | 확장 가능한 6-DOF 파지                  | 단일 객체 다관절 파지              | 혼잡 씬 다관절 파지                 | 물리 기반 6-DOF 파지        |
| **그리퍼 종류 (Gripper Types)**       | 평행 그리퍼 (Franka, Robotiq), 흡착     | 다관절 (ShadowHand, Allegro, MANO) | 다관절 (LEAP Hand)                  | 평행 그리퍼 (Franka)        |
| **객체 수 (# Objects)**               | ~8,500                                  | ~5,300                             | ~1,300                              | ~8,800                      |
| **파지 수 (# Grasps)**                | > 5,700만                               | ~130만                             | > 4억 2,700만                       | ~1,770만                    |
| **씬 복잡도 (Scene Complexity)**      | 주로 단일 객체                          | 단일 객체                          | 고도로 혼잡함                       | 단일 객체                   |
| **생성 방법 (Generation Method)**     | 휴리스틱 샘플링 + 물리 시뮬 + 확산 모델 | 미분 가능 힘 폐쇄 최적화           | 단일 객체 합성 후 씬 내 충돌 필터링 | 휴리스틱 샘플링 + 물리 시뮬 |
| **시뮬레이션 환경 (Simulation Env.)** | NVIDIA Isaac Sim                        | NVIDIA Isaac Gym                   | NVIDIA Isaac Gym                    | PyBullet (via FleX)         |
| **출처 (Source Snippets)**            | 1                                       | 20                                 | 23                                  | 13                          |

## 5.  파지 안정성 및 품질 평가의 이론적 기반

데이터 중심의 파지 모델을 학습시키고 평가하기 위해서는 '좋은 파지'가 무엇인지 정량적으로 정의할 수 있는 기준이 필요하다. 이러한 기준은 로보틱스 분야에서 수십 년간 연구되어 온 이론적 개념들에 뿌리를 두고 있다.

### 5.1  힘 폐쇄 (Force Closure): 안정성의 이분법적 척도

파지 안정성을 논할 때 가장 기본이 되는 개념은 **힘 폐쇄(Force Closure)**이다.32 힘 폐쇄란, 그리퍼가 각 접촉점을 통해 물체에 적절한 힘을 가함으로써, 어떠한 방향으로든 물체에 가해지는 임의의 외부 힘과 토크(이를 합쳐 **렌치(wrench)**라 부른다)에 저항하여 물체의 움직임을 완전히 구속할 수 있는 상태를 의미한다.34

힘 폐쇄가 달성된 파지는 이론적으로 절대 실패하지 않는, 완벽하게 안정적인 파지로 간주된다. 외부에서 물체를 밀거나 당기거나 비틀려고 해도, 그리퍼는 손가락의 힘을 조절하여 그에 상응하는 반력을 만들어내 물체를 제자리에 고정시킬 수 있다.32 이 개념은 마찰력의 존재를 가정하며, 마찰이 없는 이상적인 상황에서의 **형태 폐쇄(Form Closure)**와는 구분된다.33

힘 폐쇄는 파지의 성공 여부를 판단하는 '성공' 또는 '실패'의 이분법적(binary) 척도를 제공한다. 이 때문에 DexGraspNet 2.0과 같은 데이터셋은 생성되는 모든 파지 데이터가 힘 폐쇄 조건을 만족하도록 최적화 과정을 거친다.24 이는 데이터셋에 포함된 '성공' 레이블의 이론적 안정성을 보장하기 위함이다.

### 5.2  렌치 공간 (Wrench Space): 파지 능력의 정량화

힘 폐쇄는 파지가 안정적인지 아닌지만을 말해줄 뿐, '얼마나' 안정적인지에 대한 정보는 제공하지 못한다. 두 개의 다른 파지가 모두 힘 폐쇄를 만족하더라도, 하나는 겨우 안정성을 유지하는 반면 다른 하나는 매우 큰 외부 방해에도 견딜 수 있다. 이러한 파지의 '품질'을 정량적으로 평가하기 위해 **렌치 공간(Wrench Space)**이라는 개념이 도입되었다.

렌치(Wrench)는 물체의 한 기준점(보통 질량 중심)에 작용하는 3차원 힘 벡터 $f$와 3차원 토크 벡터 $\tau$를 결합한 6차원 벡터 $w =^T$로 정의된다.36

**파지 렌치 공간(Grasp Wrench Space, GWS)**은 특정 파지 구성을 통해 로봇이 물체에 가할 수 있는 모든 가능한 렌치들의 집합을 의미한다.37 각 접촉점에서는 마찰 원뿔(friction cone) 내에서 힘을 가할 수 있는데, 이 모든 접촉점에서 나오는 힘들을 조합하여 만들어낼 수 있는 모든 렌치들이 6차원 공간상에서 하나의 볼록 다포체(convex hull)를 형성하며, 이것이 바로 GWS이다.37

GWS를 이용하면 힘 폐쇄를 기하학적으로 쉽게 판별할 수 있다. 만약 GWS가 6차원 렌치 공간의 원점을 내부에 포함한다면, 이는 양의 방향과 음의 방향 모두에 대해 렌치를 가할 수 있다는 의미이므로, 그 파지는 힘 폐쇄를 만족한다.38

### 5.3  파지 품질 메트릭 (Grasp Quality Metrics)

GWS의 형태와 크기는 파지의 품질을 나타내는 연속적인(continuous) 척도를 제공한다. 가장 널리 사용되는 파지 품질 메트릭은 **GWS에 내접하는 가장 큰 6차원 초구(hypersphere)의 반지름**이다.37 이 반지름 값은 해당 파지가 모든 방향의 외부 방해에 대해 최소한으로 저항할 수 있는 힘/토크의 크기를 나타낸다. 따라서 반지름이 클수록 더 강인하고 안정적인 파지로 평가된다. 이 외에도 GWS의 부피 자체를 품질 척도로 사용하기도 한다.37

이러한 이론적 메트릭들은 파지 데이터 생성 과정에서 최적화의 목표 함수로 사용되거나, 생성된 파지의 품질을 평가하는 기준으로 활용된다. 그러나 이러한 분석적 메트릭과 실제 물리적 성공 사이에는 미묘한 차이가 존재한다.

예를 들어, DexGraspNet과 GraspGen과 같은 최신 데이터 생성 파이프라인은 힘 폐쇄와 같은 분석적 개념을 최적화 과정에서 활용하기는 하지만24, 최종적인 '성공' 레이블은 Isaac Sim과 같은 고충실도 물리 시뮬레이터 내에서의 동역학적 테스트(예: 물체를 들어 올리고 흔들기)를 통해 결정한다.1 이는 '좋은 파지'의 정의가 이론적 전능함(모든 가능한 외력에 대한 저항)에서 실용적 유효성(특정 작업 시뮬레이션에서의 성공)으로 이동하고 있음을 시사한다. 이 새로운 정의는 더 현실적이고 작업과 관련성이 높지만, 한편으로는 시뮬레이터의 물리적 정확성에 전적으로 의존하게 된다. 만약 시뮬레이터의 마찰 모델이나 접촉 동역학이 현실과 미세한 차이를 보인다면, 데이터셋의 '성공' 레이블 자체에 편향이 생길 수 있다. 즉, 모델이 현실에는 존재하지 않는 시뮬레이터의 특정 허점(artifact)을 이용하도록 학습될 위험이 있으며, 이는 Sim-to-Real 격차의 또 다른 원인이 될 수 있다. 결국, 시뮬레이션 기반 데이터 생성의 성공은 시뮬레이터 자체의 충실도에 달려있다는 중요한 결론에 도달하게 된다.

더 나아가, 파지의 품질은 단순히 안정성뿐만 아니라 '작업 적합성'의 관점에서도 평가될 수 있다. **작업 렌치 공간(Task Wrench Space, TWS)**은 '망치를 휘두르기'나 '병뚜껑 돌리기'와 같은 특정 작업을 수행하는 동안 물체에 가해질 것으로 예상되는 렌치들의 집합이다.39 이상적인 작업 지향적 파지는 GWS가 해당 작업의 TWS를 완전히 포함하여, 작업 중 발생하는 모든 외력에 효과적으로 대응할 수 있어야 한다. 이는 미래의 파지 데이터셋이 단순한 '안정적 파지'를 넘어 '목적 지향적 파지'로 발전해야 함을 시사한다.

## 6.  종합 고찰 및 미래 전망: Sim-to-Real의 과제와 기회

NVIDIA의 GraspGen과 같은 대규모 합성 데이터 기반 접근법은 로봇 파지 기술을 전례 없는 수준으로 발전시켰지만, 동시에 'Sim-to-Real'이라는 근본적인 도전 과제를 수면 위로 끌어올렸다. 시뮬레이션에서 완벽하게 작동하는 모델을 어떻게 현실 세계에서도 동일한 성능으로 작동하게 할 것인가의 문제는 이 분야의 성패를 가르는 핵심 과제로 남아있다.

### 6.1  해결되지 않은 과제: Sim-to-Real 격차

Sim-to-Real 격차(reality gap)는 시뮬레이션 환경과 실제 물리 세계 간의 필연적인 불일치에서 발생한다.13 이 격차의 주요 원인은 다음과 같이 분석될 수 있다:

- **물리 파라미터의 불일치:** 시뮬레이션에서 설정된 마찰 계수, 객체의 질량 및 관성 모멘트, 재질의 탄성 및 강성 등은 실제 값의 근사치일 뿐이다. 특히, 동일한 물체라도 표면의 상태나 온도에 따라 마찰이 변하는 등, 현실의 물리적 속성은 시뮬레이션이 가정하는 고정된 값이 아니다.3
- **센서 모델링의 한계:** 실제 로봇이 사용하는 RGB-D 카메라는 조명 변화에 따른 색상 왜곡, 반사 및 투명 객체로 인한 깊이 값의 소실, 센서 자체의 노이즈 등 복잡한 현상을 겪는다. 시뮬레이션에서 이러한 모든 현상을 완벽하게 재현하는 것은 매우 어렵다. 이로 인해 실제 포인트 클라우드는 시뮬레이션 데이터에 비해 위치 드리프트(positional drift)나 구조적 왜곡(structural distortion)을 포함하게 되며, 이는 3D 공간의 정밀도에 민감한 파지 모델의 성능을 저하시키는 주된 요인이 된다.13
- **접촉 동역학의 복잡성:** 그리퍼와 객체 사이의 접촉은 단순한 점 접촉이 아니라 복잡한 면 접촉이며, 특히 연체(soft body)나 다중 접촉(multi-contact) 상황에서의 상호작용은 현재의 물리 엔진으로도 완벽하게 모델링하기 어렵다.44

이러한 격차 때문에 시뮬레이션에서 99%의 성공률을 보이는 파지 정책이라도 실제 로봇에 배포되었을 때는 그 성능이 현저히 떨어질 수 있다.42

### 6.2  격차 해소를 위한 최신 연구 동향

이러한 Sim-to-Real 격차를 해소하기 위해 로보틱스 커뮤니티에서는 다양한 전략들이 활발히 연구되고 있다.

- **도메인 무작위화 (Domain Randomization):** 가장 널리 사용되는 접근법 중 하나로, 시뮬레이션 환경의 시각적 속성(조명, 텍스처, 카메라 각도)과 물리적 속성(마찰, 질량)을 매우 넓은 범위에 걸쳐 무작위로 변경하며 모델을 학습시키는 방식이다.9 이는 모델이 특정 시뮬레이션 환경의 '특징'에 과적합되는 것을 막고, 현실 세계를 포함한 '보지 못한 다양한 환경'에 대한 일반화 성능을 높이는 효과를 낳는다. "현실 세계는 단지 또 하나의 변형일 뿐(reality is just another variation)"이라는 철학에 기반한다.46
- **체계적인 현실성 증대 (Systematic Realism):** 도메인 무작위화와는 반대로, 시뮬레이션을 최대한 현실과 가깝게 만들려는 노력이다. 이는 사실적인 렌더링 기술을 사용하고, 실제 객체와 로봇의 물리 파라미터를 정밀하게 측정하여 시뮬레이터에 반영하는 것을 포함한다.
- **도메인 적응 (Domain Adaptation):** 시뮬레이션 데이터와 실제 데이터를 모두 사용하여 두 도메인 간의 특징(feature) 분포 차이를 줄이도록 모델을 학습시키는 기법이다. 생성적 적대 신경망(GAN) 등을 이용하여 시뮬레이션 이미지를 실제 이미지처럼 보이게 변환하거나, 두 도메인의 특징 공간을 정렬(align)하는 방식 등이 있다.43
- **Real-to-Sim-to-Real 폐쇄 루프:** 최근에는 전통적인 단방향 'Sim-to-Real' 패러다임을 넘어, 현실과 시뮬레이션이 상호작용하는 순환적인 폐쇄 루프(closed loop)를 구축하려는 시도가 나타나고 있다. 이 접근법은 먼저 실제 환경에서 얻은 데이터를 사용하여 시뮬레이터 자체를 보정하고 현실에 가깝게 만든다(Real-to-Sim).44 그 다음, 이렇게 개선된 시뮬레이터에서 더 나은 정책을 학습시켜 다시 현실 세계에 배포한다(Sim-to-Real). 이 순환 구조는 로봇이 현실 세계에서 겪는 경험(특히 실패 경험)을 통해 자신의 가상 훈련장을 지속적으로 개선해 나가는, 강력한 자기 개선 시스템을 구축할 수 있는 가능성을 제시한다. 이는 마치 현장에 배치된 로봇 부대가 단순한 작업 수행자가 아니라, 중앙의 '디지털 트윈' 시뮬레이터를 개선하기 위한 데이터 수집 장치로서 기능하는 것과 같다. 이러한 접근법은 Sim-to-Real 격차를 점진적으로, 그리고 체계적으로 줄여나갈 수 있는 가장 유망한 방향 중 하나로 평가된다.

### 6.3  파지 데이터 생성의 미래 방향

Sim-to-Real 문제와 더불어, 파지 데이터 생성 자체도 더욱 정교하고 다각적인 방향으로 발전할 것이다.

- **다중 모드 및 언어 기반 파지:** 현재의 파지 데이터는 대부분 기하학적 정보에 국한되어 있다. 미래의 데이터셋은 "컵의 손잡이를 잡아 옮겨줘"와 같은 자연어 지시를 이해하고 수행할 수 있도록, 시각 정보와 언어 정보를 결합한 다중 모드(multi-modal) 데이터셋으로 확장될 것이다.5 이는 인간과 로봇이 자연스럽게 소통하고 협업하는 데 필수적인 기술이다.
- **작업 및 의도 지향적 파지 (Task- and Intent-Oriented Grasping):** 단순히 물체를 안정적으로 '드는' 파지를 넘어, 후속 작업을 고려한 파지 데이터가 중요해질 것이다. 예를 들어, '칼을 자르기 위해 잡는 방식'과 '칼을 건네주기 위해 잡는 방식'은 완전히 다르다. 이러한 작업의 의도(intent)를 반영하는 데이터셋은 로봇이 더 지능적이고 상황에 맞는 행동을 하는 데 기여할 것이다.13
- **인간 데이터의 적극적 활용:** 인간의 손은 가장 뛰어난 파지 및 조작 도구이다. 모션 캡처나 비디오를 통해 수집된 방대한 양의 인간 파지 데이터를 로봇 손의 데이터로 변환(retargeting)하는 기술은, 로봇이 인간처럼 자연스럽고 효율적인 파지 전략을 학습하는 데 중요한 자원이 될 것이다.49
- **생성 모델의 고도화:** 확산 모델을 필두로 한 생성 모델 기술은 계속해서 발전할 것이다. 미래에는 물리 법칙을 생성 과정에 직접 통합하거나(physics-informed models), 더 적은 데이터로도 고품질의 파지를 생성할 수 있는 모델들이 등장하여 데이터 생성의 효율성과 품질을 한 단계 더 끌어올릴 것이다.51

### 6.4  NVIDIA의 전략적 포지셔닝과 물리 AI의 미래

NVIDIA의 Grasp Data Generation에 대한 고찰은 단순히 하나의 기술이나 데이터셋을 넘어, 물리적 세계와 상호작용하는 차세대 AI, 즉 **'물리 AI(Physical AI)'** 시대를 주도하려는 NVIDIA의 거대한 비전과 전략의 일부로 이해해야 한다.2

NVIDIA는 이 분야에서 독보적인 수직 통합 생태계를 구축하고 있다.

1. **컴퓨팅 하드웨어 (GPU):** 대규모 시뮬레이션과 딥러닝 모델 학습에 필수적인 가속 컴퓨팅 인프라를 제공한다.
2. **시뮬레이션 플랫폼 (Omniverse & Isaac Sim):** 물리적으로 정확한 디지털 트윈을 생성하고, 합성 데이터를 만들며, 로봇을 테스트하는 가상 세계를 제공한다.
3. **AI 모델 및 프레임워크 (GraspGen, GR00T, Isaac Lab):** 특정 로봇 기술(파지)을 위한 사전 학습된 모델과, 범용 휴머노이드 로봇을 위한 파운데이션 모델, 그리고 로봇 학습을 위한 도구들을 제공한다.

이 세 가지 요소는 NVIDIA가 '세 개의 컴퓨터(Three-Computer)' 솔루션이라고 부르는 아키텍처를 형성하며, 시뮬레이션에서의 학습, 디지털 트윈을 통한 검증, 그리고 실제 로봇으로의 배포에 이르는 물리 AI 개발의 전체 파이프라인을 아우른다.2

결론적으로, NVIDIA의 Grasp Data Generation 기술은 로보틱스의 데이터 병목 현상을 해결하는 중요한 열쇠이자, 자사의 하드웨어 및 소프트웨어 플랫폼의 가치를 극대화하는 전략적 도구이다. 대규모 고품질 합성 데이터를 통해 로봇의 지능을 발전시키고, 그 과정에서 NVIDIA의 기술 생태계에 대한 의존도를 높임으로써, 다가오는 물리 AI 시대의 표준을 정립하고 시장을 선도하려는 NVIDIA의 장기적인 청사진이 담겨 있는 것이다. 이들의 노력은 로봇이 가상 세계에서 배우고 현실 세계에서 행동하는 미래를 더욱 앞당기고 있다.

#### **참고 자료**

1. NVIDIA AI Releases GraspGen: A Diffusion-Based Framework for 6-DOF Grasping in Robotics - MarkTechPost, 8월 20, 2025에 액세스, https://www.marktechpost.com/2025/07/26/nvidia-ai-releases-graspgen-a-diffusion-based-framework-for-6-dof-grasping-in-robotics/
2. AI for Robotics | NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/industries/robotics/
3. New NVIDIA Research Helps Robots Improve their Grasp, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/new-nvidia-research-helps-robots-improve-their-grasp/
4. GraspGen: A Diffusion-based Framework for 6-DOF Grasping with On-Generator Training, 8월 20, 2025에 액세스, https://arxiv.org/html/2507.13097v1
5. Sim-Grasp: Learning 6-DOF Grasp Policies for Cluttered Environments Using a Synthetic Benchmark, 8월 20, 2025에 액세스, https://ntrs.nasa.gov/api/citations/20250002252/downloads/Sim_Grasp_final_1.pdf
6. Real-Time Generative Grasping with Spatio-Temporal Sparse Convolution - Oregon State University, 8월 20, 2025에 액세스, https://research.engr.oregonstate.edu/rdml/sites/research.engr.oregonstate.edu.rdml/files/icra23_0859_fi_0.pdf
7. Deep Learning Approaches to Grasp Synthesis: A Review - Electrical Engineering and Computer Science, 8월 20, 2025에 액세스, https://web.eecs.umich.edu/~stellayu/teach/2023action/papers/2022newburyGraspSynthesisReview.pdf
8. Closing the Loop for Robotic Grasping: A Real-time, Generative Grasp Synthesis Approach - Robotics, 8월 20, 2025에 액세스, https://www.roboticsproceedings.org/rss14/p21.pdf
9. NVIDIA Research Puts Smart Picking Within Grasp | NVIDIA Technical Blog, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/nvidia-research-puts-smart-picking-within-grasp/
10. Sim-to-Real Transfer of Accurate Grasping with Eye-In-Hand Observations and Continuous Control - Research at NVIDIA, 8월 20, 2025에 액세스, [https://research.nvidia.com/sites/default/files/pubs/2017-12_Sim-to-Real-Transfer-of/Sim-to-Real%20Transfer%20of%20Accurate%20Grasping%20with%20Eye-In-Hand%20Observations%20and%20Continuous%20Control.pdf](https://research.nvidia.com/sites/default/files/pubs/2017-12_Sim-to-Real-Transfer-of/Sim-to-Real Transfer of Accurate Grasping with Eye-In-Hand Observations and Continuous Control.pdf)
11. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 20, 2025에 액세스, https://developer.nvidia.com/isaac/sim
12. Official repo for GraspGen: A Diffusion-based Framework for 6-DOF Grasping - GitHub, 8월 20, 2025에 액세스, https://github.com/NVlabs/GraspGen
13. ACRONYM: A Large-Scale Grasp Dataset Based on Simulation | Request PDF, 8월 20, 2025에 액세스, https://www.researchgate.net/publication/355429221_ACRONYM_A_Large-Scale_Grasp_Dataset_Based_on_Simulation
14. Grasping Synthetic Data Generation - Isaac Sim Documentation, 8월 20, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/latest/synthetic_data_generation/tutorial_replicator_grasping_sdg.html
15. What Is Isaac Sim? - NVIDIA Omniverse, 8월 20, 2025에 액세스, https://docs.omniverse.nvidia.com/isaacsim
16. Grasp Editor - Isaac Sim Documentation, 8월 20, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/robot_setup/grasp_editor.html
17. nvidia/PhysicalAI-Robotics-GraspGen · Datasets at Hugging Face, 8월 20, 2025에 액세스, https://huggingface.co/datasets/nvidia/PhysicalAI-Robotics-GraspGen
18. R²D²: Adapting Dexterous Robots with NVIDIA Research Workflows and Models - Edge AI and Vision Alliance, 8월 20, 2025에 액세스, [https://www.edge-ai-vision.com/2025/04/r%C2%B2d%C2%B2-adapting-dexterous-robots-with-nvidia-research-workflows-and-models/](https://www.edge-ai-vision.com/2025/04/r²d²-adapting-dexterous-robots-with-nvidia-research-workflows-and-models/)
19. Large-Scale Grasp Dataset from Foundation Models - Infovaya, 8월 20, 2025에 액세스, https://events.infovaya.com/uploads/documents/pdfviewer/a8/e2/133987-0520.pdf
20. PKU-EPIC/DexGraspNet - GitHub, 8월 20, 2025에 액세스, https://github.com/PKU-EPIC/DexGraspNet
21. DexGraspNet: A Large-Scale Robotic Dexterous Grasp Dataset for General Objects Based on Simulation - He Wang, 8월 20, 2025에 액세스, https://pku-epic.github.io/DexGraspNet/
22. DexGraspNet: Large-Scale Dataset for Dexterous Grasping - Emergent Mind, 8월 20, 2025에 액세스, https://www.emergentmind.com/papers/2210.02697
23. DexGraspNet 2.0: Learning Generative Dexterous Grasping in Large-scale Synthetic Cluttered Scenes | alphaXiv, 8월 20, 2025에 액세스, https://www.alphaxiv.org/overview/2410.23004v1
24. DexGraspNet 2.0: Learning Generative Dexterous Grasping in Large-scale Synthetic Cluttered Scenes - OpenReview, 8월 20, 2025에 액세스, https://openreview.net/pdf/885c47df6263f6be3a883d6b1bcd3e55048d7f2d.pdf
25. DexGraspNet 2.0: Learning Generative Dexterous Grasping in Large-scale Synthetic Cluttered Scenes - GitHub, 8월 20, 2025에 액세스, https://raw.githubusercontent.com/mlresearch/v270/main/assets/zhang25j/zhang25j.pdf
26. Spotlight: Galbot Builds a Large-Scale Dexterous Hand Dataset for Humanoid Robots Using NVIDIA Isaac Sim, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/spotlight-galbot-builds-a-large-scale-dexterous-hand-dataset-for-humanoid-robots-using-nvidia-isaac-sim/
27. DexGraspNet: A Large-Scale Robotic Dexterous Grasp Dataset for General Objects Based on Simulation - Tengyu Liu, 8월 20, 2025에 액세스, https://tengyu.ai/assets/pdf/ICRA23_DexGraspNet.pdf
28. DexGraspNet 2.0: Learning Generative Dexterous Grasping in Large-scale Synthetic Cluttered Scenes - He Wang, 8월 20, 2025에 액세스, https://pku-epic.github.io/DexGraspNet2.0/
29. DexGraspNet 2.0: Learning Generative Dexterous Grasping in Large-scale Synthetic Cluttered Scenes - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2410.23004v1
30. ACRONYM: A Large-Scale Grasp Dataset Based on Simulation - Research at NVIDIA, 8월 20, 2025에 액세스, https://research.nvidia.com/publication/2021-05_acronym-large-scale-grasp-dataset-based-simulation
31. ACRONYM: A Large-Scale Grasp Dataset Based on Simulation - Google Sites, 8월 20, 2025에 액세스, https://sites.google.com/view/graspdataset
32. Mastering Force Closure in Robotics - Number Analytics, 8월 20, 2025에 액세스, https://www.numberanalytics.com/blog/force-closure-linear-algebra-robotics
33. On the Closure Properties of Robotic Grasping * - Centro di Ricerca Enrico Piaggio, 8월 20, 2025에 액세스, https://www.centropiaggio.unipi.it/sites/default/files/grasp-IJRR95.pdf
34. The Synthesis of Force-Closure Grasps - MIT, 8월 20, 2025에 액세스, [ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-861.pdf](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-861.pdf)
35. 12.2.3. Force Closure – Modern Robotics, 8월 20, 2025에 액세스, https://modernrobotics.northwestern.edu/nu-gm-book-resource/12-2-3-force-closure/
36. 38. Grasping - Computer Science & Engineering, 8월 20, 2025에 액세스, https://www.cse.lehigh.edu/~trink/Courses/RoboticsII/reading/Grasping-Chapter38ofSpringerHanbookOfRobotics_ed2.pdf
37. Grasp Planning - How to Choose a Suitable Task Wrench Space, 8월 20, 2025에 액세스, https://www.dlr.de/de/rm/downloads/roboter_und_systeme/hand/icra2004grasp.pdf/@@download/file/icra2004grasp.pdf
38. Grasping - WPI, 8월 20, 2025에 액세스, https://users.wpi.edu/~zli11/teaching/rbe550_2017/slides/10-Grasping.pdf
39. Task-Oriented Dexterous Hand Pose Synthesis Using Differentiable Grasp Wrench Boundary Estimator - He Wang, 8월 20, 2025에 액세스, https://pku-epic.github.io/TaskDexGrasp/
40. web.stanford.edu, 8월 20, 2025에 액세스, http://web.stanford.edu/class/cs237b/pdfs/lecture/cs237b_lecture_7.pdf
41. A Fast Algorithm for Grasp Quality Evaluation Using the Object Wrench Space - UC Merced robotics, 8월 20, 2025에 액세스, https://robotics.ucmerced.edu/sites/g/files/ufvvjh1576/f/page/documents/pqhforqows.pdf
42. (PDF) Sim-to-Real Transfer in Robotics: Addressing the Gap between Simulation and Real- World Performance - ResearchGate, 8월 20, 2025에 액세스, https://www.researchgate.net/publication/390101654_Sim-to-Real_Transfer_in_Robotics_Addressing_the_Gap_between_Simulation_and_Real-_World_Performance
43. Rethinking the Gap between Simulation and Real World in Grasp Detection - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2410.06521v1
44. Grasp Stability Prediction with Sim-to-Real Transfer from Tactile Sensing - Publish, 8월 20, 2025에 액세스, https://publish.illinois.edu/robotouch/grasp-stability-prediction-with-sim-to-real-transfer-from-tactile-sensing/
45. Research Highlight: Sim-to-Real Transfer for Tactile-Based Robot Grasping, 8월 20, 2025에 액세스, https://www.cs.cmu.edu/news/2022/sim-to-real-transfer
46. [R] Sim2Real – Using Simulation to Train Real-Life Grasping Robots : r/MachineLearning, 8월 20, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/ab0hiv/r_sim2real_using_simulation_to_train_reallife/
47. Real–Sim–Real Transfer for Real-World Robot Control Policy Learning with Deep Reinforcement Learning - MDPI, 8월 20, 2025에 액세스, https://www.mdpi.com/2076-3417/10/5/1555
48. DexVLG: Dexterous Vision-Language-Grasp Model at Scale | alphaXiv, 8월 20, 2025에 액세스, https://www.alphaxiv.org/overview/2507.02747v1
49. DexGrasp Anything: Towards Universal Robotic Dexterous Grasping with Physics Awareness - CVF Open Access, 8월 20, 2025에 액세스, http://openaccess.thecvf.com/content/CVPR2025/papers/Zhong_DexGrasp_Anything_Towards_Universal_Robotic_Dexterous_Grasping_with_Physics_Awareness_CVPR_2025_paper.pdf
50. GRAB, 8월 20, 2025에 액세스, https://grab.is.tue.mpg.de/
51. DexGrasp Anything: Towards Universal Robotic Dexterous Grasping with Physics Awareness - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2503.08257v1
52. FastGrasp: Efficient Grasp Synthesis with Diffusion - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2411.14786v1
53. What Is NVIDIA's Three-Computer Solution for Robotics?, 8월 20, 2025에 액세스, https://blogs.nvidia.com/blog/three-computers-robotics/
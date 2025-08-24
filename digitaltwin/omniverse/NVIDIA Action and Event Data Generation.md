[NVidia Omniverse](./index.md)
# NVIDIA Action and Event Data Generation


인공지능(AI)의 발전은 물리 세계와 상호작용하는 자율 시스템, 즉 '물리 AI(Physical AI)'의 시대를 열고 있다. 이러한 물리 AI 시스템이 주변 환경을 정확하게 인지(perceive), 이해(understand), 그리고 상호작용(interact)하기 위해서는 정적인 이미지나 텍스트 데이터를 넘어선 새로운 차원의 데이터가 필수적이다.1 바로 'Action(행동)'과 'Event(사건)'으로 대표되는 동적 데이터이다. 여기서 'Action'은 로봇 팔의 정교한 움직임, 자율주행차의 조향 및 가감속과 같은 시스템의 구체적인 행위를 의미하며, 'Event'는 객체 간의 충돌, 보행자의 갑작스러운 도로 진입, 예상치 못한 장비 고장과 같은 시나리오 기반의 복합적인 사건을 포함한다. 이러한 동적 데이터는 물리 세계의 인과 관계와 시간적 연속성을 담고 있어, 자율 시스템이 복잡하고 예측 불가능한 현실에 대응하는 능력을 학습하는 데 결정적인 역할을 한다.1

그러나 이러한 필수적인 동적 데이터를 실제 세계에서 수집하는 과정은 여러 근본적인 한계에 직면한다. 첫째, 대규모의 실제 데이터를 수집하고, 각 프레임마다 정교한 주석(annotation)을 다는 레이블링 작업은 막대한 시간과 비용을 수반하는 노동 집약적 과정이다.3 둘째, 의료, 금융, 자율주행 등 민감한 정보를 다루는 분야에서는 GDPR(유럽 일반 개인정보 보호법), HIPAA(미국 건강 보험 양도 및 책임에 관한 법)와 같이 강화되는 개인정보보호 규제로 인해 데이터의 수집과 활용이 심각하게 제약받는다.3 셋째, 모델의 강건성(robustness) 확보에 필수적인 희귀 사례, 즉 '엣지 케이스(edge case)' 데이터는 현실에서 확보하기가 극히 어렵거나 사실상 불가능하다. 교통사고, 산업 현장의 위험 상황, 악천후 조건에서의 주행 등은 의도적으로 재현하기 어렵고 위험하기 때문이다.1 마지막으로, 수집된 데이터는 특정 지역, 인구 통계, 환경 조건에 편향될 수 있으며, 이는 AI 모델의 공정성과 일반화 성능을 저해하는 심각한 문제로 이어진다.3

이러한 실제 데이터의 한계를 극복하기 위한 대안으로 '합성 데이터 생성(Synthetic Data Generation, SDG)' 기술이 부상하고 있다. 합성 데이터는 실제 데이터를 대체하거나 보완할 목적으로 컴퓨터 시뮬레이션이나 알고리즘을 통해 인공적으로 생성된 정보를 의미하며, 생성 과정에서 자동으로 완벽한 레이블이 부여된다는 결정적인 장점을 가진다.6 이는 AI 시대의 '새로운 석유'로 비유되는 데이터를 더 이상 희소한 자원으로 여기지 않고, 필요에 따라 저렴하고 효과적으로 생산할 수 있는 '연료'로 전환시키는 패러다임의 변화를 의미한다.6

이러한 변화의 중심에 NVIDIA가 있다. NVIDIA는 단순히 데이터를 생성하는 것을 넘어, 물리적으로 정확한 법칙이 지배하는 가상 세계, 즉 '디지털 트윈(Digital Twin)'을 구축하고 그 안에서 무한에 가까운 고품질 Action 및 Event 데이터를 생성함으로써 물리 AI의 개발을 근본적으로 가속화하려는 비전을 제시한다. 이 비전의 핵심에는 협업 및 시뮬레이션 플랫폼인 Omniverse가 자리 잡고 있다.6

전통적인 AI 개발 패러다임이 현실 세계에서 데이터를 '수집'하고 정제하는 데 막대한 노력을 기울였다면, NVIDIA가 주도하는 새로운 패러다임은 가상 세계에서 필요한 데이터를 '설계'하고 '생성'하는 데이터 엔지니어링으로의 전환을 의미한다. 이러한 접근 방식에서 데이터는 더 이상 개발의 제약 조건(constraint)이 아니라, AI 모델의 특정 요구사항에 맞춰 능동적으로 제어하고 생성할 수 있는 설계 변수(design parameter)가 된다. 이는 AI 개발의 병목 현상을 데이터 확보의 어려움에서 시뮬레이션 환경의 정교함과 생성 알고리즘의 성능으로 이동시키며, 미래 AI 기술의 경쟁력이 가상 세계 구축 및 데이터 생성 능력에 의해 좌우될 것임을 시사한다. 본 보고서는 NVIDIA의 Action 및 Event 데이터 생성 기술 스택을 심층적으로 고찰함으로써, 시뮬레이션과 생성형 AI의 융합이 어떻게 물리 AI 시대를 현실로 만들고 있는지 체계적으로 분석하고자 한다.


NVIDIA의 합성 데이터 생성(SDG) 전략은 단순히 그럴듯한 이미지를 만들어내는 것을 넘어, 물리 법칙이 지배하는 가상 세계를 창조하고 그 안에서 현실과 구분하기 어려운 데이터를 생성하는 데 초점을 맞춘다. 이를 가능하게 하는 기술적 기반은 개방형 3D 생태계, 정교한 물리 엔진, 그리고 사실적인 렌더링 기술의 유기적인 결합에 있다.


NVIDIA SDG의 근간을 이루는 것은 Omniverse 플랫폼이다. Omniverse는 물리적으로 정확한 디지털 트윈을 구축하고 시뮬레이션하기 위한 협업 플랫폼으로, 현실 세계의 법칙을 가상 공간에 그대로 이식하는 것을 목표로 한다.13


Omniverse의 핵심은 Pixar가 개발하고 오픈소스로 공개한 OpenUSD 프레임워크다.1 OpenUSD는 복잡한 3D 장면을 구성하고, 다양한 3D 저작 도구 간에 데이터를 비파괴적으로 교환하며 협업할 수 있도록 설계된 표준이다. 이는 특정 소프트웨어에 종속되지 않는 개방형 생태계를 구축하는 기반이 된다. 예를 들어, Blender, Autodesk Maya, 3ds Max 등 업계 표준 툴에서 제작된 3D 에셋들이 Omniverse Connector를 통해 원활하게 통합될 수 있다.16 이러한 상호운용성은 방대한 3D 에셋 라이브러리를 구축하고, 다양한 분야의 전문가들이 실시간으로 협업하여 복잡한 가상 환경을 구축하는 것을 가능하게 한다. 이 생태계의 데이터베이스 서버이자 협업 엔진 역할을 하는 것이 Omniverse Nucleus로, 사용자들이 에셋, 데모, 프로젝트 데이터를 저장하고 공유하며 실시간으로 동기화할 수 있도록 지원한다.18


Omniverse는 NVIDIA의 고성능 물리 엔진인 PhysX를 통합하여 가상 세계 내 객체들의 움직임과 상호작용을 물리 법칙에 따라 정확하게 시뮬레이션한다.2 PhysX는 강체(rigid body)의 충돌, 연체(soft body)의 변형, 관절의 마찰 및 구동력, 유체 역학 등 광범위한 물리 현상을 실시간으로 계산할 수 있다.2 이는 'Action'과 'Event' 데이터의 동역학적 사실성(Dynamical Fidelity)을 보장하는 핵심 요소다. 예를 들어, 로봇 팔이 물체를 집을 때의 무게 중심 이동, 자율주행차가 특정 노면에서 제동할 때의 마찰력 변화, 충돌 시의 파편 움직임 등이 물리적으로 타당하게 계산된다. 이러한 동역학적 정확성은 특히 소프트웨어 인 루프(Software-in-the-Loop, SIL) 테스트에서 매우 중요하다. SIL은 실제 하드웨어 없이 제어 알고리즘을 시뮬레이션 환경에서 검증하는 단계로, 시뮬레이션된 센서 입력과 액추에이터의 반응이 현실과 일치해야만 유의미한 테스트가 가능하기 때문이다.21


물리적으로 정확한 동역학 시뮬레이션이 '어떻게 움직이는가'를 결정한다면, 물리 기반 렌더링(Physically Based Rendering, PBR)은 '어떻게 보이는가'를 결정한다. PBR은 빛이 재질의 표면과 상호작용하는 방식을 물리 법칙에 근거하여 모델링함으로써, 사진과 같은 극사실적인 이미지를 생성하는 렌더링 기법이다.22 이는 AI 모델, 특히 컴퓨터 비전 모델이 시뮬레이션 환경을 실제 환경으로 인식하게 만들어 'Sim-to-Real' 격차를 줄이는 데 결정적인 역할을 한다.


PBR의 이론적 핵심에는 렌더링 방정식(Rendering Equation)의 특수한 형태인 반사율 방정식(Reflectance Equation)이 있다.22 이 방정식은 특정 표면의 한 지점에서 특정 방향으로 반사되어 나가는 빛의 양(복사휘도, Radiance)을 계산하는 물리 모델이다. 방정식은 다음과 같이 표현된다.
$$
L_o(p, \omega_o) = \int_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p, \omega_i) (n \cdot \omega_i) d\omega_i
$$
이 방정식의 각 항은 다음과 같은 물리적 의미를 가진다 25:

- $L_o(p, \omega_o)$: 지점 $p$에서 나가는 방향 $\omega_o$로의 반사광 복사휘도(Outgoing Radiance). 이는 최종적으로 뷰어(카메라)에 도달하여 픽셀의 색상과 밝기를 결정한다.
- $\int_{\Omega}$: 지점 $p$를 중심으로 하는 반구(hemisphere) $\Omega$ 내의 모든 입사 방향에 대한 적분을 의미한다. 이는 직접 광원뿐만 아니라 주변 객체에서 반사된 간접광까지 모두 고려함을 뜻한다.
- $f_r(p, \omega_i, \omega_o)$: 양방향 반사 분포 함수(Bidirectional Reflectance Distribution Function, BRDF). 들어오는 빛의 방향($\omega_i$)과 나가는 빛의 방향($\omega_o$)에 따라 빛이 얼마나 반사되는지를 결정하는 함수로, 재질의 고유한 특성(알베도, 거칠기, 금속성 등)을 정의한다.
- $L_i(p, \omega_i)$: 지점 $p$로 들어오는 방향 $\omega_i$에서의 입사광 복사휘도(Incoming Radiance)이다.
- $(n \cdot \omega_i)$: 입사광 벡터 $\omega_i$와 표면의 법선 벡터 $n$의 내적. 이는 램버트의 코사인 법칙(Lambert's Cosine Law)에 따라 입사각이 커질수록 표면에 도달하는 빛의 양이 줄어드는 효과를 모델링한다.

결론적으로 이 방정식은 특정 지점에서 보이는 색상이 해당 지점 주변의 모든 방향에서 들어오는 빛이 재질의 특성과 상호작용하여 시선 방향으로 반사된 결과물의 총합임을 수학적으로 기술한 것이다.


반사율 방정식은 해석적으로 풀기 어려운 적분 형태이므로, 이를 근사하기 위한 계산이 필요하다. NVIDIA의 RTX 기술은 하드웨어 가속 레이 트레이싱(Ray Tracing)과 패스 트레이싱(Path Tracing)을 통해 이 방정식을 물리적으로 정확하게, 그리고 실시간에 가깝게 계산하는 역할을 수행한다.4 레이 트레이싱은 광선(ray)을 카메라로부터 장면으로 역추적하여 빛의 경로를 시뮬레이션하고, 이를 통해 사실적인 그림자, 반사, 굴절 효과를 구현한다. 패스 트레이싱은 한 단계 더 나아가 각 광선이 표면에 닿을 때마다 여러 방향으로 흩어지는 간접광까지 몬테카를로 기법으로 샘플링하여 계산함으로써, 전역 조명(Global Illumination) 효과를 매우 정밀하게 시뮬레이션한다.29 RTX 기술은 이러한 복잡한 연산을 GPU의 전용 RT 코어를 통해 가속하여, SDG 과정에서 대량의 고품질 이미지를 효율적으로 생성할 수 있게 한다.30


물리적으로 정확한 시뮬레이션 환경이 갖춰지면, 그 안에서 다양한 데이터를 생성하는 방법론이 필요하다. NVIDIA는 전통적인 절차적 콘텐츠 생성(Procedural Content Generation, PCG) 방식과 최신 생성형 AI 기술을 융합하여 데이터 생성의 효율성과 다양성을 극대화한다.


PCG는 사전에 정의된 규칙과 알고리즘에 기반하여 데이터를 자동으로 생성하는 기술이다.31 게임 개발에서 맵이나 아이템을 자동으로 생성하는 데 오랫동안 사용되어 온 기법이다. 합성 데이터 생성의 맥락에서 PCG는 시뮬레이션 환경의 다양한 파라미터(예: 객체 위치, 조명 색상)를 규칙에 따라 변경하며 대량의 데이터를 생성하는 데 활용된다. 3부에서 자세히 다룰 도메인 랜덤화(Domain Randomization)는 이러한 PCG 기법의 대표적인 예시로 볼 수 있다.


생성형 AI는 PCG의 개념을 한 차원 더 발전시킨다. 규칙 기반의 생성을 넘어, 대규모 데이터의 근본적인 분포와 패턴을 학습하여 기존에 없던 새롭고 사실적인 콘텐츠를 스스로 생성해내는 능력을 갖추고 있다.33 NVIDIA의 SDG 파이프라인은 다양한 생성형 AI 모델을 적극적으로 활용한다. 예를 들어, 생성적 적대 신경망(GAN)이나 변이형 오토인코더(VAE)는 사실적인 이미지를 생성하는 데 사용될 수 있으며, 최근 각광받는 확산 모델(Diffusion Model)은 텍스트나 이미지 프롬프트를 기반으로 고품질의 시각 콘텐츠를 생성한다. 또한, 거대 언어 모델(LLM)은 텍스트 데이터 생성뿐만 아니라, 자연어 명령을 통해 시뮬레이션 시나리오 자체를 생성하고 제어하는 역할까지 수행한다.9 이러한 생성형 AI 기술들은 2부에서 소개될 데이터 생성 파이프라인의 핵심 구성 요소로 작동하며, 데이터 생성의 자동화와 지능화를 이끈다.

NVIDIA의 SDG 전략에서 '물리적 정확성'은 단일한 개념이 아니라, 두 가지 핵심 축으로 구성된 이중적 의미를 지닌다. 첫 번째 축은 PBR과 RTX 기술을 통해 달성되는 '시각적 사실성(Visual Fidelity)'이며, 이는 생성된 데이터가 AI 모델의 시각 센서(카메라)에 얼마나 현실적으로 보이는지를 결정한다. 두 번째 축은 PhysX 엔진을 통해 구현되는 '동역학적 사실성(Dynamical Fidelity)'으로, 이는 가상 세계의 객체들이 물리 법칙에 따라 얼마나 현실적으로 움직이고 상호작용하는지를 결정한다. 이 두 가지 사실성이 유기적으로 결합되어야만, AI 모델은 실제 세계와 유사한 인과 관계 속에서 'Action'과 'Event'를 학습할 수 있다. 예를 들어, 자율주행 AI가 젖은 아스팔트 노면의 시각적 특징(시각적 사실성)을 인식하고, 그에 따라 마찰 계수가 낮아져 제동 거리가 길어지는 물리 현상(동역학적 사실성)을 함께 경험하며 학습해야만 실제 비 오는 날의 주행에 대처할 수 있다. 이처럼 시각과 동역학을 아우르는 통합된 물리적 정확성은 단순한 3D 모델링이나 게임 엔진의 기능을 넘어서는 NVIDIA Omniverse 플랫폼의 핵심적인 차별점이다. 이는 '디지털 트윈'을 단순한 시각화 도구가 아닌, 실제 세계의 법칙이 작동하는 정밀한 가상 실험실로 기능하게 하며, Sim-to-Real 문제 해결의 가장 근본적인 전제 조건을 만족시킨다.


NVIDIA는 Omniverse라는 물리적으로 정확한 토대 위에, Action 및 Event 데이터를 체계적으로 생성하기 위한 정교한 파이프라인을 구축했다. 이 파이프라인은 맞춤형 데이터 생성을 위한 프레임워크인 Omniverse Replicator를 중심으로, 생성형 AI 기술을 통해 그 효율성과 확장성을 극대화하는 구조를 가진다.


Omniverse Replicator는 물리적으로 정확한 합성 데이터를 생성하기 위한 핵심 소프트웨어 개발 키트(SDK)이자 프레임워크다.13 이는 Isaac Sim과 같은 Omniverse 기반 애플리케이션에 확장 기능(extension) 형태로 통합되어, 사용자가 특정 목적에 맞는 데이터 생성 워크플로우를 손쉽게 구축할 수 있도록 지원한다.13 Replicator는 데이터 생성 과정을 자동화하고, 생성된 데이터에 완벽한 Ground Truth 레이블을 부여하는 역할을 수행한다.


Replicator 파이프라인은 주로 세 가지 핵심 구성 요소로 이루어진다.4

1. **트리거(Trigger)와 스케줄러(Scheduler):** 데이터 생성의 시점과 빈도를 제어한다. 예를 들어, 시뮬레이션의 매 프레임마다 데이터를 생성하도록 설정하거나, 특정 이벤트(예: 객체가 특정 영역에 진입)가 발생했을 때만 데이터를 생성하도록 트리거를 설정할 수 있다. 이는 필요한 데이터를 효율적으로 수집하는 데 중요한 역할을 한다.
2. **어노테이터(Annotator):** Replicator의 가장 강력한 기능 중 하나로, 렌더링 파이프라인에서 직접 Ground Truth 정보를 추출하여 다양한 형태의 주석을 자동으로 생성한다.4 이는 수동 레이블링 과정에서 발생하는 시간, 비용, 그리고 인간의 오류를 원천적으로 제거한다. 지원되는 주석의 종류는 매우 다양하며, 2D/3D 경계 상자(Bounding Box), 시맨틱 및 인스턴스 분할(Semantic/Instance Segmentation) 마스크, 깊이 맵(Depth Map), 표면 법선(Surface Normals) 등을 포함한다.1
3. **라이터(Writer):** 어노테이터가 생성한 이미지와 주석 데이터를 특정 딥러닝 프레임워크가 요구하는 형식으로 변환하여 저장한다.4 예를 들어, 객체 탐지 모델 훈련에 널리 사용되는 KITTI 데이터셋 형식으로 출력을 저장하는 KittiWriter가 기본적으로 제공되어, 생성된 데이터를 별도의 변환 과정 없이 즉시 훈련 파이프라인에 투입할 수 있다.39


Replicator는 모델의 아키텍처나 하이퍼파라미터를 변경하는 대신, 훈련 데이터의 품질과 다양성을 조작하여 모델 성능을 개선하는 '데이터 중심 AI(Data-centric AI)' 접근 방식을 실현하는 핵심 도구다.4 개발자는 모델이 특정 시나리오에서 실패하는 것을 발견하면, Replicator를 사용하여 해당 시나리오와 유사하지만 다양한 변형을 가진 데이터를 대량으로 생성하여 모델을 재훈련시킬 수 있다. 이처럼 시뮬레이션 세계를 학습 가능한 파라미터 세트로 변환함으로써, 빠르고 반복적인 데이터 중심의 AI 모델 개선이 가능해진다.40


NVIDIA는 Replicator가 제공하는 견고한 파이프라인 위에 최신 생성형 AI 기술을 접목하여 데이터 생성의 사실성, 다양성, 자동화 수준을 한 단계 더 끌어올렸다. 이는 각기 다른 강점을 가진 전문화된 생성형 AI 모델들이 협력적으로 작동하는 생태계를 통해 이루어진다.


NVIDIA Cosmos는 시뮬레이션과 현실 사이의 미세한 시각적 격차를 메우기 위해 설계된 월드 파운데이션 모델(World Foundation Model, WFM) 플랫폼이다.41 Cosmos는 Omniverse에서 생성된 물리 기반의 렌더링 결과물(Ground Truth 비디오, 세분화 마스크, 깊이 맵 등)과 텍스트 지시어를 입력으로 받아, 이를 기반으로 훨씬 더 사실적인(photorealistic) 이미지나 비디오를 생성한다.13 이 과정에서 조명, 날씨, 재질, 배경 등 다양한 시각적 요소를 자연스럽게 변형시켜 하나의 시뮬레이션 시나리오를 수백, 수천 개의 사실적인 변형 데이터셋으로 증강할 수 있다.42 특히 Cosmos Transfer 모델은 시공간적 제어 맵(spatiotemporal control maps)이라는 기술을 사용하여 원본 시뮬레이션의 객체 배치, 움직임, 장면 구성을 그대로 유지하면서 시각적 스타일만 변환하는 정교한 제어가 가능하다.41 이는 '3D-to-Real' 변환을 통해 시뮬레이션 데이터의 품질을 한 차원 높이는 핵심적인 역할을 수행한다.


거대 언어 모델(LLM)은 합성 데이터 생성 파이프라인에서 '자동화'와 '지능화'를 담당한다. NVIDIA의 Nemotron-4 340B 모델군은 특정 도메인(헬스케어, 금융 등)에 맞는 고품질의 합성 텍스트 데이터를 생성하여 LLM 훈련 데이터 부족 문제를 해결하는 데 직접적으로 사용된다.43 더 나아가, LLM은 자연어와 코드 사이를 변환하는 능력을 통해 데이터 생성 과정 자체를 혁신한다. 예를 들어, USD Code API는 "창고 바닥에 선반 3개를 놓고, 그 위에 상자 5개를 무작위로 쌓아줘"와 같은 자연어 프롬프트를 Omniverse Replicator에서 실행 가능한 USD Python 코드로 즉시 변환해준다.38 이는 3D 프로그래밍에 익숙하지 않은 개발자나 기획자도 손쉽게 복잡한 3D 시나리오를 생성하고 제어할 수 있게 하여, 데이터 생성의 접근성을 획기적으로 높인다.


시뮬레이션의 다양성을 확보하기 위해서는 장면에 배치될 다양한 3D 에셋과 배경 환경이 필요하다. NVIDIA Edify는 최신 확산 모델(Diffusion Model) 기술을 기반으로 텍스트나 이미지 프롬프트로부터 고품질 3D 에셋과 환경을 생성하는 역할을 한다.13 예를 들어, Edify 3D NIM은 "녹슨 파란색 드럼통"과 같은 텍스트 프롬프트로부터 즉시 3D 모델을 생성하여 장면에 배치할 수 있으며, Edify 360 NIM은 360도 고명암비(HDRI) 배경 이미지를 생성하여 시뮬레이션 환경에 사실적인 조명과 배경을 손쉽게 적용할 수 있게 한다.38 이는 시뮬레이션 환경을 구축하는 데 필요한 시간과 노력을 크게 줄여주며, 데이터셋의 시각적 다양성을 무한에 가깝게 확장하는 것을 가능하게 한다.


NVIDIA의 SDG 파이프라인에 통합된 다양한 생성형 AI 모델들의 역할을 명확히 구분하고, 이들이 어떻게 시너지를 내는지 체계적으로 보여주기 위해 다음 표와 같이 정리할 수 있다. 이 표를 통해 각 모델의 기술적 특성과 실제 활용 방식을 한눈에 파악할 수 있다.

| 모델 계열    | 핵심 기술                              | 주요 기능                                           | 파이프라인 내 역할                                           | 관련 Snippet |
| ------------ | -------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ------------ |
| **Cosmos**   | World Foundation Model (WFM)           | 시뮬레이션 데이터의 사실성 증강, 시나리오 변형 생성 | **데이터 증강(Augmentation):** Omniverse 렌더링 결과물을 입력받아 더 사실적인 '3D-to-Real' 데이터로 변환 | 13           |
| **Nemotron** | Large Language Model (LLM)             | 합성 텍스트 데이터 생성, 자연어-코드 변환           | **시나리오 자동화:** 자연어 프롬프트를 통해 Replicator의 장면 구성 및 랜덤화 스크립트(Python) 생성 | 38           |
| **Edify**    | Diffusion Model                        | 텍스트/이미지 기반 3D 에셋 및 360도 배경 생성       | **에셋 생성(Asset Creation):** 시뮬레이션 장면에 필요한 객체와 배경을 동적으로 생성하여 다양성 확보 | 13           |
| **GAN/VAE**  | 생성적 적대 신경망 / 변이형 오토인코더 | 이미지, 텍스트 등 다양한 데이터 생성                | **기반 기술:** 합성 데이터 생성의 고전적 접근법으로, 특정 데이터 유형 생성에 활용 | 9            |

NVIDIA의 SDG 파이프라인은 단일 기술이 아닌, 여러 전문화된 생성형 AI 모델들이 유기적으로 결합된 계층적 시스템으로 이해할 수 있다. 첫 번째 계층인 **Foundation** 단계에서는 Omniverse와 Replicator가 물리 법칙에 기반한 '기초 데이터'의 뼈대를 생성한다. 두 번째 계층인 **Augmentation** 단계에서는 Edify가 이 뼈대에 살을 붙이고(3D 에셋 추가), Cosmos가 피부를 입혀(사실성 증강) 시각적 완성도를 높인다. 마지막 계층인 **Automation** 단계에서는 Nemotron과 같은 LLM이 두뇌 역할을 하여 자연어 명령으로 이 모든 복잡한 과정을 지휘하고 자동화한다. 이러한 계층적 파이프라인 구조는 SDG의 미래가 하나의 만능 모델이 아닌, 각기 다른 강점을 가진 전문 모델들의 협력적 생태계(Ecosystem of Generative Models)가 될 것임을 명확히 보여준다. 개발자는 이 생태계 안에서 필요한 도구들을 레고 블록처럼 조합하여, 자신의 목적에 맞는 최적의 맞춤형 데이터 생성 파이프라인을 구축하게 될 것이다.


시뮬레이션에서 생성된 합성 데이터로 학습한 AI 모델을 실제 세계에 성공적으로 적용하기 위해서는 'Sim-to-Real' 격차를 해소하는 것이 가장 중요한 과제다. 이 격차는 시뮬레이션 환경과 실제 환경 사이의 시각적, 물리적 불일치에서 비롯된다. NVIDIA는 이 문제를 해결하기 위해 도메인 랜덤화(Domain Randomization)라는 핵심 알고리즘을 기반으로, 이를 점차 지능적인 방식으로 발전시키는 전략을 취하고 있다.


도메인 랜덤화는 Sim-to-Real 전이 학습의 가장 기본적이면서도 강력한 기법 중 하나다. 그 핵심 철학은 시뮬레이션을 현실과 똑같이 만드는 것을 포기하고, 대신 극도로 다양한 시뮬레이션 환경을 만들어 AI 모델이 특정 환경의 비본질적인 특성에 과적합(overfitting)되는 것을 방지하는 데 있다.46 즉, 시뮬레이션 환경의 변화무쌍함 속에서 모델이 강인해져, 결국 실제 환경을 그저 수많은 변형 중 하나로 인식하게 만드는 것이다.46


DR의 목표는 시뮬레이션 환경(Source Domain)의 다양한 파라미터를 체계적으로 무작위화(randomize)함으로써, 학습된 정책(policy)이 보지 못했던 실제 환경(Target Domain)에서도 높은 일반화 성능을 보이도록 하는 것이다.49 이는 모델이 특정 텍스처나 조명 조건에 의존하는 '지름길 학습(shortcut learning)'을 방지하고, 객체의 형태나 움직임과 같은 본질적인 특징에 집중하도록 유도한다.


DR에서 무작위화되는 파라미터는 크게 시각적 요소와 물리적 요소로 나눌 수 있다.

- **시각적(Visual) 파라미터:** AI 모델의 '눈'에 해당하는 카메라 센서 입력에 영향을 미치는 요소들이다. 여기에는 광원의 위치, 색상, 강도, 개수, 배경 이미지나 텍스처, 객체의 색상과 재질, 카메라의 위치, 방향, 화각(field of view) 등이 포함된다.49
- **물리적(Physical) 파라미터:** AI 모델의 '몸'에 해당하는 로봇이나 차량의 동역학에 영향을 미치는 요소들이다. 객체의 질량, 크기, 마찰 계수, 탄성 계수, 로봇 관절의 강성(stiffness) 및 댐핑(damping), PID 제어 게인, 센서 노이즈, 액추에이터의 지연 시간 등이 여기에 해당한다.52


DR은 수학적으로 환경 파라미터 벡터 $\xi$를 특정 확률 분포 $p(\xi)$에서 샘플링하는 과정으로 모델링할 수 있다. 강화 학습 에이전트의 정책 $\pi_{\theta}$는 이 파라미터 분포에 대한 기대 누적 보상 $J(\theta, \xi)$를 최대화하도록 학습된다.54
$$
\max_{\theta} \mathbb{E}_{\xi \sim p(\xi)}[J(\theta, \xi)]
$$
초기의 단순한 DR(Uniform DR)에서는 각 파라미터 $\xi_i$를 사전에 정의된 범위 $[\xi_{i, \text{low}}, \xi_{i, \text{high}}]$ 내에서 균등 분포(Uniform Distribution)를 따라 독립적으로 샘플링했다.53


단순한 DR은 모든 파라미터를 독립적으로 무작위화하기 때문에, 현실에서는 절대 발생하지 않을 비현실적인 조합(예: 도로 한가운데에 생성된 건물, 하늘에 떠 있는 자동차)을 만들어낼 수 있다. 이러한 비현실적인 데이터는 학습 효율을 떨어뜨리고 때로는 모델 성능에 부정적인 영향을 미칠 수 있다. 이러한 문제를 해결하기 위해 제안된 것이 구조적 도메인 랜덤화(Structured Domain Randomization, SDR)이다.51


SDR은 무작위화를 수행하되, 현실 세계가 가지는 구조적, 맥락적, 의미론적 제약 조건을 보존하는 방식이다.56 예를 들어, 자율주행 시나리오를 생성할 때, 자동차는 차선 내에 위치하고 보행자는 인도나 횡단보도 위에 위치하도록 제약을 가하는 것이다. 이를 통해 랜덤화의 범위를 '가능하고 의미 있는(plausible and meaningful)' 공간으로 한정하여 학습 데이터의 품질과 효율성을 높인다.


SDR은 일반적으로 계층적인 생성 과정을 따른다. NVIDIA의 연구에서 제안된 방식은 다음과 같다 51:

1. **전역 파라미터 랜덤화:** 도로의 곡률, 전체적인 조명 조건, 카메라의 기본 위치 등 장면의 전반적인 특성을 결정하는 전역 파라미터를 무작위로 선택한다.
2. **컨텍스트 스플라인 생성:** 선택된 전역 파라미터에 따라 차선, 인도, 건물 경계선 등 장면의 기본 구조를 형성하는 '컨텍스트 스플라인(context splines)'을 생성한다.
3. **객체 배치 및 랜덤화:** 생성된 컨텍스트 스플라인 위에 자동차, 보행자, 교통 표지판 등의 객체를 배치한다. 이때 각 객체의 위치, 방향, 색상, 텍스처 등은 현실적인 범위 내에서 다시 무작위화된다.

이러한 계층적 접근은 랜덤화의 다양성을 유지하면서도 생성된 장면의 현실성을 보장한다.


DR/SDR은 종종 도메인 적응(Domain Adaptation, DA)과 비교된다. 두 기법 모두 Sim-to-Real 격차를 해소하려는 목표를 공유하지만, 접근 방식에 근본적인 차이가 있다.

- **도메인 랜덤화 (DR):** 주로 시뮬레이션 데이터만을 사용하며, 실제 데이터가 거의 또는 전혀 필요하지 않다. 시뮬레이션 환경의 다양성을 극대화하여 모델 자체가 어떤 도메인에도 강인한 일반화 성능을 갖도록 훈련시키는 데 초점을 맞춘다.53
- **도메인 적응 (DA):** 소량이라도 실제 환경(타겟 도메인)의 데이터를 활용한다. 소스 도메인(시뮬레이션)과 타겟 도메인(현실) 간의 데이터 분포 차이를 줄이는 방향으로 모델을 학습시키거나, 생성적 적대 신경망(GAN) 등을 이용해 시뮬레이션 이미지를 실제 이미지 스타일로 변환하는 기법을 사용한다.58

요약하자면, DR은 모델이 특정 도메인에 '의존하지 않도록' 만드는 강건성(robustness) 전략이고, DA는 특정 타겟 도메인으로의 '전이(transfer)'를 돕는 적응(adaptation) 전략이다. 실제 응용에서는 두 기법이 상호 보완적으로 사용되어 Sim-to-Real 성능을 극대화하기도 한다.

도메인 랜덤화 기술의 발전 과정은 '맹목적(Blind)' 탐색에서 '지능적(Intelligent)' 탐색으로의 진화로 요약할 수 있다. 초기의 균등 분포 기반 DR은 가능한 모든 파라미터 공간을 무차별적으로 탐색하는 '맹목적' 단계에 해당한다.53 이는 Sim-to-Real 전이에 효과적이었지만, 비현실적인 데이터를 다수 생성하여 학습 효율성이 떨어지는 단점이 있었다. 이에 대한 개선으로 등장한 SDR은 도로 구조나 물리 법칙과 같은 현실의 제약 조건을 랜덤화 과정에 도입함으로써, 탐색 공간을 의미 있는 영역으로 한정하는 '구조적' 단계로 발전했다.51 최근의 연구 동향은 여기서 한 걸음 더 나아가, 실제 환경에서의 성능 피드백을 바탕으로 랜덤화 분포 자체를 동적으로 최적화하는 '지능적' 단계로 진입하고 있다. 예를 들어, 베이지안 최적화(Bayesian Optimization)를 사용하여 실제 로봇의 성공/실패 경험을 바탕으로 가장 유용한 시뮬레이션 파라미터 분포를 찾아내거나 55, 현재 정책의 성공 확률을 유지하는 제약 하에 시뮬레이션 동역학 분포의 엔트로피를 최대화하여 탐색과 안정성 사이의 균형을 맞추는 접근법이 연구되고 있다.60 이는 Sim-to-Real 문제를 '어떤 환경에서 학습해야 가장 효율적으로 배울 수 있는가?'라는 메타러닝(Meta-Learning) 문제로 재정의하는 것이다. 이러한 지능적 랜덤화는 데이터 생성 과정을 또 다른 최적화의 대상으로 삼음으로써, AI 개발의 자동화 수준을 한 단계 더 끌어올릴 잠재력을 지닌다.


NVIDIA의 Action 및 Event 데이터 생성 기술은 물리 AI가 적용되는 핵심 분야인 로보틱스와 자율주행에서 그 진가를 발휘한다. 각 분야에 특화된 시뮬레이션 플랫폼인 Isaac Sim과 DRIVE Sim은 물리적으로 정확한 디지털 트윈 환경을 제공하며, 이를 통해 생성된 방대한 양의 합성 데이터는 실제 로봇과 자율주행차의 지능을 고도화하는 데 결정적인 역할을 한다.


NVIDIA Isaac Sim은 Omniverse 플랫폼을 기반으로 구축된 로봇 시뮬레이션 애플리케이션으로, AI 기반 로봇의 개발, 테스트, 훈련을 위한 사실적인 가상 환경을 제공한다.2 Isaac Sim은 물리 엔진 PhysX를 통해 정확한 동역학을 시뮬레이션하고, Replicator를 통해 대규모 합성 데이터를 생성하는 핵심 역할을 수행한다.2 한편, Isaac Lab은 강화 학습(RL) 및 모방 학습(IL)과 같은 로봇 정책 학습에 특화된 경량 프레임워크로, 대규모 병렬 시뮬레이션을 통해 훈련 시간을 획기적으로 단축시킨다.2


로봇 조작은 물체를 정확하게 인식하고, 파지하며, 옮기는 일련의 복잡한 작업을 포함한다. Isaac Sim은 이러한 조작 작업을 위한 Action 및 Event 데이터를 생성하는 데 탁월한 환경을 제공한다.

- 사례: 창고 내 팔레트 잭 탐지 및 조작

  물류 창고의 자율 이동 로봇(AMR)이 흔히 사용되는 운반 장비인 팔레트 잭을 정확히 탐지하고 회피하거나 조작하는 능력은 매우 중요하다. 실제 창고 환경에서 다양한 종류의 팔레트 잭, 조명 조건, 가려짐(occlusion) 상황에 대한 데이터를 수집하는 것은 거의 불가능하다.40

- 방법론

  이 문제를 해결하기 위해 Isaac Sim 내에 가상 창고 환경을 구축하고, Replicator를 사용하여 데이터 생성 파이프라인을 구성한다. 이 파이프라인은 반복적인 훈련 과정을 거치며 점진적으로 정교화된다.40

  1. **1차 반복:** 팔레트 잭의 색상과 위치(pose), 카메라의 위치를 무작위화하여 기본적인 데이터셋을 생성한다. 이 데이터로 훈련된 초기 모델은 주변의 다른 객체들까지 팔레트 잭으로 오인하는 등 많은 오탐지(false positive)를 보인다.

  2. **2차 반복:** 1차 반복에 더해, 배경과 객체의 텍스처(texture) 및 주변광(ambient light)을 무작위화한다. 이를 통해 모델은 특정 색상이나 형태뿐만 아니라 다양한 질감과 조명 조건에서도 객체를 인식하는 능력을 학습하게 되며, 오탐지가 줄어드는 결과를 보인다.

  3. **3차 반복:** 데이터셋의 다양성을 극대화하기 위해 장면에 다른 창고 객체들(상자, 선반 등)을 '방해물(distractors)'로 추가하여 무작위로 배치한다. 이는 모델이 복잡하고 어수선한 실제 환경에서도 목표 객체를 강건하게 탐지하는 능력을 기르는 데 도움을 주며, 최종적으로 탐지 정확도를 크게 향상시킨다.40

     생성된 이미지와 경계 상자 주석은 KITTI 형식으로 저장되어, NVIDIA TAO Toolkit과 같은 모델 훈련 프레임워크에 바로 사용된다.4


로봇이 안정적으로 움직이고 보행하는 능력을 학습하는 것은 로보틱스의 근본적인 과제 중 하나다. Isaac Lab은 대규모 병렬 시뮬레이션을 통해 이러한 이동 정책을 효율적으로 학습시킨다.

- 사례: 4족 보행 로봇(Spot)의 보행 제어

  Boston Dynamics의 4족 보행 로봇 Spot이 평지에서 사용자의 조이스틱 입력에 따라 목표 선속도 및 각속도를 추종하며 걷도록 하는 정책을 학습하는 것을 목표로 한다.63

- 방법론

  Isaac Lab 환경에서 수천 개의 Spot 로봇 시뮬레이션 인스턴스를 동시에 실행한다 (본 사례에서는 4,096개). 각 인스턴스에서는 도메인 랜덤화 기법이 적용되어 로봇의 질량, 마찰 계수, 모터 강도 등 물리적 파라미터가 미세하게 변경된다. 로봇은 강화 학습 알고리즘(PPO)을 통해 목표 속도를 잘 추종할수록 높은 보상을 받는 방식으로 정책을 학습한다.

- 결과

  NVIDIA RTX 4090 GPU 환경에서 약 4시간의 시뮬레이션 훈련만으로 안정적인 보행 정책을 성공적으로 학습했다. 주목할 점은, 이렇게 시뮬레이션에서만 학습된 정책을 별도의 미세 조정 없이 실제 Spot 로봇에 배포했을 때(zero-shot transfer), 시뮬레이션과 거의 동일한 수준의 안정적인 보행 성능을 보였다는 것이다.63 이는 Isaac Sim의 정교한 물리 시뮬레이션과 도메인 랜덤화가 Sim-to-Real 격차를 효과적으로 해소했음을 입증하는 강력한 사례다.


산업 현장의 안전 모니터링, 협동 로봇, 인간-컴퓨터 상호작용 등에서 인간의 행동을 정확하게 인식하는 것은 매우 중요하다. 그러나 추락이나 충돌과 같은 희귀하고 위험한 행동 데이터는 수집이 어렵고, 개인정보보호 문제도 수반된다.8

- SynthDa 파이프라인

  NVIDIA는 이러한 문제를 해결하기 위해 SynthDa라는 모듈식 합성 데이터 증강 파이프라인을 제안했다.8 이 파이프라인은 소수의 실제 인간 동작 캡처 데이터(3D 골격 정보)를 기반으로 무한에 가까운 합성 비디오 데이터를 생성한다.

- **방법론**

  1. 실제 3D 골격 모션을 가상 아바타에 적용(retargeting)한다.
  2. 이 아바타를 조명, 배경, 카메라 각도가 무작위화된 다양한 가상 환경에 렌더링한다.
  3. 'Synthetic mix'(실제 모션과 AI 생성 모션의 보간)와 'Real mix'(두 실제 모션의 보간) 기법을 통해, 실제 기록되지는 않았지만 물리적으로 충분히 가능한 새로운 중간 행동들을 생성한다.8 예를 들어, '걷기'와 '뛰기' 모션을 보간하여 '빠르게 걷기'나 '천천히 뛰기'와 같은 다양한 변형을 만들어낸다.

- 의의

  SynthDa는 실제 데이터 수집의 한계를 넘어, 특히 희귀하지만 중요한 안전 관련 이벤트 데이터를 체계적으로 생성함으로써 AI 모델이 엣지 케이스에 더욱 강건하게 대응할 수 있도록 훈련시키는 새로운 가능성을 제시한다.8


시뮬레이션 기반 학습의 실효성은 결국 실제 환경에서의 성공률로 입증된다. 다음 표는 NVIDIA 및 관련 연구에서 보고된 구체적인 Sim-to-Real 전이 성공률을 요약한 것으로, 합성 데이터 기반 훈련의 강력한 잠재력을 보여준다.

| 과제(Task)             | 로봇/시스템    | 핵심 기술/알고리즘          | Sim-to-Real 성공률   | 관련 Snippet |
| ---------------------- | -------------- | --------------------------- | -------------------- | ------------ |
| **작은 구체 잡기**     | 로봇 팔        | 모방 학습, Eye-in-Hand 비전 | 90%                  | 64           |
| **4개 큐브 쌓기**      | 로봇 팔        | 강화 학습 (SPOT 프레임워크) | 100% (시뮬 13%→100%) | 65           |
| **4개 큐브 행 만들기** | 로봇 팔        | 강화 학습 (SPOT 프레임워크) | 100% (시뮬 13%→99%)  | 65           |
| **4족 보행**           | Spot 로봇      | 강화 학습, 도메인 랜덤화    | Zero-shot 전이 성공  | 63           |
| **다중 작업(PnP 등)**  | Franka 로봇 팔 | 모방 학습 (Co-Training)     | 평균 37.9% 성능 향상 | 66           |
| **다양한 조작 과제**   | 로봇 팔        | 모방 학습 (Re³Sim)          | Zero-shot 평균 58%   | 67           |


자율주행 기술의 안전성을 확보하기 위해서는 인간 운전자가 평생 경험하기 힘든 수억, 수십억 킬로미터의 주행 테스트가 필요하다. NVIDIA DRIVE Sim은 이러한 대규모 검증 및 확인(V&V)을 물리적 제약 없이 수행할 수 있도록 설계된 시뮬레이션 플랫폼이다.68 Omniverse를 기반으로 구축된 DRIVE Sim은 물리적으로 정확한 디지털 트윈 환경에서 사실적인 센서 데이터를 생성하고, 위험한 엣지 케이스 시나리오를 안전하게 반복 테스트하는 환경을 제공한다.30


DRIVE Sim은 단순한 도로 환경 렌더링을 넘어, 살아있는 교통 생태계를 시뮬레이션한다. 주변 차량, 보행자, 자전거 등 다양한 교통 행위자(agent)들이 각자의 행동 모델에 따라 동적으로 움직이며 복잡하고 예측 불가능한 상호작용을 만들어낸다.28 NVIDIA는 Foretellix와 같은 광범위한 에코시스템 파트너와의 협력을 통해, 사실적인 3D 도시 모델, 정밀한 도로 네트워크, 다양한 교통 시나리오 카탈로그를 DRIVE Sim에 통합하여 시뮬레이션의 충실도와 다양성을 지속적으로 확장하고 있다.28


자율주행차의 '눈' 역할을 하는 센서 데이터를 정확하게 시뮬레이션하는 것은 인식(perception) 모델 훈련의 성패를 좌우한다. DRIVE Sim은 NVIDIA RTX의 실시간 레이 트레이싱 기술을 활용하여 카메라, LiDAR, Radar 센서의 데이터를 물리 법칙에 기반하여 정밀하게 생성한다.28

- **카메라 시뮬레이션:** 렌즈 왜곡, 노출, 심도 등 광학적 특성을 모델링하고, 특히 일출이나 일몰 시 태양 빛이 카메라에 직접 비치는 역광(glare) 상황과 같은 인식의 엣지 케이스를 사실적으로 재현한다.28
- **LiDAR 및 Radar 시뮬레이션:** 레이저 빔이나 전파가 물체에 부딪혀 반사되어 돌아오는 물리적 과정을 시뮬레이션한다. 재질에 따른 반사율 차이, 다중 경로 반사, 도플러 효과 등을 고려하여 실제 센서와 매우 유사한 포인트 클라우드 또는 탐지 데이터를 생성한다.


자율주행 시스템의 안전성을 검증하는 가장 중요한 과정은 위험하고 드문 엣지 케이스에 대한 대응 능력을 테스트하는 것이다. DRIVE Sim은 이러한 시나리오를 안전하고 확장 가능한 방식으로 생성하는 데 특화되어 있다.

- **필요성:** 주차된 차량 뒤에서 갑자기 아이가 튀어나오는 상황이나, 적색 신호에 교차로로 돌진하는 차량과 같은 시나리오는 실제 도로에서 테스트하는 것이 불가능에 가깝다.71 시뮬레이션은 이러한 위험 상황을 수만, 수백만 번 반복하며 AI의 반응을 체계적으로 검증할 수 있는 유일한 방법이다.42
- **STRIVE 모델을 이용한 적대적 시나리오 생성:** NVIDIA 연구진이 개발한 STRIVE(STress-Test dRIVE)는 '현실적이면서도 위험한' 시나리오를 자동으로 생성하는 AI 모델이다.73 STRIVE는 실제 주행 데이터로 학습한 교통 흐름 모델(Conditional VAE)을 기반으로 한다. 특정 실제 주행 기록을 입력받은 뒤, 교통 흐름 모델의 잠재 공간(latent space) 내에서 주변 차량의 궤적을 최적화하여 자율주행차와의 충돌을 유발하도록 시나리오를 '재창조'한다. 이때, 생성된 궤적이 학습된 교통 흐름의 확률 분포에서 크게 벗어나지 않도록 규제(regularization)함으로써 시나리오의 현실성을 보장한다. 이 기술을 통해 개발자는 자율주행 시스템의 잠재적 취약점을 체계적으로 발견하고 보완할 수 있다.75
- **LLM을 활용한 시나리오 생성:** 최근에는 텍스트와 다이어그램으로 구성된 실제 교통사고 보고서를 다중모달 LLM(Multi-modal LLM)이 해석하여, 해당 사고 상황을 3D 시뮬레이션 시나리오로 자동 변환하는 'CrashAgent'와 같은 연구도 활발히 진행되고 있다.76 이는 방대한 양의 실제 사고 데이터를 체계적인 테스트 시나리오로 전환할 수 있는 새로운 가능성을 열어준다.


합성 데이터는 AI 개발의 여러 난제를 해결할 잠재력을 지니고 있지만, 그 유용성은 생성된 데이터의 '품질'에 의해 결정된다. 따라서 체계적인 품질 평가 프레임워크를 구축하고, 기술의 내재적 한계와 윤리적 문제를 명확히 인식하며, 미래 발전 방향을 조망하는 것이 매우 중요하다.


생성된 합성 데이터가 실제 데이터를 효과적으로 대체하거나 보완할 수 있는지 판단하기 위해서는 다차원적인 평가가 필요하다. 학계와 산업계에서는 일반적으로 세 가지 핵심 차원에서 품질을 평가한다.77

- **충실도 (Fidelity):** 이 차원은 합성 데이터가 원본 실제 데이터의 통계적 특성을 얼마나 잘 보존하고 있는지를 측정한다. 평가 지표로는 각 변수의 분포를 비교하는 히스토그램 유사도 점수, 변수 간의 선형적 관계를 비교하는 상관관계 점수, 그리고 비선형적 관계까지 포괄하는 상호 정보량 점수 등이 사용된다.77 충실도가 높은 데이터는 실제 데이터와 통계적으로 거의 구별할 수 없다.
- **유용성 (Utility):** 이 차원은 합성 데이터의 실용적 가치를 평가한다. 가장 직접적인 평가 방법은 '실제 데이터로 훈련하고 실제 데이터로 테스트한 모델(Train on Real, Test on Real)'의 성능과 '합성 데이터로 훈련하고 실제 데이터로 테스트한 모델(Train on Synthetic, Test on Real)'의 성능을 비교하는 것이다.77 만약 후자의 성능이 전자에 근접하거나 이를 능가한다면, 해당 합성 데이터는 높은 유용성을 가진다고 판단할 수 있다.
- **개인정보보호 (Privacy):** 이 차원은 합성 데이터로부터 원본 데이터에 포함된 개인의 민감한 정보가 유추될 위험이 없는지를 평가한다. 예를 들어, 합성 데이터셋의 특정 레코드가 원본 데이터셋의 특정 개인과 일대일로 대응될 확률을 측정하여 식별 위험도를 평가할 수 있다.79

NVIDIA는 이러한 일반적인 프레임워크에 더해, 자사의 생성 파이프라인에 특화된 품질 관리 메커니즘을 내장하고 있다.

- **Nemotron-4 Reward 모델:** LLM 훈련용 합성 텍스트 데이터를 생성할 때, Nemotron-4 340B-Reward 모델이 생성된 응답을 평가하는 '심판' 역할을 한다. 이 모델은 유용성(helpfulness), 정확성(correctness), 일관성(coherence), 복잡성(complexity), 장황함(verbosity)의 다섯 가지 기준에 따라 점수를 매기고, 일정 기준을 통과한 고품질 데이터만을 필터링하여 최종 훈련 데이터셋의 품질을 보장한다.1

- **임베딩 모델 기반 평가:** 검색 증강 생성(RAG) 시스템의 성능 평가를 위한 질의-응답(QA) 쌍을 합성할 때, 사전 훈련된 임베딩 모델을 '판단자(judge)'로 활용한다.81 생성된 질문과 원본 문서의 코사인 유사도를 계산하여, 너무 쉽거나(유사도 높음) 관련성이 없는(유사도 낮음) QA 쌍을 필터링함으로써 평가 데이터셋의 품질을 자동으로 제어한다.

  궁극적으로, 합성 데이터의 품질은 실제 데이터 기반의 검증을 통해 최종 확인된다. 아무리 정교하게 생성된 데이터라 할지라도, 이를 통해 훈련된 모델은 반드시 실제 세계의 데이터를 대상으로 한 벤치마크 테스트를 통과해야만 그 가치를 인정받을 수 있다.4


합성 데이터는 강력한 도구이지만, 만병통치약은 아니다. 기술의 효과적인 활용을 위해서는 내재된 한계와 잠재적인 윤리적 문제를 명확히 이해해야 한다.

- **편향 증폭 (Bias Amplification):** 합성 데이터 생성 모델은 주로 실제 데이터를 기반으로 학습된다. 만약 이 원본 데이터에 사회적, 인종적, 성별 편향이 내재되어 있다면, 생성 모델은 이 편향을 그대로 학습하고 생성 과정에서 오히려 증폭시킬 수 있다.7 이는 AI 모델의 공정성을 심각하게 훼손할 수 있는 문제로, 다양한 데이터 소스를 활용하고 편향 완화 기술을 적용하는 노력이 필요하다.9
- **모델 붕괴 (Model Collapse):** AI가 생성한 합성 데이터만을 사용하여 다음 세대의 AI 모델을 반복적으로 훈련시킬 경우, 생성 모델의 미세한 오류나 편향이 누적되고 증폭되어 결국 모델이 현실과 동떨어진 결과를 내놓으며 성능이 점차 저하되는 '모델 붕괴' 현상이 발생할 수 있다.36 이를 방지하기 위해서는 합성 데이터와 고품질의 실제 데이터를 주기적으로 혼합하여 모델을 '정화'하는 과정이 필수적이다.
- **현실과의 미묘한 차이 (Subtle Reality Gap):** 현재의 기술로는 시뮬레이션이 실제 세계의 모든 복잡성, 물리적 미묘함, 예측 불가능한 이상치(outlier)를 완벽하게 포착하는 것은 불가능하다.85 합성 데이터는 현실의 '통계적 속성'을 모방할 수는 있지만, 현실 그 자체는 아니다. 이 미세한 차이가 여전히 Sim-to-Real 전이의 실패 요인으로 작용할 수 있다.
- **윤리적 책임:** 딥페이크(Deepfake) 기술에서 볼 수 있듯이, 고도로 사실적인 합성 데이터는 가짜 뉴스, 명예훼손, 사기 등 악의적인 목적으로 사용될 수 있다.3 생성된 데이터가 현실과 혼동될 경우, 그로 인해 발생하는 문제의 책임 소재가 불분명해질 수 있다.7 따라서 기술 개발과 함께 생성된 콘텐츠의 출처를 명확히 하는 워터마킹 기술이나, 기술의 오용을 방지하기 위한 강력한 윤리적 가이드라인 및 법적 규제 마련이 시급하다.


이러한 한계에도 불구하고, 합성 데이터 생성 기술은 물리 AI의 미래를 şekillendirecek 핵심 동력으로 자리매김하고 있다.

- **월드 파운데이션 모델 (World Foundation Model, WFM)의 부상:** NVIDIA Cosmos와 같은 WFM은 특정 작업이 아닌, '세계' 자체를 이해하고 생성하는 것을 목표로 하는 차세대 파운데이션 모델이다.1 이 모델들은 텍스트, 이미지, 비디오뿐만 아니라 물리 법칙, 인간 행동 패턴 등 다양한 양식의 데이터를 통합적으로 학습하여, 상호작용이 가능한 동적인 가상 세계를 생성하고 시뮬레이션한다.1 WFM은 미래의 로봇과 자율주행차의 핵심 '인지 엔진' 역할을 수행하며, 최소한의 미세 조정만으로 다양한 하위 작업에 적용될 수 있는 범용성을 제공할 것이다.
- **완전 자동화된 데이터 생성 공장 (AI Factories):** 미래의 AI 개발은 특정 모델 훈련에 필요한 맞춤형 합성 데이터를 주문 즉시 대규모로 '생산'하는 'AI 데이터 공장(AI Factories)'의 형태를 띨 것이다.87 개발자는 필요한 데이터의 종류, 양, 특성을 정의하기만 하면, 고도로 자동화된 SDG 파이프라인이 최적의 데이터를 생성하여 제공하게 된다. 이는 AI 개발의 속도와 효율성을 현재와 비교할 수 없을 정도로 향상시킬 것이다.88
- **합성 데이터의 지배:** 글로벌 시장조사 기관 Gartner는 2030년이 되면 AI 모델 학습에 사용되는 데이터 중 합성 데이터의 비중이 실제 데이터를 넘어설 것이라고 예측했다.89 이는 합성 데이터가 더 이상 실제 데이터의 보조 수단이 아니라, AI 개발을 이끄는 핵심 데이터 소스로 자리매김할 것임을 의미한다.6

결론적으로, NVIDIA가 주도하는 합성 데이터 생성 기술은 데이터 부족, 개인정보보호, 비용이라는 AI 개발의 오랜 장벽을 허무는 강력한 해결책을 제시하고 있다. 그러나 동시에 편향 증폭, 모델 붕괴, 현실과의 미세한 괴리라는 새로운 기술적 과제를 제기하는 '양날의 검(double-edged sword)'과 같은 속성을 지닌다.91 미래 AI 생태계의 성패는 단순히 더 많은 데이터를 생성하는 능력을 넘어, '어떻게 하면 고품질의, 편향이 없으며, 신뢰할 수 있는 합성 데이터를 생성하고 검증할 것인가'라는 질문에 답하는 능력에 달려 있을 것이다. 생성 과정의 투명성을 확보하고, 합성 데이터와 실제 데이터를 상호 보완적으로 활용하며, 지속적인 품질 관리와 윤리적 성찰을 병행하는 것이 성공적인 물리 AI 시대를 여는 핵심 열쇠가 될 것이다.


1. Synthetic Data for AI & 3D Simulation Workflows | Use Case - NVIDIA, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/use-cases/synthetic-data/
2. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 21, 2025에 액세스, https://developer.nvidia.com/isaac/sim
3. 합성 데이터란? 정의, 활용 사례, 생성 방법, 8월 21, 2025에 액세스, https://kr.appen.com/blog/synthetic-data/
4. Omniverse Replicator를 통한 맞춤형 합성 데이터 생성 파이프라인 ..., 8월 21, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/build-custom-synthetic-data-generation-pipelines-with-omniverse-replicator/
5. 자율주행 AI 모델에서 합성데이터 활용해보기 - HMG Developers, 8월 21, 2025에 액세스, https://developers.hyundaimotorgroup.com/blog/620
6. 정확한 AI 모델을 만드는 '합성 데이터' 해부하기 | NVIDIA Blog, 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/what-is-synthetic-data/?nv_excludes=7057,7066
7. 합성 데이터(Synthetic Data), '보완'에서 '핵심 전략'으로, 8월 21, 2025에 액세스, https://www.jongwon.net/blog/synthetic-data
8. Improving Synthetic Data Augmentation and Human Action Recognition with SynthDa, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/improving-synthetic-data-augmentation-and-human-action-recognition-with-synthda/
9. What is Synthetic Data Generation (SDG)? | NVIDIA Glossary, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/glossary/synthetic-data-generation/
10. 엔비디아(NVIDIA)는 합성 데이터(Synthetic Data)를 생성하여 Ai 훈련 활용, 8월 21, 2025에 액세스, https://contents.premium.naver.com/engview/engtv/contents/250111222025415ih
11. 합성 데이터란? - NVIDIA Blog Korea, 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/what-is-synthetic-data-2/
12. 합성 데이터를 이용한 AI 가속화 가이드 - NVIDIA, 8월 21, 2025에 액세스, https://www.nvidia.com/ko-kr/deep-learning-ai/resources/accelerating-ai-with-synthetic-data-ebook/
13. How to Build a Generative AI-Enabled Synthetic Data Pipeline with OpenUSD - NVIDIA, 8월 21, 2025에 액세스, https://resources.nvidia.com/en-us-omniverse-usd/how-to-build-a-gener
14. 더 스마트한 로봇을 가속화하는 옴니버스 아이작 심에서 2배의 시뮬레이션 도약을 제공하는 AWS의 NVIDIA GPU, 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/gpu-aws-omniverse-isaac-sim-robots/
15. Generating Synthetic Data for Physical AI With NVIDIA Cosmos - YouTube, 8월 21, 2025에 액세스, https://www.youtube.com/watch?v=ZSxYgW-zHiU
16. NVIDIA and Unity Powering the Next Generation of Smart Cities and Industrial Experiences, 8월 21, 2025에 액세스, https://www.xrom.in/post/nvidia-and-unity-powering-the-next-generation-of-smart-cities-and-industrial-experiences
17. NVIDIA Omniverse: Quickly Creating 3D Digital Twins - Neurom, 8월 21, 2025에 액세스, https://neurom.in/nvidia-omniverse-quickly-creating-3d-digital-twins/
18. [NVIDIA Omniverse] Isaac Sim 설치 - 삼SAM - 티스토리, 8월 21, 2025에 액세스, [https://challenge-sam.tistory.com/entry/NVIDIA-Omniverse-Isaac-Sim-%EC%84%A4%EC%B9%98](https://challenge-sam.tistory.com/entry/NVIDIA-Omniverse-Isaac-Sim-설치)
19. Exploring the Intersection of NVIDIA Omniverse, Digital Twins, and the Industrial Metaverse Through Advanced 3D Modeling Technologies - Dell, 8월 21, 2025에 액세스, https://www.delltechnologies.com/asset/en-us/products/workstations/industry-market/exploring-the-intersection-of-nvidia-omniverse-digital-twins-and-3d-modeling.pdf
20. What Is Isaac Sim? - NVIDIA Omniverse, 8월 21, 2025에 액세스, https://docs.omniverse.nvidia.com/isaacsim
21. AI 및 3D 시뮬레이션 워크플로우를 위한 합성 데이터 | 활용 사례 - NVIDIA, 8월 21, 2025에 액세스, https://www.nvidia.com/ko-kr/use-cases/synthetic-data/
22. [번역강좌] 1. 이론 - 물리 기반 렌더링(Physically Based Rendering), 8월 21, 2025에 액세스, https://mawile.tistory.com/354
23. What Is PBR (Physically Based Rendering)? A complete guide - Chaos, 8월 21, 2025에 액세스, https://www.chaos.com/blog/what-is-pbr-physically-based-rendering-a-complete-guide
24. Theory - LearnOpenGL, 8월 21, 2025에 액세스, https://learnopengl.com/PBR/Theory
25. Physically Based Rendering - Wolfram Language Documentation, 8월 21, 2025에 액세스, https://reference.wolfram.com/language/tutorial/PhysicallyBasedRendering.html
26. EasyPBR: A Lightweight Physically-Based Renderer - Autonomous Intelligent Systems, 8월 21, 2025에 액세스, https://www.ais.uni-bonn.de/papers/GRAPP_2021_Rosu_EasyPBR.pdf
27. Physically Based Rendering - Coding Labs, 8월 21, 2025에 액세스, http://www.codinglabs.net/article_physically_based_rendering.aspx
28. NVIDIA DRIVE Sim 자율주행차 테스트 시뮬레이션 해부 | NVIDIA Blog, 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/nvidia-sim-ecosystem-diverse-proving-ground-self-driving-vehicles/?nv_excludes=6169,6172
29. Procedural Modeling and Physically Based Rendering for Synthetic Data Generation in Automotive Applications - ResearchGate, 8월 21, 2025에 액세스, https://www.researchgate.net/publication/320465033_Procedural_Modeling_and_Physically_Based_Rendering_for_Synthetic_Data_Generation_in_Automotive_Applications
30. NVIDIA DRIVE Sim Ecosystem Creates Diverse Proving Ground for Self-Driving Vehicles, 8월 21, 2025에 액세스, https://blogs.nvidia.com/blog/drive-sim-ecosystem-diverse-proving-ground-self-driving-vehicles/
31. 절차적 생성 - 나무위키, 8월 21, 2025에 액세스, [https://namu.wiki/w/%EC%A0%88%EC%B0%A8%EC%A0%81%20%EC%83%9D%EC%84%B1](https://namu.wiki/w/절차적 생성)
32. [논문]절차적 생성 알고리즘을 이용한 3차원 게임월드 제작, 8월 21, 2025에 액세스, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO201816842429394
33. AI 생성 콘텐츠란 무엇인가요? - IBM, 8월 21, 2025에 액세스, https://www.ibm.com/kr-ko/think/insights/ai-generated-content
34. 생성형 AI 정의 및 작동 방식 – Adobe Firefly, 8월 21, 2025에 액세스, https://www.adobe.com/kr/products/firefly/discover/how-generative-ai-work.html
35. 생성형 AI란? | 예시, 사용 사례 - SAP, 8월 21, 2025에 액세스, https://www.sap.com/korea/products/artificial-intelligence/what-is-generative-ai.html
36. 합성 데이터란 무엇인가요? - IBM, 8월 21, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/synthetic-data
37. [태그:] synthetic data - NVIDIA Technical Blog, 8월 21, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/tag/synthetic-data/
38. How to Build a Generative AI-Enabled Synthetic Data Pipeline for Perception-Based Physical AI | NVIDIA Technical Blog, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/how-to-build-a-generative-ai-enabled-synthetic-data-pipeline-for-perception-ai/
39. 합성 데이터를 사용하여 창고 팔레트 잭을 감지하도록 자율 모바일 로봇을 훈련하는 방법, 8월 21, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/how-to-train-autonomous-mobile-robots-to-detect-warehouse-pallet-jacks-using-synthetic-data/
40. How to Train Autonomous Mobile Robots to Detect Warehouse Pallet Jacks Using Synthetic Data | NVIDIA Technical Blog, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/how-to-train-autonomous-mobile-robots-to-detect-warehouse-pallet-jacks-using-synthetic-data/
41. Scale Synthetic Data and Physical AI Reasoning with NVIDIA Cosmos World Foundation Models, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/scale-synthetic-data-and-physical-ai-reasoning-with-nvidia-cosmos-world-foundation-models/
42. Autonomous Vehicle Simulation | Use Cases - NVIDIA, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/use-cases/autonomous-vehicle-simulation/
43. Nemotron-4 340B - Nvidia의 합성 데이터 생성 파이프라인 - 틸노트, 8월 21, 2025에 액세스, https://tilnote.io/pages/66700edfd5089413b2607573
44. NVIDIA, 거대 언어 모델 훈련용 개방형 합성 데이터 생성 파이프라인 ..., 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/nemotron-4-synthetic-data-generation-llm-training/
45. Synthetic Data Generation with Generative AI Reference Workflow ..., 8월 21, 2025에 액세스, https://docs.omniverse.nvidia.com/genai-nim-digitaltwin.html
46. [1703.06907] Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/abs/1703.06907
47. Domain Randomization - Steven Gong, 8월 21, 2025에 액세스, https://stevengong.co/notes/Domain-Randomization
48. Episode 14: Domain Randomization: A Key Technique for Sim2Real Transfer, 8월 21, 2025에 액세스, https://bettercallshao.com/episode-14-domain-randomization-a-key-technique-for-sim2real-transfer/
49. Creating Synthetic Data with Nvidia Omniverse Replicator - Edge Impulse Documentation, 8월 21, 2025에 액세스, https://docs.edgeimpulse.com/experts/computer-vision-projects/nvidia-omniverse-replicator
50. Randomizer Examples - Replicator - NVIDIA Omniverse, 8월 21, 2025에 액세스, https://docs.omniverse.nvidia.com/extensions/latest/ext_replicator/randomizer_details.html
51. Structured Domain Randomization Makes Deep Learning More Accessible | NVIDIA Technical Blog, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/structured-domain-randomization-makes-deep-learning-more-accessible/
52. [isaacsim.replicator.domain_randomization] Isaac Sim Replicator Domain Randomization, 8월 21, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/py/source/extensions/isaacsim.replicator.domain_randomization/docs/index.html
53. Domain Randomization for Sim2Real Transfer | Lil'Log, 8월 21, 2025에 액세스, https://lilianweng.github.io/posts/2019-05-05-domain-randomization/
54. Neural Posterior Domain Randomization - Proceedings of Machine Learning Research, 8월 21, 2025에 액세스, https://proceedings.mlr.press/v164/muratore22a/muratore22a.pdf
55. Data-efficient Domain Randomization with Bayesian Optimization - Honda Research Institute Europe, 8월 21, 2025에 액세스, https://www.honda-ri.de/pubs/pdf/4615.pdf
56. Structured Domain Randomization - Emergent Mind, 8월 21, 2025에 액세스, https://www.emergentmind.com/topics/structured-domain-randomization
57. What is the difference between domain randomization and data augmentation?, 8월 21, 2025에 액세스, https://datascience.stackexchange.com/questions/75972/what-is-the-difference-between-domain-randomization-and-data-augmentation
58. SIM2Real Transfer - andrew.cmu.ed, 8월 21, 2025에 액세스, https://www.andrew.cmu.edu/course/10-403/slides/S19sim2real.pdf
59. Data Augmentation vs. Domain Adaptation—A Case Study in Human Activity Recognition, 8월 21, 2025에 액세스, https://www.mdpi.com/2227-7080/8/4/55
60. Domain Randomization via Entropy Maximization - OpenReview, 8월 21, 2025에 액세스, https://openreview.net/forum?id=GXtmuiVrOM
61. Isaac Sim - NVIDIA Technical Blog, 8월 21, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/recent-posts/?products=Isaac+Sim
62. Fast-Track Robot Learning in Simulation Using NVIDIA Isaac Lab | NVIDIA Technical Blog, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/fast-track-robot-learning-in-simulation-using-nvidia-isaac-lab/
63. Closing the Sim-to-Real Gap: Training Spot Quadruped Locomotion with NVIDIA Isaac Lab, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/closing-the-sim-to-real-gap-training-spot-quadruped-locomotion-with-nvidia-isaac-lab/
64. Sim-to-Real Transfer of Accurate Grasping with Eye-In-Hand Observations and Continuous Control - Research at NVIDIA, 8월 21, 2025에 액세스, [https://research.nvidia.com/sites/default/files/pubs/2017-12_Sim-to-Real-Transfer-of/Sim-to-Real%20Transfer%20of%20Accurate%20Grasping%20with%20Eye-In-Hand%20Observations%20and%20Continuous%20Control.pdf](https://research.nvidia.com/sites/default/files/pubs/2017-12_Sim-to-Real-Transfer-of/Sim-to-Real Transfer of Accurate Grasping with Eye-In-Hand Observations and Continuous Control.pdf)
65. “Good Robot!”: Efficient Reinforcement Learning for Multi-Step Visual Tasks with Sim to Real Transfer - Research at NVIDIA, 8월 21, 2025에 액세스, https://research.nvidia.com/publication/2020-10_good-robot-efficient-reinforcement-learning-multi-step-visual-tasks-sim-real
66. Sim-and-Real Co-Training: A Simple Recipe for Vision-Based Robotic Manipulation, 8월 21, 2025에 액세스, https://co-training.github.io/
67. Re3Sim: Generating High-Fidelity Simulation Data via 3D-Photorealistic Real-to-Sim for Robotic Manipulation - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2502.08645v3
68. 셉톤, DRIVE Sim™에 라이다 모델 추가, 8월 21, 2025에 액세스, https://www.autoelectronics.co.kr/article/articleView.asp?idx=4830
69. Scaling Autonomous Vehicle Simulation with NVIDIA DRIVE Sim and Omniverse SE50003 | GTC Digital Spring 2023 | NVIDIA On-Demand, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/on-demand/session/gtcspring23-se50003/
70. 자율주행 자동차 개발 위한 NVIDIA Drive Sim의 새로운 AI 도구, 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/drive-sim-omniverse-neural-ai-digital-twin/
71. GPU 활용 도로시스템 시뮬레이션으로 자율주행 안전성을 강화한 엔비디아 - NVIDIA 블로그, 8월 21, 2025에 액세스, https://blogs.nvidia.co.kr/blog/self-driving-simulation/
72. Accelerate Autonomous Vehicle AI Training and Development - NVIDIA, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/solutions/autonomous-vehicles/ai-training/
73. STRIVE - Research at NVIDIA, 8월 21, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/STRIVE/
74. Generating Useful Accident-Prone Driving Scenarios via a Learned Traffic Prior - Research at NVIDIA, 8월 21, 2025에 액세스, https://research.nvidia.com/labs/toronto-ai/STRIVE/docs/strive.pdf
75. Generating Potential Accident Scenarios for Autonomous Vehicles Using AI - NVIDIA DRIVE Labs Ep. 27 - YouTube, 8월 21, 2025에 액세스, https://www.youtube.com/watch?v=BXf9xTR6hJo
76. CrashAgent: Crash Scenario Generation via Multi-modal Reasoning - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2505.18341v1
77. How to evaluate the quality of the synthetic data – measuring from the perspective of fidelity, utility, and privacy | Artificial Intelligence - AWS, 8월 21, 2025에 액세스, https://aws.amazon.com/blogs/machine-learning/how-to-evaluate-the-quality-of-the-synthetic-data-measuring-from-the-perspective-of-fidelity-utility-and-privacy/
78. A Multi-Faceted Evaluation Framework for Assessing Synthetic Data Generated by Large Language Models - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2404.14445v1
79. 합성 데이터 개념 (개인정보보호위원회, 인하대, 서울대) - LeafHT, 8월 21, 2025에 액세스, https://leafht.tistory.com/142
80. Nvidia, LLM 훈련을 위한 합성 데이터 생성 파이프라인 공개 - GeekNews, 8월 21, 2025에 액세스, https://news.hada.io/topic?id=15376
81. Evaluating and Enhancing RAG Pipeline Performance Using Synthetic Data, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/evaluating-and-enhancing-rag-pipeline-performance-using-synthetic-data/
82. AI의 합성 데이터: 이점, 사용 사례, 과제 및 응용 프로그램 - Shaip, 8월 21, 2025에 액세스, https://ko.shaip.com/blog/synthetic-data-and-ai/
83. How Generative AI Is Revolutionizing Training Data with Synthetic Datasets - Dataversity, 8월 21, 2025에 액세스, https://www.dataversity.net/how-generative-ai-is-revolutionizing-training-data-with-synthetic-datasets/
84. Nvidia Reportedly Bought a Synthetic Data Firm. So What's Synthetic Data? - CNET, 8월 21, 2025에 액세스, https://www.cnet.com/tech/services-and-software/nvidia-reportedly-bought-a-synthetic-data-firm-so-whats-synthetic-data/
85. 컴퓨터 비전에서 합성 데이터란 무엇인가요? | 울트라 애널리틱스 - Ultralytics, 8월 21, 2025에 액세스, https://www.ultralytics.com/ko/blog/what-is-synthetic-data-in-computer-vision-an-overview
86. 합성 데이터란 무엇인가요? - AWS, 8월 21, 2025에 액세스, https://aws.amazon.com/ko/what-is/synthetic-data/
87. Generative AI Solutions | Smarter AI Runs on NVIDIA, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/solutions/ai/generative-ai/
88. 엔닷라이트, 엔비디아 'GTC 2025'서 3D 합성 데이터 생성 솔루션 공개 - 와우테일, 8월 21, 2025에 액세스, https://wowtale.net/2025/03/19/238570/
89. “2030년, 합성데이터가 실제 데이터 사용량 추월” 키워드로 살펴본 AI 시장 전망 - AI 매터스, 8월 21, 2025에 액세스, https://aimatters.co.kr/news-report/ai-report/23901/
90. 생성형 AI 시대, 데이터 부족 해결사 '합성 데이터'가 뜬다 [트랜 D] - 중앙일보, 8월 21, 2025에 액세스, https://www.joongang.co.kr/article/25318236
91. 합성 데이터: AI의 미래를 위한 양날의 검 - Unite.AI, 8월 21, 2025에 액세스, https://www.unite.ai/ko/synthetic-data-a-double-edged-sword-for-the-future-of-ai/


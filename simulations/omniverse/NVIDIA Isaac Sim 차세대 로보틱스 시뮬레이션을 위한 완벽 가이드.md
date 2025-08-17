# NVIDIA Isaac Sim: 차세대 로보틱스 시뮬레이션을 위한 완벽 가이드

## 1.  "왜" 시뮬레이션인가: 진보된 로보틱스 시뮬레이션의 필요성

### 서론: 물리 AI의 여명

인공지능(AI)의 발전은 단순히 디지털 정보를 처리하는 것을 넘어, 이제 물리적 세계를 인식하고, 추론하며, 상호작용하는 새로운 시대로 접어들고 있다. 이를 '물리 AI(Physical AI)'라 칭하며, 언어 모델과 같은 순수 디지털 AI와는 근본적으로 구별된다.1 물리 AI는 자율주행차, 물류 로봇, 휴머노이드 등 우리의 삶과 산업을 혁신할 잠재력을 지닌 기술의 다음 개척지다.

하지만 이러한 물리 AI 시스템, 즉 로봇을 개발하고 훈련시키는 과정은 막대한 도전 과제를 안고 있다. 실제 로봇 프로토타입을 제작하는 데는 상당한 비용이 소요되며, 개발 과정에서 하드웨어가 손상될 위험도 크다.3 또한, 실제 환경에서 데이터를 수집하는 것은 매우 느리고 비효율적이며, 특히 드물지만 치명적인 '롱테일(long-tail)' 시나리오(예: 갑작스러운 장애물 출현, 조명 변화)를 모두 포괄하기는 거의 불가능하다.6

### 1.1 "시뮬레이션 우선(Sim-First)" 개발 패러다임

이러한 물리 AI 시대의 도전 과제에 대한 가장 강력한 해답은 바로 '시뮬레이션 우선(Sim-First)' 접근 방식이다.2 이는 로봇 개발의 패러다임을 근본적으로 바꾸는 개념으로, 실제 하드웨어를 제작하기 전에 가상의 환경에서 로봇의 설계, 테스트, 훈련을 대부분 완료하는 것을 의미한다. 이 패러다임이 제공하는 핵심적인 이점은 다음과 같다.

- **개발 가속화 및 비용 절감:** 시뮬레이션을 통해 개발자들은 수많은 물리적 프로토타입 없이도 가상 환경에서 빠르게 설계를 반복하고 테스트할 수 있다. 이는 개발 비용과 시장 출시 기간을 획기적으로 단축시킨다.3 또한, 가상 테스트는 고가의 하드웨어가 손상될 위험을 원천적으로 차단하여 안정적인 개발 환경을 보장한다.5
- **안전성 강화:** 잠재적으로 위험한 시나리오(예: 인간 작업자와의 충돌, 장비 오작동)를 물리적 위험 없이 안전하게 테스트할 수 있다.3 이는 특히 예측 불가능한 환경에서 인간과 로봇이 협력해야 하는 애플리케이션의 안전성을 검증하는 데 필수적이다.
- **확장 가능한 테스트 및 검증:** 시뮬레이션은 현실 세계에서는 불가능한 규모의 테스트를 가능하게 한다. 단일 로봇의 동작부터 공장이나 창고 전체를 복제한 '디지털 트윈' 내에서 수백, 수천 대의 로봇 플릿(fleet)이 상호작용하는 복잡한 시나리오까지 시뮬레이션하고 검증할 수 있다.1
- **AI 모델 부트스트래핑:** AI 모델 훈련의 가장 큰 병목 현상 중 하나인 데이터 부족 문제를 해결한다. 실제 데이터를 구하기 어렵거나 제한적인 경우, 시뮬레이션을 통해 물리적으로 정확하고 완벽하게 레이블링된 대규모 합성 데이터를 생성하여 AI 모델 훈련의 초기 단계를 지원(bootstrap)할 수 있다.1

이러한 패러다임의 전환은 단순히 기술적인 변화를 넘어, 로보틱스 엔지니어링의 문화를 바꾸고 있다. 전통적인 기계 공학의 긴 설계-제작-테스트 주기는 소프트웨어 개발의 지속적 통합/지속적 배포(CI/CD)와 유사한 형태로 진화하고 있다. 로봇의 소프트웨어를 가상 환경에서 지속적으로 테스트하고 검증하는 '소프트웨어 인 더 루프(Software-in-the-Loop, SIL)' 방식은 이제 표준이 되고 있다.1 이는 로보틱스 개발이 '로보옵스(RoboOps)'라는 새로운 DevOps 문화로 나아가고 있음을 시사하며, 기계, 전자, 소프트웨어 엔지니어링이 더욱 통합되고 민첩한 방식으로 협업해야 함을 의미한다. 클라우드 기반 시뮬레이션과 플릿 관리 기능은 이러한 새로운 개발 문화를 직접적으로 지원하는 핵심 요소다.7

### 1.2 NVIDIA Isaac Sim 소개: 물리 AI를 위한 플랫폼

NVIDIA Isaac Sim은 바로 이 '시뮬레이션 우선' 패러다임을 실현하기 위해 탄생한 포괄적인 플랫폼이다.1 Isaac Sim은 단순한 '시뮬레이터'를 넘어, NVIDIA의 핵심 기술을 집약한 '레퍼런스 애플리케이션'이자 완전한 스택의 개발 환경이다.1 이는 개발자들이 지능형 로봇을 위한 AI를 개발, 테스트, 훈련하고 실제 환경에 배포하는 전 과정을 가속화하도록 설계되었다.6

## 2.  아키텍처의 기반: Isaac Sim, Omniverse, 그리고 USD의 내부

### 2.1 핵심 철학: 모듈식, 확장성, 그리고 협업 플랫폼

Isaac Sim의 아키텍처는 유연성과 통합을 핵심 철학으로 삼는다. 이는 기존의 로봇 개발 도구들을 대체하는 것이 아니라, 이들과 협력하고 그 기능을 강화하는 것을 목표로 한다.15 Isaac Sim은 오픈 소스 구성 요소와 유연한 C++ 및 Python API를 제공하여, 개발자가 필요에 따라 전체 애플리케이션으로 사용하거나 특정 모듈만을 자신의 맞춤형 파이프라인에 통합할 수 있도록 지원한다.15

### 2.2 NVIDIA Omniverse: 산업을 위한 협업 메타버스

Isaac Sim을 이해하기 위해서는 그 기반이 되는 NVIDIA Omniverse를 먼저 이해해야 한다. Omniverse는 실시간 3D 개발 및 협업 플랫폼으로, Isaac Sim이 구동되는 기반층(foundational layer) 역할을 한다.1 이는 3D 툴과 가상 세계를 구축하고 연결하기 위한 '운영체제'와 같다.

- **Omniverse Nucleus:** Nucleus는 데이터베이스이자 협업 엔진으로, '라이브 싱크(live sync)' 기능을 통해 여러 다른 애플리케이션과 사용자 간의 실시간 동기화를 가능하게 한다.6 덕분에 전 세계에 흩어져 있는 개발팀이 마치 한 공간에 있는 것처럼 동일한 3D 씬(scene)을 동시에 작업할 수 있다. 이는 대규모 프로젝트의 협업 효율성을 극대화하는 핵심 기술이다.
- **Omniverse Kit:** Kit는 네이티브 Omniverse 애플리케이션과 확장 기능(extension)을 구축하기 위한 툴킷이다. Isaac Sim 자체가 바로 이 Kit를 사용해 만들어졌다.15 이러한 모듈식 구조 덕분에 Isaac Sim은 사용자의 필요에 맞게 확장되고 커스터마이징될 수 있다.
- **커넥터(Connectors):** Omniverse는 커넥터를 통해 Maya, Blender, CATIA와 같은 다양한 3D 콘텐츠 제작 도구들을 하나의 일관된 워크플로우로 연결한다. 이를 통해 각 분야의 전문가들이 자신이 가장 선호하는 도구를 사용하면서도 원활하게 협업할 수 있다.

### 2.3 Universal Scene Description (USD): 3D 세계의 공용어

Omniverse와 Isaac Sim의 협업 및 상호운용성의 핵심에는 픽사(Pixar)가 개발한 오픈 소스 3D 씬 기술인 Universal Scene Description (USD)이 있다.15 USD는 단순한 파일 포맷을 넘어, 복잡한 3D 씬을 구성하고 조립하기 위한 강력한 프레임워크다.

- **레이어링과 컴포지션(Layering and Composition):** USD의 가장 큰 특징은 비파괴적인(non-destructive) 레이어링 개념이다. 기본 모델, 재질, 애니메이션, 수정 사항 등 각기 다른 '의견(opinion)'을 담은 여러 레이어들을 조합하여 최종 씬을 구성한다.15 이는 여러 개발자가 각자의 작업을 독립적으로 수행하면서도 이를 손쉽게 통합할 수 있게 해주는 USD의 '비밀 병기'와 같다.
- **확장성과 상호운용성:** USD의 구조는 다양한 도구와 워크플로우를 연결하는 공통 언어 역할을 한다. 이를 통해 Omniverse와 같은 플랫폼의 이상적인 중추가 된다.15
- **로보틱스와의 관련성:** 로보틱스에서 USD는 단순히 로봇과 환경의 시각적 외형을 정의하는 데 그치지 않는다. 물리적 속성, 관절(joint), 센서 모델 등 로봇의 모든 정보를 포괄적으로 표현하는 디지털 트윈의 완벽한 청사진 역할을 한다.1 최근 발표된 Isaac Sim 5.0의 새로운 로봇 및 센서 스키마는 바로 이 USD를 기반으로 표준화되었다.18

### 2.4 NVIDIA PhysX 5: GPU 가속 물리로 현실을 시뮬레이션하다

Isaac Sim의 심장부에는 NVIDIA의 고성능 오픈 소스 물리 엔진인 PhysX 5가 자리 잡고 있다.1

- **핵심 기능:** PhysX 5는 로보틱스 시뮬레이션에 필수적인 다양한 기능을 제공한다. 강체(rigid body) 및 연체(soft body) 동역학, 차량 동역학, 복잡한 관절(미믹 조인트, 축소 좌표계 관절 등), 그리고 SDF(Signed Distance Field) 충돌체 등을 정밀하게 시뮬레이션한다.1 또한 파괴 및 골절(Blast), 유체 및 화염(Flow) 시뮬레이션과 같은 특수 기능도 지원하여 시뮬레이션의 범위를 넓힌다.22
- **GPU 가속의 이점:** 다른 많은 시뮬레이터와의 결정적인 차이점은 PhysX가 GPU에서 가속된다는 점이다.21 이는 수천 개의 시뮬레이션 환경을 동시에 병렬로 처리할 수 있게 해주며, 이는 특히 강화학습(Reinforcement Learning)에서 압도적인 성능 우위를 가져다준다. 이는 CPU 기반 시뮬레이터와는 비교할 수 없는 장점이다.24

### 2.5 NVIDIA RTX 렌더링: 사실적인 시뮬레이션의 열쇠

현대의 로봇은 카메라, 라이다(LiDAR)와 같은 시각 센서에 크게 의존한다.25 따라서 시뮬레이션된 센서 데이터와 실제 센서 데이터 간의 차이, 즉 '인식의 Sim-to-Real 갭'을 최소화하는 것이 매우 중요하다.

- **RTX 기술:** Isaac Sim은 NVIDIA의 RTX 실시간 레이 트레이싱(Ray Tracing) 기술을 활용하여 사진처럼 사실적인(photorealistic) 렌더링을 구현한다.12 이는 물리 기반 재질(PBR), 정확한 그림자, 반사, 굴절, 그리고 동적인 전역 조명(global illumination)을 포함한다.
- **고충실도 센서 시뮬레이션:** 이러한 사실적인 렌더링은 곧바로 고충실도의 센서 시뮬레이션으로 이어진다. RGB 카메라, 깊이 카메라, 라이다 등은 실제 세계와 거의 흡사한 데이터를 생성하며, 이는 강인한(robust) 인식 AI 모델을 훈련시키는 데 결정적인 역할을 한다.1

Isaac Sim의 아키텍처를 구성하는 이 네 가지 요소(Omniverse, USD, PhysX, RTX)는 독립적으로 작동하는 것이 아니라, 서로의 강점을 증폭시키는 긴밀하게 통합된 생태계를 형성한다. USD는 복잡한 씬을 정의할 수 있는 데이터 기반을 제공하고, Omniverse는 이 씬을 협업적으로 관리하고 운영하는 플랫폼 역할을 한다. RTX 렌더링은 USD의 상세한 씬 정보를 활용하여 사실적인 시각 및 센서 데이터를 생성하며, 이는 NVIDIA의 RTX GPU를 필요로 한다. 마지막으로 PhysX 5는 이 씬을 GPU 상에서 빠르고 정확하게 시뮬레이션하여, 특히 대규모 병렬 훈련에서 압도적인 성능을 보여준다. 이 모든 과정이 NVIDIA의 하드웨어 위에서 최적으로 작동하도록 설계되었다. 결국, USD가 복잡한 디지털 트윈을 가능하게 하고, Omniverse가 이를 관리하며, RTX가 사실적으로 렌더링하고, PhysX가 GPU에서 빠르게 시뮬레이션하는 이 인과 관계의 사슬은 강력한 기술적 선순환 구조를 만들어낸다. 이는 Isaac Sim의 우수한 기능을 온전히 활용하기 위해 사용자가 자연스럽게 NVIDIA의 고성능 GPU를 선택하게 만드는, 소프트웨어가 하드웨어 판매를 견인하는 고도의 풀스택 전략의 핵심이다.

## 3.  핵심 역량 및 워크플로우: 데이터부터 배포까지

Isaac Sim은 로봇 개발의 전 과정을 지원하기 위해 네 가지 핵심 워크플로우를 제공한다. 이들은 개별적인 기능이 아니라, 서로 유기적으로 연결되어 개발의 시너지를 창출한다.

### 3.1 워크플로우 1: Isaac Replicator를 이용한 합성 데이터 생성(SDG)

AI 개발의 가장 큰 난관 중 하나는 '데이터 병목 현상'이다. 강인한 인식 모델을 훈련시키기 위해서는 방대하고 다양하며 정확하게 레이블링된 데이터셋이 필요하지만, 이를 실제 환경에서 수집하는 것은 엄청난 비용과 시간이 소요된다.6

Isaac Sim의 합성 데이터 생성(SDG) 엔진인 **Isaac Replicator**는 이 문제에 대한 강력한 해결책을 제시한다.6 개발자는 Replicator를 통해 프로그래밍 방식으로 물리적으로 정확한 합성 데이터셋을 생성할 수 있다. 이 데이터셋에는 경계 상자(bounding box), 세분화 마스크(segmentation mask), 깊이 맵, 자세 추정(pose estimation) 등 완벽한 정답(ground-truth) 레이블이 자동으로 포함된다.6

이 과정의 핵심 기술은 **도메인 무작위화(Domain Randomization)**이다. Replicator는 조명, 텍스처, 객체의 위치, 색상, 카메라 각도 등 씬의 다양한 파라미터를 체계적으로 무작위화하여 매우 다양한 데이터셋을 생성한다.1 이는 AI 모델이 훈련 데이터의 특정 시각적 맥락을 암기하는 대신, 객체의 본질적인 특징(예: 팔레트 잭의 '형태')을 학습하도록 강제한다. 그 결과, 실제 환경의 다양한 변화에 훨씬 더 강인한 성능을 보이는 모델이 탄생한다.17

`SceneBlox`와 같은 최신 기능을 사용하면 창고와 같은 산업 환경 전체를 절차적으로 생성하여 더욱 현실적인 데이터를 만들 수 있다.6

### 3.2 워크플로우 2: Isaac Lab으로 로봇의 두뇌 훈련하기

로봇에게 보행이나 조작과 같이 복잡하고 동적인 작업을 가르치는 것은 전통적인 프로그래밍 방식으로는 매우 어렵다. 강화학습(Reinforcement Learning, RL)은 이에 대한 강력한 대안을 제공한다.28

NVIDIA의 강화학습 툴은 Isaac Gym에서 **Isaac Lab**으로 진화했다. Isaac Gym이 GPU 가속 강화학습의 가능성을 처음으로 증명했다면 6, Isaac Lab은 이를 계승하여 Isaac Sim 위에 구축된 더욱 통합적이고 모듈화된 오픈 소스 프레임워크다.1

Isaac Lab의 핵심은 **대규모 병렬 훈련(Massively Parallel Training)**이다. GPU로 가속되는 PhysX 엔진을 활용하여 단일 GPU에서 수천 개의 시뮬레이션 환경을 동시에 실행한다.24 이를 통해 RL 에이전트는 극히 짧은 시간 안에 방대한 양의 경험 데이터를 수집할 수 있으며, 이는 훈련 시간을 수 주에서 수 분 또는 수 시간 단위로 극적으로 단축시킨다.6 이는 CPU 코어 수에 제약을 받는 CPU 기반 강화학습과는 근본적인 차이를 보인다.24 Cart-Pole과 같은 고전적인 제어 문제부터 다양한 지형에서의 4족 보행, 정교한 손 조작에 이르기까지 다양한 복잡한 작업들을 Isaac Lab을 통해 성공적으로 훈련시킬 수 있다.28

### 3.3 워크플로우 3: 가상과 현실의 연결 (Sim-to-Real)

로봇 시뮬레이션의 근본적인 과제는 **'Sim-to-Real 갭'**이다. 이는 시뮬레이션과 실제 세계 간의 미세한 차이로 인해, 시뮬레이션에서 완벽하게 작동하던 정책(policy)이 실제 로봇에서는 실패하는 현상을 말한다.34

Isaac Sim은 이 간극을 메우기 위해 다각적인 전략을 사용한다.

- **고충실도 물리:** PhysX 5를 사용하여 현실적인 동역학을 모델링한다. 특히 개선된 액추에이터 모델과 관절 마찰 모델은 로봇 제어의 정확성을 높이는 데 결정적이다.18
- **사실적인 센싱:** RTX 렌더링을 통해 고충실도의 카메라 및 라이다 데이터를 생성하여, 로봇이 가상 세계에서 보는 것과 실제 세계에서 보는 것의 차이를 최소화한다.12
- **도메인 무작위화:** 합성 데이터 생성뿐만 아니라, Sim-to-Real을 위한 핵심 기술이기도 하다. 시뮬레이션 환경의 시각적, 물리적 파라미터를 무작위화함으로써, AI 모델이 시뮬레이션의 특정 조건에 과적합(overfitting)되는 것을 방지하고 실제 환경의 예측 불가능성에 더 잘 대응하도록 만든다.17
- **정책 배포:** Isaac Lab에서 훈련된 정책은 ONNX와 같은 표준 형식으로 변환되어 실제 로봇의 임베디드 시스템에서 효율적으로 추론(inference)될 수 있다.35

### 3.4 워크플로우 4: 로보틱스 생태계와의 완벽한 통합 (ROS & ROS2)

로봇 운영체제(Robot Operating System, ROS)는 로보틱스 커뮤니티의 사실상 표준 프레임워크다. 따라서 모든 주요 시뮬레이션 도구는 ROS와의 원활한 통합을 지원해야 한다.

Isaac Sim은 ROS 및 ROS2를 위한 강력한 **브리지(Bridge)**를 제공한다.1 이를 통해 개발자들은 

**소프트웨어 인 더 루프(Software-in-the-Loop, SIL)** 테스트를 수행할 수 있다. 즉, 개발자는 기존의 ROS 기반 로봇 제어 스택을 자신의 컴퓨터에서 실행하면서, 마치 실제 하드웨어에 연결된 것처럼 Isaac Sim 내의 가상 로봇과 통신하게 할 수 있다.1 이는 실제 로봇에 코드를 배포하기 전에 버그를 잡고, 알고리즘을 검증하며, 전체 시스템의 안정성을 테스트하는 데 매우 귀중한 과정이다. NVIDIA는 ROS2 Humble 및 Jazzy와 같은 최신 버전을 지원하고, 표준화된 ROS2 시뮬레이션 인터페이스를 채택함으로써 ROS 커뮤니티에 대한 강력한 지원 의지를 보여주고 있다.12

이 네 가지 핵심 워크플로우는 로봇 개발의 선순환 구조를 형성한다. 개발자는 먼저 ROS 브리지를 사용하여 기존 내비게이션 스택을 가상 로봇에 연결해 초기 SIL 테스트를 수행한다. 그 다음, 로봇의 인식 성능을 높이기 위해 Isaac Replicator(SDG)를 사용하여 특정 객체(예: 팔레트)에 대한 방대한 합성 데이터셋을 생성하고, 이를 통해 강력한 객체 탐지 모델을 훈련시킨다. 복잡한 조작 작업을 위해서는 Isaac Lab(RL)을 통해 대규모 병렬 시뮬레이션으로 제어 정책을 훈련시킨다. 이 모든 과정에서 고충실도 물리 및 렌더링, 도메인 무작위화와 같은 Sim-to-Real 기술이 적용되어, 훈련된 인식 모델과 제어 정책이 실제 하드웨어에서도 잘 작동하도록 보장한다. 결국 ROS 통합은 전체 시스템의 검증을, SDG는 시스템의 '눈'을, RL은 시스템의 '손과 발'을 강화하며, Sim-to-Real 기술은 이 모든 가상에서의 노력이 현실에서 결실을 맺도록 보장한다. 이 통합된 파이프라인이야말로 Isaac Sim이 제공하는 궁극적인 가치 제안이다.

## 4.  경쟁 환경: 비교 분석

로보틱스 시뮬레이터를 선택하는 것은 프로젝트의 성공에 큰 영향을 미치는 중요한 결정이다. 모든 요구사항을 만족시키는 단 하나의 '최고의' 시뮬레이터는 존재하지 않으며, 프로젝트의 목표, 예산, 기술적 요구사항에 따라 최적의 선택이 달라진다.5 이 장에서는 Isaac Sim을 다른 주요 시뮬레이터들과 비교하여 사용자가 정보에 입각한 결정을 내릴 수 있도록 돕는다.

### 4.1 Isaac Sim vs. Gazebo

Gazebo는 ROS 커뮤니티에서 오랫동안 사용되어 온 오픈 소스 표준 시뮬레이터로, 그래픽보다는 물리 시뮬레이션의 정확성에 중점을 둔다.5 반면 Isaac Sim은 사진 같은 사실적인 그래픽과 GPU 가속을 무기로 등장한 현대적인 도전자다.25

- **하드웨어 요구사항:** Gazebo는 인텔 i5 CPU와 일반적인 GPU만으로도 구동될 만큼 요구사항이 낮다. 하지만 Isaac Sim은 고사양의 CPU(인텔 i7/라이젠 7 이상), 32GB 이상의 RAM, 그리고 결정적으로 최신 NVIDIA RTX GPU를 필요로 하는 매우 높은 사양을 요구한다.40
- **물리 시뮬레이션:** Gazebo는 ODE, Bullet, DART 등 여러 물리 엔진을 선택적으로 사용할 수 있는 반면, Isaac Sim은 NVIDIA의 PhysX 5만을 사용한다.40 과거에는 PhysX가 일부 로봇 작업에서 다른 엔진보다 정확도가 떨어진다는 평가도 있었으나, NVIDIA는 로보틱스 분야를 위해 지속적으로 엔진을 개선해왔다.40 다만, Gazebo는 드론(UAV)이나 수중 로봇(UUV) 시뮬레이션에 필요한 공기역학 및 유체역학을 더 잘 지원하는 반면, Isaac Sim은 이 분야에서 아직 대안이 되지 못한다.40
- **그래픽 및 센서 시뮬레이션:** 이 부분에서 Isaac Sim은 압도적인 우위를 보인다. RTX 레이 트레이싱 기술을 통한 사실적인 렌더링은 Gazebo의 기본적인 렌더링과 비교할 수 없을 정도로 높은 품질의 시각적 결과물을 만들어낸다.25 이는 곧바로 카메라와 라이다 센서 데이터의 품질로 이어지며, 인식 기반 AI 모델을 개발하는 데 결정적인 차이를 만든다.
- **생태계와 철학:** Gazebo는 커뮤니티가 주도하는 완전한 오픈 소스 프로젝트다. Isaac Sim은 일부 구성 요소가 오픈 소스화되었지만, 근본적으로는 NVIDIA의 하드웨어 및 소프트웨어 생태계에 깊숙이 통합된 기업 주도형 제품이다.42

이러한 비교를 통해 볼 때, NVIDIA는 Gazebo를 시장에서 몰아내려는 것이 아니라, Gazebo가 제공하지 못하는 새로운 고부가가치 시장을 창출하려는 전략을 구사하고 있음을 알 수 있다. 실제로 NVIDIA는 Gazebo 커넥터를 제공하여 두 시뮬레이터 간의 자산 이동을 지원하는데 6, 이는 경쟁자를 제거하려는 기업의 행동이라기보다는 기존 생태계와 통합하려는 시도로 해석된다. 이 전략은 "기본적인 알고리즘 프로토타이핑이나 UAV 시뮬레이션은 Gazebo를 사용하고, 사진처럼 사실적인 데이터로 인식 AI를 훈련시키거나 대규모 병렬 처리로 강화학습 정책을 훈련시켜야 할 때는 Isaac Sim으로 '졸업'하라"는 메시지를 전달한다. 이는 결과적으로 Gazebo를 로보틱스 커뮤니티의 보편적인 진입점으로 남겨두면서, Isaac Sim을 최첨단 AI 개발과 대규모 산업용 애플리케이션을 위한 프리미엄 도구로 포지셔닝하는 계층화된 생태계를 구축하는 것이다.

#### 4.1.1 표 1: Isaac Sim vs. Gazebo 기능 비교 매트릭스

| 기능                  | NVIDIA Isaac Sim                                          | Gazebo                                                 |
| --------------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| **주요 초점**         | 사진처럼 사실적인 렌더링, GPU 가속, AI 훈련               | 물리 시뮬레이션의 정확성, ROS 생태계 통합              |
| **하드웨어 요구사항** | 매우 높음 (고성능 CPU, 32GB+ RAM, NVIDIA RTX GPU 필수) 40 | 낮음 (i5 CPU, 일반 GPU) 40                             |
| **물리 엔진**         | NVIDIA PhysX 5 40                                         | ODE, Bullet, DART, Simbody (선택 가능) 41              |
| **그래픽 충실도**     | 매우 높음 (실시간 레이 트레이싱) 25                       | 제한적임 (기본 렌더링) 45                              |
| **센서 충실도**       | 매우 높음 (특히 카메라, 라이다) 25                        | 제한적임 45                                            |
| **ROS 통합**          | 강력한 ROS/ROS2 브리지 제공 1                             | 네이티브에 가까운 완벽한 통합 42                       |
| **비용 / 라이선스**   | 개인/연구용 무료, 상업용 라이선스 필요 가능성 45          | 완전 무료, 오픈 소스 (Apache 2.0) 42                   |
| **주요 사용자**       | AI 연구원, 대규모 산업 디지털 트윈 개발자                 | 일반 로보틱스 개발자, 학계, ROS 사용자                 |
| **강점**              | 합성 데이터 생성, 대규모 강화학습, 시각 기반 AI 훈련      | 접근성, 낮은 진입 장벽, 다양한 물리 엔진, UAV/UUV 지원 |

### 4.2 Isaac Sim vs. MuJoCo

이 비교는 강화학습(RL)에 초점을 맞춘다. MuJoCo는 RL 연구 커뮤니티에서 매우 존경받는 빠르고 정확한 물리 엔진이다.

두 시뮬레이터의 가장 큰 구조적 차이는 **GPU 대 CPU**이다. 전통적인 MuJoCo 기반의 강화학습 워크플로우는 물리 시뮬레이션을 CPU에서 실행한다. 이는 병렬로 실행할 수 있는 환경의 수가 CPU 코어 수(일반적으로 8~16개)에 의해 제한됨을 의미한다.24

반면, Isaac Sim(Isaac Lab을 통해)은 물리 시뮬레이션 자체를 **GPU에서 실행**한다. 이 덕분에 수천 개(2048, 4096개 이상)의 환경을 동시에 병렬로 처리할 수 있다.24 이는 훈련 시간을 획기적으로 단축시키는 Isaac Sim의 RL 분야에서의 '킬러 기능'이다.

### 4.3 Unity 및 Unreal Engine과의 비교

Unity(ML-Agents 포함)나 Unreal Engine과 같은 게임 엔진 역시 로보틱스 시뮬레이션에 사용된다.5 Isaac Sim은 이들과 차별화되는 명확한 초점을 가지고 있다. 게임 엔진이 뛰어난 그래픽을 갖춘 범용 플랫폼인 반면, Isaac Sim은 로보틱스와 산업용 디지털 트윈을 위해 특별히 제작된 플랫폼이다. 물리(PhysX), 데이터 표준(USD), 로보틱스 생태계(ROS)와의 깊은 통합은 Isaac Sim이 게임이 아닌, 물리적으로 정확한 디지털 트윈을 만드는 것을 핵심 임무로 삼고 있음을 보여준다.14

## 5.  Isaac Sim의 실제 활용: 실제 애플리케이션 및 사례 연구

이 장에서는 이론을 넘어, Isaac Sim이 오늘날 실제 세계의 문제들을 어떻게 해결하고 있는지 구체적인 사례를 통해 살펴본다. 이 사례들의 공통적인 핵심 가치는 바로 '디지털 트윈'이다. Isaac Sim은 단순히 로봇 하나를 시뮬레이션하는 것이 아니라, 로봇이 작동할 전체 운영 환경을 가상으로 복제하여 그 안에서 로봇을 테스트하고 최적화한다. 이는 Isaac Sim이 단순한 로보틱스 툴을 넘어, '인더스트리 4.0'과 산업 메타버스를 구현하는 핵심 동력임을 보여준다.

### 5.1 사례 연구: BMW 그룹의 스마트 팩토리

BMW 그룹은 고도로 맞춤화된 차량을 생산하기 위해 수백만 개의 부품을 관리해야 하는 엄청난 물류 복잡성에 직면해 있다.47 이 문제를 해결하기 위해 BMW는 NVIDIA Isaac 플랫폼을 도입하여 공장 물류 시스템을 혁신하고 있다.47

- **엔드-투-엔드 시스템:** BMW는 실제 공장에 로봇을 배치하기 전에, 공장의 완벽한 디지털 트윈을 Isaac Sim 내에 구축했다. 이 가상 공장에서 AI 기반 물류 로봇을 훈련하고 테스트하며, 전체 물류 흐름을 최적화한다.47
- **AI 로봇 개발:** Isaac 플랫폼 기반의 단일 소프트웨어 아키텍처 위에서 부품을 자율적으로 운송하는 내비게이션 로봇과 부품을 집어 분류하는 조작 로봇을 개발하고 있다.47
- **훈련 파이프라인:** AI 모델 훈련에는 NVIDIA DGX 시스템을, 합성 데이터 증강을 위한 부품 렌더링에는 Quadro GPU를, 그리고 시뮬레이션 기반 훈련 및 검증에는 Isaac Sim을 사용하는 완벽한 엔드-투-엔드 파이프라인을 구축했다.47 Omniverse를 기반으로 하기에 전 세계의 BMW 엔지니어들이 동일한 시뮬레이션 환경에서 협업할 수 있다.
- **최종 목표:** 이 프로젝트의 목표는 더욱 진보된 '적시생산(Just-in-Time)' 시스템을 구현하고, 생산 효율성을 극대화하며, 최종적으로는 이 AI 기반 물류 시스템을 전 세계 BMW 공장으로 확대 적용하는 것이다.47

### 5.2 창고 및 물류 자동화

BMW의 사례는 창고 및 물류 자동화라는 더 넓은 영역으로 확장될 수 있다. 이 분야에서 Isaac Sim은 자율 이동 로봇(AMR) 플릿을 개발하고 테스트하는 데 핵심적인 역할을 한다.1

- **인식:** Isaac Sim에서 생성된 합성 데이터를 사용하여 AMR이 팔레트, 선반, 사람과 같은 객체를 정확하게 탐지하고 식별하도록 훈련시킨다.10
- **플릿 관리:** 창고 전체의 디지털 트윈을 구축하여, 수백 대의 로봇으로 구성된 플릿의 움직임을 시뮬레이션하고 최적화한다. 이때 NVIDIA cuOpt와 같은 동적 경로 계획 엔진과 통합하여 전체 물류 효율성을 극대화할 수 있다.7
- **인간-로봇 협업:** 최근 업데이트된 기능으로, 시뮬레이션 환경에 가상의 인간 작업자를 추가하여 로봇과 인간이 안전하고 효율적으로 상호작용하는 시나리오를 검증할 수 있다.12 이는 미래의 협동 작업 환경을 설계하는 데 매우 중요하다.

### 5.3 새로운 개척지와 기타 응용 분야

Isaac Sim의 활용 범위는 공장과 창고를 넘어 다양한 산업으로 확장되고 있다.

- **헬스케어:** 'Isaac for Healthcare' 플랫폼은 의료 환경에서 사용될 AI 기반 로봇을 개발하는 데 활용된다.1
- **건설:** Gole Robotics는 시뮬레이션을 통해 자율 건설 로봇을 설계하여 로봇 무게를 20% 줄이고 개발 비용을 10% 이상 절감하는 성과를 거두었다.4
- **휴머노이드 로봇:** Agility Robotics, Sanctuary AI 등 세계 유수의 휴머노이드 로봇 기업들이 개발 및 훈련 과정에서 Isaac Sim을 적극적으로 활용하고 있다.1
- **일반 산업 자동화:** 두산 로보틱스나 프랑카 에미카(Franka Emika)와 같은 기업들은 Isaac Sim을 사용하여 로봇 팔의 조작 작업을 시뮬레이션에서 훈련시킨 후, 이를 실제 로봇으로 원활하게 이전하는 'Sim-to-Real' 워크플로우를 구현하고 있다.49

## 6.  시작하기: Isaac Sim으로의 첫걸음

이 장은 Isaac Sim을 처음 접하는 사용자를 위해 소프트웨어를 설치하고 기본적인 작업을 수행하는 과정을 안내하는 실용적인 가이드다. Isaac Sim은 강력한 만큼 초기 설정 과정이 다소 복잡할 수 있으나, 이는 플랫폼의 강력한 성능과 협업 기능을 위한 기반이 된다. NVIDIA는 이러한 진입 장벽을 낮추기 위해 새로운 로봇 임포트 마법사나 다양한 튜토리얼을 제공하는 등 지속적으로 사용자 경험을 개선하고 있다.19

### 6.1 시스템 요구사항 및 하드웨어 구성

Isaac Sim을 시작하기 전 가장 먼저 확인해야 할 것은 시스템 요구사항이다. Isaac Sim은 상당한 하드웨어 성능을 요구하므로, 자신의 시스템이 이를 충족하는지 확인하는 것이 매우 중요하다.40 NVIDIA Omniverse Launcher의 Exchange 탭에서 'Compatibility Checker' 앱을 실행하여 시스템 호환성을 미리 확인하는 것이 좋다.43

#### 6.1.1 표 2: NVIDIA Isaac Sim 시스템 요구사항

| 요소          | 최소 사양                          | 권장 사양                          | 이상적인 사양                                        |
| ------------- | ---------------------------------- | ---------------------------------- | ---------------------------------------------------- |
| **운영체제**  | Ubuntu 20.04/22.04, Windows 10/11  | Ubuntu 20.04/22.04, Windows 10/11  | Ubuntu 20.04/22.04, Windows 10/11                    |
| **CPU**       | Intel Core i7 (7세대), AMD Ryzen 5 | Intel Core i7 (9세대), AMD Ryzen 7 | Intel Core i9 (X-시리즈), AMD Ryzen 9 (Threadripper) |
| **코어 수**   | 4                                  | 8                                  | 16                                                   |
| **RAM**       | 32GB*                              | 64GB*                              | 64GB*                                                |
| **저장 공간** | 50GB SSD                           | 500GB SSD                          | 1TB NVMe SSD                                         |
| **GPU**       | GeForce RTX 3070                   | GeForce RTX 4080                   | RTX Ada 6000                                         |
| **VRAM**      | 8GB*                               | 16GB*                              | 48GB*                                                |

*참고: Isaac Sim의 고급 기능(예: 대규모 씬, Isaac Lab 훈련)을 사용하려면 더 많은 RAM과 VRAM이 권장된다. Isaac Sim 컨테이너는 Linux에서만 지원된다.43

### 6.2 설치 및 설정 가이드

다음은 Isaac Sim을 설치하고 설정하는 단계별 과정이다.52

1. **NVIDIA Omniverse Launcher 설치:** NVIDIA 공식 웹사이트에서 Omniverse Launcher를 다운로드하여 설치한다. Launcher는 Isaac Sim을 포함한 모든 Omniverse 애플리케이션을 관리하는 허브 역할을 한다.
2. **Isaac Sim 설치:** Launcher를 실행한 후, 'Exchange' 탭으로 이동하여 'Isaac Sim'을 검색하고 설치한다.
3. **Nucleus 서버 설정:** Nucleus는 USD 파일과 프로젝트 데이터를 저장하고 공유하는 협업 서버다. Launcher의 'Nucleus' 탭에서 로컬 Nucleus 서버를 설치하고 실행할 수 있다. 이는 개인 프로젝트를 위해서도 필요하며, 'localhost'에 서버를 설정하고 관리자 계정을 생성하는 것으로 간단히 시작할 수 있다.
4. **클라우드 접근:** 로컬 하드웨어가 부족한 팀을 위해, 클라우드에서 Isaac Sim을 사용하는 옵션도 제공된다.12

### 6.3 가이드 투어: 첫 번째 시뮬레이션

설치가 완료되었다면, 간단한 'Hello World' 스타일의 튜토리얼을 통해 Isaac Sim에 익숙해져 보자.52

1. **Isaac Sim 실행:** Omniverse Launcher에서 Isaac Sim을 실행한다. 초기 로딩에 다소 시간이 걸릴 수 있다.
2. **인터페이스 둘러보기:** Isaac Sim의 UI는 크게 네 부분으로 구성된다.55
   - **Viewport:** 3D 씬이 렌더링되는 메인 창.
   - **Stage:** 씬을 구성하는 모든 객체(프림)들이 계층 구조로 표시되는 창.
   - **Property Panel:** Stage에서 선택한 객체의 속성을 보고 편집하는 창.
   - **Content Browser:** 로봇, 환경, 재질 등 미리 만들어진 3D 에셋을 탐색하고 불러오는 창.
3. **환경과 로봇 불러오기:** Content Browser를 사용하여 미리 준비된 환경(예: 'warehouse')과 로봇 모델(예: 'Franka')을 Viewport로 드래그 앤 드롭하여 씬에 추가한다.14
4. **기본적인 상호작용:** 상단의 'Play' 버튼을 눌러 시뮬레이션을 시작한다. Stage에서 로봇의 관절을 선택하고 Property Panel에서 값을 조절하여 로봇을 움직여보자. 물리 엔진이 작동하는 것을 실시간으로 확인할 수 있다.14
5. **샘플 스크립트 실행:** Isaac Sim은 GUI뿐만 아니라 Python 스크립트를 통한 제어도 지원한다. 내장된 예제 스크립트 중 하나를 실행하여 로봇이 프로그래밍적으로 제어되는 것을 확인해보자.54

## 7.  물리 AI의 미래: Isaac Sim 로드맵

이 마지막 장에서는 Isaac Sim의 미래 발전 방향과 AI 및 로보틱스의 미래에서 이 플랫폼이 수행할 중심적인 역할을 분석한다. NVIDIA의 전략은 단순히 더 나은 시뮬레이터를 만드는 것을 넘어, 물리 AI 개발의 전 과정을 아우르는 통합 플랫폼을 구축하는 데 있다.

### 7.1 최신 로드맵 분석 (Isaac Sim 5.0 이상)

최근 발표된 Isaac Sim 5.0 개발자 프리뷰와 GTC 발표 내용을 통해 NVIDIA의 전략적 방향을 엿볼 수 있다.18

- **개방성과 확장성:** Isaac Sim의 핵심 확장 기능들을 GitHub를 통해 오픈 소스화한 것은 매우 중요한 전략적 변화다.19 이는 커뮤니티의 참여를 유도하고 더 빠른 혁신을 촉진하여, Isaac Sim을 사실상의 표준 플랫폼으로 만들려는 의도다.
- **표준화:** 로봇과 센서를 위한 표준화된 USD 스키마(예: OmniSensor)를 도입하고, 표준 ROS2 시뮬레이션 인터페이스를 채택하는 것은 생태계 확장을 위한 핵심 전략이다.18 이는 하드웨어 제조사와 개발자들이 자신의 제품과 코드를 Isaac Sim 생태계에 더 쉽게 통합할 수 있도록 진입 장벽을 낮춘다.
- **Sim-to-Real 강화:** 새로운 액추에이터 및 마찰 모델, 사실적인 깊이 센서 노이즈 모델 등 물리 및 센서 시뮬레이션의 지속적인 개선은 Sim-to-Real 갭을 줄이려는 NVIDIA의 끊임없는 노력을 보여준다.18
- **성능과 확장성:** Omniverse Fabric 통합을 통한 시뮬레이션 속도 향상과 멀티 GPU 테스트 최적화는 Isaac Sim이 대규모 산업용 애플리케이션을 목표로 하고 있음을 명확히 한다.18

### 7.2 거대한 비전: 파운데이션 모델과의 시너지

NVIDIA의 궁극적인 비전은 범용 로봇 개발을 위한 통합 플랫폼을 구축하는 것이다. 이는 물리 AI 개발의 '성스러운 삼위일체'로 요약될 수 있다.2

1. **NVIDIA Isaac Sim:** 고충실도의 가상 세계. 로봇을 테스트하고 훈련하는 '체육관(Gym)' 역할을 한다.
2. **NVIDIA Cosmos:** '월드 파운데이션 모델' 플랫폼. 방대하고 다양한 동적 가상 환경과 합성 데이터를 대규모로 생성하는 '세계 창조 엔진'이다.1 Cosmos가 Isaac Sim의 세계를 채우게 될 것이다.
3. **Project GR00T:** 휴머노이드 로봇을 위한 '범용 로봇 파운데이션 모델'. Cosmos가 생성한 세계와 Isaac Sim이라는 체육관에서 훈련받을 AI '선수'의 두뇌에 해당한다.2

이 세 가지 기둥은 다음과 같이 상호작용한다. Cosmos가 훈련장을 만들고, Isaac Sim이 훈련이 일어나는 장소를 제공하며, GR00T는 그 안에서 훈련받는 AI다. 이는 물리 AI를 구축하기 위한 완벽한 엔드-투-엔드 파이프라인을 형성한다.

이러한 NVIDIA의 로보틱스 전략은 실리콘부터 소프트웨어, AI 모델에 이르기까지 모든 것을 아우르는 완벽한 수직 통합 전략이다. 그 기반에는 AI 모델 훈련을 위한 DGX, 시뮬레이션을 위한 OVX, 그리고 로봇에 탑재되는 AGX(Jetson)라는 세 가지 컴퓨팅 플랫폼이 있다.2 그 위에 Omniverse, Isaac Sim과 같은 플랫폼 소프트웨어가 구축되며, 이는 NVIDIA 하드웨어에 최적화되어 있다. 그리고 이제는 Cosmos와 GR00T라는 파운데이션 AI 모델 자체를 직접 구축하고 있다. 마지막으로 BMW, 두산 로보틱스 등 전 세계 로보틱스 산업과의 공격적인 파트너십을 통해 이 풀 스택의 채택을 가속화하고 있다.47 이는 NVIDIA가 단순한 부품 공급업체를 넘어, 다가오는 로봇 혁명의 중심 허브가 되려는 거대한 플랫폼 전략의 일환이다.

### 7.3 결론 및 개발자를 위한 제언

NVIDIA Isaac Sim은 단순한 시뮬레이터를 넘어, 차세대 AI 기반 로보틱스를 위한 기초 플랫폼이다. 사진처럼 사실적인 그래픽, GPU 가속을 통한 압도적인 성능, 그리고 NVIDIA 생태계와의 긴밀한 통합은 타의 추종을 불허하는 강점이다. 물론 높은 하드웨어 요구사항과 초기 학습 곡선이라는 도전 과제도 존재한다.

하지만 이 보고서의 분석이 보여주듯이, Isaac Sim을 마스터하기 위한 초기 투자는 미래를 위한 전략적인 경력 투자가 될 것이다. 이는 개발자들을 물리 AI와 산업 디지털화라는 거대한 기술 전환의 최전선에 서게 할 것이다. 로보틱스의 미래는 Isaac Sim과 같은 플랫폼 위에서 구축될 것이며, 지금이야말로 이 혁명에 동참해야 할 때다.

#### **참고 자료**

1. Isaac Sim - Robotics Simulation and Synthetic Data Generation ..., accessed July 1, 2025, https://developer.nvidia.com/isaac/sim
2. Physical AI and Robotics Conference Sessions | NVIDIA GTC 2025, accessed July 1, 2025, https://www.nvidia.com/gtc/sessions/physical-ai-and-robotics/
3. ROS2와 isaac sim 연동 시 이점 - 알루의 일상 - 티스토리, accessed July 1, 2025, https://memorize-tail.tistory.com/387
4. 생산 비용을 절감하고 오류를 없애는 GOLE ROBOTICS | 고객 사례 - SolidWorks, accessed July 1, 2025, https://www.solidworks.com/ko/customer-story/gole-robotics-reduces-production-costs-and-eliminates-errors
5. 로봇 시뮬레이션 환경 개발의 모든 것 - wntdev - 티스토리, accessed July 1, 2025, https://wntdev.tistory.com/199
6. [AI넷] [Isaac Sim으로 AI 로봇의 개발, 테스트 및 교육 가속화] NVIDIA는이미 NVIDIA Isaac Sim을 발표했다. 로보틱스 시뮬레이션 및 합성 데이터 생성(SDG) 도구인 이 NVIDIA Omniverse 애플리케이션은 로보틱스에서 AI의 개발, 테스트 및 훈련을 가속화한다., accessed July 1, 2025, http://www.ainet.link/17987
7. 로보틱스 시뮬레이션 | 사용 사례 | NVIDIA, accessed July 1, 2025, https://www.nvidia.com/ko-kr/use-cases/robotics-simulation/
8. NVIDIA Isaac Sim, ROS 및 Nimbus를 통한 멀티 로봇 환경 개발, accessed July 1, 2025, https://developer.nvidia.com/ko-kr/blog/develop-a-multi-robot-environment-with-nvidia-isaac-sim-ros-and-nimbus/
9. 엔비디아, 디지털트윈-실시간 AI 결합 소개..."AI 에이전트로 비용 절감" - AI타임스, accessed July 1, 2025, https://www.aitimes.com/news/articleView.html?idxno=158179
10. Isaac Sim - NVIDIA Technical Blog, accessed July 1, 2025, https://developer.nvidia.com/ko-kr/blog/recent-posts/?products=Isaac+Sim
11. Databricks와 NVIDIA Omniverse를 이용한 인식 AI를 위한 확장 가능한 합성 데이터 생성 파이프라인 구축, accessed July 1, 2025, https://www.databricks.com/kr/blog/building-scalable-synthetic-data-generation-pipelines-perception-ai-databricks-and-nvidia
12. 지능형 로봇 시뮬레이션 향상시키는 NVIDIA Isaac Sim 업데이트 ..., accessed July 1, 2025, https://blogs.nvidia.co.kr/blog/isaac-sim-major-updates/
13. blogs.nvidia.co.kr, accessed July 1, 2025, [https://blogs.nvidia.co.kr/blog/isaac-sim-major-updates/#:~:text=CES%202023%EC%97%90%EC%84%9C%20NVIDIA%20Isaac,%ED%95%98%EB%8A%94%20%EB%A1%9C%EB%B4%87%20%EC%8B%9C%EB%AE%AC%EB%A0%88%EC%9D%B4%EC%85%98%20%EB%8F%84%EA%B5%AC%EC%9E%85%EB%8B%88%EB%8B%A4.](https://blogs.nvidia.co.kr/blog/isaac-sim-major-updates/#:~:text=CES 2023에서 NVIDIA Isaac,하는 로봇 시뮬레이션 도구입니다.)
14. isaac sim 튜토리얼 - 알루의 일상, accessed July 1, 2025, https://memorize-tail.tistory.com/155
15. What Is Isaac Sim? — Isaac Sim 4.2.0 (OLD) - NVIDIA Omniverse, accessed July 1, 2025, https://docs.omniverse.nvidia.com/isaacsim/latest/index.html
16. NVIDIA Isaac - AI Robot Development Platform, accessed July 1, 2025, https://developer.nvidia.com/isaac
17. 합성 데이터를 사용하여 창고 팔레트 잭을 감지하도록 자율 모바일 로봇을 훈련하는 방법, accessed July 1, 2025, https://developer.nvidia.com/ko-kr/blog/how-to-train-autonomous-mobile-robots-to-detect-warehouse-pallet-jacks-using-synthetic-data/
18. Isaac Sim 5.0 Developer Preview - NVIDIA Developer Forums, accessed July 1, 2025, https://forums.developer.nvidia.com/t/isaac-sim-5-0-developer-preview/336340
19. Isaac Sim and Isaac Lab Are Now Available for Early Developer Preview | NVIDIA Technical Blog, accessed July 1, 2025, https://developer.nvidia.com/blog/isaac-sim-and-isaac-lab-are-now-available-for-early-developer-preview/
20. Advanced Sensor Physics, Customization, and Model Benchmarking Coming to NVIDIA Isaac Sim and NVIDIA Isaac Lab | NVIDIA Technical Blog - NVIDIA Developer, accessed July 1, 2025, https://developer.nvidia.com/blog/advanced-sensor-physics-customization-and-model-benchmarking-coming-to-nvidia-isaac-sim-and-nvidia-isaac-lab/
21. NVIDIA Isaac Sim - GitHub, accessed July 1, 2025, https://github.com/isaac-sim
22. PhysX SDK - Latest Features & Libraries - NVIDIA Developer, accessed July 1, 2025, https://developer.nvidia.com/physx-sdk
23. NVIDIA Open Sources PhysX 5 for 3D Simulation - General - ROS Discourse, accessed July 1, 2025, https://discourse.ros.org/t/nvidia-open-sources-physx-5-for-3d-simulation/28262
24. [Lab Meeting] Isaacgym과 Mujoco 시뮬레이터 사이의 Sim2Sim Gap ..., accessed July 1, 2025, https://www.youtube.com/watch?v=0rgd-fuL-wc
25. Isaac Sim: Photorealistic Rendering for Next-Gen Robot Development - Medium, accessed July 1, 2025, https://medium.com/black-coffee-robotics/isaac-sim-photorealistic-rendering-for-next-gen-robot-development-37ca8186c1d9
26. Isaac Sim으로 AI 로봇의 개발, 테스트 및 트레이닝 신속하게 진행하기 - NVIDIA Developer, accessed July 1, 2025, https://developer.nvidia.com/ko-kr/blog/expedite-the-development-testing-and-training-of-ai-robots-with-isaac-sim/
27. 시뮬레이션과 현실의 간극 좁히기: NVIDIA Isaac Lab을 통한 Spot 사족보행 트레이닝, accessed July 1, 2025, https://developer.nvidia.com/ko-kr/blog/closing-the-sim-to-real-gap-training-spot-quadruped-locomotion-with-nvidia-isaac-lab/
28. 로봇 시뮬레이션 강화학습 Isaac Gym 짧은 사용기, accessed July 1, 2025, https://brunch.co.kr/@@54nN/43
29. Isaac Lab – 데모 실행 및 핵심 키워드 정리 - Pollux AI, accessed July 1, 2025, https://www.pollux.ai/ko/tech-blog/isaac-lab-running-demo-scripts-and-understanding-core-keywords
30. Isaac Lab - Isaac Sim Documentation, accessed July 1, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/isaac_lab_tutorials/index.html
31. NVIDIA Isaac Lab, accessed July 1, 2025, https://developer.nvidia.com/isaac/lab
32. Welcome to Isaac Lab!, accessed July 1, 2025, https://isaac-sim.github.io/IsaacLab/
33. Interactive Scene - Isaac Lab Tutorial 1 (Reinforcement Learning) - YouTube, accessed July 1, 2025, https://www.youtube.com/watch?v=Y-K1cAvnSFI
34. *NEW* Isaac Sim 5.0 | Installation & Changes - YouTube, accessed July 1, 2025, https://www.youtube.com/watch?v=axy1Ze8mJd8
35. Take Control of your Robot in Isaac Sim - Real2Sim Tutorial - YouTube, accessed July 1, 2025, https://www.youtube.com/watch?v=VChG0cn05_0
36. From Simulation to Reality: Building Wheeled Robots with Isaac Lab - YouTube, accessed July 1, 2025, https://www.youtube.com/watch?v=Y4b-cz2xB1w
37. Go2 Edu sim-to-real from IsaacLab : r/unitree - Reddit, accessed July 1, 2025, https://www.reddit.com/r/unitree/comments/1if0hof/go2_edu_simtoreal_from_isaaclab/
38. 로봇운영체제(ROS)용 NVIDIA Isaac GEM으로 로봇 설계하기, accessed July 1, 2025, [https://developer.nvidia.com/ko-kr/blog/%EB%A1%9C%EB%B4%87%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9Cros%EC%9A%A9-nvidia-isaac-gem%EC%9C%BC%EB%A1%9C-%EB%A1%9C%EB%B4%87-%EC%84%A4%EA%B3%84%ED%95%98%EA%B8%B0/](https://developer.nvidia.com/ko-kr/blog/로봇운영체제ros용-nvidia-isaac-gem으로-로봇-설계하기/)
39. Isaac Sim 설치 및 환경 구성 가이드라인 - GitHub, accessed July 1, 2025, https://github.com/lbaa2022/IsaacSimGuidelines/blob/main/README.md
40. Robotics simulation – A comparison of two state-of-the-art solutions - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/362100025_Robotics_simulation_-_A_comparison_of_two_state-of-the-art_solutions
41. Robotics simulation – A comparison of two state-of-the-art solutions - ASIM GI, accessed July 1, 2025, https://www.asim-gi.org/fileadmin/user_upload_asim/ASIM_Publikationen_OA/AM180/a2033.arep.20_OA.pdf
42. Best Robot Simulators - Formant, accessed July 1, 2025, https://formant.io/blog/best-robot-simulators/
43. Isaac Sim Requirements — Isaac Sim 4.2.0 (OLD) - NVIDIA Omniverse, accessed July 1, 2025, https://docs.omniverse.nvidia.com/isaacsim/latest/installation/requirements.html
44. Help choosing simulators : r/robotics - Reddit, accessed July 1, 2025, https://www.reddit.com/r/robotics/comments/z905og/help_choosing_simulators/
45. Gazebo vs. Isaac : r/ROS - Reddit, accessed July 1, 2025, https://www.reddit.com/r/ROS/comments/1fc2a3q/gazebo_vs_isaac/
46. 강화학습, 시뮬레이션이 미래다. (Nvidia Isaac Sim vs Unity 3D, Unity ML-Agents) - pnltoen, accessed July 1, 2025, [https://pnltoen.tistory.com/entry/%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%8B%9C%EB%AE%AC%EB%A0%88%EC%9D%B4%EC%85%98%EC%9D%B4-%EB%AF%B8%EB%9E%98%EB%8B%A4-Nvidia-Isaac-Sim-vs-Unity-3D-Unity-ML-Agents](https://pnltoen.tistory.com/entry/강화학습-시뮬레이션이-미래다-Nvidia-Isaac-Sim-vs-Unity-3D-Unity-ML-Agents)
47. BMW, 엔비디아 '아이작' 로봇 플랫폼 도입한다 - 로봇신문, accessed July 1, 2025, http://www.irobotnews.com/news/articleView.html?idxno=20735
48. 자동차 공정 혁신 위해 '엔비디아 아이작(Isaac)' 로봇 플랫폼 도입한 BMW | NVIDIA Blog, accessed July 1, 2025, https://blogs.nvidia.co.kr/blog/bmw-group-selects-nvidia-to-redefine-factory-logistics/
49. NVIDIA and Partners Highlight Next-Generation Robotics ..., accessed July 1, 2025, https://blogs.nvidia.com/blog/automatica-robotics-2025/
50. Agile Robots and Franka Robotics at NVIDIA GTC 2025, accessed July 1, 2025, https://www.agile-robots.com/en/news/detail/agile-robots-and-franka-robotics-at-nvidia-gtc-2025
51. Release Notes - Isaac Sim Documentation, accessed July 1, 2025, https://docs.isaacsim.omniverse.nvidia.com/latest/overview/release_notes.html
52. 초급] #1 Isaac Sim 소개 및 설치 - YouTube, accessed July 1, 2025, https://m.youtube.com/watch?v=uRdfy5IacOg
53. [Isaac Sim - 초급] #1-1 설치과정 소개, accessed July 1, 2025, https://www.youtube.com/watch?v=WqVqiPqFyU0
54. NVIDIA Omniverse Isaac sim 튜토리얼 1.2 Isaac Sim WorkFlow - velog, accessed July 1, 2025, [https://velog.io/@ktjktj0629/NVIDIA-Omniverse-Isaac-sim-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC-1.2-Isaac-Sim-WorkFlow](https://velog.io/@ktjktj0629/NVIDIA-Omniverse-Isaac-sim-튜토리얼-1.2-Isaac-Sim-WorkFlow)
55. 초급] #2 Isaac Sim 기본 튜토리얼 1 - YouTube, accessed July 1, 2025, https://www.youtube.com/watch?v=8pWfoJss35o

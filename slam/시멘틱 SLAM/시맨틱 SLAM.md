# 시맨틱 SLAM에 대한 고찰

## 1.  기하학적 세계 모델에서 시맨틱 세계 모델로의 진화

자율 시스템이 주변 환경을 인식하고 상호작용하는 방식은 지난 수십 년간 근본적인 변화를 겪었다. 초기 로보틱스 연구의 핵심 과제였던 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM)은 순수한 기하학적 공간 인식의 문제를 해결하는 데 중점을 두었다. 그러나 진정한 자율성을 달성하기 위해서는 단순히 '어디에 있는가'를 아는 것을 넘어 '무엇이 있는가'를 이해하는 능력이 필수적이라는 인식이 확산되면서, 시맨틱 SLAM이라는 새로운 패러다임이 등장했다. 이 섹션에서는 전통적인 기하학적 SLAM의 원리와 본질적 한계를 재조명하고, 시맨틱 정보의 통합이 왜 필연적인 진화였는지를 논하며, 시맨틱 SLAM이 제공하는 핵심적인 이점들을 분석한다.

### 1.1  전통적 SLAM의 재조명: 원리 및 한계

전통적인 SLAM은 미지의 환경에서 로봇이나 자율 에이전트가 자신의 위치와 방향(상태)을 추정함과 동시에 주변 환경의 지도를 구축하는 근본적인 문제를 다룬다 `[1, 2]`. 이는 본질적으로 "나는 어디에 있는가?"라는 질문에 답하기 위한 기술적 프레임워크다 `[3]`. SLAM 시스템은 일반적으로 두 가지 핵심 구성 요소로 나뉜다. 첫째, 센서 데이터를 처리하여 단기적인 움직임을 추정하는 **프론트엔드(Front-end)**와, 둘째, 누적된 오차를 보정하고 전역적으로 일관된 지도와 궤적을 생성하는 **백엔드(Back-end)**다 `[1, 4]`.

초기 SLAM 연구는 확장 칼만 필터(EKF SLAM)와 같은 필터 기반 접근법이 주를 이루었으나, 계산 복잡도와 비선형성 문제로 인해 점차 그래프 기반 최적화 방법으로 대체되었다 `[5, 6]`. 그래프 SLAM, 특히 포즈 그래프 최적화(Pose-Graph Optimization)는 로봇의 궤적과 환경의 특징점(Landmark)을 그래프의 노드(Node)와 엣지(Edge)로 표현하고, 전체 그래프의 오차를 최소화하는 방식으로 전역 일관성을 확보한다 `[5, 7]`. 이러한 시스템에서 랜드마크는 주로 점(point), 선(line), 평면(plane)과 같은 기하학적 원시 요소(geometric primitives)로 정의된다 `[8, 9]`.

이러한 기하학적 접근법은 정적인 환경에서 놀라운 수준의 정확도를 달성하며 자율 이동 기술의 초석을 다졌지만, 현실 세계의 복잡성과 동적인 특성에 직면하면서 명백한 한계에 부딪혔다.

첫째, **장면 이해의 부재(Lack of Scene Understanding)**가 가장 근본적인 문제다. 전통적인 SLAM이 생성하는 지도는 기하학적으로는 정밀할 수 있으나 의미론적으로는 불모지와 같다 `[10]`. 지도는 벽과 장애물의 위치는 알려주지만, 그것이 '책상'인지 '의자'인지는 알려주지 않는다. 이러한 정보의 부재는 로봇이 "부엌 식탁에서 컵을 가져와"와 같이 인간 중심의 복잡한 임무를 수행하는 것을 원천적으로 불가능하게 만든다 `[11]`.

둘째, **도전적인 환경에서의 취약성(Brittleness in Challenging Environments)**이다. 전통적인 SLAM 알고리즘은 대부분 '정적 세계 가정(static world assumption)'에 기반한다 `[12]`. 그러나 현실 세계는 사람, 차량 등 움직이는 객체로 가득 차 있다. 이러한 동적 요소들은 정적인 랜드마크라는 가정을 위배하여 위치 추정의 심각한 오류를 유발하고 지도 오염의 원인이 된다 `[3, 13, 14]`. 또한, 특징이 거의 없는 벽이나 복도(low-texture areas) 또는 조명이 급격하게 변하는 환경에서도 안정적인 특징점을 추출하기 어려워 성능이 급격히 저하된다 `[11, 13]`.

셋째, **데이터 연관(Data Association)의 어려움**이다. 로봇이 이전에 방문했던 장소를 다시 인식하여 누적된 오차를 보정하는 루프 클로저(loop closure)는 SLAM의 정확도에 매우 중요하다. 그러나 서로 다른 장소가 시각적으로 유사해 보이는 '지각적 모호성(perceptual aliasing)' 현상 때문에 순수 기하학적 특징만으로는 강건한 루프 클로저를 달성하기 어렵다 `[14]`.

이러한 한계들은 로보틱스가 단순한 공간 이동(navigation)을 넘어 환경과의 지능적인 상호작용(interaction)으로 나아가기 위해 반드시 넘어야 할 벽이었다. 기하학적 정보만으로는 이 벽을 넘을 수 없다는 것이 명백해지면서, 시맨틱 정보의 통합은 선택이 아닌 필연이 되었다.

### 1.2  시맨틱의 필요성: 시맨틱 SLAM의 정의

시맨틱 SLAM은 전통적인 SLAM 파이프라인에 객체 종류, 속성, 기능과 같은 고수준의 의미론적 정보(semantic information)를 통합하는 진화된 패러다임이다 `[3, 12, 15]`. 이는 단순히 지도를 만드는 것을 넘어, 기계가 읽고 이해할 수 있는 풍부한 '세계 모델(world model)'을 구축하는 것을 목표로 한다 `[16]`. 즉, 순수한 기하학적 표현에서 미터법 기반의 의미론적 이해(metric-semantic understanding)로의 전환을 의미한다 `[17]`.

시맨틱 SLAM의 핵심 가치는 지도를 위치 추정을 위한 도구에서 지능적인 상호작용, 의사결정, 그리고 인간-로봇 협업을 가능하게 하는 '지식 베이스(knowledge base)'로 격상시키는 데 있다 `[11, 18, 19]`. 로봇의 공간에 대한 인지적 표현 능력을 향상시켜 인간과 유사한 방식으로 환경을 이해하고 소통할 수 있는 기반을 제공하는 것이다 `[20]`.

### 1.3  시맨틱 접근법의 핵심 이점

시맨틱 정보의 통합은 전통적 SLAM의 한계를 직접적으로 해결하며 다음과 같은 핵심적인 이점을 제공한다.

- **강건성 향상(Enhanced Robustness):** 시맨틱 SLAM은 딥러닝 모델을 통해 사람이나 차량과 같은 동적 객체를 식별하고 분할할 수 있다 `[14]`. 이를 통해 동적 객체에 속한 특징점들을 지도 작성 과정에서 제외하거나 별도로 모델링함으로써, 동적 환경에서도 위치 추정의 정확성과 지도의 일관성을 획기적으로 향상시킨다 `[8]`.
- **데이터 연관 및 루프 클로저 성능 고도화(Superior Data Association and Loop Closure):** '독특한 모양의 책장'이나 '사무실의 커피 머신'과 같이 의미론적으로 구별되는 객체는 기하학적 모호성을 뛰어넘는 강력한 랜드마크 역할을 한다. 이를 통해 데이터 연관의 신뢰도를 높이고 루프 클로저의 성공률을 크게 개선할 수 있다 `[3, 8]`. 데이터 연관의 단위가 픽셀 수준에서 객체 수준으로 격상되는 것이다 `[11]`.
- **고수준 임무 수행 능력 확보(Enabling High-Level Tasks):** 이는 시맨틱 SLAM의 가장 중요한 이점이다. 의미 정보가 포함된 지도는 로봇이 "거실 소파 옆에 있는 리모컨을 가져와"와 같은 자연어 명령을 이해하고 수행할 수 있게 한다. 이는 로봇의 인식(perception)과 행동(action) 사이의 간극을 메우는 핵심적인 역할을 한다 `[11, 21, 22]`. 궁극적으로, 시맨틱 맵은 인간과 로봇 간의 '협력적 인지 작업 공간(collaborative cognitive workspace)'을 형성하는 기반이 된다 `[23]`.

아래의 표는 전통적인 SLAM과 시맨틱 SLAM의 핵심적인 차이점을 요약하여 보여준다.

| 특징               | 전통적 SLAM                                                  | 시맨틱 SLAM                                                  |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **지도 표현**      | 점, 선, 평면 기반의 기하학적 지도 (예: 점 구름, 점유 격자 지도) | 객체, 장소, 관계를 포함하는 의미론적 지도 (예: 시맨틱 3D 메시, 장면 그래프) |
| **랜드마크 유형**  | 기하학적 특징점 (예: ORB, SIFT)                              | 의미론적 객체 (예: 의자, 책상) 또는 장소 (예: 부엌, 복도)    |
| **동적 환경 처리** | 동적 요소를 노이즈 또는 이상치로 간주하여 성능 저하 발생     | 동적 객체를 명시적으로 탐지하고 분리하여 강건성 확보         |
| **데이터 연관**    | 기하학적 외형에 의존하여 지각적 모호성에 취약                | 객체의 의미론적 정체성을 활용하여 강건하고 신뢰성 높은 연관 수행 |
| **수행 가능 임무** | 위치 추정, 경로 계획, 장애물 회피                            | 자연어 명령 이해, 객체 조작, 인간-로봇 상호작용 등 고수준 임무 |

## 2.  핵심 기술 및 아키텍처 패러다임

시맨틱 SLAM이 어떻게 작동하는지를 이해하기 위해서는 그 구성 요소와 아키텍처, 그리고 이를 가능하게 하는 핵심 기술인 딥러닝과의 공생 관계를 깊이 있게 분석해야 한다. 이 섹션에서는 시맨틱 SLAM 파이프라인의 각 단계를 해부하고, 딥러닝이 통합되는 다양한 방식을 체계적으로 분류한다. 또한, 현대 시맨틱 SLAM을 대표하는 세 가지 주요 아키텍처(기하학적, NeRF, 3D 가우시안 스플래팅)를 비교 분석하며, 특히 실제 적용의 핵심 제약 조건인 임베디드 시스템에서의 성능과 한계를 고찰한다.

### 2.1  시맨틱 SLAM 파이프라인: 구성 요소별 분석

시맨틱 SLAM 파이프라인은 전통적인 SLAM 구조를 기반으로 하되, 각 단계에 의미론적 정보를 유기적으로 통합하는 방식으로 구성된다 `[3, 15]`.

- **시맨틱 정보 추출/인식(Semantic Extraction/Perception):** 이 모듈은 시맨틱 정보가 시스템으로 유입되는 첫 관문이다. 딥러닝 기반의 컴퓨터 비전 모델들이 이 단계에서 핵심적인 역할을 수행한다. **객체 탐지(Object Detection)** 모델(예: YOLO 시리즈, SSD)은 이미지 내 객체의 위치와 클래스를 바운딩 박스 형태로 식별하고 `[13, 24]`, **시맨틱 분할(Semantic Segmentation)** 모델(예: SegNet, PSPNet)은 이미지의 모든 픽셀을 특정 클래스(예: 도로, 건물, 사람)로 분류한다 `[18, 25]`. 더 나아가 **인스턴스 분할(Instance Segmentation)** 모델(예: Mask R-CNN)은 개별 객체 인스턴스를 픽셀 단위로 구분하여, 동일한 클래스의 여러 객체를 구별할 수 있게 한다 `[8, 18]`. 이 모듈의 결과물은 후속 단계에서 사용될 픽셀 또는 객체 수준의 의미론적 레이블이다.
- **시맨틱 데이터 연관(Semantic Data Association):** 이 모듈은 시간의 흐름에 따라 관측된 의미론적 정보들을 동일한 객체 또는 장소에 일관되게 연결하는 역할을 한다. 이는 단순한 기하학적 매칭보다 훨씬 복잡한 문제로, 객체의 일부가 가려지거나 시점이 크게 바뀌어도 동일한 인스턴스임을 인식해야 한다. 객체의 의미론적 일관성(예: 한번 '의자'로 인식된 객체는 계속 '의자'일 가능성이 높음)과 같은 강력한 제약 조건을 활용하며, 확률적 접근법을 통해 불확실성을 관리한다 `[9]`.
- **백엔드 최적화(Back-end Optimization):** SLAM 시스템의 핵심인 백엔드에서는 시맨틱 정보가 새로운 형태의 제약 조건(constraint)으로 그래프 최적화 과정에 통합된다. 예를 들어, '책상은 바닥 위에 있다'와 같은 객체의 물리적 속성이나 '의자는 책상 옆에 있다'와 같은 객체 간의 공간적 관계를 그래프의 추가적인 팩터(factor)로 모델링할 수 있다 `[3, 8]`. 이러한 시맨틱 제약 조건은 기하학적 정보만으로는 해결하기 어려운 모호성을 줄여주어, 로봇의 궤적과 지도의 정확도를 동시에 향상시키는 데 기여한다.
- **시맨틱 지도 작성(Semantic Mapping):** 파이프라인의 최종 결과물로, 의미 정보가 풍부하게 포함된 지도를 생성하는 모듈이다 `[3]`. 이 지도는 단순히 점들의 집합이 아니라, 객체, 공간, 그리고 그들 간의 관계가 구조화된 형태로 표현된다. 시맨틱 지도의 구체적인 형태는 응용 목적에 따라 다양하며, 이는 섹션 3에서 자세히 다룬다.

### 2.2  딥러닝과의 공생: 통합 패턴의 유형

딥러닝 기술을 시각 SLAM(VSLAM)에 통합하는 방식은 그 깊이와 역할에 따라 세 가지 주요 패턴으로 분류할 수 있다 `[24]`. 이 분류는 시맨틱 SLAM 기술의 발전 과정을 이해하는 중요한 틀을 제공한다.

- **유형 1: 보조 모듈 추가(Auxiliary Modules):** 가장 일반적이고 널리 사용되는 접근 방식이다. 기존의 강건한 기하학적 SLAM 프레임워크(예: ORB-SLAM)를 그대로 유지하면서, 딥러닝 모델을 '부가 기능' 형태로 추가한다. 예를 들어, Mask R-CNN을 사용하여 동적 객체를 탐지하고 해당 영역의 특징점을 제거하거나 `[24]`, 딥러닝 기반 장소 인식 모델을 통해 루프 클로저 성능을 향상시키는 방식이다. 이 접근법은 기존 시스템의 안정성을 해치지 않으면서 딥러닝의 강력한 인식 능력을 활용할 수 있다는 장점이 있다.
- **유형 2: 기존 모듈 대체(Module Replacement):** 전통적인 SLAM 파이프라인의 특정 핵심 모듈을 딥러닝 기반 모듈로 완전히 대체하는, 보다 통합적인 방식이다. 예를 들어, 수작업으로 설계된 특징점 추출기(예: ORB)를 딥러닝 기반 특징점 추출 네트워크로 대체하여 텍스처가 부족한 환경에서의 성능을 개선하거나(예: DDL-SLAM `[24]`), 전통적인 자세 추정 알고리즘을 딥러닝 네트워크로 교체하는 시도가 여기에 해당한다.
- **유형 3: 종단간(End-to-End) 시스템:** 가장 급진적인 접근 방식으로, SLAM 문제 전체를 하나의 거대한 딥러닝 네트워크를 통해 해결한다. 입력 이미지 시퀀스를 받아 최종적인 카메라 궤적을 직접 출력하는 형태다(예: DeepVO, VIOLearner `[24]`). 이 방식은 데이터로부터 모든 것을 학습하므로 잠재적으로 인간이 설계한 알고리즘의 한계를 뛰어넘을 수 있지만, 아직까지는 새로운 환경에 대한 일반화 성능, 결과의 해석 가능성, 그리고 방대한 학습 데이터 요구량 등에서 어려움을 겪고 있다.

이 세 가지 통합 패턴은 전통적인 기하학적 방법에 대한 신뢰도와 학습 기반 표현에 대한 의존도 사이의 스펙트럼을 형성한다. 현재 '보조 모듈 추가' 방식이 가장 널리 사용되는 것은, ORB-SLAM과 같은 기하학적 백엔드의 검증된 강건성을 유지하면서 딥러닝의 강력한 인식 능력을 선택적으로 활용하는 것이 현실적으로 가장 안정적이고 효과적인 전략임을 시사한다.

### 2.3  아키텍처 비교: 시맨틱 기하학 vs. NeRF vs. 3D 가우시안 스플래팅

최근 시맨틱 SLAM 연구는 장면을 표현하는 방식에 따라 세 가지 주요 아키텍처로 나뉜다. 이들의 비교는 특히 자율 로봇 및 AR 기기와 같은 임베디드 시스템에서의 실용성을 평가하는 데 매우 중요하다 `[18, 26]`.

- **시맨틱 기하학적 SLAM(Semantic Geometric SLAM):** 가장 고전적이고 실용적인 접근법이다. ORB-SLAM3와 같이 잘 확립된 기하학적 SLAM 시스템을 기반으로 하여, 여기에 의미론적 레이블을 추가하는 방식이다 `[18]`.
  - **장점:** 계산 효율성이 뛰어나고 기술 성숙도가 높아, NVIDIA Jetson AGX Orin과 같은 임베디드 플랫폼에서 실시간으로 동작 가능한 정확도와 성능의 균형을 제공한다 `[18, 26]`.
  - **단점:** 지도 표현이 주로 특징점 기반의 희소(sparse)한 형태이거나, 밀집(dense)하더라도 사진과 같은 사실감(photorealism)은 부족하다.
- **시맨틱 NeRF SLAM(Semantic Neural Radiance Fields SLAM):** 장면을 다층 퍼셉트론(MLP)과 같은 신경망으로 학습된 연속적인 볼륨 함수로 표현한다 `[18]`.
  - **장점:** 매우 상세하고 사진처럼 사실적인 밀집 지도를 생성하며, 새로운 시점에서의 이미지를 렌더링(novel view synthesis)할 수 있다. 의미 정보 또한 신경망 필드 내에 직접 임베딩할 수 있다 `[18]`.
  - **단점:** 신경망을 훈련하고 렌더링하는 데 막대한 계산 자원이 필요하여, 현재 임베디드 장치에서의 실시간 사용은 거의 불가능하다 `[18, 26]`.
- **시맨틱 3D 가우시안 스플래팅 SLAM(Semantic 3D Gaussian Splatting SLAM):** NeRF의 대안으로 최근 각광받는 기술로, 장면을 수많은 3D 가우시안의 집합으로 표현한다 `[18]`.
  - **장점:** NeRF와 유사한 수준의 사실적인 렌더링 품질을 달성하면서도 렌더링 속도가 훨씬 빠르다. 의미 정보는 가우시안의 속성으로 직접 임베딩될 수 있다(예: SemGauss-SLAM `[18]`).
  - **단점:** NeRF보다 빠르지만, 여전히 상당한 컴퓨팅 자원을 요구하여 임베디드 시스템에서의 실시간 적용에는 제약이 따른다 `[18, 26]`.

NeRF와 3DGS의 등장은 SLAM 시스템의 목표 결과물이 변화하고 있음을 시사한다. 이제 목표는 단순히 위치 추정을 위한 기능적 지도를 넘어, 환경의 고품질 '디지털 트윈(digital twin)'을 생성하는 것으로 확장되고 있다. 그러나 이는 최첨단 연구와 실제 임베디드 응용 사이에 '컴퓨팅 격차(compute chasm)'를 만들어냈으며, 이 격차를 해소하는 것이 모델 최적화 및 하드웨어-알고리즘 공동 설계 분야의 주요 연구 동력이 되고 있다.

| 아키텍처            | 기본 표현 방식     | 지도 품질 (밀도/사실감) | 계산 비용 (훈련/추론) | 임베디드 시스템 적합성  | 대표 시스템                    |
| ------------------- | ------------------ | ----------------------- | --------------------- | ----------------------- | ------------------------------ |
| **시맨틱 기하학적** | 특징점, 메시, 복셀 | 낮음-중간               | 낮음                  | 높음 (실시간 가능)      | ORB-SLAM3 기반 시스템, DS-SLAM |
| **시맨틱 NeRF**     | 신경망 (MLP)       | 매우 높음 (사진 수준)   | 매우 높음             | 매우 낮음 (실시간 불가) | iMAP, DNS-SLAM                 |
| **시맨틱 3DGS**     | 3D 가우시안 집합   | 높음 (사진 수준)        | 높음                  | 낮음 (실시간 어려움)    | SGS-SLAM, SemGauss-SLAM        |

## 3.  센서 양상 및 지도 표현 방식 비교 분석

시맨틱 SLAM 시스템의 성능과 특성은 어떤 센서로 데이터를 수집하고, 수집된 정보를 어떤 구조로 지도에 표현하는지에 따라 크게 달라진다. 이 섹션에서는 시맨틱 SLAM에 사용되는 주요 센서들의 원리와 장단점을 비교하고, 의미론적 세계 모델을 구축하는 데 사용되는 다양한 지도 표현 방식들을 추상화 수준에 따라 분석한다. 이를 통해 센서 선택이 지도 구축 방식과 시스템의 전반적인 능력에 미치는 영향을 탐구한다.

### 3.1  입력 센서: 비교 개요

SLAM 시스템의 '눈' 역할을 하는 센서는 환경 정보를 수집하는 첫 단계로, 각 센서는 고유한 특성을 지닌다 `[4, 13]`.

- **단안 카메라(Monocular Camera):** 단일 카메라를 사용한다.
  - **원리:** 움직임을 통해 여러 시점에서 촬영된 2D 이미지들로부터 3D 구조를 복원한다(Structure from Motion, SfM) `[27]`.
  - **장점:** 저렴하고 크기가 작아 어디에나 적용하기 쉽다 `[28]`.
  - **단점:** 단일 이미지로는 절대적인 거리나 크기를 알 수 없는 '척도 모호성(scale ambiguity)' 문제를 가지며, 초기화를 위해 반드시 움직임이 필요하다 `[29]`.
- **스테레오 카메라(Stereo Camera):** 일정한 간격(baseline)으로 배치된 두 개의 카메라를 사용한다.
  - **원리:** 두 카메라에 맺힌 이미지의 차이(disparity)를 이용하여 삼각 측량법으로 깊이를 직접 계산한다 `[29]`.
  - **장점:** 척도 모호성 문제를 해결하고, 움직임 없이도 즉시 깊이 정보를 획득할 수 있다.
  - **단점:** 측정 가능한 깊이 범위가 베이스라인 길이에 의해 제한되며, 시차 계산에 많은 연산량이 필요하고, 두 카메라 간의 정밀한 보정(calibration)이 복잡하다 `[29]`.
- **RGB-D 카메라:** 컬러(RGB) 이미지와 픽셀별 깊이(Depth) 정보를 동시에 제공한다.
  - **원리:** 구조광(structured light)이나 비행시간 측정(Time-of-Flight, ToF)과 같은 능동적인 방식으로 빛을 쏘아 거리를 측정한다 `[29]`.
  - **장점:** 밀집된 깊이 정보를 직접 제공하여 계산 부담을 줄여준다 `[29]`.
  - **단점:** 측정 거리가 짧고, 직사광선과 같은 외부 광원에 취약하며, 투명하거나 빛을 반사하는 표면을 측정하기 어려워 주로 실내 환경에 적합하다 `[29]`.
- **라이다(LiDAR):** 레이저 스캐너를 사용하여 거리를 직접 측정한다.
  - **원리:** 레이저 펄스를 발사하고 반사되어 돌아오는 시간을 측정하여 거리를 계산한다 `[4]`.
  - **장점:** 매우 정밀한 3D 거리 측정이 가능하고, 조명 조건에 거의 영향을 받지 않으며, 측정 범위가 길다 `[30]`.
  - **단점:** 가격이 비싸고, 색상이나 질감 정보가 없는 희소한 포인트 클라우드를 생성하며, 유리와 같은 투명한 표면은 통과해버려 측정에 어려움이 있다 `[31, 32]`.
- **다중 센서 융합(Multi-Sensor Fusion):** 현대의 SLAM 시스템은 단일 센서의 한계를 극복하기 위해 여러 센서를 융합하는 방식을 채택한다. 특히 풍부한 외형과 의미 정보를 제공하는 카메라와, 움직임을 강건하게 추정하는 관성 측정 장치(IMU)를 결합한 **시각-관성 SLAM(Visual-Inertial SLAM, VI-SLAM)**은 업계 표준으로 자리 잡고 있다 `[17, 33]`. 또한, 정확한 깊이 정보를 위해 라이다를 함께 사용하는 융합 방식도 활발히 연구되고 있다 `[32, 34, 35]`. 이러한 융합은 한 센서의 약점(예: 카메라의 조명 의존성)을 다른 센서의 강점(예: 라이다의 조명 불변성)으로 보완하는 상보적 관계를 형성하여 시스템 전체의 강건성을 극대화한다.

### 3.2  시맨틱 지도 표현 방식: 객체에서 장면 그래프까지

수집된 시맨틱 정보는 지도 내에서 다양한 형태로 구조화되어 저장된다. 이러한 지도 표현 방식은 구체적인 기하학적 표현에서부터 추상적인 관계 표현에 이르기까지 계층적인 구조를 보인다. 이는 로봇의 인지 능력이 단순한 객체 라벨링에서 복잡한 공간 추론으로 발전하는 과정을 반영한다.

- **객체 중심 지도(Object-centric Maps):** 지도의 기본 단위를 기하학적 특징점이 아닌 객체 수준의 랜드마크로 구성한다. 객체는 3D 바운딩 박스나 큐브(예: CubeSLAM `[8]`), 또는 이차곡면(quadrics)과 같은 간결한 기하학적 형태로 표현된다(예: QuadricSLAM `[16]`). 이는 포인트 클라우드보다 훨씬 압축적이고 의미론적으로 풍부한 표현 방식이다.
- **밀집 시맨틱 지도(Dense Semantic Maps):** 밀집하게 재구성된 3D 기하학적 지도(포인트 클라우드, 복셀, 메시 등)의 각 구성 요소에 의미론적 레이블을 할당하는 방식이다. 결과물은 시맨틱 포인트 클라우드, 시맨틱 복셀 그리드(예: Semantic OctoMap `[25]`), 또는 각 면이 의미 정보로 채색된 완전한 3D 시맨틱 메시(예: Kimera `[8, 17]`) 형태를 띤다.
- **위상 지도(Topological Maps):** 환경을 그래프 구조로 추상화하여 표현한다. 그래프의 노드는 '방', '복도'와 같은 장소를 나타내고, 엣지는 '문'과 같은 장소 간의 연결성을 나타낸다 `[36]`. 이 방식은 정밀한 기하학적 정보를 생략하는 대신, 고수준의 경로 계획이나 인간과 유사한 방식의 길 안내에 매우 유용하다.
- **장면 그래프(Scene Graphs):** 가장 추상적이고 풍부한 정보를 담는 표현 방식이다. 장면 그래프는 객체, 방, 심지어 추상적인 개념까지 노드로 표현하고, 이들 사이의 복잡한 관계(예: '의자'는 '테이블' *옆에 있다*, '테이블'은 '부엌' *안에 있다*)를 엣지로 표현하는 계층적 그래프다 `[37]`. 이 구조는 로봇이 환경에 대해 깊이 있는 의미론적 추론을 수행할 수 있는 기반을 제공한다. 예를 들어, "컵을 찾아봐"라는 명령에 대해, 컵이 주로 부엌이나 테이블 위에 있다는 사전 지식을 활용하여 탐색 범위를 효율적으로 좁힐 수 있다.

이러한 지도 표현 방식의 계층적 발전은 시맨틱 SLAM의 궁극적인 목표가 단순히 기하학적 지도를 라벨링하는 것을 넘어, 인간의 공간 인지 방식과 유사한 구조화된 관계형 지식 베이스를 구축하는 데 있음을 보여준다.

| 센서 유형           | 원리                                       | 장점                                        | 단점                                       | 주요 응용 분야           |
| ------------------- | ------------------------------------------ | ------------------------------------------- | ------------------------------------------ | ------------------------ |
| **단안 카메라**     | 움직임을 통한 3D 구조 복원 (SfM)           | 저비용, 소형, 범용성                        | 척도 모호성, 초기화 필요                   | 모바일 AR, 저가형 로봇   |
| **스테레오 카메라** | 양안 시차를 이용한 삼각 측량               | 척도 문제 해결, 즉각적 깊이 획득            | 제한된 깊이 범위, 높은 연산량, 복잡한 보정 | 로봇 내비게이션, 드론    |
| **RGB-D 카메라**    | 능동적 광원(구조광/ToF)을 이용한 거리 측정 | 밀집 깊이 정보 직접 제공, 낮은 연산량       | 짧은 측정 거리, 실외 취약성                | 실내 로봇, 3D 스캐닝     |
| **라이다(LiDAR)**   | 레이저 펄스의 비행시간 측정(ToF)           | 고정밀, 조명 불변성, 장거리 측정            | 고비용, 색상/질감 정보 부재                | 자율주행차, 고정밀 매핑  |
| **시각-관성 융합**  | 카메라와 IMU 데이터의 강결합               | 동적 상황 및 특징 부족 환경에서 강건성 확보 | 센서 동기화 및 보정의 복잡성               | 드론, 모바일 로봇, AR/VR |

## 4.  대표적인 시스템 및 구현 사례

이전 섹션들에서 논의된 이론적 개념들은 실제 시스템으로 구현될 때 그 가치와 한계가 명확해진다. 이 섹션에서는 시맨틱 SLAM 분야에 큰 영향을 미친 대표적인 오픈소스 시스템들을 심층적으로 분석한다. 이를 통해 이론이 실제 코드와 아키텍처로 어떻게 구현되는지, 그리고 실제 환경에서 어떤 성능을 보이는지를 구체적으로 살펴본다.

### 4.1  ORB-SLAM 계보: 특징점 기반 SLAM의 초석

ORB-SLAM 계열은 시각 SLAM 분야에서 가장 성공적이고 널리 사용되는 오픈소스 시스템 중 하나로 평가받는다 `[14, 18]`. 이 시스템의 핵심은 이름에서 알 수 있듯이 ORB(Oriented FAST and Rotated BRIEF) 특징점을 사용한다는 점이다. 또한, 추적(Tracking), 지역 지도 작성(Local Mapping), 루프 클로저(Loop Closing)의 세 가지 주요 작업을 병렬로 처리하는 스레드 기반 아키텍처를 채택하여 실시간 성능과 정확성을 동시에 달성했다 `[33, 38]`.

ORB-SLAM2는 단안, 스테레오, RGB-D 카메라를 모두 지원하며 높은 강건성을 보여주었고, 후속 버전인 ORB-SLAM3는 여기에 다중 지도 관리 기능과 시각-관성 센서의 강결합(tightly-integrated) 방식을 추가하여 성능을 한 단계 더 끌어올렸다 `[33]`.

ORB-SLAM이 시맨틱 SLAM 연구의 기반 플랫폼으로 널리 채택된 이유는 명확하다. 첫째, 특징점 기반 방식은 의미론적 정보를 특정 랜드마크(ORB 특징점)에 연관시키기 용이한 구조를 제공한다. 둘째, 모듈화된 오픈소스 코드베이스는 연구자들이 새로운 기능을 추가하거나 기존 모듈을 수정하기에 매우 편리한 환경을 제공한다 `[39]`. 이로 인해 수많은 시맨틱 SLAM 연구들이 ORB-SLAM을 기반으로 파생되었다.

### 4.2  사례 연구: Kimera - 실시간 미터법-시맨틱 SLAM을 위한 모듈형 프레임워크

Kimera는 처음부터 미터법-시맨틱 SLAM을 목표로 설계된 포괄적인 오픈소스 C++ 라이브러리다 `[17, 40]`. ORB-SLAM과 같은 기존 시스템들이 기하학적 재구성에 집중한 것과 달리, Kimera는 밀집 3D 메시 재구성과 의미론적 라벨링을 시스템의 핵심 기능으로 통합했다는 점에서 차별화된다 `[17]`.

Kimera의 가장 큰 특징은 고도로 모듈화된 아키텍처에 있다. 시스템은 네 개의 핵심 모듈로 구성되며, 각 모듈은 독립적으로 실행되거나 조합하여 사용할 수 있다 `[17, 41]`:

- **Kimera-VIO:** 빠르고 정확한 상태 추정을 위한 최첨단 시각-관성 주행계(Visual-Inertial Odometry) 프론트엔드.
- **Kimera-RPGO:** 루프 클로저를 감지하고 이상치를 제거하여 전역적으로 일관된 궤적을 계산하는 강건한 포즈 그래프 최적화 백엔드.
- **Kimera-Mesher:** 장애물 회피와 같은 빠른 응답이 필요한 작업을 위해 경량의 지역 3D 메시를 신속하게 생성하는 모듈.
- **Kimera-Semantics:** 전역적으로 일관되고 밀집하며, 의미론적 정보가 주석으로 달린 최종 3D 메시를 구축하는 모듈.

Kimera의 주요 공헌은 기하학적 정보와 의미론적 정보를 실시간으로 긴밀하게 통합하는 응집력 있는 단일 라이브러리를 제공했다는 점이다 `[17]`. 이를 통해 연구자들은 VIO, SLAM, 3D 재구성 등 다양한 분야에서 자신의 아이디어를 신속하게 프로토타이핑하고 벤치마킹할 수 있는 강력한 플랫폼을 갖게 되었다. 이후 Kimera2로의 발전과 다중 로봇 시스템을 지원하는 Kimera-Multi의 등장은 Kimera의 강건성과 확장성을 다시 한번 입증했다 `[42, 43]`.

### 4.3  사례 연구: 동적 객체 처리를 통한 강건성 강화

시맨틱 정보의 가장 중요한 활용 사례 중 하나는 동적 환경에서의 SLAM 성능을 개선하는 것이다. 전통적인 SLAM은 움직이는 객체 위의 랜드마크를 정적인 것으로 오인하여 추적에 실패하거나 지도를 오염시키는 문제를 겪는다 `[14, 44]`. 시맨틱 SLAM은 이를 해결하기 위한 효과적인 해법을 제시한다.

- **DS-SLAM:** 동적 환경을 위한 시맨틱 SLAM의 선구적인 시스템 중 하나다. DS-SLAM은 ORB-SLAM2에 실시간 시맨틱 분할 네트워크인 SegNet을 결합했다 `[8, 25]`. SegNet은 이미지 내에서 사람, 자동차와 같이 움직일 가능성이 높은 객체들을 픽셀 단위로 식별한다. 이후 시스템은 이 의미론적 정보를 바탕으로 해당 영역에 속한 특징점들을 동적인 것으로 판단하고 위치 추정 및 지도 작성 과정에서 제외한다. 이 간단하면서도 효과적인 접근법을 통해 DS-SLAM은 동적 환경에서 기존 ORB-SLAM2 대비 월등히 높은 정확도를 달성했다 `[25]`.
- **YOLOv8-ORB-SLAM3:** 앞서 설명한 딥러닝 통합 패턴 중 '보조 모듈 추가' 방식의 대표적인 최신 사례다 `[38]`. 이 시스템은 최첨단 객체 탐지 모델인 YOLOv8을 사용하여 이미지 내의 동적 객체(주로 사람)를 빠르고 정확하게 탐지한다. 탐지된 객체의 바운딩 박스 내에 위치한 ORB 특징점들은 불안정한 것으로 간주되어 ORB-SLAM3의 핵심 파이프라인에서 제거된다. 이를 통해 원본 ORB-SLAM3의 뛰어난 성능을 유지하면서도 동적 요소로 인한 오차 발생을 효과적으로 억제할 수 있다 `[38]`.

이러한 시스템들의 발전 과정은 시맨틱 SLAM 분야의 성숙도를 보여준다. 초기에는 YOLO+ORB-SLAM처럼 기존의 강력한 기하학적 코어에 시맨틱 기능을 '추가'하는 방식이 주를 이루었다면, Kimera와 같은 시스템은 아키텍처 설계 단계부터 시맨틱 정보를 '내재적으로 통합'하는 방향으로 나아가고 있다. 이는 의미론적 정보가 더 이상 부가 기능이 아닌, 시스템 설계의 핵심 요소, 즉 '일급 시민(first-class citizen)'으로 자리 잡고 있음을 의미한다.

| 시스템명             | 기반 SLAM 알고리즘      | 시맨틱 방법                  | 핵심 기여 및 특징                                            | 강점                                                   | 한계                                                    |
| -------------------- | ----------------------- | ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------- |
| **DS-SLAM**          | ORB-SLAM2               | SegNet (시맨틱 분할)         | 동적 환경에서의 강건성 향상을 위한 최초의 완전한 시맨틱 SLAM 시스템 중 하나 | 동적 객체 특징점 제거를 통한 정확도 향상               | SegNet의 분할 성능과 속도에 의존                        |
| **Kimera**           | 자체 VIO 및 포즈 그래프 | 딥러닝 기반 2D 시맨틱 레이블 | 기하학, 의미론, 메시 생성을 통합한 모듈형 실시간 프레임워크  | 고도로 통합되고 유연한 아키텍처, 밀집 시맨틱 메시 생성 | 복잡한 시스템 구조, 높은 CPU 자원 사용                  |
| **YOLOv8-ORB-SLAM3** | ORB-SLAM3               | YOLOv8 (객체 탐지)           | 최신 객체 탐지기를 활용하여 기존 SOTA SLAM의 동적 환경 성능을 실용적으로 개선 | 구현이 비교적 간단하고 효과적임                        | 객체 탐지 성능에 의존, 탐지되지 않는 동적 객체에는 취약 |
| **SEO-SLAM**         | 팩터 그래프 기반        | MLLM (멀티모달 LLM)          | 이종 MLLM 에이전트를 비동기적으로 활용하여 개방형 어휘 기반의 시맨틱 레이블링 및 맵 정제 | 유사 객체 구별 능력 탁월, 장면 변화에 적응 가능        | MLLM 추론에 따른 잠재적 지연 시간, 높은 계산 비용       |

## 5.  응용 분야 및 산업적 영향

시맨틱 SLAM 기술은 이론적 탐구를 넘어 다양한 산업 분야에서 실질적인 가치를 창출하며 그 영향력을 확대하고 있다. 자율주행차부터 서비스 로봇, 증강현실에 이르기까지, 기계가 인간의 환경을 더 깊이 이해하게 되면서 이전에는 불가능했던 새로운 응용 서비스들이 가능해지고 있다. 이 섹션에서는 시맨틱 SLAM이 핵심적인 역할을 하는 주요 응용 분야를 살펴보고, 기술의 상업화 현황과 시장 성장성을 분석하여 그 경제적 중요성을 조명한다.

### 5.1  자율주행차: 복잡한 인간 중심 공간에서의 항법

자율주행 기술에서 SLAM은 GPS 신호가 약하거나 없는 도심의 빌딩 숲, 터널, 실내 주차장 등에서 차량의 위치를 정밀하게 추정하는 핵심 기술이다 `[45, 46]`. 전통적인 기하학적 SLAM이 장애물 회피에 중점을 둔다면, 시맨틱 SLAM은 한 걸음 더 나아가 도로 표지판, 차선, 신호등, 보행자, 다른 차량 등을 인식하고 그 의미를 이해함으로써 상황에 맞는 주행 결정을 내릴 수 있게 한다 `[6, 47]`.

특히 **자동 발렛 주차(Automated Valet Parking, AVP)**는 시맨틱 SLAM의 가치가 극대화되는 대표적인 응용 사례다 `[48]`. 주차장은 기둥이나 벽면의 형태가 반복적이고 특징점이 부족하여 전통적인 SLAM이 어려움을 겪는 대표적인 환경이다. 반면, 시맨틱 SLAM은 주차선, 주행 유도 화살표, 주차 가능 공간 표시, 각종 표지판과 같은 풍부한 의미론적 랜드마크를 활용한다. 이를 통해 반복적인 구조 속에서도 차량의 위치를 강건하고 정밀하게 추정하며, 복잡한 주차 기동을 성공적으로 수행할 수 있는 기반을 마련한다 `[48]`.

### 5.2  서비스 로봇과 HRI: '협력적 인지 작업 공간'의 구현

가정, 병원, 물류창고 등에서 활용되는 서비스 로봇이 진정으로 유용해지기 위해서는 인간 및 주변 환경과 의미 있는 방식으로 상호작용할 수 있어야 한다 `[2, 22]`. 단순히 장애물을 피해 움직이는 것을 넘어, 인간의 지시를 이해하고 복잡한 작업을 수행해야 하기 때문이다.

이때 시맨틱 SLAM은 인간과 로봇 간의 **'협력적 인지 작업 공간(collaborative cognitive workspace)'**을 구축하는 토대를 제공한다 `[23, 49]`. 시맨틱 지도는 인간과 로봇이 공유하는 일종의 '공통된 이해의 장' 역할을 한다. 예를 들어, 사용자가 "소파 근처에 엎질러진 것 좀 치워줘"라고 명령하면, 로봇은 자신의 시맨틱 지도에서 '소파'라는 객체의 위치를 인지하고 해당 위치로 이동하여 작업을 수행할 수 있다 `[50]`. 이처럼 시맨틱 맵은 자연어 명령 이해, 상황인지 기반 행동 결정 등 고도화된 인간-로봇 상호작용(Human-Robot Interaction, HRI)을 가능하게 하는 핵심 요소다 `[21, 22]`.

### 5.3  증강현실(AR): 환경 인지 기반의 몰입형 경험 창출

마커(marker) 없이 실제 환경에 가상 객체를 자연스럽게 증강시키는 마커리스 AR의 구현은 미지의 환경에서 사용자 기기의 위치와 자세를 안정적으로 추적하는 SLAM 기술에 크게 의존한다 `[51, 52]`. 그러나 단순히 가상 객체를 현실 공간에 띄우는 것만으로는 진정한 몰입감을 제공하기에 부족하다.

시맨틱 SLAM은 AR 애플리케이션이 주변 환경을 '이해'하게 함으로써 이 문제를 해결한다. 예를 들어, AR 게임 속 가상 캐릭터가 실제 소파 뒤에 숨거나 실제 테이블 위로 점프하는 등 현실 세계의 객체와 상호작용하며 보다 현실감 있는 움직임을 보일 수 있다 `[51]`. 이처럼 시맨틱 정보는 가상과 현실의 상호작용을 한 차원 높은 수준으로 끌어올려, AR을 단순한 시각적 효과를 넘어 강력한 상황인지 컴퓨팅 플랫폼으로 발전시키는 핵심 동력이 된다 `[53]`.

**EnvSLAM**은 이러한 비전을 현실화한 구체적인 사례다. 스마트폰과 같은 모바일 기기에서의 실시간 AR 구동을 목표로 개발된 이 시스템은 정확성과 효율성의 균형을 맞추어, 차세대 AR 앱에 필요한 의미론적 맥락을 제공한다 `[51]`.

### 5.4  시장 분석: 상업화 및 성장 궤도

SLAM 기술 시장은 학술 연구 단계를 넘어 폭발적인 성장을 거듭하며 핵심 기술 산업으로 자리매김하고 있다. 복수의 시장 분석 보고서에 따르면, 전 세계 SLAM 기술 시장은 2023년 약 4억 7,845만 달러에서 2032년 78억 달러 규모로 성장할 것으로 예측되며, 연평균 성장률(CAGR)은 36.43%에 달한다 `[54, 55]`. 또 다른 보고서는 2024년 9억 7,771만 달러에서 2033년 285억 달러로 성장(CAGR 44.3%)할 것으로 전망하는 등, 매우 높은 성장 잠재력을 공통적으로 시사하고 있다 `[56]`.

이러한 성장은 다양한 응용 분야에서의 수요 증가에 기인한다. 2023년 기준, **로보틱스** 분야가 전체 시장의 39.6%를 차지하며 가장 큰 비중을 보였고, 이는 물류 자동화 및 서비스 로봇 시장의 확대와 직결된다 `[54, 55]`. 한편, 향후 가장 높은 성장률이 기대되는 분야는 **AR/VR**로, 몰입형 경험에 대한 수요 증가가 SLAM 기술의 채택을 가속화할 것으로 전망된다 `[54]`.

이러한 시장 성장 속에서 기술 생태계 또한 성숙하고 있다. 구글, 애플, 마이크로소프트, 아마존과 같은 거대 기술 기업들은 자사의 핵심 플랫폼(AR, 로보틱스)에 SLAM 기술을 내재화하기 위해 막대한 투자를 하고 있다 `[54, 57]`. 동시에, SLAM 기술을 전문으로 하는 스타트업들이 등장하여 특정 산업 분야에 최적화된 솔루션을 제공하며 시장을 공략하고 있다.

- **Slamcore:** 임페리얼 칼리지 런던에서 스핀오프한 기업으로, 시맨틱 인식을 핵심 파이프라인에 통합한 비전 기반 공간 AI 소프트웨어(Slamcore SDK) 및 하드웨어 모듈(Slamcore Aware)을 산업 및 로보틱스 분야에 공급한다 `[58, 59, 60, 61, 62, 63]`.
- **Bosch:** 자사의 물류 로봇 ACTIVE Shuttle 등에 SLAM 기술을 적극적으로 활용하며, ROKIT Locator라는 소프트웨어 컴포넌트를 시장에 제공하고 있다 `[64]`.
- **Outsight:** 자율주행차를 겨냥하여 SLAM-on-Chip 알고리즘을 탑재한 '3D 시맨틱 카메라'를 개발하여 공급한다 `[65]`.

이처럼 거대 기업의 플랫폼 통합과 전문 스타트업의 솔루션 제공이라는 이원적 구조는 SLAM 기술이 더 이상 틈새 기술이 아닌, 데이터베이스나 운영체제처럼 기술 스택의 근간을 이루는 핵심 기술로 자리 잡고 있음을 보여준다.

## 6.  핵심 도전 과제와 사회적 고려사항

시맨틱 SLAM 기술이 널리 보급되고 책임감 있게 사용되기 위해서는 해결해야 할 기술적, 윤리적, 사회적 과제들이 산적해 있다. 이 섹션에서는 기술의 강건한 실제 환경 적용을 가로막는 기술적 난제들을 분석하고, 개인의 가장 사적인 공간을 매핑하는 과정에서 발생하는 프라이버시 딜레마를 심도 있게 다룬다. 마지막으로, 단순한 충돌 회피를 넘어 '의미론적 안전성'이라는 새로운 개념을 통해 로봇의 안전 기준을 어떻게 재정립해야 하는지 논한다.

### 6.1  기술적 난제: 강건한 실제 환경 적용을 향한 길

- **계산 복잡성 및 임베디드 시스템의 한계:** 2장에서 논의했듯이, 정교한 시맨틱 분석과 고품질 지도 생성을 위한 최신 알고리즘들은 막대한 계산 자원을 필요로 한다 `[3, 12, 14, 18, 26]`. 이는 자원 제약이 심한 모바일 로봇, 드론, AR 글래스와 같은 임베디드 시스템에 SLAM을 탑재하는 데 가장 큰 걸림돌로 작용한다. 이 문제를 해결하기 위해서는 경량화된 딥러닝 모델 설계, 알고리즘 최적화뿐만 아니라, SLAM 연산을 전담하는 하드웨어 가속기(hardware accelerator) 개발과 같은 **하드웨어-소프트웨어 공동 설계(co-design)** 접근법이 필수적이다 `[18, 66]`.
- **확장성 및 장기간 운영(Long-Term Operation):** 로봇이 며칠, 몇 주, 혹은 그 이상 장기간에 걸쳐 작동할 경우, 지도의 크기는 무한정 커질 수 있다. 이는 메모리 부족과 연산량 증가로 이어져 시스템의 성능을 저하시키는 주요 원인이 된다 `[20, 67]`. '평생 학습 SLAM(Lifelong SLAM)'이라는 연구 분야는 이러한 문제를 해결하기 위해, 불필요한 정보를 효과적으로 잊거나 요약하고, 변화된 부분만 효율적으로 갱신하는 지도 관리 전략을 연구한다.
- **실제 환경의 비예측성에 대한 강건성:** 실험실 환경과 달리 실제 세계는 예측 불가능한 변수로 가득하다. 극심한 조명 변화, 악천후, 그리고 단순한 객체 필터링 수준을 넘어서는 매우 복잡하고 동적인 군중 상황 등은 여전히 SLAM 시스템의 안정성을 위협하는 주요 요인이다 `[68, 69]`. 이러한 극한 상황을 체계적으로 평가하고 알고리즘의 한계를 시험할 수 있는 포괄적이고 도전적인 벤치마크 데이터셋의 부족 또한 기술 발전을 더디게 하는 요인으로 지적된다 `[68]`.

### 6.2  프라이버시 딜레마: 가장 사적인 공간의 지도화

시맨틱 SLAM 기술의 발전은 심각한 프라이버시 문제를 야기한다. 특히 로봇 청소기와 같은 소비자용 로봇은 우리의 가장 사적인 공간인 집 안을 돌아다니며 상세한 지도를 생성한다.

- **가정 내 데이터 수집:** 로봇 청소기는 카메라, 라이다 등의 센서를 이용해 집의 평면도, 가구 배치와 같은 구조적 정보를 포함하는 정밀한 지도를 작성한다 `[70, 71]`. 최신 모델들은 장애물 인식을 위해 카메라로 이미지를 직접 촬영하며, 이 과정에서 가족 구성원의 모습이나 집안의 사적인 장면들이 의도치 않게 포착될 수 있다 `[70]`. 이러한 정보가 클라우드 서버로 전송될 경우, 이는 '가정생활의 디지털 초상화'가 되어 외부 유출의 위험에 노출된다 `[70]`.
- **위험성과 실제 유출 사고:** 가장 큰 위험은 해킹 등을 통한 데이터의 비인가 접근, 데이터 유출, 그리고 기업의 데이터 오남용이다. 2022년에 발생한 한 사건은 이러한 위험을 극명하게 보여주었다. iRobot사의 Roomba 개발 버전이 촬영한 테스트 이미지 중, 화장실에 앉아 있는 여성의 사적인 사진을 포함한 민감한 이미지들이 데이터 주석 작업을 거쳐 소셜 미디어에 유출되는 사건이 발생했다 `[70, 72]`. 비록 상용 제품이 아닌 동의를 받은 테스터의 기기에서 발생한 일이지만, 이 사건은 로봇이 수집한 시각 데이터가 어떻게 통제 불가능한 방식으로 유출될 수 있는지를 보여주며 소비자들에게 큰 경각심을 주었다.
- **기업의 책임과 사용자 대응 방안:** 제조사들은 데이터 암호화, 익명화 등의 조치를 통해 사용자 데이터를 보호하고 책임감 있게 사용한다고 주장한다 `[70]`. 그러나 사용자는 기업의 개인정보 처리 방침을 꼼꼼히 확인하고, 강력한 와이파이 보안 설정(예: WPA3 암호화, IoT 기기 전용 네트워크 분리), 정기적인 소프트웨어 업데이트, 기기에 부착된 센서의 종류와 데이터 수집 범위를 명확히 인지하는 등 스스로 프라이버시를 보호하기 위한 적극적인 노력을 기울일 필요가 있다 `[72]`.

### 6.3  안전성 확보: 충돌 회피를 넘어 시맨틱 안전성으로

전통적인 로봇 안전의 개념은 주로 '충돌 회피', 즉 물리적으로 장애물과 부딪히지 않는 것에 초점을 맞추어 왔다. 이는 로봇 안전의 필요조건이지만, 인간과 함께 생활하고 작업하는 환경에서는 결코 충분조건이 될 수 없다 `[73]`.

- **'시맨틱 안전성(Semantic Safety)'의 개념:** 시맨틱 SLAM이 제공하는 환경 이해 능력은 로봇 안전의 패러다임을 한 단계 발전시킨다. '시맨틱 안전 필터(semantic safety filter)'는 이러한 새로운 안전 개념을 구현하는 프레임워크다 `[74]`. 이 필터는 단순히 기하학적 충돌 가능성뿐만 아니라, 객체의 의미와 상황적 맥락을 고려하여 인간의 '상식'에 부합하는 안전 규칙을 강제한다.
- **시맨틱 제약 조건의 유형 `[74]`:**
  - **공간 관계 제약(Spatial Relationship Constraints):** 객체 간의 위험한 공간적 관계를 금지한다. (예: "물이 든 컵을 노트북 *위로* 이동시키지 말 것")
  - **행동 제약(Behavioral Constraints):** 특정 객체를 다룰 때 로봇의 행동 방식(속도, 가속도 등)을 제한한다. (예: "날카로운 칼을 들고 있을 때는 더 *천천히* 움직일 것")
  - **자세 기반 제약(Pose-based Constraints):** 객체의 특성에 따라 로봇의 끝점(end-effector) 자세를 제한한다. (예: "가득 찬 컵은 항상 *수평을 유지*할 것")

이러한 시맨틱 안전성은 객체의 속성과 잠재적 상호작용에 대한 추론을 기반으로 하므로, 훨씬 더 미묘하고 인간의 안전 기준에 부합하는 로봇 행동을 구현할 수 있게 한다 `[74]`. 시맨틱 SLAM의 성공은 역설적으로 그 기술이 생성하는 지도의 상세함과 풍부함 때문에 새로운 윤리적, 사회적 책임을 동반한다. 지도가 더 정교해질수록, 그 지도를 활용한 혜택과 오용의 가능성 모두 커지기 때문이다. 따라서 '시맨틱 안전 필터'와 같은 기술은 선택 사항이 아닌, 인간 중심 환경에서 작동하는 모든 지능형 로봇 시스템의 필수 구성 요소가 되어야 한다.

## 7.  새로운 지평: 미래 연구 및 개발 동향

시맨틱 SLAM 분야는 인공지능의 다른 분야들과 융합하며 빠르게 발전하고 있다. 이 섹션에서는 현재 연구의 최전선에 있는 네 가지 핵심적인 미래 방향-파운데이션 모델의 통합, 평생 학습 SLAM, 체화된 AI, 그리고 강화 학습 기반의 능동적 SLAM-을 탐구한다. 이를 통해 기술이 나아갈 방향을 예측하고, 차세대 지능형 자율 시스템의 청사진을 제시한다.

### 7.1  파운데이션 모델의 부상: 개방형 세계로의 확장

- **폐쇄된 어휘의 한계:** 현재 대부분의 시맨틱 SLAM 시스템은 PASCAL VOC나 COCO와 같이 미리 정해진 수의 객체 클래스를 가진 데이터셋으로 훈련된다 `[25]`. 이는 시스템이 훈련 데이터에 없었던 새로운 객체를 인식하거나 이해하는 데 명백한 한계를 가진다.
- **파운데이션 모델의 통합:** 이러한 한계를 극복하기 위한 가장 중요한 최신 동향은 비전-언어 모델(Vision-Language Models, VLM)이나 멀티모달 거대 언어 모델(Multimodal Large Language Models, MLLM)과 같은 대규모 사전 훈련된 **파운데이션 모델(Foundation Models)**을 SLAM 파이프라인에 통합하는 것이다 `[18, 75, 76]`.
- **새로운 능력의 발현:** 파운데이션 모델은 '개방형 어휘(open-vocabulary)' 인식 능력을 제공한다. 즉, 시스템은 더 이상 '의자'라는 한정된 레이블에 얽매이지 않고, "꽃무늬 패턴이 있는 빨간 팔걸이 의자"와 같이 풍부하고 서술적인 방식으로 객체를 이해할 수 있게 된다 `[75]`. 또한, 웹 스케일 데이터로부터 학습된 상식 추론(common-sense reasoning) 능력을 SLAM 시스템에 부여한다 `[77]`.
- **사례 연구: SEO-SLAM:** 이 프레임워크는 파운데이션 모델을 SLAM에 정교하게 통합하는 새로운 아키텍처를 제시한다. 단일 거대 모델에 모든 추론을 의존하는 대신, 객체 검증, 레이블 정제, 랜드마크 병합 등 특정 작업에 특화된 여러 개의 MLLM 에이전트를 **비동기적으로(asynchronously)** 운영한다 `[78, 79, 80]`. 이를 통해 실시간 SLAM 파이프라인의 성능 저하를 최소화하면서도 LLM의 강력한 의미 이해 능력을 효율적으로 활용한다. 이는 SLAM 시스템이 실시간 기하학적 추론은 직접 수행하되, 복잡한 의미론적 추론은 강력한 외부 모델에 '아웃소싱'하는 새로운 하이브리드 인지 아키텍처의 등장을 예고한다.
- **사례 연구: LP-SLAM:** 이 시스템은 LLM을 활용하여 장면 내의 텍스트가 랜드마크로 사용될 수 있는지 판단하고, 생성된 지도를 기반으로 사용자에게 자연어 길 안내를 제공하는 등, 언어 이해를 SLAM의 핵심 기능으로 더욱 깊이 통합한 사례다 `[81]`.

### 7.2  평생 학습 SLAM: 변화하는 환경에서의 지속적인 자율성

- **도전 과제:** 현실 세계는 정적이지 않다. 가구의 위치가 바뀌고, 계절이 변하며, 건물이 리모델링된다. 한 번 구축된 지도는 시간이 지나면 낡은 정보가 되어 쓸모없어진다. **평생 학습 SLAM(Lifelong SLAM)**은 환경의 변화에 지속적으로 적응하면서, 매번 처음부터 지도를 다시 만들 필요 없이 기존 지도를 효율적으로 갱신하는 것을 목표로 한다 `[20]`.
- **시맨틱 변화 탐지:** 의미론적 정보는 효율적인 평생 학습의 핵심 열쇠다. 시스템은 단순히 기하학적 변화를 감지하는 것을 넘어, '의자'가 '테이블'로 교체되었다는 것과 같은 **의미론적 변화(semantic change)**를 감지할 수 있다.
- **사례 연구: Lifelong LERF:** 이 시스템은 언어 임베딩 방사 필드(Language Embedded Radiance Field, LERF)라는 밀집 시맨틱 표현을 사용한다. 시스템은 자신이 구축한 지도에서 특정 시점으로 렌더링한 이미지의 언어 임베딩과, 새로 촬영한 실제 이미지의 언어 임베딩을 비교하여 변화가 발생한 영역을 3D 공간상에서 정확히 탐지한다. 그 후, 변화가 감지된 영역의 지도 정보만을 선택적으로 갱신함으로써 전체 지도를 다시 계산하는 비효율을 피한다 `[82]`.

### 7.3  체화된 AI: 지능의 초석으로서의 시맨틱 매핑

- **체화된 AI(Embodied AI)의 정의:** 이 분야는 로봇과 같은 물리적 실체(body)를 가진 에이전트가 환경과의 상호작용을 통해 지능을 학습하고 발전시키는 것을 목표로 한다 `[83]`.
- **시맨틱 지도의 핵심적 역할:** 체화된 에이전트가 "내 열쇠를 찾아서 거실로 가져다줘"와 같이 장기적이고 복잡한 임무를 수행하기 위해서는, 세상에 대한 지속적이고 구조화된 내부 표현(internal representation)이 필수적이다. **시맨틱 지도는 바로 이 '인지 지도(cognitive map)'의 역할을 수행한다** `[20, 21, 84, 85]`. 즉, 지도는 더 이상 항법의 부산물이 아니라, 에이전트의 추론, 계획, 의사결정의 중심에 있는 핵심 데이터 구조가 된다 `[84]`.
- **SLAM에서 시맨틱 매핑으로의 초점 이동:** 체화된 AI 연구, 특히 시뮬레이션 환경에서는 완벽한 위치 정보가 주어진다고 가정하는 경우가 많다. 따라서 연구의 초점은 위치 추정 문제(SLAM)에서, 주어진 임무를 가장 잘 해결할 수 있는 최적의 지도 표현을 구축하는 문제(**시맨틱 매핑**)로 이동한다 `[86]`. "지도의 최적 구조는 무엇인가?", "어떤 정보를 지도에 인코딩해야 하는가?"와 같은 질문이 핵심 연구 주제가 된다 `[21]`.

### 7.4  능동적 SLAM: 지능적 탐색을 위한 강화 학습의 역할

- **탐색 문제(Exploration Problem):** 로봇은 가장 짧은 시간 안에 가장 정확한 지도를 만들기 위해 다음 순간 어디로 이동해야 하는가? 이 문제를 **능동적 SLAM(Active SLAM)**이라고 한다 `[20]`.
- **프론티어 기반에서 학습 기반으로:** 전통적인 탐색 방법은 주로 '프론티어 기반(frontier-based)' 접근법, 즉 알려진 영역과 미지 영역의 경계 중 가장 가까운 곳으로 이동하는 방식을 사용했다 `[87]`. 이는 단기적으로는 합리적이지만, 전체적인 탐색 효율성 측면에서는 근시안적인 결정을 내릴 수 있다.
- **강화 학습(Reinforcement Learning, RL)의 도입:** 최근에는 강화 학습을 통해 지능적인 탐색 정책을 학습하려는 시도가 활발히 이루어지고 있다. 에이전트는 지도의 적용 범위(coverage)를 넓히거나 불확실성을 줄이는 행동에 대해 보상을 받으며, 시행착오를 통해 최적의 탐색 전략을 학습한다 `[87]`.
- **사례 연구: RASLS:** 이 접근법은 강화 학습 정책에 시맨틱 정보를 통합한다. 예를 들어, '사무실 환경에서는 책상과 의자가 함께 발견될 가능성이 높다'와 같은 객체 배치에 대한 사전 지식을 활용하여 보상 함수를 설계한다. 이를 통해 순수 기하학적 정보만으로는 알 수 없는 의미론적 맥락을 바탕으로 더 효율적이고 지능적인 탐색을 수행할 수 있다 `[88]`.

이러한 미래 연구 방향들은 한때 개별적으로 발전해 온 AI 분야들(로보틱스, 컴퓨터 비전, 자연어 처리, 강화 학습)이 '시맨틱 SLAM'이라는 공통의 문제 아래 융합되고 있음을 보여준다. 현대의 시맨틱 SLAM 연구는 더 이상 로보틱스의 하위 분야가 아니라, '에이전트가 어떻게 세상을 인식하고 상호작용하며 내적 모델을 구축하는가'라는 인공 일반 지능(AGI)의 근본적인 질문에 답하기 위한 핵심적인 장이 되고 있다.

## 8.  종합 및 미래 전망

본 보고서는 시맨틱 SLAM 기술의 근원적인 필요성에서부터 핵심 아키텍처, 대표적인 시스템, 다양한 응용 분야, 그리고 미래 연구 방향에 이르기까지 다각적인 고찰을 수행했다. 전통적인 기하학적 SLAM이 '어디에 있는가'의 문제를 해결하며 자율 시스템의 기반을 닦았다면, 시맨틱 SLAM은 '무엇이 있는가'를 이해하게 함으로써 시스템이 환경과 진정으로 상호작용하고 인간과 협력할 수 있는 새로운 차원의 문을 열었다. 분석을 통해 도출된 핵심적인 결론과 향후 연구 및 전략적 개발을 위한 제언은 다음과 같다.

### 8.1  시맨틱 SLAM 현황에 대한 종합적 결론

- **패러다임의 전환: 항법에서 상호작용으로:** 시맨틱 SLAM으로의 발전은 단순한 기술적 개선이 아니라, 로봇 공학의 목표가 'A 지점에서 B 지점으로 이동하는 것(항법)'에서 '환경을 이해하고 그 안에서 유의미한 작업을 수행하는 것(상호작용)'으로 전환되었음을 의미하는 근본적인 패러다임의 변화다.
- **'컴퓨팅 격차'의 존재:** NeRF, 3D 가우시안 스플래팅, 파운데이션 모델과 같은 최첨단 기술들은 놀라운 수준의 지도 품질과 의미 이해 능력을 보여주지만, 막대한 계산 자원을 요구한다. 이로 인해 연구의 최전선과 실제 임베디드 기기에서의 적용 사이에는 상당한 '컴퓨팅 격차'가 존재하며, 이는 기술 상용화의 주요 병목 현상으로 작용하고 있다.
- **지도의 진화: 인지 모델로서의 지도:** 시맨틱 지도의 표현 방식은 객체 중심 지도에서 장면 그래프로 발전하며 점차 추상화되고 있다. 이는 지도가 더 이상 단순한 공간 표현이 아니라, 추론과 계획을 지원하는 인간의 '인지 지도'와 유사한 구조화된 지식 베이스, 즉 '세계 모델'로 진화하고 있음을 시사한다.
- **AI 분야의 융합 허브:** 시맨틱 SLAM은 이제 로보틱스 및 컴퓨터 비전의 독립된 분야를 넘어, 자연어 처리(LLM 통합), 강화 학습(능동적 탐색), 그리고 인공 일반 지능(체화된 AI) 연구가 교차하고 융합되는 핵심적인 허브 역할을 하고 있다. 이는 시맨틱 SLAM이 지능형 에이전트의 근본적인 인식 및 모델링 문제를 다루기 때문이다.

### 8.2  미래 연구 및 전략적 개발을 위한 제언

본 분석을 바탕으로 학계와 산업계가 나아가야 할 방향에 대해 다음과 같이 제언한다.

- **학계를 위한 제언:**
  1. **불확실성 정량화:** 딥러닝 기반의 의미 인식 결과에 내재된 불확실성을 수학적으로 모델링하고 정량화하는 연구에 집중해야 한다. 이는 시스템의 신뢰성과 안전성을 보장하는 데 필수적이다.
  2. **강건한 벤치마크 개발:** 장기간 운영, 극심한 환경 변화, 복잡한 동적 상호작용 등 실제 세계의 어려움을 반영하는 새롭고 도전적인 벤치마크 데이터셋을 구축하고 공유해야 한다.
  3. **효율적인 표현 방식 탐구:** 높은 표현력을 가지면서도 계산적으로 효율적인 새로운 지도 표현 방식(예: 하이브리드, 스파스 텐서 기반 표현)에 대한 기초 연구를 강화해야 한다.
- **산업계를 위한 제언:**
  1. **알고리즘-하드웨어 공동 설계 투자:** '컴퓨팅 격차'를 해소하기 위해 SLAM 및 시맨틱 연산에 특화된 하드웨어 가속기 개발 등 알고리즘과 하드웨어의 공동 설계에 대한 전략적 투자가 필요하다.
  2. **프라이버시 및 안전성 중심 설계:** 기술 개발 초기 단계부터 프라이버시 보호 기술(Privacy by Design)과 시맨틱 안전성 프레임워크를 핵심 설계 원칙으로 채택하여 사회적 신뢰를 확보해야 한다.
  3. **개발자 생태계 구축:** SLAM의 복잡성을 추상화하여 다양한 응용 분야의 개발자들이 쉽게 공간 지능을 자신의 제품에 통합할 수 있도록 지원하는 고수준의 SDK(소프트웨어 개발 키트) 및 플랫폼 개발에 주력해야 한다.

궁극적으로 시맨틱 SLAM 기술이 지향하는 목표는 우리가 살아가는 환경에 대한 지속적이고, 질의 가능하며, 평생 학습하는 '시맨틱 디지털 트윈'을 구축하는 것이다. 이러한 디지털 트윈은 차세대 자율 로봇, 지능형 AR, 그리고 스마트 환경을 구현하는 핵심적인 인프라가 될 것이며, 이를 향한 학계와 산업계의 협력적 노력이 미래 기술의 지형을 결정할 것이다.

#### **참고 자료**

1. SLAM-past-present-future.pdf - cs.wisc.edu, accessed July 18, 2025, https://pages.cs.wisc.edu/~jphanna/teaching/25spring_cs639/resources/SLAM-past-present-future.pdf
2. SLAM 기술: 공간 지능의 핵심 동력, accessed July 18, 2025, https://brunch.co.kr/@donghyungshin/148
3. Semantic Visual Simultaneous Localization and Mapping: A Survey - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/391793050_Semantic_Visual_Simultaneous_Localization_and_Mapping_A_Survey
4. Lidar Slam And Visual Slam Complete Introduction and Comparison, accessed July 18, 2025, https://www.altversebot.com/blog/2024/11/21/lidar-slam-and-visual-slam-complete-introduction-and-comparison/
5. The Types of SLAM Algorithms - Medium, accessed July 18, 2025, https://medium.com/@nahmed3536/the-types-of-slam-algorithms-356196937e3d
6. A Review of SLAM Techniques and Security in Autonomous Driving - Advanced Robotics and Automation (ARA) Laboratory - University of Nevada, Reno, accessed July 18, 2025, https://ara.cse.unr.edu/wp-content/uploads/2014/12/Singandhupe_La_IRC18.pdf
7. A comprehensive survey of advanced SLAM techniques - E3S Web of Conferences, accessed July 18, 2025, https://www.e3s-conferences.org/articles/e3sconf/pdf/2024/71/e3sconf_wfces2024_05004.pdf
8. Semantic SLAM - velog, accessed July 18, 2025, https://velog.io/@thkweon/Semantic-SLAM
9. Unifying Geometry, Semantics, and Data Association in SLAM - Existential Robotics Laboratory, accessed July 18, 2025, https://existentialrobotics.org/pages/semantic-slam.html
10. slambook-ko/ch14.md at master - GitHub, accessed July 18, 2025, https://github.com/slam-research-group-kr/slambook-ko/blob/master/ch14.md
11. An Overview on Visual SLAM: From Tradition to Semantic - MDPI, accessed July 18, 2025, https://www.mdpi.com/2072-4292/14/13/3010
12. Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey - arXiv, accessed July 18, 2025, https://arxiv.org/html/2505.12384v1
13. Semantic Visual Simultaneous Localization and Mapping: A Survey - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/2209.06428
14. A Comparative Review on Enhancing Visual Simultaneous Localization and Mapping with Deep Semantic Segmentation - PubMed Central, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11174785/
15. Semantic Visual Simultaneous Localization and Mapping: A Survey - Bohrium, accessed July 18, 2025, https://www.bohrium.com/paper-details/semantic-visual-simultaneous-localization-and-mapping-a-survey/1136593196445335585-3813
16. SemanticSLAM.ai, accessed July 18, 2025, http://www.semanticslam.ai/
17. Kimera: an Open-Source Library for Real-Time Metric ... - MIT, accessed July 18, 2025, https://www.mit.edu/~arosinol/papers/Rosinol20icra-Kimera.pdf
18. [Literature Review] Is Semantic SLAM Ready for Embedded ..., accessed July 18, 2025, https://www.themoonlight.io/en/review/is-semantic-slam-ready-for-embedded-systems-a-comparative-survey
19. Enhancing Mobile Robot Navigation Through Semantic SLAM Integration - Frontiers, accessed July 18, 2025, https://www.frontiersin.org/research-topics/50985/enhancing-mobile-robot-navigation-through-semantic-slam-integration
20. Semantic Mapping in Indoor Embodied AI -- A Comprehensive Survey and Future Directions | Request PDF - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/387954358_Semantic_Mapping_in_Indoor_Embodied_AI_--_A_Comprehensive_Survey_and_Future_Directions
21. Semantic Mapping in Indoor Embodied AI – A Comprehensive Survey and Future Directions, accessed July 18, 2025, https://arxiv.org/html/2501.05750v1
22. An Object-oriented Navigation Strategy for Service Robots Leveraging Semantic Information, accessed July 18, 2025, https://tohoku.elsevierpure.com/en/publications/an-object-oriented-navigation-strategy-for-service-robots-leverag
23. Semantic SLAM for Collaborative Cognitive Workspaces - AAAI, accessed July 18, 2025, https://aaai.org/papers/0013-fs04-05-013-semantic-slam-for-collaborative-cognitive-workspaces/
24. Deep Learning for Visual SLAM: The State-of-the-Art and Future ..., accessed July 18, 2025, https://www.mdpi.com/2079-9292/12/9/2006
25. DS-SLAM: A Semantic Visual SLAM towards Dynamic Environments - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/1809.08379
26. [2505.12384] Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey, accessed July 18, 2025, https://arxiv.org/abs/2505.12384
27. [논문]SLAM 기술의 과거와 현재, accessed July 18, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO201411560021024
28. Semantic visual simultaneous localization and mapping (SLAM) using deep learning for dynamic scenes - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10588701/
29. Five Things You Need to Know About SLAM Simultaneous ... - BasicAI, accessed July 18, 2025, https://www.basic.ai/blog-post/slam-simultaneous-localization-and-mapping
30. What's the difference between vision-based and LiDAR-based SLAM?, accessed July 18, 2025, https://www.exyn.com/news/vision-vs-lidar-slam
31. Are there any advantages to using a LIDAR for SLAM vs a standard RGB camera?, accessed July 18, 2025, https://robotics.stackexchange.com/questions/10695/are-there-any-advantages-to-using-a-lidar-for-slam-vs-a-standard-rgb-camera
32. SLAM and 3D Semantic Reconstruction Based on the Fusion of ..., accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9920633/
33. ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/343179441_ORB-SLAM3_An_Accurate_Open-Source_Library_for_Visual_Visual-Inertial_and_Multi-Map_SLAM
34. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11902412/
35. Visual SLAM: What Are the Current Trends and What to Expect? - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9735432/
36. Learning Semantic Maps from Natural Language ... - Robotics, accessed July 18, 2025, https://www.roboticsproceedings.org/rss09/p04.pdf
37. Scene Understanding, Semantic SLAM, Implicit ... - Niko Sünderhauf, accessed July 18, 2025, https://nikosuenderhauf.github.io/projects/sceneunderstanding/
38. YOLOv8-ORB-SLAM3: Semantic SLAM with dynamic ... - GitHub, accessed July 18, 2025, https://github.com/Glencsa/YOLOv8-ORB-SLAM3
39. Semantic visual simultaneous localization and mapping (SLAM) using deep learning for dynamic scenes - PeerJ, accessed July 18, 2025, https://peerj.com/articles/cs-1628/
40. [1910.02490] Kimera: an Open-Source Library for Real-Time Metric-Semantic Localization and Mapping - arXiv, accessed July 18, 2025, https://arxiv.org/abs/1910.02490
41. MIT-SPARK/Kimera: Index repo for Kimera code - GitHub, accessed July 18, 2025, https://github.com/MIT-SPARK/Kimera
42. [2401.06323] Kimera2: Robust and Accurate Metric-Semantic SLAM in the Real World, accessed July 18, 2025, https://arxiv.org/abs/2401.06323
43. Kimera-Multi: Robust, Distributed, Dense Metric-Semantic SLAM for Multi-Robot Systems, accessed July 18, 2025, https://mit.edu/sparklab/2023/08/25/Kimera-Multi__Robust_Distributed_Dense_Metric-Semantic_SLAM_for_Multi-Robot-Systems.html
44. Semantic SLAM Based on Deep Learning in Endocavity Environment - MDPI, accessed July 18, 2025, https://www.mdpi.com/2073-8994/14/3/614
45. (PDF) Visual SLAM Methods for Autonomous Driving Vehicles - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/383230620_Visual_SLAM_Methods_for_Autonomous_Driving_Vehicles
46. Simultaneous Localization and Mapping (SLAM) for Autonomous Driving: Concept and Analysis - MDPI, accessed July 18, 2025, https://www.mdpi.com/2072-4292/15/4/1156
47. Semantic Scene Understanding in Robotics - The ERC Blog, accessed July 18, 2025, https://erc-bpgc.github.io/blog/blog/semantic_scene/
48. Semantic Visual-inertial SLAM for Automated ... - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/ACCV2024/papers/Oh_Semantic_Visual-inertial_SLAM_for_Automated_Valet_Parking_ACCV_2024_paper.pdf
49. Semantic SLAM for Collaborative Cognitive Workspaces - Frank Dellaert, accessed July 18, 2025, https://dellaert.github.io/files/Dellaert04ss.pdf
50. Collaborative Mobile Robotics for Semantic Mapping: A Survey - MDPI, accessed July 18, 2025, https://www.mdpi.com/2076-3417/12/20/10316
51. EnvSLAM: Combining SLAM Systems and Neural Networks to ..., accessed July 18, 2025, https://www.mdpi.com/2220-9964/10/11/772
52. Simultaneous Localization and Mapping for Augmented Reality (PDF) - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/232638940_Simultaneous_Localization_and_Mapping_for_Augmented_Reality_PDF
53. SLAM: Enabling Algorithms for Robotics and Augmented Reality - Impact case study : Results and submissions : REF 2021, accessed July 18, 2025, https://results2021.ref.ac.uk/impact/1a6f38f0-ec7f-47aa-b662-7d8805cf9d37?page=1
54. Simultaneous Localization & Mapping Market Size, Growth 2032, accessed July 18, 2025, https://www.snsinsider.com/reports/simultaneous-localization-and-mapping-market-6161
55. Simultaneous Localization and Mapping (SLAM) Market to Hit USD 7811.04 Billion by 2032, at a CAGR of 36.43% | SNS Insider - GlobeNewswire, accessed July 18, 2025, https://www.globenewswire.com/news-release/2025/04/10/3059347/0/en/Simultaneous-Localization-and-Mapping-SLAM-Market-to-Hit-USD-7811-04-Billion-by-2032-at-a-CAGR-of-36-43-SNS-Insider.html
56. SLAM Technology Market Size, Share, trends 2033, accessed July 18, 2025, https://www.marketgrowthreports.com/market-reports/slam-technology-maket-100047
57. Simultaneous Localization and Mapping (SLAM) Technology Market Share - MarkNtel, accessed July 18, 2025, https://www.marknteladvisors.com/research-library/slam-technology-market.html
58. SLAM Technology | V-SLAM Robotics & Software | Slamcore, accessed July 18, 2025, https://www.slamcore.com/technology/
59. Slamcore – Vision-Based SLAM for Automation & AI Navigation - Flowcate, accessed July 18, 2025, https://flowcate.com/slamcore-vision-based-slam/
60. SLAMcore – Spatial AI Localisation & Mapping - Amadeus Capital Partners, accessed July 18, 2025, https://www.amadeuscapital.com/company/slamcore/
61. About Us - Slamcore, accessed July 18, 2025, https://www.slamcore.com/about/
62. Slamcore | Unlock the potential of RTLS with vision, accessed July 18, 2025, https://www.slamcore.com/
63. Pick the solution that fits your fleet - Slamcore, accessed July 18, 2025, https://www.slamcore.com/products/
64. Simultaneous localization and mapping - Bosch Research, accessed July 18, 2025, https://www.bosch.com/stories/simultaneous-localization-and-mapping/
65. 5 Top Simultaneous Localization & Mapping Startups - StartUs Insights, accessed July 18, 2025, https://www.startus-insights.com/innovators-guide/5-top-simultaneous-localization-and-mapping-startups/
66. Hardware Acceleration for SLAM in Mobile Systems - SciOpen, accessed July 18, 2025, https://www.sciopen.com/article/10.1007/s11390-021-1523-5
67. Automatic Memory Management in ORB SLAM-3, accessed July 18, 2025, https://cse.buffalo.edu/tech-reports/2024-01.pdf
68. Challenges of Indoor SLAM: A multi-modal multi-floor dataset for SLAM evaluation - arXiv, accessed July 18, 2025, https://arxiv.org/html/2306.08522
69. Review: Issues and Challenges of Simultaneous Localization and Mapping (SLAM) Technology in Autonomous Robot - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/369646388_Review_Issues_and_Challenges_of_Simultaneous_Localization_and_Mapping_SLAM_Technology_in_Autonomous_Robot
70. Are Robot Vacuums Spying on You? A Deep Dive into Privacy ..., accessed July 18, 2025, https://vacuumwars.com/are-robot-vacuums-spying-on-you/
71. A Deep Look Into Privacy and Security Of Vacuum Robot - Cal Poly Pomona, accessed July 18, 2025, https://www.cpp.edu/cyberfair/poster-information/documents/vacuum-robot-abstract.pdf
72. Robot Vacuum Privacy Concerns, Explained - Ecovacs, accessed July 18, 2025, https://www.ecovacs.com/us/blog/robot-vacuum-privacy-concerns
73. Scene Understanding by Reasoning Stability and Safety - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/272682689_Scene_Understanding_by_Reasoning_Stability_and_Safety
74. Semantically Safe Robot Manipulation: From ... - OpenReview, accessed July 18, 2025, https://openreview.net/pdf/383695475801189fa3ad4a80f07f6abf51a49585.pdf
75. Learning from Feedback: Semantic Enhancement for Object SLAM Using Foundation Models - arXiv, accessed July 18, 2025, https://arxiv.org/html/2411.06752v1
76. Learning from Feedback: Semantic Enhancement for Object SLAM Using Foundation Models - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/385721155_Learning_from_Feedback_Semantic_Enhancement_for_Object_SLAM_Using_Foundation_Models
77. Semantics in Large-language models | by Dr Vaishak Belle - Medium, accessed July 18, 2025, https://medium.com/@vaishakbelle/semantics-in-large-language-models-aa71e6f9f4c9
78. Semantic Enhancement for Object SLAM with Heterogeneous Multimodal Large Language Model Agents - arXiv, accessed July 18, 2025, https://arxiv.org/html/2411.06752v2
79. SEO-SLAM: Semantic Enhancement for Object SLAM with Heterogeneous Multimodal Large Language Model Agents - Jungseok Hong, accessed July 18, 2025, https://jungseokhong.com/SEO-SLAM/
80. [2411.06752] Semantic Enhancement for Object SLAM with Heterogeneous Multimodal Large Language Model Agents - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2411.06752
81. LP-SLAM: Language-Perceptive RGB-D SLAM system based on ..., accessed July 18, 2025, https://paperswithcode.com/paper/lp-slam-language-perceptive-rgb-d-slam-system
82. Lifelong LERF: Local 3D Semantic Inventory Monitoring Using FogROS2 - arXiv, accessed July 18, 2025, https://arxiv.org/html/2403.10494v1
83. Embodied AI Workshop, accessed July 18, 2025, https://embodied-ai.org/
84. Semantic Mapping in Indoor Embodied AI – A Survey on Advances, Challenges, and Future Directions - arXiv, accessed July 18, 2025, https://arxiv.org/html/2501.05750v2
85. [2501.05750] Semantic Mapping in Indoor Embodied AI -- A Survey on Advances, Challenges, and Future Directions - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2501.05750
86. Semantic Mapping in Indoor Embodied AI - A Survey on Advances, Challenges, and Future Directions | OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=USgQ38RG6G
87. REINFORCEMENT LEARNING HELPS SLAM: LEARNING TO BUILD MAPS - Semantic Scholar, accessed July 18, 2025, https://pdfs.semanticscholar.org/4d1d/c03b7ad8573ba5ec2ff72ee43606e93bcd4d.pdf
88. RASLS: Reinforcement Learning Active SLAM Approach with Layout ..., accessed July 18, 2025, https://www.researchgate.net/publication/383890797_RASLS_Reinforcement_Learning_Active_SLAM_Approach_with_Layout_Semantic
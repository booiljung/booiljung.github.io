# NetVLAD와 시각 항법 기술에 대한 인간을 위한 안내서
[딥러닝 기반 장소 인식 (Deep Learning-based Place Recognition)](./index.md)


자율 주행 자동차, 배송 드론, 서비스 로봇이 우리 삶에 스며들면서 이들이 스스로 길을 찾는 능력, 즉 자율 항법은 더 이상 공상 과학의 영역이 아닙니다. 이러한 자율 시스템의 핵심에는 주변 세계를 이해하는 능력이 있으며, 그중에서도 '시각'은 가장 풍부하고 강력한 감각 중 하나입니다. 인간이 낯선 도시를 탐험할 때 눈에 의존하듯, 로봇 역시 카메라를 통해 세상을 인식하고 자신의 위치를 파악할 수 있습니다.1

본 보고서는 로봇이 '보는' 행위의 기본 원리에서부터, 이전에 방문했던 장소를 기억하게 만드는 정교한 딥러닝 기술에 이르기까지의 여정을 안내합니다. 이 여정의 중심에는 시각적 장소 인식(Visual Place Recognition) 분야의 기념비적인 기술인 NetVLAD가 있습니다. 이 안내서를 통해 우리는 로봇이 단순한 픽셀의 나열을 넘어 의미 있는 '장소'로 인식하고, 이를 통해 복잡한 세상 속에서 자신의 길을 찾아가는 놀라운 과정을 탐험하게 될 것입니다.


NetVLAD를 이해하기에 앞서, 우리는 먼저 그것이 해결하고자 하는 더 넓은 문제의 장, 즉 시각 기반 항법(Vision-Based Navigation)의 세계를 이해해야 합니다. 왜 이 기술이 필요하며, 어떤 어려움이 따르는지를 아는 것이 모든 논의의 출발점입니다.


시각 기반 항법의 핵심 목표는 로봇이나 차량(UAV, 자율주행차 등)이 카메라와 같은 시각 센서를 이용해 주변 환경을 이해하고 자신의 위치를 결정하도록 하는 것입니다.2 이 과정은 크게 세 가지 필수적인 절차로 구성됩니다.2

1. **위치 추정(Localization):** "나는 지금 어디에 있는가?"라는 근본적인 질문에 답하는 과정입니다.
2. **지도 작성(Mapping):** "내 주변 세계는 어떻게 생겼는가?"를 파악하며 환경의 지도를 구축하는 과정입니다.
3. **경로 계획(Path Planning):** "A 지점에서 B 지점까지 어떻게 안전하게 갈 수 있는가?"를 계산하는 과정입니다.

이는 마치 사람이 낯선 건물에 들어가 탐험하는 과정과 유사합니다. 우리는 끊임없이 주변을 둘러보며 머릿속에 지도를 그리고(Mapping), 그 지도 안에서 자신의 현재 위치를 파악하며(Localization), 출구를 향해 나아갑니다(Path Planning).5


자율 시스템은 주로 세 가지 핵심 센서를 통해 세상을 인식합니다: 카메라(시각), GPS(위성), 그리고 LiDAR(레이저)입니다.2 각 센서는 고유한 장단점을 가지며, 이는 시각 기반 시스템이 왜 중요한 틈새를 차지하는지를 설명해 줍니다.

- **카메라(시각):** 색상, 질감 등 풍부하고 밀도 높은 정보를 제공하며, 비용이 저렴하고 수동적인 센서(스스로 신호를 방출하지 않음)라는 장점이 있습니다.2 하지만 조명 변화(낮과 밤, 눈부심), 날씨에 취약하며, 단일 카메라(모노큘러)로는 깊이를 직접 측정할 수 없는 한계가 있습니다.3 사용되는 카메라 종류로는 인간의 양안 시각을 모방해 깊이를 인지하는 스테레오 카메라, 색상과 깊이 정보를 함께 제공하는 RGB-D 카메라, 넓은 시야각을 가진 어안 카메라 등이 있습니다.2
- **GPS:** 전 지구적 범위를 포괄하지만 실내, 빌딩 숲(도심 협곡), 지하 또는 신호 방해 시 작동하지 않는 치명적인 단점이 있습니다.3
- **LiDAR:** 레이저를 이용해 매우 정밀한 3차원 점 구름(point cloud)을 제공하지만, 가격이 비싸고 악천후에 성능이 저하될 수 있으며, 색상이나 질감 정보가 부족합니다.5

이러한 특성 때문에 특히 GPS가 작동하지 않는 환경에서 시각 기반 시스템의 중요성이 부각됩니다. 센서의 선택은 "어느 것이 최고인가"의 문제가 아니라 "주어진 임무와 환경에 어느 것이 최적인가"의 문제입니다. 최근 연구는 각 센서의 약점을 보완하기 위해 여러 센서를 함께 사용하는 센서 퓨전(Sensor Fusion)으로 나아가고 있지만 10, 동시에 카메라만으로 항법을 수행하려는 독립적인 연구 흐름도 매우 활발합니다. 이는 시각이 단순히 '저렴한 대안'이 아니라, 근본적으로 다르고 강력한 감각 양식임을 시사합니다. 시각이 마주한 조명과 시점 변화라는 도전 과제들이 바로 NetVLAD와 같은 혁신적인 알고리즘을 탄생시킨 원동력입니다.

| 표 1: 주요 항법 센서 비교 |                        |                             |                               |                                     |
| ------------------------- | ---------------------- | --------------------------- | ----------------------------- | ----------------------------------- |
| **센서**                  | **감지 원리**          | **주요 데이터**             | **핵심 강점**                 | **핵심 약점**                       |
| **카메라 (시각)**         | 수동적 빛 수집         | 2D 이미지 픽셀 (색상, 질감) | 풍부한 데이터, 저비용, 수동적 | 조명/날씨 의존적, 스케일 모호성     |
| **GPS**                   | 위성 삼각 측량         | 전 지구적 좌표              | 전 지구적 커버리지, 절대 위치 | 실내/지하 불가, 신호 취약성         |
| **LiDAR**                 | 능동적 빛 방출 및 반사 | 3D 점 구름                  | 높은 정밀도, 직접적인 3D 측정 | 고비용, 악천후 영향, 색상 정보 부재 |


SLAM(Simultaneous Localization and Mapping, 동시적 위치 추정 및 지도 작성)은 로봇 공학의 고전적인 '닭과 달걀의 문제'로 비유됩니다.5 지도를 만들려면 현재 위치를 알아야 하고, 현재 위치를 알려면 지도가 필요합니다. SLAM은 이 두 가지 문제를 동시에 해결하는 기술입니다.

특히 카메라를 사용하는 SLAM을 Visual SLAM(vSLAM)이라고 부릅니다.2 로봇은 연속적인 이미지를 촬영하고, 이미지 간의 변화를 분석하여 주변 환경의 지도(주로 특징점들로 구성된 3D 점 구름)를 생성함과 동시에 그 지도 안에서 자신의 움직임을 추적합니다.5


이제 논의의 초점을 일반적인 항법에서 NetVLAD가 해결하기 위해 설계된 특정 문제, 즉 시각적 장소 인식(Visual Place Recognition, VPR)으로 좁혀보겠습니다. 이는 다음 장에서 다룰 기술적인 '어떻게'에 대한 '왜'를 설명하는 핵심적인 부분입니다.


VPR은 로봇이 현재 보고 있는 장면이 이전에 방문했던 장소와 일치하는지를 오직 시각 정보만을 사용해 정확하게 알아내는 기술입니다.13 이는 본질적으로 '이미지 검색' 문제로 볼 수 있습니다. 즉, 질의(query) 이미지가 주어졌을 때, 이전에 촬영된 위치 정보가 태그된 방대한 데이터베이스에서 가장 일치하는 이미지를 찾아내는 것입니다.16

VPR의 가장 중요한 로봇 공학 응용 분야는 **SLAM에서의 루프 폐쇄(Loop Closure)**입니다. 로봇이 이전에 매핑했던 지역으로 돌아왔을 때, VPR은 그 '루프'를 인식하게 해줍니다. 이 인식은 그동안 누적된 지도상의 오차(drift)를 수정하는 강력한 제약 조건을 제공하여, 전역적으로 일관되고 정확한 지도를 완성하게 합니다.13


VPR이 어려운 이유는 두 가지 핵심적인 도전 과제 때문입니다.

- **외형 변화(Appearance Change):** 이는 *같은 장소*가 다르게 보이는 문제입니다. 낮과 밤, 맑은 날과 비 오는 날, 여름과 겨울과 같은 극적인 환경 변화나, 건물을 정면에서 볼 때와 측면에서 볼 때 같은 시점 변화가 대표적인 예입니다.13 이러한 변화는 원본 픽셀 데이터를 완전히 바꾸어 버리기 때문에 단순한 이미지 비교로는 동일한 장소임을 알아내기 어렵습니다. 따라서 VPR의 목표는 이러한 변화에 **불변(invariant)**한 표현을 찾는 것입니다.14
- **지각적 모호성(Perceptual Aliasing):** 이는 *다른 장소*가 똑같이 보이는 더 교묘한 문제입니다. 사무실 건물의 똑같이 생긴 복도들, 고속도로의 반복적인 구간, 도시의 비슷하게 생긴 상점 정면 등이 좋은 예입니다.15 이는 SLAM에서 치명적인 실패를 유발하는 주요 원인으로, 잘못된 루프 폐쇄는 전체 지도를 망가뜨릴 수 있습니다.18 따라서 VPR의 또 다른 목표는 장소를 매우 **구별 가능하게(discriminative)** 표현하는 것입니다.

결국 VPR의 딜레마는 외형 변화에 대해서는 불변성을 유지하면서도, 지각적 모호성을 극복할 만큼 충분히 구별 가능한 이미지 표현을 만드는 데 있습니다. 이는 단순히 기술적 장애물을 넘는 것이 아니라, '장소'라는 개념을 추상화하는 근본적인 문제입니다. 즉, 피상적인 변화에도 불구하고 장소를 독특하고 인식 가능하게 만드는 인간의 직관과 일치하는 수학적 표현을 만드는 것이 핵심 과제입니다. 이는 곧 장소를 나타내는 완벽한 '지문'을 찾는 과정이며, 다음 장에서 다룰 이미지 기술자(descriptor)의 진화 과정에 대한 완벽한 배경이 됩니다.


단일 프레임이 아닌 연속된 이미지 시퀀스를 사용하면 VPR의 강인성을 크게 향상시킬 수 있습니다.15 단일 이미지는 지각적 모호성으로 인해 혼란을 줄 수 있지만, 그 지점에 이르기까지의 시각적 흐름, 즉 시퀀스는 고유할 가능성이 높습니다. 이러한 시간적 맥락은 어려운 장면을 구별하는 데 도움을 줍니다.15 고전적인 SeqSLAM이 이러한 접근법의 대표적인 예입니다.23 NetVLAD는 주로 단일 이미지를 처리하지만, VPR 분야를 전체적으로 이해하기 위해 이 개념을 아는 것은 중요합니다.


이 장에서는 NetVLAD 이전의 기술들을 살펴봄으로써, 아이디어의 논리적 흐름과 NetVLAD가 극복하고자 했던 한계를 명확히 보여주는 중요한 배경 지식을 제공합니다. SIFT에서 VLAD에 이르는 진화 과정은 **정밀도와 확장성** 사이의 근본적인 타협점을 찾는 여정이었습니다. 이 과정은 매우 정밀하지만 확장 불가능한 지역 정보에서 시작하여, 그 정보를 점점 더 정교하게 요약하는 전역적인 '요약본'으로 나아가는 과정으로 이해할 수 있습니다.


초기 접근법은 SIFT(Scale-Invariant Feature Transform)나 SURF와 같은 지역 특징점 기술자(local feature descriptor)를 사용했습니다.14 이 알고리즘들은 이미지에서 눈에 띄는 핵심점(코너, 독특한 질감의 조각 등)을 찾고, 이를 회전이나 크기 변화에 강인한 고차원 벡터로 기술합니다.24 하지만 단일 이미지는 수천 개의 이런 기술자를 가질 수 있어, 이를 대규모 데이터베이스와 일일이 비교하는 것은 계산적으로나 메모리 측면에서 비효율적이었습니다.25 이는 기술자들을 하나의 표현으로 '통합'해야 할 필요성을 낳았습니다.


BoVW(Bag of Visual Words) 모델은 텍스트 문서 분석에서 강력한 유추를 가져와 이 문제를 해결하려 했습니다.27

1. **'시각적 어휘' 생성:** 먼저, 훈련 데이터셋에서 추출한 방대한 양의 지역 기술자(예: SIFT)들을 k-평균(k-means) 같은 클러스터링 알고리즘으로 묶어, 예를 들어 1,000개의 대표적인 '시각적 단어'(클러스터의 중심점)로 구성된 어휘집을 만듭니다.28
2. **히스토그램 생성:** 새로운 이미지가 주어지면, 그 이미지의 지역 기술자들을 추출하여 어휘집에서 가장 가까운 '시각적 단어'에 할당합니다. 그리고 이 단어들의 빈도를 세어 히스토그램을 만듭니다.27 이 히스토그램이 바로 이미지의 최종적인 고정 크기 기술자가 됩니다.

BoVW는 압축성 면에서 큰 진전을 이루었지만, 매우 거친 표현 방식이었습니다. 어떤 시각적 단어가 몇 번 나타났는지만 셀 뿐, 특징들의 분포나 외형에 대한 중요한 정보를 버립니다. 이는 마치 어떤 문서에 '달리다'라는 단어가 10번 나온다는 사실은 알지만, 그 문서가 스포츠에 관한 것인지 정치에 관한 것인지 알 수 없는 것과 같습니다.31


VLAD(Vector of Locally Aggregated Descriptors)는 BoVW의 정보 손실과 양자화 오류를 극복하기 위해 제안된 직접적인 개선안입니다.33 핵심적인 차이점은 각 클러스터(시각적 단어)에 몇 개의 기술자가 속하는지를 단순히 세는 대신, VLAD는 기술자들과 할당된 클러스터 중심점 사이의 

**차이(잔차, residual)의 합**을 계산한다는 것입니다.30

이를 비유하자면, BoVW가 단순한 득표수 집계라면, VLAD는 각 후보에 대한 득표수를 셀 뿐만 아니라 각 그룹의 유권자들이 그룹의 평균적인 입장과 얼마나 다른지를 합산하여 그들의 의견 강도까지 측정하는 여론조사와 같습니다. 이 과정은 특징 분포에 대한 고차원 통계 정보를 포착하여, 더 작은 어휘집 크기로도 훨씬 더 구별력 있는 기술자를 만들어냈습니다.31

하지만 VLAD에도 한계는 남았습니다. 기술자를 *단 하나*의 가장 가까운 클러스터에 할당하는 '강성 할당(hard assignment)' 방식은 불연속적이고 미분 불가능한 연산입니다. 이는 딥러닝 네트워크에 통합하여 종단간(end-to-end) 훈련을 할 수 없다는 치명적인 문제를 야기했습니다. 이것이 바로 NetVLAD가 해결한 핵심 문제입니다.

| 표 2: VPR을 위한 이미지 통합 기술의 진화 |                                         |                  |                      |                        |
| ---------------------------------------- | --------------------------------------- | ---------------- | -------------------- | ---------------------- |
| **기술**                                 | **핵심 아이디어**                       | **표현**         | **핵심 강점**        | **핵심 약점**          |
| **지역 특징점 (예: SIFT)**               | 주요 핵심점 기술                        | N개 벡터의 집합  | 높은 정밀도          | 대규모 매칭에 비효율적 |
| **BoVW**                                 | '시각적 단어'의 출현 빈도 계산          | K차원 히스토그램 | 압축적, 고정 크기    | 높은 정보 손실         |
| **VLAD**                                 | 클러스터 중심에 대한 특징점의 잔차 누적 | K x D 차원 벡터  | BoVW보다 구별력 높음 | 미분 불가능한 할당     |




이 장은 보고서의 핵심으로, NetVLAD 자체를 해부합니다. 데이터가 네트워크를 통과하는 흐름에 따라 구조를 설명하고, NetVLAD의 성공이 아키텍처뿐만 아니라 영리한 데이터 소싱과 손실 함수 설계의 결합체임을 보여줍니다.


NetVLAD는 VLAD와 유사한 메커니즘을 특별한 풀링(pooling) 계층으로 신경망에 직접 통합한 새로운 CNN 아키텍처입니다.17 핵심 아이디어는 픽셀에서 최종 기술자에 이르는 전체 과정을 

**종단간(end-to-end) 학습 가능**하게 만드는 것이었습니다.17 이는 이전 기술들이 도달할 수 없었던 목표였습니다.


1. 1단계: CNN 백본 (특징 추출기)

   입력 이미지는 먼저 VGG16이나 ResNet과 같은 표준 CNN을 통과합니다.40 최종 분류 계층을 사용하는 대신, 마지막 

   *컨볼루션* 계층 중 하나(예: conv5)의 출력을 가져옵니다.17 이 출력은 높이 x 너비 x 깊이 차원의 풍부한 3D 텐서로, 특정 (x, y) 위치의 각 벡터는 이미지의 해당 부분에 대한 고수준 지역 기술자 역할을 합니다. 이것이 NetVLAD 계층의 입력이 됩니다.42

2. 2단계: NetVLAD 계층 (미분 가능한 통합기)

   이것이 혁신의 심장부입니다. 1단계에서 얻은 N개의 지역 기술자들을 단일 K x D 벡터로 통합합니다.

   - **VLAD의 문제점:** 표준 VLAD는 '강성 할당'을 사용하는데, 이는 미분 불가능하여 역전파(backpropagation)에 필요한 그래디언트 흐름을 끊어버립니다.44
   - **NetVLAD의 해결책: 유연한 할당(Soft Assignment).** NetVLAD는 이를 '유연한 할당' 메커니즘으로 대체합니다. 각 지역 기술자는 *모든* 클러스터에 대해 "얼마나 속해 있는지"를 나타내는 가중치를 할당받습니다. 이는 보통 소프트맥스(softmax) 함수를 통해 계산되며, 이 가중치 할당은 부드럽고 미분 가능한 함수입니다.37
   - **학습 가능한 파라미터:** 결정적으로, 클러스터 중심(ck)과 유연한 할당 파라미터(wk,bk)는 모두 네트워크의 *학습 가능한 파라미터*입니다. 네트워크는 단순히 이를 사용하는 것이 아니라, 장소 인식 작업에 최적화된 클러스터와 할당 함수를 스스로 학습합니다.37

3. 3단계: 통합 및 정규화

   이 계층은 VLAD와 마찬가지로 각 클러스터에 대한 잔차의 가중 합을 계산하지만, 유연한 할당 가중치를 사용합니다.37 결과로 나온 K x D 행렬은 성능 향상과 시각적 '폭발성'(하나의 지배적인 특징이 기술자 전체를 압도하는 현상)을 억제하기 위해 정규화(클러스터 내 정규화 및 L2 정규화)됩니다.36

4. 4단계: 최종 기술자

   정규화된 K x D 행렬은 하나의 긴 벡터로 펼쳐집니다. 종종 주성분 분석(PCA)이 최종 단계로 사용되어 이 벡터의 차원을 관리하기 쉬운 크기(예: 4096차원)로 줄이면서 대부분의 정보를 유지합니다.17 이 최종적이고 압축된 벡터가 바로 입력 이미지에 대한 고유하고 학습 가능한 지문, 즉 

   **NetVLAD 기술자**입니다.


이러한 네트워크를 훈련시키려면 방대한 양의 데이터가 필요합니다. 연구자들은 구글 스트리트 뷰 타임머신(Google Street View Time Machine)을 영리하게 활용했는데, 이는 같은 장소의 이미지를 다른 시간에 제공하지만 GPS 레이블이 부정확한 '약한 지도 학습(weak supervision)' 데이터입니다.17

학습의 엔진은 **삼중항 순위 손실(Triplet Ranking Loss)** 함수입니다.37

- 훈련 시, 모델에는 세 개의 이미지로 구성된 '삼중항(triplet)'이 주어집니다: **기준(Anchor)** 이미지, **긍정(Positive)** 이미지(같은 장소의 다른 이미지), 그리고 **부정(Negative)** 이미지(다른 장소의 이미지).
- 손실 함수의 목표는 기준과 긍정 기술자 사이의 거리가 기준과 부정 기술자 사이의 거리보다 특정 여유(margin) 값 이상으로 *더 작아지도록* 네트워크의 파라미터를 조정하는 것입니다.44
- **비유:** 이는 어린아이를 가르치는 것과 같습니다. "우리 집을 찍은 이 사진과 *저* 사진은, 다른 집을 찍은 이 사진보다 너에게 더 '가깝게' 느껴져야 해." 수백만 개의 이런 예시를 보여줌으로써, 네트워크는 '장소'라는 의미에 따라 군집화되는 기술자를 생성하는 법을 배웁니다.

NetVLAD의 핵심 기술 혁신은 미분 가능한 VLAD 계층을 만든 것입니다. 이 '유연한 할당' 메커니즘은 종단간 훈련의 문을 여는 열쇠였습니다. 이는 고전적인 알고리즘의 성공적인 아이디어를 가져와 미분 불가능한 부분을 미분 가능한 근사치로 대체하는 강력한 설계 패턴을 보여줍니다. 이를 통해 고전적인 컴퓨터 비전의 지식과 종단간 딥러닝의 강력한 힘을 융합할 수 있었습니다.


이 장에서는 이론에서 실천으로 나아가 NetVLAD의 성능을 평가하고 그 영향과 유산을 탐구합니다. NetVLAD의 한계점은 실패가 아니라, 커뮤니티에 다음 연구 질문을 명확하게 제시하여 새로운 혁신의 순환을 이끈 생산적인 촉매제 역할을 했습니다.


- **강인성(Robustness):** 조명(낮/밤), 날씨, 계절, 시점의 극심한 변화에 놀라운 강인성을 보여줍니다.19 다양한 데이터에 대한 종단간 훈련은 자동차나 보행자와 같은 방해 요소를 무시하고 '장소'의 본질을 학습하도록 만듭니다.17
- **압축성과 효율성(Compactness and Efficiency):** 대규모 데이터베이스에서 최근접 이웃 검색 알고리즘(k-d 트리, Faiss 등)을 사용하여 매우 효율적으로 검색할 수 있는 압축된 고정 크기 전역 기술자를 생성합니다.17 이는 수천 개의 지역 특징점을 매칭하는 것에 비해 엄청난 이점이며, 공격적인 차원 축소 후에도 효과적입니다.47
- **다용성("플러그 앤 플레이"):** NetVLAD 계층은 모듈식이어서 다양한 CNN 백본(VGG, ResNet)에 "연결"할 수 있으며, LiDAR 점 구름(PointNetVLAD)과 같은 다른 데이터 유형에도 적용할 수 있습니다.47


- **공간 정보 손실:** 순서 없는 풀링 방식으로서 NetVLAD는 이미지 전체의 특징을 통합하면서 정확한 공간적 배치를 버립니다.47 이는 정밀한 기하학적 검증이 필요하거나, 특징의 공간적 배열이 유일한 구별 단서가 되는 극단적인 시점 변화의 경우 문제가 될 수 있습니다.
- **전역적 초점:** 기술자의 전역적인 특성 때문에 매우 모호한 위치(지각적 모호성)를 구별하는 데 중요한 미세한 세부 사항을 놓칠 수 있습니다.47
- **데이터 및 훈련 요구:** 잘 일반화되기 위해서는 방대하고 다양한 훈련 데이터가 필요합니다. 미세 조정 없이는 훈련 데이터와 매우 다른 도메인에서 테스트할 때 성능이 저하될 수 있습니다.47


NetVLAD는 이야기의 끝이 아니라 새로운 장의 시작이었습니다. 그 한계는 다음 세대의 혁신에 직접적인 영감을 주었습니다.

- **Patch-NetVLAD:** 공간 정보 손실 문제를 해결하기 위해 NetVLAD 로직을 특징 맵의 더 작은 패치에 적용합니다. 먼저 전역 NetVLAD 기술자로 대략적인 위치를 찾은 다음, 더 국소적인 '패치' 기술자들을 매칭하여 순위를 재조정함으로써 전역적 방법과 지역적 방법의 장점을 결합합니다.20
- **SuperVLAD 및 경량화 변형:** 연구는 아키텍처를 더 효율적으로 만들고 일반화 성능을 향상시키는 데 초점을 맞추었습니다. SuperVLAD는 클러스터 중심을 제거하여 모델을 단순화하고, 파라미터를 줄이며, 도메인 외부 데이터에 대한 성능을 향상시켰습니다.51 Ghost-dil-NetVLAD와 같은 다른 변형들은 모바일 로봇과 같은 자원 제약적인 플랫폼을 위한 경량화 버전을 만드는 데 중점을 둡니다.47


- **로봇 공학 및 SLAM:** 주요 응용 분야는 모바일 로봇(예: 창고 로봇, 드론)의 SLAM 시스템에서 루프 폐쇄 감지입니다.20 넓은 공간을 항해하는 로봇은 NetVLAD를 사용하여 이전에 방문한 복도를 인식하고, 지도를 수정하며, 강인한 장기 자율성을 달성할 수 있습니다.
- **자율 주행:** 대략적인 전역 위치 추정에 사용됩니다. 도시의 자율 주행 차량은 이미지를 캡처하고 NetVLAD를 사용하여 사전에 구축된 도시의 시각적 지도와 매칭함으로써 "나는 이 블록에 있다"와 같은 빠르고 강인한 위치 추정치를 얻어 GPS를 보완할 수 있습니다.17
- **증강 현실:** 사용자의 위치를 인식하고 현실 세계 위에 올바른 디지털 정보를 겹쳐 표시하는 데 사용될 수 있습니다.17


우리는 시각 기반 항법의 근본적인 도전에서부터 이미지 기술자의 진화를 거쳐, 마침내 딥러닝 기반의 해결책인 NetVLAD에 이르기까지의 여정을 살펴보았습니다.

NetVLAD의 핵심 기여는 고전적인 컴퓨터 비전 알고리즘(VLAD)의 원리를 딥러닝 특징 학습(CNN)의 힘과 성공적으로 융합한, 학습 가능한 종단간 아키텍처를 창조한 것입니다. 이는 VPR 분야에서 새로운 기술 수준을 제시하고 기초적인 벤치마크가 되었습니다. NetVLAD의 성공은 약하게 레이블링된 대규모 데이터로부터 직접 작업별 표현을 학습하는 것의 엄청난 힘을 입증했습니다.

결론적으로 NetVLAD는 최종적인 해답이 아니라, 하나의 초석으로 자리매김했습니다. 이 기술은 장소 인식을 위한 새로운 패러다임을 확립했으며, 미분 가능한 풀링, 검색을 위한 종단간 훈련, 그리고 압축적이고 강인한 기술자 생성과 같은 핵심 개념들은 오늘날 로봇 공학과 인공지능 분야의 현대적인 위치 추정 시스템 설계에 계속해서 영향을 미치고 있습니다.


1. A Distributed Vision-Based Navigation System for Khepera IV Mobile Robots - PMC, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7570577/
2. (PDF) A survey on vision-based UAV navigation - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/322441239_A_survey_on_vision-based_UAV_navigation
3. Vision-Based Navigation Techniques for Unmanned Aerial Vehicles: Review and Challenges - MDPI, accessed July 1, 2025, https://www.mdpi.com/2504-446X/7/2/89
4. SLAM(동시적 위치추정 및 지도작성)이란 – MATLAB 및 Simulink - 매스웍스, accessed July 1, 2025, https://kr.mathworks.com/discovery/slam.html
5. 스스로 만드는 위치 지도 SLAM 기술의 원리 - LG디스커버리랩, accessed July 1, 2025, https://www.lgdlab.or.kr/contents/6
6. monglory.tistory.com, accessed July 1, 2025, [https://monglory.tistory.com/31#:~:text=LiDAR%EB%8A%94%20%EC%9E%90%EC%9C%A8%EC%A3%BC%ED%96%89%EC%97%90%EC%84%9C,%EC%A0%95%ED%99%95%ED%95%98%EA%B2%8C%20%ED%8C%90%EB%8B%A8%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.](https://monglory.tistory.com/31#:~:text=LiDAR는 자율주행에서,정확하게 판단할 수 있습니다.)
7. 자율주행의 눈이 되어주는 센서 - 테크 포커스, accessed July 1, 2025, https://techfocus.kr/ts_job/5
8. Vision-Based Navigation for Urban Air Mobility: A Survey - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/376618069_Vision-Based_Navigation_for_Urban_Air_Mobility_A_Survey
9. A Survey on Vision-based Localization and Geo-Referencing Technology for Advanced Air Mobility - Korea Science, accessed July 1, 2025, https://koreascience.kr/article/JAKO202464143225405.page
10. '자율주행'을 가능하게 만드는 핵심 기술은 무엇인가? - 전문가칼럼, accessed July 1, 2025, https://kixxman.com/adas-core-technology
11. SLAM 기술의 변화와 흐름 - MINT Lab, accessed July 1, 2025, https://mint-lab.github.io/mint-lab/papers/Choi21_kros.pdf
12. 슬램(SLAM: Simultaneous Localization And Mapping) - 클라우드의 데일리 리포트 - 티스토리, accessed July 1, 2025, https://clouds-daily.tistory.com/273
13. Visual place recognition from end-to-end semantic scene text features - Frontiers, accessed July 1, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2024.1424883/full
14. Place Recognition: An Overview of Vision Perspective - MDPI, accessed July 1, 2025, https://www.mdpi.com/2076-3417/8/11/2257
15. Visual Place Recognition. Introduction | by SOMIK DHAR - Medium, accessed July 1, 2025, https://medium.com/@sd5023/visual-place-recognition-8999307ebb2f
16. Context-Based Visual-Language Place Recognition - arXiv, accessed July 1, 2025, https://arxiv.org/html/2410.19341v1
17. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content_cvpr_2016/papers/Arandjelovic_NetVLAD_CNN_Architecture_CVPR_2016_paper.pdf
18. On the Estimation of Image-matching Uncertainty in Visual Place Recognition - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Zaffar_On_the_Estimation_of_Image-matching_Uncertainty_in_Visual_Place_Recognition_CVPR_2024_paper.pdf
19. Feature Complementation Architecture for Visual Place Recognition - arXiv, accessed July 1, 2025, https://arxiv.org/html/2506.12401v1
20. Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Hausler_Patch-NetVLAD_Multi-Scale_Fusion_of_Locally-Global_Descriptors_for_Place_Recognition_CVPR_2021_paper.pdf
21. [1707.03470] Place recognition: An Overview of Vision Perspective - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1707.03470
22. Perceptual aliasing. Two distant locations in a building share similar... - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/figure/Perceptual-aliasing-Two-distant-locations-in-a-building-share-similar-visual-appearance_fig1_321815533
23. Visual Place Recognition using HMM Sequence Matching - Carnegie Mellon University's Robotics Institute, accessed July 1, 2025, https://www.ri.cmu.edu/pub_files/2014/9/phansen_IROS_2014.pdf
24. Classifying Facial Emotions via Machine Learning | by Sumit Kumar Singh - Medium, accessed July 1, 2025, https://sks147.medium.com/classifying-facial-emotions-via-machine-learning-5aac111932d3
25. Why should we use bag of visual words (or vlad) instead of storing descriptors? - Stack Overflow, accessed July 1, 2025, https://stackoverflow.com/questions/44686898/why-should-we-use-bag-of-visual-words-or-vlad-instead-of-storing-descriptors
26. Leveraging Deep Visual Descriptors for Hierarchical Efficient Localization - Proceedings of Machine Learning Research, accessed July 1, 2025, http://proceedings.mlr.press/v87/sarlin18a/sarlin18a.pdf
27. Bag-of-words model in computer vision - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Bag-of-words_model_in_computer_vision
28. Bag of visual words | Computer Vision and Image Processing Class Notes - Fiveable, accessed July 1, 2025, https://library.fiveable.me/computer-vision-and-image-processing/unit-5/bag-visual-words/study-guide/gKqlaeA7CzYBBncs
29. MLRF Lecture 04 - LRDE, accessed July 1, 2025, https://www.lrde.epita.fr/~jchazalo/teaching/MLRF/202105_IMAGE_SCIA_S8/lecture_04-02_CBIR.pdf
30. VLAD- An extension of Bag of Words - A Positronic Brain - WordPress.com, accessed July 1, 2025, https://ameyajoshi005.wordpress.com/2014/03/29/vlad-an-extension-of-bag-of-words/
31. Mastering VLAD in Computer Vision - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/mastering-vlad-in-computer-vision
32. 촬영 환경 변화에 강인한 NetVLAD 제작 기술, accessed July 1, 2025, https://www.kibme.org/resources/journal/20230817150716998.pdf
33. VLAD for Image Retrieval: A Deep Dive - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/vlad-for-image-retrieval
34. Large Scale Image Retrieval Using Vector of Locally Aggregated Descriptors - Fabrizio Falchi, accessed July 1, 2025, https://falchi.isti.cnr.it/Draft/2013-SISAP-VLAD-DRAFT.pdf
35. Bag of Features vs Vector of Locally Aggregated Descriptors - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/319251089_Bag_of_Features_vs_Vector_of_Locally_Aggregated_Descriptors
36. HIERARCHICAL MULTI-VLAD FOR IMAGE RETRIEVAL Yitong Wang*†, Ling-Yu Duan*†∗, Jie Lin, accessed July 1, 2025, https://idm.pku.edu.cn/__local/3/EF/D8/CBAA95F8BE04C9D8906E86FC074_A4F6E514_133748.pdf?e=.pdf
37. NetVLAD Place-Recognition Head - Emergent Mind, accessed July 1, 2025, https://www.emergentmind.com/topics/netvlad-place-recognition-head
38. arXiv:1511.07247v3 [cs.CV] 2 May 2016, accessed July 1, 2025, https://www.di.ens.fr/~josef/publications/Arandjelovic16.pdf
39. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - PubMed, accessed July 1, 2025, https://pubmed.ncbi.nlm.nih.gov/28622667/
40. NetVLAD: CNN architecture for weakly supervised place recognition - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/311611050_NetVLAD_CNN_architecture_for_weakly_supervised_place_recognition
41. An End-to-End Trainable Multi-Column CNN for Scene Recognition in Extremely Changing Environment - MDPI, accessed July 1, 2025, https://www.mdpi.com/1424-8220/20/6/1556
42. VLAD Encoded Deep Convolutional Features for Unconstrained Face Verification, accessed July 1, 2025, https://engineering.jhu.edu/vpatel36/wp-content/uploads/2018/08/ICPR2016_FV_CNN_final.pdf
43. NetVLAD: CNN architecture for weakly supervised place recognition[2] - HAYEUP - 티스토리, accessed July 1, 2025, https://jhyeup.tistory.com/entry/NetVLAD-CNN-architecture-for-weakly-supervised-place-recognition2
44. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - Medium, accessed July 1, 2025, https://medium.com/data-science/netvlad-cnn-architecture-for-weakly-supervised-place-recognition-ce64b08bebaf
45. Non-local NetVLAD Encoding for Video Classification - Google Research, accessed July 1, 2025, https://research.google.com/youtube8m/workshop2018/c_04.pdf
46. [논문리뷰] NetVLAD : CNN architecture for weakly supervised place recognition - YouTube, accessed July 1, 2025, https://m.youtube.com/watch?v=XIA-9JjNA34&pp=ygUII25ldHZsYWQ%3D
47. NetVLAD Place-Recognition Head - Emergent Mind, accessed July 1, 2025, https://www.emergentmind.com/articles/netvlad-place-recognition-head
48. NetVLAD: CNN architecture for weakly supervised place recognition | PPT - SlideShare, accessed July 1, 2025, https://www.slideshare.net/slideshow/netvlad-cnn-architecture-for-weakly-supervised-place-recognition/63134037
49. (PDF) TriLoc-NetVLAD: Enhancing Long-term Place Recognition in Orchards with a Novel LiDAR-Based Approach - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/386909291_TriLoc-NetVLAD_Enhancing_Long-term_Place_Recognition_in_Orchards_with_a_Novel_LiDAR-Based_Approach
50. Contextual Patch-NetVLAD: Context-Aware Patch Feature Descriptor and Patch Matching Mechanism for Visual Place Recognition - MDPI, accessed July 1, 2025, https://www.mdpi.com/1424-8220/24/3/855
51. SuperVLAD: Compact and Robust Image Descriptors for Visual Place Recognition, accessed July 1, 2025, https://openreview.net/forum?id=bZpZMdY1sj
52. [2103.01486] Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2103.01486
53. SuperVLAD: Compact and Robust Image Descriptors for Visual Place Recognition, accessed July 1, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/hash/0b135d408253205ba501d55c6539bfc7-Abstract-Conference.html
54. Ghost-dil-NetVLAD: A Lightweight Neural Network for Visual Place Recognition - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2112.11679
55. NetVLAD: CNN architecture for weakly supervised place recognition - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/paper/netvlad-cnn-architecture-for-weakly


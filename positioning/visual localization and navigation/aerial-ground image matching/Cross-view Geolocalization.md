# Cross-view Geolocalization

## 제1장. 서론: GPS를 넘어선 시각적 위치 특정 기술의 부상

### 0.1  크로스뷰 지오로컬라이제이션(CVGL)의 정의 및 중요성

크로스뷰 지오로컬라이제이션(Cross-View Geo-localization, CVGL)은 지상에서 촬영된 질의 이미지(query image)의 지리적 위치를, 방대한 항공 또는 위성 이미지 데이터베이스와 매칭하여 추정하는 컴퓨터 비전 기술이다.1 이 기술은 단순히 GPS의 대체재를 넘어, 기계가 환경을 의미론적으로 이해하게 하는 근본적인 패러다임의 전환을 의미한다. 초기 연구 동기는 도심 협곡, 실내, 혹은 재난 지역과 같이 위성 항법 시스템(GPS) 신호가 약하거나 수신 불가능한 환경에서 자율 시스템의 위치 인식을 보완하거나 대체하는 데 있었다.3

전통적인 위치 특정 방식은 동일한 지상 시점에서 촬영된 방대한 이미지 데이터베이스를 구축해야 했으나, 이는 대규모 지역을 모두 포괄하고 모든 시점, 날씨, 시간대별 조건을 촬영하는 데 현실적인 어려움이 따른다.6 CVGL은 이러한 한계를 극복하기 위해 상대적으로 수집과 갱신이 용이한 항공 및 위성 데이터를 참조 데이터베이스로 활용함으로써 실용성을 크게 높였다.7

이 기술의 진정한 중요성은 위치 추정이라는 기능적 목표를 넘어서는 데 있다. CVGL 모델이 성공적으로 작동하기 위해서는 극적인 시점 차이를 극복하고 시점 불변(view-invariant) 특징을 학습해야 한다. 이는 단순한 픽셀이나 질감 매칭을 넘어서, "도로 옆에 건물이 있다" 또는 "공원이 나무들로 둘러싸여 있다"와 같은 고차원적인 의미론적 개념과 공간적 배치(spatial layout)를 이해해야 함을 의미한다.8 결과적으로, 성공적인 CVGL 시스템은 시각 데이터로부터 환경의 의미론적 지도를 암묵적으로 구축하는 것과 같다. 이는 단순히 위성 신호로 좌표를 삼각 측량하는 것보다 훨씬 복잡한 인지적 과업이며, 기계가 장면의 지리적 맥락을 '읽고' 이해하는 능력을 부여한다는 점에서 그 의의가 매우 크다.

### 0.2  핵심 난제: 시점, 스케일, 조명, 계절 변화에 따른 외형 불일치

CVGL 기술의 핵심적인 기술적 난제는 본질적으로 '정체성 보존 특징(identity-preserving features)'으로부터 '외란 변수(nuisance variables)'를 분리(disentangle)하는 문제로 귀결된다. 지상 뷰와 항공 뷰는 극적인 시점 차이로 인해 객체의 스케일, 형태, 그리고 가려짐(occlusion) 현상에서 큰 불일치를 보인다.1 이러한 기하학적 차이는 외란 변수의 한 축을 형성한다.

또 다른 주요 외란 변수는 시시각각 변하는 환경 조건이다. 동일한 장소일지라도 조명, 날씨, 계절, 시간의 변화에 따라 전혀 다른 시각적 특징을 나타내게 된다.6 예를 들어, 여름의 무성한 녹음은 겨울에는 앙상한 나뭇가지로 변하고, 낮의 선명한 그림자는 밤에는 존재하지 않는다. 이러한 변화는 장소의 본질적인 정체성(도로 구조, 건물 배치 등)을 바꾸지 않으면서 외형만을 크게 변화시킨다. 또한, 차량이나 보행자와 같은 움직이는 동적 요소들은 정적인 배경 정보와 분리되어야 할 노이즈로 작용하여 매칭을 더욱 어렵게 만든다.11

초기 CVGL 방법들이 사용했던 SIFT나 HoG와 같은 특징점들은 이러한 외란 변수에 매우 민감하여 성능에 한계가 있었다.2 반면, 대조 학습(contrastive learning)을 사용하는 현대의 딥러닝 접근법은 이러한 문제를 해결하기 위해 설계되었다. 모델은 동일한 위치의 이미지 쌍(positive pair)은 외란 변수와 관계없이 임베딩 공간에서 가깝게, 다른 위치의 이미지 쌍(negative pair)은 멀게 만들도록 학습된다. 이 과정을 통해 모델은 위치의 정체성을 나타내는 본질적인 특징을 학습하고, 외형 변화를 유발하는 외란 변수에는 둔감해지도록 강제된다. 따라서 CVGL 기술의 발전 과정은 이러한 외란 변수에 강인한 특징 표현을 만들어내는 능력의 고도화 과정으로 해석할 수 있다.

### 0.3  주요 응용 분야: 자율주행, 로보틱스, 증강현실, 재난 대응

CVGL 기술의 잠재력은 다양한 미래 산업 분야에서 확인되고 있다. 가장 대표적인 분야는 자율주행차와 로보틱스로, GPS 음영 지역에서도 센티미터 수준의 정밀한 측위를 가능하게 하여 안전성과 신뢰성을 높인다.6 또한, 사용자의 현재 위치와 방향을 정확히 파악하여 가상 정보를 현실 세계에 정합시키는 광역 증강현실(Wide Area AR)의 핵심 기반 기술로도 활용된다.6 이 외에도 GPS 신호에 의존하지 않는 드론의 자율 비행 3, 통신망이 두절된 재난 지역에서의 신속한 탐색 및 구조 활동 13, 그리고 SNS 등에 올라온 출처 불명의 뉴스 사진이나 사건 현장 사진의 발생 위치를 추적하는 포렌식 분석 15 등 사회적, 산업적으로 중요한 여러 분야에서 그 활용 가치가 매우 높다.

## 1.  크로스뷰 지오로컬라이제이션의 원리 및 기술적 진화

### 1.1  초기 접근법: 수동 특징점(Hand-crafted Features) 기반 매칭

딥러닝 기술이 보편화되기 이전, CVGL 연구는 주로 수동으로 설계된 지역 특징점(hand-crafted local features)에 의존했다. 연구자들은 SIFT(Scale-Invariant Feature Transform), HoG(Histogram of Oriented Gradients), GIST, 색상 히스토그램과 같은 알고리즘을 사용하여 이미지에서 핵심적인 시각 정보를 추출하고, 이 특징점들의 유사도를 비교하여 이미지 간의 매칭을 시도했다.2 이러한 접근법은 특정 변환(크기, 회전 등)에 대해서는 어느 정도 강인성을 보였으나, 지상 뷰와 항공 뷰 사이의 극심한 시점 변화나 급격한 조명 변화 앞에서는 매우 취약했다.2 두 이미지 간의 직접적인 특징점 대응 관계를 찾는 데 초점을 맞췄기 때문에, 외형적 유사성이 거의 없는 크로스뷰 환경에서는 안정적인 성능을 기대하기 어려웠다.

### 1.2  딥러닝의 등장: CNN과 샴 네트워크(Siamese Network)를 활용한 특징 학습

딥러닝, 특히 합성곱 신경망(CNN)의 등장은 CVGL 분야에 근본적인 패러다임 전환을 가져왔다.1 이 전환의 핵심은 지오로컬라이제이션 문제를 '특징점 매칭 문제'에서 '메트릭 공간 학습 문제'로 재정의한 데 있다. 이 개념적 전환을 구현한 대표적인 아키텍처가 바로 샴 네트워크(Siamese Network)이다.

샴 네트워크는 동일한 구조와 가중치를 공유하는 두 개의 CNN 인코더(또는 단일 인코더를 공유하여 사용)로 구성된다. 지상 이미지와 항공 이미지가 각각의 인코더를 통과하면, 각 이미지를 대표하는 고차원의 특징 벡터(feature vector), 즉 임베딩(embedding)이 생성된다.1 학습의 목표는 이 임베딩 공간의 기하학적 구조를 조작하는 것이다. Triplet Loss와 같은 메트릭 러닝(metric learning) 손실 함수를 사용하여, 같은 장소를 촬영한 긍정 쌍(positive pair)의 임베딩 간 거리는 최소화하고, 다른 장소를 촬영한 부정 쌍(negative pair)의 임베딩 간 거리는 최대화하도록 모델을 학습시킨다.17

이러한 접근법은 특정 픽셀이나 영역을 직접 비교하는 대신, 이미지 전체의 추상적인 의미를 담은 임베딩을 통해 유사도를 판단한다. 추론 시에는 질의 이미지의 임베딩을 계산한 뒤, 참조 데이터베이스 내에서 가장 가까운 임베딩을 가진 항공 이미지를 찾는 최근접 이웃 탐색(nearest-neighbor search)을 수행하면 된다. 이처럼 직접적인 매칭에서 임베딩 공간에서의 유사도 검색으로 문제를 추상화한 것이 딥러닝 기반 CVGL의 핵심 혁신이며, 이를 통해 시점 변화에 훨씬 강인하고 추상적인 특징 표현을 학습하는 것이 가능해졌다.

### 1.3  기하학적 정합 기법: 폴라 변환(Polar Transform)과 조감도(BEV) 변환

딥러닝 모델이 종단간(end-to-end)으로 시점 차이를 학습하는 것은 매우 어려운 과제이다. 따라서 일부 연구들은 순수한 특징 학습과 원시 픽셀 매칭 사이의 '중간 지대' 접근법을 채택했다. 이는 기하학에 대한 사전 지식(domain knowledge)을 활용하여 입력 데이터를 신경망에 투입하기 전에 미리 정렬된 상태로 가공함으로써 학습 문제를 단순화하는 전략이다.

대표적인 기법인 폴라 변환(polar transformation)은 위성 이미지를 중심점을 기준으로 펼쳐, 지상 파노라마 이미지와 유사한 방사형 구조로 변환하는 방식이다.8 이 변환을 통해 위성 이미지의 북쪽 방향이 파노라마 이미지의 특정 열에 대응되므로, 도로의 방향과 같은 핵심적인 방사형 구조가 두 뷰 간에 정렬되는 효과를 얻는다.19

최근에는 이와 반대로, 지상 뷰 이미지를 명시적으로 조감도(Bird's-Eye-View, BEV)로 변환하여 위성 이미지와의 시점 차이를 직접적으로 줄이려는 연구도 활발히 진행되고 있다.19 이러한 기하학적 사전 정렬 기법들은 모델이 극복해야 할 '도메인 갭(domain gap)'을 줄여주어, 이미 더 유사한 형태로 가공된 입력을 바탕으로 학습을 진행하므로 성능과 효율성을 크게 향상시킬 수 있다. 이는 CVGL 시스템 설계 시, 기하학적 불변성을 얼마나 명시적으로 설계할 것인지, 아니면 모델이 암묵적으로 학습하도록 할 것인지에 대한 근본적인 설계 철학의 차이를 보여준다.

## 2.  최신 딥러닝 아키텍처의 적용: 트랜스포머와 어텐션 메커니즘

### 2.1  트랜스포머(ViT) 기반 인코더의 구조적 이점

최근 몇 년간 컴퓨터 비전 분야를 지배하고 있는 트랜스포머(Transformer) 아키텍처는 CVGL 분야에서도 그 잠재력을 입증하고 있다. 기존 CNN은 커널 연산을 통해 지역적인 수용 영역(local receptive field) 내의 정보만을 처리하므로, 이미지 전반에 걸친 광역적인 컨텍스트를 파악하는 데 한계가 있었다. 반면, Vision Transformer (ViT)와 같은 트랜스포머 기반 모델은 셀프 어텐션(self-attention) 메커니즘을 통해 이미지 내 모든 패치(patch) 간의 관계를 직접 계산한다. 이를 통해 이미지의 특정 부분에 국한되지 않고 전체적인 전역 컨텍스트와 장거리 의존성(long-range dependencies)을 효과적으로 포착할 수 있으며, 이는 CVGL에서 지상 뷰와 항공 뷰의 전체적인 장면 구조를 이해하는 데 매우 유리하게 작용한다.1

### 2.2  크로스 어텐션을 통한 지상-위성 뷰 특징 간의 상관관계 모델링

트랜스포머의 또 다른 강력한 기능은 크로스 어텐션(cross-attention) 메커니즘이다. 이는 두 개의 다른 입력 시퀀스(여기서는 지상 뷰 특징과 항공 뷰 특징) 간의 상호작용을 모델링하는 데 사용된다. CVGL에서는 각 뷰에 대해 독립적인 인코더로 특징을 추출한 후, 크로스 어텐션 레이어를 통해 한쪽 뷰의 특징을 쿼리(Query)로, 다른 쪽 뷰의 특징을 키(Key)와 값(Value)으로 사용하여 두 특징 맵을 융합한다.4 이 과정을 통해 모델은 "지상 뷰의 이 영역은 항공 뷰의 저 영역과 가장 관련이 깊다"와 같은 미묘한 의미론적, 공간적 상관관계를 명시적으로 학습할 수 있다. 이는 단순히 두 특징 벡터의 거리를 계산하는 것을 넘어, 두 뷰 간의 구조적인 대응 관계를 직접적으로 찾아내기 때문에 훨씬 정교한 매칭을 가능하게 한다.

### 2.3  계층적 어텐션을 활용한 전역 및 지역 컨텍스트 통합

더 나아가, 일부 연구에서는 인간 전문가가 지도를 볼 때 도시, 지역, 거리 순으로 스케일을 바꿔가며 위치를 찾는 방식에 영감을 받아 계층적 접근법을 도입하고 있다.6 계층적 크로스 어텐션(hierarchical cross-attention)은 이미지 내의 시각 정보와 다양한 지리적 수준(계층) 간의 관계를 동시에 학습한다. 예를 들어, 모델은 이미지의 전반적인 풍경(e.g., 해안 도시, 산악 지역)을 통해 대략적인 지역을 추론하고, 동시에 특정 건물이나 도로의 형태를 통해 세부 위치를 특정하는 학습을 진행할 수 있다. 이러한 다중 스케일 접근법은 전역적인 컨텍스트와 지역적인 세부 정보를 통합하여 보다 강인하고 정확한 위치 추정을 가능하게 한다.

## 3.  ICCV 2023 핵심 연구 동향 심층 분석

세계 최고 권위의 컴퓨터 비전 학회인 ICCV 2023에서는 CVGL 분야의 최신 연구 동향을 조망할 수 있는 중요한 논문들이 발표되었다. 특히 두드러지는 경향은 복잡한 모델 아키텍처 설계 경쟁에서 벗어나, 보다 지능적인 데이터 활용 전략과 실제 환경의 노이즈에 대한 강인성 확보로 연구의 초점이 이동하고 있다는 점이다.

### 3.1  Sample4Geo: 대조 학습과 하드 네거티브 샘플링의 재조명

Fabian Deuser 등이 발표한 "Sample4Geo: Hard Negative Sampling For Cross-View Geo-Localisation"은 CVGL 분야에서 연구 방향의 중요한 전환점을 시사한다.18 이 연구는 폴라 변환과 같은 복잡한 기하학적 전처리나 별도의 어텐션 모듈 없이, '기성품(off-the-shelf)'에 가까운 단순한 Siamese ConvNeXt 아키텍처와 대칭적 InfoNCE 손실 함수만을 사용하여 여러 벤치마크에서 기존 최고 성능(SOTA)을 뛰어넘었다.18

이러한 성공의 핵심은 모델 아키텍처가 아닌, 훈련 데이터를 구성하는 '커리큘럼(curriculum)'에 있었다. 즉, 모델 중심 최적화에서 데이터 중심 최적화로의 전환을 보여준다.

- **초기 단계 (GPS 기반 지리적 샘플링):** 모델 학습 초기에는 아직 특징 표현이 제대로 형성되지 않았기 때문에, 시각적 유사도만으로는 효과적인 '어려운 예제(hard negative)'를 찾기 어렵다. 이 문제를 해결하기 위해, 저자들은 GPS 좌표를 활용하여 현재 위치(positive)와 지리적으로는 가깝지만 다른 위치의 이미지들을 하드 네거티브로 샘플링했다. 예를 들어, 같은 도시의 다른 거리는 건물 스타일이나 도로 폭 등이 유사하여 모델이 구분하기 어렵다. 이러한 샘플들을 초기에 집중적으로 학습시킴으로써, 모델은 거시적인 차이(e.g., 도시 vs. 시골)가 아닌 미세한 차이에 집중하도록 강제되어 강력한 초기 특징을 학습하게 된다.18
- **후기 단계 (시각적 유사도 기반 동적 샘플링, DSS):** 모델이 어느 정도 학습되어 의미 있는 임베딩 공간이 형성된 후에는, 학습 전략을 전환한다. 훈련 데이터셋 전체에 대해 추론을 수행하여, 각 질의 이미지에 대해 임베딩 공간에서 가장 가까운, 즉 모델이 가장 헷갈려 하는 샘플들을 동적으로 찾아내 하드 네거티브로 사용한다. 이 과정을 주기적으로 반복함으로써 모델은 지속적으로 자신의 약점을 보완하고 더욱 정교한 구분 능력을 갖추게 된다.18

이 논문의 실험 결과는 이러한 데이터 샘플링 전략의 효과를 명확히 보여준다. 무작위 샘플링에 비해 GPS 기반 및 동적 유사도 샘플링을 함께 사용했을 때, 특히 일반화 성능을 측정하는 VIGOR cross-area 벤치마크에서 성능이 극적으로 향상되었다.18 이는 모델의 최종 성능을 결정하는 데 있어, 훈련 과정에서 모델에게 제시되는 문제의 난이도가 네트워크 아키텍처의 복잡성보다 더 지배적인 요소일 수 있음을 시사한다. 이는 향후 연구가 새로운 네트워크 구조 탐색뿐만 아니라, 지능적인 데이터 샘플링, 증강, 그리고 커리큘럼 학습 분야에서 더 큰 성과를 거둘 수 있음을 암시한다.

### 3.2  View Consistent Purification for Accurate Cross-View Localization

Shan Wang 등이 제안한 이 연구는 많은 CVGL 연구들이 채택하는 '이미지 검색(retrieval)' 패러다임에서 한 걸음 더 나아가, '정밀 측위 및 자세 추정(fine-grained localization and pose estimation)'으로 목표를 확장했다는 점에서 주목할 만하다.11 이 연구는 이미지 검색과 전통적인 기하학적 컴퓨터 비전 사이의 간극을 메우려는 시도로 볼 수 있다.

대부분의 CVGL 방법론은 "이 이미지는 어디에서 찍혔는가?"라는 질문에 답하며, 갤러리에서 가장 유사한 참조 이미지를 찾아 그 위치 레이블을 반환한다.1 반면, 이 논문은 "카메라의 정확한 3D/6D 자세는 무엇인가?"라는 질문에 답하고자 한다. 실제로 이 모델은 0.5m 미만의 측면/종방향 위치 오차와 2도 미만의 방향 오차를 달성하는 것을 목표로 한다.11

이러한 높은 정밀도를 달성하기 위해, 연구는 다음과 같은 핵심 아이디어를 제안한다.

- **시점 일관적 정제(View Consistent Purification):** 정밀한 기하학적 정렬을 위해서는 동적 객체(차량, 보행자)가 단순한 노이즈가 아니라 오차의 직접적인 원인이 된다는 점에 주목한다. 따라서 딥러닝 특징을 활용하여 지상 뷰와 위성 뷰 양쪽에서 시점 변화에 관계없이 일관되게 관찰되는 '안정적인' 핵심점(keypoint)만을 탐지하고, 지면에서 벗어난(off-the-ground) 동적 객체와 관련된 특징은 능동적으로 제거한다.11
- **기하학적 모델링과 공간 임베딩:** 정제된 핵심점들을 바탕으로 두 뷰 간의 호모그래피(homography) 변환을 추정하여 기하학적 관계를 명시적으로 모델링한다. 또한, 카메라의 내부(intrinsic) 및 외부(extrinsic) 파라미터 정보를 공간 임베딩(spatial embedding) 형태로 특징에 주입하여, 순수한 시각 정보만으로는 해결하기 어려운 모호성을 줄이고 매칭 정확도를 향상시킨다.11

이 연구는 딥러닝 기반의 이미지 검색과 강인한 기하학적 검증을 통합하는 새로운 방향을 제시한다. 이는 CVGL 기술이 대략적인 위치 추정에서 벗어나, 로보틱스 및 자율주행 분야에서 요구하는 고정밀 자세 추정 기술로 발전해 나갈 수 있는 가능성을 보여준다.

### 3.3  ICCV 2023의 기타 관련 연구 및 워크숍 동향

ICCV 2023에서는 CVGL과 밀접하게 관련된 분야들에서도 중요한 연구들이 다수 발표되었다. 시각적 장소 인식(Visual Place Recognition) 세션에서는 3D 포인트 클라우드를 활용하여 항공-지상 간 3D 장소 인식을 수행하는 `CrossLoc3D`나, LiDAR 데이터를 조감도 이미지로 변환하여 장소를 인식하는 `BEVPlace`와 같은 연구가 발표되어 다중 모달리티 융합의 중요성을 다시 한번 확인시켜 주었다.21 이미지 매칭(Image Matching) 세션에서도 가려짐(occlusion)에 강인한 매칭 기법이나 장면의 맥락을 이해하는 매칭 기법들이 제안되었다.21

또한, 항공 포인트 클라우드로부터 도시 규모의 건물 구조를 학습하기 위한 대규모 데이터셋 `Building3D`의 공개는 향후 관련 연구를 촉진할 중요한 기반을 마련했다.21 학회 본 행사 외에도, CVPR 2023에서 개최된 "실세계 시각 지오로컬라이제이션의 최신 동향" 튜토리얼에서는 크로스뷰 및 크로스모달 지오로컬라이제이션이 핵심 주제로 심도 있게 다루어지는 등, 이 분야에 대한 학계의 높은 관심이 지속되고 있음을 알 수 있다.6

## 4.  다중 모달리티의 융합: 시각 정보를 넘어서

CVGL 기술은 지상 이미지와 항공/위성 이미지라는 두 가지 시각적 모달리티를 넘어서, 텍스트, 3D 포인트 클라우드, 구조화된 지도 데이터 등 더욱 다양한 정보를 융합하는 방향으로 빠르게 진화하고 있다. 이는 문제를 단순한 패턴 매칭에서 복합적인 추론의 영역으로 확장시키고 있다.

### 4.1  텍스트와 자연어 기반 지오로컬라이제이션: 새로운 상호작용 패러다임

최근 가장 혁신적인 연구 방향 중 하나는 자연어 설명을 쿼리로 사용하는 것이다.22 이는 CVGL 패러다임을 수동적인 '인식(recognition)'에서 능동적인 '인간 참여형 추론(human-in-the-loop reasoning)'으로 근본적으로 변화시킨다. 사용자가 "파란 지붕 집 옆에 있는 빨간 우체통 근처"와 같은 자연어 문장을 입력하면, 시스템이 이를 해석하여 해당하는 위성 이미지를 검색해주는 방식이다. 이 기술은 특히 GPS가 없는 상황에서 보행자에게 길을 안내하거나, 긴급 구조 요원에게 조난자의 위치를 설명하는 등 직관적인 상호작용이 중요한 응용 분야에서 엄청난 잠재력을 가진다.22

이 새로운 패러다임은 새로운 기술적 과제를 제시한다. 기존의 '도메인 갭'이 지상 뷰와 항공 뷰라는 두 시각 모달리티 사이의 문제였다면, 이제는 상징적이고 추상적인 '언어'의 영역과 시각적/기하학적인 '이미지'의 영역 사이의 간극을 메워야 한다. 모델은 "파란 지붕", "~의 옆에"와 같은 언어적 개념을 이미지에 존재하는 시각적 특징 및 공간적 관계와 '연결(grounding)'하는 법을 배워야 한다.

이러한 연구를 위해 새로운 데이터셋은 필수적이다. `CVG-Text` 데이터셋은 지상/위성/OSM 이미지와 함께, 거대 언어 모델(LMM)을 활용하여 생성된 고품질 자연어 설명을 쌍으로 제공하는 최초의 벤치마크이다.25 이와 함께 제안된 

`CrossText2Loc` 모델은 긴 텍스트를 효과적으로 인코딩하고 대조 학습을 통해 텍스트와 이미지 임베딩을 정렬하며, 나아가 왜 특정 장소를 검색했는지에 대한 '이유'까지 제공하는 설명가능성(explainability)까지 목표로 한다.23 이는 컴퓨터 비전, 자연어 처리, 공간 추론이 결합된 고차원적인 인공지능 연구의 중요한 이정표라 할 수 있다.

### 4.2  LiDAR 및 3D 포인트 클라우드 활용: 정밀한 3차원 공간 정보의 통합

LiDAR 센서는 레이저를 사용하여 주변 환경과의 거리를 정밀하게 측정하므로, 조명이나 날씨 변화에 매우 강인한 3차원 기하 정보를 제공한다.6 이러한 특성은 외형 변화에 민감한 RGB 카메라의 단점을 보완해 줄 수 있다. 

`CrossLoc3D`나 `BEVPlace`와 같은 연구들은 LiDAR 포인트 클라우드를 이미지와 융합하여 위치 추정의 정확성과 강인성을 크게 향상시키는 가능성을 보여주었다.21 과거에는 대규모 3D 지도 구축 비용이 매우 높았으나 22, 최근 항공 LiDAR 측량 데이터의 가용성이 증가하면서 7, 이를 참조 데이터베이스로 활용하는 새로운 CVGL 연구의 길이 열리고 있다.

### 4.3  시맨틱 맵(GIS, OSM)과의 연동: 구조화된 지리 정보 활용

참조 데이터베이스로 위성 이미지 대신, 도로망, 건물 외곽선, 수역 등의 정보가 벡터 형태로 구조화된 시맨틱 맵(Semantic Map)을 사용하는 접근법은 CVGL 문제를 '외형 기반 매칭(appearance-based matching)'에서 '구조 기반 매칭(structure-based matching)'으로 전환시킨다. OpenStreetMap(OSM)이나 지리 정보 시스템(GIS) 데이터는 계절, 날씨, 조명과 같은 일시적인 외형 정보를 전혀 포함하지 않고, 오직 장소의 기하학적, 위상학적 구조만을 담고 있다.25

이러한 접근법은 외형 변화에 대한 궁극적인 강인성을 제공할 수 있다. 맑은 날의 도로나 눈 덮인 도로는 시각적으로 매우 다르지만, OSM 데이터 상에서는 동일한 선(line)으로 표현된다. 이 방식이 성공하려면, 모델은 먼저 지상 뷰 이미지로부터 "내 앞에는 도로가 있고, 왼쪽에는 건물이 있다"와 같은 의미론적 구조를 파싱(parsing)해내야 한다. 그 후, 이 구조적 정보를 OSM 데이터베이스에서 검색하여 위치를 찾는 것이다. 이러한 아이디어는 Castaldo 등이 ICCV 2015에서 발표한 초기 연구에서부터 제안되었다. 이 연구에서는 이미지에서 도로, 건물 등의 시맨틱 영역을 분할하고, 이들의 공간적 배치를 기술하는 디스크립터를 설계하여 GIS 맵과 매칭하는 방법을 제시했다.9 이는 위치의 일시적인 시각적 외형으로부터 영속적인 기하학적 구조를 완전히 분리해내는 매우 강력한 개념으로, 전천후 고신뢰성 위치 추정을 위한 중요한 연구 방향이다.

## 5.  핵심 데이터셋 및 평가 지표 심층 분석

모든 딥러닝 연구와 마찬가지로, CVGL 분야의 발전은 고품질의 대규모 벤치마크 데이터셋에 의해 견인된다. 각 데이터셋은 서로 다른 기술적 과제를 제시하며, 모델의 성능을 특정 관점에서 평가하는 기준이 된다.

### 5.1  표준 벤치마크: CVUSA, CVACT, VIGOR 데이터셋

- **CVUSA & CVACT:** 이 두 데이터셋은 CVGL 연구에서 가장 널리 사용되는 표준 벤치마크이다. CVUSA는 미국 전역의 시골 및 교외 지역을, CVACT는 호주 캔버라의 도심 지역을 대상으로 한다. 두 데이터셋 모두 지상 뷰 파노라마 이미지와 위성 이미지가 카메라 방향에 맞춰 정렬(aligned)된 형태로 제공되어, 초기 알고리즘 개발 및 비교 평가에 용이하다.18
- **VIGOR:** 이 데이터셋은 보다 현실적인 시나리오를 가정하여 설계되었다. 지상 이미지와 위성 이미지가 완벽히 중앙 정렬되어 있지 않으며, 하나의 지상 뷰 영역을 여러 개의 위성 이미지가 부분적으로 포함하는 '준-긍정(semi-positive)' 샘플을 도입하여 매칭 난이도를 높였다.30 특히 VIGOR의 가장 중요한 특징은 'Cross-Area' 평가 설정이다. 이는 훈련에 사용된 도시(e.g., 뉴욕, 시애틀)와 전혀 다른 새로운 도시(e.g., 샌프란시스코, 시카고)에서 모델의 성능을 평가하는 방식으로, 모델이 특정 지역의 특징에 과적합되지 않고 얼마나 잘 일반화되는지를 측정하는 핵심적인 척도로 사용된다.18
- **University-1652:** 이 데이터셋은 지상 뷰 대신 드론 뷰와 위성 뷰를 매칭하는 태스크를 다룬다. 주로 대학 캠퍼스 내 건물들을 대상으로 하며, 하나의 위성 이미지에 여러 장의 드론 이미지가 대응되는 1:N 매칭 문제를 해결해야 한다.18

### 5.2  차세대 벤치마크: 새로운 도전 과제 제시

최근에는 기존 벤치마크의 한계를 넘어 새로운 기술적 과제를 제시하는 차세대 데이터셋들이 등장하고 있다.

- **CVG-Text:** 자연어 설명을 새로운 모달리티로 추가하여, 텍스트 기반 지오로컬라이제이션이라는 새로운 연구 분야를 개척한 최초의 데이터셋이다.25
- **SetVL-480K:** 기존의 단일 이미지나 순차적인 비디오 프레임이 아닌, 순서에 상관없는 '이미지 셋(set)'을 쿼리로 사용하는 `Set-CVGL`이라는 새로운 태스크를 제안한다. 이는 인간이 주변을 둘러보며 위치를 파악하는 행동을 모사한 것으로, 다양한 관점의 시각 정보를 통합하는 능력을 평가한다.32
- **CVGlobal:** 미국 중심의 기존 데이터셋에서 벗어나 전 세계 여러 대륙의 다양한 도시 데이터를 포함한다. 또한, 시간대 변화(cross-temporal)에 따른 매칭, 위성 이미지 대신 지도 데이터를 사용한 매칭 등 더욱 현실적이고 도전적인 시나리오를 포함하여 알고리즘의 종합적인 성능을 평가한다.19

### 5.3  평가 지표 및 현실적 성능 측정

CVGL 모델의 성능은 주로 이미지 검색(retrieval)의 정확도를 측정하는 지표를 통해 평가된다. 가장 널리 사용되는 것은 **Recall@k (R@k)**로, 검색 결과 상위 k개 안에 정답 이미지가 포함될 확률을 의미한다. R@1, R@5, R@10 등이 주로 사용된다. 또한, 순위까지 고려하는 **Average Precision (AP)**도 중요한 평가 지표이다.18 VIGOR 데이터셋의 등장 이후로는, 단순히 검색 성공 여부를 넘어 실제 GPS 좌표를 이용하여 미터(meter) 단위의 물리적인 측위 오차를 직접 계산하는 방식도 도입되어, 알고리즘의 현실적인 성능을 보다 정밀하게 측정하고 있다.30

| 데이터셋 이름 (Dataset) | 주요 지역 (Location) | 데이터 모달리티 (Modalities) | 핵심 과제 (Key Challenge)                    | 데이터 규모 (Size) | 공개 연도 (Year) |
| ----------------------- | -------------------- | ---------------------------- | -------------------------------------------- | ------------------ | ---------------- |
| **CVUSA**               | 미국 (시골/교외)     | 지상 파노라마, 위성          | 기본 크로스뷰 매칭, 정렬된 뷰                | 44,416 쌍          | 2015             |
| **CVACT**               | 호주 캔버라 (도심)   | 지상 파노라마, 위성          | 도심 환경, 정렬된 뷰                         | 44,416 쌍          | 2018             |
| **VIGOR**               | 미국 4개 도시        | 지상 파노라마, 위성          | 비정렬 뷰, Semi-positives, Cross-Area 일반화 | ~195k 이미지       | 2021             |
| **University-1652**     | 701개 대학           | 드론, 위성                   | 1:N 매칭, 건물 단위 인식                     | ~50k 이미지        | 2020             |
| **CVG-Text**            | 3개 도시 (글로벌)    | 지상, 위성, OSM, **텍스트**  | 자연어 기반 검색, 다중 모달리티 융합         | 30,000+ 좌표       | 2024             |
| **SetVL-480K**          | 6개 도시 (글로벌)    | 지상 **이미지 셋**, 위성     | Set-based 쿼리, 다중 시점 융합               | 480,000+ 이미지    | 2024             |
| **CVGlobal**            | 글로벌 도시          | 지상, 위성, **지도**         | 글로벌 다양성, 시간 변화(Cross-temporal)     | -                  | 2024             |

*표 1. 주요 크로스뷰 지오로컬라이제이션 데이터셋 특성 비교*

| 모델 (Model)   | R@1 (Same-Area) | R@1 (Cross-Area) | 주요 방법론 (Key Methodology)            | 발표 (Conference/Year) |      |
| -------------- | --------------- | ---------------- | ---------------------------------------- | ---------------------- | ---- |
| **SAFA**       | 33.93%          | 8.20%            | 어텐션 기반 특징 집계                    | CVPR 2020              |      |
| **TransGeo**   | 61.48%          | 18.99%           | 순수 트랜스포머 기반 인코더              | CVPR 2022              |      |
| **SAIG-D**     | 65.23%          | 33.05%           | 의미론적 정렬 및 기하학적 추론           | -                      |      |
| **Sample4Geo** | **77.86%**      | **61.70%**       | **하드 네거티브 샘플링, 대칭적 InfoNCE** | **ICCV 2023**          |      |

표 2. VIGOR 데이터셋(Cross-Area) 기반 최신 모델 성능 비교 (R@1). Sample4Geo가 일반화 성능에서 압도적인 우위를 보임을 확인할 수 있다. 18 기반으로 재구성.

## 6.  기술적 난제와 미래 연구 방향

### 6.1  일반화 성능 및 도메인 적응 문제 (Generalization and Domain Adaptation)

현재 CVGL 기술이 직면한 가장 큰 난제는 단연 일반화(generalization) 성능이다. VIGOR 데이터셋의 cross-area 실험 결과에서 명확히 드러나듯이, 특정 도시나 환경에서 학습된 모델이 한 번도 보지 못한 새로운 환경에서는 성능이 급격히 저하되는 경향이 있다.18 이는 모델이 위치의 본질적인 구조를 학습하기보다 훈련 데이터에 존재하는 특정 스타일(e.g., 건물 양식, 식생, 도로 표지판)에 과적합되기 때문이다. 이 문제를 해결하기 위해서는 특정 도메인에 종속되지 않는, 보다 보편적이고 추상적인 특징 표현을 학습하는 것이 관건이며, 도메인 적응(domain adaptation) 및 도메인 일반화(domain generalization) 기술의 도입이 시급하다.

### 6.2  비지도/준지도 학습의 가능성 (Unsupervised/Semi-supervised Learning)

대부분의 성공적인 CVGL 모델은 정확하게 위치가 쌍으로 연결된(paired) 대규모의 레이블링된 데이터셋을 필요로 한다. 그러나 이러한 데이터를 수집하고 정렬하는 작업은 막대한 비용과 시간을 요구한다.33 현실 세계에는 레이블이 없는(unlabeled) 방대한 양의 지상 및 항공 이미지가 존재한다. 따라서 이러한 비지도(unsupervised) 데이터를 효과적으로 활용하는 것은 CVGL 기술의 확장성과 실용성을 위해 필수적이다. 최근에는 레이블이 없는 데이터로부터 자동으로 의사 레이블(pseudo-label)을 생성하고, 이를 소량의 레이블된 데이터와 함께 사용하여 점진적으로 모델 성능을 개선하는 준지도(semi-supervised) 학습 방식이 활발히 연구되고 있다.5

### 6.3  설명가능 AI(XAI)와 능동적 지오로컬라이제이션 (Explainable AI & Active Geo-localization)

모델이 특정 위치를 정답으로 판단한 근거를 제시하지 못하는 '블랙박스' 문제는 기술의 신뢰성을 저해하는 주요 요인이다. 특히 자율주행이나 구조 활동과 같이 안전이 중요한 분야에서는 모델의 판단 과정을 인간이 이해하고 신뢰할 수 있어야 한다. 따라서 모델이 "이미지의 이 부분과 위성 지도의 저 부분이 유사하기 때문에 여기라고 판단했다"와 같이 시각적 또는 텍스트적으로 근거를 설명하는 설명가능 AI(XAI)의 도입이 매우 중요하다. `CrossText2Loc` 모델이 검색 이유를 제공하거나 `GeoGuess` 태스크가 상세한 설명을 요구하는 것은 이러한 흐름을 반영한다.24

더 나아가, 능동적 지오로컬라이제이션(Active Geo-localization, AGL)은 드론과 같은 에이전트가 목표물을 찾기 위해 단순히 주어진 이미지를 분석하는 것을 넘어, 스스로 이동하며 순차적으로 얻는 시각 정보를 통합하여 위치를 추론하는 보다 동적이고 복잡한 문제 설정이다.13 이는 미래의 지능형 로봇이 수행해야 할 탐색 및 측위 작업의 핵심 기술이 될 것이다.

### 6.4  프라이버시 및 감시와 관련된 윤리적 고찰 (Ethical Considerations: Privacy and Surveillance)

지오로컬라이제이션 기술은 그 강력한 성능만큼이나 심각한 윤리적 문제를 내포하고 있다. 이 기술은 이론적으로 어떤 이미지로부터든 촬영 위치를 추정할 수 있으므로, 개인의 동의 없이 일상 사진만으로 동선이나 거주지를 파악하는 등 심각한 프라이버시 침해를 야기할 수 있다.15 또한, 국가나 특정 집단에 의해 대규모 감시 도구로 악용될 가능성도 존재한다. 따라서 데이터셋을 구축할 때부터 개인을 식별할 수 있는 정보를 제거하는 등 프라이버시 보호 조치가 반드시 고려되어야 하며 34, 기술의 개발과 함께 오용을 방지하기 위한 법적, 제도적, 사회적 논의가 병행되어야 한다.

## 7.  결론: 차세대 자율 시스템을 위한 핵심 기술로서의 전망

### 8.1. 핵심 연구 결과 요약 및 기술적 의의

ICCV 2023을 포함한 최근 연구 동향을 종합해 볼 때, 크로스뷰 지오로컬라이제이션 기술은 중요한 변곡점을 맞이하고 있다. 과거의 연구가 더 복잡하고 정교한 네트워크 아키텍처를 설계하는 데 집중했다면, 이제는 연구의 초점이 다음과 같은 방향으로 이동하고 있다. 첫째, `Sample4Geo`가 보여주었듯, 지능적인 데이터 샘플링과 같은 학습 전략의 고도화를 통해 모델의 본질적인 학습 능력을 극대화하는 방향이다. 둘째, `View Consistent Purification`처럼 실제 환경에 존재하는 동적 객체나 노이즈를 명시적으로 처리하여 시스템의 강인성을 확보하는 방향이다.

또한, 텍스트, 3D 포인트 클라우드, 시맨틱 맵 등 다중 모달리티 정보의 적극적인 융합은 CVGL을 단순한 이미지 검색 문제를 넘어, 여러 종류의 데이터를 이해하고 종합하여 결론을 내리는 복합적인 공간 추론(spatial reasoning) 문제로 격상시키고 있다. 이는 인공지능이 인간의 공간 인지 능력에 한 걸음 더 다가서고 있음을 의미한다.

### 7.1  미래 기술 발전 방향 및 산업적 파급 효과 전망

향후 CVGL 기술은 한 번도 보지 못한 환경에 대한 높은 일반화 성능, 레이블 없는 데이터를 활용하는 비지도 학습 능력, 그리고 인간과 상호작용하고 자신의 판단을 설명할 수 있는 설명가능성을 갖추는 방향으로 발전할 것이다. 이러한 기술적 성숙은 다양한 미래 산업에 막대한 파급 효과를 가져올 것으로 기대된다.

완전 자율주행 시스템은 CVGL을 통해 모든 도로와 기상 조건에서 끊김 없는 고정밀 측위를 보장받을 수 있다. 도심 항공 모빌리티(UAM)는 복잡한 도시 상공에서 안전한 항법을 위한 핵심 보조 시스템으로 이를 활용할 것이다. 지능형 배송 로봇과 서비스 로봇은 실내외를 넘나들며 자신의 위치를 정확히 파악하고 임무를 수행할 수 있게 된다. 이처럼, 다중 모달리티 크로스뷰 지오로컬라이제이션은 차세대 자율 시스템의 '눈'과 '공간 인지 능력'을 책임지는 핵심 기반 기술로서, 그 중요성이 날로 커질 것으로 전망된다.

#### **참고 자료**

1. (PDF) Cross-View Geo-Localization: A Survey - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/386207589_Cross-view_Geo-localization_A_Survey
2. Cross-View Image Sequence Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/WACV2023/papers/Zhang_Cross-View_Image_Sequence_Geo-Localization_WACV_2023_paper.pdf
3. Multimodal Drone Cross-view Geo-localization Based on Vision-language Model, accessed July 1, 2025, https://robot.sia.cn/en/article/doi/10.13973/j.cnki.robot.240283?viewType=citedby-info
4. Cross-Attention Between Satellite and Ground Views for Enhanced Fine-Grained Robot Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/WACV2024/papers/Yuan_Cross-Attention_Between_Satellite_and_Ground_Views_for_Enhanced_Fine-Grained_Robot_WACV_2024_paper.pdf
5. Unleashing Unlabeled Data: A Paradigm for Cross-View Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Li_Unleashing_Unlabeled_Data_A_Paradigm_for_Cross-View_Geo-Localization_CVPR_2024_paper.pdf
6. CVPR 2023: A Comprehensive Tour and Recent Advancements toward Real-world Visual Geo-Localization - SRI, accessed July 1, 2025, https://www.sri.com/research/information-computing-sciences/computer-vision/cvpr-2023-a-comprehensive-tour-and-recent-advancements-toward-real-world-visual-geo-localization/
7. CVPR 2021 tutorial on Cross-view and Cross-modal Visual Geo-Localization - SRI, accessed July 1, 2025, https://www.sri.com/research/information-computing-sciences/computer-vision/cvpr-2021-tutorial-on-cross-view-and-cross-modal-visual-geo-localization/
8. Feature Relation Guided Cross-View Image Based Geo-Localization - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/15/20/5029
9. Semantic Cross-View Matching - Stanford Computational Vision and Geometry Lab, accessed July 1, 2025, https://cvgl.stanford.edu/papers/castaldo_iccv15.pdf
10. Cross-view geo-localization: a survey - arXiv, accessed July 1, 2025, https://arxiv.org/html/2406.09722v1
11. View Consistent Purification for Accurate Cross-View Localization - ICCV 2023 Open Access Repository, accessed July 1, 2025, https://openaccess.thecvf.com/content/ICCV2023/html/Wang_View_Consistent_Purification_for_Accurate_Cross-View_Localization_ICCV_2023_paper.html
12. UAV Geo-Localization Dataset and Method Based on Cross-View Matching - PMC, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11548418/
13. GOMAA-Geo: GOal Modality Agnostic Active Geo-localization - OpenReview, accessed July 1, 2025, [https://openreview.net/forum?id=gPCesxD4B4&referrer=%5Bthe%20profile%20of%20Yevgeniy%20Vorobeychik%5D(%2Fprofile%3Fid%3D~Yevgeniy_Vorobeychik1)](https://openreview.net/forum?id=gPCesxD4B4&referrer=[the+profile+of+Yevgeniy+Vorobeychik](/profile?id%3D~Yevgeniy_Vorobeychik1))
14. NeurIPS Poster GOMAA-Geo: GOal Modality Agnostic Active Geo-localization, accessed July 1, 2025, https://neurips.cc/virtual/2024/poster/94144
15. A news picture geo-localization pipeline based on deep learning and street view images, accessed July 1, 2025, https://www.tandfonline.com/doi/full/10.1080/17538947.2022.2121437
16. Leveraging cross-view geo-localization with ensemble learning and temporal awareness, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10062671/
17. IML-Net: A Framework for Cross-View Geo-Localization with Multi-Domain Remote Sensing Data - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/16/7/1249
18. Sample4Geo: Hard Negative Sampling For ... - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Deuser_Sample4Geo_Hard_Negative_Sampling_For_Cross-View_Geo-Localisation_ICCV_2023_paper.pdf
19. Cross-view image geo-localization with Panorama-BEV Co-Retrieval Network - European Computer Vision Association, accessed July 1, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/05379.pdf
20. [2408.05475] Cross-view image geo-localization with Panorama-BEV Co-Retrieval Network, accessed July 1, 2025, https://arxiv.org/abs/2408.05475
21. 52CV/ICCV-2023-Papers - GitHub, accessed July 1, 2025, https://github.com/52CV/ICCV-2023-Papers
22. Where am I? Cross-View Geo-localization with Natural Language Descriptions - arXiv, accessed July 1, 2025, https://arxiv.org/html/2412.17007v1
23. [Literature Review] Where am I? Cross-View Geo-localization with Natural Language Descriptions - Moonlight | AI Colleague for Research Papers, accessed July 1, 2025, https://www.themoonlight.io/en/review/where-am-i-cross-view-geo-localization-with-natural-language-descriptions
24. [2412.17007] Where am I? Cross-View Geo-localization with Natural Language Descriptions, accessed July 1, 2025, https://arxiv.org/abs/2412.17007
25. Where am I? Cross-View Geo-localization with Natural Language Descriptions - arXiv, accessed July 1, 2025, https://arxiv.org/html/2412.17007v2
26. yejy53/CVG-Text - GitHub, accessed July 1, 2025, https://github.com/yejy53/CVG-Text
27. Cross-View Geo-localization with Natural Language Descriptions - Junyan Ye, accessed July 1, 2025, https://yejy53.github.io/CVG-Text/
28. Optimal Feature Transport for Cross-View Image Geo-Localization - GitHub, accessed July 1, 2025, https://github.com/YujiaoShi/cross_view_localization_CVFT
29. Crossview USA (CVUSA) - MVRL, accessed July 1, 2025, https://mvrl.cse.wustl.edu/datasets/cvusa/
30. VIGOR: Cross-View Image Geo-Localization Beyond One-to-One Retrieval - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Zhu_VIGOR_Cross-View_Image_Geo-Localization_Beyond_One-to-One_Retrieval_CVPR_2021_paper.pdf
31. VIGOR Dataset - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/dataset/vigor
32. Cross-View Image Set Geo-Localization - arXiv, accessed July 1, 2025, https://arxiv.org/html/2412.18852v1
33. Unleashing Unlabeled Data: A Paradigm for Cross-View Geo-Localization - arXiv, accessed July 1, 2025, https://arxiv.org/html/2403.14198v1
34. Game4Loc: A UAV Geo-Localization Benchmark from Game Data | Proceedings of the AAAI Conference on Artificial Intelligence, accessed July 1, 2025, https://ojs.aaai.org/index.php/AAAI/article/view/32409
35. GeoGuess: Multimodal Reasoning based on Hierarchy of Visual Information in Street View - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2506.16633
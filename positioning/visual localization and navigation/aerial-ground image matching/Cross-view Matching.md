# Cross-view Matching

## 서론: Cross-view Matching의 서막: 기본 개념과 비전

컴퓨터 비전 분야는 인간의 시각적 인식 능력을 기계로 구현하려는 목표를 향해 끊임없이 발전해왔다. 이 과정에서 가장 근본적이면서도 도전적인 과제 중 하나는 서로 다른 조건에서 포착된 시각 정보 간의 의미론적 연결고리를 찾는 것이다. 'Cross-view Matching(Cross-view Matching)'은 이러한 도전의 최전선에 있는 기술로, 서로 다른 플랫폼이나 극단적으로 다른 시점(viewpoint)에서 촬영된 이미지들 속에서 동일한 객체나 장면을 식별하고 대응시키는 컴퓨터 비전의 한 분야이다.1 대표적인 예로, 지상에서 사람이 촬영한 스트리트뷰 이미지와 하늘 위 인공위성이 촬영한 항공 이미지를 매칭하여 "이 사진이 어디에서 촬영되었는가?"라는 질문에 답하는 것을 들 수 있다.

본 보고서에서 다루는 컴퓨터 비전의 'Cross-view Matching'은 데이터베이스 분야에서 사용되는 '크로스 테이블(Cross Table)' 5이나 데이터 집합을 연결하는 '조인(JOIN)' 연산 6과는 개념적으로 완전히 구별된다. 후자가 정형화된 데이터 테이블 간의 관계를 정의하는 것이라면, 전자는 비정형 시각 데이터에 내재된 의미적, 기하학적 관계를 이해하려는 시도라는 점에서 본질적인 차이가 있다. 이러한 용어의 명확화는 본 기술의 고유한 특성과 난이도를 이해하는 첫걸음이다.

### 0.1  핵심 과제: 대응점 문제(Correspondence Problem)의 확장

Cross-view Matching의 기술적 뿌리는 컴퓨터 비전의 가장 근본적인 문제로 알려진 '대응점 문제(Correspondence Problem)'에 있다.7 컴퓨터 비전의 선구자 카나데 타케오(Takeo Kanade)가 "컴퓨터 비전의 세 가지 근본적인 문제는 대응, 대응, 그리고 대응이다"라고 언급했을 정도로, 대응점 문제는 3차원 장면을 서로 다른 시점에서 촬영한 두 개 이상의 이미지에서 동일한 지점을 찾아 연결하는 모든 시각 이해 작업의 기초가 된다.7

Cross-view Matching은 이 대응점 문제를 가장 극단적인 시나리오로 확장한 것이다. 일반적인 대응점 문제는 스테레오 비전이나 파노라마 생성처럼 시점 변화가 비교적 작고 제어된 환경을 다루는 경우가 많다. 하지만 Cross-view Matching은 지상 뷰와 항공 뷰처럼 시점이 90도 이상 차이 나고, 원근 왜곡과 객체의 외형이 완전히 달라지는 환경을 다룬다.8 예를 들어, 위성 이미지에서는 건물의 '지붕'이 보이지만 지상 이미지에서는 '정면'이 보인다. 이처럼 픽셀 수준의 외형은 완전히 다르지만, 인간은 이를 '같은 건물'이라는 의미론적(semantic) 정보로 인식한다. 초기 컴퓨터 비전 알고리즘이 이러한 '의미론적 격차(semantic gap)'를 극복하지 못했기 때문에, Cross-view Matching은 단순한 특징점 비교를 넘어 장면의 구조와 맥락을 이해하는 방향으로 발전할 수밖에 없었다. 따라서 이 분야의 기술적 진화는 의미론적 격차를 해소하려는 끊임없는 노력의 역사와 같다.

### 0.2  유사 기술과의 비교 분석: 스테레오 매칭, 이미지 스티칭

Cross-view Matching의 독창성을 이해하기 위해, 대응점 문제에 기반한 다른 컴퓨터 비전 기술들과 비교 분석하는 것이 유용하다. 스테레오 매칭(Stereo Matching)과 이미지 스티칭(Image Stitching)은 대표적인 관련 기술이지만, 목표와 해결하려는 문제의 성격에서 뚜렷한 차이를 보인다.

**표 1: 유사 컴퓨터 비전 기술 비교**

| 기술 (Task)                                   | 목표 (Goal)                                                  | 입력 (Input)                                                 | 시점 차이 (Viewpoint Difference) | 핵심 난제 (Core Challenge)                                   |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------- | ------------------------------------------------------------ |
| **Cross-view Matching (Cross-view Matching)** | 서로 다른 플랫폼/시점의 이미지에서 동일 객체/장면 식별       | 지상 이미지, 항공/위성 이미지 등 이종(heterogeneous) 이미지 쌍 | 매우 큼 (예: 90도 이상)          | 극심한 외형 및 기하학적 변환, 의미론적 격차(semantic gap) 극복 8 |
| **스테레오 매칭 (Stereo Matching)**           | 정류된(rectified) 이미지 쌍으로부터 깊이(depth) 또는 시차(disparity) 맵 추정 | 동일 플랫폼에서 촬영된 동종(homogeneous) 이미지 쌍           | 작고 제어됨                      | 폐색(occlusion), 텍스처가 없는 영역, 반복적인 패턴에서의 정확한 시차 계산 10 |
| **이미지 스티칭 (Image Stitching)**           | 여러 장의 겹치는 이미지를 하나의 파노라마 이미지로 합성      | 일부 겹치는 영역을 가진 동종 이미지 시퀀스                   | 작거나 중간 정도                 | 정확한 이미지 정렬(alignment), 노출 차이 및 시차(parallax)로 인한 블렌딩(blending) 오류 최소화 12 |

스테레오 매칭은 주로 3D 재구성이나 깊이 인식을 위해 작은 시점 차이를 활용하는 반면, Cross-view Matching은 극심한 시점 차이 자체를 극복해야 하는 과제를 안고 있다.10 이미지 스티칭은 시각적으로 부드러운 파노라마를 만드는 것이 목표이지만, Cross-view Matching은 외형적으로 전혀 다른 두 이미지가 동일한 지리적 실체(geographic entity)를 가리키는지 여부를 판단하는 것이 목표다.7 한편, 일반적인 용어로 사용되는 '교차 시야(cross-view)'는 스테레오스코픽 이미지를 3D로 인지하기 위해 두 눈을 교차시키는 인간의 시각 기법을 지칭하는 것으로, 본 보고서에서 다루는 컴퓨터 비전 기술과는 무관하다.14 이처럼 Cross-view Matching은 다른 기술들과 근본적인 문제 의식을 공유하면서도, 훨씬 더 어렵고 비구조적인 조건에서 의미론적 이해를 요구한다는 점에서 독자적인 영역을 구축하고 있다.

### 0.3  초기 접근법: SIFT와 SURF를 이용한 특징점 기반 매칭

딥러닝 기술이 보편화되기 이전, Cross-view Matching을 포함한 이미지 매칭 문제들은 주로 수작업 특징(hand-crafted features)에 기반한 알고리즘으로 해결되었다. 그중 가장 대표적인 것이 SIFT(Scale-Invariant Feature Transform)와 SURF(Speeded-Up Robust Features)이다.15 이 알고리즘들은 이미지의 크기 변화(scale), 회전(rotation), 조명 변화에 강인한 지역 특징점(keypoint)과 이를 설명하는 기술자(descriptor)를 추출하여 매칭의 근거로 삼는다.

- **SIFT (Scale-Invariant Feature Transform):** SIFT는 여러 단계의 정교한 과정을 거친다. 먼저, 입력 이미지에 다양한 수준의 가우시안 블러링(Gaussian blurring)을 적용하고 이미지 크기를 점진적으로 줄여나가는 가우시안 피라미드(Gaussian Pyramid)를 구축하여 '스케일 공간(scale space)'을 생성한다.17 이 스케일 공간에서 인접한 블러링 이미지 간의 차이를 계산한 가우시안 차분(Difference of Gaussians, DoG) 영상을 만든다.18 DoG 영상의 한 픽셀을 주변 26개 픽셀(공간적, 스케일적 이웃)과 비교하여 극대 또는 극소값을 갖는 지점을 특징점 후보로 선정한다.17 이렇게 찾은 특징점에 대해 주변 픽셀들의 그래디언트 방향 히스토그램을 계산하여 주된 방향(dominant orientation)을 할당함으로써 회전 불변성을 확보한다. 마지막으로, 특징점의 주된 방향을 기준으로 16x16 픽셀 영역을 정규화하고, 이를 4x4의 작은 영역으로 나누어 각 영역의 그래디언트 방향 히스토그램을 계산한다. 이들을 모두 연결하여 128차원의 벡터, 즉 SIFT 기술자를 생성한다.17
- **SURF (Speeded-Up Robust Features):** SURF는 SIFT의 강력한 성능을 유지하면서 계산 속도를 획기적으로 개선한 알고리즘이다.15 SIFT가 이미지 자체의 크기를 바꾸는 대신, SURF는 적분 이미지(Integral Image)라는 자료구조를 활용하여 이미지 크기는 고정한 채 박스 필터(Box Filter)의 크기를 바꿔가며 스케일 공간을 근사한다.19 적분 이미지를 사용하면 특정 영역의 픽셀 합을 매우 빠르게 계산할 수 있어, 헤시안 행렬(Hessian Matrix)의 행렬식(determinant)을 효율적으로 근사 계산하는 것이 가능하다.17 이 행렬식의 값이 극대인 지점을 특징점으로 검출한다. 특징점의 방향은 Haar-wavelet 응답을 이용하여 결정하며, 기술자는 주변 영역의 Haar-wavelet 응답 합계를 기반으로 64차원 벡터로 생성되어 SIFT보다 더 작고 빠르다.17

이러한 고전적인 방법들은 조명이나 회전 등 일반적인 변환에 대해서는 뛰어난 성능을 보였지만, Cross-view Matching의 근본적인 난제 앞에서는 한계를 드러냈다. 지상 뷰와 항공 뷰 간의 극심한 원근 왜곡과 시점 변화는 SIFT나 SURF가 의존하는 지역적인 그래디언트 패턴과 픽셀 구조를 완전히 바꿔버린다.8 위성 사진 속 건물의 지붕과 지상 사진 속 건물의 벽면은 픽셀 수준에서 어떠한 공통점도 찾기 어렵다. 결국 이 알고리즘들은 '의미론적 격차'를 해소하지 못하고, 안정적인 대응점을 찾는 데 실패했다.21 이러한 한계는 단순히 더 강인한 특징점을 찾는 수준을 넘어, 장면의 전체적인 구조와 의미를 이해할 수 있는 새로운 패러다임, 즉 딥러닝 기반 접근법의 등장을 필연적으로 만들었다.22

## 1.  핵심 응용 분야: 크로스뷰 지리적 위치 추정 (CVGL)

Cross-view Matching 기술의 수많은 응용 분야 중 가장 활발하게 연구되고 있으며 가장 큰 파급력을 지닌 분야는 단연 '크로스뷰 지리적 위치 추정(Cross-View Geo-Localization, CVGL)'이다. CVGL은 자율 시스템의 항법 능력과 직결되는 핵심 기술로, 이 분야의 발전을 견인하는 주요 동력 역할을 하고 있다. 본 장에서는 CVGL의 정의와 중요성을 살펴보고, 자율주행, 로보틱스 등 구체적인 응용 사례를 탐구한 뒤, 이 기술이 극복해야 할 복합적인 난제들을 심층적으로 분석한다.

### 1.1  CVGL의 정의와 중요성: GPS 음영 지역에서의 자율 항법

CVGL은 위치 정보가 없는 쿼리 이미지(query image, 예: 차량의 블랙박스 영상 캡처)가 어디에서 촬영되었는지를, GPS와 같은 지리 정보 태그가 부착된 방대한 참조 이미지 데이터베이스(reference image database, 예: 위성 지도)와 매칭하여 알아내는 작업이다.1 기술적으로 이 과정은 주어진 쿼리 이미지와 가장 유사한 참조 이미지를 데이터베이스에서 찾아내는 이미지 검색(Image Retrieval) 문제로 정의될 수 있다.3

CVGL의 중요성은 현대 자율 시스템이 크게 의존하는 위성 항법 시스템(GNSS/GPS)의 본질적인 한계에서 비롯된다. GNSS 신호는 고층 빌딩이 밀집한 도심 협곡(urban canyon), 터널, 실내, 혹은 짙은 숲 아래에서 신호가 차단되거나 다중 경로 반사(multipath effect)로 인해 심각한 오차를 유발할 수 있다.24 이러한 'GPS 음영 지역'에서 자율주행차나 로봇, 무인 항공기(UAV)가 자신의 위치를 잃는 것은 치명적인 사고로 이어질 수 있다. CVGL은 바로 이러한 상황에서 시각 정보만을 이용해 위치를 추정함으로써, GNSS의 보조 수단(supplement) 또는 완전한 대체 수단(replacement)으로서 기능한다.1 이는 자율 시스템의 운행 가능 영역을 확장하고 안전성과 신뢰성을 획기적으로 높이는 데 결정적인 역할을 한다.

### 1.2  주요 응용 사례: 자율주행, 로보틱스, 무인 항공기(UAV), 증강현실

CVGL 기술은 높은 정밀도와 강건성이 요구되는 다양한 미래 산업 분야에서 핵심적인 역할을 수행할 것으로 기대된다.

- **자율주행 및 로보틱스:** 자율주행차나 배송 로봇은 차량에 장착된 카메라로 촬영한 실시간 주변 환경 이미지를 사전에 구축된 고정밀 항공 이미지 데이터베이스와 매칭하여 자기 위치를 정밀하게 추정(self-localization)할 수 있다.1 이는 GPS 오차를 보정하고, 차선 단위의 정밀한 주행을 가능하게 하는 핵심 기술이다.
- **무인 항공기(UAV):** 군사 작전이나 재난 지역 탐사와 같이 GPS 신호가 의도적으로 교란되거나(jamming) 수신이 불가능한 환경에서 UAV의 자율 항법은 매우 중요하다. CVGL은 UAV가 비행 중 촬영하는 하방 카메라 이미지를 위성 지도와 실시간으로 대조하여, 외부 도움 없이 스스로 경로를 계획하고 임무를 수행할 수 있게 하는 핵심 기술로 활용된다.2
- **증강현실(AR):** 실외에서 몰입감 있는 AR 경험을 제공하기 위해서는 가상의 디지털 객체를 현실 세계의 정확한 위치에 흔들림 없이 고정해야 한다. 이를 위해서는 사용자의 스마트폰이나 AR 글래스의 정확한 지리적 자세(geo-pose), 즉 위치와 방향을 알아야 한다.28 CVGL은 사용자의 카메라 뷰를 위성 지도와 매칭하여 이 지오-포즈를 정밀하게 추정함으로써, 가상 포켓몬이 특정 건물 앞에 정확히 나타나게 하거나, 가상의 길 안내 화살표가 실제 도로 위에 그려지도록 하는 것을 가능하게 한다.26

### 1.3  기술적 난제 심층 분석

CVGL이 이처럼 유망한 기술임에도 불구하고, 실제 환경에 널리 적용되기까지는 수많은 기술적 난제를 해결해야 한다. 이 난제들은 단일한 문제가 아니라 여러 요인이 복합적으로 얽혀 있어, 단순히 우수한 매칭 알고리즘 하나만으로는 해결이 어렵다. 이는 CVGL이 단순한 알고리즘 수준의 문제를 넘어, 다양한 모듈이 유기적으로 결합된 '시스템 수준(system-level)'의 접근이 필요함을 시사한다.

#### 1.3.1  극심한 시점 및 외형 변화

CVGL의 가장 근본적이고 어려운 과제는 지상 뷰와 항공 뷰 사이의 극심한 시점 차이에서 비롯되는 외형 및 기하학적 불일치이다.2 지상에서는 수직으로 보이는 건물의 벽면이 항공 뷰에서는 점이나 면으로 보이고, 도로의 폭이나 건물의 상대적 위치 관계가 완전히 다르게 투영된다. 이러한 차이는 전통적인 특징 기반 매칭을 거의 불가능하게 만들며, 딥러닝 모델에게도 여전히 가장 큰 도전 과제이다.

#### 1.3.2  환경적 요인

동일한 장소라 할지라도 촬영 조건에 따라 이미지는 천차만별로 달라질 수 있다.

- **스케일 및 방향(Scale and Orientation):** 항공 이미지는 넓은 지역을 일정한 스케일로 담지만, 지상 이미지는 촬영 거리에 따라 객체의 스케일이 크게 변한다.2 더욱이, 대부분의 경우 지상 쿼리 이미지의 촬영 방향(북쪽이 어디인지)을 알 수 없기 때문에, 매칭을 시도해야 할 가능한 방향의 탐색 공간이 매우 넓어져 모호성과 계산 복잡도를 증가시킨다.31
- **조명, 날씨, 계절(Illumination, Weather, Season):** 하루 중 시간대에 따른 그림자의 변화, 맑은 날과 흐린 날의 조명 차이, 여름의 무성한 나뭇잎과 겨울의 앙상한 가지 등 계절적 변화는 동일한 장소의 외형을 극적으로 바꾸어 매칭을 어렵게 만든다.8

#### 1.3.3  장면의 복잡성

현실 세계의 장면은 정적이지 않고 복잡한 요소들로 가득 차 있다.

- **객체 가림(Occlusion):** 키가 큰 건물, 나무, 대형 차량 등이 중요한 랜드마크나 도로를 가려서 매칭에 필요한 핵심 정보를 유실시킬 수 있다.22
- **동적 객체(Dynamic Objects):** 도로 위의 자동차, 버스, 보행자 등은 끊임없이 움직이는 '비정형적 소음'이다.9 이들은 특정 위치의 고유한 특징이 아니므로, 매칭 과정에서 오히려 방해 요소(redundant features)로 작용하여 성능을 저하시킨다. 강건한 CVGL 시스템은 이러한 동적 객체를 탐지하고 특징 추출 과정에서 무시하는 능력을 갖추어야 한다.
- **유사한 장면(Similar Scenes):** 격자형 도시 구조에서 많은 도로는 시각적으로 매우 유사하게 보인다. 이러한 장소들은 외형적으로는 비슷하지만 지리적으로는 완전히 다른 위치에 있어, 모델이 쉽게 혼동하고 오매칭을 유발하는 '하드 네거티브(hard negative)' 샘플로 작용한다.21

#### 1.3.4  데이터 및 센서 제약

사용 가능한 데이터와 센서의 물리적 한계 또한 중요한 제약 조건이다.

- **제한된 시야각(Limited Field of View, FoV):** 초기 CVGL 연구들은 360도 파노라마 이미지를 사용하여 최대한 많은 정보를 확보하는 데 초점을 맞췄다.1 하지만 스마트폰이나 차량용 블랙박스 등 대부분의 상용 카메라는 시야각이 제한적이다.31 제한된 FoV의 이미지는 담고 있는 정보량이 훨씬 적어 매칭의 난이도를 급격히 높인다.1 이 '제한된 FoV' 문제는 CVGL 기술이 고가의 연구용 프로토타입을 넘어, 일반 소비자가 사용하는 기기에 탑재되기 위해 반드시 해결해야 할 핵심 병목 구간이다. 이 문제의 해결은 기술의 상업적 대중화를 여는 열쇠와 같다.
- **시간적 불일치(Temporal Misalignment):** 참조용 위성 이미지는 수개월 또는 수년 전에 촬영된 것일 수 있다. 그 사이에 새로운 건물이 들어서거나 기존 건물이 철거되는 등 장면 자체가 변할 수 있어 데이터베이스와 실제 환경 간의 불일치를 야기한다.21
- **데이터셋 문제:** 대부분의 연구는 여전히 단일 이미지를 독립적으로 매칭하는 작업에 집중하고 있어, 실제 주행 환경처럼 연속적인 이미지 스트림에서 얻을 수 있는 시간적 맥락(temporal context)을 간과하고 있다.21 또한, 이러한 연속적인 궤적을 현실적으로 담아낸 대규모 고품질 데이터셋도 부족한 실정이다.21 강건한 위치 추정을 위해서는 파티클 필터(Particle Filter)와 같은 시간적 필터링 기법을 통합하여 현재의 불확실한 추정치를 과거의 안정적인 추정치와 결합하는 시스템 수준의 접근이 필수적이다.1

## 2.  딥러닝 혁명: CVGL을 위한 현대적 접근법

SIFT와 같은 수작업 특징 기반 방법론의 명백한 한계에 직면하면서, CVGL 분야는 2010년대 중반부터 딥러닝을 전면적으로 수용하며 패러다임의 전환을 맞이했다. 이는 단순히 알고리즘을 바꾸는 것을 넘어, 특징을 '설계'하는 시대에서 데이터로부터 특징을 '학습'하는 시대로의 근본적인 변화였다. 본 장에서는 샴 네트워크를 시작으로 GAN, 그리고 현재 주류로 자리 잡은 트랜스포머에 이르기까지, CVGL을 위한 현대적 딥러닝 접근법의 발전 계보를 추적하고 각 기술의 핵심 원리와 혁신성을 심층적으로 분석한다.

### 2.1  패러다임의 전환: 특징 공학에서 엔드투엔드 학습으로

딥러닝 이전의 접근법은 개발자가 직접 특징 추출 방식을 고안하는 특징 공학(feature engineering)에 의존했다. 그러나 이는 극심한 시점 변화에 강인한 특징을 설계하는 데 근본적인 어려움이 있었다. 딥러닝, 특히 컨볼루션 신경망(Convolutional Neural Network, CNN)의 등장은 이러한 판도를 바꾸었다.22 대규모 이미지 데이터셋을 통해 모델이 스스로 데이터에 내재된 패턴을 학습하여, 특정 작업에 최적화된 강력한 특징 표현(feature representation)을 엔드투엔드(end-to-end) 방식으로 추출할 수 있게 된 것이다. Workman과 Jacobs가 AlexNet을 CVGL에 처음 도입한 연구는 딥러닝 특징이 기존의 수작업 특징보다 월등한 성능을 보임을 입증하며 새로운 시대의 서막을 열었다.32 이후 CVGL 연구는 샴 네트워크, 삼중항 손실 등 다양한 딥러닝 아키텍처와 학습 기법을 탐구하는 방향으로 빠르게 전개되었다.

### 2.2  샴 네트워크(Siamese Network)와 거리 학습(Metric Learning)

딥러닝 기반 CVGL의 초기 모델들은 대부분 샴 네트워크 구조를 채택했다.

- **기본 구조:** 샴 네트워크는 동일한 구조와 가중치를 공유하는(weight-sharing) 두 개의 동일한 CNN 브랜치로 구성된다.22 하나의 브랜치에는 지상 이미지가, 다른 브랜치에는 항공 이미지가 입력된다. 각 CNN은 이미지를 처리하여 고차원의 특징 벡터로 변환하는데, 이 변환된 벡터가 존재하는 가상의 공간을 '임베딩 공간(embedding space)'이라고 한다.
- **학습 목표:** 샴 네트워크의 학습 목표는 이 임베딩 공간의 구조를 잘 만드는 것이다. 즉, '의미적으로 유사한 입력은 가깝게, 그렇지 않은 입력은 멀게' 배치하는 것이다. 이를 위해 거리 학습(Metric Learning) 기법이 사용된다. 대표적인 손실 함수인 삼중항 손실(Triplet Loss)은 기준이 되는 앵커(anchor) 이미지, 앵커와 같은 위치에서 촬영된 긍정(positive) 이미지, 그리고 전혀 다른 위치에서 촬영된 부정(negative) 이미지, 이렇게 세 개의 샘플을 한 단위(triplet)로 사용한다.24 학습 과정에서 모델은 앵커와 긍정 샘플 간의 거리는 최소화하고, 앵커와 부정 샘플 간의 거리는 특정 마진(margin) 이상으로 최대화하도록 강제된다.29 이러한 방식으로 학습된 모델은 새로운 이미지가 주어졌을 때, 이를 임베딩 공간의 적절한 위치로 매핑하여 가장 가까운 이웃을 찾는 방식으로 매칭을 수행할 수 있다. 샴 네트워크는 적은 데이터로도 효과적인 학습이 가능하여, 소량의 데이터만으로 학습하는 Few-shot learning, 특히 단 한 장의 데이터로 학습하는 One-shot learning의 핵심적인 기반 기술이기도 하다.36

### 2.3  생성적 적대 신경망(GAN)의 활용: 도메인 적응을 통한 시점 변환

샴 네트워크가 임베딩 공간에서의 '거리'를 학습하는 간접적인 방식으로 시점 차이를 극복하려 했다면, 생성적 적대 신경망(Generative Adversarial Network, GAN)은 더 직접적인 해법을 제시했다. GAN은 지상 뷰와 항공 뷰 사이의 극심한 '도메인 격차(domain gap)' 자체를 줄이는 것을 목표로 한다.23

- **접근법:** GAN 기반 CVGL 모델은 한쪽 뷰의 이미지를 다른 쪽 뷰의 스타일로 '번역(translation)'하는 이미지 생성 작업을 수행한다. 예를 들어, 생성자(Generator) 네트워크는 위성 이미지를 입력받아 그에 해당하는 사실적인 지상 파노라마 이미지를 생성하도록 학습된다. 판별자(Discriminator) 네트워크는 생성된 이미지가 실제 지상 이미지와 얼마나 유사한지를 판별하며, 이 둘의 적대적 경쟁을 통해 생성자의 성능이 향상된다. CVM-Net, CDtE와 같은 모델들은 이러한 이미지 합성과 매칭을 하나의 프레임워크 안에서 동시에 수행하도록 설계되었다.9
- **한계:** 이 접근법은 시각적으로 매우 인상적인 결과를 보여주었지만, 몇 가지 고질적인 문제를 안고 있었다. GAN의 학습 과정은 매우 불안정하여 최적의 균형점을 찾기 어렵고, 생성된 이미지의 다양성이 부족해지는 모델 붕괴(model collapse) 현상이 자주 발생한다. 또한, 고품질 이미지를 생성하기 위해 상당한 계산 자원을 소모하는 것도 실용화의 걸림돌로 작용했다.23

### 2.4  트랜스포머(Transformer)의 부상: 전역적 문맥 이해의 새로운 지평

CVGL 연구의 흐름은 암시적 거리 학습(샴 네트워크)에서 명시적 시점 변환(GAN)으로, 그리고 다시 더 강력한 암시적 학습 모델인 트랜스포머로 이동하는 흥미로운 궤적을 보인다. 이는 명시적인 기하학적 사전 지식을 주입하려는 시도와, 데이터로부터 복잡한 관계를 스스로 학습하게 하려는 시도 사이의 '밀고 당기기' 과정으로 해석될 수 있다. 결국, 충분히 강력한 모델이 데이터로부터 직접 기하학을 학습하는 것이 수작업으로 기하학을 설계하는 것보다 더 효과적이라는 방향으로 수렴하고 있으며, 트랜스포머는 그 중심에 있다.

#### 2.4.1  CNN 기반 모델의 한계와 극좌표 변환(Polar Transform)

트랜스포머의 부상을 이해하기 위해서는 기존 CNN 기반 모델의 근본적인 한계를 먼저 살펴봐야 한다. CNN의 핵심 연산인 컨볼루션(convolution)은 본질적으로 지역적(local)이다. 즉, 작은 필터(커널)를 이미지 전체에 이동시키며 특징을 추출하기 때문에, 이미지의 전역적인 구조나 멀리 떨어진 픽셀 간의 상관관계를 파악하는 데 구조적인 한계가 있다.38

이러한 한계를 보완하기 위해, 많은 CNN 기반 모델들은 '극좌표 변환(Polar Transform)'이라는 영리한 전처리 기법에 의존했다.9 이는 항공 이미지를 극좌표계로 변환하여, 항공 이미지 중심의 동심원을 지상 파노라마 이미지의 수평선처럼 펼쳐주는 기하학적 변환이다.30 이를 통해 두 이미지의 기하학적 레이아웃을 인위적으로 유사하게 만들어 CNN이 매칭하기 쉽게 만들어주었다.23 SAFA와 같은 모델들이 이 기법을 사용하여 큰 성능 향상을 이루었다.24 하지만 이 방법은 지상-항공 이미지 간의 기하학적 대응 관계에 대한 강한 가정을 전제로 하며, 지상 이미지가 정확히 항공 이미지의 중앙에 위치하지 않거나, 조명 조건이 크게 다를 경우 변환이 왜곡되어 성능이 저하되는 명백한 한계를 가졌다.23

#### 2.4.2  트랜스포머의 강점

자연어 처리 분야에서 시작된 트랜스포머는 비전 분야로 넘어오면서 CVGL의 난제들을 해결할 새로운 가능성을 제시했다.

- **전역적 문맥 모델링(Global Context Modeling):** 트랜스포머의 핵심인 셀프 어텐션(Self-Attention) 메커니즘은 이미지를 여러 개의 패치(patch)로 나눈 뒤, 모든 패치 쌍 간의 상호 관련성을 직접 계산한다. 이는 CNN의 지역적인 수용 필드(receptive field)와 달리, 이미지의 첫 번째 레이어부터 전역적인 문맥과 장거리 의존성을 포괄적으로 모델링할 수 있게 해준다.38 이를 통해 시각적으로 모호한 장면에서도 더 정확한 판단을 내릴 수 있다.
- **위치 인코딩(Positional Encoding):** 트랜스포머는 각 패치의 순서나 공간적 위치 정보를 알기 위해 위치 인코딩을 명시적으로 사용한다. 이 덕분에 모델은 극좌표 변환과 같은 외부의 기하학적 전처리 없이도, 두 뷰의 패치들 간의 복잡한 기하학적 대응 관계를 데이터로부터 직접 학습할 수 있다.23 이는 훨씬 더 유연하고 일반화 성능이 높은 접근 방식이다.

#### 2.4.3  주요 트랜스포머 모델 분석: L2LTR, Cross-view Transformer, TransGeo

- **L2LTR/EgoTR:** 트랜스포머를 CVGL에 성공적으로 적용한 초기 모델 중 하나다. 이 모델은 일반적인 셀프 어텐션을 넘어, 인접한 트랜스포머 레이어의 특징 맵 간에 어텐션을 계산하는 'Self-cross Attention'이라는 독창적인 메커니즘을 제안했다. 이를 통해 정보의 흐름을 원활하게 하고 학습 안정성과 표현력을 높였다.39
- **Cross-view Transformer:** 자율주행 시나리오에 특화된 모델로, 여러 대의 카메라에서 들어오는 이미지를 입력받아 하나의 조감도(bird's-eye-view) 또는 지도 뷰(map-view) 의미론적 분할 맵으로 변환한다. 각 카메라의 고유한 내부(intrinsic) 및 외부(extrinsic) 파라미터를 기반으로 한 위치 임베딩을 사용하여, 여러 시점의 정보를 하나의 정규화된 공간으로 통합하는 기하학적 변환을 암시적으로 학습한다.41
- **TransGeo:** 극좌표 변환과 같은 전처리에 전혀 의존하지 않는 '순수 트랜스포머(pure transformer)' 접근법을 제시하며 CVGL 분야에 큰 반향을 일으켰다. TransGeo는 트랜스포머의 고유한 장점을 극대화하여, 기존 CNN 기반 SOTA 모델들보다 훨씬 적은 계산 비용으로 더 높은 성능을 달성했다.37

#### 2.4.4  TransGeo 심층 탐구: 'Attend and Zoom-in' 전략의 혁신성

TransGeo의 성공은 단순히 트랜스포머를 적용한 것을 넘어, 그 구조적 유연성을 극대화한 혁신적인 학습 전략에 기인한다. 'Attend and Zoom-in'으로 명명된 이 전략은 계산 효율성이라는 새로운 경쟁의 장을 열었다.38 초기 CVGL 연구가 주로 정확도(recall@k) 향상에만 집중했다면, TransGeo는 제한된 계산 예산 내에서 어떻게 성능을 극대화할 것인가라는 실용적인 질문에 답한다.

이 전략은 두 단계로 구성된다 38:

1. **1단계 (Attend - 집중):** 먼저, 전체 이미지를 저해상도로 처리하여 트랜스포머를 통과시킨다. 그리고 모델 내부의 어텐션 맵을 분석하여 어떤 이미지 패치들이 매칭에 중요한 정보(informative patches)를 담고 있고, 어떤 패치들이 불필요한 정보(예: 하늘, 가려진 영역)를 담고 있는지 식별한다.
2. **2단계 (Zoom-in - 확대):** 다음 학습 단계에서는 정보량이 적다고 판단된 패치들을 과감히 버린다(non-uniform cropping). 그리고 여기서 절약된 계산 자원(GPU 메모리, 연산 시간)을 정보량이 많다고 판단된 핵심 패치들의 해상도를 높이는 데 집중적으로 재할당한다.

이 "집중하고 확대하는" 방식은 마치 인간이 중요한 단서를 찾기 위해 특정 부분을 자세히 들여다보는 시각적 주의(visual attention) 과정과 매우 유사하다. 이는 추가적인 계산 비용 없이, 혹은 오히려 비용을 줄이면서도 모델의 성능을 향상시키는 매우 효율적인 혁신이다.37 이처럼 효율성을 새로운 혁신의 척도로 제시한 것은, CVGL 연구가 학술적인 정확도 경쟁을 넘어 실제 차량의 ECU나 모바일 기기와 같은 자원 제약적인 환경에 탑재되기 위한 실용적인 단계로 나아가고 있음을 보여주는 중요한 신호다.

**표 2: CVGL을 위한 딥러닝 접근법 비교 분석**

| 접근법 (Approach)                                    | 핵심 원리 (Core Principle)                                   | 시점 차이 해결 방식 (Handling of Viewpoint Gap)              | 주요 강점 (Key Strengths)                                    | 한계 (Limitations)                                           | 대표 모델 (Representative Models)                |
| ---------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| **샴 네트워크 (Siamese CNNs)**                       | 거리 학습(Metric Learning)을 통해 특징 공간에서 유사/비유사 이미지 쌍의 거리를 조절 | 임베딩 공간 학습을 통한 암시적(implicit) 해결                | 개념이 간단하고, Few-shot learning에 효과적. 삼중항 손실 등으로 안정적인 학습 가능. | 시점 변환이 극심할 때 특징 표현력에 한계. 전역적 문맥 파악 능력 부족. | CVM-Net 37, Vo & Hays 32                         |
| **GAN 기반 방법 (GAN-based Methods)**                | 생성적 적대 신경망을 이용해 한쪽 뷰 이미지를 다른 쪽 뷰 스타일로 변환 | 이미지 도메인 변환을 통한 명시적(explicit) 해결              | 시각적으로 인상적인 시점 변환 가능. 도메인 격차를 직접적으로 줄임. | 학습 불안정, 모델 붕괴(model collapse) 위험, 높은 계산 비용. 23 | CDtE 9, Regmi et al. 34                          |
| **트랜스포머 기반 방법 (Transformer-based Methods)** | 셀프 어텐션과 위치 인코딩으로 전역적 문맥과 기하학적 관계를 데이터로부터 직접 학습 | 전역적 문맥 및 위치 정보 학습을 통한 강력한 암시적(implicit) 해결 | 전역적 정보 모델링에 탁월. 극좌표 변환 등 외부 전처리 불필요. 높은 성능과 효율성. | 대규모 데이터셋 학습이 필요할 수 있음.                       | L2LTR 39, TransGeo 38, Cross-view Transformer 41 |

## 3.  Cross-view Matching의 확장: 다양한 응용 분야 탐색

크로스뷰 지리적 위치 추정(CVGL)이 Cross-view Matching 연구를 주도하는 핵심 동력이지만, 이 기술의 적용 범위는 훨씬 더 넓다. 서로 다른 시점에서 포착된 시각 정보를 연결하는 능력은 보안, 감시, 엔터테인먼트 등 다양한 분야에서 새로운 가능성을 열어준다. 본 장에서는 CVGL 외에 Cross-view Matching이 핵심적인 역할을 하는 두 가지 주요 응용 분야, 즉 '사람 재식별(Person Re-Identification)'과 '증강현실(Augmented Reality)'에 대해 심도 있게 탐색한다.

### 3.1  사람 재식별(Person Re-Identification): 다중 카메라 네트워크에서의 개인 추적

사람 재식별(Person Re-ID)은 CCTV와 같이 서로 겹치지 않는 여러 카메라 뷰에 등장하는 동일 인물을 식별해내는 기술이다.44 이는 본질적으로 서로 다른 카메라라는 '교차 시점(cross-view)'에서 촬영된 사람 이미지를 매칭하는 문제로, 공공 안전, 실종자 수색, 지능형 교통 시스템 등에서 매우 중요하다. Re-ID 시스템은 조명 변화, 다양한 신체 자세, 시점 변화, 다른 물체에 의한 가림 등 수많은 현실적인 어려움을 극복하고 동일인의 정체성(identity)을 강건하게 유지해야 한다.46

#### 3.1.1  교차 도메인 및 교차 모달리티(Cross-Domain & Cross-Modality)

Re-ID의 난이도는 데이터가 수집되는 환경과 센서의 종류에 따라 더욱 복잡해진다.

- **교차 도메인(Cross-Domain) Re-ID:** 특정 쇼핑몰(source domain)에서 수집된 레이블링된 데이터로 학습한 모델을, 전혀 다른 공항(target domain)의 레이블 없는 데이터에 적용하는 시나리오를 다룬다.45 두 도메인은 카메라 종류, 조명 환경, 배경 등이 모두 다르기 때문에, 소스 도메인에서 학습된 모델은 타겟 도메인에서 심각한 성능 저하를 겪는다. 이를 해결하기 위해 타겟 도메인의 스타일을 모방하거나, 도메인에 무관한 일반적인 특징을 학습하는 연구가 활발히 진행되고 있다.
- **교차 모달리티(Cross-Modality) Re-ID:** 이는 Re-ID 문제의 복잡성을 한 차원 더 높인다. 예를 들어, 주간에는 일반적인 RGB 카메라로, 야간에는 적외선(IR) 카메라로 촬영된 이미지를 매칭하는 경우가 대표적이다.35 이 문제는 단순히 시점이나 환경이 다른 것을 넘어, 데이터를 표현하는 방식(modality) 자체가 다르다. RGB 이미지는 색상 정보를 담고 있지만, IR 이미지는 온도 정보를 담고 있다. 성공적인 교차 모달리티 매칭을 위해서는 두 모달리티에 공통적으로 존재하는 '공유 정보'(modality-shared information, 예: 사람의 실루엣, 골격 구조)와 각 모달리티에만 고유하게 존재하는 '특화 정보'(modality-specific information, 예: RGB의 옷 색깔, IR의 체열 패턴)를 모두 효과적으로 활용해야 한다.35 이는 단순히 공통된 특징 공간을 찾는 것을 넘어, 모델이 '정체성'과 관련된 정보와 '모달리티'와 관련된 정보를 분리하여 이해(disentangle)해야 함을 의미한다. 이러한 접근은 Re-ID 문제를 더 넓은 연구 분야인 '분리 표현 학습(disentangled representation learning)'과 연결시키며, 해당 분야의 발전이 Re-ID 성능 향상에 직접적으로 기여할 수 있음을 시사한다.

#### 3.1.2  관련 기술 동향

이러한 도전 과제들을 해결하기 위해 Re-ID 분야에서는 다양한 딥러닝 기술이 연구되고 있다.

- **특징 학습(Feature Learning):** 사람 이미지 전체에서 추출하는 전역적 특징(global features)과, 신체 부위(머리, 상체, 하체 등)별로 나누어 추출하는 지역적 특징(local features)을 결합하여 더 분별력 있는 표현을 학습한다.45
- **GAN 활용:** 교차 도메인 문제에서 소스 도메인의 이미지를 타겟 도메인의 스타일로 변환하는 데 GAN이 사용된다. 이를 통해 레이블이 있는 소스 데이터를 타겟 도메인 환경에 맞게 '가상으로' 생성하여 모델을 학습시킨다.45
- **해싱(Hashing):** 대규모 CCTV 네트워크와 같이 수백만 장의 이미지를 다뤄야 하는 실용적인 시나리오에서는 매칭 속도가 매우 중요하다. 해싱은 고차원의 특징 벡터를 64비트나 128비트 같은 짧은 이진 코드(binary code)로 압축하는 기술이다. 이진 코드 간의 해밍 거리(Hamming distance) 계산은 유클리드 거리 계산보다 훨씬 빠르므로, 대규모 데이터베이스에서도 실시간에 가까운 검색이 가능하다.46
- **교차 해상도(Cross-Resolution):** 카메라의 성능이나 피사체와의 거리에 따라 사람 이미지의 해상도가 크게 달라지는 현실적인 문제를 해결하기 위한 연구도 활발하다. 저해상도 이미지의 해상도를 복원하거나, 해상도 변화에 강인한 특징을 학습하는 것을 목표로 한다.49

### 3.2  증강현실(Augmented Reality): 정밀한 지리적 자세 추정(Geo-Pose Estimation)

증강현실(AR)은 현실 세계 위에 가상의 디지털 정보를 겹쳐 보여주는 기술이다. 몰입감 있는 AR 경험의 핵심은 가상 객체가 마치 실제 세계의 일부인 것처럼 정확한 위치와 방향으로 안정적으로 고정되는 것이다. 특히 건물이나 도시 전체를 무대로 하는 대규모 실외 AR에서는 사용자의 디바이스(스마트폰, AR 글래스)가 지구상의 어디에 있으며 어느 방향을 바라보고 있는지를 밀리미터 단위로 정밀하게 아는 것이 필수적이다.28

이때 Cross-view Matching, 특히 CVGL 기술은 가상 세계와 현실 세계를 연결하는 '닻(anchor)'과 같은 역할을 수행한다.

- **AR에서의 역할:** 기존의 모바일 AR 시스템은 주로 SLAM(Simultaneous Localization and Mapping)과 같은 기술을 사용하여 주변 환경을 기준으로 상대적인 위치를 추적한다. 이는 단기적으로는 매우 정밀하지만, 시간이 지나거나 넓은 공간을 이동하면 오차가 누적되는 드리프트(drift) 현상이 발생한다. 또한, GPS는 전역적인 위치를 알려주지만 오차 범위가 수 미터에 달해 AR에 필요한 정밀도를 제공하지 못한다.28 CVGL은 이러한 한계를 극복하는 결정적인 해결책을 제공한다. 사용자의 카메라로 들어오는 실시간 영상(지상 뷰)을 정적인 위성 지도(항공 뷰)와 매칭함으로써, 드리프트가 없는 절대적인 전역 지리-자세(geo-pose)를 주기적으로 획득하여 누적된 오차를 보정하고 시스템을 재정렬한다.29
- **가상 객체 배치의 정확도 향상:** CVGL을 통해 정밀한 지오-포즈를 얻게 되면, AR 시스템은 가상 객체를 현실 공간에 매우 정확하게 배치할 수 있다. 예를 들어, 특정 건물의 정문에 가상의 안내원을 세우거나, 역사적 장소 위에 과거의 모습을 재현한 3D 모델을 겹쳐 보여주는 등의 정교한 상호작용이 가능해진다. 최신 연구들은 단순히 위치(latitude, longitude)뿐만 아니라 방향(heading)까지 동시에 추정하는 모델을 개발하여 AR 애플리케이션에 요구되는 높은 수준의 정밀도를 달성하고 있다.28 AlignNet과 같은 모델은 위성 이미지와 OpenStreetMap의 지도 데이터를 융합하고, 반복적인 최적화 과정을 통해 정렬 정확도를 더욱 높이는 접근법을 제시한다.51

결론적으로, AR 분야에서 CVGL은 단순히 또 하나의 위치 추정 기술이 아니다. 이는 대규모의 영속적인(persistent) AR 경험, 즉 현실 세계와 디지털 세계가 완벽하게 융합된 '메타버스'를 구현하기 위한 가장 근본적인 기반 기술 중 하나이다. CVGL이 제공하는 이 강력한 '전역 앵커' 없이는, 도시 전체를 캔버스 삼아 펼쳐지는 미래의 AR 애플리케이션은 실현되기 어렵다.

## 4.  최신 연구 동향 및 미래 전망

Cross-view Matching 분야는 인공지능 기술의 급속한 발전과 함께 역동적으로 진화하고 있다. 특히 CVPR, ECCV 등 최고 권위의 컴퓨터 비전 학회에서 발표되는 연구들은 이 분야의 미래를 가늠할 중요한 지표가 된다. 2024년을 기점으로 최신 연구들은 단일 이미지 매칭이라는 기존의 틀을 넘어 비디오, 이미지 세트, 심지어 자연어와 같은 새로운 차원으로 문제의 범위를 확장하고 있다. 본 장에서는 이러한 최신 연구 동향을 분석하고, 이를 바탕으로 Cross-view Matching 기술의 미래 과제와 연구 방향을 조망한다.

### 4.1  단일 이미지를 넘어서: 비디오 및 이미지 세트 기반 CVGL

기존의 CVGL 연구는 대부분 단일 쿼리 이미지를 데이터베이스와 매칭하는 정적인 문제에 집중해왔다. 그러나 자율주행차나 로봇이 실제로 경험하는 세계는 정지된 스냅샷이 아닌, 연속적인 이미지 스트림, 즉 비디오이다.21 이 연속적인 데이터에는 개별 이미지에는 없는 풍부한 시간적, 공간적 맥락이 담겨 있으며, 이를 활용하는 것은 CVGL의 성능과 강건성을 한 단계 끌어올릴 잠재력을 지닌다. 이러한 문제의식은 연구 패러다임을 '인식(perception)'에서 '인지(cognition)'로 전환시키고 있다. 즉, 단순히 시각 패턴을 매칭하는 것을 넘어, 시간적 일관성과 공간적 맥락을 '추론(reasoning)'하는 방향으로 나아가고 있는 것이다.

#### 4.1.1  ECCV 2024 하이라이트: GAReT 모델 분석

2024년 유럽 컴퓨터 비전 학회(ECCV)에서 발표된 GAReT(Geolocalization with Adapters and Auto-Regressive Transformers) 모델은 이러한 흐름을 대표하는 최신 연구다.52 GAReT은 교차 시점 '비디오' 지리적 위치 추정을 위한 완전 트랜스포머 기반 모델로, 기존 방법들이 가진 여러 문제점을 해결하고자 한다. 기존 비디오 기반 방법들은 종종 카메라의 내부 파라미터나 주행 기록계(odometry) 데이터와 같은 추가 정보에 의존했으며, 각 비디오 프레임을 독립적으로 예측하여 시간적으로 튀거나 일관되지 않은 GPS 궤적을 생성하는 한계가 있었다.52

GAReT은 다음과 같은 두 가지 혁신적인 모듈로 이 문제를 해결한다 53:

- **GeoAdapter:** 이 모듈은 사전 학습된 강력한 단일 이미지 위치 추정 모델을 '재활용'하는 영리한 방법을 제시한다. 비디오의 모든 프레임에 대해 무거운 모델을 처음부터 학습시키는 대신, 기존 이미지 모델의 표현력을 효율적으로 집계하고 비디오 입력에 맞게 미세 조정하는 경량의 '어댑터(adapter)'를 사용한다. 이는 계산 효율성을 극대화하는 실용적인 접근법이다.
- **TransRetriever:** 이 모듈은 시간적 일관성을 확보하는 역할을 한다. 각 프레임에 대해 가능한 상위 k개의 위치 후보를 먼저 예측한 뒤, 인코더-디코더 구조의 트랜스포머를 사용하여 이전 프레임의 최종 예측값을 참고하여 현재 프레임의 최적 위치를 자기회귀적(auto-regressively)으로 결정한다. 이는 마치 문장을 생성하듯, 과거의 맥락을 바탕으로 가장 자연스러운 다음 위치를 추론하는 것과 같다.

#### 4.1.2  Set-CVGL: 다양한 관점 통합의 잠재력

비디오가 '순서가 있는' 연속적인 데이터라면, '순서가 없는' 이미지 집합을 활용하려는 시도도 등장했다. Set-CVGL(Cross-View Image Set Geo-Localization)은 인간이 낯선 곳에서 자신의 위치를 파악할 때, 한 방향만 보는 것이 아니라 주변을 둘러보거나 몇 걸음 움직여 여러 각도에서 정보를 수집하는 행동에서 영감을 얻은 새로운 태스크다.26

- **개념:** 고정된 카메라로 촬영된 비디오 시퀀스와 달리, Set-CVGL은 다양한 위치와 각도에서 독립적으로 촬영된 이미지들의 '집합(set)'을 쿼리로 사용한다. 이를 통해 훨씬 더 풍부하고 다양한 관점의 시각적 단서를 통합하여, 단일 이미지나 고정된 시퀀스로는 해결하기 어려운 모호성을 해소하고 위치 추정의 신뢰도를 크게 향상시킬 수 있다.26
- **FlexGeo 모델:** 이 새로운 태스크를 위해 제안된 FlexGeo 모델은 이미지 세트의 순서에 의존하지 않고 특징을 효과적으로 융합하기 위해 '유사도 기반 특징 융합기(Similarity-guided Feature Fuser, SFF)'라는 모듈을 사용한다.26

### 4.2  새로운 입력 양식: 텍스트 기반 지리적 위치 추정 (Text-guided Geo-Localization)

연구의 지평은 시각 정보를 넘어 언어의 영역으로까지 확장되고 있다. 최근 제안된 '텍스트 기반 지리적 위치 추정'은 이미지 쿼리 대신, "파란 지붕 집 옆 사거리"와 같은 자연어 묘사(natural language description)를 입력받아 해당하는 위치의 위성 이미지나 지도 데이터를 검색하는 혁신적인 패러다임이다.56

이 기술은 보행자 내비게이션이나 긴급 구조 요청과 같이 시각적 정보가 불완전하거나 없는 상황에서 엄청난 잠재력을 가진다. 예를 들어, 조난자가 전화로 주변 지형지물을 묘사하면 구조대가 그 위치를 위성 지도로 즉시 파악할 수 있다. 이 과제를 해결하기 위해서는 대규모 언어 모델(LLM)을 활용하여 고품질의 장면 묘사 텍스트를 생성하는 기술과, 이 텍스트 임베딩과 이미지 임베딩을 효과적으로 매칭하는 CrossText2Loc과 같은 멀티모달 모델 개발이 핵심적인 연구 주제로 부상하고 있다.56

### 4.3  3D 인간 이해와 자기 지도 학습으로의 확장

Cross-view Matching의 개념은 2D 이미지 검색을 넘어, 3D 구조를 이해하고 생성하는 더 복잡한 비전 태스크의 핵심적인 학습 신호(supervisory signal)로도 활용되고 있다. 2024년 CVPR에서 발표된 한 연구는 3D 인간 이해를 위해 교차 시점(스테레오 카메라 쌍)과 교차 자세(비디오 속 시간적으로 인접한 프레임 쌍) 정보를 활용하는 자기 지도 학습(self-supervised learning) 방법을 제안했다.57

이 방법의 핵심 원리는 한쪽 뷰의 이미지 일부를 가리고(masking), 다른 쪽 뷰의 이미지를 힌트로 삼아 가려진 부분을 복원하도록 모델을 학습시키는 것이다.57 이 과정에서 모델은 명시적인 3D 레이블 없이도, 서로 다른 뷰 사이의 일관성(consistency)을 유지하기 위해 자연스럽게 3D 구조와 인간의 움직임에 대한 강력한 사전 지식(prior)을 학습하게 된다. 이는 대규모의 레이블링된 데이터셋 구축이 어려운 많은 분야에서 '자기 지도 학습'이 얼마나 강력한 대안이 될 수 있는지를 보여준다. CVGL 분야에서도, 정확한 GPS 태그가 없는 방대한 양의 스트리트뷰 비디오와 위성 이미지를 활용하여, 시간적/공간적 일관성을 자기 지도 신호로 삼아 모델을 사전 학습시키는 방식이 미래의 확장성과 일반화 성능을 확보하는 핵심 전략이 될 것이다.

### 4.4  종합: 해결 과제와 향후 연구 방향 제언

지금까지의 분석을 종합하면, Cross-view Matching 연구는 단일 이미지에서 비디오/이미지 세트로, 이미지 쿼리에서 텍스트 쿼리로, 그리고 2D 위치 추정에서 3D 구조 이해로 그 범위를 빠르게 확장하고 있다.26 이러한 흐름 속에서 미래 연구는 다음과 같은 방향으로 심화될 것으로 전망된다.

- **효율성과 강건성:** 자율주행차나 모바일 기기에 실제 탑재되기 위해서는 실시간 처리가 가능한 경량화 모델 개발이 필수적이다.41 또한, 야간, 악천후, 계절 변화 등 열악한 환경에서도 안정적인 성능을 유지하는 강건성(robustness) 향상 연구는 계속해서 중요한 과제로 남을 것이다.25
- **데이터 융합(Data Fusion):** 위성 이미지, 항공 이미지, 도로 지도(OpenStreetMap), 3D 도시 모델, 텍스트 정보 등 사용 가능한 모든 종류의 이종(heterogeneous) 데이터를 효과적으로 융합하는 멀티모달(multi-modal) 접근법이 더욱 중요해질 것이다.51
- **해석 가능성(Explainability):** 현재의 딥러닝 모델은 '블랙박스'처럼 작동하여 왜 특정 위치를 매칭했는지 설명하기 어렵다. 미래의 시스템은 "이 건물과 저 교차로가 일치하기 때문에 여기로 판단했다"와 같이 시각적, 텍스트적으로 판단의 근거를 제시하는 해석 가능한 AI(XAI) 기술을 접목하여 사용자의 신뢰를 확보해야 할 것이다.56
- **대규모 동적 환경 적응:** 도시는 끊임없이 변화한다. 새로운 건물이 생기고 도로가 바뀐다. 한 번 학습된 모델이 이러한 변화에 뒤처지지 않기 위해서는, 새로운 데이터를 지속적으로 학습하며 지식을 업데이트하는 평생 학습(lifelong learning) 또는 연속 학습(continual learning) 패러다임의 도입이 필수적이다.

결론적으로, Cross-view Matching 기술은 정적인 이미지 검색 문제를 넘어, 동적인 세계를 이해하고 추론하며 상호작용하는 지능형 시스템의 핵심 시각 능력으로 진화하고 있다. 앞으로 이 기술은 자율 시스템의 눈이 되어 미지의 환경을 항해하고, 증강현실을 통해 우리의 현실 인식을 확장하며, 수많은 데이터를 연결하여 새로운 가치를 창출하는 데 결정적인 역할을 수행할 것이다.

#### **참고 자료**

1. Cross-View Matching for Vehicle Localization by Learning ..., accessed July 1, 2025, https://www.researchgate.net/publication/355657390_Cross-View_Matching_for_Vehicle_Localization_by_Learning_Geographically_Local_Representations
2. A Faster and More Effective Cross-View Matching Method of UAV and Satellite Images for UAV Geolocalization - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/13/19/3979
3. A Part-aware Attention Neural Network for Cross-view Geo-localization between UAV and Satellite, accessed July 1, 2025, https://alife-robotics.org/article/vol9issue3/9311.pdf
4. Full article: Multi-level representation learning via ConvNeXt-based network for unaligned cross-view matching, accessed July 1, 2025, https://www.tandfonline.com/doi/full/10.1080/10095020.2024.2439385
5. 크로스 테이블 행별 총합계 표시, accessed July 1, 2025, https://docs.tibco.com/pub/sfire-cloud/14.4.0/doc/html/ko-KR/TIB_sfire_client/client/topics/ko-KR/displaying_grand_totals_for_rows_in_cross_tables.html
6. [SQL] 조인 개념 및 크로스 조인/ 내부 조인/ 외부 및 셀프 조인 - 착해지는 중 입니다., accessed July 1, 2025, https://come-alive.tistory.com/29
7. Correspondence problem - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Correspondence_problem
8. Semantic Cross-View Matching - Stanford Computational Vision and Geometry Lab, accessed July 1, 2025, https://cvgl.stanford.edu/papers/castaldo_iccv15.pdf
9. A Cross-View Image Matching Method with Feature Enhancement - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/15/8/2083
10. Learning Intra-view and Cross-view Geometric Knowledge for Stereo Matching - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Gong_Learning_Intra-view_and_Cross-view_Geometric_Knowledge_for_Stereo_Matching_CVPR_2024_paper.pdf
11. Is there a difference between Image Alignment and Stereo Rectification for stereo correspondence or matching? - Signal Processing Stack Exchange, accessed July 1, 2025, https://dsp.stackexchange.com/questions/25995/is-there-a-difference-between-image-alignment-and-stereo-rectification-for-stere
12. Image Alignment and Stitching: A Tutorial1 - Washington, accessed July 1, 2025, https://courses.cs.washington.edu/courses/cse576/05sp/papers/MSR-TR-2004-92.pdf
13. Image Alignment and Stitching: A Tutorial - cs.wisc.edu, accessed July 1, 2025, https://pages.cs.wisc.edu/~dyer/cs534/papers/szeliski-alignment-tutorial.pdf
14. What's the difference between parallel view and cross view? : r/ParallelView - Reddit, accessed July 1, 2025, https://www.reddit.com/r/ParallelView/comments/1glxdam/whats_the_difference_between_parallel_view_and/
15. velog.io, accessed July 1, 2025, [https://velog.io/@checking_pks/SURF-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98#:~:text=SURF%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%EC%9D%B4%EB%9E%80%3F,-Speeded%2DUp%20Robust&text=%ED%8A%B9%EC%A7%95%EC%A0%90%EC%9D%84%20%ED%86%B5%ED%95%98%EC%97%AC%20%EB%8F%99%EC%9D%BC%ED%95%9C%20%EC%9E%A5%EB%A9%B4,%EB%B9%A0%EB%A5%B8%20%EC%84%B1%EB%8A%A5%EC%9D%98%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%EC%9E%85%EB%8B%88%EB%8B%A4.](https://velog.io/@checking_pks/SURF-알고리즘#:~:text=SURF 알고리즘이란%3F,-Speeded-Up Robust&text=특징점을 통하여 동일한 장면,빠른 성능의 알고리즘입니다.)
16. OpenCV 4로 배우는 컴퓨터 비전과 머신 러닝: 14.2.1 크기 불변 특징점 알고리즘 - 3, accessed July 1, 2025, https://thebook.io/006939/0595/
17. Keypoint Detector : SIFT & SURF Algorithm 비교 - velog, accessed July 1, 2025, [https://velog.io/@yyk9612/Keypoint-Detector-SIFT-SURF-Algorithm-%EB%B9%84%EA%B5%90](https://velog.io/@yyk9612/Keypoint-Detector-SIFT-SURF-Algorithm-비교)
18. [CV] Scale Invariant Feature Transform (SIFT) : 영상의 스케일에 불변한 Feature, accessed July 1, 2025, https://mvje.tistory.com/79
19. [OpenCV] SURF 알고리즘, accessed July 1, 2025, [https://velog.io/@checking_pks/SURF-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98](https://velog.io/@checking_pks/SURF-알고리즘)
20. [Computer Vision / Image Precessing] SURF (Speeded-Up Robust Features) - 주머니 속 메모장 - 티스토리, accessed July 1, 2025, https://alex-an0207.tistory.com/165
21. Leveraging cross-view geo-localization with ensemble learning and temporal awareness | PLOS One, accessed July 1, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0283672
22. (PDF) Cross-View Geo-Localization: A Survey - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/386207589_Cross-view_Geo-localization_A_Survey
23. GeoViewMatch: A Multi-Scale Feature-Matching Network for Cross-View Geo-Localization Using Swin-Transformer and Contrastive Learning - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/16/4/678
24. Cross-View Matching for Vehicle Localization by Learning Geographically Local Representations, accessed July 1, 2025, https://intelligent-vehicles.org/wp-content/uploads/2022/03/XiaRAL2021_Cross-view-geographic-local.pdf
25. A Cross-View Geo-Localization Algorithm Using UAV Image and Satellite Image - MDPI, accessed July 1, 2025, https://www.mdpi.com/1424-8220/24/12/3719
26. Cross-View Image Set Geo-Localization - arXiv, accessed July 1, 2025, https://arxiv.org/html/2412.18852v1
27. Cross-view geo-localization: a survey - arXiv, accessed July 1, 2025, https://arxiv.org/html/2406.09722v1
28. Cross-View Visual Geo-Localization for Outdoor Augmented Reality - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/369592227_Cross-View_Visual_Geo-Localization_for_Outdoor_Augmented_Reality
29. [2303.15676] Cross-View Visual Geo-Localization for Outdoor Augmented Reality - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2303.15676
30. Feature Relation Guided Cross-View Image Based Geo-Localization - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/15/20/5029
31. ArcGeo: Localizing Limited Field-of-View Images using Cross-view Matching | Request PDF, accessed July 1, 2025, https://www.researchgate.net/publication/379703471_ArcGeo_Localizing_Limited_Field-of-View_Images_using_Cross-view_Matching
32. Where Am I Looking At? Joint Location and Orientation Estimation by Cross-View Matching - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content_CVPR_2020/papers/Shi_Where_Am_I_Looking_At_Joint_Location_and_Orientation_Estimation_CVPR_2020_paper.pdf
33. TransGeo: Transformer Is All You Need for Cross-view Image Geo-localization | Request PDF - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/363906289_TransGeo_Transformer_Is_All_You_Need_for_Cross-view_Image_Geo-localization
34. Cross-View Image Sequence Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/WACV2023/papers/Zhang_Cross-View_Image_Sequence_Geo-Localization_WACV_2023_paper.pdf
35. Cross-Modality Person Re-Identification With Shared-Specific Feature Transfer - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content_CVPR_2020/papers/Lu_Cross-Modality_Person_Re-Identification_With_Shared-Specific_Feature_Transfer_CVPR_2020_paper.pdf
36. Siamese Neural Networks (샴 네트워크) 개념 이해하기 - tyami's study blog, accessed July 1, 2025, [https://tyami.github.io/deep%20learning/Siamese-neural-networks/](https://tyami.github.io/deep learning/Siamese-neural-networks/)
37. TransGeo: Transformer Is All You Need for Cross-view Image Geo-localization | Request PDF - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/359709594_TransGeo_Transformer_Is_All_You_Need_for_Cross-view_Image_Geo-localization
38. TransGeo: Transformer Is All You Need for Cross-View Image Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2022/papers/Zhu_TransGeo_Transformer_Is_All_You_Need_for_Cross-View_Image_Geo-Localization_CVPR_2022_paper.pdf
39. Cross-view Geo-localization with Layer-to-Layer Transformer, accessed July 1, 2025, https://proceedings.neurips.cc/paper/2021/file/f31b20466ae89669f9741e047487eb37-Paper.pdf
40. [2107.00842] Cross-view Geo-localization with Evolving Transformer - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2107.00842
41. Cross-View Transformers for Real-Time Map-View Semantic Segmentation - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2022/papers/Zhou_Cross-View_Transformers_for_Real-Time_Map-View_Semantic_Segmentation_CVPR_2022_paper.pdf
42. Cross-view Transformers for real-time Map-view Semantic Segmentation - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2205.02833
43. [2204.00097] TransGeo: Transformer Is All You Need for Cross-view Image Geo-localization, accessed July 1, 2025, https://arxiv.org/abs/2204.00097
44. Person Re-Identification | Papers With Code, accessed July 1, 2025, https://paperswithcode.com/task/person-re-identification
45. Cross-Domain Person Re-Identification Based on Feature Fusion Invariance - MDPI, accessed July 1, 2025, https://www.mdpi.com/2076-3417/14/11/4644
46. Learning Cross-View Binary Identities for Fast Person Re-Identification - IJCAI, accessed July 1, 2025, https://www.ijcai.org/Proceedings/16/Papers/342.pdf
47. Cross-Modality Person Re-Identification Based on Heterogeneous Center Loss and Non-Local Features - PubMed Central, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8304692/
48. [1801.01760] Crossing Generative Adversarial Networks for Cross-View Person Re-identification - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1801.01760
49. [2105.11722] Deep High-Resolution Representation Learning for Cross-Resolution Person Re-identification - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2105.11722
50. Cross-View Visual Geo-Localization for Outdoor Augmented Reality - Niluthpol Chowdhury Mithun, accessed July 1, 2025, https://niluthpol.github.io/Niluthpol_Mithun_Files/Ground_to_Aerial_GeoLocalization_VR.pdf
51. Cross-View Outdoor Localization in Augmented Reality by Fusing Map and Satellite Data, accessed July 1, 2025, https://www.mdpi.com/2076-3417/13/20/11215
52. arxiv.org, accessed July 1, 2025, https://arxiv.org/abs/2408.02840
53. GAReT: Cross-view Video Geolocalization with Adapters and Auto-Regressive Transformers - ECVA | European Computer Vision Association, accessed July 1, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/html/7875_ECCV_2024_paper.php
54. ECCV Poster GAReT: Cross-view Video Geolocalization with Adapters and Auto-Regressive Transformers, accessed July 1, 2025, https://eccv.ecva.net/virtual/2024/poster/2461
55. The official implementation of "GAReT: Cross-view Video Geolocalization with Adapters and Auto-Regressive Transformers" [European Conference on Computer Vision (ECCV) 2024] - GitHub, accessed July 1, 2025, https://github.com/manupillai308/GAReT
56. Where am I? Cross-View Geo-localization with Natural Language Descriptions - arXiv, accessed July 1, 2025, https://arxiv.org/html/2412.17007v1

CVPR 2024 Open Access Repository, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024/html/Armando_Cross-view_and_Cross-pose_Completion_for_3D_Human_Understanding_CVPR_2024_paper.html
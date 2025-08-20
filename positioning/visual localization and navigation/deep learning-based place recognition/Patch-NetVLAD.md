# Patch-NetVLAD





## 1.  시각적 장소 인식(VPR)의 본질과 난제





### 1.1  VPR의 정의: 단순한 이미지 검색을 넘어선 위치 추정 문제



시각적 장소 인식(Visual Place Recognition, VPR)은 주어진 쿼리 이미지(query image)의 시각 정보만을 활용하여, 지리 정보가 사전에 구축된 대규모 참조 이미지 데이터베이스(reference database) 내에서 해당 이미지가 촬영된 장소를 정확하게 식별하는 근본적인 컴퓨터 비전 과제이다.1 본질적으로 VPR은 '이전에 방문했던 장소를 다시 인식하는' 특정 맥락을 가진 이미지 검색(image retrieval) 문제로 정의될 수 있다.2 이는 단순히 시각적으로 유사한 이미지를 찾는 것을 넘어, 쿼리 이미지와 데이터베이스 이미지가 동일한 물리적 공간을 담고 있는지를 판단하는 고차원적인 추론을 요구한다.

VPR 연구에서 '장소(place)'의 정의는 매우 중요하다. 응용 분야에 따라 그 정의가 달라질 수 있지만, 로보틱스 및 자율주행 분야에서는 실용적인 관점에서 '두 이미지 간에 유의미한 시각적 중첩(visual overlap)이 존재할 때 동일한 장소로 간주한다'는 정의가 널리 사용된다.2 여기서 시각적 중첩이란 건물, 도로, 특정 랜드마크 등 장면의 정적인 요소들이 두 이미지에 공통으로 나타나는 것을 의미한다. 이 정의는 장소 인식을 성공적으로 수행한 후에, 매칭된 이미지 쌍으로부터 카메라의 상대적인 6-DoF(자유도) 포즈(위치 및 방향)를 추정하는 시각적 위치 추정(visual localization)이나 지도 작성(mapping)과 같은 후속 작업으로 자연스럽게 연결될 수 있는 기반을 제공한다.

이러한 응용 지향적인 맥락은 VPR을 일반적인 이미지 검색과 근본적으로 구분 짓는다. 일반 이미지 검색이 '고양이'나 '해변'과 같은 카테고리 수준의 시맨틱 유사성이나 전반적인 색감, 질감의 유사성에 초점을 맞춘다면, VPR은 극심한 외관 변화에도 불구하고 동일한 '인스턴스(instance)' 즉, 세상에 단 하나뿐인 특정 장소를 식별해야 하는 훨씬 더 까다로운 제약 조건을 가진다. 이러한 제약 조건 때문에 VPR 시스템은 단순한 시각적 유사성을 넘어, 장면의 기하학적, 구조적 일관성을 이해하고 판단할 수 있는 능력을 갖추어야 한다. 이는 왜 Patch-NetVLAD와 같은 후속 연구들이 단순히 전역적인 특징 벡터를 비교하는 것을 넘어, 패치 기반의 공간적 일관성(spatial consistency)을 검증하는 방향으로 발전했는지를 설명하는 핵심적인 이유가 된다.9 즉, VPR은 응용의 필요성이 기술의 발전 방향을 이끄는 대표적인 분야이다.



### 1.2  로보틱스와 자율주행에서의 핵심 역할: SLAM 루프 클로저(Loop Closure)를 중심으로



VPR은 현대 로보틱스와 자율주행 시스템에서 필수불가결한 핵심 기술로 자리매김하고 있다. 이는 크게 두 가지 측면에서 중요한 역할을 수행한다. 첫째, 사전 구축된 맵이 있는 환경에서 로봇이나 차량이 자신의 현재 위치를 파악하는 독립적인 위치 결정(stand-alone localization) 능력의 기반이 된다. 둘째, 더욱 중요하게는 SLAM(Simultaneous Localization and Mapping, 동시적 위치 추정 및 지도 작성) 시스템의 성능과 안정성을 좌우하는 핵심 구성 요소로 기능한다.2

SLAM 시스템에서 VPR의 가장 결정적인 응용은 '루프 클로저(Loop Closure Detection)'이다.2 SLAM을 수행하는 로봇은 자신의 움직임을 추정(Odometry)하며 이동 경로를 누적해 나가는데, 이 과정에서 작은 오차들이 필연적으로 쌓이게 된다. 이러한 누적 오차(accumulated drift)는 로봇이 오랜 시간 또는 넓은 영역을 탐사할수록 점점 커져, 결국 생성된 지도가 실제 환경과 크게 어긋나는 결과를 초래한다. 루프 클로저는 로봇이 이전에 방문했던 장소를 다시 방문했을 때, VPR을 통해 "현재 위치가 과거의 특정 지점과 동일하다"는 사실을 인식하는 과정이다. 이 인식이 성공하면, 과거의 위치와 현재의 위치 사이에 강력한 기하학적 제약 조건이 생성된다. SLAM 시스템은 이 제약 조건을 활용하여 그동안 쌓였던 누적 오차를 전체 경로에 걸쳐 재분배하고 수정함으로써, 전역적으로 일관되고(globally consistent) 정확한 지도를 완성할 수 있다. 따라서 VPR의 정확도와 강인함은 장기 자율주행(long-term autonomy)의 성공 여부를 판가름하는 핵심 기술이라 할 수 있다.



### 1.3  VPR의 근본적인 3대 과제



VPR 기술이 실제 환경에서 안정적으로 작동하기 위해서는 극복해야 할 여러 근본적인 난제들이 존재한다. 이들은 크게 세 가지로 분류할 수 있다.

첫째, **외관 및 시점 변화(Appearance and Viewpoint Change)**이다. 이는 VPR이 직면한 가장 크고 본질적인 어려움이다. 동일한 물리적 장소라 할지라도, 촬영 조건에 따라 그 모습은 극적으로 변할 수 있다. 조명의 변화(예: 낮과 밤, 맑은 날과 흐린 날), 계절의 변화(예: 여름의 녹음과 겨울의 눈 덮인 풍경), 날씨의 변화 등은 이미지의 색상, 밝기, 대비를 완전히 바꾸어 놓는다.2 또한, 쿼리 이미지와 데이터베이스 이미지가 서로 다른 위치나 각도에서 촬영될 경우 발생하는 시점 변화는 이미지 내 객체들의 상대적인 크기, 형태, 배치를 왜곡시켜 매칭을 매우 어렵게 만든다.2

둘째, **지각적 모호성(Perceptual Aliasing)**이다. 이는 서로 다른 장소가 시각적으로 매우 유사하게 보이는 현상을 의미한다.2 예를 들어, 현대적인 건물의 반복적인 유리창 패턴, 아파트 단지의 유사한 동들, 실내 환경의 끝없이 이어지는 복도, 고속도로의 비슷한 풍경 등은 VPR 시스템에 큰 혼란을 야기한다. 심지어 어떤 경우에는, 조건 변화가 극심한 동일 장소의 두 이미지보다, 지각적으로 모호한 서로 다른 장소의 두 이미지가 더 유사하게 보일 수도 있다. 이는 VPR 시스템이 단순한 픽셀 수준의 유사성을 넘어 장면의 고유한 구조적, 의미적 특징을 파악해야 하는 이유를 명확히 보여준다.

셋째, **동적 객체 및 폐색(Dynamic Objects and Occlusions)**이다. 실제 환경은 정적이지 않다. 도로 위의 자동차, 보행자, 공사 현장 등 동적인 객체들은 장소의 본질적인 모습과는 무관하지만 이미지의 상당 부분을 차지하며 특징 추출 과정을 방해한다.2 또한, 주차된 트럭이나 무성한 나뭇잎 등이 건물의 중요한 외벽이나 간판과 같은 핵심 랜드마크를 가리는 폐색(occlusion) 현상 역시 VPR 시스템이 의존해야 할 유용한 정보를 제거하여 성능 저하의 주요 원인이 된다. 성공적인 VPR 시스템은 이러한 일시적이고 비본질적인 요소들을 효과적으로 무시하고, 장면에 항상 존재하는 안정적인 정적 구조물에 집중할 수 있어야 한다.



## 2.  패러다임의 서막: NetVLAD 아키텍처와 그 한계





### 2.1  NetVLAD의 작동 원리: 학습 가능한 전역 디스크립터(Global Descriptor)의 탄생



2016년 CVPR에서 발표된 NetVLAD는 시각적 장소 인식 분야에 딥러닝 기반의 종단간(end-to-end) 학습 패러다임을 본격적으로 도입한 기념비적인 연구이다.18 이전까지의 VPR 연구는 SIFT와 같은 수작업 특징(hand-crafted feature)을 추출하고 이를 BoW(Bag-of-Words)나 VLAD(Vector of Locally Aggregated Descriptors)와 같은 기법으로 집계하는 파이프라인에 의존했다. NetVLAD는 이러한 전통적인 VLAD 표현에서 영감을 받아, 이를 미분 가능한(differentiable) 형태로 일반화한 새로운 'NetVLAD 레이어'를 제안함으로써 전체 VPR 파이프라인을 CNN(Convolutional Neural Network) 아키텍처 내에서 한 번에 학습할 수 있는 길을 열었다.18

NetVLAD의 핵심 목표는 CNN의 중간 계층(예: VGG-16의 conv5)에서 추출된 수많은 고차원 지역 특징(local feature)들을 순서에 상관없이(orderless) 하나의 압축된 고정 길이 벡터, 즉 전역 디스크립터(global descriptor)로 효율적으로 집계(aggregate)하는 것이다.18 CNN의 컨볼루션 레이어 출력은 

H×W×D 형태의 특징 맵으로, 이는 공간적으로 H×W개의 위치에 존재하는 D차원 지역 디스크립터들의 집합으로 볼 수 있다. NetVLAD 레이어는 이 수천 개의 지역 디스크립터들을 입력으로 받아, 장소의 핵심적인 시각 정보를 응축한 단일 벡터를 출력한다. 이렇게 생성된 전역 디스크립터는 이미지 전체를 대표하며, 데이터베이스에 저장된 다른 이미지들의 전역 디스크립터와 유클리드 거리(Euclidean distance)나 코사인 유사도(cosine similarity)를 통해 매우 빠르고 효율적으로 비교될 수 있다.18 이로써 대규모 데이터베이스에서도 실시간에 가까운 검색이 가능해졌다.



### 2.2  핵심 메커니즘: 미분 가능한 '소프트 할당(Soft Assignment)'의 역할



NetVLAD가 종단간 학습을 가능하게 한 기술적 핵심은 기존 VLAD의 '하드 할당(hard assignment)' 방식을 미분 가능한 '소프트 할당(soft assignment)'으로 대체한 데에 있다.18 전통적인 VLAD는 먼저 K-means와 같은 클러스터링 알고리즘을 통해 데이터셋의 모든 지역 특징들을 대표하는 

K개의 클러스터 중심(cluster center), 즉 '시각 단어(visual word)' 사전을 구축한다. 그 후, 특정 이미지의 각 지역 디스크립터를 이 사전에서 가장 가까운 단 하나의 클러스터 중심에 배타적으로 할당한다. 이 '가장 가까운'을 찾는 과정은 미분이 불가능하여, 역전파(backpropagation) 기반의 딥러닝 프레임워크에 통합될 수 없었다.

NetVLAD는 이 문제를 해결하기 위해, 각 지역 디스크립터 xi가 단 하나의 클러스터에 속하는 대신, 모든 K개의 클러스터 ck에 대해 소속될 확률, 즉 부드러운 가중치(soft assignment weight) $\bar{a}_k(x_i)$를 갖도록 설계했다. 이 가중치는 다음과 같은 수식으로 계산된다 18:

aˉk(xi)=∑k′ewk′Txi+bk′ewkTxi+bk

여기서 wk와 bk는 클러스터 k에 대한 학습 가능한(learnable) 파라미터이다. 이 수식은 본질적으로 소프트맥스(softmax) 함수와 동일한 형태로, 모든 가중치의 합이 1이 되도록 정규화하며 전체 과정이 미분 가능하다. 이렇게 계산된 소프트 할당 가중치는 각 지역 디스크립터와 클러스터 중심 간의 잔차(residual)를 가중 합산하는 데 사용되어 최종 NetVLAD 디스크립터를 생성한다. 이 과정에서 클러스터 중심 ck뿐만 아니라, 할당 가중치를 결정하는 파라미터 wk, bk까지 모두 역전파를 통해 장소 인식 작업에 최적화되도록 학습된다.19

이러한 소프트 할당 메커니즘은 단순히 미분 가능성을 확보하기 위한 기술적 해결책을 넘어, VPR 문제에 대한 더 유연한 표현 방식을 제시했다. 하드 할당이 각 특징을 하나의 시각 단어에 강제로 귀속시키는 반면, 소프트 할당은 하나의 시각적 특징이 여러 시각 단어의 의미에 걸쳐 있을 수 있다는 모호성을 자연스럽게 모델링한다. 예를 들어, '붉은 벽돌 건물의 창문 모서리'라는 특징은 '붉은색', '벽돌 질감', '건물', '창문', '모서리' 등 여러 시맨틱 클러스터에 부분적으로 기여할 수 있다. 이러한 유연성은 극심한 외관 변화 속에서도 특징의 본질적인 속성을 보존하는 데 도움을 주어, 결과적으로 전역 디스크립터의 강인성을 높이는 데 기여한 것으로 해석할 수 있다.



### 2.3  전역적 집계 방식의 근본적 한계: 공간 정보의 손실과 시점 변화에 대한 취약성



NetVLAD는 강력하고 효율적인 전역 디스크립터를 생성하는 데 성공했지만, 그 집계 방식에는 근본적인 한계가 내재되어 있었다. 가장 큰 문제는 지역 특징들을 하나의 전역 벡터로 '풀링(pooling)'하는 과정에서 각 특징이 가지고 있던 원래의 공간적 위치 정보(spatial information)가 대부분 소실된다는 점이다.11 NetVLAD는 순서 없는(orderless) 집계 방식을 채택했기 때문에, 이미지의 왼쪽 위에 있던 특징과 오른쪽 아래에 있던 특징이 최종 디스크립터에 미치는 영향은 그들의 내용에만 의존할 뿐, 위치와는 무관하게 된다.

이러한 공간 정보의 부재는 특히 **시점 변화(viewpoint change)**에 매우 취약한 결과를 낳는다.5 예를 들어, 정면에서 촬영한 건물 이미지와 측면에서 비스듬히 촬영한 건물 이미지는 동일한 장소를 담고 있지만, 건물과 주변 객체들의 상대적인 위치, 크기, 원근감이 완전히 달라진다. NetVLAD와 같은 전역적 접근법은 이러한 구조적 변화를 제대로 처리하지 못하고, 두 이미지가 완전히 다른 외관을 가졌다고 판단하여 매칭에 실패할 가능성이 높다. 즉, 이미지 전체의 '외관'에만 의존하는 방식은 장면의 3차원 구조에 대한 이해가 부족하여 시점 변화에 강인하지 못하다.

또한, 이미지의 일부 영역에만 변화가 생기는 경우에도 문제가 발생한다. 예를 들어, 장소의 핵심적인 랜드마크는 그대로인데 도로에 다른 차가 주차되어 있거나, 광고판이 바뀌는 등 국소적인 변화가 발생했을 때, NetVLAD는 이러한 변화를 이미지 전체의 변화로 인식하여 디스크립터 값이 크게 달라질 수 있다.5

결론적으로, NetVLAD의 성공(강력한 전역 디스크립터 제공)과 그 명확한 한계(공간 정보 부재로 인한 시점 변화 취약성)는 자연스럽게 VPR 연구 커뮤니티에 다음과 같은 질문을 던졌다: "NetVLAD의 뛰어난 특징 표현력과 학습 능력을 유지하면서, 어떻게 소실된 공간 정보를 다시 도입하여 시점 변화에 대한 강인성을 확보할 수 있을까?" 이 질문에 대한 가장 직접적이고 성공적인 답변 중 하나가 바로 다음에 소개될 Patch-NetVLAD이다. Patch-NetVLAD는 NetVLAD의 한계를 정면으로 돌파하기 위해, NetVLAD의 집계 방식을 이미지 전체가 아닌 '패치' 단위로 적용하고, 생성된 패치 디스크립터들을 마치 지역 특징점처럼 사용하여 공간적 일관성을 검증하는 하이브리드 접근법을 제시함으로써 VPR 패러다임의 새로운 진화를 이끌었다.5



## 3.  Patch-NetVLAD: 지역적 특징과 전역적 특징의 혁신적 융합





### 3.1  핵심 철학: '지역-전역(Locally-Global)' 디스크립터 개념의 도입



Patch-NetVLAD는 기존 VPR 연구의 주류를 이루던 두 가지 접근법, 즉 전역 디스크립터(global descriptor)와 지역 디스크립터(local descriptor)의 이분법적 구도를 극복하기 위한 혁신적인 철학을 제시한다.9 전역 디스크립터는 이미지 전체를 하나의 벡터로 표현하여 외관 변화(조명, 계절 등)에 강인하지만 공간 정보를 잃어버려 시점 변화에 취약하다. 반면, 지역 디스크립터는 SIFT나 SURF와 같이 이미지 내 특정 키포인트(keypoint)를 기술하여 공간적 정밀도가 높고 기하학적 검증에 유리하지만, 극심한 외관 변화나 텍스처가 부족한 영역에서는 안정적인 추출이 어렵다는 단점이 있다.

Patch-NetVLAD는 이 두 접근법의 상호 보완적인 강점을 결합하고 약점을 최소화하기 위해 '지역-전역(Locally-Global)' 디스크립터라는 새로운 개념을 도입했다.5 핵심 아이디어는 CNN을 통해 얻은 이미지의 고차원 특징 맵(feature map)을 고정된 크기의 여러 '패치(patch)'로 조밀하게 나누고, 각 개별 패치에 대해 NetVLAD와 같은 전역적 집계 방식(global aggregation)을 적용하는 것이다. 이렇게 생성된 각 패치의 디스크립터는 그 패치라는 '지역(local)' 내의 특징들을 '전역적(global)'으로 요약한 표현이므로 '지역-전역적'이라고 불린다.

이러한 접근법은 전통적인 "지역 특징 추출 후 전역으로 집계(local-to-global)"하는 방식을 뒤집어, "전역적 표현(NetVLAD 잔차)에서 지역적 특징(패치 디스크립터)을 파생"시키는 역발상적인 설계를 채택했다.5 이로써 전역 디스크립터가 갖는 VPR에 최적화된 풍부한 표현력과 외관 변화에 대한 강인함을 패치 수준에서 유지하면서도, 각 패치가 특징 맵 상의 특정 위치 정보를 갖게 되어 지역 디스크립터처럼 공간적 매칭과 기하학적 검증을 수행할 수 있게 되었다. 이는 두 접근법의 장점을 한 프레임워크 안에서 유기적으로 통합한 중요한 진전이다.5



### 3.2  다중 스케일(Multi-Scale) 특징 추출 및 융합: 변화에 강인한 표현력 확보



Patch-NetVLAD는 단일 크기의 패치에만 의존하지 않고, 서로 다른 크기를 갖는 여러 스케일(multi-scale)의 패치들을 동시에 활용하여 장소 인식의 강인성을 한층 더 끌어올렸다.9 쿼리 이미지와 데이터베이스 이미지 간에는 카메라와의 거리 차이 등으로 인해 스케일 변화가 빈번하게 발생한다. 다중 스케일 접근법은 이러한 변화에 효과적으로 대응하기 위한 전략이다.

구체적으로, 작은 크기의 패치는 이미지의 세밀한 디테일이나 텍스처 정보를 포착하는 데 유리하고, 큰 크기의 패치는 건물이나 도로와 같은 더 넓은 영역의 구조적인 레이아웃 정보를 담는 데 유리하다. Patch-NetVLAD는 이렇게 서로 다른 스케일의 정보를 상호 보완적으로 활용하기 위해, 여러 크기의 패치들로부터 각각 '지역-전역 디스크립터'를 추출한다. 그 후, 각 스케일에서 계산된 공간 매칭 점수들을 가중 합산하여 최종 유사도 점수를 도출하는 융합(fusion) 과정을 거친다.5

논문의 Ablation Study 결과는 이러한 다중 스케일 융합의 효과를 명확히 입증한다. 단일 스케일만 사용했을 때보다 3개의 서로 다른 패치 크기(예: 2x2, 4x4, 8x8 픽셀 크기의 특징 맵 패치)를 융합했을 때 모든 데이터셋에서 Recall@N 성능이 일관되게 향상되었다.25 흥미로운 점은 융합하는 패치 크기의 종류를 5개까지 늘려도 성능이 더 이상 유의미하게 향상되지 않았다는 사실이다.25 이는 무조건 많은 스케일을 사용하는 것보다, 상호 보완적인 정보를 제공하는 적절한 수의 스케일 조합을 찾는 것이 중요함을 시사한다. 또한, 각 스케일에 부여하는 가중치를 변경하더라도 성능 저하가 크지 않아, 제안된 융합 방식이 하이퍼파라미터 선택에 비교적 둔감하고 강인함(robust)을 보여주었다.25



### 3.3  계산 효율성 극대화: IntegralVLAD와 깊이별 확장 컨볼루션(Depthwise Dilated Convolution)



다중 스케일 접근법은 성능 향상에 매우 효과적이지만, 각 스케일의 패치에 대해 개별적으로 특징 집계를 수행한다면 계산 비용이 스케일의 수에 비례하여 크게 증가하는 문제가 발생한다.9 Patch-NetVLAD는 이 문제를 해결하기 위해 '적분 이미지(Integral Image)' 기법에서 영감을 받은 'IntegralVLAD' 또는 '적분 특징 공간(integral feature space)'이라는 매우 효율적인 계산 방법을 제안했다.9

적분 이미지는 이미지의 특정 영역의 픽셀 합을 네 번의 참조만으로 빠르게 계산하는 기법이다. 이와 유사하게, IntegralVLAD는 특정 크기의 패치에 대한 VLAD 디스크립터가 그 패치를 구성하는 가장 작은 단위(1x1) 패치들의 VLAD 디스크립터들의 합으로 근사될 수 있다는 아이디어에 기반한다. 이를 통해, 전체 특징 맵에 대해 한 번만 누적 합 연산을 수행하여 '적분 특징 맵'을 미리 계산해두면, 어떤 크기의 패치 디스크립터라도 매우 적은 비용으로 즉시 유도해낼 수 있다.

실제 구현에서는 이 적분(누적 합) 과정을 **깊이별 확장 컨볼루션(Depthwise Dilated Convolution)**이라는 최신 CNN 연산을 통해 매우 영리하고 효율적으로 수행한다.9

- **확장 컨볼루션(Dilated or Atrous Convolution):** 이 연산은 컨볼루션 커널(kernel)의 가중치들 사이에 일정한 간격(dilation rate)으로 0을 삽입하여 커널을 '팽창'시키는 효과를 낸다. 이를 통해 파라미터 수나 계산량을 늘리지 않고도 커널이 바라보는 영역, 즉 수용 영역(receptive field)을 기하급수적으로 확장할 수 있다.26 Patch-NetVLAD에서는 서로 다른 dilation rate를 가진 확장 컨볼루션을 적용하여 다양한 크기의 패치 영역에 대한 집계를 효과적으로 모사한다.
- **깊이별 컨볼루션(Depthwise Convolution):** 일반적인 컨볼루션이 입력 특징 맵의 모든 채널을 한 번에 계산하는 것과 달리, 깊이별 컨볼루션은 각 채널에 대해 독립적으로 컨볼루션을 수행한다.29 이는 채널 간의 상호작용은 고려하지 않지만, 계산량을 채널 수만큼 대폭 줄여주는 효과가 있다.

Patch-NetVLAD는 이 두 기술을 결합한 **깊이별 확장 컨볼루션**을 사용하여, 다중 스케일 패치 특징 집계를 단 한 번의 효율적인 연산 패스로 처리한다. 이 덕분에 다중 스케일 융합이 주는 성능 향상의 이점을 거의 추가적인 계산 비용 없이 누릴 수 있게 되었으며, 이는 Patch-NetVLAD의 학술적 성과뿐만 아니라 실용적 가치를 높이는 핵심적인 기술적 기여라고 할 수 있다.



### 3.4  2단계 검색 및 재순위화(Re-ranking) 파이프라인 상세 분석



Patch-NetVLAD는 대규모 데이터베이스 환경에서의 효율성과 높은 정확도를 동시에 달성하기 위해, VPR 분야의 고전적인 'coarse-to-fine' 전략을 딥러닝 프레임워크에 성공적으로 구현한 2단계 검색 파이프라인을 채택했다.5 이 파이프라인의 성공은 이후 많은 2단계 VPR 모델들의 표준적인 접근법으로 자리 잡는 데 큰 영향을 미쳤다.

1단계: 전역적 후보군 검색 (Global Retrieval)

첫 번째 단계에서는 속도가 매우 빠른 전역 디스크립터를 사용하여 검색 범위를 대폭 좁힌다. 쿼리 이미지가 주어지면, 먼저 원본 NetVLAD 모델을 사용하여 이미지 전체를 대표하는 단일 전역 디스크립터를 추출한다. 이 쿼리 디스크립터를 데이터베이스에 저장된 수백만 개의 이미지들의 전역 디스크립터와 비교하여, 가장 유사도가 높은 상위 N개(논문 실험에서는 N=100)의 후보 이미지를 빠르게 검색한다.9 이 단계는 계산 비용이 저렴한 벡터 간 거리 계산만으로 이루어지므로, 전체 데이터베이스를 대상으로 하더라도 매우 효율적으로 수행될 수 있다. 이 과정의 목표는 정답일 가능성이 높은 소수의 후보군을 신속하게 선별하는 것이다.

2단계: 패치 단위 매칭과 공간적 점수 계산을 통한 정밀 재순위화 (Patch-level Re-ranking)

두 번째 단계에서는 1단계에서 추려진 소수의 후보 이미지들에 대해서만 계산 비용이 더 높은 정밀 매칭을 수행한다.9 이 단계의 과정은 다음과 같다.

1. **지역-전역 디스크립터 추출:** 쿼리 이미지와 1단계에서 검색된 각 후보 이미지에 대해, 앞서 설명한 다중 스케일 '지역-전역' 패치 디스크립터들을 추출한다.
2. **상호 최근접 이웃 매칭 (Mutual Nearest Neighbors Matching):** 두 이미지(쿼리-후보)의 패치 디스크립터 세트 간에 상호 매칭을 수행한다. 쿼리의 패치 A가 후보 이미지의 패치 B를 가장 가까운 이웃으로 선택하고, 동시에 패치 B 역시 패치 A를 가장 가까운 이웃으로 선택할 때, 이 둘을 신뢰할 수 있는 대응점 쌍으로 간주한다. 이를 통해 잠재적인 대응점 목록을 생성한다.9
3. **공간 점수 계산 (Spatial Scoring):** 생성된 대응점 쌍들의 공간적 일관성(spatial consistency)을 평가하여 최종 유사도 점수인 '공간 점수'를 계산한다. Patch-NetVLAD는 사용자의 요구(성능 vs. 속도)에 따라 두 가지 방식의 공간 점수 계산법을 제공한다.
   - **성능 중심 (RANSAC 기반):** RANSAC(Random Sample Consensus) 알고리즘을 사용하여 대응점들 간의 기하학적 관계를 가장 잘 설명하는 호모그래피(homography) 변환 행렬을 추정한다. 최종 점수는 이 변환 모델과 일치하는 대응점의 수, 즉 인라이어(inliers)의 개수로 결정된다. 이 방식은 기하학적으로 매우 정확한 매칭을 보장하여 최고의 성능을 내지만, 반복적인 모델 추정 과정으로 인해 계산 비용이 높다.9
   - **속도 중심 (Rapid Spatial Scoring):** RANSAC의 대안으로, 매칭된 패치 쌍들의 특징 맵 상 위치 변위(displacement)가 얼마나 일관된 방향성을 갖는지를 측정하는 더 빠른 방식을 사용한다. 이는 시점 변화 시 장면 요소들의 전반적인 움직임 일관성을 근사하는 방법으로, RANSAC보다 훨씬 빠르지만 정확도는 약간 저하될 수 있다.9
4. **재순위화 및 최종 결과 도출:** 각 후보 이미지에 대해 계산된 공간 점수를 기준으로, 1단계에서 얻은 후보 목록의 순위를 재조정(re-rank)한다. 가장 높은 공간 점수를 받은 후보 이미지가 최종적으로 가장 유사한 장소로 결정된다.9

이러한 2단계 파이프라인은 전체 시스템의 처리 속도를 현실적인 수준으로 유지하면서도, 최종 정확도는 정밀한 2단계 매칭 덕분에 높게 유지할 수 있는 최적의 균형점을 제공한다. 이는 Patch-NetVLAD가 학술적 성능뿐만 아니라 실제 로보틱스 시스템에 적용될 수 있는 실용성을 갖추었음을 의미하며, ECCV 2020 VPR 챌린지 우승과 같은 성과로 이어진 핵심 요인 중 하나이다.9



## 4.  성능 분석 및 실험적 검증





### 4.1  주요 벤치마크 데이터셋에서의 SOTA 성능 비교



Patch-NetVLAD의 성능은 다양한 환경과 조건에서 촬영된 여러 도전적인 실제 데이터셋을 통해 종합적으로 검증되었다. 논문은 제안된 방법이 계절, 조명, 구조물 등 외관의 변화와 이동, 회전 등 시점의 변화 모두에 대해 매우 강인한 성능을 보이며, 다수의 벤치마크에서 기존 최첨단(State-Of-The-Art, SOTA) 모델들을 능가하는 결과를 달성했음을 보여준다.9

주요 평가에 사용된 데이터셋은 도심 환경의 시점 및 시간 변화를 다루는 **Pittsburgh 30k**, 전 세계 여러 도시에서 장기간에 걸쳐 수집되어 극심한 외관 변화를 포함하는 **Mapillary Street-Level Sequences (MSLS)**, 계절 변화가 뚜렷한 **Nordland**, 낮/밤 및 날씨 변화가 포함된 **RobotCar Seasons v2**, 그리고 **Extended CMU Seasons**, **Tokyo 24/7** 등이다.32

평가는 VPR의 두 가지 주요 태스크에 대해 이루어졌다. 첫째, **시각적 장소 인식(VPR)** 태스크에서는 **Recall@N** 지표가 사용되었다. 이는 상위 N개의 검색 결과 안에 정답 이미지가 포함될 확률을 의미하며, N이 작을수록 더 정확한 검색 성능을 나타낸다. 둘째, **시각적 위치 추정(Visual Localization)** 태스크에서는 쿼리 이미지의 포즈를 특정 오차 범위(예: 0.25m 및 2도) 내에서 정확하게 추정한 비율(**Accuracy @ threshold**)을 측정했다.32

성능 비교 결과는 Patch-NetVLAD의 우수성을 명확하게 보여준다.

- 모든 데이터셋에서 기존의 전역 디스크립터 기반 방법들(NetVLAD, SFRS 등)을 적게는 6%에서 많게는 330%까지의 상대적 성능 향상을 보이며 압도했다.9
- 당시 강력한 로컬 특징 매칭 기법으로 알려진 SuperGlue 기반 파이프라인과 비교했을 때도, 최대 54%의 상대적 성능 향상을 기록하며 우위를 점했다.9
- 특히 정밀한 포즈 추정 능력이 요구되는 RobotCar Seasons v2와 Extended CMU Seasons 데이터셋의 시각적 위치 추정 벤치마크에서는 모든 오차 허용 범위에서 1위를 차지하며, 제안된 방법의 높은 정밀도를 입증했다.32
- 이러한 객관적인 성능은 2020년 ECCV 학회에서 열린 **Facebook Mapillary Visual Place Recognition Challenge에서의 우승**으로 이어져, Patch-NetVLAD의 실질적인 성능과 가치를 공인받았다.9

아래 표는 주요 벤치마크 데이터셋에서 Patch-NetVLAD와 다른 SOTA 모델들의 성능을 요약하여 비교한 것이다.

**Table 2: 주요 벤치마크 데이터셋 Recall@N 성능 비교**

| Model                | Dataset            | Recall@1 (%) | Recall@5 (%) | Recall@10 (%) |
| -------------------- | ------------------ | ------------ | ------------ | ------------- |
| NetVLAD 32           | Pittsburgh-30k     | 83.9         | 91.2         | 93.1          |
| NetVLAD 32           | Mapillary-val      | 57.3         | 70.8         | 76.1          |
| SuperGlue 9          | Mapillary-val      | -            | -            | 79.5 (R@100)  |
| **Patch-NetVLAD** 32 | **Pittsburgh-30k** | **88.7**     | **94.5**     | **95.8**      |
| **Patch-NetVLAD** 32 | **Mapillary-val**  | **79.5**     | **86.2**     | **87.7**      |
| **Patch-NetVLAD** 32 | **Nordland**       | **58.4**     | **74.6**     | **80.5**      |
| **Patch-NetVLAD** 32 | **Tokyo247**       | **86.0**     | **88.6**     | **90.5**      |

*참고: 위 표의 수치는 논문 및 Papers With Code에서 보고된 값을 기반으로 재구성되었으며, 모델 설정(e.g., 성능 중심)에 따라 약간의 차이가 있을 수 있다.*



### 4.2  구성 요소별 기여도 분석: Ablation Study를 통한 핵심 성능 요인 탐색



Patch-NetVLAD의 높은 성능이 어떤 구성 요소에서 비롯되었는지 파악하기 위해, 논문과 보충 자료에서는 광범위한 제거 연구(Ablation Study)를 수행했다.9 이 연구들은 제안된 각 모듈의 기여도를 정량적으로 분석하고 시스템의 강인함을 검증하는 데 목적이 있다.

가장 중요한 분석 중 하나는 **다중 스케일 융합의 효과**에 대한 것이었다. Mapillary 데이터셋을 대상으로 한 실험에서, 단일 스케일의 패치(예: 크기 5)만을 사용했을 때의 Recall@1은 77.0%였던 반면, 3개의 다른 스케일(크기 2, 5, 8)을 융합했을 때는 79.5%로 성능이 유의미하게 향상되었다.25 이는 다중 스케일 접근법이 스케일 변화에 대한 강인성을 높여 실제 성능 향상으로 이어진다는 것을 명확히 보여준다.

또한, 융합하는 **패치 크기의 조합이나 각 스케일에 부여하는 가중치**를 변경하더라도 성능이 크게 저하되지 않는다는 점이 확인되었다.25 예를 들어, (2, 5, 8) 조합 대신 (2, 4, 6)이나 (4, 6, 8)과 같은 다른 조합을 사용해도 Recall@1 성능은 79% 수준을 유지했다. 이는 제안된 다중 스케일 융합 방식이 특정 하이퍼파라미터 값에 민감하지 않고, 다양한 설정에서도 안정적인 성능을 내는 강인한(robust) 방법론임을 시사한다.

흥미롭게도, 융합하는 **패치 스케일의 수를 무조건 늘리는 것이 항상 좋은 결과를 가져오지는 않았다**. 3개의 스케일을 융합했을 때 최적의 성능을 보였으며, 그 수를 5개까지 늘렸을 때는 오히려 성능이 약간 감소하는 경향을 보였다.25 이는 추가되는 스케일이 기존 스케일들과 중복되는 정보를 제공하거나, 오히려 노이즈로 작용할 수 있음을 의미한다. 따라서 상호 보완적인 정보를 제공하는 최적의 스케일 조합을 찾는 것이 중요하며, Patch-NetVLAD는 3개의 스케일 융합을 통해 그 균형점을 효과적으로 찾았다.

결론적으로, 이러한 Ablation Study들은 Patch-NetVLAD의 SOTA 성능이 어느 한 가지 기법이 아닌, 지역-전역 디스크립터, 다중 스케일 융합, 효율적인 2단계 파이프라인 등 각 구성 요소들이 유기적으로 결합하여 만들어낸 시너지 효과의 결과물임을 입증한다. 이는 VPR이 단일 알고리즘의 성능 경쟁을 넘어, 잘 설계된 시스템 엔지니어링의 문제임을 보여주는 좋은 사례이다.



### 4.3  정성적 분석: 성공 및 실패 사례와 데이터셋 주석 오류에 대한 고찰



정량적인 수치 분석을 넘어, Patch-NetVLAD의 실제 작동 방식을 이해하기 위해 논문의 보충 자료에서는 정성적 결과에 대한 논의를 포함하고 있다.25 이러한 분석은 모델의 강점과 약점을 직관적으로 보여주고, 때로는 평가 지표가 담지 못하는 미묘한 측면을 드러낸다.

주목할 만한 부분은 일부 실패 사례(failure cases)에 대한 분석이다. 연구진은 Patch-NetVLAD가 오답을 낸 경우를 면밀히 조사한 결과, 몇몇 사례는 모델의 성능 한계가 아니라 **데이터셋의 Ground-Truth 주석 자체에 오류가 있었기 때문**임을 발견했다.25 즉, Patch-NetVLAD는 시각적으로 명백히 올바른 장소를 매칭했지만, 데이터셋에 기록된 GPS 기반의 정답지가 부정확하여 해당 매칭이 오답으로 처리된 것이다. 이는 모델의 성능이 때로는 평가에 사용되는 데이터셋의 품질에 의해 과소평가될 수 있음을 보여주는 강력한 증거이다. 더 나아가, 이는 Patch-NetVLAD가 특정 벤치마크의 통계적 패턴에 과적합된 것이 아니라, 인간의 판단에 더 가까운, 진정한 의미의 장소 인식 능력을 갖추었을 가능성을 시사하는 중요한 발견이다.

또한, 경쟁 모델과의 정성적 비교를 통해 Patch-NetVLAD의 강인함을 확인할 수 있다. 예를 들어, DELG와 같은 모델은 Nordland나 Pittsburgh 데이터셋에서는 좋은 성능을 보이지만, 매우 정밀한 포즈 정확도를 요구하는 RobotCar Seasons v2 데이터셋에서는 성능이 상대적으로 저하되는 경향을 보인다. 반면, Patch-NetVLAD는 다양한 유형의 데이터셋과 요구 조건에 걸쳐 일관되게 높은 성능을 유지하는 모습을 보였다.25 이는 Patch-NetVLAD의 접근법이 특정 데이터셋의 특성에 편향되지 않고, 보다 일반적인 VPR 문제 해결 능력을 갖추고 있음을 의미한다. 이러한 정성적 분석은 Patch-NetVLAD가 단순히 벤치마크 점수를 높이는 것을 넘어, 실제 세계의 복잡성과 불확실성에 대응할 수 있는 신뢰도 높은 솔루션임을 뒷받침한다.



## 5.  실제 구현 및 구성 가능성 탐구





### 5.1  공식 코드베이스(QVPR/Patch-NetVLAD) 구조 및 핵심 기능 분석



Patch-NetVLAD의 학술적 기여는 그 실용성을 뒷받침하는 잘 구성된 공식 코드베이스를 통해 더욱 빛을 발한다. 퀸즐랜드 공과대학교(QUT)의 로보틱스 및 자율 시스템 연구 그룹(QVPR)이 관리하는 공식 GitHub 저장소(`QVPR/Patch-NetVLAD`)는 MIT 라이선스 하에 공개되어 있어, 학계 및 산업계 연구자들이 자유롭게 접근하고 활용할 수 있다.33 이 코드베이스는 단순한 논문 구현체를 넘어, VPR 연구 및 개발을 위한 하나의 플랫폼으로서 기능하도록 설계되었다.

코드베이스는 사용자가 VPR 파이프라인의 주요 단계를 쉽게 실행하고 실험할 수 있도록 모듈화된 파이썬 스크립트들로 구성되어 있다.

- **`feature_extract.py`:** 이 스크립트는 지정된 이미지 디렉토리에서 Patch-NetVLAD의 핵심인 다중 스케일 지역-전역 디스크립터와 2단계 파이프라인의 첫 단계를 위한 NetVLAD 전역 디스크립터를 추출하는 역할을 한다. 사용자는 설정 파일을 통해 추출할 패치 크기, PCA 차원 등을 손쉽게 제어할 수 있다.34
- **`feature_match.py`:** 사전에 추출된 특징 파일들을 입력받아, 쿼리 이미지셋과 데이터베이스 이미지셋 간의 매칭을 수행한다. 이 스크립트는 1단계 전역 검색과 2단계 패치 기반 재순위화를 모두 포함하며, Ground-Truth 파일이 제공될 경우 Recall@N 성능을 자동으로 계산하여 평가 결과를 출력한다.34
- **`match_two.py`:** 두 개의 특정 이미지 파일을 직접 입력받아 이들 간의 유사도 점수를 계산하고, RANSAC을 통해 검증된 패치 대응점을 시각화하여 보여준다. 이는 모델의 작동 방식을 직관적으로 이해하고 디버깅하는 데 매우 유용하다.34
- **`train.py`:** Mapillary Street-level Sequences와 같은 대규모 데이터셋을 사용하여 새로운 NetVLAD 백본 네트워크를 처음부터 학습시키거나, 기존 체크포인트에서 학습을 재개하는 기능을 제공한다.34
- **`add_pca.py`:** 학습이 완료된 모델에 차원 축소를 위한 PCA(주성분 분석) 레이어를 추가하고 학습하는 스크립트이다. 이는 디스크립터의 크기를 줄여 저장 공간과 매칭 속도를 향상시키는 데 필수적이다.34

이처럼 잘 분리된 기능별 스크립트와 명확한 가이드는 연구의 재현성(reproducibility)을 높이고, 후속 연구자들이 Patch-NetVLAD를 강력한 베이스라인으로 삼아 새로운 아이디어를 신속하게 검증할 수 있는 연구 생태계를 조성하는 데 크게 기여했다.



### 5.2  사전 학습 모델 및 주요 스크립트 활용법



Patch-NetVLAD의 가장 큰 실용적 장점 중 하나는 사용자가 복잡하고 많은 시간이 소요되는 학습 과정을 거치지 않고도 즉시 모델의 성능을 테스트하고 활용할 수 있도록 다양한 사전 학습 모델(pre-trained models)을 제공한다는 점이다.34 Mapillary, Pittsburgh 등 주요 벤치마크 데이터셋에 대해 최적화된 모델들이 제공되며, 이 모델들은 

`feature_extract.py` 스크립트를 처음 실행할 때 `pretrained_models`라는 폴더에 자동으로 다운로드된다. 사용자는 `download_models.py` 스크립트를 사용하여 모든 모델을 한 번에 받을 수도 있다.34

제공된 README 파일에는 Pittsburgh 30k 데이터셋을 예시로 VPR 파이프라인을 실행하는 구체적인 명령어가 단계별로 제시되어 있다.34 사용자는 이 가이드를 따라 자신의 데이터셋 경로만 수정하면, 간단한 커맨드라인 명령어 몇 줄로 전체 프로세스를 재현할 수 있다.

1. 먼저 `feature_extract.py`를 사용하여 데이터베이스 이미지들과 쿼리 이미지들에 대한 특징을 각각 추출하여 파일로 저장한다.
2. 다음으로 `feature_match.py`를 실행하여, 1단계에서 생성된 특징 파일들을 이용해 매칭을 수행한다.
3. 이때 Ground-Truth 파일의 경로를 인자로 제공하면, 스크립트는 자동으로 `recalls.txt` 파일에 Recall@1, @5, @10 등의 성능 지표를 계산하여 저장해준다.

또한, 새로운 데이터셋에 대한 평가를 원하는 사용자를 위해 GPS 좌표로부터 Ground-Truth 파일을 생성하는 방법에 대한 가이드까지 제공하고 있어 34, 코드의 확장성과 활용도를 크게 높였다. 이러한 사용자 친화적인 설계는 Patch-NetVLAD가 학계를 넘어 산업 현장에서도 폭넓게 채택될 수 있는 기반이 되었다.



### 5.3  성능과 속도의 트레이드오프: `performance.ini` vs. `speed.ini` 하이퍼파라미터 심층 비교



Patch-NetVLAD의 설계 철학 중 하나는 사용자의 다양한 요구사항에 맞춰 성능, 속도, 저장 공간 간의 균형을 유연하게 조절할 수 있는 구성 가능성(configurable framework)을 제공하는 것이다.9 이러한 조절은 

`configs` 폴더 내에 위치한 `.ini` 형식의 설정 파일을 통해 이루어진다. 코드베이스는 대표적으로 최고의 정확도를 목표로 하는 `performance.ini`와, 빠른 처리 속도를 우선시하는 `speed.ini` 두 가지 설정을 기본으로 제공한다.34

비록 제공된 자료에서 두 파일의 전체 내용이 직접적으로 확인되지는 않지만, 논문의 내용, Ablation Study 결과, 그리고 코드 구조를 종합하여 두 설정 간의 핵심 하이퍼파라미터 차이를 아래와 같이 논리적으로 추론하고 비교할 수 있다.

**Table 3: `performance.ini` vs. `speed.ini` 하이퍼파라미터 비교 (추론 기반)**

| Parameter            | `performance.ini` (Value) | `speed.ini` (Value) | Impact on Performance/Speed                                  |
| -------------------- | ------------------------- | ------------------- | ------------------------------------------------------------ |
| `patch_sizes`        | `2, 5, 8` (예시)          | `5` (예시)          | **성능**: 여러 스케일이 스케일 변화에 대한 강인성을 높여 성능 향상. **속도**: 단일 스케일이 특징 추출 및 매칭 계산량을 줄여 속도 향상. |
| `nMS`                | `3` (추정)                | `1` (추정)          | **성능**: 다중 스케일 융합(nMS > 1)은 성능에 긍정적. **속도**: 융합 과정이 생략(nMS = 1)되면 속도 향상. |
| `output_dim`         | `4096`                    | `128`               | **성능**: 고차원 디스크립터는 정보 손실이 적어 정확도에 유리. **속도**: 저차원 디스크립터는 벡터 비교 및 매칭 속도가 빠르고 메모리 사용량 감소. |
| `num_clusters`       | `64`                      | `32` or `16` (추정) | **성능**: 더 많은 클러스터는 특징 공간을 세밀하게 표현하여 성능에 도움. **속도**: 클러스터 수가 적을수록 NetVLAD 집계 연산이 빨라짐. |
| `matcher`            | `RANSAC`                  | `fast`              | **성능**: RANSAC은 기하학적 검증을 통해 매우 정확한 매칭을 보장하여 성능 극대화. **속도**: `fast` (Rapid Spatial Scoring) 방식은 RANSAC의 반복적인 추정 과정을 생략하여 속도를 수십 배 향상. |
| `ransac_dist_thresh` | `5` (예시)                | `10` (예시)         | **성능**: 더 엄격한(작은) 임계값은 오매칭을 줄여 정밀도를 높임. **속도**: 더 관대한(큰) 임계값은 더 많은 인라이어를 찾아낼 수 있으나, 속도에 미치는 영향은 `matcher` 방식에 비해 미미함. |

이 표는 Patch-NetVLAD의 '구성 가능성'이라는 추상적인 장점을 구체적인 수치로 보여준다. 예를 들어, 자율주행차의 실시간 루프 클로저와 같이 속도가 매우 중요한 응용에서는 `speed.ini` 설정을 기반으로 파라미터를 튜닝할 수 있다. 반면, 오프라인 환경에서 지도를 생성하거나 벤치마크에서 최고의 성능을 기록하는 것이 목표라면 `performance.ini` 설정이 적합하다. 이처럼 사용자가 자신의 응용 목적과 하드웨어 제약(예: 자원 제한이 있는 임베디드 시스템)에 맞춰 성능과 속도 간의 트레이드오프를 직접 제어할 수 있다는 점은 Patch-NetVLAD의 중요한 실용적 강점이다.



## 6.  비판적 분석과 미래 방향





### 6.1  내재적 한계점 분석: '특징 폭발(Feature Burstiness)' 문제와 계산 비용



Patch-NetVLAD는 CNN 기반 VPR 기술의 정점을 보여주었지만, 그 기반이 되는 VLAD 아키텍처로부터 비롯된 내재적 한계점을 가지고 있다. 가장 대표적인 것이 **'특징 폭발(Feature Burstiness)'** 문제이다.38 이는 이미지 내에 하늘, 도로, 벽, 나뭇잎, 창문 등과 같이 시각적으로 반복적이고 구조적으로 덜 중요한 패턴이 넓은 영역을 차지할 경우 발생하는 현상이다. VLAD의 집계 과정에서 이러한 반복적인 특징들이 디스크립터 공간의 특정 클러스터에 과도하게 집중적으로 할당되어, 해당 클러스터의 표현을 지배하게 된다. 결과적으로, 장소를 식별하는 데 더 중요한 정보를 담고 있는 독특한 랜드마크(예: 간판, 특정 조형물)의 영향력이 상대적으로 감소하여 최종 전역 디스크립터의 식별력(discriminativeness)이 저하될 수 있다. VLAD-BuFF와 같은 후속 연구는 이러한 문제를 해결하기 위해 특징 간의 자기 유사성(self-similarity)을 기반으로 반복적인 특징의 가중치를 동적으로 낮추는 메커니즘을 제안하며, 이는 VLAD 계열 방법론의 지속적인 연구 과제임을 보여준다.38

또 다른 한계는 **계산 비용**이다. Patch-NetVLAD의 2단계 재순위화 파이프라인은 전체 데이터베이스를 대상으로 정밀 매칭을 수행하는 것보다 훨씬 효율적이지만, 2단계에서 수행되는 패치 디스크립터 추출 및 매칭 과정 자체는 여전히 상당한 계산 자원과 메모리를 요구한다.1 특히 최고의 성능을 내는 

`performance.ini` 설정과 같이, 다중 스케일의 고차원 패치 디스크립터를 사용하고, 수백 개의 후보 이미지에 대해 RANSAC 기반의 기하학적 검증을 수행할 경우, 실시간 처리가 필수적인 일부 로보틱스 응용에서는 부담이 될 수 있다. Hot-NetVLAD와 같은 연구는 모든 패치를 사용하는 대신 장소 인식에 중요한 패치만 선택적으로 사용함으로써 이러한 메모리 및 계산 요구량을 줄이려는 시도를 하기도 했다.1



### 6.2  CNN 이후의 시대: VPR 분야의 트랜스포머(Transformer) 및 파운데이션 모델(Foundation Model) 부상



Patch-NetVLAD가 발표된 2021년 이후, 컴퓨터 비전 분야 전반에 걸쳐 패러다임의 전환이 일어났으며, 이는 VPR 연구에도 지대한 영향을 미쳤다. 바로 **트랜스포머(Transformer)**와 대규모 데이터로 사전 학습된 **파운데이션 모델(Foundation Model)**의 부상이다.1

- **트랜스포머 아키텍처:** 자연어 처리 분야에서 시작된 트랜스포머는 셀프 어텐션(self-attention) 메커니즘을 통해 입력 시퀀스 내의 모든 요소 간의 장거리 의존성(long-range dependency)을 효과적으로 모델링하는 능력으로 주목받았다. 이를 비전 분야에 적용한 Vision Transformer(ViT)는 이미지를 여러 패치(patch)의 시퀀스로 간주하고, 패치 간의 전역적인 관계 및 맥락을 파악하는 데 탁월한 성능을 보였다.40 이는 컨볼루션 연산의 지역적인 수용 영역(local receptive field)에 본질적인 한계를 가지는 CNN과 대비되는 강점이다. PlaceFormer와 같은 VPR 모델들은 이러한 트랜스포머 아키텍처를 기반으로 하여, 이미지의 중요한 부분을 동적으로 식별하고 더 강인한 특징 표현을 학습한다.1
- **파운데이션 모델:** DINOv2, CLIP 등과 같이 웹 스케일의 방대한 이미지-텍스트 데이터로 사전 학습된 파운데이션 모델들은 특정 다운스트림 태스크에 대한 미세조정(fine-tuning) 없이도 매우 강력하고 일반화된 특징 표현 능력을 보여준다.4 VPR 분야에서는 AnyLoc과 같은 연구가 DINOv2 특징을 그대로 사용(zero-shot)하여, 별도의 VPR 전용 학습 없이도 다양한 환경과 조건에서 기존의 특화된 모델들을 능가하는 성능을 달성할 수 있음을 보여주었다.47 이는 VPR 문제가 대규모 일반 지식(general knowledge)을 통해 해결될 수 있음을 시사하며, 데이터셋 구축과 모델 학습에 대한 기존의 접근 방식에 근본적인 질문을 던졌다.



### 6.3  경쟁 모델과의 개념적 비교: Patch-NetVLAD vs. PlaceFormer vs. DINOv2 기반 접근법



VPR 기술의 발전 경로는 이전 세대 모델의 한계를 극복하려는 시도를 통해 명확하게 드러난다. Patch-NetVLAD, PlaceFormer, 그리고 DINOv2 기반 접근법은 이러한 진화의 각 단계를 대표한다.

**Table 1: NetVLAD, Patch-NetVLAD, PlaceFormer 핵심 메커니즘 비교**

| 구분 (Aspect)        | NetVLAD                                 | Patch-NetVLAD                                              | PlaceFormer                                               |
| -------------------- | --------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------- |
| **기반 아키텍처**    | CNN (e.g., VGG-16)                      | CNN (e.g., VGG-16)                                         | Vision Transformer (ViT)                                  |
| **핵심 특징 단위**   | 이미지 전체에 대한 단일 전역 디스크립터 | 특징 맵의 고정된 그리드 패치에 대한 '지역-전역' 디스크립터 | 트랜스포머의 동적 패치 토큰(Patch Token)                  |
| **강인성 확보 전략** | 학습 가능한 소프트 할당 기반 집계       | 다중 스케일 패치 융합 및 기하학적 재순위화                 | 셀프 어텐션을 통한 전역적 맥락 모델링 및 중요 패치 선별   |
| **공간 정보 활용**   | 집계 과정에서 대부분 소실               | 패치 위치를 통한 기하학적 검증(RANSAC)으로 복원            | 어텐션 맵을 통해 패치 간의 공간적 관계를 암시적으로 학습  |
| **주요 한계점**      | 시점 변화에 취약, 공간 정보 부재        | 모든 패치에 대한 매칭으로 인한 계산 비효율성, 특징 폭발    | 대규모 데이터 학습 의존성, CNN 대비 약한 지역적 귀납 편향 |

이러한 진화의 과정을 더 깊이 살펴보면, 각 단계가 이전 단계의 명백한 한계를 해결하기 위한 직접적인 반응이었음을 알 수 있다.

1. **NetVLAD**는 강력한 전역 디스크립터를 제공했지만, **공간 정보를 잃어버렸다**.11
2. **Patch-NetVLAD**는 패치 단위 매칭과 기하학적 검증을 통해 **공간 정보를 성공적으로 복원**했다. 하지만 모든 패치를 동등하게 다루고 exhaustive하게 매칭함으로써 **계산적으로 비효율적**이라는 새로운 문제를 낳았다.1
3. **PlaceFormer**는 트랜스포머의 어텐션 메커니즘을 활용하여 장소 인식에 '중요한' 패치만 동적으로 골라내어 매칭함으로써, Patch-NetVLAD의 **비효율 문제를 해결**하고자 했다.1 이는 VPR의 초점을 '어떻게 잘 표현할까'에서 '무엇을 표현할까'로 이동시켰다.
4. 하지만 PlaceFormer와 같은 모델들도 여전히 VPR 전용 데이터셋에 대한 감독 학습(supervised learning)을 필요로 한다. **DINOv2 기반 접근법**은 여기서 한 걸음 더 나아가, "VPR 데이터가 아닌, 세상의 거의 모든 이미지로부터 학습된 범용적 사전 지식을 이용하면 더 강력하지 않을까?"라는 근본적인 질문을 던졌다.42 이는 학습의 패러다임 자체를 '특화 학습'에서 '지식의 전이 및 적응'으로 바꾸는 시도이다. SelaVPR과 같은 모델은 파운데이션 모델을 기반으로 하되, VPR 작업을 위한 최소한의 경량 어댑터(adapter)만 추가로 학습하여 두 패러다임의 장점을 결합하려는 종합(synthesis)을 시도하고 있다.50

이러한 흐름은 VPR 연구가 '특정 문제를 풀기 위한 특화된 모델 설계'에서 '거대한 사전 지식을 특정 문제에 효율적으로 적응시키는 방법론 연구'로 이동하고 있음을 보여주는 거시적인 트렌드이다.



### 6.4  Patch-NetVLAD가 후속 연구에 미친 영향과 그 유산



비록 최신 트랜스포머 및 파운데이션 모델 기반 접근법들이 SOTA 성능을 경신하고 있지만, Patch-NetVLAD가 VPR 분야에 남긴 영향과 유산은 지대하다.

첫째, Patch-NetVLAD는 **'전역 특징으로 빠르게 후보군을 찾고(coarse retrieval), 지역 특징으로 정밀하게 재순위화한다(fine re-ranking)'는 2단계 파이프라인을 VPR 분야의 사실상 표준(de facto standard)으로 확립**하는 데 결정적인 역할을 했다.5 이 구조는 대규모 데이터베이스에 대한 확장성(scalability)과 높은 정확도 사이의 균형을 맞추는 매우 효과적인 프레임워크로, 이후 TransVPR, R2Former 등 수많은 후속 연구에서 채택되었다.

둘째, **'지역-전역 디스크립터'라는 혁신적인 개념**은 전역 디스크립터의 강인함과 지역 디스크립터의 정밀함을 어떻게 효과적으로 결합할 수 있는지에 대한 구체적인 청사진을 제시했다. 이는 단순히 두 종류의 특징을 병렬로 추출하여 사용하는 것을 넘어, 하나의 VPR 최적화된 특징 공간에서 두 종류의 특징을 유기적으로 파생시키는 새로운 방향을 열었다. Contextual Patch-NetVLAD와 같이 Patch-NetVLAD를 직접적으로 계승하고 확장하여, 패치 주변의 컨텍스트 정보를 활용하거나 패치에 동적인 가중치를 부여하는 연구들이 그 대표적인 예이다.24

결론적으로 Patch-NetVLAD는 CNN 기반 VPR 기술의 정점이자, 그 가능성을 극한까지 탐색한 연구로 평가받는다. 동시에, 그 내재적 한계점들은 자연스럽게 다음 세대 기술인 트랜스포머 기반 접근법이 해결해야 할 과제를 명확히 제시하는 '다리' 역할을 수행했다. 따라서 Patch-NetVLAD는 VPR 기술 발전사에서 한 시대를 성공적으로 마무리하고 다음 시대로의 전환을 이끈 중요한 학술적, 기술적 유산으로 기록될 것이다.



## 7. 결론: Patch-NetVLAD의 기여와 시사점 요약



본 보고서는 CVPR 2021에서 발표된 **Patch-NetVLAD: Multi-scale Fusion of Locally-Global Descriptors for Place Recognition** 논문에 대한 심층적인 분석을 제공했다. 분석을 통해 도출된 핵심 결론과 시사점은 다음과 같다.

**Patch-NetVLAD의 핵심 기여:**

1. **혁신적인 하이브리드 디스크립터 제시:** '지역-전역(Locally-Global)' 디스크립터라는 새로운 개념을 통해, 기존의 전역 디스크립터가 갖는 외관 변화에 대한 강인함과 지역 디스크립터가 갖는 시점 변화에 대한 강인함을 성공적으로 결합했다. 이는 VPR 분야의 오랜 난제였던 두 접근법의 장점 통합에 대한 구체적이고 효과적인 해결책을 제시한 것이다.
2. **효율적인 다중 스케일 융합 구현:** IntegralVLAD라는 개념을 도입하고 이를 깊이별 확장 컨볼루션을 통해 효율적으로 구현함으로써, 추가적인 계산 비용을 최소화하면서 다중 스케일 정보 융합의 성능 향상 이점을 온전히 누릴 수 있게 했다. 이는 학술적 성능뿐만 아니라 실제 시스템 적용을 고려한 실용적인 설계의 결과물이다.
3. **표준 파이프라인 정립:** '전역 검색 후 지역 재순위화'로 이어지는 2단계 파이프라인을 성공적으로 구현하고 그 효과를 입증함으로써, 이후 VPR 연구의 표준적인 프레임워크를 제시했다. 이는 대규모 데이터베이스에 대한 확장성과 높은 정확도를 동시에 달성하는 검증된 방법론으로 자리 잡았다.
4. **객관적 성능 입증:** 다수의 도전적인 벤치마크 데이터셋에서 SOTA 성능을 달성하고, ECCV 2020 VPR 챌린지에서 우승함으로써 제안된 방법론의 우수성을 객관적으로 입증했다. 또한, 잘 구성된 코드와 사전 학습 모델을 공개하여 연구의 재현성을 높이고 후속 연구를 촉진하는 데 크게 기여했다.

시사점 및 미래 방향:

Patch-NetVLAD는 CNN 기반 VPR 아키텍처의 성능을 극한까지 끌어올린 중요한 연구이다. 이 연구는 VPR 문제가 단순히 더 좋은 특징 추출기를 만드는 것을 넘어, 추출된 특징을 어떻게 효과적으로 집계하고, 공간 정보를 보존하며, 계산 효율성을 고려하여 파이프라인을 설계하는지에 따라 성능이 크게 좌우되는 시스템 엔지니어링의 문제임을 명확히 보여주었다.

비록 VPR 분야의 패러다임이 트랜스포머와 파운데이션 모델로 빠르게 이동하면서 Patch-NetVLAD가 더 이상 최고의 성능을 내는 모델은 아닐지라도, 이 연구가 제기한 문제의식과 해결 방식은 여전히 유효하다. '어떻게 하면 지역적 정밀성과 전역적 맥락을 동시에 포착할 것인가?', '성능과 속도 사이의 트레이드오프를 어떻게 관리할 것인가?'와 같은 질문들은 아키텍처가 바뀌어도 VPR 연구의 핵심 과제로 남아있다.

결론적으로, Patch-NetVLAD는 CNN 시대의 VPR 기술을 집대성하고, 그 한계를 명확히 함으로써 다음 기술 세대로의 전환을 자연스럽게 이끈 중요한 학술적, 기술적 유산으로 평가할 수 있다. 이 연구에 대한 깊이 있는 이해는 현재의 VPR 기술 트렌드를 파악하고 미래 연구 방향을 설정하는 데 있어 필수적인 기반이 될 것이다.



#### **참고 자료**

1. arxiv.org, accessed July 1, 2025, https://arxiv.org/html/2401.13082v2
2. Visual Place Recognition. Introduction | by SOMIK DHAR - Medium, accessed July 1, 2025, https://medium.com/@sd5023/visual-place-recognition-8999307ebb2f
3. Collaborative Visual Place Recognition through Federated Learning - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024W/FedVision-2024/papers/Dutto_Collaborative_Visual_Place_Recognition_through_Federated_Learning_CVPRW_2024_paper.pdf
4. Visual place recognition for aerial imagery: A survey - arXiv, accessed July 1, 2025, https://arxiv.org/html/2406.00885v1
5. Patch-NetVLAD is a novel condition and viewpoint invariant visual place... - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/figure/Patch-NetVLAD-is-a-novel-condition-and-viewpoint-invariant-visual-place-recognition_fig1_349776464
6. (PDF) Visual Place Recognition: A Tutorial - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/374128870_Visual_Place_Recognition_A_Tutorial
7. Visual Place Recognition: Building an Evaluation Framework for Model Robustness, accessed July 1, 2025, http://essay.utwente.nl/100860/1/Gosa_BA_EEMCS.pdf
8. Visual Place Recognition: A Tutorial - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/374128870_Visual_Place_Recognition_A_Tutorial/fulltext/650ef47cd5293c106cd8b0cb/Visual-Place-Recognition-A-Tutorial.pdf?origin=scientificContributions
9. Patch-NetVLAD: Multi-Scale Fusion of Locally ... - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Hausler_Patch-NetVLAD_Multi-Scale_Fusion_of_Locally-Global_Descriptors_for_Place_Recognition_CVPR_2021_paper.pdf
10. MubarizZaffar/VPR_Tutorial: A basic tutorial (theory and practicals) for Visual Place Recognition. - GitHub, accessed July 1, 2025, https://github.com/MubarizZaffar/VPR_Tutorial
11. NetVLAD Place-Recognition Head - Emergent Mind, accessed July 1, 2025, https://www.emergentmind.com/articles/netvlad-place-recognition-head
12. This may be the author's version of a work that was submitted/accepted for publication in the following source: Hausler, Steph - QUT ePrints, accessed July 1, 2025, https://eprints.qut.edu.au/213030/1/CVPR_2021_Patch_NetVLAD.pdf
13. Visual Place Recognition Based on Dynamic Difference and Dual-Path Feature Enhancement - MDPI, accessed July 1, 2025, https://www.mdpi.com/1424-8220/25/13/3947
14. arXiv:2412.06153v1 [cs.CV] 9 Dec 2024, accessed July 1, 2025, https://arxiv.org/pdf/2412.06153
15. [2103.01486] Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2103.01486
16. (PDF) Visual Place Recognition: A Survey - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/284754832_Visual_Place_Recognition_A_Survey
17. Visual Place Recognition: A Survey - QUT ePrints, accessed July 1, 2025, https://eprints.qut.edu.au/105651/1/visual_place_recognition_a_survey_revised_final.pdf
18. NetVLAD Place-Recognition Head - Emergent Mind, accessed July 1, 2025, https://www.emergentmind.com/topics/netvlad-place-recognition-head
19. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - Medium, accessed July 1, 2025, https://medium.com/data-science/netvlad-cnn-architecture-for-weakly-supervised-place-recognition-ce64b08bebaf
20. NetVLAD: CNN architecture for weakly supervised place recognition, accessed July 1, 2025, https://www.di.ens.fr/~josef/publications/Arandjelovic17.pdf
21. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content_cvpr_2016/papers/Arandjelovic_NetVLAD_CNN_Architecture_CVPR_2016_paper.pdf
22. Non-local NetVLAD Encoding for Video ... - Google Research, accessed July 1, 2025, https://research.google.com/youtube8m/workshop2018/c_04.pdf
23. Unified Depth-Guided Feature Fusion and Reranking for Hierarchical Place Recognition, accessed July 1, 2025, https://www.mdpi.com/1424-8220/25/13/4056
24. Contextual Patch-NetVLAD: Context-Aware Patch Feature Descriptor and Patch Matching Mechanism for Visual Place Recognition - PMC, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10857504/
25. Patch-NetVLAD: Multi-Scale Fusion of Locally ... - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2021/supplemental/Hausler_Patch-NetVLAD_Multi-Scale_Fusion_CVPR_2021_supplemental.pdf
26. A complete walkthrough of convolution operations - viso.ai, accessed July 1, 2025, https://viso.ai/deep-learning/convolution-operations/
27. Dilated Convolution - GeeksforGeeks, accessed July 1, 2025, https://www.geeksforgeeks.org/machine-learning/dilated-convolution/
28. A Primer on Atrous Convolutions and Depth-wise Separable Convolutions, accessed July 1, 2025, https://towardsdatascience.com/a-primer-on-atrous-convolutions-and-depth-wise-separable-convolutions-443b106919f5/
29. Full article: An efficient and accurate deep learning method for tree species classification that integrates depthwise separable convolution and dilated convolution using hyperspectral data - Taylor & Francis Online, accessed July 1, 2025, https://www.tandfonline.com/doi/full/10.1080/17538947.2024.2307999
30. 7 Different Convolutions for designing CNNs that will Level-up your Computer Vision project, accessed July 1, 2025, https://medium.com/codex/7-different-convolutions-for-designing-cnns-that-will-level-up-your-computer-vision-project-fec588113a64
31. Contextual Patch-NetVLAD: Context-Aware Patch Feature Descriptor and Patch Matching Mechanism for Visual Place Recognition - MDPI, accessed July 1, 2025, https://www.mdpi.com/1424-8220/24/3/855
32. Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition, accessed July 1, 2025, https://paperswithcode.com/paper/patch-netvlad-multi-scale-fusion-of-locally
33. QVPR - GitHub, accessed July 1, 2025, https://github.com/QVPR
34. QVPR/Patch-NetVLAD: Code for the CVPR2021 paper ... - GitHub, accessed July 1, 2025, https://github.com/QVPR/Patch-NetVLAD
35. feature_extract.py - QVPR/Patch-NetVLAD - GitHub, accessed July 1, 2025, https://github.com/QVPR/Patch-NetVLAD/blob/main/feature_extract.py
36. Patch-NetVLAD/match_two.py at main - GitHub, accessed July 1, 2025, https://github.com/QVPR/Patch-NetVLAD/blob/main/match_two.py
37. Patch-NetVLAD/match_realtime.py at main - GitHub, accessed July 1, 2025, https://github.com/QVPR/Patch-NetVLAD/blob/main/match_realtime.py
38. This may be the author's version of a work that was submitted/accepted for publication in the following source: Khaliq, Ahmad, - QUT ePrints, accessed July 1, 2025, https://eprints.qut.edu.au/252913/1/_ECCV_2024_Burstiness_NetVLAD.pdf
39. VLAD-BuFF: Burst-aware Fast Feature Aggregation for Visual Place ..., accessed July 1, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/06086.pdf
40. A survey of the Vision Transformers and their CNN-Transformer based Variants - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2305.09880
41. [2405.18065] EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition, accessed July 1, 2025, https://arxiv.org/abs/2405.18065
42. Towards Seamless Adaptation of Pre-trained Models for Visual Place Recognition, accessed July 1, 2025, https://openreview.net/forum?id=TVg6hlfsKa
43. Vision Transformers vs CNNs at the Edge, accessed July 1, 2025, https://www.edge-ai-vision.com/2024/03/vision-transformers-vs-cnns-at-the-edge/
44. Vision Transformer vs. CNN: A Comparison of Two Image Processing Giants - Medium, accessed July 1, 2025, https://medium.com/@hassaanidrees7/vision-transformer-vs-cnn-a-comparison-of-two-image-processing-giants-d6c85296f34f
45. [2401.13082] PlaceFormer: Transformer-based Visual Place Recognition using Multi-Scale Patch Selection and Fusion - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2401.13082
46. What are Foundation Models? - Generative AI - AWS, accessed July 1, 2025, https://aws.amazon.com/what-is/foundation-models/
47. AnyLoc: Towards Universal Visual Place Recognition, accessed July 1, 2025, https://anyloc.github.io/
48. EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition - arXiv, accessed July 1, 2025, https://arxiv.org/html/2405.18065v2
49. Towards Universal Visual Place Recognition - AnyLoc, accessed July 1, 2025, https://anyloc.github.io/assets/AnyLoc.pdf
50. Lu-Feng/SelaVPR: Official repository for the ICLR 2024 paper "Towards Seamless Adaptation of Pre-trained Models for Visual Place Recognition". - GitHub, accessed July 1, 2025, https://github.com/Lu-Feng/SelaVPR
51. ETR: An Efficient Transformer for Re-Ranking in Visual Place Recognition - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/WACV2023/papers/Zhang_ETR_An_Efficient_Transformer_for_Re-Ranking_in_Visual_Place_Recognition_WACV_2023_paper.pdf
52. ETR: An Efficient Transformer for Re-ranking in Visual Place Recognition - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/368317128_ETR_An_Efficient_Transformer_for_Re-ranking_in_Visual_Place_Recognition
# 변화하는 환경에서의 강인한 Visual Place Recognition (VPR)



Visual Place Recognition (VPR)은 이전에 방문했던 장소를 오직 시각 정보, 즉 이미지만을 사용하여 재인식하는 능력으로 정의된다.1 이는 자율적으로 움직이는 로봇이나 인공지능 시스템이 주변 환경 속에서 자신의 위치를 파악하고자 하는 근본적인 질문, "나는 어디에 있는가?"에 답하는 핵심적인 과정이다. VPR 시스템은 현재 입력된 시각 정보가 이전에 구축한 환경 지도(map) 내에 포함된 장소인지, 만약 그렇다면 어느 장소인지를 판단하는 믿음(belief)을 보고할 수 있어야 한다.5

본질적으로 VPR은 이미지 검색(Image Retrieval) 문제의 한 형태로 볼 수 있다.1 주어진 질의(query) 이미지와 가장 유사한 이미지를 대규모 데이터베이스에서 찾아내는 것이 목표이기 때문이다. 그러나 VPR은 '장소'라는 지리적, 공간적 맥락이 추가된다는 점에서 일반적인 이미지 검색과 근본적인 차이를 보인다. 이 맥락적 정보는 단순히 시각적으로 유사한 이미지를 찾는 것을 넘어, 로봇의 이동 경로에서 얻어지는 순차적인 이미지 정보나 시간의 흐름에 따른 환경 변화 등을 고려할 수 있는 중요한 단서를 제공한다.1 따라서 VPR은 단순한 이미지 매칭 기술이 아니라, 자율 시스템의 '인지(Cognition)' 능력의 핵심 요소로 기능한다. SLAM에서의 오차 보정이나 GPS 음영 지역에서의 위치 추정 같은 응용 사례들은 모두 시스템이 자신의 상태와 환경을 지속적으로 파악하고, 과거의 기억(지도)과 현재의 관측을 비교하여 불일치를 인지하고 교정하는 고차원적인 인지 활동에 해당한다. 이처럼 VPR은 기술적 문제 해결을 넘어, 로봇에게 환경에 대한 장기적인 기억과 상황 인지 능력을 부여하는 근간 기술이라 할 수 있다.


VPR은 다양한 자율 시스템에서 필수적인 역할을 수행하며, 특히 로보틱스와 자율 주행 분야에서 그 중요성이 두드러진다.


SLAM은 로봇이 미지의 환경을 탐험하면서 동시에 자신의 위치를 추정하고 지도를 작성하는 기술이다. 이 과정에서 로봇의 움직임을 추정할 때 작은 오차들이 계속 누적되어 드리프트(drift) 현상이 발생하는데, VPR은 이를 해결하는 데 결정적인 역할을 한다. VPR을 통해 루프 폐쇄(Loop-Closure Detection)를 수행할 수 있기 때문이다.5 루프 폐쇄란 로봇이 이전에 방문했던 장소로 되돌아왔음을 인식하는 과정으로, 이 인식을 통해 "과거의 그 위치"와 "현재의 이 위치"가 동일하다는 강력한 제약 조건을 생성할 수 있다. 이 제약 조건은 누적된 경로 오차를 전역적으로 최적화하고 수정하여, 일관성 있고 정확한 지도를 구축하는 데 필수적이다.8


GPS 신호가 약하거나 수신되지 않는 환경, 예를 들어 고층 빌딩이 밀집한 도심 협곡(urban canyon), 실내, 터널 등에서 VPR은 로봇이나 차량의 전역 위치를 추정하는 강력한 대안을 제공한다.9 사전에 구축된, 위치 정보가 태깅된 이미지 데이터베이스(지도)와 현재 카메라로 입력된 이미지를 비교하여 가장 일치하는 장소를 찾아냄으로써 자신의 위치를 추정할 수 있다. 이는 로봇이 자신의 위치를 완전히 잃어버렸을 때 초기 위치를 찾는 '납치된 로봇 문제(Kidnapped-Robot Problem)'를 해결하는 데도 직접적으로 적용된다.8


자율 주행 시스템에서 VPR은 여러 핵심 모듈과 긴밀하게 상호작용하며 시스템의 안정성과 정확성을 높이는 기반 기술로 작동한다.9

- **고정밀 측위 (High-Precision Localization):** VPR은 고정밀 지도(HD Map)와 결합하여 GPS가 불안정한 지역에서도 차량의 정확한 위치를 핀포인트로 추정하는 측위 모듈의 핵심 구성 요소로 기능한다.9
- **환경 인식 지원 (Environment Perception Support):** VPR은 현재 위치 정보를 환경 인식 모듈에 피드백으로 제공한다. 이를 통해 인식 모듈은 실시간 센서 데이터와 지도 정보를 비교하여 차선이나 교통 표지판 인식 오류를 보정할 수 있다.9
- **경로 계획의 기반 (Path Planning Foundation):** VPR이 제공하는 정확한 위치 정보는 경로 계획 모듈이 현재 도로 상황에 맞춰 최적의 경로를 선택하고, 실시간 교통 변화에 따라 경로를 동적으로 수정하는 데 필수적인 기반 데이터가 된다.9


이 튜토리얼의 핵심 주제는 '변화하는 조건' 하에서의 VPR이다. 실제 세계는 정적이지 않고 끊임없이 변화하며, 이러한 변화는 장소의 시각적 외형을 극단적으로 바꾸어 VPR 시스템의 성능을 심각하게 저하시킨다.12 이는 단순히 이미지의 픽셀 값이 변하는 수준을 넘어, 장소의 시각적 정체성 자체를 뒤흔드는 근본적인 문제이다. 하루 동안의 조명 변화(주간/야간), 계절의 변화(여름/겨울), 날씨의 변화(맑음/비/눈)는 동일한 장소를 전혀 다른 모습으로 보이게 만드는 주된 요인들이다.1 이러한 극심한 외형 변화에 강인한 장소 인식 기술을 개발하는 것이 VPR 연구의 가장 중요한 목표 중 하나이다.



VPR 문제는 수학적으로 다음과 같이 공식화할 수 있다. 위치 정보가 알려진 장소들로 구성된 데이터베이스 이미지 집합 $DB = {I_i}$와, 위치를 알고자 하는 하나 이상의 질의 이미지 집합 $Q = {I_j}$가 주어진다. 이때 VPR의 목표는 질의 이미지 $I_j$와 데이터베이스 이미지 $I_i$가 동일한 물리적 장소를 촬영한 것인지 판별하여, 일치하는 매칭 쌍 $(I_i, I_j)$을 찾아내는 것이다.1

이 매칭을 수행하기 위해서는 각 이미지를 비교 가능한 형태로 변환하는 과정이 필수적이다. 이를 위해 각 이미지 $I$는 고유한 숫자 벡터인 **디스크립터(descriptor)** $d$로 인코딩된다. 이상적인 디스크립터는 다음과 같은 특성을 가져야 한다 1:

- **유사성(Similarity):** 같은 장소를 촬영한 두 이미지 $I_a$와 $I_p$(positive pair)로부터 추출된 디스크립터 $d_a$와 $d_p$는 특징 공간(feature space)에서 서로 가까워야 한다. 즉, 두 디스크립터 간의 거리 $distance(d_a, d_p)$가 작아야 한다.
- **비유사성(Dissimilarity):** 서로 다른 장소를 촬영한 두 이미지 $I_a$와 $I_n$(negative pair)으로부터 추출된 디스크립터 $d_a$와 $d_n$은 특징 공간에서 서로 멀어야 한다. 즉, 거리 $distance(d_a, d_n)$가 커야 한다.

개념적으로, 모든 질의 이미지 디스크립터와 데이터베이스 이미지 디스크립터 간의 모든 쌍별 유사도(pairwise similarity)를 계산하여 **유사도 행렬(Similarity Matrix)** $S$를 구성할 수 있다. 이 행렬의 원소 $s_ij$는 질의 이미지 $I_j$의 디스크립터와 데이터베이스 이미지 $I_i$의 디스크립터 간의 유사도를 나타내며, 이 행렬은 최종 매칭 결정을 내리는 기반이 된다.1


대부분의 VPR 시스템은 다음과 같은 3단계의 일반적인 파이프라인을 따른다.

1. **특징 추출 (Feature Extraction):** 이미지의 원본 픽셀 데이터로부터 의미 있는 시각적 정보를 추출하는 단계이다. 초기 VPR 시스템에서는 SIFT(Scale-Invariant Feature Transform), SURF(Speeded Up Robust Features)와 같은 수동 특징점(hand-crafted features)을 주로 사용했다. 하지만 최근에는 합성곱 신경망(Convolutional Neural Networks, CNNs)을 통해 이미지의 복잡하고 추상적인 정보를 담고 있는 심층 특징(deep features)을 추출하는 것이 표준적인 접근법이 되었다.2
2. **디스크립터 인코딩 (Descriptor Encoding):** 추출된 지역적 또는 전역적 특징들을 고정된 크기의 벡터, 즉 **전역 디스크립터(global descriptor)**로 압축하고 인코딩하는 단계이다. 이 디스크립터는 이미지 전체의 핵심적인 정보를 요약하여 담고 있어야 하며, 일반적으로 128차원에서 4,096차원 또는 그 이상의 고차원 벡터로 표현된다.1 이 단계의 목표는 이미지의 정보를 효율적으로 비교하고 검색할 수 있는 형태로 만드는 것이다.
3. **매칭 (Matching):** 질의 이미지로부터 생성된 전역 디스크립터를 데이터베이스에 저장된 모든 이미지의 디스크립터와 비교하여 가장 유사한 후보 이미지를 찾는 단계이다. 디스크립터 간의 유사도는 주로 유클리드 거리(Euclidean distance)나 코사인 유사도(cosine similarity)와 같은 거리 척도(distance metric)를 사용하여 계산된다.7 계산된 거리가 가장 작은(또는 유사도가 가장 큰) 데이터베이스 이미지가 최종 매칭 결과로 선택된다.



VPR에서 '장소'를 어떻게 정의하는가는 매우 중요한 문제이다. 이 튜토리얼에서는 실용적인 관점에서 다음과 같은 정의를 채택한다: **두 이미지가 동일한 건물, 구조물 등 공통된 시각적 콘텐츠를 공유하여 상당한 수준의 시각적 중첩(visual overlap)을 가질 때, 두 이미지는 '같은 장소'에서 촬영된 것으로 간주한다**.1

이러한 정의는 VPR의 목표와 직결되는 철학적 문제이자 기술적 문제에 대한 실용적인 해답을 제시한다. VPR의 궁극적인 목표 중 하나는 단순히 이미지를 검색하는 것을 넘어 로봇의 정확한 위치를 파악하는 '측위(localization)'이다.5 정확한 위치를 계산하기 위해서는, 매칭된 두 이미지 간의 상대적인 카메라 포즈(위치와 방향의 3차원 변환)를 추정할 수 있어야 한다. 상대 포즈를 계산하려면, 두 이미지에서 공통적으로 관측되는 특징점들이 충분히 존재해야 하는데, 이것이 바로 '시각적 중첩'의 본질이다. 만약 '장소'를 단순히 'GPS 좌표가 10m 이내'와 같이 비시각적인 기준으로 정의한다면, 두 이미지가 서로 반대 방향을 보고 있어 시각적 중첩이 전혀 없음에도 '같은 장소'로 레이블링될 수 있다. 이 경우, VPR 시스템이 매칭에 성공하더라도 후속 단계인 상대 포즈 계산이 불가능해져 최종 목표인 측위로 이어질 수 없다. 따라서 '시각적 중첩'을 장소의 정의로 채택하는 것은 VPR을 단순 검색 문제에서 측위 문제로 확장하기 위한 필연적인 귀결이다.


VPR 문제는 데이터의 수집 방식에 따라 크게 두 가지 유형으로 나뉜다.

- **단일 세션 VPR (Single-session VPR):** 하나의 연속적인 이미지 시퀀스 내에서 매칭을 수행하는 경우로, 질의 집합 $Q$와 데이터베이스 집합 $DB$가 동일하다 ($Q = DB$). 이 유형은 주로 SLAM 시스템에서 로봇이 자신의 경로를 따라 이동하다가 이전에 방문했던 지점을 다시 인식하는 루프 폐쇄 감지에 사용된다. 이 경우, 시간적으로 바로 직전에 촬영된 이미지와의 사소한 매칭을 억제하는 것이 중요한 기술적 고려사항이 된다.3
- **다중 세션 VPR (Multi-session VPR):** 서로 다른 시간에(예: 여름과 겨울), 다른 기상 조건에서, 또는 다른 플랫폼으로(예: 로봇과 스마트폰) 촬영된 두 개의 분리된 이미지 집합 간의 매칭을 수행하는 경우이다 ($Q ∩ DB = ∅$). 이 유형은 본 튜토리얼의 핵심 주제인 '변화하는 조건'에서의 VPR 강인성을 연구하고 평가하는 데 필수적이다.3


VPR 시스템이 실제 환경에서 마주하는 가장 큰 어려움은 환경의 동적인 변화이다. 이러한 변화는 크게 네 가지 범주로 나눌 수 있으며, 이들은 종종 복합적으로 발생하여 문제를 더욱 어렵게 만든다.


동일한 장소의 시각적 외형이 시간의 흐름에 따라 극적으로 변하는 현상으로, VPR 성능 저하의 가장 주된 원인이다.

- **조명 변화 (Illumination Change):** 하루 중 시간대에 따른 조명 변화, 특히 주간과 야간의 차이는 가장 극단적인 외형 변화를 야기한다.12 태양광의 방향과 강도에 따라 그림자가 길어지거나 사라지고, 해질녘에는 전체적인 색감이 붉게 변한다. 야간에는 인공 조명, 네온사인, 차량 전조등에 의한 강한 하이라이트와 깊은 그림자가 발생하며, 이는 이미지의 밝기 분포(histogram)와 텍스처(texture)를 주간과 완전히 다르게 만든다. 이러한 변화는 인간의 시각 시스템에도 도전적이며, 전통적인 특징 기반 VPR 방법들의 성능을 급격히 저하시키는 주된 요인이다.14
- **계절 및 날씨 변화 (Seasonal and Weather Change):** 계절의 변화는 장소의 모습을 근본적으로 바꾼다. 여름의 무성한 녹음과 단풍, 겨울의 앙상한 나뭇가지와 눈 덮인 풍경은 동일한 장소의 색상, 텍스처, 심지어는 가려짐으로 인한 구조 정보까지 변화시킨다.11 또한, 비, 눈, 안개와 같은 악천후는 이미지의 대비(contrast)와 선명도를 저하시키고, 빗방울이나 눈송이가 이미지에 노이즈를 추가하며, 젖은 표면은 빛을 불규칙하게 반사시켜 외형을 왜곡한다.15


로봇이나 차량이 동일한 장소에 다른 경로로 접근하거나 다른 시점에서 관찰할 때 발생하는 문제이다.1 예를 들어, 반대편 차선에서 같은 교차로를 촬영하거나, 다른 각도에서 건물을 바라볼 경우, 이미지에 담기는 객체의 형태, 크기, 상대적 위치, 원근감이 크게 달라진다. 이로 인해 지역 특징점(local feature) 기반의 매칭이 어려워진다. 특히, 앞서 언급한 극심한 외형 변화와 급격한 시점 변화가 동시에 발생할 때(예: 겨울 밤에 반대편 차선에서 장소 인식), VPR 문제는 가장 어려운 난이도에 직면하게 된다.5


이는 서로 다른 물리적 위치에 있는 장소들이 시각적으로 매우 유사하게 보이는 문제이다.1 실내 환경의 반복적인 복도와 사무실, 도시 환경의 유사한 건물 외관, 고속도로의 비슷한 구간, 자연 환경의 유사한 숲길 등이 대표적인 예이다. 지각적 모호성은 VPR 시스템이 잘못된 위치를 정답으로 확신하게 만드는 심각한 오류의 원인이 된다. 특히, 변화하는 조건 하에서는 같은 장소의 두 이미지가 서로 다르게 보이는 정도보다, 다른 장소의 두 이미지가 더 유사하게 보일 수 있어 문제를 더욱 복잡하게 만든다.5


실제 환경은 정적이지 않으며, 다양한 동적 요소들을 포함한다.

- **동적 객체 및 가림 (Dynamic Objects and Occlusion):** 도로 위의 다른 차량, 보행자, 자전거 등은 장소의 고유한 특징을 일시적으로 가리는 주요 원인이다.5 주차된 차량의 위치가 바뀌거나, 버스가 건물을 가리는 등의 상황은 VPR 시스템에 잘못된 정보를 제공하여 매칭 실패를 유발할 수 있다.
- **장기적 구조 변화 (Long-term Structural Change):** 장기간에 걸쳐 로봇이 운용되는 경우, 환경 자체의 구조가 변할 수 있다. 새로운 건물이 들어서거나 기존 건물이 철거되는 공사, 도로 구조의 변경, 가로수의 성장 등은 데이터베이스에 저장된 과거의 장소 정보와 현재의 관측 사이에 영구적인 불일치를 만들어낸다.17

VPR의 이러한 핵심 난제들은 독립적으로 작용하기보다 서로의 영향을 증폭시키는 복합적인 관계를 가진다. 예를 들어, 야간의 조명 변화는 세부적인 텍스처나 색상 정보를 소실시켜 구조적으로 유사한 장소들(예: 다른 상점이 있는 두 개의 복도)을 구별 불가능하게 만들어 지각적 모호성을 심화시킨다. 또한, 시점 변화는 동적 객체에 의한 가림의 패턴을 예측 불가능하게 만들어, 이전에는 보였던 정적 구조물이 가려지고 새로운 동적 객체가 나타나는 등 상황을 복잡하게 만든다. 따라서 강인한 VPR 시스템은 이러한 문제들을 개별적으로 해결하는 것을 넘어, 여러 변화 요인이 복합적으로 작용하는 실제 상황에 효과적으로 대응할 수 있는 통합적이고 근본적인 불변 특징(invariant feature)을 학습해야 할 필요가 있다.


딥러닝이 VPR 분야를 지배하기 이전, 연구자들은 컴퓨터 비전의 전통적인 기법들을 활용하여 장소 인식 문제를 해결하고자 했다. 이러한 초기 접근법들은 주로 수동으로 설계된 특징점(hand-crafted features)과 이를 집계하는 방식에 의존했다.


초기 VPR 연구는 이미지에서 안정적이고 반복적으로 검출할 수 있는 지역 특징점(local feature points)을 찾는 데 집중했다. 대표적인 알고리즘으로 SIFT(Scale-Invariant Feature Transform)와 SURF(Speeded Up Robust Features)가 있다.

- **기본 원리:** 이 알고리즘들은 이미지 내에서 코너나 엣지와 같이 주변과 구별되는 독특한 지점, 즉 **키포인트(keypoint)**를 먼저 검출한다. 그 후, 각 키포인트 주변의 픽셀 영역에서 그래디언트의 방향과 크기 분포와 같은 정보를 계산하여 고유한 숫자 벡터인 **디스크립터(descriptor)**를 생성한다.19 SIFT는 128차원, SURF는 64차원 디스크립터를 생성한다.
- **장점:** SIFT는 이미지의 크기(scale) 변화와 회전(rotation)에 강인한 불변성을 가지도록 설계되었고, SURF는 SIFT의 성능을 유지하면서 적분 이미지(integral image) 등을 활용하여 계산 속도를 크게 향상시켰다. 또한 그래디언트 기반 특성 덕분에 어느 정도의 조명 변화에도 강인함을 보인다.19
- **핵심 한계:** 이들의 가장 큰 한계는 수동으로 설계된(hand-crafted) 특징이라는 점이다. 이들은 주로 기하학적 변환(크기, 회전)에 대한 불변성을 목표로 설계되었기 때문에, VPR의 핵심 난제인 극심한 외형 변화, 즉 측광학적(photometric) 변화에는 매우 취약하다. 주간과 야간, 여름과 겨울처럼 이미지의 픽셀 값과 그래디언트 분포가 비선형적으로 완전히 바뀌는 상황에서는 동일한 물리적 지점에서 안정적인 키포인트를 반복적으로 검출하거나, 일관된 디스크립터를 생성하는 데 실패한다.22 결국, VPR 문제에 객체 인식용으로 설계된 도구를 그대로 적용한 것이 전통적 접근법의 근본적인 한계였다. 이는 VPR 문제가 요구하는 '외형 변화에 대한 불변성'을 데이터로부터 직접 학습해야 한다는 새로운 패러다임, 즉 딥러닝의 필요성을 명확히 보여주었다.


개별 특징점들을 직접 매칭하는 것은 계산 비용이 높고 대규모 데이터베이스에 적용하기 어렵기 때문에, 이미지 전체를 하나의 고정된 벡터로 표현하는 전역 디스크립터(global descriptor)를 만드는 방법이 필요했다. Bag-of-Visual-Words (BoVW)는 이를 위한 대표적인 초기 기법으로, 자연어 처리의 Bag-of-Words 모델에서 영감을 받았다.25


BoVW 모델은 다음과 같은 3단계 파이프라인으로 구성된다.

1. **특징 추출 (Feature Extraction):** 데이터베이스의 모든 이미지에서 SIFT나 SURF와 같은 다수의 지역 디스크립터들을 추출한다.27
2. **코드북 생성 (Codebook Generation):** 추출된 모든 지역 디스크립터들을 하나의 거대한 집합으로 모은다. 이 집합에 대해 K-means와 같은 클러스터링 알고리즘을 적용하여 K개의 클러스터 중심점(centroid)을 찾는다. 이 K개의 중심점들이 바로 **'시각 단어 사전(visual vocabulary)'** 또는 **코드북(codebook)**이 된다.26 각 시각 단어는 특정 유형의 지역 패턴(예: 특정 모양의 창문, 벽돌 텍스처)을 대표한다.
3. **양자화 및 히스토그램 생성 (Quantization and Histogram Generation):** 새로운 이미지가 주어지면, 먼저 지역 디스크립터들을 추출한다. 그 후, 각 디스크립터를 코드북 내에서 가장 가까운 시각 단어에 할당(양자화)한다. 마지막으로, 이미지 내에 각 시각 단어가 몇 번 등장하는지를 세어 빈도수 **히스토그램(histogram)**을 만든다. 이 K차원의 히스토그램이 해당 이미지의 전역 디스크립터가 된다.29 두 이미지의 장소 유사도는 이 히스토그램 벡터 간의 유사도(예: 코사인 유사도)로 측정된다.


BoVW는 이미지 검색 분야에서 큰 성공을 거두었지만, VPR 문제, 특히 변화하는 조건 하에서는 여러 근본적인 한계를 드러냈다.

- **공간 정보 손실 (Loss of Spatial Information):** BoVW의 가장 치명적인 단점은 특징들의 공간적 배치 정보를 완전히 무시한다는 것이다.20 '창문', '문', '지붕'이라는 시각 단어들이 올바른 순서로 배열되어 '집'을 구성하는지, 아니면 무작위로 흩어져 있는지를 구분하지 못한다. 이로 인해 전혀 다른 구조를 가진 장소가 우연히 비슷한 시각 단어 구성을 가질 경우, 잘못된 매칭을 유발할 수 있다.
- **양자화 오류 (Quantization Error):** 디스크립터를 가장 가까운 단 하나의 시각 단어에 할당하는 '하드 할당(hard assignment)' 방식은 정보 손실을 야기한다. 어떤 디스크립터가 두 시각 단어의 경계에 위치할 경우, 미세한 차이로 인해 완전히 다른 단어에 할당될 수 있으며, 이는 디스크립터의 풍부한 정보를 잃게 만든다.30
- **변화에 대한 근본적 취약성:** BoVW 모델의 성능은 기반이 되는 지역 디스크립터(SIFT/SURF)의 품질에 크게 의존한다. 앞서 언급했듯이 이들 지역 디스크립터 자체가 극심한 외형 변화에 취약하기 때문에, 이를 기반으로 구축된 BoVW 히스토그램 역시 환경 변화에 따라 불안정하게 변동할 수밖에 없다.22

이러한 한계들은 VPR 문제가 단순히 지역 패턴의 집합이 아니라, 공간적 구조와 외형 변화에 대한 강인성을 동시에 고려해야 하는 복잡한 문제임을 보여준다. 이는 데이터로부터 직접 의미론적이고 강인한 표현을 학습하는 딥러닝 기반 접근법의 등장을 촉발하는 계기가 되었다.


전통적인 VPR 기법들이 수동으로 설계된 특징의 한계에 부딪히면서, 연구의 패러다임은 데이터로부터 직접 장소에 대한 강인한 표현(representation)을 학습하는 딥러닝으로 전환되었다.23 특히, 대규모 이미지 데이터셋으로 사전 훈련된 합성곱 신경망(CNN)이 VPR 문제 해결의 강력한 도구로 부상했다.


연구자들은 ImageNet과 같은 대규모 데이터셋으로 사전 훈련된 VGG, ResNet 등의 CNN 모델의 중간 계층에서 추출된 특징 맵(feature map)이 조명이나 계절 변화와 같은 외형 변화에 대해 상당한 불변성을 보인다는 사실을 발견했다.12 이는 CNN이 이미지의 저수준 텍스처부터 고수준의 의미론적 객체 정보까지 계층적인 특징을 학습하기 때문이다. 이러한 심층 특징(deep features)을 이미지 전체에 대해 집계(aggregate)하여 하나의 전역 디스크립터로 만드는 방식이 새로운 VPR의 주류로 자리 잡기 시작했다.


NetVLAD는 전통적인 VLAD(Vector of Locally Aggregated Descriptors) 기법을 딥러닝 프레임워크 내에서 종단간(end-to-end)으로 학습할 수 있도록 설계한 획기적인 아키텍처이다.32 이는 CNN 백본(backbone)의 마지막에 부착할 수 있는 미분 가능한 새로운 계층으로 제안되었다.


전통적인 VLAD는 K-means 클러스터링을 사용하여 코드북을 만들고, 각 지역 특징을 가장 가까운 클러스터 중심에 할당하는 '하드 할당' 방식을 사용한다. 이 과정은 미분 불가능하여 신경망의 역전파(backpropagation) 과정에 통합될 수 없다는 문제가 있었다.35

NetVLAD는 이 문제를 **소프트 할당(soft-assignment)** 개념을 도입하여 해결했다. CNN을 통과한 $N$개의 $D$차원 지역 디스크립터 $x_i$가 주어졌을 때, 각 디스크립터가 $K$개의 클러스터 중 $k$번째 클러스터에 속할 가중치 $a_k(x_i)$를 학습 가능한 파라미터 $w_k$, $b_k$와 소프트맥스 함수를 사용하여 계산한다.
$$
\bar{a}_k(\mathbf{x}_i) = \frac{e^{\mathbf{w}_k^T \mathbf{x}_i + b_k}}{\sum_{k'} e^{\mathbf{w}_{k'}^T \mathbf{x}_i + b_{k'}}}
$$
이 소프트 가중치 $a_k(x_i)$는 $x_i$가 클러스터 $k$에 얼마나 기여하는지를 나타내는 연속적인 값이다. 이 가중치를 사용하여 각 클러스터 $k$에 대한 VLAD 벡터 $V_k$를 계산한다. $c_k$는 $k$번째 클러스터의 학습 가능한 중심점(centroid)이다.
$$
\mathbf{V}(k,j) = \sum_{i=1}^{N} \bar{a}_k(\mathbf{x}_i) (x_i(j) - c_k(j))
$$
이 식은 각 클러스터 $k$에 대해, 모든 지역 디스크립터 $x_i$와 클러스터 중심 $c_k$ 간의 잔차(residual)를 소프트 가중치로 가중 합산한 것이다. 이렇게 계산된 $K$개의 $D$차원 벡터 $V_k$들을 하나의 긴 벡터로 펼치고(reshape), 내부 정규화(intra-normalization)와 L2 정규화(L2-normalization)를 거쳐 최종적으로 $K x D$ 차원의 NetVLAD 전역 디스크립터를 생성한다.32 이 모든 과정은 미분 가능하므로, 전체 네트워크를 역전파를 통해 종단간으로 학습시킬 수 있다.


NetVLAD의 또 다른 중요한 기여는 대규모의, 완벽하게 정제되지 않은 실제 세계 데이터를 활용하여 모델을 학습시키는 실용적인 프레임워크를 제시한 것이다. 이를 위해 **약지도 학습(weakly supervised learning)** 방식과 **트리플렛 손실(triplet loss)** 기반의 랭킹 손실 함수를 사용했다.32

- **데이터셋:** 연구진은 수동으로 '같은 장소' 이미지 쌍을 만드는 대신, GPS 정보가 태깅된 대규모의 Google Street View Time Machine 데이터셋을 활용했다. 이 데이터셋은 동일한 장소를 다른 시간대, 다른 계절에 촬영한 이미지들을 포함하지만, GPS 정보에는 오차가 있어 정확한 이미지 쌍 레이블은 없다.32
- **약한 레이블 생성:** 주어진 질의 이미지 $q$에 대해, GPS 거리를 기준으로 '잠재적 정답(potential positive)' 집합과 '명백한 오답(definite negative)' 집합을 정의한다. 예를 들어, 질의 이미지로부터 10m 이내에서 촬영된 이미지들은 잠재적 정답으로, 25m 이상 떨어진 이미지들은 명백한 오답으로 간주한다.35
- **손실 함수:** 학습 목표는 질의 이미지의 디스크립터 $d_q$가 잠재적 정답 이미지의 디스크립터 $d_p$보다는 가깝고, 명백한 오답 이미지의 디스크립터 $d_n$보다는 멀어지도록 하는 것이다. 이를 위해 트리플렛 손실 함수를 사용한다. 구체적으로, 질의 $q_i$에 대해 잠재적 정답 집합 내에서 가장 어려운(거리가 가장 먼) 정답 $p_i*$를 찾고, 이 $p_i*$와의 거리보다 모든 오답 $n_ij$와의 거리가 최소한 $m$(마진)만큼 더 멀어지도록 네트워크를 학습시킨다.35

$$
L(\{ \mathbf{p}_i^* \}) = \sum_i \sum_{j} \max(0, d^2(\mathbf{d}_{q_i}, \mathbf{d}_{p_i^*}) - d^2(\mathbf{d}_{q_i}, \mathbf{d}_{n_{ij}}) + m)
$$

이 '대규모 약지도 데이터 + 트리플렛 손실' 프레임워크는 이후 VPR 연구에서 표준적인 학습 방식으로 자리 잡았으며, VPR 분야가 실험실 수준의 소규모 데이터셋에서 벗어나 실제 세계의 대규모 데이터로 확장되는 결정적인 계기가 되었다.38


GeM 풀링은 CNN 특징 맵을 집계하는 또 다른 효과적인 방법으로, 최대 풀링(max pooling)과 평균 풀링(average pooling)을 일반화한 형태이다.39

- **수학적 공식:** $k$번째 특징 맵 $X_k$에 대해, GeM 풀링의 결과 $f_k$는 다음과 같이 정의된다.
  $$
  f_k = \left( \frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x^p \right)^{\frac{1}{p}}
  $$
  여기서 $p$는 풀링 파라미터이다. $p=1$이면 평균 풀링이 되고, $p$가 무한대로 가면($p → ∞$) 최대 풀링과 동일해진다. 이 파라미터 $p$는 미분 가능하므로 역전파를 통해 데이터로부터 학습될 수 있다.39

- **특징:** GeM은 NetVLAD에 비해 훨씬 간단하면서도 경쟁력 있는 성능을 제공한다. 특히, 생성되는 디스크립터의 차원이 낮아(예: 256~2048차원) 메모리 사용량과 검색 속도 면에서 큰 이점을 가진다. 이로 인해 실시간성과 메모리 효율성이 중요한 대규모 응용 분야에 매우 적합하다.41


이미지 내의 모든 영역이 장소 인식에 동등하게 중요한 것은 아니다. 건물, 독특한 구조물, 도로 표지판과 같은 정적이고 변별력 있는 영역은 장소의 정체성을 나타내는 중요한 단서가 되는 반면, 하늘, 도로, 움직이는 자동차나 보행자 같은 영역은 변화가 심하거나 여러 장소에서 공통적으로 나타나 혼란을 유발할 수 있다.43

**어텐션 메커니즘(Attention Mechanism)**은 이러한 통찰에 기반하여, 신경망이 이미지의 특정 영역에 '집중'하도록 학습하는 기법이다.44 VPR에 적용될 경우, 어텐션 메커니즘은 장소 인식에 중요한 영역에 더 높은 가중치를 부여하고, 중요하지 않거나 오해의 소지가 있는 영역의 영향력을 줄인다.

- **종류:** VPR에는 다양한 어텐션 기법이 활용된다. **공간적 어텐션(spatial attention)**은 특징 맵의 어느 '위치'가 중요한지를 학습하고, **채널 어텐션(channel attention)**은 어떤 '특징' 채널이 더 유용한지를 학습한다. 최근에는 트랜스포머 아키텍처의 핵심인 **자기 어텐션(self-attention)**을 활용하여 이미지 내 모든 위치 간의 관계를 종합적으로 고려함으로써 전역적인 맥락을 파악하는 연구가 활발히 진행되고 있다.44

어텐션의 도입으로 VPR 모델은 단순히 특징을 집계하는 것을 넘어, 이미지 내에서 '어디를' 그리고 '무엇을' 중점적으로 봐야 할지를 능동적으로 학습하게 되었고, 이는 변화하는 환경에서의 강인성을 한층 더 높이는 중요한 진전이 되었다.


단일 이미지 기반의 딥러닝 VPR 기법들이 큰 성공을 거두었지만, 여전히 극복해야 할 한계점들이 존재했다. 단일 이미지가 가진 정보의 불완전성을 인정하고 이를 보완하려는 시도에서 다양한 고급 VPR 기법들이 등장했다. 이러한 기법들은 시간적, 외형적, 감각적 차원에서 정보를 보강하여 단일 시각 정보의 한계를 극복하고자 한다.


단일 이미지는 지각적 모호성에 취약할 수 있다. 예를 들어, 특정 복도의 한 장면만으로는 다른 복도와 구분하기 어려울 수 있다. 하지만 연속된 여러 프레임을 함께 보면, 복도를 따라 이동하면서 나타나는 문이나 창문의 배열과 같은 시간적, 공간적 맥락을 통해 장소를 명확히 구분할 수 있다. 이러한 아이디어에 기반한 것이 시퀀스 기반 VPR이다.5

- **SeqSLAM의 원리:** SeqSLAM은 이 분야의 선구적인 연구로, 개별 이미지의 유사도에만 의존하지 않고 이미지 시퀀스 전체의 유사도 패턴을 비교한다.5 질의 시퀀스(예: 최근 10개 프레임)와 데이터베이스의 모든 가능한 시퀀스 간의 유사도 행렬을 계산한다. 만약 로봇이 이전에 방문했던 경로를 따라가고 있다면, 이 유사도 행렬에는 밝은 대각선 형태의 경로가 나타날 것이다. SeqSLAM은 이러한 가장 일관된 대각선 경로를 찾는 방식으로 동작하여, 개별 이미지의 매칭 실패에도 불구하고 전체 시퀀스로는 올바른 장소를 찾아낼 수 있다.48 다만, 이 방식은 차량이나 로봇의 이동 속도가 일정하다는 가정을 전제로 하므로, 속도 변화가 심한 경우 성능이 저하될 수 있다.49
- **최신 동향:** 최근 연구들은 SeqSLAM의 아이디어를 발전시켜, CNN으로 추출한 단일 프레임 디스크립터들을 순환 신경망(RNN)이나 트랜스포머를 사용하여 하나의 **시퀀스 디스크립터**로 집계하는 방식을 제안한다.5 또는 두 시퀀스 간의 매칭 점수 자체를 학습 가능한 메트릭으로 정의하기도 한다. 이러한 접근법들은 시퀀스 정보를 보다 유연하고 강인하게 활용하여 VPR 성능을 향상시킨다.48


주간 이미지로 구축된 데이터베이스에 야간 질의 이미지를 매칭하는 것은 VPR의 가장 어려운 문제 중 하나이다. 이는 두 이미지의 '도메인(domain)'이 다르기 때문이다. **도메인 적응(domain adaptation)**은 이러한 도메인 간의 격차를 줄이는 것을 목표로 하며, 생성적 적대 신경망(Generative Adversarial Networks, GANs)이 강력한 도구로 활용된다.

- **CycleGAN의 원리:** CycleGAN은 쌍으로 된 학습 데이터(paired data), 즉 동일한 장면을 주간과 야간에 정확히 같은 위치와 각도에서 촬영한 이미지가 없이도 도메인 간 이미지 변환을 학습할 수 있는 획기적인 모델이다.50
  - CycleGAN은 두 개의 생성자(Generator $G_XY$: 도메인 X → Y, $G_YX$: 도메인 Y → X)와 두 개의 판별자(Discriminator $D_X$, $D_Y$)로 구성된다.
  - **적대적 손실(Adversarial Loss):** 생성자 $G_XY$는 X 도메인의 이미지를 입력받아 Y 도메인 스타일의 이미지를 생성하고, 판별자 $D_Y$는 이 생성된 이미지가 실제 Y 도메인의 이미지와 구별할 수 없도록 학습한다.
  - **순환 일관성 손실(Cycle-Consistency Loss):** CycleGAN의 핵심 아이디어로, X 도메인의 이미지를 Y 도메인으로 변환($G_XY(x)$)했다가 다시 X 도메인으로 복원($G_YX(G_XY(x))$)했을 때, 원래 이미지 $x$와 최대한 유사해야 한다는 제약 조건을 추가한다. 즉, $G_YX(G_XY(x)) ≈ x$가 성립해야 한다. 이 손실 함수는 이미지의 내용과 구조를 보존하면서 스타일만 변환하도록 유도하는 강력한 정규화(regularization) 역할을 한다.50
- **VPR에의 응용:** VPR 파이프라인의 전처리 단계에서 CycleGAN을 사용할 수 있다. 예를 들어, 야간에 촬영된 질의 이미지를 학습된 생성자를 통해 '가상의 주간 이미지'로 변환한다. 이렇게 변환된 이미지는 주간 이미지로 구축된 데이터베이스와 동일한 도메인에 속하게 되므로, 기존의 VPR 기법을 사용하여 훨씬 더 높은 정확도로 매칭을 수행할 수 있다.52


카메라는 풍부한 색상과 텍스처 정보를 제공하지만 조명, 날씨 등 외형 변화에 매우 민감하다. 반면, LiDAR(Light Detection and Ranging) 센서는 레이저를 사용하여 주변 환경의 정밀한 3차원 구조(포인트 클라우드) 정보를 얻으며, 이러한 외형 변화에 상대적으로 덜 민감하다. 이 두 센서의 상보적인 특성을 활용하는 **다중 모달 퓨전(Multi-modal Fusion)**은 VPR의 강인성을 획기적으로 향상시킬 수 있는 잠재력을 가진다.55

- **퓨전 방식:**
  - **초기 퓨전 (Early Fusion):** 센서의 원본 데이터(raw data) 수준에서 융합하는 방식이다. 예를 들어, 3D LiDAR 포인트 클라우드의 각 점에 해당하는 2D 이미지의 픽셀 값을 찾아 색상 정보를 입히는(decorate) 방식이 있다.55
  - **후기/심층 퓨전 (Late/Deep Fusion):** 각 센서의 데이터를 별도의 신경망에서 독립적으로 처리하여 고수준의 심층 특징(deep feature)을 추출한 후, 이 특징들을 융합하는 방식이다. 예를 들어, DeepFusion과 같은 연구에서는 이미지 특징과 LiDAR 특징 간의 상관관계를 동적으로 학습하기 위해 교차 어텐션(cross-attention) 메커니즘을 사용하여 두 모달리티의 특징을 효과적으로 정렬하고 융합하는 방법을 제시했다.55
- **효과:** 다중 모달 퓨전은 특히 악천후나 야간과 같이 단일 시각 센서의 정보가 심하게 손상되는 극한 상황에서 시스템의 안정성과 신뢰성을 크게 향상시킨다. LiDAR가 제공하는 안정적인 구조 정보가 카메라의 불확실한 시각 정보를 보완해주기 때문이다.57


최근 VPR 연구는 컴퓨터 비전 분야의 가장 큰 변화인 트랜스포머(Transformer)와 파운데이션 모델(Foundation Model)의 영향을 받고 있다.

- **트랜스포머:** 자연어 처리에서 시작된 트랜스포머 아키텍처는 **자기 어텐션(self-attention)** 메커니즘을 통해 이미지의 모든 픽셀(또는 패치) 간의 관계를 계산하여 전역적인 맥락과 장거리 의존성(long-range dependency)을 효과적으로 포착한다. 이는 CNN의 지역적인 수용 영역(receptive field)의 한계를 극복하고, 이미지 전체의 구조적 관계를 이해하는 데 도움을 주어 VPR 성능을 향상시킨다.45
- **파운데이션 모델:** DINOv2와 같이 웹 규모의 방대한 데이터로 사전 훈련된 파운데이션 모델은 특정 작업에 대한 미세조정(fine-tuning) 없이도 매우 강력하고 일반화된 특징 표현을 제공한다.61 이러한 모델을 VPR에 적용한 연구들은 기존의 VPR 전용 데이터셋으로 학습한 모델들을 능가하는 성능을 보여주었으며, 이는 특정 환경에 과적합되지 않는 **'범용(Universal)' VPR 시스템**의 개발 가능성을 시사한다.63

이러한 고급 기법들은 VPR 연구가 단일 이미지의 표현을 정교화하는 단계를 넘어, 시간, 외형, 감각 등 다차원적인 정보를 통합하여 실제 세계의 복잡성에 대응하는 방향으로 진화하고 있음을 보여준다.


새로운 VPR 알고리즘의 성능을 객관적으로 평가하고 기존 연구와 공정하게 비교하기 위해서는 표준화된 성능 평가 지표와 벤치마크 데이터셋의 사용이 필수적이다.


VPR 성능 평가에서 가장 널리 사용되는 표준 지표는 **Recall@N (R@N)**이다.64

- **정의:** Recall@N은 전체 질의 이미지 중에서, 검색된 상위 N개의 후보 이미지 중 하나 이상이 정답(ground truth) 위치의 특정 거리 임계값(예: 25m) 내에 포함되는 질의의 비율을 의미한다. 즉, 시스템이 상위 N개의 후보 안에 정답을 포함시키는 데 얼마나 성공적인지를 나타내는 척도이다.
- **해석:**
  - **Recall@1 (R@1):** 가장 유사도가 높다고 판단된 첫 번째 후보가 정답인 비율이다. 이는 시스템의 '최고 정확도'를 나타낸다.
  - **Recall@5 (R@5), Recall@10 (R@10):** 상위 5개, 10개 후보 중 정답이 포함될 비율을 나타낸다. N이 커질수록 값은 일반적으로 높아지며, 이는 후속 처리 단계(예: 기하학적 검증)에서 사용할 후보군을 얼마나 잘 제공하는지를 평가하는 데 사용된다.65
- **기타 지표:** Recall@N과 함께, 정밀도-재현율 곡선(Precision-Recall Curve)과 그 곡선 아래의 면적인 AUC(Area Under the Curve) 또한 시스템의 전반적인 성능을 평가하는 데 중요한 지표로 사용된다.11


변화하는 조건에서의 VPR 성능을 엄격하게 평가하기 위해, 연구 커뮤니티는 다양한 실제 환경의 어려움을 담고 있는 여러 벤치마크 데이터셋을 구축해왔다. 각 데이터셋은 특정한 VPR 난제에 초점을 맞추고 있어, 알고리즘의 강점과 약점을 다각도로 분석하는 데 사용된다. 연구자들은 자신의 알고리즘이 해결하고자 하는 특정 VPR 난제에 가장 적합한 데이터셋을 선택하여 성능을 검증해야 한다. 예를 들어, 계절 변화에 대한 강인성을 테스트하려는 연구는 Nordland를, 도심에서의 주야간 및 시점 변화에 대한 성능을 검증하려는 연구는 Tokyo 24/7이나 Oxford RobotCar를 선택하는 것이 적절하다. 이러한 데이터셋의 특징을 명확히 이해하고 선택하는 것은 VPR 연구의 재현성과 공정한 평가를 위해 필수적이다.


| 데이터셋                         | 환경                  | 주요 난제                                                    | 규모 (이미지 수)                  | 비고                                               |
| -------------------------------- | --------------------- | ------------------------------------------------------------ | --------------------------------- | -------------------------------------------------- |
| **Nordland** 67                  | 자연/시골 (기차 경로) | 극심한 **계절 변화** (봄, 여름, 가을, 겨울)                  | 각 계절별 약 3,450장              | 시점 변화가 거의 없어 순수한 외형 변화 연구에 적합 |
| **Oxford RobotCar** 18           | 도심                  | **날씨, 조명(주간/야간), 동적 객체(교통), 장기적 변화(공사)** | 2,000만 장 이상 (1000km 주행)     | 다중 모달 센서(LiDAR, GPS/INS) 데이터 포함         |
| **Pittsburgh 30k (Pitts30k)** 67 | 도심                  | 상당한 **시점 변화**, 반복적인 구조물                        | 약 30,000장 (DB) + 6,816장 (쿼리) | 대규모 VPR 및 시점 불변성 연구의 표준 벤치마크     |
| **Tokyo 24/7** 67                | 도심                  | 극심한 **주간/야간 변화**, 강한 시점 변화                    | 약 76,000장 (DB) + 315장 (쿼리)   | 조명 변화에 대한 강인성 평가에 특화됨              |

이러한 벤치마크 데이터셋의 발전사는 VPR 연구 커뮤니티가 문제의 복잡성을 점차 깊이 이해해왔음을 보여주는 증거이다. Nordland 데이터셋이 계절 변화라는 단일 요인에 집중하여 문제의 영향을 분리 분석하려는 초기 시도였다면 74, Tokyo 24/7과 Pitts30k는 각각 조명과 시점이라는 또 다른 핵심 난제에 초점을 맞추었다.71 여기서 한 걸음 더 나아가 Oxford RobotCar 데이터셋은 1년 동안 동일 경로를 반복 주행하며 계절, 날씨, 조명, 교통, 심지어 공사로 인한 구조적 변화까지 모든 요인이 복합적으로 작용하는 '장기 자율성(long-term autonomy)'이라는 실제 문제에 근접했다.18 최근에는 Mapillary Street-Level Sequences (MSLS)와 같이 수년에 걸친 데이터를 포함하여 '생애주기(lifelong)' VPR이라는 개념을 도입한 데이터셋도 등장했다.68 이처럼 데이터셋의 진화 과정은 VPR 연구의 목표가 '특정 변화에 강인한 알고리즘' 개발에서 '예측 불가능한 실제 환경에서 지속적으로 동작하는 시스템' 구축으로 상향 조정되었음을 명확히 보여준다.


VPR 기술은 상당한 발전을 이루었지만, 실제 세계의 복잡하고 예측 불가능한 환경에서 인간 수준의 강인성과 신뢰성을 갖추기까지는 여전히 해결해야 할 과제들이 남아있다. 미래 VPR 연구는 기술을 실험실 환경에서 벗어나 실제 사회 시스템으로 통합하기 위한 필수 조건들을 해결하는 방향으로 나아가고 있다.


현재의 VPR 시스템은 대부분 고정된 데이터베이스를 기반으로 동작한다. 하지만 실제 환경은 끊임없이 변화한다. **생애주기 VPR (Lifelong VPR)**은 로봇이나 자율주행차가 몇 주, 몇 달, 몇 년에 걸쳐 장기간 운용될 때 마주하는 지속적인 환경 변화에 스스로 적응하는 능력을 목표로 한다.6

이는 단순히 외형 변화에 강인한 표현을 학습하는 것을 넘어, 다음과 같은 능력을 요구한다:

- **지속적인 학습 (Continual Learning):** 새로운 장소를 지도에 추가하고, 변화된 장소의 정보를 업데이트하며, 더 이상 유효하지 않은 오래된 정보를 점진적으로 잊는 능력이 필요하다.75 이는 새로운 지식을 학습할 때 기존 지식을 잊어버리는 '치명적 망각(catastrophic forgetting)' 문제를 해결해야 함을 의미한다.
- **변화 모델링 및 예측:** 환경 변화를 시간의 함수로 명시적으로 모델링하여, 특정 시간(예: 내일 아침)에 장소가 어떻게 보일지를 예측하고 이를 위치 추정에 활용하려는 연구도 진행되고 있다.77
- **적응성:** 장기 자율성은 시스템이 사전에 정의된 가정들이 더 이상 유효하지 않은 상황에서도 지속적으로 동작할 수 있는 적응 능력을 필요로 한다.78

생애주기 VPR은 VPR 시스템의 '지속성'을 확보하기 위한 핵심 과제로, 진정한 의미의 장기 자율성을 실현하기 위한 필수적인 연구 방향이다.80


현대의 딥러닝 기반 VPR 모델들은 높은 성능을 보이지만, 그 결정 과정이 복잡하여 인간이 이해하기 어려운 '블랙박스(black-box)' 모델인 경우가 많다.81 자율 주행과 같이 인간의 안전과 직결되는 응용 분야에서, 시스템이 왜 특정 장소를 매칭했는지, 혹은 왜 매칭에 실패했는지에 대한 이유를 설명할 수 없다면 그 시스템을 신뢰하고 도입하기 어렵다.

**설명가능 AI (Explainable AI, XAI)**는 이러한 문제를 해결하기 위해 모델의 결정을 인간이 이해할 수 있는 형태로 설명하는 기술을 연구하는 분야이다.81 VPR에 XAI를 결합하면 다음과 같은 이점을 얻을 수 있다.

- **신뢰성 및 투명성 확보:** 시스템의 판단 근거를 제시함으로써 사용자와 규제 기관의 신뢰를 얻을 수 있다.83 예를 들어, 위치 인식 실패로 인한 사고 발생 시, 그 원인을 파악하고 책임을 규명하는 데 중요한 단서를 제공할 수 있다.
- **모델 디버깅 및 개선:** Saliency Map이나 CAM(Class Activation Mapping)과 같은 시각화 기법을 통해 모델이 이미지의 어떤 영역(예: 특정 건물의 간판)에 집중하여 결정을 내렸는지 확인할 수 있다.83 만약 모델이 변화가 심한 동적 객체(예: 버스 광고판)에 집중하고 있다면, 이를 파악하고 모델을 개선하는 데 활용할 수 있다.

XAI는 VPR 기술의 '신뢰성'과 '투명성'을 확보하기 위한 필수적인 요소로, 기술의 사회적 수용성을 높이는 데 결정적인 역할을 할 것이다.


VPR 시스템이 실제 로봇이나 차량에 탑재되어 동작하기 위해서는 제한된 계산 자원 내에서 실시간으로 대규모 데이터베이스를 검색할 수 있어야 한다. 이는 '실용성'의 문제로, 알고리즘의 정확도만큼이나 중요한 연구 주제이다.

- **효율적인 검색:** 수백만 장 이상의 이미지를 포함하는 데이터베이스에서 모든 디스크립터를 순차적으로 비교하는 것은 비효율적이다. 따라서 Faiss와 같은 라이브러리를 활용한 **근사 최근접 이웃(Approximate Nearest Neighbor, ANN)** 검색 알고리즘을 사용하여 속도와 정확도 사이의 균형을 맞추는 것이 일반적이다.
- **모델 경량화:** 디스크립터의 차원을 줄이거나(차원 축소), 네트워크의 파라미터 수를 줄이는(모델 경량화) 연구가 필요하다. 또한, 모델 **양자화(Quantization)**를 통해 가중치의 정밀도를 낮춰 연산 속도를 높이고 메모리 사용량을 줄이는 기법도 활발히 연구되고 있다.30

이 세 가지 미래 연구 방향—생애주기, 설명가능성, 효율성—은 VPR 기술이 순수한 알고리즘 개발 단계를 넘어, 실제 사회 인프라의 일부로 통합되기 위해 반드시 해결해야 할 과제들을 보여준다. 이는 VPR 연구가 시스템 공학, 안전, 그리고 사회적 수용성의 문제까지 포괄하는 종합적인 분야로 확장되고 있음을 시사한다.


본 튜토리얼은 변화하는 환경 속에서 강인한 Visual Place Recognition (VPR)을 달성하기 위한 여정을 포괄적으로 탐색했다. VPR의 기본 정의와 SLAM, 자율 주행에서의 핵심적인 역할에서 시작하여, 조명, 계절, 시점 변화와 같은 실제 환경이 제기하는 근본적인 난제들을 심층적으로 분석했다.

전통적인 수동 특징점(SIFT/SURF)과 Bag-of-Visual-Words 기반 접근법의 한계를 살펴봄으로써, VPR 문제가 단순한 기하학적 불변성을 넘어 극심한 외형 변화에 대한 강인한 표현 학습을 요구함을 확인했다. 이는 딥러닝 패러다임으로의 전환을 필연적으로 이끌었다. NetVLAD와 같은 획기적인 아키텍처는 종단간 학습과 약지도 학습 프레임워크를 통해 대규모 실제 데이터를 활용하는 길을 열었으며, GeM 풀링과 어텐션 메커니즘은 효율성과 표현력을 더욱 향상시켰다.

더 나아가, 단일 이미지의 정보적 한계를 극복하기 위한 고급 기법들을 다루었다. 시퀀스 기반 매칭은 시간적 맥락을, CycleGAN을 이용한 도메인 적응은 외형적 격차를, 그리고 LiDAR-카메라 퓨전은 감각적 다각화를 통해 VPR의 강인성을 새로운 차원으로 끌어올렸다. 표준화된 벤치마크 데이터셋과 평가 지표는 이러한 기술들의 발전을 객관적으로 측정하고 촉진하는 중요한 역할을 해왔다.

현재 VPR 기술은 트랜스포머와 대규모 파운데이션 모델의 등장으로 또 한 번의 중요한 전환점을 맞이하고 있다. 이러한 모델들은 전례 없는 일반화 성능을 보여주며 '범용 VPR'의 가능성을 열고 있다. 앞으로의 VPR 연구는 단순히 인식 정확도를 높이는 것을 넘어, 장기 자율성을 위한 생애주기 학습(Lifelong VPR), 시스템의 신뢰성을 보장하기 위한 설명가능 AI(XAI)의 통합, 그리고 실제 시스템에 탑재 가능한 효율성 최적화라는 세 가지 핵심 축을 중심으로 발전해 나갈 것이다. VPR은 계속해서 자율 시스템이 변화하는 세상을 이해하고 상호작용하는 데 있어 가장 근본적인 인지 능력 중 하나로 자리매김할 것으로 전망된다.


1. Visual Place Recognition: A Tutorial - SciSpace, 8월 18, 2025에 액세스, https://scispace.com/pdf/visual-place-recognition-a-tutorial-32l64wgd.pdf
2. [2303.03281] Visual Place Recognition: A Tutorial - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2303.03281
3. (PDF) Visual Place Recognition: A Tutorial - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/374128870_Visual_Place_Recognition_A_Tutorial
4. SVS-VPR: A Semantic Visual and Spatial Information-Based Hierarchical Visual Place Recognition for Autonomous Navigation in Challenging Environmental Conditions - PMC, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10857550/
5. Visual Place Recognition. Introduction | by SOMIK DHAR - Medium, 8월 18, 2025에 액세스, https://medium.com/@sd5023/visual-place-recognition-8999307ebb2f
6. (PDF) Visual Place Recognition: A Survey - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/284754832_Visual_Place_Recognition_A_Survey
7. MubarizZaffar/VPR_Tutorial: A basic tutorial (theory and practicals) for Visual Place Recognition. - GitHub, 8월 18, 2025에 액세스, https://github.com/MubarizZaffar/VPR_Tutorial
8. Visual Place Recognition for Autonomous Mobile Robots - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2218-6581/6/2/9
9. LiDAR-Based Place Recognition For Autonomous Driving: A Survey - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2306.10561v3
10. NYU-VPR: Long-Term Visual Place Recognition Benchmark with View Direction and Data Anonymization Influences, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9394449/
11. Challenges of the Visual Place Recognition in Changing Environments... - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/Challenges-of-the-Visual-Place-Recognition-in-Changing-Environments-Dataset-Query_fig3_278027950
12. Visual Place Recognition in Changing Environments - Niko Sünderhauf, 8월 18, 2025에 액세스, https://nikosuenderhauf.github.io/projects/placerecognition/
13. Long-term Visual Place Recognition Under Varying Conditions - Trepo, 8월 18, 2025에 액세스, https://trepo.tuni.fi/handle/10024/151007
14. Visual Place Recognition in Changing Environments - bonndoc, 8월 18, 2025에 액세스, https://bonndoc.ulb.uni-bonn.de/xmlui/handle/20.500.11811/8010
15. (PDF) Effects of weather conditions, light conditions, and road lighting on vehicle speed, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/301609586_Effects_of_weather_conditions_light_conditions_and_road_lighting_on_vehicle_speed
16. Visual Place Recognition in Changing Environments with Sequence Representations on the Distance-Space Domain - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2075-1702/11/5/558
17. Long-term Visual Place Recognition, 8월 18, 2025에 액세스, https://webpages.tuni.fi/vision/public_data/publications/icpr2022_longterm_visual_place_recognition.pdf
18. Oxford RobotCar Dataset, 8월 18, 2025에 액세스, https://robotcar-dataset.robots.ox.ac.uk/
19. A Novel Image Retrieval Based on Visual Words Integration of SIFT and SURF | PLOS One, 8월 18, 2025에 액세스, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0157428
20. Evaluation of SIFT and SURF using Bag of Words Model on a Very Large Dataset, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/261047996_Evaluation_of_SIFT_and_SURF_using_Bag_of_Words_Model_on_a_Very_Large_Dataset
21. A Comparison of SIFT, SURF and ORB on OpenCV | by Mikhail Kennerley - Medium, 8월 18, 2025에 액세스, https://mikhail-kennerley.medium.com/a-comparison-of-sift-surf-and-orb-on-opencv-59119b9ec3d0
22. OptiCorNet: Optimizing Sequence-Based Context Correlation for Visual Place Recognition, 8월 18, 2025에 액세스, https://arxiv.org/html/2507.14477v1
23. DINO-Mix enhancing visual place recognition with foundational vision model and feature mixing - PMC, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11437288/
24. Graph-Based Place Recognition in Image Sequences with CNN Features - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/327037021_Graph-Based_Place_Recognition_in_Image_Sequences_with_CNN_Features
25. Bag-of-words model in computer vision - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Bag-of-words_model_in_computer_vision
26. Bag of Words Models for visual categorization - Gil's CV blog, 8월 18, 2025에 액세스, https://gilscvblog.com/2013/08/23/bag-of-words-models-for-visual-categorization/
27. Bag of Features: Simplifying Image Recognition for Non-Experts - - Analytics Vidhya, 8월 18, 2025에 액세스, https://www.analyticsvidhya.com/blog/2023/01/bag-of-features-simplifying-image-recognition-for-non-experts/
28. 69 - Image classification using Bag of Visual Words (BOVW) - YouTube, 8월 18, 2025에 액세스, https://www.youtube.com/watch?v=PRceoMWcv1U
29. Bag of Visual Words | Pinecone, 8월 18, 2025에 액세스, https://www.pinecone.io/learn/series/image-search/bag-of-visual-words/
30. Bag of visual words | Computer Vision and Image Processing Class Notes - Fiveable, 8월 18, 2025에 액세스, https://library.fiveable.me/computer-vision-and-image-processing/unit-5/bag-visual-words/study-guide/gKqlaeA7CzYBBncs
31. What is bag of words? | IBM, 8월 18, 2025에 액세스, https://www.ibm.com/think/topics/bag-of-words
32. NetVLAD: CNN architecture for weakly supervised place recognition - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/284578950_NetVLAD_CNN_architecture_for_weakly_supervised_place_recognition
33. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content_cvpr_2016/papers/Arandjelovic_NetVLAD_CNN_Architecture_CVPR_2016_paper.pdf
34. NetVLAD: CNN architecture for weakly supervised place recognition, 8월 18, 2025에 액세스, https://www.di.ens.fr/willow/research/netvlad/
35. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - Medium, 8월 18, 2025에 액세스, https://medium.com/data-science/netvlad-cnn-architecture-for-weakly-supervised-place-recognition-ce64b08bebaf
36. Triplet Loss: Intro, Implementation, Use Cases - V7 Labs, 8월 18, 2025에 액세스, https://www.v7labs.com/blog/triplet-loss
37. Introduction to Triplet Loss | Baeldung on Computer Science, 8월 18, 2025에 액세스, https://www.baeldung.com/cs/triplet-loss
38. NetVLAD: CNN architecture for weakly supervised place recognition - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/311611050_NetVLAD_CNN_architecture_for_weakly_supervised_place_recognition
39. Fine-tuning CNN Image Retrieval with No Human Annotation - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/1711.02512
40. Enhanced VPR with Miners, Losses, and Aggregators using GSV-Cities - Ali Al Housseini, 8월 18, 2025에 액세스, https://www.alialhousseini.com/media/pub/2024/09/15/Enhanced_VPR_with_Miners__Losses__and_Aggregators_using_GSV_Cities.pdf
41. [Literature Review] SuperPlace: The Renaissance of Classical Feature Aggregation for Visual Place Recognition in the Era of Foundation Models - Moonlight, 8월 18, 2025에 액세스, https://www.themoonlight.io/en/review/superplace-the-renaissance-of-classical-feature-aggregation-for-visual-place-recognition-in-the-era-of-foundation-models
42. SuperPlace: The Renaissance of Classical Feature Aggregation for Visual Place Recognition in the Era of Foundation Models - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2506.13073v1
43. Learning Context Flexible Attention Model for Long-Term Visual Place Recognition - Research Collection, 8월 18, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/20.500.11850/282829/4/submitted_final_pdfchecked_Learning_Context_Flexible_Attention_Model_for_Long_term_Visual_Place_Recognition.pdf
44. Attention Mechanisms for Computer Vision - GeeksforGeeks, 8월 18, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/attention-mechanisms-for-computer-vision/
45. Enhancing Visual Place Recognition With Hybrid Attention Mechanisms in MixVPR, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/385330088_Enhancing_Visual_Place_Recognition_with_Hybrid_Attention_Mechanisms_in_MixVPR
46. Visual place recognition from end-to-end semantic scene text features - Frontiers, 8월 18, 2025에 액세스, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2024.1424883/full
47. Improving Visual Place Recognition with Sequence-Matching Receptiveness Prediction, 8월 18, 2025에 액세스, https://arxiv.org/html/2503.06840v1
48. Sequence-Based Filtering for Visual Route-Based Navigation: Analyzing the Benefits, Trade-Offs and Design Choices - ePrints Soton, 8월 18, 2025에 액세스, https://eprints.soton.ac.uk/473475/1/Sequence_Based_Filtering_for_Visual_Route_Based_Navigation_Analyzing_the_Benefits_Trade_Offs_and_Design_Choices.pdf
49. CaseVPR: Correlation-Aware Sequential Embedding for Sequence-to-Frame Visual Place Recognition | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/388971148_CaseVPR_Correlation-Aware_Sequential_Embedding_for_Sequence-to-Frame_Visual_Place_Recognition
50. How CycleGAN Works? | ArcGIS API for Python - Esri Developer, 8월 18, 2025에 액세스, https://developers.arcgis.com/python/latest/guide/how-cyclegan-works/
51. Unpaired Image-To-Image Translation Using Cycle-Consistent Adversarial Networks - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content_ICCV_2017/papers/Zhu_Unpaired_Image-To-Image_Translation_ICCV_2017_paper.pdf
52. CycleGAN Project Page, 8월 18, 2025에 액세스, https://junyanz.github.io/CycleGAN/
53. Image to image Translation | Cycle GAN - Kaggle, 8월 18, 2025에 액세스, https://www.kaggle.com/code/utkarshsaxenadn/image-to-image-translation-cycle-gan
54. Day-Night Image Translations using CycleGAN - Kaggle, 8월 18, 2025에 액세스, https://www.kaggle.com/code/virajkadam/day-night-image-translations-using-cyclegan
55. DeepFusion: Lidar-Camera Deep Fusion for Multi-Modal 3D Object Detection - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Li_DeepFusion_Lidar-Camera_Deep_Fusion_for_Multi-Modal_3D_Object_Detection_CVPR_2022_paper.pdf
56. Lidar-Camera Deep Fusion for Multi-Modal 3D Detection - Google Research, 8월 18, 2025에 액세스, https://research.google/blog/lidar-camera-deep-fusion-for-multi-modal-3d-detection/
57. Deep LiDAR-Radar-Visual Fusion for Object Detection in Urban Environments - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2072-4292/15/18/4433
58. [2203.08195] DeepFusion: Lidar-Camera Deep Fusion for Multi-Modal 3D Object Detection - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2203.08195
59. LCPR: A Multi-Scale Attention-Based LiDAR-Camera Fusion Network for Place Recognition | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/376825839_LCPR_A_Multi-Scale_Attention-Based_LiDAR-Camera_Fusion_Network_for_Place_Recognition
60. [2505.14068] Place Recognition Meet Multiple Modalitie: A Comprehensive Review, Current Challenges and Future Directions - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2505.14068
61. TeTRA-VPR: A Ternary Transformer Approach for Compact Visual Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2503.02511v1
62. EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2405.18065v2
63. Towards Universal Visual Place Recognition - AnyLoc, 8월 18, 2025에 액세스, https://anyloc.github.io/assets/AnyLoc.pdf
64. The RecallRate@N curves for all 10 VPR techniques generated on the 12... - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/The-RecallRateN-curves-for-all-10-VPR-techniques-generated-on-the-12-datasets-by_fig3_351408419
65. Recall as a function of the number of candidates N in the image filtering stage. - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/Recall-as-a-function-of-the-number-of-candidates-N-in-the-image-filtering-stage_fig5_336621495
66. Revisit Anything: Visual Place Recognition via Image Segment Retrieval - European Computer Vision Association, 8월 18, 2025에 액세스, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/08592.pdf
67. Automatic download VPR datasets in a standard format - GitHub, 8월 18, 2025에 액세스, https://github.com/gmberton/VPR-datasets-downloader
68. General place recognition performance for the different season... - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/General-place-recognition-performance-for-the-different-season-pairings-using-sequence_fig2_283623386
69. Downloads - Oxford RobotCar Dataset, 8월 18, 2025에 액세스, https://robotcar-dataset.robots.ox.ac.uk/downloads/
70. 1 year, 1000 km: The Oxford RobotCar dataset | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/311240888_1_year_1000_km_The_Oxford_RobotCar_dataset
71. Visual Place Recognition with Repetitive Structures | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/261479452_Visual_Place_Recognition_with_Repetitive_Structures
72. 24/7 Place Recognition by View Synthesis, 8월 18, 2025에 액세스, http://www.ok.ctrl.titech.ac.jp/~torii/project/247/
73. Unified Depth-Guided Feature Fusion and Reranking for Hierarchical Place Recognition, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC12252300/
74. Place recognition across seasons on the Nordland dataset. Despite the... | Download Scientific Diagram - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/Place-recognition-across-seasons-on-the-Nordland-dataset-Despite-the-extreme-appearance_fig5_271140542
75. [2009.09929] CVPR 2020 Continual Learning in Computer Vision Competition: Approaches, Results, Current Challenges and Future Directions - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2009.09929
76. Advancing autonomy through lifelong learning: a survey of autonomous intelligent systems, 8월 18, 2025에 액세스, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2024.1385778/full
77. Long-term autonomy of Mobile Robots in Changing Environments - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/324124232_Long-term_autonomy_of_Mobile_Robots_in_Changing_Environments
78. Robot Adaptation to Environment Changes in Long-Term Autonomy - ScalAR Lab, 8월 18, 2025에 액세스, https://scalar.seas.upenn.edu/wp-content/uploads/2020/06/ICRAW18_Adaptation.pdf
79. Challenges in Achieving Long-term Autonomy - Notre Dame Sites, 8월 18, 2025에 액세스, https://sites.nd.edu/autonomy-symposium/files/2018/11/Long-Term-Autonomys.pdf
80. General Place Recognition Survey: Towards Real-World Autonomy - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2405.04812v2
81. Explainable artificial intelligence - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Explainable_artificial_intelligence
82. Explainable Image Classification: The Journey So Far and the Road Ahead - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2673-2688/4/3/33
83. All you need to know about explainable AI (XAI) - Ultralytics, 8월 18, 2025에 액세스, https://www.ultralytics.com/blog/all-you-need-to-know-about-explainable-ai
84. Towards Explainable Artificial Intelligence (XAI): A Data Mining Perspective - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2401.04374v2


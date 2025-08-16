# Aerial-Ground Image Matching 기술 심층 분석: 원리, 최신 동향 및 미래 전망





## I. 서론: 하늘과 땅의 시각적 연결 - Aerial-Ground Image Matching의 서막



Aerial-Ground Image Matching 기술은 하늘의 관점에서 촬영된 항공 및 위성 이미지와 지상의 관점에서 촬영된 이미지를 정밀하게 정합하여 동일한 지점이나 객체를 식별하고 연결하는 컴퓨터 비전의 핵심 분야이다. 이 기술은 서로 다른 시점, 축척, 조명, 촬영 시간으로 인해 발생하는 극심한 외형적 차이, 즉 '도메인 갭(Domain Gap)'을 극복하는 것을 목표로 한다. 이러한 간극은 특히 항공기와 지상 카메라 간의 '초광각 베이스라인(ultra-wide baseline)' 상황에서 두드러지며, 기존의 컴퓨터 비전 기술로는 해결하기 어려운 근본적인 난제로 여겨져 왔다.1

이 기술의 중요성은 응용 분야의 확장과 함께 점차 커지고 있다. 초기에는 드론으로 촬영한 항공 사진의 왜곡을 보정하여 농경지의 정확한 면적을 계산하거나 토양 상태를 분석하는 정밀 농업과 같은 정적(static) 분석 분야에서 주로 활용되었다.3 그러나 기술이 발전함에 따라, 그 응용 범위는 실시간 동적(dynamic) 항법 시스템으로 확장되었다. 대표적으로, 위성항법시스템(GNSS) 신호가 취약하거나 두절되는 도심 협곡, 실내, 전파 방해 환경에서 도심항공모빌리티(AAM)나 자율주행차가 자신의 절대 위치를 정밀하게 추정하는 데 필수적인 보조 항법 기술로 주목받고 있다.4 또한, 출처가 불분명한 지상 이미지를 위성 이미지 데이터베이스와 대조하여 촬영 위치를 특정하는 등, 국가 안보 및 허위 정보 분석 분야에서도 그 가치가 입증되고 있다.6

이처럼 Aerial-Ground Image Matching 기술의 발전사는 단순히 더 정확한 알고리즘을 개발하는 과정을 넘어, '무엇을 위해 이 기술을 사용하는가?'라는 응용 목표의 고도화에 따라 필연적으로 이루어진 패러다임 전환의 역사라 할 수 있다. 본 보고서는 이러한 기술적 진화의 맥락 속에서 Aerial-Ground Image Matching의 기본 원리부터 고전적 접근법의 한계, 그리고 이를 극복하기 위한 딥러닝 기반의 혁신적인 방법론들을 심층적으로 분석한다. 나아가 주요 응용 사례와 현재 기술이 마주한 난제, 그리고 미래 연구 방향을 조망함으로써 이 분야에 대한 포괄적이고 깊이 있는 이해를 제공하고자 한다.



## 1. II. 기하학적 정합의 원리: 왜곡된 시각을 바로잡는 기술



Aerial-Ground Image Matching의 근간에는 서로 다른 시점에서 발생한 기하학적 왜곡을 이해하고 보정하려는 노력이 자리 잡고 있다. 이 문제를 해결하기 위한 두 가지 대표적인 접근법은 '정사보정(Orthorectification)'과 '호모그래피(Homography)'로, 이는 각각 '표준화'와 '관계 모델링'이라는 다른 철학에 기반한다. 이 두 개념의 차이를 이해하는 것은 후속 기술들의 발전 방향을 파악하는 데 중요한 단초를 제공한다.



### 1.1  정사보정 (Orthorectification): 이미지를 지도로 변환하는 기술



정사보정은 항공 사진에 필연적으로 포함되는 기하학적 왜곡을 제거하여, 모든 지점이 마치 수직 상공에서 내려다본 것처럼 보이게 만드는 과정이다.3 항공 카메라는 비스듬한 각도로 촬영하기 때문에 지형의 높낮이, 카메라의 기울어짐 등으로 인해 이미지 중심부와 주변부의 축척이 달라지고 객체가 기울어져 보이는 왜곡이 발생한다. 정사보정은 이러한 왜곡을 보정하여 "마치 지도처럼 정확한 축척과 방향을 가진 이미지", 즉 정사영상을 생성하는 것을 목표로 한다.3 이는 원본 이미지의 시점을 제거하고 모든 이미지를 '수직 상공'이라는 단일한 표준 시점으로 통일하는 

**표준화(Standardization)** 접근법에 해당한다.

정사보정의 정확도는 두 가지 핵심 데이터에 크게 의존한다. 첫째는 지형의 높낮이 정보를 담고 있는 **수치표고모델(DEM, Digital Elevation Model)**이며, 이는 지형으로 인한 왜곡을 보정하는 데 필수적이다. 둘째는 카메라 렌즈 자체의 왜곡 특성을 보정하기 위한 **카메라 캘리브레이션 데이터**이다.3

이렇게 표준화된 정사영상은 절대적인 면적이나 거리가 중요한 분야에서 강력한 도구가 된다. 예를 들어, 정밀 농업 분야에서는 정사보정된 이미지를 통해 농경지의 실제 면적을 정확히 측정하여 정부 보조금 신청 자료로 활용하거나, 명확한 시각적 증거를 통해 토지 경계 분쟁을 해결하는 데 사용된다. 또한, 다중분광 이미지와 결합하여 토양 유형을 매핑하고 유기물 함량을 추정하는 등 정밀한 공간 분석을 가능하게 한다.3



### 1.2  호모그래피 (Homography): 두 시점 간의 기하학적 관계 모델링



호모그래피는 정사보정과 달리 원본 이미지들의 시점을 그대로 유지하면서, 두 이미지 간의 기하학적 관계를 수학적으로 모델링하는 접근법이다. 동일한 평면을 서로 다른 두 위치에서 촬영했을 때, 한 이미지의 픽셀 좌표 $(u, v)$가 다른 이미지의 픽셀 좌표 $(u', v')$로 어떻게 변환되는지를 나타내는 선형 변환 관계를 호모그래피라고 한다. 이 관계는 다음과 같은 3×3 행렬, 즉 **호모그래피 행렬(Homography Matrix, H)**로 표현된다.7

![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M403 1759 V84 H666 V0 H319 V1759 v0 v1759 h347 v-84 H403z M403 1759 V0 H319 V1759 v0 v1759 h84z"></path></svg>)u′v′1![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M347 1759 V0 H0 V84 H263 V1759 v0 v1759 H0 v84 H347z M347 1759 V0 H263 V1759 v0 v1759 h84z"></path></svg>)=H![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M403 1759 V84 H666 V0 H319 V1759 v0 v1759 h347 v-84 H403z M403 1759 V0 H319 V1759 v0 v1759 h84z"></path></svg>)uv1![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M347 1759 V0 H0 V84 H263 V1759 v0 v1759 H0 v84 H347z M347 1759 V0 H263 V1759 v0 v1759 h84z"></path></svg>)=![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M403 1759 V84 H666 V0 H319 V1759 v0 v1759 h347 v-84 H403z M403 1759 V0 H319 V1759 v0 v1759 h84z"></path></svg>)h11h21h31h12h22h32h13h23h33![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M347 1759 V0 H0 V84 H263 V1759 v0 v1759 H0 v84 H347z M347 1759 V0 H263 V1759 v0 v1759 h84z"></path></svg>)![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M403 1759 V84 H666 V0 H319 V1759 v0 v1759 h347 v-84 H403z M403 1759 V0 H319 V1759 v0 v1759 h84z"></path></svg>)uv1![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.667em" height="3.600em" viewBox="0 0 667 3600"><path d="M347 1759 V0 H0 V84 H263 V1759 v0 v1759 H0 v84 H347z M347 1759 V0 H263 V1759 v0 v1759 h84z"></path></svg>)

이 호모그래피 행렬 H는 두 시점 간의 회전(rotation), 이동(translation), 스케일링(scaling), 투영(projection) 변환을 모두 포함하는 강력한 모델이다.8 따라서 호모그래피 추정(Homography Estimation)은 여러 장의 이미지를 이어 붙여 파노라마 이미지를 만드는 이미지 모자이크, 카메라의 움직임을 추적하는 단안 SLAM(Simultaneous Localization and Mapping), 3D 카메라 자세 복원 등 다양한 컴퓨터 비전 응용의 핵심적인 역할을 수행한다.7 이는 두 뷰 사이의 상대적 움직임이나 관계 자체가 중요한 응용 분야에 더 직접적인 해법을 제공하는 

**관계 모델링(Relational Modeling)** 철학에 기반한다.

이처럼 정사보정과 호모그래피는 시점 차이라는 동일한 문제를 다른 관점에서 접근하며, 이러한 철학적 차이는 딥러닝 시대의 기술 발전에도 영향을 미치고 있다. 예를 들어, 지상 이미지를 항공 이미지처럼 보이도록 '합성'하는 연구는 정사보정의 '표준화' 철학을, 두 이미지로부터 직접 변환 행렬을 추정하는 연구는 호모그래피의 '관계 모델링' 철학을 계승한 것으로 볼 수 있다.



## 2. III. 고전적 접근법: 특징점 기반 매칭의 시대



딥러닝이 등장하기 전, 이미지 매칭은 수십 년간 '특징점(feature point)'을 기반으로 한 접근법이 주를 이루었다. 이 방식은 인간이 이미지를 인식하는 방식과 유사하게, 이미지에서 눈에 띄는 고유한 지점을 찾고 그 주변의 정보를 비교하여 두 이미지 간의 대응 관계를 찾는 직관적인 아이디어에 기반한다.



### 2.1  3단계 파이프라인: 탐지, 기술, 정합



고전적인 특징점 기반 매칭은 일반적으로 세 단계의 파이프라인으로 구성된다 9:

1. **특징 추출 (Feature Extraction / Detection):** 이미지 내에서 스케일, 회전, 조명 변화에도 불구하고 안정적으로 다시 검출될 수 있는 특별한 지점, 즉 '특징점(코너, 블롭 등)'을 탐지하는 단계이다.
2. **특징 기술 (Feature Description):** 탐지된 각 특징점 주변의 지역적 픽셀 패턴을 분석하여, 해당 특징점을 고유하게 표현하는 수학적인 벡터, 즉 '기술자(descriptor)'를 생성하는 단계이다. 이 기술자는 다른 특징점과 구별되는 '지문'과 같은 역할을 한다.
3. **특징 매칭 (Feature Matching):** 첫 번째 이미지의 특징점 기술자들을 두 번째 이미지의 기술자들과 비교하여, 가장 유사한(예: 유클리드 거리가 가장 가까운) 기술자 쌍을 찾아 대응점(correspondence)을 설정하는 단계이다. 이 과정에서 잘못된 매칭(outlier)을 제거하기 위해 RANSAC(Random Sample Consensus)과 같은 강인한 추정 알고리즘이 함께 사용된다.



### 2.2  주요 알고리즘 비교 분석: SIFT, SURF, ORB



이 3단계 파이프라인을 구현하는 다양한 알고리즘이 제안되었으며, 그중 SIFT, SURF, ORB가 가장 대표적이다. 이들 알고리즘은 정확도와 계산 속도 사이의 트레이드오프 관계를 명확히 보여준다.

- **SIFT (Scale-Invariant Feature Transform):** Lowe에 의해 제안된 SIFT는 이미지의 스케일 변화와 회전에 불변하는 특징점을 추출하여 이미지 매칭 분야에 혁명을 일으켰다. 조명 변화에도 강인하여 매우 높은 정확도와 신뢰성을 자랑하지만, 기술자 생성 과정이 복잡하여 계산 비용이 매우 높다는 치명적인 단점이 있다.9 이로 인해 실시간 처리가 요구되는 응용에는 적합하지 않았다.
- **SURF (Speeded Up Robust Features):** SIFT의 높은 계산 비용 문제를 해결하기 위해 등장했다. 적분 이미지(Integral Image)와 Haar 웨이브릿 반응을 사용하여 특징 추출 및 기술 과정을 가속화함으로써 SIFT와 유사한 성능을 유지하면서도 훨씬 빠른 속도를 제공했다.9
- **ORB (Oriented FAST and Rotated BRIEF):** FAST 코너 탐지기와 BRIEF 기술자를 결합하고, 회전 불변성을 추가한 알고리즘이다. SIFT나 SURF에 비해 정확도는 다소 떨어지지만, 계산 속도가 월등히 빠르다는 장점이 있다.7 이러한 특성 덕분에 계산 자원이 제한되고 실시간 응답성이 중요한 드론(UAV) 영상 처리나 모바일 응용에서 널리 채택되었다.11

**표 1: 고전적 특징점 기반 알고리즘 비교**

| 알고리즘 | 핵심 아이디어                                                | 강인성                                    | 계산 비용 | 주요 장점                             | 주요 단점                                         |
| -------- | ------------------------------------------------------------ | ----------------------------------------- | --------- | ------------------------------------- | ------------------------------------------------- |
| **SIFT** | DoG (Difference of Gaussians) 기반 특징점 탐지, Gradient Histogram 기반 기술자 | 스케일, 회전, 조명, 시점 변화에 매우 강인 | 높음      | 최고의 정확도와 신뢰성 9              | 매우 느린 속도, 높은 계산 복잡도 9                |
| **SURF** | 적분 이미지를 이용한 Hessian 행렬 기반 탐지, Haar 웨이브릿 반응 기반 기술자 | 스케일, 회전, 조명 변화에 강인            | 중간      | SIFT와 유사한 성능에 더 빠른 속도 9   | SIFT보다 정확도 약간 저하, 특허 문제              |
| **ORB**  | FAST 코너 탐지 + BRIEF 이진 기술자                           | 회전, 제한된 스케일 변화에 강인           | 낮음      | 매우 빠른 속도, 실시간 응용에 적합 11 | SIFT/SURF 대비 낮은 정확도, 큰 시점 변화에 취약 7 |



### 2.3  고전적 접근법의 명확한 한계



이러한 고전적 방법들은 특정 조건 하에서 뛰어난 성능을 보였지만, Aerial-Ground Image Matching과 같은 극한의 환경에서는 명백한 한계를 드러냈다. 그 실패의 근본 원인은 개별 알고리즘의 성능 부족이 아니라, 패러다임 자체에 내재된 **'지역성(Locality)'**의 한계에 있다.

SIFT, SURF, ORB 모두 이미지 전체의 맥락을 고려하는 것이 아니라, 특징점 주변의 작은 픽셀 영역(local patch) 정보에만 의존하여 기술자를 생성한다.9 그러나 항공 이미지에 보이는 거대한 빌딩 옥상과 지상 이미지에 보이는 빌딩 정면의 벽돌 텍스처는 '지역적'으로 완전히 다른 패턴을 가진다. 따라서 두 뷰에서 동일한 지역적 특징을 찾는 것은 거의 불가능하다. 이 문제는 "지역적 대응점 매칭 접근법은 (초광각 베이스라인에서) 실패한다(local correspondence matching approaches fail)"는 지적으로 명확히 요약된다.2 즉, 항공-지상 매칭은 지역 정보만으로는 해결할 수 없는, 이미지 전체의 

**전역적(global)** 기하학 및 의미론적 이해를 요구하는 문제이다. 이 근본적인 한계는 결국 이미지 전체의 맥락을 학습할 수 있는 딥러닝으로의 패러다임 전환을 필연적으로 이끌었다.



## 3. IV. 딥러닝 혁명: 학습 기반 이미지 매칭의 도래



고전적 방법론이 마주한 '지역성'의 한계와 '초광각 베이스라인' 문제에 대한 해답으로 딥러닝이 등장했다. 딥러닝은 이미지 매칭의 패러다임을 근본적으로 바꾸어 놓았다.



### 3.1  패러다임의 전환: End-to-End 학습



딥러닝의 가장 큰 혁신은 고전적 방법론의 분절된 3단계 파이프라인(탐지-기술-정합)을 하나의 통합된, 종단간(End-to-End) 훈련 가능한 프레임워크로 대체한 것이다.8 이는 인간이 직접 설계한 규칙 기반의 특징(hand-crafted features)에서, 대규모 데이터로부터 모델이 스스로 최적의 특징 표현(learned representations)을 학습하는 방식으로의 전환을 의미한다.6 이 새로운 패러다임은 항공-지상 이미지 간의 극심한 '도메인 갭'을 공략하기 위한 다양한 전략의 등장을 촉발했다. 이러한 전략들은 크게 '임베딩(Embedding)', '합성(Synthesis)', '아키텍처(Architecture)'라는 세 가지 축으로 나누어 볼 수 있다.



### 3.2  주요 딥러닝 방법론 분석





#### 3.2.1  CNN 기반 샴 네트워크 (CNN-based Siamese Networks): 임베딩 전략



초기 딥러닝 접근법은 도메인 갭을 '뛰어넘을' 수 있는 강인한 특징 표현을 학습하는 데 집중했다. 이를 위한 대표적인 구조가 **샴 네트워크(Siamese Network)**이다. 샴 네트워크는 동일한 구조와 가중치를 공유하는 두 개의 컨볼루션 신경망(CNN)으로 구성된다.2 항공 이미지와 지상 이미지가 각각의 CNN을 통과하여 저차원의 특징 벡터, 즉 '임베딩(embedding)'으로 변환된다. 학습 과정에서는 같은 장소를 촬영한 이미지 쌍(positive pair)의 임베딩 벡터 간 거리는 가깝게, 서로 다른 장소를 촬영한 이미지 쌍(negative pair)의 거리는 멀어지도록 모델을 훈련시킨다. 이러한 '유사도 측정 학습(metric learning)'을 통해, 네트워크는 시점 차이에 불변하는 특징 공간을 학습하게 된다.12 CVM-Net과 같은 모델은 VGG16과 NetVLAD를 결합한 샴 네트워크 구조를 사용하여 항공-지상 이미지의 전역 기술자(global descriptor)를 학습하고, 이미지 검색 방식으로 위치를 추정했다.12



#### 3.2.2  생성적 적대 신경망 (Generative Adversarial Networks, GANs): 합성 전략



두 번째 전략은 도메인 갭을 뛰어넘는 대신, 갭 자체를 '제거'하는 것이다. **생성적 적대 신경망(GAN)**은 이러한 아이디어를 구현하는 강력한 도구다. GAN을 이용하면 한 도메인의 이미지를 다른 도메인의 스타일로 변환, 즉 '이미지 합성(image synthesis)'이 가능하다.6 예를 들어, 지상에서 촬영한 파노라마 이미지를 입력받아 해당 위치의 위성 이미지처럼 보이는 이미지를 생성할 수 있다. 이렇게 되면 어려운 '항공-지상' 매칭 문제가 상대적으로 쉬운 '항공-항공' 매칭 문제로 변환된다. 이처럼 GAN은 도메인 적응(domain adaptation)을 통해 두 이미지 간의 시각적 차이를 직접적으로 줄여주어 후속 매칭 단계의 성능을 크게 향상시킨다.14



#### 3.2.3  트랜스포머와 어텐션 (Transformers and Attention): 아키텍처 전략



세 번째 전략은 도메인 갭을 '이해하고 처리'하는 데 더 적합한 새로운 아키텍처를 도입하는 것이다. 자연어 처리 분야에서 시작된 **트랜스포머(Transformer)**는 비전 분야에서도 그 강력함을 입증했다. 트랜스포머의 핵심인 **셀프 어텐션(self-attention)** 메커니즘은 이미지 내의 모든 픽셀(또는 패치) 쌍 간의 관계를 계산하여, 이미지의 전역적인 맥락과 장거리 의존성(long-range dependencies)을 효과적으로 포착한다.6 이는 지역적 특징에만 얽매였던 CNN의 한계를 극복하고, 항공-지상 이미지 간의 복잡한 기하학적, 구조적 관계를 이해하는 데 매우 유리하다. AerialFormer는 트랜스포머 인코더를 통해 항공 이미지의 다중 스케일 특징을 추출하고, 이를 경량 CNN 디코더와 결합하여 항공 이미지 분할에서 뛰어난 성능을 보였다.15 최근에는 ETQ-Matcher와 같이 트랜스포머를 특징 매칭에 직접 적용하여 전역적 상관관계를 찾는 연구들이 활발히 진행되고 있다.16



#### 3.2.4  하이브리드 아키텍처 (Hybrid Architectures)



최신 연구들은 이러한 전략들을 융합하는 방향으로 나아가고 있다. 대표적으로 CNN의 강력한 지역 특징 추출 능력과 트랜스포머의 전역적 관계 이해 능력을 결합한 **하이브리드 아키텍처**가 있다. 일반적으로 CNN을 모델의 초반부에 사용하여 고품질의 지역 특징 맵을 생성하고, 트랜스포머가 이 특징 맵을 입력받아 전역적인 매칭을 수행하는 구조를 가진다.6 이는 각 아키텍처의 장점을 극대화하는 효과적인 접근법으로 평가받는다.

**표 2: 딥러닝 기반 항공-지상 매칭 방법론 분류**

| 대분류               | 모델 예시       | 핵심 아키텍처                          | 주요 아이디어/기여                                           | 해결하려는 문제                               |
| -------------------- | --------------- | -------------------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| **샴 네트워크 기반** | CVM-Net 12      | VGG16 + NetVLAD                        | 샴 구조를 이용한 전역 기술자 학습, 가중 소프트 마진 랭킹 손실 함수 | 메트릭 학습을 통한 이미지 검색 기반 위치 추정 |
|                      | SAFA 17         | ResNet + 극좌표 변환 + 공간 어텐션     | 극좌표 변환으로 기하학적 불일치 완화, 공간 인식 특징 집계(SAFA) | 시점 차이로 인한 기하학적, 외형적 불일치 해소 |
| **GAN 기반**         | Regmi et al. 13 | Conditional GAN + 샴 네트워크          | 지상 이미지를 항공 이미지로 합성하여 도메인 갭 축소, 특징 융합 | 극심한 도메인 차이를 쉬운 문제로 변환         |
| **트랜스포머 기반**  | AerialFormer 15 | 트랜스포머 인코더 + 경량 MD-CNN 디코더 | 트랜스포머로 전역 컨텍스트 포착, CNN으로 지역 정보 집계      | 항공 이미지의 복잡한 배경, 작은 객체 분할     |
|                      | ETQ-Matcher 16  | Quadtree-Attention Transformer         | 쿼드트리 기반 어텐션으로 효율적인 전역-지역 특징 융합        | 복잡한 도시 환경에서의 강인한 특징 매칭       |
| **하이브리드**       | Wang et al. 6   | CNN + Transformer                      | CNN으로 특징 추출 후 트랜스포머로 관계 모델링                | CNN의 지역성과 트랜스포머의 전역성 결합       |



## 4. V. 핵심 난제 극복 전략: 거대한 시점 차이를 넘어서



최첨단 딥러닝 모델의 성능은 단순히 더 크고 깊은 네트워크를 만드는 것에서 나오지 않는다. 오히려 기하학적, 의미론적, 데이터 공학적 '사전 지식'을 학습 파이프라인에 영리하게 주입하여 모델이 풀어야 할 문제 자체를 단순화시키는 '지식 주입(Knowledge Injection)' 전략에서 비롯된다. 이는 딥러닝 모델이라는 '블랙박스'에 인간의 통찰력을 더하는 과정이라 할 수 있다.



### 4.1  기하학적 사전 지식 활용: 극좌표 변환



Aerial-Ground Image Matching의 가장 큰 어려움은 기하학적 불일치이다. 이 문제를 완화하기 위해 "지상 파노라마 뷰와 항공 뷰 사이에는 방사형-선형의 기하학적 대응 관계가 존재한다"는 사전 지식을 활용할 수 있다. **극좌표 변환(Polar Transform)**은 이 아이디어를 구현한 기술이다. 항공 이미지의 중심점을 기준으로 이미지를 극좌표계로 변환하면, 동심원은 수평선으로, 방사선은 수직선으로 펼쳐진다. 이 변환된 항공 이미지는 지상 파노라마 이미지와 유사한 구조를 갖게 되어, 딥러닝 모델이 학습해야 할 기하학적 변환의 복잡도를 크게 낮춰준다.17 SAFA와 같은 모델은 이 변환을 전처리 단계로 도입하여, 모델이 내용적 특징에 더 집중할 수 있도록 함으로써 성능을 극적으로 향상시켰다.17



### 4.2  의미론적 정보 통합: 시맨틱 분할



항공-지상 이미지는 계절, 날씨, 시간에 따라 모습이 크게 변하는 나무, 자동차, 그림자와 같은 불안정한 요소들을 많이 포함한다. 반면, 도로, 건물, 다리와 같은 인공 구조물은 상대적으로 변하지 않는 안정적인 특징이다. **시맨틱 분할(Semantic Segmentation)**은 "도로와 건물은 중요하고, 나무와 차는 덜 중요하다"는 의미론적 지식을 모델에 주입하는 방법이다. 이미지의 각 픽셀을 '도로', '건물', '식생' 등 의미 있는 클래스로 분류한 시맨틱 분할 마스크를 원본 이미지와 함께 모델의 입력으로 사용한다.6 이를 통해 모델은 일시적이거나 변화가 심한 요소는 무시하고, 위치 추정에 핵심적인 안정적인 구조물에 '집중(focus)'하도록 학습된다. 이 전략은 조명, 계절, 동적 객체 변화에 대한 모델의 강인성을 높이는 데 매우 효과적이다.14



### 4.3  데이터 부족 문제 해결: 유사-합성 데이터셋 구축



딥러닝 모델을 훈련시키기 위해서는 대규모의 정답(정확히 매칭된 항공-지상 이미지 쌍) 데이터셋이 필요하다. 하지만 현실 세계에서 이러한 데이터를 정밀하게 구축하는 것은 엄청난 비용과 노력이 들며, 이는 "정확한 매칭을 하려면 데이터가 필요한데, 데이터를 만들려면 정확한 매칭 기술이 필요한" 딜레마에 빠지게 한다. 이 문제를 해결하기 위해 "실제 데이터가 부족할 때, 고품질 3D 모델을 활용하여 데이터를 만들 수 있다"는 데이터 공학적 지식이 활용된다. Google Earth와 같은 전 지구적 3D 텍스처 메시로부터 다양한 고도와 시점의 항공 이미지를 렌더링하여 **'유사-합성(pseudo-synthetic)'** 데이터를 생성한다. 그리고 MegaDepth와 같은 실제 크라우드소싱 지상 이미지를 이 합성 데이터와 동일한 좌표계에 정합하여 대규모 하이브리드 데이터셋(예: AerialMegaDepth)을 구축한다.1 이 접근법은 확장 가능한 방식으로 딥러닝 모델에 필요한 방대한 양의 기하학적 감독(geometric supervision)을 제공하며, 실제 이미지에 대한 모델의 일반화 성능을 크게 향상시킨다.1



### 4.4  환경 변화 대응: 조명, 계절, 날씨



장기적인 자율 항법 시스템이 마주하는 가장 큰 현실적인 난제는 극심한 환경 변화이다. 낮과 밤의 조명 변화, 여름과 겨울의 계절 변화, 맑은 날과 흐린 날의 날씨 변화는 이미지의 외형을 완전히 바꾸어 매칭을 매우 어렵게 만든다.19 이러한 문제에 대응하기 위해 다음과 같은 전략들이 연구되고 있다.

- **강인한 디스크립터 학습:** 다양한 환경 조건에서 촬영된 데이터를 대규모로 학습하여, 변화 자체를 포용할 수 있는 강인한 임베딩 공간을 만드는 방법이다.22
- **다중 모달리티 융합 (Multi-modal Fusion):** 가시광선 카메라의 한계를 보완하기 위해 다른 종류의 센서 데이터를 융합하는 전략이다. 예를 들어, 조명 변화에 둔감한 열적외선(TIR) 이미지 23, 날씨나 조명에 거의 영향을 받지 않는 합성 개구 레이더(SAR) 이미지 24, 또는 정밀한 3차원 구조 정보를 제공하는 LiDAR 데이터 21를 가시광선 이미지와 함께 사용하여 시스템 전체의 강인성을 확보한다.



## 5. VI. 주요 응용 분야 및 사례 연구



Aerial-Ground Image Matching 기술은 다양한 산업 분야에서 혁신을 주도하고 있다. 이 기술의 다양한 활용 사례를 관통하는 핵심 기능은, 전통적인 신호 기반 GPS가 작동하지 않거나 정밀도가 부족한 상황에서 '시각 정보'를 이용해 절대적인 위치와 방향을 제공하는, 이른바 **'시각적 GPS(Visual GPS)'**로서의 역할이다.



### 5.1  자율 항법의 눈: GNSS 음영 지역 극복





#### 5.1.1  사례 연구 1: 도심항공모빌리티 (AAM/UAM)



미래 교통수단으로 주목받는 도심항공모빌리티(AAM)의 가장 큰 기술적 허들 중 하나는 도심 환경에서의 안전한 항법이다. 고층 빌딩이 밀집한 도심 협곡(urban canyon)에서는 GNSS 신호가 다중 경로 오차를 일으키거나 완전히 차단되어 위치 정보의 신뢰성이 급격히 저하된다.4 이는 AAM의 안전 운항에 치명적인 위협이 될 수 있다.

이 문제의 해결책으로 영상 기반 위치 추정 기술이 부상하고 있다. AAM 비행체에 탑재된 카메라가 실시간으로 촬영하는 영상을, 사전에 구축된 고정밀 3차원 공간정보(위성/항공 사진 기반 지형 모델, 3D 건물 모델 등)와 정합하는 것이다.4 이 기술은 'Geo-Referencing'이라고도 불리며, 현재 촬영된 영상에 실제 지리 좌표를 부여하여 GNSS 없이도 비행체의 절대 위치와 자세를 센티미터 수준으로 정밀하게 추정할 수 있게 한다.4 한국전자통신연구원(ETRI) 등 국내외 여러 연구기관에서 AAM의 안전 운항을 위한 핵심 기술로 이 분야를 활발히 연구하고 있다.26



#### 5.1.2  사례 연구 2: 자율주행 자동차



자율주행 자동차가 차선을 정확히 유지하고 복잡한 교차로를 안전하게 통과하기 위해서는 일반적인 GPS의 미터급 정확도를 훨씬 뛰어넘는 수십 센티미터 이내의 정밀한 위치 정보가 필수적이다.29 이를 위해 자율주행 기술은 **HD맵(정밀지도)**을 활용한다. HD맵은 차선, 신호등, 표지판 등 도로의 모든 정적 정보를 센티미터급 정밀도로 담고 있는 3차원 지도이다. 자율주행차는 자신의 카메라나 LiDAR 센서로 주변 환경을 인식하고, 이 정보를 HD맵의 정보와 실시간으로 매칭하여 지도상에서 자신의 정확한 위치를 찾는 **'측위(localization)'**를 수행한다.30 여기서 Aerial-Ground Image Matching 기술은 HD맵 제작 단계에서 항공 사진과 지상 측량 데이터를 결합하거나, 주행 중인 차량이 촬영한 영상과 HD맵을 정합하는 데 핵심적인 역할을 한다.



### 5.2  정밀 농업 (Precision Agriculture)



정밀 농업은 데이터를 기반으로 농업 자원을 효율적으로 관리하여 생산성을 극대화하는 것을 목표로 한다. 드론으로 촬영하고 정사보정한 고해상도 항공 이미지는 정밀 농업의 핵심 데이터 소스이다.3

- **정확한 면적 계산 및 관리:** 왜곡이 제거된 정사영상을 통해 실제 경작 면적을 정확히 측정하고, 토지 경계를 명확히 하여 분쟁을 예방한다.
- **토양 및 작물 분석:** 다중분광 또는 초분광 카메라로 촬영한 이미지를 분석하여 토양의 유기물 함량이나 수분 상태를 파악하고, 식생 지수(NDVI 등)를 계산하여 작물의 성장 상태, 스트레스, 병충해 발생 여부를 조기에 진단한다. 이를 통해 비료나 농약을 필요한 곳에만 정밀하게 살포하여 비용을 절감하고 환경을 보호할 수 있다.
- **수확량 예측:** 시계열 항공 이미지를 분석하여 작물의 생육 패턴을 추적하고, 이를 기반으로 전체 수확량을 예측하여 유통 및 판매 계획을 수립하는 데 활용한다.



### 5.3  지리정보 및 보안 (Geospatial Intelligence and Security)



Aerial-Ground Image Matching은 **'Cross-view Geo-localization'** 이라는 이름으로 지리정보 및 보안 분야에서 중요한 역할을 수행한다. 이 기술은 출처를 알 수 없는 한 장의 지상 이미지(예: 테러 용의자가 SNS에 게시한 사진)를 단서로, 전 세계를 커버하는 방대한 위성 이미지 데이터베이스와 매칭하여 해당 사진이 촬영된 지리적 위치를 특정하는 것을 목표로 한다. 이는 극심한 시점 차이를 극복해야 하는 매우 도전적인 문제이지만, 딥러닝 기술의 발전으로 정확도가 크게 향상되었다. 이 기술은 허위 정보나 선전·선동에 사용된 이미지의 출처를 추적하고, 범죄 현장을 파악하는 등 저널리즘, 법의학 분석, 국가 안보 분야에서 강력한 정보 분석 도구로 활용될 잠재력을 가지고 있다.6



## 6. VII. 결론: 미래 전망 및 연구 방향





### 7.1 현재 기술 요약 및 한계



Aerial-Ground Image Matching 기술은 고전적인 특징점 기반 방식의 한계를 극복하고, 딥러닝을 통해 비약적인 발전을 이루었다. 현재의 최첨단 기술은 CNN, 트랜스포머, GAN 등 다양한 딥러닝 아키텍처를 융합하고, 극좌표 변환이나 시맨틱 분할과 같은 사전 지식을 주입하는 하이브리드 모델이 주류를 이루고 있다. 이를 통해 과거에는 불가능하다고 여겨졌던 극심한 시점과 환경 변화 속에서도 상당한 수준의 매칭 성능을 달성했다.

하지만 상용화를 위해서는 여전히 해결해야 할 과제들이 남아있다. 현재의 SOTA 모델들은 대부분 매우 크고 무거워 실제 드론이나 차량의 제한된 컴퓨팅 자원에 탑재하기 어렵다. 또한, 학습 데이터에 없었던 극단적인 조명이나 계절 변화에 대한 강인성은 여전히 부족하며, 성능은 대규모의 고품질 학습 데이터셋에 크게 의존한다는 한계를 가진다.



### 6.1  미래 연구 방향 및 미해결 과제



이러한 한계를 극복하고 기술을 한 단계 더 발전시키기 위한 미래 연구는 단순히 벤치마크 점수를 높이는 것을 넘어, 실제 세계의 복잡성과 역동성 속에서 신뢰성 있게 작동하는 완전한 **'자율 지각 시스템(Autonomous Perception System)'**을 구축하는 것을 목표로 해야 한다. 이를 위한 핵심 연구 방향은 다음과 같다.

- **실시간 성능 및 경량화 (Real-time Performance and Lightweight Models):** 시스템의 **효율성(Efficiency)**을 높이기 위해, 모델 압축, 지식 증류, 양자화 등의 기술을 적용하여 현재 SOTA 모델의 성능을 유지하면서도 계산 자원이 제한된 엣지 디바이스에서 실시간으로 작동할 수 있는 경량 모델 개발이 시급하다.11
- **평생 학습 및 적응성 (Lifelong Learning and Adaptability):** 시스템의 **적응성(Adaptability)**을 확보하는 것이 중요하다. 도시는 끊임없이 변화하므로(신축 건물, 도로 공사 등), 한 번 학습된 모델이 새로운 환경 변화에 지속적으로 적응하고 성능 저하 없이 장기간 작동할 수 있는 '평생 학습(Lifelong Learning)' 또는 '연속 학습(Continual Learning)' 능력은 실제 상용화를 위한 핵심 기술이다.19
- **강인성을 위한 다중 모달리티 융합 (Multi-modal Fusion for Robustness):** 시스템의 **강인성(Robustness)**을 극대화해야 한다. 가시광선 카메라가 취약한 야간이나 악천후 환경을 극복하기 위해, 열적외선(TIR) 23, 합성 개구 레이더(SAR) 24, 라이다(LiDAR) 21 등 이종 센서 데이터를 효과적으로 융합하는 연구가 더욱 중요해질 것이다.
- **비지도/자기지도 학습 (Unsupervised/Self-supervised Learning):** 시스템의 **학습 자율성(Autonomy in Learning)**을 높여야 한다. 레이블링 비용이 많이 드는 지도 학습의 한계에서 벗어나, 레이블이 없는 방대한 양의 데이터로부터 스스로 유용한 특징을 학습하는 비지도/자기지도 학습 방법론은 기술의 확장성과 범용성을 높이는 데 결정적인 역할을 할 것이다.32
- **더 나은 벤치마크 데이터셋 (Better Benchmark Datasets):** 기술 발전을 올바른 방향으로 이끌기 위해, 현재의 벤치마크들이 다루지 못하는 더 다양하고 도전적인 시나리오(예: 극심한 계절/조명 변화, 넓은 지역, 다양한 기상 조건)를 포함하는 대규모의 표준화된 벤치마크 데이터셋 구축이 필요하다.33

결론적으로, Aerial-Ground Image Matching 기술의 미래는 개별 알고리즘의 성능 향상을 넘어, 효율적이고, 강인하며, 스스로 학습하고 적응하는 자율 지각 시스템을 향해 나아가고 있다. 이러한 시스템이 완성될 때, 비로소 하늘과 땅의 시각 정보를 완벽하게 연결하여 진정한 의미의 자율 항법과 지능형 공간 인식의 시대를 열게 될 것이다.

#### **참고 자료**

1. AerialMegaDepth: Learning Aerial-Ground Reconstruction and View Synthesis - arXiv, accessed July 1, 2025, https://arxiv.org/html/2504.13157v1
2. Learning to Match Aerial Images With Deep Attentive Architectures - The Computer Vision Foundation, accessed July 1, 2025, https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Altwaijry_Learning_to_Match_CVPR_2016_paper.pdf
3. 항공사진 정사보정 방법: 농업 현장에서 활용하는 정확한 항공 이미지 처리 기술 - 재능넷, accessed July 1, 2025, https://www.jaenung.net/tree/27219
4. Advanced Air Mobility를 위한 영상 기반 위치 추정 및 Geo-Referencing 기술 동향, accessed July 1, 2025, [https://ettrends.etri.re.kr/ettrends/209/0905209001/001-009.%20%EC%B5%9C%EC%9D%98%ED%99%98_209%ED%98%B8%20%EC%B5%9C%EC%A2%85.pdf](https://ettrends.etri.re.kr/ettrends/209/0905209001/001-009. 최의환_209호 최종.pdf)
5. Advanced Air Mobility를 위한 영상 기반 위치 추정 및 Geo-Referencing 기술 동향, accessed July 1, 2025, https://ettrends.etri.re.kr/ettrends/209/0905209001/
6. Enhancing Ground-to-Aerial Image Matching for Visual Misinformation Detection Using Semantic Segmentation - arXiv, accessed July 1, 2025, https://arxiv.org/html/2502.06288v2
7. Unsupervised Deep Homography: A Fast and Robust Homography ..., accessed July 1, 2025, https://arxiv.org/pdf/1709.03966
8. A Review of Homography Estimation: Advances and Challenges, accessed July 1, 2025, https://www.mdpi.com/2079-9292/12/24/4977
9. Real-time and high precision feature matching between blur aerial images - PMC, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9484699/
10. Comparison of Deep Learning and Feature Matching Methods For Homography Estimation, accessed July 1, 2025, https://hammer.purdue.edu/articles/thesis/Comparison_of_Deep_Learning_and_Feature_Matching_Methods_For_Homography_Estimation/10700231
11. Traditional image matching by ORB [6]. The corresponding matched key... - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/figure/Traditional-image-matching-by-ORB-6-The-corresponding-matched-key-points-are-linked-by_fig25_348174979
12. CVM-Net: Cross-View Matching Network for Image-Based ... - GitHub, accessed July 1, 2025, https://github.com/david-husx/crossview_localisation
13. kregmi/cross-view-image-matching - GitHub, accessed July 1, 2025, https://github.com/kregmi/cross-view-image-matching
14. arXiv:2404.11302v2 [cs.CV] 23 May 2024, accessed July 1, 2025, https://arxiv.org/pdf/2404.11302
15. [2306.06842] AerialFormer: Multi-resolution Transformer for Aerial Image Segmentation - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2306.06842
16. ETQ-Matcher: Efficient Quadtree-Attention-Guided Transformer for Detector-Free Aerial–Ground Image Matching - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/17/7/1300
17. YujiaoShi/cross_view_localization_SAFA - GitHub, accessed July 1, 2025, https://github.com/YujiaoShi/cross_view_localization_SAFA
18. YujiaoShi/cross_view_localization_DSM - GitHub, accessed July 1, 2025, https://github.com/YujiaoShi/cross_view_localization_DSM
19. CV4RA/SOTA-Place-Recognitioner - GitHub, accessed July 1, 2025, https://github.com/CV4RA/SOTA-Place-Recognitioner
20. DINO-Mix enhancing visual place recognition with foundational vision model and feature mixing - PMC, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11437288/
21. Satellite Image-based Localization via Learned Embeddings - TTIC, accessed July 1, 2025, https://www.ttic.edu/ripl/assets/publications/kim17.pdf
22. Semantic Visual Localization - Andreas Geiger, accessed July 1, 2025, https://www.cvlibs.net/publications/Schoenberger2018CVPR.pdf
23. XoFTR: Cross-modal Feature Matching Transformer - arXiv, accessed July 1, 2025, https://arxiv.org/html/2404.09692v1
24. lan-cz/cnn-matching: Source code and datadset for "Deep learning algorithm for feature matching of cross modality remote sensing images" - GitHub, accessed July 1, 2025, https://github.com/lan-cz/cnn-matching
25. RISS 검색 - 학술지 상세보기, accessed July 1, 2025, https://m.riss.kr/search/detail/DetailView.do?p_mat_type=3a11008f85f7c51d&control_no=90e320e6ac2d4a1c&v_control_no=8df4d97bc78235a9ffe0bdc3ef48d419
26. Advanced Air Mobility를 위한 ... - ETRI Knowledge Sharing Platform, accessed July 1, 2025, https://ksp.etri.re.kr/ksp/article/read?id=69932
27. 지능형 미래사회를 위한 디지털 융합기술 전망 - ETRI Electronics and Telecommunications Trends - 한국전자통신연구원, accessed July 1, 2025, https://ettrends.etri.re.kr/ettrends/209/
28. Advanced Air Mobility를 위한 영상 기반 위치 추정 및 Geo-Referencing 기술 동향, accessed July 1, 2025, https://koreascience.kr/article/JAKO202464143225405.page?&lang=ko
29. 자율주행 자동차엔 '특별한 지도'가 필요하다?! - Samsung Newsroom, accessed July 1, 2025, [https://news.samsung.com/kr/%EC%9E%90%EC%9C%A8%EC%A3%BC%ED%96%89-%EC%9E%90%EB%8F%99%EC%B0%A8%EC%97%94-%ED%8A%B9%EB%B3%84%ED%95%9C-%EC%A7%80%EB%8F%84%EA%B0%80-%ED%95%84%EC%9A%94%ED%95%98%EB%8B%A4](https://news.samsung.com/kr/자율주행-자동차엔-특별한-지도가-필요하다)
30. KR102219843B1 - 자율주행을 위한 위치 추정 장치 및 방법 - Google Patents, accessed July 1, 2025, https://patents.google.com/patent/KR102219843B1/ko
31. 자율주행의 눈 고정밀지도…이젠 AI가 척척 만들어준다 - 한국경제, accessed July 1, 2025, https://www.hankyung.com/article/2020091111211
32. Modeling 3D Layout For Group Re-Identification | Request PDF - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/363906337_Modeling_3D_Layout_For_Group_Re-Identification
33. ericzzj1989/Awesome-Image-Matching - GitHub, accessed July 1, 2025, https://github.com/ericzzj1989/Awesome-Image-Matching


# DeepFace (얼굴 인식)

## 서론

'통제되지 않은 환경(in-the-wild)'에서의 얼굴 인식은 컴퓨터 비전 분야의 오랜 난제였다. 조명, 포즈, 표정, 가림(occlusion) 등 예측 불가능한 변수들은 기존의 알고리즘이 실세계 환경에서 유의미한 성능을 내는 것을 가로막는 근본적인 장벽으로 작용했다.1 이러한 배경에서 Labeled Faces in the Wild (LFW)와 같은 벤치마크는 알고리즘의 강건함과 일반화 성능을 측정하는 중요한 시금석으로 부상했다.3 2014년, 페이스북 AI 연구팀이 발표한 DeepFace는 이 LFW 벤치마크에서 97.35%라는 경이적인 정확도를 기록하며 학계와 산업계에 거대한 충격을 안겼다.1 이는 당시 인간의 인식 능력(약 97.53%)에 거의 근접한 수치로, 기계가 인간의 고유한 시각 능력에 도달할 수 있다는 가능성을 현실로 만들었다.3 이 사건은 딥러닝이 얼굴 인식 분야의 지배적인 패러다임으로 자리 잡는 결정적 전환점이 되었다.

본 보고서는 DeepFace의 기술적 혁신을 심층적으로 분석하고, 이를 딥러닝 이전 시대의 기술적 맥락 안에 위치시켜 그 의미를 조명한다. 나아가 DeepFace가 후속 연구인 FaceNet, VGGFace, ArcFace 등에 미친 영향을 추적하며 기술 발전의 계보를 확립하고, 마지막으로 이 눈부신 기술적 성취가 야기한 복잡한 사회적, 윤리적 함의를 비판적으로 고찰하는 것을 목표로 한다.

## 1.  딥러닝 이전의 얼굴 인식: 수제 특징점의 시대

딥러닝이 얼굴 인식 분야를 지배하기 이전, 연구자들은 이미지로부터 유의미한 정보를 추출하기 위해 정교하게 설계된 '수제 특징점(hand-crafted features)'에 의존했다. 이 시대의 방법론들은 통제된 환경에서는 상당한 성과를 보였으나, LFW와 같은 '통제되지 않은 환경'에서는 근본적인 한계에 부딪혔다.

### 1.1 주요 방법론 분석

딥러닝 이전 시대를 대표하는 두 가지 주요 방법론은 LBP(Local Binary Patterns)와 Fisher Vectors(FV)였다.

- **LBP (Local Binary Patterns):** LBP는 이미지의 텍스처를 분석하기 위한 효과적인 기술자(descriptor)였다. 그 원리는 간단하다. 특정 픽셀을 중심으로 주변 8개 이웃 픽셀의 밝기 값을 비교하여, 중심 픽셀보다 밝으면 1, 어두우면 0으로 이진화된 코드를 생성한다. 이 8비트 이진 코드는 해당 영역의 미세한 텍스처 패턴을 나타낸다.7 LBP는 조명 변화에 비교적 강건하다는 장점이 있었지만, 얼굴의 포즈가 크게 바뀌거나 표정 변화가 심할 경우 특징값 자체가 심하게 왜곡되어 성능이 저하되는 명백한 한계를 가졌다.8 이를 극복하기 위해 'High-dimensional LBP'와 같은 변형이 등장했다. 이는 얼굴의 주요 랜드마크(눈, 코, 입 등) 주변에서 여러 개의 이미지 패치를 추출하고, 각 패치에서 LBP 특징을 계산한 뒤 이를 모두 연결하여 하나의 긴 벡터로 만드는 방식이다. 이를 통해 더 풍부한 정보를 담고자 했으나, 근본적인 표현력의 한계를 넘어서지는 못했다.1
- **Fisher Vectors (FV):** FV는 LBP보다 더 정교한 통계적 모델링에 기반한다. 이 방법은 이미지 전체에서 SIFT(Scale-Invariant Feature Transform)와 같은 저수준 특징들을 밀집하게(densely) 추출한 뒤, 이 특징들의 분포를 GMM(Gaussian Mixture Model)으로 모델링한다.10 최종적으로 FV는 추출된 특징들이 학습된 GMM 모델로부터 얼마나 벗어나는지를 그래디언트 벡터 형태로 인코딩한 것이다. 이는 이미지의 전반적인 특징 분포를 매우 높은 차원의 벡터로 표현하여 뛰어난 판별력을 얻는 방식이며, 실제로 딥러닝 등장 직전 LFW 벤치마크에서 최고 수준의 성능을 기록하기도 했다.10

### 1.2 LFW 벤치마크의 역할과 기술적 정체기

2007년에 공개된 LFW 데이터셋은 얼굴 인식 연구의 방향을 완전히 바꾸어 놓았다.3 이전의 ORL, FERET과 같은 데이터셋들이 통제된 조명과 정면 포즈 위주였던 반면 12, LFW는 웹에서 수집된 실제 사진들로 구성되어 포즈, 조명, 표정, 연령, 인종, 가림 등 온갖 변수들을 포함하고 있었다. 이는 기존 방법론들의 일반화 성능을 시험하는 혹독한 무대가 되었고, 수제 특징점 기반 기술의 한계를 명확히 드러냈다.

DeepFace가 발표되기 직전인 2013년에서 2014년 초, LFW 벤치마크의 최고 정확도는 'TL Joint Bayesian'과 같은 방법론이 기록한 96.33% 수준이었다.1 이러한 시스템들은 앞서 설명한 LBP나 FV 같은 수제 특징점 추출 방식과 Joint Bayesian, SVM(Support Vector Machine)과 같은 정교한 통계적 분류 모델을 결합한 형태였다. 성능 향상은 점진적으로 이루어졌고, 이는 더 복잡한 특징 공학과 여러 모델을 조합하는 앙상블 기법에 크게 의존했다. 이는 분야 전체가 근본적인 표현력의 한계라는 벽에 부딪혀 새로운 돌파구를 찾지 못하는 기술적 정체기에 접어들었음을 시사했다.

이 시대의 본질적인 한계는 '특징 공학(feature engineering)' 그 자체에 있었다. LBP나 SIFT 같은 특징들은 인간이 사전에 정의한 규칙에 기반하기 때문에, 'in-the-wild' 환경에서 나타나는 무한에 가까운 변수들을 모두 포착할 수 없었다. 연구자들은 다중 스케일, 다중 랜드마크, 고차원 벡터화, 복잡한 분류기 결합 등 정교한 기법들을 동원했지만, 이는 문제의 근원인 '특징 표현의 취약성'을 해결하지 못하는 임시방편에 가까웠다. LFW 데이터셋은 바로 이 취약성을 수면 위로 드러냈고, 결과적으로 데이터로부터 직접 강건한 특징을 학습하는 '특징 학습(feature learning)' 패러다임으로의 전환이 왜 필연적이었는지를 역설했다. DeepFace는 이 거대한 전환을 이끈 최초의 성공 사례였다.

## 2.  DeepFace의 등장: 인간 수준 성능의 서막

2014년, DeepFace의 등장은 기술적 정체기에 있던 얼굴 인식 분야에 거대한 파문을 일으켰다. 이는 단순히 정확도를 약간 개선한 연구가 아니었다. 얼굴 인식을 바라보는 관점과 접근 방식 자체를 근본적으로 바꾼 패러다임의 전환이었다. DeepFace의 성공은 정교한 얼굴 정렬, 전례 없는 규모의 데이터, 그리고 깊은 신경망이라는 세 가지 요소의 시너지 효과에 기인한다.

### 2.1  파이프라인과 핵심 기여

DeepFace는 현대 얼굴 인식 시스템의 표준이 된 '탐지(Detect) → 정렬(Align) → 표현(Represent) → 분류(Classify)'의 4단계 파이프라인을 명확하게 제시했다.13 이전 연구들이 각 단계를 개별적으로 최적화하는 데 집중했다면, DeepFace는 특히 **정렬**과 **표현** 단계의 유기적 관계에 주목했다. 즉, 매우 정교한 정렬 과정을 통해 입력 이미지의 변수를 최소화하면, 심층 신경망이 오직 '신원(identity)' 정보에만 집중하여 훨씬 더 강력한 특징 표현을 학습할 수 있다는 점을 입증한 것이다.1

### 2.2  혁신적 얼굴 정렬: 3D 모델링의 도입

DeepFace의 가장 독창적인 기여 중 하나는 3D 얼굴 모델을 이용한 정면화(frontalization) 과정이었다. 연구팀은 단순한 2D 변환(회전, 크기 조절 등)만으로는 얼굴의 3차원 구조에서 비롯되는 비강체 변형(non-rigid deformation)이나 정면을 벗어난 회전(out-of-plane rotation) 문제를 해결할 수 없다는 점을 정확히 인지했다.1

DeepFace의 3D 정렬 과정은 다음과 같이 진행된다 1:

1. **초기 2D 정렬:** 먼저 눈, 코, 입 중심 등 6개의 주요점을 탐지하여 이미지를 대략적으로 회전시키고 크기를 조절한다.
2. **세부 랜드마크 탐지:** 2D로 정렬된 이미지 위에서 67개의 세밀한 랜드마크를 추가로 탐지한다.
3. **3D 모델 피팅:** 이 67개의 점을 일반적인(generic) 3D 얼굴 모델의 해당 지점과 대응시켜, 각 2D 이미지에 최적화된 3D-2D 카메라 투영 행렬을 추정한다.
4. **정면화:** 추정된 카메라 행렬을 이용해 3D 모델을 가상으로 회전시켜 완벽한 정면 뷰를 생성한다. 이때, 67개의 랜드마크가 형성하는 삼각형 메시(mesh)를 기반으로 '부분적 아핀 변환(Piecewise Affine Transformation)'을 적용하여 각 삼각형 영역을 독립적으로 워핑(warping)한다. 이 방식은 얼굴 전체를 하나의 강체처럼 변환할 때 발생하는 질감의 부자연스러운 왜곡을 최소화한다.5

이 정교한 정면화 과정은 입력 이미지에 포함된 가장 큰 변수인 '포즈'를 효과적으로 제거하는 강력한 정규화(normalization) 역할을 수행했다. 그 결과, 후속 신경망은 다양한 각도와 자세에 대응하는 법을 배울 필요 없이, 오직 사람을 구별하는 미세한 특징 학습에만 집중할 수 있게 되었다.

### 2.3  심층 신경망 구조와 학습

정면화된 얼굴 이미지는 DeepFace의 심층 신경망으로 입력되어 고유한 특징 벡터로 변환된다.

- **네트워크 아키텍처:** DeepFace 네트워크는 당시 기준으로 매우 깊은 9개의 학습 가능한 계층으로 구성되었다.5 초기에는 일반적인 컨볼루션 레이어(C1, C2)를 사용하여 엣지나 코너 같은 저수준 특징을 추출한다. 하지만 중간 계층(L3, L4, L5, L6)에는 가중치를 공유하지 않는(un-shared weights) '지역 연결 계층(Locally Connected Layers)'이라는 독특한 구조를 사용했다. 이는 얼굴의 각기 다른 영역(예: 눈, 코, 입)이 공간적으로 고유한 통계적 특성을 가질 것이라는 해부학적 직관에 기반한다. 가중치 공유를 제거함으로써 각 지역에 특화된 특징을 학습할 수 있도록 한 것이다. 마지막으로 두 개의 완전 연결 계층(F7, F8)이 이 모든 특징을 종합하여 4096차원의 고차원 특징 벡터, 즉 'DeepFace 표현'을 생성한다.

| 계층 (Layer)     | 종류 (Type)              | 필터/크기 (Filter/Size) | 출력 맵 크기 (Output Map Size) | 파라미터 수 (# Parameters) |
| ---------------- | ------------------------ | ----------------------- | ------------------------------ | -------------------------- |
| **입력 (Input)** | 이미지 (Image)           | -                       | 152x152x3                      | -                          |
| **C1**           | Convolution + ReLU       | 32 @ 11x11x3            | 142x142x32                     | ~3.9K                      |
| **M1**           | Max-pooling              | 3x3, stride 2           | 71x71x32                       | 0                          |
| **C2**           | Convolution + ReLU       | 16 @ 9x9x16             | 63x63x16                       | ~20.7K                     |
| **L3**           | Locally Connected + ReLU | 16 @ 9x9x16             | 55x55x16                       | ~3.9M                      |
| **L4**           | Locally Connected + ReLU | 16 @ 7x7x16             | 49x49x16                       | ~4.8M                      |
| **L5**           | Locally Connected + ReLU | 16 @ 5x5x16             | 45x45x16                       | ~3.3M                      |
| **F6**           | Fully Connected + ReLU   | -                       | 4096                           | ~134M                      |
| **F7**           | Fully Connected (Output) | -                       | 4030                           | ~16.5M                     |
| **Total**        |                          |                         |                                | **~162.5M**                |

- **대규모 데이터셋 학습:** DeepFace 성공의 또 다른 핵심은 'Social Face Classification (SFC)'이라는 페이스북 내부 데이터셋의 활용이었다. 이 데이터셋은 **4,000명 이상의 신원에 대한 440만 개의 얼굴 이미지**로 구성되어, 당시 공개된 어떤 데이터셋과도 비교할 수 없는 규모를 자랑했다.1 이처럼 방대한 데이터는 1억 2천만 개가 넘는 파라미터를 가진 깊은 네트워크가 과적합(overfitting) 없이 강력한 일반화 성능을 갖춘 특징을 학습하는 데 결정적인 연료 역할을 했다.1
- **학습 전략:** 학습은 마지막 계층에 4000개 이상의 출력(신원의 수)을 갖는 Softmax 손실 함수를 사용하여 다중 클래스 분류 문제로 진행되었다. 즉, 네트워크의 학습 목표는 입력된 얼굴 이미지가 4000여 명의 인물 중 누구인지를 정확히 맞추는 것이었다.

### 2.4  LFW 벤치마크 성능 분석

SFC 데이터셋으로 학습된 DeepFace 네트워크는 LFW 벤치마크에서 그 성능을 입증했다. 평가 시에는 학습에 사용된 마지막 분류 계층(F7)을 제거하고, 그 직전의 완전 연결 계층(F6)에서 나온 4096차원 벡터를 해당 얼굴의 고유한 '표현'으로 사용했다. LFW 테스트는 간단했다. 두 개의 이미지에서 각각 이 4096차원 특징 벡터를 추출한 뒤, 두 벡터 간의 유사도(예: 거리)를 측정하여 동일인 여부를 판별하는 것이다. 이 간단한 방식만으로 DeepFace는 **97.35%**라는 정확도를 달성했다.1 이는 당시 최고 기록이었던 96.33%의 오류율을 27% 이상 극적으로 감소시킨 혁신적인 결과였으며, 인간의 인식률(97.53%)과 거의 차이가 없는 수준이었다.1

DeepFace의 성공은 단일 기술의 승리가 아니었다. 이는 세 가지 핵심 요소의 완벽한 시너지 효과에서 비롯되었다. 첫째, **3D 정렬**은 입력 데이터의 복잡성을 크게 낮추어 신경망이 풀어야 할 문제를 단순화시켜 주었다. 둘째, **대규모 데이터**는 신경망이 다양한 변화 속에서도 불변하는 '신원'의 본질을 학습할 수 있도록 풍부한 사례를 제공했다. 마지막으로, **깊은 네트워크**는 이 방대한 데이터로부터 복잡하고 추상적인 특징 계층을 학습할 수 있는 충분한 '용량(capacity)'을 가졌다. 이 세 요소는 서로를 보완하며 성능을 극대화했다. 3D 정렬이 문제의 난이도를 낮추고, 대규모 데이터가 학습의 질을 높이며, 깊은 네트워크가 학습의 깊이를 더하는 이 상호보완적 구조는 DeepFace 성공의 핵심 공식이자, 이후 현대 딥러닝 시스템 설계의 기본 원칙으로 자리 잡았다.

## 3.  DeepFace 이후의 진화: 임베딩과 손실 함수의 혁신

DeepFace는 딥러NING을 얼굴 인식의 주류로 만들었지만, 동시에 새로운 연구 과제를 제시했다. DeepFace의 접근 방식은 그 자체로 완결된 것이 아니라, 더 정교하고 효율적인 방법론으로 나아가는 중요한 출발점이었다. 이후 연구들은 주로 얼굴 표현을 학습하는 '방식', 즉 손실 함수와 임베딩 공간 최적화에 집중하며 기술을 한 단계 더 발전시켰다.

### 3.1  FaceNet (2015): 임베딩 공간의 직접 최적화와 Triplet Loss

구글 연구팀이 2015년에 발표한 FaceNet은 DeepFace의 근본적인 접근 방식에 의문을 제기했다. DeepFace는 얼굴 인식을 수천 개의 클래스 중 하나로 맞추는 '분류(classification)' 문제로 정의했다. 하지만 실제 얼굴 검증(verification)은 학습 데이터에 존재하지 않는 새로운 사람을 인식해야 하는 '열린 집합(open-set)' 문제에 더 가깝다. 분류 문제에 최적화된 신경망의 중간 계층 특징이 얼굴 간의 '유사도'를 측정하는 데에도 최적이라는 보장은 없었다.15

FaceNet은 이 문제를 해결하기 위해 얼굴 인식을 '거리 학습(metric learning)' 문제로 재정의했다.15 그 핵심에는 'Triplet Loss'라는 혁신적인 손실 함수가 있었다.

- **Triplet Loss:** 이 손실 함수는 세 개의 이미지, 즉 기준이 되는 '앵커(anchor)', 앵커와 동일 인물인 '포지티브(positive)', 그리고 앵커와 다른 인물인 '네거티브(negative)'로 구성된 triplet을 이용해 학습한다. 학습 목표는 임베딩 공간상에서 '앵커-포지티브 간의 거리'가 '앵커-네거티브 간의 거리'보다 항상 특정 마진($\alpha$) 이상 작아지도록 강제하는 것이다.16

- **수학적 정의:** 손실 함수 $L$은 모든 triplet에 대해 다음과 같이 정의된다.
  $$
  L = \sum_{i}^{N} \left [ \left \| f(x_{i}^{a}) - f(x_{i}^{p}) \right \|_{2}^{2} - \left \| f(x_{i}^{a}) - f(x_{i}^{n}) \right \|_{2}^{2} + \alpha \right ]_{+}
  $$
  여기서 $f(x)$는 이미지 $x$를 유클리드 공간으로 매핑하는 임베딩 함수이며, $[z]_{+} = \max(0, z)$를 의미한다.

- **효과:** 이 방식은 Softmax처럼 간접적으로 특징을 학습하는 대신, 임베딩 공간 자체를 '같은 사람은 가깝게, 다른 사람은 멀게' 만드는 방향으로 직접 최적화한다.16 이는 얼굴 검증 작업의 목표와 정확히 일치하는 접근법이다. 그 결과, FaceNet은 DeepFace보다 훨씬 압축적이면서(4096차원 vs 128차원) 일반화 성능이 뛰어난 임베딩을 학습할 수 있었다. FaceNet은 이 접근법을 통해 LFW 벤치마크에서 **99.63%**라는, 사실상 인간의 능력을 뛰어넘는 정확도를 달성하며 새로운 시대를 열었다.15

### 3.2  VGGFace2 (2018): 데이터셋의 규모와 다양성의 힘

DeepFace와 FaceNet의 눈부신 성공은 대규모 데이터셋의 중요성을 명확히 보여주었다. 하지만 이들이 사용한 데이터셋은 각각 페이스북과 구글의 내부 자산으로, 학계 연구자들은 접근할 수 없었다. 이러한 '데이터 격차'는 공정한 연구 경쟁을 저해하는 요소였다.20

옥스퍼드 대학의 VGG(Visual Geometry Group) 연구팀은 이러한 문제를 해결하기 위해 2018년 VGGFace2 데이터셋을 공개했다.

- **VGGFace2 데이터셋:** 이 데이터셋은 **9,131명의 신원에 대한 331만 장의 이미지**로 구성된 대규모 공개 데이터셋이다.21 VGGFace2의 구축 목표는 단순히 이미지의 양을 늘리는 것을 넘어, 다음과 같은 질적 향상을 추구했다: (i) 많은 신원 수와 신원 당 충분한 이미지 수를 동시에 확보하고, (ii) 포즈, 나이, 인종 등 인구통계학적 다양성을 극대화하며, (iii) 자동화 및 수동 검수 단계를 통해 레이블 노이즈를 최소화하는 것이었다.22
- **영향:** VGGFace2와 같은 고품질 공개 데이터셋의 등장은 학계 연구자들이 산업계의 거대 기업들과 대등한 조건에서 모델을 훈련하고 검증할 수 있는 중요한 토대를 마련했다. 이는 얼굴 인식 연구의 발전이 특정 모델 아키텍처나 손실 함수뿐만 아니라, '데이터' 그 자체의 질과 양에 얼마나 크게 의존하는지를 다시 한번 확인시켜 준 계기가 되었다.



### 3.3  ArcFace (2019): 결정 경계 극대화를 위한 각도 마진 손실



FaceNet의 Triplet Loss는 개념적으로 우수했지만, 학습에 사용할 효과적인 triplet을 선별하는 과정(triplet mining)이 매우 복잡하고 계산 비용이 높다는 실질적인 단점이 있었다.17 반면, DeepFace가 사용한 Softmax 손실 함수는 학습이 간단하고 안정적이지만 특징의 판별력이 상대적으로 부족했다.

2019년에 발표된 ArcFace는 Softmax의 단순함과 안정성을 유지하면서도 Triplet Loss 수준의 높은 판별력을 확보하는 우아한 해결책을 제시했다.24

- **핵심 아이디어: Additive Angular Margin Loss:** ArcFace의 핵심은 특징 공간을 기하학적으로 해석하는 데 있다. 먼저, 심층 신경망을 통해 추출된 특징 벡터와 마지막 분류 계층의 가중치 벡터(각 클래스의 중심점으로 해석 가능)를 모두 L2 정규화한다. 이 과정을 통해 모든 특징 벡터는 단위 하이퍼스피어(hypersphere) 상에 분포하게 된다. 그 후, Softmax 확률을 계산하기 직전에, 특정 샘플($x_i$)과 그 정답 클래스($y_i$)에 해당하는 가중치 벡터($W_{y_i}$) 사이의 **각도**($\theta_{y_i}$)에 직접 고정된 마진($m$)을 더해준다($\cos(\theta_{y_i} + m)$).24
- **기하학적 의미:** 이 방식은 하이퍼스피어라는 특징 공간 위에서 클래스 간의 '각도 거리'를 직접적으로 벌려주는 효과를 가진다. 즉, 같은 클래스에 속한 특징들은 중심점에 더 가깝게 모이도록(intra-class compactness) 하고, 다른 클래스의 중심점들과는 더 멀리 떨어지도록(inter-class discrepancy) 강제하여 결정 경계를 기하학적으로 명확하게 만든다.24
- **효과:** ArcFace는 Triplet Loss의 복잡한 샘플링 과정 없이도 매우 안정적으로 학습하면서 최첨단 성능을 달성했다. 그 명확한 기하학적 해석과 구현의 용이성 덕분에 ArcFace와 그 변형들은 현재 수많은 얼굴 인식 시스템에서 표준적인 손실 함수 중 하나로 널리 사용되고 있다.27

DeepFace 이후의 기술 발전사는 얼굴 인식 문제를 어떻게 '정의'할 것인가에 대한 끊임없는 탐구의 과정이었다. DeepFace는 이 문제를 N-way '분류' 문제로 접근했다. FaceNet은 더 본질에 가까운 '거리 학습' 문제로 전환하여, 두 얼굴이 같은 사람인지 직접 묻는 방식으로 최적화를 수행했다. 그러나 이 방식은 학습 데이터 구성의 복잡성을 야기했다. ArcFace는 다시 Softmax의 분류 프레임워크로 돌아왔지만, "어떻게 하면 특징 공간 자체를 기하학적으로 더 판별력 있게 만들 수 있을까?"라는 새로운 질문을 던졌다. 그 해답으로 '각도'에 주목하여, 하이퍼스피어 상에서 클래스 중심 간의 각도 거리를 직접 벌리는 방식을 고안했다. 이는 Triplet Loss의 복잡성 없이도 거리 학습의 목표를 달성하는 우아한 해결책이었다. 이처럼 문제 정의가 정교해질수록, 솔루션은 더 강력하고 효율적으로 진화했다.

| 구분 (Category)                    | Pre-Deep Learning (LBP/FV)                        | DeepFace (2014)                                 | FaceNet (2015)                                    | ArcFace (2019)                                               |
| ---------------------------------- | ------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| **핵심 원리 (Core Principle)**     | 수제 특징 공학 (Hand-crafted Feature Engineering) | 특징 학습 (Feature Learning via Classification) | 임베딩 공간 최적화 (Embedding Space Optimization) | 특징 공간 기하학적 최적화 (Geometrical Optimization of Feature Space) |
| **특징 추출 (Feature Extraction)** | LBP, SIFT 등                                      | 9-Layer Deep Neural Network                     | Deep ConvNet (e.g., Inception)                    | Deep ConvNet (e.g., ResNet)                                  |
| **손실 함수 (Loss Function)**      | N/A (SVM, Joint Bayesian 등 분류기 사용)          | Softmax Loss (Cross-Entropy)                    | Triplet Loss                                      | Additive Angular Margin Loss (Softmax 기반)                  |
| **LFW 정확도 (LFW Accuracy)**      | ~96.33%                                           | 97.35%                                          | 99.63%                                            | >99.8%                                                       |
| **주요 기여 (Key Contribution)**   | 'In-the-wild' 문제의 기반 마련                    | 딥러닝 패러다임의 시작, 3D 정렬 도입            | Metric Learning 접근법, 임베딩 직접 학습          | 기하학적 해석 기반의 고판별력 손실 함수                      |



## 4.  기술적 성취의 이면: 윤리적 쟁점과 사회적 영향



DeepFace가 달성한 인간 수준의 성능은 기술적 경이로움이었지만, 그 이면에는 오늘날까지도 논란이 되는 심각한 윤리적 쟁점들이 자리 잡고 있다. DeepFace의 성공을 가능케 한 바로 그 요소들, 즉 대규모 데이터와 강력한 일반화 성능은 역설적으로 알고리즘 편향과 프라이버시 침해라는 그림자를 드리웠다.



### 4.1  알고리즘 편향: 데이터셋의 그림자



얼굴 인식 기술의 공정성 문제는 훈련 데이터의 질과 구성에 깊이 뿌리내리고 있다.

- **원인:** DeepFace가 학습에 사용한 SFC 데이터셋이나 이후의 VGGFace2와 같이 웹에서 대규모로 수집된 데이터셋은 필연적으로 현실 세계의 인구통계학적 불균형을 그대로, 혹은 더 왜곡된 형태로 반영한다. 예를 들어, 인터넷에는 특정 인종(백인), 성별(남성)의 유명인이나 공인의 사진이 압도적으로 많다. 이러한 편향된 데이터로 학습된 모델은 데이터가 풍부한 다수 집단에 대해서는 높은 정확도를 보이지만, 데이터가 부족한 소수 집단(예: 유색인종, 여성)에 대해서는 현저히 낮은 정확도를 보이는 '알고리즘 편향'을 학습하게 된다.29
- **실태:** 실제로 수많은 후속 연구를 통해 상용 얼굴 인식 시스템들이 특히 유색인종 여성에게서 가장 높은 오류율을 보인다는 사실이 반복적으로 입증되었다.30 이는 훈련 데이터의 편향이 알고리즘의 성능 불균형으로 직결된다는 강력한 증거다. 결국 DeepFace의 성공을 가능케 한 핵심 무기였던 '대규모 데이터'가, 사회적 공정성의 관점에서는 특정 집단을 차별하는 '독'으로 작용할 수 있음이 드러난 것이다.



### 4.2  프라이버시 침해와 대규모 감시 사회의 우려



알고리즘 편향 문제와 더불어, 데이터 수집 과정의 정당성과 기술의 오용 가능성은 더 큰 사회적 우려를 낳는다.

- **데이터 수집의 문제:** DeepFace의 SFC 데이터셋은 페이스북 사용자들이 올린 사진에 기반하며, VGGFace 데이터셋 역시 구글 이미지 검색을 통해 수집되었다. 이 과정에서 이미지에 등장하는 개인들로부터 자신의 생체 정보가 인공지능 모델 훈련에 사용된다는 점에 대해 명시적인 동의를 구하는 절차는 사실상 없었다.20 이는 개인의 가장 민감한 생체 정보인 얼굴을 동의 없이 수집하여 상업적, 연구적 목적으로 활용하는 것에 대한 심각한 프라이버시 침해 문제를 제기한다.
- **감시 기술의 확산:** DeepFace가 인간과 대등한 수준의 인식 성능을 입증함으로써, 얼굴 인식 기술은 단순한 사진 태깅 기능을 넘어 법 집행, 공공 안전, 출입 통제 등 사회 시스템 전반으로 확산될 수 있는 기술적 타당성을 확보했다.14 그러나 이는 정부나 거대 기업에 의한 '대규모 감시(mass surveillance)' 사회로의 이행을 가속화할 수 있다는 심각한 위험을 내포한다. 시민들은 공공장소에서 익명으로 존재할 권리를 상실하고, 자신의 모든 행적이 기록될 수 있다는 불안감 속에서 집회나 시위의 자유와 같은 기본적인 시민적 권리 행사를 위축받을 수 있다.29 흥미롭게도 DeepFace 논문 저자들 스스로도 이러한 위험을 예견했다. 그들은 논문 서두에서 "기계와 인간 시각 시스템 간의 성능 격차는 지금까지 이러한 (사회적) 함의를 다루는 것을 막는 완충재 역할을 해왔다"고 서술하며, 자신들의 연구가 바로 그 완충재를 제거했음을 암시했다.1

얼굴 인식 기술의 발전사는 '성능 지상주의'가 어떻게 윤리적 문제를 부차적인 것으로 만들었는지를 보여주는 대표적인 사례다. 더 높은 정확도를 달성하기 위해 더 큰 데이터셋을 추구하는 과정에서, 데이터 수집의 윤리성(동의 여부)과 데이터 분포의 공정성(편향 문제)은 성능 향상이라는 목표 아래 쉽게 간과되었다. DeepFace의 성공은 기술적 목표 달성을 위해 어떤 가치들이 희생될 수 있는지, 그리고 그 성능과 윤리 사이의 트레이드오프 관계를 극명하게 드러냈다. 이 연구는 기술 커뮤니티가 자신들의 창조물이 갖는 사회적 책임에 대해 본격적으로 논의하게 만드는 중요한 계기가 되었다.



## 5. 결론



DeepFace는 단순히 LFW 벤치마크의 정확도를 몇 퍼센트 끌어올린 연구로 평가될 수 없다. 이 연구는 정교한 3D 정렬, 깊은 신경망, 그리고 대규모 데이터라는 세 가지 축을 유기적으로 결합하여, 얼굴 인식을 인간의 직관과 규칙에 의존하는 '공학의 영역'에서 데이터로부터 표현을 스스로 학습하는 '학습의 영역'으로 완전히 전환시킨 '패러다임 전환자(paradigm shifter)'였다. 이는 얼굴 인식 분야의 역사에서 하나의 '특이점(singularity)'으로 기록될 만하다.

기술적으로 DeepFace가 제시한 분류 기반의 특징 학습 접근법은 그 자체의 한계에도 불구하고, FaceNet의 '거리 학습'과 ArcFace의 '기하학적 최적화'로 이어지는 후속 혁신들의 중요한 출발점이 되었다. DeepFace가 풀어낸 문제와 그 방식이 남긴 과제들이 있었기에, 후속 연구들은 더 본질적이고 정교한 방향으로 나아갈 수 있었다.

그러나 DeepFace의 가장 중요한 유산은 기술적 성취를 넘어 사회적 담론에 있다. 이 연구는 기술적 가능성의 문을 활짝 연 동시에, 알고리즘의 공정성, 데이터 프라이버시, 그리고 대규모 감시 사회라는 판도라의 상자를 열었다. 따라서 미래의 얼굴 인식 연구는 단순히 정확도를 100%에 가깝게 만드는 기술적 도전을 넘어, 알고리즘의 편향을 완화하고, 개인의 프라이버시를 보호하며, 기술의 오남용을 방지할 수 있는 사회적, 제도적, 그리고 기술적 안전장치를 함께 개발해야 하는 무거운 책임을 안게 되었다. DeepFace의 진정한 유산은 우리에게 그 책임을 상기시키는 데 있다.



#### **참고 자료**

1. DeepFace: Closing the Gap to Human-Level Performance in Face ..., 8월 18, 2025에 액세스, https://www.cs.toronto.edu/~ranzato/publications/taigman_cvpr14.pdf
2. Face recognition using deep learning methods a review - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/345187326_Face_recognition_using_deep_learning_methods_a_review
3. Surpassing Human-Level Face Verification Performance on LFW with GaussianFace - AAAI Publications, 8월 18, 2025에 액세스, https://ojs.aaai.org/index.php/AAAI/article/view/9797/9656
4. Labelled Faces in the Wild (LFW) Dataset - Kaggle, 8월 18, 2025에 액세스, https://www.kaggle.com/datasets/jessicali9530/lfw-dataset
5. DeepFace: Closing the Gap to Human-Level Performance in Face Verification - Meta AI, 8월 18, 2025에 액세스, https://ai.meta.com/research/publications/deepface-closing-the-gap-to-human-level-performance-in-face-verification/
6. Face verification performance on LFW by humans is almost perfect ( 99 - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/Face-verification-performance-on-LFW-by-humans-is-almost-perfect-99-20-when-people_fig4_224223972
7. Face Recognition Using LBP, FLD and SVM with Single Training Sample Per Person - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/profile/Mustafa-Al-Dabagh/publication/319482465_Face_Recognition_Using_LBP_FLD_and_SVM_with_Single_Training_Sample_Per_Person/links/59ae6408aca272f8a167aa8e/Face-Recognition-Using-LBP-FLD-and-SVM-with-Single-Training-Sample-Per-Person.pdf
8. DEEP LEARNING FOR FACE RECOGNITION: A CRITICAL ANALYSIS - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/pdf/1907.12739
9. FISHER VECTOR ENCODED DEEP CONVOLUTIONAL FEATURES FOR UNCONSTRAINED FACE VERIFICATION, 8월 18, 2025에 액세스, https://engineering.jhu.edu/vpatel36/wp-content/uploads/2018/08/ICIP_FVDCNN_2016_camera_ready_final.pdf
10. Fisher Vector Faces in the Wild - BMVA Archive, 8월 18, 2025에 액세스, https://www.bmva-archive.org.uk/bmvc/2013/Papers/paper0008/paper0008.pdf
11. landmark-based fisher vector representation for, 8월 18, 2025에 액세스, https://engineering.jhu.edu/vpatel36/wp-content/uploads/2018/08/FV_icip2015.pdf
12. Past, Present, and Future of Face Recognition: A Review - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2079-9292/9/8/1188
13. DeepFace: Closing the Gap to Human-Level Performance in Face Verification - SciSpace, 8월 18, 2025에 액세스, https://scispace.com/papers/deepface-closing-the-gap-to-human-level-performance-in-face-3wcmadzjn3
14. DeepFace: Closing the Gap to Human-Level Performance in Face Verification | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/263564119_DeepFace_Closing_the_Gap_to_Human-Level_Performance_in_Face_Verification
15. FaceNet: A Unified Embedding for Face Recognition and Clustering - The Computer Vision Foundation, 8월 18, 2025에 액세스, https://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Schroff_FaceNet_A_Unified_2015_CVPR_paper.pdf
16. FaceNet: A Unified Embedding for Face Recognition and Clustering, 8월 18, 2025에 액세스, http://arxiv.org/pdf/1503.03832
17. Triplet loss - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Triplet_loss
18. Triplet Loss: Intro, Implementation, Use Cases - V7 Labs, 8월 18, 2025에 액세스, https://www.v7labs.com/blog/triplet-loss
19. FaceNet: A Unified Embedding for Face Recognition and Clustering - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/273471270_FaceNet_A_Unified_Embedding_for_Face_Recognition_and_Clustering
20. VGG Face - Exposing.ai, 8월 18, 2025에 액세스, https://exposing.ai/vgg_face/
21. ox-vgg/vgg_face2 - GitHub, 8월 18, 2025에 액세스, https://github.com/ox-vgg/vgg_face2
22. [1710.08092] VGGFace2: A dataset for recognising faces across pose and age - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/1710.08092
23. VGGFace2 Dataset - Papers With Code, 8월 18, 2025에 액세스, https://paperswithcode.com/dataset/vggface2-1
24. (PDF) ArcFace: Additive Angular Margin Loss for Deep Face ..., 8월 18, 2025에 액세스, https://www.researchgate.net/publication/322674945_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition
25. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/352263965_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition
26. Sub-center ArcFace: Boosting Face Recognition by Large-scale Noisy Web Faces, 8월 18, 2025에 액세스, https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123560715.pdf
27. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - InsightFace, 8월 18, 2025에 액세스, https://insightface.ai/arcface
28. ArcFace: Additive Angular Margin Loss for Deep Face Recognition | by Mostafa Gazar | 1-minute papers | Medium, 8월 18, 2025에 액세스, https://medium.com/1-minute-papers/arcface-additive-angular-margin-loss-for-deep-face-recognition-d02297605f8d
29. Ethical Considerations in the Use of Facial Recognition for Public Safety, 8월 18, 2025에 액세스, https://publicsafety.ieee.org/topics/ethical-considerations-in-the-use-of-facial-recognition-for-public-safety/
30. Ethics of Facial Recognition: Key Issues and Solutions, 8월 18, 2025에 액세스, https://learn.g2.com/ethics-of-facial-recognition
31. Face recognition detection based on deep learning - Combinatorial Press, 8월 18, 2025에 액세스, [https://combinatorialpress.com/article/jcmcc/Volume%20127/127bp2/Face%20recognition%20detection%20based%20on%20deep%20learning.pdf](https://combinatorialpress.com/article/jcmcc/Volume 127/127bp2/Face recognition detection based on deep learning.pdf)


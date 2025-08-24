# 대조 학습 (Contrastive Learning)
[대조 학습(Contrastive Learning)](./index.md)



지난 10년간 딥러닝의 발전은 대규모 레이블 데이터셋에 기반한 지도 학습(Supervised Learning) 패러다임에 의해 주도되어 왔다. 그러나 이러한 성공은 막대한 양의 데이터를 수동으로 주석 처리해야 하는 비용과 시간의 문제를 야기했으며, 이는 특정 도메인에서의 모델 확장성을 저해하는 근본적인 한계로 작용했다.1 이러한 배경 속에서, 데이터 자체로부터 감독 신호(supervisory signal)를 생성하여 레이블에 대한 의존성을 극복하려는 자기 지도 학습(Self-Supervised Learning, SSL)이 표현 학습(representation learning)의 새로운 패러다임으로 부상했다.3 SSL은 레이블이 없는 방대한 양의 비정형 데이터를 활용하여, 특정 다운스트림 작업(downstream task)에 유용하게 전이될 수 있는 일반적인 표현을 학습하는 것을 목표로 한다.5

SSL의 발전은 크게 두 가지 주요 흐름으로 나눌 수 있다. 첫째는 생성적 방법(generative methods)으로, 데이터의 일부를 마스킹하고 이를 예측하거나(e.g., Masked Autoencoders), 입력 데이터 전체를 재구성(reconstruction)하는 것을 목표로 한다.7 둘째는 대조적 방법(contrastive methods)으로, 이 보고서의 핵심 주제이며, 데이터 샘플 간의 관계를 학습하는 데 초점을 맞춘다.8


대조 학습의 근본 철학은 '비교를 통한 학습'이라는 직관적인 개념에 뿌리를 두고 있다.9 이는 인간이 세상을 이해하는 방식과 유사하다. 예를 들어, 우리는 수달이나 그리즐리 곰이 무엇인지 명확히 알지 못더라도, 여러 사진을 비교함으로써 어떤 사진들이 같은 동물을 보여주는지 구별할 수 있다.10 대조 학습은 이러한 원리를 기계 학습 모델에 적용한다.

핵심 목표는 고차원의 임베딩 공간(embedding space) 내에서 의미론적으로 유사한 샘플 쌍, 즉 '긍정적 쌍(positive pairs)'의 표현은 서로 가깝게 끌어당기고, 유사하지 않은 샘플 쌍인 '부정적 쌍(negative pairs)'의 표현은 서로 멀리 밀어내는 것이다.1 이처럼 샘플들을 서로 대조(contrast)하여 판별적인(discriminative) 표현을 학습하는 것이 대조 학습의 정수이다.9 이러한 접근법은 모델이 레이블 없이도 데이터의 내재적인 의미 구조를 파악하도록 유도하며, 결과적으로 모델의 견고성(robustness)과 일반화(generalization) 성능을 향상시킨다.1


대조 학습의 특성을 명확히 이해하기 위해서는 SSL의 다른 한 축인 생성 모델과의 비교가 필수적이다. 이 두 접근법의 차이는 단순히 기술적인 차이를 넘어, 표현 학습의 목표에 대한 근본적인 관점 차이를 드러낸다.

- **학습 목표:** 생성 모델(e.g., Autoencoders, GANs, MAE)의 목표는 입력 데이터의 분포를 학습하여 이를 재구성하거나 새로운 데이터를 생성하는 것이다.7 이는 '이 데이터가 어떻게 생겼는가?'라는 질문에 답하는 과정이며, 종종 픽셀 수준의 저수준 디테일까지 포착해야 하는 계산적으로 부담이 큰 작업이다.7 반면, 대조 학습의 목표는 판별이다. 즉, '이 데이터가 다른 데이터와 어떻게 다른가?'라는 질문에 답하며, 샘플들을 구별하는 데 필요한 고수준의 추상적이고 의미론적인 표현을 학습하는 데 집중한다.13
- **학습 신호와 출력:** 생성 모델은 주로 재구성 오차(reconstruction error)나 가능도(likelihood)와 같은 손실 함수를 입력 공간에서 직접 계산한다.13 모델의 출력은 원본 데이터와 같은 형태의 새로운 데이터(e.g., 이미지, 텍스트)이다. 이와 대조적으로, 대조 학습은 표현 공간(representation space)에서 작동하는 대조 손실(e.g., InfoNCE, Triplet loss)을 사용하며, 모델의 최종 출력은 다운스트림 작업에 사용될 임베딩 벡터이다.13
- **일반화 특성:** 생성 모델은 데이터의 모든 세부 사항을 재현하려 하기 때문에 때로는 훈련 데이터의 미세한 편향이나 노이즈에 과적합될 수 있다.13 반면, 대조 학습은 샘플 간의 본질적인 차이에 집중하고 저수준의 노이즈는 무시하는 경향이 있어, 보다 일반화 가능하고 다양한 다운스트림 작업에 효과적으로 전이될 수 있는 특징을 학습하는 데 유리하다.1

이러한 차이점은 AI 연구의 패러다임 변화를 시사한다. 초기 딥러닝이 대규모 레이블 데이터셋을 통해 '정답 맞히기'에 집중했다면, SSL의 등장은 레이블 없는 데이터의 잠재력을 활용하여 '관계 이해하기'로 초점을 이동시켰다. 이 과정에서, '관계'를 직접적으로 모델링하는 대조적 접근법이 특정 작업에 얽매이지 않는 범용적인 표현을 학습하는 데 있어 생성적 접근법보다 더 효율적이고 효과적임이 여러 연구를 통해 입증되면서, 현대 표현 학습의 주류로 자리매김하게 되었다.



대부분의 현대 대조 학습 모델은 공통적인 아키텍처 프레임워크를 공유한다. 이는 일반적으로 두 개의 동일한 가중치를 공유하는 샴 네트워크(Siamese network) 구조를 기반으로 하며, 네 가지 핵심 구성 요소로 이루어진다.11

1. **데이터 증강(Data Augmentation) 모듈:** 하나의 원본 데이터 샘플로부터 의미론적으로는 동일하지만 외형적으로는 다른 두 개의 '뷰(view)'를 생성한다. 이 두 뷰가 '긍정적 쌍'을 형성한다.16
2. **인코더(Encoder) 네트워크:** 증강된 뷰들을 입력받아 고차원의 표현 벡터 $h$를 추출하는 심층 신경망이다. 주로 이미지 분야에서는 ResNet이나 Vision Transformer(ViT)가, 자연어 처리에서는 BERT와 같은 Transformer 기반 모델이 사용된다.4
3. **프로젝션 헤드(Projection Head) 네트워크:** 인코더의 출력 $h$를 대조 손실이 계산되는 더 낮은 차원의 잠재 공간(latent space)으로 매핑하여 최종 표현 $z$를 생성한다. 일반적으로 작은 다층 퍼셉트론(MLP)으로 구성된다.18
4. **대조 손실(Contrastive Loss) 함수:** 잠재 공간 상에서 긍정적 쌍의 표현 $z$는 가깝게, 부정적 쌍의 표현은 멀어지도록 모델의 가중치를 업데이트하는 목적 함수이다.

학습이 완료된 후, 다운스트림 작업에는 프로젝션 헤드 $g(·)$는 버리고 인코더 $f(·)$의 출력인 표현 $h$를 사용한다.18 이 분리된 구조는 표현의 질을 향상시키는 데 결정적인 역할을 한다. 프로젝션 헤드의 존재는 '정보 손실 가설(Information Loss Hypothesis)'을 뒷받침하는 것으로 해석될 수 있다. 즉, 대조 손실을 계산하는 잠재 공간 

$z$는 데이터 증강에 불변인 특징(e.g., 색상, 방향)만을 남기도록 강제된다. 이 과정에서 증강에 의해 변형된 정보는 손실될 수 있다.19 그러나 이러한 정보는 일부 다운스트림 작업(e.g., 빨간 차와 노란 차 구분)에는 매우 중요할 수 있다.20 따라서 프로젝션 헤드는 대조 작업을 위한 '희생양' 역할을 수행한다. 

$z = g(h)$를 통해 증강 불변성을 학습하는 동안, 그 이전의 표현 공간 $h$에는 색상, 질감, 방향 등 다운스트림 작업에 유용할 수 있는 더 풍부한 정보가 보존된다. 이로 인해 $h$는 더 일반적이고 다양한 작업에 효과적인 표현이 된다. 이는 대조 학습 프레임워크가 의도적으로 정보 병목 현상을 만들어 표현의 계층을 분리하는 정교한 메커니즘임을 시사한다.


데이터 증강은 긍정적 쌍을 생성하는 핵심적인 과정으로, 모델이 학습해야 할 불변성(invariance)을 정의하는 역할을 한다. 어떤 종류의 변환에 대해 모델이 불변해야 하는지를 가르침으로써, 학습되는 표현의 질을 결정한다.17


컴퓨터 비전 분야에서 데이터 증강은 모델이 색상 히스토그램이나 특정 픽셀 값과 같은 저수준의 피상적인 특징에 과적합되는 것을 방지하고, 객체의 형태, 구조, 질감과 같은 본질적인 특징에 집중하도록 강제하는 역할을 한다.9 SimCLR 논문에서는 다양한 증강 기법을 체계적으로 실험한 결과, **무작위 자르기 및 리사이징(random cropping and resizing)**과 **색상 왜곡(color distortion)**의 조합이 효과적인 표현 학습에 필수적임을 밝혔다.9 무작위 자르기만 사용할 경우, 모델은 단순히 이미지의 색상 분포를 보고 같은 이미지를 식별하려는 '꼼수'를 학습할 수 있기 때문에, 색상 왜곡을 통해 이러한 지름길(shortcut)을 차단해야 한다.16

이 외에도 일반적으로 사용되는 증강 기법은 다음과 같다 9:

- **기하학적 변환:** 무작위 회전(Rotation), 좌우 및 상하 뒤집기(Flipping), 아핀 변환(Affine transformation).
- **외형 변환:** 가우시안 블러(Gaussian blur), 소벨 필터링(Sobel filtering).
- **노이즈 추가:** 가우시안 노이즈(Gaussian noise), 소금-후추 노이즈(Salt-and-pepper noise).

증강의 강도와 종류는 매우 중요한 하이퍼파라미터이며, 도메인과 데이터셋의 특성에 따라 최적의 전략이 달라질 수 있다.17 최근에는 GAN이나 확산 모델(diffusion model)을 이용해 원본 이미지와 의미적으로 유사하면서도 외형적으로는 더 다양한 합성 이미지를 생성하여 긍정적 쌍으로 활용하려는 연구도 활발히 진행되고 있다.22


텍스트 데이터에 대한 증강은 원본 문장의 의미를 보존하면서 변형을 가해야 하므로 이미지보다 더 까다로운 과제이다.23 자연어 처리(NLP) 분야에서 사용되는 주요 증강 기법은 다음과 같다:

- **역번역(Back-translation):** 원본 문장을 다른 언어로 번역했다가 다시 원래 언어로 번역하여 의미는 유사하지만 어휘나 구조가 다른 문장을 생성한다.23
- **어휘 편집(Lexical Edits):** EDA(Easy Data Augmentation) 기법으로 알려져 있으며, 동의어 대체(Synonym Replacement), 무작위 단어 삽입(Random Insertion), 교환(Random Swap), 삭제(Random Deletion) 등을 포함한다.23
- **드롭아웃(Dropout):** SimCSE 모델은 별도의 증강 기법 없이, Transformer 모델의 내장된 드롭아웃을 노이즈로 활용하는 혁신적인 방법을 제안했다. 동일한 입력 문장을 인코더에 두 번 통과시키되, 매번 다른 드롭아웃 마스크가 적용되도록 하여 미세하게 다른 임베딩을 생성하고 이를 긍정적 쌍으로 간주한다. 이는 매우 간단하면서도 강력한 증강 기법으로 입증되었다.3


대조 학습의 목표는 손실 함수를 통해 수학적으로 정의된다. 다양한 손실 함수가 제안되었지만, 크게 마진 기반 손실과 정보 이론 기반 손실로 나눌 수 있다.


Triplet Loss는 대조 학습 초기에 널리 사용된 손실 함수로, 앵커(anchor, $s_a$), 긍정적 샘플(positive, $s_+$), 부정적 샘플(negative, $s_-$)로 구성된 삼중항(triplet)을 기반으로 한다.9 이 손실 함수의 목표는 임베딩 공간에서 앵커와 긍정적 샘플 간의 거리($d(s_a, s_+)$)가 앵커와 부정적 샘플 간의 거리($d(s_a, s_-)$)보다 특정 마진($m$) 이상 작아지도록 강제하는 것이다.11

수식은 다음과 같이 표현된다 27:
$$
L = max(d(s_a, s_+) - d(s_a, s_-) + m, 0)
$$
여기서 $d(·, ·)$는 유클리드 거리와 같은 거리 함수이다. 손실은 $d(s_a, s_+) + m > d(s_a, s_-)$일 때만 발생하며, 이는 모델이 이미 충분히 잘 분리된 샘플에는 집중하지 않고 구별하기 어려운 샘플에 집중하도록 유도한다. 따라서 성능을 극대화하기 위해서는 앵커와 구별하기 어려운 '어려운 부정적 샘플(hard negatives)'을 효과적으로 샘플링하는 것이 매우 중요하다.9 하지만 한 번에 하나의 부정적 샘플만 고려하기 때문에, 배치 내의 풍부한 정보를 활용하는 InfoNCE 계열 손실 함수에 비해 정보 효율성이 떨어질 수 있다.


InfoNCE(Information Noise-Contrastive Estimation)는 자기 지도 학습에서 가장 널리 사용되는 손실 함수 중 하나로, 학습 문제를 하나의 긍정적 샘플을 여러 개의 부정적 샘플과 구별하는 다중 클래스 분류 문제로 재구성한다.27 이 손실 함수는 앵커와 긍정적 샘플 간의 상호 정보량(Mutual Information)의 하한(lower bound)을 최대화하는 것과 이론적으로 연결되어, 표현 학습에 대한 강력한 이론적 기반을 제공한다.29

SimCLR 프레임워크에서 제안된 NT-Xent(Normalized Temperature-scaled Cross Entropy) Loss는 InfoNCE의 구체적인 구현 형태이다. 미니배치 내에 $N$개의 원본 이미지가 있고, 각각 2개의 증강 뷰를 생성하면 총 $2N$개의 샘플이 존재한다. 이 때, 하나의 샘플 $i$에 대해, 동일한 원본 이미지에서 파생된 다른 뷰 $j$는 유일한 긍정적 샘플이 되고, 나머지 $2(N-1)$개의 샘플은 모두 부정적 샘플이 된다.14

긍정적 쌍 $(i, j)$에 대한 NT-Xent 손실 함수는 다음과 같이 정의된다:
$$
L_{i,j} = -log \frac{\exp(\text{sim}(z_i, z_j)/\tau)}{\sum_{k=1}^{2N} \mathbb{1}_{[k \neq i]} \exp(\text{sim}(z_i, z_k)/\tau)}
$$
여기서 $sim(u, v) = u^T v / (||u|| ||v||)$는 코사인 유사도, $τ$는 온도(temperature) 하이퍼파라미터, $1_{[k!= i]}$는 $k!= i$일 때 1인 지시 함수(indicator function)이다.14 이 손실 함수는 본질적으로 긍정적 쌍에 대한 소프트맥스 확률을 최대화하는 것과 같다.


온도 파라미터 $τ$는 대조 손실 함수의 동작을 미세 조정하는 매우 중요한 역할을 한다.32

- **Hardness-Aware 속성:** 온도는 모델이 '어려운 부정적 샘플'(앵커와 유사도가 높은 부정적 샘플)에 얼마나 집중할지를 결정한다. $τ$ 값이 낮을수록, 유사도가 높은(즉, 구별하기 어려운) 부정적 샘플에 대한 손실 값과 그래디언트가 기하급수적으로 커진다. 이는 모델이 쉬운 샘플은 무시하고 어려운 샘플 간의 미세한 차이를 학습하도록 강제하는 'hardness-aware' 특성을 부여한다.34
- **균일성(Uniformity):** 임베딩이 단위 초구(unit hypersphere) 상에 얼마나 고르게 분포되어 있는지를 나타내는 균일성은 표현의 질을 평가하는 중요한 척도이다. 낮은 온도 $τ$는 어려운 부정적 샘플을 강하게 밀어내는 효과 때문에, 임베딩이 특정 영역에 뭉치지 않고 공간 전체에 더 균일하게 퍼지도록 만든다. 이는 표현의 분별력을 높여 다운스트림 작업 성능 향상에 기여한다.34
- **균일성-관용성 딜레마(Uniformity-Tolerance Dilemma):** 그러나 온도를 너무 낮추면 부작용이 발생할 수 있다. 의미론적으로는 유사하지만 다른 인스턴스인 샘플(e.g., 다른 품종의 개 이미지)까지 과도하게 밀어내어, 데이터의 내재적인 의미 구조를 해칠 수 있다. 이는 대조 학습이 직면하는 '균일성-관용성 딜레마'로, 높은 균일성을 추구할수록 의미적으로 유사한 샘플에 대한 관용성(tolerance)이 낮아지는 상충 관계를 의미한다. 따라서, 최적의 $τ$ 값을 찾는 것은 이 두 가지 속성 사이의 균형을 맞추는 중요한 과정이다.34


대조 학습의 기본 원리를 바탕으로, 다양한 아키텍처들이 제안되었다. 이들은 크게 부정적 쌍을 명시적으로 활용하는 대조적 방법론과, 이를 회피하려는 비-대조적 방법론으로 나눌 수 있다. 또한, 레이블 정보를 활용하여 성능을 더욱 끌어올리는 지도 대조 학습도 중요한 흐름이다.


이 방법론들은 긍정적 쌍을 끌어당기는 힘과 함께 부정적 쌍을 밀어내는 반발력을 학습의 핵심 동력으로 사용한다.


SimCLR(A Simple Framework for Contrastive Learning of Visual Representations)은 이름에서 알 수 있듯이, 복잡한 아키텍처적 장치(e.g., 메모리 뱅크, 모멘텀 인코더) 없이 대조 학습의 핵심 요소만으로 높은 성능을 달성할 수 있음을 보여준 선구적인 모델이다.18 SimCLR의 성공은 세 가지 핵심 요소의 시너지 효과에 기인한다 18:

1. **강력한 데이터 증강 조합:** 앞서 언급했듯이, 무작위 자르기와 색상 왜곡의 조합이 효과적인 표현 학습에 결정적임을 밝혔다.
2. **비선형 프로젝션 헤드:** 인코더 출력과 대조 손실 계산 사이에 비선형 MLP를 추가하는 것이 표현의 질을 크게 향상시킴을 입증했다.
3. **대규모 배치 사이즈와 긴 학습:** SimCLR는 성능을 극대화하기 위해 4096 또는 8192와 같은 매우 큰 배치 사이즈를 사용한다.38 큰 배치는 미니배치 내에서 더 많고 다양한 부정적 샘플을 제공하여, 모델이 더 견고하고 일반화된 표현을 학습하도록 돕는다.14 손실 함수로는 NT-Xent를 사용한다.

SimCLR의 단순함과 강력함은 이후 연구에 큰 영향을 미쳤지만, 막대한 계산 자원을 요구하는 대규모 배치 사이즈 의존성은 실용적인 적용에 큰 장벽이 되었다.


MoCo(Momentum Contrast)는 SimCLR의 대규모 배치 사이즈 문제를 해결하기 위해 제안된 아키텍처이다.40 GPU 메모리 제약에서 벗어나, 효율적으로 대규모의 일관된 부정적 샘플을 활용하는 것이 핵심 목표이다. 이를 위해 두 가지 독창적인 메커니즘을 도입했다 42:

1. **Dictionary as a Queue:** 부정적 샘플들을 저장하는 딕셔너리를 큐(queue) 자료구조로 구현한다. 각 학습 스텝에서 현재 미니배치의 인코딩된 키(key)들은 큐에 추가(enqueue)되고, 가장 오래된 배치의 키들은 제거(dequeue)된다. 이를 통해 딕셔너리의 크기를 미니배치 크기와 분리하여, 작은 배치 사이즈로 학습하면서도 수만 개의 부정적 샘플을 유지할 수 있다.41

2. **Momentum Encoder:** 큐에 저장된 키들의 표현 일관성(consistency)을 유지하는 것이 중요하다. 큐의 키들은 이전 스텝들에서 서로 다른 버전의 인코더에 의해 생성되었기 때문이다. 만약 키 인코더($f_k$)가 쿼리 인코더($f_q$)와 함께 매 스텝마다 그래디언트 업데이트를 받는다면, 큐 안의 키들이 급격히 변해 일관성이 깨질 것이다. 이를 방지하기 위해, MoCo는 키 인코더를 직접 학습하는 대신, 쿼리 인코더의 가중치를 모멘텀 기반의 지수 이동 평균(Exponential Moving Average, EMA)으로 매우 느리게 업데이트한다. 수식은 다음과 같다:
   $$
   θ_k \larr m \theta_k + (1-m) \theta_q
   $$
   여기서 $\theta_k$와 $\theta_q$는 각각 키 인코더와 쿼리 인코더의 파라미터이며, $m$은 높은 값(e.g., 0.999)을 갖는 모멘텀 계수이다.40 이 모멘텀 업데이트 덕분에 키 인코더는 매우 부드럽게 변화하며, 큐에 저장된 키들의 표현 일관성이 유지된다.42

MoCo는 이 두 가지 장치를 통해 계산 효율성과 성능 사이의 균형을 성공적으로 맞추었으며, 이후 MoCo v2, v3 등으로 발전하며 대조 학습 연구의 중요한 축을 형성했다.


부정적 쌍을 사용하는 방법론은 '거짓 음성' 문제와 같은 잠재적 위험을 내포하고 있으며, 대규모 부정적 샘플을 처리하기 위한 복잡성을 야기한다. 이에 부정적 쌍 없이 긍정적 쌍만으로 학습하려는 시도들이 등장했으며, 이들은 '표현 붕괴(representation collapse)'라는 새로운 문제를 해결해야 했다.


BYOL은 부정적 쌍을 전혀 사용하지 않고도 SOTA 성능을 달성하여 학계에 큰 충격을 주었다.44 BYOL의 핵심은 두 개의 비대칭적인 네트워크를 통해 한 뷰의 표현으로 다른 뷰의 표현을 예측하는 자기 증류(self-distillation) 방식이다.

- **Online & Target Network:** BYOL은 온라인(online) 네트워크와 타겟(target) 네트워크라는 두 개의 네트워크를 사용한다. 온라인 네트워크는 학습을 통해 가중치가 업데이트되며, 한 증강 뷰를 입력받아 다른 증강 뷰의 타겟 표현을 예측하도록 훈련된다. 타겟 네트워크는 타겟 표현을 생성하며, 직접적인 그래디언트 업데이트를 받지 않는다.44
- **붕괴 방지 메커니즘:** 부정적 쌍의 반발력 없이 붕괴를 방지하기 위해 BYOL은 두 가지 핵심 장치를 사용한다 44:
  1. **Slow-Moving Average Update:** 타겟 네트워크의 가중치는 온라인 네트워크의 가중치를 EMA 방식으로 매우 느리게 업데이트한다 (MoCo의 모멘텀 인코더와 유사). 이는 온라인 네트워크가 예측해야 할 타겟이 급격히 변하지 않도록 하여 학습을 안정시킨다.44
  2. **Prediction Head:** 온라인 네트워크에만 추가적인 예측 헤드(prediction head)를 두어 온라인과 타겟 네트워크 간의 구조적 비대칭성을 만든다. 이 비대칭 구조가 모델이 상수 벡터를 출력하는 자명한 해로 수렴하는 것을 막는 데 결정적인 역할을 한다.

이 두 가지 메커니즘의 조합을 통해 BYOL은 부정적 쌍 없이도 효과적으로 표현 붕괴를 방지하고 강력한 표현을 학습할 수 있음을 보여주었다.


SwAV는 개별 샘플들을 직접 비교하는 대신, 이들을 소수의 '프로토타입(prototypes)'으로 클러스터링하고, 클러스터 할당의 일관성을 학습하는 독특한 접근법을 취한다.47

- **온라인 클러스터링:** 학습 과정 중에 미니배치 내 데이터의 임베딩을 K개의 학습 가능한 프로토타입 벡터(클러스터 중심)에 온라인으로 할당한다. 각 이미지 뷰는 특정 클러스터에 속한다는 '코드(code)'를 부여받는다.49
- **Swapped Prediction:** SwAV의 핵심 손실 함수는 '교환된 예측(swapped prediction)'이다. 이미지의 한 뷰에서 얻은 임베딩 $z_1$을 사용하여, 다른 뷰가 할당된 코드 $q_2$를 예측하고, 그 반대 과정($z_2$로 $q_1$ 예측)도 동시에 수행한다. 손실 함수는 $L = l(z_1, q_2) + l(z_2, q_1)$ 형태로, 동일한 이미지에서 파생된 뷰들은 동일한 프로토타입(클러스터)에 할당되어야 한다는 일관성을 강제한다.47
- **균등 분할(Equipartition):** 모든 임베딩이 하나의 프로토타입으로 붕괴되는 것을 막기 위해, Sinkhorn-Knopp 알고리즘을 사용하여 각 프로토타입에 할당되는 샘플의 수가 거의 균등하도록 제약을 가한다. 이는 부정적 쌍의 반발력과 유사한 역할을 수행한다.50

SwAV는 개별 인스턴스 비교에서 클러스터 수준의 비교로 관점을 전환함으로써, 배치 사이즈에 덜 민감하고 효율적인 학습을 가능하게 했다.


지도 대조 학습은 기존의 레이블 정보를 대조 학습 프레임워크에 통합하여, 자기 지도 학습의 장점과 지도 학습의 정확성을 결합하려는 시도이다.


기존 지도 학습의 표준인 교차 엔트로피(Cross-Entropy) 손실은 각 샘플을 독립적으로 분류하여 클래스 간의 관계를 명시적으로 모델링하지 않는다. 반면, 지도 대조 학습(SCL)은 레이블 정보를 활용하여 긍정적/부정적 쌍의 정의를 확장한다.51

- **쌍 구성:** 앵커 샘플이 주어졌을 때, 동일한 클래스에 속하는 다른 모든 샘플들은 긍정적 샘플로, 다른 클래스에 속하는 모든 샘플들은 부정적 샘플로 간주된다.1
- **SupCon Loss:** 이 손실 함수는 임베딩 공간에서 동일 클래스에 속하는 샘플들의 클러스터를 더 조밀하게 만들고, 다른 클래스의 클러스터들은 서로 멀리 밀어내는 것을 목표로 한다.51
- **한계 (Intra-class Repulsion):** 그러나 SupCon 손실 함수의 표준적인 공식은 분모의 정규화 항에 앵커와 동일한 클래스에 속하는 다른 긍정적 샘플들을 포함한다. 이로 인해 이들 긍정적 샘플들이 부정적 샘플처럼 취급되어 앵커로부터 밀려나는 '클래스 내 반발'이라는 의도치 않은 현상이 발생할 수 있다. 이는 학습된 표현의 클래스 내 분산을 증가시켜 성능을 저해할 수 있다.28


SINCERE(Supervised InfoNCE REvisited) Loss는 SupCon의 클래스 내 반발 문제를 해결하기 위해 제안된 개선된 손실 함수이다.28 SINCERE는 InfoNCE의 근본 원칙, 즉 "타겟 분포(긍정적 샘플)에 속하는 것으로 알려진 샘플은 노이즈 분포(부정적 샘플)의 일부로 취급되어서는 안 된다"는 원칙을 지도 학습 환경에 일관되게 적용한다.28

이를 위해 SINCERE 손실 함수는 분모의 부정적 샘플 집합에서 앵커와 동일한 클래스에 속하는 다른 모든 긍정적 샘플들을 명시적으로 제외한다. 이 간단한 수정을 통해 클래스 내 반발 현상을 원천적으로 제거하고, 더 명확하게 클래스 간 분리를 학습할 수 있도록 한다.28


아래 표는 앞서 논의된 주요 대조 및 비-대조 학습 모델들의 핵심적인 특징을 요약하여 비교한다. 각 모델의 철학과 기술적 선택이 어떻게 다른 장단점과 적용 시나리오로 이어지는지를 한눈에 파악할 수 있다.

| 특징                   | SimCLR              | MoCo                    | BYOL                             | SwAV                          |
| ---------------------- | ------------------- | ----------------------- | -------------------------------- | ----------------------------- |
| **핵심 아이디어**      | 대규모 배치 내 대조 | 모멘텀 인코더 & 큐      | 자기 예측 및 증류                | 온라인 클러스터링             |
| **부정적 쌍 활용**     | 예 (배치 내 샘플)   | 예 (큐 기반 딕셔너리)   | 아니오                           | 아니오 (암시적)               |
| **붕괴 방지 전략**     | 부정적 쌍의 반발력  | 부정적 쌍의 반발력      | 비대칭 구조 (예측기) + EMA       | 온라인 클러스터링 + 균등 분할 |
| **배치 사이즈 의존성** | 매우 높음           | 낮음                    | 낮음                             | 낮음                          |
| **주요 구성요소**      | NT-Xent Loss        | Momentum Encoder, Queue | Online/Target Network, Predictor | Prototypes, Sinkhorn-Knopp    |
| **손실 함수**          | NT-Xent             | InfoNCE                 | MSE                              | Swapped Prediction Loss       |


대조 학습의 성공은 컴퓨터 비전을 넘어 자연어 처리(NLP)와 이미지-텍스트와 같은 멀티모달(multi-modal) 영역으로 빠르게 확장되었다. 이는 대조 학습 프레임워크의 유연성과 범용성을 입증한다.


SimCSE(Simple Contrastive Learning of Sentence Embeddings)는 기존 문장 임베딩 방법론의 한계를 극복하기 위해 대조 학습을 적용한, 간단하면서도 매우 효과적인 프레임워크이다.25 SimCSE는 비지도 및 지도 두 가지 방식으로 제안되었다.


비지도 SimCSE는 텍스트 데이터 증강의 어려움을 독창적인 방식으로 해결했다. 별도의 복잡한 증강 파이프라인 없이, 동일한 입력 문장을 사전 훈련된 언어 모델(e.g., BERT, RoBERTa) 인코더에 두 번 독립적으로 통과시킨다. Transformer 아키텍처에 내장된 표준 드롭아웃(dropout)은 매번 다른 뉴런을 비활성화하므로, 결과적으로 동일한 입력에 대해 미세하게 다른 두 개의 임베딩 $(h_i, h_i')$이 생성된다. 이 두 임베딩을 긍정적 쌍으로 간주하고, 미니배치 내의 다른 문장들로부터 얻은 임베딩을 부정적 쌍으로 사용하여 대조 학습을 수행한다.25 이 방식은 드롭아웃이 최소한의 데이터 증강 역할을 효과적으로 수행할 수 있음을 보여주었으며, 드롭아웃을 제거하면 표현 붕괴가 발생함을 실험적으로 입증했다.25


지도 SimCSE는 자연어 추론(Natural Language Inference, NLI) 데이터셋을 대조 학습에 활용하여 성능을 극대화한다. NLI 데이터셋은 전제(premise)와 가설(hypothesis) 문장 쌍, 그리고 그들 간의 관계(함의, 모순, 중립) 레이블로 구성된다.

- **긍정적 쌍:** 전제 문장과 '함의(entailment)' 관계에 있는 가설 문장은 의미적으로 유사하므로, 이들을 긍정적 쌍으로 사용한다 (e.g., (premise, entailment_hypothesis)).
- **어려운 부정적 쌍(Hard Negatives):** 전제 문장과 '모순(contradiction)' 관계에 있는 가설 문장은 어휘적으로는 겹치는 부분이 많아 유사해 보이지만 의미적으로는 상반된다. 이러한 쌍을 부정적 쌍으로 사용하면, 모델이 단순한 단어 중복을 넘어 문장의 미묘한 의미 차이를 학습하도록 강제하는 매우 효과적인 '어려운 부정적 쌍'이 된다 (e.g., (premise, contradiction_hypothesis)).25

이러한 접근법들은 대조 학습의 본질이 단순히 '동일한 인스턴스'를 식별하는 것을 넘어, '의미론적 관계'를 학습하는 방향으로 확장될 수 있음을 명확히 보여준다. 감독 신호의 형태(드롭아웃 노이즈, NLI 레이블)가 학습되는 표현 공간의 구조를 결정하며, 이는 대조 학습 프레임워크의 강력한 유연성을 시사한다.


CLIP(Contrastive Language-Image Pre-training)은 대조 학습을 멀티모달 영역으로 확장하여, 이미지와 텍스트라는 서로 다른 양식(modality)의 데이터를 하나의 공유된 임베딩 공간에 매핑하는 데 성공한 혁신적인 모델이다.54


CLIP은 인터넷에서 수집한 4억 개의 (이미지, 텍스트 캡션) 쌍으로 구성된 방대한 데이터셋을 사용하여 사전 훈련된다.54 학습 목표는 간단하고 직관적이다. 미니배치 내에 $N$개의 이미지와 $N$개의 텍스트가 주어지면, 총 $N x N$개의 가능한 (이미지, 텍스트) 쌍이 형성된다. 이 중에서 실제로 짝을 이루는 $N$개의 긍정적 쌍을 식별하도록 모델을 훈련한다.54

이를 위해 이미지 인코더(ViT 또는 ResNet)와 텍스트 인코더(Transformer)를 동시에 훈련한다. 매칭되는 이미지-텍스트 쌍의 임베딩 간 코사인 유사도는 최대화하고, 매칭되지 않는 $N^2 - N$개의 부정적 쌍의 유사도는 최소화하는 대조 손실을 사용한다.55 이 과정을 통해 모델은 이미지의 시각적 개념과 이를 설명하는 텍스트의 의미론적 개념을 연결하는 방법을 학습하게 된다.


CLIP의 가장 강력한 특징은 특정 분류 작업에 대한 추가적인 미세조정(fine-tuning) 없이도 새로운 작업을 수행할 수 있는 제로샷(zero-shot) 전이 학습 능력이다.56 제로샷 분류 과정은 다음과 같다 57:

1. **프롬프트 엔지니어링:** 분류하고자 하는 클래스 이름들(e.g., 'dog', 'cat', 'car')을 "a photo of a [class name]."과 같은 자연어 문장 템플릿으로 변환한다.
2. **텍스트 임베딩 생성:** 이 프롬프트 문장들을 텍스트 인코더에 입력하여 각 클래스에 해당하는 텍스트 임베딩을 얻는다.
3. **이미지 임베딩 생성:** 분류할 이미지를 이미지 인코더에 입력하여 이미지 임베딩을 얻는다.
4. **유사도 비교 및 분류:** 이미지 임베딩과 모든 클래스의 텍스트 임베딩 간의 코사인 유사도를 계산한다. 가장 높은 유사도 점수를 보인 텍스트 프롬프트에 해당하는 클래스를 최종 예측 결과로 선택한다.55

이처럼 CLIP은 대조 학습을 통해 시각적 표현과 언어적 표현을 성공적으로 정렬(align)함으로써, 전례 없는 수준의 유연성과 일반화 성능을 달성했다.


대조 학습을 통해 사전 훈련된 강력한 표현은 다양한 다운스트림 작업의 성능을 크게 향상시키는 기반 기술로 활용된다.


초기 대조 학습 모델들은 주로 이미지 전체에 대한 단일 표현 벡터를 학습하는 이미지 분류 작업에 초점을 맞추었다. 그러나 이러한 전역적(global) 표현은 객체의 위치를 예측하거나(객체 탐지) 픽셀 단위로 클래스를 분류해야 하는(의미론적 분할) 조밀한 예측(dense prediction) 작업에는 최적이 아닐 수 있다.59 이를 해결하기 위해 대조 학습을 조밀한 예측 작업에 맞게 변형하려는 연구들이 진행되었다.

- **객체 탐지(Object Detection):**
  - **SoCo (Self-supervised object-level contrastive learning):** 선택적 검색(selective search)과 같은 알고리즘을 사용하여 이미지 내에서 객체 후보 영역(object proposal)을 생성하고, 각 후보 영역을 독립적인 인스턴스로 취급하여 객체 수준(object-level)의 대조 학습을 수행한다. 이를 통해 객체의 위치 및 크기 변화에 불변인 표현을 학습하여 탐지 성능을 높인다.59
  - **DetCo (Unsupervised Contrastive Learning for Object Detection):** 이미지의 전역적 표현(global image)과 이미지 내의 지역적 패치(local patches) 간의 대조 학습을 수행한다. 또한, 백본 네트워크의 여러 계층에서 특징을 추출하여 각각에 대조 손실을 적용하는 다단계 감독(multi-level supervision)을 통해, 다양한 크기의 객체를 탐지하는 데 유용한 다중 스케일 표현을 학습한다.60
- **의미론적 분할(Semantic Segmentation):**
  - 픽셀 수준(pixel-level) 또는 패치 수준(patch-level)에서 대조 학습을 적용한다. 동일한 의미론적 클래스에 속하는 픽셀 또는 패치들의 표현은 긍정적 쌍으로 간주하여 가깝게 만들고, 다른 클래스에 속하는 픽셀/패치들은 부정적 쌍으로 간주하여 멀게 만든다.61 이를 통해 모델은 객체 경계를 더 명확하게 구분하고 픽셀 단위의 의미를 더 잘 이해하게 된다.61


자연어 처리 분야에서 대조 학습은 고품질의 문장 및 텍스트 임베딩을 생성하는 데 핵심적인 역할을 한다.

- **의미론적 텍스트 유사성(Semantic Textual Similarity, STS):** SimCSE와 같은 모델을 통해 학습된 문장 임베딩은 두 문장이 의미적으로 얼마나 유사한지를 측정하는 STS 벤치마크에서 기존의 지도 학습 기반 모델들을 능가하는 최첨단(SOTA) 성능을 달성했다.25
- **텍스트 분류 및 기타 다운스트림 작업:** 대조 학습으로 사전 훈련된 인코더는 텍스트 분류, 감성 분석, 자연어 추론, 기계 번역 등 다양한 NLP 다운스트림 작업에서 강력한 초기화(initialization)를 제공한다. 소량의 레이블 데이터로 미세조정(fine-tuning)할 경우, 처음부터 지도 학습을 하는 것보다 훨씬 높은 성능을 보이는 경우가 많다.23


대조 학습은 괄목할 만한 성공을 거두었지만, 여전히 해결해야 할 여러 이론적, 실용적 도전 과제들을 안고 있다. 이러한 문제들을 이해하는 것은 대조 학습의 한계를 파악하고 미래 연구 방향을 설정하는 데 중요하다.



'거짓 음성' 문제는 인스턴스 판별(instance discrimination)에 기반한 자기 지도 대조 학습의 근본적인 한계에서 비롯된다. 이 패러다임에서는 앵커(anchor) 이미지와 다른 모든 이미지를 부정적 샘플로 간주한다.65 그러나 무작위로 샘플링된 부정적 샘플 중에는 앵커와는 다른 인스턴스이지만 의미론적으로는 동일한 클래스에 속하는 샘플(e.g., 앵커가 '페르시안 고양이' 사진일 때, 다른 '샴 고양이' 사진)이 포함될 수 있다. 이렇게 잘못 지정된 부정적 샘플을 '거짓 음성'이라고 한다.65

모델은 손실 함수에 따라 이 거짓 음성 샘플을 앵커로부터 멀리 밀어내도록 학습된다. 이는 결국 동일한 클래스 내의 샘플들을 임베딩 공간에서 흩어지게 만들어, 클래스 내 분별력을 저해하고 학습된 표현의 질을 심각하게 손상시킨다.66 이 문제는 특히 ImageNet과 같이 클래스 수가 많고 데이터가 방대한 대규모 데이터셋에서 샘플링 확률상 더 빈번하게 발생하여 성능 저하의 주요 원인이 된다.65


거짓 음성 문제와는 반대편에 '어려운 부정적 샘플(hard negative samples)'의 개념이 있다. 이는 앵커와 다른 클래스에 속하면서도 임베딩 공간에서 매우 가까워 모델이 구별하기 어려운 샘플을 의미한다 (e.g., '개'와 '늑대').69 이러한 어려운 샘플들을 집중적으로 학습하면 모델의 판별 경계가 더 정교해져 성능이 향상될 수 있다.70 거짓 음성 문제를 해결하고 어려운 부정적 샘플을 효과적으로 활용하기 위한 접근법은 다음과 같다:

- **거짓 음성 탐지 및 제거:** 학습이 진행됨에 따라 임베딩 공간이 점차 의미론적 구조를 갖게 된다는 점에 착안하여, 온라인 클러스터링 등을 통해 앵커와 같은 클러스터에 속하는 부정적 샘플을 거짓 음성으로 동적으로 탐지한다. 이렇게 탐지된 거짓 음성은 손실 계산에서 제외(elimination)하거나, 심지어 추가적인 긍정적 샘플로 취급(attraction)할 수 있다.65
- **어려운 부정적 샘플 채굴(Hard Negative Mining):** 레이블 정보 없이 어려운 부정적 샘플을 효과적으로 샘플링하는 기법을 개발한다. 예를 들어, 현재 임베딩 공간에서 앵커와 가장 가까운 부정적 샘플들을 선택하거나, 데이터 믹싱(mixing) 기법을 통해 합성된 어려운 샘플을 생성할 수 있다.41
- **편향 보정 손실 함수(Debiased Contrastive Loss):** 무작위 샘플링으로 인해 거짓 음성이 포함될 확률을 통계적으로 추정하고, 이를 보정하도록 손실 함수를 재설계하는 접근법도 있다.65


표현 붕괴는 학습된 임베딩이 표현력을 잃고 자명하거나 축소된 형태로 수렴하는 현상을 말하며, 대조 학습의 안정성을 위협하는 주요 문제이다.


- **완전 붕괴(Complete Collapse):** 모델이 모든 입력에 대해 동일한 상수 벡터를 출력하는 최악의 경우이다. 이는 손실을 0으로 만드는 가장 쉬운 방법 중 하나로, 특히 부정적 쌍의 반발력이 없는 비-대조적 방법(e.g., BYOL, SimSiam)에서 주요하게 해결해야 할 문제이다.30
- **차원 붕괴(Dimensional Collapse):** 임베딩 벡터가 사용 가능한 전체 표현 공간을 활용하지 못하고, 그보다 더 낮은 차원의 부분 공간(subspace)에 집중적으로 분포하는 현상이다. 이는 대조적 방법에서도 발생할 수 있다. 예를 들어, 2048차원 공간을 사용하도록 설정했지만 실제로는 128차원 정도의 정보만 표현하는 경우이다. 이는 표현의 다양성을 심각하게 저해하여 다운스트림 작업 성능을 떨어뜨린다.72


표현 붕괴를 방지하기 위해 다양한 아키텍처적 장치들이 고안되었다.

- **비-대조적 방법의 붕괴 방지:** BYOL과 SimSiam은 비대칭 아키텍처(온라인 네트워크에만 예측 헤드 추가), stop-gradient, EMA 업데이트와 같은 정교한 메커니즘을 조합하여 완전 붕괴를 효과적으로 방지한다.14
- **차원 붕괴 방지:** 차원 붕괴의 한 가지 원인으로 프로젝션 헤드의 가중치 행렬이 학습 과정에서 낮은 순위(low-rank)로 수렴하는 것이 지목된다. 이를 해결하기 위해 프로젝터의 가중치에 직교성(orthogonality) 제약을 가하거나(CLOP), 프로젝터를 분석적으로 대체하거나(DirectCLR), 표현의 공분산 행렬을 단위 행렬에 가깝게 만드는 정규화 항(Barlow Twins, VICReg)을 추가하는 등의 연구가 진행되고 있다.73



SimCLR과 같은 초기 모델들은 대조 학습의 성능이 배치 사이즈에 크게 의존한다는 것을 보여주었다. 배치 사이즈가 클수록 미니배치 내에 더 많고 다양한 부정적 샘플이 포함되어, 손실 함수의 그래디언트 추정치가 더 안정적이고 정확해지며, 결과적으로 모델이 더 우수한 표현을 학습할 수 있다.38

하지만 이러한 의존성은 대조 학습의 실용성을 저해하는 가장 큰 장벽 중 하나이다. 수천 개의 샘플로 구성된 배치를 처리하기 위해서는 막대한 양의 GPU 메모리와 계산 자원이 필요하며, 이는 많은 연구자나 개발자들이 접근하기 어려운 수준이다.76


이러한 한계를 극복하기 위해 다양한 연구가 진행되었다.

- **아키텍처적 접근:** MoCo는 메모리 큐를 도입하여 배치 사이즈와 부정적 샘플의 수를 성공적으로 분리했다.40
- **손실 함수 재설계:** DCL(Decoupled Contrastive Learning)은 InfoNCE 손실 함수에서 긍정적 쌍과 부정적 쌍의 기여도를 분리하여, 작은 배치 사이즈에서도 학습 효율이 저하되는 '결합(coupling)' 문제를 완화했다.78
- **효율적인 계산 기법:** 유사도 행렬 전체를 메모리에 올리지 않고, 이를 작은 타일(tile)로 나누어 순차적으로 계산함으로써 메모리 사용량을 획기적으로 줄이는 방법이 제안되었다.76
- **이론적 분석:** 작은 배치 사이즈가 왜 그래디언트 편향(gradient bias)을 유발하는지를 이론적으로 분석하고, 베이지안 데이터 증강과 같은 기법을 통해 이 편향을 보정하려는 연구도 진행되고 있다.80

이러한 도전 과제들은 서로 깊이 연관되어 있다. 대조 학습은 본질적으로 '샘플링의 질과 양'이라는 딜레마에 직면해 있다. 더 나은 표현을 위해 부정적 샘플의 '양'을 늘리려 하면(대규모 배치), 그 안에 '거짓 음성'이 포함될 확률이 높아져 샘플링의 '질'이 저하된다. 반대로 거짓 음성을 피하고 '어려운' 부정적 샘플만을 선별하여 '질'을 높이려 하면, 모델이 소수의 샘플에 과적합되어 표현의 다양성을 잃고 '차원 붕괴'에 빠질 위험이 있다. 이 순환적인 관계는 대조 학습의 최적화가 단순히 손실 함수를 최소화하는 것을 넘어, 샘플링 전략, 손실 함수 설계, 아키텍처 정규화가 복합적으로 작용하는 섬세한 균형 잡기 과정임을 명확히 보여준다.



대조 학습은 지난 몇 년간 기계 학습, 특히 자기 지도 표현 학습 분야에서 가장 중요한 패러다임 중 하나로 확고히 자리 잡았다. 레이블이 없는 방대한 데이터로부터 지도 학습에 필적하거나 이를 능가하는 강력하고 전이 가능한 표현을 학습할 수 있음을 증명함으로써, 딥러닝 모델의 학습 방식을 근본적으로 변화시켰다.

컴퓨터 비전에서 시작하여 자연어 처리, 멀티모달, 그래프 학습 등 다양한 분야로 성공적으로 확장되었으며, SimCLR, MoCo, BYOL, CLIP, SimCSE와 같은 혁신적인 모델들은 각 분야의 기술 수준을 한 단계 끌어올렸다. 대조 학습은 단순히 성능을 향상시키는 기술을 넘어, 데이터의 내재적 구조와 의미론적 관계를 모델이 스스로 학습하도록 하는 강력한 프레임워크를 제공했다는 점에서 그 의의가 매우 크다.


대조 학습은 눈부신 성과를 거두었지만, 여전히 많은 미해결 과제를 남기고 있으며, 이는 곧 미래 연구의 유망한 방향을 제시한다.

- **이론적 기반 강화:** 대조 학습이 왜, 그리고 어떻게 효과적으로 작동하는지에 대한 이론적 이해는 아직 초기 단계에 머물러 있다. 손실 함수의 기하학적 특성, 표현 공간의 동역학, 데이터 증강과 학습된 불변성 간의 관계 등에 대한 더 깊고 엄밀한 수학적 분석이 필요하다.34
- **데이터 증강의 자동화 및 일반화:** 현재의 데이터 증강 기법은 대부분 수동으로 설계되며, 특정 도메인과 데이터셋에 크게 의존하는 경향이 있다. 이는 새로운 문제에 대조 학습을 적용할 때마다 많은 시행착오를 요구한다. 학습 과정에서 다운스트림 작업에 최적화된 증강 정책을 자동으로 학습하는 ViewMaker와 같은 연구는 이러한 한계를 극복할 중요한 열쇠가 될 수 있다.84
- **샘플링 효율성 및 견고성 개선:** 거짓 음성 문제와 대규모 배치 사이즈 의존성은 여전히 대조 학습의 실용성을 저해하는 주요 요인이다. 이를 근본적으로 해결하기 위한 더 정교하고 효율적인 샘플링 전략, 편향에 강건한 손실 함수 설계, 그리고 자원 제약이 심한 환경에서도 잘 작동하는 경량화된 대조 학습 프레임워크에 대한 연구가 지속적으로 필요하다.66
- **생성 모델과의 시너지:** 대조 학습의 강력한 판별 능력과 생성 모델의 풍부한 데이터 분포 모델링 능력을 결합하려는 시도가 주목받고 있다. 생성 모델이 고품질의 '어려운' 긍정적/부정적 샘플을 합성하여 대조 학습을 돕거나, 두 프레임워크를 공동으로 훈련하여 상호 보완적인 표현을 학습하는 하이브리드 모델은 차세대 자기 지도 학습의 중요한 방향이 될 것이다.8
- **복잡한 관계 및 구조 학습:** 현재의 대조 학습은 주로 '유사/비유사'라는 이분법적 관계에 초점을 맞추고 있다. 앞으로는 객체 간의 구성적 관계(compositional structures), 인과 관계, 계층적 관계 등 더 복잡하고 구조화된 세계 모델을 학습하는 방향으로 대조 학습의 개념을 확장하는 연구가 필요하다. 이는 진정한 의미의 '이해'에 한 걸음 더 다가가는 길이 될 것이다.84

결론적으로, 대조 학습은 표현 학습의 지평을 넓힌 혁신적인 패러다임이며, 앞으로도 이론적 깊이를 더하고 실용적 한계를 극복하며 생성 모델과 같은 다른 패러다임과의 융합을 통해 인공지능 기술의 발전을 지속적으로 견인할 것으로 기대된다.


1. An Introduction to Contrastive Learning for Computer Vision - Lightly, 8월 24, 2025에 액세스, https://www.lightly.ai/blog/contrastive-learning
2. The Benefits and Limitations of Contrastive Learning in AI - AITechTrend, 8월 24, 2025에 액세스, https://aitechtrend.com/the-benefits-and-limitations-of-contrastive-learning-in-ai/
3. How do contrastive learning and self-supervised learning work together? - Milvus, 8월 24, 2025에 액세스, https://milvus.io/ai-quick-reference/how-do-contrastive-learning-and-selfsupervised-learning-work-together
4. Contrastive Learning: A Powerful Approach to Self-Supervised Representation in Machine Learning - Netguru, 8월 24, 2025에 액세스, https://www.netguru.com/blog/contrastive-learning
5. What is Contrastive Learning? A 2025 Guide - Viso Suite, 8월 24, 2025에 액세스, https://viso.ai/deep-learning/contrastive-learning/
6. A Survey on Contrastive Self-Supervised Learning - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2227-7080/9/1/2
7. Self-Supervised Learning for Time Series: Contrastive or Generative? - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2403.09809v1
8. [2006.08218] Self-supervised Learning: Generative or Contrastive - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2006.08218
9. The Beginner's Guide to Contrastive Learning - V7 Labs, 8월 24, 2025에 액세스, https://www.v7labs.com/blog/contrastive-learning-guide
10. Investigating Contrastive Pair Learning's Frontiers in Supervised, Semisupervised, and Self-Supervised Learning - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11355693/
11. What is Contrastive Learning? A guide. - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/contrastive-learning-machine-learning/
12. Contrastive Learning: A Comprehensive Guide | by Juan C ..., 8월 24, 2025에 액세스, https://medium.com/@juanc.olamendy/contrastive-learning-a-comprehensive-guide-69bf23ca6b77
13. Contrastive Learning vs. Generative Modeling: What's the Difference ..., 8월 24, 2025에 액세스, https://blog.gopenai.com/contrastive-learning-vs-1253368ed8a5
14. Self-Supervised Visual Representation Learning on Food Images, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2303.09046
15. Contrastive Learning with Image Deformation and Refined NT-Xent Loss for Urban Morphology Discovery - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2220-9964/14/5/196
16. Tutorial 17: Self-Supervised Contrastive Learning with SimCLR — UvA DL Notebooks v1.2 documentation, 8월 24, 2025에 액세스, https://uvadlc-notebooks.readthedocs.io/en/latest/tutorial_notebooks/tutorial17/SimCLR.html
17. What is the role of data augmentation in contrastive learning? - Milvus, 8월 24, 2025에 액세스, https://milvus.io/ai-quick-reference/what-is-the-role-of-data-augmentation-in-contrastive-learning
18. A Simple Framework for Contrastive Learning of Visual ..., 8월 24, 2025에 액세스, https://proceedings.mlr.press/v119/chen20j/chen20j.pdf
19. Notes on A Simple Framework for Contrastive Learning of Visual Representations(SimCLR), 8월 24, 2025에 액세스, https://hackmd.io/@mathurpulkit/HJX91MnJK
20. What Should Not Be Contrastive in Contrastive Learning - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=CZ8Y3NzuVzO
21. Benchmarking Data Augmentation for Contrastive Learning in Static Sign Language Recognition, 8월 24, 2025에 액세스, https://www.esann.org/sites/default/files/proceedings/2025/ES2025-142.pdf
22. Contrastive Learning with Synthetic Positives - European Computer Vision Association, 8월 24, 2025에 액세스, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/05479.pdf
23. Contrastive Learning In NLP - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/nlp/contrastive-learning-in-nlp/
24. An End-to-End Contrastive Self-Supervised Learning Framework for Language Understanding | Transactions of the Association for Computational Linguistics - MIT Press Direct, 8월 24, 2025에 액세스, https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00521/114047/An-End-to-End-Contrastive-Self-Supervised-Learning
25. SimCSE: Simple Contrastive Learning of Sentence Embeddings ..., 8월 24, 2025에 액세스, https://aclanthology.org/2021.emnlp-main.552/
26. (PDF) SimCSE: Simple Contrastive Learning of Sentence Embeddings - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/350991491_SimCSE_Simple_Contrastive_Learning_of_Sentence_Embeddings
27. Contrastive Representation Learning | Lil'Log, 8월 24, 2025에 액세스, https://lilianweng.github.io/posts/2021-05-31-contrastive/
28. SINCERE: Supervised Information Noise-Contrastive Estimation ..., 8월 24, 2025에 액세스, https://arxiv.org/pdf/2309.14277
29. [2105.13003] Rethinking InfoNCE: How Many Negative Samples Do You Need? - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2105.13003
30. f-MICL: Understanding and Generalizing InfoNCE-based Contrastive Learning - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2402.10150v1
31. the nt-xent loss upper bound - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2205.03169
32. [2501.17683] Temperature-Free Loss Function for Contrastive Learning - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2501.17683
33. Temperature as Uncertainty in Contrastive Learning, 8월 24, 2025에 액세스, https://sslneurips21.github.io/files/CameraReady/tau_paper.pdf
34. Understanding the Behaviour of Contrastive Loss - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2021/papers/Wang_Understanding_the_Behaviour_of_Contrastive_Loss_CVPR_2021_paper.pdf
35. Not All Semantics are Created Equal: Contrastive Self-supervised Learning with Automatic Temperature Individualization, 8월 24, 2025에 액세스, https://proceedings.mlr.press/v202/qiu23a/qiu23a.pdf
36. Simple Temperature Cool-down in Contrastive Framework for Unsupervised Sentence Representation Learning - ACL Anthology, 8월 24, 2025에 액세스, https://aclanthology.org/2024.findings-eacl.37.pdf
37. [2002.05709] A Simple Framework for Contrastive Learning of Visual Representations - ar5iv, 8월 24, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2002.05709
38. Exploring SimCLR: A Simple Framework for Contrastive Learning of ..., 8월 24, 2025에 액세스, https://sthalles.github.io/simple-self-supervised-learning/
39. SimCLR: A Simple Framework for Contrastive Learning of Visual Representations: A Brief Summary - Medium, 8월 24, 2025에 액세스, https://medium.com/@kdwa1252043/simclr-a-simple-framework-for-contrastive-learning-of-visual-representations-a-brief-summary-a307f51e5230
40. How Does MoCo Learn Without Labels? The Magic Behind Momentum Contrast Explained | by Praneeta Mallela | The ML Intuition | Medium, 8월 24, 2025에 액세스, https://medium.com/the-ml-intuition/moco-momentum-contrast-for-efficient-self-supervised-representation-learning-feb41bfdb1f5
41. 39: MoCO v1 & v2 Explained - Casual GAN Papers, 8월 24, 2025에 액세스, https://www.casualganpapers.com/self-supervised-representation-learning-encoder/MoCo-explained.html
42. Momentum Contrast for Unsupervised Visual ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content_CVPR_2020/papers/He_Momentum_Contrast_for_Unsupervised_Visual_Representation_Learning_CVPR_2020_paper.pdf
43. Overcoming Dimensional Collapse in Self-supervised Contrastive Learning for Medical Image Segmentation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2402.14611v1
44. Bootstrap your own latent: A new approach to self-supervised ..., 8월 24, 2025에 액세스, https://arxiv.org/pdf/2006.07733
45. BYOL for Audio: Self-Supervised Learning for General-Purpose Audio Representation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2103.06695
46. Branching Out for Better BYOL - NeurIPS 2021 Workshop: Self-Supervised Learning - Theory and Practice, 8월 24, 2025에 액세스, [https://sslneurips21.github.io/files/CameraReady/Multi_Target_BYOL%20.pdf](https://sslneurips21.github.io/files/CameraReady/Multi_Target_BYOL .pdf)
47. Unsupervised Learning of Visual Features by Contrasting Cluster ..., 8월 24, 2025에 액세스, https://arxiv.org/pdf/2006.09882
48. Unsupervised Learning of Visual Features by Contrasting Cluster Assignments - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2006.09882
49. Information Maximization Clustering via Multi-View Self-Labelling - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2103.07368
50. Unsupervised Visual Representation Learning with SwAV - Wandb, 8월 24, 2025에 액세스, https://wandb.ai/authors/swav-tf/reports/Unsupervised-Visual-Representation-Learning-with-SwAV--VmlldzoyMjg3Mzg
51. Supervised Contrastive Learning - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=MpdbFLXOOIw&pp=0gcJCfwAo7VqN5tD
52. SINCERE: Supervised Information Noise-Contrastive Estimation REvisited - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2309.14277v4
53. arXiv:2310.06918v2 [cs.CL] 20 Oct 2023, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2310.06918
54. OpenAI CLIP: Bridging Text and Images | by Frank Morales Aguilera | The Deep Hub, 8월 24, 2025에 액세스, https://medium.com/thedeephub/openai-clip-bridging-text-and-images-aaf3cd20299e
55. Contrastive Language-Image Pre-training - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Contrastive_Language-Image_Pre-training
56. OpenAI CLIP: ConnectingText and Images (Paper Explained) - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=T9XSU0pKX2E
57. CLIP: Connecting text and images | OpenAI, 8월 24, 2025에 액세스, https://openai.com/index/clip/
58. [R] CLIP: Connecting Text and Images (from OpenAI) : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/kr7bp9/r_clip_connecting_text_and_images_from_openai/
59. Aligning Pretraining for Detection via Object-Level Contrastive Learning - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/pdf?id=8PA2nX9v_r2
60. DetCo: Unsupervised Contrastive Learning for ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2021/papers/Xie_DetCo_Unsupervised_Contrastive_Learning_for_Object_Detection_ICCV_2021_paper.pdf
61. Contrastive Learning for Label Efficient Semantic Segmentation | by Hey Amit - Medium, 8월 24, 2025에 액세스, https://medium.com/@heyamit10/contrastive-learning-for-label-efficient-semantic-segmentation-3563442cc765
62. Semantic-Aware Contrastive Learning for Multi-object Medical Image Segmentation - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10524443/
63. Contextrast: Contextual Contrastive Learning for Semantic Segmentation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2404.10633v2
64. medium.com, 8월 24, 2025에 액세스, [https://medium.com/@juanc.olamendy/contrastive-learning-a-comprehensive-guide-69bf23ca6b77#:~:text=The%20application%20of%20contrastive%20learning,sentiment%20analysis%2C%20and%20machine%20translation.](https://medium.com/@juanc.olamendy/contrastive-learning-a-comprehensive-guide-69bf23ca6b77#:~:text=The application of contrastive learning,sentiment analysis%2C and machine translation.)
65. INCREMENTAL FALSE NEGATIVE DETECTION ... - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/pdf?id=PSNHQ8B5os
66. Discovering Global False Negatives On the Fly for Self-supervised Contrastive Learning, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.20612v1
67. Incremental False Negative Detection for Contrastive Learning - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=dDjSKKA5TP1
68. Boosting Contrastive Self-Supervised Learning with False Negative Cancellation, 8월 24, 2025에 액세스, https://research.google/pubs/boosting-contrastive-self-supervised-learning-with-false-negative-cancellation/
69. Contrastive Learning with Hard Negative Samples | by It's Amit | Medium, 8월 24, 2025에 액세스, https://mr-amit.medium.com/contrastive-learning-with-hard-negative-samples-2cccb609fa0c
70. Contrastive Learning with Hard Negative Samples | OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=CR1XOQ0UTh-
71. Hard Negative Mixing for Contrastive Learning, 8월 24, 2025에 액세스, https://papers.neurips.cc/paper_files/paper/2020/file/f7cade80b7cc92b991cf4d2806d6bd78-Paper.pdf
72. Understanding Collapse in Non-Contrastive Siamese Representation Learning - CMU School of Computer Science, 8월 24, 2025에 액세스, https://www.cs.cmu.edu/~dpathak/papers/eccv22.pdf
73. Understanding Dimensional Collapse in Contrastive Self-supervised ..., 8월 24, 2025에 액세스, https://openreview.net/forum?id=YevsQ05DEN7
74. Implicit Contrastive Representation Learning with Guided Stop-gradient, 8월 24, 2025에 액세스, https://proceedings.neurips.cc/paper_files/paper/2023/hash/6274172f7d981a8d58bbfd52342a9d1f-Abstract-Conference.html
75. Preventing Collapse in Contrastive Learning with Orthonormal Prototypes (CLOP) - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2403.18699
76. Breaking the Memory Barrier: Near Infinite Batch Size Scaling for Contrastive Loss - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2410.17243v1
77. The Effect of Batch Size on Contrastive Self-Supervised Speech Representation Learning, 8월 24, 2025에 액세스, https://arxiv.org/html/2402.13723v1
78. Decoupled Contrastive Learning - European Computer Vision Association, 8월 24, 2025에 액세스, https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136860653.pdf
79. Why do We Need Large Batchsizes in Contrastive Learning? A Gradient-Bias Perspective, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/Why-do-We-Need-Large-Batchsizes-in-Contrastive-A-Chen-Zhang/6b4cf65a971d19bac5185395e0f4c78993b0320e
80. Why do we need large batch sizes in contrastive learning? A gradient-bias perspective - Amazon Science, 8월 24, 2025에 액세스, https://www.amazon.science/publications/why-do-we-need-large-batch-sizes-in-contrastive-learning-a-gradient-bias-perspective
81. Why do We Need Large Batchsizes in Contrastive Learning? A Gradient-Bias Perspective, 8월 24, 2025에 액세스, https://scholars.duke.edu/individual/pub1588096
82. Why do We Need Large Batchsizes in Contrastive Learning? A Gradient-Bias Perspective, 8월 24, 2025에 액세스, https://proceedings.neurips.cc/paper_files/paper/2022/hash/db174d373133dcc6bf83bc98e4b681f8-Abstract-Conference.html
83. Understanding Self-supervised Contrastive Learning through Supervised Objectives | OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=cmE97KX2XM
84. Contrastive Learning: Big Data Foundations and Applications, 8월 24, 2025에 액세스, https://bigdataieee.org/BigData2023/files/Tutorial6_ContrastiveLearning.pdf
85. GenView: Enhancing View Quality with Pretrained Generative Model for Self-Supervised Learning - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2403.12003v2
86. Self-Supervised Generative-Contrastive Learning of Multi-Modal Euclidean Input for 3D Shape Latent Representations: A Dynamic Switching Approach - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2301.04612v2


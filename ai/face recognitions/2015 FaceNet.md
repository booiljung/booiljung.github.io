# Google FaceNet (A Unified Embedding for Face Recognition and Clustering, CVPR 2015)



FaceNet이 등장하기 이전, 안면 인식 기술은 비제약적인 환경(unconstrained environments)에서 수많은 도전에 직면했다. 실제 환경에서 촬영된 얼굴 이미지는 조명 조건, 얼굴의 포즈, 안경이나 마스크 등으로 인한 가려짐(occlusion) 등 다양한 변수에 의해 크게 영향을 받았다.1 이러한 요인들은 기존 알고리즘의 성능을 심각하게 저하시켰으며, 특히 수천, 수만 명에 이르는 대규모 신원을 실시간으로 정확하게 검증하고 식별해야 하는 확장성(scalability) 문제는 해결하기 어려운 난제로 남아있었다.6 기존 방법론들은 이러한 현실 세계의 복잡성을 효과적으로 처리하지 못했으며, 이는 안면 인식 기술의 실용적인 적용을 가로막는 주요한 장벽으로 작용했다.


2015년 Google 연구팀이 발표한 FaceNet은 이러한 문제에 대한 근본적인 패러다임 전환을 제시했다.6 FaceNet의 핵심 철학은 안면 이미지를 특정 인물로 '분류(classify)'하는 전통적인 접근법에서 벗어나, 얼굴 이미지 자체를 고차원의 의미론적 특징 공간(semantic feature space)으로 '매핑(mapping)'하는 문제로 재정의한 것에 있다.7 이 시스템은 심층 컨볼루션 신경망(Deep Convolutional Neural Network, DCNN)을 사용하여 안면 이미지를 128차원의 조밀한(compact) 유클리드 공간으로 직접 사상(mapping)한다.1 이 임베딩 공간의 핵심적인 특징은 두 벡터 간의 L2 거리(squared L2 distance)가 곧 두 얼굴의 유사도(similarity)에 직접적으로 비례하도록 학습된다는 점이다.7 즉, 동일 인물의 얼굴 이미지들은 임베딩 공간에서 서로 가까운 위치에 군집하고, 서로 다른 인물의 이미지들은 멀리 떨어지게 된다.


FaceNet의 이러한 접근법은 Facebook의 DeepFace와 같은 이전 세대의 심층 신경망 기반 모델들과 근본적인 차이를 보인다.4 기존 모델들은 대규모 신원 데이터셋을 이용해 다중 클래스 분류기(multi-class classifier)를 학습시킨 후, 네트워크의 중간 단계에 위치한 병목 계층(intermediate bottleneck layer)의 출력을 얼굴의 특징 표현(feature representation)으로 사용하는 방식을 취했다.9 이 방식은 두 가지 주요한 단점을 내포했다. 첫째, 임베딩을 간접적으로(indirectness) 학습하기 때문에 병목 계층의 특징이 얼굴 인식 작업에 최적화되었다고 보장할 수 없었다. 둘째, 특징 벡터의 차원이 수천 개에 달해 매우 비효율적(inefficiency)이었다.10 더 큰 문제는 이러한 표현이 훈련 데이터에 포함되지 않았던 새로운 얼굴에 대해 얼마나 잘 일반화될 수 있는지 희망에 의존해야 했다는 점이다. FaceNet은 이러한 병목 계층을 사용하는 대신, 임베딩 자체를 손실 함수를 통해 직접 최적화하는 엔드투엔드(end-to-end) 학습 방식을 채택함으로써 이러한 한계를 정면으로 돌파했다.7

FaceNet의 진정한 혁신은 단순히 새로운 CNN 아키텍처를 제시한 것이 아니라, 안면 인식이라는 문제 자체를 '재정의'한 데 있다. 기존 연구가 "이 얼굴은 누구인가?"라는 분류(classification) 문제에 집중했다면, FaceNet은 "이 얼굴은 저 얼굴과 얼마나 유사한가?"라는 메트릭 학습(metric learning) 문제로 관점을 전환했다. 이러한 철학적 전환은 안면 인식 기술을 '닫힌 집합(closed-set)' 문제에서 '열린 집합(open-set)' 문제로 확장하는 결정적인 계기가 되었다. 닫힌 집합 문제는 훈련 시 학습한 고정된 수의 인물들 내에서만 식별이 가능한 반면, 열린 집합 문제는 훈련 시 보지 못했던 새로운 인물이 등장하더라도 시스템에 유연하게 대처할 수 있어야 한다. DeepFace와 같이 Softmax 손실 함수에 기반한 분류 모델은 새로운 인물이 추가될 때마다 모델의 마지막 계층을 재구성하고 전체 모델을 재학습해야 하는 심각한 확장성 제약을 가졌다.4 반면, FaceNet은 특정 인물 ID에 종속되지 않는 범용적인 '얼굴 특징 추출기'를 학습한다. 따라서 한번 잘 학습된 FaceNet 모델이 있다면, 새로운 인물을 시스템에 추가하는 과정은 모델을 재학습하는 것이 아니라, 해당 인물의 얼굴 사진 한 장을 임베딩하여 그 128차원 벡터를 데이터베이스에 추가하는 간단한 작업으로 축소된다. 이는 단 한 장의 사진으로 학습하는 '원샷 학습(one-shot learning)'을 가능하게 하여, 안면 인식 시스템의 실용성, 확장성, 유연성을 근본적으로 개선한 패러다임의 전환이었다.14



FaceNet은 얼굴 감지, 정렬, 특징 추출, 비교로 이어지는 복잡한 파이프라인을 하나의 통합된 학습 시스템으로 구현하려는 시도의 결과물이다. 시스템의 입력으로는 스케일과 위치가 대략적으로 정렬된(roughly aligned) 얼굴 이미지가 사용된다.9 이 이미지는 심층 CNN 모델을 통과하여 최종적으로 128차원의 임베딩 벡터를 출력하는 엔드투엔드 구조를 가진다.7 이 과정의 핵심은 중간 산출물에 대한 별도의 감독(supervision) 없이, 오직 Triplet Loss라는 단일 목표 함수를 통해 전체 네트워크의 가중치가 역전파(backpropagation) 알고리즘에 의해 최적화된다는 점이다. 이는 시스템 전체가 얼굴 유사도를 측정하는 단 하나의 목표를 향해 유기적으로 학습되도록 만든다.


FaceNet 논문에서는 임베딩을 생성하기 위한 CNN 백본(backbone)으로 두 가지 주요 아키텍처를 실험하고 그 성능을 비교 분석했다.8


첫 번째 아키텍처는 당시 ImageNet 대회에서 우승하며 주목받았던 Google의 GoogLeNet(Inception v1)을 기반으로 한다.9 Inception 모델의 핵심은 'Inception 모듈'이라는 독창적인 구조에 있다. 이 모듈은 1x1, 3x3, 5x5 등 다양한 크기의 컨볼루션 필터와 맥스 풀링(max pooling) 연산을 병렬로 수행한 후, 그 결과들을 채널 차원에서 모두 결합(concatenate)하는 방식을 사용한다.14 이러한 구조는 모델이 이미지 내의 다양한 스케일에서 나타나는 특징들을 동시에 포착할 수 있도록 한다. 특히, 3x3 및 5x5 컨볼루션 이전에 1x1 컨볼루션을 '병목(bottleneck)' 계층으로 사용하여 입력 채널의 차원을 축소함으로써, 전체 연산량과 파라미터 수를 획기적으로 줄이는 효과를 가져온다.9


두 번째 아키텍처는 Zeiler와 Fergus가 제안한 ZF-Net 모델을 기반으로 한, 보다 전통적인 순차적 구조의 CNN이다.8 두 모델의 비교를 통해 Inception 모델의 효율성이 명확히 드러난다. ZF-Net 기반 모델은 약 1억 4천만 개의 방대한 파라미터를 가지는 반면, Inception 기반 모델은 약 750만 개의 파라미터만을 가진다. 이는 파라미터 수를 20배 가까이 줄인 것이다.8 놀랍게도, 파라미터 수의 극적인 차이에도 불구하고 두 모델을 훈련하는 데 필요한 연산량(FLOPS, Floating Point Operations Per Second)은 약 16억 FLOPS로 비슷한 수준이었다.8 이는 Inception 모델이 훨씬 적은 파라미터로도 높은 수준의 표현력을 유지하며, 모델의 경량화와 효율성 측면에서 큰 진보를 이루었음을 보여준다.

Inception 모델의 채택은 단순히 성능이 더 좋은 아키텍처를 선택하는 문제를 넘어, FaceNet의 핵심 학습 전략을 실현하기 위한 필연적인 선택이었다. FaceNet의 Triplet Loss 학습 방법론은 효과적인 학습을 위해 수천 개의 샘플로 구성된 매우 큰 미니배치(mini-batch)를 필요로 한다.15 이러한 대규모 배치를 처리하기 위해서는 모델의 순전파 및 역전파 과정에서 막대한 계산 비용과 메모리가 요구된다. 1억 4천만 개의 파라미터를 가진 ZF-Net과 같은 무거운 모델로는 이러한 대규모 훈련을 감당하기가 현실적으로 어렵다.8 반면, Inception 모델은 1x1 컨볼루션을 활용한 독창적인 설계 덕분에 파라미터 수를 극적으로 줄이면서도 표현력을 유지했다.9 따라서 Inception 모델의 선택은 FaceNet의 핵심 학습 전략인 '대규모 온라인 Triplet Mining'을 현실적으로 구현 가능하게 만든 필수적인 전제 조건이었다. 효율적인 아키텍처가 뒷받침되지 않았다면, Triplet Loss라는 강력한 이론의 잠재력을 최대로 이끌어내기 어려웠을 것이다.


CNN을 통과하여 얻어진 최종 128차원 특징 벡터는 L2 정규화(L2 Normalization)라는 후처리 단계를 거친다.8 이 과정은 벡터의 각 요소를 제곱하여 더한 값의 제곱근, 즉 유클리드 노름(Euclidean norm)으로 벡터의 모든 요소를 나누어, 최종 벡터의 노름이 1이 되도록 만드는 것이다. 수학적으로는 $\lVert f(x) \rVert_2 = 1$을 만족하도록 조정된다.

이 L2 정규화는 중요한 기하학적 의미를 가진다. 이는 모든 얼굴 임베딩 벡터를 원점 중심이고 반지름이 1인 128차원 초구(hypersphere)의 표면 위로 투영하는 것과 같다. 이 과정을 통해 임베딩 벡터의 '크기' 또는 '길이' 정보는 제거되고, 오직 '방향' 정보만이 남게 된다. 이는 조명 변화와 같이 얼굴의 본질적인 특징과는 무관하지만 벡터의 크기에 영향을 줄 수 있는 요인들의 효과를 줄여준다. 결과적으로, 모든 임베딩이 동일한 스케일을 갖게 되므로 벡터 간의 비교가 단순화되고, 이는 Triplet Loss 학습 과정을 더욱 안정시키는 효과를 가져온다.



FaceNet의 핵심에는 Triplet Loss라는 독창적인 손실 함수가 자리 잡고 있다. 이 손실 함수는 심층 신경망이 얼굴의 미묘한 차이를 학습하여 강력한 판별력을 갖는 임베딩 공간을 만들도록 유도한다.

- **구성 요소:** 이름에서 알 수 있듯이, Triplet Loss는 세 개의 이미지 샘플로 구성된 'triplet'을 기본 단위로 사용한다. 각 triplet은 기준이 되는 이미지인 **Anchor**($x_i^a$), Anchor와 동일한 인물의 다른 이미지인 **Positive**($x_i^p$), 그리고 Anchor와는 다른 인물의 이미지인 **Negative**($x_i^n$)로 구성된다.13

- **목표:** 학습의 근본적인 목표는 임베딩 함수 $f(\cdot)$에 의해 매핑된 특징 공간에서, Anchor와 Positive 사이의 제곱 유클리드 거리가 Anchor와 Negative 사이의 제곱 유클리드 거리보다 항상 일정 거리 이상 작아지도록 만드는 것이다. 이 '일정 거리'를 양의 마진(positive margin) $ \alpha $라고 하며, 이 조건을 **triplet 제약 조건(triplet constraint)**이라 부른다.13
  $$
  \lVert f(x_i^a) - f(x_i^p) \rVert_2^2 + \alpha < \lVert f(x_i^a) - f(x_i^n) \rVert_2^2
  $$
  **손실 함수:** 이 제약 조건을 모든 triplet $i$에 대해 만족시키기 위해, 손실 함수 $L$은 다음과 같이 정의된다. `max` 함수를 사용하여 제약 조건이 이미 만족된 경우(괄호 안의 값이 음수일 경우)에는 손실을 0으로 만들어 더 이상 학습에 영향을 주지 않도록 하고, 위반된 경우에만 양의 손실 값을 갖도록 설계되었다.8
  $$
  L = \sum_{i}^{N} \max\left( \left[ \lVert f(x_i^a) - f(x_i^p) \rVert_2^2 - \lVert f(x_i^a) - f(x_i^n) \rVert_2^2 + \alpha \right], 0 \right)
  $$
  **마진($ \alpha $)의 역할:** 마진 $ \alpha $는 동일 인물(intra-class)의 임베딩들이 형성하는 클러스터와 다른 인물(inter-class)의 임베딩들 사이에 안전한 '완충 지대'를 설정하는 역할을 한다. 이는 모델이 단순히 두 인물을 간신히 구분하는 경계면을 학습하는 것을 넘어, 명확하고 강건한 분리 경계를 형성하도록 강제한다. 이를 통해 모델의 일반화 성능이 크게 향상된다.16


Triplet Loss의 잠재력을 최대로 활용하기 위해서는 어떤 triplet을 사용하여 모델을 학습시킬지가 매우 중요하다.

- **문제점:** 훈련 데이터셋에서 가능한 모든 triplet을 생성하여 학습에 사용하는 것은 두 가지 큰 문제를 야기한다. 첫째, triplet의 수는 조합적으로 폭발하여 계산적으로 거의 불가능하다. 둘째, 대부분의 무작위로 선택된 triplet은 이미 triplet 제약 조건을 쉽게 만족하기 때문에 손실 값이 0이 되어 학습에 아무런 기여를 하지 못한다.15 따라서 학습에 실질적으로 도움이 되는 '어려운(hard)' triplet을 효과적으로 선별하는 전략이 필수적이다.

- **온라인(Online) Triplet Mining:** FaceNet은 이 문제를 해결하기 위해 **온라인 triplet mining**이라는 혁신적인 방법을 제안했다.6 이는 훈련 시작 전에 전체 데이터셋에서 triplet을 미리 생성하는 오프라인 방식과 달리, 각 학습 스텝마다 구성된 미니배치(mini-batch) 내에서 동적으로 triplet을 찾아 생성하는 방식이다.15 이 전략의 효율성을 극대화하기 위해, FaceNet은 한 번에 1,800개의 샘플과 같이 매우 큰 크기의 미니배치를 사용하여 어려운 triplet을 찾을 수 있는 충분한 탐색 공간을 확보했다.15

- **Semi-Hard Negative 선택:** 온라인 마이닝 과정에서 어떤 Negative 샘플을 선택할 것인지가 학습의 성패를 좌우한다. FaceNet 연구팀은 실험을 통해 **semi-hard negative**를 선택하는 것이 가장 효과적임을 발견했다.15

  - **Hardest Negatives의 함정:** 이론적으로는 Anchor로부터 Positive보다도 더 가까운 Negative, 즉 $ \lVert f(x^a) - f(x^n) \rVert_2^2 < \lVert f(x^a) - f(x^p) \rVert_2^2 $를 만족하는 '가장 어려운' Negative를 선택하는 것이 학습에 가장 효과적일 것처럼 보인다. 하지만 실제로는 학습 초기에 잘못 임베딩된 샘플이나 노이즈가 많은 이미지(예: 잘못 레이블링된 데이터)가 hardest negative로 선택될 가능성이 높다. 모델이 이러한 병적인(pathological) 샘플에 과도하게 집중하게 되면, 전체적인 임베딩 공간의 구조가 무너지는 모델 붕괴(collapse) 현상이 발생하거나 나쁜 지역 최솟값(local minima)에 빠지기 쉽다.15

  - **Semi-Hard의 정의:** Semi-hard negative는 Anchor로부터 Positive보다는 멀리 떨어져 있지만($ \lVert f(x^a) - f(x^n) \rVert_2^2 > \lVert f(x^a) - f(x^p) \rVert_2^2 $), 여전히 triplet 제약 조건을 위반하는($ \lVert f(x^a) - f(x^p) \rVert_2^2 + \alpha > \lVert f(x^a) - f(x^n) \rVert_2^2 $) 샘플을 의미한다. 즉, 다음 조건을 만족하는 Negative이다.15
    $$
    \lVert f(x_i^a) - f(x_i^p) \rVert_2^2 < \lVert f(x_i^a) - f(x_i^n) \rVert_2^2 < \lVert f(x_i^a) - f(x_i^p) \rVert_2^2 + \alpha
    $$
    **효과:** 이 전략은 모델이 너무 쉬워서 배울 것이 없는 triplet과, 너무 어려워서 학습을 방해하는 triplet을 모두 회피하게 한다. 대신, 현재 모델의 능력으로 극복 가능한 적절한 난이도의 샘플에 집중하도록 유도하여 학습의 안정성과 수렴 속도를 크게 향상시킨다.

Semi-hard negative mining 전략은 단순한 최적화 기법을 넘어, Triplet Loss 학습 과정에서 발생하는 '안정성-효율성 딜레마'를 해결한 핵심적인 공학적 타협점이다. Triplet Loss의 목표는 클래스 간 경계를 효과적으로 밀어내는 것이며, 이를 위해서는 경계 근처에 있거나 경계를 침범한 샘플, 즉 hard negative에 집중해야 한다. 이론적으로 가장 이상적인 'hardest mining' 전략은 실제 대규모의 노이즈 섞인 데이터셋에서는 불안정하다는 경험적 발견에 기반하여, FaceNet은 실용주의적 접근을 택했다. Semi-hard negative는 안정적인 학습(모델 붕괴 방지)과 효율적인 학습(손실 값 > 0) 사이의 균형을 맞추는 '최적의 학습 구간(sweet spot)'에 있는 샘플들이다. 이들은 현재 모델에게는 도전적이지만, 극복 불가능할 정도로 어렵지는 않다. 결과적으로, semi-hard mining은 Triplet Loss라는 강력한 이론을 실제 문제에 성공적으로 적용 가능하게 만든 결정적인 실용적 해법이라 할 수 있다.


- **vs. Softmax Loss:** Softmax 손실 함수는 훈련 데이터에 존재하는 고정된 수의 클래스를 분류하는 데 최적화되어 있다. 이는 학습 데이터에 없던 새로운 인물이 등장하는 '열린 집합(open-set)' 인식 문제에는 부적합하다.16 반면, Triplet Loss는 특정 클래스 ID에 종속되지 않는 범용적인 메트릭 공간 자체를 학습하므로, 새로운 인물에 대한 확장성이 뛰어나다.
- **vs. Contrastive Loss:** Contrastive Loss는 샘플 '쌍(pair)'을 기반으로 작동한다. 동일 클래스 쌍의 거리는 좁히고, 다른 클래스 쌍의 거리는 특정 마진 이상으로 넓히도록 학습한다.15 이는 각 쌍을 독립적으로 고려하는 '탐욕적(greedy)' 접근 방식이다.15 이에 반해, Triplet Loss는 Anchor를 기준으로 Positive와 Negative의 '상대적 거리(relative distance)'를 동시에 고려한다. 이는 단순히 다른 클래스를 밀어내는 것을 넘어, 클래스 간의 경계를 보다 명확하게 형성하는 데 더 효과적이다.15


FaceNet의 혁신적인 접근법은 당시 안면 인식 분야의 주요 벤치마크에서 전례 없는 성능을 기록하며 그 우수성을 입증했다.


FaceNet의 성능은 두 개의 표준 벤치마크 데이터셋을 통해 주로 평가되었다.

- **Labeled Faces in the Wild (LFW):** LFW는 비제약적인 실제 환경에서 촬영된 13,000개 이상의 얼굴 이미지로 구성된 데이터셋으로, 얼굴 검증(verification) 분야에서 가장 널리 사용되는 표준 벤치마크이다.2
- **YouTube Faces DB (YTF):** YTF는 유튜브 비디오에서 추출한 3,425개의 비디오 클립으로 구성된 데이터셋으로, 비디오 기반의 얼굴 검증 성능을 평가하는 데 사용된다.1


- **LFW 성능:** FaceNet은 LFW 데이터셋의 표준 평가 프로토콜(unrestricted, labeled outside data)에서 **99.63%**라는 경이적인 정확도를 달성했다.6 이 수치는 당시 최고 성능을 기록했던 DeepFace의 97.35% 10와 비교했을 때, 남은 오류율을 30% 이상 극적으로 감소시킨 놀라운 성과였다.8 이 최고 성능은 LFW 이미지에 별도의 얼굴 감지기를 적용하여 얼굴 영역을 정밀하게 추출하는 전처리를 수행했을 때 얻어졌다.8
- **YouTube Faces DB 성능:** 비디오 기반 검증 벤치마크인 YTF에서도 FaceNet은 95.12%라는 높은 정확도를 기록하며, 당시 최고 수준이었던 DeepFace(91.4%)와 DeepID(93.5%)의 성능을 크게 뛰어넘었다.6

FaceNet이 LFW에서 달성한 99.63%의 정확도는 단순한 수치적 향상을 넘어, 안면 인식 기술의 역사에서 상징적인 이정표가 되었다. 당시 인간이 LFW 데이터셋에서 기록하는 정확도는 약 97.5% 수준으로 알려져 있었다.4 DeepFace가 97.35%의 정확도를 달성하며 "인간 수준 성능에 근접했다"고 평가받았지만 22, FaceNet은 이를 훌쩍 뛰어넘어 특정 조건 하에서는 기계가 인간의 얼굴 인식 능력을 능가할 수 있음을 보여주었다. 이러한 '초인적(super-human)' 수준에 가까운 성능은 학계를 넘어 산업계에 큰 반향을 일으켰다. 이전까지 안면 인식이 특정 조건에서만 제한적으로 작동하는 기술로 여겨졌다면, FaceNet의 등장은 스마트폰 잠금 해제, 금융 거래 본인 인증, 물리적 출입 통제 등 높은 신뢰성이 요구되는 실생활 응용 분야로의 기술 확대를 본격적으로 가시화하는 계기가 되었다.23 결과적으로, 99.63%라는 수치는 기술적 성취일 뿐만 아니라, 안면 인식 기술에 대한 사회적 기대와 투자를 촉발하고, 동시에 이후에 논의될 윤리적 문제를 수면 위로 끌어올린 중요한 변곡점(inflection point)이었다.


FaceNet의 Triplet Loss 기반 학습 방식은 모델이 얼굴의 본질적인 정체성(identity) 특징에 집중하고, 조명, 포즈, 가려짐과 같은 비본질적인 변화에는 둔감해지도록 유도한다. FaceNet 논문에서는 이를 시각적으로 증명하는 예시들을 제시했다.9 극단적인 포즈 변화나 심한 조명 차이에도 불구하고, 동일 인물의 얼굴 이미지 쌍은 임베딩 공간에서 매우 가까운 거리(예: 0.78)를 유지한 반면, 외관상 유사해 보이는 다른 인물의 이미지 쌍은 먼 거리(예: 1.33)를 유지했다.9 이는 FaceNet이 학습을 통해 변화에 강인한(robust) 특징 표현을 성공적으로 학습했음을 보여주는 강력한 증거이다.2


FaceNet의 기술적 위치와 기여를 명확히 이해하기 위해서는 이전 세대와 이후 세대의 대표적인 모델들과의 비교 분석이 필수적이다.


- **접근 방식의 차이:** DeepFace는 '정렬 후 분류(Align -> Classify)'라는 명확한 2단계 접근법을 따랐다. 먼저, 정교한 3D 모델을 사용하여 모든 얼굴 이미지를 정면화(frontalization)하는 강력한 정렬 단계를 거친다. 그 후, 정렬된 이미지를 입력으로 받아 수천 개의 클래스(인물)를 분류하도록 학습된 심층 신경망의 중간 계층 출력을 최종 특징 벡터로 사용했다.4 반면, FaceNet은 '직접 임베딩 학습(Direct Embedding Learning)' 방식을 채택했다. 상대적으로 간단한 2D 정렬만을 거친 후, Triplet Loss라는 목표 함수를 통해 임베딩 공간 자체를 엔드투엔드 방식으로 직접 최적화했다.9
- **얼굴 정렬(Alignment) 전략:** DeepFace의 성공에서 3D 정렬은 파이프라인의 핵심적인 요소로 강조되었다.4 이는 정확한 인식을 위해 정교한 전처리가 필수적이라는 당시의 통념을 반영한다. FaceNet은 이러한 통념에 도전했다. 스케일과 평행이동 정도만 보정하는 '대략적인(roughly)' 2D 정렬만으로도 충분히 높은 성능을 달성할 수 있음을 증명한 것이다.9 이는 수백만 장에 달하는 방대한 학습 데이터의 다양성과 강력한 손실 함수가 정교한 전처리의 필요성을 상당 부분 대체할 수 있음을 시사하는 중요한 발견이었다.


- **손실 함수의 진화:** FaceNet의 Triplet Loss가 유클리드 공간에서 '거리(distance)' 기반의 마진을 적용했다면, ArcFace는 손실 함수를 한 단계 더 발전시켰다. ArcFace는 **Additive Angular Margin Loss**라는 새로운 손실 함수를 제안했는데, 이는 특징 벡터들이 놓인 초구(hypersphere) 위에서 '각도(angle)' 기반의 마진을 직접 적용하는 방식이다.26

- **특징 공간의 형태:** FaceNet은 L2 정규화를 통해 모든 임베딩을 초구 표면에 위치시켰지만, 손실 함수 자체는 여전히 유클리드 거리를 최적화했다. ArcFace는 여기서 한 걸음 더 나아가, 특징 공간이 초구라는 기하학적 특성에 착안하여 손실 함수 자체가 초구 위의 두 점 사이의 최단 거리인 측지 거리(geodesic distance), 즉 '각도'를 직접 다루도록 설계했다.27 ArcFace는 Softmax 손실을 변형하여, 특정 클래스에 해당하는 특징 벡터와 가중치 벡터 사이의 각도 

  $ \theta $에 직접 마진 $ m $을 더하는 $ \cos(\theta + m) $ 형태로 로짓(logit)을 계산한다. 이는 모든 각도에 걸쳐 일정하고 선형적인 마진을 보장하여 클래스 간 분리도를 극대화하고, 학습 안정성과 최종 성능을 크게 향상시켰다.27

- **성능:** ArcFace는 FaceNet 이후 등장한 SphereFace, CosFace 등 여러 마진 기반 손실 함수들의 아이디어를 계승하고 발전시켜, LFW 벤치마크에서 99.8% 이상의 정확도를 달성하며 FaceNet의 성능을 뛰어넘는 새로운 기술 표준을 제시했다.30


아래 표는 안면 인식 기술의 패러다임이 어떻게 진화해왔는지를 요약하여 보여준다. DeepFace의 '정렬+분류'에서 FaceNet의 '임베딩+거리', 그리고 ArcFace의 '임베딩+각도'로 이어지는 핵심 아이디어의 발전 과정을 한눈에 파악할 수 있다.

| 특징 (Feature)  | DeepFace (2014)          | FaceNet (2015)                  | ArcFace (2019)                   |
| --------------- | ------------------------ | ------------------------------- | -------------------------------- |
| **핵심 접근법** | 다중 클래스 분류         | 직접 임베딩 학습                | 특징-각도 공간에서의 마진 최대화 |
| **얼굴 정렬**   | 3D 모델 기반 정면화      | 간단한 스케일/이동 변환         | 데이터 증강의 일부로 처리        |
| **특징 벡터**   | 마지막 은닉층(병목) 출력 | 모델의 최종 출력 (128-D)        | L2 정규화된 최종 출력 (512-D)    |
| **손실 함수**   | Softmax Loss             | Triplet Loss                    | Additive Angular Margin Loss     |
| **마진 개념**   | 없음                     | 유클리드 거리 마진 ($ \alpha $) | 각도 공간 마진 ($ m $)           |
| **LFW 정확도**  | 97.35%                   | 99.63%                          | > 99.80%                         |


FaceNet이 제시한 통합 임베딩 접근법은 높은 정확도와 유연성을 바탕으로 안면 인식 기술의 응용 범위를 획기적으로 넓혔다.


일단 FaceNet 모델을 통해 얼굴 이미지를 128차원의 임베딩 벡터로 변환하고 나면, 이 벡터를 표준적인 특징 벡터로 사용하여 다양한 컴퓨터 비전 작업을 손쉽게 구현할 수 있다.6

- **안면 검증 (Verification, 1:1 매칭):** 두 개의 얼굴 이미지가 같은 사람인지를 확인하는 작업이다. 각 이미지로부터 임베딩 벡터를 추출한 뒤, 두 벡터 사이의 L2 거리를 계산한다. 이 거리가 사전에 정의된 임계값(threshold)보다 작으면 동일 인물, 크면 다른 인물로 판단한다.9
- **안면 인식 (Recognition, 1:N 매칭):** 입력된 얼굴 이미지가 데이터베이스에 등록된 인물 중 누구인지를 식별하는 작업이다. 입력 이미지의 임베딩을 계산한 후, 데이터베이스에 저장된 모든 인물의 임베딩 벡터들과의 거리를 비교하여 가장 가까운(거리가 가장 작은) 인물을 찾아낸다.9
- **안면 클러스터링 (Clustering):** 레이블이 없는 대규모 사진 컬렉션에서 동일 인물의 사진들을 자동으로 그룹화하는 작업이다. 모든 사진에서 얼굴 임베딩을 추출한 뒤, k-means나 DBSCAN과 같은 표준 클러스터링 알고리즘을 적용하여 임베딩 공간에서 가까이 모여 있는 벡터들을 같은 그룹으로 묶는다.9


FaceNet의 높은 성능과 범용성은 다양한 상업적 및 개인적 응용의 문을 열었다.

- **보안 및 인증 시스템:** 스마트폰의 얼굴 잠금 해제, 기업의 출입 통제 시스템, 은행 앱의 본인 인증 등 높은 수준의 보안과 신뢰성이 요구되는 분야에서 핵심 기술로 활용된다.23
- **소셜 미디어 및 사진 관리:** 구글 포토와 같은 서비스에서 사진 속 인물을 자동으로 태깅하고, 사용자가 특정 인물의 사진을 쉽게 검색할 수 있도록 앨범을 인물별로 자동 정리하는 데 사용된다.24


FaceNet은 단독으로 사용되기보다는, 전체 안면 인식 파이프라인의 핵심 구성 요소로서 다른 기술들과 유기적으로 결합하여 사용되는 경우가 일반적이다.

- **얼굴 감지기(Face Detector)와의 결합:** 실제 이미지나 비디오 스트림에서 안면 인식을 수행하기 위해서는 먼저 얼굴의 위치와 크기를 찾아내는 얼굴 감지 단계가 선행되어야 한다. 이를 위해 MTCNN, YOLO, MediaPipe, Dlib과 같은 고성능 얼굴 감지기들이 FaceNet의 전처리 단계로 널리 사용된다.3
- **분류기(Classifier)와의 결합:** 특정 그룹(예: 한 회사의 직원들) 내에서 인물을 분류하는 닫힌 집합 문제의 경우, FaceNet으로 추출한 128차원 임베딩을 입력 특징으로 사용하여 SVM(Support Vector Machine), KNN(K-Nearest Neighbors)과 같은 전통적인 머신러닝 분류기를 학습시키기도 한다. 이는 특정 데이터셋에 대한 분류 성능을 더욱 최적화하는 데 도움이 될 수 있다.3

FaceNet의 등장은 안면 인식 기술 개발 패러다임을 '모놀리식(monolithic)' 시스템에서 '모듈화(modularized)'된 파이프라인으로 전환시키는 데 결정적인 기여를 했다. 과거의 시스템들이 감지부터 인식까지의 모든 과정이 긴밀하게 결합된 단일체로 개발되었다면, FaceNet은 '얼굴 임베딩 생성기'라는 매우 명확하고 범용적인 역할을 정의했다.6 이 임베딩 벡터(128-D vector)는 그 자체로 표준화된 데이터 인터페이스 역할을 한다. 이로 인해 개발자들은 파이프라인의 다른 구성 요소들을 마치 레고 블록처럼 자유롭게 교체하고 최적화할 수 있게 되었다. 예를 들어, 실시간성이 중요한 응용에서는 가벼운 감지기인 MediaPipe를, 정확성이 최우선인 환경에서는 무거운 감지기인 YOLO를 FaceNet 앞에 결합할 수 있다.3 마찬가지로, FaceNet이 추출한 임베딩 뒤에는 간단한 거리 비교 로직(검증)을 붙일 수도 있고, 복잡한 SVM 분류기(특정 그룹 내 인식)를 연결할 수도 있다.3 이러한 모듈화는 각 분야의 전문가들이 자신의 전문 영역(감지, 임베딩, 분류)에 집중하여 기술을 발전시키고, 이를 유연하게 조합하여 최적의 시스템을 구축할 수 있는 건강한 기술 생태계를 조성했다. FaceNet은 이 생태계의 핵심 '엔진' 역할을 수행하며 기술 발전을 가속화했다.


FaceNet은 안면 인식 기술의 비약적인 발전을 이끌었지만, 동시에 여러 기술적 한계와 심각한 윤리적 문제를 드러냈다.


FaceNet, 특히 논문에서 제안된 초기 모델들은 수백만 개의 파라미터를 가지며 상당한 연산량을 요구했다. 이로 인해 스마트폰이나 CCTV 카메라와 같은 리소스가 제한된 모바일 및 임베디드 기기에 직접 배포하기에는 어려움이 따랐다.2 이후 MobileFaceNet과 같이 모델을 경량화하려는 많은 후속 연구가 진행되었지만, 이러한 시도는 종종 모델 크기와 정확도 사이의 트레이드오프 관계를 수반하여, 크기를 줄이는 대신 정확도가 하락하는 결과를 낳기도 했다.2


FaceNet과 같은 데이터 주도(data-driven) 방식의 성공은 전적으로 대규모의, 잘 정제되고 레이블링된 학습 데이터셋의 가용성에 의존한다.10 수백만 장에 달하는 고품질의 얼굴 이미지 데이터셋을 구축하는 것은 막대한 비용과 시간이 소요되는 작업이다. 또한, 학습 데이터셋의 품질, 다양성, 그리고 균형이 모델의 최종 성능과 강건성을 결정짓는 핵심적인 요인이 된다.


- **문제 현상:** FaceNet을 비롯한 많은 안면 인식 기술은 인종, 성별, 연령과 같은 인구통계학적 그룹에 따라 정확도에서 현저한 차이를 보이는 '알고리즘 편향' 문제를 안고 있다.34 다수의 연구 결과에 따르면, 이러한 시스템들은 백인 남성에 대해서는 매우 높은 정확도를 보이지만, 유색인종, 특히 어두운 피부색을 가진 여성에게서는 오류율이 급격히 증가하는 경향이 뚜렷하게 나타난다.35
- **발생 원인:** 이러한 편향이 발생하는 가장 근본적인 원인은 훈련 데이터의 불균형에 있다.34 대부분의 대규모 얼굴 데이터셋은 웹에서 수집되는데, 이는 현실 세계의 인구 분포를 공정하게 반영하지 못하고 특정 인구 집단(예: 백인 남성)의 이미지를 과대 대표하는 경향이 있다. 모델은 데이터에서 다수를 차지하는 그룹의 특징은 잘 학습하지만, 상대적으로 데이터가 부족한 소수 집단의 특징은 제대로 학습하지 못한다. 이는 결국 데이터 수집 과정에 내재된 사회적, 역사적 편견이 기술에 그대로 투영되고 증폭되는 결과를 초래한다.35


- **프라이버시 침해:** 안면 인식 기술의 확산은 심각한 개인정보보호 문제를 야기한다. 이 기술은 개인의 명시적인 동의 없이 공공장소에서 무차별적으로 사람들의 신원을 식별하고, 그들의 동선을 추적하는 데 사용될 수 있다. 이는 헌법이 보장하는 익명으로 살아갈 권리와 프라이버시권을 근본적으로 위협한다.35
- **사회적 통제:** 정부나 법 집행 기관이 이러한 기술을 대규모 감시 시스템에 도입할 경우, 이는 국민을 통제하고 억압하는 강력한 도구로 악용될 위험이 있다. 집회나 시위에 참여한 사람들을 식별하거나, 특정 정치적 견해를 가진 사람들을 감시하는 데 사용될 수 있으며, 이는 표현의 자유와 민주주의의 근간을 위축시키는 '냉각 효과(chilling effect)'를 가져올 수 있다.35 특히, 이민자나 사회적 소수자와 같은 취약 계층을 표적으로 삼아 불공정한 단속과 차별을 가하는 데 사용될 수 있다는 심각한 우려가 지속적으로 제기된다.35

FaceNet의 기술적 성공과 윤리적 실패는 동전의 양면과 같다. FaceNet의 경이로운 성능은 수백만 장의 얼굴 이미지로 구성된 거대한 데이터셋을 통해 학습함으로써 가능했다.40 그러나 이 성공의 핵심 요인이었던 '대규모 데이터'가 바로 '데이터 편향'이라는 가장 심각한 실패의 근본 원인이기도 하다. 웹에서 크롤링된 데이터는 현실 세계의 인구 분포를 공정하게 반영하지 못하며, 특정 인종과 성별이 과대 대표되는 경향이 있다.36 이러한 불균형한 데이터로 학습된 모델은 자연스럽게 데이터에서 다수를 차지하는 그룹에 대해서는 높은 성능을, 소수 그룹에 대해서는 낮은 성능을 보이게 된다.34 결국, FaceNet의 성능을 높인 '데이터의 양(quantity)'이 동시에 공정성을 해치는 '데이터의 편향(bias)'을 야기한 것이다. 이는 기술적 최적화가 사회적 공정성을 자동으로 보장하지 않으며, 때로는 오히려 기존의 불평등을 악화시킬 수 있음을 보여주는 강력한 사례이다. 이 딜레마는 단순히 더 많은 데이터를 추가하는 것만으로는 해결될 수 없으며, 데이터 수집 단계에서부터 공정성을 고려하는 근본적인 성찰과 새로운 학습 방법론의 개발이 필요함을 시사한다.



Google의 FaceNet은 안면 인식 연구의 역사에서 하나의 분기점을 형성했다. 이 시스템은 안면 인식 문제를 전통적인 '분류'의 틀에서 벗어나 '메트릭 학습'의 관점으로 재정의했으며, 이를 효과적으로 구현하기 위해 Triplet Loss와 온라인 마이닝이라는 강력한 방법론을 제시했다. 이로 인해 연구의 흐름은 단순히 더 깊은 네트워크를 쌓아 분류 정확도를 높이는 경쟁에서, 더 우수한 임베딩 공간을 설계하고 학습하는 방향으로 전환되었다. FaceNet의 성공은 이후 ArcFace와 같이 마진 기반의 다양한 손실 함수들이 활발히 연구되고 발전하는 직접적인 기폭제가 되었다.


FaceNet이 제시한 '통합 임베딩' 접근법은 이제 안면 인식을 넘어, 유사 이미지 검색, 보행자 재인식(person re-identification), 객체 추적 등 다양한 컴퓨터 비전 분야에서 표준적인 방법론 중 하나로 확고히 자리 잡았다. 미래의 연구는 임베딩의 성능을 높이는 것을 넘어, 임베딩 벡터의 각 차원이 어떤 시각적 특징(예: 코의 모양, 눈썹의 각도)에 해당하는지 설명할 수 있는 해석 가능성(interpretability)을 높이는 방향으로 나아갈 것이다. 또한, 수백만 장의 데이터 없이도 소수의 데이터만으로 강건한 임베딩을 학습할 수 있는 퓨샷 학습(few-shot learning) 및 자기 지도 학습(self-supervised learning) 연구가 더욱 중요해질 전망이다.


FaceNet의 눈부신 기술적 성공은 역설적으로 안면 인식 기술의 어두운 그림자와 사회적 책임에 대한 중요한 질문을 우리 사회에 던졌다. 미래의 안면 인식 기술은 단순히 벤치마크 점수를 1% 더 높이는 것을 목표로 삼아서는 안 된다. 대신, 기술이 특정 인구 집단에 불이익을 주지 않도록 알고리즘의 편향을 적극적으로 완화하고(fairness), 개인의 프라이버시를 침해하지 않도록 설계 단계부터 보호하며(privacy), 기술이 어떻게 작동하고 왜 그런 결정을 내렸는지 투명하게 설명할 수 있는(explainability) 방향으로 나아가야 한다. 이는 단순한 기술적 도전 과제를 넘어, 개발자, 정책 입안자, 그리고 시민 사회가 함께 머리를 맞대고 사회적 합의를 이루어 나가야 할 우리 시대의 중대한 과제이다.34


1. [PDF] FaceNet: A unified embedding for face recognition and clustering | Semantic Scholar, 8월 18, 2025에 액세스, https://www.semanticscholar.org/paper/FaceNet%3A-A-unified-embedding-for-face-recognition-Schroff-Kalenichenko/5aa26299435bdf7db874ef1640a6c3b5a4a2c394
2. Lightweight and Resource-Constrained Learning Network for Face Recognition with Performance Optimization - PubMed Central, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC7662273/
3. Efficient Face Recognition System for Operating in Unconstrained Environments - PMC, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8466208/
4. DeepFace: Closing the Gap to Human-Level Performance in Face ..., 8월 18, 2025에 액세스, https://www.cs.toronto.edu/~ranzato/publications/taigman_cvpr14.pdf
5. Face Authentication using MTCNN and FaceNet - IJRASET, 8월 18, 2025에 액세스, https://www.ijraset.com/research-paper/face-authentication-using-mtcnn-and-facenet
6. FaceNet: A Unified Embedding for Face Recognition and Clustering - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/273471270_FaceNet_A_Unified_Embedding_for_Face_Recognition_and_Clustering
7. (PDF) FaceNet: A unified embedding for face recognition and clustering (2015) | Florian Schroff | 8289 Citations - SciSpace, 8월 18, 2025에 액세스, https://scispace.com/papers/facenet-a-unified-embedding-for-face-recognition-and-135p2gog88
8. FaceNet - Using Facial Recognition System - GeeksforGeeks, 8월 18, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/facenet-using-facial-recognition-system/
9. FaceNet: A Unified Embedding for Face Recognition and Clustering, 8월 18, 2025에 액세스, http://arxiv.org/pdf/1503.03832
10. FaceNet: A Unified Embedding for Face Recognition and Clustering - The Computer Vision Foundation, 8월 18, 2025에 액세스, https://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Schroff_FaceNet_A_Unified_2015_CVPR_paper.pdf
11. DeepFace: Closing the Gap to Human-Level Performance in Face Verification - Meta AI, 8월 18, 2025에 액세스, https://ai.meta.com/research/publications/deepface-closing-the-gap-to-human-level-performance-in-face-verification/
12. DeepFace: Closing the Gap to Human-Level Performance in Face Verification, 8월 18, 2025에 액세스, https://www.semanticscholar.org/paper/DeepFace%3A-Closing-the-Gap-to-Human-Level-in-Face-Taigman-Yang/9f2efadf66817f1b38f58b3f50c7c8f34c69d89a
13. FaceNet, 8월 18, 2025에 액세스, http://llcao.net/cu-deeplearning17/pp/class10_FaceNet.pdf
14. Face Recognition | Keras, FaceNet, Inception model - Kaggle, 8월 18, 2025에 액세스, https://www.kaggle.com/code/mishki/face-recognition-keras-facenet-inception-model
15. Triplet loss - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Triplet_loss
16. Using Triplet Loss for Face Recognition Systems - DEV Community, 8월 18, 2025에 액세스, https://dev.to/endlessmessages/using-triplet-loss-for-face-recognition-systems-ne5
17. Triplet Loss: Intro, Implementation, Use Cases - V7 Labs, 8월 18, 2025에 액세스, https://www.v7labs.com/blog/triplet-loss
18. Triplet Loss - Advanced Intro - Towards Data Science, 8월 18, 2025에 액세스, https://towardsdatascience.com/triplet-loss-advanced-intro-49a07b7d8905/
19. TensorFlow Addons Losses: TripletSemiHardLoss, 8월 18, 2025에 액세스, https://www.tensorflow.org/addons/tutorials/losses_triplet
20. FaceNet | Keras - Kaggle, 8월 18, 2025에 액세스, https://www.kaggle.com/datasets/utkarshsaxenadn/facenet-keras
21. [1503.03832] FaceNet: A Unified Embedding for Face Recognition and Clustering - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/1503.03832
22. DeepFace: Closing the Gap to Human-Level Performance in Face Verification - SciSpace, 8월 18, 2025에 액세스, https://scispace.com/papers/deepface-closing-the-gap-to-human-level-performance-in-face-3wcmadzjn3
23. Making Face Recognition Smarter with FaceNet & Mediapipe - StatusNeo, 8월 18, 2025에 액세스, https://statusneo.com/facenet-and-mediapipe-powerful-face-recognition-integration/
24. Face Recognition with FaceNet & Dlib: AI Guide - Mue AI, 8월 18, 2025에 액세스, https://www.muegenai.com/docs/datascience/computer_vision_applications/advanced_topics/face_recognition
25. FACE AUTHENTICATION USING YOLOV8 AND FACENET - International Journal of Scientific Research and Engineering Development, 8월 18, 2025에 액세스, https://www.ijsred.com/volume8/issue3/IJSRED-V8I3P222.pdf
26. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/352263965_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition
27. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/322674945_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition
28. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - InsightFace, 8월 18, 2025에 액세스, https://insightface.ai/arcface
29. ArcFace: Additive Angular Margin Loss for Deep ... - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content_CVPR_2019/papers/Deng_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition_CVPR_2019_paper.pdf
30. Comparison of Face Recognition Accuracy of ArcFace, Facenet and Facenet512 Models on Deepface Framework (2023) | Andrian Muzakki Firmansyah | 5 Citations - SciSpace, 8월 18, 2025에 액세스, https://scispace.com/papers/comparison-of-face-recognition-accuracy-of-arcface-facenet-o614ksrw
31. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - YouTube, 8월 18, 2025에 액세스, https://www.youtube.com/watch?v=H1qEp_czI1I
32. Research and Analysis of Facial Recognition Based on FaceNet, DeepFace, and OpenFace - ITM Web of Conferences, 8월 18, 2025에 액세스, https://www.itm-conferences.org/articles/itmconf/pdf/2025/01/itmconf_dai2024_03009.pdf
33. Research and Analysis of Facial Recognition Based on FaceNet, DeepFace, and OpenFace, 8월 18, 2025에 액세스, https://www.itm-conferences.org/articles/itmconf/abs/2025/01/itmconf_dai2024_03009/itmconf_dai2024_03009.html
34. Review of Demographic Bias in Face Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2502.02309v1
35. Biased Technology: The Automated Discrimination of Facial ..., 8월 18, 2025에 액세스, https://www.aclu-mn.org/en/news/biased-technology-automated-discrimination-facial-recognition
36. The Problem of Bias in Facial Recognition | Strategic Technologies Blog - CSIS, 8월 18, 2025에 액세스, https://www.csis.org/blogs/strategic-technologies-blog/problem-bias-facial-recognition
37. NIST Study Evaluates Effects of Race, Age, Sex on Face Recognition Software, 8월 18, 2025에 액세스, https://www.nist.gov/news-events/news/2019/12/nist-study-evaluates-effects-race-age-sex-face-recognition-software
38. The ethics of facial recognition technologies, surveillance, and accountability in an age of artificial intelligence: a comparative analysis of US, EU, and UK regulatory frameworks, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8320316/
39. Facial Recognition Algorithms: A Systematic Literature Review - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2313-433X/11/2/58
40. DeepFace: Closing the Gap to Human-Level Performance in Face Verification | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/263564119_DeepFace_Closing_the_Gap_to_Human-Level_Performance_in_Face_Verification
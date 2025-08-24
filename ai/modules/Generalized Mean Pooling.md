# 일반화 평균 풀링(Generalized Mean Pooling)
[딥러닝 모듈 (Modules)](./index.md)



컨볼루션 신경망(Convolutional Neural Networks, CNN) 아키텍처에서 풀링(pooling) 계층은 합성곱 계층과 함께 핵심적인 구성 요소로 기능한다. 풀링의 가장 기본적인 역할은 합성곱 계층을 통과한 특징 맵(feature map)의 공간적 차원(spatial dimensions), 즉 너비(width)와 높이(height)를 점진적으로 줄이는 것이다.1 이러한 다운샘플링(downsampling) 과정은 여러 가지 중요한 이점을 제공한다. 첫째, 특징 맵의 크기를 줄임으로써 후속 계층에서 처리해야 할 파라미터의 수를 크게 감소시킨다. 이는 모델의 전체적인 계산 복잡도를 낮추고 학습 속도를 향상시키는 데 기여한다.1 둘째, 파라미터 수를 줄이는 것은 모델이 훈련 데이터에 과도하게 최적화되는 과적합(overfitting) 현상을 억제하는 효과적인 정규화(regularization) 기법으로 작용한다.1

차원 축소 외에도 풀링은 모델에 국소적 변환 불변성(local translation invariance)을 부여하는 중요한 역할을 수행한다.2 이는 입력 이미지 내에서 객체가 약간 이동하거나 회전하는 등의 미세한 변위가 발생하더라도, 풀링 연산을 통해 추출된 특징은 상대적으로 안정적으로 유지됨을 의미한다. 풀링 윈도우 내에서 특징들을 하나의 대표값으로 요약함으로써, 특징의 정확한 위치 정보보다는 존재 유무에 더 집중하게 되기 때문이다. 이러한 불변성은 객체 탐지나 분류와 같이 객체의 위치보다는 존재 자체가 중요한 태스크에서 모델의 강건함(robustness)을 높이는 데 결정적인 역할을 한다.


전통적으로 가장 널리 사용되는 풀링 기법은 최대 풀링(Max Pooling)과 평균 풀링(Average Pooling)이다. 이 두 기법은 특징 맵의 정보를 집약(aggregate)하는 방식에서 근본적인 차이를 보인다.

**최대 풀링(Max Pooling)**, 또는 MAC(Maximum Activations of Convolutions)은 풀링 윈도우 내에서 가장 큰 활성화 값을 선택하는 방식이다.6 이 방식은 이미지의 가장 두드러지고(salient) 날카로운 특징, 예를 들어 엣지(edge), 코너(corner), 혹은 밝은 픽셀 등을 효과적으로 추출한다.3 특정 객체를 식별하는 데 있어 핵심적인 단서가 되는 소수의 강력한 특징을 보존하는 데 유리하다. 그러나 최대 풀링은 가장 큰 활성화 값 하나를 제외한 나머지 모든 정보를 버리기 때문에, 윈도우 내의 다른 유용한 컨텍스트 정보가 손실될 수 있다는 명백한 단점을 가진다.3

**평균 풀링(Average Pooling)**, 또는 SPoC(Sum-Pooled Convolutional features)는 윈도우 내 모든 활성화 값의 산술 평균을 취한다.6 이 방식은 윈도우 내의 모든 정보를 종합하여 부드럽고(smoothed) 일반화된 특징 표현을 생성한다.7 이미지의 전반적인 질감(texture)이나 분포와 같은 컨텍스트를 보존하는 데 유리하며, 최대 풀링에 비해 노이즈에 덜 민감하다. 하지만 모든 값을 평균내는 과정에서 가장 두드러진 특징의 신호 강도가 희석되어, 식별력이 높은 핵심 특징의 중요도가 약화될 수 있다.8

이 두 전통적인 풀링 방식은 '특징 선택(selection)'과 '특징 요약(summarization)'이라는 두 가지 상반된 철학 사이의 근본적인 트레이드오프를 보여준다. 최대 풀링은 가장 강력한 신호 하나를 선택하여 희소하고(sparse) 강렬한 표현을 만들지만 주변 컨텍스트를 잃는다. 반면, 평균 풀링은 모든 신호를 요약하여 조밀하고(dense) 부드러운 표현을 만들지만 핵심 신호의 강도를 약화시킨다. 문제는 특정 태스크나 데이터셋에 따라 최적의 정보 집약 전략이 달라진다는 점이다. 인스턴스 수준의 이미지 검색에서는 특정 랜드마크의 고유한 첨탑과 같은 국소적 특징이 중요하므로 최대 풀링이 유리할 수 있지만, 일반적인 이미지 분류에서는 전반적인 질감 패턴이 더 중요하여 평균 풀링이 더 강건할 수 있다. 이처럼 고정된(fixed) 연산 방식은 데이터나 태스크의 특성에 유연하게 적응할 수 없다는 본질적인 한계를 드러낸다.11 바로 이 지점에서, 두 극단적인 전략 사이의 스펙트럼을 탐색하고 데이터로부터 최적의 풀링 방식을 학습할 수 있는 일반화된 프레임워크의 필요성이 대두되었고, 이는 일반화 평균 풀링(Generalized Mean Pooling, GeM)의 탄생으로 이어졌다.



GeM 풀링의 수학적 기반은 일반화 평균(Generalized Mean), 또는 거듭제곱 평균(Power Mean)으로 알려진 개념에 있다. 일반화 평균은 주어진 양수 집합에 대한 평균을 계산하는 하나의 통일된 프레임워크를 제공한다. 이는 우리가 흔히 사용하는 산술 평균(arithmetic mean), 기하 평균(geometric mean), 조화 평균(harmonic mean) 등을 모두 특수한 경우로 포함하는 더 넓은 개념이다. 일반화 평균은 실수 $p$를 파라미터로 가지며, 이 $p$ 값에 따라 평균의 성격이 달라진다. GeM 풀링은 바로 이 일반화 평균의 개념을 CNN의 특징 맵 집약 과정에 적용한 것이다.


GeM 풀링은 CNN의 `k`번째 특징 맵 $\mathcal{X}_k$가 주어졌을 때, 각 채널별로 하나의 스칼라 값을 출력한다. 그 결과 벡터 $\mathbf{f}$의 `k`번째 요소 $f_k$는 다음과 같이 수학적으로 정의된다.13
$$
f_k = \left( \frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x^p \right)^{\frac{1}{p}}
$$
여기서 $x$는 특징 맵 $\mathcal{X}_k$ 내에 있는 각각의 활성화(activation) 값을 의미하며, `ReLU` 활성화 함수를 통과했다고 가정하여 모든 $x$는 음수가 아니다.12

$|\mathcal{X}_k|$는 해당 특징 맵의 공간적 크기, 즉 총 활성화의 개수(`H x W`)이다.

이 공식에서 가장 핵심적인 요소는 파라미터 $p$이다. 이 $p$는 풀링 연산의 동작 방식을 결정하는 '조절 손잡이(knob)' 역할을 한다.

- $p$ 값이 클수록, 공식 내의 $x^p$ 항은 값이 큰 $x$에 대해 기하급수적으로 커진다. 결과적으로 전체 합($\sum x^p$)에서 값이 큰 소수의 활성화가 대부분을 차지하게 되고, 값이 작은 활성화의 영향력은 거의 무시된다. 이는 네트워크가 소수의 두드러진, 즉 가장 활성화가 강한 특징에 집중하도록 유도한다.16
- 반대로 $p$ 값이 작을수록(예: 1에 가까울수록), 각 활성화 값 $x$는 $x^p$ 항에서 비교적 균등한 영향력을 갖게 된다. 이는 네트워크가 특정 값에 치우치지 않고 윈도우 내의 모든 활성화 정보를 전반적으로 고려하도록 만든다.


GeM 풀링의 가장 중요한 특징은 파라미터 $p$의 극한값을 통해 기존의 최대 풀링과 평균 풀링을 완벽하게 표현할 수 있다는 점이다. 즉, GeM은 두 풀링 기법을 포괄하는 일반화된 프레임워크이다.13

- **평균 풀링 ($p=1$)**: 파라미터 $p$에 1을 대입하면, GeM 공식은 다음과 같이 변형된다.
  $$
  f_k = \left( \frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x^1 \right)^{\frac{1}{1}} = \frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x
  $$
  이는 정확히 평균 풀링(Average Pooling)의 정의와 일치한다.13

- **최대 풀링 ($p \to \infty$)**: 파라미터 $p$가 무한대로 발산하는 극한을 취하면, GeM 공식은 $L_p$ 노름(norm)의 성질에 따라 윈도우 내에서 가장 큰 값으로 수렴한다.
  $$
  \lim_{p \to \infty} \left( \frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x^p \right)^{\frac{1}{p}} = \max_{x \in \mathcal{X}_k}(x)
  $$
  이는 최대 풀링(Max Pooling)의 결과와 동일하다.10

이러한 특성은 GeM을 단순한 공학적 트릭이 아닌, $L_p$-공간 이론에 기반한 수학적으로 잘 정의된 연산으로 격상시킨다. GeM은 사실상 $L_p$-노름 풀링 연산으로, 평균 풀링은 $L_1$-노름(의 스케일 버전)에, 최대 풀링은 $L_∞$-노름에 해당한다. $p$는 이 $L_1$과 $L_∞$ 노름 사이의 연속적인 스펙트럼을 탐색하는 파라미터가 된다. 더 중요한 것은, $L_p$-노름이 $p$와 입력 $x$에 대해 (거의 모든 곳에서) 미분 가능하다는 사실이다. 바로 이 미분 가능성이 $p$를 경사 하강법 기반의 딥러닝 프레임워크 내에서 학습 가능한 파라미터로 만들 수 있는 이론적 근거를 제공하며, GeM이 고정된 연산자를 넘어 '적응형' 풀링 계층으로 기능할 수 있게 하는 핵심 열쇠이다.



GeM 풀링의 유연성은 파라미터 $p$를 고정된 하이퍼파라미터로 사용하는 것만으로도 발휘될 수 있다. 실제로 이미지 검색 분야의 여러 실험에서 $p=3$과 같은 특정 값을 고정하여 사용했을 때, 최대 풀링이나 평균 풀링보다 일관되게 우수한 성능을 보였다.11 그러나 GeM의 진정한 잠재력은 $p$를 네트워크의 다른 가중치(weight)나 편향(bias)처럼 학습 가능한(learnable) 파라미터로 취급할 때 발현된다.13

$p$를 학습 가능하게 만드는 것의 핵심 동기는 모델이 데이터와 주어진 태스크에 가장 적합한 정보 집약 전략을 스스로 찾도록 하는 데 있다. CNN의 각 채널은 서로 다른 종류의 시각적 패턴(예: 특정 질감, 색상, 혹은 복잡한 형태)을 감지하도록 학습된다. 어떤 채널에서는 전반적인 패턴의 평균적인 활성화가 중요할 수 있고(평균 풀링에 가까운 전략, 낮은 $p$), 다른 채널에서는 특정 객체의 결정적인 부분에 대한 강한 활성화가 중요할 수 있다(최대 풀링에 가까운 전략, 높은 $p$). $p$를 학습시킴으로써, 네트워크는 손실 함수를 최소화하는 과정에서 각 채널(또는 전체 계층)에 가장 이상적인 풀링 방식을 데이터 기반으로 결정하게 된다.12 이는 수동적인 하이퍼파라미터 튜닝의 필요성을 줄이고, 모델의 표현력을 극대화하여 최종적으로 성능 향상을 이끌어낸다.13


GeM 연산은 $p$와 입력 $x$에 대해 미분 가능하므로, 표준적인 역전파(backpropagation) 알고리즘을 통해 $p$에 대한 그래디언트를 계산하고 경사 하강법(gradient descent)으로 업데이트할 수 있다.15 손실 함수 $L$에 대한 $p$의 그래디언트는 연쇄 법칙(chain rule)에 따라 계산된다.

먼저, GeM의 출력 $f_k$에 대한 입력 활성화 $x_i$의 편미분($\frac{\partial f_k}{\partial x_i}$)은 다음과 같다.15
$$
\frac{\partial f_k}{\partial x_i} = \frac{1}{|\mathcal{X}_k|} (f_k)^{1-p} x_i^{p-1}
$$
다음으로, 출력 $f_k$에 대한 파라미터 $p$의 편미분($\frac{\partial f_k}{\partial p}$)은 로그 미분법을 이용하여 유도할 수 있으며, 그 결과는 다음과 같다.15
$$
\frac{\partial f_k}{\partial p} = \frac{\partial}{\partial p} \exp\left(\frac{1}{p} \ln\left(\frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x^p\right)\right) = \frac{f_k}{p^2} \left( \frac{\sum_{x \in \mathcal{X}_k} x^p \ln(x)}{\sum_{x \in \mathcal{X}_k} x^p} - \ln(f_k) \right)
$$
최종적으로 손실 함수 $L$에 대한 $p$의 그래디언트는 $\frac{\partial L}{\partial p} = \sum_k \frac{\partial L}{\partial f_k} \frac{\partial f_k}{\partial p}$로 계산된다. 이 그래디언트 값을 이용하여 옵티마이저는 손실을 줄이는 방향으로 $p$ 값을 자동으로 조정한다.20


실제 학습 과정에서 $p$ 값의 동역학을 관찰하면 흥미로운 현상을 발견할 수 있다. 대부분의 이미지 인식 태스크에서 $p$는 1보다 큰 값으로 수렴하는 경향을 보인다. 이는 순수한 평균 풀링보다는 어느 정도 두드러진 특징을 강조하는, 즉 최대 풀링에 가까운 전략이 더 효과적임을 경험적으로 뒷받침한다.12

그러나 GeM과 특정 종류의 손실 함수를 결합할 때 예상치 못한 부작용이 발생할 수 있다는 점이 중요한 연구를 통해 밝혀졌다. 특히, ArcFace와 같이 특징 벡터 간의 각도 마진을 최대화하는 마진 기반 분류 손실(margin-based classification loss)과 학습 가능한 GeM을 함께 사용하면, 학습된 $p$ 값이 경험적으로 알려진 최적값에서 벗어나는 현상이 관찰되었다.14 한 연구에서는 ArcFace의 마진 값을 증가시킬수록, 학습된 $p$ 값이 최적값으로 알려진 4.0에서 점점 더 멀어지는 것을 시각적으로 보여주었다.14

이 현상은 두 최적화 목표 간의 간섭으로 설명될 수 있다. ArcFace 손실 함수는 최종적으로 생성된 전역 디스크립터 벡터들이 하이퍼스피어(hypersphere) 상에서 클래스별로 잘 분리되도록, 즉 벡터의 '방향'을 최적화하는 데 강력한 그래디언트 압력을 가한다. 반면, GeM의 $p$ 파라미터는 특징 맵 내 활성화 값들의 '크기' 분포를 어떻게 집약하여 디스크립터를 생성할지를 결정한다. 이 두 최적화 과정이 충돌할 때, ArcFace의 강력한 제약 조건을 만족시키기 위해 옵티마이저가 $p$ 값을 조절하는 '손쉬운 길'을 택할 수 있다. 예를 들어, 특징 맵의 활성화를 극단적으로 뾰족하게 만들거나(큰 $p$) 평탄하게 만들어(작은 $p$) 일시적으로 손실을 낮출 수 있지만, 이는 검색 태스크 자체에 가장 식별력 있는 디스크립터를 생성하는 최적의 $p$ 값과는 다를 수 있다. 이 발견은 손실 함수와 풀링 계층 간의 복잡한 상호작용을 이해하고, 필요에 따라 $p$의 학습을 분리하거나 제약을 가하는 등의 새로운 최적화 전략의 필요성을 시사한다.



이미지 검색(Image Retrieval)은 주어진 질의(query) 이미지와 의미적으로 유사한 이미지들을 대규모 데이터베이스에서 찾아내는 컴퓨터 비전의 핵심 분야이다.21 특히, 특정 건물, 상품, 예술 작품과 같이 고유한 객체 인스턴스를 찾는 인스턴스 수준 이미지 검색(Instance-level Image Retrieval)은 매우 도전적인 과제이다.16 성공적인 인스턴스 검색 시스템은 동일한 객체라도 시점(viewpoint), 조명, 배율(scale)이 크게 다르거나, 다른 객체에 의해 일부가 가려지거나(occlusion), 복잡한 배경 속에 존재하는 등 다양한 변형에도 강건한 이미지 표현(representation), 즉 디스크립터(descriptor)를 추출할 수 있어야 한다.24 이러한 강건한 디스크립터를 생성하는 것이 바로 이 분야의 핵심 과제이다.


CNN 기반 이미지 검색 시스템은 일반적으로 네트워크의 마지막 합성곱 계층에서 출력된 고차원 특징 맵을 풀링 연산을 통해 하나의 고정된 크기의 벡터, 즉 전역 디스크립터(global descriptor)로 압축한다.14 이 전역 디스크립터의 품질이 검색 시스템의 전체 성능을 좌우한다.

GeM 풀링은 이 과정에서 탁월한 성능을 발휘한다. 학습 가능한 파라미터 $p$를 통해 GeM은 이미지의 두드러진(salient) 특징에 선택적으로 집중하는 능력을 갖는다. $p$ 값을 높이면, 특징 맵에서 활성화가 강하게 나타나는 소수의 공간적 위치에 더 큰 가중치를 부여하게 된다. 이는 검색 대상 객체와 관련된 식별력 높은 특징은 강조하고, 무관한 배경(background clutter)이나 비정보적인 영역의 영향은 효과적으로 억제하는 결과를 낳는다.16 이러한 '선택적 집중' 능력은 기존의 최대 풀링이나 평균 풀링이 제공하지 못하는 유연성을 부여하며, 결과적으로 훨씬 더 식별력 있고 강건한 전역 디스크립터를 생성하게 하여 이미지 검색 성능을 극적으로 향상시킨다.15

GeM은 Radenović 등이 2018년에 발표한 "Fine-tuning CNN Image Retrieval with No Human Annotation" 논문에서 처음 제안된 이후, 이미지 검색 및 랜드마크 인식 분야에서 사실상의 표준(de facto standard) 풀링 기법으로 빠르게 자리 잡았다.15


GeM의 실증적 우수성은 여러 표준 이미지 검색 벤치마크 데이터셋에서 일관되게 입증되었다. 특히, 실제와 유사한 어려운 검색 시나리오를 포함하는 Revisited Oxford 5k (ROxford5k)와 Revisited Paris 6k (RParis6k) 데이터셋에서 GeM은 기존 풀링 기법들을 큰 차이로 능가하는 성능을 보였다.24

아래 표는 ResNet101과 VGG16 아키텍처를 기반으로 여러 풀링 기법의 성능을 평균 정밀도(mean Average Precision, mAP)로 비교한 결과를 요약한 것이다. 이 결과들은 GeM, 특히 학습 가능한 단일 파라미터 $p$를 사용한 GeM이 다른 기법들에 비해 일관되게 높은 검색 정확도를 달성함을 명확하게 보여준다.15

| Backbone  | Pooling | ROxford5k (M) | ROxford5k (H) | RParis6k (M) | RParis6k (H) |
| --------- | ------- | ------------- | ------------- | ------------ | ------------ |
| ResNet101 | MAC     | -             | -             | -            | -            |
| ResNet101 | SPoC    | -             | -             | -            | -            |
| ResNet101 | GeM     | **65.4**      | **40.1**      | **76.7**     | **55.2**     |
| VGG16     | MAC     | -             | -             | -            | -            |
| VGG16     | GeM     | **60.9**      | **32.9**      | **69.3**     | **44.2**     |

GeM의 성공은 이미지 검색 시스템의 설계 패러다임 자체에 중요한 변화를 가져왔다. 전통적인 고성능 검색 시스템은 두 단계로 구성되었다: 1) 전역 디스크립터를 이용한 빠른 초기 검색, 2) 상위 검색 결과에 대해 비용이 많이 드는 지역 특징(local feature) 매칭을 수행하여 순위를 재조정하는 재순위화(re-ranking).14 이 재순위화 단계는 정확도를 높이는 데 필수적이었지만, 지역 특징을 저장하고 매칭하는 데 막대한 저장 공간과 계산 비용을 요구하는 시스템의 병목 구간이었다.14

GeM의 등장은 이 패러다임에 도전할 수 있는 길을 열었다. GeM을 통해 생성된 고품질 전역 디스크립터는 그 자체만으로도 매우 높은 검색 성능을 달성할 수 있게 되었기 때문이다. "SuperGlobal"과 같은 최신 연구들은 이러한 가능성을 극대화하여, 지역 특징 기반의 재순위화 단계를 완전히 제거하고 오직 GeM 기반의 전역 디스크립터만으로 초기 검색과 재순위화를 모두 수행하는 새로운 파이프라인을 제안했다.14 놀랍게도 이 접근법은 기존의 복잡한 2단계 시스템보다 더 높은 정확도를 달성하면서도, 계산 비용과 저장 공간 요구량을 수십 배에서 수백 배까지 줄이는 데 성공했다. 이는 GeM이 단순히 더 나은 풀링 계층을 넘어, 지역 특징이 필수적이라는 오랜 통념을 깨고 훨씬 더 효율적이고 확장 가능한 차세대 이미지 검색 시스템을 가능하게 하는 핵심 기술(enabling technology)임을 증명한다.


GeM이 제시한 '학습 가능한 적응형 풀링'이라는 개념은 수많은 후속 연구에 영감을 주었으며, GeM의 기본 아이디어를 더욱 발전시킨 다양한 변형 모델들이 제안되었다. 이 변형 모델들은 풀링 연산에 공간적, 채널별 컨텍스트를 도입하거나 새로운 아키텍처에 맞게 조정함으로써 성능을 한 단계 더 끌어올렸다.


기본적인 GeM은 특징 맵 내의 모든 공간적 위치(spatial location)를 동등하게 취급한다는 한계를 가진다. 그러나 실제 이미지에서는 검색 대상이 되는 객체는 일부 영역에만 존재하고, 나머지 대부분은 관련 없는 배경이나 오히려 검색을 방해하는 요소(distractor)로 채워져 있는 경우가 많다.12

가중 일반화 평균 풀링(Weighted GeM, wGeM)은 이러한 한계를 극복하기 위해 제안되었다.12 wGeM의 핵심 아이디어는 GeM 연산을 수행하기 전에, 학습 가능한 2D 가중치 맵(trainable 2D weight map), 즉 '소프트 마스크(soft mask)'를 특징 맵에 곱해주는 것이다. 이 가중치 맵은 네트워크가 역전파를 통해 각 공간적 위치의 '중요도'를 학습하게 한다. 결과적으로 네트워크는 이미지 내에서 객체 식별에 중요한 영역에는 높은 가중치를 부여하여 집중하고, 불필요한 배경 영역에는 낮은 가중치를 부여하여 그 영향을 억제하는 방법을 스스로 터득하게 된다.12 wGeM은 별도의 경계 상자(bounding box)와 같은 강력한 지도(supervision) 정보 없이도, 이미지 검색 태스크 자체의 손실 함수만으로 시각적 어텐션(visual attention)과 유사한 메커니즘을 학습할 수 있다는 점에서 큰 의미를 가진다.12


비전 트랜스포머(Vision Transformer, ViT)의 등장은 컴퓨터 비전 분야의 아키텍처 패러다임을 바꾸었다. CNN과 달리 ViT는 이미지를 여러 패치(patch)로 나누고, 셀프 어텐션(self-attention) 메커니즘을 통해 패치 간의 전역적인 관계를 모델링한다. CNN을 위해 설계된 GeM을 ViT에 효과적으로 적용하기 위한 연구의 결과로 그룹 일반화 평균 풀링(Group GeM, GGeM)이 제안되었다.6

GGeM의 동기는 ViT의 멀티-헤드 어텐션(multi-head attention) 구조에 대한 깊은 분석에서 비롯된다. ViT의 최종 활성화 맵을 분석해보면, 각 채널이 이미지의 서로 다른 공간적 영역이나 의미적 부분에 집중하는 특성을 보인다. 즉, 채널별로 역할 분담이 이루어진다. 기존 GeM은 이러한 채널별 차이를 고려하지 않고 모든 채널에 단일 $p$ 파라미터를 적용한다. GGeM은 이 문제를 해결하기 위해, 전체 채널을 여러 그룹으로 나누고, 각 그룹마다 별도의 공유된 $p$ 파라미터를 학습시킨다.6 이는 ViT의 멀티-헤드 구조와 자연스럽게 부합하며, 각 채널 그룹의 특성에 맞는 최적의 풀링 전략을 학습할 수 있게 한다. 결과적으로 GGeM은 중요한 채널의 신호를 증폭시키고 헤드 간의 불필요한 의존성을 낮추어, ViT 기반 모델의 이미지 분류 및 검색 성능을 유의미하게 향상시켰다.6


wGeM이 암시적으로 어텐션을 학습했다면, 어텐션 메커니즘을 GeM과 명시적으로 결합하려는 시도도 이루어졌다. 어텐션 인식 GeM(Attention-aware GeM, AGeM)은 이러한 접근법의 대표적인 예이다.33 AGeM은 별도의 어텐션 모듈을 사용하여 이미지 내의 중요한 키포인트(keypoint)나 영역에 해당하는 특징을 먼저 강화하고, 이렇게 가중치가 부여된 특징 맵을 GeM 풀링으로 집약한다. 이를 통해 배경 클러터에 대한 강건성을 더욱 높이고, 객체의 핵심적인 부분에 집중하여 더욱 정제된 디스크립터를 생성할 수 있다.24

이 외에도 GeM의 기본 아이디어를 다양한 방식으로 확장한 연구들이 존재한다. 예를 들어, 이미지를 여러 영역으로 나누어 각 영역에 GeM을 적용하는 Regional-GeM이나, 다양한 스케일의 이미지에서 추출한 특징 맵에 GeM을 적용하는 Scale-GeM 등이 제안되었다.14 이러한 다양한 변형 모델들은 GeM이 가진 개념적 유연성과 뛰어난 확장성을 보여주는 증거이다.

GeM과 그 변형 모델들의 발전 과정은 풀링의 역할에 대한 인식이 어떻게 진화해왔는지를 명확히 보여준다. 초기의 최대/평균 풀링이 고정된 '정보 집약' 연산자였다면, GeM은 집약 '방식'을 학습하는 단계로 나아갔다. 여기서 더 나아가 wGeM과 AGeM은 '어디를' 집약할지(공간적 컨텍스트)를 학습했고, GGeM은 ViT라는 새로운 아키텍처의 특성('어떤 채널 그룹'을 어떻게 집약할지, 즉 건축적 컨텍스트)에 맞게 집약 방식을 적응시켰다. 이는 풀링 계층이 더 이상 수동적인 다운샘플링 도구가 아니라, 입력 데이터의 내용과 모델 아키텍처의 특성을 모두 고려하여 능동적으로 특징을 선택하고 증류(distill)하는 지능적인 모듈로 진화하고 있음을 나타낸다. 이는 풀링의 역할에 대한 근본적인 패러다임 전환을 의미한다.



GeM 풀링은 PyTorch와 같은 딥러닝 프레임워크에서 `nn.Module`을 상속받아 비교적 간단하게 구현할 수 있다. 핵심은 파라미터 $p$를 $nn.Parameter$로 등록하여 네트워크의 다른 파라미터들과 함께 학습 과정에서 그래디언트가 계산되고 업데이트되도록 하는 것이다.34

다음은 PyTorch를 이용한 GeM 풀링 계층의 구현 예시이다.

```Python
import torch
import torch.nn as nn
import torch.nn.functional as F

class GeM(nn.Module):
    """
    Generalized Mean Pooling layer.
    """
    def __init__(self, p=3, eps=1e-6, learnable=True):
        """
        Args:
            p (float): Initial value for the pooling parameter.
            eps (float): A small epsilon value to prevent numerical instability.
            learnable (bool): Whether the parameter p is learnable.
        """
        super(GeM, self).__init__()
        if learnable:
            # Register p as a learnable parameter
            self.p = nn.Parameter(torch.ones(1) * p)
        else:
            self.p = p
        self.eps = eps

    def forward(self, x):
        # The forward pass implements the GeM formula.
        # x.clamp(min=self.eps) is crucial for numerical stability, preventing log(0) or pow(0, large_p) issues.
        # F.avg_pool2d is used to efficiently compute the mean of powered activations.
        return F.avg_pool2d(x.clamp(min=self.eps).pow(self.p), (x.size(-2), x.size(-1))).pow(1./self.p)

    def __repr__(self):
        # For pretty printing the module
        return self.__class__.__name__ + '(' + 'p=' + '{:.4f}'.format(self.p.data.tolist() if isinstance(self.p, nn.Parameter) else self.p) + ', eps=' + str(self.eps) + ')'
```

구현 시 $p$의 초기값 설정은 학습의 안정성과 최종 성능에 영향을 미칠 수 있다. 경험적으로 $p=3$은 많은 이미지 검색 태스크에서 좋은 출발점으로 알려져 있으며, 안정적인 학습을 위해 너무 크거나 1에 너무 가까운 값으로 시작하는 것은 피하는 것이 좋다.11


GeM의 유연성은 구현 시 수치적 불안정성(numerical instability)이라는 잠재적 위험을 동반한다. 주요 문제점과 해결 방안은 다음과 같다.

- **언더플로우/오버플로우(Underflow/Overflow)**: 입력 활성화 $x$의 값이 0에 매우 가깝거나 음수인 경우, 혹은 $p$ 값이 매우 클 경우 $x^p$ 연산은 수치적으로 불안정해질 수 있다. $x$가 0에 가까우면 언더플로우가 발생하여 정보가 소실될 수 있고, $x$가 1보다 크고 $p$가 매우 크면 오버플로우가 발생하여 무한대(`inf`) 값이 될 수 있다.36
- **그래디언트 소실/폭발(Vanishing/Exploding Gradients)**: $p$가 매우 커져 GeM이 최대 풀링처럼 동작하게 되면, $p$ 자체에 대한 그래디언트는 0에 가까워져 더 이상 학습이 진행되지 않을 수 있다.12 또한, $x$가 0에 가까울 때 $p$에 대한 그래디언트 공식에 포함된 $ln(x)$ 항이 음의 무한대로 발산하여 그래디언트 폭발을 일으킬 수 있다.

이러한 문제들을 해결하기 위한 실용적인 기법들은 다음과 같다.

1. **엡실론 클램핑(Epsilon Clamping)**: $x^p$ 연산을 수행하기 전에, 입력 텐서 $x$의 모든 원소 값이 아주 작은 양수 엡실론($eps$) 이상이 되도록 `clamp` 함수를 적용한다. 위 코드 예시의 $x.clamp(min=self.eps)$가 이 역할을 수행하며, 0 또는 음수 값으로 인한 $log(0)$나 $pow(negative, non-integer)$와 같은 정의되지 않은 연산을 방지한다.34
2. **안정적인 `L_p`-노름 계산**: 대규모 수치 계산에서 $L_p$-노름의 안정성을 높이기 위해 사용되는 기법을 차용할 수 있다. 예를 들어, $x^p$를 계산하기 전에 텐서 $x$의 모든 값을 $x$의 최댓값으로 나누어 0과 1 사이로 정규화하고, 최종 결과에 최댓값을 다시 곱해주는 방식이다. 이는 $log-sum-exp$ 트릭과 유사한 원리로, 큰 값으로 인한 오버플로우를 효과적으로 방지한다.36
3. **`p` 값 제약**: 학습 과정에서 $p$ 값이 비정상적으로 크거나 작아지는 것을 방지하기 위해, 업데이트된 $p$ 값을 특정 범위(예: $p >= 1$) 내로 강제로 제한(clipping)하는 방법을 사용할 수 있다.


GeM 풀링은 기존 풀링 기법에 비해 약간의 추가 연산을 요구하지만, 그 오버헤드는 미미한 수준이다. GeM의 주요 추가 연산은 각 활성화 값에 대한 거듭제곱($pow$) 연산과 최종 결과에 대한 거듭제곱근 연산이다. 이러한 연산들은 현대의 GPU 아키텍처에서 매우 효율적으로 병렬 처리될 수 있다.37

또한, 학습 가능한 파라미터 $p$는 채널별로 독립적인 값을 갖게 하더라도 그 수가 매우 적으며, 일반적으로는 모든 채널이 공유하는 단일 스칼라 값이다. 따라서 GeM 계층을 추가하더라도 모델의 전체 파라미터 수는 거의 증가하지 않는다. 결과적으로, GeM은 상당한 성능 향상을 가져다주는 것에 비해 계산 비용과 메모리 요구량 증가는 매우 적어, 효율성이 매우 높은 기법이라고 평가할 수 있다.37

그러나 GeM의 유연성은 '숨겨진 비용'을 수반한다. 학습 가능한 $p$는 모델에 새로운 자유도를 부여하지만, 동시에 학습의 안정성과 재현성에 대한 새로운 과제를 제기한다. $p$의 학습 동역학은 초기값, 손실 함수의 종류, 입력 데이터의 스케일 등 여러 요인에 민감하게 반응할 수 있다. 따라서 GeM을 성공적으로 활용하기 위해서는 단순히 계층을 추가하는 것을 넘어, 적절한 $p$ 초기값 선택, 수치 안정을 위한 $eps$ 값 설정, 특정 손실 함수와의 잠재적 충돌 고려 등 세심한 공학적 접근이 요구된다. 이는 GeM의 '플러그 앤 플레이(plug-and-play)' 특성이 모든 조건에서 보장되는 것은 아니며, 그 잠재력을 최대한 이끌어내기 위해서는 숙련된 적용이 필요함을 시사한다.



일반화 평균 풀링(GeM)은 지난 몇 년간 딥러닝, 특히 컴퓨터 비전 분야에서 풀링 기법의 발전에 중요한 이정표를 제시했다. GeM의 핵심적인 장점과 잠재적인 단점을 종합하면 다음과 같다.

**장점:**

- **일반성과 유연성**: 최대 풀링과 평균 풀링을 하나의 통일된 프레임워크 안에 포괄함으로써, 두 기법의 장점을 취사선택할 수 있는 유연성을 제공한다.6
- **적응성**: 핵심 파라미터 $p$를 학습 가능하게 만들어, 모델이 데이터의 특성과 특정 태스크의 요구사항에 맞춰 최적의 정보 집약 전략을 스스로 학습할 수 있게 한다.13
- **뛰어난 성능**: 특히 인스턴스 수준 이미지 검색과 같은 세밀한(fine-grained) 인식 태스크에서 기존 풀링 기법들을 압도하는 성능 향상을 일관되게 보여주었다.15
- **높은 효율성**: 달성하는 성능 향상 폭에 비해 계산 비용 및 모델 파라미터 증가가 거의 없어 매우 효율적인 기법이다.37

**단점:**

- **수치 불안정성**: 큰 $p$ 값이나 0에 가까운 입력 활성화 값에 대해 수치적으로 불안정해질 수 있어, 구현 시 $eps$ 클램핑과 같은 안정화 기법이 필수적이다.12
- **최적화의 복잡성**: 특정 손실 함수(예: 마진 기반 손실)와 결합될 때 파라미터 $p$가 차선책으로 수렴할 위험이 존재하며, 이는 추가적인 하이퍼파라미터 튜닝이나 별도의 학습 전략을 요구할 수 있다.14
- **균일한 처리**: wGeM이나 GGeM과 같은 변형 모델과 달리, 기본적인 GeM은 특징 맵의 모든 공간적 위치와 채널을 균일하게 처리하므로, 컨텍스트에 따른 중요도 차이를 반영하지 못한다.12


GeM의 등장은 풀링 연구의 패러다임을 고정된 연산자에서 학습 가능한 적응형 모듈로 전환시키는 결정적인 계기가 되었다. 이는 풀링 계층이 더 이상 수동적인 다운샘플링 도구가 아니라, 모델의 표현력을 적극적으로 향상시키는 능동적인 구성 요소가 될 수 있음을 보여주었다.

GeM을 기반으로 한 wGeM, GGeM, AGeM과 같은 후속 연구들은 이러한 방향성을 더욱 심화시키고 있다. 이들은 풀링에 공간적 어텐션, 채널별 적응, 아키텍처 특성 고려 등 더 높은 수준의 '지능'을 부여한다. 이는 풀링의 역할이 단순한 '정보 압축'을 넘어, 주어진 컨텍스트 속에서 가장 유용한 특징을 선별하고 증류(distill)하는 정교한 과정으로 발전하고 있음을 시사한다.

미래의 풀링 기법은 더욱 정교한 어텐션 메커니즘과 결합되거나, 데이터의 통계적 분포에 따라 동적으로 연산 방식을 바꾸거나, 혹은 그래프 신경망이나 포인트 클라우드 처리와 같은 비-격자(non-grid) 데이터에 특화된 새로운 형태의 일반화된 풀링으로 발전할 가능성이 높다.

이러한 진화의 흐름 속에서 GeM은 딥러닝의 두 가지 핵심 아이디어, 즉 컨볼루션이 추구하는 '불변성(invariance)'과 어텐션이 추구하는 '선택성(selectivity)' 사이를 잇는 중요한 다리 역할을 수행했다고 평가할 수 있다. 전통적인 풀링은 주로 국소적 변환에 대한 고정된 불변성을 제공하는 데 목적이 있었다. GeM은 학습 가능한 $p$를 통해 어떤 종류의 불변성(평균과 같은 부드러운 불변성 혹은 최대와 같은 뾰족한 불변성)을 가질지 데이터로부터 학습하게 함으로써, 불변성 자체를 적응적으로 만들었다. 더 나아가 wGeM과 같은 후속 연구는 여기에 명시적인 선택성(어텐션)을 결합하여, 모델이 '어떤 정보에 대해 불변성을 가질 것인가'를 동적으로 결정하게 했다. 이는 풀링의 패러다임을 '고정된 불변성 연산'에서 '학습된 선택적 불변성 모듈'로 전환시킨 중요한 개념적 진보이며, GeM은 이러한 진보의 문을 연 선구적인 연구로서 그 역사적, 기술적 중요성을 계속 유지할 것이다.


1. A Comparison of Pooling Methods for Convolutional Neural Networks - MDPI, 8월 19, 2025에 액세스, https://www.mdpi.com/2076-3417/12/17/8643
2. CNN | Introduction to Pooling Layer - GeeksforGeeks, 8월 19, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/cnn-introduction-to-pooling-layer/
3. A Comprehensive Exploration of Pooling in Neural Networks: A Python demo to Pooling in CNN - Paperspace Blog, 8월 19, 2025에 액세스, https://blog.paperspace.com/a-comprehensive-exploration-of-pooling-in-neural-networks/
4. A Gentle Introduction to Pooling Layers for Convolutional Neural Networks - MachineLearningMastery.com, 8월 19, 2025에 액세스, https://machinelearningmastery.com/pooling-layers-for-convolutional-neural-networks/
5. Pooling: Overview - Unsupervised Feature Learning and Deep Learning Tutorial, 8월 19, 2025에 액세스, http://deeplearning.stanford.edu/tutorial/supervised/Pooling/
6. arXiv:2212.04114v1 [cs.CV] 8 Dec 2022 - Sanghyuk Chun, 8월 19, 2025에 액세스, https://sanghyukchun.github.io/home/media/papers/ko2022ggem.pdf
7. Maxpooling vs minpooling vs average pooling | by Madhushree Basavarajaiah - Medium, 8월 19, 2025에 액세스, https://medium.com/@bdhuma/which-pooling-method-is-better-maxpooling-vs-minpooling-vs-average-pooling-95fb03f45a9
8. Why is max pooling necessary in convolutional neural networks? - Cross Validated, 8월 19, 2025에 액세스, https://stats.stackexchange.com/questions/288261/why-is-max-pooling-necessary-in-convolutional-neural-networks
9. A improved pooling method for convolutional neural networks - PMC, 8월 19, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10796389/
10. Parameterized Pooling Layers - gudgud96's Blog, 8월 19, 2025에 액세스, https://gudgud96.github.io/2020/11/25/param-pooling/
11. On Global Feature Pooling for Fine-grained Visual Categorization - OpenReview, 8월 19, 2025에 액세스, https://openreview.net/forum?id=H1lkCTVFwB
12. (PDF) Weighted Generalized Mean Pooling for Deep Image Retrieval - ResearchGate, 8월 19, 2025에 액세스, https://www.researchgate.net/publication/327995368_Weighted_Generalized_Mean_Pooling_for_Deep_Image_Retrieval
13. 【Pooling Method】What is GeM(Generalized Mean Pooling) - Zenn, 8월 19, 2025에 액세스, https://zenn.dev/yuto_mo/articles/c7c6c06f505b6f
14. Global Features are All You Need for Image Retrieval and Reranking - CVF Open Access, 8월 19, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2023/papers/Shao_Global_Features_are_All_You_Need_for_Image_Retrieval_and_ICCV_2023_paper.pdf
15. (PDF) Fine-tuning CNN Image Retrieval with No Human Annotation, 8월 19, 2025에 액세스, https://www.researchgate.net/publication/320920418_Fine-tuning_CNN_Image_Retrieval_with_No_Human_Annotation
16. Instance-level Recognition | Towards Data Science, 8월 19, 2025에 액세스, https://towardsdatascience.com/instance-level-recognition-6afa229e2151/
17. Global Features are All You Need for Image Retrieval and Reranking - Hugging Face, 8월 19, 2025에 액세스, https://huggingface.co/papers/2308.06954
18. [1711.02512] Fine-tuning CNN Image Retrieval with No Human Annotation - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/abs/1711.02512
19. Group Generalized Mean Pooling for Vision Transformer | Request PDF - ResearchGate, 8월 19, 2025에 액세스, https://www.researchgate.net/publication/366136072_Group_Generalized_Mean_Pooling_for_Vision_Transformer
20. Backpropagation - Wikipedia, 8월 19, 2025에 액세스, https://en.wikipedia.org/wiki/Backpropagation
21. Learning global image representation with generalized‐mean pooling and smoothed average precision for large‐scale CBIR - ResearchGate, 8월 19, 2025에 액세스, https://www.researchgate.net/publication/370776946_Learning_global_image_representation_with_generalized-mean_pooling_and_smoothed_average_precision_for_large-scale_CBIR
22. GeM Pooling Explained with PyTorch Implementation and Introduction to Image Retrieval, 8월 19, 2025에 액세스, https://amaarora.github.io/posts/2020-08-30-gempool.html
23. Fine-tuning CNN Image Retrieval with No Human Annotation - arXiv, 8월 19, 2025에 액세스, http://arxiv.org/pdf/1711.02512
24. On Train-Test Class Overlap and Detection for Image Retrieval - CVF Open Access, 8월 19, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024/papers/Song_On_Train-Test_Class_Overlap_and_Detection_for_Image_Retrieval_CVPR_2024_paper.pdf
25. Utilising Second-Order Information for Deep Image Retrieval - OpenCV, 8월 19, 2025에 액세스, https://opencv.org/blog/utilising-second-order-information-for-deep-image-retrieval/
26. SOLAR: Second-Order Loss and Attention for Image Retrieval, 8월 19, 2025에 액세스, https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123700256.pdf
27. How to Build an Image Search Engine to Find Similar Images | by EORA.ai | Medium, 8월 19, 2025에 액세스, https://medium.com/@eora/how-to-build-an-image-search-engine-to-find-similar-images-c6ec429b9a27
28. tuananh1007/CNN-Image-Retrieval-in-PyTorch - GitHub, 8월 19, 2025에 액세스, https://github.com/tuananh1007/CNN-Image-Retrieval-in-PyTorch
29. Revisiting Oxford and Paris: Large-Scale Image Retrieval Benchmarking - CVF Open Access, 8월 19, 2025에 액세스, https://openaccess.thecvf.com/content_cvpr_2018/papers/Radenovic_Revisiting_Oxford_and_CVPR_2018_paper.pdf
30. The comparison of performance (mAP) of global max pooling(MAC), average... | Download Scientific Diagram - ResearchGate, 8월 19, 2025에 액세스, https://www.researchgate.net/figure/The-comparison-of-performance-mAP-of-global-max-poolingMAC-average-poolingSPoC-and_tbl1_335621775
31. Global Features are All You Need for Image Retrieval and Reranking - ResearchGate, 8월 19, 2025에 액세스, https://www.researchgate.net/publication/373116356_Global_Features_are_All_You_Need_for_Image_Retrieval_and_Reranking
32. [2212.04114] Group Generalized Mean Pooling for Vision Transformer - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/abs/2212.04114
33. [1811.00202] Attention-Aware Generalized Mean Pooling for Image Retrieval - arXiv, 8월 19, 2025에 액세스, https://arxiv.org/abs/1811.00202
34. TypeError: avg_pool2d(): argument 'kernel_size' (position 2) must be tuple of ints, not tuple, 8월 19, 2025에 액세스, https://discuss.pytorch.org/t/typeerror-avg-pool2d-argument-kernel-size-position-2-must-be-tuple-of-ints-not-tuple/151655
35. Adding own Pooling Algorithm to Pytorch - #6 by hendryx - vision, 8월 19, 2025에 액세스, https://discuss.pytorch.org/t/adding-own-pooling-algorithm-to-pytorch/54742/6
36. Numerically stable p-norms — Graduate Descent - Tim Vieira, 8월 19, 2025에 액세스, https://timvieira.github.io/blog/post/2014/11/10/numerically-stable-p-norms/
37. Learning the Best Pooling Strategy for Visual Semantic Embedding - CVF Open Access, 8월 19, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2021/papers/Chen_Learning_the_Best_Pooling_Strategy_for_Visual_Semantic_Embedding_CVPR_2021_paper.pdf
38. Generalizing Pooling Functions in CNNs: Mixed, Gated, and Tree - University of California San Diego, 8월 19, 2025에 액세스, https://pages.ucsd.edu/~ztu/publication/pami_gpooling.pdf
39. Generalizing Pooling Functions in Convolutional Neural Networks: Mixed, Gated, and Tree, 8월 19, 2025에 액세스, https://proceedings.mlr.press/v51/lee16a.html


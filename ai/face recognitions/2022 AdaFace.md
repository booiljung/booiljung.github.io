# AdaFace (저품질 얼굴 인식을 위한 품질 적응형 마진의 원리와 의의, CVPR 2022)
[얼굴인식 문제](./index.md)

<iframe width="560" height="315" src="https://www.youtube.com/embed/eKzaMquLJmc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



지난 10년간 딥러닝 기술의 발전은 얼굴 인식(Face Recognition, FR) 분야에 혁명적인 변화를 가져왔다. 대규모 데이터셋과 깊은 컨볼루션 신경망(Deep Convolutional Neural Networks, DCNNs)의 결합을 통해, LFW(Labeled Faces in the Wild), MegaFace와 같은 통제된 환경의 벤치마크에서 기계는 이미 인간의 인식 능력을 상회하는 정확도를 달성했다.1 이러한 성과는 금융, 보안, 모바일 인증 등 다양한 산업 분야에 얼굴 인식 기술이 상용화되는 기폭제가 되었다.

그러나 실험실 수준의 높은 성능과 실제 환경(in-the-wild)에서의 성능 사이에는 여전히 깊은 간극이 존재한다. 우리가 일상에서 마주하는 얼굴 이미지는 대부분 비제약적(unconstrained) 환경에서 수집된다. 감시 카메라 영상, 저조도 환경에서 촬영된 스마트폰 사진, 군중 속에서 포착된 얼굴 등은 극심한 조명 변화, 낮은 해상도, 비정면 자세, 가림(occlusion), 모션 블러 등 다양한 품질 저하 요소를 포함한다.3 이러한 저품질 이미지는 얼굴의 고유한 식별 정보를 심각하게 훼손하며, 최첨단 딥러닝 모델조차도 성능이 급격히 저하되는 결과를 초래한다.1 특히 '비제약적 저해상도 얼굴 인식(Unconstrained Low-Resolution Face Recognition, LRFR)'은 현재 얼굴 인식 기술이 실세계 문제에 완벽하게 적용되기 위해 반드시 해결해야 할 핵심적인 병목 지점으로 남아있다.1


저품질 이미지 문제를 해결하기 위해, 기존의 많은 연구는 '어려운 샘플(hard samples)'에 집중하는 전략을 채택해왔다. 이는 모델이 쉽게 분류하는 샘플보다 오분류하거나 확신도가 낮은 샘플에 더 큰 가중치를 부여하여 학습을 강화하는 방식, 즉 하드 샘플 마이닝(hard sample mining)이다.8 이 접근법은 모델의 판별력을 높이는 데 일정 부분 기여했지만, 저품질 데이터가 만연한 실제 환경에서는 예기치 않은 부작용을 낳았다.

문제의 핵심은 저품질 이미지가 단순히 인식하기 '어려운' 수준을 넘어, 신원 정보 자체가 복구 불가능할 정도로 소실되어 본질적으로 '식별이 불가능한(unidentifiable)' 경우가 존재한다는 점이다.3 예를 들어, 극심한 블러로 인해 눈, 코, 입의 형태를 전혀 알아볼 수 없는 얼굴이나, 그림자에 가려져 얼굴 윤곽조차 파악하기 힘든 이미지는 어떤 모델이라도 정확한 신원을 추론할 수 없다.

이러한 식별 불가능한 이미지들은 모델 학습 과정에서 '독(poison)'과 같은 역할을 한다. 모델의 관점에서 이 이미지들은 항상 오분류될 수밖에 없으므로, 영원히 가장 '어려운 샘플'로 남게 된다. 따라서 하드 샘플 마이닝 전략은 필연적으로 이러한 식별 불가능한 샘플에 과도하게 집중하게 된다. 모델은 존재하지 않는 신원 정보를 억지로 학습하려 시도하는 과정에서, 의상 색상, 배경의 질감, 이미지의 해상도와 같은 신원과 무관한 시각적 단서(spurious correlations)에 의존하여 손실 함수를 최소화하려는 잘못된 방향으로 학습을 진행한다.3 이는 결국 모델이 학습 데이터에 과적합되고 새로운 데이터에 대한 일반화 성능을 심각하게 저해하는 결과를 초래한다.

결론적으로, 얼굴 인식의 핵심 난제는 단순히 '어려운 샘플'의 존재가 아니라, '어려우면서 식별 가능한 샘플'과 '어려우면서 식별 불가능한 샘플'을 구분하지 못하는 기존 학습 전략의 근본적인 맹점에 있다. 이 두 그룹의 샘플은 학습 과정에서 완전히 다른 방식으로 취급되어야 한다는 문제 인식이 바로 AdaFace가 제안하는 새로운 패러다임의 출발점이다.



딥러닝 기반 얼굴 인식 모델의 학습에는 전통적으로 Softmax 손실 함수가 널리 사용되어 왔다. Softmax 손실은 다중 클래스 분류 문제에서 모델의 출력을 확률 분포로 변환하고, 이를 실제 레이블과의 교차 엔트로피(cross-entropy)를 통해 최소화함으로써 효과적으로 특징을 학습시킨다. 그러나 전통적인 Softmax 손실은 특징 벡터가 각 클래스로 '분류'되는 데에만 초점을 맞출 뿐, 특징 공간 내에서 클래스 간의 분리도(separability)를 극대화하도록 직접적으로 유도하지는 않는다.4 이로 인해 학습된 특징은 클래스 내 분산(intra-class variance)이 크고 클래스 간 거리(inter-class distance)가 작아지는 경향이 있으며, 이는 학습 데이터에 없던 새로운 신원을 판별해야 하는 개방형 집합(open-set) 인식 문제에서 심각한 한계로 작용한다.

이러한 문제를 해결하기 위해, 딥러닝 얼굴 인식 분야에서는 특징 공간 상의 의사결정 경계(decision boundary)에 명시적인 '마진(margin)'을 도입하는 연구들이 활발하게 진행되었다.2 마진 기반 손실 함수의 핵심 아이디어는, 모델이 단순히 정답 클래스를 맞추는 것을 넘어, 정답 클래스의 특징이 다른 모든 오답 클래스의 특징들로부터 일정 거리 이상의 '안전지대', 즉 마진을 확보하도록 강제하는 것이다. 이를 통해 클래스 내 특징들은 더욱 밀집하게 응축되고(compactness), 클래스 간 특징들은 더욱 멀리 분리되어(separability), 결과적으로 훨씬 더 판별력 높은(discriminative) 특징 표현을 학습할 수 있게 된다.


마진 기반 손실 함수의 초기 연구들은 모든 클래스와 모든 샘플에 동일한 크기의 고정된 마진을 적용하는 방식으로 발전했다.

- **SphereFace (A-Softmax):** 2017년에 제안된 SphereFace는 마진 기반 손실 함수의 가능성을 연 중요한 연구다.3 이 방법은 Softmax 손실의 마지막 완전 연결 계층의 가중치 벡터를 L2 정규화하여, 특징 벡터와 가중치 벡터 간의 내적(inner product)을 코사인 유사도와 특징 벡터의 크기(norm)의 곱으로 해석했다. 나아가 특징 벡터가 단위 초구(hypersphere) 위에 분포하도록 유도하며, 각도 공간(θ)에 곱셈 형태의 마진(m)을 도입한 $cos(m\theta)$를 제안했다. 이를 통해 각도 공간에서 클래스 간의 분리를 강화하는 새로운 길을 열었다.

- **CosFace (Additive Margin):** SphereFace의 아이디어를 계승하면서도 학습의 불안정성과 구현의 복잡성을 개선한 CosFace는 2018년에 등장했다.3 CosFace는 가중치뿐만 아니라 특징 벡터까지 모두 L2 정규화하여 특징의 크기 변화에 따른 영향을 배제하고, 오직 각도 정보에만 집중했다. 마진은 각도 공간이 아닌 코사인 공간에 덧셈 형태로 적용($cos(\theta) - m$)되어, 더 직관적이고 안정적인 최적화 과정을 가능하게 했다.

- **ArcFace (Additive Angular Margin):** 현재까지도 가장 널리 사용되는 강력한 베이스라인 중 하나인 ArcFace는 2019년에 제안되었다.14 ArcFace는 CosFace와 같이 특징과 가중치를 모두 정규화하지만, 마진을 각도 공간에 직접 덧셈 형태로 적용($cos(\theta + m)$)한다. 이는 특징 공간에서 실제 각도만큼의 마진을 확보하는 것과 동일하여, 코사인 공간 마진보다 기하학적으로 훨씬 더 직관적이고 명확한 의사결정 경계를 형성한다.3 이러한 명료성과 뛰어난 성능 덕분에 ArcFace는 이후 등장하는 수많은 얼굴 인식 연구의 표준 베이스라인으로 자리 잡았다.


SphereFace, CosFace, ArcFace는 얼굴 인식 성능을 크게 향상시켰지만, 모든 데이터 샘플에 동일한 고정 마진을 적용한다는 근본적인 한계를 공유한다. 이는 "one-size-fits-all" 접근법으로, 데이터의 다양한 특성을 고려하지 못한다. 이러한 한계를 극복하기 위해 마진을 동적으로 조절하려는 '적응형' 연구들이 등장하기 시작했다.

- **CurricularFace:** 2020년에 제안된 CurricularFace는 커리큘럼 학습(curriculum learning)의 개념을 마진에 도입했다.3 학습 초기 단계에서는 마진을 작게 설정하여 모델이 상대적으로 쉬운 샘플에 집중하도록 하고, 학습이 진행됨에 따라 점진적으로 마진을 증가시켜 어려운 샘플까지 학습하도록 유도한다. 이는 학습의 난이도를 동적으로 조절한다는 점에서 진일보했지만, 이러한 커리큘럼이 모든 이미지 샘플에 동일하게 적용된다는 점에서 개별 샘플의 특성을 반영하지는 못하는 한계가 있었다.
- **MagFace:** 2021년에 제안된 MagFace는 특징 벡터의 L2 노름(크기)이 이미지 품질과 유의미한 상관관계가 있음을 발견하고 이를 활용했다.8 MagFace는 고품질 이미지일수록 더 큰 특징 노름을 갖도록 학습을 유도하고, 이 노름 값을 마진 계산에 반영하여 품질에 따라 어느 정도 적응적인 학습이 이루어지도록 했다. 하지만 MagFace의 주된 목표는 특징 노름 자체를 품질 지표로 활용하여 임베딩의 표현력을 높이는 것이었고, 이미지 품질에 따라 학습의 '목표' 자체를 근본적으로 바꾸는 데까지는 나아가지 못했다.

이러한 시도들은 고정 마진의 문제를 인식하고 해결하려는 중요한 발걸음이었다. 마진 기반 손실 함수의 역사는 '전역적이고 고정된 제약'에서 '지역적이고 동적인 제약'으로 나아가는 패러다임의 전환 과정으로 볼 수 있다. 1세대 모델들(ArcFace 등)이 모든 샘플에 동일한 마진 $m$을 적용했다면, 2세대 모델(CurricularFace)은 마진을 학습 시간의 함수인 $m(t)$로 바꾸었다. 그리고 3세대 모델(MagFace, AdaFace)은 마진을 개별 샘플 $i$의 고유 속성(품질)의 함수인 $m(quality_i)$로 발전시켰다. AdaFace는 이러한 흐름의 정점에 서서, 각 데이터 샘플의 품질을 가장 중요한 제어 신호로 사용하여 가장 개인화되고 효과적인 학습 목표를 실시간으로 설정하는 방식을 제안함으로써, 적응형 마진 연구의 새로운 지평을 열었다.



AdaFace의 가장 중요한 공헌은 기술적인 측면을 넘어선 철학적 전환에 있다. 기존의 방법들이 '어떻게 하면 더 어려운 샘플을 잘 학습할까?'라는 질문에 집중했다면, AdaFace는 "어려운 샘플을 강조하는 전략은 이미지의 품질에 따라 근본적으로 달라져야 한다"고 주장한다.3 이는 모든 샘플에 일관된 학습 원칙을 적용하려던 기존의 관점에서 벗어나, 샘플의 상태에 따라 학습의 목표 자체를 유연하게 바꾸는 새로운 패러다임을 제시한 것이다.

이 철학은 다음과 같은 이중 전략(dual strategy)으로 구체화된다 3:

1. **고품질 이미지의 경우:** 고품질 이미지는 조명이 좋고, 해상도가 높으며, 가림이 없는 등 신원을 특정할 수 있는 풍부한 정보를 담고 있다. 이러한 이미지에 대해서는 모델이 미세한 차이까지 구분해내는 높은 판별력을 갖추도록 학습시켜야 한다. 따라서, 모델이 이미 쉽게 분류하는 샘플보다는 분류 경계에 가깝거나 오분류된 '어렵지만 식별 가능한(hard but recognizable)' 샘플에 집중하도록 강한 학습 신호(gradient)를 부여하는 것이 유리하다.
2. **저품질 이미지의 경우:** 저품질 이미지는 정보가 왜곡되거나 소실되어 신원을 특정하기 어렵다. 특히 '식별 불가능한' 수준의 이미지에 대해 어려운 샘플을 강조하는 전략을 고수하면, 모델은 존재하지 않는 정보를 억지로 학습하려다 노이즈에 과적합될 위험이 크다. 따라서 이러한 이미지에 대해서는 학습 전략을 역전시켜, 식별 불가능할 정도로 어려운 샘플은 과감히 무시하고, 그나마 식별 가능한 단서를 가진 상대적으로 '쉬운' 샘플에 집중하는 것이 모델의 안정성과 일반화 성능에 더 유리하다.

이처럼 이미지 품질이라는 단일 축을 기준으로 학습의 강도와 방향을 동적으로 조절하는 것이 AdaFace의 핵심 아이디어다.


이미지 품질에 따라 학습 전략을 바꾸기 위해서는 먼저 각 이미지의 품질을 정량적으로 평가할 수 있어야 한다. 이를 위해 별도의 이미지 품질 평가(Image Quality Assessment, IQA) 모듈을 설계하여 신경망에 추가할 수도 있지만, 이는 모델의 복잡도를 높이고 추가적인 계산 비용을 발생시킨다.

AdaFace는 이 문제를 매우 독창적이고 효율적인 방식으로 해결했다. 바로 딥러닝 모델이 스스로 추출한 특징 벡터(feature vector)의 L2 노름(Euclidean norm, $\Vert z \Vert$)이 이미지 품질을 나타내는 효과적인 대리 지표(proxy)가 될 수 있음을 실험적으로 증명한 것이다.3 잘 학습된 얼굴 인식 모델은 깨끗하고 정보가 풍부한 고품질 이미지에 대해서는 신경망의 여러 계층에서 일관되고 강한 활성화(activation)를 보이며, 이러한 신호들이 누적되어 최종적으로 큰 크기의 특징 벡터, 즉 높은 노름 값을 생성한다. 반대로, 흐릿하거나 가려진 저품질 이미지에 대해서는 불확실하고 약한 활성화를 보이며, 이는 작은 크기의 특징 벡터, 즉 낮은 노름 값으로 이어진다.

이 발견은 매우 중요한 의미를 갖는다. 특징 노름은 네트워크가 주어진 입력 이미지로부터 얼마나 '확신에 찬' 특징을 추출했는지를 나타내는 '내재적 자기 평가(implicit self-assessment)' 신호로 해석될 수 있다. AdaFace는 이 내재적 신호를 명시적인 학습 제어 신호로 전환했다. 이는 추가적인 모듈이나 계산 오버헤드 없이, 모델의 자연스러운 속성을 활용하여 품질 적응형 학습을 가능하게 하는 우아하고 효율적인 접근법이다.3


AdaFace는 특징 노름이라는 품질 지표를 기반으로, 각 학습 샘플에 대한 손실 함수의 형태 자체를 실시간으로 변형시킨다.8 이는 단순히 마진의 크기를 조절하는 것을 넘어, 샘플의 품질에 따라 학습의 목표와 그래디언트의 분포를 근본적으로 바꾸는 것을 의미한다.

- **높은 노름(고품질) 샘플:** 특징 노름 값이 큰 샘플에 대해서는 손실 함수가 ArcFace와 유사하게, 혹은 그보다 더 강하게 어려운 샘플을 강조하는 형태로 변형된다. 이는 모델이 분류 경계에서 멀리 떨어진, 판별하기 어려운 샘플에 대해 더 큰 그래디언트를 받도록 하여, 고품질 이미지로부터 미세하고 판별력 높은 특징을 학습하도록 강하게 유도한다.8
- **낮은 노름(저품질) 샘플:** 특징 노름 값이 작은 샘플에 대해서는 손실 함수가 CosFace와 유사하거나, 심지어 음수 각도 마진(negative angular margin)처럼 작동하는 형태로 변형된다. 이는 분류 경계에 가까운 샘플에 대한 학습은 유지하되, 경계에서 매우 멀리 떨어진, 즉 '식별 불가능할' 정도로 어려운 샘플에 대해서는 그래디언트의 크기를 급격히 줄인다.8 결과적으로 모델은 이러한 노이즈성 샘플들을 학습 과정에서 사실상 무시하게 되어, 과적합을 방지하고 안정적인 학습을 이어갈 수 있다.

이처럼 AdaFace는 네트워크의 '자기 평가' 결과를 손실 함수에 직접 되먹임하는 정교한 피드백 루프(feedback loop)를 구축한다. 네트워크의 확신도(노름)가 높으면 더 어려운 과제(hard sample mining)를 부여하고, 확신도가 낮으면 더 쉬운 과제(de-emphasizing hard samples)를 부여하는 것이다. 이는 외부의 감독 없이 네트워크 스스로 자신의 상태를 진단하고 학습 난이도를 조절하는 '자가 조절 학습(self-regulating learning)'의 한 형태로, 매우 강력하고 일반화 가능한 원리라 할 수 있다.



AdaFace 손실 함수는 ArcFace와 같은 마진 기반 Softmax 손실 함수의 구조를 기반으로 한다. 핵심적인 차이는 실제 클래스($y_i$)에 대한 로짓(logit) 계산 시, 이미지 품질에 따라 적응적으로 변하는 각도 마진 항 $g_{angle}$과 가산 마진 항 $g_{add}$를 추가하는 것이다. 전체 손실 함수는 다음과 같이 정의된다.3

$$
\mathcal{L}_{AdaFace} = - \log \frac{\exp (s (\cos (\theta_{y_i} + g_{\text{angle}}) - g_{\text{add}}))}{\exp (s (\cos (\theta_{y_i} + g_{\text{angle}}) - g_{\text{add}})) + \sum_{j \neq y_i}^C \exp (s \cos \theta_j)}
$$
이 수식을 이해하기 위해 각 구성 요소를 상세히 살펴볼 필요가 있다. 아래 표는 AdaFace 손실 함수에 사용된 주요 파라미터와 그 역할을 요약한 것이다.

| 기호                                 | 정의                                            | 역할                                                         | 논문 설정값 |
| ------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------ | ----------- |
| $s$                                  | 스케일 파라미터(Scale Parameter)                | 코사인 유사도 값을 스케일링하여 Softmax 함수의 입력 범위를 조절하고 수렴을 돕는다. | 64          |
| $\theta_j$                           | 특징 벡터와 가중치 벡터 간의 각도               | 입력 이미지의 특징 벡터 $\mathbf{z}_i$와 클래스 $j$의 가중치 벡터 $\mathbf{W}_j$ 사이의 각도. $\cos \theta_j$는 유사도를 의미한다. | -           |
| $m$                                  | 기본 마진 하이퍼파라미터(Margin Hyperparameter) | 적응형 마진 항 $g_{angle}$과 $g_{add}$의 기본 크기를 결정한다. | 0.4         |
| $\mathbf{z}_i$                       | 특징 벡터(Feature Vector)                       | 입력 이미지 $x_i$가 딥러닝 모델을 통과하여 생성된 임베딩 벡터. | -           |
| $\widehat{\Vert \mathbf{z}_i \Vert}$ | 정규화된 특징 노름(Normalized Feature Norm)     | 이미지 품질의 대리 지표. `[-1, 1]` 범위의 값을 가지며 마진 항을 조절하는 핵심 입력이다. | -           |
| $g_{angle}$                          | 적응형 각도 마진(Adaptive Angular Margin)       | $\widehat{\Vert \mathbf{z}_i \Vert}$에 따라 동적으로 변하는 각도 마진. | -           |
| $g_{add}$                            | 적응형 가산 마진(Adaptive Additive Margin)      | $\widehat{\Vert \mathbf{z}_i \Vert}$에 따라 동적으로 변하는 가산 마진. | -           |


적응형 마진을 계산하기 위한 핵심 입력은 이미지 품질의 대리 지표인 정규화된 특징 노름 $\widehat{\Vert \mathbf{z}_i \Vert}$이다. 이는 개별 샘플의 특징 노름 $\Vert \mathbf{z}_i \Vert$를 그대로 사용하지 않고, 현재 학습 미니배치(mini-batch)의 통계량을 이용해 정규화하는 과정을 거친다. 이는 학습 과정에서 노름 값의 전반적인 스케일 변화에 강건한 상대적인 품질 지표를 얻기 위함이다. 계산 과정은 다음과 같다.3

$$
\widehat{\Vert \mathbf{z}_i \Vert} = \text{clip}\left(\frac{\Vert \mathbf{z}_i \Vert - \mu_z}{\sigma_z / h}, -1, 1\right)
$$

- $\mu_z$와 $\sigma_z$는 각각 미니배치 내 모든 특징 노름들의 평균과 표준편차다. 작은 배치 크기에서 불안정할 수 있으므로, 지수 이동 평균(Exponential Moving Average, EMA)을 사용하여 안정화된다.
- $h$는 분포 집중도(concentration)를 제어하는 하이퍼파라미터다. 이 값을 조절하여 정규화된 노름 값이 대부분 `[-1, 1]` 범위 내에 위치하도록 유도한다. 논문에서는 $h=0.33$을 사용했다.3
- $\text{clip}(\cdot, -1, 1)$ 함수는 정규화된 노름 값이 `[-1, 1]` 범위를 벗어나지 않도록 강제한다. 이는 극단적인 노름 값을 가진 샘플이 학습에 미치는 영향을 제한하여 안정성을 높이는 역할을 한다. 또한, 이 연산은 역전파(back-propagation) 시 그래디언트의 흐름을 차단(stop gradient)하는 효과가 있다. 이는 모델이 마진을 쉽게 만족시키기 위해 인위적으로 모든 특징의 노름을 낮추는 방향으로 최적화되는 것을 방지하는 중요한 안전장치다.3


정규화된 특징 노름 $\widehat{\Vert \mathbf{z}_i \Vert}$는 두 개의 적응형 마진 항, $g_{angle}$과 $g_{add}$를 계산하는 데 사용된다. 이 두 항은 서로 다른 공간(각도, 코사인)에서 마진을 조절하며, 이들의 조합을 통해 이미지 품질에 따른 정교한 학습 제어가 이루어진다.3

$$
g_{\text{angle}} = -m \cdot \widehat{\Vert \mathbf{z}_i \Vert}
$$

$$
g_{\text{add}} = m \cdot \widehat{\Vert \mathbf{z}_i \Vert} + m
$$

이 두 항이 전체 손실 함수 $\mathcal{L}_{AdaFace}$의 $\cos(\theta_{y_i} + g_{angle}) - g_{add}$ 부분에 적용되어 최종 로짓을 결정한다.


AdaFace의 진정한 효과는 손실 함수의 형태 변화가 학습 샘플의 난이도에 따른 그래디언트 크기(gradient scale)를 어떻게 조절하는지를 통해 이해할 수 있다. 그래디언트의 크기는 해당 샘플이 모델 파라미터 업데이트에 얼마나 큰 영향을 미치는지를 결정한다.

- 고품질 이미지 ($\widehat{\Vert \mathbf{z}_i \Vert} \to 1$):

  이 경우, $\widehat{\Vert \mathbf{z}_i \Vert}$는 1에 가까워진다.

  - $g_{angle} \to -m$

  - $g_{add} \to m \cdot 1 + m = 2m$

    따라서 로짓 항은 근사적으로 $s(\cos(\theta_{y_i} - m) - 2m)$ 형태가 된다. 여기서 중요한 부분은 $\cos(\theta_{y_i} - m)$이다. 이는 각도 공간에서 마진을 '음수'로 적용하는 것과 같으며, 결정 경계를 정답 클래스 방향으로 더욱 공격적으로 밀어붙이는 효과를 낸다. 이 형태는 결정 경계에서 멀리 떨어진, 즉 코사인 유사도가 낮은 '어려운 샘플'에 대해 매우 큰 그래디언트를 생성한다.8 이는 모델이 고품질 이미지의 미세한 차이를 학습하여 높은 판별력을 갖추도록 강하게 압박하는 전략이다.

- 저품질 이미지 ($\widehat{\Vert \mathbf{z}_i \Vert} \to -1$):

  이 경우, $\widehat{\Vert \mathbf{z}_i \Vert}$는 -1에 가까워진다.

  - $g_{angle} \to -m \cdot (-1) = +m$

  - $g_{add} \to m \cdot (-1) + m = 0$

    따라서 로짓 항은 근사적으로 $s \cos(\theta_{y_i} + m)$ 형태가 된다. 이는 정확히 ArcFace의 형태와 같다. ArcFace와 같은 양수 각도 마진은 결정 경계에 가까운 샘플에 대해서는 그래디언트를 높이지만, 경계에서 멀리 떨어진 매우 어려운 샘플에 대해서는 그래디언트를 낮추는 특성이 있다.8 즉, 저품질 이미지에 대해서는 AdaFace가 '식별 불가능할' 정도로 어려운 샘플에 대한 학습을 의도적으로 억제(de-emphasize)하는 방향으로 작동한다. 이는 모델이 노이즈성 데이터에 과적합되는 것을 방지하고 학습의 안정성을 높이는 효과를 가져온다.

이처럼 AdaFace는 특징 노름이라는 간단한 지표를 통해, 고품질 이미지에 대해서는 '어려운 샘플 집중' 전략을, 저품질 이미지에 대해서는 '매우 어려운 샘플 무시' 전략을 하나의 손실 함수 내에서 매끄럽게 전환한다.



AdaFace의 성능은 그 효과를 다각적으로 검증하기 위해 다양한 품질의 데이터셋에서 포괄적으로 평가되었다. 사용된 주요 벤치마크 데이터셋은 이미지 품질에 따라 다음과 같이 분류할 수 있다.3

- **고품질 데이터셋:**
  - **LFW, CFP-FP, AgeDB, CALFW, CPLFW:** 이들은 주로 웹에서 수집된 상대적으로 깨끗하고 고품질의 이미지로 구성된 전통적인 얼굴 검증(verification) 벤치마크다. 모델의 기본적인 판별력을 평가하는 데 사용된다.
- **혼합 품질 데이터셋:**
  - **IJB-B, IJB-C:** 이 데이터셋들은 실제 환경에서의 얼굴 인식을 모사하기 위해 설계되었다. 유명인의 고품질 이미지뿐만 아니라, 저화질 비디오에서 추출된 프레임, 다양한 포즈와 표정의 이미지들이 다수 포함되어 있어 품질의 편차가 매우 크다.3
- **저품질 데이터셋:**
  - **IJB-S, TinyFace:** 이 데이터셋들은 감시(surveillance) 환경과 같이 열악한 조건에서 촬영된 이미지들을 주로 포함한다. 특히 TinyFace는 이름에서 알 수 있듯이 매우 작고 저해상도의 얼굴 이미지로만 구성되어 있어, 저품질 환경에서의 모델 강건성을 평가하는 데 핵심적인 역할을 한다.3

이처럼 품질 스펙트럼의 양극단을 모두 포함하는 데이터셋 구성은 AdaFace가 제안하는 '품질 적응형' 전략의 유효성을 입증하는 데 필수적이었다.


실험 결과는 AdaFace의 가설을 강력하게 뒷받침했다. 특히 저품질 및 혼합 품질 데이터셋에서 기존의 최첨단(State-of-the-Art, SoTA) 모델들을 압도적인 성능 차이로 능가하는 결과를 보였다.

- **저품질/혼합 품질 데이터셋에서의 우월성:** IJB-B, IJB-C, IJB-S, TinyFace와 같은 도전적인 벤치마크에서 AdaFace는 ArcFace, CosFace, CurricularFace 등 모든 베이스라인 모델 대비 유의미한 성능 향상을 기록했다. 예를 들어, MS1MV2 데이터셋으로 학습했을 때, IJB-B와 IJB-C에서 기존 SoTA 모델 대비 오류율을 각각 11%, 9% 상대적으로 감소시켰다.3 TinyFace 데이터셋에서는 두 번째로 좋은 모델과 비교하여 Rank-1 정확도를 평균 3.5% 이상 개선했다.3 이는 AdaFace가 식별 불가능한 이미지에 대한 과적합을 효과적으로 방지하고, 저품질 환경에서도 강건하고 일반화 성능이 뛰어난 특징 표현을 학습했음을 직접적으로 증명하는 결과다.8
- **고품질 데이터셋에서의 성능 유지:** LFW와 같은 고품질 데이터셋에서는 기존 SoTA 모델들과 대등하거나 미세하게 우수한 성능을 유지했다. 이는 AdaFace가 저품질 데이터에 대한 강건함을 확보하는 과정에서 고품질 데이터에 대한 판별력을 희생하지 않았음을 의미한다.3 즉, AdaFace는 특정 환경에만 특화된 것이 아니라, 넓은 품질 스펙트럼에 걸쳐 안정적으로 높은 성능을 제공하는 범용적인 솔루션임을 보여준다.
- **정성적 평가 (데모 비교):** 수치적인 결과 외에도, 공식 GitHub 저장소에서 제공하는 ArcFace와의 실시간 비디오 데모는 AdaFace의 우수성을 직관적으로 보여준다.17 원본 영상, 약간 흐린 영상(blur+), 그리고 매우 흐린 영상(blur++) 세 가지 조건에서 두 모델의 성능을 비교했을 때, 영상이 흐려질수록 ArcFace의 인식 신뢰도(코사인 유사도)는 급격히 떨어지는 반면, AdaFace는 상대적으로 안정적인 높은 신뢰도를 유지하며 정확한 인물을 지속적으로 식별해냈다. 이는 AdaFace가 실제 저품질 비디오 스트림 환경에서 얼마나 효과적으로 작동하는지를 보여주는 강력한 정성적 증거다.

아래의 표들은 ResNet-100 아키텍처를 기반으로 다양한 학습 데이터셋(MS1MV2, MS1MV3, WebFace4M)을 사용하여 주요 벤치마크에서 AdaFace와 다른 SoTA 모델들의 성능을 비교한 결과다.3

**표 1: 고품질 및 혼합 품질 데이터셋 성능 비교 (1:1 검증 정확도(%) 및 TAR@FAR=1e-4(%)**

| 방법                | 학습 데이터   | LFW       | CFP-FP    | CPLFW     | AgeDB     | IJB-B     | IJB-C     |
| ------------------- | ------------- | --------- | --------- | --------- | --------- | --------- | --------- |
| CosFace             | MS1MV2        | 99.81     | 98.12     | 92.28     | 98.11     | 94.80     | 96.37     |
| ArcFace             | MS1MV2        | 99.83     | 98.27     | 92.08     | 98.28     | 94.25     | 96.03     |
| CurricularFace      | MS1MV2        | 99.80     | 98.37     | 93.13     | 98.32     | 94.80     | 96.10     |
| MagFace             | MS1MV2        | 99.83     | 98.46     | 92.87     | 98.17     | 94.51     | 95.97     |
| **AdaFace (m=0.4)** | **MS1MV2**    | **99.82** | **98.49** | **93.53** | **98.05** | **95.67** | **96.89** |
| ArcFace*            | WebFace4M     | 99.83     | 99.19     | 94.35     | 97.95     | 95.75     | 97.16     |
| **AdaFace (m=0.4)** | **WebFace4M** | **99.80** | **99.17** | **94.63** | **97.90** | **96.03** | **97.39** |

**표 2: 저품질 데이터셋 성능 비교 (IJB-S: Rank-1(%), TinyFace: Rank-1(%)**

| 방법                | 학습 데이터   | IJB-S (S-to-S) | IJB-S (S-to-B) | IJB-S (S-to-S) | TinyFace  |
| ------------------- | ------------- | -------------- | -------------- | -------------- | --------- |
| PFE                 | MS1MV2        | 50.16          | 53.60          | 9.20           | -         |
| ArcFace             | MS1MV2        | 57.35          | 57.36          | -              | -         |
| CurricularFace*     | MS1MV2        | 62.43          | 63.81          | 19.54          | 63.68     |
| **AdaFace (m=0.4)** | **MS1MV2**    | **65.26**      | **66.27**      | **23.74**      | **68.21** |
| ArcFace*            | WebFace4M     | 69.26          | 70.31          | 32.13          | 71.11     |
| **AdaFace (m=0.4)** | **WebFace4M** | **70.42**      | **70.93**      | **35.05**      | **72.02** |

*S-to-S: Surveillance-to-Single, S-to-B: Surveillance-to-Booking, S-to-S: Surveillance-to-Surveillance*

두 표를 종합적으로 분석하면, AdaFace의 독특한 장점이 명확하게 드러난다. 표 1에서 볼 수 있듯이 고품질 데이터셋(LFW, CFP-FP 등)에서는 기존 SoTA 모델들과의 성능 차이가 크지 않지만, 혼합 품질 데이터셋인 IJB-B와 IJB-C로 넘어가면서부터 AdaFace의 우위가 뚜렷해지기 시작한다. 그리고 표 2의 저품질 데이터셋(IJB-S, TinyFace)에서는 그 격차가 더욱 벌어지며, AdaFace가 압도적인 성능을 보임을 확인할 수 있다. 이는 '고품질 성능은 유지하면서 저품질 성능을 극대화'하는 AdaFace의 핵심 목표가 성공적으로 달성되었음을 보여주는 객관적인 증거다.



AdaFace는 단순히 하나의 뛰어난 성능을 가진 모델을 제안한 것을 넘어, 얼굴 인식 연구 커뮤니티에 중요한 방향성을 제시했다. 가장 큰 기술적 의의는 이미지 '품질'이라는, 이전까지는 주로 전처리나 데이터 증강의 영역으로 간주되던 요소를 손실 함수 설계의 핵심 변수로 명시적으로 도입했다는 점이다.3 이는 모델이 데이터의 이질성(heterogeneity)을 수동적으로 받아들이는 것을 넘어, 능동적으로 활용하여 학습 전략을 최적화해야 한다는 새로운 관점을 제시했다.

이러한 선구적인 접근 방식은 많은 후속 연구에 영감을 주었다. 예를 들어, 클래스 불균형 문제를 해결하기 위해 클래스별 샘플 수에 따라 마진을 조절하는 JAMsFace 18나, 샘플의 난이도를 측정하는 새로운 지표를 도입하여 적응형 페널티를 부과하는 AdaSin 14과 같은 연구들은 모두 AdaFace가 제시한 '적응형 마진'이라는 큰 틀 안에서 발전한 결과물이라 볼 수 있다.

또한, AdaFace는 그 강력한 성능과 명확한 개념 덕분에 저품질 얼굴 인식 분야의 중요한 베이스라인으로 자리매김했다. 최근에는 CLIP과 같은 거대 언어-비전 파운데이션 모델(Foundation Model)과 도메인 특화 모델의 성능을 비교하는 연구에서, AdaFace는 ArcFace와 함께 도메인 특화 모델의 대표주자로 사용되고 있다.19 심지어 AdaFace가 식별하기 어려워하는 저신뢰도 샘플을 파운데이션 모델이 해결하거나, 두 모델을 융합하여 상호 보완적으로 성능을 더욱 향상시키는 연구가 진행되는 등, AdaFace는 현재 진행형으로 학계에 영향을 미치고 있다.19


AdaFace는 혁신적인 접근법을 제시했지만, 몇 가지 내재적인 한계 또한 가지고 있다.

- **품질 프록시의 불완전성:** 특징 벡터의 노름은 대부분의 경우 이미지 품질을 효과적으로 대변하지만, 완벽한 지표는 아니다. 예를 들어, 조명이 매우 균일하고 정면을 응시하고 있어 객관적으로는 고품질이지만, 피부 톤이 매우 단조롭거나 텍스처가 거의 없는 얼굴(low texture)은 낮은 노름 값을 가질 수 있다. 반대로, 저해상도 이미지에 매우 독특하고 강한 특징을 가진 액세서리(예: 화려한 안경, 짙은 화장)가 포함된 경우, 신원 정보와 무관하게 높은 노름 값을 가질 수 있다. 이러한 예외적인 경우, AdaFace의 품질 적응형 전략이 오작동할 가능성이 존재한다.
- **일반적인 딥러닝 모델의 한계:** AdaFace 역시 다른 최첨단 딥러닝 모델들과 마찬가지로, 높은 성능을 위해 대규모의 학습 데이터와 상당한 계산 자원을 필요로 한다. 모델의 파라미터 수가 많아 메모리 요구량이 크고 연산 복잡도가 높기 때문에, 이를 경량화하여 모바일 플랫폼이나 임베디드 시스템과 같은 엣지 디바이스에 직접 배포하는 데에는 여전히 기술적 장벽이 존재한다.20
- **미해결 과제:** AdaFace는 전반적인 이미지 품질 저하 문제에 강건한 모습을 보이지만, 특정 유형의 변인에 대해서는 명시적인 모델링을 수행하지 않는다. 예를 들어, 얼굴이 90도 측면을 향하는 극단적인 자세 변화나, 마스크나 손으로 얼굴의 절반 이상이 가려지는 심각한 가림(occlusion) 문제에 대해서는 여전히 인식 성능이 저하될 수 있다.9 이러한 특정 변인들을 해결하기 위해서는 별도의 정규화 기법이나 데이터 증강, 혹은 이를 전문적으로 다루는 모듈과의 결합이 필요할 수 있다.


AdaFace가 제시한 원리와 현재의 한계점들은 다음과 같은 흥미로운 미래 연구 방향을 제시한다.

- **적응형 원리의 확장:** AdaFace의 가장 큰 유산은 '데이터의 이질성을 학습 과정에 내재화하는 방법론'을 제시했다는 점이다. 이 핵심 철학은 이미지 품질을 넘어 다른 문제 영역으로 확장될 수 있다. 예를 들어, 클래스 내 특징 벡터의 분산도를 측정하여 클래스 불균형(class imbalance) 문제에 대응하는 마진을 동적으로 설정하거나, 학습 과정에서 발생하는 그래디언트의 크기나 안정성을 모니터링하여 샘플별 학습률(learning rate)을 동적으로 조절하는 등의 연구가 가능하다. 이는 모델이 데이터의 다양한 통계적 특성에 스스로 적응하는, 한 단계 더 높은 수준의 '메타-러닝(meta-learning)'으로 발전할 수 있는 잠재력을 가진다.
- **더 정교한 품질/난이도 지표 탐색:** 특징 노름을 대체하거나 보완할 수 있는 더 정교한 지표를 탐색하는 연구도 유망하다. 예를 들어, 모델의 불확실성(uncertainty)을 측정하는 기법(e.g., Monte Carlo Dropout)을 활용하거나, 주의(attention) 메커니즘을 통해 모델이 이미지의 어느 부분에 집중하는지를 분석하여 해당 샘플의 인식 가능성을 판단할 수 있다.
- **도메인 특화 모델과 파운데이션 모델의 융합:** 최근 연구가 보여주듯 19, 미래의 얼굴 인식 시스템은 단일 모델에 의존하기보다 여러 모델의 장점을 결합하는 하이브리드 형태가 될 가능성이 높다. AdaFace와 같이 특정 도메인(얼굴 신원 판별)에 고도로 전문화된 모델이 정밀한 1차 판단을 수행하고, CLIP과 같이 방대한 시각적-언어적 지식을 가진 파운데이션 모델이 그 판단의 근거를 설명하거나, AdaFace가 낮은 신뢰도를 보이는 모호한 경우에 대해 보조적인 추론을 제공하는 방식으로 상호 보완적인 시스템을 구축할 수 있다.

결론적으로, AdaFace는 저품질 얼굴 인식 성능을 획기적으로 개선한 SoTA 모델일 뿐만 아니라, 딥러닝 모델이 현실 세계의 불완전하고 이질적인 데이터에 더 강건하게 적응할 수 있도록 하는 일반적인 설계 원칙을 제시한 중요한 연구로 평가될 수 있다. 이는 얼굴 인식을 넘어, 데이터 품질이 균일하지 않은 모든 현실 세계의 머신러닝 문제에 적용될 수 있는 강력하고 일반적인 패러다임의 시작을 알리는 신호탄이다.


1. Low-Resolution Face Recognition - CORE, 8월 24, 2025에 액세스, https://core.ac.uk/download/pdf/219803514.pdf
2. UnifiedFace: A Uniform Margin Loss Function for Face Recognition - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2076-3417/13/4/2350
3. AdaFace: Quality Adaptive Margin for Face ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Kim_AdaFace_Quality_Adaptive_Margin_for_Face_Recognition_CVPR_2022_paper.pdf
4. [2204.00964] AdaFace: Quality Adaptive Margin for Face Recognition - ar5iv - arXiv, 8월 24, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2204.00964
5. Low-Quality Face Recognition in the Wild - Notre Dame CVRL, 8월 24, 2025에 액세스, [https://cvrl.nd.edu/projects/?project_name=Low-Quality%20Face%20Recognition%20in%20the%20Wild](https://cvrl.nd.edu/projects/?project_name=Low-Quality+Face+Recognition+in+the+Wild)
6. Understanding the Impact of Image Quality in Face Processing Algorithms - SciTePress, 8월 24, 2025에 액세스, https://www.scitepress.org/PublishedPapers/2021/104865/104865.pdf
7. (PDF) Face Recognition from Low Resolution Images - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/286992675_Face_Recognition_from_Low_Resolution_Images
8. AdaFace: Quality Adaptive Margin for Face Recognition - Computer Vision Lab - Michigan State University, 8월 24, 2025에 액세스, http://cvlab.cse.msu.edu/project-adaface.html
9. AdaFace: Quality Adaptive Margin for Face Recognition | Request PDF - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/359729553_AdaFace_Quality_Adaptive_Margin_for_Face_Recognition
10. LittleFaceNet: A Small-Sized Face Recognition Method Based on RetinaFace and AdaFace - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2313-433X/11/1/24
11. An Equalized Margin Loss for Face Recognition - UCL Discovery, 8월 24, 2025에 액세스, https://discovery.ucl.ac.uk/10089852/1/TMM-JingnaSun-EqM-UCL.pdf
12. A note on margin-based loss functions in classification - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/222399422_A_note_on_margin-based_loss_functions_in_classification
13. Unifying Margin-Based Softmax Losses in Face Recognition - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2023/papers/Zhang_Unifying_Margin-Based_Softmax_Losses_in_Face_Recognition_WACV_2023_paper.pdf
14. AdaFace: Quality Adaptive Margin for Face Recognition - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/AdaFace%3A-Quality-Adaptive-Margin-for-Face-Kim-Jain/549572ffc0f5adb6640a8d8fc93c8d7e62008985
15. [1801.07698] ArcFace: Additive Angular Margin Loss for Deep Face Recognition - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1801.07698
16. [2204.00964] AdaFace: Quality Adaptive Margin for Face Recognition - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2204.00964
17. mk-minchul/AdaFace - GitHub, 8월 24, 2025에 액세스, https://github.com/mk-minchul/AdaFace
18. JAMsFace: joint adaptive margins loss for deep face recognition - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/371637949_JAMsFace_joint_adaptive_margins_loss_for_deep_face_recognition
19. Foundation versus Domain-specific Models: Performance Comparison, Fusion, and Explainability in Face Recognition - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2507.03541v2
20. EdgeFace : Efficient Face Recognition Model for Edge Devices - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2307.01838v2


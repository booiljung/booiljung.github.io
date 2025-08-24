# Mixture-of-Transformers (MoT) 아키텍처
[Modality](./index.md)


인공지능(AI) 분야, 특히 거대 언어 모델(LLM)의 발전은 텍스트라는 단일 양식(modality)의 한계를 넘어 텍스트, 이미지, 음성 등 여러 양식을 단일 프레임워크 내에서 통합적으로 처리하는 멀티모달 시스템으로 확장되고 있다.1 이러한 멀티모달 파운데이션 모델은 인간과 유사한 방식으로 정보를 이해하고 생성할 수 있는 잠재력을 지니고 있지만, 그 이면에는 심각한 기술적 도전과제가 존재한다. 바로 '스케일링의 딜레마'이다.


멀티모달 모델을 훈련하는 것은 텍스트 전용 LLM에 비해 "상당히 더 큰 데이터셋과 연산 자원을 요구한다".1 이는 단순히 처리할 데이터의 양이 늘어나는 것을 넘어, 각 양식이 갖는 고유한 통계적, 구조적 특성 때문에 발생하는 근본적인 문제이다. 텍스트는 이산적인 토큰의 순차적 나열인 반면, 이미지는 공간적 픽셀 관계의 연속적 표현이며, 음성은 시간적 파형의 복잡한 패턴을 갖는다.

이러한 이질적인 데이터를 단일한 '밀도(dense)' 아키텍처, 즉 모든 가중치(parameter)를 공유하는 모델로 처리하려는 시도는 필연적으로 연산 비용의 폭발적인 증가를 야기한다. 더 큰 문제는, 단일 파라미터 집합이 서로 다른 양식의 상충하는 학습 목표를 동시에 최적화해야 하는 과정에서 "부정적 간섭(negative interference)" 또는 "훈련 충돌(training conflicts)"이 발생할 수 있다는 점이다.6 예를 들어, 이미지의 공간적 특징을 학습하는 데 최적화된 가중치는 텍스트의 문법적 구조를 학습하는 데 방해가 될 수 있다. 이로 인해 모델의 성능은 저하되고 훈련은 불안정해지며, 이를 해결하기 위해 모델의 크기와 데이터셋을 무작정 키우는 것은 지속 불가능한 '무차별적 스케일링(brute-force scaling)'에 불과하다.


이러한 막대한 연산 비용은 AI 연구 개발 생태계에 심각한 불균형을 초래한다. 최첨단 파운데이션 모델의 개발 및 탐색 능력은 소수의 "고급 사용자 및 업계 리더"에게 국한되며, 이는 학계 및 광범위한 커뮤니티의 접근을 막는 "암묵적인 기술 장벽"으로 작용한다.1 이러한 상황은 연구의 다양성을 저해하고 혁신을 특정 집단에 종속시킬 위험이 있다.

따라서 모델의 효율성을 높이는 것은 단순히 비용을 절감하는 기술적 최적화를 넘어, "첨단 AI에 대한 접근을 민주화"하고 8, 더 넓은 연구 커뮤니티가 인류의 지식을 발전시키는 데 동참할 수 있도록 하는 중요한 과제이다. 효율적인 아키텍처는 더 적은 자원으로도 고성능 모델을 훈련하고 실험할 수 있게 함으로써, AI 기술 발전의 과실을 모두가 누릴 수 있는 기반을 마련한다.


이러한 스케일링의 딜레마를 극복하기 위한 가장 유망한 전략 중 하나는 '희소성(sparsity)'을 아키텍처에 도입하는 것이다. 희소성은 모델의 모든 파라미터를 매 계산마다 활성화하는 대신, 입력에 따라 필요한 부분(파라미터의 부분집합)만을 조건부로 활성화하는 '조건부 연산(conditional computation)' 개념에 기반한다.

이 분야에서 가장 대표적인 접근법은 '전문가 혼합(Mixture-of-Experts, MoE)' 모델이다.5 MoE는 모델 내에 여러 개의 '전문가(expert)' 네트워크(주로 피드포워드 네트워크)를 두고, 학습 가능한 '라우팅 게이트(routing gate)'가 각 입력 토큰을 처리할 가장 적합한 전문가 소수에게 동적으로 할당하는 방식이다.10 이를 통해 모델의 전체 파라미터 수를 크게 늘리면서도, 실제 연산량(FLOPs)은 활성화되는 전문가 수에만 비례하도록 제어할 수 있어 연산 효율적인 스케일링이 가능하다.


MoE가 범용적인 스케일링 솔루션으로서 가능성을 보였지만, 멀티모달 데이터의 이질성이라는 근본적인 문제를 직접적으로 다루지는 않는다. 이러한 배경 속에서, Meta AI와 스탠포드 대학의 연구진들은 멀티모달 스케일링 문제를 정면으로 겨냥한 새로운 희소 아키텍처, '트랜스포머 혼합(Mixture-of-Transformers, MoT)'을 제안했다.1

MoT는 MoE와는 다른, 보다 구조화된 형태의 희소성을 도입한다. MoT의 핵심 아이디어는 모델의 전문성을 토큰 단위의 유연성에 맡기는 대신, 데이터의 가장 명확한 구분 기준인 '양식(modality)'을 기준으로 파라미터를 분리하는 것이다.2 이는 무차별적 스케일링에서 벗어나, 데이터의 구조적 특성을 활용하여 지능적이고 체계적으로 모델을 확장하려는 아키텍처 설계 철학의 중요한 전환을 의미한다. 즉, MoT의 등장은 단일 구조의 모델이 모든 종류의 데이터를 효과적으로 처리할 수 있다는 기존의 가정에 의문을 제기하고, 이질적인 데이터에는 이질적인 처리 방식이 더 효율적이라는 가설을 아키텍처 수준에서 증명하려는 시도이다. MoT의 성공은 이 가설을 입증하며, 차세대 멀티모달 시스템을 위한 새로운 설계 청사진을 제시했다.


Mixture-of-Transformers (MoT) 아키텍처는 멀티모달 학습의 효율성을 극대화하기 위해 설계된 정교한 구조물이다. 그 핵심은 '양식별 파라미터 분리', '결정론적 라우팅', 그리고 '전역적 자기 어텐션'이라는 세 가지 기둥 위에 세워져 있다. 이 세 요소는 서로 유기적으로 결합하여 각 데이터 양식의 고유한 특성을 전문적으로 처리하면서도, 양식 간의 풍부한 상호작용을 가능하게 하는 이중 목표를 달성한다.


MoT의 가장 근본적인 혁신은 임베딩 레이어를 제외한 트랜스포머의 모든 *비-임베딩(non-embedding)* 파라미터를 양식별로 분리(decoupling)하는 것이다.1 이는 모델이 각 데이터 유형에 맞춤화된, 특화된 처리를 수행하도록 하는 '양식 인지 희소성(modality-aware sparsity)'을 구현하는 핵심 메커니즘이다.8 분리되는 주요 구성 요소는 다음과 같다.

- **피드포워드 네트워크 (Feed-Forward Networks, FFNs):** 트랜스포머 블록에서 연산의 상당 부분을 차지하는 FFN이 양식별로 독립적으로 존재한다. 예를 들어, 텍스트, 이미지, 음성을 처리하는 모델이라면 각각을 위한 별도의 '텍스트 FFN', '이미지 FFN', '음성 FFN' 전문가를 갖게 된다. FFN은 통상적으로 트랜스포머 비-임베딩 파라미터의 약 2/3를 차지하므로, 이 부분의 분리는 MoT 효율성의 가장 큰 비중을 차지한다.14
- **어텐션 프로젝션 행렬 (Attention Projection Matrices):** 자기 어텐션 메커니즘 내부의 쿼리(Q), 키(K), 값(V)을 생성하기 위한 가중치 행렬(Wq,Wk,Wv)과 어텐션 결과의 출력 프로젝션을 위한 가중치 행렬(Wo) 역시 양식별로 분리된다.14 이는 각 양식이 자신만의 방식으로 다른 토큰과의 관계를 질의하고 표현할 수 있도록 한다.
- **레이어 정규화 (Layer Normalization):** MoT는 각 양식별 전문가 모듈에 독립적인 레이어 정규화 파라미터를 적용한다. 이는 일부 이전의 희소 모델들이 정규화 파라미터를 공유했던 것과 차별화되는 지점으로, 각 양식의 활성화 값 분포를 보다 안정적으로 제어하여 학습 효율을 높이는 데 기여한다.14

이러한 철저한 분리를 통해 MoT는 각 양식의 고유한 데이터 구조와 통계적 패턴을 학습하는 데 최적화된 독립적인 처리 경로를 제공하며, 밀도 모델에서 발생하던 훈련 충돌 문제를 근본적으로 완화한다.6


MoT의 두 번째 핵심 기둥은 토큰을 해당 전문가에게 보내는 '라우팅(routing)' 방식이다. 복잡한 학습 기반의 게이트 네트워크를 사용하는 MoE와 달리, MoT는 매우 단순하고 효율적인 "결정론적(deterministic)" 라우팅 메커니즘을 채택한다.14

이 메커니즘은 `modality_masks`라는 사전 정의된 불리언(boolean) 마스크를 통해 구현된다.14

`modality_masks`는 입력 시퀀스의 각 토큰이 어떤 양식에 속하는지를 명시하는 역할을 한다. 예를 들어, 텍스트 토큰과 이미지 토큰이 번갈아 나타나는 시퀀스가 입력되면, `modality_masks`는 텍스트 토큰의 위치를 표시하는 마스크와 이미지 토큰의 위치를 표시하는 마스크로 구성된다. 모델은 이 마스크를 사용하여 텍스트 토큰은 '텍스트 전문가' 구성 요소(텍스트 FFN, 텍스트 어텐션 프로젝션 등)로, 이미지 토큰은 '이미지 전문가' 구성 요소로 기계적으로, 즉 학습 과정 없이 라우팅한다.

이러한 "간단한 규칙 기반 접근법(simple rule-based approach)" 5은 MoT의 핵심적인 설계 철학을 보여준다. 이는 라우팅을 위한 별도의 네트워크를 훈련시킬 필요가 없고, MoE에서 종종 문제가 되는 "라우팅 변동(routing fluctuation)"이나 전문가 간의 "부하 불균형(load imbalance)" 문제를 해결하기 위한 복잡한 보조 손실 함수(auxiliary loss)가 필요 없게 만들어 훈련 안정성과 효율성을 크게 향상시킨다.12


파라미터가 양식별로 분리되어 있음에도 불구하고, MoT는 단순히 독립적인 모델들의 집합이 아니다. 양식 간의 의미 있는 이해와 통합은 "전역적 자기 어텐션(global self-attention)" 메커니즘을 통해 이루어진다.1 이 과정은 다음과 같이 진행된다.

1. **양식별 프로젝션:** 전체 입력 시퀀스의 토큰들은 `modality_masks`에 따라 각자의 양식에 맞는 Wq,Wk,Wv 프로젝션 전문가에게 라우팅되어 처리된다.14
2. **통합 시퀀스 재구성:** 이렇게 생성된 양식별 쿼리, 키, 값 벡터들은 다시 원래의 토큰 순서를 유지한 채 하나의 통합된 시퀀스로 재결합(re-pack)된다.6
3. **전역적 자기 어텐션 연산:** 이 통합된 전체 시퀀스에 대해 표준적인 전역 자기 어텐션 연산이 수행된다. 이 단계에서 모든 토큰은 양식에 관계없이 다른 모든 토큰을 참조할 수 있게 되며, 이를 통해 풍부한 "양식 간 상호작용(cross-modal interactions)"이 발생한다.6 예를 들어, 이미지의 '고양이' 토큰이 텍스트의 '야옹' 토큰에 직접적으로 어텐션을 부여할 수 있다.
4. **양식별 출력 프로젝션:** 어텐션 연산의 결과물은 다시 `modality_masks`에 따라 각 양식에 맞는 출력 프로젝션(Wo) 및 레이어 정규화 전문가에게 라우팅되어 최종 출력을 형성한다.14

이처럼 MoT 아키텍처는 멀티모달 이해를 위한 정교한 '분할 정복(divide and conquer)' 전략을 구현한다. 이는 *구조적 특징 추출*의 문제(분리된 FFN 및 프로젝션 행렬이 담당)와 *문맥적 의미 통합*의 문제(전역적 자기 어텐션이 담당)를 명확히 분리하는 접근법이다. 분리된 구성 요소들은 각 양식에 특화된 저수준의 통계적 패턴(예: 이미지 패치의 질감, 텍스트 토큰의 구문적 관계)을 학습하는 데 전문화된다. 반면, 전역 어텐션은 이러한 전문화된 처리의 결과물들을 바탕으로 양식에 구애받지 않는 고수준의 의미적 관계(예: 처리된 '고양이' 이미지 패치와 처리된 'feline' 텍스트 토큰 간의 연관성)를 찾아낸다.

이러한 설계는 멀티모달 이해가 어떻게 작동하는지에 대한 강력하고 우아한 구조적 가설을 모델에 내장한 것과 같다. 이는 모든 것이 얽혀 있는 밀도 모델보다 해석 가능성이 높고, 라우팅이 동적이고 복잡한 MoE 모델보다 훈련이 안정적이다. 이 설계 철학이야말로 MoT가 멀티모달 과제에서 뛰어난 성능과 효율성을 보이는 핵심적인 이유일 것이다.


Mixture-of-Transformers (MoT) 아키텍처의 혁신성을 제대로 평가하기 위해서는 기존의 지배적인 패러다임인 표준 밀도(Dense) 트랜스포머와 전문가 혼합(Mixture-of-Experts, MoE) 모델과의 다각적인 비교가 필수적이다. MoT는 이 두 아키텍처의 장점을 취하고 단점을 보완하려는 시도에서 탄생했으며, 그 구조적 차이는 성능, 효율성, 그리고 확장성 측면에서 중요한 함의를 가진다.


MoT와 밀도 트랜스포머의 가장 큰 차이점은 연산 효율성과 전문화에 있다.

- **연산 효율성:** 밀도 모델은 모든 입력에 대해 전체 파라미터를 사용하므로, 모델의 크기가 커질수록 연산량(FLOPs)이 직접적으로 증가한다. 반면 MoT는 조건부 연산을 통해 입력 토큰의 양식에 따라 파라미터의 일부만 활성화한다. 그 결과, MoT는 밀도 모델과 동등하거나 더 나은 성능을 훨씬 적은 FLOPs와 실제 훈련 시간(wall-clock time)으로 달성할 수 있다.1
- **전문화와 '양식세(Modality Tax)':** 밀도 모델은 단일 파라미터 집합이 텍스트, 이미지, 음성 등 이질적인 데이터의 특성을 모두 학습해야 하는 부담을 안고 있다. 이는 각 양식에 최적화된 전문성을 확보하기 어렵게 만들며, 때로는 "양식세(modality tax)"라고 불리는 성능 저하와 훈련 충돌을 야기한다.6 MoT는 양식별로 파라미터를 분리함으로써 이러한 문제를 근본적으로 회피한다. 각 '전문가 트랜스포머'는 자신이 맡은 양식에만 집중하여 고도로 전문화된 표현을 학습할 수 있다.
- **파라미터 수:** 동일한 연산량(FLOPs)을 기준으로 비교할 때, MoT는 밀도 모델보다 더 많은 총 파라미터 수를 가진다. 그러나 이 파라미터들은 희소하게 활성화되므로, 실제 연산 비용은 훨씬 낮다. 이는 더 큰 모델 용량(capacity)을 효율적으로 활용하는 희소 모델의 전형적인 특징이다.


MoT와 MoE는 모두 희소 아키텍처라는 공통점을 가지지만, 그 철학과 구현 방식에는 근본적인 차이가 존재한다. 이 둘의 비교는 MoT의 독창성을 이해하는 데 매우 중요하다.

- **라우팅 메커니즘:** MoE의 핵심은 학습 가능한 게이트 네트워크를 통해 각 *토큰*을 처리할 전문가를 동적으로 선택하는 것이다.5 이 라우팅은 유연하지만, 훈련 과정에서 어떤 전문가에게 토큰이 쏠리는 '부하 불균형'이나 동일한 토큰이 훈련 단계에 따라 다른 전문가에게 할당되는 '라우팅 변동'과 같은 불안정성을 내포한다.12 이를 해결하기 위해 복잡한 보조 손실 함수가 필요하다. 반면, MoT는 사전 정의된 

  `modality_masks`를 사용하여 *양식*에 따라 토큰을 결정론적으로 라우팅한다.14 이 단순함은 훈련 안정성을 크게 높이고, 라우팅 오버헤드를 최소화한다. 멀티모달 맥락에서는 이러한 간단한 규칙 기반 라우팅이 복잡한 학습 기반 라우팅보다 더 효과적이라는 연구 결과도 있다.5

- **희소성의 단위:** MoT는 거시적인 '양식 수준(modality-level)'의 희소성을 구현한다. 즉, 전문화의 단위가 텍스트, 이미지, 음성과 같은 데이터의 종류다. 반면, MoE는 미시적인 '토큰 수준(token-level)'의 희소성을 구현한다. 전문가들은 특정 주제(예: 과학, 역사)나 문체(예: 격식체, 구어체)에 따라 전문화될 수 있지만, 이는 데이터에서부터 학습되어야 한다.

- **파라미터 확장성:** MoE 모델의 총 파라미터 수는 전문가의 수에 비례하여 증가한다. 전문가 수를 4배로 늘린 MoE-4x 모델은 파라미터 수도 거의 4배가 된다. 반면, MoT의 파라미터 수는 처리하는 *양식의 수*에 비례하여 증가한다.5 이는 새로운 양식을 추가할 때 MoT가 MoE보다 더 유리한 파라미터 대 연산량 비율을 가짐을 의미한다.5

- **성능 비교:** 광범위한 실험에서 MoT는 MoE-4x와 같은 MoE 기준 모델보다, 특히 텍스트가 아닌 양식(이미지, 음성)에서 더 우수한 성능과 더 큰 실제 훈련 시간 단축 효과를 보였다.5 또한 MoE는 훈련 데이터와 평가 데이터의 분포가 다를 경우 성능이 불안정해질 수 있는 약점이 지적되었다.8


아래 표는 세 아키텍처의 핵심적인 차이점을 요약한 것이다.

| **특성**               | **밀도 트랜스포머 (Dense Transformer)** | **전문가 혼합 (MoE) 트랜스포머**          | **트랜스포머 혼합 (MoT)**                   |
| ---------------------- | --------------------------------------- | ----------------------------------------- | ------------------------------------------- |
| **희소성 유형**        | 없음 (모든 파라미터 활성화)             | 미세 단위 (토큰 수준) 희소성              | 거대 단위 (양식 수준) 희소성                |
| **라우팅 메커니즘**    | 없음 (단일 경로)                        | 학습 기반 동적 라우팅 (게이트 네트워크)   | 규칙 기반 결정론적 라우팅 (모달리티 마스크) |
| **파라미터 활성화**    | 100%                                    | 조건부 (상위 k개 전문가)                  | 조건부 (해당 양식의 전문가)                 |
| **주요 장점**          | 구조적 단순성                           | 범용적이고 유연한 모델 용량 확장          | 멀티모달에 특화된 높은 효율성 및 안정성     |
| **주요 단점/오버헤드** | 높은 연산 비용, 양식 간 간섭            | 라우팅 복잡성, 부하 불균형, 통신 오버헤드 | 토큰 그룹핑 오버헤드, 양식 내 전문화 부재   |

일부에서는 MoT가 "기존 MoE 기술을 멀티모달 맥락에 적용한 점진적인 혁신"에 가깝다고 평가하기도 한다.17 이러한 비판은 MoT가 '전문가'와 '라우팅'이라는 MoE의 기본 개념을 차용했다는 점에서 일리가 있다. 그러나 MoT의 진정한 혁신성은 그 적용의 '특수성'과 '단순성'에 있다. MoT는 MoE의 복잡성을 과감히 버리고, '양식'이라는 멀티모달 데이터의 가장 강력한 구조적 선험적 지식(inductive bias)을 아키텍처에 직접 주입했다. 이 결정이 가져온 압도적인 성능 및 안정성 향상은 MoT가 단순한 변종이 아닌, 특정 문제 영역에 대한 깊은 통찰을 바탕으로 한 독자적인 패러다임임을 증명한다.

결론적으로, MoT의 설계는 MoE의 범용적인 불가지론(agnosticism)에서 벗어난 철학적 전환을 보여준다. MoE가 일반적인 스케일링 도구라면, MoT는 멀티모달 데이터의 본질에 대한 강한 가설, 즉 '양식이야말로 가장 효과적인 전문화의 축'이라는 가설을 내장한 특수 목적 아키텍처이다. MoT가 비텍스트 양식에서 MoE를 능가하는 결과 5는 단일 전문가 풀이 모든 양식을 처리하도록 강제하는 것이 최적이 아님을 시사한다. 데이터의 '이미지다움'이나 '음성다움'이 개별 토큰의 의미 내용보다 전문화에 더 중요한 특징일 수 있다는 것이다. 이는 MoT가 MoE의 또 다른 유형이 아니라, 특정 문제 영역에 대한 토큰 기반 라우팅의 보편성에 도전하는 아키텍처임을 의미한다. MoT의 성공은 복잡하고 이질적인 데이터에 대해, 학습된 유연성과 설계된 구조적 편향 사이의 균형을 맞추는 것이 얼마나 중요한지에 대한 중요한 교훈을 제공하며, 이는 미래 아키텍처 설계에 깊은 영향을 미칠 것이다.


아키텍처의 우수성은 이론적 우아함뿐만 아니라, 엄격한 실증적 데이터를 통해 입증되어야 한다. Mixture-of-Transformers (MoT)는 다양한 멀티모달 설정과 모델 규모에 걸쳐 광범위한 실험을 통해 그 효율성과 효과성을 체계적으로 입증했다. 본 장에서는 원본 연구에서 제시된 정량적 결과를 종합하여, MoT가 밀도(Dense) 모델 및 MoE 모델 대비 갖는 성능상의 이점을 명확히 분석한다.


Chameleon은 텍스트와 이미지를 번갈아 입력받아 다음 토큰(텍스트 또는 이미지)을 예측하는 자기회귀(autoregressive) 방식으로 훈련되는 대표적인 멀티모달 생성 모델 설정이다.

- **연산량(FLOPs) 감소:** 70억(7B) 파라미터 규모에서, MoT 모델은 밀도 모델과 동등한 최종 성능(validation loss)을 달성하면서 사용된 총 FLOPs는 **55.8%**에 불과했다.1 이는 동일한 성능을 거의 절반의 연산 비용으로 달성했음을 의미한다.
- **훈련 가속화:** MoT는 훈련 속도에서도 압도적인 우위를 보였다. 밀도 모델이 최종 성능에 도달하기까지 걸린 훈련 단계(training steps)의 **45.5%**만으로 동일한 수준의 손실 값에 도달했다.5 이러한 가속 효과는 특히 연산 집약적인 이미지 양식에서 더욱 두드러져, 이미지 양식의 손실 값을 일치시키는 데에는 단 **34.8%**의 훈련 단계만이 필요했다.5
- **실제 훈련 시간(Wall-Clock Time) 단축:** 이론적인 FLOPs 감소는 실제 훈련 시간 단축으로 이어진다. NVIDIA A100 GPU를 탑재한 AWS 인스턴스에서 측정한 결과, MoT는 밀도 모델 수준의 이미지 품질을 달성하는 데 **47.2%**의 시간만을 소요했으며, 텍스트 품질은 **75.6%**의 시간으로 달성했다.2 이는 MoT의 구조가 분산 시스템에서 효율적으로 작동함을 시사한다.


Chameleon 설정에 음성 양식을 추가하여 3개의 양식을 동시에 처리하는 더욱 복잡한 환경에서도 MoT의 효율성은 더욱 빛을 발했다.

- **연산량(FLOPs) 감소:** 음성이 추가되었을 때, 효율성 증가는 더욱 극적으로 나타났다. MoT는 밀도 모델 기준선과 유사한 음성 처리 성능을 단 **37.2%**의 FLOPs로 달성했다.1 이는 양식의 수가 늘어날수록 MoT의 희소성이 더 큰 비용 절감 효과를 가져온다는 것을 보여준다.

- **훈련 가속화:** 음성 양식의 사전 훈련 손실 값을 기준으로, MoT는 밀도 모델이 소요한 훈련 단계의 **22.9%**만으로 동일한 수준에 도달했다.5 3개 양식 전체를 기준으로 보았을 때도, MoT는 총 훈련 단계의 

  **55.8%** 지점에서 밀도 모델의 최종 검증 손실 값과 동등한 수준을 달성했다.5


Transfusion은 텍스트에 대해서는 자기회귀 목표를, 이미지에 대해서는 확산(diffusion) 모델 목표를 사용하는 등, 양식별로 상이한 훈련 목표를 갖는 복합적인 설정이다.

- **연산량(FLOPs) 감소:** 이러한 복잡한 다중 목표 환경에서도 MoT의 장점은 명확했다. 7B 규모의 MoT 모델은 밀도 모델 기준선의 이미지 양식 성능과 동등한 수준을 달성하면서 FLOPs는 **3분의 1** 수준만 사용했다.1
- **훈련 가속화:** 이미지 양식의 사전 훈련을 기준으로, MoT는 밀도 모델 대비 **30%**의 훈련 단계만으로 동일한 성능에 도달하여 상당한 훈련 가속 효과를 보였다.5
- **소규모 모델의 우월성:** MoT의 효율성은 더 작은 모델이 더 큰 밀도 모델을 능가하는 현상으로도 나타났다. 7억 6천만(760M) 파라미터를 가진 MoT 모델은, 그보다 거의 두 배 큰 14억(1.4B) 파라미터의 밀도 모델을 프레셰 인셉션 거리(FID)와 같은 핵심 이미지 생성 지표에서 능가했다.1 이는 MoT가 파라미터를 훨씬 효율적으로 사용하여 더 높은 성능을 이끌어냄을 증명한다.


MoT의 성능 우위는 특정 모델 크기에 국한되지 않고, 3,700만(37M)에서 70억(7B) 파라미터에 이르는 광범위한 모델 규모에서 일관되게 나타났다.5 특히 MoE-4x 모델이 모델 크기가 커짐에 따라 효율성 증가세가 둔화되는(diminishing returns) 경향을 보인 반면, MoT는 꾸준한 성능 이점을 유지했다.5

아래 표는 주요 실험 설정에서 MoT가 밀도 모델 대비 보인 성능 및 효율성 향상을 요약한 것이다.

| **설정**                    | **양식**        | **주요 지표**         | **밀도 모델 대비 MoT 성능** |
| --------------------------- | --------------- | --------------------- | --------------------------- |
| **Chameleon (7B)**          | 이미지 & 텍스트 | 총 FLOPs              | **55.8%** 1                 |
|                             | 이미지          | Wall-Clock Time       | **47.2%** 2                 |
|                             | 텍스트          | Wall-Clock Time       | **75.6%** 2                 |
|                             | 이미지          | 훈련 단계 (손실 일치) | **34.8%** 5                 |
| **Chameleon + 음성 (1.5B)** | 음성 포함       | 총 FLOPs              | **37.2%** 1                 |
|                             | 음성            | 훈련 단계 (손실 일치) | **22.9%** 5                 |
| **Transfusion (7B)**        | 이미지          | 총 FLOPs              | **약 33.3%** 1              |
|                             | 이미지          | 훈련 단계 (손실 일치) | **30.0%** 5                 |

이러한 수치들은 MoT 아키텍처의 가치를 명백히 보여준다. 특히 주목할 점은 양식의 수가 2개(텍스트+이미지)에서 3개(텍스트+이미지+음성)로 늘어났을 때, FLOPs 효율성이 55.8%에서 37.2%로 더욱 향상되었다는 것이다. 이는 MoT의 효율성 이득이 양식의 수에 대해 초선형적(super-linear) 관계를 가질 수 있음을 시사한다. 즉, 아키텍처가 처리해야 할 데이터의 이질성이 증가할수록 MoT의 가치 제안은 더욱 강력해진다.

이러한 경향은 MoT가 단순히 효율적인 아키텍처를 넘어, 미래의 '다중-양식(many-modality)' AI를 위한 확장 가능한 청사진임을 암시한다. AI가 텍스트, 이미지, 음성을 넘어 비디오, 센서 데이터, 3D 정보 등 더욱 다양한 양식을 통합하는 방향으로 나아갈수록, 밀도 아키텍처는 연산적으로 다루기 불가능해질 것이다. MoT는 이러한 '양식의 저주(curse of modality)'에 맞서 싸울 수 있는 명확한 구조적 경로를 제시하며, 미래 지향적인 설계 패턴으로서의 중요성을 입증한다.


아키텍처의 이론적 우수성이 실제 시스템에서의 높은 성능으로 직결되기 위해서는 구현의 용이성과 시스템 수준의 효율성이 뒷받침되어야 한다. MoT는 그 설계의 단순함과 예측 가능성 덕분에 이론적 연산량(FLOPs) 절감 효과를 뛰어넘는 실제 훈련 시간(wall-clock time) 단축을 달성한다. 본 장에서는 MoT의 코드 수준 구현 방식을 살펴보고, 시스템 오버헤드 및 분산 훈련 환경에서의 이점을 분석하여 이론과 실제 사이의 간극을 탐구한다.


MoT 아키텍처는 기존 트랜스포머 모델 위에 모듈식으로 확장될 수 있도록 설계되었다. Facebook Research가 공개한 공식 GitHub 저장소의 구현 가이드 14는 MoT의 핵심 구성 요소인 `ModalityUntiedFeedForward`와 `ModalityUntiedAttention` 클래스를 통해 그 작동 방식을 명확히 보여준다.

- **`ModalityUntiedFeedForward` 클래스:** 이 클래스는 트랜스포머의 피드포워드 네트워크(FFN)를 양식별로 분리하는 역할을 한다.
  - **초기화:** 생성자(`__init__`)는 처리할 양식의 수(`n_modalities`)를 인자로 받아, 각 양식에 해당하는 FFN 전문가(expert)와 레이어 정규화(LayerNorm) 계층을 `torch.nn.ModuleList` 형태로 초기화한다. 이는 각 양식이 독립적인 파라미터를 갖도록 보장한다.
  - **순전파(Forward Pass):** `forward` 메소드는 입력 텐서 `x`와 `modality_masks`를 입력받는다. `modality_masks`를 사용하여 각 양식에 속하는 토큰들을 선별하고, 해당 양식의 FFN 전문가와 정규화 계층으로 라우팅하여 처리한다. 처리된 각 양식의 출력은 별도로 수집된 후, 다시 `modality_masks`를 이용해 원래의 순서대로 하나의 출력 텐서로 병합된다. 이 과정이 MoT의 결정론적 라우팅을 코드 수준에서 구현한 것이다.14
- **`ModalityUntiedAttention` 클래스:** 이 클래스는 표준 어텐션 메커니즘을 확장하여 양식별 프로젝션을 구현한다.
  - **초기화:** FFN과 마찬가지로, 각 양식에 대한 독립적인 쿼리(Wq), 키(Wk), 값(Wv), 출력(Wo) 프로젝션 행렬과 관련 정규화 계층들을 초기화한다.
  - **순전파(Forward Pass):** 작동 흐름은 '분리-통합-분리'의 3단계로 요약할 수 있다.
    1. **분리 (QKV 프로젝션):** 입력 토큰들은 `modality_masks`에 따라 각자의 Wq,Wk,Wv 전문가에게 라우팅되어 양식별 쿼리, 키, 값 벡터로 변환된다.
    2. **통합 (전역 어텐션):** 양식별로 생성된 Q, K, V 벡터들은 다시 하나의 통합된 시퀀스로 재결합된다. 이 통합 시퀀스에 대해 전역적 자기 어텐션 연산이 수행되어 양식 간 상호작용이 일어난다.
    3. **분리 (출력 프로젝션):** 어텐션의 결과물은 다시 `modality_masks`에 따라 각자의 Wo 전문가와 정규화 계층으로 라우팅되어 최종 출력을 형성한다.14

이러한 모듈식 구현은 기존 트랜스포머 코드베이스에 비교적 쉽게 통합될 수 있어, MoT 아키텍처의 실용성을 높이는 중요한 요소이다.


어떤 희소 아키텍처든 희소성을 구현하기 위한 추가적인 오버헤드가 발생한다. MoT와 MoE의 오버헤드를 비교하면 MoT 설계의 실용적 이점이 더욱 명확해진다.

- **MoT의 오버헤드:** MoT는 토큰을 양식별로 그룹화하고 다시 병합하는 과정에서 약간의 "토큰 그룹핑(token-grouping)" 오버헤드를 발생시킨다.5 그러나 이는 결정론적 마스킹 연산에 기반하므로 계산적으로 복잡하지 않으며, "성실한 엔지니어링(diligent engineering)"을 통해 최소화할 수 있다.5
- **MoE의 오버헤드:** MoE의 오버헤드는 훨씬 더 크고 복잡하다. 첫째, "복잡한 라우팅(complex routing)" 메커니즘 자체의 연산 비용이 존재한다. 둘째, 토큰을 각기 다른 전문가(GPU)에게 분배하고 결과를 다시 취합하는 과정에서 상당한 통신 오버헤드(주로 all-to-all 연산)가 발생한다. 셋째, 부하 불균형을 방지하기 위한 보조 손실 함수 계산이 추가적인 연산 부담을 준다.5

결론적으로 MoT의 오버헤드는 MoE에 비해 훨씬 작고 예측 가능하여, 시스템 수준에서의 효율성을 높이는 데 기여한다.


대규모 모델 훈련은 필연적으로 수백, 수천 개의 GPU를 사용하는 분산 환경에서 이루어진다. 이러한 환경에서 MoT의 설계는 MoE 대비 뚜렷한 강점을 보인다.

- **정적 연산 그래프와 부하 균형:** MoT의 결정론적 라우팅은 훈련 시작 전에 전체 연산 및 통신 그래프가 고정됨을 의미한다. 이는 모든 GPU가 예측 가능한 양의 작업을 수행하게 하여, MoE에서 고질적으로 발생하는 "부하 불균형" 문제를 원천적으로 방지한다.12 MoE에서는 특정 전문가에게 토큰이 몰릴 경우 해당 전문가를 담당하는 GPU는 과부하 상태가 되고, 다른 GPU는 유휴 상태가 되어 전체 시스템의 효율을 떨어뜨린다.
- **통신 최적화:** MoT의 예측 가능한 통신 패턴은 시스템 엔지니어들이 통신과 계산을 효과적으로 중첩(overlap)시키는 등 다양한 최적화 기법을 적용하기 용이하게 만든다. 반면, MoE의 동적이고 데이터에 의존적인 통신 패턴은 최적화를 훨씬 더 어렵게 만든다.

이러한 시스템 수준의 이점은 MoT의 실제 훈련 시간(wall-clock time) 개선 효과 2가 이론적인 FLOPs 절감률을 상회하는 이유를 설명해준다.8 이는 MoT의 설계자들이 단순히 수학적 우아함만을 추구한 것이 아니라, 대규모 분산 하드웨어에서의 실제적인 고성능 구현을 염두에 두고 알고리즘과 시스템을 함께 설계(co-design)했음을 보여준다. MoT의 단순성은 약점이 아니라, 하드웨어 활용률을 극대화하기 위해 의도된 '기능(feature)'이다. 이는 알고리즘이 진공 상태에서 개발되는 것이 아니라, 기반이 되는 하드웨어 및 시스템과 함께 공동으로 설계될 때 가장 큰 실용적 가치를 창출할 수 있다는 성숙한 엔지니어링 관점을 보여주는 대표적인 사례이다.


Mixture-of-Transformers (MoT)의 영향력은 단일 연구 논문에 머무르지 않고, 실제 대규모 모델에 적용되고 새로운 아키텍처 패밀리에 영감을 주면서 그 가치를 확장하고 있다. MoT의 핵심 원리인 '양식 인지 희소성'은 이제 특정 아키텍처를 넘어, 효율적인 멀티모달 AI를 구축하기 위한 검증된 '설계 패턴(design pattern)'으로 자리 잡고 있다.


MoT 아키텍처의 잠재력이 실제 최첨단 시스템에서 어떻게 구현되었는지를 보여주는 가장 대표적인 사례는 ByteDance의 **BAGEL** 모델이다.18

- **모델 개요:** BAGEL은 텍스트, 이미지, 비디오의 통합된 이해와 생성을 지원하는 오픈소스 통합 멀티모달 파운데이션 모델이다. 총 140억(14B) 개의 파라미터(활성 파라미터는 7B)를 보유한 대규모 모델로, 현재 오픈소스 비전-언어 모델(VLM) 중 최고 수준의 성능을 자랑한다.21
- **MoT 아키텍처 채택:** BAGEL의 문서와 관련 분석 자료들은 모델의 뛰어난 성능이 "선택적으로 양식별 파라미터를 활성화하여 결과를 최적화하는 독특한 **mixture-of-transformers-experts (MoT)**" 아키텍처 덕분이라고 명시적으로 밝히고 있다.21 이는 MoT가 학술적 개념을 넘어, 실제 상용 수준의 시스템에서 핵심적인 역할을 수행할 수 있음을 입증한다.
- **고급 기능 구현:** BAGEL은 단순한 이미지 캡셔닝이나 생성을 넘어, 자유로운 형식의 이미지 편집, 다중 시점 합성, 심지어 가상 세계 항해와 같은 복잡한 작업을 수행할 수 있다.18 이는 MoT 구조가 양식별 전문성과 양식 간 통합을 효과적으로 조화시켜 고차원적인 멀티모달 추론 능력을 이끌어낼 수 있음을 보여준다.
- **구조적 유사성:** 최근 제안된 UniFork라는 다른 모델의 논문에서는, 특정 구성에서 "BAGEL에서 제안된 Mixture-of-Transformers 설계와 구조적으로 유사해진다"고 언급하며, 이는 얕은 레이어를 공유하고 깊은 레이어에서 작업별(양식별) 브랜치로 나뉘는 Y자형 구조일 수 있음을 시사한다.24 이는 MoT의 개념이 다양한 형태로 변주되어 적용되고 있음을 보여준다.


MoT의 설계 원리는 트랜스포머 아키텍처의 경계를 넘어 다른 모델 패밀리로 성공적으로 이식되었다. 이는 MoT가 하나의 특정 구현이 아니라, 이식 가능한 일반적인 원칙임을 증명한다.

- **MoT로부터의 직접적인 영감:** 스탠포드, CMU, Meta FAIR의 연구진들은 트랜스포머의 유력한 대안으로 부상한 상태 공간 모델(State Space Models, SSMs)의 멀티모달 처리 효율성을 높이기 위해, "Mixture-of-Transformers (MoT)에서 직접적으로 영감을 받아" **Mixture-of-Mamba (MoM)** 아키텍처를 개발했다.25
- **핵심 원리의 이식:** MoM은 MoT의 핵심 철학인 "양식 인지 희소성"을 Mamba라는 SSM 아키텍처에 그대로 적용한다.16 구체적으로, Mamba 블록 내부에서 입력, 중간, 출력 프로젝션 레이어들을 양식에 따라 분리하고, 순환적 상태 전이 및 컨볼루션과 같은 핵심 연산 요소는 공유하는 방식을 취한다.16
- **유사한 성능 향상:** 결과는 MoT의 성공을 그대로 재현했다. MoM은 밀도 Mamba 모델과 동등한 성능을 훨씬 적은 FLOPs로 달성했다. 예를 들어, Transfusion 설정에서 이미지 손실 값을 일치시키는 데 단 **34.76%**의 FLOPs만을 사용했다.16 이는 '양식별 분리'라는 MoT의 원리가 트랜스포머의 자기 어텐션 메커니즘에만 국한되지 않는, 보다 일반적인 원칙임을 강력하게 시사한다.

MoT가 하나의 특정 아키텍처에서 시작하여, 실제 대규모 모델(BAGEL)에 채택되고, 심지어 전혀 다른 아키텍처(Mamba)에 영감을 주어 새로운 패밀리(MoM)를 탄생시킨 이 과정은 MoT가 AI 커뮤니티에 미친 깊은 영향력을 보여준다. 이는 MoT가 단순히 하나의 논문으로 끝나는 것이 아니라, 멀티모달 AI를 위한 새로운 설계 패러다임을 확립했음을 의미한다. '양식에 따라 연산을 분리한다'는 이 단순하고도 강력한 아이디어는 이제 트랜스포머, SSM 등 기반 시퀀스 처리 메커니즘에 관계없이 효율적인 멀티모달 모델을 구축하고자 할 때 고려할 수 있는 핵심적인 도구 중 하나가 되었다. 이는 MoT를 특정 논문을 넘어, AI 아키텍처의 역사에 중요한 개념적 기여를 한 사례로 평가할 수 있게 한다.


Mixture-of-Transformers (MoT)는 멀티모달 AI의 스케일링 문제를 해결하기 위한 강력하고 효율적인 패러다임을 제시했다. 양식이라는 명확한 구조적 기준을 활용하여 전문성과 통합성을 동시에 달성하는 이 아키텍처는 이론적, 실증적으로 그 가치를 입증했다. 본 장에서는 지금까지의 분석을 종합하고, 아키텍처의 한계를 인식하며, MoT와 MoE를 결합하는 하이브리드 모델을 포함한 가장 유망한 미래 연구 방향을 조망하고자 한다.


모든 기술적 진보와 마찬가지로 MoT 역시 한계와 비판점을 가지고 있으며, 이를 인정하는 것은 균형 잡힌 평가와 미래 발전을 위해 필수적이다.

- **점진적 혁신성:** 일부에서는 MoT의 개념이 기존 MoE 기술을 멀티모달이라는 특정 영역에 적용한 것으로, "점진적인 혁신(incremental novelty)"에 해당한다고 평가한다.17 이 비판은 MoT가 완전히 새로운 개념을 창조한 것이 아니라 기존 아이디어를 정교하게 조합했다는 점에서 타당한 측면이 있다. 그러나 그 조합이 가져온 압도적인 효율성과 안정성은 그 자체로 중요한 혁신으로 평가받을 가치가 충분하다.
- **정적 희소성의 한계:** MoT의 초기 평가는 주로 통제된 실험 환경에서 이루어졌다. 현재의 MoT는 양식별로 고정된 희소성 패턴을 사용하지만, 실제 애플리케이션의 요구사항이나 실시간 연산 부하에 따라 양식별 희소성 패턴을 동적으로 조절하는 방안에 대한 탐구는 향후 과제로 남아있다.27 예를 들어, 특정 작업에서는 이미지 처리에 더 많은 연산 자원을 할당하고 텍스트에는 적게 할당하는 동적 조정이 필요할 수 있다.


MoT의 미래에서 가장 흥미롭고 유망한 방향은 바로 MoT와 MoE를 결합하는 하이브리드 모델의 가능성이다. 이는 MoT의 구조적 안정성과 MoE의 유연한 전문성을 모두 활용하려는 시도로, 원본 MoT 논문 자체에서도 "탐색적인(exploratory)" 실험을 통해 그 잠재력을 시사했다.8 이러한 하이브리드 접근법은 "훨씬 더 유능한 멀티모달 AI"를 구축하기 위한 유망한 경로로 강조되고 있다.27

이 하이브리드 모델의 핵심 개념은 MoT의 양식별 전문가 '타워(tower)' 내부에 MoE 레이어를 '중첩(nesting)'시키는 것이다.5 예를 들어, 3개 양식(텍스트, 이미지, 음성)을 처리하는 하이브리드 모델은 다음과 같은 2단계 희소 계층 구조를 가질 수 있다.

1. **거시적 희소성 (MoT):** 최상위 레벨에서는 MoT가 작동한다. 입력 토큰은 양식에 따라 결정론적으로 '텍스트 타워', '이미지 타워', '음성 타워'로 라우팅된다. 이 단계는 안정적이고 거시적인 전문화를 담당한다.
2. **미시적 희소성 (MoE):** 각 양식별 타워 내부에는 독립적인 MoE 레이어가 존재한다. 예를 들어, '텍스트 타워'는 내부적으로 4개의 전문가를 가진 MoE 레이어를 포함하여, 코드, 학술 논문, 소설, 대화 등 다양한 종류의 텍스트를 처리하는 데 미시적으로 전문화될 수 있다. 마찬가지로 '이미지 타워'는 사진, 삽화, 다이어그램 등을 처리하는 내부 전문가들을 가질 수 있다.

이러한 계층적 희소 모델은 구조화되고 도메인에 특화된 희소성(MoT)과 유연하고 범용적인 희소성(MoE) 사이의 변증법적 종합을 나타낸다. 이는 두 접근법의 주요 약점을 해결하고, 각각의 장점만을 취하는 '두 세계의 최고(best-of-both-worlds)' 아키텍처를 구현한다.

- **MoT의 한계 보완:** MoT는 양식 내에서의 세분화된 전문성이 부족할 수 있다. 하이브리드 모델은 MoE를 통해 이를 보완한다.
- **MoE의 한계 보완:** MoE는 이질적인 양식이 섞여 있을 때 라우팅이 불안정하고 비효율적일 수 있다. 하이브리드 모델은 MoT를 통해 먼저 양식을 분리함으로써 MoE가 구조적으로 유사한 데이터 내에서만 작동하도록 하여 안정성과 효율성을 높인다.


MoT와 MoE의 결합은 단순히 두 기술을 합치는 것을 넘어, 미래 AI를 구축하는 방법에 대한 심오한 통찰을 제공한다. 이는 확장성의 열쇠가 단일한 거대 희소 메커니즘이 아니라, 데이터의 각기 다른 추상화 수준(양식, 하위 도메인, 토큰)에 맞춰진 *희소 메커니즘의 계층(hierarchy of sparse mechanisms)*에 있음을 시사한다.

이러한 계층적 접근법은 관리 가능한 복잡성으로 막대한 규모의 확장을 가능하게 하는 강력하고 일반화 가능한 패러다임이다. MoT 아키텍처는 그 자체로 멀티모달 AI의 효율성을 한 단계 끌어올렸으며, 더 나아가 하이브리드 모델이라는 새로운 지평을 열어주었다. 앞으로 AI가 점점 더 다양하고 복잡한 데이터 양식을 원활하게 통합해야 하는 과제에 직면함에 따라, MoT와 그로부터 파생된 계층적 희소 아키텍처는 차세대 파운데이션 모델 개발을 이끄는 핵심적인 청사진 역할을 할 것으로 기대된다.27


1. Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models | Request PDF - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/385630710_Mixture-of-Transformers_A_Sparse_and_Scalable_Architecture_for_Multi-Modal_Foundation_Models
2. Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models - Hugging Face, accessed July 16, 2025, https://huggingface.co/papers/2411.04996
3. Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models - Consensus, accessed July 16, 2025, https://consensus.app/papers/mixtureoftransformers-a-sparse-and-scalable-zhou-luo/371f6bdc68be509eb066796817f15736/
4. [2411.04996] Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2411.04996
5. Mixture-of-Transformers: A Deep Dive into Scalable Multi-Modal Models - Clio AI, accessed July 16, 2025, https://www.clioapp.ai/research/mixture-of-transformers
6. Mixture-of-Transformers (MoT): A New Era in Multi-Modal AI Efficiency - Medium, accessed July 16, 2025, https://medium.com/@sahin.samia/mixture-of-transformers-mot-a-new-era-in-multi-modal-ai-efficiency-e3ac2699eeb0
7. Meta AI Researchers Introduce Mixture-of-Transformers (MoT): A Sparse Multi-Modal Transformer Architecture that Significantly Reduces Pretraining Computational Costs - MarkTechPost, accessed July 16, 2025, https://www.marktechpost.com/2024/11/13/meta-ai-researchers-introduce-mixture-of-transformers-mot-a-sparse-multi-modal-transformer-architecture-that-significantly-reduces-pretraining-computational-costs/
8. Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models | OpenReview, accessed July 16, 2025, https://openreview.net/forum?id=Nu6N69i8SB
9. Learning Self-supervised User Representations Using Contextualized Mixture of Experts in Transformers - Amazon Science, accessed July 16, 2025, https://assets.amazon.science/e3/d2/309706cc4d4e8fb615237eac444c/learning-self-supervised-user-representations-using-contextualized-mixture-of-experts-in-transformers.pdf
10. Mixture-of-Transformers: A Sparse and Scalable Architecture for, accessed July 16, 2025, https://www.alphaxiv.org/overview/2411.04996v1
11. Daily Papers - Hugging Face, accessed July 16, 2025, [https://huggingface.co/papers?q=Mixture-of-Modality-Experts%20(MoME)%20Transformer](https://huggingface.co/papers?q=Mixture-of-Modality-Experts+(MoME)+Transformer)
12. StableMoE: Stable Routing Strategy for Mixture of Experts | Request PDF - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/361056961_StableMoE_Stable_Routing_Strategy_for_Mixture_of_Experts
13. Meta AI Researchers Introduce Mixture-of-Transformers (MoT): A Sparse Multi-Modal Transformer Architecture that Significantly Reduces Pretraining Computational Costs : r/machinelearningnews - Reddit, accessed July 16, 2025, https://www.reddit.com/r/machinelearningnews/comments/1gqzkgu/meta_ai_researchers_introduce/
14. facebookresearch/Mixture-of-Transformers: Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models. TMLR 2025. - GitHub, accessed July 16, 2025, https://github.com/facebookresearch/Mixture-of-Transformers
15. Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=rubEGu7WWq0
16. Mixture-of-Mamba: Enhancing Multi-Modal State ... - OpenReview, accessed July 16, 2025, https://openreview.net/pdf/4d8a9a4ac50088d179da68d41e2224b4ae6835f5.pdf
17. Mixture-of-Transformers: A Sparse and Scalable Architecture for ..., accessed July 16, 2025, https://openreview.net/forum?id=fn2U1VYfQ5
18. BAGEL: The Open-Source Unified Multimodal Model - ByteDance Seed, accessed July 16, 2025, https://seed.bytedance.com/blog/seed-research-bagel-the-open-source-unified-multimodal-model-an-all-in-one-model
19. [2505.14683] Emerging Properties in Unified Multimodal Pretraining - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2505.14683
20. Bytedance Seed-Multimodal, accessed July 16, 2025, https://seed.bytedance.com/direction/multimodal
21. Run BAGEL VLM on a DigtialOcean GPU Droplet - DigitalOcean, accessed July 16, 2025, https://www.digitalocean.com/community/tutorials/bagel-vlm-gpu-droplet
22. ByteDance-Seed/Bagel: Open-source unified multimodal model - GitHub, accessed July 16, 2025, https://github.com/ByteDance-Seed/Bagel
23. Getting Started with BAGEL VLM on a DigitalOcean GPU Droplet: A Step-by-Step Guide, accessed July 16, 2025, https://dedirock.com/blog/getting-started-with-bagel-vlm-on-a-digitalocean-gpu-droplet-a-step-by-step-guide/
24. UniFork: Exploring Modality Alignment for Unified Multimodal Understanding and Generation - arXiv, accessed July 16, 2025, https://arxiv.org/html/2506.17202v1
25. Topic 28: What is Mixture-of-Mamba? - Hugging Face, accessed July 16, 2025, https://huggingface.co/blog/Kseniase/mixtureofmamba
26. Topic 28: What is Mixture-of-Mamba? - Turing Post, accessed July 16, 2025, https://www.turingpost.com/p/mixtureofmamba
27. Mixture-of-Transformers: Sparse Multi-Modal Model - Emergent Mind, accessed July 16, 2025, https://www.emergentmind.com/papers/2411.04996
28. Mixture-of-Transformers: A Sparse and Scalable Architecture for Multi-Modal Foundation Models - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=o9mSFm3E7T0


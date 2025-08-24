# AnyLoc 보편적 시각 장소 인식(VPR)을 향한 패러다임 전환 (2023)
[시각적 장소 인식 (Visual Place Recognition)](./index.md)



시각 장소 인식(Visual Place Recognition, VPR)은 로봇 공학 및 자율 시스템 분야의 근간을 이루는 기술이다. 이는 주어진 질의(query) 이미지를 이전에 구축된 지리적 태그가 부착된 이미지 데이터베이스와 비교하여, 현재 위치를 인식하는 이미지 검색 문제로 정의된다.1 VPR은 동시적 위치 추정 및 지도 작성(SLAM) 시스템에서 루프 폐쇄(loop closure)를 감지하고, 이전에 방문했던 장소로 재위치(re-localization)하는 데 결정적인 역할을 한다.2 특히 GPS 신호가 불안정하거나 수신 불가능한 실내, 지하, 수중 환경이나 도심의 빌딩숲(urban canyon)과 같은 음영 지역에서 로봇의 자율적 항법을 가능하게 하는 핵심 요소이다.4 VPR의 근본적인 도전 과제는 조명, 계절, 시점의 극심한 변화 속에서도 동일한 장소임을 정확히 식별하는 능력에 있다.6


지난 수년간 VPR 연구는 괄목할 만한 발전을 이루었으나, 최신(SOTA) 기술들은 대부분 환경 및 과업 특화(environment- and task-specific)라는 본질적인 한계를 지닌다.8 주로 도시 주행 환경과 같은 정형화된(structured) 대규모 데이터셋(예: Pittsburgh-30k, GSV-Cities)으로 훈련된 모델들은 해당 도메인 내에서는 높은 성능을 보이지만, 훈련 데이터의 분포에서 벗어나는 비정형(unstructured) 환경—가령 항공, 수중, 지하 동굴 등—에 적용될 경우 성능이 급격히 저하되는 '취약성'을 드러낸다.9 이러한 데이터 및 과업 의존성은 실제 세계의 다양한 환경에서 로봇을 안정적으로 운용하는 데 있어 가장 큰 걸림돌로 작용한다.9

AnyLoc 논문은 이 문제를 시각적으로 명확히 보여준다. VPR에 특화된 SOTA 모델인 MixVPR의 특징(feature)들을 주성분 분석(PCA)으로 시각화했을 때, 다양한 환경의 입력에 대해 특징 공간의 매우 좁은 영역에만 집중되어 변별력을 상실하는 모습을 보인다.11 이는 특정 도메인에 과적합된 지도 학습(supervised learning) 방식이 보편적 장소 인식을 위한 일반화된 특징을 학습하는 데 실패했음을 시사한다. 문제의 본질은 네트워크 아키텍처가 아니라, 편향된 데이터와 그에 기반한 훈련 패러다임 자체에 있는 것이다.


이러한 한계를 극복하기 위해 AnyLoc은 '보편적(universal)' VPR 솔루션의 필요성을 제기한다. 보편적 VPR은 다음과 같은 세 가지 축에서 강인함을 갖추어야 한다 1:

- **Anywhere:** 도시, 실내, 야외, 항공, 수중, 지하 등 환경의 종류에 구애받지 않고 원활하게 작동해야 한다.
- **Anytime:** 낮과 밤의 변화, 계절의 전환, 일시적인 객체의 등장과 같은 시간적 변화에 강인해야 한다.
- **Anyview:** 카메라의 관점이나 시야각 변화에 불변하는 인식 성능을 유지해야 한다.

이러한 보편성을 달성하는 것은 새로운 환경에 로봇을 배치할 때마다 막대한 비용과 시간이 소요되는 재훈련(re-training)이나 미세조정(fine-tuning) 과정 없이 즉시 임무를 수행할 수 있도록 하는, 'out-of-the-box' 적용 가능성의 핵심이다.8



AnyLoc은 기존 VPR 연구의 패러다임을 근본적으로 전환한다. VPR 특화 데이터셋으로 모델을 훈련하는 대신, 기성품(off-the-shelf) 대규모 자기 지도 학습(self-supervised) 파운데이션 모델(foundation model)에서 추출한 범용 특징 표현(general-purpose feature representation)이 보편적 VPR을 위한 최적의 기질(substrate)이라는 가설을 제시하고 증명한다.1 이 접근법은 VPR 과업에 대한 어떠한 종류의 훈련이나 미세조정도 필요로 하지 않는다는 점에서 혁신적이다.8 DINOv2와 같은 파운데이션 모델은 특정 과업이 아닌, 방대하고 다양한 데이터로부터 세상에 대한 시각적 지식을 학습했기 때문에, 어떤 VPR 특화 데이터셋보다도 풍부하고 일반화 성능이 뛰어난 특징을 이미 내포하고 있다는 것이 핵심 통찰이다.1


AnyLoc은 공동 임베딩(joint-embedding) 방식의 DINO/DINOv2, 대조 학습(contrastive learning) 방식의 CLIP, 마스크 자동 인코딩(masked autoencoding) 방식의 MAE 등 여러 파운데이션 모델을 비교 분석했다.1 그 결과, DINOv2가 VPR에 가장 적합한 백본(backbone)으로 선정되었다. DINOv2는 LVD-142M이라는 거대한 데이터셋을 통해 자기 지도 학습 방식으로 훈련되어, 장소 인식에 필수적인 시점 및 외형 변화에 대한 강인함을 명시적인 VPR 훈련 없이도 이미 갖추고 있다.8 이는 객체의 의미론적 부분(semantic parts)과 구조를 이해하도록 유도하는 DINOv2의 학습 목표에서 비롯된 자연스러운 결과이다.


AnyLoc의 기술적 핵심은 파운데이션 모델을 사용하는 방식에 있다. 단순히 비전 트랜스포머(ViT)의 최종 글로벌 표현인 $$ 토큰을 사용하는 것만으로도 기존의 많은 지도 학습 기반 모델보다 나은 성능을 보이지만, 이는 최적이 아니다.16 AnyLoc의 진정한 혁신은 ViT 아키텍처의 중간 레이어(intermediate layer)에서 픽셀 단위(per-pixel) 혹은 패치 단위(patch-level)의 지역 특징(local feature)을 추출하는 데 있다.1 입력 이미지 $I$가 ViT를 통과하면 $N$개의 패치 토큰 $t_i$가 생성되는데 8, AnyLoc은 이 개별 토큰들을 장소 표현의 기본 단위로 활용한다.

특히, 연구진은 ViT의 어텐션 블록 내 여러 "패싯(facet)"(key, query, value, token)을 분석하여, DINOv2-ViT-G 모델의 후반부 레이어(L31)에서 추출한 $value$ 패싯이 VPR에 가장 변별력 높은 특징을 제공함을 실험적으로 밝혔다.16 이는 $value$ 패싯이 배경과 핵심 객체 간의 대조(contrast)가 가장 높아, 이미지 내 방해 요소를 효과적으로 걸러내는 데 유리하기 때문이다.1 이는 VPR의 특징을 '학습'하는 문제에서, 사전 학습된 모델의 내재된 지식 중 가장 유용한 부분을 '선택하고 집합'하는 문제로 재정의한 것이다.


추출된 다수의 지역 특징들을 하나의 전역적인 장소 표현 벡터(global descriptor)로 압축하기 위해, AnyLoc은 비지도 집합(unsupervised aggregation) 기법을 사용한다. 이 과정은 VPR 특화 레이블 없이 진행되므로 모델의 보편성을 유지하는 데 결정적이다.

- **VLAD (Vector of Locally Aggregated Descriptors):** VLAD는 이미지에서 추출된 $N$개의 지역 특징 벡터 집합 ${x_i}$와, 사전에 K-means 클러스터링 등으로 계산된 $K$개의 클러스터 중심(어휘집) ${c_k}$를 사용하여 장소 기술자를 생성한다.

  1. 각 지역 특징 $x_i$는 가장 가까운 클러스터 중심 $c_{NN(x_i)}$에 할당된다.
  2. 각 클러스터 $k$에 대해, 해당 클러스터에 할당된 모든 특징들과 중심 벡터 간의 차이(잔차, residual)의 합 $v_k$를 계산한다.

  $$
  v_k = \sum_{i=1, NN(x_i)=k}^{N} (x_i - c_k)
  $$
  
  1. 모든 $v_k$ 벡터를 순서대로 이어 붙여 $K x D$ 차원의 거대한 VLAD 벡터 $V =^T$를 생성한다. 여기서 $D$는 지역 특징의 차원이다.
  2. 마지막으로 이 벡터를 power-normalization과 L2-normalization을 통해 정규화하여 최종 기술자를 얻는다.
  
- **GeM (Generalized Mean Pooling):** GeM은 어휘집이 필요 없는 더 간단한 집합 방식이다. $N$개의 지역 특징 벡터 $x_i$에 대해, 각 차원별 일반화 평균을 계산한다.

  $$
  f_{GeM} = \left( \frac{1}{|\mathcal{X}|} \sum_{x \in \mathcal{X}} x^p \right)^{\frac{1}{p}}
  $$
  여기서 $p$는 학습 가능한 파라미터이며, $p=1$이면 평균 풀링, $p$가 무한대에 가까워지면 최대 풀링과 유사하게 동작한다.

실험 결과, 경성 할당(hard-assignment) 기반의 VLAD가 가장 우수한 성능을 보였으며, GeM은 성능과 저장 공간 효율성 사이에서 좋은 절충안을 제공하는 것으로 나타났다.16


AnyLoc은 VLAD 어휘집 구성 방식에서도 독창적인 접근법을 제시한다. 모든 데이터셋에 적용되는 단일 '전역 어휘집'이나 각 데이터셋마다 개별적으로 만드는 '맵 특화 어휘집' 대신, '도메인 특화 어휘집(domain-specific vocabulary)'이라는 개념을 도입했다.9

그 과정은 다음과 같다:

1. 여러 데이터셋의 모든 이미지에 대해 GeM 풀링을 적용하여 전역 기술자를 추출한다.
2. PCA 등을 이용해 이 고차원 기술자들을 저차원으로 투영한다.
3. 투영된 공간에서 유사한 환경(예: 모든 항공 데이터셋, 모든 실내 데이터셋)의 이미지들이 자연스럽게 군집을 이루는 '도메인'을 발견한다.11
4. 각 도메일에 속한 데이터셋들의 지역 특징들만 모아 해당 도메인을 위한 별도의 VLAD 어휘집을 구축한다.

이 전략은 전역 어휘집이나 맵 특화 어휘집보다 더 나은 성능을 보이며, 약 6%의 추가적인 성능 향상을 가져왔다.1 이는 완전한 비지도 방식의 보편성을 유지하면서도, 환경 유형에 대한 약한 형태의 사전 정보를 활용하여 성능을 극대화하는 영리한 절충안이다.



AnyLoc의 보편성을 검증하기 위해, 연구진은 극도로 다양한 12개의 공개 데이터셋을 활용했다. 이 데이터셋들은 환경의 종류, 시간 변화, 시점 변화 등 VPR의 주요 난제들을 포괄하도록 신중하게 선택되었다.

- **정형 환경:** Baidu Mall, Gardens Point, 17 Places (실내), Pittsburgh-30k, St Lucia (도시 야외).9
- **비정형 환경:** Hawkins, Laurel Caverns (지하 동굴), Nardo-Air, VP-Air (항공), Mid-Atlantic Ridge (수중).9
- **시간/시점 변화:** Oxford RobotCar (주야간 및 장기적 변화), Nordland (계절 변화), ALTO (시점 변화).4

성능 평가는 주로 Recall@N 지표를 사용했다. 이는 질의 이미지에 대해 상위 N개의 검색 결과 중 하나 이상이 실제 정답 위치(ground truth)와 일치하는 경우의 비율을 의미하며, 특히 N=1일 때의 성능(R@1)이 가장 중요한 척도로 간주된다.


AnyLoc은 기존 SOTA VPR 모델들과의 비교에서 압도적인 성능을 입증하며, 특정 데이터셋에서는 최대 4배 높은 성능을 달성했다고 보고했다.8 특히 기존 모델들이 취약했던 비정형 환경과 극심한 외형 변화 환경에서 그 격차는 더욱 두드러졌다.

**표 1: 주요 VPR 벤치마크에서의 SOTA 모델 성능 비교 (Recall@1, %)**

| 데이터셋 유형        | 데이터셋       | MixVPR 9 | CosPlace 9 | DINOv2-CLS | AnyLoc-VLAD-DINOv2 |
| -------------------- | -------------- | -------- | ---------- | ---------- | ------------------ |
| **정형 (도시)**      | St Lucia       | 64.4     | 41.6       | 70.1       | **82.3**           |
| **비정형 (지하)**    | Laurel Caverns | 낮음     | 낮음       | 중간       | **매우 높음**      |
| **비정형 (항공)**    | VP-Air         | 낮음     | 낮음       | 중간       | **높음**           |
| **시간 변화 (계절)** | Nordland       | 낮음     | 낮음       | 중간       | **매우 높음**      |

*주: 표의 수치는 논문의 정량적 주장과 경향성을 바탕으로 재구성되었음. DINOv2-CLS 및 AnyLoc-VLAD의 수치는 상대적 성능 우위를 나타내기 위한 예시임.*

이 결과는 VPR의 진정한 난제가 기존 벤치마크에서의 소수점 단위 성능 경쟁이 아니라, 미지의 비정형 환경에 대한 일반화 성능 확보에 있었음을 명확히 보여준다. AnyLoc은 도시 환경 데이터로만 훈련된 CosPlace와 같은 모델에 비해 도시 환경에서는 그 우위가 상대적으로 적었지만, 비정형 환경에서는 성능 격차를 압도적으로 벌리며 일반화 능력의 차이를 증명했다.


AnyLoc의 설계가 최적임을 보이기 위해 연구진은 다양한 요소에 대한 어블레이션 연구(ablation study)를 수행했다.9 이 연구는 AnyLoc의 높은 성능이 단순히 강력한 백본 모델 때문이 아니라, 각 구성 요소의 신중한 선택과 조합의 결과임을 입증한다.

**표 2: AnyLoc 설계 요소에 대한 어블레이션 연구 결과 (Recall@1, %)**

| 분석 요소           | 구성                   | 성능 (R@1) | 결론                                                         |
| ------------------- | ---------------------- | ---------- | ------------------------------------------------------------ |
| **파운데이션 모델** | AnyLoc-VLAD-DINO       | 낮음       | \multirow{2}{*}{DINOv2가 DINO보다 우수함}                    |
|                     | AnyLoc-VLAD-DINOv2     | **높음**   |                                                              |
| **집합 방식**       | DINOv2-CLS             | 낮음       | \multirow{3}{*}{지역 특징 집합이 필수적이며, VLAD가 가장 효과적임} |
|                     | AnyLoc-GeM-DINOv2      | 중간       |                                                              |
|                     | AnyLoc-VLAD-DINOv2     | **높음**   |                                                              |
| **VLAD 어휘집**     | 전역(Global) 어휘집    | 낮음       | \multirow{3}{*}{도메인 특화 어휘집이 성능을 추가로 향상시킴} |
|                     | 데이터셋(Dataset) 특화 | 중간       |                                                              |
|                     | 도메인(Domain) 특화    | **높음**   |                                                              |

*주: 표의 수치는 논문에서 보고된 성능 경향을 나타냄.*

이 분석을 통해 DINOv2 모델의 우수성, $$ 토큰을 넘어선 지역 특징 집합의 중요성, 그리고 도메인 특화 어휘집의 효과가 정량적으로 증명되었다.


AnyLoc 프로젝트 페이지에 공개된 정성적 검색 결과 시각화 자료는 수치적 성능을 넘어 직관적인 이해를 돕는다.13 항공 이미지 데이터셋인 VPAir나 지하 동굴 데이터셋인 Laurel Caverns에서, AnyLoc은 극심한 시점 차이와 조명 변화에도 불구하고 정확한 장소를 검색해내는 놀라운 능력을 보여준다.13 또한 DINO와 DINOv2의 어휘집 시각화 비교를 통해, DINOv2의 특징 공간이 의미론적으로 더 잘 구조화되어 있고 변별력이 높음을 확인할 수 있다.13



AnyLoc의 가장 큰 학술적 기여는 VPR 분야에 놀랍도록 간단하면서도 강력한 새로운 기준점(baseline)을 제시했다는 점이다.1 이 연구는 VPR의 일반화 문제를 해결하기 위한 전략이 특화된 모델을 훈련하는 것이 아니라, 거대 데이터로 사전 학습된 범용 모델의 지식을 효과적으로 활용하는 것임을 보여주며 연구의 방향을 전환시켰다. 실제로 AnyLoc 발표 이후, VISTA, CriSALAD 등 많은 후속 연구들이 AnyLoc을 성능 비교를 위한 표준 벤치마크로 채택하고 있다.4

더 나아가, AnyLoc의 어블레이션 연구는 VPR을 일종의 탐침(probe)으로 사용하여 파운데이션 모델의 내부 작동 방식을 탐구하는 방법론적 가능성을 열었다. 어떤 내부 특징(레이어, 패싯)이 장소 인지에 가장 효과적인지를 분석함으로써, ViT와 같은 거대 모델이 시각적 세계를 어떻게 표상하고 있는가에 대한 통찰을 제공한다.


AnyLoc의 압도적인 성능에도 불구하고, 실용적인 배포에는 명백한 한계가 존재한다. 가장 치명적인 단점은 VLAD 기술자로 생성된 맵의 크기이다.8 VLAD 기술자의 차원은 

$K x D$로, DINOv2-ViT-G의 특징 차원 $D$가 1536에 달해 매우 크다. 이로 인해 AnyLoc의 맵 크기는 VISTA와 같은 경쟁 방법에 비해 1200배 이상 커질 수 있다.8 이렇게 거대한 맵은 제한된 메모리와 대역폭을 가진 로봇 플랫폼에 저장하거나 로봇 간에 통신하기에 비현실적이며, 이는 실제 현장 적용에 큰 장벽이 된다.8

또한, 미세조정되지 않은 DINOv2 백본은 장소 인식과 무관한 동적 객체(보행자, 차량 등)의 특징까지 포착할 수 있어, 경우에 따라 매칭 성능을 저해하는 요인으로 작용할 수 있다.8


AnyLoc이 제시한 높은 성능과 명확한 한계는 VPR 연구 커뮤니티에 새로운 연구 방향을 제시했다. AnyLoc은 성능의 상한선을 크게 끌어올리면서, '제한된 자원 하에서 어떻게 그 성능을 유지할 것인가'라는 새로운 문제를 정의했다. 후속 연구들은 크게 두 가지 방향으로 분화되었다.

첫 번째는 AnyLoc의 성능을 유지하면서 기술자의 크기와 계산 비용을 획기적으로 줄이는 연구이다.

- **VISTA:** 객체 기반의 희소(sparse) 맵 표현을 사용하여 맵 크기를 기존의 0.6% 수준으로 줄여 AnyLoc의 약점을 정면으로 해결하고자 한다.8
- **TeTRA-VPR:** 트랜스포머 모델을 삼진화(ternary quantization)하여 메모리가 제한된 플랫폼을 위한 경량화 모델을 개발한다.6

두 번째는 AnyLoc의 파운데이션 모델 접근법을 계승하고 발전시키는 연구이다.

- **SegVLAD (Revisit Anything):** 이미지 전체가 아닌, 의미론적으로 분할된 '객체'나 '영역'에 VLAD를 적용하여 부분적인 겹침(partial overlap) 상황에서의 인식 성능을 높인다. 이 연구는 DINOv2-AnyLoc을 주요 백본으로 활용할 수 있음을 명시한다.2
- **CriSALAD:** 이미지 간 상호 정보를 활용하여 파운데이션 모델의 특징 추출 강인성을 더욱 향상시킨다.8

향후 VPR 연구는 AnyLoc의 조밀한(dense) 특징과 VISTA의 희소한 객체 표현을 결합한 하이브리드 모델, 검색 후보군을 재평가하기 위한 멀티모달 대형 언어 모델(LLM)의 활용 14, 그리고 개인정보를 보호하며 분산된 데이터를 학습하는 연합 학습(federated learning) 기법의 도입 18 등으로 발전할 것으로 전망된다. AnyLoc은 VPR 연구의 새로운 시대를 연 기념비적인 연구로, 그 영향력은 앞으로도 계속될 것이다.


1. Towards Universal Visual Place Recognition - AnyLoc, accessed August 22, 2025, https://anyloc.github.io/assets/AnyLoc.pdf
2. Revisit Anything: Visual Place Recognition via Image Segment Retrieval - arXiv, accessed August 22, 2025, https://arxiv.org/html/2409.18049v1
3. Revisit Anything: Visual Place Recognition via Image Segment Retrieval - European Computer Vision Association, accessed August 22, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/08592.pdf
4. Visual place recognition for aerial imagery: A survey - arXiv, accessed August 22, 2025, https://arxiv.org/html/2406.00885v1
5. NYU-VPR: Long-Term Visual Place Recognition Benchmark with View Direction and Data Anonymization Influences, accessed August 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9394449/
6. Visual Place Recognition | Papers With Code, accessed August 22, 2025, https://paperswithcode.com/task/visual-place-recognition/codeless
7. Visual Place Recognition in Changing Environments - Niko Sünderhauf, accessed August 22, 2025, https://nikosuenderhauf.github.io/projects/placerecognition/
8. AnyLoc : Towards Universal Visual Place Recognition | Request PDF - ResearchGate, accessed August 22, 2025, https://www.researchgate.net/publication/376574112_AnyLoc_Towards_Universal_Visual_Place_Recognition
9. AnyLoc: Towards Universal Visual Place Recognition https://anyloc.github.io/ - arXiv, accessed August 22, 2025, https://arxiv.org/html/2308.00688
10. [2308.00688] AnyLoc: Towards Universal Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/abs/2308.00688
11. AnyLoc: Towards Universal Visual Place Recognition - ResearchGate, accessed August 22, 2025, https://www.researchgate.net/publication/372827893_AnyLoc_Towards_Universal_Visual_Place_Recognition
12. AnyLoc: Towards Universal Visual Place Recognition - CMU Robotics Institute, accessed August 22, 2025, https://www.ri.cmu.edu/publications/anyloc-towards-universal-visual-place-recognition/
13. AnyLoc: Towards Universal Visual Place Recognition, accessed August 22, 2025, https://anyloc.github.io/
14. [PDF] AnyLoc: Towards Universal Visual Place Recognition - Semantic Scholar, accessed August 22, 2025, https://www.semanticscholar.org/paper/AnyLoc%3A-Towards-Universal-Visual-Place-Recognition-Keetha-Mishra/7777ffb5f2f1d52b7080725332035b81af2994c9
15. Meet AnyLoc: The Latest Universal Method For Visual Place Recognition (VPR), accessed August 22, 2025, https://www.marktechpost.com/2023/08/08/meet-anyloc-the-latest-universal-method-for-visual-place-recognition-vpr/
16. Paper page - AnyLoc: Towards Universal Visual Place Recognition - Hugging Face, accessed August 22, 2025, https://huggingface.co/papers/2308.00688
17. VISTA: Monocular Segmentation-Based Mapping for Appearance and View-Invariant Global Localization - arXiv, accessed August 22, 2025, https://arxiv.org/html/2507.11653v1
18. Collaborative Visual Place Recognition through Federated Learning - CVF Open Access, accessed August 22, 2025, https://openaccess.thecvf.com/content/CVPR2024W/FedVision-2024/papers/Dutto_Collaborative_Visual_Place_Recognition_through_Federated_Learning_CVPRW_2024_paper.pdf
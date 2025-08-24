# SAFA 교차 시점 지리적 위치 추정
[딥러닝 기반 장소 인식 (Deep Learning-based Place Recognition)](./index.md)













본 보고서는 사용자가 질의한 "SAFA: Semantic-Aware Feature Aggregation for Geo-Localization (CVPR 2020)"에 대한 심층 분석을 제공합니다. 그러나 연구 자료 1를 종합적으로 검토한 결과, 해당 분야에서 'SAFA'라는 약어로 널리 알려진 기념비적인 논문은 Yujiao Shi 등이 발표한 **"Spatial-Aware Feature Aggregation for Image based Cross-View Geo-Localization"**이며, 이는 

**NeurIPS 2019** 학회에 게재되었습니다. 사용자의 질의에 포함된 'Semantic-Aware'와 'CVPR 2020'은 해당 분야에서 흔히 발생하는 혼동으로 판단됩니다. 이는 SAFA가 공간적(Spatial) 관계에 집중함으로써 갖는 강점과 동시에 의미론적(Semantic) 정보 부재라는 한계를 내포하고 있음을 역설적으로 시사하며, 본 보고서 전반에 걸쳐 중요한 분석의 축이 될 것입니다. 따라서 본 보고서는 이 NeurIPS 2019 논문을 중심으로, 사용자의 원래 의도에 가장 부합하는 심층적이고 전문적인 분석을 제공하는 것을 목표로 합니다.






교차 시점 지리적 위치 추정(CVGL)은 지상에서 촬영된 쿼리 이미지(예: 구글 스트리트뷰 파노라마)를 GPS 정보가 태깅된 방대한 항공 또는 위성 이미지 데이터베이스와 매칭하여 이미지의 정확한 지리적 위치를 특정하는 기술입니다.3 이 기술의 중요성은 현대 사회의 핵심 기술들과 깊이 연관되어 있습니다. 자율주행 차량이 도심 협곡이나 터널 입구에서 GPS 신호를 잃었을 때, 로봇이 복잡한 실내외 환경을 원활하게 탐색해야 할 때, 또는 증강현실(AR) 애플리케이션이 가상 객체를 현실 세계에 정밀하게 중첩시켜야 할 때, CVGL은 기존 위치 측정 시스템의 한계를 보완하거나 대체할 수 있는 강력한 대안을 제시합니다.3 이처럼 CVGL은 단순한 이미지 검색 문제를 넘어, 차세대 기술들의 안정성과 신뢰성을 담보하는 기반 기술로서 그 가치가 매우 높습니다.






SAFA는 CVGL 문제에 대한 접근 방식에 있어 **패러다임의 전환**을 이룬 기념비적인 연구로 평가받습니다. SAFA 이전의 연구들은 지상과 하늘이라는 두 극단적으로 다른 시점 사이의 기하학적 변환 문제와 객체의 외형 및 의미를 학습하는 특징 추출 문제를 하나의 거대한 딥러닝 모델에 맡겨, 비효율적이고 '맹목적인(blind and forceful)' 학습을 강요하는 경향이 있었습니다.2 이는 마치 외국어 번역과 시 짓기를 동시에 배우라고 요구하는 것과 같아 네트워크에 과도한 학습 부담을 주었습니다.

SAFA의 진정한 혁신은 특정 네트워크 모듈의 발명이 아니라, 문제 해결에 대한 철학적 전환에 있습니다. SAFA는 이 두 가지 근본적으로 다른 문제를 **명시적으로 분리(decouple)**하는 혁신적인 2단계 접근법을 제안했습니다.3 먼저, 고전적인 컴퓨터 비전 기법인 극좌표 변환을 통해 기하학적 구조를 대략적으로 맞추고, 그 다음에 잘 정렬된 도메인 위에서 딥러닝 네트워크가 특징 대응 관계 학습에만 집중하도록 한 것입니다. 이 접근법은 딥러닝의 강력한 기능 근사 능력과 고전적 기하학의 명시적 지식을 지능적으로 결합한 사례로, '언제 딥러닝을 사용하지 않을 것인가'에 대한 깊은 통찰을 보여줍니다. 본 보고서는 SAFA의 이러한 핵심 철학을 시작으로, 그 방법론을 정밀하게 해부하고, 성능을 비판적으로 평가하며, 후속 연구에 미친 지대한 영향과 내재적 한계까지 다각도로 조명할 것입니다.

------











교차 시점 지리적 위치 추정의 핵심적인 어려움은 지상 이미지와 항공(위성) 이미지라는 두 도메인 간에 존재하는 극심한 불일치에서 비롯됩니다. 이 간극은 단순히 이미지가 달라 보이는 수준을 넘어, 기하학적, 외형적, 시간적 차원에서 복합적으로 나타납니다.

- **시점 차이 (Perspective Differences):** 가장 근본적인 문제입니다. 지상 카메라는 지면과 거의 평행한 시점에서 360도 파노라마 뷰를 포착하는 반면, 위성 이미지는 지면을 수직으로 내려다보는 조감도(bird's-eye view)를 제공합니다.3 이로 인해 건물의 측면이 보이는 지상 뷰와 지붕만 보이는 항공 뷰 사이에는 직접적인 픽셀 단위의 비교가 거의 불가능합니다. 동일한 도로라 할지라도 지상에서는 소실점을 향해 뻗어 나가는 형태로 보이지만, 항공에서는 일정한 폭을 가진 선으로 보입니다. 이러한 극단적인 시점 변화는 전통적인 특징점 매칭 알고리즘을 무력화시키는 주된 원인입니다.
- **스케일 변화 (Scale Variations):** 두 시점에서 포착되는 객체들의 상대적 크기는 극심하게 다릅니다.7 지상 이미지에서는 가까운 나무가 건물보다 크게 보일 수 있지만, 항공 이미지에서는 건물이 나무보다 훨씬 큰 면적을 차지합니다. 이러한 스케일의 비일관성은 이미지 전체의 특징을 요약하는 전역 디스크립터(global descriptor)를 생성하는 데 큰 어려움을 안겨줍니다.
- **외형 불일치 (Appearance Discrepancies):** 두 이미지가 촬영된 시점의 차이로 인해 발생하는 외형의 불일치는 매우 다양합니다.
  - **조명 및 계절 변화:** 하루 중 시간대에 따른 그림자의 방향과 길이 변화, 계절에 따른 초목의 색상 변화(여름의 녹색 잎과 겨울의 앙상한 가지) 등은 동일한 장소의 외형을 완전히 다르게 만듭니다.7
  - **가려짐 및 동적 객체 (Occlusion and Dynamic Objects):** 지상 이미지에서는 다른 건물, 나무, 버스, 트럭과 같은 동적 객체에 의해 중요한 랜드마크가 가려질 수 있습니다. 항공 이미지에서는 구름이나 그림자에 의해 지표면이 가려질 수 있습니다. 이러한 가려짐은 매칭에 필요한 핵심 정보를 소실시켜 위치 추정의 실패로 이어질 수 있습니다.7






이러한 근본적인 도전을 해결하기 위해 SAFA 이전에도 다양한 시도들이 있었습니다. 이들의 역사는 컴퓨터 비전 기술의 발전사와 궤를 같이하며, 점차 정교화되었지만 명확한 한계를 드러냈습니다.

- **수동 특징 기반 방법 (Hand-crafted Feature-based Methods):** 초기 연구들은 SIFT (Scale-Invariant Feature Transform), HOG (Histogram of Oriented Gradients), GIST와 같은 수동으로 설계된 특징 추출 알고리즘에 의존했습니다.3 이 방법들은 특정 변환(크기, 회전)에 대해서는 어느 정도 강건성을 보였지만, CVGL의 극심한 시점 변화 앞에서는 매우 취약했습니다. 특징 기술자가 직접 알고리즘을 설계해야 했기 때문에 새로운 환경이나 데이터에 대한 적응력이 현저히 떨어지는 문제도 있었습니다.7 결국, 이 접근법은 성능의 병목 현상을 초래하며 한계에 부딪혔습니다.
- **초기 딥러닝 기반 방법 (Early Deep Learning-based Methods):** 딥러닝의 부상과 함께 CVGL 문제 역시 새로운 국면을 맞이했습니다. 연구자들은 딥러닝의 강력한 특징 학습 능력을 활용하여 이 문제를 해결하고자 했습니다.
  - **이미지 검색 문제로의 접근:** 대부분의 초기 딥러닝 모델들은 CVGL을 표준적인 이미지 검색(image retrieval) 문제로 정의했습니다.3 즉, 지상 쿼리 이미지와 가장 유사한 항공 이미지를 데이터베이스에서 찾아내는 것을 목표로 삼았습니다.
  - **샴 네트워크와 거리 학습:** 이를 위해 두 개의 동일한(또는 유사한) 네트워크 브랜치가 특징을 각각 추출하고, 그 특징 벡터 간의 거리를 학습하는 샴 네트워크(Siamese Network) 구조가 널리 사용되었습니다.7 Triplet Loss와 같은 손실 함수를 사용하여, 같은 장소를 찍은 이미지 쌍(positive pair)의 특징 벡터는 가깝게, 다른 장소를 찍은 이미지 쌍(negative pair)의 특징 벡터는 멀게 만드는 거리 학습(metric learning)을 수행했습니다.
  - **CVM-Net의 등장:** 이 시기의 대표적인 모델은 **CVM-Net (Cross-View Matching Network)**입니다. CVM-Net은 CNN(Convolutional Neural Network)을 통해 이미지의 지역 특징(local features)을 추출하고, 이를 당시 이미지 검색 분야에서 최첨단 기술이었던 NetVLAD 레이어를 이용해 하나의 강력한 전역 디스크립터(global descriptor)로 집계하는 방식을 제안했습니다.12






딥러닝이라는 강력한 도구를 사용했음에도 불구하고, CVM-Net과 같은 초기 모델들의 성능은 만족스럽지 못했습니다. 그 이유는 딥러닝의 강점을 잘못 적용했기 때문입니다.

- **문제의 본질 간과:** 이 모델들은 CVGL을 단순히 동일 도메인 내 이미지 검색 문제의 연장선으로 취급했습니다.3 그러나 CVGL의 핵심은 극심한 도메인 차이, 특히 

  **구조화된 기하학적 불일치**에 있습니다. 이들은 이러한 기하학적 및 외형적 차이를 명시적으로 고려하지 않고, 오직 방대한 데이터와 네트워크의 학습 능력에만 의존하는 '맹목적이고 강력한(blind and forceful)' 학습 절차를 따랐습니다.2

- **학습 부담의 과중:** 결과적으로, 네트워크는 두 가지 매우 이질적이고 어려운 문제를 동시에 해결하도록 강요받았습니다. 하나는 항공 뷰를 지상 뷰의 파노라마 형태로 변환하는 고도의 **기하학적 대응 관계** 학습이고, 다른 하나는 조명과 계절 변화에도 강건한 **특징 대응 관계** 학습입니다.3 강력한 함수 근사기인 딥러닝 모델에게 너무나도 복잡한 함수를 근사하도록 요구한 셈입니다. 이는 네트워크에 과도한 학습 부담을 주어 최적의 해로 수렴하는 것을 방해했고, 결국 낮은 재현율(recall rate)이라는 실망스러운 성능으로 귀결되었습니다.3

수동 특징의 시대가 '특징 표현력'의 한계라는 병목을 겪었다면, 초기 딥러닝 시대는 '학습 패러다임'의 한계라는 새로운 병목에 직면한 것입니다. 특징 자체는 강력해졌지만, 그 특징을 학습시키는 방식이 문제의 본질을 제대로 반영하지 못했습니다. 바로 이 지점에서 SAFA는 새로운 해법을 제시하며 등장하게 됩니다.

------






SAFA의 아키텍처는 CVGL 문제의 복잡성을 정면으로 돌파하기보다, 이를 영리하게 분해하여 해결하는 철학에 기반합니다. 이는 기하학적 구조 맞추기와 내용 기반의 특징 매칭이라는 두 개의 연주자가 각자의 파트를 완벽하게 연주한 후 조화로운 앙상블을 이루는 '이중주'에 비유할 수 있습니다.






SAFA의 가장 중요한 지적 기여는 CVGL이라는 하나의 거대한 문제를 두 개의 더 작고 다루기 쉬운 하위 문제로 명시적으로 분리(decouple)한 것입니다.3

1. **기하학적 정렬 (Geometric Alignment):** 먼저, 딥러닝 모델 외부에서 고전적인 기하학 변환을 통해 두 이미지 도메인의 구조를 대략적으로 맞춥니다.
2. **특징 대응 학습 (Feature Correspondence Learning):** 그 다음, 구조적으로 유사해진 이미지들을 입력으로 받아 딥러닝 모델이 내용 기반의 특징 매칭에만 집중하도록 합니다.

이러한 분리 전략은 네트워크가 극심한 시점 차이라는 복잡한 기하학적 변환을 암묵적으로 학습해야 하는 엄청난 부담에서 벗어나게 해줍니다. 대신 네트워크는 '단순한 특징 매핑 작업(simple feature mapping task)'에만 집중할 수 있게 되어, 학습 과정이 훨씬 안정되고 효율적으로 진행되며, 최종적으로는 더 높은 성능으로 이어집니다.3






SAFA의 첫 번째 단계는 딥러닝이 아닌, 잘 알려진 기하학적 원리를 활용하는 것입니다.

- **관찰 기반의 접근:** 이 단계는 "항공 이미지에서 동일한 방위각(azimuth) 방향을 따라 배열된 픽셀들은, 지상 파노라마 이미지에서는 대략적으로 하나의 수직(vertical) 열에 해당한다"는 통찰력 있는 관찰에서 시작됩니다.1 즉, 항공 이미지의 중심에서 특정 각도로 뻗어 나가는 선 위의 점들은 지상 파노라마의 특정 수직선에 매핑된다는 기하학적 사전 지식(prior knowledge)입니다.
- **작동 원리:** 이 원리를 구현하기 위해, SAFA는 항공 이미지에 일반적인 **극좌표 변환(polar transform)**을 적용합니다. 이 변환은 항공 이미지의 중심을 원점으로 하여 각 픽셀의 데카르트 좌표 $(x, y)$를 극좌표 $(r, \theta)$(반경, 각도)로 변환한 후, 이를 다시 2D 이미지 평면에 펼쳐놓는 과정입니다. 구체적으로, 변환된 이미지의 가로축은 각도($\theta$)에, 세로축은 반경(r)에 대응됩니다. 그 결과, 원형의 항공 이미지는 직사각형의 '펼쳐진' 이미지로 변형되며, 이는 360도 지상 파노라마 이미지와 유사한 구조적 레이아웃을 갖게 됩니다.3
- **효과와 한계:**
  - **효과:** 이 단순한 전처리 단계는 두 이미지 도메인 간의 극심한 기하학적 간극을 상당 부분 해소합니다. 변환 후 항공 이미지의 도로는 지상 파노라마의 도로와 유사한 수평선 형태로 나타나고, 건물들의 상대적 좌우 배치도 비슷해집니다. 이는 후속 딥러닝 네트워크가 특징을 학습하기 훨씬 용이한 환경을 조성합니다.3
  - **한계:** 그러나 극좌표 변환은 완벽한 해결책이 아닙니다. 이 변환은 순수한 기하학적 연산이므로, **장면의 내용(scene content)을 전혀 고려하지 않습니다(agnostic)**.1 이로 인해 이미지 중심에서 멀리 떨어진 객체는 심하게 늘어나거나 왜곡되는 등 심각한 **시각적 왜곡(visual distortion)**이 발생합니다.3 이처럼 불완전하게 정렬되고 왜곡된 특징을 보정하는 것이 바로 2단계의 역할입니다.






1단계에서 발생한 문제를 해결하고, 남아있는 미세한 불일치를 해소하기 위해 SAFA는 독창적인 어텐션 기반 특징 집계 모듈을 제안합니다.

- **필요성:** 극좌표 변환으로 인해 발생한 왜곡을 완화하고, 두 이미지에서 의미적으로는 동일하지만 외형적으로는 여전히 다른 특징들을 임베딩 공간상에서 더욱 가깝게 만들기 위해, 특징의 '공간적 위치'를 고려하는 메커니즘이 필요합니다.3
- **SPE (Spatial-aware Position Embedding) 모듈:**
  - SAFA의 핵심 모듈인 SPE는 이 역할을 수행합니다. 이 모듈은 입력된 특징 맵(feature map)에서 **어느 위치의 특징이 더 중요한지를 학습**하여, 위치별로 다른 가중치를 부여(re-weight)한 후 이를 합산하여 최종적인 전역 디스크립터를 생성합니다.3
  - 이 과정은 단순히 중요한 객체의 특징만 뽑아내는 것을 넘어, **객체들 간의 상대적인 위치, 즉 공간적 레이아웃 정보(layout information)를 디스크립터에 인코딩**하는 효과를 가집니다.3 예를 들어, '도로 위에 자동차가 있고 그 옆에 나무가 있다'는 공간적 구성을 특징 벡터에 담아내어 판별력을 극대화합니다.
- **다중 SPE 모듈을 통한 특징 집계 (Feature Aggregation via Multiple SPEs):**
  - 단일 어텐션 맵은 특정 종류의 공간적 패턴에만 집중할 수 있습니다. 특징 표현의 **강건성(robustness)**을 확보하기 위해, SAFA는 동일한 구조를 가지지만 서로 다른 가중치로 초기화된 여러 개의 독립적인 SPE 모듈(논문에서는 M=8 사용)을 병렬로 사용합니다.3
  - 각 SPE 모듈은 학습 과정에서 서로 다른 공간적 레이아웃에 집중하도록 자연스럽게 분화됩니다. 예를 들어, 어떤 모듈은 도로망과 같은 거시적인 구조에, 다른 모듈은 건물들의 밀집도와 같은 미시적인 패턴에, 또 다른 모듈은 녹지 공간의 분포에 집중할 수 있습니다.
  - 이렇게 각기 다른 관점에서 공간 정보를 포착한 여러 개의 임베딩된 특징 벡터들을 최종적으로 **연결(concatenate)**하여 하나의 길고 풍부한 이미지 디스크립터를 형성합니다. 이는 마치 여러 전문가가 각자의 관점에서 장면을 분석한 뒤 종합 보고서를 작성하는 것과 같아, 매우 강건하고 판별력 높은 특징 표현을 가능하게 합니다.3






SAFA의 전체 파이프라인은 이러한 핵심 아이디어들을 통합하여 구성됩니다.

- **샴 네트워크 (Siamese Network):** 두 개의 브랜치를 가진 샴 네트워크 구조를 기반으로 합니다. 하나는 지상 파노라마 이미지를, 다른 하나는 극좌표 변환된 항공 이미지를 입력으로 받습니다. 각 브랜치는 VGG16의 일부를 백본(backbone) 네트워크로 사용하여 초기 특징 맵을 추출합니다.3
- **SAFA 모듈 적용:** 각 브랜치에서 추출된 특징 맵은 SAFA 모듈(다중 SPE 모듈 포함)을 통과하여 최종적인 전역 디스크립터로 집계됩니다.
- **Triplet Loss 기반 학습:** 두 브랜치에서 나온 최종 디스크립터 쌍을 사용하여 거리 학습을 수행합니다. 구체적으로, 가중치가 적용된 소프트 마진 랭킹 손실(weighted soft-margin ranking loss), 즉 Triplet Loss를 사용하여, 매칭되는 이미지 쌍(positive pair)의 디스크립터 간 거리는 최소화하고, 매칭되지 않는 이미지 쌍(negative pair)의 디스크립터 간 거리는 특정 마진 이상으로 최대화하도록 네트워크를 학습시킵니다.3

이러한 구조는 1단계의 기하학적 정렬과 2단계의 공간 인식 특징 집계라는 두 가지 핵심 전략이 시너지를 내도록 설계되었습니다. 극좌표 변환은 SAFA 모듈이 처리하기 좋은 입력을 만들어주고, SAFA 모듈은 극좌표 변환의 단점을 보완해 줍니다. 이 영리한 협력 관계가 바로 SAFA의 경이로운 성능의 비결입니다.

------






SAFA의 혁신적인 아키텍처는 두 개의 표준 벤치마크 데이터셋에 대한 광범위한 실험을 통해 그 효과를 입증했습니다. 이 장에서는 SAFA가 실험을 수행한 환경과 그 결과를 상세히 분석하여, 제안된 방법론의 실질적인 기여도를 평가합니다.






SAFA의 성능은 당시 CVGL 분야에서 표준으로 사용되던 두 개의 대규모 데이터셋에서 측정되었습니다. 이 데이터셋들의 특성을 이해하는 것은 SAFA의 성능과 한계를 정확히 파악하는 데 매우 중요합니다.

| 특징 구분                     | CVUSA (Cross-View USA)                                       | CVACT (Cross-View ACT)                                       |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **수집 지역**                 | 미국 전역의 다양한 도시 및 농촌 지역                         | 호주 캔버라(ACT) 지역                                        |
| **이미지 쌍 구성**            | 지상 파노라마 이미지와 위성 이미지                           | 지상 파노라마 이미지와 위성 이미지                           |
| **데이터 규모 (일반적 사용)** | 훈련용 약 35,532 쌍, 검증용 약 8,884 쌍                      | 훈련용 86,469 쌍, 검증용 21,249 쌍, 테스트용 92,802 쌍       |
| **지상 이미지**               | 360도 파노라마 (Panoramic)                                   | 360도 파노라마 (Panoramic)                                   |
| **위성 이미지 크기**          | 750x750 픽셀                                                 | 1200x1200 픽셀                                               |
| **핵심 가정 및 특징**         | - 지상 이미지가 위성 이미지의 중앙에 완벽히 정렬됨- 다양한 환경(도심, 교외)을 포함 | - 훈련/검증/테스트 세트가 명확히 분리됨- CVUSA와 마찬가지로 중앙 정렬 가정을 따름- CVUSA보다 더 도전적인 장면 포함 |
| **데이터 출처**               | 2                                                            | 16                                                           |

**Table 1: 교차 시점 지리적 위치 추정 벤치마크 데이터셋 (CVUSA & CVACT)**

이 두 데이터셋의 가장 중요한 공통점은 **'중앙 정렬 가정(centered alignment assumption)'**입니다.20 즉, 모든 지상 쿼리 이미지는 정확히 그 위치에 해당하는 위성 이미지의 중앙에 위치하도록 데이터가 구성되어 있습니다. 이 가정은 SAFA의 극좌표 변환이 효과적으로 작동하는 데 결정적인 역할을 합니다. 위성 이미지의 중심을 극좌표 변환의 원점으로 사용하면, 변환된 이미지가 지상 파노라마와 기하학적으로 잘 정렬되기 때문입니다. 그러나 이 가정은 동시에 SAFA의 한계를 암시하기도 합니다. 실제 세계에서는 이러한 완벽한 정렬을 보장할 수 없기 때문입니다.






SAFA는 발표 당시 기존의 최첨단(State-of-the-Art, SOTA) 모델들을 압도적인 성능 차이로 능가하며 CVGL 분야에 큰 충격을 주었습니다. 성능은 주로 상위 K개의 검색 결과에 정답 이미지가 포함될 확률을 나타내는 **Recall@K (R@K)** 지표로 평가되었습니다.3

| 모델            | 데이터셋  | R@1 (%)  | R@5 (%)  | R@10 (%) | R@1% (%) |
| --------------- | --------- | -------- | -------- | -------- | -------- |
| CVM-Net (2018)  | CVUSA     | 22.5     | -        | -        | -        |
| Liu & Li (2019) | CVUSA     | 40.7     | -        | 76.4     | 91.1     |
| **SAFA (Ours)** | **CVUSA** | **89.8** | **96.9** | **98.1** | **99.6** |
| CVM-Net (2018)  | CVACT     | 20.1     | -        | -        | -        |
| Liu & Li (2019) | CVACT     | 46.9     | -        | 72.8     | 89.2     |
| **SAFA (Ours)** | **CVACT** | **81.0** | **92.8** | **94.8** | **98.2** |

Table 2: SAFA와 이전 SOTA 모델 성능 비교 (Recall@K, %)

참고: 위 표의 수치는 SAFA 논문 3에 보고된 값을 기반으로 재구성되었습니다. CVACT의 경우 검증 세트(validation set) 기준 성능입니다.

위 표에서 볼 수 있듯이, SAFA의 성능 향상은 점진적인 개선이 아닌, 비약적인 도약이었습니다.

- **CVUSA 데이터셋:** Top-1 재현율(R@1)에서 이전 SOTA 모델 대비 2배 이상, CVM-Net 대비 거의 4배에 가까운 성능 향상을 기록했습니다.3 90%에 육박하는 R@1은 당시로서는 거의 해결 불가능해 보였던 CVGL 문제의 '해결 가능성'을 보여준 놀라운 결과였습니다.
- **CVACT 데이터셋:** 더 도전적인 CVACT 데이터셋에서도 R@1 81.0%라는 높은 성능을 달성하며, 제안된 방법론의 일반화 가능성을 입증했습니다.3

이러한 압도적인 결과는 SAFA가 제안한 '기하학적/특징적 대응 관계 분리'라는 접근법이 이론적으로 타당할 뿐만 아니라, 실질적으로도 매우 효과적임을 강력하게 증명했습니다.3 이 성공은 CVGL 분야의 연구 방향에 큰 영향을 미쳤고, SAFA는 이후 수많은 연구에서 비교 기준으로 활용되는 표준 모델로 자리매김했습니다.






SAFA의 놀라운 성능이 단일 요소가 아닌, 각 구성 요소의 유기적인 결합 덕분임을 증명하기 위해 논문에서는 상세한 제거 실험(ablation study)을 수행했습니다. 이 분석은 SAFA 아키텍처의 설계 타당성을 입증하는 데 핵심적인 역할을 합니다.

| 모델 구성                         | CVUSA R@1 (%) | CVACT (Val) R@1 (%) | 분석                                                         |
| --------------------------------- | ------------- | ------------------- | ------------------------------------------------------------ |
| **Baseline** (VGG + FC)           | 42.87         | 42.92               | 극좌표 변환과 SAFA 모듈이 없는 기본 모델의 성능.             |
| **Baseline + Polar**              | 75.94         | 55.48               | 극좌표 변환만 추가해도 성능이 대폭 상승. 기하학적 정렬의 중요성을 입증. |
| **Baseline + SAFA (M=8)**         | 63.31         | 59.54               | 공간 어텐션 집계 모듈 자체도 상당한 성능 향상을 가져옴.      |
| **Full Model (Polar + SAFA M=1)** | 85.93         | 76.53               | 극좌표 변환과 단일 SPE 모듈 결합 시 큰 시너지 효과 발생.     |
| **Full Model (Polar + SAFA M=4)** | 88.35         | 79.86               | SPE 모듈 수를 늘리면 다양한 공간 정보를 포착하여 성능이 추가로 향상됨. |
| **Full Model (Polar + SAFA M=8)** | **89.84**     | **81.03**           | M=8에서 최적의 성능을 보이며, 다중 어텐션 집계 전략의 유효성을 최종 확인. |

Table 3: SAFA 구성 요소별 제거 실험 결과 (Ablation Study)

참고: 위 표는 SAFA 논문 3의 Table 2와 Table 3에 제시된 결과를 종합하여 재구성되었습니다.

제거 실험은 다음과 같은 중요한 사실들을 명확히 보여줍니다.

1. **시너지 효과의 입증:** 극좌표 변환만 추가했을 때의 성능 향상(CVUSA에서 +33.07%p)과 SAFA 모듈만 추가했을 때의 성능 향상(CVUSA에서 +20.44%p)을 단순히 더한 것보다, 두 요소를 함께 사용했을 때의 성능(최종 89.84%)이 훨씬 더 높습니다. 이는 두 구성 요소가 서로의 단점을 보완하고 장점을 극대화하는 **긍정적인 시너지 효과**를 내고 있음을 정량적으로 증명합니다. 극좌표 변환은 SAFA 모듈이 더 쉽게 학습할 수 있는 환경을 만들고, SAFA 모듈은 극좌표 변환으로 인한 왜곡을 보정해 줍니다.
2. **각 구성 요소의 독립적 기여:** 극좌표 변환과 SAFA 모듈은 각각 독립적으로도 상당한 성능 향상을 가져옵니다. 이는 두 아이디어 모두 CVGL 문제 해결에 있어 본질적으로 유효한 접근법임을 시사합니다.3
3. **다중 임베딩의 효과:** SPE 모듈의 개수(M)를 1개에서 8개로 늘려감에 따라 성능이 꾸준히 향상되는 것은, 다양한 관점에서 공간 정보를 집계하는 전략이 단일 관점보다 더 풍부하고 강건한 특징 표현을 만든다는 가설을 뒷받침합니다.3

이처럼 철저한 실험적 검증은 SAFA가 단순한 '운 좋은' 성공이 아니라, 명확한 철학과 체계적인 설계를 바탕으로 이루어낸 필연적인 성과임을 보여줍니다. 그러나 동시에, 이 실험들이 수행된 '이상적인' 벤치마크 환경은 SAFA의 내재적 한계를 감추는 역할도 했습니다. 이 한계점들은 다음 장에서 심층적으로 다루어질 것입니다.

------






SAFA는 의심할 여지 없이 CVGL 분야의 기술적 수준을 한 단계 끌어올렸지만, 그 방법론에는 명확한 한계와 가정이 내재되어 있습니다. 이러한 한계를 비판적으로 평가하는 것은 SAFA를 정확히 이해하고, 이 분야의 후속 연구 흐름을 파악하는 데 필수적입니다. SAFA의 성공이 역설적으로 그 자신의 한계를 드러내는 연구의 장을 열었다고 볼 수 있습니다.






SAFA의 핵심 아이디어인 극좌표 변환은 그 효과만큼이나 뚜렷한 단점을 가지고 있습니다.

- **정보 왜곡 및 손실:** 극좌표 변환은 순수하게 기하학적 관계에만 기반하므로, 이미지의 **내용(content)을 전혀 고려하지 않습니다**.3 이로 인해 변환 과정에서 심각한 왜곡이 발생합니다. 특히, 항공 이미지의 중심에서 멀리 떨어진 외곽 영역의 객체들은 방사형으로 길게 늘어지는 현상이 나타나, 원래의 형태를 알아보기 어렵게 됩니다. 이러한 왜곡은 후속 CNN이 유의미한 특징을 학습하는 데 방해가 될 수 있으며, 때로는 중요한 시각적 정보를 손실시키는 결과를 낳습니다. 후속 연구들은 SAFA와 같은 기하학 기반 방법들이 여전히 판별적인 영역(discriminative regions)을 정확히 인식하는 데 어려움을 겪는 주된 원인으로 이 왜곡 문제를 지적합니다.23
- **후속 연구의 비판적 시각:** SAFA 이후의 연구들은 극좌표 변환이 두 도메인을 '완전히' 정렬할 수 없으며, 변환으로 생성된 이미지는 종종 세부 정보가 부족하고 부자연스럽다고 평가합니다.23 예를 들어, 생성적 적대 신경망(GAN)을 이용해 더 사실적인 이미지를 합성하려는 시도도 있었지만, 이 역시 합성된 이미지의 품질 문제라는 또 다른 한계에 부딪혔습니다.23 결국, 극좌표 변환은 문제 해결을 위한 영리한 '근사치'일 뿐, 완벽한 해결책은 아니라는 점이 명확해졌습니다.






SAFA의 놀라운 성능은 특정 가정 하에서만 유효하며, 이는 일반화 성능에 대한 근본적인 의문을 제기합니다.

- **중앙 정렬 의존성:** 앞서 언급했듯이, SAFA의 성능은 쿼리 지상 이미지가 참조 위성 이미지의 **중앙에 완벽히 정렬되어 있다는 CVUSA 및 CVACT 데이터셋의 강한 가정**에 크게 의존합니다.20 SAFA 논문 자체는 테스트 시 약간의 중심 이동은 SPE 모듈이 완화할 수 있다고 주장하지만 3, 이 가정이 크게 깨지는 현실 세계의 시나리오에서는 성능을 보장하기 어렵습니다.
- **현실 세계와의 괴리:** 실제 자율주행차나 로봇이 수집하는 이미지는 참조 지도 데이터베이스의 특정 지점과 정확히 중앙 정렬될 가능성이 거의 없습니다. 오히려 쿼리 위치는 여러 참조 이미지에 걸쳐 있거나, 참조 이미지의 가장자리에 위치할 수 있습니다. VIGOR와 같은 최신 데이터셋은 바로 이러한 **비정렬(unaligned) 및 부분적 중첩(partially overlapping)** 시나리오를 포함하여 현실성을 높였습니다.6 이러한 데이터셋에서 극좌표 변환과 같은 고정된 기하학적 변환은 중심점의 불일치로 인해 오히려 심각한 오정렬을 유발하여 성능을 저하시킬 수 있습니다.23
- **제한된 화각(Limited FoV) 문제:** SAFA의 방법론은 입력 이미지가 360도 전체 시야를 담고 있는 **파노라마 이미지임을 전제로 설계**되었습니다. 그러나 대부분의 실제 애플리케이션(예: 차량 블랙박스, 스마트폰 카메라)에서 얻는 이미지는 화각이 제한적입니다. 제한된 화각 이미지에는 파노라마에 비해 훨씬 적은 정보가 담겨 있어, SAFA와 같은 전역적 레이아웃에 의존하는 모델의 성능은 급격히 저하될 수 있습니다. 이 문제를 해결하기 위해서는 파노라마 이미지와는 다른 별도의 접근 전략이 필요합니다.11

결론적으로, SAFA의 설계는 당시의 이상적인 벤치마크 환경에서는 최적의 성능을 발휘했지만, 그로 인해 현실 세계의 '지저분한(messy)' 조건에 대해서는 취약한 구조를 갖게 되었습니다. 이는 벤치마크 성능이 연구의 유일한 목표가 될 때 발생할 수 있는 '과적합'의 한 예시이며, 고도로 전문화되었지만 널리 일반화되기 어려운 솔루션으로 이어질 수 있다는 중요한 교훈을 줍니다.






사용자가 처음 질의했던 "Semantic-Aware"라는 키워드는 SAFA의 또 다른 핵심적인 한계를 정확하게 가리킵니다.

- **공간적(Spatial) 정보에 대한 집중:** SAFA는 이름 그대로 **'공간 인식(Spatial-Awareness)'**에 모든 것을 집중합니다. 즉, 객체가 '무엇'인지에는 관심이 없고, 오직 '어디에' 위치하며 다른 객체들과 '어떤 구조'를 이루는지에만 초점을 맞춥니다. 이는 기하학적 불일치라는 가장 큰 문제를 해결하기 위한 전략적 선택이었지만, 동시에 의미론적(Semantic) 정보라는 중요한 단서를 포기하는 결과를 낳았습니다.
- **의미론적 정보의 중요성:** 외형 변화에 대한 강건성을 확보하기 위해서는 의미론적 이해가 필수적입니다. 예를 들어, 여름에는 녹색 잎이 무성하고 겨울에는 앙상한 가지만 남는 나무가 있다고 가정해 봅시다. 공간적 정보에만 의존하는 SAFA는 이 두 상태를 전혀 다른 특징으로 인식할 수 있습니다. 그러나 의미론적 정보를 활용하는 모델은 두 이미지 모두에서 '나무'라는 객체를 인식하고, 그 외형 변화에 덜 민감하게 반응할 수 있습니다. 마찬가지로, 공사로 인해 건물의 색상이 바뀌거나, 특정 브랜드의 간판이 다른 것으로 교체되는 경우에도, '건물'이나 '상점'이라는 의미론적 범주를 이해하는 것이 더 강건한 매칭으로 이어질 수 있습니다.8
- **후속 연구 동향:** SAFA의 이러한 한계를 극복하기 위해, 후속 연구들은 적극적으로 의미론적 정보를 통합하는 방향으로 나아갔습니다. 일부 연구는 명시적인 의미 분할(semantic segmentation) 네트워크를 전처리 단계에 사용하여 '도로', '건물', '초목' 등의 영역을 구분하고, 각 영역별로 특징을 다르게 처리합니다.10 또 다른 연구들은 트랜스포머와 같은 아키텍처를 통해 이미지 패치들을 의미론적 부분(semantic parts)으로 클러스터링하여 특징을 집계하는 방식을 제안하기도 했습니다.27

SAFA가 CVGL 분야의 1차적인 문제, 즉 '거친 기하학적 매칭'을 매우 효과적으로 해결했기 때문에, 연구 커뮤니티는 그 다음 단계의 더 정교한 문제들, 예를 들어 "비정렬 이미지는 어떻게 처리할 것인가?", "외형 변화에 강건해지기 위해 의미론을 어떻게 통합할 것인가?"와 같은 질문들로 나아갈 수 있었습니다. 이처럼 SAFA의 한계에 대한 비판은 그 실패를 의미하는 것이 아니라, 오히려 그 성공이 있었기에 비로소 제기될 수 있었던 새로운 연구 질문들이라는 점에서 그 의의가 큽니다.

------






SAFA는 단순히 높은 성능을 기록한 하나의 논문을 넘어, 이후 CVGL 연구의 방향성에 지대한 영향을 미친 이정표와 같습니다. SAFA가 제시한 아이디어는 후속 연구자들에게 영감을 주었고, 동시에 그 한계는 새로운 패러다임의 등장을 촉진하는 계기가 되었습니다.






SAFA의 가장 큰 유산은 CVGL 문제 해결에 있어 **명시적인 기하학적 사전 지식(explicit geometric prior)**을 활용하는 접근법의 유효성을 강력하게 입증했다는 점입니다.

- **패러다임 정립:** SAFA 이전의 딥러닝 모델들이 기하학적 변환을 암묵적인 학습에만 의존했던 것과 달리, SAFA는 극좌표 변환이라는 명시적인 도구를 통해 문제를 단순화하고 성능을 극적으로 향상시켰습니다. 이는 "기하학적 정렬을 먼저 수행하고 특징을 학습한다"는 새로운 패러다임의 효용성을 증명한 것이며, 이후 수많은 CNN 기반 후속 연구들이 이 패러다임을 따르게 되었습니다.6
- **영감의 원천:** SAFA의 성공은 연구자들이 극좌표 변환 외에도 다양한 기하학적 제약 조건이나 변환 방법을 딥러닝 프레임워크에 통합하려는 시도를 촉발시키는 계기가 되었습니다. SAFA는 이 분야에서 기하학 기반 접근법의 사실상 표준(de facto standard)이자 비교 기준으로 자리 잡았습니다.






SAFA가 정립한 패러다임은 트랜스포머(Transformer) 아키텍처의 등장과 함께 새로운 도전을 맞이하게 됩니다. 이는 컴퓨터 비전 분야 전반의 거대한 흐름, 즉 명시적 사전 지식 주입에서 데이터 기반의 종단간(end-to-end) 학습으로의 전환을 반영합니다.

- **TransGeo의 도전:** 2022년 발표된 **TransGeo**는 CVGL 분야에 또 다른 패러다임 전환을 예고했습니다.29 TransGeo는 SAFA와 같이 극좌표 변환이라는 명시적인 기하학적 변환에 의존하는 대신, 트랜스포머의 핵심 메커니즘인 **셀프 어텐션(self-attention)과 위치 인코딩(positional encoding)**을 활용합니다. 이를 통해 이미지 패치들 간의 전역적인 상관관계(global correlation)와 공간적 구성을 데이터로부터 

  **암묵적으로 학습**합니다.

- **명시적 변환 vs. 암묵적 학습:** SAFA가 "모델에게 기하학을 명시적으로 가르치는" 접근법이라면, TransGeo는 "모델이 방대한 데이터로부터 스스로 기하학을 배우게 하는" 접근법에 가깝습니다. 트랜스포머는 이미지 전체를 패치 단위로 나누고, 각 패치가 다른 모든 패치와 어떤 관계를 맺고 있는지를 어텐션 메커니즘을 통해 학습하기 때문에, 고정된 기하학적 변환보다 훨씬 더 유연하고 데이터에 적응적인 공간 관계를 파악할 수 있습니다. 이는 SAFA의 가정이 깨지는 비정렬 데이터셋에서 특히 더 강력한 일반화 성능으로 이어졌습니다.

- **계산 효율성:** 또한 TransGeo는 극좌표 변환과 같은 별도의 전처리 과정이 필요 없기 때문에, 전체 파이프라인이 더 단순하고 추론 속도가 빠르다는 장점을 제시했습니다.29

SAFA에서 TransGeo로의 전환은 CVGL 분야의 기술적 성숙도를 보여주는 상징적인 변화입니다. SAFA는 고전적 기하학과 딥러닝을 영리하게 결합한 하이브리드 접근법의 정점을 보여주었고, 트랜스포머는 더 강력한 학습 능력으로 이러한 사전 지식조차도 데이터로부터 직접 학습할 수 있는 가능성을 열었습니다.






모델 아키텍처의 발전은 필연적으로 벤치마크 데이터셋의 발전을 동반합니다. 새로운 모델의 장점을 입증하고 기존 모델의 한계를 드러내기 위해서는 더 현실적이고 도전적인 평가 기준이 필요하기 때문입니다.

- **VIGOR 데이터셋의 등장:** 2021년 제안된 **VIGOR (Cross-View Image Geo-localization beyond One-to-one Retrieval)** 데이터셋은 CVGL 연구를 실험실 환경에서 현실 세계로 한 걸음 더 나아가게 만들었습니다.6
- **"일대일 검색을 넘어서 (Beyond One-to-one Retrieval)":** VIGOR는 CVUSA/CVACT와 근본적으로 다른 두 가지 특징을 가집니다.
  1. **비정렬(Unaligned):** 쿼리 이미지가 참조 위성 이미지의 중앙에 위치하지 않습니다. 쿼리 위치는 참조 이미지 내에서 임의의 오프셋을 가질 수 있습니다.6
  2. **다대다(Many-to-many):** 하나의 쿼리 장소가 여러 참조 이미지에 걸쳐 부분적으로 포함될 수 있어, 완벽한 일대일 대응 관계가 존재하지 않습니다.6
- **SAFA의 한계 노출:** 이러한 현실적인 설정은 SAFA와 같이 강한 기하학적 정렬 가정에 기반한 모델들의 취약점을 명확히 드러냈습니다. 쿼리 위치가 중심에서 벗어날 경우, SAFA의 극좌표 변환은 올바른 기하학적 대응 관계를 형성하지 못하고 오히려 심각한 왜곡을 유발하여 성능을 저하시킵니다.23

결국, 새로운 모델(TransGeo)의 등장과 새로운 데이터셋(VIGOR)의 개발은 서로 맞물려 돌아가는 톱니바퀴처럼 CVGL 분야의 발전을 이끌었습니다. SAFA가 CVUSA라는 기존의 경기장에서 너무나 압도적인 성공을 거두었기에, 연구 커뮤니티는 더 어려운 경기장인 VIGOR를 만들 필요성을 느꼈습니다. 그리고 이 새로운 경기장은 SAFA와 같은 기존 챔피언의 약점을 드러내고, TransGeo와 같은 새로운 유형의 선수들이 활약할 수 있는 무대를 제공했습니다. 이처럼 모델과 벤치마크의 상호작용 및 공동 진화는 이 분야의 연구를 건강하게 발전시키는 핵심 동력입니다.

------






SAFA는 교차 시점 지리적 위치 추정(CVGL) 분야의 역사에서 단순한 하나의 SOTA(최첨단) 모델을 넘어, 문제 해결 방식에 대한 근본적인 고찰을 이끌어낸 중요한 이정표로 기록됩니다. SAFA의 성공과 한계를 종합적으로 분석함으로써, 우리는 이 분야의 과거를 이해하고 미래를 조망할 수 있는 귀중한 교훈을 얻을 수 있습니다.






- SAFA의 가장 큰 기여는 복잡하게 얽혀 있던 CVGL 문제를 **기하학적 정렬과 특징 학습이라는 두 개의 하위 문제로 분리**하는 획기적인 아이디어를 제시하고 그 유효성을 입증한 것입니다. 이 접근법은 당시 정체되어 있던 분야의 성능을 비약적으로 향상시키며 **게임 체인저** 역할을 했습니다.
- 방법론적으로는, 고전적인 기하학 원리인 **극좌표 변환**과 딥러닝 기반의 **다중 공간 어텐션 집계(SAFA 모듈)**를 결합하여, 강력한 사전 지식을 딥러닝 프레임워크에 효과적으로 통합하는 성공적인 선례를 남겼습니다.






- **문제 분리의 힘:** 가장 정교하고 복잡한 종단간 딥러닝 모델만이 항상 최적의 해법은 아니라는 점을 명확히 보여주었습니다. 때로는 어려운 문제를 더 작고 다루기 쉬운 하위 문제로 나누고, 각 문제에 가장 적합한 도구(고전적인 방법과 딥러닝의 결합 등)를 사용하는 것이 훨씬 더 효과적일 수 있다는 중요한 공학적 지혜를 제공했습니다.
- **벤치마크의 양면성:** SAFA의 성공과 그 이후의 한계 노출 과정은 벤치마크의 양면성을 극적으로 보여줍니다. 특정 벤치마크에 과도하게 최적화된 솔루션은 기록적인 성능을 달성할 수 있지만, 동시에 현실 세계의 다양성과 복잡성을 반영하지 못하는 '온실 속 화초'가 될 위험이 있습니다. 이는 모델의 발전과 벤치마크의 발전이 반드시 함께 가야 한다는 점을 시사합니다.






SAFA가 길을 연 이후, CVGL 분야는 더욱 복잡하고 현실적인 문제들을 해결하기 위해 나아가고 있습니다. 현재 이 분야가 직면한 주요 도전 과제와 미래 연구 방향은 다음과 같습니다.

- **강건성 (Robustness):** 계절 변화, 극심한 조명 변화(예: 낮과 밤), 악천후(비, 눈, 안개), 센서 노이즈 등 현실에서 마주할 수 있는 다양한 외형 변화에 강건한 모델을 개발하는 것이 여전히 중요한 과제입니다. 이를 위해 데이터 증강, 도메인 일반화, 의미론적 정보 활용 등 다양한 기법이 연구되고 있습니다.21
- **효율성 (Efficiency):** 실제 자율주행차나 모바일 로봇, AR 기기와 같은 자원 제약적인 플랫폼에 탑재되기 위해서는 모델의 경량화와 실시간 추론 속도 확보가 필수적입니다. 이는 모델 압축, 지식 증류, 효율적인 네트워크 아키텍처 설계 등의 연구로 이어지고 있습니다.26
- **비지도/자기지도 학습 (Unsupervised/Self-supervised Learning):** 정확하게 위치가 정렬된 대규모 지상-항공 이미지 쌍을 구축하는 데는 막대한 비용과 노력이 소요됩니다.34 따라서 GPS 태그나 명시적인 쌍 정보 없이, 온라인에서 쉽게 수집할 수 있
- 
- 는 방대한 비정형 데이터를 활용하여 위치를 추정하는 비지도 및 자기지도 학습 패러다임이 미래 CVGL의 핵심 기술로 부상하고 있습니다.34
- **의미론과 기하학의 통합:** 미래의 CVGL 모델은 SAFA의 공간적 접근과 후속 연구들의 의미론적 접근을 유기적으로 통합하는 방향으로 나아갈 것입니다. 즉, 장면의 '구조(어디에)'와 '내용(무엇이)'을 모두 이해하여, 시점과 외형 변화에 모두 강건한 하이브리드 특징 표현을 학습하는 것이 목표입니다.10
- **윤리적 고찰 (Ethical Considerations):** CVGL 기술의 발전은 필연적으로 심각한 사회적, 윤리적 문제를 동반합니다. 이 기술은 개인의 동의 없이 실시간 위치를 추적하고, 이동 경로를 분석하여 사생활을 침해하거나, 대규모 감시 시스템에 악용될 수 있는 잠재적 위험을 내포하고 있습니다.36 법 집행 기관이 범죄 수사에 위치 데이터를 활용하는 긍정적 측면도 있지만 39, 기술의 오남용 가능성은 항상 존재합니다. 따라서 연구자들은 알고리즘 개발 단계에서부터 데이터 수집의 투명성, 사용 목적의 명확성, 프라이버시 보호 기술(privacy-by-design) 등을 통합하고, 기술의 사회적 영향에 대한 깊은 성찰을 병행해야 할 책임이 있습니다.40

결론적으로, SAFA는 CVGL 분야에 명확한 해법과 동시에 새로운 질문을 던진 중요한 연구입니다. SAFA가 이룩한 성과 위에서, 미래의 연구는 더 강건하고, 효율적이며, 일반화 성능이 뛰어나고, 궁극적으로는 윤리적으로 책임감 있는 방향으로 나아가야 할 것입니다.




1. Spatial-Aware Feature Aggregation for Image based Cross-View Geo-Localization - NIPS, accessed July 1, 2025, https://papers.nips.cc/paper/9199-spatial-aware-feature-aggregation-for-image-based-cross-view-geo-localization
2. YujiaoShi/cross_view_localization_SAFA - GitHub, accessed July 1, 2025, https://github.com/YujiaoShi/cross_view_localization_SAFA
3. Spatial-Aware Feature Aggregation for Image based Cross-View ..., accessed July 1, 2025, http://papers.neurips.cc/paper/9199-spatial-aware-feature-aggregation-for-image-based-cross-view-geo-localization.pdf
4. Rethinking Visual Geo-localization for Large-Scale Applications - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/paper/rethinking-visual-geo-localization-for-large/review/
5. CVPR 2023: A Comprehensive Tour and Recent Advancements toward Real-world Visual Geo-Localization - SRI, accessed July 1, 2025, https://www.sri.com/research/information-computing-sciences/computer-vision/cvpr-2023-a-comprehensive-tour-and-recent-advancements-toward-real-world-visual-geo-localization/
6. VIGOR: Cross-View Image Geo-Localization Beyond One-to-One Retrieval - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Zhu_VIGOR_Cross-View_Image_Geo-Localization_Beyond_One-to-One_Retrieval_CVPR_2021_paper.pdf
7. Cross-view geo-localization: a survey - arXiv, accessed July 1, 2025, https://arxiv.org/html/2406.09722v1
8. (PDF) A review of Visual-Based Localization - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/337411335_A_review_of_Visual-Based_Localization
9. Feature Relation Guided Cross-View Image Based Geo-Localization - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/15/20/5029
10. SFD2: Semantic-guided Feature Detection and Description - arXiv, accessed July 1, 2025, https://arxiv.org/html/2304.14845
11. Cross-View Image Sequence Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/WACV2023/papers/Zhang_Cross-View_Image_Sequence_Geo-Localization_WACV_2023_paper.pdf
12. (PDF) CVM-Net: Cross-View Matching Network for Image-Based Ground-to-Aerial Geo-Localization - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/324438344_CVM-Net_Cross-View_Matching_Network_for_Image-Based_Ground-to-Aerial_Geo-Localization
13. CVM-Net: Cross-View Matching Network for Image-Based Ground-to-Aerial Geo-Localisation | crossview_localisation - Sixing Hu, accessed July 1, 2025, https://david-husx.github.io/crossview_localisation/
14. ArcGeo: Localizing Limited Field-of-View Images Using Cross-View Matching - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/WACV2024/papers/Shugaev_ArcGeo_Localizing_Limited_Field-of-View_Images_Using_Cross-View_Matching_WACV_2024_paper.pdf
15. Crossview USA (CVUSA) - MVRL, accessed July 1, 2025, https://mvrl.cse.wustl.edu/datasets/cvusa/
16. wtyhub/SAFA_LPN - GitHub, accessed July 1, 2025, https://github.com/wtyhub/SAFA_LPN
17. Optimal Feature Transport for Cross-View Image Geo-Localization - GitHub, accessed July 1, 2025, https://github.com/YujiaoShi/cross_view_localization_CVFT
18. Awesome Geo-localization - Zhedong Zheng, accessed July 1, 2025, https://www.zdzheng.xyz/Awesome-Geo-localization
19. (a) Our used CVACT train (blue) / validation (green) / test (red) data... - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/figure/a-Our-used-CVACT-train-blue-validation-green-test-red-data-splits-b-One_fig1_348174909
20. VIGOR: Cross-View Image Geo-localization beyond One-to-one Retrieval - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/355882288_VIGOR_Cross-View_Image_Geo-localization_beyond_One-to-one_Retrieval
21. Benchmarking the Robustness of Cross-view Geo-localization Models Supplementary Material, accessed July 1, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/11762-supp.pdf
22. Reviews: Spatial-Aware Feature Aggregation for Image based Cross-View Geo-Localization, accessed July 1, 2025, https://proceedings.neurips.cc/paper/2019/file/ba2f0015122a5955f8b3a50240fb91b2-Reviews.html
23. Cross-View Geo-Localization With Step-Adaptive Iterative Refinement, accessed July 1, 2025, [https://csse.szu.edu.cn/attachment/cglr/1681787728_TGRS%202022.10-Its_Okay_to_Be_Wrong_Cross-View_Geo-Localization_With_Step-Adaptive_Iterative_Refinement.pdf](https://csse.szu.edu.cn/attachment/cglr/1681787728_TGRS 2022.10-Its_Okay_to_Be_Wrong_Cross-View_Geo-Localization_With_Step-Adaptive_Iterative_Refinement.pdf)
24. GeoViewMatch: A Multi-Scale Feature-Matching Network for Cross-View Geo-Localization Using Swin-Transformer and Contrastive Learning - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/16/4/678
25. VIGOR Dataset - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/dataset/vigor
26. ICT-Net: A Framework for Multi-Domain Cross-View Geo-Localization with Multi-Source Remote Sensing Fusion - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/17/12/1988
27. arXiv:2401.01574v1 [cs.CV] 3 Jan 2024, accessed July 1, 2025, https://arxiv.org/pdf/2401.01574
28. VAGeo: View-specific Attention for Cross-View Object Geo-Localization - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2501.07194
29. TransGeo: Transformer Is All You Need for Cross-View Image Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2022/papers/Zhu_TransGeo_Transformer_Is_All_You_Need_for_Cross-View_Image_Geo-Localization_CVPR_2022_paper.pdf
30. A Cross-View Image Matching Method with Feature Enhancement - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/15/8/2083
31. Benchmarking the Robustness of Cross-view Geo-localization Models - ECCV 2024, accessed July 1, 2025, https://eccv.ecva.net/virtual/2024/poster/2096
32. Visual Place Recognition - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/task/visual-place-recognition/codeless?page=2
33. A Benchmark Comparison of Visual Place Recognition Techniques for Resource-Constrained Embedded Platforms | Request PDF - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/354808156_A_Benchmark_Comparison_of_Visual_Place_Recognition_Techniques_for_Resource-Constrained_Embedded_Platforms
34. Unleashing Unlabeled Data: A Paradigm for Cross-View Geo-Localization - arXiv, accessed July 1, 2025, https://arxiv.org/html/2403.14198v1
35. Unleashing Unlabeled Data: A Paradigm for Cross-View Geo-Localization - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Li_Unleashing_Unlabeled_Data_A_Paradigm_for_Cross-View_Geo-Localization_CVPR_2024_paper.pdf
36. The Societal Impact of Surveillance, accessed July 1, 2025, https://www.numberanalytics.com/blog/societal-impact-of-surveillance
37. How location tracking is raising the stakes on privacy protection | EY - It, accessed July 1, 2025, https://www.ey.com/en_it/insights/forensic-integrity-services/how-location-tracking-is-raising-the-stakes-on-privacy-protection
38. The power of where: geolocation revolution in information security - Hertie School, accessed July 1, 2025, https://www.hertie-school.org/en/digital-governance/research/blog/detail/content/the-power-of-where-geolocation-revolution-in-information-security
39. Enhancing Investigations: The Role of Geolocation Data in Solving Cold Cases, accessed July 1, 2025, https://www.blueforcelearning.com/blog/the-role-of-geolocation-data-in-solving-cold-cases
40. Google Street View privacy concerns - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Google_Street_View_privacy_concerns
41. Google-Contributed Street View Imagery Policy, accessed July 1, 2025, https://www.google.com/streetview/policy/

THE GOOGLE STREET VIEW WI-FI SCANDAL AND ITS REPERCUSSIONS FOR PRIVACY REGULATION - Monash University, accessed July 1, 2025, https://www.monash.edu/__data/assets/pdf_file/0011/141230/vol-39-3-burdon-and-mckillop.pdf
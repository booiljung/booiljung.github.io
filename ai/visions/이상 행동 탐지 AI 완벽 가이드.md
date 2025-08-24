# 이상 행동 탐지 AI 완벽 가이드
[비전 AI](./index.md)



이상 행동 탐지(Anomaly Detection) 기술의 핵심 가치는 단순히 문제를 찾는 것을 넘어, 비즈니스 및 운영 전략의 근본적인 패러다임 전환을 이끌어내는 데 있습니다. 과거 시스템은 문제가 발생한 *후에야* 이를 해결하는 반응적(Reactive) 대응에 머물렀습니다. 이는 상당한 비용 손실, 브랜드 평판 하락, 심각한 보안 침해로 이어지곤 했습니다.1 전통적인 방식은 사람이 직접 데이터를 추적하거나, 사전에 정의된 단순한 임계값(Threshold)에 의존했습니다.2

그러나 데이터가 폭발적으로 증가하면서 수동 추적은 사실상 불가능해졌고, 이는 자동화된 솔루션의 필요성을 절실하게 만들었습니다.1 인공지능(AI) 기반 이상 행동 탐지는 이러한 한계를 극복하고, 잠재적인 문제가 심각한 장애로 확산되기 *전에* 이를 예측하고 식별하는 선제적(Proactive) 방법론을 가능하게 합니다.5 이러한 전환은 시스템의 견고성, 보안, 효율성을 유지하는 데 필수적입니다.4


이상 행동 탐지는 주어진 데이터 세트의 예상되는 패턴에서 현저하게 벗어나는 희귀한 항목, 이벤트, 또는 관측치를 식별하는 프로세스입니다.7 오늘날 이 기술은 현대 기업의 필수적인 역량으로 자리 잡았습니다. 탐지 시스템이 없다면, 조직은 수년간 쌓아온 브랜드 자산과 수익을 한순간에 잃을 수 있으며, 민감한 고객 정보 유출과 같은 치명적인 보안 침해에 직면하게 됩니다.1 또한, 탐지되지 않은 이상치로 인해 왜곡된 데이터는 부정확한 분석으로 이어져 잘못된 비즈니스 의사결정을 초래할 수 있습니다.7

이 기술의 적용 범위는 매우 광범위하여 사이버 보안, 금융 사기 탐지, 헬스케어, 제조업의 예지보전 등 다양한 산업 분야에서 현대 AI 기반 운영의 초석 역할을 하고 있습니다.8

이러한 맥락에서, 강력한 이상 행동 탐지 시스템의 구현은 단순히 내부 운영 효율성을 높이는 기술적 도구를 넘어, 디지털 시대에 조직의 신뢰도를 대변하는 지표가 되고 있습니다. 초기의 이상 탐지는 시스템 상태나 네트워크 트래픽과 같은 내부 지표에 집중하여 비용 절감과 효율성 증대를 목표로 했습니다.2 그러나 온라인 뱅킹, 전자상거래 등 시스템이 고객과 직접적으로 연결되면서, 서비스 중단이나 보안 침해와 같은 이상 현상은 최종 사용자에게 즉각적인 피해를 주게 되었습니다.1 이제 기업이 이상 현상을 얼마나 선제적으로 탐지하고 완화할 수 있는지는 고객의 데이터와 서비스를 안전하게 보호하는 신뢰할 수 있는 기업이라는 사실을 시장에 증명하는 중요한 신호가 되었습니다. 이는 IT 중심의 우려에서 벗어나 브랜드 가치와 직결되는 약속으로 진화했음을 의미합니다.1


본 보고서는 이상 행동 탐지 AI 기술에 대한 포괄적이고 심층적인 이해를 제공하는 것을 목표로 합니다. 독자들은 기술의 근본 원리와 핵심 알고리즘부터 실제 산업 적용 사례, 그리고 미래의 도전 과제에 이르기까지 체계적인 여정을 통해 이 분야의 전문가 수준의 통찰력을 얻게 될 것입니다.



이상 행동 탐지의 핵심 과제는 '정상(Normal)' 행동에 대한 모델을 구축하는 것입니다. 이는 결코 간단한 작업이 아닌데, '정상'의 기준은 복잡하고 동적이며 특정 맥락에 따라 달라질 수 있기 때문입니다.3 AI 기반 시스템은 방대한 양의 과거 데이터를 분석하여 이 기준선(Baseline)을 학습합니다.13 이 과정은 다음과 같은 단계를 포함합니다.

- **문제 정의 (Problem Definition):** 가장 중요하고 선행되어야 할 단계는 해결하고자 하는 문제를 정확하게 정의하는 것입니다. 예를 들어, 단순히 '문자와 숫자를 인식'하는 것이 아니라 '자동차 번호판의 문자와 숫자를 인식'하는 것으로 문제를 구체화해야 합니다. 문제 정의가 잘못되면 이후의 모든 과정이 성공적으로 진행되더라도 원하는 결과를 얻을 수 없습니다.14
- **데이터 수집 및 품질 확보 (Data Collection & Quality):** 모델이 정의하는 '정상'의 범위는 전적으로 학습 데이터에 의존합니다. 따라서 학습 데이터는 분석 대상을 대표할 수 있어야 하며, 정확하고 충분한 양이 확보되어야 합니다.13 편향되거나 불완전한 학습 데이터는 모델 실패의 주요 원인이 됩니다.16
- **행동 프로파일링 (Behavioral Profiling):** 시스템은 특정 사용자나 개체(Entity)의 시간에 따른 표준 행동 패턴을 분석하여 프로필을 생성합니다. 이 프로필은 새로운 활동을 비교하고 평가하는 기준점이 됩니다.11

이렇게 구축된 정상 패턴에서 현저하게 벗어나는 데이터 포인트나 이벤트가 바로 '이상(Anomaly)' 또는 '이상치(Outlier)'로 정의됩니다.1 이러한 이상 현상은 의도적일 수도(예: 계획된 할인 행사에 따른 매출 급증) 있고, 비의도적일 수도(예: 시스템 오류, 보안 침해) 있습니다.17

여기서 중요한 점은 '이상'의 정의가 단순히 통계적인 문제를 넘어 본질적으로 비즈니스 전략적 결정이라는 사실입니다. 기술적으로 이상은 통계적 편차로 측정되지만 7, 어느 정도의 편차를 의미 있는 이상으로 판단할 것인가 하는 임계값 설정은 순수하게 통계만으로 결정되지 않습니다. 이상 탐지 전략은 비즈니스 문제와 관련된 핵심 성과 지표(KPI)를 식별하는 것에서 시작됩니다.1 예를 들어, 금융 사기 탐지에서 임계값을 낮추면(민감도를 높이면) 실제 사기를 놓칠 확률(거짓 음성, False Negative)은 줄어들지만, 정상 거래를 사기로 잘못 판단하는 경우(거짓 양성, False Positive)가 늘어나 고객 경험을 해치게 됩니다. 반대로 임계값을 높이면 그 반대의 결과가 나타납니다.18 AI는 통계적 신호를 제공하지만, 그 신호에 어떤 의미를 부여하고 행동으로 옮길 것인가는 결국 비즈니스 차원의 위험 관리 결정인 것입니다.


이상의 유형을 정확히 이해하는 것은 효과적인 탐지 알고리즘을 선택하는 데 매우 중요합니다. 이상 현상은 크게 세 가지 유형으로 분류할 수 있습니다.

- **점 이상 (Point Anomalies / Global Outliers):** 데이터셋의 나머지 데이터와 비교했을 때, 하나의 데이터 포인트가 독립적으로 비정상적인 경우를 의미합니다. 예를 들어, 평균 거래액이 5만 원인 사용자가 갑자기 1,000만 원을 결제하는 경우가 이에 해당합니다.3 이는 가장 단순하고 직관적인 유형의 이상입니다.
- **문맥적 이상 (Contextual Anomalies):** 데이터 포인트 자체는 정상 범위에 속하지만, 특정 문맥(Context) 안에서 발생했을 때 비정상으로 간주되는 경우입니다. 예를 들어, 서버의 CPU 사용량이 오후 3시에 급증하는 것은 정상이지만, 모든 사용이 중단된 새벽 3시에 동일하게 급증한다면 이는 문맥적 이상입니다.3
- **집단적 이상 (Collective Anomalies):** 개별 데이터 포인트들은 정상적으로 보이지만, 이들이 집합을 이루었을 때 비정상적인 패턴을 형성하는 경우입니다. 예를 들어, 여러 IP 주소에서 발생하는 개별적으로는 미미한 수준의 네트워크 요청들이 모여 분산 서비스 거부(DDoS) 공격을 구성하는 경우가 대표적인 집단적 이상입니다.3


이 섹션에서는 이상 행동 탐지에 사용되는 세 가지 주요 머신러닝 학습 패러다임을 비교 분석하고, 각각의 작동 방식과 장단점을 심도 있게 탐구합니다.


- **원리:** 지도 학습(Supervised Learning)은 모든 데이터 포인트가 '정상(Normal)' 또는 '비정상(Abnormal)'으로 명확하게 레이블링된 데이터셋을 사용하여 모델을 훈련시킵니다.20 모델은 두 클래스를 구분하는 결정 경계(Decision Boundary)를 학습하여 새로운 데이터가 어느 클래스에 속하는지 분류합니다.
- **알고리즘:** 로지스틱 회귀(Logistic Regression), 결정 트리(Decision Tree), 서포트 벡터 머신(SVM), 신경망(Neural Network) 등이 사용됩니다.24
- **장점:** 학습 데이터에 잘 표현된 *알려진* 유형의 이상에 대해서는 매우 높은 정확도를 달성할 수 있습니다.23
- **단점 및 과제:**
  - **레이블링 병목 현상:** 정확하게 레이블된 데이터를 확보하는 것, 특히 드물게 발생하는 이상 현상에 대한 데이터를 모으는 것은 매우 어렵고 시간과 비용이 많이 소요됩니다.23
  - **데이터 불균형 문제:** 이상 탐지 데이터셋은 본질적으로 정상 데이터가 비정상 데이터보다 훨씬 많은 불균형(Imbalanced) 구조를 가집니다. 이로 인해 표준적인 분류 모델은 단순히 다수 클래스(정상)로만 예측해도 높은 정확도를 보여 성능 평가가 왜곡될 수 있습니다.26
  - **새로운 위협 탐지 불가:** 지도 학습 모델은 훈련 데이터에서 학습한 패턴과 유사한 이상 현상만 탐지할 수 있습니다. 따라서 이전에 알려지지 않은 새로운 유형의 공격이나 이상 패턴에는 효과적으로 대응하기 어렵습니다.23 이러한 한계로 인해 실제 이상 탐지 시나리오에서는 순수한 지도 학습 기법이 드물게 사용됩니다.7


- **원리:** 비지도 학습(Unsupervised Learning)은 레이블이 없는 데이터를 기반으로 데이터 자체의 내재된 구조와 패턴을 학습합니다.20 모델은 '정상' 데이터의 고유한 특성을 학습하고, 이 구조에서 벗어나는 데이터를 이상으로 간주합니다.
- **중요성:** 이 접근법은 레이블링된 이상 데이터가 없는 대부분의 실제 환경에 적용할 수 있기 때문에 이상 탐지 분야에서 가장 보편적으로 사용됩니다.7
- **기법:**
  - **클러스터링 기반 (Clustering-based):** 정상 데이터는 크고 밀집된 클러스터(군집)를 형성하고, 이상치는 어떤 클러스터에도 속하지 않거나 매우 작은 클러스터를 형성한다는 가정에 기반합니다 (예: K-평균, DBSCAN).5
  - **밀도 기반 (Density-based):** 데이터 공간에서 밀도가 낮은 지역에 위치한 포인트를 이상치로 식별합니다 (예: Local Outlier Factor - LOF).4
  - **재구성 기반 (Reconstruction-based):** 오토인코더와 같은 딥러닝 모델이 정상 데이터를 압축하고 복원하는 방법을 학습합니다. 이상 데이터는 복원이 잘 되지 않아 높은 재구성 오류(Reconstruction Error)를 보입니다.33
- **장점:** 알려지지 않은 새로운 유형의 이상을 탐지할 수 있으며, 비용이 많이 드는 데이터 레이블링 과정이 필요 없습니다.23
- **단점:** 정답 레이블이 없기 때문에 결과를 객관적으로 평가하기 어렵고 주관적일 수 있습니다. 또한, 거짓 양성 비율이 상대적으로 높고 학습 데이터의 노이즈에 민감할 수 있습니다.23


- **원리:** 준지도 학습(Semi-supervised Learning)은 소량의 레이블된 데이터와 대량의 레이블 없는 데이터를 결합하여 모델을 훈련시킵니다.20 일반적으로 

  *정상 데이터에 대해서만 레이블링된* 데이터셋을 사용합니다.

- **작동 방식:** 모델은 레이블된 정상 데이터의 특징을 학습하여 정상 데이터 주변에 매우 정교하고 촘촘한 경계를 설정합니다. 이후 새로운 데이터가 이 경계 밖으로 벗어나면 이상으로 분류합니다.17 원-클래스 SVM(One-Class SVM)이 이 접근법의 대표적인 예입니다.17

- **장점:** '두 세계의 장점'을 모두 취하는 접근법으로, 지도 학습의 비현실적인 레이블링 요구를 피하면서도, 레이블된 정상 데이터를 강력한 가이드로 사용하여 순수 비지도 학습보다 높은 정확도를 달성할 수 있습니다.23

- **단점:** 모델의 성능은 초기에 제공된 레이블된 정상 데이터의 품질과 대표성에 크게 의존합니다. 만약 레이블 없는 데이터에 초기 학습 데이터에는 없었던 새로운 유형의 *정상* 행동 패턴이 포함되어 있다면, 이를 이상으로 잘못 판단할 수 있습니다.39


| 패러다임        | 입력 데이터 요구사항                              | 핵심 원리             | 주요 장점                                             | 주요 단점                                          | 이상적인 활용 사례                              |
| --------------- | ------------------------------------------------- | --------------------- | ----------------------------------------------------- | -------------------------------------------------- | ----------------------------------------------- |
| **지도 학습**   | 모든 데이터가 '정상' 또는 '비정상'으로 레이블링됨 | 분류 (Classification) | 알려진 이상 유형에 대한 높은 정확도                   | 레이블링 비용이 높고, 새로운 유형의 이상 탐지 불가 | 스팸 메일 탐지 (클래스가 명확히 정의됨)         |
| **비지도 학습** | 레이블 없는 데이터                                | 패턴/구조 발견        | 새로운 유형의 이상 탐지 가능, 레이블링 비용 없음      | 높은 거짓 양성률, 결과가 주관적일 수 있음          | 네트워크 침입 탐지 (새로운 공격 유형 발생)      |
| **준지도 학습** | 일부 데이터만 레이블링됨 (주로 '정상' 데이터)     | 정상성 경계 모델링    | 비지도 학습보다 높은 정확도, 실용적인 데이터 요구사항 | 초기 레이블 데이터의 품질에 성능이 크게 의존       | 산업 설비 고장 탐지 (정상 작동 데이터는 풍부함) |


이 섹션에서는 일반적인 패러다임을 넘어, 이상 행동 탐지를 구동하는 구체적인 알고리즘과 모델의 기술적 원리를 깊이 있게 살펴봅니다.


이러한 방법들은 종종 이상 탐지의 기초를 형성하며, 다른 복잡한 모델의 기준선(Baseline) 역할을 합니다.

- **가우시안 혼합 모델 (Gaussian Mixture Models, GMM):** 데이터가 여러 개의 가우시안 분포(정규 분포)의 혼합으로 생성되었다고 가정합니다. GMM은 데이터를 여러 클러스터(각각이 하나의 가우시안 분포)로 모델링하고, 어떤 클러스터에서도 생성될 확률이 낮은 데이터 포인트를 이상치로 식별합니다.40 GMM은 K-평균(K-Means)과 달리 구형뿐만 아니라 타원형 클러스터도 모델링할 수 있어 더 유연합니다.42
- **아이솔레이션 포레스트 (Isolation Forest):** 이상치는 '소수이면서 다르다'는 특성 때문에 정상 데이터보다 쉽게 고립(isolate)될 수 있다는 원리에 기반한 트리 기반 알고리즘입니다. 무작위로 데이터를 분할하는 결정 트리들을 여러 개 생성(앙상블)하여, 이상치가 평균적으로 더 적은 분할만으로 고립되는지(즉, 트리의 루트에서 더 짧은 경로를 갖는지)를 측정합니다.15 특히 고차원 데이터셋에서 효과적입니다.
- **Local Outlier Factor (LOF):** 특정 데이터 포인트의 지역적 밀도를 주변 이웃들의 밀도와 비교하는 밀도 기반 알고리즘입니다. 주변에 비해 밀도가 현저히 낮은 지점을 이상치로 탐지하므로, 데이터셋 내에 밀도가 다양한 클러스터가 존재할 때도 효과적으로 작동합니다.4


- **핵심 개념:** 오토인코더(Autoencoder, AE)는 입력 데이터를 출력으로 그대로 복사하도록 학습되는 비지도 신경망입니다. 입력 데이터를 저차원의 '잠재 공간(Latent Space)' 표현으로 압축하는 **인코더(Encoder)**와, 이 압축된 표현으로부터 원본 입력을 복원하는 **디코더(Decoder)**로 구성됩니다.33
- **이상 탐지 메커니즘:** 핵심 아이디어는 오토인코더를 *오직 정상 데이터로만* 훈련시키는 것입니다.19 이렇게 훈련된 모델은 정상적인 데이터 패턴을 매우 잘 재구성하게 됩니다. 그러나 모델이 한 번도 본 적 없는 비정상적인 데이터가 입력되면, 익숙하지 않은 패턴이기 때문에 이를 제대로 복원하지 못합니다. 이로 인해 원본 입력과 복원된 출력 간의 차이인 **재구성 오류(Reconstruction Error)**가 크게 발생하며, 이 오류 값에 특정 임계값을 설정하여 이상을 탐지합니다.19
- **변형 및 발전:**
  - **변이형 오토인코더 (Variational Autoencoder, VAE):** 잠재 공간의 확률적 분포를 학습하는 생성 모델의 일종으로, 새로운 정상 데이터를 생성할 수 있으며 노이즈에 더 강건한 경향이 있습니다.45
  - **앙상블 오토인코더:** 서로 다른 구조를 가진 여러 오토인코더를 결합(앙상블)하여 사용하면, 특히 노이즈가 많은 데이터에서 모델의 안정성과 정확도를 향상시킬 수 있습니다.35
  - **하이브리드 모델:** 오토인코더는 다른 모델과 결합될 수 있습니다. 예를 들어, 시공간적 특성을 가진 데이터 처리를 위해 인코더에 CNN을, 디코더에 LSTM을 사용하는 하이브리드 구조를 만들 수 있습니다.35


- **순차적 데이터의 특성:** 센서 데이터나 주가와 같은 시계열 데이터는 시간적 순서와 의존성을 가지므로, 데이터 포인트의 순서가 매우 중요합니다. 일반적인 모델들은 이러한 시간적 맥락을 포착하기 어렵습니다.
- **순환 신경망 (Recurrent Neural Networks, RNN):** 순차적 데이터를 위해 특별히 설계된 신경망으로, 이전 타임스텝의 출력을 현재 타임스텝의 입력으로 다시 사용하는 순환 구조를 통해 '기억'을 구현합니다.19
- **RNN의 한계 - 기울기 소실 문제:** 표준 RNN은 시퀀스가 길어질수록 오래전의 정보를 기억하는 데 어려움을 겪습니다 (장기 의존성 문제). 이는 '기울기 소실(Vanishing Gradient)' 문제 때문으로, RNN의 기억이 단기적이라는 치명적인 약점으로 작용합니다.19
- **장단기 메모리 (Long Short-Term Memory, LSTM):** 이러한 RNN의 한계를 극복하기 위해 등장한 특별한 종류의 RNN입니다. LSTM은 셀 내부에 정보를 조절하는 '게이트(Gate)' (입력, 출력, 망각 게이트)를 가지고 있어, 중요한 정보는 오래 기억하고 불필요한 정보는 잊어버리는 방식으로 장기 의존성을 효과적으로 학습할 수 있습니다.19
- **이상 탐지를 위한 LSTM-오토인코더:** 이 강력한 조합은 오토인코더의 인코더와 디코더 내부에 LSTM 레이어를 사용합니다. 정상적인 시계열 데이터로 훈련하여 복잡한 시간적 패턴을 학습합니다. 새로운 데이터 시퀀스가 학습된 정상적인 동적 패턴에서 벗어날 때(예: 비정상적인 시간에 발생하는 CPU 사용량 급증), LSTM-오토인코더는 이를 정확하게 재구성하지 못하고 높은 재구성 오류를 발생시켜 이상으로 탐지합니다.19


- **CNN의 한계:** 합성곱 신경망(CNN)은 이미지의 지역적 특징(Local Feature)을 추출하는 데 매우 뛰어나지만, '시야(Field of View)'가 제한적이라는 한계가 있습니다. 이미지의 전반적인 맥락이나 멀리 떨어진 픽셀 간의 관계(Global Context, Long-range Dependency)를 이해하는 데 어려움을 겪습니다.59
- **트랜스포머의 장점 - 셀프 어텐션:** 본래 자연어 처리(NLP) 분야에서 개발된 트랜스포머(Transformer) 모델과 이를 비전 분야에 적용한 비전 트랜스포머(ViT)는 이미지를 여러 개의 패치(Patch) 시퀀스로 취급합니다. 핵심 메커니즘인 **셀프 어텐션(Self-Attention)**은 특정 패치를 처리할 때 다른 모든 패치와의 연관성을 계산하여 중요도를 가중치로 부여합니다. 이를 통해 CNN이 놓치기 쉬운 복잡하고 장거리의 시공간적 관계를 효과적으로 포착할 수 있습니다.59
- **비디오 이상 탐지(VAD) 적용:** 비디오 분석에서 이는 매우 중요합니다. ViT는 한 프레임에 있는 객체와 여러 후속 프레임에 걸친 객체의 위치 및 상태 변화 간의 관계를 분석하여, 시간의 흐름에 따른 움직임, 방향, 복잡한 상호작용을 효과적으로 모델링할 수 있습니다.61 이는 점진적으로 발생하거나 문맥에 따라 달라지는 이상 행동을 탐지하는 데 더 강력한 성능을 보입니다.62
- **하이브리드 접근법:** 연구자들은 ViT를 다른 아키텍처와 결합하여 성능을 더욱 향상시키고 있습니다. 예를 들어, 특징 추출을 위해 ViT를 사용하고, 비디오 내 객체 간의 관계를 더 잘 이해하기 위해 시공간적 어텐션 블록을 추가하는 방식이 연구되고 있습니다.59


- **핵심 개념:** 생성적 적대 신경망(Generative Adversarial Networks, GAN)은 서로 경쟁하는 두 개의 신경망으로 구성됩니다. 하나는 실제 데이터와 유사한 가짜 데이터를 생성하는 **생성자(Generator)**이고, 다른 하나는 실제 데이터와 가짜 데이터를 구별하는 **판별자(Discriminator)**입니다.45
- **이상 탐지 메커니즘:** GAN을 정상 데이터로만 훈련시키면, 생성자는 실제와 같은 '정상' 데이터를 만드는 데 특화되고, 판별자는 정상 데이터의 분포에서 벗어나는 모든 것을 '가짜'로 식별하는 전문가가 됩니다. 새로운 데이터가 입력되었을 때 판별자가 이를 '가짜'(즉, 학습된 정상 분포에 속하지 않음)로 판단하면, 해당 데이터는 이상으로 간주됩니다. 이 접근법은 데이터 불균형 문제를 완화하기 위해 합성 데이터를 생성하는 데에도 활용될 수 있습니다.45

이러한 기술의 발전은 중요한 흐름을 보여줍니다. 이상 탐지 시스템은 더 이상 범용 알고리즘에 의존하지 않고, 데이터의 근본적인 특성(순차성, 공간성 등)에 맞춰 고도로 전문화된 하이브리드 아키텍처를 채택하고 있습니다. 초기 머신러닝 방법들은 데이터를 특징 공간의 점으로 취급했지만, 딥러닝의 발전은 데이터 유형에 특화된 아키텍처를 낳았습니다. 시계열 데이터에는 시간적 의존성을 모델링하는 LSTM이, 이미지와 비디오 데이터에는 전역적 맥락을 이해하는 트랜스포머가 표준으로 자리 잡고 있습니다.19 오토인코더는 비지도 탐지를 위한 강력한 프레임워크 역할을 하지만, 그 내부 구성 요소는 이제 이러한 특화된 모델들로 대체되고 있습니다(예: LSTM-AE, Transformer-AE).56 이는 "어떤 알고리즘이 최고인가?"라는 질문에서 "내 데이터의 구조를 가장 잘 표현하는 아키텍처는 무엇인가?"라는 질문으로 진화했음을 보여주는, 해당 분야의 성숙을 의미합니다.


이 섹션에서는 앞서 논의된 기술적 원리들이 실제 산업 현장에서 어떻게 구체적인 가치를 창출하고 있는지, 주요 분야별 사례를 통해 살펴봅니다.


- **문제점:** 금융 사기 수법은 날로 지능화되고 있으며, 기존의 규칙 기반(Rule-based) 시스템은 변화하는 패턴에 신속하게 대응하기 어렵습니다.65
- **AI 기반 해결책:** AI FDS는 거래 내역, 사용자 행동, 접속 위치, 기기 정보 등 방대한 데이터를 실시간으로 분석하여 미세한 사기 징후를 포착합니다.12
- **주요 사례:**
  - **KB국민카드:** 위험도가 높은 해외 거래에 AI를 우선 적용한 후, 국내 거래로 확대하여 사용자의 접속 정보와 같은 비거래 데이터까지 분석에 활용하고 있습니다.12
  - **토스뱅크(Toss Bank):** AI를 활용하여 새롭게 등장하는 사기 패턴을 탐지하고, 의심스러운 로그인 시도 시 영상 통화와 같은 추가 인증을 요구하여 명의 도용을 방지합니다.12
  - **JP Morgan Chase:** 글로벌 금융사의 대표적인 성공 사례로, AI 모델 도입 후 거짓 양성(오탐)을 50% 줄이고 사기 탐지율을 25% 향상시켜 보안과 고객 경험을 동시에 개선했습니다.66
- **핵심 동향:** 단순한 거래 모니터링을 넘어, 복잡하고 다차원적인 데이터 속에서 미묘한 패턴을 찾아내는 AI의 능력을 활용하여 사용자의 전반적인 행동을 분석하는 방향으로 진화하고 있습니다.12


- **문제점:** 예측 불가능한 설비 고장은 막대한 생산 중단 비용을 초래하고, 전체 공급망에 연쇄적인 문제를 일으킬 수 있습니다.6
- **AI 기반 해결책 (예지보전):** 설비에 부착된 센서(진동, 온도, 전류 등)로부터 실시간 데이터를 수집하고 AI가 이를 분석하여 고장이 발생하기 *전에* 이상 징후를 예측합니다.6 이를 통해 계획적인 유지보수가 가능해져 가동 중단을 최소화합니다.
- **사례 및 효과:**
  - **S사 사례:** AI 예지보전 및 머신비전 시스템 도입 후, **설비 가동 중단 시간을 40% 절감**하고 **제품 불량률을 70% 이상 감소**시키는 성과를 거두었습니다.6
  - **모터센스(MotorSense) 솔루션:** 자동화 창고의 스태커 크레인(Stacker Crane)이나 생산 라인의 다관절 로봇에 적용되어 모터의 상태를 실시간으로 모니터링하고 치명적인 고장을 예방합니다.72
  - **주요 모델:** 정상 상태의 센서 데이터로 학습하여 편차를 감지하는 오토인코더 기반 모델이 널리 사용되며 70, 시계열 데이터의 특성을 분석하기 위해 RNN과 LSTM도 활용됩니다.6


- **문제점:** 수천 개의 CCTV 영상을 사람이 24시간 수동으로 감시하는 것은 비효율적이고 인적 오류에 취약하여 긴급 상황 발생 시 대응이 늦어집니다.73
- **AI 기반 해결책:** AI 영상 분석 기술이 실시간으로 CCTV 영상을 분석하여 침입, 폭행, 군중 밀집 등 특정 이상 행동을 자동으로 탐지하고 경보를 발생시킵니다.
- **사례 (한국딥러닝):**
  - **탐지 이벤트:** 침입, 쓰러짐, 싸움, 군중 밀집, 침수 등 6가지 유형의 안전사고.73
  - **적용 AI 모델:** 영상 프레임 분석에 특화된 **PEL4VAD**와 특징 편차를 측정하는 **BN-WVAD** 모델을 활용했습니다.73
  - **정량적 효과:**
    - **대응 시간:** 평균 8-10분에서 **평균 2분 이하**로 대폭 단축되었습니다.
    - **골든타임 확보율:** **75% 이상 향상**되었습니다.
    - **관제 효율:** 1인당 감시 화면 수가 30대에서 10대로 줄어, 관제 효율이 **60% 이상 개선**되었습니다.74


- **사이버 보안:** 네트워크 침입 탐지 시스템(NIDS)에 AI를 적용하여 네트워크 트래픽을 분석하고, DDoS 공격이나 포트 스캐닝과 같은 알려진 위협뿐만 아니라 이전에 알려지지 않은 새로운 유형의 사이버 공격까지 탐지합니다.9 한국과학기술정보연구원(KISTI)과 같은 기관에서도 이러한 목적의 AI 알고리즘을 개발하고 있습니다.77
- **헬스케어:** 웨어러블 기기나 의료 장비에서 수집되는 환자의 생체 데이터를 실시간으로 모니터링하여 심박수 이상과 같은 건강 문제의 조기 징후를 발견합니다.9 또한, X-ray나 MRI와 같은 의료 영상에서 사람이 놓치기 쉬운 미세한 이상을 찾아내 질병을 조기에 진단하는 데에도 활용됩니다.9


| 산업/응용 분야              | 핵심 지표                | 정량적 개선 효과 | 출처/사례 연구                  |
| --------------------------- | ------------------------ | ---------------- | ------------------------------- |
| **공공 안전 (지능형 CCTV)** | 대응 시간 단축           | 75% ~ 80% 감소   | 한국딥러닝 74                   |
| **제조업 (예지보전)**       | 설비 가동 중단 시간 감소 | 40% 감소         | S사 사례 6                      |
| **제조업 (예지보전)**       | 제품 불량률 감소         | 70% 이상 감소    | S사 사례 6                      |
| **금융 (FDS)**              | 사기 탐지율 증가         | 25% 증가         | JP Morgan Chase 66              |
| **금융 (FDS)**              | 거짓 양성(오탐) 감소     | 50% ~ 97% 감소   | JP Morgan Chase 66, FraudNet 78 |


이 마지막 장에서는 이상 행동 탐지 AI가 직면한 주요 기술적 난제들을 분석하고, 이 분야의 흥미로운 미래 발전 방향을 조망합니다.


- **데이터 불균형 문제:** 이상 현상은 정의상 드물게 발생합니다. 이러한 클래스 불균형은 모델 훈련에 큰 어려움을 줍니다. 모델이 단순히 다수 클래스(정상)로만 예측해도 높은 정확도를 기록할 수 있어, 소수 클래스(이상)를 제대로 탐지하지 못하는 문제가 발생합니다.26
- **불균형 해결 방안:**
  - **데이터 레벨 접근법 (리샘플링):**
    - **오버샘플링 (Oversampling):** 소수 클래스(이상) 데이터의 수를 늘리는 방법입니다. 단순 복제 방식도 있지만, **SMOTE(Synthetic Minority Over-sampling Technique)**와 같이 기존 소수 데이터를 기반으로 새로운 합성 데이터를 생성하는 고급 기법이 널리 사용됩니다.26
    - **언더샘플링 (Undersampling):** 다수 클래스(정상) 데이터의 수를 줄이는 방법입니다. **토멕 링크(Tomek Links)**나 **ENN(Edited Nearest Neighbours)**과 같은 기법은 클래스 간 경계에 위치한 다수 클래스 데이터를 제거하여 결정 경계를 더 명확하게 만듭니다.26
  - **알고리즘 레벨 접근법:**
    - **비용 민감 학습 (Cost-Sensitive Learning):** 소수 클래스를 잘못 분류했을 때 더 큰 비용(패널티)을 부과하여 모델이 소수 클래스에 더 집중하도록 유도하는 방식입니다.26
    - **단일 클래스 분류 문제로 전환:** 비지도 또는 준지도 학습 방식을 채택하여 정상 클래스 자체를 모델링하는 데 집중함으로써 데이터 불균형 문제를 근본적으로 회피합니다.82
- **오탐과 미탐의 상충 관계:**
  - **거짓 양성 (False Positive, 오탐):** 정상 이벤트를 이상으로 잘못 탐지하는 경우입니다. 이는 '경보 피로(Alert Fatigue)'를 유발하여, 보안팀이 수많은 가짜 경보에 지쳐 실제 위협을 놓치게 만들 수 있습니다.3
  - **거짓 음성 (False Negative, 미탐):** 실제 이상 현상을 탐지하지 못하고 놓치는 경우입니다. 이는 보안 침해, 재정적 손실, 안전사고로 이어질 수 있는 가장 치명적인 오류입니다.18
  - **해결책 - 동적 임계값 및 문맥 분석:** AI 기반 솔루션은 시간대나 계절성과 같은 문맥을 고려하여 경보 임계값을 동적으로 조정하고, 여러 개의 약한 신호를 종합하여 하나의 신뢰도 높은 경보를 생성함으로써, 미탐을 늘리지 않으면서 오탐을 획기적으로 줄일 수 있습니다.85


- **'블랙박스' 문제:** 많은 고성능 딥러닝 모델은 그 결정 과정을 이해하기 어려운 '블랙박스'와 같습니다. 모델이 예측 결과를 내놓아도 *왜* 그런 결정을 내렸는지 설명하지 못합니다. 이러한 투명성 부족은 책임성이 중요한 금융, 의료, 법률과 같은 고위험 분야에서 AI 도입의 큰 걸림돌이 됩니다.88
- **XAI의 역할:** 설명가능 AI(Explainable AI)는 모델의 결정을 사람이 이해할 수 있는 형태로 만드는 것을 목표로 합니다. 이상 탐지 분야에서 이는 특정 데이터가 *어떤 특징*이나 *어떤 패턴* 때문에 이상으로 탐지되었는지 설명하는 것을 의미합니다.88
- **기대 효과:** XAI는 모델에 대한 신뢰를 구축하고, 모델 디버깅을 도우며, 편향을 탐지하여 공정성을 보장하고, 나아가 규제 요건을 충족하는 데 필수적입니다.88
- **연구 동향:** 최근 연구는 단순히 정확도를 높이는 것을 넘어 설명가능성에 초점을 맞추고 있습니다. LIME, SHAP과 같은 기술들이 이상 탐지에 맞게 변형되고 있으며, 모델 자체가 해석 가능한 새로운 아키텍처도 개발되고 있습니다.93 이는 '무엇'을 탐지했는지를 넘어 '왜' 탐지했는지로 나아가는 중요한 전환입니다.94


- **감시의 딜레마:** 이상 행동 탐지는 본질적으로 CCTV나 사용자 데이터에 대한 모니터링을 포함합니다. 이는 대규모 감시, 데이터 오용, 익명성 및 개인의 자유 침해와 같은 심각한 프라이버시 문제를 야기합니다.95
- **주요 윤리적 위험:**
  - **데이터 노출:** 시스템은 금융, 의료 등 민감한 데이터에 접근해야 하며, 이는 데이터 유출의 위험을 내포합니다.95
  - **알고리즘 편향:** 만약 학습 데이터가 과거의 사회적 편견(예: 특정 인종에 대한 과잉 단속)을 반영한다면, AI는 이러한 차별을 영속시키고 증폭시킬 수 있습니다.16
  - **동의 및 투명성 부족:** 개인들은 자신의 데이터가 어떻게 수집되고 활용되는지에 대한 충분한 정보나 동의 없이 감시받는 경우가 많습니다.99
- **완화 전략:**
  - **기술적 방안:** 필요한 데이터만 수집하는 **데이터 최소화**, **암호화**, **익명화** 처리, 그리고 데이터를 중앙 서버로 보내지 않고 각 기기에서 로컬로 모델을 학습하는 **연합 학습(Federated Learning)** 등이 있습니다.95
  - **정책 및 거버넌스:** GDPR과 같은 강력한 데이터 보호법 제정, 투명성과 책임성 요구, AI 시스템에 대한 정기적인 윤리 감사 등이 필요합니다.16


- **실시간 탐지를 위한 엣지 AI:** 데이터를 중앙 클라우드로 전송하는 대신, 스마트 카메라나 센서와 같은 엣지 디바이스에서 직접 처리하는 방식입니다.
  - **장점:** 데이터 전송에 따른 지연 시간을 획기적으로 줄여 진정한 실시간 대응을 가능하게 하고, 데이터를 로컬에 유지하여 프라이버시를 강화하며, 네트워크 대역폭 요구량을 감소시킵니다.104
  - **예시:** 스마트홈 시스템이 라즈베리파이와 같은 엣지 디바이스에서 직접 아이솔레이션 포레스트(가벼운 1차 필터링)와 LSTM-오토인코더(심층 분석)를 결합한 하이브리드 모델을 실행하여 이상을 탐지합니다.104
- **멀티모달 AI:** 미래의 강력한 탐지 시스템은 여러 소스(양식, Modality)의 데이터를 융합하는 방향으로 나아갈 것입니다.
  - **개념:** 비디오, 오디오, 텍스트, 센서 데이터를 결합하여 상황에 대한 더 완전하고 풍부한 맥락을 이해하는 기술입니다.
  - **예시:** 로보틱스 분야에서 시각 데이터(카메라), 힘 데이터(센서), 언어 모델을 결합하여 로봇 자체의 하드웨어 결함과 예기치 않은 장애물과 같은 환경적 이상을 동시에 탐지합니다.107 '전문가 혼합(Mixture-of-Experts)' 프레임워크는 특정 상황에 가장 신뢰할 수 있는 데이터 소스를 동적으로 선택하여 탐지 정확도를 높입니다.107

결론적으로, 이상 행동 탐지 AI의 미래 발전은 **성능(정확도, 저지연), 프라이버시(데이터 보안), 설명가능성(투명성, 신뢰)** 이라는 세 가지 핵심 요소 사이의 근본적인 상충 관계(Trilemma)를 어떻게 해결하느냐에 달려 있습니다. 이 중 하나를 극대화하면 다른 요소에 부정적인 영향을 미칠 수 있습니다. 예를 들어, 최고의 성능을 위해 모든 원본 데이터를 사용하면 프라이버시가 침해될 수 있고 95, 가장 성능이 좋은 블랙박스 모델은 설명가능성이 떨어집니다.88 따라서 미래의 가장 진보된 솔루션은 단일 차원을 극대화하기보다는, 특정 비즈니스, 윤리, 규제 환경에 맞춰 이 세 축 사이에서 최적의 균형점을 찾는 시스템이 될 것입니다.


본 보고서는 단순한 규칙 기반 시스템에서 출발하여 복잡하고, 문맥을 이해하며, 미래를 예측하는 AI로 진화해 온 이상 행동 탐지 기술의 여정을 종합적으로 살펴보았습니다. 이 기술은 이제 디지털 경제의 보안, 효율성, 그리고 신뢰를 지탱하는 핵심 기반 기술로 자리매김했습니다.

앞으로의 과제는 단순히 더 강력한 알고리즘을 개발하는 것을 넘어, **신뢰할 수 있고, 윤리적이며, 설명 가능한** 시스템을 구축하는 것입니다.88 XAI 기술의 통합, 강력한 프라이버시 보호 기술의 적용, 그리고 인간이 최종 의사결정 과정에 참여하는 거버넌스 체계의 확립이 무엇보다 중요해질 것입니다.

따라서 이 기술을 도입하고자 하는 조직은 명확한 비즈니스 문제 정의에서 출발하여 14, 고품질 데이터 확보에 투자하고 13, 특정 활용 사례에 가장 적합한 학습 패러다임과 아키텍처를 신중하게 선택하며, 처음부터 윤리 및 프라이버시 문제를 선제적으로 해결하는 전략적 접근을 취해야 합니다.16 최종 목표는 단순히 지능적인 시스템을 넘어, 사회적으로 책임감 있는 시스템을 구축하는 것이 되어야 할 것입니다.


1. 이상 탐지란 무엇인가요? - AWS, accessed July 3, 2025, https://aws.amazon.com/ko/what-is/anomaly-detection/
2. (PDF) Anomaly Detection with Artificial Intelligence - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/390925171_Anomaly_Detection_with_Artificial_Intelligence
3. Behavior Anomaly Detection: Techniques & Best Practices - Exabeam, accessed July 3, 2025, https://www.exabeam.com/explainers/ueba/behavior-anomaly-detection-techniques-and-best-practices/
4. AI in anomaly detection: Use cases, methods, algorithms and solution - LeewayHertz, accessed July 3, 2025, https://www.leewayhertz.com/ai-in-anomaly-detection/
5. Anomaly Detection Using AI & Machine Learning - Nile Secure, accessed July 3, 2025, https://nilesecure.com/ai-networking/anomaly-detection-ai
6. AI 예지보전으로 산업안전 혁신한 사례 - VLM OCR 기술력 독보적 1위 ..., accessed July 3, 2025, [https://www.koreadeep.com/blog/ai-%EC%98%88%EC%A7%80%EB%B3%B4%EC%A0%84](https://www.koreadeep.com/blog/ai-예지보전)
7. 이상 현상 감지란 무엇인가요? | IBM, accessed July 3, 2025, https://www.ibm.com/kr-ko/topics/anomaly-detection
8. Anomaly detection - Wikipedia, accessed July 3, 2025, https://en.wikipedia.org/wiki/Anomaly_detection
9. What is Anomaly Detection| Machine learning used cases - Datrics AI, accessed July 3, 2025, https://www.datrics.ai/articles/anomaly-detection-definition-best-practices-and-use-cases
10. 이상 징후 탐지 솔루션, Zenius AI의 주요기능과 특장점 - 브레인즈컴퍼니, accessed July 3, 2025, https://www.brainz.co.kr/Various-Topics/view/id/383
11. 이상 탐지: AI를 통한 사용자 행동 이상 탐지 시스템으로 보안을 강화하고 사기 방지를 효과적으로 실현하는 전략, accessed July 3, 2025, [https://epart.com/%EC%9D%B4%EC%83%81-%ED%83%90%EC%A7%80-ai%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%82%AC%EC%9A%A9%EC%9E%90-%ED%96%89%EB%8F%99-%EC%9D%B4%EC%83%81-%ED%83%90%EC%A7%80-%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9C%BC%EB%A1%9C/](https://epart.com/이상-탐지-ai를-통한-사용자-행동-이상-탐지-시스템으로/)
12. '꼼짝마!'...AI 접목 금융 이상거래 탐지 '한 끗' 차별화 - 지디넷코리아, accessed July 3, 2025, https://zdnet.co.kr/view/?no=20240415092812
13. AI-Powered Anomaly Detection 2024 Ultimate Guide | Boost Efficiency - Rapid Innovation, accessed July 3, 2025, https://www.rapidinnovation.io/post/ai-in-anomaly-detection-for-businesses
14. [All Around AI 2편] AI 알고리즘의 기본 개념과 작동 원리 - SK하이닉스 뉴스룸, accessed July 3, 2025, https://news.skhynix.co.kr/all-around-ai-2/
15. Anomaly Detection in Data: Core Principles - Eyer.ai, accessed July 3, 2025, https://www.eyer.ai/blog/anomaly-detection-in-data-core-principles/
16. The Dark Side of AI: Top Data Security Threats and How to Prevent Them - Zylo, accessed July 3, 2025, https://zylo.com/blog/ai-data-security/
17. What Is Anomaly Detection? | IBM, accessed July 3, 2025, https://www.ibm.com/think/topics/anomaly-detection
18. AI는 어떻게 '사기 거래'를 잡아낼까? - LG CNS, accessed July 3, 2025, https://www.lgcns.com/blog/cns-tech/ai-data/6844/
19. Anomaly Detection with LSTM Autoencoders in Time Series Data ..., accessed July 3, 2025, https://nearshore-it.eu/articles/anomaly-detection-with-lstm/
20. 지도 학습과 비지도 학습 | Google Cloud, accessed July 3, 2025, https://cloud.google.com/discover/supervised-vs-unsupervised-learning?hl=ko
21. 기계학습의 종류 : 지도학습, 비지도학습, 준지도학습, 강화학습 - Allen's 데이터 맛집, accessed July 3, 2025, [https://allensdatablog.tistory.com/entry/%EA%B8%B0%EA%B3%84%ED%95%99%EC%8A%B5%EC%9D%98-%EC%A2%85%EB%A5%98-%EC%A7%80%EB%8F%84%ED%95%99%EC%8A%B5-%EB%B9%84%EC%A7%80%EB%8F%84%ED%95%99%EC%8A%B5-%EC%A4%80%EC%A7%80%EB%8F%84%ED%95%99%EC%8A%B5-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5](https://allensdatablog.tistory.com/entry/기계학습의-종류-지도학습-비지도학습-준지도학습-강화학습)
22. [인공지능] 지도학습, 비지도학습, 강화학습 - 삶은 확률의 구름, accessed July 3, 2025, https://ebbnflow.tistory.com/165
23. Pro's and con's of supervised vs unsupervised algorithms for scalable anomaly detection, accessed July 3, 2025, https://www.eyer.ai/blog/pros-and-cons-of-supervised-vs-unsupervised-algorithms-for-scalable-anomaly-detection/
24. What's the difference between supervised and unsupervised machine learning - AWS, accessed July 3, 2025, https://aws.amazon.com/compare/the-difference-between-machine-learning-supervised-and-unsupervised/
25. (PDF) Comparison of Supervised, Semi-supervised and Unsupervised Learning Methods in Network Intrusion Detection System (NIDS) Application - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/321977517_Comparison_of_Supervised_Semi-supervised_and_Unsupervised_Learning_Methods_in_Network_Intrusion_Detection_System_NIDS_Application
26. 10 Techniques to handle imbalance class in ... - Analytics Vidhya, accessed July 3, 2025, https://www.analyticsvidhya.com/blog/2020/07/10-techniques-to-deal-with-class-imbalance-in-machine-learning/
27. Master AI Anomaly Detection: The Definitive Guide | SmartDev, accessed July 3, 2025, https://smartdev.com/ai-anomaly-detection/
28. Artificial Intelligence's role in anomaly detection - Eyer.ai, accessed July 3, 2025, https://www.eyer.ai/blog/artificial-intelligences-role-in-anomaly-detection/
29. 지도 학습 vs 비지도 학습 (Supervised learning VS Unsupervised learning) - CAI - 티스토리, accessed July 3, 2025, https://kjh-ai-blog.tistory.com/12
30. 이상치 탐지(Anomaly Detection), AI를 활용한 사고 발생 예방 기술! - 이랜서, accessed July 3, 2025, https://www.elancer.co.kr/blog/detail/131
31. [데이터전처리] Outlier(이상치/이상값/특이값/특이치 등) 탐지 방법(detection method) : 4. DBSCAN 알고리즘 with 파이썬, accessed July 3, 2025, https://claryk.tistory.com/7
32. 이상 탐지 넌 무엇이냐? #2, accessed July 3, 2025, https://datanetworkanalysis.github.io/2020/03/03/understanding_outlier2
33. 머신러닝 기반 메트릭 데이터 이상탐지 – 기술이야기 - 브레인즈컴퍼니, accessed July 3, 2025, https://www.brainz.co.kr/tech-story/view/id/54
34. Anomaly Detection with Autoencoders - Kaggle, accessed July 3, 2025, https://www.kaggle.com/code/harshsingh2209/anomaly-detection-with-autoencoders
35. Deep Learning for Anomaly Detection: A Survey - velog, accessed July 3, 2025, https://velog.io/@kyyle/Deep-Learning-for-Anomaly-Detection-A-Survey
36. [머신러닝 시스템의 종류] 지도학습/비지도학습/준지도학습/강화학습 - 봉식이와 캔따개, accessed July 3, 2025, https://bong-sik.tistory.com/29
37. What is semi-supervised anomaly detection? - Milvus, accessed July 3, 2025, https://milvus.io/ai-quick-reference/what-is-semisupervised-anomaly-detection
38. Semi-Supervised Learning: The Bridge Between Supervised and Unsupervised Learning | by Dr. Fatih Hattatoglu | Academy Team | Medium, accessed July 3, 2025, https://medium.com/academy-team/semi-supervised-learning-the-bridge-between-supervised-and-unsupervised-learning-a4c9942b814b
39. Comparison of Supervised, Unsupervised, Semi-Supervised and Reinforcement - IFoA Data Science, accessed July 3, 2025, https://ifoadatascienceresearch.github.io/tutorial/comparison/
40. docs.edgeimpulse.com, accessed July 3, 2025, [https://docs.edgeimpulse.com/docs/edge-impulse-studio/learning-blocks/anomaly-detection-gmm#:~:text=Each%20Gaussian%20component%20in%20the,data%20points%20with%20low%20probabilities.](https://docs.edgeimpulse.com/docs/edge-impulse-studio/learning-blocks/anomaly-detection-gmm#:~:text=Each Gaussian component in the,data points with low probabilities.)
41. A Network Traffic Anomaly Detection Method Based on Gaussian Mixture Model - MDPI, accessed July 3, 2025, https://www.mdpi.com/2079-9292/12/6/1397
42. Anomaly detection (GMM) - Edge Impulse Documentation, accessed July 3, 2025, https://docs.edgeimpulse.com/docs/edge-impulse-studio/learning-blocks/anomaly-detection-gmm
43. Gaussian Mixture Model - GeeksforGeeks, accessed July 3, 2025, https://www.geeksforgeeks.org/machine-learning/gaussian-mixture-model/
44. 이상치(Outlier) 판단 기준, accessed July 3, 2025, https://esj205.oopy.io/72782730-23e4-43cf-8799-f3cdcbcb57b9
45. 생성형 AI(Generative AI)란 무엇을 의미하나요? - Databricks, accessed July 3, 2025, https://www.databricks.com/kr/discover/generative-ai
46. 6강) AI를 활용한 사기 검출 밑 탐지- auto-encoder 예시 - Kaggle, accessed July 3, 2025, https://www.kaggle.com/code/linakeepgoing/6-ai-auto-encoder
47. Auto Encoder Decoder-Based Anomaly Detection with the Lakehouse Paradigm - YouTube, accessed July 3, 2025, https://www.youtube.com/watch?v=v8dzXskvF6c
48. Complete Guide to Anomaly Detection with AutoEncoders using Tensorflow, accessed July 3, 2025, https://www.analyticsvidhya.com/blog/2022/01/complete-guide-to-anomaly-detection-with-autoencoders-using-tensorflow/
49. 3. Autoencoders and anomaly detection - Reproducible Machine Learning for Credit Card Fraud detection - Practical handbook, accessed July 3, 2025, https://fraud-detection-handbook.github.io/fraud-detection-handbook/Chapter_7_DeepLearning/Autoencoders.html
50. Demystifying Neural Networks: Anomaly Detection with AutoEncoder | by Dagang Wei, accessed July 3, 2025, https://medium.com/@weidagang/demystifying-anomaly-detection-with-autoencoder-neural-networks-1e235840d879
51. [D] Struggling with Autoencoder-Based Anomaly Detection for Fraud Detection – Need Guidance : r/MachineLearning - Reddit, accessed July 3, 2025, https://www.reddit.com/r/MachineLearning/comments/1gl92zm/d_struggling_with_autoencoderbased_anomaly/
52. 시계열 이상탐지 (1) 배경 지식 - 복습 블로그, accessed July 3, 2025, https://boksup.tistory.com/70
53. 순환 신경망(RNN)이란 무엇인가요? - IBM, accessed July 3, 2025, https://www.ibm.com/kr-ko/think/topics/recurrent-neural-networks
54. [시계열] 주가 예측을 위한 RNN/LSTM/GRU 기술적 가이드 (번역) - 다이엔 스페이스, accessed July 3, 2025, https://diane-space.tistory.com/331
55. 시계열 데이터를 위한 LSTM 모델, accessed July 3, 2025, [https://allensdatablog.tistory.com/entry/%EC%8B%9C%EA%B3%84%EC%97%B4-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%A5%BC-%EC%9C%84%ED%95%9C-LSTM-%EB%AA%A8%EB%8D%B8](https://allensdatablog.tistory.com/entry/시계열-데이터를-위한-LSTM-모델)
56. Anomaly Detection Using an Ensemble of Multi-Point LSTMs - PMC, accessed July 3, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10670439/
57. Anomaly Detection in Time Series Data using LSTM Autoencoders | by Zhong Hong, accessed July 3, 2025, https://medium.com/@zhonghong9998/anomaly-detection-in-time-series-data-using-lstm-autoencoders-51fd14946fa3
58. LSTM AE를 이용한 시계열 데이터 이상 탐지 - (1) 개요 - velog, accessed July 3, 2025, [https://velog.io/@jonghne/LSTM-AE%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%8B%9C%EA%B3%84%EC%97%B4-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%9D%B4%EC%83%81-%ED%83%90%EC%A7%80-1-%EA%B0%9C%EC%9A%94](https://velog.io/@jonghne/LSTM-AE를-이용한-시계열-데이터-이상-탐지-1-개요)
59. Enhancing Video Anomaly Detection Using a Transformer ... - MDPI, accessed July 3, 2025, https://www.mdpi.com/1999-4893/17/7/286
60. An overview of transformers for video anomaly detection - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/391698146_An_overview_of_transformers_for_video_anomaly_detection
61. (PDF) TransAnomaly: Video Anomaly Detection Using Video Vision Transformer, accessed July 3, 2025, https://www.researchgate.net/publication/354227217_TransAnomaly_Video_Anomaly_Detection_Using_Video_Vision_Transformer
62. MT-CMVAD: A Multi-Modal Transformer Framework for Cross-Modal Video Anomaly Detection - MDPI, accessed July 3, 2025, https://www.mdpi.com/2076-3417/15/12/6773
63. 인공지능 기반 이상감지 기술 - 소프트웨어정책연구소, accessed July 3, 2025, [https://spri.kr/posts/view/23193?code=data_all&study_type=&board_type=industry_trend](https://spri.kr/posts/view/23193?code=data_all&study_type&board_type=industry_trend)
64. DS Graduate Research Presentation- Transformer-based Anomaly Detection - YouTube, accessed July 3, 2025, https://www.youtube.com/watch?v=R6w1qmIG52Y
65. 킁킁!킁! 어디서 사기 냄새 안나요? : FDS 시스템에 AI 적용하기 - 카카오뱅크 기술블로그, accessed July 3, 2025, https://tech.kakaobank.com/posts/2310-applying-ai-into-fds-system/
66. AI가 금융 사기를 혁신적으로 막아내다: JP Morgan Chase의 성공 사례 - AI넷, accessed July 3, 2025, [https://m.ainet.link/a.html?uid=17641&sc=](https://m.ainet.link/a.html?uid=17641&sc)
67. KB국민카드, 인공지능(AI) 기반 이상금융거래 탐지 시스템 고도화 - Korea IT Times, accessed July 3, 2025, https://www.koreaittimes.com/news/articleView.html?idxno=132057
68. 시중은행, AI로 금융사기 잡는다... FDS 고도화 속도 - IT조선, accessed July 3, 2025, https://it.chosun.com/news/articleView.html?idxno=2023092141612
69. "멈추기 전에 예측한다"...제조업 게임체인저 'AI 예지보전' - 헬로티, accessed July 3, 2025, https://www.hellot.net/news/article.html?no=101874
70. Predictive Maintenance | Real-World AI - 마키나락스, accessed July 3, 2025, https://www.makinarocks.ai/real-world-ai/predictive-maintenance/
71. 스마트팩토리 예측 유지보수(예지보전)란 무엇이고, 어떤 이점이 있나요? - AHHA Labs, accessed July 3, 2025, https://ahha.ai/2024/01/08/predictive_maintenance/
72. AI 기반 산업용 모터 고장 예측 솔루션, accessed July 3, 2025, https://imgnew.megazone.com/2021/09/210915-epapyrus.pdf
73. AI 지능형 CCTV 시스템 구축 사례, accessed July 3, 2025, https://brunch.co.kr/@victoriachoi/59
74. AI 지능형 CCTV 시스템 구축 사례 - VLM OCR 기술력 독보적 1위 ..., accessed July 3, 2025, https://www.koreadeep.com/blog/ai-cctv
75. AI 기반 이상행위 및 위협징후 탐지에 대한 동향 - ManuscriptLink, accessed July 3, 2025, https://www.manuscriptlink.com/society/kips/conference/ack2024/file/downloadSoConfManuscript/abs/KIPS_C2024B0287
76. 인공지능 기반 침입 탐지 시스템 - 중부대학교, accessed July 3, 2025, http://isweb.joongbu.ac.kr/~jbuis/2022/report-2022-4.pdf
77. KISTI "AI로 연구망 침입 탐지" - 지디넷코리아, accessed July 3, 2025, https://zdnet.co.kr/view/?no=20190320110557
78. FraudNet - AI Fraud Detection for Enterprises, accessed July 3, 2025, https://www.fraud.net/
79. 불균형 데이터 처리, 언더 샘플링, 오버 샘플링 - 차곡차곡 - 티스토리, accessed July 3, 2025, https://eewjddms.tistory.com/87
80. Anomaly Detection Handbook: Dealing with unbalanced data and anomaly detection algorithms | by Yassine Lazaar | Medium, accessed July 3, 2025, https://medium.com/@yassin.lazar/anomaly-detection-handbook-dealing-with-unbalanced-data-and-anomaly-detection-algorithms-1821868d2991
81. [논문]머신러닝을 위한 불균형 데이터 처리 방법 : 샘플링을 위주로 - 한국과학기술정보연구원, accessed July 3, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO201900937327481
82. Top 5 Strategies to Handle Imbalanced Data Classification - Numerous.ai, accessed July 3, 2025, https://numerous.ai/blog/imbalanced-data-classification
83. How does anomaly detection handle imbalanced datasets? - Milvus, accessed July 3, 2025, https://milvus.io/ai-quick-reference/how-does-anomaly-detection-handle-imbalanced-datasets
84. AI 할루시네이션이란 무엇인가요? - Google Cloud, accessed July 3, 2025, https://cloud.google.com/discover/what-are-ai-hallucinations?hl=ko
85. How Do You Reduce False Positives and False Negatives? - Anodot, accessed July 3, 2025, https://www.anodot.com/learning-center/false-positive-and-false-negative/
86. Combating the Challenges of False Positives in AI-Driven Anomaly Detection Systems and Enhancing Data Security in the Cloud - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/381303129_Combating_the_Challenges_of_False_Positives_in_AI-Driven_Anomaly_Detection_Systems_and_Enhancing_Data_Security_in_the_Cloud
87. 탐지 구분 ( 정탐, 오탐, 미탐 ) - Maker's security - 티스토리, accessed July 3, 2025, https://maker5587.tistory.com/9
88. The Rise of Explainable AI (XAI): A Critical Trend for 2025 and Beyond - AlgoAnalytics, accessed July 3, 2025, https://blog.algoanalytics.com/2025/05/05/the-rise-of-explainable-ai-xai-a-critical-trend-for-2025-and-beyond/
89. a survey on XAI-based anomaly detection for IoT - TÜBİTAK Academic Journals, accessed July 3, 2025, https://journals.tubitak.gov.tr/cgi/viewcontent.cgi?article=4075&context=elektrik
90. A Survey on Explainable Anomaly Detection - arXiv, accessed July 3, 2025, https://arxiv.org/pdf/2210.06959
91. [보고서]설명가능한 인공지능(explainable AI, XAI)의 필요성과 연구 동향, accessed July 3, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchReport.do?cn=KOSEN000000000001223
92. [시장동향] 지능화/고도화되는 사이버 위협에 AI 기술로 대응한다 - 컴퓨터월드, accessed July 3, 2025, https://www.comworld.co.kr/news/articleView.html?idxno=50333
93. (PDF) XAI-IoT: An Explainable AI Framework for Enhancing Anomaly Detection in IoT Systems - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/380672461_XAI-IoT_An_Explainable_AI_Framework_for_Enhancing_Anomaly_Detection_in_IoT_Systems
94. 신뢰할 수 있는 인공지능? Causal AI 동향 - 브런치스토리, accessed July 3, 2025, https://brunch.co.kr/@95923c1769ee43d/2
95. What are the privacy concerns in anomaly detection? - Milvus, accessed July 3, 2025, https://milvus.io/ai-quick-reference/what-are-the-privacy-concerns-in-anomaly-detection
96. What are the privacy concerns in anomaly detection? - Zilliz Vector Database, accessed July 3, 2025, https://zilliz.com/ai-faq/what-are-the-privacy-concerns-in-anomaly-detection
97. The Ethics of AI Surveillance - Number Analytics, accessed July 3, 2025, https://www.numberanalytics.com/blog/ethics-of-ai-surveillance
98. Ethical Considerations in AI-Powered Surveillance Systems: Balancing Security and Privacy in the Digital Age - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/385881507_Ethical_Considerations_in_AI-Powered_Surveillance_Systems_Balancing_Security_and_Privacy_in_the_Digital_Age
99. AI and Ethics in Surveillance: Balancing Security and Privacy in a Digital World - PhilArchive, accessed July 3, 2025, https://philarchive.org/archive/MOSAAE-2
100. 인공지능 감시에 의한 권력의 확대와 그 규범적 대응방안 연구 - 정보통신정책연구원, accessed July 3, 2025, https://kisdi.re.kr/eng/report/e/fileView.do?key=m2102103219640&arrMasterId=4333452&id=1829356
101. 인공지능(AI) 윤리와 사회문제 - 3편 - 카이스트신문, accessed July 3, 2025, https://times.kaist.ac.kr/news/articleView.html?idxno=21987
102. 인공지능 기술을 이용한 국가의 사회감시 체계 현황과 주요 쟁점, accessed July 3, 2025, https://www.ifans.go.kr/knda/com/fileupload/FileDownloadView.do?storgeId=c61b04e5-0182-4c75-ad21-828ecacfb855&uploadId=13801815779622668&fileSn=1
103. (PDF) Ethical and Legal Implications of AI-Driven Surveillance: Balancing Security and Privacy in a Regulated Environment - ResearchGate, accessed July 3, 2025, https://www.researchgate.net/publication/385747354_Ethical_and_Legal_Implications_of_AI-Driven_Surveillance_Balancing_Security_and_Privacy_in_a_Regulated_Environment
104. Edge AI for Real-Time Anomaly Detection in Smart Homes - MDPI, accessed July 3, 2025, https://www.mdpi.com/1999-5903/17/4/179
105. AI 기술 발전과 윤리적 도전: 규제 동향과 기업의 대응 전략 - Goover, accessed July 3, 2025, https://seo.goover.ai/report/202503/go-public-report-ko-7a62bcd8-8c25-4ac8-a108-f821de210724-0-0.html
106. Real-Time Anomaly Detection in IT Environments Using AI-Driven RMM - Algomox, accessed July 3, 2025, https://www.algomox.com/resources/blog/real_time_anomaly_detection_ai_driven_rmm_it_environments.html
107. [2506.19077] Multimodal Anomaly Detection with a Mixture-of-Experts - arXiv, accessed July 3, 2025, https://arxiv.org/abs/2506.19077
108. MACS 솔루션 | Multi-modal AI for CCTV, drone, robot solution | AI 영상분석 기술 | 피아스페이스 PIAspace, accessed July 3, 2025, https://www.pia.space/home-pages/macs
109. Feature relevance XAI in anomaly detection: Reviewing approaches and challenges - PMC, accessed July 3, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9944120/
110. The Ethics of AI in Monitoring and Surveillance | NICE Actimize, accessed July 3, 2025, https://www.niceactimize.com/blog/fmc-the-ethics-of-ai-in-monitoring-and-surveillance/


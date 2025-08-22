---
layout: page
title: DiMSUM 모델
permalink: /ai/transformers/diffusion models/advanced dit models/DiMSUM 모델
---


본 보고서는 인공지능 및 컴퓨터 비전 분야에서 등장한 여러 'DiMSUM'이라는 이름의 모델 및 프레임워크를 명확히 구분하고, 그중 가장 주목받는 **DiMSUM (Diffusion Mamba) 이미지 생성 모델**에 대해 심층적으로 분석한다. 2024년 NeurIPS 학회에서 발표된 이 모델은 기존 생성 모델의 패러다임을 바꾸는 혁신적인 하이브리드 아키텍처를 제시한다.1

DiMSUM의 핵심은 공간(spatial) 정보와 주파수(frequency) 정보를 동시에 활용하여 이미지 생성의 귀납적 편향(inductive bias)을 강화하는 것이다. 이를 위해 모델은 세 가지 핵심 구성 요소를 유기적으로 결합한다. 첫째, **웨이블릿 맘바(Wavelet Mamba)**는 이산 웨이블릿 변환(DWT)을 통해 이미지를 여러 주파수 하위 대역으로 분해하고, 이를 상태 공간 모델(SSM)인 맘바(Mamba)로 처리하여 효율적으로 장거리 주파수 관계를 포착한다.3 둘째, **교차-어텐션 융합(Cross-Attention Fusion)** 레이어는 공간 정보와 주파수 정보 처리 결과를 지능적으로 통합하여 시너지를 극대화한다.2 셋째, 

**전역 공유 트랜스포머(Globally-Shared Transformer)** 블록은 맘바의 선형적 스캔 방식이 놓칠 수 있는 전역적 관계를 보완하여 모델의 전반적인 이해도를 높인다.3

실증적 검증 결과, DiMSUM은 CelebA-HQ, LSUN Church, ImageNet-1K와 같은 표준 벤치마크에서 기존의 강력한 모델인 DiT(Diffusion Transformer), SiT, DIFFUSSM을 능가하는 최고 수준의 FID(Fréchet Inception Distance) 점수를 달성했다.1 특히, DiT와 유사한 파라미터 수를 가지면서도 추론 속도는 약 42% 더 빠르고, 학습 수렴 속도 또한 현저히 개선되어 효율성 측면에서 큰 장점을 보였다.1

그러나 모델의 복잡성, 스캔 순서 문제의 완전한 해결 여부, 확장성 주장의 타당성 등에 대한 동료 심사 과정에서의 비판과 논쟁 역시 존재했다.5 본 보고서는 이러한 비판과 저자들의 답변을 상세히 분석하여 DiMSUM 모델에 대한 균형 잡힌 시각을 제공한다. 결론적으로 DiMSUM은 생성 모델 아키텍처의 진화 과정에서 중요한 이정표이며, 특히 공간-주파수 융합이라는 새로운 접근법을 통해 향후 고효율, 고성능 생성 AI 연구에 중요한 방향성을 제시한다.


인공지능 연구가 기하급수적으로 발전함에 따라, 새로운 모델과 기술에 부여되는 이름 또한 빠르게 증가하고 있다. 이 과정에서 동일한 이름이 전혀 다른 분야의 상이한 개념을 지칭하는 '전문용어 포화(terminological saturation)' 현상이 발생하곤 한다. 'DiMSUM'이라는 용어는 이러한 현상의 대표적인 사례로, 명확한 이해를 위해서는 우선 이 용어가 지칭하는 다양한 개념들을 구분하는 작업이 필수적이다. 이는 독자의 잠재적 혼란을 방지하고, 본 보고서의 분석 대상을 명확히 하여 전문성과 신뢰도를 확보하는 과정이다.


현재 학계와 산업계에서 'DiMSUM' 또는 유사한 이름으로 불리는 주요 개념들은 다음과 같다.

- **DIMSUM (분산 컴퓨팅 알고리즘):** 스탠포드 대학에서 발표한 연구로, 'Dimension Independent Matrix Square using MapReduce'의 약자이다. 이는 MapReduce 프로그래밍 모델을 사용하여 거대한 희소 행렬(sparse matrix)의 특이값(singular values)을 분산 환경에서 효율적으로 계산하기 위한 알고리즘이다. 특히 행(row)의 개수(m)가 매우 큰 경우(m≈1013)에 통신 비용 의존성을 없애는 데 중점을 둔다.6
- **DiMSum (생명정보학 파이프라인):** 'Deep Mutational Scanning (DMS) data' 분석을 위한 오류 모델 및 파이프라인의 이름이다. 이 R 기반 소프트웨어 패키지는 원시 시퀀싱 데이터로부터 단백질, RNA 등의 변이(variant)가 생존 적합성(fitness)에 미치는 영향을 정량적으로 측정하고, 실험 과정에서 발생할 수 있는 다양한 오류 원인을 진단하는 엔드투엔드 솔루션을 제공한다.7
- **DimSum (프로그램 검증 프레임워크):** 'A Decentralized Approach to Multi-language Semantics and Verification'의 약자로, Coq 증명 보조 도구 내에 구현된 프레임워크이다. 이는 서로 다른 프로그래밍 언어(예: 고수준 언어 Rec, 저수준 어셈블리 언어 Asm)로 작성된 프로그램 구성 요소들을 독립적으로 정의하고 검증하면서도, 필요할 때 이들을 결합하고 번역할 수 있는 탈중앙화된 접근법을 제시한다.10
- **DIM-SUM (스마트 유틸리티 데이터 처리):** 'Dynamic IMputation for Smart Utility Management'의 약자이다. 이는 스마트 수도, 전기 계량기 등에서 발생하는 시계열 데이터의 결측치(missing values)를 효과적으로 처리하기 위한 전처리 프레임워크이다. 실제 데이터에서 관찰되는 다양한 결측 패턴을 학습하여 강건한 보간(imputation) 모델을 훈련시키는 것을 목표로 한다.13
- **기타 동음이의어:** 이 외에도 1992년에 발표된 다중지수 모델 판별을 위한 전문가 시스템 14, 중국 광둥 지방의 전통 음식인 딤섬(Dim sum, 點心)과 그 문화 16, 관련된 3D 모델링 및 디자인 프로젝트 18, 그리고 다양한 상업 및 커뮤니티(예: DimSumLabs)의 이름으로도 사용된다.21


상기한 여러 개념 중, 본 보고서는 가장 최근에 발표되어 학문적 논의의 중심에 있는 모델에 초점을 맞춘다. 따라서 본 보고서가 심층적으로 분석할 대상은 Phung 연구팀이 2024년 NeurIPS 학회에서 발표한 **DiMSUM: Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation** 모델이다.1 이 모델은 최신 인공지능 생성 모델 분야의 기술적 흐름을 대표하며, 그 성능과 독창적인 아키텍처로 인해 학계의 큰 주목을 받고 있다. 이후 본 보고서에서 'DiMSUM'은 별도의 언급이 없는 한 이 이미지 생성 모델을 지칭한다.


DiMSUM 모델의 등장을 이해하기 위해서는 최근 몇 년간 생성 모델, 특히 확산 모델(Diffusion Model)의 기반 아키텍처가 어떻게 변화해왔는지를 살펴봐야 한다. 이 변화의 핵심에는 **표현력**과 **효율성**이라는 두 가지 가치 사이의 근본적인 긴장 관계가 존재하며, DiMSUM은 이 긴장을 해소하기 위한 독창적인 해법을 제시한다.


초기 확산 모델은 주로 U-Net 아키텍처를 기반으로 했으나, 곧이어 등장한 트랜스포머(Transformer) 기반 확산 모델, 즉 DiT(Diffusion Transformer)가 새로운 표준으로 자리 잡았다.3 DiT의 강력함은 모든 입력 토큰(이미지 패치) 간의 관계를 전역적으로 계산하는 **셀프-어텐션(self-attention) 메커니즘**에서 비롯된다.3 이는 이미지의 복잡하고 미묘한 컨텍스트를 포착하는 데 탁월한 성능을 보이며, 생성되는 이미지의 품질을 획기적으로 향상시켰다.

그러나 이러한 강력한 표현력에는 값비싼 대가가 따랐다. 셀프-어텐션은 입력 시퀀스의 길이에 대해 제곱에 비례하는(O(N2)) 계산 복잡도를 가진다.3 이는 고해상도 이미지를 처리할 때 토큰의 수가 급격히 증가함에 따라 훈련 및 추론에 필요한 시간과 메모리 자원이 기하급수적으로 늘어나는 문제를 야기했다. 이 '제곱의 저주'는 트랜스포머 기반 모델의 확장성을 저해하는 근본적인 한계로 작용했다.


트랜스포머의 계산 비용 문제를 해결할 대안으로 자연어 처리(NLP) 분야에서 부상한 것이 바로 **상태 공간 모델(State-Space Model, SSM)**이다. 특히 맘바(Mamba)는 SSM을 기반으로 한 아키텍처로, 시퀀스 길이에 대해 **선형 시간 복잡도(O(N))**를 달성하여 학계에 큰 충격을 주었다.23 맘바는 긴 시퀀스를 매우 효율적으로 모델링할 수 있는 능력 덕분에 음성, 유전체 분석 등 다양한 분야로 빠르게 확산되었고, 곧 컴퓨터 비전 분야에도 적용되기 시작했다.26


하지만 맘바를 비전 태스크에 적용하는 데에는 근본적인 어려움이 존재했다. 맘바는 본질적으로 1차원 시퀀스를 처리하도록 설계되었기 때문에, 2차원 구조를 가진 이미지를 처리하기 위해서는 이미지 패치들을 1차원으로 펼쳐서 특정 순서에 따라 배열해야 했다.2 이 과정에서 모델의 성능이 어떤 스캔 순서(예: 양방향, 교차 스캔, 지그재그 스캔)를 선택하는지에 크게 의존하게 되는 

**'약한 귀납적 편향(weak inductive bias)'** 문제가 발생했다.3 이는 모든 토큰 간의 관계를 순서에 무관하게 계산하는 트랜스포머와 대조되는 맘바의 치명적인 약점이었다. 수많은 비전 맘바(Vision Mamba) 연구들이 이 스캔 순서 문제를 해결하기 위해 다양한 방법을 제안했으며, DiMSUM 역시 이 문제에 대한 독창적인 해결책을 제시하는 과정에서 탄생했다.26 결국 DiMSUM의 하이브리드 아키텍처는 트랜스포머의 표현력과 맘바의 효율성이라는 두 마리 토끼를 모두 잡으려는 시도이자, 어느 한쪽만으로는 완벽한 해결책이 될 수 없다는 현실을 인정한 공학적 타협의 산물이라고 볼 수 있다.


DiMSUM은 단순한 아키텍처 교체가 아닌, 여러 혁신적인 아이디어를 융합한 복합적인 시스템이다. 그 설계 철학의 중심에는 공간과 주파수라는 두 가지 관점을 통합하여 이미지의 본질을 더 깊이 이해하려는 시도가 자리 잡고 있다.


DiMSUM은 **플로우 매칭(Flow Matching)** 목적 함수를 사용하여 훈련되는 확산 모델 프레임워크 내에 구축된 **하이브리드 맘바-트랜스포머 네트워크**이다.3 전체적인 데이터 흐름은 다음과 같다. 먼저 입력 이미지는 인코더를 통해 저차원의 잠재 공간 맵(latent map)으로 변환된다. 이 잠재 맵은 여러 개의 DiMSUM 블록으로 구성된 시퀀스를 통과하며 정교하게 처리된다. 마지막으로, 처리된 잠재 맵은 디코더를 통해 최종 출력 이미지로 복원된다.3


DiMSUM의 가장 핵심적인 가설은 이미지 정보를 **공간 영역(spatial domain)**과 **주파수 영역(frequency domain)**에서 동시에 처리하는 것이 더 풍부하고 강건한 귀납적 편향을 제공한다는 것이다.1 공간 영역은 픽셀들이 어떻게 배열되어 있는지를 다루는 반면, 주파수 영역은 픽셀 값의 변화율, 즉 이미지의 전반적인 구조(저주파)와 세밀한 디테일(고주파)을 다룬다. DiMSUM은 이 두 영역의 정보를 모두 활용하여 이미지의 국소적 특징과 전역적 구조를 동시에 파악한다.


DiMSUM의 복잡하고도 정교한 아키텍처는 세 가지 핵심 구성 요소의 상호작용으로 이루어진다.


이 구성 요소는 주파수 영역의 정보를 효율적으로 처리하는 역할을 담당한다.

- **이산 웨이블릿 변환 (DWT):** 먼저, 하르(Haar) 웨이블릿과 같은 이산 웨이블릿 변환을 사용하여 입력된 잠재 맵을 여러 개의 주파수 하위 대역(subband)으로 분해한다.3 이 과정은 이미지를 전반적인 윤곽을 나타내는 저주파 성분(LL)과 수평(LH), 수직(HL), 대각선(HH) 방향의 디테일을 나타내는 고주파 성분으로 분리하는 것과 같다.
- **분리된 시퀀스 스캔:** 이렇게 분리된 각 주파수 하위 대역은 크기가 더 작고 정보가 압축되어 있다. 맘바는 이 작은 시퀀스들을 각각 스캔함으로써 계산 효율성을 높이는 동시에, 공간 영역에서는 파악하기 어려운 장거리 주파수 간의 상관관계를 효과적으로 포착한다.2


이 레이어는 공간 영역을 처리한 결과와 주파수 영역을 처리한 결과를 지능적으로 통합하는 역할을 한다.

- **정보의 시너지:** 표준적인 공간 스캔 맘바의 출력과 웨이블릿 맘바의 출력을 단순히 합치는 것이 아니라, **'쿼리 교환(query-swapped)' 교차-어텐션** 기법을 사용한다.2 이는 한쪽 영역의 정보(예: 공간)를 쿼리(query)로 사용하여 다른 쪽 영역의 정보(예: 주파수)에서 핵심적인 특징을 추출하고, 그 반대의 과정도 수행하여 두 정보를 동적으로 융합하는 방식이다. 이를 통해 두 영역의 정보가 서로를 보완하며 최종 표현력을 극대화한다.


이 구성 요소는 맘바 아키텍처의 고질적인 약점을 보완하는 안전장치 역할을 한다.

- **전역적 컨텍스트 보완:** 맘바의 선형 스캔 방식은 본질적으로 국소적인 정보 흐름에 강점을 가지지만, 이미지 전체를 아우르는 전역적인 컨텍스트를 완벽하게 포착하는 데는 한계가 있다. 전역 공유 트랜스포머 블록은 일종의 **'토큰 믹싱 레이어(token-mixing layer)'**로 작동하여, 모든 이미지 패치 간의 장거리 의존성을 계산한다.3
- **약한 귀납적 편향 해결:** 이 트랜스포머 블록은 맘바의 성능을 저해하는 '수동 정의 스캔 순서' 문제에 대한 근본적인 해결책을 제공한다. 스캔 순서에 구애받지 않고 전역적인 관계를 포착함으로써 모델의 안정성과 성능을 크게 향상시킨다.3

이러한 복잡한 구조는 동료 심사 과정에서 "지나치게 복잡하다"는 비판을 받기도 했다.5 하지만 저자들은 반박 과정에서 주파수 정보를 단순히 트랜스포머에 통합하려는 초기 실험이 노이즈 패턴만을 생성하며 실패했다고 밝혔다.5 이는 DiMSUM의 복잡성이 불필요한 과잉 설계가 아니라, 공간-주파수 융합이라는 어려운 문제를 해결하기 위해 반드시 필요한, 정교하게 설계된 다단계 솔루션임을 시사한다. 즉, 이 모델에서는 복잡성 자체가 해결책의 일부인 셈이다.


DiMSUM의 아키텍처적 우수성은 정량적인 실험 결과를 통해 입증된다. 특히 이미지 생성 품질과 계산 효율성 측면에서 기존의 대표적인 모델들과의 비교는 DiMSUM의 가치를 명확하게 보여준다.


이미지 생성 모델의 품질을 평가하는 표준 지표인 FID(Fréchet Inception Distance) 점수(낮을수록 좋음)를 통해 DiMSUM의 성능을 확인할 수 있다. 아래 표는 주요 벤치마크 데이터셋에서 DiMSUM과 다른 모델들의 성능을 비교한 것이다.

**표 1: 주요 모델별 FID 점수 비교**

| 모델           | 파라미터(M) | CelebA-HQ 256 FID ↓ | LSUN Church 256 FID ↓ | ImageNet-1K 256 (CFG) FID ↓ |
| -------------- | ----------- | ------------------- | --------------------- | --------------------------- |
| DiT-L/2        | -           | -                   | 4.85                  | 3.04                        |
| DIFFUSSM-L/2   | -           | -                   | 4.63                  | 2.52                        |
| SiT-L/2        | -           | -                   | 4.12                  | 2.27                        |
| **DiMSUM-L/2** | 460M        | **4.62**            | **3.76**              | **2.11**                    |

데이터 출처: 1

표 1에서 볼 수 있듯이, DiMSUM-L/2 모델은 460M개의 파라미터를 가지고 CelebA-HQ, LSUN Church, ImageNet-1K 세 가지 데이터셋 모두에서 비교 대상 모델들보다 가장 낮은, 즉 가장 우수한 FID 점수를 기록했다.1 특히 이미지 분류의 표준 벤치마크인 ImageNet-1K에서 2.11이라는 기록적인 FID 점수를 달성한 것은 DiMSUM이 매우 높은 품질과 다양성을 갖춘 이미지를 생성할 수 있음을 강력하게 시사한다.

여기서 주목할 점은 저자들이 DiT뿐만 아니라 **SiT**와도 비교했다는 사실이다. DiMSUM은 플로우 매칭(Flow Matching)이라는 최신 훈련 기법을 사용하는데, 기존 DiT는 다른 방식으로 훈련되었다.4 만약 DiMSUM과 DiT만 비교했다면, 성능 향상이 아키텍처 때문인지 훈련 기법 때문인지 불분명했을 것이다. 저자들은 DiT를 플로우 매칭으로 재훈련한 SiT를 비교군에 포함함으로써, 훈련 방식을 통제하고 순수하게 

**아키텍처의 우수성**을 입증하는 정교한 실험 설계를 보여주었다.5 이는 DiMSUM의 성능 향상이 우연이 아닌, 잘 설계된 구조에서 비롯되었음을 뒷받침하는 중요한 근거가 된다.


DiMSUM의 또 다른 핵심적인 장점은 효율성이다. 이는 동료 심사 과정에서 제기된 추론 속도에 대한 비판에 대응하여 저자들이 제시한 데이터를 통해 명확히 확인할 수 있다.

**표 2: DiMSUM-L/2 vs. DiT-L/2 효율성 지표 비교**

| 지표                  | DiMSUM-L/2 | DiT-L/2 |
| --------------------- | ---------- | ------- |
| 파라미터 (M)          | 460M       | -       |
| GFLOPs                | -          | -       |
| 훈련 반복/초          | 더 빠름    | -       |
| 추론 지연 시간 (초) ↓ | **2.2**    | 3.8     |

데이터 출처: 5 (OpenReview 저자 반박 자료 재구성)

표 2는 DiMSUM이 DiT와 유사한 규모의 모델임에도 불구하고 추론에 걸리는 시간이 3.8초에서 2.2초로 크게 단축되었음을 보여준다.5 이는 약 42%의 속도 향상으로, 실제 서비스 환경에서 매우 큰 이점을 제공한다. 또한, 저자들은 DiMSUM이 더 적은 훈련 반복 횟수로도 최고 성능에 도달하여 훈련 수렴 속도가 빠르다고 보고했다.1 이러한 효율성은 맘바 아키텍처의 선형 복잡도와 웨이블릿 변환을 통한 효율적인 정보 압축이 성공적으로 결합된 결과로 분석된다.


DiMSUM은 뛰어난 성능에도 불구하고, NeurIPS 학회에 제출되었을 때 동료 심사 과정에서 여러 날카로운 비판에 직면했다. 이러한 비판과 그에 대한 저자들의 답변을 살펴보는 것은 모델의 장단점을 보다 객관적이고 깊이 있게 이해하는 데 필수적이다. 이 과정은 과학적 진보가 어떻게 비판과 수정을 통해 이루어지는지를 보여주는 생생한 사례이기도 하다.5


- **리뷰어의 지적 (Weakness):** "제안된 방법은 너무 복잡하다. 이미지 공간과 주파수 공간을 모두 스캔하고, 맘바 프레임워크에 트랜스포머 블록까지 추가하여 전체적인 방법론이 복잡해졌다".5
- **저자의 답변 및 분석 (Rebuttal & Analysis):** 저자들은 이 지적을 직접적으로 반박하기보다, 복잡성이 불가피했음을 시사하는 근거를 제시했다. 저자들의 답변에 따르면, 주파수 정보를 단순히 트랜스포머 모델에 통합하려는 더 간단한 시도는 학습에 실패하고 노이즈 이미지만을 생성했다.5 이는 DiMSUM의 복잡한 구조가 임의적인 것이 아니라, 공간-주파수 융합이라는 새로운 기능을 구현하기 위한 필수적인 절충안(trade-off)이었음을 의미한다. 즉, 새로운 성능을 얻기 위해 구조적 복잡성을 감수해야 했던 것이다.


- **리뷰어의 지적 (Weakness):** "논문에서 언급한 '수동으로 정의된 스캔 순서' 문제는 근본적으로 해결되지 않았다. 주파수 영역의 표현 역시 2차원이므로, 주파수 영역에서의 스캔 문제도 공간 영역과 동일하게 존재한다".5
- **저자의 답변 및 분석 (Rebuttal & Analysis):** 저자들은 이 문제를 '완전히 해결'한 것이 아니라 '완화'하는 것이 목표였음을 명확히 했다. 그들의 **"웨이블릿 공간에서의 윈도우 스캔 전략"**은 각 주파수 하위 대역 내에서 효율적으로 전역 정보를 포착함으로써 스캔 순서에 대한 의존성의 부정적인 영향을 줄인다.5 이는 초기 논문의 주장을 보다 정교하게 다듬은 것으로, 문제의 본질을 정확히 이해하고 있음을 보여준다.


- **리뷰어의 지적 (Weakness):** "초기 논문에는 제안된 프레임워크와 트랜스포머 간의 실제 추론 속도 비교가 포함되어 있지 않다".5
- **저자의 답변 및 분석 (Rebuttal & Analysis):** 이는 매우 타당하고 직접적인 비판이었다. 저자들은 답변서에 DiMSUM-L/2가 DiT-L/2보다 훨씬 빠르다는(2.2초 vs 3.8초) 구체적인 수치를 담은 표를 제시하며 이 비판을 정면으로 수용하고 보완했다.5 이는 동료 심사 과정이 어떻게 논문의 주장을 구체적인 데이터로 강화하는 데 기여하는지를 보여주는 훌륭한 예이다.


- **리뷰어의 지적 (Weakness):** "모델에 다수의 트랜스포머 블록이 포함된 것을 고려할 때, 이를 '맘바 기반' 모델이라고 부르는 것은 오해의 소지가 있다".5
- **저자의 답변 및 분석 (Rebuttal & Analysis):** 저자들은 이 지점을 인정하고 용어를 **"하이브리드 맘바-트랜스포머 네트워크"**로 수정하겠다고 답변했다.5 이는 학문적 논의를 통해 용어의 명확성을 높이고 연구 내용을 더 정확하게 전달하려는 협력적인 과학적 개선 과정을 잘 보여준다.


- **리뷰어의 지적 (Weakness):** "제안된 모델의 파라미터 수가 10억 개 미만인 상황에서, 제목에 '확장 가능'이라는 표현을 사용하는 것은 적절하지 않을 수 있다".5
- **저자의 답변 및 분석 (Rebuttal & Analysis):** 저자들은 '확장성'의 의미를 더 넓게 재정의하며 자신들의 주장을 방어했다. 그들은 확장성이란 단순히 모델의 크기뿐만 아니라, **"다양한 데이터셋과 계산 자원을 처리하면서도 수용 가능한 시간 내에 정확한 결과를 생성하는 능력"**을 포함한다고 주장했다.5 이러한 재정의는 확장성의 초점을 원시적인 크기에서 기능적 효율성으로 전환시키려는 시도이다.


DiMSUM은 복잡하지만 효과적인 하이브리드 아키텍처를 통해 생성 모델 분야에 중요한 기여를 했다. 이 모델의 등장은 단순히 새로운 SOTA(State-of-the-Art) 모델의 출현을 넘어, 향후 생성 AI 아키텍처가 나아갈 방향에 대한 깊은 통찰을 제공한다.


DiMSUM의 가장 중요한 기여는 **공간 정보와 주파수 정보를 융합하는 새로운 패러다임**을 성공적으로 증명했다는 점이다. 웨이블릿 맘바, 교차-어텐션 융합, 전역 공유 트랜스포머라는 세 가지 구성 요소의 유기적인 결합을 통해, 순수 트랜스포머 모델 대비 월등한 이미지 품질과 계산 효율성을 동시에 달성할 수 있음을 보여주었다. 이는 트랜스포머의 표현력과 맘바의 효율성 사이의 오랜 긴장 관계를 해소할 수 있는 구체적인 청사진을 제시한 것으로 평가할 수 있다.


저자들은 DiMSUM 아키텍처를 기반으로 한 후속 연구의 가능성을 제시했다. 대표적으로 대규모 **텍스트-이미지 생성(text-to-image generation)** 모델이나 **멀티모달(multimodal) 확산 모델**로의 확장이 있다.31 DiMSUM의 효율성과 확장성은 더 크고 복잡한 데이터를 처리해야 하는 이러한 차세대 모델의 기반이 되기에 충분한 잠재력을 가지고 있다. 한편, 악의적인 이미지 생성과 같은 기술의 오용 가능성에 대한 우려도 존재한다. 이에 대해 저자들은 최근 발전하고 있는 AI 보안 관련 연구를 통해 이러한 위험을 완화할 수 있을 것이라는 자신감을 내비쳤다.5


DiMSUM은 '비전 맘바(Vision Mamba)'라는 더 큰 연구 흐름 속에서 이해해야 한다. 맘바를 2D 이미지에 적용하기 위한 여러 시도들이 경쟁적으로 이루어지고 있으며, DiMSUM의 공간-주파수 융합 접근법은 그중 하나이다. 예를 들어, **Vim**은 양방향 스캔을 통해 26, **LocalMamba**는 윈도우 기반 스캔을 통해 30, 그리고 **Mamba®**는 레지스터 토큰을 삽입하는 방식을 통해 32 2D 스캔 문제를 해결하고자 했다.

결론적으로, DiMSUM을 이 경쟁의 최종 승자나 완결된 아키텍처로 보기보다는, **"효율적이고 고성능인 시각적 생성을 위한 최적의 아키텍처는 무엇인가?"**라는 거대한 과학적 질문에 대한 중요한 데이터 포인트로 보아야 한다. DiMSUM의 성공은 공간-주파수 융합이라는 새로운 연구 방향의 가능성을 강력하게 입증했으며, 이는 향후 비전 맘바 아키텍처 경쟁에 중요한 변수가 될 것이다. AI 분야는 이제 이 복잡하고 강력한 하이브리드 접근법을 시험하고, 이를 바탕으로 더 나은 모델을 구축해 나갈 것이다.


1. VinAIResearch/DiMSUM: DiMSUM: Diffusion Mamba - A ... - GitHub, accessed July 19, 2025, https://github.com/VinAIResearch/DiMSUM
2. Revision History for DiMSUM: Diffusion Mamba - A Scalable... - OpenReview, accessed July 19, 2025, https://openreview.net/revisions?id=9NozArelrQ
3. DiMSUM : Diffusion Mamba - A Scalable and Unified Spatial ... - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/39bc6e3cbf5a1991d33dc10ebff9a9cf-Paper-Conference.pdf
4. NeurIPS Poster DiMSUM: Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/95642
5. DiMSUM: Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=KqbLzSIXkm&referrer=%5Bthe%20profile%20of%20Hoang%20Phan%5D(%2Fprofile%3Fid%3D~Hoang_Phan1)](https://openreview.net/forum?id=KqbLzSIXkm&referrer=[the+profile+of+Hoang+Phan](/profile?id%3D~Hoang_Phan1))
6. Dimension Independent Matrix Square using MapReduce (DIMSUM) - Stanford University, accessed July 19, 2025, https://stanford.edu/~rezab/papers/dimsum.pdf
7. DiMSum error model estimates multiplicative and additive error sources... - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/DiMSum-error-model-estimates-multiplicative-and-additive-error-sources-in-fitness-scores_fig2_343698845
8. (PDF) DiMSum: An error model and pipeline for analyzing deep mutational scanning data and diagnosing common experimental pathologies - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/343698845_DiMSum_An_error_model_and_pipeline_for_analyzing_deep_mutational_scanning_data_and_diagnosing_common_experimental_pathologies
9. DiMSum: an error model and pipeline for analyzing deep mutational scanning data and diagnosing common experimental pathologies | bioRxiv, accessed July 19, 2025, https://www.biorxiv.org/content/10.1101/2020.06.25.171421v1
10. DimSum: A Decentralized Approach to Multi-language Semantics and Verification, accessed July 19, 2025, https://plv.mpi-sws.org/dimsum/
11. DimSum: A Decentralized Approach to Multi-language Semantics and Verification - Iris Project, accessed July 19, 2025, https://iris-project.org/pdfs/2023-popl-dimsum.pdf
12. DimSum: A Decentralized Approach to Multi-language Semantics and Verification - KOPS, accessed July 19, 2025, https://kops.uni-konstanz.de/bitstreams/37425f85-63c8-4abb-98b8-23c34352f0e6/download
13. [2506.20023] DIM-SUM: Dynamic IMputation for Smart Utility Management - arXiv, accessed July 19, 2025, http://arxiv.org/abs/2506.20023
14. DIMSUM: an expert system for multiexponential model discrimination - PubMed, accessed July 19, 2025, https://pubmed.ncbi.nlm.nih.gov/1566840/
15. DIMSUM: an expert system for multiexponential model discrimination | American Journal of Physiology-Endocrinology and Metabolism, accessed July 19, 2025, https://journals.physiology.org/doi/10.1152/ajpendo.1992.262.4.E546
16. Dim sum - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Dim_sum
17. The Brief History of Dim Sum - EB Frozen Food, accessed July 19, 2025, https://www.ebfood.com.my/brief-history-of-dim-sum/
18. 3D Dimsum Models - Browse & Download Formats - TurboSquid, accessed July 19, 2025, https://www.turbosquid.com/Search/3D-Models/dimsum
19. dimsum - design.by.yuan, accessed July 19, 2025, https://designbyyuan.com/projects/dimsum
20. Chinese Dim Sum - 3D model by You Zhao Ying (@zero.you019) [50f5aa9] - Sketchfab, accessed July 19, 2025, https://sketchfab.com/3d-models/chinese-dim-sum-50f5aa9fa9b746f2afffd270e340b7e3
21. Tag: AI - Dim Sum Labs, accessed July 19, 2025, https://www.dimsumlabs.com/tag/ai/
22. Tag: architecture - Dim Sum Labs, accessed July 19, 2025, https://www.dimsumlabs.com/tag/architecture/
23. DiM: Diffusion Mamba for Efficient High-Resolution Image Synthesis - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.14224v1
24. [2405.14224] DiM: Diffusion Mamba for Efficient High-Resolution Image Synthesis - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2405.14224
25. From S4 to Mamba: A Comprehensive Survey on Structured State Space Models - arXiv, accessed July 19, 2025, https://www.arxiv.org/pdf/2503.18970
26. hustvl/Vim: [ICML 2024] Vision Mamba: Efficient Visual Representation Learning with Bidirectional State Space Model - GitHub, accessed July 19, 2025, https://github.com/hustvl/Vim
27. [2401.09417] Vision Mamba: Efficient Visual Representation Learning with Bidirectional State Space Model - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2401.09417
28. radarFudan/Awesome-state-space-models - GitHub, accessed July 19, 2025, https://github.com/radarFudan/Awesome-state-space-models
29. [2502.20764] Visual Attention Exploration in Vision-Based Mamba Models - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2502.20764
30. [2403.09338] LocalMamba: Visual State Space Model with Windowed Selective Scan - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2403.09338
31. DiMSUM: Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=KqbLzSIXkm&referrer=%5Bthe%20profile%20of%20Quan%20Dao%5D(%2Fprofile%3Fid%3D~Quan_Dao1)](https://openreview.net/forum?id=KqbLzSIXkm&referrer=[the+profile+of+Quan+Dao](/profile?id%3D~Quan_Dao1))
32. Vision Mamba Also Needs Registers - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.14858v2


# 구글 TPU에 대한 심층 고찰: AI 가속의 새로운 패러다임





## I. 서론: AI 가속의 새로운 패러다임, 구글 TPU의 탄생



인공지능(AI), 특히 딥러닝의 발전은 전례 없는 규모의 연산 능력을 요구하며 기존 컴퓨팅 아키텍처의 한계를 드러냈습니다. 범용 프로세서인 CPU(Central Processing Unit)는 복잡하고 순차적인 작업에는 능하지만, 딥러닝의 핵심을 이루는 대규모 병렬 연산에는 비효율적입니다.1 그래픽 처리를 위해 탄생한 GPU(Graphics Processing Unit)는 수천 개의 코어를 활용한 병렬 처리 능력 덕분에 AI 시대의 초기 동력원 역할을 훌륭히 수행했습니다.3 하지만 GPU조차도 근본적으로는 범용성을 염두에 둔 설계였기에, 폰 노이만 구조(Von Neumann architecture)에서 비롯되는 본질적인 문제, 즉 프로세서의 연산 속도를 메모리 접근 속도가 따라가지 못하는 '폰 노이만 병목 현상(Von Neumann bottleneck)'으로부터 자유롭지 못했습니다.5 데이터에 굶주린 거대 AI 모델 앞에서, 이는 성능과 효율성의 발목을 잡는 족쇄였습니다.

이러한 배경 속에서 특정 애플리케이션에 맞춰 하드웨어 자체를 최적화하는 주문형 반도체(ASIC, Application-Specific Integrated Circuit)라는 개념이 대두되었습니다.7 범용성이 아닌, 오직 단 하나의 목적을 위해 모든 자원을 집중하여 극강의 효율을 추구하는 철학입니다. 구글의 텐서 처리 장치(TPU, Tensor Processing Unit)는 바로 이 ASIC 철학이 딥러닝과 만난 가장 상징적인 결과물입니다.

TPU의 탄생은 구글이 반도체 시장에 진출하려는 공격적인 시도가 아니었습니다. 오히려, 그것은 자사의 핵심 서비스 생존을 위한 방어적이고 전략적인 필연에 가까웠습니다. 2010년대 초반, 구글 검색(RankBrain), 구글 포토, 구글 번역과 같은 핵심 서비스들은 빠르게 딥러닝 기술을 흡수하며 성장했습니다.3 이러한 AI 기반 서비스의 폭발적인 증가는 기존 CPU와 GPU만으로는 감당하기 힘든 수준의 연산 수요를 낳았습니다. 구글 내부 분석에 따르면, 이 추세가 계속될 경우 막대한 비용을 들여 데이터센터 수를 두 배로 늘려야 할지도 모르는 상황에 직면했습니다.10 이는 경제적으로나 물리적으로 지속 불가능한 시나리오였습니다. 결국 구글은 상용 하드웨어에 의존하는 대신, 자사의 신경망 워크로드에 완벽하게 맞춰진, 전력 효율과 비용 효율이 극대화된 자체 칩을 개발하기로 결정했습니다. 이처럼 TPU는 구글의 AI 제국을 지탱하기 위한 내부적 필요에서 태어났으며, 이러한 탄생 배경은 초기 TPU 아키텍처가 추론(inference)에 집중하는 등 그 기술적 방향성을 깊이 있게 형성했습니다.



## 1. II. TPU 핵심 아키텍처 심층 분석: 시스톨릭 어레이와 폰 노이만 병목 현상의 극복



TPU의 혁신성은 단순히 코어 수를 늘리거나 클럭 속도를 높이는 데 있지 않습니다. 그 핵심은 CPU, GPU와는 근본적으로 다른 데이터 처리 방식과 아키텍처 철학에 있습니다.



### 1.1 데이터 처리 방식의 근본적 차이: CPU, GPU, TPU



프로세서의 구조를 비교하면 각자의 역할과 한계가 명확히 드러납니다.

- **CPU**: 소수의 강력한 코어, 대용량 캐시 메모리, 복잡한 제어 로직을 갖추고 있습니다. 단일 스레드의 지연 시간(latency)을 최소화하는 데 최적화되어 있어, 운영체제나 복잡한 애플리케이션 실행에 적합합니다.1
- **GPU**: 수천 개의 단순한 연산 유닛(ALU)을 탑재하여 대규모 병렬 처리에 특화되어 있습니다. 그래픽 렌더링이나 과학 계산처럼 동일한 연산을 수많은 데이터에 동시에 적용하는 작업에서 높은 처리량(throughput)을 보입니다. 하지만 여전히 각 연산을 위해 메모리에서 데이터를 가져오는 범용 프로세서의 틀 안에서 작동합니다.2
- **TPU**: 특정 목적을 위해 설계된 도메인 특화 아키텍처(Domain-Specific Architecture)입니다. 핵심 연산 유닛은 범용 ALU가 아닌, 오직 행렬 곱셈만을 위해 존재하는 '행렬 곱셈 유닛(MXU, Matrix Multiply Unit)'입니다.3



### 1.2 데이터 흐름의 재정의: 시스톨릭 어레이 (Systolic Array)



TPU 아키텍처의 심장은 '시스톨릭 어레이'입니다. 이는 폰 노이만 병목 현상을 극복하기 위한 혁신적인 데이터 흐름 구조입니다.5 기존 프로세서가 '데이터 가져오기(fetch) -> 연산(execute) -> 결과 저장(store)'의 반복적인 과정을 거치는 반면, 시스톨릭 어레이는 심장이 혈액을 박동치듯 데이터를 프로세서 전체에 리드미컬하게 흘려보냅니다.8

작동 원리는 다음과 같습니다. 먼저, 신경망의 가중치(weights) 값들이 2차원 격자 형태로 배열된 수많은 연산 유닛(Processing Element, PE)에 미리 로드됩니다. 그 후, 입력 데이터가 한쪽 끝에서부터 배열로 흘러 들어갑니다. 각 PE는 입력 데이터를 받아 가중치와 곱하고, 그 결과를 누산기(accumulator)에 더한 뒤, 다음 PE로 전달합니다. 이 과정이 마치 파동처럼 배열 전체로 퍼져나가며, 최종적인 행렬 곱셈 결과가 반대쪽 끝에서 출력됩니다.5

이 구조의 가장 큰 장점은 메모리 접근을 극적으로 줄인다는 점입니다. 하나의 데이터 요소가 배열에 한 번 들어오면, 수많은 PE에 의해 재사용되므로 반복적인 메모리 읽기/쓰기 작업이 사라집니다. 이는 연산 자체보다 데이터 이동에 더 많은 시간과 에너지를 소모하는 폰 노이만 병목을 정면으로 돌파하는 해결책입니다. 즉, TPU는 '연산 중심'이 아닌 '데이터 흐름 중심'으로 설계된 하드웨어이며, 그 구조 자체가 행렬 곱셈 알고리즘을 물리적으로 구현한 형태라고 할 수 있습니다.



### 1.3 TPU의 심장: MXU와 HBM



- **행렬 곱셈 유닛 (Matrix Multiply Unit, MXU)**: 시스톨릭 어레이의 물리적 구현체입니다. 예를 들어 TPU v3는 128x128 크기의 MXU 두 개를 탑재하고 있으며, 단일 클럭 사이클에 수만 개의 곱셈-누산 연산을 동시에 수행할 수 있습니다.5 또한, 대부분의 신경망에서 충분한 정확도를 보이는 8비트 정수(INT8)나 bfloat16과 같은 저정밀도 연산에 최적화되어 있어, 와트당 성능을 극대화합니다.8
- **고대역폭 메모리 (High-Bandwidth Memory, HBM)**: 강력한 MXU가 굶주리지 않도록 데이터를 공급하려면 초고속 메모리가 필수적입니다. TPU는 v2부터 HBM을 칩 패키지 내에 직접 통합하여, CPU나 초기 GPU가 사용하던 외부 DDR 메모리와는 비교할 수 없는 수준의 메모리 대역폭을 확보했습니다.6 이는 시스톨릭 어레이가 쉼 없이 작동할 수 있게 하는 생명선과 같습니다.

**표 1: CPU, GPU, TPU 아키텍처 비교**

| 기능                | CPU (Central Processing Unit)   | GPU (Graphics Processing Unit)   | TPU (Tensor Processing Unit)       |
| ------------------- | ------------------------------- | -------------------------------- | ---------------------------------- |
| **핵심 철학**       | 범용성, 저지연 단일 스레드 성능 | 범용 병렬 처리, 고처리량         | 특정 목적용(AI), 최대 효율성       |
| **주요 연산 유닛**  | 소수의 강력한 코어 (ALU)        | 수천 개의 단순한 코어 (ALU)      | 행렬 곱셈 유닛 (MXU)               |
| **메모리 아키텍처** | 대용량 캐시, 외부 DDR 메모리    | 대용량 GDDR 메모리               | 칩 내장 고대역폭 메모리 (HBM)      |
| **병렬 처리 모델**  | 명령어 수준 병렬 처리           | 대규모 데이터 병렬 처리          | 시스톨릭 어레이 (데이터 흐름 병렬) |
| **주요 병목 현상**  | 폰 노이만 병목, 순차 처리 한계  | 폰 노이만 병목, 메모리 대역폭    | (최소화됨) HBM 대역폭              |
| **이상적 워크로드** | 운영체제, 복잡한 응용 프로그램  | 그래픽, 과학 시뮬레이션, 범용 AI | 대규모 신경망 학습 및 추론         |

자료: 1 기반 종합



## 2. III. TPU의 진화: 세대별 발전 과정과 성능 혁신



TPU의 발전사는 AI 산업의 진화와 그 궤를 같이합니다. 각 세대의 TPU는 당시 AI 분야가 직면한 가장 큰 기술적, 경제적 병목 현상을 해결하기 위한 구글의 응답이었습니다.



### 2.1 추론에서 학습으로: v1에서 v2로의 도약



2015년에 등장한 1세대 TPU는 구글 서비스의 추론(inference) 부하를 해결하기 위해 설계된 8비트 정수(INT8) 연산 전용 가속기였습니다.10 당시에는 이미 학습된 모델을 빠르고 효율적으로 실행하는 것이 가장 큰 과제였기 때문입니다. 진정한 변혁은 2017년 TPU v2의 등과 함께 시작되었습니다. v2는 bfloat16 부동소수점 연산을 지원하여 '학습(training)' 기능을 갖추었고, 고대역폭 메모리(HBM)를 통합하여 본격적인 모델 개발용 하드웨어로 자리매김했습니다.6 이는 AI 연구자와 개발자들이 TPU를 사용하여 새로운 모델을 처음부터 만들 수 있게 되었음을 의미하는 중대한 전환점이었습니다.



### 2.2 성능 확장의 시대: v3와 v4



TPU v3(2018년)는 성능을 한층 더 끌어올렸으며, 특히 증가한 전력 밀도를 감당하기 위해 '수랭식 냉각' 시스템을 도입한 점이 주목할 만합니다.6 이는 TPU가 단순한 가속기를 넘어 고성능 컴퓨팅(HPC) 영역으로 진입했음을 보여주는 상징적인 변화였습니다. TPU v4(2021년)는 성능과 효율성 면에서 또 한 번의 거대한 도약을 이루었습니다. TPU v4는 구글의 5,400억 파라미터 거대 언어 모델(LLM)인 PaLM을 학습시키는 데 6,144개의 칩이 동원되는 등, 초거대 AI 모델 시대를 여는 핵심 동력이 되었습니다.6



### 2.3 현대 시대: v5e, v6e(Trillium), 그리고 v7(Ironwood)



최신 세대 TPU는 전략적 다변화를 보여줍니다.

- **TPU v5e**: 단순히 최고 성능만을 추구하던 것에서 벗어나, '달러당 성능'과 '와트당 성능'을 최적화하는 데 초점을 맞춘 모델입니다.3 이는 생성형 AI의 폭발적인 성장으로 인해 천문학적으로 치솟은 학습 및 추론 비용을 낮춰 AI 기술의 접근성을 높이려는 전략적 선택이었습니다.12
- **TPU v6e (Trillium)**: 2024년에 발표된 6세대 TPU는 v5e의 효율성 기조를 이어가면서도 칩당 최대 4.7배 향상된 압도적인 성능, 두 배로 늘어난 HBM 용량과 대역폭, 더 빨라진 상호연결 기술을 선보였습니다. 구글의 차세대 LLM인 Gemini 2.0 학습에 활용되었습니다.6
- **TPU v7 (Ironwood)**: 2025년 말 공개 예정인 7세대는 '추론의 시대(Age of Inference)'를 정조준합니다. 192GB에 달하는 막대한 HBM 용량과 연산 능력을 갖추고, '생각하는 모델(thinking models)'을 효율적으로 서비스하기 위해 특별히 설계되었습니다.6



### 2.4 특화된 가속: SparseCore의 부상



TPU v5p부터 도입되고 후속 세대에서 강화된 'SparseCore'는 ASIC 철학의 정수를 보여줍니다. 이는 추천 시스템, 순위 모델, 그리고 일부 생성형 AI에서 흔히 발견되는 초거대 희소 임베딩(sparse embedding) 테이블 처리를 전담하는 TPU 내의 또 다른 가속기입니다.13 이 까다로운 작업을 주력 연산 유닛인 TensorCore에서 분리(offload)함으로써 전체 시스템의 효율을 극대화합니다.16

이처럼 TPU의 발전 로드맵은 단순한 기술 사양의 나열이 아닙니다. 이는 AI 산업의 흐름을 읽고 다음 병목이 무엇일지를 예측하여 선제적으로 해결하려는 구글의 전략적 연대기입니다. 2015년의 병목은 '추론'이었고, 2017-2021년의 병목은 '학습'이었습니다. 2023년의 병목은 '경제성'이었으며, 미래의 병목은 '대규모 추론 서비스'가 될 것입니다. TPU의 각 세대는 이러한 시대적 과제에 대한 구글의 해답인 셈입니다.

**표 2: TPU 세대별 핵심 사양 비교**

| 세대               | 발표 연도   | 공정 (nm) | 메모리 (유형 & 용량) | 메모리 대역폭 (GB/s) | 최고 성능 (bfloat16 TFLOPS) | 전력 (W) | 핵심 특징 / 전략                     |
| ------------------ | ----------- | --------- | -------------------- | -------------------- | --------------------------- | -------- | ------------------------------------ |
| **v1**             | 2016        | 28        | DDR3 8GiB            | 34                   | 11.5                        | 75       | 추론 전용, INT8 연산                 |
| **v2**             | 2017        | 16        | HBM 16GiB            | 600                  | 46                          | 280      | 학습 기능 추가, HBM 도입             |
| **v3**             | 2018        | 16        | HBM 32GiB            | 900                  | 126                         | 220      | 성능 향상, 수랭식 냉각               |
| **v4**             | 2021        | 5         | HBM 16GiB            | 819                  | 275                         | 170      | PaLM 등 초거대 모델 학습             |
| **v5e**            | 2023        | 5         | HBM 95GiB            | 2765                 | 197                         | 미공개   | 비용 효율성(달러당 성능) 최적화      |
| **v5p**            | 2023        | 4         | HBM 32GiB            | 1640                 | 459                         | 미공개   | 최고 성능 추구, SparseCore 도입      |
| **v6e (Trillium)** | 2024        | ?         | HBM 192GiB           | 7200                 | 918                         | 미공개   | Gemini 2.0 학습, v5e 대비 4.7배 성능 |
| **v7 (Ironwood)**  | 2025 (예정) | ?         | HBM 192GiB           | 7200                 | ?                           | 미공개   | '추론의 시대' 특화, 대용량 메모리    |

자료: 6 기반 종합



## 3. IV. 칩을 넘어 시스템으로: TPU Pod, AI 하이퍼컴퓨터, 그리고 광학 회로 스위치(OCS)



현대 AI의 거대한 모델들은 단일 칩의 성능만으로는 감당할 수 없습니다. 진정한 경쟁력은 수천, 수만 개의 칩을 얼마나 효율적으로 연결하고 제어하는가, 즉 시스템 수준의 아키텍처에서 나옵니다. 구글은 이 분야에서 가장 혁신적인 접근 방식을 보여주고 있습니다.



### 3.1 Pod의 힘: 선형적인 확장성



TPU Pod는 수천 개의 TPU 칩을 전용 고속, 저지연 상호연결망(ICI, Inter-Chip Interconnect)으로 묶은 거대한 컴퓨팅 클러스터입니다.6 이 설계의 핵심은 워크로드를 수많은 칩에 분산시켰을 때, 성능이 칩 수에 비례하여 거의 선형적으로(linearly) 확장된다는 점입니다. 이는 일반적인 이더넷(Ethernet) 네트워킹 환경에서는 달성하기 매우 어려운 목표입니다.6 예를 들어, TPU v4 Pod 하나는 4,096개의 칩을 포함하며, 이들이 마치 하나의 거대한 칩처럼 유기적으로 동작합니다.6



### 3.2 AI 하이퍼컴퓨터: 총체적 시스템



구글은 자사의 AI 인프라를 단순히 '칩'이나 '서버'가 아닌 'AI 하이퍼컴퓨터(AI Hypercomputer)'라고 부릅니다.13 이는 단순한 마케팅 용어를 넘어, 구글의 핵심 철학을 담고 있습니다. AI 하이퍼컴퓨터는 컴퓨팅(TPU), 네트워킹(ICI/OCS), 스토리지, 소프트웨어(JAX, Pathways) 등 AI 워크로드에 필요한 모든 구성 요소를 처음부터 함께 설계하고 최적화한 총체적인 시스템(holistic system)을 의미합니다.13 각 부품이 따로 노는 것이 아니라, 오직 하나의 목표, 즉 거대 AI 워크로드를 가장 효율적으로 실행하기 위해 완벽하게 통합된 유기체인 셈입니다.



### 3.3 네트워킹의 혁명: 광학 회로 스위치 (Optical Circuit Switch, OCS)



OCS는 시스톨릭 어레이만큼이나 혁신적인 기술입니다. TPU v4 이후의 Pod에서 구글은 데이터센터 네트워킹의 패러다임을 바꾸는 OCS를 도입했습니다.3 기존의 전기 스위치 기반 네트워크에서는 데이터 패킷이 목적지까지 도달하기 위해 여러 스위치를 거쳐야 하며, 이 과정에서 지연 시간과 전력 소모가 발생합니다. 반면 OCS는 Pod 내의 어떤 두 칩 사이든, 빛을 이용해 물리적인 직접 경로를 동적으로 생성할 수 있습니다.17

이 네트워크는 필요에 따라 실시간으로 재구성(reconfigurable)이 가능합니다. 이는 특정 AI 모델이 요구하는 독특한 통신 패턴에 맞춰 네트워크 토폴로지 자체를 최적화할 수 있음을 의미합니다.17 예를 들어, 어떤 모델은 올-투-올(all-to-all) 통신이 중요하고, 다른 모델은 링(ring) 형태의 통신이 중요할 수 있습니다. OCS는 이러한 요구에 맞춰 광경로를 즉시 변경하여 통신 효율을 극대화합니다.

구글의 진정한 경쟁 우위는 TPU 칩 자체보다, 이처럼 전례 없는 규모를 가능하게 하는 시스템 수준의 아키텍처, 특히 OCS에 있습니다. 경쟁사가 TPU와 유사한 칩을 설계할 수는 있겠지만, 데이터센터 전체를 아우르는 OCS 기반의 광학 패브릭(optical fabric)을 복제하는 것은 훨씬 더 어려운 과제입니다. NVIDIA의 NVLink/NVSwitch가 단일 서버 내에서의 강력한 스케일업(scale-up) 솔루션이라면, 구글의 OCS는 데이터센터 전체를 아우르는 혁명적인 스케일아웃(scale-out) 솔루션입니다. 이 시스템 수준의 혁신이야말로 칩 아키텍처 자체보다 더 깊고 방어하기 어려운 '해자(moat)'인 것입니다.



## 4. V. 성능 벤치마크 분석: TPU 대 GPU, MLPerf를 통해 본 경쟁 구도



하드웨어의 성능을 객관적으로 평가하기 위해서는 표준화된 벤치마크가 필수적입니다. AI 분야에서 가장 권위 있는 벤치마크는 업계 표준 컨소시엄인 MLCommons가 주관하는 MLPerf입니다. MLPerf는 학습(Training)과 추론(Inference) 성능을 다양한 모델과 시나리오에 걸쳐 측정하여 공정한 비교의 잣대를 제공합니다.19



### 4.1 학습 성능 대결



MLPerf 및 여러 연구 결과는 TPU와 GPU 간의 치열한 경쟁을 보여줍니다.

- 구글이 발표한 연구에 따르면, TPU v4는 동시대의 경쟁 칩인 NVIDIA A100 GPU보다 1.2배에서 1.7배 더 빠른 학습 성능을 보였습니다.22
- TPU v5p는 이전 세대인 v4보다 거대 언어 모델(LLM) 학습 속도가 최대 2.8배 빠르다고 보고되었으며 13, 이는 NVIDIA의 H100과 직접적으로 경쟁하는 성능 수준입니다.23
- 최신 6세대 TPU인 Trillium(v6e)은 LaMDA나 GPT와 같은 고밀도 LLM을 v5e보다 최대 4배 더 빠르게 훈련할 수 있다고 알려졌습니다.15



### 4.2 추론 및 비용 효율성 (달러당 성능)



추론 성능과 비용 효율성은 TPU가 특히 강점을 보이는 영역입니다. 대규모 서비스를 운영하는 기업에게는 초기 학습 비용보다 지속적으로 발생하는 추론 비용이 훨씬 중요하기 때문입니다.

- TPU v5e는 추론 워크로드에서 v4 대비 달러당 성능이 2.7배 더 높습니다.12
- 한 분석에 따르면, LLM 학습 시 TPU v5e 클러스터는 H100 클러스터보다 토큰당 비용(cost per billion tokens)이 50-70% 더 저렴할 수 있습니다.24
- 최신 Trillium TPU는 v5e에 비해 달러당 추론 성능이 3배 더 높다고 발표되었습니다.25



### 4.3 MLPerf 5.0 심층 분석 (Trillium vs. v5e)



가장 최근의 MLPerf 5.0 결과는 Trillium의 비약적인 성능 향상을 명확히 보여줍니다.

- **LLM 추론**: JetStream 추론 엔진을 사용했을 때, Trillium은 Mixtral 8x7B 모델에서 v5e 대비 2.8배, Llama 2 70B 모델에서는 2.9배 더 높은 처리량을 기록했습니다.25
- **이미지 생성**: Stable Diffusion XL 모델의 추론 성능(초당 쿼리 수)은 v5e 대비 3.5배 향상되었습니다.25
- **구글 클라우드의 NVIDIA GPU 제출**: 구글은 TPU뿐만 아니라 자사 클라우드에서 제공하는 NVIDIA GPU의 성능 결과도 제출합니다. MLPerf 5.0에서 구글은 NVIDIA H200 GPU(A3 Ultra VM)의 강력한 성능을 입증했으며, 업계에서 유일하게 차세대 칩인 NVIDIA B200(A4 VM)의 결과를 제출하여 다양한 하드웨어 포트폴리오를 제공하려는 전략을 보여주었습니다.25

이러한 벤치마크 결과는 구글의 TPU 전략이 두 갈래로 진행되고 있음을 시사합니다. 한편으로는 v5p와 같은 'p' 모델을 통해 NVIDIA의 최상위 칩(H100/H200)과 최고 성능을 두고 경쟁하고, 다른 한편으로는 v5e, Trillium과 같은 'e' 모델을 통해 비용 효율성(TCO)에서 상대를 압도하려는 것입니다. 이는 고객에게 어려운 선택지를 제시합니다. 절대적으로 가장 빠른 속도가 필요하다면 여전히 H100/H200을 선택할 수 있지만, 비용에 민감한 대규모 추론 서비스나 학습 워크로드를 운영한다면 TPU의 달러당 성능이 훨씬 매력적일 수 있습니다. 이 이중 제품 전략은 단순히 '누가 더 빠른가'의 경쟁을 넘어 'AI 예산을 어떻게 가장 현명하게 사용할 것인가'라는 경제적 논쟁으로 경쟁의 구도를 재편하며, NVIDIA의 고마진 비즈니스 모델에 효과적으로 도전하고 있습니다.26

**표 3: 최신 MLPerf 벤치마크 결과 비교 (Inference v5.0 기준)**

| 벤치마크 (모델)                       | 시스템                                 | 메트릭                | 결과 (높을수록 좋음) |
| ------------------------------------- | -------------------------------------- | --------------------- | -------------------- |
| **LLM 추론 (Llama 3.1 405B)**         | Google Cloud A4 (8x NVIDIA B200)       | Server - Queries/Sec  | 118.9                |
| **LLM 추론 (Llama 2 70B)**            | Google Cloud Trillium (8x TPU v6e)     | Server - Queries/Sec  | 10,790               |
| **LLM 추론 (Llama 2 70B)**            | Google Cloud A3 Ultra (8x NVIDIA H200) | Server - Queries/Sec  | 10,310               |
| **이미지 생성 (Stable Diffusion XL)** | Google Cloud Trillium (8x TPU v6e)     | Offline - Samples/Sec | 185.7                |
| **이미지 생성 (Stable Diffusion XL)** | Google Cloud A3 Ultra (8x NVIDIA H200) | Offline - Samples/Sec | 148.2                |

자료: 25 기반 종합. MLPerf Inference v5.0 Datacenter Closed 부문 결과. 절대적인 성능 비교는 칩 수, 소프트웨어 스택 등 다양한 요인에 따라 달라질 수 있음.



## 5. VI. 소프트웨어 생태계와 개발 환경: JAX와 PyTorch/XLA의 도전과 기회



최고의 하드웨어도 그것을 쉽고 효율적으로 사용할 수 있는 소프트웨어가 없다면 무용지물입니다. TPU의 잠재력을 끌어내는 열쇠는 소프트웨어 생태계에 있으며, 이는 기회인 동시에 가장 큰 도전 과제이기도 합니다.



### 5.1 핵심 열쇠: XLA 컴파일러



TPU의 마법은 XLA(Accelerated Linear Algebra) 컴파일러에서 시작됩니다. XLA는 TensorFlow, PyTorch, JAX와 같은 프레임워크에서 생성된 고수준 연산 그래프를 입력받아, TPU의 시스톨릭 어레이 아키텍처에 최적화된 기계 코드로 컴파일하는 역할을 합니다.5 이 과정을 통해 하드웨어의 성능을 극한까지 활용할 수 있습니다.



### 5.2 새로운 패러다임: JAX와 함수형 프로그래밍



JAX는 NumPy와 매우 유사한 API를 제공하여 많은 개발자에게 친숙하게 다가가지만, 그 내부는 가속기(GPU/TPU)와 자동 미분에 최적화되어 있습니다.28 JAX를 사용하는 데 있어 가장 큰 장벽은 '함수형 프로그래밍(Functional Programming)' 패러다임을 따라야 한다는 점입니다. JAX의 핵심 기능인 

`jit` (Just-In-Time compilation), `grad` (autodiff) 등은 외부 상태를 변경하지 않는 '순수 함수(pure functions)'에 대해서만 올바르게 작동하도록 설계되었습니다.28

이는 전역 변수를 수정하거나, 함수 내에서 `print` 문을 사용하는 등의 '부작용(side effect)'이 있는 코드를 지양해야 함을 의미합니다. PyTorch와 같은 명령형(imperative) 스타일에 익숙한 개발자에게 이는 완전히 새로운 사고방식을 요구하며, 상당한 학습 곡선을 유발합니다. 만약 순수 함수 규칙을 제대로 지키지 않으면, 프로그램은 오류를 내뱉지 않고 조용히 잘못된 결과를 계산할 수 있어 디버깅을 매우 어렵게 만듭니다.28



### 5.3 간극을 메우다: PyTorch/XLA



거대한 PyTorch 사용자 커뮤니티를 지원하기 위해 구글은 PyTorch/XLA 라이브러리를 제공합니다. 이 라이브러리를 사용하면 기존 PyTorch 코드를 최소한의 수정만으로 TPU에서 실행할 수 있습니다.30 하지만 이 편의성은 새로운 복잡성을 동반합니다.

- **지연 실행 (Lazy Execution)**: PyTorch/XLA는 연산을 즉시 실행하지 않고 그래프에 기록해 두었다가, 실제 결과값이 필요한 시점에 한꺼번에 컴파일하여 실행합니다.31 이는 최적화에는 유리하지만, 오류가 발생했을 때 어느 코드 라인에서 문제가 생겼는지 추적하기 어렵게 만듭니다.
- **재컴파일 (Recompilation)**: 학습 도중 입력되는 텐서의 크기(shape)가 바뀌면 XLA는 그래프를 새로 컴파일해야 합니다. 이 재컴파일 과정은 매우 비용이 많이 들어 성능을 급격히 저하시키는 주된 원인이 되며, 초심자들이 가장 흔하게 겪는 문제입니다.32
- **멀티코어 프로그래밍**: Colab의 TPU 8개 코어를 모두 사용하려면 `xmp.spawn`과 같은 고유한 멀티프로세싱 방식을 사용해야 하는데, 이는 일반적인 GPU의 데이터 병렬 처리(Data Parallelism)와는 다른 접근 방식입니다.30



### 5.4 CUDA라는 거대한 성벽



이 모든 상황의 배경에는 NVIDIA의 CUDA 플랫폼이 있습니다. CUDA는 2006년에 출시된 이래 십수 년간 성숙하며 방대하고 안정적인 생태계를 구축했습니다. 수백만 명의 개발자에게 익숙하며, 대부분의 AI 프레임워크와 라이브러리가 CUDA를 중심으로 발전해왔습니다. 이 강력한 소프트웨어 생태계는 종종 경쟁사에게 하드웨어 자체의 성능 격차보다 더 넘기 힘든 '해자(moat)'로 작용합니다.26

결국 TPU와 GPU 사이의 선택은 하드웨어의 잠재 성능뿐만 아니라, 개발자의 '인지적 부하(cognitive overhead)'에 의해 결정되는 경우가 많습니다. 새로운 프로그래밍 패러다임을 배우거나(JAX), 호환성 계층의 특이한 동작을 디버깅하는(PyTorch/XLA) 데 드는 시간과 노력은 많은 팀에게 잠재적인 성능 향상보다 더 큰 비용으로 다가올 수 있습니다. 따라서 약간 덜 빠르더라도 예측 가능하고 마찰이 적은 GPU 환경이, 잠재적으로는 더 빠르지만 복잡하고 위험 부담이 있는 TPU 환경보다 합리적인 선택이 될 수 있습니다. 구글의 과제는 단순히 더 좋은 하드웨어를 만드는 것을 넘어, 그 하드웨어를 사용하는 경험을 경쟁자만큼이나 매끄럽게 만드는 것입니다.



## 6. VII. 산업 적용 사례 연구: LLM 시대를 가속하는 TPU



TPU의 이론적 우수성은 실제 산업 현장에서 어떻게 발현되고 있을까요? 특히 생성형 AI가 촉발한 전례 없는 컴퓨팅 자원 부족 사태는 TPU가 그 가치를 증명하는 중요한 시험대가 되었습니다.



### 6.1 사례 연구 1: 카카오의 '카나나(Kanana)' LLM



카카오는 자체 AI 브랜드 '카나나'의 거대 언어 모델을 개발하는 과정에서 심각한 GPU 자원 부족과 수급 불안정 문제에 직면했습니다.14 이 문제를 해결하기 위해 카카오는 TPU v3부터 최신 Trillium(v6e)에 이르기까지 TPU를 도입하는 과감한 결정을 내렸습니다. TPU 도입의 핵심적인 이유는 강력한 컴퓨팅 자원을 안정적으로 확보할 수 있다는 점이었습니다.36 이를 통해 카카오는 자원 제약에서 벗어나 대규모 한국어 모델을 신속하게 개발하고 고도화할 수 있었습니다. 한 관계자는 "처음에는 연구원들도 낯선 시스템에 대한 불안감이 있었지만, 필요한 시스템 자원을 정확히 예측하고 확보할 수 있다는 점에서 GPU와 비교할 수 없을 만큼 좋았다"고 언급하며, TPU가 제공하는 예측 가능성과 안정성을 높이 평가했습니다.36



### 6.2 사례 연구 2: 엔씨소프트의 '바르코(VARCO)' LLM



국내 대표 게임사인 엔씨소프트는 자체 언어 모델 '바르코' 개발에 구글 클라우드 TPU를 적극적으로 활용했습니다. 바르코는 게임 내에서 플레이어와 실시간으로 상호작용하는 NPC(Non-Player Character)를 만드는 등 다양한 AI 서비스의 기반이 됩니다.37 엔씨소프트가 TPU를 선택한 주된 이유는 ▲학습 시간 단축을 통한 생산성 향상 ▲GPU 대비 뛰어난 비용 효율성 ▲GPU 수급난 속에서의 탄력적인 인프라 운영 가능성이었습니다.37 특히 엔씨소프트는 "클라우드 TPU가 GPU 대비 달러당 거의 2배 높은 성능을 제공하면서, 대규모 AI 학습 워크로드의 성능과 비용을 최적화할 수 있었다"고 밝혔습니다.37



### 6.3 적용 분야의 확장



TPU의 활용은 IT 기업에만 국한되지 않습니다. 글로벌 제약사 바이엘(Bayer)과 딥지노믹스(Deep Genomics)는 신약 개발 및 RNA 치료제 연구에 TPU를 도입하여 AI 모델을 효율적으로 구동하고 있으며 15, 세일즈포스(Salesforce)는 자사의 기반 모델 사전 학습에 TPU v5p를 활용하여 훈련 속도를 크게 향상시켰다고 보고했습니다.13 이는 TPU가 빅테크의 내부 서비스를 넘어, 다양한 과학 및 엔터프라이즈 영역으로 빠르게 확산되고 있음을 보여줍니다.

이 사례들은 중요한 시사점을 던져줍니다. 생성형 AI 붐으로 인한 만성적인 GPU 부족 사태는 역설적으로 구글 TPU에게 가장 강력한 마케팅 도구가 되었습니다. GPU를 구하지 못해 발을 동동 구르던 기업들은 어쩔 수 없이 대안을 평가해야만 했습니다. 그리고 그 과정에서 많은 기업들이 TPU가 단순히 '차선책'이 아니라, 비용, 성능, 그리고 무엇보다 '가용성' 측면에서 자사의 대규모 AI 전략에 더 적합한 '최선책'일 수 있음을 발견하게 된 것입니다. 결국 NVIDIA의 시장 지배력이 만들어낸 '희소성'이라는 문제가, 그 최대 경쟁자에게 시장의 발판을 마련해준 셈입니다.



## 7. VIII. 개발자를 위한 TPU 활용 가이드: Colab에서 클라우드까지



TPU를 활용하는 것은 더 이상 소수의 전문가에게만 국한된 일이 아닙니다. 구글은 무료 개발 환경부터 대규모 클라우드 서비스까지 다양한 접근 경로를 제공합니다.



### 7.1 무료로 시작하기: 구글 코랩 (Google Colaboratory)



TPU를 처음 접하는 개발자나 학생에게 구글 코랩은 최고의 출발점입니다. 코랩 노트북에서 몇 번의 클릭만으로 TPU를 사용할 수 있습니다.

1. **런타임 유형 변경**: 코랩 메뉴에서 '런타임' > '런타임 유형 변경'을 선택합니다.
2. **하드웨어 가속기 선택**: 드롭다운 메뉴에서 'TPU'를 선택하고 저장합니다.39
3. **TPU 연결 확인**: 코드를 실행하여 TPU가 정상적으로 할당되었는지 확인할 수 있습니다. 일반적으로 `os.environ`을 확인하여 TPU 주소를 얻습니다.40
4. **구글 드라이브 연동**: 대용량 데이터나 코드를 관리하기 위해 구글 드라이브를 마운트하여 사용하는 것이 일반적입니다.39

코랩은 컴퓨터 사양에 관계없이 누구나 브라우저만으로 TPU를 포함한 구글의 강력한 하드웨어를 실험해볼 수 있는 환경을 제공합니다.42



### 7.2 전문적인 개발 환경: 구글 클라우드 플랫폼 (GCP)



실제 상용 서비스나 대규모 연구를 위해서는 GCP를 사용해야 합니다. GCP에서는 주로 두 가지 방식으로 TPU를 활용합니다.

- **Cloud TPU VM**: 최신 접근 방식으로, 개발자에게 TPU가 연결된 가상 머신(VM)에 대한 직접적인 SSH 접근 권한을 제공합니다.5 이를 통해 VM에 필요한 라이브러리를 자유롭게 설치하고, 디버깅 로그에 직접 접근하는 등 훨씬 유연하고 편리한 개발이 가능합니다.
- **Google Kubernetes Engine (GKE)**: 대규모의 컨테이너화된 학습 및 추론 워크로드를 오케스트레이션하는 데 사용됩니다. GKE를 통해 TPU 리소스를 탄력적으로 관리하고 배포를 자동화할 수 있습니다.43



### 7.3 실용 예제: 멀티코어 PyTorch/XLA 학습



Colab의 TPU는 8개의 코어를 가지고 있으며, 이를 모두 활용하면 학습 속도를 크게 높일 수 있습니다. PyTorch/XLA를 사용한 멀티코어 학습의 핵심 개념은 다음과 같습니다.30

1. **라이브러리 임포트**: `torch_xla`와 `torch_xla.distributed.xla_multiprocessing` (줄여서 `xmp`)를 임포트합니다.
2. **`map_fn` 함수 정의**: 8개의 각 코어(프로세스)에서 실행될 학습 루프 전체를 하나의 함수(`map_fn`) 안에 정의합니다. 이 함수 안에서 모델 초기화, 데이터 로더 생성, 학습 및 평가 로직이 모두 포함됩니다.
3. **`xmp.spawn`으로 실행**: `xmp.spawn` 함수를 호출하여 위에서 정의한 `map_fn`을 8개의 프로세스에 걸쳐 실행시킵니다. `xmp.spawn`은 각 프로세스에 적절한 TPU 코어를 할당하고 병렬 학습을 시작합니다. 이는 일반적인 GPU 프로그래밍과는 다른, TPU 멀티코어 활용을 위한 고유한 패턴입니다.



### 7.4 데이터 처리의 핵심: 구글 클라우드 스토리지 (GCS)



대규모 TPU 학습 시, 데이터는 로컬 머신이 아닌 구글 클라우드 스토리지(GCS) 버킷에 저장되어야 합니다. 코드 내에서 데이터셋의 경로를 지정할 때 `gs://<버킷_이름>/<데이터_경로>`와 같은 GCS 경로를 사용해야 합니다.40 이는 모든 TPU 노드가 데이터에 빠르고 일관되게 접근할 수 있도록 하기 위한 필수적인 과정입니다.



## 8. IX. 종합 고찰 및 미래 전망: AI 반도체 시장의 지각 변동과 TPU의 역할



구글 TPU는 단순한 고성능 칩을 넘어, AI 반도체 시장의 판도를 바꾸고 미래 컴퓨팅의 방향성을 제시하는 중요한 플레이어입니다. TPU에 대한 종합적인 고찰은 AI 하드웨어의 근본적인 트레이드오프와 미래 시장의 동학을 이해하는 데 필수적입니다.



### 8.1 전문가의 딜레마: 특화와 범용성



TPU와 GPU의 경쟁은 '특화(Specialization)'와 '범용성(Generalization)'이라는 고전적인 공학적 트레이드오프를 극명하게 보여줍니다.

- **GPU는 '스위스 군용 칼'**: 그래픽 처리, 과학 계산, 다양한 AI 워크로드 등 여러 작업을 능숙하게 처리하는 유연한 범용 도구입니다.44 이 범용성은 방대한 개발자 생태계와 결합하여 강력한 진입 장벽을 구축합니다.
- **TPU는 '외과 의사의 메스'**: 오직 신경망의 텐서 연산이라는 단 하나의 작업을 가장 빠르고 효율적으로 수행하기 위해 설계된 정밀 도구입니다.44 이 특화성은 대규모 AI 워크로드에서 압도적인 성능과 효율성을 제공하지만, 유연성이 부족하고 특정 생태계에 종속된다는 단점을 가집니다.



### 8.2 왕좌에 대한 도전: AI 가속기 시장의 동학



AI 가속기 시장은 폭발적인 성장을 거듭하고 있습니다. 2024년 약 255억 달러 규모에서 2033년에는 2,568억 달러를 넘어설 것으로 전망됩니다.45 현재 이 시장은 NVIDIA가 약 90%에 달하는 점유율로 독점적인 지위를 누리고 있습니다.46 하지만 이 구도에 균열이 생기기 시작했습니다. 구글, 아마존, 마이크로소프트와 같은 하이퍼스케일러들은 NVIDIA에 대한 의존도를 줄이고 비용을 절감하기 위해 자체 맞춤형 칩(ASIC) 개발에 막대한 투자를 하고 있습니다.47 구글 TPU의 성공적인 상용화와 산업계 도입은 이러한 '자체 칩' 개발 트렌드를 선도하며, NVIDIA의 아성에 실질적인 위협이 될 수 있음을 증명하고 있습니다.49



### 8.3 미래는 추론, 효율성, 그리고 시스템에 있다



앞으로 AI 가속기 시장의 경쟁은 세 가지 핵심 축을 중심으로 전개될 것입니다.

1. **추론의 시대 도래**: 모델을 한 번 학습시키는 비용도 크지만, 수십억 명의 사용자에게 서비스를 제공하는 데 드는 누적 추론 비용은 이를 훨씬 상회할 것입니다.26 Ironwood와 같이 추론에 특화된 TPU의 등장은 구글이 이 거대한 미래 시장을 전략적으로 겨냥하고 있음을 보여줍니다.16
2. **전력이라는 궁극적 제약**: 데이터센터가 기하급수적으로 팽창하면서, 이제는 컴퓨팅 성능보다 전력 공급과 냉각이 성장의 발목을 잡는 새로운 병목으로 떠오르고 있습니다.48 맞춤형 설계와 수랭식 냉각 등을 통해 달성한 TPU의 뛰어난 '와트당 성능'은 앞으로 더욱 중요한 경쟁 우위가 될 것입니다.16
3. **시스템을 넘어선 경쟁**: 미래의 승자는 가장 빠른 칩을 가진 회사가 아니라, 가장 효율적이고 확장성 있는 '시스템'을 가진 회사가 될 것입니다. 칩, 네트워킹(OCS), 스토리지, 소프트웨어를 하나로 묶는 구글의 'AI 하이퍼컴퓨터' 전략은 이러한 미래를 향한 청사진입니다. 하드웨어와 소프트웨어의 경계가 허물어지고, 양자를 통합하여 최적화하는 능력이 경쟁의 핵심이 될 것입니다.51

**표 4: TPU vs. GPU 장단점 및 선택 가이드**

| 결정 요인                | GPU (예: NVIDIA H100)         | TPU (예: Google v5e/v6e)           | 추천 / 최적 활용 사례                                 |
| ------------------------ | ----------------------------- | ---------------------------------- | ----------------------------------------------------- |
| **단일 칩 최고 성능**    | 매우 높음                     | 매우 높음 (p 모델)                 | 최고 성능이 절대적으로 중요한 연구 및 개발            |
| **대규모 확장성**        | 높음 (NVLink/NVSwitch)        | 극도로 높음 (Pod/OCS)              | 수천 개 이상의 칩을 사용하는 초거대 모델 학습         |
| **달러당 성능**          | 보통                          | 매우 높음 (e 모델)                 | 비용에 민감한 대규모 추론 서비스, 상용 AI 학습        |
| **와트당 성능**          | 보통                          | 높음                               | 전력 및 냉각이 제약된 데이터센터 환경                 |
| **개발자 생태계/유연성** | 매우 넓고 성숙 (CUDA)         | 제한적 (JAX, PyTorch/XLA)          | 다양한 모델과 프레임워크를 사용하는 빠른 프로토타이핑 |
| **학습 곡선**            | 낮음                          | 높음 (특히 JAX)                    | 새로운 프로그래밍 패러다임 학습 의지가 있는 팀        |
| **주요 사용 사례**       | 프로토타이핑, 범용 AI, 그래픽 | 대규모 모델 학습, 비용 효율적 추론 | 구글 클라우드 기반의 대규모 AI 워크로드               |

자료: 24 기반 종합

결론적으로 구글 TPU는 AI 시대의 컴퓨팅 패러다임을 근본적으로 바꾸고 있는 혁신적인 기술입니다. 시스톨릭 어레이와 광학 회로 스위치라는 독보적인 아키텍처를 통해 대규모 AI 워크로드에서 탁월한 성능과 효율성을 제공하며, NVIDIA가 지배하는 시장에 의미 있는 대안을 제시하고 있습니다. 미래 AI 경쟁의 핵심이 '효율성'과 '시스템'으로 이동함에 따라, TPU의 전략적 가치는 더욱 커질 것으로 전망됩니다.



#### **참고 자료**

1. [HW] CPU, GPU, TPU 비교 - 100s FE.log - 티스토리, accessed July 13, 2025, https://100s.tistory.com/28
2. [트렌드] HW알아보기 with CPU, GPU, NPU, DPU - 낭만주의자, accessed July 13, 2025, https://nangmandeveloper.tistory.com/20
3. Tensor Processing Unit(TPU) - Google Cloud, accessed July 13, 2025, https://cloud.google.com/tpu?hl=ko
4. CPU GPU NPU TPU 훑어보기 - velog, accessed July 13, 2025, https://velog.io/@pnuaid1020/CPU-GPU-NPU-TPU
5. TPU 아키텍처 | Google Cloud, accessed July 13, 2025, https://cloud.google.com/tpu/docs/system-architecture-tpu-vm?hl=ko
6. TPU - 나무위키, accessed July 13, 2025, https://namu.wiki/w/TPU
7. TPU란 무엇인가? - Jordano - 티스토리, accessed July 13, 2025, https://jordano-jackson.tistory.com/110
8. google TPU 논문 리뷰: In-Datacenter Performance Analysis of a Tensor Processing Unit (1) - TPU 구조 및 명령어 - 전공할 시간 - 티스토리, accessed July 13, 2025, https://justmajor.tistory.com/7
9. [반알못을 부탁해] AI 최강기업 구글의 핵심기술 'TPU' - 바이라인네트워크, accessed July 13, 2025, https://byline.network/2020/12/17-114/
10. 구글 TPU 들여보기 - 골수공돌이의 탐구실, accessed July 13, 2025, [https://www.donghyun53.net/%EA%B5%AC%EA%B8%80-tpu-%EB%93%A4%EC%97%AC%EB%B3%B4%EA%B8%B0/](https://www.donghyun53.net/구글-tpu-들여보기/)
11. Keras 및 TPU를 사용한 컨볼루션 신경망 - Google Codelabs, accessed July 13, 2025, https://codelabs.developers.google.com/codelabs/keras-flowers-convnets?hl=ko
12. Performance per dollar of GPUs and TPUs for AI inference | Google Cloud Blog, accessed July 13, 2025, https://cloud.google.com/blog/products/compute/performance-per-dollar-of-gpus-and-tpus-for-ai-inference
13. Introducing Cloud TPU v5p and AI Hypercomputer | Google Cloud Blog, accessed July 13, 2025, https://cloud.google.com/blog/products/ai-machine-learning/introducing-cloud-tpu-v5p-and-ai-hypercomputer
14. 구글 클라우드 "6세대 TPU 성능 4.7배 향상…카카오도 LLM 학습에 활용" - Daum, accessed July 13, 2025, https://v.daum.net/v/20250116125535120
15. 카카오도 반한 TPU '트릴리움'...구글 "LLM 학습에 최적 AI 칩" - Daum, accessed July 13, 2025, https://v.daum.net/v/20250116153933195
16. Ironwood: The first Google TPU for the age of inference, accessed July 13, 2025, https://blog.google/products/google-cloud/ironwood-tpu-age-of-inference/
17. Google Details TPUv4 and Its Crazy Optically Reconfigurable AI Network : r/OpenAI - Reddit, accessed July 13, 2025, https://www.reddit.com/r/OpenAI/comments/1659hg3/google_details_tpuv4_and_its_crazy_optically/
18. Google Details TPUv4 and its Crazy Optically Reconfigurable AI ..., accessed July 13, 2025, https://www.servethehome.com/google-details-tpuv4-and-its-crazy-optically-reconfigurable-ai-network/
19. Benchmark MLPerf Training | MLCommons Version 2.0 Results, accessed July 13, 2025, https://mlcommons.org/benchmarks/training/
20. Benchmark MLPerf Inference: Datacenter | MLCommons V3.1, accessed July 13, 2025, https://mlcommons.org/benchmarks/inference-datacenter/
21. Cloud TPU breaks AI inference records | Google Cloud Blog, accessed July 13, 2025, https://cloud.google.com/blog/products/ai-machine-learning/cloud-tpu-breaks-scalability-records-for-ai-inference
22. CPU•GPU•NPU•TPU의 차이 - 테크튜브, accessed July 13, 2025, https://www.techtube.co.kr/news/articleView.html?idxno=3644
23. Google is rapidly turning into a formidable opponent to BFF Nvidia — the TPU v5p AI chip powering its hypercomputer is faster and has more memory and bandwidth than ever before, beating even the mighty H100 | TechRadar, accessed July 13, 2025, https://www.techradar.com/pro/google-is-rapidly-turning-into-a-formidable-opponent-to-bff-nvidia-the-tpu-v5p-ai-chip-powering-its-hypercomputer-is-faster-and-has-more-memory-and-bandwidth-than-ever-before-beating-even-the-mighty-h100
24. Cloud AI Platforms Comparison: AWS Trainium vs ... - CloudExpat, accessed July 13, 2025, https://www.cloudexpat.com/blog/comparison-aws-trainium-google-tpu-v5e-azure-nd-h100-nvidia/
25. AI Hypercomputer inference updates for Google Cloud TPU and GPU, accessed July 13, 2025, https://cloud.google.com/blog/products/compute/ai-hypercomputer-inference-updates-for-google-cloud-tpu-and-gpu
26. Google TPU v5p beats Nvidia H100 - Hacker News, accessed July 13, 2025, https://news.ycombinator.com/item?id=39148544
27. Introduction to Cloud TPU | Google Cloud, accessed July 13, 2025, https://cloud.google.com/tpu/docs/intro-to-tpu
28. Why You Should (or Shouldn't) be Using Google's JAX in 2023 - AssemblyAI, accessed July 13, 2025, https://www.assemblyai.com/blog/why-you-should-or-shouldnt-be-using-jax-in-2023
29. Why You Should (or Shouldn't) be Using Google's JAX in 2023, accessed July 13, 2025, https://www.assemblyai.com/blog/why-you-should-or-shouldnt-be-using-jax-in-2023/
30. Colab에서 PyTorch 모델 TPU로 학습하기 - Beomi's Tech blog, accessed July 13, 2025, https://beomi.github.io/2020/02/24/Pytorch-with-TPU-on-Colab/
31. PyTorch on XLA Devices — PyTorch/XLA master documentation, accessed July 13, 2025, https://docs.pytorch.org/xla/release/2.1/index.html
32. Troubleshoot — PyTorch/XLA master documentation, accessed July 13, 2025, https://docs.pytorch.org/xla/release/r2.6/learn/troubleshoot.html
33. TPU vs GPU: What's the Difference in 2025? - CloudOptimo, accessed July 13, 2025, https://www.cloudoptimo.com/blog/tpu-vs-gpu-what-is-the-difference-in-2025/
34. TPU vs GPU in AI: A Comprehensive Guide to Their Roles and Impact on Artificial Intelligence - Wevolver, accessed July 13, 2025, https://www.wevolver.com/article/tpu-vs-gpu-in-ai-a-comprehensive-guide-to-their-roles-and-impact-on-artificial-intelligence
35. [현장] "GPU 병목 넘는다"…구글 클라우드 '트릴리움' TPU로 AI 혁신 본격화 - 지디넷코리아, accessed July 13, 2025, https://zdnet.co.kr/view/?no=20250116114457
36. 카카오의 AI 모델 '카나나', 구글 클라우드 TPU로 남들과 다른 길을 걷게 된 이야기 - Google Cloud, accessed July 13, 2025, https://cloud.google.com/blog/ko/topics/customers/kakao-kanana-ai-forging-path-with-google-cloud-tpu-trillium
37. 구글 클라우드, TPU 활용해 엔씨소프트 AI 언어모델 개발 지원 ..., accessed July 13, 2025, https://www.dailysecu.com/news/articleView.html?idxno=151138
38. 구글 클라우드, 클라우드 TPU 활용해 엔씨소프트 AI 대형언어모델 개발 지원 - 인공지능신문, accessed July 13, 2025, https://www.aitimes.kr/news/articleView.html?idxno=29347
39. [Google Colaboratory] 코랩으로 GPU, TPU 사용법 - 개발자 우성우 - 티스토리, accessed July 13, 2025, https://wscode.tistory.com/30
40. Colab TPU 이용방법 - choice - 티스토리, accessed July 13, 2025, https://choice-life.tistory.com/73
41. [COLAB] google colab 이란? & 사용법 - ChangHun's portfolio & Study - 티스토리, accessed July 13, 2025, https://kimchangheon.tistory.com/46
42. Colab 시작하기 - Colab - Google, accessed July 13, 2025, https://colab.research.google.com/?hl=ko
43. Tensor Processing Units (TPUs) - Google Cloud, accessed July 13, 2025, https://cloud.google.com/tpu
44. TPU와 GPU의 차이점 완전 정리 - artificialintelligencemachinelearning, accessed July 13, 2025, [https://artificialintelligencemachine.tistory.com/entry/%F0%9F%93%8A-TPU%EC%99%80-GPU%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EC%99%84%EC%A0%84-%EC%A0%95%EB%A6%AC](https://artificialintelligencemachine.tistory.com/entry/📊-TPU와-GPU의-차이점-완전-정리)
45. AI Accelerator Market Size & Share | Industry Report, 2033, accessed July 13, 2025, https://www.grandviewresearch.com/industry-analysis/ai-accelerator-market-report
46. AI 반도체 시장의 현황과 전망 | 인사이트리포트 | 삼성SDS, accessed July 13, 2025, https://www.samsungsds.com/kr/insights/ai-semiconductor.240306.html
47. TPU란? - 뜻 & 정의 - KB의 생각, accessed July 13, 2025, https://kbthink.com/dictionary/view.html?dictId=KED-00016336
48. 반도체부터 서비스까지…AI 생태계 알면 돈이 보인다 - 모바일한경, accessed July 13, 2025, [https://plus.hankyung.com/apps/newsinside.view?aid=202406247156c&category=&sns=y](https://plus.hankyung.com/apps/newsinside.view?aid=202406247156c&category&sns=y)
49. “엔비디아 GPU 게 섰거라”… 구글 AI 칩 'TPU' 수요 급증 - 조선비즈, accessed July 13, 2025, https://biz.chosun.com/it-science/ict/2024/12/30/QZZK2UA6HRCDRBJXFVXLVGTZYI/
50. '엔비디아 독주 끝날까?'... 구글, 오픈AI에 TPU 공급 - 매거진한경 - 한국경제, accessed July 13, 2025, https://magazine.hankyung.com/business/article/202506308898b
51. AI 시대의 반도체 산업 전망 - 브런치, accessed July 13, 2025, https://brunch.co.kr/@donghyungshin/149


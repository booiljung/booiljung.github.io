[FPGA 에 의한 컴퓨팅](./index.md)
# FPGA 기반 인공지능 추론 가속화


인공지능(AI)의 시대가 도래하면서, 복잡한 신경망 모델을 효율적으로 실행하기 위한 하드웨어 가속 기술의 중요성이 그 어느 때보다 부각되고 있습니다. 이 보고서는 재구성 가능한 하드웨어의 정점인 FPGA(Field-Programmable Gate Array)를 활용한 인공지능 추론 가속 기술에 대해 심층적으로 분석합니다. FPGA의 근본적인 아키텍처 원리부터 시작하여 GPU, ASIC 등 다른 가속기와의 경쟁 구도를 다각적으로 조명하고, 실제 기술 구현 방법론과 산업 적용 사례를 통해 그 가능성과 한계를 탐구합니다. 최종적으로는 FPGA 기반 AI 가속 기술이 직면한 과제와 미래 전망을 제시함으로써, 기술 전문가와 전략 결정권자에게 통찰력 있는 시각을 제공하는 것을 목표로 합니다.


FPGA가 어떻게 AI와 같은 연산 집약적 워크로드에 강력한 솔루션이 될 수 있는지를 이해하기 위해서는 먼저 그 독특한 아키텍처를 근본적으로 이해해야 합니다. FPGA는 고정된 구조를 가진 CPU나 GPU와 달리, 사용자가 현장에서(Field) 회로를 프로그래밍할 수 있는(Programmable) 논리 게이트의 배열(Gate Array)로 구성된 반도체 소자입니다. 이러한 재구성 가능성은 FPGA의 가장 큰 특징이자, AI 가속화의 핵심 동력이 됩니다.


FPGA의 내부는 수많은 기본 구성 요소들이 상호 연결된 복잡한 구조를 가집니다. AI 가속기 설계 관점에서 중요한 핵심 요소들은 다음과 같습니다.1

- **구성 가능 논리 블록 (CLB/ALM):** FPGA의 가장 기본적인 빌딩 블록으로, 구성 가능 논리 블록(Configurable Logic Block, CLB) 또는 적응형 논리 모듈(Adaptive Logic Module, ALM)이라 불립니다. 이 블록 내부에는 조회 테이블(Look-Up Table, LUT)과 플립플롭(Flip-Flop)이 포함되어 있어, 사용자가 원하는 조합 및 순차 논리 회로를 구현하는 기반이 됩니다.3 LUT는 진리표 형태로 논리 함수를 저장하며, 이를 통해 임의의 디지털 로직을 생성할 수 있습니다.
- **프로그래밍 가능 인터커넥트 (Programmable Interconnects):** 칩 전체에 거미줄처럼 퍼져 있는 배선 자원과 스위치 박스의 네트워크입니다. 이 인터커넥트를 프로그래밍하여 수많은 논리 블록들을 서로 연결함으로써, 사용자가 설계한 특정 데이터 경로와 맞춤형 회로를 물리적으로 구성하게 됩니다.2
- **디지털 신호 처리 슬라이스 (DSP Slices):** 곱셈-누산(Multiply-Accumulate, MAC)과 같은 고성능 산술 연산을 위해 특별히 설계된 하드웨어 블록입니다.2 AI 신경망 연산의 대부분이 MAC 연산으로 이루어지기 때문에, 고효율의 전용 DSP 슬라이스는 FPGA를 AI 가속기로 활용하는 데 있어 가장 중요한 자원 중 하나입니다.4
- **블록 RAM (BRAM):** 칩 내부에 분산되어 있는 소규모 메모리 블록입니다. 데이터 저장, 버퍼링, 캐시 구현 등에 사용되며, 칩 외부의 느린 메모리(DRAM) 접근을 최소화하여 데이터 흐름을 최적화하고 병목 현상을 완화하는 데 결정적인 역할을 합니다.3
- **입출력 블록 (I/O Blocks):** FPGA를 외부 세계와 연결하는 구성 가능한 인터페이스입니다. PCIe, 이더넷, DDR 메모리 컨트롤러 등 매우 다양한 표준을 지원하도록 프로그래밍할 수 있어, 각종 센서, 네트워크, 호스트 시스템과의 유연한 연결을 가능하게 합니다.2

FPGA의 동작 원리는 하드웨어 기술 언어(Hardware Description Language, HDL)인 Verilog나 VHDL로 기술된 회로 설계를 "비트스트림(bitstream)"이라는 설정 파일로 변환(컴파일)하여 FPGA에 로드하는 것입니다.1 이 비트스트림은 각 LUT의 기능, 인터커넥트의 연결 경로, DSP와 BRAM의 동작 방식 등을 정의합니다. 일단 프로그래밍이 완료되면, FPGA는 특정 애플리케이션에 완벽하게 맞춰진 전용 하드웨어 가속기처럼 동작하게 됩니다.1


FPGA의 독특한 아키텍처는 AI 가속에 이상적인 세 가지 핵심적인 장점을 제공합니다.

- **내재된 병렬성 (Inherent Parallelism):** 명령어를 순차적으로 실행하는 CPU와 달리, FPGA의 아키텍처는 근본적으로 병렬적입니다. 프로그램이 명령어의 나열이 아닌 물리적인 회로 그 자체이기 때문에, 수천 개의 연산을 동시에 수행할 수 있습니다.1 이는 행렬 곱셈이나 컨볼루션과 같이 대규모 병렬 연산이 지배적인 AI 알고리즘의 특성과 완벽하게 부합합니다.1
- **깊은 파이프라이닝 (Deep Pipelining):** FPGA는 복잡한 연산을 매우 잘게 쪼개어 여러 단계의 파이프라인으로 구성할 수 있습니다. 데이터가 이 파이프라인을 통해 물 흐르듯 연속적으로 처리되므로, 실시간 AI 추론과 같은 스트리밍 데이터 애플리케이션에서 처리량을 극대화할 수 있습니다.5 각 파이프라인 단계가 독립적으로 동작하기 때문에 전체 시스템의 클록 주파수가 낮더라도 높은 처리 성능을 달성할 수 있습니다.
- **낮고 결정적인 지연 시간 (Low and Deterministic Latency):** FPGA는 운영체제(OS)나 복잡한 스케줄링 오버헤드 없이 로직을 하드웨어에서 직접 구현하기 때문에 매우 낮은 지연 시간을 달성할 수 있습니다.3 더 중요한 것은 이 지연 시간이 예측 가능하다는 점, 즉 '결정적(deterministic)'이라는 것입니다.9 자율주행, 산업용 로봇, 고빈도 금융 거래와 같이 응답 시간이 조금이라도 어긋나면 치명적인 결과를 초래할 수 있는 실시간 시스템에서 이는 타협할 수 없는 요구사항입니다.6

이러한 결정적 저지연 특성은 단순한 성능 지표를 넘어섭니다. 이는 CPU나 GPU의 운영체제 스케줄러, 인터럽트, 복잡한 메모리 계층 구조에서 발생하는 예측 불가능한 지연 시간 변동(jitter) 때문에 구현이 불가능하거나 신뢰성이 떨어지는 특정 종류의 실시간 제어 시스템을 가능하게 하는 근본적인 동력입니다.7 예를 들어, 자율주행 차량의 제어 루프나 로봇 팔 제어는 마이크로초 단위의 보장된 응답 시간을 요구하며, 여기서의 지연은 시스템의 안전과 직결됩니다.12 따라서 많은 경우 FPGA를 선택하는 이유는 단순히 처리량이 높아서가 아니라, 시스템이 안전하고 정확하게 작동하는 데 필요한 신뢰성과 예측 가능성을 제공하기 때문입니다. 이는 FPGA를 처리량에 최적화된 GPU와는 다른 시장 영역에 위치시킵니다.


강력한 장점에도 불구하고 FPGA는 몇 가지 내재된 한계를 가지고 있으며, 이는 설계 시 반드시 고려되어야 합니다.

- **설계 복잡성:** 전통적으로 FPGA를 프로그래밍하는 것은 HDL과 디지털 회로 설계에 대한 깊은 전문 지식을 요구합니다.1 이는 CPU나 GPU를 위한 소프트웨어 개발에 비해 학습 곡선이 매우 가파르며, "프로그래밍 가능성의 격차(programmability gap)"는 FPGA 도입의 가장 큰 장벽 중 하나로 작용해왔습니다.14
- **비용 및 전력:** FPGA는 대량 생산 시 주문형 반도체(ASIC)에 비해 개당 단가가 높으며, 프로그래밍 가능한 패브릭의 오버헤드로 인해 더 많은 전력을 소모할 수 있습니다.1 하지만 특정 워크로드에 맞춰 회로를 최적화할 수 있기 때문에, GPU와 비교했을 때 와트당 성능(performance-per-watt)은 오히려 더 우수할 수 있습니다.3
- **클록 속도:** FPGA는 일반적으로 최신 CPU나 GPU의 기가헤르츠(GHz) 단위 클록 속도에 비해 훨씬 낮은 수백 메가헤르츠(MHz) 단위로 동작합니다.3 FPGA의 성능 우위는 빠른 클록 속도가 아닌, 공간적으로 펼쳐진 막대한 병렬성에서 비롯됩니다.

FPGA 내부의 CLB, DSP, BRAM과 같은 자원들은 단순한 기능 목록이 아니라, 설계자가 사용할 수 있는 한정된 '예산'으로 간주해야 합니다. FPGA 기반 AI 가속기 설계는 이 예산을 최적으로 할당하는 과정과 같습니다. AI 워크로드는 MAC 연산이 지배적인데 18, 이 연산은 범용 로직인 CLB로 구현할 수도 있고, 고성능 전용 블록인 DSP 슬라이스로 구현할 수도 있습니다.2 면적, 전력, 성능 측면에서 DSP를 사용하는 것이 압도적으로 효율적이기 때문에 2, FPGA AI 설계의 핵심 제약 조건 중 하나는 사용 가능한 DSP 슬라이스의 개수가 됩니다. 이 '예산'이 소진되면, 설계자는 덜 효율적인 범용 로직을 사용해야 하므로 성능과 효율이 급격히 저하됩니다. 이는 FPGA의 하드웨어 예산에 맞추기 위해 AI 모델 자체를 수정해야 하는 하드웨어-소프트웨어 공동 설계(co-design) 접근법의 필요성을 시사합니다.


AI 추론을 위한 하드웨어 플랫폼은 크게 FPGA, GPU(Graphics Processing Unit), ASIC(Application-Specific Integrated Circuit)의 세 가지로 나눌 수 있습니다. 각 기술은 고유한 장단점을 가지며, 특정 애플리케이션 요구사항에 따라 최적의 선택이 달라집니다. 이 섹션에서는 세 가지 기술을 성능, 전력 효율, 비용, 유연성 등 다각적인 측면에서 심층 비교 분석합니다.


- **GPU (높은 처리량):** 본래 그래픽 렌더링을 위해 설계된 GPU는 수천 개의 코어를 이용한 대규모 데이터 병렬 처리에 특화되어 있습니다. 따라서 대규모 데이터 배치를 한 번에 처리하는 AI 모델 학습이나 대용량 배치 추론에서 가장 높은 원시 처리량(TFLOPs)을 제공합니다.8 하지만, 실시간 스트리밍 데이터처럼 배치 크기가 작은 경우(예: batch-1)나 데이터가 희소(sparse)할 경우, 많은 코어가 유휴 상태에 빠져 효율이 급격히 떨어지는 단점이 있습니다.20
- **FPGA (낮은 지연 시간 및 높은 효율):** FPGA는 데이터가 도착하는 즉시 처리할 수 있는 맞춤형 파이프라인을 구축할 수 있어, 배치 크기가 1인 실시간 스트리밍 데이터 추론에서 빛을 발합니다. 이는 매우 낮은 지연 시간을 가능하게 합니다.6 최고 TFLOPs 수치는 하이엔드 GPU보다 낮을 수 있지만, 지연 시간에 민감한 실제 워크로드에서의 '유효 처리량'은 GPU를 능가할 수 있습니다.21
- **ASIC (최고 성능):** ASIC은 특정 애플리케이션(예: 구글의 TensorFlow)만을 위해 처음부터 설계된 맞춤형 칩입니다. 불필요한 모든 로직이 제거되었기 때문에, 해당 특정 작업에 대해서는 타의 추종을 불허하는 최고의 성능과 효율을 보여줍니다.6


- **전력 효율 (와트당 성능):** FPGA는 동일한 추론 워크로드에 대해 GPU보다 훨씬 전력 효율적이며, 연구에 따르면 종종 3배에서 10배까지 적은 전력을 소모하는 것으로 나타납니다.8 이는 특정 작업에 필요한 로직만 활성화하는 맞춤형 회로 덕분이며, GPU는 범용 아키텍처로 인한 오버헤드가 크기 때문입니다.3 물론 모든 가속기 중에서는 ASIC이 가장 전력 효율이 높습니다.15 이러한 특성으로 인해 FPGA는 전력 제약이 심한 엣지 컴퓨팅 환경이나, 데이터 센터의 운영 비용과 탄소 발자국을 줄이는 데 이상적인 솔루션이 됩니다.8
- **비용 동역학:**
  - **초기 개발 비용 (NRE):** ASIC은 설계 및 제작에 수백만 달러에 달하는 막대한 비반복적 엔지니어링(Non-Recurring Engineering, NRE) 비용과 긴 개발 기간을 필요로 합니다. 따라서 매우 안정적이고 대량 생산이 보장된 애플리케이션에만 적합합니다.15 반면 FPGA는 NRE 비용이 없습니다.
  - **개당 단가:** 대량 생산 시, ASIC의 개당 단가가 가장 저렴합니다. FPGA는 ASIC보다 단가가 높지만, 하이엔드 GPU보다는 비용 효율적일 수 있습니다.1
  - **총소유비용 (TCO):** 알고리즘이 빠르게 변화하는 분야에서는 FPGA의 재구성 가능성이 큰 장점이 됩니다. 5~7년의 제품 수명 주기 동안, 더 잦은 하드웨어 교체가 필요한 GPU 기반 시스템에 비해 FPGA가 더 낮은 총소유비용을 가질 수 있습니다.22


- **유연성과 재구성 가능성:** 이는 FPGA를 정의하는 가장 핵심적인 특징입니다. FPGA는 현장에서 재프로그래밍하여 새로운 AI 모델, 알고리즘, 또는 산업 표준에 대응할 수 있으며, 이는 '미래 보장(future-proofing)'을 제공합니다.1 이는 제작 시 기능이 영구적으로 고정되는 ASIC과 극명한 대조를 이룹니다.6 GPU는 소프트웨어 수준의 유연성을 제공하지만, 하드웨어 아키텍처 자체는 고정되어 있습니다.
- **트레이드오프:** FPGA의 프로그래밍 가능성은 전용으로 제작된 ASIC에 비해 낮은 집적도와 효율성이라는 대가를 치릅니다. 시스템 설계자가 직면하는 핵심 딜레마는 이 스펙트럼 상에서 어떤 지점을 선택할 것인가입니다: 최대의 유연성(FPGA), 최대의 처리량(GPU), 또는 고정된 작업에 대한 최대의 효율성(ASIC).15

가속기 선택은 정적인 결정이 아니라 AI 알고리즘 자체의 진화와 동적으로 상호작용하는 과정입니다. AI 연구는 매우 빠르게 진행되어 새로운 모델 아키텍처가 끊임없이 등장합니다.17 긴 설계 주기와 고정된 기능을 가진 ASIC은 시장에 출시될 때쯤이면 지배적인 알고리즘의 변화로 인해 이미 구식이 될 위험이 있습니다.15 GPU는 소프트웨어를 통해 적응할 수 있지만, 그 하드웨어는 본질적으로 밀집 행렬 연산에 최적화되어 있습니다.8 만약 미래의 모델이 극단적인 희소성이나 그래프 기반 연산과 같은 다른 종류의 계산에 크게 의존하게 된다면 GPU의 효율성은 떨어질 수 있습니다. 반면, FPGA는 이러한 근본적인 알고리즘 변화에 하드웨어 수준에서 적응할 수 있는 독보적인 위치에 있습니다.1 마이크로소프트가 Bing 검색 엔진에 FPGA를 도입한 것은 바로 이러한 알고리즘 불확실성에 대한 '보험'으로서의 가치를 입증한 사례입니다.3

이러한 맥락에서 FPGA, GPU, ASIC의 고전적인 정의는 점차 흐려지고 있습니다. 각 벤더들이 서로의 장점을 흡수하며 경계가 모호해지는 융합 현상이 나타나고 있습니다. 예를 들어, 최신 AI-FPGA는 더 이상 순수한 '게이트의 바다'가 아닙니다. AMD의 Versal이나 인텔의 Agilex 시리즈는 AI 엔진이나 텐서 블록과 같은 ASIC과 유사한 전용 하드웨어 블록을 내장하고 있습니다.4 AMD는 Versal을 '적응형 컴퓨팅 가속 플랫폼(ACAP)'이라 칭하며 전통적인 FPGA의 개념을 넘어서고 있습니다.4 마찬가지로, GPU는 AI 워크로드 가속을 위해 '텐서 코어'라는 ASIC과 유사한 행렬 곱셈 전용 유닛을 통합했습니다.27 이러한 융합은 유연성과 효율성 사이에서 더 나은 균형점을 제공하려는 시장의 요구에 의해 주도되고 있으며, 미래는 어느 한 기술이 독식하는 것이 아니라 프로그래밍 가능한 로직, 전용 엔진, 범용 코어가 동일한 실리콘 위에서 공존하는 이기종 플랫폼(heterogeneous platform)의 시대가 될 것임을 시사합니다.

다음 표는 AI 추론을 위한 세 가지 가속기 유형의 핵심적인 트레이드오프를 요약하여 보여줍니다.

| 지표                        | FPGA                               | GPU                                    | ASIC                       |
| --------------------------- | ---------------------------------- | -------------------------------------- | -------------------------- |
| **주요 사용 사례**          | 저지연 추론, 실시간 스트리밍       | 고처리량 학습, 배치 추론               | 대용량, 고정 기능 배포     |
| **성능 (처리량)**           | 모델에 따라 중-고                  | 매우 높음                              | 특정 작업에 대해 가장 높음 |
| **성능 (지연 시간)**        | 매우 낮음, 결정적                  | 높고, 가변적                           | 매우 낮음, 결정적          |
| **전력 효율 (와트당 성능)** | 높음 ~ 매우 높음                   | 보통                                   | 가장 높음                  |
| **유연성 (재구성 가능성)**  | 매우 높음 (하드웨어 및 소프트웨어) | 낮음 (소프트웨어만)                    | 없음 (고정된 하드웨어)     |
| **개발 복잡성/시간**        | 높음 (HDL/HLS 필요)                | 낮음 (표준 소프트웨어 프레임워크 사용) | 매우 높음 (전체 칩 설계)   |
| **초기 비용 (NRE)**         | 없음                               | 없음                                   | 매우 높음                  |
| **개당 단가 (대량)**        | 보통                               | 높음                                   | 낮음                       |
| **데이터 정밀도**           | 완전 맞춤형 가능                   | 고정 (예: FP32/16, INT8)               | 설계 시 고정               |

표 2.1: AI 추론을 위한 FPGA, GPU, ASIC의 다각적 비교. 데이터 소스:.1


FPGA가 AI 추론에 '왜' 적합한지를 이해했다면, 이제 '어떻게' 그것이 가능한지를 기술적으로 파고들 차례입니다. 이 파트에서는 AI 알고리즘이 FPGA라는 실리콘 위에서 고성능 가속기로 구현되는 구체적인 과정과 핵심 기술들을 상세히 다룹니다. 맞춤형 하드웨어 엔진 설계부터 모델을 배포하는 소프트웨어 워크플로우까지, 전 과정을 심층적으로 분석합니다.


이 섹션에서는 FPGA 상에 효율적인 AI 추론 엔진을 구축하는 데 사용되는 구체적인 기술적 방법론을 탐구합니다. 이는 단순히 알고리즘을 하드웨어로 변환하는 것을 넘어, FPGA의 아키텍처적 특성을 최대한 활용하여 성능을 극대화하는 과정입니다.


현대의 복잡한 AI 모델 전체를 하나의 거대한 하드웨어 코어로 구현하는 것은 비효율적입니다. 대신, 작고 재사용 가능하며 고도로 최적화된 'CNN 코어' 또는 'AI 코어'를 FPGA 패브릭 내에 설계하는 것이 일반적인 접근 방식입니다.18

- **아키텍처:** 일반적인 CNN 코어는 3x3과 같은 작은 고정 크기의 컨볼루션 연산을 수행하도록 설계됩니다. 이 코어는 입력 특징 맵(feature map)의 픽셀과 가중치(weight)를 곱하는 병렬 MAC 유닛 배열(주로 DSP 슬라이스에 구현됨), 그 결과를 합산하는 덧셈기 트리(adder tree), 그리고 편향(bias)을 더하고 ReLU와 같은 활성화 함수를 적용하는 로직으로 구성됩니다.18
- **확장성:** 5x5 또는 그 이상의 더 큰 특징 맵을 처리하기 위해, 소프트웨어 드라이버는 큰 문제를 여러 개의 3x3 타일로 분할하여 동일한 하드웨어 코어에 순차적으로 공급합니다. 이처럼 하드웨어를 재사용하는 방식은 한정된 FPGA 자원을 효율적으로 활용하는 핵심 전략입니다.18 FPGA의 자원 예산 내에서 허용하는 만큼 여러 개의 코어를 병렬로 인스턴스화하여 전체 처리량을 높일 수 있습니다.


FPGA 기반 AI 가속기의 성능은 연산 유닛의 속도보다 데이터 흐름 아키텍처의 효율성에 의해 더 크게 좌우됩니다. 칩 외부의 느린 DRAM과 FPGA 내부의 빠른 연산 유닛 간에 데이터를 이동시키는 것은 성능과 전력 소비의 주요 병목 지점입니다.20 고성능을 달성하기 위한 핵심은 데이터 재사용을 극대화하고 가능한 한 오랫동안 데이터를 칩 내부에 유지하는 것입니다.

- **온칩 메모리 계층:** FPGA는 내장된 블록 RAM(BRAM)을 활용하여 가중치, 편향, 중간 특징 맵 등을 저장하기 위한 맞춤형 메모리 계층, 즉 캐시와 버퍼를 생성합니다.24 이를 통해 연산에 필요한 데이터에 빠르게 접근하고 외부 메모리 접근 횟수를 획기적으로 줄일 수 있습니다.
- **데이터 무버 (Data Movers):** '데이터 무버'라고 불리는 전용 상태 머신(state machine)은 외부 메모리(예: PCIe를 통한 DDR)에서 온칩 BRAM으로 데이터를 가져오고, 이를 다시 정해진 순서에 따라 AI 코어에 공급하는 복잡한 데이터 흐름을 관리합니다.5 이 데이터 무버의 설계는 종종 AI 코어 자체만큼이나 복잡하고 중요하며, 전체 가속기 성능에 결정적인 영향을 미칩니다. 연산 코어가 데이터가 없어 멈추거나(starved), 결과를 쓸 곳이 없어 지연되는(stalled) 상황을 방지하는 것이 데이터 무버의 핵심 역할입니다. 성공적인 FPGA AI 가속은 본질적으로 연산 로직 설계의 문제가 아니라, 메모리 아키텍처와 데이터 흐름 엔지니어링의 문제라고 할 수 있습니다.


AI 모델은 일반적으로 32비트 부동소수점(FP32) 숫자를 사용하여 학습됩니다. 하지만 추론 단계에서는 정확도의 큰 손실 없이 훨씬 낮은 정밀도의 숫자를 사용해도 충분한 경우가 많습니다.6

- **양자화 (Quantization):** 이 과정은 학습된 모델의 FP32 가중치와 활성화 값을 8비트 정수(INT8)와 같은 저정밀도 형식으로 변환하는 것을 의미합니다.6
- **장점:**
  - **메모리 절감:** INT8 데이터는 FP32에 비해 4배 적은 메모리와 대역폭을 필요로 하므로, 메모리 병목 현상을 크게 완화합니다.29
  - **성능 향상:** 정수 연산은 부동소수점 연산보다 하드웨어로 구현하기에 훨씬 빠르고 면적 효율적입니다. 이를 통해 동일한 FPGA 면적에 더 많은 병렬 연산 유닛을 집적할 수 있습니다.31
  - **저전력:** 정수 연산은 훨씬 적은 전력을 소모합니다.
- **FPGA의 이점:** FPGA는 데이터 정밀도에 대한 세밀한 제어를 제공합니다. FP16/INT8 등으로 정밀도가 고정된 GPU와 달리, FPGA는 4비트, 2비트 등 임의의 맞춤형 비트 폭(custom bit-width)을 구현할 수 있어 특정 모델에 대한 최적화 가능성을 더욱 높여줍니다.7

양자화와 같은 기술은 단순한 '최적화' 기법을 넘어, 복잡하고 거대한 최신 AI 모델을 자원이 제한된 FPGA와 같은 하드웨어에 탑재 가능하게 만드는 '하드웨어 구현 가능 기술(Hardware-Enabling Technology)'입니다. 최신 트랜스포머 모델은 매우 거대하지만 26, FPGA의 온칩 메모리(BRAM)는 수 메가바이트 수준으로 매우 제한적입니다.17 양자화를 통해 메모리 요구량을 4배 이상 줄이지 않으면, 모델의 극히 일부만 온칩에 저장할 수 있어 끊임없이 느린 외부 DRAM에 접근해야만 할 것입니다.29 따라서 양자화는 모델을 '더 빠르게' 만드는 것을 넘어, 애초에 배포가 가능하도록 만드는 근본적인 역할을 합니다.


학습이 완료된 많은 신경망은 가중치 값의 상당 부분이 0인 '희소(sparse)'한 특성을 보입니다.34 0과의 곱셈은 불필요한 연산이므로, 이를 건너뛸 수 있다면 상당한 성능 향상을 기대할 수 있습니다.

- **구조적 희소성 vs. 비구조적 희소성:**
  - **비구조적 희소성 (Unstructured Sparsity):** 0이 텐서 내에 무작위로 분포하는 경우입니다. 이를 가속화하려면 0을 건너뛰기 위한 복잡한 하드웨어가 필요하며, 이는 높은 면적 및 전력 오버헤드를 유발할 수 있습니다.34
  - **구조적 희소성 (Structured Sparsity):** 희소성이 일정한 패턴(예: 4개의 값 중 2개는 0이 아님)을 갖도록 강제하는 경우입니다. 이는 하드웨어에서 가속하기 훨씬 용이합니다. NVIDIA의 A100 GPU는 2:4 구조적 희소성을 지원합니다.34
- **FPGA 구현:** 최근 연구는 1:4 또는 2:4와 같은 패턴에 기반하여 동적으로 연산을 건너뛸 수 있는 'SST 슬라이스'와 같이 구조적 희소성을 위한 전용 지원을 갖춘 FPGA 아키텍처를 만드는 데 초점을 맞추고 있습니다. 이는 낮은 면적 오버헤드로 상당한 속도 향상을 약속합니다.34


FPGA의 잠재력을 최대한 활용하기 위해서는 하드웨어 전문가가 아닌 더 넓은 개발자 커뮤니티가 쉽게 접근할 수 있어야 합니다. 이 섹션에서는 FPGA AI 개발의 복잡성을 낮추고, 소프트웨어 중심의 개발 경험을 제공하기 위한 도구와 워크플로우를 자세히 살펴봅니다.


- **개념:** 고수준 합성(High-Level Synthesis, HLS) 도구는 개발자가 C, C++, Python과 같은 고수준 언어로 알고리즘을 작성하면 이를 FPGA용 HDL 코드로 자동 변환(컴파일)해주는 기술입니다.3
- **영향:** HLS는 Verilog/VHDL 전문 지식 없이도 소프트웨어 엔지니어나 데이터 과학자가 FPGA를 활용할 수 있게 함으로써 진입 장벽을 획기적으로 낮춥니다. 이는 전통적인 RTL 설계 방식에 비해 개발 시간을 극적으로 단축시킵니다.31
- **한계:** 강력한 도구임에도 불구하고, HLS는 순수한 소프트웨어 프로그래밍과는 다른 사고방식을 요구합니다. 개발자는 효율적인 HLS 코드를 작성하기 위해 여전히 병렬성, 파이프라이닝과 같은 하드웨어 개념을 이해해야 합니다. 또한, 자동 생성된 하드웨어는 숙련된 엔지니어가 직접 코딩한 RTL 설계보다 최적화 수준이 떨어지는 경우가 많습니다.

이러한 추상화는 '새는 파이프(leaky pipe)'와 같습니다. HLS나 AI 툴체인이 높은 수준의 추상화를 제공하지만, 하드웨어에 대한 인식을 완전히 제거하지는 못합니다. HLS 도구는 C++ 코드를 하드웨어로 변환하지만, 결과물의 성능은 C++ 코드가 어떻게 구조화되었는지(예: 파이프라이닝과 병렬성을 지정하는 프라그마 사용)에 크게 의존합니다.35 순진하게 작성된 C++ 코드는 느리고 비효율적인 하드웨어를 생성할 뿐입니다. 마찬가지로, AI 컴파일러는 신경망을 FPGA의 자원(DSP, BRAM 등)에 매핑하는 결정을 내리는데 36, 만약 모델의 아키텍처가 기본 하드웨어와 잘 맞지 않으면(예: 지원되지 않는 레이어 유형 사용) 성능이 저하됩니다. 따라서 최적의 성능을 달성하기 위해서는 여전히 개발자가 대상 FPGA 아키텍처의 능력과 한계를 이해하고, 효과적으로 가속될 수 있는 모델을 설계하거나 선택하는 하드웨어-소프트웨어 공동 설계가 필요합니다.


FPGA 벤더들은 AI 모델을 FPGA에 배포하는 과정을 버튼 하나로 해결하는 소프트웨어와 같은 경험을 제공하는 것을 목표로 합니다. 일반적인 흐름은 '모델 학습 -> 양자화 -> 컴파일 -> 배포'의 단계를 따릅니다.

- **인텔의 생태계 (OpenVINO):**
  - **워크플로우:** 사용자는 TensorFlow, PyTorch 등에서 사전 학습된 모델을 가져와 OpenVINO 모델 옵티마이저를 사용하여 중간 표현(Intermediate Representation, IR)으로 변환합니다.36
  - 그 다음, FPGA AI Suite 컴파일러가 이 IR을 특정 FPGA 타겟용 바이너리 실행 파일로 컴파일하며, 이 과정에서 AI 텐서 블록과 같은 자원에 맞게 최적화합니다.26
  - 최종적으로 애플리케이션에서는 런타임 API를 사용하여 이 바이너리를 로드하고 추론을 실행합니다.36
- **AMD의 생태계 (Vitis AI):**
  - **워크플로우:** Vitis AI는 유사한 엔드-투-엔드 흐름을 제공합니다. FP32 모델을 INT8로 변환하는 양자화 도구와, FPGA 상의 AI 엔진 오버레이인 DPU(Deep Learning Processor Unit)에서 실행 가능한 `.xmodel` 파일을 생성하는 컴파일러를 포함합니다.33
  - Vitis AI 런타임(VART)은 호스트 애플리케이션(임베디드 ARM 코어나 외부 CPU에서 실행)이 DPU를 제어하고 데이터 전송을 관리하기 위한 API를 제공합니다.33

이러한 생태계의 발전은 AI 가속화 시장에서 FPGA 벤더들이 더 이상 단순한 실리콘 칩을 파는 것이 아니라, 완전한 하드웨어+소프트웨어 통합 솔루션을 판매하고 있음을 의미합니다. 소프트웨어 툴체인(Vitis AI, OpenVINO)의 품질과 사용 편의성은 칩 자체의 원시 성능만큼이나, 혹은 그 이상으로 중요해졌습니다. AI 가속화의 주 사용자는 데이터 과학자와 소프트웨어 엔지니어이며 14, 이들에게 하드웨어의 복잡성을 추상화해주는 소프트웨어 중심의 개발 경로는 필수적입니다. 따라서 AMD와 인텔 간의 AI 시장 경쟁은 실리콘 아키텍처뿐만 아니라, 소프트웨어 개발 경험, 문서화, 커뮤니티 지원과 같은 생태계의 성숙도를 두고 치열하게 전개되고 있습니다.20


- **표준 프레임워크:** 인텔과 AMD의 툴체인은 모두 TensorFlow, PyTorch와 같은 대중적인 AI 프레임워크와 원활하게 통합되도록 설계되었습니다. 이는 데이터 과학자들이 모델 학습을 위해 이미 익숙한 도구를 그대로 사용할 수 있게 해주므로 매우 중요합니다.14
- **컨테이너화 (Docker):** FPGA 개발 도구의 복잡한 의존성들은 종종 Docker 컨테이너로 패키징되어 제공됩니다.33 이는 설치 과정을 단순화하고 일관성 있고 재현 가능한 개발 환경을 보장하여, 개발자의 생산성을 크게 향상시킵니다.


이 파트에서는 FPGA 시장을 주도하는 핵심 기업들을 살펴보고, 이 기술이 거대한 데이터 센터부터 특수한 임베디드 시스템에 이르기까지 실제 환경에서 어떻게 적용되고 있는지 구체적인 사례를 통해 탐구합니다.


FPGA 시장은 인텔(구 알테라)과 AMD(구 자일링스)라는 두 거대 기업이 지배하고 있습니다. 이들 기업은 AI 시대를 맞아 각각 독자적인 전략과 주력 제품군을 통해 치열한 경쟁을 벌이고 있습니다.


- **제품 라인업:** 인텔의 주력 제품군은 Agilex 패밀리이며, 저전력에 최적화된 Agilex 3부터 고성능 Agilex 9까지 다양한 계층으로 구성됩니다.39 이 중 Agilex 5와 7 시리즈가 특히 AI에 중점을 두고 있습니다.40
- **아키텍처 철학:** 인텔의 접근 방식은 AI 기능을 프로그래밍 가능한 패브릭에 직접 '주입(infuse)'하는 것입니다. Agilex 5는 AI 텐서 블록(AI Tensor Block)을 도입했는데, 이는 DSP와 유사하지만 INT8, bfloat16과 같은 AI 친화적인 정밀도에 최적화된 강화된(hardened) 연산 블록으로, 패브릭 전체에 분산되어 있습니다.26 이에 앞서 출시된 Stratix 10 NX는 최대 143 INT8 TOPS 성능을 제공하는 AI 텐서 블록을 패브릭 내에 통합하여 이러한 접근법의 선구자 역할을 했습니다.27
- **주요 특징:** Agilex 디바이스는 FPGA 패브릭과 Arm 코어를 탑재한 하드 프로세서 시스템(HPS)을 결합하고, PCIe 5.0, CXL, 고속 이더넷과 같은 최신 인터페이스를 지원합니다.26 목표는 엣지 및 데이터 센터 AI 워크로드를 위한 고도로 통합된 솔루션을 제공하는 것입니다.


- **제품 라인업:** AMD의 Versal 제품군은 전통적인 FPGA에서 '적응형 컴퓨팅 가속 플랫폼(ACAP)'으로의 전환을 상징합니다.4 AI를 위한 핵심 시리즈로는 Versal AI Core, AI Edge, HBM 시리즈가 있습니다.25
- **아키텍처 철학:** Versal의 전략은 '이기종 통합(heterogeneous integration)'에 기반합니다. 하나의 디바이스에 세 가지 유형의 연산 엔진을 통합합니다:
  1. **스칼라 엔진 (Scalar Engines):** 제어 및 복잡한 의사 결정을 위한 범용 Arm 프로세서.
  2. **적응형 엔진 (Adaptable Engines):** 맞춤형 하드웨어 가속을 위한 전통적인 FPGA 패브릭 (CLB, DSP).
  3. **지능형 엔진 (Intelligent Engines):** 'AI 엔진'이라 불리는 VLIW/SIMD 벡터 프로세서 배열로, AI/ML 및 DSP 워크로드에 고도로 최적화되어 있습니다. 이 엔진들은 칩 상에서 뚜렷하게 구분되는 강화된 블록을 형성하며, 네트워크 온 칩(Network-on-Chip, NoC)을 통해 패브릭과 연결됩니다.4
- **주요 특징:** AI 엔진은 본질적으로 타일 형태로 배열된 특수 프로세서로, 프로그래밍 가능한 로직이 데이터 전처리, 후처리 및 맞춤형 로직을 처리하는 동안 막강한 AI 연산 능력을 제공하는 가속기 블록 역할을 합니다.20

이러한 전략적 움직임은 CPU 거대 기업인 인텔과 AMD가 FPGA 회사를 인수한 것이 단순히 FPGA를 더 많이 판매하기 위함이 아니라, 미래 컴퓨팅의 핵심이 될 포괄적인 이기종 컴퓨팅 플랫폼을 구축하기 위한 전략적 결정이었음을 보여줍니다. 미래의 고성능 컴퓨팅은 CPU, GPU, FPGA/가속기 등 다양한 유형의 프로세서를 결합하여 복잡한 워크로드를 처리하는 방향으로 나아가고 있습니다.6 두 회사는 자사의 CPU 중심 로드맵만으로는 AI와 같은 특수 워크로드의 폭발적인 성장에 대응하기에 불충분하다는 것을 인식했습니다. FPGA 기술을 확보함으로써, 복잡한 시스템의 여러 부분을 유연하게 연결하고 가속할 수 있는 '접착제'와 같은 재구성 가능 하드웨어 기술을 손에 넣은 것입니다.38 Versal과 Agilex 같은 신제품들은 단순히 CPU가 부착된 FPGA가 아니라, 처음부터 이기종 컴퓨팅을 위해 깊이 통합된 플랫폼입니다. 이는 장기적으로 프로그래밍 가능한 로직이 CPU와 더욱 긴밀하게 통합되어, 미래의 서버 및 클라이언트 프로세서의 표준 기능으로 맞춤형 명령어와 워크로드별 가속을 제공하는 방향으로 나아갈 것임을 시사합니다.38


- **인텔의 '주입' 방식:** AI 기능을 패브릭 전체에 분산시킴으로써, 인텔은 세밀한 유연성을 제공합니다. 이는 맞춤형 로직과 AI 연산 간의 긴밀하고 낮은 지연 시간의 결합이 필요한 워크로드에 유리할 수 있습니다.
- **AMD의 '통합' 방식:** 강력한 전용 AI 엔진 배열을 생성함으로써, AMD는 전력 효율적인 ASIC과 유사한 블록에서 막대한 양의 원시 AI 연산 능력을 제공합니다. 여기서 프로그래밍 가능한 패브릭은 데이터 형태 변환 및 맞춤형 기능을 위한 매우 유연하고 강력한 '보조 프로세서' 역할을 합니다.
- **목표 시장:** 두 회사 모두 데이터 센터 가속, 5G/무선 통신, 자동차, 산업, 항공우주/방위 산업 등 동일한 광범위한 시장을 목표로 합니다.6 미묘한 아키텍처의 차이로 인해 특정 애플리케이션에서는 한 플랫폼이 다른 플랫폼보다 더 적합할 수 있습니다.

다음 표는 두 선도적인 AI-FPGA 플랫폼의 아키텍처 및 기능 수준을 직접 비교하여, 각기 다른 철학을 구체적으로 보여줍니다.

| 기능                          | AMD Versal AI 시리즈                                | 인텔 Agilex 5/7 시리즈                              |
| ----------------------------- | --------------------------------------------------- | --------------------------------------------------- |
| **전체 아키텍처**             | 이기종 ACAP: 스칼라 + 적응형 + 지능형 엔진          | AI 주입 패브릭을 갖춘 SoC FPGA                      |
| **주요 AI 연산 유닛**         | AI 엔진 배열: VLIW/SIMD 벡터 프로세서               | AI 텐서 블록: 강화된 행렬 연산 유닛                 |
| **AI 유닛 통합 방식**         | NoC로 연결된 별도의 대형 배열                       | 프로그래밍 가능 패브릭 전체에 분산                  |
| **프로그래밍 가능 로직 유닛** | 구성 가능 논리 블록 (CLB)                           | 적응형 논리 모듈 (ALM)                              |
| **인터커넥트**                | 계층적 라우팅 + 네트워크 온 칩 (NoC)                | 메시 기반 라우팅 (Hyperflex 아키텍처)               |
| **임베디드 프로세서**         | 듀얼 코어 Arm Cortex-A72 + 듀얼 코어 Arm Cortex-R5F | 듀얼 코어 Arm Cortex-A76 + 듀얼 코어 Arm Cortex-A55 |
| **개발 소프트웨어**           | Vitis 통합 소프트웨어 플랫폼 / Vitis AI             | Quartus Prime / OpenVINO를 포함한 FPGA AI Suite     |

표 5.1: 인텔 Agilex와 AMD Versal AI 시리즈의 아키텍처 및 기능 비교. 데이터 소스:.4


이 섹션에서는 FPGA가 다양한 산업 분야에서 어떻게 성공적으로 배포되고 있는지 구체적인 사례를 통해 살펴봅니다. 이러한 사례들은 FPGA의 이론적 장점이 실제 세계에서 어떻게 가치로 전환되는지를 명확히 보여줍니다.


- **마이크로소프트 Bing & 프로젝트 브레인웨이브:** 이는 FPGA 데이터 센터 적용의 기념비적인 사례입니다. 마이크로소프트는 Bing 검색 엔진의 순위 결정 알고리즘을 가속하기 위해 데이터 센터에 FPGA를 대규모로 배포했습니다. 그 결과 처리량이 50% 증가하는 성과를 거두었습니다.3 FPGA는 네트워크에 연결된 서비스 형태로 배포되어, 수많은 CPU가 추론 작업을 FPGA 풀에 오프로드할 수 있게 했습니다. 이를 통해 GPU로는 달성하기 어려운 매우 낮은 지연 시간(배치 크기 1)으로 높은 처리량을 달성했으며 21, 이는 FPGA가 대규모 실시간 웹 서비스 가속에 매우 효과적임을 입증했습니다.
- **스마트NIC 및 인프라 오프로드:** FPGA는 스마트 네트워크 인터페이스 카드(SmartNIC)에 사용되어 네트워킹, 스토리지, 보안과 같은 작업을 호스트 CPU로부터 오프로드합니다. AI 맥락에서, FPGA는 서버로 스트리밍되는 네트워크 데이터를 실시간으로 전처리, 필터링하고 심지어 추론까지 수행할 수 있습니다. 이는 지연 시간을 줄이고 CPU/GPU의 자원을 확보하는 데 기여합니다.24


- **산업 및 로보틱스:** FPGA는 다축 로봇 팔을 제어하고, 여러 센서의 데이터를 융합하며, 실시간 제어를 위한 머신러닝 알고리즘을 실행하는 데 사용됩니다.42 낮은 전력 소모와 결정적인 지연 시간은 공장 자동화 환경에서 매우 중요합니다.
- **스마트 카메라 및 비전 시스템:** 스마트 공장의 결함 감지나 지능형 영상 감시와 같은 애플리케이션에서 FPGA는 고해상도 비디오 스트림을 실시간으로 처리하고, 장치 자체에서 객체 탐지 및 분류를 위한 CNN을 실행할 수 있습니다.42 이는 대량의 비디오 데이터를 클라우드로 전송할 필요를 없애 대역폭을 절약하고 개인 정보 보호를 강화합니다.
- **IoT 및 소비자 기기:** 저전력 FPGA는 스마트 모니터의 사용자 감지, 음성 제어, 복잡한 IoT 시스템의 센서 퓨전과 같은 광범위한 엣지 디바이스에서 활용됩니다.46


- **센서 퓨전:** 자율주행 차량은 LiDAR, 레이더, 카메라 등 다양한 센서에 의존합니다. FPGA는 이러한 이기종 데이터를 실시간으로 융합하는 데 이상적입니다. 병렬 아키텍처와 다재다능한 I/O는 막대한 양의 이기종 데이터 스트림을 동시에 처리하여 차량 주변 환경에 대한 통일된 360도 시야를 생성할 수 있습니다.13
- **실시간 인식 및 제어:** FPGA는 인식 파이프라인을 가속하여, 안전 운행에 필수적인 초저지연 시간으로 객체 탐지, 차선 탐지, 교통 표지판 인식 등을 위한 AI 모델(예: CNN)을 실행합니다.13 또한 계획된 경로를 조향 및 제동을 위한 정밀한 명령으로 변환하는 제어 시스템에도 사용됩니다.49
- **안전 및 이중화:** FPGA의 신뢰성은 안전이 중요한 시스템에 사용될 수 있게 하며, 자동차 안전 표준인 ISO 26262를 준수하기 위해 이중화 로직과 페일-세이프(fail-safe) 메커니즘을 구현하는 데 적합합니다.48

이러한 애플리케이션에서 FPGA가 선택되는 주된 이유는 단순히 연산 능력 때문만이 아닙니다. 다양한 센서 및 네트워크 프로토콜과 고속으로 직접 인터페이스할 수 있는 능력, 즉 'I/O 이점'이 FPGA의 숨겨진 초능력입니다. GPU나 CPU는 LiDAR나 고속 의료용 탐지기와 같은 특수 센서에 연결하기 위해 호스트 시스템과 다양한 인터페이스 칩이 필요하며, 이 과정에서 지연 시간과 복잡성이 추가됩니다.48 반면, FPGA의 유연한 I/O 블록은 거의 모든 디지털 인터페이스를 직접 구현하도록 프로그래밍할 수 있습니다.2 이를 통해 FPGA는 센서로부터 데이터를 직접 '수신'하여 호스트 CPU나 PCIe 버스를 거치지 않고 저지연 파이프라인에서 처리한 후 결과를 출력할 수 있습니다.10 이러한 '인라인 처리(bump-in-the-wire)' 능력은 실시간 스트리밍 애플리케이션에서 근본적인 아키텍처 우위를 제공하며, 자율주행과 같이 센서 집약적인 분야에서 FPGA 채택이 증가하는 핵심 이유입니다.


- **의료 영상 가속 (MRI, CT, PET):** FPGA는 진단 장비 내부에서 영상 재구성 알고리즘을 가속하는 데 사용됩니다. 원시 센서 데이터를 실시간으로 처리하고 FFT와 같은 복잡한 필터와 변환을 적용하여 CPU보다 훨씬 빠르게 선명한 이미지를 생성함으로써 환자의 스캔 시간을 단축시킵니다.51
- **AI 기반 진단:** FPGA는 컴퓨터 보조 진단을 위한 AI 추론을 가속하는 데 사용됩니다. 사례 연구에 따르면 FPGA는 MRI 스캔에서 뇌종양을 탐지하거나 53 유방 조영술 이미지에서 유방암을 탐지하는 CNN을 높은 정확도와 효율성으로 실행합니다.54 낮은 지연 시간은 내시경 검사와 같은 시술 중에 실시간 피드백을 가능하게 하며, 여기서 FPGA는 AI가 생성한 하이라이트를 라이브 비디오 피드 위에 오버레이할 수 있습니다.55
- **유전체학 및 실험실 자동화:** FPGA는 개인 맞춤형 의료를 가능하게 하는 DNA 시퀀싱을 가속하는 데 사용됩니다. 이 과정에서는 방대한 양의 데이터를 염기 서열 분석을 위해 처리해야 합니다.51


이 마지막 파트에서는 FPGA의 광범위한 채택을 가로막는 장애물들을 분석하고, AI 환경에서 FPGA의 미래를 형성할 기술 및 시장 동향을 전망합니다.


이 섹션에서는 역사적으로 FPGA의 AI 분야 활용을 제한해 온 주요 과제들과 이를 극복하기 위한 업계의 노력을 솔직하게 검토합니다.


- **핵심 문제:** 전문적인 HDL 지식의 필요성은 FPGA 도입에 있어 가장 큰 진입 장벽입니다.1 하드웨어 설계에 필요한 사고방식은 소프트웨어 개발과는 근본적으로 다릅니다.17
- **업계의 대응:** 이 문제를 해결하기 위해 업계는 더 높은 수준의 추상화를 향한 다각적인 노력을 기울이고 있습니다.
  - **고수준 합성 (HLS):** C/C++ 기반 개발을 가능하게 합니다.3
  - **AI 특화 툴체인:** Vitis AI와 OpenVINO는 소프트웨어 우선의 워크플로우를 제공합니다.33
  - **AI 주도 설계:** AI 자체를 사용하여 FPGA 구성을 최적화하고, 어려운 설계 과정을 자동화하는 연구가 진행 중입니다.35

이러한 노력의 중심에는 '사용 편의성 대 성능'이라는 근본적인 긴장 관계가 존재합니다. 시장은 더 넓은 채택을 위해 소프트웨어와 유사한 쉬운 도구를 요구하지만 14, HLS와 같은 고수준 추상화 도구는 필연적으로 세부 사항을 사용자로부터 숨깁니다. 자동화된 컴파일 과정은 수동으로 최적화된 RTL 설계만큼 높은 성능이나 효율을 보장하지 못할 수 있습니다. FPGA의 궁극적인 힘은 비트 수준의 세밀한 맞춤화 능력에 있는데 7, 고수준 도구는 본질적으로 이러한 저수준 제어에 대한 접근을 제한합니다. 따라서 FPGA 벤더들은 소프트웨어 개발자를 유치하기 위해 더 많은 추상화를 제공하면서도, 하드웨어를 매력적으로 만드는 바로 그 성능과 효율성을 희생하지 않아야 하는 끊임없는 도전에 직면해 있습니다. FPGA 도구의 미래는 높은 수준의 생산성을 제공하면서도, 필요한 경우 전문가가 저수준으로 '내려가' 최적화를 수행할 수 있는 '스위트 스폿'을 찾는 데 달려 있습니다.


- **핵심 문제:** FPGA는 제한된 온칩 메모리(BRAM)를 가지고 있으며, 칩 외부 DRAM과의 대역폭은 종종 고성능 연산 엔진에 데이터를 공급하지 못하는 병목 현상을 유발합니다.20
- **업계의 대응:**
  - **아키텍처적 해결:** 고대역폭 메모리(High-Bandwidth Memory, HBM)를 FPGA 패키지에 직접 통합하여(예: 인텔 Agilex M-시리즈, AMD Versal HBM 시리즈) 메모리 대역폭을 대폭 향상시킵니다.25
  - **알고리즘적 해결:** 양자화 및 희소성과 같은 기술은 AI 모델의 메모리 점유 공간을 줄여 온칩 자원으로 관리하기 용이하게 만드는 데 매우 중요합니다.23
  - **시스템 수준 해결:** CXL과 같은 새로운 인터커넥트 표준은 호스트 메모리에 대한 더 효율적인 접근을 제공합니다 (섹션 8.2 참조).

7.3 핵심 방법론으로서의 하드웨어-소프트웨어 공동 설계

- **개념:** 최고의 성능을 달성하기 위해서는 AI 모델(소프트웨어)과 가속기 아키텍처(하드웨어)를 함께 설계하는 협력적 접근 방식이 필수적입니다. 데이터 과학자가 단순히 "모델을 벽 너머로 던져" 하드웨어 엔지니어에게 넘기는 방식으로는 최적의 결과를 얻을 수 없습니다.
- **실제적 의미:** 이는 FPGA의 자원(DSP, AI 엔진 등)에 효율적으로 매핑되는 레이어와 연산을 가진 AI 모델을 선택하고, 대상 모델 클래스에 최적의 데이터 흐름을 제공하도록 FPGA 가속기를 설계하는 과정을 포함합니다.30 이러한 긴밀한 피드백 루프는 성공을 위해 필수적입니다.


이 섹션에서는 FPGA 기반 AI의 차세대를 정의할 새로운 기술 동향과 기술들을 탐구합니다.


- **현재 상황:** FPGA 생태계는 주요 벤더들의 독점적인 폐쇄 소스 도구에 의해 지배되어 왔습니다.38
- **새로운 동향:** 오픈소스 FPGA 툴체인(예: Yosys, nextpnr)과 오픈소스 IP 코어(예: GitHub)를 향한 움직임이 커지고 있습니다. 오픈소스 RISC-V ISA의 부상 또한 FPGA를 위한 표준적이고 개방된 프로세서 코어를 제공하는 핵심 동력입니다.3
- **잠재적 영향:** 오픈소스 도구는 진입 비용을 낮추고, 더 큰 커뮤니티를 육성하며, 연구자와 개발자가 도구 자체에 기여하고 실험할 수 있게 함으로써 혁신을 가속화할 수 있습니다.3


- **컴퓨트 익스프레스 링크 (Compute Express Link, CXL):** CXL은 CPU, 가속기, 메모리 장치가 높은 대역폭, 낮은 지연 시간, 그리고 캐시 일관성을 유지하며 통신할 수 있도록 하는 개방형 표준 인터커넥트입니다.35
- **FPGA에 미치는 영향:** CXL은 FPGA에 있어 게임 체인저입니다. 이는 FPGA가 데이터 센터에서 일급 시민(first-class citizen)이 되어 호스트 CPU의 메모리 풀에 직접 접근할 수 있게 해줍니다. 이는 전통적인 PCIe보다 훨씬 효율적인 경로를 제공하여 메모리 병목 현상을 직접적으로 해결합니다.57 또한 필요에 따라 FPGA 풀을 메모리 및 컴퓨팅 풀에 연결하는 분산형 시스템(disaggregated systems)의 생성을 가능하게 합니다.


- **개념:** FPGA의 복잡한 설계 공간(배치, 라우팅, 구성)은 AI 및 머신러닝 최적화 기술을 적용하기에 이상적인 대상입니다.
- **적용:** 연구자들은 강화 학습과 같은 기술을 사용하여 설계 공간을 자동으로 탐색하고, 주어진 AI 모델에 대해 지연 시간, 처리량, 면적의 균형을 맞추는 최적의 가속기 구성을 찾고 있습니다.56 미래에는 AI가 어려운 FPGA 설계 과정의 상당 부분을 자동화할 수 있을 것입니다.35


- **종합:** FPGA는 AI 가속 환경에서 독특하고 필수적인 틈새 시장을 차지하고 있습니다. GPU나 ASIC을 직접적으로 대체하는 것이 아니라, 성능, 전력 효율성, 그리고 가장 중요하게는 유연성의 독보적인 조합을 제공합니다.
- **미래 궤도:** AI 분야에서 FPGA의 미래는 독립형 가속기가 아니라, 더 큰 이기종 시스템의 통합 구성 요소로서의 역할에 있습니다. 그 진화는 더 나은 도구를 통해 소프트웨어-하드웨어 격차를 해소하고, CXL과 같은 표준을 통해 데이터 센터 및 시스템 수준 아키텍처에 더 깊이 통합되는 방향으로 나아가고 있습니다.
- **전략적 권장 사항:** AI 솔루션을 개발하는 조직에게 FPGA는 전략적인 옵션으로 고려되어야 합니다. 특히 실시간, 저지연 성능, 높은 전력 효율성이 요구되거나 알고리즘이 빠르게 진화하는 분야에서 그 가치가 빛을 발합니다. FPGA의 잠재력을 최대한 활용하기 위해서는 HLS 및 최신 툴체인에 대한 전문성을 확보하는 것이 중요할 것입니다.


1. FPGA란 무엇인가? - 쭌3이의 Blog - 티스토리, accessed July 14, 2025, [https://june3lee.tistory.com/entry/FPGA%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80](https://june3lee.tistory.com/entry/FPGA란-무엇인가)
2. FPGA 세부 가이드 : 구조, 작업 원칙, 기능 - 전자 구성 요소 유통 업체, accessed July 14, 2025, https://www.ic-components.kr/blog/fpga-detailed-guide-structure,working-principle,features.jsp
3. FPGA 기반 회로 설계 및 구현: 하드웨어의 마법 세계로 떠나볼까요? - 재능넷, accessed July 14, 2025, https://www.jaenung.net/tree/24773
4. The difference between XIlinx FPGA and Intel FPGA_VEMEKO, accessed July 14, 2025, https://www.vemeko.com/blog/67147.html
5. FPGA Acceleration(가속화), 꼭 알아두어야 할 그것. - 고양이 미로 - 티스토리, accessed July 14, 2025, https://rubber-tree.tistory.com/114
6. AI 가속기란 무엇인가요? - IBM, accessed July 14, 2025, https://www.ibm.com/kr-ko/think/topics/ai-accelerator
7. How FPGA lost to the AI race - amar jay - Medium, accessed July 14, 2025, https://amananjay.medium.com/how-fpga-lost-to-the-ai-race-f810161e2ced
8. FPGA vs. GPU: Comparison for High-Performance Computing and AI - JAK Electronics, accessed July 14, 2025, https://www.jakelectronics.com/blog/fpga-vs-gpu-comparison-for-highperformance-computing-and-ai
9. FPGA vs. GPU: Understanding the Differences and Choosing the Right Technology for Your Application | Xecor, accessed July 14, 2025, https://www.xecor.com/blog/fpga-vs-gpu
10. FPGA vs. GPU for Deep Learning Applications - Intel, accessed July 14, 2025, https://www.intel.com/content/www/us/en/fpga-solutions/artificial-intelligence/fpga-gpu.html
11. Powering the Future of On-Device AI with FPGAs - Embedded, accessed July 14, 2025, https://www.embedded.com/powering-the-future-of-on-device-ai-with-fpgas/
12. Case Study: Successful AI Projects Using FPGA | OrhanErgun.net Blog, accessed July 14, 2025, https://orhanergun.net/case-study-successful-ai-projects-using-fpga
13. The Application of FPGA in Automotive Autonomous Driving, accessed July 14, 2025, https://www.vemeko.com/blog/67146.html
14. Field Programmable Gate Arrays (FPGAs) for Artificial Intelligence (AI), accessed July 14, 2025, https://www.intel.com/content/www/us/en/learn/fpga-for-ai.html
15. FPGA, 구조화된 ASIC, 셀 기반 ASIC 비교 - 인텔, accessed July 14, 2025, https://www.intel.co.kr/content/www/kr/ko/products/programmable/fpga-vs-structured-asic.html
16. AI 시대에서 FPGA 반도체의 역할 - 지식 맛집 - 티스토리, accessed July 14, 2025, https://tristanchoi.tistory.com/661
17. Why are FPGAs not dominating GPUs for neural network inference in the market? - Reddit, accessed July 14, 2025, https://www.reddit.com/r/FPGA/comments/18m05xu/why_are_fpgas_not_dominating_gpus_for_neural/
18. [인공지능 가속기 설계] Artificial Intelligence Accelerator Design (Using Zynq-7000 FPGA, CDMA, AXI) - traumhaft - 티스토리, accessed July 14, 2025, https://sunhong-dev.tistory.com/19
19. FPGA vs. GPU for Deep Learning Applications - IBM, accessed July 14, 2025, https://www.ibm.com/think/topics/fpga-vs-gpu
20. Is this true? Wild claims about Relevance of FPGA's in the Future of AI - Reddit, accessed July 14, 2025, https://www.reddit.com/r/FPGA/comments/1htxgf6/is_this_true_wild_claims_about_relevance_of_fpgas/
21. FPGA-accelerated machine learning inference as a service for particle physics computing - the ECE Department Shared Server - University of Washington, accessed July 14, 2025, https://people.ece.uw.edu/hauck/publications/AcceleratedMachineLearning.pdf
22. The Role Of FPGAs In AI Acceleration - Fidus Systems, accessed July 14, 2025, https://fidus.com/blog/the-role-of-fpgas-in-ai-acceleration/
23. FPGA-based Acceleration for Convolutional Neural Networks: A Comprehensive Review, accessed July 14, 2025, https://arxiv.org/html/2505.13461v1
24. Feeding The Datacenter Inference Beast A Heavy Diet Of FPGAs - The Next Platform, accessed July 14, 2025, https://www.nextplatform.com/2020/07/31/feeding-the-datacenter-inference-beast-a-heavy-diet-of-fpgas/
25. AMD Versal Adaptive SoCs, accessed July 14, 2025, https://www.amd.com/en/products/adaptive-socs-and-fpgas/versal.html
26. FPGAs for Artificial Intelligence (AI) | Altera®, accessed July 14, 2025, https://www.intel.com/content/www/us/en/fpga-solutions/artificial-intelligence/overview.html
27. Beyond Peak Performance: Comparing the Real Performance of AI-Optimized FPGAs and GPUs - Intel, accessed July 14, 2025, https://www.intel.com/content/dam/www/central-libraries/us/en/documents/wp-beyond-peak-performance-ai-optimized.pdf
28. AMD Versal™ AI Core Series - Avnet, accessed July 14, 2025, https://www.avnet.com/americas/products/cp/amd-versal-adaptive-socs/versal-ai-core/
29. A survey on FPGA-based accelerator for ML models This work was supported by the China Scholarship Council - arXiv, accessed July 14, 2025, https://arxiv.org/html/2412.15666v1
30. AI/ML Acceleration on Heterogeneous platforms – FPGA/PARALLEL COMPUTING LAB, accessed July 14, 2025, https://sites.usc.edu/fpga/ai-ml/
31. Flexibility: FPGAs and CAD in Deep Learning Acceleration - Intel, accessed July 14, 2025, https://cdrdv2-public.intel.com/650439/wp-01283-flexibility-fpgas-and-cad-in-deep-learning-acceleration.pdf
32. Hardware Accelerators for Artificial Intelligence - arXiv, accessed July 14, 2025, http://arxiv.org/pdf/2411.13717
33. Vitis AI Tutorial #1 - 작업 환경 - velog, accessed July 14, 2025, https://velog.io/@oilyhand_01/Vitis-AI-Tutorial-1
34. Systolic Sparse Tensor Slices: FPGA Building Blocks for Sparse and Dense AI Acceleration, accessed July 14, 2025, https://arxiv.org/html/2502.03763v1
35. Using FPGAs for High-Performance Computing: Challenges and Opportunities, accessed July 14, 2025, https://runtimerec.com/using-fpgas-for-high-performance-computing-challenges-and-opportunities/
36. FPGA AI Suite - AI 추론 개발 플랫폼 | Altera - 인텔, accessed July 14, 2025, https://www.intel.co.kr/content/www/kr/ko/products/details/fpga/development-tools/fpga-ai-suite.html
37. Comparative Analysis of FPGA and GPU Performance for Machine Learning-Based Track Reconstruction at LHCb - arXiv, accessed July 14, 2025, https://arxiv.org/html/2502.02304v1
38. Future of FPGAs? : r/FPGA - Reddit, accessed July 14, 2025, https://www.reddit.com/r/FPGA/comments/185cskg/future_of_fpgas/
39. Altera FPGAs & SoC FPGAs | Accelerating Innovators, accessed July 14, 2025, https://www.altera.com/
40. Agilex™ FPGA and SoC FPGA Family - Intel, accessed July 14, 2025, https://www.intel.com/content/www/us/en/products/details/fpga/agilex.html
41. embedded world: Altera optimizes FPGAs for edge AI, accessed July 14, 2025, https://www.embedded.com/embedded-world-altera-optimizes-fpgas-for-edge-ai/
42. Altera launches Agilex 3 FPGAs for the intelligent edge - GamesBeat, accessed July 14, 2025, https://gamesbeat.com/altera-launches-agilex-3-fpgas-for-the-intelligent-edge/
43. Versal AI Core - Xilinx Wiki - Confluence, accessed July 14, 2025, https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/747012115/Versal+AI+Core
44. FPGA Artificial Intelligence (AI) Design | FPGA AI Projects - Promwad, accessed July 14, 2025, https://promwad.com/services/embedded/fpga-design/ai
45. [Blog] Revolutionizing Edge AI: The Role of FPGAs in Smart Camera Optimization, accessed July 14, 2025, https://www.latticesemi.com/en/Blog/2024/06/17/15/24/Revolutionizing-Edge-AI-The-Role-of-FPGAs-in-Smart-Camera-Optimization
46. [Blog] Contextual AI: Enhancing Edge Intelligence with FPGA Technology, accessed July 14, 2025, https://www.latticesemi.com/en/Blog/2025/02/12/20/42/Contextual-AI-Enhancing-Edge-Intelligence-with-FPGA-Technology
47. Edge Computing with FPGAs | Efinix, Inc., accessed July 14, 2025, https://www.efinixinc.com/blog/edge-computing-with-fpgas.html
48. Automotive FPGA - Altera® FPGAs - Intel, accessed July 14, 2025, https://www.intel.com/content/www/us/en/fpga-solutions/automotive/overview.html
49. The Role of FPGAs in Autonomous Vehicle Development - Fpga Insights, accessed July 14, 2025, https://fpgainsights.com/fpga/role-of-fpgas-in-autonomous-vehicle/
50. FPGAs in Self-Driving Cars: Accelerating Perception and Decision-Making - Fpga Insights, accessed July 14, 2025, https://fpgainsights.com/fpga/fpgas-in-self-driving-cars-accelerating-perception-and-decision-making/
51. The Application of FPGA in the Medical Field, accessed July 14, 2025, https://www.vemeko.com/blog/67179.html
52. Medical Imaging Process Accelerated in FPGA Hardware by 82x over Software Line of Reaction Estimation for a PET scanner Optimize - the ECE Department Shared Server - University of Washington, accessed July 14, 2025, https://people.ece.uw.edu/hauck/publications/LOR_eetimes.pdf
53. A Quality of Service Analysis of FPGA-Accelerated Conv2D Architectures for Brain Tumor Multi-Classification - Tech Science Press, accessed July 14, 2025, https://www.techscience.com/cmc/online/detail/23751/pdf
54. FPGA Hardware Acceleration of AI Models for Real-Time Breast Cancer Classification, accessed July 14, 2025, https://www.mdpi.com/2673-2688/6/4/76
55. Accelerate Medical Device Development with FPGAs - DornerWorks, accessed July 14, 2025, https://www.dornerworks.com/blog/medical-device-development-fpgas/
56. Design Methodologies for Deep Learning Accelerators on Heterogeneous Architectures - arXiv, accessed July 14, 2025, https://arxiv.org/pdf/2311.17815
57. A Comprehensive Simulation Framework for CXL Disaggregated Memory - arXiv, accessed July 14, 2025, https://arxiv.org/html/2411.02282v3


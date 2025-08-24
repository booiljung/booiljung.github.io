[NVIDIA Jetson AGX Orin](./index.md)
# NVIDIA Jetson AGX Orin 플랫폼의 결정론적 및 항공전자 네트워크 통합


본 보고서는 상용 기성품(COTS) AI 컴퓨터인 NVIDIA Jetson AGX Orin가 차세대 항공우주 및 국방 시스템의 핵심 미션 컴퓨터로서의 잠재력을 심층 분석한다. 분석 대상은 AFDX, TTEthernet, TSN, ARINC 429, MIL-STD-1553 등 5가지 핵심 통신 프로토콜의 적용 가능성이다. 분석 결과, Jetson AGX Orin는 강력한 컴퓨팅 성능과 최신 I/O 인터페이스를 바탕으로 해당 프로토콜들을 통합할 수 있는 높은 잠재력을 지니고 있음을 확인했다.

각 프로토콜의 통합은 주로 PCIe 및 M.2와 같은 표준 인터페이스를 통해 전문 하드웨어 카드를 추가하는 방식으로 이루어진다. 이 과정에서 가장 중요한 변수는 하드웨어의 물리적 호환성이 아니라, Jetson Orin의 `aarch64` 아키텍처를 지원하는 Linux 드라이버의 가용성이다.

- **MIL-STD-1553**은 Jetson Orin의 M.2 슬롯에 직접 장착 가능한 상용 카드가 존재하여 통합 타당성이 **'높음'**으로 평가되었다.
- **ARINC 429, TTEthernet, AFDX**는 PCIe 기반의 강력한 하드웨어 솔루션이 존재하지만, `aarch64` 드라이버 지원 여부를 각 벤더사에 확인해야 하므로 타당성은 **'중간'**으로 평가된다.
- **TSN**은 Jetson Orin에 내장된 이더넷 컨트롤러의 하드웨어 지원 여부가 불확실하여 타당성이 **'중간-낮음'**으로 평가되나, 전문 PCIe 카드를 통해 구현할 수 있다.

결론적으로, Jetson AGX Orin는 최신 AI 기반 항공전자 시스템 개발을 위한 강력하고 유연한 플랫폼이다. 본 보고서는 시스템 통합을 위한 구체적인 전략과 우선순위가 명시된 실행 로드맵을 제시하여, 성공적인 기술 도입을 위한 지침을 제공한다.


NVIDIA Jetson AGX Orin는 단순한 개발 보드를 넘어, 항공우주 및 국방 분야에서 요구하는 고성능 미션 컴퓨팅의 핵심 요소로 자리매김할 잠재력을 가지고 있다. 이 섹션에서는 Jetson AGX Orin의 핵심 아키텍처, I/O 역량, 그리고 소프트웨어 기반을 분석하여, 이 플랫폼이 왜 차세대 지능형 시스템에 적합한지 규명한다.


Jetson AGX Orin의 성능은 단순한 수치를 넘어, 항공전자 시스템이 직면한 복잡한 데이터 처리 요구사항을 해결할 수 있는 능력에서 비롯된다.

- **처리 성능:** Jetson AGX Orin 64GB 모듈은 12코어 Arm Cortex-A78AE CPU와 64개의 텐서 코어를 포함한 2048코어 NVIDIA Ampere 아키텍처 GPU를 탑재하여, 최대 275 TOPS(INT8)의 AI 연산 성능을 제공한다.1 이는 전통적인 항공전자 컴퓨터의 성능을 훨씬 뛰어넘는 수준으로, AI 기반 센서 퓨전, 자율 항법, 복잡한 데이터 분석과 같은 작업을 실시간으로 처리할 수 있는 기반이 된다.2
- **메모리 및 스토리지:** 204.8 GB/s의 대역폭을 가진 64GB LPDDR5 메모리와 64GB의 eMMC 5.1 온보드 스토리지를 갖추고 있다.1 이 고속 메모리 아키텍처는 여러 처리 엔진에 데이터를 끊임없이 공급하고, 고대역폭 센서로부터 입력되는 방대한 데이터셋을 효과적으로 처리하는 데 필수적이다.
- **기능 안전(Functional Safety)의 함의:** Arm Cortex-A78**AE** CPU의 탑재는 이 플랫폼의 가장 중요한 특징 중 하나이다. 여기서 'AE'는 'Automotive Enhanced'를 의미하며, 이는 기능 안전(FuSa) 애플리케이션을 위해 설계된 기능들이 포함되어 있음을 시사한다.1 항공 및 자동차 분야에서 시스템 인증을 받기 위한 핵심 전제 조건인 스플릿-록(Split-Lock) 모드와 같은 기능이 이에 해당한다. 이러한 설계는 NVIDIA가 Jetson Orin을 단순한 소비자용 제품을 넘어, 높은 신뢰성과 안전성이 요구되는 시장을 목표로 하고 있음을 명확히 보여준다. 즉, Jetson Orin 모듈 자체가 사전 인증된 제품은 아니지만, Cortex-A78AE 코어에 내장된 이중 코어 록스텝(Dual-Core Lock-Step, DCLS)과 같은 하드웨어 기능은 시스템 통합 업체가 항공전자 분야의 DAL(Design Assurance Level)과 같은 안전 무결성 수준을 달성하기 위한 인증 과정에서 활용할 수 있는 결정적인 기반을 제공한다. 이는 Orin 플랫폼을 단순 '고성능' COTS 보드에서 '잠재적 인증 가능' 고성능 COTS 플랫폼으로 격상시키는 중요한 요소이다.


Jetson AGX Orin는 풍부하고 현대적인 I/O 구조를 통해 다양한 항공전자 인터페이스를 통합할 수 있는 유연성을 제공한다.

- **PCI Express (PCIe):** 2개의 x8, 1개의 x4, 2개의 x1 레인을 포함하는 광범위한 PCIe Gen4를 지원한다.1 Jetson AGX Orin 개발자 키트 캐리어 보드는 x8 Gen4로 동작하는 x16 슬롯과 다수의 M.2 슬롯을 제공하여, 전문 I/O 카드를 통합하기 위한 고속의 다목적 인터페이스를 보장한다.6
- **M.2 슬롯:** 개발자 키트는 PCIe x4 Gen4를 지원하는 M.2 Key M 슬롯과 PCIe x1 Gen4, USB 등을 지원하는 M.2 Key E 슬롯을 제공한다.6 이 슬롯들은 NVMe 스토리지나 소형 통신 모듈과 같은 최신 주변기기를 추가하는 데 매우 중요하다.
- **네이티브 네트워킹:** 모듈 자체에 1개의 10GbE와 1개의 1GbE 컨트롤러가 내장되어 있으며, 개발자 키트에서는 RJ45 포트로 제공된다.1 이는 별도의 하드웨어 없이도 즉시 고대역폭 네트워크 연결을 가능하게 한다.
- **기타 I/O:** 40핀 헤더를 통해 CAN, SPI, I2C, UART와 같은 인터페이스를 제공하여, 저대역폭 제어 및 진단 작업에 유용하게 사용될 수 있다.4

이처럼 다양한 고속 인터페이스의 조합은 시스템 설계자에게 큰 유연성을 부여한다. 5가지의 각기 다른 전문 I/O를 연결해야 하는 본 보고서의 요구사항에 대해, AGX Orin는 충분한 고속 PCIe 레인을 제공한다.1 이를 통해 시스템 설계자는 주 PCIe 슬롯에 고대역폭 AFDX/TTEthernet 카드를, M.2 슬롯에 MIL-STD-1553 카드를, 그리고 M.2 어댑터를 이용한 PCIe Mini Card에 ARINC 429 카드를 동시에 연결하면서도 I/O 병목 현상을 피할 수 있다. 이 강력한 I/O 확장성은 본 타당성 분석의 물리적 기반이 된다.


Jetson Orin의 소프트웨어 환경은 최신 기술을 기반으로 하지만, 이로 인해 특수 산업 분야에서는 새로운 도전 과제가 발생할 수 있다.

- **보드 지원 패키지(BSP):** NVIDIA는 공식 BSP로 Jetson Linux(구 L4T)를 제공한다. 최신 36.x 버전은 Linux 커널 5.15와 Ubuntu 22.04 루트 파일 시스템을 기반으로 한다.10
- **아키텍처:** 전체 소프트웨어 스택은 `aarch64`(ARM64) 아키텍처를 위해 구축되었다.12 이는 제3자 드라이버 호환성을 결정하는 가장 중요한 요소이다.
- **개발 생태계:** JetPack SDK는 BSP와 함께 CUDA, cuDNN, TensorRT 등 AI 개발 라이브러리를 번들로 제공하여 포괄적인 개발 환경을 지원한다.2

최신 5.15 커널의 사용은 보안 및 일반 하드웨어 지원 측면에서 큰 장점이다.10 그러나 항공우주와 같이 변화가 느린 산업에서는 오히려 걸림돌이 될 수 있다. 이들 산업의 하드웨어 벤더들은 종종 구형 장기 지원(LTS) 커널(예: L4T 구버전에서 사용된 4.9 커널 13)용 드라이버만 제공하고 검증하는 경우가 많다. 이로 인해 '드라이버 갭'이 발생할 수 있다. 시스템 통합 업체는 벤더의 드라이버를 최신 커널에 맞게 백포팅(backporting)하거나, 벤더에게 업데이트된 드라이버를 요청해야 하는 상황에 직면할 수 있다. 이러한 소프트웨어 통합 리스크는 종종 하드웨어 통합 리스크보다 더 중요하게 고려되어야 한다.


항공전자 시스템에서 결정론적 통신과 실시간 응답성은 타협할 수 없는 요구사항이다. 이 섹션에서는 Jetson Orin의 내장 기능(TSN)과 Linux에서 하드 실시간(hard real-time) 성능을 확보하기 위한 표준적인 방법(PREEMPT_RT)을 평가한다.


TSN은 표준 이더넷을 통해 결정론적 통신을 구현하기 위한 IEEE 802.1 표준의 집합이다. Jetson Linux가 사용하는 Linux 커널 5.15는 시간 인식 셰이핑(time-aware shaping), 프레임 선점(preemption), 스케줄링 등 다수의 TSN 구성 요소를 광범위하게 지원한다.10

하지만 커널 지원만으로는 완전한 TSN 기능을 구현할 수 없다. 특히 시간 동기화를 위한 IEEE 802.1AS(gPTP)나 시간 인식 셰이퍼(Time-Aware Shaper)를 위한 802.1Qbv와 같은 핵심 기능은 이더넷 컨트롤러(MAC/PHY) 하드웨어의 명시적인 지원을 필요로 한다. Jetson Orin의 데이터시트에는 10GbE/1GbE 컨트롤러의 정확한 모델명이 명시되어 있지 않아 1, 문서만으로는 TSN 기능 지원 여부를 확신할 수 없다.

소프트웨어의 잠재력과 하드웨어의 현실 사이에는 명확한 인과 관계가 존재한다. 최신 Linux 커널은 TSN 구현의 *잠재력*을 제공하지만, 이 잠재력은 기반이 되는 실리콘(하드웨어)이 이를 지원할 때만 실현될 수 있다. 애플리케이션에서 커널, 드라이버를 거쳐 하드웨어로 이어지는 의존성 사슬에서, 만약 MAC이 필요한 하드웨어 큐나 타임스탬핑 기능을 갖추지 못했다면 전체 기능은 실패한다. 따라서 Jetson Orin의 내장 포트를 이용한 TSN 구현은 그 자체로 별도의 연구 과제가 되며, 사용된 하드웨어에 대한 심층적인 조사나 NVIDIA에 직접적인 문의가 필요하다. 더 신뢰성 있는 대안은 알려진 TSN 지원 하드웨어를 탑재한 전용 PCIe 카드를 사용하는 것이다.


마이크로초 수준의 결정론적 응답이 필요한 하드 실시간 작업을 위해서는 표준 Linux 커널만으로는 부족하다. PREEMPT_RT 패치는 Linux를 하드 실시간 운영 체제(RTOS)로 전환하기 위한 사실상의 표준이다.14 Jetson에 실시간 커널을 구축하는 일반적인 과정은 커널 소스 다운로드, RT 패치 적용, 커널 및 모듈 빌드, 그리고 디바이스 플래싱 순으로 진행된다.16

그러나 이 과정은 간단하지 않으며, 개발자 커뮤니티의 수많은 논의를 통해 공통적인 문제점과 해결책이 알려져 있다.12 PREEMPT_RT 패치 적용은 시작에 불과하며, 진정한 실시간 성능을 달성하기 위해서는 플랫폼 아키텍처에 대한 이해를 바탕으로 한 상당한 수준의 시스템 튜닝과 디버깅이 필요하다. 이는 시스템 설계자가 이 튜닝 및 검증 단계를 위해 상당한 엔지니어링 시간을 예산에 반영해야 함을 의미한다. 타당성은 높지만, 그 노력은 결코 작지 않다.

**표 II-1: Jetson Orin을 위한 PREEMPT_RT 통합 문제 해결 가이드**

| 문제 설명                      | 일반적인 증상                                                | 근본 원인 분석                                               | 권장 해결책                                                  | 관련 자료 |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| **디스플레이 실패**            | 부팅 스플래시 화면 후 검은 화면 출력, DisplayPort 신호 없음  | PREEMPT_RT 커널로 재빌드 시, 독점 NVIDIA 디스플레이 드라이버에 대한 커널 모듈 의존성 파일(`modules.dep`)이 업데이트되지 않아 드라이버 로딩 실패 | 1. 시리얼 콘솔로 부팅 후 타겟에서 sudo depmod -a 명령어 실행2. 플래싱 전 호스트 PC의 chroot 환경에서 의존성 파일 수동 업데이트 | 18        |
| **성능 저하 또는 개선 없음**   | `cyclictest` 실행 시, 지연 시간(latency)이 표준 커널보다 높거나 유사하게 측정됨 | 1. 부적절한 테스트 방법론 (비교군인 비-RT 커널에 부하를 주지 않음)2. CPU 주파수 스케일링, 절전 기능 등 백그라운드 서비스의 간섭 | 1. sudo jetson_clocks로 CPU/GPU 클럭 고정2. stress와 같은 도구로 시스템에 부하를 주어 비교 기준 설정3. CPU 선호도(affinity) 설정을 통해 실시간 작업을 특정 코어에 고정 | 12        |
| **CPU 코어 간 지연 시간 편차** | `cyclictest` 결과, 특정 CPU 코어 클러스터에서 다른 클러스터보다 높은 지연 시간이 관측됨 | Jetson Orin의 이기종 멀티 코어 아키텍처(예: 성능 코어와 효율 코어 클러스터)와 관련된 전력 관리 정책 및 캐시 구조의 차이 | 1. 실시간 작업은 가장 예측 가능한 성능을 보이는 코어 클러스터에 고정2. nvpmodel 또는 커널 파라미터 조정을 통해 전력 관리 정책 튜닝 | 22        |
| **부팅 실패**                  | RT 패치 적용 후 시스템이 부팅되지 않거나 부팅 루프에 빠짐    | 커널 설정(config) 오류, 부트로더와 커널 이미지 간의 비호환성, 또는 잘못된 플래싱 절차 | 1. 커널 설정 시 PREEMPT_RT 관련 옵션 외에 필수 드라이버가 비활성화되지 않았는지 확인2. NVIDIA 공식 문서에 따라 플래싱 절차를 정확히 준수3. 시리얼 콘솔 로그를 통해 부팅 과정의 에러 메시지 분석 | 19        |


이 섹션에서는 두 가지 핵심 고속 결정론적 이더넷 표준의 통합 가능성을 평가한다. 여기서 핵심은 하드웨어 가용성과 `aarch64` 드라이버 지원 간의 연결 고리이다.


TTEthernet은 시간 트리거(time-triggered), 비율 제한(rate-constrained, ARINC 664와 유사), 그리고 표준 이더넷 트래픽을 단일 네트워크에서 통합하는 고도의 결정론적, 내고장성 이더넷 표준이다.

- **하드웨어 솔루션:** 이 분야의 주요 벤더는 TTTech Aerospace이며, 주력 제품은 TTEthernet PCIe 카드이다.27 이 카드는 클럭 동기화 알고리즘을 포함한 전체 TTEthernet 프로토콜 스택을 FPGA에 오프로드하여 호스트에는 표준 PCIe 인터페이스를 제공한다. 하드웨어 통합 자체는 간단하다. TTTech의 PCIe 카드는 Jetson Orin의 PCIe 슬롯과 물리적으로 완벽하게 호환된다.6

- **드라이버 및 소프트웨어 지원:** 제품 데이터시트에는 "Linux 드라이버 사용 가능 (64비트/32비트 Linux)"이라고 명시되어 있으나 27, 아키텍처(x86_64 vs 

  `aarch64`)는 특정하지 않았다. TTTech는 Linux용 개발 시스템도 제공하고 있다.28

이 프로젝트의 성패는 전적으로 `aarch64` 드라이버의 가용성에 달려있다. 물리적 연결과 프로토콜의 복잡성은 이미 해결된 문제이다. 남은 유일한 변수는 이들을 연결하는 소프트웨어 '접착제', 즉 드라이버이다. 만약 제공되는 Linux 드라이버가 소스 코드 형태로 제공되어 `aarch64` 아키텍처용으로 컴파일이 가능하다면 통합은 현실적이다. 만약 사전 컴파일된 `aarch64` 바이너리가 존재한다면 통합 리스크는 매우 낮아진다. 데이터시트에서 "다른 운영 체제로의 포팅은 요청 시 제공될 수 있다"고 언급한 점은 27 긍정적인 신호이다. 따라서 가장 중요한 첫 단계는 TTTech에 `aarch64` 드라이버 지원 여부를 공식적으로 문의하는 것이다.


AFDX는 에어버스 A380/A350, 보잉 787 등 최신 민항기에서 사용되는 결정론적 네트워킹 표준으로, 가상 링크(Virtual Link, VL)를 통해 대역폭과 지연 시간을 보장하는 데 중점을 둔다.

- **하드웨어 솔루션:** AIM-Online은 APE-FDX-2 PCIe 카드를 통해 핵심적인 솔루션을 제공한다.31 TTEthernet 카드와 마찬가지로, 이 제품은 온보드 SoC(Xilinx Zynq)를 사용하여 전체 프로토콜 스택을 관리함으로써 호스트 CPU의 부하를 최소화한다.32

- **드라이버 및 소프트웨어 지원:** AIM의 데이터시트는 일관되게 "Windows 및 Linux"용 드라이버를 제공한다고 명시하고 있다.31 그러나 공개된 문서에서는 

  `aarch64` 호환성이 명시되어 있지 않아 직접적인 확인이 필요하다.31 AIM은 완전한 보드 소프트웨어 패키지(BSP)와 API를 제공한다.31

AFDX의 통합 경로와 리스크 프로파일은 TTEthernet과 기능적으로 동일하다. 즉, 전문 PCIe 카드를 사용하는 하드웨어 기반 솔루션이며, 그 타당성은 벤더가 `aarch64` Linux 드라이버를 제공하는지에 따라 결정된다. 역사적으로 항공전자 테스트 및 시뮬레이션 시장은 x86 기반 시스템이 지배해왔다. Jetson과 같은 고성능 ARM 기반 SoC의 등장은 비교적 새로운 추세이므로, `aarch64` 지원이 기본적으로 제공되지 않을 수 있다. 따라서 TTEthernet과 AFDX의 Jetson Orin 통합 타당성은 '중간'으로 평가되며, 이는 벤더의 `aarch64` 드라이버 지원 여부에 달려있다.


이 섹션에서는 두 가지 핵심 직렬 데이터 버스 표준인 ARINC 429와 MIL-STD-1553의 통합을 분석한다. 이들 프로토콜의 경우, 더 작고 현대적인 폼팩터의 가용성이 중요한 차별점이 된다.


ARINC 429는 상업용 및 수송 항공기에서 널리 사용되는 단방향, 점대점 직렬 데이터 버스이다.

- **하드웨어 솔루션:** AIM-Online의 AMEE429-x는 PCIe Mini Card 폼팩터로 제공되는 주목할 만한 제품이다.34 이 폼팩터는 임베디드 시스템에 매우 적합하며, 간단한 어댑터를 통해 Jetson의 M.2 슬롯에 사용할 수 있다. DDC, UEI와 같은 다른 벤더들도 다양한 폼팩터의 솔루션을 제공한다.35 상대적으로 저대역폭 인터페이스를 위해 큰 PCIe 슬롯을 사용하는 것은 비효율적이므로, PCIe Mini Card 형태의 ARINC 429 인터페이스는 소형, 경량, 저전력(SWaP)에 최적화된 시스템 설계에 상당한 이점을 제공한다.

- **드라이버 및 소프트웨어 지원:** AIM은 AMEE429-x용 Linux 드라이버를 제공하지만 34, 

  `aarch64` 지원 여부는 공개 문서에 명시되어 있지 않아 직접적인 문의가 필요하다.34

컴팩트한 폼팩터의 가용성은 Jetson Orin 기반의 소형 시스템 구축에 매력적인 요소이다. 이는 전체 시스템 설계를 더 효율적으로 만들고, 고속 PCIe 슬롯을 AFDX 카드나 고성능 프레임 그래버와 같은 더 까다로운 주변 장치를 위해 남겨둘 수 있게 한다. 드라이버 확인 문제로 인해 타당성은 '중간-높음'으로 평가되지만, 하드웨어 솔루션 자체는 매우 매력적이다.


MIL-STD-1553은 군용 항공전자 시스템에서 지배적으로 사용되는 명령/응답 방식의 이중 중복 직렬 데이터 버스이다.39

- **하드웨어 솔루션:**

  - **핵심 발견:** ALPHI Technology의 M.2-1553-2 카드는 이 분석에서 가장 중요한 발견 중 하나이다.41 이 카드는 M.2 2242(30x42mm) Key B & M 모듈 형태로 두 개의 이중 중복 1553 채널을 제공한다. 이는 Jetson Orin 개발자 키트의 M.2 M-Key 슬롯에 직접 장착할 수 있는 완벽한 솔루션이다.
  - **기타 벤더:** DDC, UEI, AIT와 같은 다른 벤더들도 PCIe, XMC, USB, 이더넷 등 다양한 폼팩터의 MIL-STD-1553 솔루션을 제공한다.44

- **드라이버 및 소프트웨어 지원:** ALPHI Technology M.2-1553-2 데이터시트에는 "Linux® 드라이버"가 사용 가능하다고 명시되어 있다.41

  `aarch64`가 명시적으로 확인되지는 않았지만, M.2라는 현대적인 폼팩터는 이 제품의 목표 시장이 Jetson과 같은 최신 ARM 기반 SoC를 포함하고 있음을 강력하게 시사한다.

MIL-STD-1553과 같은 전문 군용 항공전자 인터페이스가 M.2라는 현대적인 COTS 폼팩터로 제공된다는 사실은 중요한 시장 변화를 나타낸다. 이는 전통적인 국방 임베디드 시장이 Jetson 제품군과 같은 차세대 고성능 COTS SoC와 호환되도록 제품을 적극적으로 조정하고 있음을 보여준다. 과거에 MIL-STD-1553 인터페이스는 VME나 cPCI와 같은 크고 전문화된 보드 또는 PMC/XMC 메자닌 형태로 제공되었다. M.2로의 전환은 41 새로운 시스템의 '두뇌'가 점점 더 Jetson과 같은 소형의 강력한 SoM(System-on-Module)이 되고 있음을 인정한 것이다. 벤더들은 이러한 SoM의 네이티브 인터페이스에 직접 연결되는 I/O 솔루션을 만들어내고 있으며, 이는 시스템 통합을 극적으로 단순화하고 SWaP를 줄이며, 군용 애플리케이션에서 Jetson과 같은 플랫폼을 사용하는 장벽을 낮춘다. 이로 인해 MIL-STD-1553 통합의 타당성은 최종 드라이버 확인을 전제로 **'높음'**으로 평가된다.


이 마지막 섹션에서는 분석 결과를 실용적으로 종합하여, 구체적인 전략과 실행 가능한 로드맵을 제공한다.


- **전략 1: 기성품을 활용한 프로토타이핑:** 표준 NVIDIA Jetson AGX Orin 개발자 키트를 기반으로 시작하는 접근 방식이다.6 이 방법은 III 및 IV 섹션에서 식별된 제3자 인터페이스 카드들을 PCIe 및 M.2 슬롯에 장착하여 초기 타당성 검증, 드라이버 개발, 소프트웨어 통합을 진행하는 것이다. 이는 가장 빠르고 저렴하게 프로젝트를 시작할 수 있는 경로이다.
- **전략 2: 맞춤형/강화형 캐리어 보드를 통한 양산:** 실제 운용 환경에 배치 가능한 강화(rugged) 시스템으로 나아가는 경로이다. 이는 Diamond Systems 47, ADLINK 5, 또는 Connect Tech 51와 같은 전문 업체와 협력하는 것을 포함한다. 이들 회사는 필요한 I/O(Diamond Systems의 경우 MIL-STD-1553을 명시적으로 포함 48)를 보드에 직접 통합하고, 강화 커넥터, 향상된 전원 공급 장치, 그리고 군용 및 항공우주 환경에 적합한 전도 또는 대류 냉각 솔루션을 제공하는 맞춤형 캐리어 보드를 설계 및 제조하는 서비스를 제공한다.47 실제로 Quantum Systems는 자사의 Vector 군용 드론에 듀얼 Jetson Orin 모듈을 사용하여 이 분야에서의 실제 배치 사례를 보여주고 있다.54

이러한 제3자 벤더 생태계는 COTS Jetson 모듈과 실제 배치된 항공전자 시스템의 요구사항 사이의 간극을 메워주는 명확한 경로를 제공한다. 국방 분야에서 COTS를 사용할 때의 주요 리스크 중 하나인 '강화 갭(ruggedization gap)'은 이러한 전문 업체들을 통해 해결될 수 있다. 표준 개발 키트는 실험실 등급의 도구이지만, Diamond Systems와 같은 회사는 Jetson과 같은 검증된 SoM을 기반으로 특정 애플리케이션에 맞는 강화 시스템을 구축하는 것을 비즈니스 모델로 삼고 있다.49 이들이 MIL-STD-1553을 맞춤형 I/O 기능으로 명시하고 48 맞춤형 Jetson 캐리어에 대한 사례 연구를 보유하고 있다는 사실은 48, 이미 이러한 종류의 문제를 해결한 경험이 있음을 의미하며, 이는 양산 단계로의 전환에 대한 리스크를 크게 줄여준다.


아래 표는 각 프로토콜의 통합 타당성을 요약한 것이다. 이는 프로젝트 계획, 리스크 관리, 자원 배분에 직접적인 정보를 제공한다.

**표 V-1: Jetson AGX Orin의 프로토콜 통합 타당성 매트릭스**

| 프로토콜         | 주요 하드웨어 인터페이스         | 핵심 벤더        | `aarch64` 드라이버 상태    | 주요 과제                                         | 전체 타당성   |
| ---------------- | -------------------------------- | ---------------- | -------------------------- | ------------------------------------------------- | ------------- |
| **MIL-STD-1553** | M.2                              | ALPHI Technology | 확인 필요 (가능성 높음)    | 드라이버 컴파일 및 검증                           | **높음**      |
| **ARINC 429**    | PCIe Mini Card (M.2 어댑터 사용) | AIM-Online       | 벤더 확인 필요             | `aarch64` 드라이버 확보                           | **중간-높음** |
| **TTEthernet**   | PCIe                             | TTTech Aerospace | 벤더 확인 필요             | `aarch64` 드라이버 확보                           | **중간**      |
| **AFDX**         | PCIe                             | AIM-Online       | 벤더 확인 필요             | `aarch64` 드라이버 확보                           | **중간**      |
| **TSN**          | 내장 이더넷 / PCIe               | (다수)           | 커널 지원, 하드웨어 미확인 | 내장 컨트롤러의 TSN 기능 확인 또는 전용 카드 사용 | **중간-낮음** |

**시스템 아키텍트를 위한 실행 로드맵:**

1. **하드웨어 조달:** Jetson AGX Orin 64GB 개발자 키트 6와 ALPHI Technology의 M.2-1553-2 카드 41를 초기 테스트베드로 확보한다.
2. **벤더 협의 (핵심 경로):** TTTech, AIM-Online, ALPHI Technology에 즉시 연락하여 Jetson Linux(커널 5.15)에 대한 `aarch64` Linux 드라이버의 가용성과 지원을 공식적으로 요청한다. 이는 가장 높은 우선순위를 갖는 조치이다.
3. **실시간 커널 검증:** 동시에, 기본 Jetson Orin에서 PREEMPT_RT 패치 실험을 시작한다. 본 보고서의 문제 해결 가이드(표 II-1)를 활용하여 디스플레이 드라이버 및 성능 튜닝과 관련된 알려진 문제를 해결하고, 안정적인 저지연 RTOS 환경을 구축한다.
4. **우선순위 기반 통합 테스트:** 가장 타당성이 높은 프로토콜인 M.2 MIL-STD-1553 카드 통합 및 테스트부터 시작한다. 이를 통해 빠른 성공 사례를 확보하고 플랫폼에 대한 신뢰를 구축한다.
5. **테스트 확장:** 벤더의 피드백에 따라 TTEthernet, AFDX, ARINC 429 카드를 조달하고 테스트를 확장한다.
6. **양산 계획:** 소프트웨어 타당성이 입증되면, Diamond Systems와 같은 맞춤형 캐리어 보드 벤더와 협력하여 48 실제 운용 환경을 위한 강화 시스템 설계 프로세스를 시작한다. 이들에게 필요한 I/O 목록과 환경 사양을 제공한다.


1. NVIDIA Jetson AGX Orin Series - Mouser Electronics, accessed July 13, 2025, https://www.mouser.com/datasheet/2/744/Seeed_Studio_06132023_102110758-3216382.pdf
2. NVIDIA Jetson AGX Orin | Datasheet - Open Zeka, accessed July 13, 2025, https://openzeka.com/en/wp-content/uploads/2022/08/Jetson-AGX-Orin-Module-Series-Datasheet.pdf
3. NVIDIA® Jetson AGX Orin™ 64GB Developer Kit - Stereolabs, accessed July 13, 2025, https://www.stereolabs.com/store/products/nvidia-jetson-agx-orin-developer-kit
4. NVIDIA Jetson AGX Orin Developer Kit, Server-class AI performance at the edge, up to 275 TOPS, Options for 32GB/64GB memory - Waveshare, accessed July 13, 2025, https://www.waveshare.com/jetson-agx-orin-developer-kit.htm
5. NVIDIA Jetson Orin | Edge AI Platforms - ADLINK Technology, accessed July 13, 2025, https://www.adlinktech.com/en/nvidia-jetson-orin
6. NVIDIA Jetson AGX Orin 64GB Developer Kit - SparkFun Electronics, accessed July 13, 2025, https://www.sparkfun.com/nvidia-jetson-agx-orin-64gb-developer-kit.html
7. Jetson AGX Orin Developer Kit User Guide - Hardware Layout, accessed July 13, 2025, https://developer.nvidia.com/embedded/learn/jetson-agx-orin-devkit-user-guide/developer_kit_layout.html
8. NVIDIA Jetson AGX Orin Developer Kit | Datasheet, accessed July 13, 2025, https://static6.arrow.com/aropdfconversion/arrowresources/3e0f93dc18d2fbe2cceb9b8218d496713cc9b980/jetson-agx-orin-developer-kit-datasheet-nvidia-a4-2203098-r1-web.pdf
9. NVIDIA Jetson AGX Orin 64GB Developer Kit - Open Zeka, accessed July 13, 2025, https://openzeka.com/wp-content/uploads/2023/03/jetson-agx-orin-64GB-developer-kit-datasheet-web-us.pdf
10. Jetson Linux - NVIDIA Developer, accessed July 13, 2025, https://developer.nvidia.com/embedded/jetson-linux
11. Jetson Linux - NVIDIA Developer, accessed July 13, 2025, https://developer.nvidia.com/embedded/jetson-linux-r3643
12. Jetson orin AGX PREEMPT-RT RT-test - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/jetson-orin-agx-preempt-rt-rt-test/330785
13. L4T - NVIDIA Developer, accessed July 13, 2025, https://developer.nvidia.com/embedded/linux-tegra-r321
14. Is it possible to working real time with Linux? : r/embedded - Reddit, accessed July 13, 2025, https://www.reddit.com/r/embedded/comments/10p23zm/is_it_possible_to_working_real_time_with_linux/
15. Real-time kernel for NVIDIA AGX Xavier | by Roman - Medium, accessed July 13, 2025, https://r7vme.medium.com/real-time-kernel-for-nvidia-agx-xavier-b660e107a211
16. Real Time Kernel On Nvidia Jetson TX2 - Numerickly, accessed July 13, 2025, https://www.numerickly.com/2018/12/30/real-time-kernel-on-nvidia-jetson-tx2/
17. Jetson AGX Orin - RT Linux without reflashing - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/jetson-agx-orin-rt-linux-without-reflashing/283832
18. No display with PREEMPT_RT patches - Jetson AGX Orin - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/no-display-with-preempt-rt-patches/240876
19. Orin nx16g Install the preempt_rt patch - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/orin-nx16g-install-the-preempt-rt-patch/288952
20. JetPack 5.1 RT patch not working - Jetson AGX Orin - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/jetpack-5-1-rt-patch-not-working/247771
21. Topics tagged preempt_rt - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/tag/preempt_rt
22. Observing Latency Discrepancies on NVIDIA Jetson Nano with PREEMPT_RT, accessed July 13, 2025, https://forums.developer.nvidia.com/t/observing-latency-discrepancies-on-nvidia-jetson-nano-with-preempt-rt/329935
23. PREEMPT_RT and regular kernel show similar latencies using cyclictest, accessed July 13, 2025, https://forums.developer.nvidia.com/t/preempt-rt-and-regular-kernel-show-similar-latencies-using-cyclictest/273435
24. RT Latency Tuning - Jetson Orin NX - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/rt-latency-tuning/253193
25. Real time kernel cycle test bad results - Jetson Nano - NVIDIA Developer Forums, accessed July 13, 2025, https://forums.developer.nvidia.com/t/real-time-kernel-cycle-test-bad-results/276685
26. Cyclictest under load on nvidia jetson agx RT-Kernel, accessed July 13, 2025, https://forums.developer.nvidia.com/t/cyclictest-under-load-on-nvidia-jetson-agx-rt-kernel/108111
27. PCIe Card - TTTECH, accessed July 13, 2025, https://www.tttech.com/sites/default/files/documents/TTTech_TTE-PCIe_Card-Flyer.pdf
28. TTE-Development System A664 for Linux - TTTECH, accessed July 13, 2025, https://www.tttech.com/aerospace/products/dev-test-systems/tte-development-system-a664-for-linux
29. TTE-Verify - TTTECH, accessed July 13, 2025, https://www.tttech.com/aerospace/products/software-tools/tte-verify
30. TTE-Network Interface Cards - TTTECH, accessed July 13, 2025, https://www.tttech.com/aerospace/products/end-systems/tte-network-interface-cards
31. APE-FDX-2 AFDX/ARINC664P7 Card for PCIe - AIM Online - AFDX, accessed July 13, 2025, https://www.aim-online.com/products/ape-fdx-2/
32. APE-FDX-2 - MB Electronique, accessed July 13, 2025, https://www.mbelectronique.fr/Web_PDF/1260_2020033009535780_aim-ds-ape-fdx-Specifications.pdf
33. AFDX / ARINC664P7 Modules with a Difference - AIM Online, accessed July 13, 2025, https://www.aim-online.com/products-overview/modules/afdx-arinc664p7/
34. AMEE429-x Rugged Embedded ARINC429 PCIe Mini Card, accessed July 13, 2025, https://www.aim-online.com/products/amee429-x/
35. ARINC 429 PCI CARD - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/discontinued-1/dd-42924i5-300
36. 24-channel ARINC 429 Interface - United Electronic Industries, accessed July 13, 2025, https://www.ueidaq.com/products/24-channel-arinc-429-interface
37. APXX429-x ARINC429 Card for PCI - AIM Online - 429, accessed July 13, 2025, https://www.aim-online.com/products/apxx429-x/
38. AMEE429-x | Aim Online, accessed July 13, 2025, https://www.aim-online.com/wp-content/uploads/2024/05/aim-ds-amee429-x-02-230901.pdf
39. MIL-STD-1553 - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/MIL-STD-1553
40. What is MIL-STD-1553? - Assured Systems, accessed July 13, 2025, https://www.assured-systems.com/faq/what-is-mil-std-1553/
41. M.2-1553-2 - ALPHI Technology, accessed July 13, 2025, https://www.alphitech.com/products/m-dot-2/m-2-1553-2/
42. ALPHI Technology, accessed July 13, 2025, https://www.alphitech.com/
43. MIL-STD-1553 - Products – ALPHI Technology, accessed July 13, 2025, https://www.alphitech.com/products/?_functions=mil-std-1553
44. MIL-STD-1553 - Avionics Interface Technologies - A Teradyne Company, accessed July 13, 2025, https://aviftech.com/ait_product_category/mil-std-1553/
45. MIL-STD-1553 Dual Channel Interface Board - United Electronic Industries, accessed July 13, 2025, https://www.ueidaq.com/products/dual-channel-mil-std-1553-interface-board
46. MIL-STD-1553 BC-RT-MT Integrated Terminal - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/connectivity/discontinued-3/bu-61559-bus-61559?partNumber=BUS-61559
47. NVIDIA ® Jetson Embedded Computing Solutions - Diamond Systems, accessed July 13, 2025, https://www.diamondsystems.com/products/nvidiasolutions.php
48. Custom Products and Solutions - Diamond Systems, accessed July 13, 2025, https://www.diamondsystems.com/solutions/custom-products-and-solutions
49. Custom Embedded Solutions - Diamond Systems, accessed July 13, 2025, https://www.diamondsystems.com/solutions/custom
50. NVIDIA Jetson Platform | Edge AI - ADLINK Technology, accessed July 13, 2025, https://www.adlinktech.com/en/nvidia-jetson-platform
51. Connect Tech Quark Carrier and NVIDIA Jetson Nano Module with Thermal Transfer Plate -25°C to +85°C - WDL Systems, accessed July 13, 2025, https://www.wdlsystems.com/CTI-Quark-Carrier-for-NVIDIA-Jetson-Xavier-NX-and-Nano.-40C-to-85C_2
52. Manufacturer: Connect-Tech, Processor: Carrier-Board-Only, NVIDIA-Jetson-Nano - WDL Systems, accessed July 13, 2025, https://www.wdlsystems.com/custitem_ff_manufacturer/Connect-Tech/custitem_ff_processor/Carrier-Board-Only,NVIDIA-Jetson-Nano
53. Jetson Ruggedized Systems - NVIDIA Developer, accessed July 13, 2025, https://developer.nvidia.com/embedded/jetson/jetson-ruggedized-systems
54. Discover Vector AI - The smart mid-range ISR sUAS - Quantum Systems, accessed July 13, 2025, https://quantum-systems.com/vector/


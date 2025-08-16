# 비행 제어 컴퓨터에 Xenomai와 리눅스의 적용에 대한 심층 고찰

## 서론: 현대 항공 전자의 딜레마 - 성능 대 인증

### 0.1 비행 제어 컴퓨터(FCC)의 진화: 기계적 연결에서 자율 비행의 두뇌로

현대 항공기의 핵심은 플라이 바이 와이어(Fly-By-Wire, FBW) 시스템이며, 그 중심에는 비행 제어 컴퓨터(Flight Control Computer, FCC)가 자리 잡고 있습니다. 과거의 항공기가 조종간과 조종면을 케이블과 도르래 같은 기계적 장치로 연결했던 것과 달리, FBW 시스템에서 FCC는 조종사의 조종 입력을 전기 신호로 변환하여 각 조종면에 전달하는 역할을 수행합니다.1 이는 단순히 기계적 연결을 전기적 신호로 대체하는 것을 넘어, 비행 제어의 패러다임을 근본적으로 바꾸었습니다. FCC는 조종사의 입력을 그대로 전달하는 것이 아니라, 비행 상태, 속도, 고도 등 수많은 데이터를 실시간으로 분석하여 항공기가 안정적인 비행 봉투(flight envelope) 내에서 최적의 기동을 할 수 있도록 제어 입력을 능동적으로 보정하고 안정화합니다.

최근 무인 항공기(UAV)와 도심 항공 모빌리티(UAM)의 부상으로 FCC의 역할은 더욱 확장되고 복잡해졌습니다. 단순한 비행 제어를 넘어 비행 관리 시스템(Flight Management System, FMS)과 통합되어 자율 경로 비행, 임무 수행, 실시간 고장 진단 및 대응, 심지어 머신러닝 기반의 지능형 비행 제어 알고리즘까지 수행해야 합니다.2 이러한 고도의 연산 능력 요구는 자연스럽게 더 강력하고 유연한 소프트웨어 플랫폼의 필요성으로 이어졌습니다. 또한, 항공 시스템의 안전은 절대적인 가치이므로, FCC는 극도의 신뢰성을 보장해야 합니다. 이를 위해 이중화(Dual FCC) 구성과 같은 다중화 아키텍처를 채택하고, 시스템의 정상 동작을 지속적으로 감시하는 와치독 타이머(watchdog timer)와 같은 매커니즘을 통해 한 시스템에 오류가 발생하더라도 예비 시스템으로 즉시 제어권을 전환하여 비행 안전을 유지합니다.5 이처럼 FCC는 단순한 신호 변환 장치에서 항공기의 모든 움직임을 관장하고 비상 상황에 대처하는 '두뇌'로 진화했으며, 이는 필연적으로 운영체제(OS) 수준에서의 근본적인 변화를 요구하게 되었습니다.

### 0.2 폐쇄적인 세계에서의 오픈 소스의 유혹

전통적으로 항공 전자(Avionics) 산업은 극도의 보수성과 폐쇄성을 특징으로 합니다. 안전과 직결되는 만큼, 수십 년간 검증된 독점(proprietary) 기술과 자체 개발한 실시간 운영체제(RTOS)를 사용하는 것이 표준처럼 여겨져 왔습니다. 이러한 시스템은 신뢰성은 높지만, 개발 비용이 막대하고 기술 발전 속도가 더디며 특정 공급업체에 종속되는(vendor lock-in) 단점을 안고 있습니다. 반면, 상용 기성품(Commercial-Off-The-Shelf, COTS) 및 오픈 소스 소프트웨어(Open-Source Software, OSS)의 세계, 특히 리눅스(Linux)는 정반대의 특성을 가집니다. 리눅스는 전 세계 수많은 개발자가 참여하는 거대한 생태계를 기반으로 빠른 속도로 발전하며, 방대한 하드웨어 지원, 풍부한 네트워킹 및 파일 시스템, 그래픽 사용자 인터페이스(GUI) 등 고기능성 라이브러리를 제공합니다.7 무엇보다 라이선스 비용이 없고 소스 코드가 공개되어 있어 투명성을 확보하고 자유롭게 수정할 수 있다는 점은 매력적인 장점입니다.10

이러한 배경에서 항공 전자 산업은 중대한 갈림길에 서 있습니다. 앞서 언급한 FCC의 기능적 복잡성 증가는 리눅스가 제공하는 강력한 기능과 개발 생태계를 필요로 하지만, 항공 시스템의 엄격한 실시간성과 안전 요구사항은 범용 운영체제(General-Purpose Operating System, GPOS)인 리눅스의 본질과 상충됩니다. 여기서 본 보고서의 핵심적인 연구 질문이 도출됩니다: 과연 범용 운영체제인 리눅스 위에 Xenomai와 같은 오픈 소스 실시간 프레임워크를 결합하여 FCC가 요구하는 경성 실시간(Hard Real-Time) 결정성(determinism)을 달성할 수 있는가? 그리고 더 중요하게는, 이러한 시스템이 상업용 항공기에 탑재되기 위해 필수적인 DO-178C와 같은 엄격한 항공 안전 인증 절차를 통과할 수 있는가?

### 0.3 보고서의 논지와 구조

본 보고서는 FCC 애플리케이션을 위한 Xenomai/리눅스 소프트웨어 스택에 대한 깊이 있고 비판적인 분석을 수행할 것입니다. 본 보고서의 핵심 논지는 다음과 같습니다: Xenomai와 리눅스의 조합은 UAV나 연구용 플랫폼과 같이 인증 요구가 없거나 낮은 수준의 시스템에서는 강력한 성능과 유연성을 제공하는 탁월한 선택지이지만, 높은 설계 보증 등급(Design Assurance Level, DAL)의 인증 항공 전자에 직접 적용하는 것은 리눅스의 본질적인 특성으로 인해 근본적으로 실현 불가능합니다. 따라서 이러한 시스템에서 리눅스의 강력한 기능을 활용하는 가장 현실적인 경로는 직접 인증이 아니라, 인증된 ARINC 653 파티셔닝 프레임워크 내에서 아키텍처적으로 격리하여 사용하는 것입니다.

이러한 논지를 증명하기 위해, 본 보고서는 다음과 같은 구조로 전개됩니다. 먼저 실시간 시스템의 기본 개념을 정립하여 FCC의 근본적인 요구사항을 정의합니다. 이후 Xenomai의 핵심인 듀얼 커널 아키텍처를 심층적으로 해부하고, 성능 분석을 통해 기술적 가능성과 한계를 명확히 합니다. 다음으로 항공 안전 인증의 가장 큰 장벽인 DO-178C의 요구사항과 리눅스의 근본적인 비호환성을 분석하고, 이에 대한 아키텍처적 해결책으로서 ARINC 653 파티셔닝을 제시합니다. 마지막으로, 모든 분석을 종합하여 전략적 결론과 향후 전망을 제시함으로써, 현대 항공 전자 시스템 개발의 방향성에 대한 통찰을 제공하고자 합니다.

이러한 분석의 흐름은 단순한 기술 비교를 넘어, 항공 전자 개발 패러다임의 근본적인 전환을 탐색하는 과정이 될 것입니다. 즉, 인증되었지만 기능이 제한된 레거시 시스템에 머무를 것인가, 아니면 강력하고 혁신적이지만 인증되지 않은 오픈 소스의 세계를 안전하게 통합할 방법을 찾을 것인가라는 전략적 질문에 대한 답을 모색하는 것입니다. FCC의 역할이 단순한 신호 변환기에서 복잡한 '두뇌'로 진화함에 따라, 이러한 질문에 대한 답을 찾는 것은 더 이상 선택이 아닌 필수가 되었습니다.3

## 1. 결정론의 초석: 실시간 시스템에 대한 심층 탐구

### 1.1 실시간의 정의: 단순한 속도를 넘어서

실시간(Real-Time) 시스템이라는 용어는 종종 '매우 빠른' 시스템과 혼동되지만, 그 본질은 속도가 아닌 예측 가능성(predictability)에 있습니다. 실시간 시스템에서 연산의 정확성은 논리적 결과뿐만 아니라 그 결과가 도출되는 시간에도 의존합니다.13 즉, 정해진 시간 제약(deadline) 내에 작업을 완료하는 것을 보장하는 것이 핵심입니다. 이러한 특성에 따라 실시간 시스템은 세 가지로 분류됩니다.13

- **경성 실시간 시스템 (Hard Real-Time System):** 마감 시한을 놓치는 것이 치명적인 시스템 오류로 간주되는 경우입니다. 예를 들어, 비행 제어 시스템에서 조종면을 특정 시간 내에 움직이지 못하면 항공기 제어 불능으로 이어질 수 있습니다. FCC는 바로 이 영역에 속합니다.
- **준경성 실시간 시스템 (Firm Real-Time System):** 마감 시한을 놓친 결과는 가치가 없어지지만, 시스템 전체의 실패로 이어지지는 않는 경우입니다. 예를 들어, 공장 자동화 라인의 로봇 팔이 특정 작업을 놓치면 해당 제품은 불량이 되지만, 시스템 전체가 멈추지는 않습니다.
- **연성 실시간 시스템 (Soft Real-Time System):** 마감 시한을 지키는 것이 바람직하지만, 놓치더라도 시스템 성능이 다소 저하될 뿐 치명적인 결과를 초래하지는 않는 경우입니다. 온라인 동영상 스트리밍에서 일부 프레임이 누락되는 경우가 이에 해당합니다.

### 1.2 실시간 운영체제(RTOS)의 핵심 원리

경성 실시간 요구사항을 충족시키기 위해 설계된 운영체제가 바로 실시간 운영체제(RTOS)입니다. RTOS는 일반 운영체제(GPOS)와는 다른 설계 철학을 가지며, 다음과 같은 핵심 원리를 통해 결정론적 동작을 보장합니다.15

- **우선순위 기반 선점형 스케줄링 (Priority-Based Preemptive Scheduling):** RTOS 스케줄러의 가장 중요한 임무는 언제나 준비된(ready) 태스크 중 가장 우선순위가 높은 태스크가 CPU를 점유하도록 보장하는 것입니다. 예를 들어, 센서 데이터를 읽는 높은 우선순위의 태스크는 로그를 기록하는 낮은 우선순위의 태스크를 즉시 중단시키고(선점하고) 실행될 수 있어야 합니다.15 이는 시스템의 가장 중요한 작업이 지연 없이 처리되도록 하는 핵심 매커니즘입니다.
- **제한된 인터럽트 지연 시간 (Bounded Interrupt Latency):** 하드웨어 인터럽트가 발생한 시점부터 해당 인터럽트를 처리하는 서비스 루틴(ISR)이 실행되기까지 걸리는 시간, 즉 인터럽트 지연 시간이 매우 짧고 예측 가능한 상한값(upper bound)을 가져야 합니다. GPOS에서는 이 시간이 가변적일 수 있지만, RTOS에서는 이를 최소화하고 결정론적으로 관리하여 외부 이벤트에 대한 신속하고 일관된 반응을 보장합니다.13
- **자원 관리 및 동기화 (Resource Management and Synchronization):** 여러 태스크가 메모리나 I/O 장치와 같은 공유 자원에 접근할 때, 예측 불가능한 지연이나 교착 상태(deadlock)가 발생하지 않도록 관리해야 합니다. RTOS는 세마포어(semaphore), 뮤텍스(mutex)와 같은 동기화 매커니즘을 제공하여 자원 접근을 제어합니다. 특히, 낮은 우선순위 태스크가 높은 우선순위 태스크가 필요로 하는 자원을 점유하여 높은 우선순위 태스크의 실행을 막는 '우선순위 역전(priority inversion)' 현상을 방지하는 정교한 매커니즘을 포함해야 합니다.15
- **메모리 관리 (Memory Management):** 경성 실시간 시스템에서 메모리 관리는 매우 중요합니다. 시스템은 재부팅 없이 장기간 안정적으로 동작해야 하므로, 할당된 메모리가 해제되지 않고 유실되는 메모리 누수(memory leak)는 절대로 허용되지 않습니다. 이러한 이유로, 실행 시간에 예측 불가능한 지연을 유발할 수 있는 동적 메모리 할당(dynamic memory allocation)은 기피되거나, 고정된 크기의 블록을 미리 할당해두는 등 매우 제한적인 방식으로 사용됩니다.18

이러한 RTOS의 원리들을 종합해 보면, 표준 리눅스와 같은 범용 운영체제(GPOS)가 왜 본질적으로 경성 실시간 작업에 부적합한지 명확해집니다. GPOS는 평균적인 처리량(throughput)과 사용자 간의 공평성(fairness)을 최적화하도록 설계되었습니다. 복잡한 스케줄러, 디스크 I/O를 위한 가상 메모리 시스템(페이징), 그리고 커널의 비선점 구간 등은 모두 최악의 경우 지연 시간(worst-case latency)을 예측 불가능하게 만드는 요소들입니다.19

FCC의 핵심 제어 루프는 센서 값을 읽고, 제어 법칙을 계산하며, 조종면을 구동하는 일련의 과정을 수십 밀리초(ms) 이내의 엄격하고 반복적인 마감 시한 내에 완료해야 합니다.5 이 루프에서 발생하는 아주 작은 예측 불가능한 지연(지터, jitter)이라도 제어 불안정성을 야기하고, 최악의 경우 항공기 제어 불능으로 이어질 수 있습니다. 따라서 문제는 리눅스가 '나쁜' 운영체제라는 것이 아니라, 그 설계 철학이 FCC의 핵심 요구사항과 근본적으로 상충된다는 점입니다. 이는 리눅스 기반 시스템에 경성 실시간 동작을 '강제'할 수 있는 특별한 매커니즘의 필요성을 명확하게 제시하며, 바로 이 지점에서 Xenomai의 듀얼 커널 아키텍처가 등장하게 됩니다.

## 2. 듀얼 커널 패러다임: Xenomai 아키텍처 해부

### 2.1 코발트 코어: 리눅스와 나란히 실행되는 실시간 전담 커널

Xenomai는 리눅스 커널과 협력하여 사용자 공간 애플리케이션에 경성 실시간 컴퓨팅 지원을 제공하는 소프트웨어 프레임워크입니다.20 Xenomai의 핵심 전략은 리눅스 자체를 실시간으로 만드는 것이 아니라, 리눅스와는 별개의 독립적인 실시간 커널을 함께 실행하는 것입니다. 이것이 바로 **듀얼 커널(Dual-Kernel)** 아키텍처이며, 이 실시간 전담 커널은 **코발트(Cobalt)** 라는 코드명으로 불립니다.22

이 아키텍처의 기본 원리는 매우 명확합니다. 코발트 마이크로커널(또는 "뉴클리어스")은 표준 리눅스 커널과 병렬로, 그리고 그보다 더 높은 우선순위에서 실행됩니다.13 코발트의 관점에서 보면, 거대한 리눅스 커널 전체는 단지 하나의 가장 낮은 우선순위를 가진 태스크에 불과합니다. 따라서 코발트는 실행해야 할 실시간 태스크가 없을 때만 리눅스 커널에게 CPU 제어권을 넘겨줍니다.22 만약 리눅스가 실행되는 도중에 더 높은 우선순위의 실시간 태스크가 실행 준비 상태가 되면, 코발트는 즉시 리눅스의 실행을 중단시키고(선점하고) 해당 실시간 태스크를 실행합니다. 이 구조는 리눅스 커널의 내부 동작(스케줄링, 인터럽트 처리 등)이 아무리 복잡하고 비결정적이더라도, 실시간 태스크의 실행에 영향을 미치지 못하도록 원천적으로 차단합니다. 이것이 Xenomai가 경성 실시간 결정성을 보장하는 가장 근본적인 원리입니다.

### 2.2 인터럽트 파이프라인(Adeos/I-pipe 및 Dovetail): 지연 시간의 문지기

듀얼 커널 모델이 성공적으로 작동하기 위한 핵심 기술은 하드웨어 인터럽트를 리눅스 커널보다 먼저 가로채는 능력에 있습니다. 이를 위해 저수준의 하드웨어 추상화 계층(Hardware Abstraction Layer, HAL)이 필요합니다.17

역사적으로 이 역할을 수행한 것은 **Adeos I-pipe (Interrupt Pipeline)** 입니다. Adeos는 하드웨어와 운영체제 커널들 사이에 위치하여 인터럽트 중재자 역할을 합니다. 이는 인터럽트 처리 '도메인(domain)'들의 파이프라인을 생성하는데, Xenomai(코발트 코어)가 이 파이프라인의 최상단에 위치합니다. 하드웨어 인터럽트가 발생하면, Adeos는 이를 먼저 Xenomai 도메인에 전달합니다. 만약 이 인터럽트가 실시간 처리가 필요한 것이라면, Xenomai는 이를 즉시 처리하여 결정론적인 지연 시간을 보장합니다. 만약 실시간 처리가 필요 없는 인터럽트라면, Xenomai는 이를 파이프라인의 다음 단계인 리눅스 도메인으로 그냥 통과시킵니다.19

최신 리눅스 커널과의 통합 및 유지보수성을 향상시키기 위해 I-pipe를 계승하여 발전한 것이 **도브테일(Dovetail)** 인터페이스입니다. 도브테일은 실시간 코어를 위해 높은 우선순위의 '대역 외(out-of-band)' 실행 단계를 커널에 도입합니다. 이로 인해 리눅스 커널의 관점에서는 모든 장치 인터럽트가 마치 마스킹할 수 없는 인터럽트(Non-Maskable Interrupt, NMI)처럼 동작하게 됩니다.27 즉, 리눅스 커널이 내부적으로 인터럽트를 비활성화(masking)하는 작업을 수행 중이더라도, 도브테일은 이를 무시하고 실시간 인터럽트를 즉시 처리할 수 있습니다. 이는 듀얼 커널 시스템의 반응성을 극대화하는 핵심적인 진화입니다.30

### 2.3 두 개의 세계: 실시간 태스크의 삶

Xenomai 환경에서 태스크는 두 가지 실행 컨텍스트 중 하나에 속하게 됩니다.25

- **주 모드 (Primary Mode):** 실시간 컨텍스트입니다. 이 모드에 있는 태스크는 Xenomai의 코발트 스케줄러에 의해 직접 관리되며, 우선순위 기반 선점형 스케줄링과 결정론적 동작을 보장받습니다.
- **부 모드 (Secondary Mode):** 비실시간 컨텍스트입니다. 이 모드에 있는 태스크는 표준 리눅스 스케줄러에 의해 관리되며, 일반적인 리눅스 프로세스와 동일하게 동작합니다.

모든 태스크는 초기에 부 모드에서 시작됩니다. 태스크가 Xenomai가 제공하는 고유의 실시간 시스템 콜(예: `rt_task_create()`, `rt_timer_spin()`)을 호출하면, 이 태스크는 주 모드로 전환되어 실시간 속성을 획득합니다. 하지만 여기서 매우 중요한 제약사항이 발생합니다. 만약 주 모드에서 실행 중인 태스크가 표준 리눅스 시스템 콜(예: 파일 입출력을 위한 `printf()`, `write()` 등)을 호출하면, 이 태스크는 그 즉시 강제로 부 모드로 전환됩니다. 그리고 해당 리눅스 시스템 콜이 완료될 때까지 모든 실시간 보증을 상실하게 됩니다.25 이 '모드 스위치(mode switch)'는 듀얼 커널 아키텍처의 가장 큰 특징이자 개발자가 반드시 이해해야 할 핵심 제약사항입니다. 이 경계를 잘못 관리하면 예측 불가능한 지연 시간이 발생하여 시스템의 실시간성을 해칠 수 있습니다.

### 2.4 Xenomai 4/EVL로의 진화: 복잡성 길들이기

이러한 '두 개의 세계' 문제와 I-pipe의 유지보수 어려움을 해결하기 위해 등장한 것이 Xenomai 4와 **EVL(Eventually-Less-Violent) 코어**입니다.28

Xenomai 4의 핵심은 앞서 언급한 **도브테일(Dovetail)** 인터페이스입니다. 도브테일의 주된 개발 동기는 오래된 I-pipe 코드를 새로운 리눅스 커널 버전에 맞게 이식하는 유지보수의 악몽을 해결하는 것이었습니다.28 EVL 코어는 이 도브테일 인터페이스를 최대한 활용하도록 처음부터 새롭게 설계된 더 단순하고 확장성 있는 실시간 코어입니다.31

특히 Xenomai 4는 실시간(주 모드)과 비실시간(부 모드) 세계 사이의 간극을 메우기 위한 중요한 추상화 개념들을 도입했습니다. 이는 개발자가 겪는 고통을 직접적으로 해결하려는 시도입니다.

- **크로스-버퍼 (Cross-buffer, xbuf):** 실시간(out-of-band) 스레드와 비실시간(in-band) 스레드 간에 데이터를 교환하기 위한 양방향, 잠금 없는(lock-free) 통신 채널입니다. 이를 통해 실시간 스레드는 비실시간 스레드와 데이터를 주고받으면서도 모드 전환으로 인한 성능 저하 없이 주 모드에 계속 머무를 수 있습니다.31
- **파일 프록시 (File proxy):** 실시간 스레드가 로그 파일 기록과 같은 비실시간 I/O 작업을 직접 수행하는 대신, 이 작업을 리눅스 도메인에서 실행되는 전담 워커(worker) 스레드에 위임하는 매커니즘입니다. 실시간 제어 루프 내에서 로그를 남기고 싶을 때, 파일 프록시를 사용하면 주 모드에서 부 모드로 강제 전환되는 것을 피할 수 있습니다.31

이러한 Xenomai 4의 진화는 듀얼 커널 아키텍처가 가진 근본적인 문제점을 인정한 결과입니다. `xbuf`와 파일 프록시는 단순히 지연 시간을 줄이는 기술이 아니라, 개발자가 두 세계의 경계를 더 안전하고 쉽게 넘나들 수 있도록 돕는, 즉 **개발자 편의성(developer ergonomics)**을 향상시키는 도구입니다. 따라서 Xenomai의 발전 역사는 듀얼 커널 아키텍처가 가진 날카로운 모서리를 계속해서 다듬어, 두 세계의 장점은 취하고 단점은 최소화하려는 지속적인 노력의 과정으로 볼 수 있습니다.

## 3. 성능, 장점, 그리고 내재된 한계

### 3.1 정량적 성능 분석: 지연 시간과 지터

Xenomai/리눅스 시스템의 실질적인 성능을 평가하기 위해서는 정량적인 데이터 분석이 필수적입니다. 이와 관련하여 가장 중요한 연구 중 하나는 Barbalace 등이 수행한 벤치마크 테스트입니다. 이 연구는 항공 전자 시스템에 널리 사용되는 PowerPC 아키텍처(MPC7455 프로세서) 기반의 VMEbus 보드에서 Xenomai, VxWorks, RTAI, 그리고 표준 리눅스 2.6의 성능을 직접 비교했습니다.19

연구 결과는 매우 명확한 시사점을 제공합니다. 아래 표는 해당 연구에서 측정한 인터럽트 처리 후 태스크 재스케줄링이 없는 경우와 있는 경우의 평균 지연 시간(mean delay)과 지터(jitter)를 요약한 것입니다. 지터는 지연 시간의 변동성을 나타내는 척도로, 그 값이 작을수록 시스템의 예측 가능성이 높다는 것을 의미합니다.

**표 3: RTOS 성능 벤치마크 요약 (지연 시간 및 지터)**

| 운영체제                                                     | 재스케줄링 없는 평균 지연 (μs) | 재스케줄링 없는 지터 (μs) | 재스케줄링 있는 평균 지연 (μs) | 재스케줄링 있는 지터 (μs) |      |
| ------------------------------------------------------------ | ------------------------------ | ------------------------- | ------------------------------ | ------------------------- | ---- |
| VxWorks                                                      | 2.5                            | 0.8                       | 6.5                            | 0.8                       |      |
| Linux 2.6                                                    | 12.0                           | 80.0                      | 25.0                           | 80.0                      |      |
| RTAI                                                         | 2.5                            | 0.5                       | 6.0                            | 0.6                       |      |
| Xenomai                                                      | 2.5                            | 0.6                       | 6.2                            | 0.7                       |      |
| 출처: Barbalace, A. et al. (2008). Performance Comparison of VxWorks, Linux, RTAI, and Xenomai in a Hard Real-time Application. 19 |                                |                           |                                |                           |      |

분석 결과, Xenomai와 RTAI는 업계 표준 상용 RTOS인 VxWorks와 대등한 수준의 성능을 보여주었습니다. 평균 지연 시간은 약 2.5μs로 매우 낮았고, 지터 역시 1μs 미만으로 극히 안정적이었습니다. 이는 듀얼 커널 방식이 경성 실시간 요구사항을 충족시킬 수 있는 기술적 역량을 갖추고 있음을 명백히 증명합니다. 반면, 표준 리눅스 2.6 커널은 평균 지연 시간도 높았지만, 무엇보다 지터가 80μs에 달해 예측 불가능성이 매우 크다는 것을 보여줍니다.19

또 다른 라즈베리 파이 3 기반의 연구 사례는 이러한 성능을 다른 관점에서 조명합니다. 해당 연구에서 Xenomai Cobalt 시스템의 최악 지연 시간은 82μs로 측정되었습니다. 이 값은 프로젝트가 요구했던 엄격한 경성 실시간 요구사항(±10μs)을 충족하지 못했습니다. 하지만 연구진은 대부분의 측정값은 훨씬 작은 편차를 보였기 때문에, 이 시스템이 치명적인 오류를 유발하지 않는 준경성(firm) 실시간 작업에는 충분히 적합하다고 결론 내렸습니다.33 이는 기술의 성능 자체만큼이나, 그 기술을 적용 대상의 요구사항 수준에 맞게 선택하는 것이 중요함을 시사합니다.

### 3.2 Xenomai/리눅스의 장점: 오픈 소스의 힘

Xenomai와 리눅스를 결합한 시스템은 여러 가지 강력한 장점을 제공합니다.

- **비용과 유연성:** 라이선스 비용이 없고 특정 공급업체에 종속되지 않는다는 점은 가장 큰 경제적 이점입니다. 이는 특히 대량 생산되는 드론이나 비용에 민감한 프로젝트에서 중요한 고려사항이 됩니다.11
- **생태계와 개발 민첩성:** 개발자는 애플리케이션의 비실시간(non-real-time) 부분을 개발할 때 리눅스가 제공하는 방대한 도구, 라이브러리, 드라이버 생태계를 그대로 활용할 수 있습니다. 예를 들어, 복잡한 네트워크 통신, 데이터 로깅을 위한 파일 시스템, 지상 관제 시스템(GCS)을 위한 그래픽 인터페이스 등을 손쉽게 구현할 수 있어 전체 개발 속도를 크게 향상시킬 수 있습니다.10
- **마이그레이션 경로:** Xenomai는 VxWorks, pSOS 등 기존 상용 RTOS의 API를 에뮬레이션하는 '스킨(skin)' 기능을 제공합니다. 이를 통해 기존에 작성된 레거시 코드를 리눅스 기반 환경으로 비교적 쉽게 이전할 수 있는 경로를 제공합니다.10
- **투명성:** 소스 코드가 모두 공개되어 있어 시스템의 내부 동작을 깊이 있게 이해하고, 필요에 따라 직접 수정하거나 보안 감사를 수행할 수 있습니다. 이는 독점적인 '블랙박스' 시스템에서는 불가능한 장점입니다.11

### 3.3 내재된 도전 과제: 성능의 대가

이러한 장점에도 불구하고, Xenomai/리눅스 접근 방식은 본질적인 한계와 도전 과제를 안고 있습니다.

- **아키텍처의 복잡성:** 듀얼 커널 모델은 개념적으로 이해하기 어렵습니다. 개발자는 항상 주 모드와 부 모드라는 두 개의 실행 컨텍스트를 인지하고, 그 사이를 오가는 엄격한 규칙을 준수해야 합니다. 이는 개발 과정에서 실수를 유발할 수 있는 잠재적 위험 요소입니다.25
- **드라이버 딜레마 (RTDM):** 실시간 태스크는 표준 리눅스 드라이버를 사용할 수 없습니다. 이는 리눅스 드라이버가 예측 불가능한 지연을 유발하는 잠금(lock)이나 동적 메모리 할당 등을 포함하고 있기 때문입니다. 따라서 개발자는 필요한 하드웨어에 대한 **실시간 드라이버 모델(Real-Time Driver Model, RTDM)** 기반의 드라이버를 직접 작성하거나 찾아야 합니다. 이는 리눅스의 '방대한 드라이버 지원'이라는 장점을 실시간 부분에서는 상당 부분 상쇄시키는 중대한 제약입니다.22
- **디버깅의 어려움:** Xenomai의 코발트 코어와 리눅스 커널의 경계에서 발생하는 문제를 디버깅하는 것은 매우 복잡할 수 있습니다. 문제의 원인이 실시간 코드에 있는지, 리눅스 커널과의 상호작용에 있는지, 혹은 하드웨어에 있는지 파악하기 어렵기 때문입니다.35
- **리눅스 태스크의 기아(Starvation) 가능성:** 실시간 태스크의 실행은 보장되지만, 만약 실시간 작업 부하가 너무 높으면 리눅스 커널과 그 위에서 동작하는 일반 애플리케이션들은 CPU 시간을 충분히 할당받지 못해 반응성이 떨어지거나 멈추는 '기아 현상'을 겪을 수 있습니다. 이는 비행에 치명적이지는 않지만 중요한 기능(예: 데이터 링크 통신)의 성능에 영향을 줄 수 있습니다.25

결론적으로, Xenomai의 듀얼 커널(코발트) 방식은 싱글 커널 방식인 PREEMPT_RT 리눅스보다 낮은 지연 시간과 지터를 제공하지만, 훨씬 가파른 학습 곡선과 제한적인 프로그래밍 모델을 감수해야 하는 명확한 트레이드오프가 존재합니다.10 성능 데이터는 Xenomai가 기술적으로 FCC의 경성 실시간 요구를 충족할 수 있음을 보여주지만 19, RTDM이나 모드 전환과 같은 실제적인 개발의 어려움은 구현 과정에서 오류가 발생할 위험을 높입니다. 따라서 Xenomai/리눅스 솔루션을 선택하는 것은 단순히 기술적 결정이 아니라, 개발팀의 역량, 성숙한 개발 프로세스, 그리고 철저한 검증 능력까지 고려해야 하는 복합적인 문제입니다. 리눅스 생태계의 장점은 양날의 검과 같습니다. 그 풍부한 기능을 활용할 수는 있지만, 매우 좁고 신중하게 제어된 인터페이스를 통해서만 접근해야 합니다.

## 4. 인증의 관문: DO-178C와 항공기 감항성 확보의 길

### 4.1 DO-178C의 이해: 항공 전자 소프트웨어의 황금률

항공기에 탑재되는 모든 상용 소프트웨어는 비행에 적합하다는 감항성(airworthiness)을 입증해야 하며, 이를 위해 미국 연방항공청(FAA), 유럽항공안전청(EASA)과 같은 인증 기관이 사용하는 핵심 표준이 바로 **DO-178C (Software Considerations in Airborne Systems and Equipment Certification)** 입니다.36

DO-178C는 특정 개발 방법론이나 도구를 강요하지 않는 '목표 기반(objective-based)' 표준입니다. 즉, 소프트웨어 개발 생명주기 동안 '무엇을' 달성해야 하는지에 대한 목표들을 정의하고, 그 목표들이 만족되었음을 입증하는 증거를 요구합니다.38 DO-178C의 핵심 원칙은 다음과 같습니다.39

- **엄격한 계획:** 소프트웨어 개발의 모든 측면을 다루는 포괄적인 계획 수립.
- **완벽한 요구사항 추적성:** 시스템의 상위 요구사항부터 설계, 소스 코드, 테스트 케이스에 이르기까지 모든 개발 산출물 간의 양방향 추적성 확보.
- **철저한 검증 및 확인:** 단위 테스트, 통합 테스트, 시스템 테스트를 포함한 포괄적인 검증 활동을 통해 오류를 탐지하고 제거.
- **강력한 형상 관리:** 모든 소프트웨어 관련 항목에 대한 변경 사항을 효과적으로 통제.

DO-178C의 가장 중요한 특징 중 하나는 소프트웨어의 고장이 항공기, 승무원, 승객에게 미치는 영향의 심각도에 따라 **설계 보증 등급(Design Assurance Levels, DALs)** 을 5단계로 나누는 것입니다. 등급이 높을수록(DAL A에 가까울수록) 충족해야 하는 목표의 수가 많아지고, 검증 과정의 독립성 요구사항도 엄격해집니다.

**표 2: DO-178C 설계 보증 등급(DALs)**

| DAL                                      | 고장 조건             | 항공기/승무원/승객에 미치는 영향                             | 목표 수 | 독립성 요구 목표 수 |      |
| ---------------------------------------- | --------------------- | ------------------------------------------------------------ | ------- | ------------------- | ---- |
| **Level A**                              | 치명적 (Catastrophic) | 항공기 손실 및 다수의 사망자를 유발할 수 있는 고장           | 71      | 30                  |      |
| **Level B**                              | 위험 (Hazardous)      | 안전 또는 성능에 중대한 악영향을 주거나 승무원의 작업 부하를 크게 증가시켜 심각한 부상이나 사망을 초래할 수 있는 고장 | 69      | 18                  |      |
| **Level C**                              | 주요 (Major)          | 안전 여유를 크게 감소시키거나 승무원 작업 부하를 현저히 증가시켜 승객의 불편이나 경미한 부상을 초래할 수 있는 고장 | 62      | 5                   |      |
| **Level D**                              | 경미 (Minor)          | 안전 여유를 약간 감소시키거나 승무원 작업 부하를 약간 증가시켜 승객의 불편을 초래할 수 있는 고장 | 26      | 2                   |      |
| **Level E**                              | 영향 없음 (No Effect) | 안전, 항공기 운용, 승무원 작업 부하에 아무런 영향을 미치지 않는 고장 | 0       | 0                   |      |
| 출처: Wind River (2022), RTCA/EUROCAE 36 |                       |                                                              |         |                     |      |

FCC와 같은 비행 제어 시스템은 일반적으로 가장 높은 등급인 DAL A 또는 DAL B로 분류됩니다. 이는 해당 소프트웨어에 대한 인증 과정이 극도로 엄격하고 비용이 많이 든다는 것을 의미합니다.

### 4.2 넘을 수 없는 벽: 표준 리눅스 인증이 불가능한 이유

현재의 리눅스 커널을 DO-178C의 높은 DAL 등급으로 인증받는 것은 현실적으로 불가능하며, 그 이유는 다음과 같습니다.

- **코드베이스의 규모와 복잡성:** 리눅스 커널은 수천만 라인의 코드로 구성되어 있습니다. DAL A에서 요구하는 수정 조건/결정 커버리지(MC/DC)와 같은 100%에 가까운 구조적 코드 커버리지를 이 방대한 코드베이스 전체에 대해 달성하는 것은 기술적으로나 재정적으로 거의 불가능합니다.41
- **요구사항의 부재:** DO-178C는 모든 코드가 검증 가능한 요구사항에 기반하여 작성될 것을 요구합니다. 그러나 리눅스는 전 세계 개발자 커뮤니티에 의해 유기적으로 개발되었으며, 이러한 공식적인 상위 요구사항 문서가 존재하지 않습니다. 따라서 요구사항 기반의 추적성을 입증할 수 없습니다.39
- **비결정성:** 앞서 논의했듯이, 표준 리눅스 커널의 설계에는 스케줄링, 메모리 관리 등에서 비롯되는 본질적인 비결정적 요소들이 포함되어 있습니다. 이는 안전 필수 시스템의 기본 원칙에 위배됩니다.43
- **불필요한 코드(Extraneous Code):** 인증된 시스템은 요구사항과 직접적으로 연결되지 않는 '죽은 코드(dead code)'나 '비활성화된 코드(deactivated code)'를 포함해서는 안 됩니다. 특정 FCC 애플리케이션의 관점에서 볼 때, 리눅스 커널의 대부분은 이러한 불필요한 코드에 해당하게 됩니다.41

### 4.3 아키텍처적 해결책: ARINC 653과 가상화를 통한 격리

이러한 '넘을 수 없는 벽'에 대한 해결책은 리눅스 자체를 인증하려는 시도를 포기하고, 대신 아키텍처를 통해 문제를 해결하는 것입니다. 여기서 핵심적인 역할을 하는 것이 **통합 모듈형 항공 전자(Integrated Modular Avionics, IMA)** 개념과 이를 지원하는 **ARINC 653** 표준입니다.

IMA는 과거에 기능별로 존재했던 수많은 전용 컴퓨터들을 하나의 고성능 공유 컴퓨팅 플랫폼으로 통합하는 아키텍처입니다.7 ARINC 653은 이러한 IMA 환경을 위한 표준 API 및 운영체제 인터페이스를 정의합니다. 그 핵심 원리는 **강력한 파티셔닝(Robust Partitioning)**입니다. 즉, 하나의 CPU 위에서 여러 애플리케이션(파티션)을 실행하되, 각 파티션이 서로에게 전혀 영향을 미칠 수 없도록 완벽하게 격리하는 것입니다.46 이 격리는 두 가지 차원에서 이루어집니다.

- **공간 분할 (Space Partitioning):** 각 파티션에 고정된 메모리 영역을 할당하고, 한 파티션이 다른 파티션의 메모리 공간에 접근하는 것을 하드웨어 메모리 관리 장치(MMU)를 통해 원천적으로 차단합니다.
- **시간 분할 (Time Partitioning):** 전체 CPU 시간을 고정된 주기의 '주요 시간 프레임(Major Time Frame)'으로 나누고, 각 파티션에 이 프레임 내에서 고정된 실행 시간 창(time window)을 할당합니다. 한 파티션은 할당된 시간 창 내에서만 실행될 수 있으며, 시간이 다 되면 강제로 실행이 중단되고 다음 파티션으로 제어권이 넘어갑니다.

이러한 파티셔닝 아키텍처는 리눅스를 안전 필수 시스템에 통합할 수 있는 매우 현실적인 전략을 제공합니다. 즉, 인증된 ARINC 653 호환 RTOS나 하이퍼바이저를 시스템의 기반으로 삼고, 그 위에 여러 파티션을 생성합니다. 비행 제어와 같이 안전이 중요한 DAL A 등급의 기능은 작고 검증 가능한 코드로 작성되어 하나의 파티션에서 실행됩니다. 그리고 복잡한 그래픽 디스플레이나 네트워크 통신과 같은 비핵심적 기능들은 인증되지 않은 표준 리눅스를 게스트 OS로 하여 다른 파티션에서 실행하는 것입니다.45 이 구조에서는 리눅스 파티션에서 오류가 발생하거나 시스템이 멈추더라도, 시간 및 공간 분할 매커니즘에 의해 비행 제어 파티션에는 아무런 영향을 미치지 않습니다.

### 4.4 인증 플랫폼 비교 분석

이러한 파티셔닝 전략은 여러 상용 RTOS 공급업체들의 핵심 비즈니스 모델이 되었습니다. 이들은 DO-178C 인증 증거 자료(certification artifacts)와 함께 인증된 파티셔닝 플랫폼을 제공합니다.

- **Wind River VxWorks 653:** 시장의 선두주자 중 하나로, DO-178C 인증을 획득한 ARINC 653 준수 플랫폼을 제공하며, 리눅스를 포함한 다양한 게스트 OS를 호스팅할 수 있습니다.47
- **SYSGO PikeOS:** 분리 커널(separation kernel) 기반의 하이퍼바이저로, 안전성과 보안에 중점을 두고 설계되었습니다. ARINC 653을 완벽하게 지원하며, 리눅스를 게스트 OS로 파티션 내에서 실행할 수 있습니다. DO-178C DAL A 인증이 가능합니다.45
- **LynxOS-178 & LynxSecure:** LynxOS-178은 DO-178C 인증을 받은 RTOS로, ARINC 653을 네이티브로 지원합니다. POSIX 표준을 준수하여 리눅스 개발자에게 친숙한 환경을 제공합니다. LynxSecure는 이들의 분리 커널/하이퍼바이저 제품입니다.50
- **DDC-I Deos:** 1998년부터 DO-178C DAL A 인증을 받아온 오랜 비행 기록을 가진 RTOS입니다. 시간 및 공간 분할을 기반으로 하며, ARINC 653 및 POSIX 인터페이스를 지원하여 다양한 애플리케이션을 통합할 수 있습니다.48

결론적으로, '항공 분야에서의 리눅스'에 대한 논의는 사실상 '항공 분야에서의 파티셔닝'에 대한 논의와 같습니다. ARINC 653 표준은 항공 산업이 리눅스와 같은 GPOS의 풍부한 기능을 안전하게 활용할 수 있도록 하는 핵심적인 기술입니다. 따라서 Xenomai/리눅스 솔루션이 높은 DAL 등급의 인증 시장에서 의미를 가지려면, 이러한 파티셔닝 프레임워크 내에 통합되는 형태로 발전해야 합니다. 예를 들어, 인증된 하이퍼바이저 위에서 하나의 파티션은 Xenomai/코발트가 경성 실시간 작업을 처리하고, 다른 파티션은 표준 리눅스가 GPOS 작업을 처리하는 아키텍처는 기술적으로 매우 강력하고 현실적인 감항성 확보 경로가 될 수 있습니다.

**표 1: 실시간 운영체제 아키텍처 비교**

| 특징                 | Xenomai (코발트/듀얼 커널)                 | PREEMPT_RT 리눅스 (싱글 커널)                       | 상용 RTOS (VxWorks, PikeOS 등)                         |
| -------------------- | ------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------ |
| **아키텍처 유형**    | 듀얼 커널 (마이크로커널 + GPOS)            | 단일 커널 (수정된 모놀리식 커널)                    | 마이크로커널 또는 모놀리식 RTOS, 파티셔닝 하이퍼바이저 |
| **실시간 보증**      | 경성 실시간 (Hard Real-Time)               | 연성/준경성 실시간 (Soft/Firm)                      | 경성 실시간 (Hard Real-Time)                           |
| **일반적 지연/지터** | 매우 낮음, 결정론적 19                     | 낮지만 듀얼 커널보다 높고 비결정적 요소 존재 10     | 매우 낮음, 결정론적, 보증됨 49                         |
| **개발 모델**        | 복잡함 (주/부 모드 분리, 전용 API 필요) 26 | 상대적으로 단순함 (표준 POSIX API 사용) 10          | 공급업체별 API, ARINC 653 등 표준 API 지원             |
| **드라이버 모델**    | 전용 RTDM 필요 22                          | 표준 리눅스 드라이버 사용 가능 (단, 지연 유발 가능) | 전용 드라이버, 인증된 드라이버 제공                    |
| **DO-178C 경로**     | 직접 인증 거의 불가능, 파티션 내 사용 가능 | 직접 인증 불가능                                    | 인증 패키지(Artifacts) 제공, DAL A까지 가능 45         |
| **핵심 장점**        | 오픈 소스, 고성능, 리눅스 생태계 활용      | 메인라인 커널 통합, 개발 용이성                     | 인증 용이성, 신뢰성, 기술 지원, 생태계                 |
| **핵심 단점**        | 복잡성, RTDM 드라이버 부재, 인증 어려움    | 경성 실시간 보증 불가, 듀얼 커널 대비 높은 지연     | 비용, 공급업체 종속성, 제한된 유연성                   |

## 5. 종합 및 전략적 제언

### 5.1 분석 결과 종합: 두 가지 활용 사례의 이야기

본 보고서의 심층 분석을 통해 Xenomai와 리눅스의 조합은 두 가지 뚜렷한 측면을 가지고 있음이 명확해졌습니다. 한편으로, 정량적 성능 데이터는 이 조합이 지연 시간과 지터 측면에서 상용 독점 RTOS에 필적하는 강력한 경성 실시간 성능을 제공할 수 있음을 보여줍니다.19 이는 기술적으로 비행 제어와 같은 까다로운 작업을 수행할 잠재력이 충분함을 의미합니다.

다른 한편으로, 이러한 성능을 이끌어내는 듀얼 커널 아키텍처는 본질적으로 복잡하며, 개발자에게 엄격한 프로그래밍 규율을 요구합니다.25 더 근본적인 문제는 기반이 되는 리눅스 운영체제가 그 방대한 규모와 비결정적인 특성으로 인해 높은 DAL 등급의 DO-178C 인증을 통과하는 것이 사실상 불가능하다는 점입니다.42

이러한 분석 결과는 Xenomai/리눅스 스택의 적용 가능성이 시스템의 '인증 요구사항'에 따라 명확하게 두 갈래로 나뉜다는 결론으로 이어집니다.

### 5.2 제언 1: 비인증 및 저수준 DAL 시스템을 위한 최적의 플랫폼

DO-178C와 같은 엄격한 항공 안전 인증이 요구되지 않는 분야, 즉 실험용 항공기, 학술 연구용 플랫폼, 그리고 상업용 및 소비자용 드론(UAV) 시장의 대부분에서는 Xenomai/리눅스가 매우 탁월한 선택지가 될 수 있습니다.4

이러한 시스템에서 Xenomai/리눅스는 안정적인 비행 제어에 필요한 경성 실시간 성능을 저비용으로 제공하면서도, 리눅스 생태계가 가진 무한한 유연성을 통해 고도의 부가 기능을 개발할 수 있는 길을 열어줍니다. 예를 들어, 컴퓨터 비전 기반의 장애물 회피, 인공지능을 활용한 자율 항법, 복잡한 데이터 링크 및 지상국 통신 프로토콜 등은 표준 리눅스가 제공하는 풍부한 라이브러리와 개발 도구를 활용하여 훨씬 빠르고 효율적으로 구현할 수 있습니다.3 라즈베리 파이나 로보틱스 프레임워크를 활용한 다양한 연구 사례들은 이미 이 분야에서 Xenomai/리눅스의 실용성과 잠재력을 입증하고 있습니다.33

### 5.3 제언 2: 높은 DAL 등급 인증 시스템을 위한 게스트 OS 전략

반면, FCC와 같이 DO-178C DAL C 등급 이상, 특히 DAL A나 B 등급의 인증이 필수적인 상업용 항공 전자 시스템의 경우, Xenomai/리눅스만을 기반으로 시스템을 구축하고 인증을 시도하는 것은 전략적으로 매우 비합리적이며 실패할 가능성이 높습니다.

이러한 시스템을 위한 가장 현실적이고 강력하게 권장되는 전략은 **통합 모듈형 항공 전자(IMA)** 아키텍처를 채택하고, **인증된 ARINC 653 파티셔닝 하이퍼바이저**를 시스템의 기반으로 사용하는 것입니다. PikeOS, LynxSecure, Deos, VxWorks 653과 같은 상용 솔루션이 이러한 역할을 수행합니다.45

이 모델에서는 리눅스가 비인증 게스트 운영체제로서 중요도가 낮은 파티션 내에서 실행됩니다. 이 파티션은 조종사를 위한 고해상도 지도 표시, 데이터 통신, 전자 비행 가방(EFB)과 같은 풍부한 애플리케이션 환경을 제공하는 역할을 합니다. 반면, 실제 비행 제어 로직과 같은 안전에 치명적인 기능은 완전히 분리된 다른 파티션 내에서 작고 검증 가능한 코드로 실행됩니다. 이 접근 방식은 리눅스의 풍부한 기능과 인증된 파티셔닝 커널의 안전 보증이라는 두 세계의 장점을 모두 취하는 최적의 전략입니다.

### 5.4 향후 전망: 간극을 메우기 위한 노력

항공 전자 소프트웨어의 미래는 오픈 소스의 혁신과 인증된 안전 필수 코어가 강력한 파티셔닝 기술을 통해 공존하는 이기종(heterogeneous) 시스템으로 나아갈 가능성이 높습니다. 이러한 흐름 속에서 몇 가지 주목할 만한 움직임이 있습니다.

리눅스 재단의 **ELISA(Enabling Linux in Safety Applications) 프로젝트**는 리눅스 기반의 안전 필수 시스템을 구축하고 인증하는 데 도움이 되는 프로세스와 도구를 만드는 것을 목표로 합니다. 이것이 당장 DAL A 인증을 위한 만병통치약은 아니지만, 더 넓은 범위의 안전 관련 애플리케이션에서 리눅스의 '인증 가능성'을 높이려는 산업계의 공동 노력을 보여준다는 점에서 의미가 큽니다.60

또한, Xenomai 4/EVL 프로젝트의 지속적인 진화는 듀얼 커널 시스템의 개발 복잡성을 낮추려는 노력을 보여줍니다. 이는 개발의 진입 장벽을 낮추고 구현 오류의 위험을 줄여, 더 많은 시스템에서 이 기술이 채택될 수 있는 기반을 마련할 수 있습니다.28

궁극적으로, 미래 항공 전자 시스템에 대한 질문은 "리눅스가 조종석에 들어갈 수 있는가?"가 아니라, "**어떻게** 리눅스를 안전하게 격리하여 포함시킬 것인가?"가 될 것입니다. Xenomai와 리눅스는 그 자체로 완벽한 해답은 아닐지라도, 이러한 미래를 향한 중요한 기술적 구성 요소이자, 오픈 소스와 안전 필수 시스템의 융합이라는 거대한 패러다임 전환을 이끄는 핵심 동력으로 계속해서 중요한 역할을 수행할 것입니다.

#### **참고 자료**

1. 항공기 전기식 비행제어장치(FBW) 개발 기술동향 - BEMIL 군사세계, accessed July 6, 2025, https://bemil.chosun.com/nbrd/bbs/view.html?b_bbs_id=10040&pn=0&num=48288
2. 2. 드론의 구성, 주요 용어 및 통신기술, accessed July 6, 2025, https://jwprogramming.tistory.com/283
3. Heterogeneous Flight Management System (FMS) Design for Unmanned Aerial Vehicles (UAVs): Current Stages, Challenges, and Opportunities - MDPI, accessed July 6, 2025, https://www.mdpi.com/2504-446X/7/6/380
4. A Review on the State of the Art in Copter Drones and Flight Control Systems - MDPI, accessed July 6, 2025, https://www.mdpi.com/1424-8220/24/11/3349
5. Korean Institute of Information Technology, accessed July 6, 2025, https://ki-it.com/_common/do.php?a=full&b=12&bidx=2472&aidx=28059
6. 드론의 안전성 향상을 위한 효율적인 비행제어장치의 설계 및 구현, accessed July 6, 2025, https://ki-it.com/xml/28059/28059.pdf
7. Kernel-level ARINC 653 partitioning for Linux | Request PDF - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/254006616_Kernel-level_ARINC_653_partitioning_for_Linux
8. 5 Military Aerospace Software Trends That Can Benefit From DO-178C - Perforce, accessed July 6, 2025, https://www.perforce.com/blog/sca/do-178c-military-aerospace-software
9. The Real-Time Linux Kernel: A Survey on PREEMPT_RT, accessed July 6, 2025, https://re.public.polimi.it/retrieve/e0c31c12-9844-4599-e053-1705fe0aef77/11311-1076057_Reghenzani.pdf
10. Real Time Linux Comparison | ICterra Information and Communication Technologies, accessed July 6, 2025, https://www.icterra.com/real-time-linux-comparison/
11. Preliminary assessment of Linux for safety related systems - OSADL, accessed July 6, 2025, https://www.osadl.org/fileadmin/dam/documents-external/rr011.pdf
12. Heterogeneous Flight Management System (FMS) Design for Unmanned Aerial Vehicles (UAVs) - Semantic Scholar, accessed July 6, 2025, https://pdfs.semanticscholar.org/1940/9c1dea6002e42f6bc2b0f1c3044cd3dc99c7.pdf
13. What is Xenomai? - DEV Community, accessed July 6, 2025, https://dev.to/khozaei/what-is-xenomai-4db
14. 실시간 운영체제 (RTOS) - 도리의 디지털라이프, accessed July 6, 2025, [https://blog.skby.net/%EC%8B%A4%EC%8B%9C%EA%B0%84-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-rtos/](https://blog.skby.net/실시간-운영체제-rtos/)
15. 실시간 운영 체제(RTOS)란 무엇인가요? - IBM, accessed July 6, 2025, https://www.ibm.com/kr-ko/think/topics/real-time-operating-system
16. 운영체제 - 위키백과, 우리 모두의 백과사전, accessed July 6, 2025, [https://ko.wikipedia.org/wiki/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C](https://ko.wikipedia.org/wiki/운영체제)
17. Real Time Linux and Xenomai - First Technology Transfer, accessed July 6, 2025, https://ftt.co.uk/Xenomai_Real_Time_Linux.php
18. 실시간 운영체제 - 위키백과, 우리 모두의 백과사전, accessed July 6, 2025, [https://ko.wikipedia.org/wiki/%EC%8B%A4%EC%8B%9C%EA%B0%84_%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C](https://ko.wikipedia.org/wiki/실시간_운영체제)
19. (PDF) Performance Comparison of VxWorks, Linux, RTAI and ..., accessed July 6, 2025, https://www.researchgate.net/publication/3151063_Performance_Comparison_of_VxWorks_Linux_RTAI_and_Xenomai_in_a_Hard_Real-time_Application
20. en.wikipedia.org, accessed July 6, 2025, [https://en.wikipedia.org/wiki/Xenomai#:~:text=Xenomai%20is%20a%20software%20framework,integrated%20into%20the%20Linux%20environment.](https://en.wikipedia.org/wiki/Xenomai#:~:text=Xenomai is a software framework,integrated into the Linux environment.)
21. Xenomai - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/Xenomai
22. Xenomai - Product wiki, accessed July 6, 2025, https://wiki.emacinc.com/wiki/Xenomai
23. Overview :: Xenomai 3, accessed July 6, 2025, https://v3.xenomai.org/overview/
24. Mastering Xenomai for Real-Time Embedded Systems - Number Analytics, accessed July 6, 2025, https://www.numberanalytics.com/blog/mastering-xenomai-real-time-embedded-systems
25. Real-Time Linux and the Xenomai system - ReTiS Lab - Scuola ..., accessed July 6, 2025, http://retis.sssup.it/~lipari/courses/str07/xenomai-handout.pdf
26. Xenomai · EmbeddedSystem - hugh712, accessed July 6, 2025, https://hugh712.gitbooks.io/embeddedsystem/content/xenomai.html
27. Kernel API - Xenomai 4, accessed July 6, 2025, https://v4.xenomai.org/dovetail/kernel-api/
28. Overview - Xenomai 4, accessed July 6, 2025, https://evlproject.org/overview/
29. Dovetail - Xenomai 4, accessed July 6, 2025, https://evlproject.org/dovetail/
30. Overview :: Xenomai 4, accessed July 6, 2025, https://v4.xenomai.org/overview/
31. The EVL core :: Xenomai 4, accessed July 6, 2025, https://v4.xenomai.org/core/
32. Xenomai 4 development process, accessed July 6, 2025, https://v4.xenomai.org/devprocess/
33. Real-Time Linux Testbench on Raspberry Pi 3 using ... - DiVA portal, accessed July 6, 2025, https://www.diva-portal.org/smash/get/diva2:1251188/FULLTEXT01.pdf
34. Linux and the Spacecraft Flight Software Environment - DigitalCommons@USU, accessed July 6, 2025, [https://digitalcommons.usu.edu/cgi/viewcontent.cgi?referer=&httpsredir=1&article=1520&context=smallsat](https://digitalcommons.usu.edu/cgi/viewcontent.cgi?referer&httpsredir=1&article=1520&context=smallsat)
35. Xenomai 2 architecture, accessed July 6, 2025, https://v3.xenomai.org/introduction/
36. What Is DO-178C? | Wind River, accessed July 6, 2025, https://www.windriver.com/solutions/learning/do-178c
37. DO 178C Tutorial - Tonex Training, accessed July 6, 2025, https://www.tonex.com/do-178c-tutorial/
38. An introduction to DO-178C | Ansys Knowledge, accessed July 6, 2025, https://innovationspace.ansys.com/knowledge/forums/topic/an-introduction-to-do-178c/
39. Top DO-178C Program Mistakes to Avoid: Ensuring Airborne Software Safety, accessed July 6, 2025, https://www.psware.com/top-do-178c-program-mistakes-to-avoid-ensuring-airborne-software-safety/
40. Differences and Challenges between DO-178B and DO-178C - Visure Solutions, accessed July 6, 2025, https://visuresolutions.com/aerospace-and-defense/do-178b-vs-do-178c/
41. DO-178C demystified: Strategies for efficient certification - LDRA, accessed July 6, 2025, https://ldra.com/wp-content/uploads/ldra/DO-178C-Technical-Briefing-v2.1.pdf
42. Avionics Software Certification and Regulation - GitHub Pages, accessed July 6, 2025, https://umm-csci.github.io/senior-seminar/seminars/spring2019/debates.pdf
43. DO-178 continues to adapt to emerging digital technologies - Emma Helfrich, accessed July 6, 2025, https://emmahelfrich.com/works-1/do-178-continues-to-adapt-to-emerging-digital-technologies
44. A Model-Based Optimization Method of ARINC 653 Multicore Partition Scheduling - MDPI, accessed July 6, 2025, https://www.mdpi.com/2226-4310/11/11/915
45. ARINC 653 | SYSGO, accessed July 6, 2025, https://www.sysgo.com/arinc-653
46. ARINC 653 - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/ARINC_653
47. What Are ARINC 653–Compliant Safety-Critical Applications? - Wind River Systems, accessed July 6, 2025, https://www.windriver.com/solutions/learning/arinc-653-compliant-safety-critical-applications
48. Industry Standards | MOSA DO-178C FAA FACE EASA ARINC 653 - DDC-I, accessed July 6, 2025, https://www.ddci.com/resources/industry-standards/
49. PikeOS vs. VxWorks Comparison - SourceForge, accessed July 6, 2025, https://sourceforge.net/software/compare/PikeOS-vs-VxWorks/
50. DO-178C Certification Tools for Safety-Critical Avionics and Beyond, accessed July 6, 2025, https://www.lynx.com/challenges/safety-certification
51. IMPORTANCE OF SYSTEM ARCHITECTURE AND PLATFORM CHOICE ON SAFETY CERTIFICATION - Lynx Software Technologies, accessed July 6, 2025, https://www.lynx.com/embedded-systems-learning-center/cost-reducing-rtos-safety-certs
52. Resources - FAQ - Lynx Software Technologies, accessed July 6, 2025, https://www.lynx.com/resources-faq
53. LynxOS-178C POSIX Real Time Operating System | More Info - Lynx Software Technologies, accessed July 6, 2025, https://www.lynx.com/products/lynxos-178-do-178c-certified-native-posix-partitioned-rtos-more-info
54. Delivering Certified, Safety-Critical Computing for the Future of Aviation - Intel, accessed July 6, 2025, https://www.intel.co.id/content/dam/www/central-libraries/us/en/documents/ddci-deos-rtos-solution-brief.pdf
55. DDC-I, Inc. - LDRA, accessed July 6, 2025, https://ldra.com/technology-partners/ddc-i-inc/
56. Deos - Edaway, accessed July 6, 2025, https://www.edaway.com/product/deos/
57. 비행제어컴퓨터 - 더피치, accessed July 6, 2025, https://thepeach.kr/fcc/
58. A Reconfigurable, Linux-Based, Flight Control System for Small UAVs - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/268570394_A_Reconfigurable_Linux-Based_Flight_Control_System_for_Small_UAVs
59. ROS2 – XENOMAI4 REAL-TIME FRAMEWORK ON RASPBERRY PI I. (Ilyas) Raoudi - University of Twente Student Theses, accessed July 6, 2025, http://essay.utwente.nl/104808/1/Raoudi_MA_EEMCS.pdf
60. maemalynn - ELISA Project, accessed July 6, 2025, https://elisa.tech/author/maemalynn/




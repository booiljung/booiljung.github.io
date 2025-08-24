[리눅스](./index.md)
# ARM SoC에서 실시간 운영체제와 리눅스의 동시 적용에 대한 고찰



현대 임베디드 시스템 설계의 패러다임은 근본적인 변화를 겪고 있다. 과거에는 각 기능이 독립적인 전자 제어 장치(Electronic Control Unit, ECU)에서 실행되는 분산형(federated) 아키텍처가 일반적이었으나 1, 기술의 발전과 시장의 요구는 여러 기능을 단일 고성능 시스템 온 칩(System-on-Chip, SoC)에 통합하는 방향으로 나아가고 있다. 이러한 통합의 중심에는 '혼합 임계 시스템(Mixed-Criticality System, MCS)'이라는 개념이 자리 잡고 있다.


혼합 임계 시스템은 단일 컴퓨팅 플랫폼 내에서 안전-필수(safety-critical) 기능과 비-안전-필수(non-safety-critical) 기능처럼 서로 다른 중요도(criticality)를 가진 여러 애플리케이션을 동시에 실행하는 시스템으로 정의된다.3 여기서 '임계성(criticality)'은 시스템 구성 요소의 고장에 대비하여 요구되는 보증 수준을 의미하며, 높은 임계성을 가진 애플리케이션일수록 설계 및 검증에 더 많은 비용과 노력이 투입된다.5 예를 들어, 자동차의 브레이크 제어 시스템은 최고 수준의 안전 임계성을 갖는 반면, 인포테인먼트 시스템은 상대적으로 낮은 임계성을 가진다.4


이러한 이질적인 기능들을 단일 SoC로 통합하려는 움직임의 가장 큰 동인은 크기, 무게, 전력 및 비용(Size, Weight, Power, and Cost, SWaP-C)의 절감이다.1 과거의 분산형 아키텍처는 차량 내 ECU의 수를 50개 이상으로 증가시켜 배선 복잡성과 전체 시스템 비용을 증대시키는 문제를 야기했다.6 고성능 SoC 하나에 여러 ECU의 기능을 통합함으로써 물리적인 부품 수와 배선을 줄이고, 전력 소비를 최적화하며, 전체적인 시스템 비용을 절감할 수 있다. 이러한 통합은 자동차뿐만 아니라 항공, 산업 자동화 등 다양한 분야에서 필수적인 요구사항으로 자리 잡고 있다.4

이러한 통합의 흐름은 단순히 경제적 효율성을 넘어, 기술적 필연성의 결과이기도 하다. 무어의 법칙에 따라 SoC의 성능이 기하급수적으로 향상되면서, 과거에는 상상할 수 없었던 복잡한 범용 운영체제(General-Purpose Operating System, GPOS) 기반의 기능(예: 고해상도 그래픽, 인공지능 기반 음성인식)을 임베디드 환경에서 구현할 수 있게 되었다.7 동시에 안전 관련 기능의 수도 증가하면서, 이 두 가지 상이한 컴퓨팅 세계를 단일 실리콘 위에 공존시켜야 하는 강력한 동기가 형성되었다. 이 과정에서 발생하는 본질적인 아키텍처의 긴장, 즉 고성능 GPOS의 비결정성과 잠재적 오류가 안전 필수 기능의 신뢰성을 훼손하지 않도록 보장해야 하는 과제가 바로 이 보고서에서 다루는 모든 기술의 근본적인 출발점이다.


MCS의 핵심 과제는 '간섭으로부터의 자유(freedom from interference)'를 보장하는 것이다.9 이는 ISO 26262와 같은 기능 안전 표준의 핵심 요구사항으로, 낮은 임계성 구성 요소의 결함이 높은 임계성 구성 요소의 동작에 영향을 미쳐서는 안 된다는 원칙이다.3 이 원칙은 두 가지 주요 기술적 요구사항으로 구체화된다.

- **공간적 격리(Spatial Isolation):** 한 애플리케이션이 다른 애플리케이션의 메모리 공간을 침범하거나 손상시키는 것을 방지하는 것이다. 이는 주로 하드웨어 메모리 관리 장치(Memory Management Unit, MMU)나 메모리 보호 장치(Memory Protection Unit, MPU)를 통해 구현된다.10 각 애플리케이션에 독립적인 주소 공간을 할당하고 하드웨어적으로 접근을 통제함으로써 강력한 격리를 달성할 수 있다.
- **시간적 격리(Temporal Isolation):** 낮은 임계성 태스크가 CPU 시간을 과도하게 점유하거나 공유 자원을 무기한 점유하여 높은 임계성 태스크가 정해진 시간 내에 작업을 완료하지 못하는(deadline miss) 상황을 방지하는 것이다.3 이는 MCS 스케줄링 연구의 핵심 주제이며, 시스템의 실시간성을 보장하는 데 결정적인 역할을 한다.5

이 두 가지 격리 요건을 충족시키기 위해 다양한 아키텍처와 기술이 개발되었으며, 이는 본 보고서의 후반부에서 심도 있게 다루어질 것이다.






혼합 임계 시스템의 핵심적인 기술적 난제는 본질적으로 다른 두 가지 철학을 가진 운영체제, 즉 실시간 운영체제(Real-Time Operating System, RTOS)와 범용 운영체제(General-Purpose Operating System, GPOS)를 하나의 플랫폼에서 조화롭게 공존시켜야 한다는 점에서 비롯된다. 두 운영체제의 근본적인 차이를 이해하는 것은 MCS 아키텍처를 설계하는 첫걸음이다.






- **핵심 철학:** RTOS의 최우선 목표는 높은 처리율(throughput)이 아니라, 정해진 시간 제약 내에서 예측 가능한 응답 시간을 보장하는 것이다.16 RTOS에서 연산의 정확성은 논리적 결과뿐만 아니라 그 결과가 도출되는 시간까지 포함한다.16
- **스케줄링:** 엄격한 우선순위 기반의 선점형(preemptive) 스케줄링을 사용한다. 이는 시스템 내에서 실행 준비가 된 태스크 중 가장 우선순위가 높은 태스크가 항상 CPU를 점유하도록 보장하는 방식이다.16 이를 통해 아무리 많은 저순위 태스크가 존재하더라도 고순위의 긴급한 태스크가 즉시 실행될 수 있다. 이는 처리량과 '공정성'을 중시하는 GPOS 스케줄러와 근본적으로 다른 접근 방식이다.20
- **주요 특징:** RTOS는 낮은 인터럽트 지연 시간(interrupt latency), 작은 커널 크기, 그리고 디스크 없이 ROM과 RAM만으로 시스템을 구성할 수 있는 능력 등 자원이 제한된 임베디드 환경에 최적화된 특징을 가진다.16 RTOS는 요구되는 시간 제약의 엄격함에 따라 경성(Hard) 실시간 시스템과 연성(Soft) 실시간 시스템으로 구분된다. 자동차의 제동 장치나 로켓 제어와 같이 데드라인을 반드시 지켜야 하는 경우는 경성 실시간 시스템에 해당하며, 약간의 시간 지연이 치명적이지 않은 비디오 스트리밍 등은 연성 실시간 시스템에 해당한다.17






- **핵심 철학:** 리눅스와 같은 GPOS는 다양한 애플리케이션과 다수의 사용자가 자원을 '공정하게' 나누어 사용하면서 전체 시스템의 처리율을 극대화하도록 설계되었다.16
- **스케줄링:** 리눅스의 CFS(Completely Fair Scheduler)와 같은 GPOS 스케줄러는 특정 태스크의 엄격한 데드라인 준수보다는 모든 태스크에게 공평한 CPU 시간을 할당하는 것을 목표로 한다.18 따라서 시스템 전체의 성능을 위해 우선순위가 높은 태스크라 할지라도 즉시 선점하지 않고 다른 저순위 태스크들을 먼저 처리할 수 있다.19
- **주요 특징:** GPOS는 방대하고 복잡한 커널을 기반으로 풍부한 기능 세트를 제공한다. 여기에는 고급 네트워킹 스택, 복잡한 파일 시스템, 그래픽 사용자 인터페이스(GUI), 동적 자원 할당 및 관리 등이 포함된다.7 이러한 복잡성과 비결정적인 동작 특성으로 인해 표준 리눅스는 경성 실시간 태스크를 처리하기에 부적합하다.23

이처럼 RTOS와 GPOS는 설계 철학, 스케줄링 방식, 주요 목표 등 모든 면에서 근본적인 차이를 보인다. 따라서 이 두 운영체제를 단일 SoC에 통합하기 위해서는 어느 한쪽의 장점을 희생시키지 않으면서 두 시스템의 공존을 가능하게 하는 정교한 아키텍처적 접근이 필수적이다.

| 특성 (Characteristic)                     | 실시간 운영체제 (RTOS)                                       | 범용 운영체제 (GPOS)                                         |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **주요 목표 (Primary Goal)**              | 시간 제약 준수 및 예측 가능성 (Timeliness & Predictability) 16 | 높은 처리율 및 자원 효율성 (High Throughput & Efficiency) 18 |
| **스케줄링 (Scheduling)**                 | 엄격한 우선순위 기반 선점형 (Strict Priority-based Preemptive) 16 | 공정성 기반 또는 다중 큐 (Fairness-based or Multi-queue) 16  |
| **결정성 (Determinism)**                  | 높음 (High)                                                  | 낮음 (Low)                                                   |
| **응답 시간 (Response Time)**             | 엄격하고 예측 가능한 응답 시간 보장 (Hard, predictable deadlines) 16 | 예측 불가능하며, 보장되지 않음 (Unpredictable, not guaranteed) 16 |
| **인터럽트 지연 (Interrupt Latency)**     | 매우 짧음 (Very Short) 16                                    | 상대적으로 김 (Relatively Long) 19                           |
| **커널 크기 (Kernel Size)**               | 작고 단순함 (Small and Simple) 16                            | 크고 복잡함 (Large and Complex) 16                           |
| **자원 관리 (Resource Management)**       | 정적 할당 위주 (Mostly Static Allocation) 16                 | 동적 할당 및 해제 (Dynamic Allocation & Deallocation) 16     |
| **주요 응용 분야 (Typical Applications)** | 자동차, 항공, 의료, 산업 제어 (Automotive, Avionics, Medical, Industrial Control) 16 | 데스크톱, 서버, 모바일 기기 (Desktop, Server, Mobile Devices) 16 |

**표 1: RTOS와 GPOS의 핵심 특징 비교**

이 표는 왜 단순한 GPOS나 RTOS 단독으로는 현대의 복잡한 혼합 임계 시스템의 요구사항을 충족시킬 수 없는지에 대한 근본적인 이유를 제시한다. 이는 아키텍트가 시스템 설계를 정당화하는 데 필요한 기초 자료이며, 이어지는 비대칭 멀티프로세싱(AMP) 및 가상화 기술 논의의 필요성을 뒷받침한다.






혼합 임계 시스템에서 RTOS와 GPOS를 동시에 운영하기 위한 요구사항이 명확해짐에 따라, 이를 구현하기 위한 두 가지 주요 아키텍처 패러다임이 등장했다: 비대칭 멀티프로세싱(Asymmetric Multiprocessing, AMP)과 하이퍼바이저를 이용한 가상화(Virtualization)이다. 각 패러다임은 고유한 장단점을 가지며, 시스템의 요구사항에 따라 선택적으로 적용된다.











비대칭 멀티프로세싱(AMP)은 멀티코어 프로세서의 각 코어가 독립적인 운영체제 인스턴스나 베어메탈(bare-metal) 코드를 실행하는 모델이다.24 이는 모든 코어가 단일 OS 이미지를 공유하는 대칭 멀티프로세싱(Symmetric Multiprocessing, SMP)과 대조된다. AMP 구성에서는 일반적으로 하나의 코어가 '마스터(master)' 역할을 수행하며 다른 코어들, 즉 '리모트(remote)' 또는 '슬레이브(slave)'의 동작을 제어한다.26 예를 들어, 고성능 Cortex-A 코어에서 리눅스를 마스터로 실행하면서, 실시간 Cortex-R 또는 Cortex-M 코어에서 RTOS를 리모트로 실행하는 것이 일반적인 구성이다. 이 방식은 본질적으로 시스템 자원(CPU, 메모리, 주변장치)을 설계 단계에서 정적으로 분할하여 각 코어에 할당하는 정적 파티셔닝(static partitioning)의 한 형태이다.28






OpenAMP(Open Asymmetric Multi-Processing)는 이러한 AMP 시스템의 구현을 표준화하고 추상화하기 위해 등장한 오픈소스 프레임워크다.29 OpenAMP는 이기종 멀티코어 시스템에서 리눅스와 RTOS/베어메탈 환경 간의 상호작용을 용이하게 하여, 특정 하드웨어에 종속되지 않는 이식성 높은 소프트웨어 개발을 가능하게 한다.32 이 프레임워크는 크게 두 가지 핵심 구성요소로 이루어진다.

- **Remoteproc (Remote Processor Framework):** Remoteproc의 주된 역할은 리모트 프로세서의 생명주기 관리(Life Cycle Management, LCM)이다.32 마스터로 동작하는 리눅스는 

  `remoteproc` 프레임워크를 사용하여 리모트 코어에 펌웨어(일반적으로 ELF 형식의 파일)를 로드하고, 코어를 시작시키거나 중지시키며, 필요에 따라 전원을 켜고 끄는 등의 제어를 수행한다.32 이를 통해 리눅스는 RTOS가 실행되는 코어를 프로그래밍 가능한 보조 프로세서처럼 다룰 수 있다.

- **RPMsg (Remote Processor Messaging):** RPMsg는 `remoteproc` 위에 구축된 프로세서 간 통신(Inter-Processor Communication, IPC) 프로토콜이다.32 이는 마스터와 리모트 간에 텍스트 기반의 이름으로 식별되는 통신 '채널'을 설정한다.37 본질적으로 RPMsg는 공유 메모리를 전송 매체로 사용하는 메시징 버스이며, 이기종 코어 간의 데이터 교환을 위한 표준화된 방법을 제공한다.






RPMsg의 효율성과 낮은 지연 시간의 핵심에는 전송 계층으로 사용되는 VirtIO 기술이 있다.34 VirtIO의 핵심 데이터 구조는 

`vring`(virtqueue)으로, 이는 공유 메모리 상에 구현된 순환 버퍼(circular buffer)이다.39

`vring`의 가장 중요한 특징은 '사용 가능한(available)' 링과 '사용된(used)' 링에 대해 각각 단일 생산자-단일 소비자(single-writer, single-reader) 모델을 채택한다는 점이다. 이 설계는 동기화를 위한 복잡한 잠금(lock)이나 뮤텍스(mutex)의 필요성을 원천적으로 제거하여, 고성능, 저지연 IPC에 이상적인 환경을 제공한다.39 이 메커니즘에 대한 자세한 내용은 3.2절에서 다룬다.






가상화는 하이퍼바이저(또는 가상 머신 모니터, VMM)라는 소프트웨어 계층을 통해 단일 하드웨어 플랫폼 위에 여러 개의 독립적인 가상 머신(Virtual Machine, VM)을 생성하고 실행하는 기술이다.40 각 VM은 자체적인 운영체제를 가질 수 있으며, 하이퍼바이저는 이들 VM 간의 자원을 격리하고 스케줄링하는 역할을 담당한다.

- **타입 1 (베어메탈) 하이퍼바이저:** 하드웨어 위에서 직접 실행되며, 그 위에 게스트 OS들이 동작한다. Xen, Jailhouse, Bao, QNX Hypervisor 등이 여기에 속한다.43 일반적으로 호스트 OS가 없어 오버헤드가 적기 때문에 임베디드 시스템에 더 선호된다.47
- **타입 2 (호스트) 하이퍼바이저:** 일반적인 호스트 OS(예: 리눅스) 위에서 하나의 애플리케이션처럼 실행된다. KVM이 대표적인 예이다. 혼합 임계 시스템에서는 호스트 OS의 비결정성과 오버헤드 때문에 타입 1에 비해 덜 사용된다.48






- **아키텍처:** Xen은 하이퍼바이저, 특권 제어 도메인인 **Dom0**, 그리고 하나 이상의 게스트 도메인인 **DomU**로 구성된다.43 Dom0는 시스템 관리, 다른 도메인 제어, 그리고 실제 하드웨어 장치 드라이버를 호스팅하는 등 매우 중요한 역할을 수행한다.43

- **실시간성 고려사항:** Xen은 본래 하드 실시간용으로 설계되지 않았지만, 특정 구성을 통해 실시간성을 확보할 수 있다. 이를 위해 **RTDS**(연성 실시간 스케줄러)와 같은 특수 스케줄러를 사용하거나, 스케줄링으로 인한 지연을 최소화하기 위해 가상 CPU(vCPU)를 물리 CPU에 고정(pinning)하는 '널 스케줄러(null scheduler)' 방식을 사용한다.23

  `vcpu-pin` 명령어를 통한 이러한 고정 방식은 결정성 확보에 매우 중요하다. Xen의 기본 스케줄러인 **Credit 스케줄러**는 공정성을 목표로 하므로 실시간 환경에는 적합하지 않다.23 CPU를 고정한 Xen 환경에서의 인터럽트 지연 시간은 수 마이크로초 수준까지 단축될 수 있다.52






- **정적 파티셔닝 모델:** Jailhouse는 풍부한 기능 대신 단순성과 최소한의 오버헤드에 최적화된 하이퍼바이저다.54 핵심 원리는 **정적 파티셔닝(static partitioning)**으로, 장치를 에뮬레이션하거나 자원을 동적으로 스케줄링하지 않는다. 대신, 기존 하드웨어(CPU, 메모리, 주변장치)를 격리된 '셀(cell)' 단위로 분할하여 게스트 소프트웨어인 '인메이트(inmate)'에 독점적으로 할당한다.28

- **아키텍처:** Jailhouse는 리눅스 시스템에 의존하지만 Xen과는 다른 방식을 사용한다. 먼저 리눅스가 부팅된 후, Jailhouse 하이퍼바이저가 커널 모듈 형태로 로드된다. 이 초기 리눅스 시스템은 '루트 셀(root cell)'이 되어 다른 '인메이트 셀'을 생성하고 관리하는 데 사용된다.54 인메이트 셀은 베어메탈 애플리케이션이나 다른 OS를 실행할 수 있다.28 특정 CPU가 인메이트 셀에 할당되면, 리눅스는 

  `cpu_down()` 호출을 통해 해당 CPU를 오프라인 상태로 만들어 더 이상 스케줄링에 사용하지 않는다.57






- **독립적인 진화:** Bao는 Jailhouse의 핵심적인 한계, 즉 리눅스에 대한 의존성을 해결하기 위해 특별히 설계된 '클린 슬레이트(clean-slate)' 정적 파티셔닝 하이퍼바이저다.58
- **아키텍처:** Bao는 진정한 의미의 베어메탈, 독립형(standalone) 하이퍼바이저다. 부팅이나 관리를 위해 특권화된 리눅스 VM에 의존하지 않는다.58 이 특징은 신뢰 컴퓨팅 기반(Trusted Computing Base, TCB)을 크게 줄여주어, 보안성을 높이고 안전-필수 애플리케이션을 위한 정형 검증(formal verification) 및 인증을 용이하게 한다.58 Bao는 설계 초기부터 보안, 안전, 실시간성 보장을 목표로 개발되었다.58






AMP와 가상화는 혼합 임계 시스템을 구현하는 서로 다른 철학을 대표한다. AMP(OpenAMP)가 성능과 낮은 오버헤드를 중시하며 코어 간의 경량화된 협력에 초점을 맞춘다면, 가상화는 강력한 격리를 통한 안전성과 보안에 더 큰 비중을 둔다. 특히 Xen에서 Jailhouse, 그리고 Bao로 이어지는 하이퍼바이저의 발전 과정은 유연성을 희생하더라도 검증 가능한 격리를 확보하려는 뚜렷한 경향을 보여준다.

OpenAMP는 본질적으로 통신 프레임워크로 29, 자체적으로 엄격한 메모리나 주변장치 격리를 강제하지 않는다. 이러한 격리는 MMU/MPU나 TrustZone과 같은 하드웨어 기능에 의존해야 하므로 9, 시스템 통합자에게 격리 증명의 부담을 지운다. 반면, 하이퍼바이저는 2단계 주소 변환과 같은 기술을 통해 VM 간의 강력한 공간적 격리를 제공한다.43

Xen은 Dom0 내의 반가상화(paravirtualized) 드라이버를 통해 유연한 자원 공유를 허용하지만 50, 이로 인해 Dom0가 거대하고 복잡한 특권 개체가 되어 TCB를 증가시키고 안전 인증을 어렵게 만든다.58 Jailhouse는 이러한 유연성을 명시적으로 거부하고 엄격한 정적 파티셔닝을 채택하여 하이퍼바이저의 복잡성을 극적으로 줄였다.54 이는 실시간성 및 안전성 분석에 더 유리하다.55 Bao는 여기서 한 걸음 더 나아가 리눅스 루트 셀이라는 마지막 비신뢰 요소를 제거함으로써 58, 하이퍼바이저 중 가장 작은 TCB를 달성하여 분리 커널(separation kernel)의 이상에 가장 근접하게 된다. 이는 인증 관점에서 매우 큰 장점이다.58

따라서 아키텍트의 선택은 하나의 스펙트럼 위에서 이루어진다. 성능에 민감한 보조 프로세서 오프로드와 같이 격리를 별도로 관리할 수 있는 경우 OpenAMP를, 풍부한 기능을 갖춘 레거시 OS들을 통합해야 할 경우 Xen을, 그리고 유연성보다 증명 가능한 정적 격리가 최우선인 안전-필수 시스템에서는 Jailhouse나 Bao를 선택하는 것이 합리적인 결정이 될 것이다.

| 기준 (Criterion)     | OpenAMP (AMP)                                               | Xen (가상화)                                                 | Jailhouse/Bao (정적 파티셔닝)                         |
| -------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| **격리 메커니즘**    | 공유 메모리/IPC 기반 협력. 하드웨어(MMU/TrustZone)에 의존 9 | 하이퍼바이저가 관리하는 완전한 VM 격리 (2단계 주소 변환) 43  | 하드웨어 자원의 정적 분할 및 독점 할당 28             |
| **실시간 결정성**    | IPC 구현에 따라 다름. RTOS 코어는 높은 결정성 유지 가능 63  | 스케줄러 및 vCPU 고정(pinning)에 따라 결정됨. 고정 시 높은 결정성 확보 가능 52 | 설계상 매우 높음 (스케줄링 없음) 55                   |
| **성능 오버헤드**    | 낮음 (직접 실행) 24                                         | 중간 (하이퍼바이저 트랩 발생) 64                             | 최소 (베어메탈에 근접) 23                             |
| **TCB 크기**         | 해당 없음 (프레임워크)                                      | 큼 (하이퍼바이저 + Dom0) 58                                  | 매우 작음 (최소 기능의 하이퍼바이저) 58               |
| **유연성/자원 공유** | 정적. 공유 자원은 신중한 설계 필요                          | 높음 (반가상화 드라이버를 통한 동적 공유) 51                 | 거의 없음 (엄격한 분할 원칙) 54                       |
| **구성 복잡성**      | 디바이스 트리, 리소스 테이블 28                             | VM 설정 파일, Toolstack 50                                   | 셀(Cell) 설정 파일 28                                 |
| **이상적 사용 사례** | 성능 민감성 보조 프로세서 오프로드                          | 기능이 풍부한 레거시 OS 통합                                 | 강력한 격리가 필요한 경성 실시간 시스템과 GPOS의 공존 |

**표 2: 아키텍처 패러다임 비교: OpenAMP 대 Xen 대 Jailhouse/Bao**






Part 2에서 논의된 아키텍처 패러다임들은 ARM의 특정 하드웨어 기능과 정교한 소프트웨어 구성에 의해 현실화된다. 이 장에서는 AMP와 가상화를 가능하게 하는 핵심적인 실리콘 수준의 기술과 실제 시스템을 구성하는 데 필요한 구체적인 소프트웨어 메커니즘을 심도 있게 분석한다.






ARM은 혼합 임계 시스템의 요구사항을 충족시키기 위해 아키텍처 수준에서 다양한 하드웨어 기능을 제공한다. 이러한 기능들은 소프트웨어만으로는 달성하기 어려운 수준의 성능과 격리를 보장하는 기반이 된다.






- **예외 레벨 (Exception Levels, ELs):** ARMv8-A 아키텍처는 4개의 예외 레벨을 정의하여 시스템 소프트웨어의 권한을 계층적으로 분리한다. EL0는 일반 사용자 애플리케이션, EL1은 운영체제 커널, EL2는 하이퍼바이저, 그리고 EL3는 보안 모니터를 위한 공간이다.48 이 하드웨어 기반의 권한 분리 모델은 하이퍼바이저가 EL2에서 실행되면서, 자신보다 낮은 권한 수준인 EL1/EL0에서 동작하는 게스트 OS들을 서로 격리하고 관리할 수 있게 하는 가상화의 근간을 이룬다.48
- **가상화 호스트 확장 (Virtualization Host Extensions, VHE):** VHE는 타입 2 또는 호스트 기반 하이퍼바이저의 성능을 획기적으로 개선하기 위해 도입된 기능이다. 기존에는 호스트 OS 커널이 EL1에서, 하이퍼바이저 기능이 EL2에서 실행되어 둘 사이의 비효율적인 컨텍스트 스위칭이 빈번하게 발생했다. VHE는 `HCR_EL2.E2H` 비트를 설정하여 호스트 OS 커널이 EL2에서 직접 실행될 수 있도록 허용한다.64 이는 EL1 시스템 레지스터에 대한 접근을 EL2의 해당 레지스터로 하드웨어적으로 리디렉션하는 방식으로 이루어지며, 불필요한 예외 레벨 전환을 제거하여 성능을 크게 향상시킨다.68






- **하드웨어 기반의 'World' 분리:** TrustZone은 SoC 전체를 보안(Secure) World와 일반(Normal) World라는 두 개의 독립적인 실행 환경으로 분할하는 하드웨어 보안 기술이다.9 보안 World에서 실행되는 소프트웨어는 시스템의 모든 자원에 접근할 수 있는 반면, 일반 World의 소프트웨어는 비보안(Non-secure)으로 지정된 자원에만 접근이 제한된다.9
- **혼합 임계 시스템에서의 적용:** 이 강력한 하드웨어 격리 메커니즘은 혼합 임계 시스템을 구현하는 데 매우 효과적으로 사용될 수 있다. 예를 들어, 안전-필수 RTOS를 보안 World에서 실행하고, 풍부한 기능을 제공하는 리눅스와 같은 GPOS를 일반 World에서 실행함으로써 하드웨어 수준에서 두 시스템을 격리할 수 있다.9 이는 특히 AMP 시스템에서 하이퍼바이저 기반 격리의 대안 또는 보완책으로 강력한 솔루션을 제공한다.69 멀티코어 시스템에서는 각 코어가 독립적으로 두 World를 전환할 수 있어 유연한 구성이 가능하다.69






- **인터럽트 처리의 과제:** 가상화 환경에서 인터럽트 처리는 시스템 성능에 큰 영향을 미치는 주요 병목 지점 중 하나다. 구형 GIC(Generic Interrupt Controller) 버전(예: GICv2)에서는 게스트 VM으로 전달되어야 할 물리 인터럽트가 발생하면, 먼저 하이퍼바이저(EL2)로 트랩(trap)이 발생한다. 그러면 하이퍼바이저는 GIC의 동작을 소프트웨어적으로 에뮬레이션하여 가상 인터럽트를 생성하고 이를 해당 게스트 VM에 주입(inject)하는 복잡한 과정을 거쳐야 했다.65 이 과정은 상당한 지연 시간을 유발하여 실시간성을 저해하는 주된 요인이었다.52
- **직접 인터럽트 주입 (Direct Interrupt Injection):** 이러한 문제를 해결하기 위해 GICv3, GICv4, 그리고 최신 GICv5와 같은 후기 GIC 버전에서는 인터럽트 직접 주입 기능이 도입되었다.65 이 기능은 하드웨어가 하이퍼바이저의 개입 없이 물리 인터럽트를 목표 vCPU로 직접 라우팅할 수 있게 해준다. 이를 통해 인터럽트 처리 과정에서 발생하는 트랩과 에뮬레이션 오버헤드를 제거하여 지연 시간을 극적으로 줄일 수 있다.72 이 기능은 가상화 환경에서 실시간 성능을 달성하기 위한 필수적인 하드웨어 지원 기술이다.

이처럼 ARM 아키텍처의 발전은 소프트웨어 아키텍처의 한계를 극복하기 위한 하드웨어 기능의 진화와 긴밀하게 연관되어 있다. 초기 ARM 아키텍처는 가상화 지원이 미비하여 게스트 OS 수정이 필요한 반가상화(paravirtualization)에 의존했다.51 ARMv7/v8에서 하드웨어 가상화 확장 기능(EL2, 2단계 주소 변환)이 도입되면서 Xen, KVM과 같은 하이퍼바이저의 ARM 포팅이 가능해졌다.48 그러나 초기 구현에서는 I/O 및 인터럽트 처리 시 발생하는 빈번하고 비용이 큰 EL2 트랩이 성능 병목으로 지적되었다.64 이에 대한 응답으로 ARM은 호스트 OS의 트랩 비용을 줄이기 위한 VHE와 68, 하이퍼바이저를 우회하여 인터럽트를 전달하는 GIC 직접 주입 기능을 개발했다.72 이 피드백 루프는 소프트웨어 구현이 하드웨어의 한계를 드러내고, 이것이 다시 차세대 실리콘 기능 설계를 견인하는 공진화(co-evolutionary) 관계를 보여준다. 따라서 시스템 아키텍트는 목표 성능을 달성하기 위해 이러한 상호작용을 이해하고 GICv3/v4와 GICv2 같이 특정 세대의 기능을 갖춘 SoC를 선택할 수 있어야 한다.






이론적인 아키텍처를 실제 하드웨어에서 동작시키기 위해서는 시스템의 자원과 통신 방식을 명시하는 구체적인 설정 파일이 필요하다. OpenAMP와 Jailhouse는 각각 디바이스 트리와 셀 설정 파일이라는 다른 접근 방식을 사용한다.






- **디바이스 트리의 역할:** 디바이스 트리는 리눅스 커널이 부팅 시점에 시스템에 연결된 하드웨어 장치들을 식별하고 설정하기 위해 사용하는 데이터 구조다.74 OpenAMP 환경에서는 리모트 코어가 사용할 자원을 정의하는 데 핵심적인 역할을 한다.
- **핵심 디바이스 트리 노드:** OpenAMP 시스템을 위한 디바이스 트리 설정은 종종 `.dtbo`(Device Tree Overlay) 파일을 통해 제공되며, 다음과 같은 필수 노드들을 포함한다.28
  - `remoteproc` 노드: 리모트 프로세서 자체를 정의한다.
  - `reserved-memory`: RTOS 펌웨어와 IPC를 위한 공유 메모리로 사용될 물리 메모리 영역을 예약한다.74 여기에 명시된 물리 주소와 크기는 리모트 펌웨어의 메모리 맵과 정확히 일치해야 한다.66
  - RPMsg 공유 메모리 노드: `vring`과 메시지 버퍼가 위치할 공유 메모리 영역을 구체적으로 정의한다.66






- **`.cell` 파일:** Jailhouse의 정적 파티셔닝은 C 언어 구조체 기반의 `.cell` 설정 파일을 통해 정의된다.28 이 소스 파일은 Jailhouse 도구에 의해 바이너리 형식으로 컴파일되어 하이퍼바이저에 전달된다.
- **구성 항목 분석:** i.MX8이나 AM57xx와 같은 실제 보드의 셀 설정 파일 예시를 통해 주요 구성 항목을 분석할 수 있다.28
  - `cpus`: 이 셀에 독점적으로 할당될 물리 CPU 코어를 비트마스크 형태로 지정한다.
  - `mem_regions`: 셀에 할당될 물리 메모리 영역(RAM, 주변장치 MMIO 주소 범위 등)의 목록을 정의한다. 각 영역에 대해 물리 시작 주소, 셀 내부에서의 가상 시작 주소, 그리고 접근 권한(읽기, 쓰기, 실행, I/O)을 명시한다.28
  - `irqchips` 및 `pci_devices`: 특정 인터럽트 컨트롤러나 PCI 장치를 셀에 독점적으로 할당한다.






RPMsg의 lock-free 통신은 `virtqueue`라는 정교한 데이터 구조에 의해 가능하다. `virtqueue`는 공유 메모리 상에 위치하며 세 가지 핵심 요소로 구성된다.39

- **디스크립터 테이블 (Descriptor Table):** 데이터 버퍼를 가리키는 디스크립터들의 배열이다. 각 디스크립터는 데이터 버퍼의 주소, 길이, 그리고 여러 버퍼를 하나의 메시지로 연결하기 위한 플래그(예: `F_NEXT`)를 포함한다.
- **사용 가능 링 (Available Ring):** 생산자(driver)가 소비자(device)에게 제공할 비어있는 버퍼의 디스크립터 인덱스를 저장하는 순환 버퍼다. 생산자는 여기에 인덱스를 추가하고 자신의 인덱스 포인터를 증가시킨다.
- **사용된 링 (Used Ring):** 소비자가 데이터 처리를 완료한 버퍼의 디스크립터 인덱스를 저장하는 순환 버퍼다. 소비자는 여기에 인덱스를 추가하고 자신의 인덱스 포인터를 증가시켜 생산자에게 버퍼가 반환되었음을 알린다.

이 구조의 핵심은 **lock-free 동작 원리**에 있다. 생산자는 오직 '사용 가능 링'에만 쓰고 '사용된 링'에서만 읽는다. 반대로 소비자는 '사용 가능 링'에서만 읽고 '사용된 링'에만 쓴다. 각 주체는 자신만의 인덱스 포인터만 수정하므로, 두 코어가 동시에 같은 메모리 위치에 접근하여 발생하는 경쟁 상태(race condition)가 원천적으로 발생하지 않는다. 따라서 복잡하고 오버헤드가 큰 lock 메커니즘이 필요 없어진다.39 데이터가 준비되었음을 알리는 통지는 선택적으로 사용되는 프로세서 간 인터럽트(종종 'kick' 또는 'doorbell'이라 불림)를 통해 이루어진다.37






혼합 임계 시스템을 실제로 구현하는 과정에서는 이론적 모델만으로는 해결하기 어려운 여러 실질적인 문제에 직면하게 된다. 성능 보장, 공유 하드웨어 관리, 복잡한 상호작용 디버깅, 그리고 보안 확보는 성공적인 시스템 구축을 위해 반드시 해결해야 할 핵심 과제들이다.






혼합 임계 시스템에서 가장 중요한 요구사항 중 하나는 높은 임계성을 가진 태스크의 실시간 성능을 보장하는 것이다. 이를 평가하는 핵심 지표는 인터럽트 지연 시간(interrupt latency)이다.






다양한 아키텍처의 실제 인터럽트 지연 시간을 정량적으로 비교하는 것은 아키텍트가 시스템의 실시간 성능을 예측하고 검증하는 데 필수적이다.

- **벤치마크 결과 분석:** 실제 하드웨어(예: Xilinx Zynq UltraScale+ MPSoC)에서 측정한 벤치마크 결과는 각 기술의 특성을 명확히 보여준다. 베어메탈 환경의 네이티브 인터럽트 지연 시간이 약 300ns라고 가정할 때, Xen 하이퍼바이저를 사용하고 vCPU를 물리 CPU에 고정(`vcpu-pin`)하면 지연 시간은 약 5µs로 증가한다. 여기서 `wfi`(Wait For Interrupt) 명령어 트랩을 비활성화하는 `vwfi=native` 옵션을 추가하면 이 지연 시간을 약 2µs까지 단축할 수 있다.52 Jailhouse와 같은 정적 파티셔닝 하이퍼바이저는 약 0.9µs라는 매우 낮은 평균 지연 시간을 보여주며 베어메탈에 근접한 성능을 보이지만, 간헐적인 지연 시간 스파이크(latency spike)가 발생할 수 있다.23 OpenAMP는 리모트 코어가 전용으로 실시간 작업을 처리할 경우, 3ms 이내의 엄격한 시간 제약을 충족할 수 있음이 확인되었다.63

| 기술 (Technology)              | 플랫폼 (Platform) | 기본 지연 시간 (µs) | 최대 지연 시간 (µs)    | 핵심 구성 (Key Configuration)                 |
| ------------------------------ | ----------------- | ------------------- | ---------------------- | --------------------------------------------- |
| **네이티브/베어메탈**          | Zynq MPSoC        | ~0.3                | -                      | -                                             |
| **OpenAMP on RT-core**         | Zynq MPSoC        | -                   | < 3000 (3ms)           | Dedicated R5 core for real-time task 63       |
| **Xen (Credit Scheduler)**     | Zynq MPSoC        | 2.7                 | 5-7                    | Default "fair" scheduler 23                   |
| **Xen (Pinned vCPU)**          | Zynq MPSoC        | 4.85                | 7.03                   | `vcpu-pin` for static assignment 52           |
| **Xen (Pinned + vwfi=native)** | Zynq MPSoC        | 1.85                | 2.65                   | `vcpu-pin` + `vwfi=native` to reduce traps 52 |
| **Jailhouse**                  | Zynq MPSoC        | ~0.9                | 1-2 (spikes up to 3µs) | Static partitioning 23                        |

**표 3: 실시간 지연 시간 벤치마크 요약**

이러한 정량적 데이터는 아키텍트가 "오버헤드가 낮다"는 질적 표현을 넘어 "지연 시간이 약 2µs"라는 구체적인 수치로 시스템을 평가하고, 스케줄링 가능성 분석 및 시스템 검증에 필요한 핵심 근거를 확보하게 해준다.






Jailhouse와 같은 정적 파티셔닝 하이퍼바이저는 평균적으로 우수한 실시간 성능을 보이지만, 예측하기 어려운 간헐적인 지연 시간 급증(latency spike) 현상이 관찰된다.23 이러한 스파이크의 원인을 분석하는 것은 시스템의 결정성을 보장하는 데 매우 중요하다.

- **캐시 효과:** 인터럽트나 태스크 선점으로 인해 현재 실행 중인 태스크의 중요 데이터나 명령어가 캐시에서 밀려날(evict) 수 있다. 이후 해당 태스크가 다시 실행될 때, 필요한 데이터를 메인 메모리에서 다시 가져와야 하므로 긴 지연 시간을 유발하는 캐시 미스(cache miss)가 발생한다. 이는 선점형 시스템에서 발생하는 고전적인 문제다.81
- **하이퍼바이저/시스템 관리 활동:** 하이퍼바이저 자체가 내부적인 관리 작업을 수행하거나, 시스템 관리 모드(SMM) 인터럽트와 같은 더 낮은 수준의 펌웨어가 하이퍼바이저의 실행을 잠시 중단시킬 때 예측 불가능한 지연이 발생할 수 있다.62
- **마이크로아키텍처 경합:** CPU 코어가 정적으로 분할되어 있더라도, 마지막 레벨 캐시(Last-Level Cache, LLC), 메모리 컨트롤러, 시스템 인터커넥트와 같은 공유 자원에 대한 접근 경합은 상당한 지연을 유발할 수 있다. 이는 예측하기 매우 어려운 문제로, 혼합 임계 시스템 연구의 주요 분야 중 하나다.59
- **I/O 및 DMA:** 우선순위가 높은 DMA(Direct Memory Access) 전송이 메모리 버스를 독점하면, CPU가 명령어 페치(instruction fetch)나 데이터 로드를 위해 메모리에 접근하려 할 때 지연(stall)이 발생할 수 있다.84






혼합 임계 시스템에서 가장 어려운 문제 중 하나는 여러 코어와 운영체제가 공유하는 하드웨어 자원을 어떻게 충돌 없이 효율적으로 관리하느냐이다.






- **문제점:** 하드웨어 캐시 일관성(cache coherency) 프로토콜을 공유하지 않는 이기종 코어(예: Cortex-A와 Cortex-M)로 구성된 AMP 시스템에서는 심각한 데이터 부정합 문제가 발생할 수 있다. 예를 들어, Cortex-A 코어가 공유 버퍼에 데이터를 쓴 경우, 이 데이터는 Cortex-A의 L1/L2 캐시에만 존재할 수 있다(dirty state). 이후 Cortex-M 코어가 메인 메모리(DDR)에서 해당 버퍼를 읽으려고 하면, 캐시에 있는 최신 데이터가 아닌 메모리의 오래된(stale) 데이터를 읽게 되어 데이터 손상과 시스템 오류를 초래한다.84

- **소프트웨어적 해결책 (수동 관리):** 비일관성(non-coherent) 시스템에서의 유일한 해결책은 소프트웨어를 통한 수동 캐시 관리다. 데이터를 생산하는 코어(Cortex-A)는 소비자 코어에 신호를 보내기 전에 반드시 **캐시 클린(clean) 또는 플러시(flush)** 연산을 수행하여 '더티(dirty)' 상태의 데이터를 캐시에서 메인 메모리로 강제로 써야 한다.84 데이터를 소비하는 코어는 메인 메모리에서 데이터를 읽기 전에 자신의 캐시에 있을지 모를 오래된 데이터를 무효화하기 위해 

  **캐시 무효화(invalidate)** 연산을 수행해야 할 수 있다.84 이러한 연산은 ARM 아키텍처에서 제공하는 특정 어셈블리 명령어(예: 구형 ARM의 

  `MCR` 또는 ARMv8의 `DC` 명령어)를 통해 수행된다.86

- **하드웨어적 해결책 (일관성 프로토콜):** Cortex-A 코어 클러스터와 같이 일관성이 보장되는 시스템에서는 AMBA ACE(AXI Coherency Extensions)와 같은 하드웨어 프로토콜이 모든 코어가 항상 동일한 메모리 뷰를 갖도록 자동으로 보장해준다. 이는 소프트웨어 복잡성을 크게 줄여주지만 하드웨어 설계 복잡성을 증가시킨다.69

- **디버깅의 어려움:** 캐시 일관성 관련 버그는 여러 코어의 캐시 상태와 메모리 접근 타이밍에 따라 간헐적으로 발생하기 때문에 디버깅하기가 극도로 어렵다.84






- **문제점:** 공유 캐시를 사용하는 멀티코어 시스템에서는 한 코어에서 실행되는 태스크가 다른 코어의 태스크가 사용하던 캐시 라인을 밀어내는 '태스크 간 캐시 경합(inter-task cache eviction)'이 발생하여 실행 시간을 예측하기 어렵게 만든다.13
- **해결책:** 캐시 파티셔닝은 캐시의 일부(예: 특정 'way'들)를 특정 코어나 중요 태스크 전용으로 예약하는 기술이다. 이를 통해 다른 코어의 간섭을 막아 시간적 격리를 강화하고, 최악 실행 시간(Worst-Case Execution Time, WCET) 분석을 용이하게 한다.13
- **구현 및 비용:** 캐시 파티셔닝은 '페이지 컬러링(page coloring)'과 같은 소프트웨어 기법이나, Intel CAT 또는 ARM DSU의 캐시 파티셔닝 기능과 같은 하드웨어 지원을 통해 구현될 수 있다.59 페이지 컬러링은 메모리 단편화와 슈퍼페이지(superpage) 사용 불가 등의 성능 저하를 유발할 수 있다. 파티셔닝은 예측 가능성을 높이는 반면, 각 코어가 사용할 수 있는 유효 캐시 크기를 줄여 전체적인 시스템 성능을 저하시킬 수 있는 트레이드오프가 존재한다.81






- **과제:** 리눅스가 실행되는 Cortex-A 코어와 RTOS가 실행되는 Cortex-M 코어가 단일 SPI 주변장치를 어떻게 안전하게 공유할 수 있는가? 직접적인 동시 접근은 데이터 손상과 시스템 불안정을 초래할 것이다.
- **아키텍처 패턴: 프록시 드라이버:** 이 문제에 대한 표준적인 해결책은 프록시(proxy) 또는 리소스 매니저(resource manager) 모델을 사용하는 것이다.31
  1. 해당 주변장치는 하나의 코어, 일반적으로 실시간 요구사항이 있는 RTOS(Cortex-M)에 독점적으로 할당된다. 리눅스 측에서는 디바이스 트리에서 해당 주변장치를 'disabled'로 설정하여 커널이 직접 제어하지 못하도록 막는다.28
  2. RTOS는 해당 SPI 주변장치를 직접 제어하는 네이티브 드라이버를 실행한다.
  3. 리눅스는 하드웨어를 직접 제어하지 않는 '프록시 SPI 드라이버'를 갖는다. 이 드라이버는 SPI 트랜잭션 요청(예: "16바이트 쓰기", "8바이트 읽기")을 RPMsg를 통해 RTOS에서 실행 중인 서비스로 전송한다.92
  4. RTOS의 서비스는 이 메시지를 수신하여 네이티브 드라이버를 통해 실제 SPI 트랜잭션을 수행하고, 그 결과를 다시 RPMsg를 통해 리눅스로 반환한다.
  5. 이러한 방식은 공유 주변장치에 대한 접근을 직렬화하며, RTOS가 게이트키퍼 역할을 하여 리눅스로부터의 비-필수적인 접근보다 실시간 접근을 우선적으로 처리할 수 있도록 보장한다.






혼합 임계 시스템의 복잡성은 디버깅과 검증에 새로운 차원의 어려움을 제시한다. 여러 코어에서 각기 다른 OS가 동시에 실행되는 환경을 효과적으로 분석하기 위해서는 기존의 단일 OS 디버깅 도구를 넘어서는 접근법이 필요하다.

- **다중 OS JTAG 디버깅:** 단일 디버거 인터페이스를 통해 서로 다른 소프트웨어 스택을 실행하는 여러 코어를 동시에 중지시키고 상태를 검사하는 것은 매우 어려운 작업이다. **Lauterbach TRACE32**와 같은 고급 JTAG 디버거는 리눅스와 FreeRTOS, SAFERTOS 등 다양한 RTOS에 대한 OS 인식(OS-awareness) 기능을 동시에 제공한다.94 이를 통해 개발자는 리눅스 애플리케이션에 브레이크포인트를 설정하고, 그 시점에서 RTOS의 태스크 목록을 확인하며, RTOS 코드를 한 단계씩 실행하는 등 시간적으로 동기화된 단일 디버깅 세션 내에서 전체 시스템의 동작을 분석할 수 있다.96
- **다중 트레이스 상관 분석:** 시스템의 동작을 비침습적으로 분석하기 위해 트레이스 도구를 활용할 수 있다. 리눅스(LTTng, ftrace 등)와 RTOS에서 각각 수집된 트레이스 데이터를 **Trace Compass**와 같은 통합 분석 도구에 로드할 수 있다.98 Trace Compass는 여러 트레이스를 공통 타임라인에 맞춰 동기화하여, 리눅스에서의 특정 이벤트(예: RPMsg 메시지 전송)가 RTOS의 태스크 활성화에 어떻게 영향을 미치는지 인과 관계를 시각적으로 분석할 수 있게 해준다.99
- **정형 검증 (Formal Verification):** 안전-필수 소프트웨어, 특히 하이퍼바이저의 정확성을 수학적으로 증명하기 위해 정형 검증의 중요성이 커지고 있다. Xen과 같이 복잡한 하이퍼바이저는 전체를 정형 검증하기 어렵지만, **Bao**나 **Jailhouse**와 같이 미니멀리즘 설계를 채택한 하이퍼바이저는 이러한 기법을 적용하기에 훨씬 용이하다.58 이는 단순한 테스트만으로는 달성할 수 없는 높은 수준의 보증을 제공하므로, 기능 안전 인증 과정에서 큰 이점을 가진다.102






혼합 임계 시스템의 격리 아키텍처는 보안 측면에서도 중요한 역할을 하지만, 새로운 공격 벡터를 고려해야 한다.

- **하이퍼바이저 사이드 채널 공격:** 가상화 환경의 주요 보안 위협 중 하나는 교차-VM 사이드 채널 공격이다. 악의적인 VM이 피해자 VM과 공유하는 마이크로아키텍처 자원, 특히 마지막 레벨 캐시(LLC)의 상태 변화를 관찰하여 피해자 VM의 비밀 정보(예: 암호화 키)를 유추하는 공격이다.104
- **완화 전략:** 앞서 예측 가능성을 위해 논의된 캐시 파티셔닝 기술은 캐시 기반 사이드 채널 공격에 대한 주요 방어 기법이기도 하다.107 캐시를 분할하여 VM 간 공유를 원천적으로 차단함으로써 정보 유출 경로를 막는 것이다. 이 외에도 다양한 소프트웨어 및 하드웨어 기반 완화 기법이 존재하지만, 이는 여전히 활발한 연구 분야로 남아있다.104

결론적으로, 혼합 임계 시스템에서 발생하는 가장 어려운 문제들(지연 시간 스파이크, 자원 경합, 디버깅)은 리눅스나 RTOS 자체의 문제가 아니라, 이들이 공유 하드웨어를 통해 상호작용하는 **인터페이스**에서 발생하는 문제다. 따라서 성공적인 시스템 아키텍트는 개별 OS 최적화를 넘어, 이러한 하드웨어 및 소프트웨어 경계 전반의 상호작용을 관리하고 검증하는 전체론적(holistic) 접근 방식을 취해야 한다. 프록시 드라이버 패턴, 캐시 파티셔닝, 다중 OS 트레이스 분석과 같은 솔루션들은 모두 이러한 인터페이스 문제를 명시적으로 해결하기 위한 기법들이다.






지금까지 논의된 기술적 원리와 아키텍처 패러다임은 실제 산업 현장, 특히 자동차 산업에서 활발하게 적용되며 차세대 시스템의 근간을 이루고 있다. 이 장에서는 자동차 디지털 콕핏을 주요 사례로 분석하고, 기능 안전 표준의 역할과 미래 기술 동향을 조망한다.






자동차 디지털 콕핏은 계기판, 인포테인먼트 시스템(IVI), 헤드업 디스플레이(HUD), 운전자 모니터링 시스템 등 다양한 임계성을 가진 기능들이 단일 SoC에 통합된 대표적인 혼합 임계 시스템이다.






주요 반도체 공급업체들은 이러한 요구사항을 충족시키기 위해 혼합 임계 개념을 적용한 정교한 SoC를 출시하고 있다.

- **NXP i.MX 8 제품군:** 이 SoC들은 강력한 Cortex-A 클러스터(A72, A53 등)와 실시간 처리를 위한 Cortex-M4/M7 코어를 함께 탑재한 이기종 멀티코어 아키텍처를 특징으로 한다.8 일반적인 구성은 A코어에서 리눅스나 안드로이드 기반의 IVI 시스템을 실행하고, M코어에서는 RTOS(예: FreeRTOS)를 실행하여 계기판 데이터 처리나 CAN 게이트웨이와 같은 실시간 작업을 담당하는 것이다.8 이 제품군은 하드웨어 가상화와 도메인 보호 기능을 명시적으로 지원하여 강력한 격리 환경을 제공한다.8
- **Texas Instruments Jacinto TDA4VM:** 이 ADAS(첨단 운전자 보조 시스템)용 프로세서는 듀얼 코어 Cortex-A72 클러스터와 다수의 Cortex-R5F 록스텝(lock-step) 코어, 그리고 전용 DSP/AI 가속기를 결합한 구조다.110 이 아키텍처는 R5F 코어가 센서 융합이나 차량 제어와 같은 안전-필수, 시간-민감성 작업을 처리하고, A72 코어는 상위 레벨의 애플리케이션과 인식 알고리즘을 담당하도록 하여 하이퍼바이저의 필요성을 최소화하면서 다중 OS 애플리케이션을 용이하게 하도록 설계되었다.110
- **Renesas R-Car 제품군:** R-Car SoC는 IVI부터 통합 콕핏에 이르기까지 광범위한 자동차 애플리케이션을 목표로 설계되었다.113 이들은 종종 다수의 Cortex-A 코어(A15, A7 등)와 다른 종류의 코어(SH-4A 등)를 포함하며 115, QNX, 리눅스 등 여러 OS를 하이퍼바이저 위에서 실행하여 계기판, IVI, 운전자 모니터링 시스템을 단일 칩에서 통합 관리하는 것을 명시적으로 지원한다.115

| SoC 제품군 (SoC Family)       | 고성능 코어 (GPOS 타겟)         | 실시간 코어 (RTOS 타겟) | 핵심 격리 기술                                     | 기능 안전 (ASIL Level) | 지원 OS/하이퍼바이저                       |
| ----------------------------- | ------------------------------- | ----------------------- | -------------------------------------------------- | ---------------------- | ------------------------------------------ |
| **NXP i.MX 8** 8              | 2-4x Cortex-A72/A53             | 2x Cortex-M4F           | 하드웨어 가상화, TrustZone, 리소스 도메인 컨트롤러 | ASIL-B 타겟 109        | Linux, Android, FreeRTOS, QNX, Green Hills |
| **TI Jacinto TDA4VM** 110     | 2x Cortex-A72                   | 6x Cortex-R5F           | 코어 클러스터 분리, MCU 아일랜드                   | ASIL-D/SIL-3 타겟 111  | Linux, RTOS (최소 하이퍼바이저 필요)       |
| **Renesas R-Car (H3/M3)** 115 | 4x Cortex-A15/A53, 4x Cortex-A7 | 1x SH-4A (H2)           | 하이퍼바이저 지원, IMP-X4(비전)                    | ASIL-B (일부 기능)     | QNX, Linux, Windows Embedded Automotive    |

**표 4: 상용 자동차용 SoC의 혼합 임계 기능 매핑**

이 표는 아키텍트가 소프트웨어 아키텍처를 선택할 때, 이를 지원할 수 있는 실제 하드웨어 플랫폼과 어떻게 연결되는지를 보여준다. 이는 이론과 실제를 잇는 중요한 다리 역할을 한다.






- **ISO 26262 표준:** ISO 26262는 자동차 전기/전자 시스템의 기능 안전을 다루는 국제 표준이다.117 이 표준은 위험 분석 및 리스크 평가를 통해 시스템에 요구되는 안전 수준을 자동차 안전 무결성 등급(Automotive Safety Integrity Level, ASIL)으로 정의하며, ASIL A(가장 낮음)부터 ASIL D(가장 높음)까지 구분한다.118
- **인증의 과제:** 혼합 임계 시스템의 핵심적인 이점은 '간섭으로부터의 자유'가 증명된다면, 높은 ASIL 등급의 구성요소(예: ASIL D 계기판)와 낮은 임계성의 구성요소(예: QM 등급의 인포테인먼트)를 통합하더라도 후자를 높은 등급으로 인증할 필요가 없다는 점이다.9 이로 인해 시스템의 격리 메커니즘 자체(하이퍼바이저 또는 AMP 프레임워크)의 인증이 매우 중요해진다.
- **인증된 솔루션:** **QNX Hypervisor for Safety**나 **VOSySmonitor**와 같은 상용 하이퍼바이저는 ISO 26262 ASIL D와 같은 높은 수준의 안전 표준에 대해 사전 인증을 받아 제공된다.46 이는 개발사가 시스템을 인증받는 데 드는 시간과 비용을 크게 절감해주는 강력한 가치를 제공한다. 반면, OpenAMP와 같은 오픈소스 프레임워크는 일반적으로 사전 인증 없이 제공되므로, 시스템 통합자가 직접 인증의 부담을 져야 한다.120






- **클라우드 네이티브 아키텍처:** SOAFEE는 ARM이 주도하는 산업 컨소시엄으로, 소프트웨어 정의 차량(Software-Defined Vehicle, SDV)을 위한 클라우드 네이티브 소프트웨어 아키텍처를 정의하는 것을 목표로 한다.123
- **목표:** SOAFEE의 핵심은 컨테이너, 오케스트레이션과 같은 클라우드 기술을 활용하여, 리눅스와 RTOS가 공존하는 플랫폼 위에서 혼합 임계 자동차 기능들을 개발하고 배포하는 것이다.123 이는 하이퍼바이저나 AMP와 같은 저수준의 격리 기술 위에 구축되는 한 단계 더 높은 수준의 추상화 계층을 의미하며, 자동차 소프트웨어 개발의 패러다임을 바꾸고 있다.






혼합 임계 시스템 분야는 여전히 활발한 연구가 진행 중인 영역이다.

- **미해결 연구 과제:** 학계의 주요 연구 동향 조사에 따르면 다음과 같은 주제들이 핵심적인 미해결 과제로 남아있다.5
  - 고급 멀티코어 혼합 임계 스케줄링 알고리즘
  - CPU 이외의 공유 자원(메모리 버스, 인터커넥트 등)에 대한 예측 가능한 관리 기법
  - 복잡한 런타임 환경에 대한 정형 검증
  - 조합적 분석(Compositional Analysis): 독립적으로 인증된 구성요소들을 통합하여 전체 시스템을 인증하는 방법론
- **미래: RISC-V와 그 너머:** 혼합 임계 시스템의 원리는 ARM에만 국한되지 않는다. RISC-V 생태계 역시 하이퍼바이저 확장 기능 개발 및 Xvisor, KVM, Bao와 같은 하이퍼바이저 포팅을 통해 관련 솔루션을 활발히 개발하고 있다.128 RISC-V의 개방형 아키텍처는 검증 가능한 혼합 임계 하드웨어/소프트웨어 공동 설계를 위한 새로운 기회를 제공할 수 있다.131






자동차 산업은 하드웨어 중심에서 소프트웨어 중심의 차량(SDV)으로 근본적인 플랫폼 전환을 겪고 있다. 이 보고서에서 논의된 AMP, 하이퍼바이저, RTOS/리눅스 공존 기술들은 이러한 전환을 가능하게 하는 필수적인 기반 인프라다. 과거의 과제가 분산된 ECU들을 단일 SoC로 '통합'하는 것이었다면, 현재의 과제는 이 통합된 하드웨어 위에서 폭발적으로 증가하는 소프트웨어의 '복잡성'을 관리하는 것이다.

SOAFEE와 같은 표준화 움직임은 업계의 관심이 이미 "혼합 임계성을 어떻게 구현할 것인가"에서 "클라우드 네이티브 워크플로우를 통해 어떻게 대규모로 관리할 것인가"로 이동하고 있음을 시사한다.123 이는 하이퍼바이저와 같은 기반 기술이 점차 보편화된 빌딩 블록이 되고, 그 위에 구축될 컨테이너 런타임, 오케스트레이션 프레임워크, 서비스 지향 미들웨어의 선택이 차세대 아키텍처의 핵심 결정사항이 될 것임을 의미한다.

따라서 미래의 시스템 아키텍트는 AMP와 가상화라는 기반 기술을 완벽히 이해하는 동시에, 이러한 기술들이 제공하는 혼합 임계 제약 조건 내에서 동작할 수 있는 더 높은 수준의 소프트웨어 스택과 개발 방법론에 대한 통찰력을 갖추어야 한다. 이는 단순한 기술적 진화를 넘어, 지능형 자율 시스템의 미래를 여는 핵심적인 패러다임 전환이다.




1. VM-Based Real-Time Services for Automotive Control Applications - TU Chemnitz, accessed July 7, 2025, https://www.tu-chemnitz.de/informatik/CAS/publications/pdfs/Masrur2010c.pdf
2. Multi-Core Devices for Safety-Critical Systems: A Survey - UPCommons, accessed July 7, 2025, https://upcommons.upc.edu/bitstream/handle/2117/331690/Multi_core.pdf
3. Mixed criticality - Wikipedia, accessed July 7, 2025, https://en.wikipedia.org/wiki/Mixed_criticality
4. What is mixed criticality? - Trenton Systems, accessed July 7, 2025, https://www.trentonsystems.com/en-us/resource-hub/blog/what-is-mixed-criticality
5. 82 A Survey of Research into Mixed Criticality Systems - University of York, accessed July 7, 2025, https://www-users.york.ac.uk/~rd17/papers/CSURreviewMCS.pdf
6. Automotive Hypervisor - BlackBerry QNX, accessed July 7, 2025, https://blackberry.qnx.com/en/ultimate-guides/automotive-hypervisor
7. 1. 임베디스 시스템 소개 - ARM Architecture, accessed July 7, 2025, http://www.jkelec.co.kr/img/lecture/arm_arch/arm_arch_2.html
8. i.MX 8 Family Applications Processor | Arm Cortex-A53/A72/M4 | NXP Semiconductors, accessed July 7, 2025, https://www.nxp.com/products/i.MX8
9. ARM TrustZone®, accessed July 7, 2025, https://www.nxp.com/docs/en/supporting-information/FTF-DES-N2020-PDF.pdf
10. How to Build a Mixed-Criticality System in Industry - White Rose Research Online, accessed July 7, 2025, https://eprints.whiterose.ac.uk/id/eprint/156999/1/WMC2019_ZheJ.pdf
11. Preparation of Papers for JCSE: Isolation of Shared Resources for Mixed-Criticality AUTOSAR Applications, accessed July 7, 2025, http://jcse.kiise.org/files/V16N3-02.pdf
12. RTOS | Real Time Operating Systems as Part of the Embedded Platform, accessed July 7, 2025, https://www.highintegritysystems.com/rtos/
13. Exploring the Arm MPAM Extension for Static Partitioning Virtualization - RepositóriUM, accessed July 7, 2025, [https://repositorium.sdum.uminho.pt/bitstream/1822/91425/1/Goncalo%20Goncalves%20Freitas.pdf](https://repositorium.sdum.uminho.pt/bitstream/1822/91425/1/Goncalo Goncalves Freitas.pdf)
14. Flexible and Dynamic Scheduling of Mixed-Criticality Systems - PMC, accessed July 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9572869/
15. A Survey of Research into Mixed Criticality Systems, accessed July 7, 2025, https://eprints.whiterose.ac.uk/125823/
16. 임베디드 시스템의 핵심 기술, RTOS란? - Part.1 (기본 개념 및 특징, FreeRTOS, OSEK/VDX, 용어 정리-Task, Deadline 등) - Graffiti di Modesty - 티스토리, accessed July 7, 2025, [https://rangvest.tistory.com/entry/%EC%8B%A4%EC%8B%9C%EA%B0%84-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9CRTOS-%EC%9E%84%EB%B2%A0%EB%94%94%EB%93%9C-%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9D%98-%ED%95%B5%EC%8B%AC-%EA%B8%B0%EC%88%A0-Part1%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90-%EB%B0%8F-%ED%8A%B9%EC%A7%95-%ED%98%95%ED%83%9C-%EC%9A%A9%EC%96%B4-%EC%A0%95%EB%A6%AC](https://rangvest.tistory.com/entry/실시간-운영체제RTOS-임베디드-시스템의-핵심-기술-Part1기본-개념-및-특징-형태-용어-정리)
17. [FreeRTOS] FreeRTOS란? - 개발새발 - 티스토리, accessed July 7, 2025, [https://kfdd6630.tistory.com/entry/FreeRTOS-FreeRTOS%EB%9E%80](https://kfdd6630.tistory.com/entry/FreeRTOS-FreeRTOS란)
18. Tag: GPOS | hamlog, accessed July 7, 2025, https://harnzzi.github.io/tags/GPOS/
19. RTOS와 GTOS <스케쥴링> 편 - hamlog, accessed July 7, 2025, [https://harnzzi.github.io/2020/02/25/RTOS%EC%99%80-GTOS-%EC%8A%A4%EC%BC%80%EC%A5%B4%EB%A7%81-%ED%8E%B8/](https://harnzzi.github.io/2020/02/25/RTOS와-GTOS-스케쥴링-편/)
20. Impact of the Linux Real-time Enhancements on the System Performances for Multi-core Intel Architectures - ResearchGate, accessed July 7, 2025, https://www.researchgate.net/profile/Nabil-Litayem/publication/252243132_Impact_of_the_Linux_Real-time_Enhancements_on_the_System_Performances_for_Multi-core_Intel_Architectures/links/54cb35ca0cf2517b756151dd/Impact-of-the-Linux-Real-time-Enhancements-on-the-System-Performances-for-Multi-core-Intel-Architectures.pdf
21. [RTOS study 01] - RTOS란? - Engineering Agit - 티스토리, accessed July 7, 2025, https://engineering-agit.tistory.com/32
22. 62. 임베디드 시스템, accessed July 7, 2025, https://www.chip1stop.com/sp/knowledge/062_embedded-system_ko
23. High Performance Real Time Linux Solution for Xilinx Ultrascale+, accessed July 7, 2025, https://www.xilinx.com/publications/events/developer-forum/2018-frankfurt/high-performance-real-time-linux-solution-for-xilinx-ultrascale-plus.pdf
24. Combining RTOS and Linux in Heterogeneous Multi-Core Systems: When and How to Use AMP Architectures - Promwad, accessed July 7, 2025, https://promwad.com/news/rtos-linux-amp-systems
25. FreeRTOS scheduling (single-core, AMP and SMP), accessed July 7, 2025, https://www.freertos.org/single-core-amp-smp-rtos-scheduling.html
26. 대칭과 비대칭 멀티 프로세싱 - Cog Factory, accessed July 7, 2025, https://clownhacker.tistory.com/10
27. [운영체제] 다중처리기 스케줄링 - 이누의 개발성장기 - 티스토리, accessed July 7, 2025, https://inuplace.tistory.com/319
28. 3.8.1. Jailhouse Hypervisor - Processor SDK Linux Automotive Documentation - http, accessed July 7, 2025, https://software-dl.ti.com/jacinto7/esd/processor-sdk-linux-jacinto7/06_01_01_02/exports/docs/linux/Foundational_Components/Virtualization/Jailhouse.html
29. The OpenAMP Project, accessed July 7, 2025, https://www.openampproject.org/
30. OpenAMP/open-amp: The main OpenAMP library implementing RPMSG, Virtio, and Remoteproc for RTOS etc - GitHub, accessed July 7, 2025, https://github.com/OpenAMP/open-amp
31. OpenAMP Framework User Reference - Texas Instruments, accessed July 7, 2025, https://git.ti.com/cgit/processor-sdk/open-amp/plain/docs/openamp_ref.pdf
32. OpenAMP Framework - Linaro, accessed July 7, 2025, https://static.linaro.org/connect/bkk19/presentations/bkk19-204.pdf
33. Adam Taylor's MicroZed Chronicles, Part 234 (UltraZed Edition 21): OpenAMP-How to use the Zynq UltraScale MPSoC's Arm Cortex-A53 and -R5 processors together in a design - AMD Adaptive Support, accessed July 7, 2025, https://adaptivesupport.amd.com/s/article/826083?language=en_US
34. Introduction to OpenAMP Library, accessed July 7, 2025, https://www.openampproject.org/docs/whitepapers/Introduction_to_OpenAMPlib_v1.1a.pdf
35. OpenAMP - 2025.1 English - UG1304, accessed July 7, 2025, https://docs.amd.com/r/en-US/ug1304-versal-acap-ssdg/OpenAMP
36. Zynq UltraScale+ MPSoC Software Developer Guide - AMD, accessed July 7, 2025, https://www.xilinx.com/support/documents/sw_manuals/xilinx2022_2/ug1137-zynq-ultrascale-mpsoc-swdev.pdf
37. Linux RPMsg framework overview - stm32mpu - ST wiki, accessed July 7, 2025, https://wiki.st.com/stm32mpu/wiki/Linux_RPMsg_framework_overview
38. Remote Processor Messaging (rpmsg) Framework - The Linux Kernel documentation, accessed July 7, 2025, https://docs.kernel.org/staging/rpmsg.html
39. RPMsg Protocol Layers - OpenAMP documentation, accessed July 7, 2025, https://openamp.readthedocs.io/en/main-next/protocol_details/rpmsg_protocol.html
40. TECHNOLOGY | 페르세우스 - PERSEUS, accessed July 7, 2025, https://www.cyberperseus.com/technology/technology.php
41. 하이퍼바이저(Hypervisor, Hyper V)란? - Red Hat, accessed July 7, 2025, https://www.redhat.com/ko/topics/virtualization/what-is-a-hypervisor
42. [Linux] 하이퍼 바이저란 - 쟈누의 기록공간, accessed July 7, 2025, https://snepbnt.tistory.com/515
43. [Server] Xen Hypervisor(Xen 하이퍼바이저) 란? (1/2) - 망나니개발자 - 티스토리, accessed July 7, 2025, https://mangkyu.tistory.com/82
44. 하이퍼바이저(hypervisor)란? - alwaysawake - 티스토리, accessed July 7, 2025, https://smelting.tistory.com/19
45. All About Hypervisors: ESXi vs Hyper-V, XenServer, Proxmox, KVM, & AHV - StorMagic, accessed July 7, 2025, https://stormagic.com/company/blog/all-about-hypervisors-esxi-vs-hyper-v-xenserver-proxmox-kvm-ahv/
46. QNX Hypervisor for Safety Pre-Certified Embedded Hypervisor, accessed July 7, 2025, https://blackberry.qnx.com/en/products/safety-certified/qnx-hypervisor-for-safety
47. (PDF) Specific Electronic Platform to Test the Influence of ..., accessed July 7, 2025, https://www.researchgate.net/publication/360864265_Specific_Electronic_Platform_to_Test_the_Influence_of_Hypervisors_on_the_Performance_of_Embedded_Systems
48. How the Xen Hypervisor Supports CPU Virtualization on ARM - Star Lab Software, accessed July 7, 2025, https://www.starlab.io/blog/how-the-xen-hypervisor-supports-cpu-virtualization-on-arm
49. Bao's static partitioning architecture. | Download Scientific Diagram - ResearchGate, accessed July 7, 2025, https://www.researchgate.net/figure/Baos-static-partitioning-architecture_fig1_338720622
50. Xen Hypervisor - 반도체 공부 - 티스토리, accessed July 7, 2025, https://zsik10000.tistory.com/6
51. Virtualization on ARM with Xen, accessed July 7, 2025, https://xenproject.org/blog/virtualization-on-arm-with-xen/
52. Xen on ARM interrupt latency, accessed July 7, 2025, https://xenproject.org/blog/xen-on-arm-interrupt-latency/
53. Xen-ARM 하이퍼바이저와 실시간 인터럽트 처리 -한국정보과학회 ..., accessed July 7, 2025, https://koreascience.kr/article/CFKO201128451825468.pub?&lang=ko&orgId=kiss
54. siemens/jailhouse: Linux-based partitioning hypervisor - GitHub, accessed July 7, 2025, https://github.com/siemens/jailhouse
55. jailhouse/FAQ.md at master - GitHub, accessed July 7, 2025, https://github.com/siemens/jailhouse/blob/master/FAQ.md
56. jailhouse/README.md at master - GitHub, accessed July 7, 2025, https://github.com/siemens/jailhouse/blob/master/README.md
57. 3.14.1. Jailhouse Hypervisor - Processor SDK Linux Documentation - http, accessed July 7, 2025, https://software-dl.ti.com/processor-sdk-linux/esd/docs/06_03_00_106/linux/Foundational_Components/Virtualization/Jailhouse.html
58. Bao: a modern lightweight embedded hypervisor - Sandro Pinto, accessed July 7, 2025, https://sandro2pinto.github.io/files/ew2020-bao.pdf
59. (PDF) Bao: A Lightweight Static Partitioning Hypervisor for Modern Multi-Core Embedded Systems - ResearchGate, accessed July 7, 2025, https://www.researchgate.net/publication/338720622_Bao_A_Lightweight_Static_Partitioning_Hypervisor_for_Modern_Multi-Core_Embedded_Systems
60. Bao: A Lightweight Static Partitioning Hypervisor for Modern Multi-Core Embedded Systems, accessed July 7, 2025, https://drops.dagstuhl.de/entities/document/10.4230/OASIcs.NG-RES.2020.3
61. Bao: A Lightweight Static Partitioning Hypervisor for Modern Multi-Core Embedded Systems, accessed July 7, 2025, https://d-nb.info/1365343391/34
62. Understanding the Jailhouse hypervisor, part 1 - LWN.net, accessed July 7, 2025, https://lwn.net/Articles/578295/
63. Evaluating Latency in Multiprocessing Embedded Systems for the Smart Grid - MDPI, accessed July 7, 2025, https://www.mdpi.com/1996-1073/14/11/3322
64. ARM Virtualization: Performance and Architectural Implications - NSF-PAR, accessed July 7, 2025, https://par.nsf.gov/servlets/purl/10310788
65. Reducing Hypervisor Overhead for Virutal Interrupt Delivery in KVM/ARM Platform, accessed July 7, 2025, https://www.researchgate.net/publication/377631028_Reducing_Hypervisor_Overhead_for_Virutal_Interrupt_Delivery_in_KVMARM_Platform
66. OpenAMP on Kria SOM - GitHub Pages, accessed July 7, 2025, https://xilinx.github.io/kria-apps-docs/openamp.html
67. Introduction to the ARMv8 Virtualization System | openEuler, accessed July 7, 2025, https://www.openeuler.org/en/blog/yorifang/2020-10-24-arm-virtualization-overview.html
68. Virtualization host extensions - Arm Developer, accessed July 7, 2025, https://developer.arm.com/documentation/102142/latest/Virtualization-host-extensions
69. ARM Security Technology Building a Secure System using ..., accessed July 7, 2025, https://developer.arm.com/documentation/PRD29-GENC-009492/latest/TrustZone-Hardware-Architecture/Processor-architecture/Multiprocessor-systems-with-the-Security-Extensions
70. Asymmetric Multi Processing (AMP) Configurations - Xilinx Wiki, accessed July 7, 2025, https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841668/Multi-OS+Support+AMP+Hypervisor?showComments=true&showCommentArea=true
71. Using ARM GIC Virtualization Extensions with seL4 - Devel - Lists, accessed July 7, 2025, https://lists.sel4.systems/hyperkitty/list/devel@sel4.systems/thread/MDO2CO2GGKWV6WTMWJPSC2JD3ZGZGPSQ/
72. Introducing GICv5: Scalable And Secure Interrupt Management For Arm - Architectures and Processors blog, accessed July 7, 2025, https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/introducing-gicv5
73. KVM/ARM: An Open-Source ARM Virtualization System, accessed July 7, 2025, https://systems.cs.columbia.edu/projects/kvm-arm/
74. Xilinx Wiki - Confluence, accessed July 7, 2025, https://xilinx-wiki.atlassian.net/wiki/display/A/Device+Tree+Tips?f=print
75. Building Device Tree Overlays for PS - 2025.1 English - UG1144, accessed July 7, 2025, https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Building-Device-Tree-Overlays-for-PS
76. Linux remoteproc framework overview - stm32mpu - ST wiki, accessed July 7, 2025, https://wiki.st.com/stm32mpu/wiki/Linux_remoteproc_framework_overview
77. 5. OpenAMP RemoteProc (Under Discussion), accessed July 7, 2025, https://openamp.readthedocs.io/en/main-next/lopper/specification/source/chapter5-remoteproc.html
78. OpenAMP Base Hardware Configurations - Xilinx Wiki, accessed July 7, 2025, [https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/1837006921/OpenAMP%20Base%20Hardware%20Configurations](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/1837006921/OpenAMP Base Hardware Configurations)
79. Jailhouse Guide - Variscite Wiki, accessed July 7, 2025, https://variwiki.com/index.php?title=Jailhouse_Guide
80. VirtIO specification - Virtual I/O Device (VIRTIO) Version 1.0 - OASIS Open, accessed July 7, 2025, https://docs.oasis-open.org/virtio/virtio/v1.0/virtio-v1.0.html
81. Evaluation of Cache Partitioning for Hard Real-Time Systems - University of York, accessed July 7, 2025, https://www-users.york.ac.uk/~rd17/papers/cachePartitioningECRTS3-complete.pdf
82. Xen slower than Jailhouse?? - Join mailing lists - Xen Project, accessed July 7, 2025, https://old-list-archives.xen.org/archives/html/xen-users/2024-02/msg00019.html
83. Challenges in Future Avionic Systems on Multi-Core Platforms - DiVA portal, accessed July 7, 2025, https://www.diva-portal.org/smash/get/diva2:929730/FULLTEXT01.pdf
84. What Is Coherency? - Semiconductor Engineering, accessed July 7, 2025, https://semiengineering.com/what-is-coherency/
85. Efficient method to flush cache memory in ARM assembly - Stack Overflow, accessed July 7, 2025, https://stackoverflow.com/questions/24492001/efficient-method-to-flush-cache-memory-in-arm-assembly
86. ARM Cache Memory, accessed July 7, 2025, [https://ocw.snu.ac.kr/sites/default/files/NOTE/Week%204_1.pdf](https://ocw.snu.ac.kr/sites/default/files/NOTE/Week 4_1.pdf)
87. Invalidating and cleaning cache memory (Armv7-A) - Arm Developer, accessed July 7, 2025, https://developer.arm.com/documentation/den0013/latest/Caches/Invalidating-and-cleaning-cache-memory
88. Data cache clean and flush - ARM946E-S Technical Reference Manual r1p1, accessed July 7, 2025, https://developer.arm.com/documentation/ddi0201/latest/caches/data-cache/data-cache-clean-and-flush
89. A Survey on Cache Management Mechanisms for Real-Time Embedded Systems, accessed July 7, 2025, https://www.researchgate.net/publication/283636467_A_Survey_on_Cache_Management_Mechanisms_for_Real-Time_Embedded_Systems
90. Classification of cache-partitioning approaches. (a) Overview of the... - ResearchGate, accessed July 7, 2025, https://www.researchgate.net/figure/Classification-of-cache-partitioning-approaches-a-Overview-of-the-index-based_fig1_283636467
91. How to choose between a hypervisor and a multicore framework - Tech Design Forum, accessed July 7, 2025, https://www.techdesignforums.com/practice/technique/how-to-choose-between-a-hypervisor-and-a-multicore-framework/
92. Hello, and welcome to this presentation of the STM32MP1 coprocessor management. It describes how the processors interact through - STMicroelectronics, accessed July 7, 2025, https://www.st.com/content/ccc/resource/training/technical/product_training/group1/5c/0a/4f/2b/a1/b8/4a/a5/STM32MP1-Software-Coprocessor_management_COPROC/files/STM32MP1-Software-Coprocessor_management_COPROC.pdf/_jcr_content/translations/en.STM32MP1-Software-Coprocessor_management_COPROC.pdf
93. RPMsg to Accelerate Transition Between Multi-SoC and Multi... Loïc Pallardy & Arnaud Pouliquen - YouTube, accessed July 7, 2025, https://www.youtube.com/watch?v=6yNXXSnMSS4
94. TRACE32® OS-aware Debugging - Lauterbach, accessed July 7, 2025, https://repo.lauterbach.com/rtos.html
95. Lauterbach Trace32 for SAFERTOS® | RTOS Debugger, accessed July 7, 2025, https://www.highintegritysystems.com/tools/trace32-for-safertos/
96. Debug Linux running on i.MX93EVK with Lauterbach TRACE32 - NXP Community, accessed July 7, 2025, https://community.nxp.com/t5/i-MX-Processors-Knowledge-Base/Debug-Linux-running-on-i-MX93EVK-with-Lauterbach-TRACE32/ta-p/2080847
97. TDA4VM: Lauterbach using Trace32 - Processors forum - TI E2E, accessed July 7, 2025, https://e2e.ti.com/support/processors-group/processors/f/processors-forum/1117577/tda4vm-lauterbach-using-trace32
98. Trace Compass User Guide - Overview - Eclipse archive, accessed July 7, 2025, https://archive.eclipse.org/tracecompass/doc/stable/org.eclipse.tracecompass.doc.user/Overview.html
99. Trace Compass - The Eclipse Foundation, accessed July 7, 2025, https://eclipse.dev/tracecompass/
100. tracevizlab/labs/101-analyze-system-trace-in-tracecompass/README.md at master - GitHub, accessed July 7, 2025, https://github.com/tuxology/tracevizlab/blob/master/labs/101-analyze-system-trace-in-tracecompass/README.md
101. Formal Modeling and Priority Handling Verification of Interrupt Injection in a Hypervisor, accessed July 7, 2025, https://www.researchgate.net/publication/391340830_Formal_Modeling_and_Priority_Handling_Verification_of_Interrupt_Injection_in_a_Hypervisor
102. Formal Methods for Embedded Systems - AEMY, accessed July 7, 2025, https://aemy.cs.hm.edu/topics/formal-embedded.html
103. Certify the Uncertified: Towards Assessment of Virtualization for Mixed-criticality in the Automotive Domain - arXiv, accessed July 7, 2025, https://arxiv.org/pdf/2205.12596
104. Arm CPU Security Update: Prefetcher Side Channels - Arm Developer, accessed July 7, 2025, https://developer.arm.com/documentation/110339/latest/
105. Exploiting Speculative Execution - Spectre Attacks, accessed July 7, 2025, https://spectreattack.com/spectre.pdf
106. (PDF) Virtualization Technology: Cross-VM Cache Side Channel Attacks make it Vulnerable, accessed July 7, 2025, https://www.researchgate.net/publication/303821673_Virtualization_Technology_Cross-VM_Cache_Side_Channel_Attacks_make_it_Vulnerable
107. QuanShield: Protecting against Side-Channels Attacks using Self-Destructing Enclaves, accessed July 7, 2025, https://arxiv.org/html/2312.11796v1
108. The EC2 approach to preventing side-channels - The Security Design of the AWS Nitro System, accessed July 7, 2025, https://docs.aws.amazon.com/whitepapers/latest/security-design-of-aws-nitro-system/the-ec2-approach-to-preventing-side-channels.html
109. NXP i.MX 8 Applications Processors, accessed July 7, 2025, https://www.nxp.com/docs/en/user-guide/Toradex_NXP_Webinar.pdf
110. TDA4VM Jacinto™ Processor - Texas Instruments - DigiKey, accessed July 7, 2025, https://www.digikey.com/en/product-highlight/t/texas-instruments/tda4vm-jacinto-processor
111. TDA4VM Processors datasheet (Rev. K) - Texas Instruments, accessed July 7, 2025, https://www.ti.com/lit/ds/symlink/tda4vm.pdf
112. TDA4VM data sheet, product information and support | TI.com - Texas Instruments, accessed July 7, 2025, https://www.ti.com/product/TDA4VM
113. R-Car Automotive System-on-Chips (SoCs) - Renesas, accessed July 7, 2025, https://www.renesas.com/en/products/automotive-products/automotive-system-chips-socs
114. High-End Cockpit & Infotainment Solution - Renesas, accessed July 7, 2025, https://www.renesas.com/en/applications/automotive/infotainment/high-end-cockpit-infotainment-solution
115. R-Car-H2 - Automotive System-on-Chip (SoC) for High-end Car Information Systems, accessed July 7, 2025, https://www.renesas.com/en/products/automotive-products/automotive-system-chips-socs/r-car-h2-automotive-system-chip-soc-high-end-car-information-systems
116. Renesas Electronics Offers Complete Automotive Integrated Cockpit Reference Solution Based on R-Car System on Chip, accessed July 7, 2025, https://www.renesas.com/en/about/press-room/renesas-electronics-offers-complete-automotive-integrated-cockpit-reference-solution-based-r-car
117. ISO 26262 Compliance & Tools - Parasoft, accessed July 7, 2025, https://www.parasoft.com/solutions/iso-26262/
118. Safety standards in the ARM® ecosystem, accessed July 7, 2025, https://community.arm.com/cfs-file/__key/telligent-evolution-components-attachments/01-2142-00-00-00-00-59-92/Safety-standards-in-the-ARM-ecosystem.pdf
119. VOSySmonitor - ISO 26262 ASIL C certification - Virtual Open Systems, accessed July 7, 2025, https://www.virtualopensystems.com/en/company/vosysmonitor-iso26262-asilc/
120. Functional Safety Certification and Training Program - UL Solutions, accessed July 7, 2025, https://www.ul.com/services/functional-safety-certification-and-training-program
121. ISO 26262: Functional Safety Certification Program (FSCP) | TÜV SÜD - TUV Sud, accessed July 7, 2025, https://www.tuvsud.com/en-us/industries/mobility-and-automotive/automotive-and-oem/iso-26262-functional-safety/iso-26262-functional-safety-certification-programme
122. Using OpenAMP to Address Mixed Safety-Critical Systems in Electronic Design magazine, accessed July 7, 2025, https://www.openampproject.org/news/mgc-electronic-design-article
123. 2. Architecture Overview - SOAFEE Architecture documentation, accessed July 7, 2025, https://architecture.docs.soafee.io/en/latest/contents/overview.html
124. SOAFEE: Cloud-Native Architecture for Software-Defined Vehicles – Arm®, accessed July 7, 2025, https://www.arm.com/company/success-library/made-possible/soafee
125. Scalable Open Architecture for the Embedded Edge (SOAFEE) - Arm Developer, accessed July 7, 2025, [https://developer.arm.com/Additional%20Resources/SOAFEE%20Website](https://developer.arm.com/Additional Resources/SOAFEE Website)
126. A Survey of Research into Mixed Criticality Systems - ResearchGate, accessed July 7, 2025, https://www.researchgate.net/publication/321231489_A_Survey_of_Research_into_Mixed_Criticality_Systems
127. Mixed Criticality Systems - A Review - White Rose Research Online, accessed July 7, 2025, https://eprints.whiterose.ac.uk/id/eprint/183619/1/MCS_review_v13.pdf
128. RISC-V Hypervisors - TIB AV-Portal, accessed July 7, 2025, https://av.tib.eu/media/47412
129. A First Look at RISC-V Virtualization from an Embedded Systems Perspective - Universidade do Minho, accessed July 7, 2025, [https://repositorium.sdum.uminho.pt/bitstream/1822/81552/1/2103.14951%20%281%29.pdf](https://repositorium.sdum.uminho.pt/bitstream/1822/81552/1/2103.14951 (1).pdf)
130. VOSySmonitoRV: a mixed-criticality solution on Linux-capable RISC-V platforms, accessed July 7, 2025, https://www.researchgate.net/publication/355924696_VOSySmonitoRV_a_mixed-criticality_solution_on_Linux-capable_RISC-V_platforms
131. Explorative surveying RISC-V open hardware and specifications for mixed-critical systems - SYSGO, accessed July 7, 2025, https://www.sysgo.com/fileadmin/user_upload/data/blog/SYSGO_2024-06_Mixed_critical_Systems_on_RISC_V_Platforms.pdf
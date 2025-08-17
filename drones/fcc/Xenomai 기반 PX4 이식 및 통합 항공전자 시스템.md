# Xenomai 기반 PX4 이식 및 통합 항공전자 시스템

## 요약

본 보고서는 PX4 자율비행 스택을 Xenomai 실시간 프레임워크에 이식하고, 이를 단일 보드 컴퓨터(SBC) 상에서 표준 리눅스와 함께 구동하여 비행 제어 컴퓨터(FCC)와 임무 컴퓨터(Mission Computer)의 기능을 통합하는 시스템 아키텍처에 대한 기술적 타당성, 구현 전략, 그리고 내재된 과제를 심층적으로 분석한다. 분석 결과, 이러한 통합 시스템은 기술적으로 구현 가능하며, 특히 안전이 중요한(safety-critical) 애플리케이션에서 비실시간 임무 소프트웨어로부터 실시간 비행 제어부를 보호하는 탁월한 아키텍처적 격리(isolation)를 제공함을 확인하였다. 그러나 이 아키텍처의 성공적인 구현은 Xenomai의 이중 커널(dual-kernel) 환경에 특화된 실시간 디바이스 드라이버(RTDM Driver) 개발에 상당한 공학적 투자를 요구하는 중대한 트레이드오프(trade-off)를 수반한다. 따라서 본 보고서는 제안된 아키텍처가 차세대 고성능 자율 드론 시스템을 위한 강력한 솔루션임을 인정하면서도, 그 구현은 실시간 시스템에 대한 깊은 전문성과 예방적 개발 및 검증 방법론에 대한 조직적 헌신을 전제로 해야 함을 결론으로 제시한다.

## 1.  통합 항공전자 패러다임: 비행과 임무 제어의 통합

### 1.1  분리형에서 통합형 시스템으로의 진화

전통적인 고성능 드론 아키텍처는 명확하게 분리된 두 개의 컴퓨팅 유닛으로 구성되는 것이 일반적이었다. 첫째는 NuttX와 같은 실시간 운영체제(RTOS)를 실행하는 자원 제약적인 비행 제어기(FC)이며, Pixhawk 시리즈가 그 대표적인 예시다. 둘째는 고수준의 임무를 처리하기 위해 별도로 탑재되는 동반 컴퓨터(Companion Computer)로, 주로 Raspberry Pi나 NVIDIA Jetson 같은 보드에서 리눅스를 구동한다.1 이러한 물리적 분리 구조는 비행 제어의 안정성을 보장하는 강력한 격리 수단을 제공했지만, 동시에 두 시스템 간의 통신 지연(latency), 크기, 무게, 전력(SWaP; Size, Weight, and Power)의 비효율성, 그리고 데이터 처리량의 병목 현상이라는 명백한 한계를 내포했다.

그러나 자율 시스템 기술이 발전함에 따라, 이러한 분리형 모델은 한계에 부딪히기 시작했다. 컴퓨터 비전, 시각적 관성 주행 거리 측정(VIO; Visual Inertial Odometry), 실시간 경로 계획과 같은 고급 자율 기능은 센서 데이터의 수집, 처리, 그리고 액추에이터 구동 간에 기존 모델이 제공할 수 있는 것보다 훨씬 더 긴밀한 통합과 낮은 지연 시간을 요구한다.3 LiDAR나 고해상도 카메라와 같은 센서로부터 방대한 데이터 스트림을 기체 내에서 직접 처리하여 실시간으로 의사결정을 내리고, 지상국(Ground Station)에 대한 의존도를 줄이는 것이 현대 자율 드론의 핵심 과제로 부상했다.3

이러한 시대적 요구는 단일 고성능 단일 보드 컴퓨터(SBC)가 비행 제어 컴퓨터(FCC)와 임무 컴퓨터의 역할을 동시에 수행하는 통합 항공전자 시스템의 등장을 촉진했다. 이 개념은 비행 제어기, 임무 컴퓨터, 그리고 통신 모듈을 하나의 제품으로 통합한 Auterion의 Skynode X와 같은 상용 제품을 통해 현실화되고 있다.4 이 패러다임의 전환은 단순히 하드웨어의 통합을 넘어, 자율 시스템 소프트웨어 아키텍처의 근본적인 변화를 의미한다. 이는 개발자들이 안전 필수적인 제어 루프를 위해 리눅스와 같은 범용 운영체제(GPOS)가 가진 본질적인 한계에 직면하게 만들었으며, 과거에는 일부 특수 분야의 관심사였던 실시간 기술을 고급 드론 개발의 주류 요구사항으로 끌어올렸다.

### 1.2  핵심 과제: 결정성 대 처리량

통합 아키텍처의 핵심에는 결정성(determinism)과 처리량(throughput)이라는 상충하는 두 가지 요구사항이 존재한다. 임무 컴퓨터의 워크로드, 즉 리눅스 애플리케이션(예: ROS, 컴퓨터 비전 라이브러리)은 높은 데이터 처리량에 최적화되어 있지만, 스케줄링, 메모리 관리, 복잡한 소프트웨어 스택으로 인해 본질적으로 비결정적(non-deterministic)이며 예측 불가능한 지연 시간을 가진다.3

반면, FCC의 워크로드, 즉 PX4 비행 제어 스택은 하드 실시간(hard real-time) 성능을 요구한다. 이는 센서 입력과 제어 루프 실행에 대해 예측 가능하고, 경계가 정해져 있으며, 낮은 지연 시간을 갖는 응답을 의미한다.6 비행 제어 루프에서 단 한 번의 데드라인(deadline) 위반은 기체의 불안정성을 초래하고 치명적인 고장으로 이어질 수 있다.

이러한 아키텍처 통합의 흐름은 다음과 같은 논리적 귀결을 낳는다.

1. 초기 드론 설계는 처리 요구사항이 명확히 구분되고 하드웨어 성능이 제한적이었기 때문에 FC와 동반 컴퓨터를 분리했다.1 FC는 기본 안정성을, 동반 컴퓨터는 간단한 원격 측정이나 고수준 명령을 처리했다.
2. 장애물 회피와 같은 고급 자율 기능은 Realsense 카메라 등에서 나오는 대용량 센서 데이터를 매우 낮은 지연 시간으로 처리해야 한다.1 이 원시 데이터를 FC로 보내는 것은 대역폭 한계로 불가능하며, 지상으로 보내는 것은 실시간 반응에 너무 느리다.
3. 따라서 데이터 처리는 강력한 온보드 컴퓨터에서 이루어져야 한다.3 SWaP를 최소화하기 위해 이 컴퓨터를 FC와 통합하는 것이 바람직하다.
4. 이 통합은 핵심적인 충돌을 야기한다: 높은 처리량의 '최선 노력(best-effort)' 리눅스 환경과 하드 실시간 비행 제어 루프의 공존. 만약 컴퓨터 비전 프로세스가 갑자기 CPU를 100% 점유하면, 표준 리눅스 스케줄러는 비행 루프의 데드라인을 보장할 수 없다.
5. 이는 시스템 아키텍트가 실시간 솔루션을 채택하도록 강제한다. 어떤 실시간 솔루션(PREEMPT_RT 또는 Xenomai)을 선택하는가는 이러한 하드웨어 통합 추세에 의해 직접적으로 결정되는 중요한 설계 결정이 된다. 이 맥락에서 Xenomai의 강력한 격리 접근 방식은 특히 매력적인 대안으로 부상한다.

따라서 본 보고서가 다루는 중심 질문은 '단일 하드웨어 상에서 이처럼 상반된 요구사항을 동시에 만족시키는 소프트웨어 시스템을 어떻게 설계할 것인가?'이다.

## 2.  하드 실시간 보장을 위한 Xenomai의 아키텍처 기반

### 2.1  이중 커널 철학: 비동기성의 수용

Xenomai는 표준 리눅스 커널과 나란히 실행되는 보조 커널(co-kernel) 또는 동반 코어(companion core)를 제공하는 실시간 프레임워크이다.8 Xenomai의 핵심 원리는 리눅스 커널 전체를 실시간으로 만들려고 시도하는 대신, 작고 검증 가능한 별도의 실시간 코어(**Cobalt**)를 제공하고 여기에 리눅스보다 절대적인 우선순위를 부여하는 것이다.9 즉, 리눅스 커널은 Cobalt 코어가 처리할 실시간 작업이 없을 때만 실행될 기회를 얻는다.

이러한 구조는 명확한 관심사 분리(separation of concerns)를 만들어낸다. Cobalt는 시간에 민감한 작업을 전담하고, 리눅스는 그 외 모든 일반적인 작업을 처리한다.10 이는 아키텍처적으로 두 개의 커널이 거의 비동기적으로 작동하며 각자의 작업 집합을 서비스하는 것으로 묘사될 수 있다.10

### 2.2  Dovetail 인터럽트 파이프라인: 실시간의 문지기

이러한 이중 커널 아키텍처를 가능하게 하는 핵심 메커니즘은 **Dovetail 인터럽트 파이프라인**(구 버전에서는 I-pipe로 불림)이다.11 Dovetail은 리눅스 커널에 적용되는 패치로, 높은 우선순위의 실행 단계를 도입한다. 이 파이프라인은 모든 하드웨어 인터럽트를 리눅스 커널이 인지하기 *전에* 가로챈다.14

파이프라인은 가로챈 인터럽트를 먼저 Cobalt 코-커널로 라우팅한다. 만약 Cobalt가 해당 인터럽트에 대한 핸들러를 등록해 두었다면, 인터럽트는 결정론적인 지연 시간 내에 즉시 처리된다. 그렇지 않은 경우, 인터럽트는 파이프라인을 따라 표준 리눅스 커널로 전파된다.16 이 메커니즘이야말로 리눅스의 스케줄링 지연과 비선점(non-preemptible) 구간으로부터 실시간 도메인을 보호하는 '방화벽' 역할을 수행한다.10

### 2.3  비교 분석: Xenomai/Cobalt 대 PREEMPT_RT

실시간 리눅스를 구현하는 또 다른 주요 접근 방식은 PREEMPT_RT 패치이다. 이 두 기술 간의 비교는 통합 항공전자 시스템의 아키텍처를 결정하는 데 있어 매우 중요하다.

- **PREEMPT_RT (단일 커널 접근 방식):** PREEMPT_RT 패치는 커널의 거의 모든 부분을 선점 가능하게 만들어 표준 리눅스 커널 자체를 더 결정적으로 만드는 것을 목표로 한다.18 이를 위해 스핀락(spinlock)과 같은 원시 동기화 객체를 선점 가능한 뮤텍스(mutex)로 대체한다.
- **지연 시간 및 지터(Jitter):** 벤치마크 결과는 일반적으로 Xenomai가 더 낮은 최악의 경우 지연 시간(worst-case latency)과 현저히 적은 지터(응답 시간의 변동)를 제공함을 보여준다. 하지만 잘 설정된 PREEMPT_RT 시스템은 많은 실제 사용자 공간 시나리오에서 유사한 수준의 성능을 달성할 수 있다.19 그러나 순수한 인터럽트 응답 시간 측면에서는 Xenomai의 코-커널이 여전히 우위를 점한다.22
- **결정적 차이점 - 아키텍처적 격리와 드라이버 모델:**
  - **PREEMPT_RT:** 표준 리눅스 드라이버를 재사용할 수 있다는 점은 개발 속도와 하드웨어 지원 측면에서 큰 장점이다. 그러나 이는 동시에 이 아키텍처의 약점이기도 하다. 잘못 작성되었지만 '유효한' 표준 드라이버 하나가 전체 시스템에 예측 불가능한 지연 시간을 유발하여 실시간 성능에 영향을 미칠 수 있다.17
  - **Xenomai:** **실시간 드라이버 모델(RTDM; Real-Time Driver Model)**에 맞춰 작성된 전용 실시간 드라이버를 요구한다. 이는 상당한 개발 부담을 의미한다.17 하지만 이는 실시간 작업에 사용되는 드라이버 자체가 결정적이며, 리눅스 커널이나 다른 리눅스 드라이버의 동작에 영향을 받지 않음을 보장한다.10 이것이 바로 Xenomai가 제공하는 핵심적인 트레이드오프다.

이러한 차이점을 기반으로, Xenomai를 선택하는 것은 타협 불가능하고 검증 가능한 실시간 격리를 위해 개발 속도와 하드웨어 채택의 용이성을 희생하는 전략적 결정이라고 볼 수 있다. 악명 높은 "RTDM 드라이버 문제"는 결함이 아니라, 이러한 강력한 격리를 얻기 위해 지불해야 하는 대가인 것이다. PREEMPT_RT를 사용하는 개발자는 표준 리눅스 드라이버가 종종 별다른 수정 없이 작동하기 때문에 새로운 하드웨어에서 시스템을 신속하게 구동할 수 있다.24 반면, Xenomai 개발자는 모든 핵심 주변 장치에 대한 RTDM 드라이버의 존재 여부를 확인해야 하며, 만약 없다면 이를 직접 포팅해야 하는 주요 과제에 직면한다.17 그러나 PREEMPT_RT 시스템에서는 새로 설치된 비핵심 USB 장치의 버그 있는 드라이버가 이론적으로 시스템 전반의 지연 시간을 유발하여 비행 제어기가 데드라인을 놓치게 할 수 있다. Xenomai 시스템에서는 동일한 버그 있는 드라이버가 리눅스 도메인에만 영향을 미칠 뿐, Cobalt 코어와 통신하는 PX4의 RTDM 드라이버는 별도의 보호된 실행 컨텍스트에 존재하므로 전혀 영향을 받지 않는다.10 따라서 Xenomai를 사용하는 결정은 잠재적인 시스템 오류의 한 종류를 원천적으로 제거하기 위해 맞춤형 격리 드라이버 스택을 만드는 데 엔지니어링 자원을 선투자하는 명시적인 선택이며, 이는 전형적인 안전 공학의 트레이드오프다.

#### 2.3.1 표 1: 통합 항공전자를 위한 Xenomai 대 PREEMPT_RT 비교 분석

| 기능                   | Xenomai/Cobalt                                   | PREEMPT_RT                                                   | 통합 항공전자 시스템에 대한 함의                             |
| ---------------------- | ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **코어 아키텍처**      | 이중 커널 (코-커널)                              | 단일, 완전 선점형 커널                                       | Xenomai는 실시간과 비실시간 워크로드를 하드웨어 수준에서 격리한다. |
| **실시간 보증**        | 격리를 통한 하드 실시간                          | 커널 전반의 선점을 통한 하드 실시간                          | Xenomai의 보증은 리눅스 도메인의 부하 변동에 덜 민감하다.    |
| **격리 수준**          | RT와 비-RT 도메인 간의 강력한 격리               | 약한 격리 (리눅스 드라이버가 RT 성능에 영향 가능)            | 임무 컴퓨터 소프트웨어의 오류가 비행 제어에 영향을 미치는 것을 방지하는 데 Xenomai가 월등히 우수하다. |
| **드라이버 모델**      | 맞춤형 RTDM 드라이버 필요                        | 표준 리눅스 드라이버 재사용 가능                             | Xenomai는 개발 비용과 복잡성이 훨씬 높지만, 드라이버 수준에서 결정성을 보장한다. |
| **개발 복잡성**        | 높음 (RTDM 드라이버 작성/포팅 필수)              | 중간 (표준 POSIX 개발, 시스템 튜닝 필요)                     | PREEMPT_RT는 초기 개발 속도가 빠르지만, Xenomai는 아키텍처적으로 더 엄격한 실시간 제약을 강제한다. |
| **장기 유지보수**      | 높음 (RTDM 드라이버가 메인라인에 뒤쳐질 위험 17) | 중간 (메인라인 리눅스와 더 가까워 업데이트 용이)             | Xenomai의 드라이버 스택은 별도의 유지보수 노력을 요구한다.   |
| **이상적인 사용 사례** | RT 실패가 용납되지 않는 혼합 임계 시스템         | RT와 비-RT 작업이 긴밀하게 통합되거나 개발 속도가 중요한 시스템 | 제안된 통합 FCC/임무 컴퓨터는 Xenomai의 이상적인 사용 사례에 해당한다. |

## 3.  이식성을 위한 PX4의 아키텍처 청사진

### 3.1  POSIX 기반

PX4의 핵심 설계 원칙은 다양한 운영체제에 걸친 이식성(portability)이다.26 이를 가능하게 하는 열쇠는 스레딩, 스케줄링, 파일 I/O 등 모든 OS 상호작용에 대해 표준 **POSIX API**에 의존한다는 점이다.28 PX4가 NuttX, 리눅스, macOS, QuRT 등에서 실행될 수 있는 이유는 이들 모두가 이 표준 인터페이스를 제공하기 때문이다.

이는 매우 중요한 설계 결정이다. 더 모놀리식(monolithic) 구조를 가진 Ardupilot과 달리, PX4는 POSIX에 의존함으로써 깊이 임베디드된 펌웨어라기보다는 표준 사용자 공간 애플리케이션의 집합과 유사한 형태를 띤다.28

### 3.2  uORB 메시지 버스: 모듈의 분리

PX4의 내부 통신 메커니즘은 **uORB (Micro Object Request Broker)**라는 발행-구독(publish-subscribe) 메시지 버스에 기반한다.29 모듈들은 서로를 직접 호출하지 않는다. 대신, `vehicle_attitude`와 같은 '토픽(topic)'에 데이터를 발행하고, `sensor_combined`와 같이 필요한 토픽을 구독한다. 이는 반응적(reactive)이고 비동기적이며 고도로 병렬화된 시스템을 만들어낸다.29

실행은 데이터 중심으로 이루어진다. 모듈은 일반적으로 구독 중인 토픽에 대한 업데이트를 기다렸다가, 계산을 수행하고, 그 결과를 발행한다.31 이 메커니즘은 공유 메모리를 기반으로 구현되어 매우 효율적이다.29

### 3.3  OS 추상화 계층(OSAL): 이식 인터페이스

PX4의 OS 추상화 계층(OSAL)은 POSIX 호출을 기본 운영체제에 매핑하는 얇은 코드 계층이다. 실제 이식 작업은 이 계층에 집중된다. 모듈의 실행 컨텍스트는 크게 두 가지로 나뉜다:

- **태스크(Tasks):** 자체 스레드에서 전용 스택과 우선순위(예: 리눅스의 `SCHED_FIFO`)를 가지고 실행된다.7
- **작업 큐(Work Queues):** 여러 모듈이 공유 스레드에서 실행되어 RAM과 컨텍스트 스위칭 오버헤드를 줄일 수 있다. 단, 이들은 협력적으로(co-operatively) 작동해야 하며 블로킹(blocking) 연산을 수행할 수 없다.29

PX4의 이러한 아키텍처는 단순히 '이식 가능'한 것을 넘어, Xenomai와 같은 이중 커널 환경에 거의 완벽하게 사전 적응되어 있다고 볼 수 있다. uORB 발행-구독 모델은 Xenomai/리눅스 분할을 그대로 반영하는 실시간/비실시간 경계를 가로지르는 모듈의 논리적, 물리적 분리를 가능하게 한다. 이식 작업은 전체를 한 번에 옮기는 거대한 작업이 아니라, PX4의 내재된 모듈성을 활용하여 각 모듈을 적절한 실행 도메인에 세밀하게 할당하는 과정으로 개념화될 수 있다. 예를 들어, 비행 안정성에 중요한 모듈들(`ekf2_selector`, `mc_att_control` 등)은 Xenomai 실시간 태스크로 컴파일되고, 비핵심적인 고수준 기능(복잡한 MAVLink 메시지 핸들러, 비필수 데이터 로거 등)은 표준 리눅스 프로세스로 실행될 수 있다. 이 두 모듈 집합 간의 uORB 통신은 섹션 5에서 논의될 도메인 간 IPC 메커니즘(예: 공유 메모리 기반 uORB 브리지)을 통해 구현된다. 이는 즉각적으로 드러나지 않을 수 있는 강력한 시너지 효과다.

## 4.  이식 프로세스: 기술적 구현 가이드

### 4.1  파트 A: POSIX 계층의 조화

이식의 첫 단계는 PX4 코드베이스가 Xenomai 라이브러리에 대해 컴파일되도록 만드는 것이다. 이는 PX4 빌드 시스템(CMake)을 수정하여, Xenomai를 타겟으로 하는 올바른 컴파일러 및 링커 플래그를 제공하는 `xeno-config` 유틸리티를 사용하도록 하는 것을 포함한다.32

- **스레딩 및 스케줄링:** PX4의 태스크 생성 호출(`px4_task_spawn_cmd` 31)을 Xenomai의 POSIX 스킨이 제공하는 

  `pthread_create`에 매핑한다. PX4가 리눅스에서 사용하는 실시간 스케줄링 정책(`SCHED_FIFO`)과 우선순위가 7 Xenomai 스케줄러에 의해 올바르게 요청되고 존중되는지 확인해야 한다.

- **동기화:** PX4가 사용하는 뮤텍스, 조건 변수와 같은 POSIX 동기화 기본 요소들이 실시간에 안전한 Xenomai POSIX 스킨 구현체에 직접 매핑되는지 검증한다.

#### 4.1.1 표 2: PX4 POSIX API와 Xenomai POSIX 스킨 매핑

| POSIX 기능          | PX4/표준 POSIX 호출         | Xenomai POSIX 스킨 동등물   | 구현 노트 및 주의사항                                        |
| ------------------- | --------------------------- | --------------------------- | ------------------------------------------------------------ |
| **태스크 생성**     | `pthread_create`            | `pthread_create`            | 태스크가 Xenomai의 주 모드(primary mode)에서 생성되는지 확인해야 한다. |
| **태스크 스케줄링** | `pthread_setschedparam`     | `pthread_setschedparam`     | `SCHED_FIFO` 정책과 실시간 우선순위가 Cobalt 코어에 의해 관리되는지 확인한다. |
| **뮤텍스**          | `pthread_mutex_*`           | `pthread_mutex_*`           | Xenomai의 실시간 안전 뮤텍스가 사용된다. 우선순위 상속(priority inheritance) 프로토콜 지원을 확인한다. |
| **세마포어**        | `sem_*`                     | `sem_*`                     | 실시간 컨텍스트에서 안전하게 사용 가능. 도메인 전환을 유발할 수 있는 블로킹 호출에 유의한다. |
| **조건 변수**       | `pthread_cond_*`            | `pthread_cond_*`            | 뮤텍스와 함께 사용 시, 대기 중인 태스크가 비실시간 태스크에 의해 지연되지 않도록 주의한다. |
| **타이머/지연**     | `timer_create`, `nanosleep` | `timer_create`, `nanosleep` | Xenomai의 고정밀 타이머를 사용한다. 도메인 전환을 유발하지 않는 실시간 안전 버전인지 확인한다. |

### 4.2  파트 B: I/O 과제 - RTDM을 이용한 실시간 드라이버

이것은 이식 과정에서 가장 중요하고 노동 집약적인 부분이다. SPI, I2C, UART 등을 위한 표준 리눅스 드라이버는 블로킹 호출을 사용하고 Cobalt 코어에 의해 관리되지 않기 때문에 실시간 태스크에서 사용할 수 없다.17

**실시간 드라이버 모델(RTDM)**은 Xenomai가 결정적이고 커널 공간에서 실행되는 드라이버를 만들기 위해 제공하는 프레임워크이다.11

- **RTDM 드라이버의 아키텍처 청사진:**
  - **디바이스 등록:** 드라이버는 `rtdm_dev_register`를 사용하여 RTDM 코어에 자신을 등록한다.
  - **컨텍스트별 핸들러:** 드라이버는 실시간(`_rt`) 및 비실시간(`_nrt`) 컨텍스트를 위한 별도의 핸들러(예: `open_rt`, `read_rt`, `ioctl_rt`)를 구현한다.34 이를 통해 동일한 장치를 Xenomai와 리눅스 애플리케이션 양쪽에서 접근할 수 있다.
  - **실시간 인터럽트 처리:** 드라이버는 `xnintr_attach`를 사용하여 Cobalt 코어에 인터럽트 서비스 루틴(ISR)을 등록한다.16 이 ISR은 높은 우선순위의 out-of-band 컨텍스트에서 실행되며, 매우 빠르고 비블로킹(non-blocking)이어야 한다. 일반적으로 데이터가 준비되었음을 대기 중인 실시간 태스크에 알리는 역할을 한다.
  - **데이터 전송:** 데이터 교환을 위해 `rtdm_copy_to_user` 및 `rtdm_copy_from_user`와 같은 실시간 안전 함수를 사용한다.38
- **예제 워크플로우 (가상 I2C IMU 드라이버):**
  1. 대상 IMU의 메인라인 리눅스 드라이버(예: `inv_mpu6050`)에서 시작한다.
  2. 새로운 RTDM 드라이버 스켈레톤(`rtdm_driver` 구조체)을 생성한다.
  3. 장치를 등록하기 위한 `probe` 함수를 구현한다.
  4. 리눅스 드라이버의 I2C 통신 로직을 `read_rt` 및 `write_rt` 핸들러로 포팅하고, 표준 커널 I2C 전송 함수를 RTDM 호환 동등물이나 필요한 경우 직접적인 레지스터 조작으로 대체한다. (참고: RTDM은 CAN, Serial에 대한 프로파일을 제공하지만, I2C/SPI는 종종 `spi-bcm283x-rtdm` 39과 같은 기존 RTDM SPI 드라이버를 참조하여 더 직접적이고 맞춤화된 구현이 필요하다.)
  5. IMU로부터의 "데이터 준비 완료" 인터럽트를 처리하기 위한 ISR을 구현한다. 이 ISR은 `rtdm_read()`를 호출한 PX4 센서 모듈 태스크를 깨우게 된다.

### 4.3  파트 C: 시스템 컴파일 및 배포

단계별 워크플로우는 다음과 같다.

1. 특정 Dovetail 패치와 호환되는 순정 리눅스 커널 소스 트리를 다운로드한다.33
2. Xenomai 소스 코드를 다운로드한다.
3. `prepare-kernel.sh` 스크립트를 실행하여 커널 소스에 Dovetail 패치를 적용한다.42
4. `make menuconfig`를 통해 커널을 설정한다. Xenomai/Cobalt 옵션, RTDM 드라이버를 활성화하고, 지연 시간을 악화시키는 기능(예: CPU 주파수 스케일링, 깊은 CPU 유휴 상태)을 비활성화한다.33
5. 커널과 모듈을 컴파일한다.
6. 새 커널을 설치한다.
7. Xenomai 사용자 공간 라이브러리와 도구를 설정하고 컴파일한다.33
8. 마지막으로, 수정된 PX4 소스 코드를 Xenomai 라이브러리에 대해 링크하여 컴파일한다.

## 5.  도메인 간 통신 아키텍처 설계

### 5.1  데이터 브리지의 필요성

PX4가 실시간 도메인에서 실행되고, 컴퓨터 비전이나 고수준 임무 계획을 위한 ROS2 노드와 같은 다른 애플리케이션들이 비실시간 리눅스 도메인에서 실행됨에 따라, 두 도메인 사이에는 견고하고 낮은 지연 시간을 갖는 통신 브리지가 필수적이다.45 이 섹션에서는 이 브리지를 위한 솔루션을 설계한다.

### 5.2  고대역폭 데이터: POSIX 공유 메모리

VIO나 로깅을 위해 리눅스 프로세스에서 소비해야 하는 IMU 측정값이나 자세 추정치와 같은 고주파 데이터의 경우, 공유 메모리가 가장 효율적인 방법이다. Xenomai의 POSIX 스킨은 다음과 같은 공유 메모리 API를 제공한다:

- `shm_open()`: 이름 있는 공유 메모리 객체를 생성하거나 연다.46
- `ftruncate()`: 메모리 객체의 크기를 설정한다.46
- `mmap()`: 객체를 실시간 태스크와 리눅스 프로세스 양쪽의 주소 공간에 매핑한다.46

매핑이 완료되면, 양쪽 모두 동일한 물리적 메모리를 가리키는 포인터를 갖게 되어 복사 없는(zero-copy) 데이터 전송이 가능해진다. 접근을 관리하고 경쟁 상태(race condition)를 방지하기 위해서는 실시간 안전 뮤텍스나 세마포어와 같은 동기화 메커니즘이 공유 메모리 영역 자체에 위치해야 한다.46

### 5.3  명령 및 제어: RTIPC 소켓

리눅스 프로세스에서 PX4 모듈로 명령을 보내는 것과 같은 메시지 기반 통신을 위해, Xenomai는 소켓 기반 API인 **RTIPC (Real-Time Inter-Process Communication)**를 제공한다.49 주요 프로토콜은 다음과 같다:

- **IDDP (Intra-Domain Datagram Protocol):** 두 실시간 태스크 간의 통신용.
- **XDDP (Cross-Domain Datagram Protocol):** 이 사용 사례의 핵심 프로토콜로, 실시간 태스크(Cobalt 도메인)와 표준 리눅스 프로세스(보조 도메인) 간의 통신을 허용한다.49

이 API는 표준 버클리 소켓 API(`socket()`, `bind()`, `sendto()`, `recvfrom()`)를 모방하여 네트워크 프로그래머에게 익숙하다.49 이는 이벤트 기반 통신을 위해 공유 메모리보다 더 구조화되고 견고한 대안을 제공한다.

잘 설계된 도메인 간 브리지는 단순한 데이터 파이프가 아니라 실시간 시스템을 위한 '충격 흡수 장치' 역할을 한다. IPC 메커니즘을 신중하게 선택함으로써, 아키텍트는 실시간 도메인과 비실시간 도메인 간의 결합(coupling) 정도를 제어할 수 있다. 예를 들어, IMU 데이터를 소비하는 리눅스 기반 VIO 알고리즘을 생각해보자. 만약 이 알고리즘이 블로킹 `recvfrom()` 호출을 사용하고 VIO 프로세스가 멈춘다면, 데이터를 `sendto()` 하려는 실시간 PX4 태스크가 잠재적으로 블록될 수 있다. 그러나 공유 메모리 링 버퍼 접근 방식을 사용하면, 실시간 PX4 태스크는 링 버퍼에 IMU 데이터를 계속 덮어쓰기만 할 뿐, VIO 프로세스가 멈추더라도 전혀 영향을 받지 않고 블록되지 않는다. 실시간 도메인은 완전히 분리되고 결정적인 상태를 유지한다. 이는 IPC 메커니즘의 선택과 구현 자체가 시스템의 실시간 보증을 확보하는 데 있어 중요한 부분임을 보여준다.

#### 5.3.1 표 3: 도메인 간 통신 방법 비교 분석

| 메커니즘              | 지연 시간               | 처리량    | 오버헤드  | 동기화                  | 대표 사용 사례                                   |
| --------------------- | ----------------------- | --------- | --------- | ----------------------- | ------------------------------------------------ |
| **POSIX 공유 메모리** | 극도로 낮음 (복사 없음) | 매우 높음 | 매우 낮음 | 수동 (사용자 구현 필요) | 스트리밍 센서 데이터, 비디오 프레임, 상태 벡터   |
| **RTIPC (XDDP 소켓)** | 낮음 (커널 버퍼링 포함) | 높음      | 낮음      | API에 의해 관리됨       | 이벤트 기반 명령, 임무 웨이포인트, 상태 업데이트 |

## 6.  현장의 현실: 디버깅, 검증, 그리고 함정

### 6.1  Out-of-Band 디버깅의 어려움

표준 디버깅 도구는 Xenomai의 실시간 컨텍스트에서 문제가 될 수 있다. out-of-band 컨텍스트(예: RTDM ISR)에서 `printk()`를 호출하면 출력이 지연되거나, 고주파 상황에서는 콘솔 드라이버를 압도하여 시스템 정지를 유발할 수 있다.51

- **필수 기법:**
  - **`raw_printk()`:** 임계 구역에서 저수준의, 직렬 콘솔로 직접 출력하는 디버깅에 사용한다.51
  - **커널 디버그 플래그:** 개발 중에는 `CONFIG_DEBUG_IRQ_PIPELINE`과 `CONFIG_DEBUG_DOVETAIL`을 활성화한다. 이들은 실시간 컨텍스트에서 비재진입(non-reentrant) 리눅스 서비스를 호출하는 것과 같은 불법적인 연산을 잡아내는 런타임 검사를 추가한다.52
  - **하드웨어 도구:** 해결하기 어려운 문제에 대해서는 JTAG 디버거나 로직 애널라이저가 매우 유용하다.53

### 6.2  조용한 살인자: 의도치 않은 도메인 전환

가장 위험한 버그는 시스템 충돌이 아니라 성능 저하이다. 실시간 태스크가 불법적인 시스템 호출(예: 표준 파일 I/O, `malloc`)을 하면 비실시간 리눅스 도메인으로 조용히 '강등'될 수 있다.54

- **탐지 및 예방:**

  - **코드 규율:** 실시간 태스크에 필요한 모든 메모리는 임계 루프에 진입하기 전에 정적으로 할당한다. `mlockall()`을 사용하여 메모리 페이지를 잠가 스와핑(swapping)을 방지한다.54

  - **모니터링:** Xenomai의 `/proc/xenomai/sched/threads` 인터페이스를 사용하여 각 실시간 태스크의 "MSW"(Mode Switches) 카운트를 모니터링한다. 증가하는 MSW 카운트는 문제의 명백한 신호이다.48 또한 Xenomai는 전환이 발생할 때 

    `SIGXCPU` 신호를 보내도록 설정할 수 있으며, 이는 디버거로 잡을 수 있다.54

### 6.3  성능 검증 및 벤치마킹

최종 단계는 시스템이 실시간 데드라인을 충족함을 증명하는 것이다. Xenomai 테스트 스위트의 `latency` 테스트를 사용하여 부하 상태에서 시스템의 인터럽트 및 스케줄링 지연 시간의 기준선을 측정한다.13 목표는 평균 지연 시간이 아니라 

**최악의 경우 지연 시간**을 찾는 것이다. 이를 위해서는 리눅스 도메인에 커널 컴파일, 네트워크 트래픽, 디스크 I/O와 같은 무겁고 예측 불가능한 스트레스를 가하면서 장시간(수 시간 또는 수 일) 테스트를 실행하여 병적인 사례를 발견해야 한다.55

테스트 중에는 SMI(시스템 관리 인터럽트), CPU 주파수 스케일링, 깊은 유휴 상태와 같이 상당하고 예측 불가능한 지연 시간을 유발할 수 있는 비결정성의 원인들을 비활성화해야 한다.44

성공적인 Xenomai 프로젝트는 '실시간 우선' 개발 문화를 요구한다. 디버깅과 검증은 나중의 일이 될 수 없으며, 개발 첫날부터 프로세스에 통합되어야 한다. 디버깅의 어려움은 코딩에 있어 더 엄격하고 예방적인 접근을 필요로 한다. 표준 리눅스 개발자는 GDB, Valgrind와 같은 풍부한 디버깅 도구에 익숙하지만, 이들 중 다수는 Xenomai의 주 실시간 도메인에서 사용할 수 없거나 신뢰할 수 없다.51 실시간 태스크의 버그는 충돌을 일으키기보다는 미묘한 지터 증가를 유발할 수 있어 사후 추적이 매우 어렵다. 따라서 개발 프로세스는 '작성 후 디버그'에서 '검증 가능성을 고려한 설계'로 전환되어야 한다. 이는 비실시간 안전 호출을 찾기 위한 엄격한 코드 검토, 모든 개발 빌드에서 커널 디버그 플래그의 의무적 사용, 스트레스 하에서 지연 시간 테스트를 포함하는 지속적인 통합을 의미한다. 팀은 실시간 컨텍스트 내에서 '금지된' 연산(동적 메모리 할당, 블로킹 I/O 등) 목록을 내재화해야 한다. 이는 기술적인 변화일 뿐만 아니라 문화적, 프로세스 수준의 변화이다.

## 7.  결론 및 향후 방향

### 7.1. 분석 결과 요약

본 보고서는 제안된 통합 항공전자 아키텍처가 기술적으로 실현 가능할 뿐만 아니라, 차세대 고도의 자율 드론을 구축하기 위한 견고한 솔루션임을 결론 내린다. 핵심적인 트레이드오프는 탁월한 실시간 격리와 시스템 안전성을 확보하는 대가로 RTDM 드라이버 개발에 상당한 엔지니어링 투자가 필요하다는 점이다. 프로젝트의 성공은 실시간 시스템에 대한 팀의 전문성과 엄격하고 예방적인 개발 및 검증 방법론에 대한 헌신에 달려 있다.

### 7.1  최종 권장 사항

- 임무 소프트웨어의 실패가 비행에 영향을 미쳐서는 안 되는 안전 필수 시스템을 구축하는 팀에게는, PREEMPT_RT보다 Xenomai 이중 커널 접근 방식이 강력하게 권장된다.
- 프로젝트 계획은 RTDM 드라이버의 개발, 포팅, 그리고 장기적인 유지보수에 상당한 시간과 자원을 할당해야 한다.
- 검증 가능성과 예방적 코딩 관행을 우선시하는 '실시간 우선' 개발 문화를 채택해야 한다.

### 7.2  향후 방향: Xenomai 4와 EVL 코어

마지막으로, Xenomai 프레임워크의 가장 큰 고충인 RTDM 드라이버 모델을 해결하기 위한 차세대 Xenomai의 방향을 조망할 필요가 있다.10 Xenomai 4의 **EVL (Even Less) 코어**는 동일한 Dovetail 파이프라인 위에 구축되었으며, 메인라인 커널과의 긴밀한 통합을 통해 완전히 분리된 포크(fork) 드라이버의 필요성을 줄이는 것을 장기적인 목표로 한다.10

현재 프로젝트를 Xenomai 3 기반으로 구축하는 것은 견고한 토대를 마련하는 것이며, 향후 Xenomai 4/EVL로의 마이그레이션 경로는 맞춤형 드라이버 스택의 장기적인 유지보수 부담을 크게 줄일 수 있는 전략적인 전망을 제공한다. 이는 기술 선택에 대한 전략적 통찰을 제공하며, 현재의 높은 개발 비용이 미래의 유지보수 효율성 향상으로 이어질 수 있음을 시사한다.

#### **참고 자료**

1. PX4 System Architecture | PX4 Guide (main), accessed July 7, 2025, https://docs.px4.io/main/en/concept/px4_systems_architecture.html
2. Drone Flight Controller and Carrier Boards - Unmanned Systems Technology, accessed July 7, 2025, https://www.unmannedsystemstechnology.com/expo/drone-flight-controller/
3. SBC powered drones for aerial inspection - Farnell, accessed July 7, 2025, https://fr.farnell.com/sbc-powered-drones-for-aerial-inspection-trc-ar
4. Skynode X – The Best All-in-One Solution For Autonomous Robots - Auterion, accessed July 7, 2025, https://auterion.com/product/skynode-x/
5. Drone Apps & APIs | PX4 Guide (main), accessed July 7, 2025, https://docs.px4.io/main/en/robotics/
6. A synchronous generalize method for preserving the quality of data for embedded real-time systems: the autopilot PX4-RT case - ResearchGate, accessed July 7, 2025, https://www.researchgate.net/publication/358233973_A_synchronous_generalize_method_for_preserving_the_quality_of_data_for_embedded_real-time_systems_the_autopilot_PX4-RT_case
7. PX4 process isolation on Linux boards / Issue #16140 / PX4/PX4-Autopilot - GitHub, accessed July 7, 2025, https://github.com/PX4/PX4-Autopilot/issues/16140
8. Mastering Xenomai for Real-Time Embedded Systems - Number Analytics, accessed July 7, 2025, https://www.numberanalytics.com/blog/mastering-xenomai-real-time-embedded-systems
9. Overview :: Xenomai 3, accessed July 7, 2025, https://v3.xenomai.org/overview/
10. Overview - Xenomai 4, accessed July 7, 2025, https://evlproject.org/overview/
11. Xenomai - Product wiki, accessed July 7, 2025, https://wiki.emacinc.com/wiki/Xenomai
12. Xenomai 2 architecture, accessed July 7, 2025, https://v3.xenomai.org/introduction/
13. Xenomai Packages - Raspberry Pi Forums, accessed July 7, 2025, https://forums.raspberrypi.com/viewtopic.php?t=74686
14. The Xenomai Kernal / GSoC2021: Bela Platform on BBAI - GitHub Pages, accessed July 7, 2025, https://dhruvag2000.github.io/Blog-GSoC21/xenomai/
15. Xenomai on the 96Boards, accessed July 7, 2025, https://www.96boards.org/blog/xenomai-on-the-96boards/
16. Xenomai: Interrupt management, accessed July 7, 2025, https://www.cs.ru.nl/lab/xenomai/api3/group__cobalt__core__irq.html
17. Real-time I/O drivers - Xenomai 4, accessed July 7, 2025, https://v4.xenomai.org/core/drivers/
18. Xenomai & Real-Time Linux - Siemens Open Source, accessed July 7, 2025, https://opensource.siemens.com/events/2022/slides/Jan_Kiszka_and_Florian_Bezdeka_Xenomai_and_Real-Time_Linux-Driving_OSS_Projects_for_Siemens.pdf
19. Performance Assessment of Linux Kernels with PREEMPT_RT on ..., accessed July 7, 2025, https://www.mdpi.com/2079-9292/10/11/1331
20. Real-time Linux explained, and contrasted with Xenomai and RTAI - LinuxGizmos.com, accessed July 7, 2025, https://linuxgizmos.com/real-time-linux-explained/
21. The Real-Time Linux Kernel: A Survey on PREEMPT_RT, accessed July 7, 2025, https://re.public.polimi.it/retrieve/e0c31c12-9844-4599-e053-1705fe0aef77/11311-1076057_Reghenzani.pdf
22. A Comparison of Real-Time Linux-Based Architectures for Embedded Musical Applications, accessed July 7, 2025, https://www.researchgate.net/publication/358135895_A_Comparison_of_Real-Time_Linux-Based_Architectures_for_Embedded_Musical_Applications
23. Performance Evaluation of GNU/Linux for Real-Time Applications - DiVA portal, accessed July 7, 2025, https://www.diva-portal.org/smash/get/diva2:158978/FULLTEXT01.pdfGoPro
24. RT preempt vs RTAI vs Xenomai for real-time linux - Stack Overflow, accessed July 7, 2025, https://stackoverflow.com/questions/31109364/rt-preempt-vs-rtai-vs-xenomai-for-real-time-linux
25. linux-realtime/doc/RealTimeLinux.md at main - GitHub, accessed July 7, 2025, https://github.com/2b-t/docker-realtime/blob/main/doc/RealTimeLinux.md
26. Flight Controller Porting Guide | PX4 Guide (main) - PX4 docs, accessed July 7, 2025, https://docs.px4.io/main/en/hardware/porting_guide.html
27. PX4 Autopilot Software - GitHub, accessed July 7, 2025, https://github.com/PX4/PX4-Autopilot
28. PX4 ChibiOS portd - PX4 Autopilot - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 7, 2025, https://discuss.px4.io/t/px4-chibios-portd/5821
29. PX4 Architectural Overview | PX4 Guide (main) - PX4 docs, accessed July 7, 2025, https://docs.px4.io/main/en/concept/architecture.html
30. PX4 Flight Stack Structure Proposal / Issue #7318 / PX4/PX4-Autopilot - GitHub, accessed July 7, 2025, https://github.com/PX4/Firmware/issues/7318
31. Main loop and master schedule - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 7, 2025, https://discuss.px4.io/t/main-loop-and-master-schedule/7731
32. Migrating to Xenomai 3, accessed July 7, 2025, https://v3.xenomai.org/legacy/migrating-to-xenomai3/
33. Real-Time Programming with Xenomai 3 - Part 1: Installation and Basic Setup, accessed July 7, 2025, https://www.ashwinnarayan.com/post/xenomai-realtime-programming/
34. The Real-Time Driver Model and First Applications - Institute for Computing and Information Sciences, accessed July 7, 2025, https://www.cs.ru.nl/lab/xenomai/RTDM-and-Applications.pdf
35. The Xenomai Project, a Linux Based RTOS - Open Source For You, accessed July 7, 2025, https://www.opensourceforu.com/2015/10/the-xenomai-project-a-linux-based-rtos/
36. Xenomai: Hard Real Time Driver Example Tutorial with MMAP using the RTDM (Real Time Driver Model) 转载 - CSDN博客, accessed July 7, 2025, https://blog.csdn.net/bitrain/article/details/22138043
37. 2008-11-07 05:02:49 Xenomai and user application - Documents - Linux Forum Archive, accessed July 7, 2025, https://ez.analog.com/dsp/software-and-development-tools/linux-blackfin/linux-forum-archive/w/documents/12019/2008-11-07-05-02-49-xenomai-and-user-application
38. Utility Services - RTDM - Xenomai 3 documentation, accessed July 7, 2025, https://doc.xenomai.org/v3/html/xeno3prm/group__rtdm__util.html
39. elk-audio/rpi-shiftreg-rtdm-driver: Xenomai real-time driver ... - GitHub, accessed July 7, 2025, https://github.com/elk-audio/rpi-shiftreg-rtdm-driver
40. Xenomai-Implementing a RTOS emulation framework on GNU/Linux - Ask Ubuntu, accessed July 7, 2025, https://askubuntu.com/questions/1399611/xenomai-implementing-a-rtos-emulation-framework-on-gnu-linux
41. How to install Xenomai?, accessed July 7, 2025, https://nrotella.github.io/download/Rotella_XenomaiGuide.pdf
42. Installing Xenomai 3, accessed July 7, 2025, https://v3.xenomai.org/installation/
43. Installing Xenomai 3.x, accessed July 7, 2025, https://doc.xenomai.org/v3/html/README.INSTALL/index.html
44. xenomai/TROUBLESHOOTING at master - GitHub, accessed July 7, 2025, https://github.com/JackieXie168/xenomai/blob/master/TROUBLESHOOTING
45. ROS2 – Xenomai4 Real-Time Framework on Raspberry Pi, accessed July 7, 2025, http://essay.utwente.nl/104808/1/Raoudi_MA_EEMCS.pdf
46. Xenomai API: Shared memory services., accessed July 7, 2025, http://www.cs.ru.nl/lab/xenomai/api2.4/group__posix__shm.html
47. How to use mmap&proc shared memory between kernel and userspace - Stack Overflow, accessed July 7, 2025, https://stackoverflow.com/questions/36762974/how-to-use-mmapproc-shared-memory-between-kernel-and-userspace
48. Best method to access shared memory remotely - Power PMAC - OMRON Forums, accessed July 7, 2025, https://forums.automation.omron.com/topic/5630-best-method-to-access-shared-memory-remotely/
49. Xenomai: Real-time IPC, accessed July 7, 2025, https://www.cs.ru.nl/lab/xenomai/api3/group__rtdm__ipc.html
50. Programming with RTnet - Xenomai 3, accessed July 7, 2025, https://v3.xenomai.org/rtnet/programming/
51. Raw printk support - Xenomai 4, accessed July 7, 2025, https://v4.xenomai.org/dovetail/porting/rawprintk/
52. Developer Notes - Xenomai 4, accessed July 7, 2025, https://v4.xenomai.org/dovetail/porting/devnotes/
53. Real Time Linux and Xenomai - First Technology Transfer, accessed July 7, 2025, https://ftt.co.uk/Xenomai_Real_Time_Linux.php
54. Development of a real-time application based on Xenomai - Webthesis, accessed July 7, 2025, https://webthesis.biblio.polito.it/11009/1/tesi.pdf
55. Running benchmarks - Xenomai 4, accessed July 7, 2025, https://evlproject.org/core/benchmarks/
56. By doing so, having some random in-band work evicting cache lines on a CPU where real-time threads briefly sleep is less likely, increasing the odds of costly cache misses, which translates positively into the latency numbers you can get. Even if EVL's small footprint core has a limited exposure to such kind of disturbance, saving a handful of microseconds is worth it when the worst case figure is already within tenths of microseconds. - :: Xenomai 4, accessed July 7, 2025, https://v4.xenomai.org/core/caveat/




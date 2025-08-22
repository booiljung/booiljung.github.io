---
layout: page
title: Lock-Free Queue의 원리
permalink: /systems/process and thread/Lock-Free Queue의 원리
---


현대의 멀티코어 프로세서 환경에서 병렬 프로그래밍은 더 이상 선택이 아닌 필수가 되었다. 다수의 스레드가 공유 자원에 안전하게 접근하도록 보장하기 위해 전통적으로 사용된 동기화 기법은 '락(Lock)'이다. 하지만 락은 그 자체로 성능 저하의 주된 원인이 되며, 시스템의 확장성을 심각하게 저해할 수 있다. 락으로 인해 발생하는 병목 현상, 컨텍스트 스위칭 오버헤드, 우선순위 역전, 데드락과 같은 문제들은 고성능 컴퓨팅 환경에서 치명적인 단점으로 작용한다.1

이러한 한계를 극복하기 위해 등장한 패러다임이 바로 '락프리(Lock-Free)' 프로그래밍이다. 락프리 자료구조는 락을 사용하지 않고도 여러 스레드가 안전하게 데이터를 공유할 수 있도록 설계된 정교한 알고리즘의 집합체다. 그중에서도 '락프리 큐(Lock-Free Queue)'는 생산자-소비자(Producer-Consumer) 패턴과 같은 비동기 작업 처리 시스템의 핵심 구성 요소로서, 스레드 간 데이터 교환을 위한 필수적인 역할을 수행한다.4 락프리 큐를 사용하면 높은 부하 상황에서도 안정적인 성능을 유지하며, 시스템 전체의 응답성과 처리량을 극대화할 수 있다.5

그러나 락프리 프로그래밍의 세계는 복잡하고 미묘하다. 단순히 뮤텍스를 제거하는 것 이상의 깊은 이해를 요구한다. 하드웨어 수준의 원자적 연산(Atomic Operation), CPU와 컴파일러의 메모리 재배치 문제, 그리고 ABA 문제와 같은 까다로운 난제들을 해결해야만 비로소 정확하고 효율적인 락프리 자료구조를 구현할 수 있다.4

본 보고서는 락프리 큐의 근본적인 원리를 심층적으로 탐구하는 것을 목표로 한다. 먼저 1부에서는 락 기반 동기화의 문제점과 락프리 프로그래밍이 필요한 이유를 살펴보고, 이를 가능하게 하는 핵심 원리인 비차단(Non-Blocking) 진행 보장, 원자적 연산, 그리고 메모리 모델에 대해 상세히 다룬다. 2부에서는 가장 대표적인 락프리 큐 알고리즘인 '마이클-스캇(Michael-Scott) MPMC 큐'와 'SPSC 링 버퍼 큐'의 내부 동작을 단계별로 분석한다. 3부에서는 락프리 구현 시 발생하는 가장 큰 난제인 ABA 문제와 그 해결책을 심도 있게 논의한다. 마지막으로 4부에서는 C++, Java, Rust, 그리고 리눅스 커널 등 실제 산업 현장에서 널리 사용되는 락프리 큐 구현체들을 비교 분석하며, 이론이 실제 어떻게 적용되는지 살펴본다. 이 보고서를 통해 독자는 락프리 큐의 동작 원리에 대한 깊이 있는 이해와 함께, 실제 시스템 설계에 적용할 수 있는 통찰을 얻게 될 것이다.

------


이 파트에서는 락프리 프로그래밍이 왜 필요한지, 그리고 이를 구현하기 위한 근본적인 도구들이 무엇인지에 대한 이론적 토대를 구축한다.


이 장에서는 락 기반 동기화가 가진 성능 및 정확성 측면의 한계를 상세히 분석하고, 락프리 설계가 필요한 근본적인 동기를 제시한다.


락(뮤텍스, 세마포어 등)은 공유 자원에 대한 동시 접근을 제어하여 데이터의 정합성을 보장하는 가장 기본적인 도구다. 그러나 이 직관적인 해결책은 고도의 동시성이 요구되는 환경에서 여러 심각한 문제를 야기한다.3

- **성능 오버헤드:** 락을 획득하고 해제하는 과정은 결코 가볍지 않다. 이 작업은 종종 사용자 모드(User Mode)에서 커널 모드(Kernel Mode)로의 전환을 필요로 하는 시스템 호출을 동반하며, 이는 상당한 CPU 사이클을 소모하는 비싼 연산이다.2 특히 성능에 민감한 코드 루프 내에서 락을 빈번하게 사용한다면, 이 오버헤드만으로도 프로그램 전체의 성능이 저하될 수 있다.3
- **확장성 병목:** 락의 가장 근본적인 문제는 실행을 직렬화(serialize)한다는 점이다. 한 스레드가 락을 소유하고 있는 동안, 동일한 자원에 접근하려는 다른 모든 스레드는 실행을 멈추고 대기(blocking)해야 한다.1 이는 멀티코어 프로세서의 이점을 무력화시키며, 부하가 증가할수록 스레드들이 락을 얻기 위해 경쟁하는 시간이 늘어나 시스템 전체의 처리량이 오히려 감소하는 스케일링 문제를 야기한다.5 심각한 경우, 락을 사용하는 멀티스레드 프로그램이 싱글스레드 버전보다 느려지는 역효과가 발생하기도 한다.3
- **시스템 전체의 정체 및 예측 불가능성:** 락으로 인한 문제는 단순히 성능 저하에 그치지 않고, 시스템 전체의 안정성을 위협할 수 있다.
  - **우선순위 역전 (Priority Inversion):** 낮은 우선순위의 스레드가 락을 점유한 상태에서 운영체제 스케줄러에 의해 선점(preemption)되면, 해당 락을 기다리는 높은 우선순위의 스레드가 무기한 대기하는 상황이 발생할 수 있다. 이는 실시간 시스템에서 치명적인 오류로 이어질 수 있다.1
  - **호위 효과 (Convoy Effect):** 락을 보유한 스레드가 타임 슬라이스 만료 등의 이유로 스케줄링에서 제외되면, 그 락을 기다리던 다른 모든 스레드들이 마치 호송단처럼 줄지어 대기하며 아무런 작업을 수행하지 못하게 된다. 이들 중 높은 우선순위의 스레드가 있더라도 속수무책이다.1
  - **데드락 (Deadlock) / 라이브락 (Livelock):** 두 개 이상의 스레드가 서로가 점유한 락을 기다리며 무한 대기에 빠지는 데드락은 락 기반 프로그래밍의 고질적인 문제다. 프로그래머의 부주의로 인해 발생하지만, 락을 여러 개 사용하는 복잡한 시스템에서는 잠재적인 위험 요소로 항상 존재한다.3


락프리 프로그래밍은 이러한 블로킹 동기화 프리미티브를 제거함으로써 위 문제들을 해결하고자 한다.1 락프리 알고리즘의 핵심적인 약속은 **'하나의 스레드를 일시 중단시켜도 다른 스레드들의 전체적인 진행을 방해하지 않는다'**는 것이다.8 이는 예측 불가능한 높은 부하 상황에서도 시스템이 멈추지 않고 안정적으로 동작하도록 보장하며, 뛰어난 성능을 제공한다.5

"락프리"라는 용어는 단순히 '뮤텍스를 사용하지 않음'보다 훨씬 심오한 의미를 내포한다. 이는 시스템 전체의 진행(progress)에 대한 보증이다. 락프리의 순수한 정의는 '뮤텍스가 없다'는 구현 방식에 대한 설명이 아니다.6 학술적이고 실용적인 정의는 더 엄격하다. 어떤 알고리즘이 락프리라고 불리기 위해서는, 주어진 시간 내에 **적어도 하나의 스레드가 자신의 작업을 완료하고 앞으로 나아가는 것** 이 보장되어야 한다.5

이는 락 기반 동기화와의 근본적인 차이점이다. 락프리 알고리즘 내에서 한 스레드가 자신의 연산을 계속해서 재시도하며 실패할 수 있다. 하지만 그 실패는 다른 스레드가 '경쟁에서 이겨' 작업을 성공적으로 마쳤다는 것을 의미한다. 즉, 시스템 전체적으로는 누군가가 계속해서 진전을 이루고 있는 것이다. 반면, 락 기반 시나리오에서는 락을 소유한 스레드가 선점당하면 그 락을 기다리는 모든 스레드의 진행이 완전히 멈춘다.7 따라서 '락프리'에서 '락'은 뮤텍스 객체뿐만 아니라, 최악의 스케줄링 상황에서도 애플리케이션 전체를 '잠글(lock up)' 수 있는 모든 가능성을 배제한다는 의미를 담고 있다.6 이 특성 때문에 락프리 알고리즘은 예측 가능한 진행이 필수적인 고가용성 및 실시간 시스템에서 핵심적인 기술로 간주된다.7


비차단 알고리즘은 보장하는 진행 수준에 따라 여러 등급으로 나뉜다. 이 등급을 정확히 이해하는 것은 락프리 자료구조의 특성과 트레이드오프를 파악하는 데 매우 중요하다.

- **장애물 없음 (Obstruction-Freedom):** 가장 약한 수준의 비차단 보장이다. 어떤 스레드가 수행하는 연산이, 그 연산을 방해하는 다른 모든 스레드가 일시 중단(suspend)된다면 유한한 단계 내에 완료되는 것을 보장한다.9 이는 이론적으로 유용한 속성이지만, 실제 경쟁 상황에서는 진행을 보장하지 않기 때문에 실용성은 다소 떨어진다.
- **락프리 (Lock-Freedom):** 가장 일반적이고 실용적인 수준의 보장이다. 알고리즘의 여러 스레드가 동시에 작업을 수행할 때, 유한한 시간 단계마다 **적어도 하나의 스레드**가 자신의 작업을 완료하는 것을 보장한다.3 이는 시스템 전체의 처리량(throughput)을 보장하지만, 개별 스레드의 기아(starvation) 상태를 허용한다. 즉, 운이 나쁜 특정 스레드는 다른 스레드들이 계속해서 성공하는 동안 영원히 재시도만 반복할 수 있다.9
- **대기 없음 (Wait-Freedom):** 가장 강력한 수준의 보장이다. **모든 스레드**가 다른 스레드의 실행 속도나 상태와 관계없이 각각 유한한 단계 내에 자신의 작업을 완료하는 것을 보장한다.1 이는 시스템 전체의 진행과 개별 스레드의 기아 방지를 모두 보장하는 가장 이상적인 상태다. 하지만 대기 없는 알고리즘은 설계가 극도로 어렵고, 보통 단일 생산자-단일 소비자(SPSC) 큐와 같이 매우 제한적인 상황에서만 구현이 가능하다.1

이러한 보장 수준의 미묘하지만 중요한 차이점은 다음 표를 통해 명확하게 비교할 수 있다. 이 표는 보고서 전반에 걸쳐 다양한 큐 알고리즘을 분류하고 평가하는 기준점으로 활용될 것이다.

| 보장 수준                             | 정의                               | 시스템 전체 진행 | 개별 스레드 기아 | 실용성 및 주요 사용 사례                  |
| ------------------------------------- | ---------------------------------- | ---------------- | ---------------- | ----------------------------------------- |
| **장애물 없음 (Obstruction-Freedom)** | 다른 스레드의 방해가 없으면 완료됨 | 보장 안 됨       | 가능             | 이론적 기반, 실제 적용 사례 적음          |
| **락프리 (Lock-Freedom)**             | 항상 최소 하나의 스레드는 완료됨   | 보장됨           | 가능             | 범용 MPMC 자료구조 (예: Michael-Scott 큐) |
| **대기 없음 (Wait-Freedom)**          | 항상 모든 스레드가 각자 완료됨     | 보장됨           | 불가능           | 고도로 특화된 SPSC 큐, 하드 실시간 시스템 |


대부분의 락프리 자료구조는 하드웨어가 제공하는 특별한 원자적 명령어를 기반으로 구축된다.5 그중 가장 핵심적인 역할을 하는 것이 바로  **Compare-And-Swap (CAS)** 연산이다.4


CAS는 '값을 비교하고 교환하는' 작업을 단 하나의, 분할 불가능한(atomic) 명령으로 처리하는 CPU 명령어다.12 이는 멀티스레드 환경에서 고질적인 문제인 '확인 후 실행(check-then-act)' 패턴의 경쟁 상태(race condition)를 원천적으로 해결한다.13 CAS의 동작 방식은 다음과 같다 12:

1. 세 개의 인자(피연산자)를 받는다: 메모리 주소 `p`, 예상되는 기존 값 `old`, 그리고 새로 바꿀 값 `new`.
2. **원자적으로** 다음 과정을 수행한다:
   - 메모리 주소 `p`에서 현재 값을 읽는다.
   - 읽은 값이 예상 값 `old`와 일치하는지 비교한다.
   - **일치할 경우:** `p`의 값을 `new`로 교체하고, 성공했다는 의미로 `true`를 반환한다.
   - **일치하지 않을 경우:** 메모리 값을 변경하지 않고, 실패했다는 의미로 `false`를 반환한다.14


CAS는 '일단 시도해보고 안되면 다시 하자'는 식의 낙관적(optimistic) 동시성 제어 메커니즘을 가능하게 한다. 스레드는 먼저 공유 변수의 값을 지역 변수로 읽어온 뒤, 이를 기반으로 새로운 결과를 계산한다. 그리고 마지막에 CAS를 사용해 자신의 변경 사항을 공유 변수에 반영하려고 시도한다.

이때 CAS 연산의 실패는 오류가 아니다. 이는 단지 '내가 계산하는 동안 다른 스레드가 먼저 값을 변경했구나'라는 유용한 정보를 제공할 뿐이다.5 CAS에 실패한 스레드는 일반적으로 루프를 돌면서 공유 변수의 값을 다시 읽고, 전체 계산 과정을 반복한다.8 이러한 재시도 루프는 때때로 '스핀락(SpinLock)'이라고도 불리지만, 운영체제 스케줄러를 호출하여 스레드를 재우는 전통적인 락과는 달리 사용자 수준에서 CPU를 소모하며 대기한다는 차이점이 있다.15

예를 들어, 두 스레드 T1과 T2가 CAS를 이용해 값 5를 변경하려는 상황을 가정해보자 17:

1. **초기 상태:** 공유 메모리의 값은 5이다.
2. **T1의 시도:** T1은 현재 값 5를 읽고, 이를 7로 바꾸려 한다. `CAS(주소, 5, 7)`을 호출한다.
3. **T1의 성공:** CPU는 주소의 값이 예상 값 5와 일치함을 확인하고, 값을 7로 변경한 뒤 `true`를 반환한다.
4. **T2의 시도:** T1의 작업과 거의 동시에, T2도 현재 값 5를 읽고 이를 8로 바꾸려 한다. `CAS(주소, 5, 8)`을 호출한다.
5. **T2의 실패:** CPU는 주소의 현재 값(7)이 T2의 예상 값(5)과 다르다는 것을 확인한다. 따라서 값을 변경하지 않고 `false`를 반환한다.
6. **T2의 재시도:** T2는 실패를 인지하고, 루프를 돌아 현재 값 7을 다시 읽은 뒤, `CAS(주소, 7, 8)`을 다시 시도하여 성공할 수 있다.


CAS는 읽기-비교-쓰기 과정을 누구의 방해도 받지 않고 한 번에 처리해야 하므로, 반드시 하드웨어 수준에서 원자적으로 지원되어야 한다. 소프트웨어로는 이를 올바르게 구현할 수 없다.5 다행히 x86, ARM 등 현대의 거의 모든 CPU 아키텍처는 이러한 원자적 명령어를 제공한다.5

또한, CAS가 올바르게 동작하기 위한 중요한 전제 조건은 **가시성(Visibility)**의 확보다. 즉, CAS 연산은 특정 CPU의 캐시에 저장된 낡은 값이 아닌, 모든 스레드에게 일관되게 보이는 메인 메모리의 최신 값을 기준으로 수행되어야 한다. 이는 뒤이어 설명할 메모리 모델과 밀접한 관련이 있다.17


락프리 프로그래밍의 복잡성은 원자적 연산 자체보다, 메모리 연산 순서가 어떻게 보장되는지에 대한 문제에서 비롯된다.


현대의 고성능 CPU와 컴파일러는 프로그램의 실행 속도를 높이기 위해 코드의 실행 순서를 임의로 재배치(reordering)하는 최적화를 수행한다. 이 재배치는 *단일 스레드*의 관점에서는 프로그램의 최종 결과가 바뀌지 않는 한 자유롭게 이루어진다.4

예를 들어, 한 스레드 내에서 다음과 같은 코드가 있다면:

x = 1;

y = 2;

컴파일러나 CPU는 두 연산 사이에 의존성이 없다고 판단하여 y = 2;를 먼저 실행할 수 있다.


이러한 재배치는 멀티코어 환경에서 심각한 문제를 일으킨다. 한 스레드(생산자)에서 작성한 코드 순서와 다른 스레드(소비자)에서 관찰되는 메모리 변경 순서가 일치하지 않을 수 있기 때문이다.4

가령, 락프리 큐에 데이터를 `push`하는 과정을 생각해보자.

1. 새로운 노드에 데이터를 쓴다.
2. 새로운 노드를 큐의 끝에 연결한다.

만약 CPU가 이 순서를 뒤바꿔, 노드를 먼저 연결하고 나중에 데이터를 쓴다면 어떻게 될까? 소비자 스레드는 새로 연결된 노드를 발견하고 접근했지만, 아직 데이터가 쓰이기 전이라 쓰레기 값을 읽게 되는 치명적인 데이터 경쟁 상태가 발생한다.

많은 프로그래머들이 `volatile` 키워드를 사용하면 이 문제를 해결할 수 있다고 오해하지만, `volatile`은 컴파일러 수준의 재배치만 방지할 뿐, CPU가 실행 중에 동적으로 수행하는 재배치는 막지 못한다. 따라서 `volatile`만으로는 멀티코어 환경의 동기화 문제를 해결하기에 턱없이 부족하다.4


멀티스레드 환경에서 올바른 동작을 보장하려면, 프로그래머가 직접 컴파일러와 CPU에게 메모리 연산의 순서를 강제해야 한다. 이를 위해 **메모리 장벽(Memory Barrier)** 또는 **메모리 펜스(Memory Fence)**라 불리는 명시적인 지시어를 사용한다.

- **메모리 장벽 (Fences):** `std::atomic_thread_fence(std::memory_order_seq_cst)`와 같은 완전한 메모리 장벽은 코드의 특정 지점을 기준으로, 그 이전의 메모리 연산이 이후의 연산보다 먼저 완료되고 다른 스레드에게 관찰되도록 보장하는 강력한 도구다. 하지만 모든 연산의 순서를 강제하므로 성능 저하를 유발할 수 있다.19
- **Acquire-Release 의미론 (Semantics):** 더 정교하고 효율적인 접근 방식은 개별 원자적 연산에 순서 제약 속성을 부여하는 것이다.
  - **릴리즈 (Release) 연산:** `release` 메모리 순서(예: `std::memory_order_release`)를 가진 쓰기 연산은, 해당 연산 이전에 발생한 모든 메모리 쓰기 작업이 다른 스레드에게 보여지는 것을 보장한다. 즉, 생산자 스레드가 데이터를 준비하고 `release` 쓰기로 플래그를 설정하면, 데이터 준비 작업이 플래그 설정보다 먼저 완료됨이 보장된다.1
  - **어콰이어 (Acquire) 연산:** `acquire` 메모리 순서(예: `std::memory_order_acquire`)를 가진 읽기 연산은, 해당 연산 이후에 오는 모든 메모리 읽기 작업이 짝이 되는 `release` 연산 이전에 발생한 쓰기 작업들을 볼 수 있도록 보장한다. 즉, 소비자 스레드가 `acquire` 읽기로 플래그를 확인했다면, 그 이후에 데이터를 읽을 때 생산자가 준비한 최신 데이터를 안전하게 읽을 수 있다.1

이 두 가지 의미론은 스레드 간에 '발생 전(happens-before)' 관계를 형성하여, 생산자가 데이터를 발행(publish)하고 소비자가 이를 안전하게 소비(consume)하는 동기화 프로토콜을 구축하는 기반이 된다.1

결론적으로, 락프리 프로그래밍의 전문성은 단순히 원자적 연산을 사용하는 것을 넘어, 복잡한 메모리 모델을 깊이 이해하고 이를 통해 스레드 간의 데이터 흐름을 정확하게 조율하는 능력에 달려있다. CAS 연산이 단일 업데이트의 원자성을 보장한다면, 메모리 순서 제어는 그 주변의 전체 프로토콜의 정확성을 보장하는 핵심 요소이다.

------


이론적 기반을 바탕으로, 이제 가장 영향력 있고 널리 사용되는 두 가지 락프리 큐 알고리즘의 내부 구조와 동작 원리를 상세히 분석한다.


1998년 Maged Michael과 Michael L. Scott에 의해 발표된 이 알고리즘은 다중 생산자-다중 소비자(Multi-Producer, Multi-Consumer, MPMC) 환경을 지원하는 고전적이면서도 매우 중요한 락프리 큐다.20 Java의 `java.util.concurrent.ConcurrentLinkedQueue`를 비롯한 수많은 현대적 구현체의 기반이 되었다.21


- **연결 리스트 (Linked List):** 큐는 단일 연결 리스트(singly-linked list) 형태의 노드들로 구성된다.20 각 노드는 데이터와 다음 노드를 가리키는 `next` 포인터를 가진다.

- **원자적 Head/Tail 포인터:** 큐는 `Head`와 `Tail`이라는 두 개의 원자적 포인터를 유지하며, 각각 큐의 맨 앞과 맨 뒤를 가리킨다.20

- **더미 노드 (Dummy Node):** 이 알고리즘의 가장 독창적인 부분은 큐를 초기화할 때 실제 데이터를 담지 않는 '더미 노드'를 하나 생성하고, `Head`와 `Tail` 포인터가 모두 이 더미 노드를 가리키게 하는 것이다.20 이 기법은 큐에 노드가 하나도 없는 상태를 없애 `Head`와 `Tail`이 절대 `null`이 되지 않도록 보장한다. 덕분에 `enqueue`와 `dequeue` 연산에서 처리해야 할 예외적인 경우의 수가 크게 줄어들어 알고리즘이 훨씬 단순해진다. 큐의 실질적인 첫 번째 원소는 `Head`가 가리키는 노드의 *다음* 노드가 된다.


새로운 데이터를 큐의 끝에 추가하는 과정은 다음과 같은 단계로 이루어진다.

1. **노드 할당 및 초기화:** 새로운 노드를 동적으로 할당하고, 전달된 데이터를 저장한다. 이 노드의 `next` 포인터는 `nullptr`로 초기화된다.20

2. **루프 진입 및 실제 Tail 찾기:** 스레드는 CAS를 사용한 재시도 루프에 진입한다.18 먼저 현재의 `Tail` 포인터 값을 지역 변수 `old_tail`에 읽어온다.

3. **노드 연결 시도:** `old_tail`이 가리키는 노드의 `next` 포인터를 `nullptr`에서 방금 생성한 새 노드로 변경하기 위해 CAS 연산을 시도한다. 의사 코드는 다음과 같다: `compare_exchange_weak(old_tail->next, nullptr, new_node)`.23

4. **경쟁 처리:**
   - **성공 시:** CAS가 성공하면, 새 노드가 리스트의 마지막에 성공적으로 연결된 것이다. 이 스레드는 다른 스레드와의 경쟁에서 이겼다. 이제 스레드는 전역 `Tail` 포인터가 자신이 방금 추가한 새 노드를 가리키도록 업데이트를 시도한다. 이 `Tail` 업데이트는 알고리즘의 정확성을 위해 필수는 아니지만, 다른 스레드들이 큐의 실제 끝을 더 빨리 찾도록 도와 성능을 향상시키는 '돕기(helping)' 메커니즘이다.18
   - **실패 시:** CAS가 실패했다는 것은, `old_tail->next`의 값이 더 이상 `nullptr`이 아니라는 의미다. 즉, 그 사이에 다른 스레드가 먼저 새 노드를 연결하는 데 성공한 것이다. 이 경우, 현재 스레드는 루프의 처음으로 돌아가 `Tail` 포인터를 다시 읽고 전체 과정을 재시도한다.18


큐의 맨 앞에서 데이터를 꺼내는 과정은 다음과 같다.

1. **루프 진입 및 Head 읽기:** 스레드는 CAS 재시도 루프에 진입하고, 현재 `Head` 포인터 값을 지역 변수 `old_head`에 읽어온다.23
2. **큐 비었는지 확인:** `Head`와 `Tail` 포인터가 같은 노드를 가리키는지 확인한다. 만약 같다면, 큐가 비어있거나 `Tail` 포인터가 뒤쳐져 있는 상태다. 이때 `Head->next`가 `nullptr`이면 큐가 비었다고 확신할 수 있다.23
3. **데이터 읽기 및 Dequeue 시도:** 큐가 비어있지 않다면, 실제 데이터는 `Head` 포인터가 가리키는 노드(현재 더미 노드)의 다음 노드에 저장되어 있다. 따라서 `old_head->next->data`에서 데이터를 읽는다.21
4. **Head 업데이트:** 이제 `Head` 포인터를 `old_head`에서 `old_head->next`로 한 칸 앞으로 옮기기 위해 CAS 연산을 시도한다. 의사 코드는 다음과 같다: `compare_exchange_weak(Head, old_head, old_head->next)`.23
5. **경쟁 처리 및 정리:**
   - **성공 시:** CAS가 성공하면 `dequeue` 연산이 완료된 것이다. `Head` 포인터가 성공적으로 앞으로 이동했으므로, 이전의 더미 노드였던 `old_head`는 더 이상 큐에서 사용되지 않는다. 이 노드의 메모리는 이제 안전하게 해제될 수 있다(바로 이 지점에서 ABA와 같은 메모리 관리 문제가 발생한다).23
   - **실패 시:** CAS가 실패했다는 것은 다른 스레드가 먼저 `dequeue`에 성공하여 `Head` 포인터를 변경했다는 의미다. 스레드는 루프를 다시 시작한다.


마이클-스캇 알고리즘의 핵심적인 설계 철학은 '뒤처지는 Tail'에 있다. 이는 생산자 스레드 간의 결합도를 낮추고 알고리즘의 락프리 속성을 보장하는 중요한 메커니즘이다.

단순한 큐라면 마지막 노드의 `next` 포인터와 전역 `Tail` 포인터를 하나의 원자적 연산으로 업데이트해야 할 것이다. 하지만 이는 널리 사용 가능한 하드웨어 명령어가 아니다. 마이클-스캇 알고리즘은 이 두 연산을 영리하게 분리했다. `enqueue` 연산은 새 노드가 `next` 포인터에 대한 첫 번째 CAS를 통해 리스트에 *연결*되는 순간 논리적으로 완료된 것으로 간주된다.18

전역 `Tail` 포인터를 업데이트하는 것은 부차적인 '돕기' 작업일 뿐, 방금 완료된 `enqueue`의 정확성에 필수적이지 않다. 이로 인해 `Tail` 포인터는 리스트의 실제 마지막 노드보다 한두 단계 '뒤처질(lag)' 수 있다.20 만약 다른 생산자나 소비자가 

`Tail`이 뒤처져 있음을 발견하면(즉, `Tail->next`가 `null`이 아니면), 자신의 작업을 진행하기 전에 먼저 `Tail` 포인터를 앞으로 옮겨주는 '돕기'를 수행한다.20

이러한 분리 덕분에 알고리즘은 락프리가 된다. 만약 한 스레드가 노드를 성공적으로 연결한 직후 `Tail`을 업데이트하기 전에 운영체제에 의해 선점되더라도, 시스템 전체는 멈추지 않는다. 다른 스레드들이 이 상태를 관찰하고, 뒤처진 `Tail`을 수정하며, 각자의 작업을 계속 진행할 수 있기 때문이다. 이는 시스템 전체의 진행을 보장하는 락프리의 정의에 완벽하게 부합한다.20


단일 생산자-단일 소비자(Single-Producer, Single-Consumer, SPSC)라는 강력한 제약을 활용하여, 대기 없음(Wait-Free) 보장과 최고의 성능을 달성하는 큐 알고리즘이다.


- **배열 기반:** 연결 리스트 대신 고정된 크기의 배열, 즉 '링 버퍼(ring buffer)' 또는 '원형 버퍼(circular buffer)'를 사용한다.25 배열의 크기는 보통 2의 거듭제곱으로 설정하는데, 이는 비싼 모듈로(modulo) 연산 대신 빠른 비트 AND 연산(

  `index & (capacity - 1)`)으로 인덱스 순환을 처리하기 위함이다.26

- **Head/Tail 인덱스:** 포인터 대신 `head`와 `tail`이라는 두 개의 정수형 인덱스를 사용한다. `head`는 다음에 읽을 위치를, `tail`은 다음에 쓸 위치를 나타낸다.25


이 알고리즘의 단순성과 탁월한 성능은 오직 하나의 스레드만이 쓰기(생산자)를 수행하고, 오직 하나의 스레드만이 읽기(소비자)를 수행한다는 제약 조건에서 직접적으로 비롯된다.25

- 생산자는 `tail` 인덱스의 유일한 소유자다.
- 소비자는 `head` 인덱스의 유일한 소유자다.

이러한 소유권 분리는 스레드 간의 경쟁을 원천적으로 차단한다.


1. 생산자는 먼저 소비자가 얼마나 데이터를 가져갔는지 확인하기 위해 `head` 인덱스 값을 읽는다. 이 읽기 작업은 소비자가 업데이트한 최신 `head` 값을 정확히 보기 위해 `acquire` 메모리 순서를 가져야 한다.25
2. `tail - head == capacity`인지 확인하여 큐가 가득 찼는지 검사한다. 가득 찼다면 실패를 반환한다.
3. 배열의 `buffer[tail & (capacity - 1)]` 위치에 새로운 아이템을 쓴다.
4. 가장 중요한 단계로, 생산자는 자신의 `tail` 인덱스 값을 1 증가시킨다. 이 쓰기 작업은 `release` 메모리 순서를 가져야 한다. 이 `release` 저장은 방금 배열에 쓴 데이터가 소비자 스레드에게 보이도록 보장하는 역할을 한다.25


1. 소비자는 생산자가 어디까지 데이터를 채웠는지 확인하기 위해 `tail` 인덱스 값을 읽는다. 이 읽기 작업은 생산자가 발행한 최신 데이터를 놓치지 않기 위해 `acquire` 메모리 순서를 가져야 한다.25
2. `head == tail`인지 확인하여 큐가 비었는지 검사한다. 비었다면 실패를 반환한다.
3. 배열의 `buffer[head & (capacity - 1)]` 위치에서 아이템을 읽는다.
4. 자신의 `head` 인덱스 값을 1 증가시킨다. 이 쓰기 작업은 `release` 메모리 순서를 가져야 하며, 이는 생산자에게 큐의 한 슬롯이 비었음을 알리는 신호가 된다.25


SPSC 큐의 설계는 CAS 연산 없이도 구현될 수 있으며, 이는 ABA 문제로부터 자유롭고 대기 없음(wait-free)을 달성하게 하는 핵심적인 특징이다.

CAS 연산이 해결하려는 근본적인 문제는 여러 스레드가 동일한 메모리 위치를 동시에 수정하려는 경쟁 상태다. SPSC 큐에서는 이러한 경쟁이 설계상 존재하지 않는다. 오직 생산자만이 `tail`을 수정하고, 오직 소비자만이 `head`를 수정하기 때문이다. 두 인덱스를 업데이트하기 위한 경쟁은 발생하지 않는다.

따라서 필요한 동기화는 오직 메모리 가시성 문제, 즉 생산자가 소비자의 진행 상황(업데이트된 `head`)을 보고, 소비자가 생산자의 진행 상황(업데이트된 `tail`)을 보는 것뿐이다. 이는 원자적 업데이트 문제가 아니라 메모리 순서 문제이므로, `acquire`/`release` 메모리 펜스만으로 완벽하게 해결할 수 있다.25 생산자는 데이터 쓰기 후 `tail` 인덱스에 `release` 저장을 수행하고, 소비자는 데이터 읽기 전 `tail` 인덱스에 `acquire` 로드를 수행한다.

CAS를 사용하지 않음으로써, 이 알고리즘은 CAS 루프와 관련된 복잡성 및 성능 문제, 그리고 가장 중요하게는 ABA 문제를 완전히 회피한다. ABA 문제는 포인터의 재사용에서 발생하는데, 정수형 인덱스를 사용하는 SPSC 큐에서는 해당되지 않는다. 이로 인해 SPSC 큐 알고리즘은 매우 빠를 뿐만 아니라, 훨씬 더 단순하고 정확성을 증명하기 용이하며, 대기 없음(wait-free)을 만족한다.10

------


이 파트에서는 포인터 기반의 락프리 자료구조를 구현할 때 마주치는 가장 악명 높고 복잡한 문제들을 다룬다.


ABA 문제는 락프리 알고리즘, 특히 CAS를 사용하는 포인터 기반 자료구조에서 발생하는 미묘하지만 치명적인 버그다.


ABA 문제는 한 스레드가 공유 메모리 위치에서 값 'A'를 읽은 후 선점(preemption)되고, 그 사이에 다른 스레드들이 끼어들어 해당 위치의 값을 'B'로 바꿨다가 다시 'A'로 되돌려 놓는 상황에서 발생한다. 선점되었던 첫 번째 스레드가 다시 실행되어 CAS 연산(`CAS(메모리 위치, 예상값 A, 새 값)`)을 시도하면, 현재 값이 자신의 예상값 'A'와 일치하기 때문에 연산이 **성공**해버린다. 하지만 실제로는 중간에 많은 변화가 있었음에도 불구하고 이를 감지하지 못해 데이터 구조가 손상될 수 있다.30


락프리 스택에서의 ABA 문제는 이 현상의 위험성을 명확하게 보여준다 30:

1. **초기 상태:** 스택의 최상단은 `Top -> A -> B -> C` 순서로 연결되어 있다.
2. **스레드 1의 Pop 시도:** 스레드 1이 `pop`을 호출한다. 스택의 `Top`을 읽어 포인터 `A`를 얻고, `Top`을 `A`의 다음 노드인 `B`로 변경할 준비를 한다. 이 시점에서 스레드 1은 스케줄러에 의해 실행이 중단된다.
3. **스레드 2의 실행:** 스레드 2가 실행되어 `pop`을 두 번 호출한다. 먼저 `A`를 꺼내고, 그다음 `B`를 꺼낸다. 이제 스택은 `Top -> C` 상태가 된다. 스레드 2는 `pop`한 노드 `A`와 `B`의 메모리를 해제(free)한다.
4. **메모리 재사용:** 스레드 2가 새로운 데이터 `D`를 스택에 `push`하기 위해 새 노드를 할당한다. 이때 메모리 할당자는 효율성을 위해 방금 해제된 노드 `A`의 메모리 주소를 그대로 재사용하여 `D`를 위한 노드를 생성한다. 결과적으로 새 노드 `D`는 과거의 노드 `A`와 **완전히 동일한 메모리 주소**를 갖게 된다.30
5. **스레드 2의 Push:** 스레드 2는 `D`를 스택에 `push`한다. 이제 스택은 `Top -> D -> C` 상태가 된다. 여기서 핵심은, 현재 `Top`이 가리키는 포인터 값(D의 주소)이 스레드 1이 처음에 읽었던 포인터 값(A의 주소)과 비트 수준에서 동일하다는 점이다.
6. **스레드 1의 재개 및 CAS 성공:** 스레드 1이 다시 실행된다. 준비해 두었던 `CAS(Top, A, B)`를 실행한다. 현재 `Top` 포인터(D를 가리킴)의 값이 자신이 예상했던 `A`의 포인터 값과 동일하므로, CAS는 이 변경이 유효하다고 판단하고 **성공**한다.
7. **데이터 손상:** `Top` 포인터는 이제 `B`를 가리키게 된다. 하지만 노드 `B`는 이미 스택에서 제거되어 메모리가 해제되었거나 다른 용도로 재사용되고 있을 수 있다. 스택은 이제 유효하지 않은 메모리를 가리키게 되며, 기존에 있던 노드 `D`와 `C`는 연결이 끊어져 유실된다. 스택 전체가 손상된 것이다.30

ABA 문제의 근본 원인은 CAS가 값의 '역사'나 '의미'가 아닌, 단지 비트 패턴의 일치 여부만을 비교하기 때문이다.31 특히 C/C++과 같이 메모리를 수동으로 관리하는 언어에서 메모리 주소가 재사용될 때 이 문제는 더욱 교묘하고 치명적인 버그로 나타난다.24


ABA 문제를 해결하기 위한 주요 기법들은 크게 두 가지 방향으로 나뉜다: 값 자체에 변화의 역사를 기록하거나, 값이 재사용되는 것을 방지하는 것이다.


- **개념:** ABA 문제를 해결하는 가장 직접적인 방법이다. 포인터만 저장하는 대신, `{포인터, 버전 카운터}` 형태의 복합적인 데이터를 원자적으로 다룬다.30
- **동작 방식:** 공유 포인터가 수정될 때마다 버전 카운터를 1씩 증가시킨다. 앞선 스택 시나리오에 이 기법을 적용하면, 스택 `Top`의 상태 변화는 `{A, 버전 1}` -> `{B, 버전 2}` -> `{C, 버전 3}` -> `{(A의 주소를 가진) D, 버전 4}`와 같이 기록된다. 나중에 스레드 1이 재개되어 `CAS(Top, {A, 버전 1}, {B,...})`를 시도하면, 현재 `Top`의 값인 `{A, 버전 4}`와 비교하게 된다. 포인터 주소는 같지만 버전 태그가 다르므로(`1!= 4`), CAS는 올바르게 **실패**한다.35
- **요구 사항:** 이 기법은 포인터와 카운터를 합친 크기(예: 64비트 시스템에서 128비트)의 데이터를 한 번에 비교하고 교환할 수 있는 하드웨어 명령어, 즉 '더블-워드 CAS(Double-Width CAS)'를 필요로 한다. 대부분의 최신 CPU는 이 명령어를 지원한다.30


- **개념:** ABA 문제를 우회적으로 해결하는 메모리 재사용 관리(Memory Reclamation) 기법이다. 어떤 스레드가 특정 노드를 아직 사용하고 있을 가능성이 있다면, 그 노드의 메모리가 해제되는 것을 막는다. 이를 통해 'A -> B -> A' 시나리오에서 해제된 노드 `A`의 주소가 재사용되는 상황 자체를 방지한다.21
- **동작 방식:**
  1. 각 스레드는 '해저드 포인터'라고 불리는 작은 크기의 스레드-로컬(thread-local) 포인터 목록을 유지한다.
  2. 스레드가 공유 포인터를 역참조하여 사용하기 전에(예: `dequeue`에서 `old_head`를 사용하기 전), 해당 포인터를 자신의 해저드 포인터 목록에 등록한다. 이는 "내가 지금 이 노드를 사용하고 있으니, 삭제하지 마시오"라고 다른 스레드들에게 알리는 행위다.37
  3. 포인터를 등록한 후에는, 해당 포인터가 여전히 유효한지 반드시 다시 확인해야 한다(예: `head` 포인터가 그 사이에 바뀌지 않았는지). 이 검증 단계는 매우 중요하다.37
  4. 어떤 스레드가 노드를 삭제하고자 할 때, 즉시 메모리를 해제하지 않고 '삭제 대기' 목록에 추가한다.
  5. 주기적으로, 스레드는 **다른 모든 스레드**의 해저드 포인터 목록을 스캔한다. 만약 자신의 '삭제 대기' 목록에 있는 노드가 다른 어떤 스레드의 해저드 포인터 목록에도 없다면, 그 노드는 현재 아무도 사용하고 있지 않음이 보장되므로 안전하게 메모리를 해제할 수 있다.24
- **장단점:** 해저드 포인터는 매우 효과적이지만, 구현이 복잡하고 상당한 오버헤드를 유발한다. 각 스레드가 주기적으로 다른 모든 스레드의 목록을 스캔해야 하므로, 스레드 수가 많아질수록 성능 저하가 발생할 수 있다.24


- **에포크 기반 재사용 관리 (Epoch-Based Reclamation, 예: RCU):** 스레드들은 '에포크(epoch)'라는 시간 단위 내에서 동작한다. 스레드가 임계 구역에 진입할 때 자신의 현재 에포크를 알린다. 데이터는 해당 데이터가 삭제된 시점의 에포크에 참여했던 모든 스레드가 그 에포크를 벗어날 때까지 메모리 해제가 지연된다. 이는 어떤 스레드도 낡은 데이터를 참조하고 있지 않음을 보장한다.7 일반적으로 해저드 포인터보다 빠르지만, 보호의 단위가 개별 포인터가 아닌 전역적인 에포크이므로 정밀도는 떨어진다.37
- **가비지 컬렉션 (Garbage Collection, GC):** Java나 C#과 같은 관리형 언어에서는 가비지 컬렉터가 ABA 문제로 인한 메모리 손상(use-after-free)을 대부분 막아준다. 어떤 스레드(스레드 1)가 객체 `A`에 대한 참조를 들고 있는 한, GC는 `A`의 메모리를 회수하지 않기 때문이다. 따라서 스레드 2가 `A`를 논리적으로 제거하고 메모리를 재사용하는 시나리오가 발생하지 않는다. 하지만 객체의 주소값이 아닌 '내용'이 ABA 형태로 변하는 논리적인 ABA 문제는 여전히 발생할 수 있다.31

이러한 해결책들은 각각의 장단점이 있으며, 개발자는 자신의 프로젝트 요구사항에 맞춰 최적의 전략을 선택해야 한다. 다음 표는 주요 ABA 문제 해결책들의 특징을 비교하여 의사결정을 돕는다.

| 기법                   | 핵심 메커니즘                           | 성능 오버헤드                       | 구현 복잡도 | 최적 사용 환경                                               |
| ---------------------- | --------------------------------------- | ----------------------------------- | ----------- | ------------------------------------------------------------ |
| **태그 포인터**        | 포인터에 버전 카운터를 결합             | 더블-워드 CAS 명령어 필요           | 낮음        | 128비트 CAS를 지원하는 하드웨어에서 ABA를 직접 방지하고자 할 때 |
| **해저드 포인터**      | 스레드별 사용 중인 포인터 목록 관리     | 모든 스레드의 해저드 목록 스캔 필요 | 높음        | 수동 메모리 관리 환경에서 정밀한 메모리 재사용 제어가 필요할 때 |
| **에포크 기반 (RCU)**  | 전역적인 '유예 기간(grace period)' 설정 | 낮지만, 메모리 해제가 지연됨        | 중간        | 읽기 작업이 쓰기 작업보다 훨씬 빈번한 시나리오               |
| **가비지 컬렉션 (GC)** | 자동 메모리 관리                        | 가변적, 'Stop-the-world' 중단 가능  | 해당 없음   | Java, C# 등 관리형 언어 환경                                 |

------


이론과 난제들을 살펴본 것을 바탕으로, 이제 C++, Java, Rust, 리눅스 커널 등 다양한 환경에서 사용되는 실제 락프리 큐 구현체들이 이러한 원리들을 어떻게 적용하고 있는지 심층적으로 분석한다.


`moodycamel::ConcurrentQueue`는 C++ 커뮤니티에서 가장 널리 알려진 고성능 MPMC 락프리 큐 라이브러리 중 하나다. 이 큐는 고전적인 마이클-스캇 알고리즘에서 벗어난 독창적인 설계를 통해 높은 처리량을 달성한다.

- **설계 철학:** C++을 위한 "가장 빠르고, 가장 완전하며, 가장 잘 테스트된" 범용 락프리 큐를 제공하는 것을 목표로 한다.39 사용 편의성을 위해 단일 헤더 파일로 제공된다.39

- **내부 아키텍처:**

  - **연결 리스트가 아닌 블록 기반:** 전통적인 노드 기반 연결 리스트를 사용하지 않는다. 대신, 원소들을 연속된 메모리 블록에 저장하여 캐시 지역성(cache locality)을 높여 성능을 향상시킨다.39

  - **생산자 기반 서브큐 (Producer-Based Sub-Queues):** 이 큐의 가장 핵심적인 혁신은 내부적으로 여러 개의 서브큐(sub-queue)로 구성되어 있다는 점이다. 각 생산자 스레드는 `ProducerToken`을 통해 자신만의 명시적인 서브큐를 할당받는다.39 이 설계는 생산자 스레드 간의 경쟁을 극적으로 줄인다. 각 생산자는 공유된 

    `Tail` 포인터를 두고 경쟁할 필요 없이, 대부분의 경우 자신에게 할당된 데이터 구조에만 접근하면 되기 때문이다.

  - **소비자 연산:** 소비자 스레드가 데이터를 꺼내려 할 때는, 비어있지 않은 서브큐를 찾기 위해 모든 생산자의 서브큐들을 순회하며 확인해야 한다.39 이 설계는 소비자의 작업을 약간 늘리는 대신 생산자 측의 성능을 극대화하는 방향으로 최적화되어 있다.

- **FIFO 보장:** `moodycamel::ConcurrentQueue`는 큐이지만, 모든 생산자를 아우르는 전역적인 선입선출(FIFO) 순서를 엄격하게 보장하지는 않는다는 점에 유의해야 한다. 오직 **동일한 생산자**가 `enqueue`한 원소들 사이의 상대적인 순서만 보장된다.42 이는 소비자가 아이템을 찾은 서브큐에서 먼저 데이터를 가져가는 구조적 특성 때문이다. 생산자 A가 T1 시점에 넣은 데이터가, 생산자 B가 T2(T1 < T2) 시점에 넣은 데이터보다 늦게 처리될 수 있다.

이러한 설계는 생산자 간의 경쟁을 제거함으로써 MPMC 확장성에서 엄청난 이점을 얻기 위해 엄격한 전역 FIFO 순서를 의도적으로 희생한 결과다. 마이클-스캇 알고리즘이 모든 생산자가 단일 `Tail` 포인터를 두고 CAS 경쟁을 벌이는 병목 지점을 가진다는 점을 고려하면, `moodycamel`의 접근 방식은 이 병목을 분산시켜 해결한 것이다. 이로 인해 이 큐는 작업의 순서보다 처리량이 더 중요한 작업 분배(work-distribution) 시나리오에 매우 적합하다.


Java의 `ConcurrentLinkedQueue`는 JDK에 내장된 MPMC 락프리 큐로, 마이클-스캇 알고리즘의 표준적인 구현체다.

- **기반:** Javadoc은 이 클래스가 마이클-스캇 알고리즘에 기반한, 경계가 없는(unbounded) 비차단(non-blocking) 스레드-안전 큐임을 명시하고 있다.22
- **구현 세부사항:**
  - `Node` 객체로 구성된 연결 리스트와 원자적 `head`, `tail` 참조를 사용한다.45
  - 내부적으로는 `sun.misc.Unsafe` (최신 Java에서는 `VarHandle`)를 통해 하드웨어의 CAS 명령어를 직접 사용하여 원자적 연산을 수행한다.46
  - **`offer(E e)` 메서드:** 마이클-스캇의 `enqueue` 로직을 구현한다. `for (;;)` 무한 루프 안에서 큐의 끝을 찾고, `p.casNext(null, newNode)`를 통해 새 노드를 연결한다. 또한, 다른 스레드가 노드를 추가했지만 `tail` 포인터를 업데이트하지 못한 '뒤처지는 tail'을 발견하면 이를 앞으로 옮겨주는 '돕기' 로직을 포함한다.45
  - **`poll()` 메서드:** `dequeue` 로직을 구현한다. 마찬가지로 경쟁 시 재시도를 위한 루프를 사용하며, `head.next`에서 아이템을 읽고 CAS를 통해 `head` 포인터를 앞으로 이동시킨다.
  - **동시 삭제 처리:** 이 구현체의 흥미로운 세부 사항 중 하나는 동시 삭제를 처리하는 방식이다. 어떤 노드가 다른 스레드에 의해 삭제되었음을 감지하기 위해, 삭제된 노드의 `next` 포인터가 **자기 자신**을 가리키도록 변경한다 (`p == q` where `q = p.next`). 이터레이터나 다른 연산이 이렇게 자기 참조 포인터를 발견하면, 자신이 리스트의 정상적인 경로에서 벗어났음(fallen off)을 인지하고 `head`부터 순회를 다시 시작해야 한다.45
- **주요 특징:**
  - **무경계 (Unbounded):** 큐의 크기에 제한이 없다.22
  - **비차단 (Non-Blocking):** `offer`는 항상 `true`를 반환하고, `poll`은 큐가 비어있으면 `null`을 반환한다. 스레드를 차단하지 않는다.44
  - **`size()`는 O(n) 연산:** 성능상 매우 중요한 주의사항으로, `size()` 메서드는 상수 시간(O(1)) 연산이 아니다. 큐의 크기를 저장하는 공유 카운터('hot field')를 두면 이 카운터 자체가 새로운 경쟁 지점이 되어 락프리 설계의 이점을 해치기 때문에, `size()`는 호출될 때마다 큐의 모든 원소를 순회하여 개수를 센다. 따라서 이 메서드는 매우 느리며, 동시성 애플리케이션에서는 거의 유용하지 않다.43


Rust 생태계에서 고성능 동시성 프로그래밍을 위해 널리 사용되는 `crossbeam-channel`은 MPMC 채널을 구현하며, 링 버퍼를 MPMC 환경에 맞게 발전시킨 독창적인 내부 설계를 가지고 있다.

- **API와 기능:** `crossbeam-channel`은 유연하고 강력한 채널 API를 제공한다. 경계가 있는(bounded), 없는(unbounded), 그리고 용량이 0인 채널을 모두 지원한다.48

  `Sender`와 `Receiver` 핸들을 복제(clone)하여 다중 생산자-다중 소비자 구조를 쉽게 만들 수 있으며 48, 차단, 비차단, 시간제한 대기 연산을 모두 제공한다.48

- **내부 설계 (Bounded Channel):**

  - **링 버퍼:** SPSC 큐와 마찬가지로 배열 기반의 링 버퍼를 사용한다.50
  - **`Slot`과 `stamp`:** 락프리 메커니즘의 핵심은 `Slot` 구조체에 있다. 버퍼의 각 슬롯은 메시지를 담는 공간과 함께, 원자적 정수 타입인 `stamp`를 가지고 있다.50
  - **상태 기계로서의 Stamp:** 이 `stamp`는 단순히 ABA 문제 해결을 위한 버전 카운터가 아니다. 이는 슬롯의 상태(비어 있음, 쓰는 중, 읽기 준비 완료, 읽는 중 등)를 인코딩하는 상태 기계 역할을 한다. 생산자는 슬롯에 데이터를 쓰기 전에 `stamp`에 대한 원자적 연산(CAS 등)을 통해 해당 슬롯을 '선점'해야 한다. `stamp`의 현재 값은 생산자에게 슬롯이 사용 가능한지, 소비자에게는 데이터가 읽을 준비가 되었는지를 알려준다.51
  - **분산된 Head/Tail:** 각 슬롯이 자신만의 `stamp`를 가짐으로써, 생산자와 소비자는 단일 `head`나 `tail` 원자적 변수를 두고 경쟁할 필요 없이 버퍼의 다른 부분에서 동시에 작업할 수 있다. 이는 경쟁 지점을 버퍼 전체에 분산시켜 성능을 높이는 핵심적인 설계다.51

`crossbeam-channel`의 설계는 링 버퍼 알고리즘의 진화를 보여준다. SPSC 링 버퍼가 `head`와 `tail`의 소유권을 분리하여 CAS를 피했다면, `crossbeam-channel`은 MPMC 환경에 맞게 각 슬롯에 상태 관리 책임을 부여함으로써 경쟁을 중앙 집중화하지 않고 분산시키는 방식으로 문제를 해결한다. `head`와 `tail` 인덱스는 여전히 생산자와 소비자를 버퍼의 대략적인 위치로 안내하는 역할을 하지만, 슬롯 사용에 대한 최종 권한은 각 슬롯의 `stamp`에 있다.50 모든 슬롯에 원자적 변수를 두는 것은 낭비가 아니라, 동기화 작업을 분산시키기 위한 의도적이고 강력한 설계 결정이다.


리눅스 커널 내에서 사용되는 `kfifo`는 매우 까다로운 저수준 시스템 환경의 요구사항을 반영한 실용주의적 설계의 좋은 예시다.

- **실용적인 설계:** `kfifo`는 리눅스 커널 내부에서 디바이스 드라이버나 네트워크 스택 등 다양한 서브시스템 간의 데이터 교환에 사용되는 원형 버퍼 구현체다.28 그 설계는 단순성과 실용성으로 주목받는다.

- **SPSC에 대해서는 락프리:** 커널 문서와 소스 코드는 단일 생산자-단일 소비자(SPSC) 환경에서 `kfifo`가 **락프리**임을 명시적으로 밝히고 있다. 이 경우 외부적인 락이 전혀 필요 없다.28

  `kfifo`는 표준적인 SPSC 알고리즘과 마찬가지로, 정확한 메모리 순서를 보장하기 위해 메모리 장벽(쓰기 측에서는 `smp_wmb()`, 읽기 측에서는 `smp_rmb()`)에 의존한다.28

- **MPMC/SPMC에 대해서는 락 기반:** `kfifo`는 범용 MPMC 락프리 큐가 아니다. 다중 생산자나 다중 소비자가 접근해야 할 경우, 커널 개발자들은 호출자가 직접 외부 락(보통 스핀락)을 사용하여 생산자들 간의 접근 또는 소비자들 간의 접근을 직렬화하도록 요구한다.53 이를 위해 

  `kfifo_in_spinlocked`와 `kfifo_out_spinlocked` 같은 래퍼 매크로가 제공된다.53

- **구현 세부사항:**

  - 동적으로 할당된 버퍼를 사용하며, 빠른 인덱스 순환을 위해 크기는 반드시 2의 거듭제곱이어야 한다.28
  - `in`(tail)과 `out`(head)을 위해 부호 없는 정수 인덱스를 사용하며, 이 인덱스들은 오버플로우되도록 내버려 둔다. 큐에 있는 원소의 개수는 단순히 `in - out`으로 계산된다.28

리눅스 커널의 `kfifo`는 실용적인 공학적 트레이드오프를 명확히 보여준다. 커널 개발자들은 가장 흔한 SPSC 사용 사례에 대해서는 락과 CAS를 모두 회피하는 극도로 빠르고 단순한 대기 없는(wait-free) 알고리즘을 선택했다. 반면, 더 복잡한 MPMC 시나리오에 대해서는, 잠재적으로 취약하고 복잡한 락프리 알고리즘을 구현하는 대신, 잘 알려지고 검증된 락(스핀락) 프리미티브를 사용하여 동기화 책임을 호출자에게 위임하는 더 안전하고 단순한 길을 택했다. 이는 모든 경우에 '순수한' 락프리 구현을 고집하기보다, 특정 상황에서 미미한 이득을 위해 거대한 복잡성을 도입하는 것을 피하고 정확성과 단순성을 우선시하는 현명한 엔지니어링 결정이다.

------


본 보고서는 락프리 큐의 근본 원리부터 대표적인 알고리즘, 그리고 실제 구현체에 이르기까지 다각적인 분석을 제공했다. 락 기반 동기화의 한계를 극복하기 위한 대안으로 출발한 락프리 프로그래밍은, 하드웨어 수준의 원자적 연산(CAS)과 정교한 메모리 순서 제어(Acquire-Release)를 통해 비차단(Non-Blocking) 진행을 보장하는 고도의 기술임을 확인했다.


락프리 큐의 세계는 크게 두 가지 설계, 즉 마이클-스캇 알고리즘으로 대표되는 **포인터 기반 연결 리스트**와 SPSC 큐로 대표되는 **인덱스 기반 링 버퍼**로 나뉜다. 연결 리스트 기반 큐는 동적인 크기 조절이 가능하고 범용 MPMC 환경에 적합하지만, 포인터 연산으로 인해 발생하는 ABA 문제와 메모리 재사용 관리라는 복잡한 난제를 안고 있다. 반면, 링 버퍼 기반 큐는 고정된 크기를 가지지만, 인덱스 연산의 단순성과 SPSC 제약을 통해 CAS 없이 대기 없는(wait-free) 수준의 압도적인 성능을 달성할 수 있다.

`moodycamel::ConcurrentQueue`와 `crossbeam-channel` 같은 현대적 구현체들은 이러한 고전적 설계들을 창의적으로 발전시켰다. `moodycamel`은 생산자별 파티셔닝을 통해 MPMC 환경의 생산자 경쟁을 제거했으며, `crossbeam-channel`은 각 슬롯에 상태 정보를 부여하여 경쟁을 분산시키는 방식으로 MPMC 링 버퍼를 구현했다. 이는 락프리 설계가 단일한 해법이 아니라, 해결하고자 하는 문제의 특성에 따라 최적의 트레이드오프를 찾는 과정임을 보여준다.


이 분야의 핵심적인 트레이드오프는 **단순성 및 안전성 vs. 경쟁 하에서의 성능**이다.

- **락 기반 큐:** 구현과 논리적 추론이 상대적으로 단순하지만, 스레드 경쟁이 심화될수록 성능이 급격히 저하되고 시스템 전체를 마비시킬 위험이 있다.
- **락프리 큐:** 높은 부하와 경쟁 속에서도 뛰어난 확장성과 안정적인 성능을 제공하지만, 정확성을 보장하기 위해 메모리 모델, ABA 문제 등 극도로 복잡한 사안들을 고려해야 한다.


락프리 큐를 실제 시스템에 도입하고자 하는 개발자를 위해 다음의 제언을 제시한다.

1. **사용 사례를 명확히 하라 (SP vs. MP, SC vs. MC):** 이것이 가장 중요한 첫걸음이다. SPSC(단일 생산자-단일 소비자) 환경이라면, 고민할 여지 없이 단순하고 빠른 링 버퍼 기반 큐가 최상의 선택이다.25 MPMC(다중 생산자-다중 소비자)가 필요하다면 선택은 더 복잡해진다.

2. **바퀴를 재발명하지 마라:** 정확한 락프리 자료구조를 처음부터 직접 구현하는 것은 "극도로 어려운" 작업이며, 미묘한 버그를 만들 가능성이 매우 높다.1 실무에서는 

   `moodycamel::ConcurrentQueue`, `crossbeam-channel`, Java의 `ConcurrentLinkedQueue` 등 수많은 전문가에 의해 검증되고 테스트된 라이브러리를 사용하는 것이 압도적으로 현명한 선택이다.4

3. **가정하지 말고 측정하라:** 락프리가 항상 더 빠르다는 것은 잘못된 통념이다. 스레드 경쟁이 거의 없는 저부하 환경에서는, CAS 루프와 메모리 장벽의 오버헤드로 인해 잘 구현된 락 기반 큐가 락프리 큐보다 더 빠를 수 있다.5 반드시 실제 운영 환경과 유사한 워크로드로 벤치마킹하여 성능을 검증해야 한다.

4. **FIFO 보장 수준을 확인하라:** 엄격한 전역 선입선출(FIFO) 순서가 비즈니스 로직에 필수적인가? 그렇다면 마이클-스캇 스타일의 큐(`ConcurrentLinkedQueue`)가 적합하다. 만약 처리량이 더 중요하고 생산자별 상대적 순서만으로 충분하다면, 파티션 기반 설계(`moodycamel::ConcurrentQueue`)가 훨씬 뛰어난 성능을 제공할 것이다.22

다음 표는 본 보고서에서 분석한 주요 락프리 큐 구현체들의 특징을 종합적으로 비교하여, 개발자가 특정 요구사항에 맞는 최적의 큐를 선택하는 데 도움을 줄 것이다.

| 구현체                                           | 동시성 모델 | 기본 자료구조    | 핵심 동기화 기법        | FIFO 보장            | 주요 강점 및 트레이드오프                                   |
| ------------------------------------------------ | ----------- | ---------------- | ----------------------- | -------------------- | ----------------------------------------------------------- |
| **마이클-스캇 큐** (예: `ConcurrentLinkedQueue`) | MPMC        | 연결 리스트      | CAS                     | 엄격한 전역 FIFO     | 고전적이고 잘 알려짐. `Tail` 포인터가 병목이 될 수 있음.    |
| **SPSC 링 버퍼** (예: `kfifo`)                   | SPSC        | 배열 (링 버퍼)   | Acquire/Release 펜스    | 엄격한 전역 FIFO     | 극도로 빠르고 대기 없음(Wait-Free). SPSC로 용도 제한.       |
| **`moodycamel::ConcurrentQueue`**                | MPMC        | 블록 기반 파티션 | 생산자-로컬 원자적 연산 | 생산자별 상대적 FIFO | 생산자 확장성이 매우 뛰어남. 전역 FIFO는 보장 안 됨.        |
| **`crossbeam-channel`**                          | MPMC        | 배열 (링 버퍼)   | 슬롯별 원자적 `stamp`   | 엄격한 전역 FIFO     | 링 버퍼에서 MPMC를 구현하며 경쟁을 분산시켜 높은 성능 달성. |


1. [C++ Thread] Lock-Free Programming - 움직이는 월e - 티스토리, accessed July 4, 2025, https://narakit.tistory.com/194
2. LockFree - 게임서버 notepad - 티스토리, accessed July 4, 2025, https://univ-developer.tistory.com/entry/LockFree
3. Lock Free - 공부 모음 - 티스토리, accessed July 4, 2025, https://ozt88.tistory.com/38
4. [Server] Lock-free - velog, accessed July 4, 2025, https://velog.io/@cedongne/Server-Lock-free
5. 멀티쓰레드 프로그래밍이 왜이리 힘드나요? - 정내훈 - NDC Replay, accessed July 4, 2025, https://ndcreplay.nexon.com/NDC2014/sessions/NDC2014_0048.html
6. (번역중) lock-free 프로그래밍 소개 - Sync the world. - 티스토리, accessed July 4, 2025, [https://syncnet.tistory.com/entry/%EB%B2%88%EC%97%AD%EC%A4%91-lockfree-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EC%86%8C%EA%B0%9C](https://syncnet.tistory.com/entry/번역중-lockfree-프로그래밍-소개)
7. Lock-Free Programming, accessed July 4, 2025, https://www.cs.cmu.edu/~410-s05/lectures/L31_LockFree.pdf
8. An Introduction to Lock-Free Programming, accessed July 4, 2025, https://preshing.com/20120612/an-introduction-to-lock-free-programming/
9. Introduction to Lock-Free and How it is used to implement Thread-safe Non-Blocking Queue in Java | by Toan Hoang | Medium, accessed July 4, 2025, https://medium.com/@hoangxuantoank13/introduction-to-lock-free-and-how-it-is-used-to-implement-thread-safe-non-blocking-queue-in-java-b0759a25058a
10. Chapter 19. Boost.Lockfree, accessed July 4, 2025, https://www.boost.org/libs/lockfree/
11. Notes on Lock Free Programming - Hacker News, accessed July 4, 2025, https://news.ycombinator.com/item?id=13892581
12. ko.wikipedia.org, accessed July 4, 2025, [https://ko.wikipedia.org/wiki/%EB%B9%84%EA%B5%90%EC%99%80_%EA%B5%90%ED%99%98#:~:text=%EB%B9%84%EA%B5%90%EC%99%80%20%EA%B5%90%ED%99%98%2C%20%EC%BB%B4%ED%8E%98%EC%96%B4%20%EC%95%A4,%EC%A3%BC%EC%96%B4%EC%A7%84%20%EA%B0%92%EC%9C%BC%EB%A1%9C%20%EC%88%98%EC%A0%95%ED%95%9C%EB%8B%A4.](https://ko.wikipedia.org/wiki/비교와_교환#:~:text=비교와 교환%2C 컴페어 앤,주어진 값으로 수정한다.)
13. 컴페어 스왑(Compare and Swap) - 박철우의 블로그 - 티스토리, accessed July 4, 2025, https://parkcheolu.tistory.com/31
14. 비교와 교환 - 위키백과, 우리 모두의 백과사전, accessed July 4, 2025, [https://ko.wikipedia.org/wiki/%EB%B9%84%EA%B5%90%EC%99%80_%EA%B5%90%ED%99%98](https://ko.wikipedia.org/wiki/비교와_교환)
15. CAS(Compare and Swap)와 SpinLock 구현 - 테일윈드 - 티스토리, accessed July 4, 2025, https://dev-record.tistory.com/50
16. [Java] atomic과 CAS 알고리즘 - 느리더라도 꾸준하게 - 티스토리, accessed July 4, 2025, https://steady-coding.tistory.com/568
17. CAS(Compare And Swap) - velog, accessed July 4, 2025, https://velog.io/@appti/CASCompare-And-Set
18. How to write a lock free Queue - Schneems, accessed July 4, 2025, https://www.schneems.com/2017/06/28/how-to-write-a-lock-free-queue/
19. Lock-Free Queue - Part I - The Book of Gehn, accessed July 4, 2025, https://book-of-gehn.github.io/articles/2020/03/22/Lock-Free-Queue-Part-I.html
20. Verifying Michael and Scott's Lock-Free Queue Algorithm using Trace Reduction, accessed July 4, 2025, [https://kk.up45.ac.id/scholarhub/Verifying%20Michael%20and%20Scott%E2%80%99s%20Lock-Free%20Queue%20Algorithm%20using%20Trace.pdf](https://kk.up45.ac.id/scholarhub/Verifying Michael and Scott’s Lock-Free Queue Algorithm using Trace.pdf)
21. DoubleLink - A Low-Overhead Lock-Free Queue - Concurrency Freaks, accessed July 4, 2025, http://concurrencyfreaks.blogspot.com/2017/01/doublelink-low-overhead-lock-free-queue.html
22. ConcurrentLinkedQueue (Java Platform SE 8 ) - Oracle Help Center, accessed July 4, 2025, https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentLinkedQueue.html
23. Introduction to Lock-free Queue | CodeSignal Learn, accessed July 4, 2025, https://codesignal.com/learn/courses/lock-free-concurrent-data-structures/lessons/introduction-to-lock-free-queue
24. Explain Michael & Scott lock-free queue alorigthm - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/40818465/explain-michael-scott-lock-free-queue-alorigthm
25. Lock-Free Single-Producer - Single Consumer Circular Queue - CodeProject, accessed July 4, 2025, https://www.codeproject.com/Articles/43510/Lock-Free-Single-Producer-Single-Consumer-Circular
26. Ring buffers and queues | The ryg blog - WordPress.com, accessed July 4, 2025, https://fgiesen.wordpress.com/2010/12/14/ring-buffers-and-queues/
27. Circular lock-free buffer - c++ - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/871234/circular-lock-free-buffer
28. linux/kernel/kfifo.c at master - GitHub, accessed July 4, 2025, https://github.com/spotify/linux/blob/master/kernel/kfifo.c
29. kernel/kfifo.c - kernel/msm.git - Git at Google - Android GoogleSource, accessed July 4, 2025, https://android.googlesource.com/kernel/msm.git/+/android-msm-2.6.35/kernel/kfifo.c
30. ABA problem - Wikipedia, accessed July 4, 2025, https://en.wikipedia.org/wiki/ABA_problem
31. The ABA Problem | Baeldung on Computer Science, accessed July 4, 2025, https://www.baeldung.com/cs/aba-concurrency
32. Lock-free in Swift: ABA Problem - Alex Shchukin - Medium, accessed July 4, 2025, https://shchukin-alex.medium.com/lock-free-in-swift-aba-problem-4d68622164da
33. Nasty ABA problem in array-based lock-free stack - julian m bucknall, accessed July 4, 2025, https://boyet.com/blog/nasty-aba-problem-in-array-based-lock-free-stack/
34. Common Pitfalls in Writing Lock-Free Algorithms - SingleStore, accessed July 4, 2025, https://www.singlestore.com/blog/common-pitfalls-in-writing-lock-free-algorithms/
35. Solving the ABA Problem in Rust with Tagged Pointers - Oleksandr Prokhorenko, accessed July 4, 2025, https://minikin.me/blog/solving-the-aba-problem-in-rust-tagged-pointers
36. Understanding and Effectively Preventing the ABA Problem in Descriptor-based Lock-free Designs, accessed July 4, 2025, https://www.osti.gov/servlets/purl/1124348
37. Solving the ABA Problem in Rust with Hazard Pointers, accessed July 4, 2025, https://minikin.me/blog/solving-the-aba-problem-in-rust-hazard-pointers
38. How to solve ABA problem in Rust (Hazard Pointers) - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/79590043/how-to-solve-aba-problem-in-rust-hazard-pointers
39. cameron314/concurrentqueue: A fast multi-producer, multi-consumer lock-free concurrent queue for C++11 - GitHub, accessed July 4, 2025, https://github.com/cameron314/concurrentqueue
40. moodycamel::ConcurrentQueue download | SourceForge.net, accessed July 4, 2025, https://sourceforge.net/projects/moodyc-concurrentqueue.mirror/
41. I need a single-consumer, multi-ad-hoc-producer queue : r/cpp - Reddit, accessed July 4, 2025, https://www.reddit.com/r/cpp/comments/1f7xycv/i_need_a_singleconsumer_multiadhocproducer_queue/
42. c++ - Are there any concurrent containers in C++11? - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/7817364/are-there-any-concurrent-containers-in-c11
43. java.util.concurrent.ConcurrentLinkedQueue - People | MIT CSAIL, accessed July 4, 2025, https://people.csail.mit.edu/dfhuynh/research/javadoc/jdk1.5.0/java/util/concurrent/ConcurrentLinkedQueue.html
44. A Guide to Concurrent Queues in Java | Baeldung, accessed July 4, 2025, https://www.baeldung.com/java-concurrent-queues
45. ConcurrentLinkedQueue Code Explanation - java - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/18696343/concurrentlinkedqueue-code-explanation
46. [JavaSpecialists 261] - Concurrent Queue Sizes and Hot Fields, accessed July 4, 2025, https://www.javaspecialists.eu/archive/Issue261-Concurrent-Queue-Sizes-and-Hot-Fields.html
47. LinkedBlockingQueue vs ConcurrentLinkedQueue | Baeldung, accessed July 4, 2025, https://www.baeldung.com/java-queue-linkedblocking-concurrentlinked
48. crossbeam::channel - Rust - Docs.rs, accessed July 4, 2025, https://docs.rs/crossbeam/latest/crossbeam/channel/index.html
49. Another novice query: mpsc or crossbeam? : r/rust - Reddit, accessed July 4, 2025, https://www.reddit.com/r/rust/comments/18z6bcz/another_novice_query_mpsc_or_crossbeam/
50. How does crossbeam implement its channels without a mutex? : r/rust - Reddit, accessed July 4, 2025, https://www.reddit.com/r/rust/comments/1hkc9ei/how_does_crossbeam_implement_its_channels_without/
51. How does a crossbeam channel work? - The Rust Programming Language Forum, accessed July 4, 2025, https://users.rust-lang.org/t/how-does-a-crossbeam-channel-work/46698
52. alpc62/lock-free-queue: C/C++Non-Blocking Lock-Free/Wait-Free Circular-Queue - GitHub, accessed July 4, 2025, https://github.com/alpc62/lock-free-queue
53. Why do kfifo need smp_wmb even with spin_lock_irqsave in kernel in version 4.9.37, accessed July 4, 2025, https://stackoverflow.com/questions/78096108/why-do-kfifo-need-smp-wmb-even-with-spin-lock-irqsave-in-kernel-in-version-4-9-3
54. Kernel development - LWN.net, accessed July 4, 2025, https://lwn.net/Articles/106560/
55. Linux Kernel: include/linux/kfifo.h File Reference - Huihoo, accessed July 4, 2025, https://docs.huihoo.com/doxygen/linux/kernel/3.7/kfifo_8h.html


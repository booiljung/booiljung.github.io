[러스트 (Rust) 프로그래밍 언어](./index.md)
# Rust 비동기 모델을 활용한 스레드리스 DJI Dock 2 다중 관제 시스템 아키텍처



본 보고서의 목표는 다수의 DJI Dock 2 및 이에 탑재된 드론을 위한 실시간, 고가용성, 확장 가능한 관제 소프트웨어의 아키텍처를 설계하고 심층적으로 고찰하는 데 있다. 이 시스템은 복수의 드론으로부터 전송되는 원격 측정 데이터(telemetry)를 실시간으로 수신 및 처리하고, 비행 임무 명령을 전송하며, 드론의 상태를 지속적으로 모니터링하는 복합적인 기능을 수행해야 한다.

이 프로젝트의 가장 중요한 기술적 요구사항이자 핵심 제약 조건은 운영체제(OS) 수준의 스레드(thread)를 명시적으로 사용하지 않는 아키텍처를 구현하는 것이다. 사용자는 "스레드는 코딩 복잡도를 상승시키고 안정성을 해친다"는 명확한 문제의식을 제시했다. 이는 전통적인 동시성 모델에서 발생하는 스레드 간의 데이터 경쟁(Data Races), 교착 상태(Deadlock), 그리고 복잡한 동기화 프리미티브(락, 뮤텍스 등) 관리에서 비롯되는 잠재적 불안정성과 예측 불가능성을 원천적으로 회피하려는 중요한 설계 목표를 반영한다.1

이러한 제약 조건 하에, 본 시스템은 다음의 기술 스택을 기반으로 구축된다:

- **프로그래밍 언어:** Rust
- **드론 통신 프로토콜:** MQTT
- **미들웨어:** Supabase, Data Distribution Service (DDS)

이 기술 스택은 메모리 안전성과 높은 성능을 보장하는 Rust를 중심으로, 표준화된 IoT 통신 프로토콜인 MQTT, 내부 시스템 컴포넌트 간의 효율적인 데이터 교환을 위한 DDS, 그리고 백엔드 서비스 구축을 간소화하는 Supabase를 결합하여 현대적이고 성능 지향적인 시스템을 구축하는 것을 목표로 한다.


전통적인 멀티스레딩 환경에서 동시성(concurrency)은 다수의 스레드가 OS 스케줄러에 의해 시분할 방식으로 CPU를 점유하는 선점형 멀티태스킹(Preemptive Multitasking)을 통해 달성된다. 이 모델은 CPU 집약적인 작업을 병렬(parallel)로 처리하는 데 효과적이지만, 공유 메모리에 대한 접근을 제어하기 위한 복잡한 동기화 메커니즘을 요구한다. 이러한 복잡성은 미묘한 버그의 원인이 되며, 시스템의 안정성을 저해하는 주요 요인으로 작용할 수 있다.1

본 보고서는 이러한 문제에 대한 근본적인 해결책으로 Rust의 **비동기 프로그래밍 모델(Asynchronous Programming Model)**을 제시한다. Rust의 비동기 모델은 스레드를 통한 선점형 멀티태스킹이 아닌, 단일 스레드 내에서의 **협력적 멀티태스킹(Cooperative Multitasking)**을 통해 동시성을 달성하는 패러다임이다.2 각 비동기 작업(task)은 I/O 대기와 같은 블로킹이 예상되는 지점에서 자발적으로 제어권을 양보(`yield`)하고, 런타임은 이 시간에 다른 준비된 작업을 실행한다. 이 방식은 I/O 바운드(I/O-bound) 작업이 대부분인 네트워크 애플리케이션, 특히 수많은 드론과 동시에 통신해야 하는 본 관제 시스템에 매우 효율적이다.

이러한 스레드리스 패러다임을 구현하기 위한 핵심 기술은 **Tokio의 `current_thread` 실행기(executor)**이다. Tokio는 Rust 생태계에서 가장 널리 사용되는 비동기 런타임으로, 기본적으로는 멀티스레드 기반의 작업 훔치기(work-stealing) 스케줄러를 사용하지만, `current_thread` 옵션을 통해 단일 OS 스레드 위에서 모든 비동기 작업을 처리하는 이벤트 루프를 구성할 수 있다.4 이는 OS 스레드를 추가로 생성하지 않으면서도 수만 개의 동시 작업을 효율적으로 관리할 수 있게 하여, 사용자가 제시한 핵심 제약 조건을 완벽하게 만족시키는 동시에 높은 성능과 안정성을 달성할 수 있는 최적의 해법이다.


본 보고서는 먼저 스레드를 대체하는 Rust의 비동기 메커니즘의 이론적 배경과 내부 동작 원리를 심층적으로 분석한다. 이를 바탕으로, Tokio의 단일 스레드 실행기 위에서 동작하는 구체적인 시스템 아키텍처를 설계하고, 각 기술 요소(MQTT, DDS, Supabase)의 통합 방안을 제시한다. 이후, DJI Dock 2의 Cloud API를 활용한 주요 제어 기능(원격 측정, 임무 관리, 영상 스트림 제어 등)의 상세 구현 방안을 다룬다. 마지막으로, 제안된 아키텍처의 성능적 특성, 안정성, 그리고 확장성 전략을 종합적으로 분석하며 결론을 도출한다. 이 보고서는 단순한 구현 가이드를 넘어, 스레드리스 패러다임에 기반한 고성능 실시간 관제 시스템 설계에 대한 깊이 있는 기술적 고찰을 제공하는 것을 목표로 한다.


Rust의 `async/await`는 전통적인 스레드 기반 동시성 모델의 패러다임을 근본적으로 전환하는 기술이다. 이는 OS 커널에 의존하는 무거운 컨텍스트 스위칭 대신, 사용자 공간에서 가볍고 효율적으로 작업 전환을 수행하는 협력적 멀티태스킹을 구현한다. 이 섹션에서는 스레드를 사용하지 않고도 어떻게 수많은 동시 작업을 처리할 수 있는지, 그 핵심 원리를 심층적으로 분석한다.


Rust 비동기 프로그래밍의 심장에는 `Future` 트레잇이 있다. `async fn` 또는 `async` 블록을 사용하면, Rust 컴파일러는 해당 코드를 `Future` 트레잇을 구현하는 익명의 구조체로 변환한다.6 이 구조체는 단순한 데이터 컨테이너가 아니라, 비동기 작업의 전체 생명주기를 관리하는 **상태 기계(State Machine)**이다.8

이 상태 기계는 코드 내의 각 `.await` 호출 지점을 기준으로 여러 상태(state)로 분할된다. 함수 내에서 선언된 지역 변수들은 전통적인 함수처럼 스택에 저장되는 것이 아니라, 이 `Future` 구조체 자체의 필드로 캡처된다. 이 덕분에 `Future`는 `poll` 호출 사이에 실행이 중단되더라도 자신의 상태(지역 변수 값 등)를 그대로 유지할 수 있다. 이는 함수 호출이 끝나고 스택 프레임이 사라진 후에도 비동기 작업의 상태를 보존하고 나중에 재개할 수 있게 하는 핵심 원리이다.8

`Future`의 가장 중요한 특징 중 하나는 "게으르다(lazy)"는 점이다. `async fn`을 호출하는 것만으로는 어떠한 코드도 실행되지 않는다. 단지 상태 기계인 `Future` 객체가 생성될 뿐이다. 실제 작업은 비동기 실행기(executor)가 해당 `Future`의 `poll` 메소드를 호출하거나, 다른 `Future` 내에서 `.await` 키워드를 통해 폴링될 때 비로소 시작된다.2 이러한 지연 실행(lazy execution) 모델은 실제로 작업이 필요한 시점까지 리소스 사용을 최소화하여 시스템의 효율성을 극대화한다.

결론적으로 `async/await`는 단순한 문법적 설탕(syntactic sugar)을 넘어서, Rust의 강력한 소유권(ownership) 및 생명주기(lifetime) 시스템과 결합하여 메모리 안전성을 완벽하게 보장하면서 동시성을 구현하는 패러다임의 근본적인 전환이다. OS 스레드의 무거운 컨텍스트와 스택을, 힙에 할당될 수 있는 가벼운 상태 기계 객체로 대체함으로써, OS 커널의 개입 없이 사용자 공간에서 수만 개의 작업을 정교하게 스케줄링하고 전환하는 것을 가능하게 한다.


`Future`가 상태 기계라면, 이 기계를 구동시키는 엔진이 필요하다. 이것이 바로 비동기 런타임이며, 실행기(Executor), Waker, 그리고 `Pin`이라는 세 가지 핵심 요소로 구성된다.

- **실행기 (Executor):** 실행기는 `Future`의 `poll` 메소드를 반복적으로 호출하여 비동기 작업을 앞으로 진척시키는 주체다. Tokio, `async-std` (현재는 개발이 중단되어 비권장 1), 

  `smol` 등 다양한 런타임이 존재하며, 본 아키텍처에서는 사실상의 표준인 Tokio를 사용한다.7 실행기는 준비된(ready) `Future`들을 관리하는 큐를 가지고 있으며, 이 큐에서 작업을 꺼내 실행한다.

- **`poll` 메소드와 `Waker`:** `Future` 트레잇의 유일한 메소드인 `poll`은 다음과 같은 시그니처를 가진다.

  ```Rust
  fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>
  ```

  `poll`의 결과는 `Poll` 열거형으로, 두 가지 상태를 가진다.

  - `Poll::Ready(T)`: 작업이 성공적으로 완료되었으며, 결과값 `T`를 반환한다.
  - `Poll::Pending`: 작업이 아직 완료되지 않았음을 의미한다. 이는 주로 네트워크 I/O나 타이머와 같은 외부 리소스를 기다리는 경우에 해당한다. `Poll::Pending`을 반환하면, `Future`는 즉시 제어권을 실행기에게 돌려주어 다른 작업을 처리할 수 있도록 한다.8

  `Poll::Pending`을 반환할 때, `Future`는 "나중에 작업이 준비되면 나를 다시 깨워달라"는 약속을 해야 한다. 이 약속을 이행하는 메커니즘이 바로 **`Waker`**이다. `poll` 메소드의 인자인 `Context`를 통해 현재 작업을 위한 `Waker` 객체를 얻을 수 있다. `Future`는 이 `Waker`를 어딘가에 저장해두었다가, 기다리던 리소스(예: TCP 소켓)가 준비되면 해당 리소스가 저장된 `waker.wake()` 메소드를 호출한다. `wake()` 호출은 실행기에게 "이 작업이 이제 준비되었으니, 다시 스케줄링 큐에 넣어달라"고 알리는 신호다. 이 `Waker` 메커니즘 덕분에 실행기는 비효율적인 반복 폴링(busy-polling)을 하지 않고, 실제로 진행 가능한 작업들만 효율적으로 처리할 수 있다.13

- Pin: Rust의 async 블록 내에서는 일반 함수처럼 변수에 대한 참조(&T)를 자유롭게 사용할 수 있다. 그런데 Future는 상태를 필드로 저장하는 구조체이고, 이 구조체는 힙에 할당되거나 다른 구조체의 일부가 될 수 있다. 만약 Future 내부에 자기 자신의 다른 필드를 가리키는 포인터(참조)가 있다면, 이 Future 객체가 메모리 상에서 이동(move)될 경우 내부 포인터는 무효화되어 댕글링 포인터(dangling pointer)가 될 것이다. 이러한 자기 참조 구조체(self-referential struct)는 안전한 Rust에서 기본적으로 허용되지 않는다.

  Pin은 이 문제를 해결하기 위해 도입된 타입이다. Pin<P> (여기서 P는 포인터 타입)는 포인터가 가리키는 값(P::Target)의 메모리 주소가 이동되지 않도록 '고정(pinning)'하는 역할을 한다. Pin으로 감싸진 값은 안전한(safe) 코드에서는 이동시킬 수 없으므로, Future 상태 기계는 내부적으로 자기 참조를 가지더라도 메모리 안전성을 유지할 수 있다. poll 메소드가 self 대신 Pin<&mut Self>를 받는 이유가 바로 이것이다. Pin은 Rust의 비동기 모델이 제로 코스트 추상화(zero-cost abstraction)를 유지하면서도 강력한 메모리 안전성을 보장하는 핵심적인 장치다.6

이 세 요소의 상호작용은 완벽한 협력적 멀티태스킹 시스템을 구성한다. `Future`는 "지금 일을 할 수 있는가?"라고 `poll`을 통해 묻고, 할 수 없다면 "준비되면 깨워달라"고 `Waker`를 등록한 뒤 잠든다. 외부 리소스는 준비가 되면 `Waker`를 깨우고, 실행기는 깨어난 작업들만 효율적으로 다시 `poll`한다. 이 모든 과정이 OS 스케줄러의 개입 없이 사용자 수준에서, 훨씬 가볍고 특화된 방식으로 이루어진다.


Tokio는 Rust 생태계에서 가장 성숙하고 널리 사용되는 비동기 런타임으로, 강력한 성능과 풍부한 유틸리티를 제공하여 사실상의 표준(de facto standard)으로 자리 잡았다.1 Tokio는 애플리케이션의 특성에 맞게 선택할 수 있는 두 가지 주요 실행기(스케줄러) 모델을 제공한다.

1. **`multi_thread` 실행기:** Tokio의 기본 실행기 모델이다. 이 모델은 시스템의 CPU 코어 수에 맞춰 여러 개의 OS 스레드로 구성된 스레드 풀을 생성한다. 각 스레드는 자신만의 로컬 작업 큐를 가지며, 자신의 큐가 비었을 때 다른 스레드의 큐에서 작업을 "훔쳐오는" **작업 훔치기(work-stealing)** 알고리즘을 사용한다.4 이 방식은 스레드 간 작업 부하를 동적으로 분산시켜 CPU 활용률을 극대화하므로, CPU 바운드 작업과 I/O 바운드 작업이 혼재된 일반적인 웹 서버나 고성능 애플리케이션에 최적화되어 있다.12
2. **`current_thread` 실행기:** `main` 함수 위에 `#[tokio::main(flavor = "current_thread")]` 어트리뷰트를 추가하여 활성화할 수 있다. 이 실행기는 이름에서 알 수 있듯이, **단 하나의 OS 스레드** 위에서 모든 비동기 작업을 처리하는 이벤트 루프를 구성한다. 여러 스레드를 관리할 필요가 없으므로 스레드 간 동기화에 따른 오버헤드가 전혀 없고, 컨텍스트 스위칭 비용이 매우 낮다. 따라서 구조가 단순하고, 실행 흐름이 예측 가능하며, 리소스 사용량이 적다는 장점이 있다.4

본 프로젝트의 핵심 제약 조건인 "스레드를 사용하지 않는 아키텍처"를 충족시키기 위해, **`current_thread` 실행기를 채택한다.** 이는 스레드 생성 및 관리에 대한 복잡성을 완전히 제거하고, 모든 동시성 제어를 Tokio 런타임의 단일 이벤트 루프에 위임하는 가장 직접적이고 명확한 방법이다. 이 선택을 통해 우리는 OS 스레드와 관련된 잠재적인 문제들(경쟁 상태, 교착 상태 등)을 설계 단계에서부터 원천적으로 배제하고, 오직 비동기 작업(task)의 논리에만 집중하여 안정적이고 예측 가능한 시스템을 구축할 수 있다.


Rust의 비동기 모델과 Tokio의 단일 스레드 실행기에 대한 이해를 바탕으로, 이제 다수의 DJI Dock 2를 제어하기 위한 구체적인 스레드리스 아키텍처를 설계한다. 이 아키텍처는 단일 이벤트 루프를 중심으로 모든 I/O 작업을 비동기 태스크로 처리하여 높은 동시성과 안정성을 동시에 달성하는 것을 목표로 한다.


제안하는 시스템의 심장은 **단일 `current_thread` Tokio 런타임**이다. 이 런타임은 시스템의 모든 비동기 작업을 스케줄링하고 실행하는 중앙 이벤트 루프(Central Event Loop) 역할을 수행한다. 아키텍처의 핵심 구성 요소는 다음과 같다.

- **중앙 이벤트 루프:** Tokio 런타임 그 자체이며, 단일 OS 스레드에서 실행된다.
- **비동기 태스크(Asynchronous Tasks):** 시스템의 모든 논리적 작업 단위는 `tokio::spawn`을 통해 생성된 비동기 태스크로 추상화된다. 각 태스크는 독립적인 실행 흐름을 가지지만, OS 스레드가 아닌 경량의 협력적 태스크(green thread)이다.
- **I/O 처리:** 외부 시스템과의 모든 I/O(MQTT, DDS, Supabase API 호출)는 논블로킹(non-blocking) 방식으로 처리된다. 태스크가 I/O를 기다리는 동안에는 제어권을 이벤트 루프에 반납하여 다른 태스크가 실행될 수 있도록 한다.
- **상태 관리:** 각 DJI Dock의 연결 상태, 드론의 현재 위치, 임무 진행 상황 등 모든 상태 정보는 각 태스크 또는 공유 상태 관리 구조체 내에서 관리된다.

이 아키텍처에서는 수백, 수천 개의 DJI Dock이 동시에 연결되더라도, 이는 단지 수백, 수천 개의 경량 비동기 태스크가 이벤트 루프에 의해 관리되는 것을 의미할 뿐, 시스템이 사용하는 OS 스레드의 수는 여전히 1개를 유지한다. 이는 극도로 높은 동시성을 매우 적은 리소스만으로 달성할 수 있게 한다.

아래는 제안하는 아키텍처의 개념적 다이어그램이다.

```
+-----------------------------------------------------------------------------------+

| 관제 소프트웨어 (단일 프로세스, 단일 OS 스레드) |
| +-------------------------------------------------------------------------------+ |
| | Tokio Runtime (flavor = "current_thread") | |
| | (이벤트 루프) | |
| +-------+---------------------------+---------------------------+-------+-------+ |
| | | | | |
| +-------v-------+         +---------v----------+        +-------v-------+ |
| | MQTT 통신 | | Dock 관리 태스크 | | Supabase I/O | |
| | 태스크 (1개) |<------->| (N개, Dock별) |<------>| 태스크 | |
| | (rumqttc | | | | (DB/Storage) | |
| | EventLoop) | +--------------------+        +---------------+ |
| +-------+-------+                   ^ |
| | | |
| | | |
| | +---------------v---------------+ |
| +---------->| DDS 데이터 버스 |<--------------------+ |
| | (내부 컴포넌트 간 통신) | |
| +---------------^---------------+ |
| | |
| +---------------v---------------+ |
| | 비즈니스 로직/임무 관리 태스크| |
| +-------------------------------+ |
| |
+---------------------------------------------------------------------------+-------+
      ^ |

| MQTT (TCP/IP) | HTTPS
      v                                                                         v
+-----+-------------------------------------------------------------------------+-----+

| 외부 시스템 |
| +-----------------+   +-----------------+   +-----------------+   +-----------+ |
| | MQTT 브로커 | | DJI Dock 2 #1 |...| DJI Dock 2 #N | | Supabase | |
| +-----------------+   +-----------------+   +-----------------+   +-----------+ |
+-----------------------------------------------------------------------------------+
```


외부의 DJI Dock들과의 통신은 전적으로 MQTT 프로토콜을 통해 이루어진다. 이 계층의 설계는 시스템 전체의 반응성과 안정성에 직접적인 영향을 미친다.

- **라이브러리 선택:** **`rumqttc`** 크레이트를 채택한다. `rumqttc`는 Tokio와 완벽하게 통합되는 순수 Rust 구현의 고성능 비동기 MQTT 클라이언트로, 안정성과 유연성 면에서 검증되었다.17

- **구현 모델:** `rumqttc`는 클라이언트 로직과 네트워크 이벤트 루프를 명확하게 분리하는 우아한 모델을 제공한다. 이는 우리의 단일 스레드 아키텍처에 이상적으로 부합한다.

  1. 애플리케이션 시작 시, `rumqttc::AsyncClient::new()`를 호출하여 `client`와 `eventloop` 두 개의 객체를 얻는다.17
  2. `eventloop` 객체는 **단 하나의 전용 비동기 태스크**에서 `loop { eventloop.poll().await; }` 형태로 계속해서 폴링되어야 한다. 이 태스크가 MQTT 브로커와의 모든 TCP 네트워크 I/O, PING 메시지 전송, 재연결 로직 등을 전담하여 처리한다. 즉, 모든 MQTT 관련 네트워크 작업이 이 중앙 집중화된 태스크에서 논블로킹 방식으로 관리된다.19
  3. `client` 객체는 `Clone`이 가능하며, `Send`와 `Sync` 트레잇을 구현하고 있어 여러 태스크 간에 안전하게 복제하고 공유할 수 있다.18 다른 비즈니스 로직 태스크(예: 임무 관리 태스크)는 이 `client`의 복사본을 사용하여 필요할 때마다 MQTT 메시지를 발행(publish)할 수 있다.

- 메시지 처리 흐름:

  DJI Cloud API는 단일 MQTT 연결을 통해 토픽(topic)으로 각 장치를 구분하는 방식을 사용한다.20 따라서 우리 시스템은 MQTT 브로커와 단일 연결만 유지하고, 모든 대상 Dock 및 드론에 대한 토픽(예: `thing/product/+/osd`)을 와일드카드(`+`)를 사용하여 구독한다.

  1. `eventloop` 폴링 태스크는 브로커로부터 메시지를 수신하면, `Incoming` 알림(notification)을 받는다.
  2. 이 태스크는 수신된 메시지의 토픽 문자열(예: `thing/product/12345ABC/osd`)을 파싱하여 메시지의 출처가 되는 장치의 고유 시리얼 번호(`sn`)를 추출한다.
  3. 추출된 `sn`을 기반으로, 해당 메시지를 처리할 책임이 있는 컴포넌트(예: `sn` '12345ABC'를 관리하는 Dock 관리 태스크)로 메시지 페이로드를 전달한다. 이 전달 과정은 시스템 내부 통신 메커니즘인 DDS 또는 `tokio::sync::mpsc` 채널을 통해 이루어진다.

이 `client`/`eventloop` 분리 모델은 I/O 처리와 비즈니스 로직을 명확하게 분리한다. 단일 `eventloop` 태스크가 모든 저수준 네트워크 작업을 중앙에서 처리함으로써 리소스 관리를 효율화하고, 다수의 `client` 인스턴스는 각 비즈니스 로직이 독립적으로 외부로 명령을 보낼 수 있게 하여 코드의 모듈성과 테스트 용이성을 크게 향상시킨다.


시스템의 규모가 커지고 기능이 복잡해짐에 따라, 내부 컴포넌트 간의 데이터 흐름을 관리하는 것은 중요한 과제가 된다. DDS는 이 문제를 해결하기 위한 강력한 미들웨어다.

- **역할 정의:** DDS는 시스템 **내부**의 여러 비동기 태스크와 컴포넌트 간의 통신을 위한 고성능, 저지연, 실시간 데이터 버스(Data Bus)로 사용된다. MQTT가 외부 세계(Dock)와의 통신을 담당하는 '관문'이라면, DDS는 시스템 내부의 '중앙 신경망' 역할을 한다. 이를 통해 각 컴포넌트는 서로의 존재나 위치를 알 필요 없이, 오직 관심 있는 데이터 자체에만 집중할 수 있다(데이터 중심, data-centric publish-subscribe). 이는 컴포넌트 간의 결합도(coupling)를 극적으로 낮추고 시스템의 유지보수성과 확장성을 향상시킨다.
- **라이브러리 선택:** 순수 Rust로 구현된 DDS 라이브러리인 **`RustDDS`** 22 또는 **`dust-dds`** 23를 채택하는 것을 권장한다. 이들은 C 라이브러리에 대한 바인딩(예: `cyclonedds-rs` 25)과 달리 외부 의존성이 없고 빌드 과정이 단순하며, Rust의 비동기 모델과 네이티브하게 통합되도록 설계되었다. 특히 `async` API를 명시적으로 지원하는 라이브러리는 Tokio 이벤트 루프와의 통합을 원활하게 한다.22
- **구현 모델:**
  1. 시스템 초기화 시, DDS `DomainParticipant`를 생성한다.
  2. 시스템 내부에서 교환될 데이터의 종류에 따라 DDS `Topic`을 정의한다. 예를 들어, `DroneTelemetry`, `MissionStatusUpdate`, `ControlCommandRequest`, `VideoStreamSignal` 등의 토픽을 생성할 수 있다. 각 토픽은 `serde`로 직렬화 가능한 Rust 구조체와 매핑된다.
  3. **발행(Publish) 예시:** MQTT 통신 태스크가 드론으로부터 OSD 메시지를 수신하면, 이를 파싱하여 `DroneTelemetry` 구조체로 변환한 뒤, `DroneTelemetry` 토픽에 대한 DDS `DataWriter`를 사용하여 데이터를 발행(write)한다.
  4. **구독(Subscribe) 예시:** 지도 UI를 업데이트하는 태스크와 비행 데이터를 로깅하는 태스크는 각각 `DroneTelemetry` 토픽에 대한 DDS `DataReader`를 생성한다. 이 `DataReader`들은 새로운 데이터가 발행될 때마다 비동기적으로 이를 수신하여 각자의 로직을 수행한다.

DDS의 도입은 처음에는 오버헤드처럼 보일 수 있다. 그러나 "다수의 드론"을 실시간으로 관제하는 복잡한 시스템에서는 필수적인 추상화 계층이다. DDS의 풍부한 QoS(Quality of Service) 설정(신뢰성, 내구성, 데드라인 등)은 실시간 관제 시스템의 요구사항을 충족시키는 데 매우 유용하며 23, 컴포넌트들이 서로를 직접 참조하지 않게 함으로써 시스템이 유기적으로 성장하고 변경될 수 있는 토대를 마련한다.


Supabase는 PostgreSQL 데이터베이스, 사용자 인증, 파일 저장소 등을 통합하여 제공하는 오픈소스 BaaS(Backend as a Service) 플랫폼이다.26 본 시스템에서는 백엔드 인프라 구축의 복잡성을 줄이고 개발 속도를 높이기 위해 Supabase를 적극적으로 활용한다.

- **역할 정의:**

  - **PostgreSQL 데이터베이스:** 비행 기록(flight logs), 임무 계획(wayline missions), 장치 정보, 사용자 계정 등 영속적으로 저장해야 하는 모든 정형 데이터를 관리한다.
  - **Storage:** 드론이 촬영한 고용량 미디어 파일(사진, 영상)을 저장하고 안전하게 제공하는 역할을 한다.
  - **Authentication:** 관제 시스템에 접근하는 운영자의 계정을 관리하고 인증하는 데 사용된다.

- **라이브러리 선택:** **`supabase-rs`** 27 또는 

  **`suparust`** 28와 같은 커뮤니티 기반의 비동기 Rust 클라이언트 라이브러리를 사용한다. 이 라이브러리들은 내부적으로 

  `reqwest`와 같은 검증된 비동기 HTTP 클라이언트를 사용하므로, Tokio 이벤트 루프 위에서 논블로킹 방식으로 원활하게 동작한다.

- 구현 모델:

  Supabase와의 모든 상호작용은 네트워크 호출을 수반하므로, 잠재적인 지연 시간(latency)을 가지고 있다. 따라서 Supabase 연동 로직은 시스템의 핵심 실시간 루프와 반드시 분리되어야 한다.

  1. 시스템 초기화 시, Supabase 클라이언트를 생성하고 `Arc<Client>` 형태로 여러 태스크에서 안전하게 공유할 수 있도록 준비한다.
  2. 데이터베이스에 데이터를 기록해야 할 때(예: 비행 로그 저장), 해당 작업을 전담하는 별도의 '로거(logger)' 태스크를 둔다.
  3. 이 로거 태스크는 DDS의 `DroneTelemetry` 토픽 등을 구독하고 있다가, 데이터를 수신하면 내부 버퍼에 쌓아둔다.
  4. 버퍼가 일정 크기에 도달하거나 특정 시간이 경과하면, 로거 태스크는 `tokio::spawn`을 통해 새로운 비동기 작업을 생성하여 Supabase 클라이언트의 비동기 메소드(예: `client.from("flight_logs").insert(buffered_data).execute().await`)를 호출한다.
  5. 이 `await` 지점에서 데이터베이스 I/O 작업이 완료될 때까지 해당 로거 태스크만 대기 상태가 되며, 시스템의 메인 이벤트 루프는 다른 실시간 작업(MQTT 메시지 처리 등)을 지연 없이 계속 처리한다.

이러한 '수신/처리'와 '영속화'의 분리 패턴은 외부 서비스의 지연 시간이 시스템 전체의 반응성을 저해하는 것을 막는 매우 중요한 아키텍처 원칙이다. 실시간 데이터는 먼저 DDS를 통해 메모리 내에서 신속하게 처리 및 전파되고, 영속화는 후순위 작업으로 비동기적이고 배치(batch) 방식으로 처리함으로써 시스템의 안정성과 반응성을 모두 확보할 수 있다.


설계된 아키텍처 위에서 DJI Dock 2의 구체적인 제어 기능을 구현하는 방안을 상세히 기술한다. 이는 DJI Cloud API의 사양을 Rust 코드로 변환하고, 시스템의 각 컴포넌트와 연동하는 과정을 포함한다.


DJI Cloud API를 통해 교환되는 모든 MQTT 메시지는 JSON 형식의 페이로드를 사용한다.29 견고하고 타입-안전한(type-safe) 소프트웨어를 구축하기 위해, 이 JSON 구조를 Rust의 `struct`로 정밀하게 모델링하는 것이 최우선 과제다. 이를 위해 Rust 생태계의 표준 직렬화/역직렬화 라이브러리인 `serde`와 `serde_json`을 사용한다.

`osd`, `state`, `services`, `events` 등 DJI가 정의한 각 MQTT 토픽의 페이로드 스키마에 맞춰 Rust 구조체를 정의한다. 예를 들어, `osd` 메시지에 포함된 드론의 위치 정보를 담기 위해 다음과 같은 구조체를 정의할 수 있다.

```Rust
use serde::{Deserialize, Serialize};

#
pub struct OsdData {
    pub longitude: f64,
    pub latitude: f64,
    pub height: f32,
    //... 기타 OSD 필드들
}

#
pub struct StateData {
    pub mode_code: u32,
    pub firmware_version: String,
    //... 기타 상태 필드들
}
```

이러한 구조체 정의는 컴파일 타임에 데이터의 유효성을 검사할 수 있게 하고, JSON 필드 이름의 오타와 같은 런타임 오류를 방지하며, 코드의 가독성과 유지보수성을 크게 향상시킨다. DJI API 문서에 흩어져 있는 방대한 정보들을 체계적으로 정리하기 위해, 주요 MQTT 토픽과 해당 데이터를 처리하는 Rust 구조체를 매핑한 다음 표를 핵심 참조 자료로 활용한다.

**Table 1: DJI Dock 2 주요 MQTT 토픽 및 데이터 구조 매핑**

| Topic Template                              | Direction    | Purpose                                      | Key JSON Fields / Rust Struct                                | Snippet Refs |
| ------------------------------------------- | ------------ | -------------------------------------------- | ------------------------------------------------------------ | ------------ |
| `thing/product/{device_sn}/osd`             | Dock -->> Cloud | 실시간 원격 측정 데이터 (0.5Hz 주기)         | `longitude`, `latitude`, `altitude`, `velocity_x`, `battery_percent` / `struct OsdData` | 21           |
| `thing/product/{device_sn}/state`           | Dock -->> Cloud | 상태 변경 시 보고되는 데이터                 | `mode_code`, `firmware_version`, `live_status`, `job_status` / `struct StateData` | 21           |
| `thing/product/{gateway_sn}/services`       | Cloud -->> Dock | 클라우드에서 장치로 보내는 명령(서비스 호출) | `method` (e.g., `flighttask_execute`), `data` (명령 파라미터) / `struct ServiceRequest<T>` | 21           |
| `thing/product/{gateway_sn}/services_reply` | Dock -->> Cloud | 서비스 명령에 대한 응답                      | `result` (성공/실패 코드), `output` (결과 데이터) / `struct ServiceReply<T>` | 21           |
| `thing/product/{gateway_sn}/events`         | Dock -->> Cloud | 장치에서 발생하는 주요 이벤트 알림           | `method` (e.g., `flighttask_progress`), `data` (이벤트 데이터) / `struct EventNotification<T>` | 21           |
| `sys/product/{gateway_sn}/status`           | Dock -->> Cloud | 장치 온라인/오프라인, 토폴로지 업데이트      | `online`, `offline`, `sub_devices` / `struct SystemStatus`   | 21           |


드론의 실시간 위치, 속도, 배터리 등의 OSD 데이터와 비행 모드, 장비 상태 등의 상태 정보는 관제 시스템의 가장 기본적인 데이터 흐름이다.

1. **수신 및 역직렬화:** 중앙 `eventloop` 태스크는 `thing/product/{device_sn}/osd` 및 `.../state` 토픽에서 메시지를 수신한다.
2. 수신된 원시 바이트(byte) 페이로드는 `serde_json::from_slice` 함수를 사용하여 사전에 정의된 `OsdData` 또는 `StateData` Rust 구조체로 역직렬화(deserialize)된다. 이 과정에서 JSON 형식이 맞지 않거나 필수 필드가 누락된 경우 즉시 오류로 처리할 수 있다.
3. **내부 전파:** 성공적으로 역직렬화된 데이터 객체는 DDS 데이터 버스의 해당 토픽(예: `DdsDroneOsd`, `DdsDroneState`)으로 발행된다. 이때, 데이터의 출처를 명확히 하기 위해 DDS 토픽의 키(key) 필드에 드론의 시리얼 번호(`sn`)를 사용한다.
4. **처리 및 영속화:** 지도 UI 업데이트 태스크, 경고 시스템 태스크, 데이터 로거 태스크 등은 각각 필요한 DDS 토픽을 구독한다. 이들은 새로운 데이터를 수신할 때마다 각자의 로직(화면 갱신, 위험 상황 판단, 데이터베이스 기록 등)을 수행한다. 특히 데이터 로거 태스크는 앞서 설계한 바와 같이, 데이터를 버퍼링하여 Supabase에 비동기적으로 배치 삽입(batch insert)한다.


관제 시스템에서 드론에게 비행 임무를 시작시키거나 특정 동작을 수행하도록 명령하는 것은 핵심 기능이다.

1. **명령 생성:** 사용자 인터페이스(UI)나 자동화된 스케줄러에서 '임무 시작'과 같은 요청이 발생하면, 이는 시스템 내부의 `ControlCommand`와 같은 DDS 토픽으로 발행된다. 이 DDS 메시지에는 대상 드론의 `sn`과 임무 ID 등의 정보가 포함된다.

2. **임무 관리 태스크:** 이 시스템의 중앙 두뇌 역할을 하는 '임무 관리 태스크'는 `ControlCommand` DDS 토픽을 구독하고 있다가 명령을 수신한다.

3. **MQTT 메시지 구성:** 임무 관리 태스크는 수신된 명령을 바탕으로 DJI Cloud API 규격에 맞는 `services` 요청 JSON 페이로드를 생성한다. 예를 들어, 비행 임무를 실행하기 위해 `method`를 `flighttask_execute`로 설정하고 `data` 필드에 필요한 임무 파일 ID 등을 포함하는 `ServiceRequest` 구조체를 만든다.32 이 구조체는 

   `serde_json::to_vec`을 사용하여 JSON 바이트 벡터로 직렬화된다.

4. **명령 발행:** 직렬화된 JSON 페이로드는 `rumqttc::AsyncClient`의 `publish` 메소드를 통해 해당 드론의 `services` 토픽(예: `thing/product/{gateway_sn}/services`)으로 발행된다.

5. **피드백 루프:** 드론은 명령을 수신하고 처리한 후, `services_reply` 토픽을 통해 응답을 보낸다. 또한, 임무가 진행됨에 따라 `flighttask_progress`와 같은 이벤트를 `events` 토픽으로 지속적으로 보고한다.32 이 정보들은 다시 MQTT 통신 계층을 통해 수신되어 DDS를 통해 임무 관리 태스크와 UI에 전파됨으로써 완전한 폐쇄 루프 제어(closed-loop control)를 형성한다.


많은 사용자들이 오해하는 부분은 MQTT를 통해 영상 데이터 자체가 전송될 것이라는 점이다. 그러나 이는 사실이 아니며, 이 점을 명확히 이해하는 것이 올바른 아키텍처 설계의 핵심이다.

MQTT는 경량 메시징 프로토콜로서, 수십 킬로바이트(KB) 이상의 대용량 바이너리 데이터를 지속적으로 스트리밍하는 데 적합하지 않다. 실시간 영상은 초당 수 메가비트(Mbps)의 대역폭을 요구하므로, 이를 MQTT로 전송하려는 시도는 브로커와 클라이언트에 즉각적인 과부하를 유발하여 시스템 전체를 마비시킬 것이다.

대신, DJI Cloud API에서 **MQTT는 영상 스트리밍을 시작하고 중지하기 위한 신호(Signaling) 채널**로만 사용된다.

1. **아키텍처 요구사항:** 관제 시스템은 WebRTC/WHIP 35 또는 RTMP 37 프로토콜을 지원하는 별도의 미디어 서버(예: Janus, SRS, LiveKit, Ant Media Server 등)와 연동되어야 한다.

2. 스트리밍 시작 흐름:

   a. 관제 시스템의 프론트엔드(웹 UI)에서 특정 드론의 영상 보기를 요청한다.

   b. 관제 소프트웨어(백엔드)는 연동된 미디어 서버에 해당 드론을 위한 스트림 수신 엔드포인트(ingest endpoint) 생성을 요청한다.

   c. 미디어 서버는 WHIP URL(예: http://media.server/rtc/v1/whip/?app=live&stream=drone123)과 같은 주소를 반환한다.

   d. 관제 소프트웨어는 이 URL을 포함하여 live_start_push MQTT 명령을 구성한다. 이 명령의 data 필드에는 url_type을 WebRTC로, url을 미디어 서버로부터 받은 주소로 설정한다.37

   e. 이 MQTT 명령은 대상 Dock의 services 토픽으로 전송된다.

   f. Dock/드론은 이 명령을 받고, 내장된 WebRTC 클라이언트를 사용하여 지정된 URL로 영상 스트림을 직접 송출하기 시작한다.

3. **영상 재생:** 프론트엔드 UI는 관제 소프트웨어나 MQTT를 통하는 것이 아니라, 미디어 서버로부터 직접 WebRTC(또는 다른 프로토콜) 스트림을 구독하여 영상을 재생한다.

이처럼 MQTT는 "어디로 영상을 보낼지"를 지시하는 제어 평면(control plane)의 역할을 하고, 실제 영상 데이터는 미디어 서버를 통해 데이터 평면(data plane)에서 완전히 분리되어 흐른다. 이 아키텍처는 각 프로토콜을 그 목적에 맞게 사용하여 시스템 전체의 효율성과 확장성을 보장한다.


제안된 스레드리스 아키텍처는 단순성과 안정성 면에서 큰 이점을 제공하지만, 그 특성상 발생하는 제약과 성능 한계를 명확히 이해하고 이에 대한 대비책을 마련해야 한다.


단일 스레드 기반의 협력적 멀티태스킹 모델이 가진 가장 큰 위험은 **블로킹(blocking) 호출**이다. 이 모델의 핵심은 모든 비동기 태스크가 자발적으로 제어권을 양보(`yield`)하여 다른 태스크에게 실행 기회를 주는 것이다. 만약 하나의 태스크가 스레드를 점유한 채로 장시간 동안 제어권을 양보하지 않으면, 이벤트 루프에 있는 다른 모든 태스크는 실행 기회를 얻지 못하고 '기아 상태(starvation)'에 빠진다. 이는 사용자 관점에서 시스템 전체가 응답하지 않는 '프리즈(freeze)' 현상으로 나타난다.5

**주요 위험 요소:**

- **동기식 I/O:** `std::fs::read`나 `std::net::TcpStream::read`와 같은 표준 라이브러리의 블로킹 I/O 함수를 직접 호출하는 경우.
- **긴 CPU 연산:** 암호화, 데이터 압축/해제, 복잡한 수학 계산 등 CPU를 많이 사용하는 코드가 `.await` 없이 연속적으로 실행되는 경우.
- **잘못 구현된 외부 라이브러리:** 비동기 함수처럼 보이지만 내부적으로 블로킹 호출을 숨기고 있는 라이브러리를 사용하는 경우.

**회피 및 대응 전략:**

1. **비동기 API 일관성 유지:** 모든 I/O 작업은 반드시 `tokio`가 제공하는 비동기 API(예: `tokio::fs`, `tokio::net`, 비동기 데이터베이스 드라이버)를 사용해야 한다. 이들은 내부적으로 논블로킹 I/O와 `Waker` 메커니즘을 사용하여 이벤트 루프와 원활하게 통합된다.
2. **CPU 집약적 작업 분할 및 양보:** 반드시 수행해야 하는 CPU 집약적 작업이 있다면, 이를 가능한 한 작은 단위의 청크(chunk)로 나누고 각 청크 처리 사이에 `tokio::task::yield_now().await`를 명시적으로 호출해야 한다. 이는 현재 태스크의 실행을 잠시 중단하고 다른 태스크에게 실행 기회를 강제로 양보하는 역할을 하여 이벤트 루프의 응답성을 유지한다.
3. **`spawn_blocking`의 신중한 사용:** 동기식 I/O나 외부 라이브러리 호출과 같이 수정이 불가능한 블로킹 코드를 반드시 실행해야 하는 경우가 있다. 이때를 위해 Tokio는 `tokio::task::spawn_blocking`이라는 최후의 수단을 제공한다. 이 함수는 인자로 받은 클로저를 Tokio가 관리하는 별도의 **블로킹 전용 스레드 풀**에서 실행시킨다. 이를 통해 메인 이벤트 루프 스레드의 블로킹을 방지할 수 있다.5 이는 "스레드를 사용하지 않는다"는 원래의 제약 조건에 대한 실용적인 예외이자 타협이다. 이 기능은 시스템의 응답성을 지키기 위해 불가피할 때만, 그 영향(별도 스레드 생성 및 컨텍스트 스위칭 비용)을 명확히 인지하고 제한적으로 사용해야 한다.


분산 시스템인 드론 관제 시스템은 네트워크 단절, 장비 오작동, 데이터 파손 등 다양한 오류 상황에 직면할 수 있다. 시스템의 복원력(resilience)은 이러한 오류 상황에서도 안정적으로 동작을 계속하거나 우아하게 복구하는 능력에 달려있다.

- **명시적 오류 처리:** Rust의 `Result<T, E>` 열거형과 `?` 연산자는 오류 처리를 코드 흐름의 일부로 강제한다. 네트워크 통신, JSON 파싱, 데이터베이스 연동 등 실패할 수 있는 모든 연산은 `Result`를 반환하도록 하고, 호출부에서 이를 명시적으로 처리하여 예기치 않은 `panic`을 방지한다.
- **자동 재연결:** `rumqttc` 라이브러리는 MQTT 브로커와의 연결이 끊어졌을 때 자동으로 재연결을 시도하는 기능을 내장하고 있다. `eventloop.poll()`은 이 과정에서 발생하는 `Disconnect`, `Reconnect`와 같은 이벤트를 애플리케이션에 알려주므로 17, 연결 상태의 변화를 실시간으로 감지하여 로깅하거나 UI에 상태를 표시하는 등의 후속 조치를 취할 수 있다.
- **견고한 태스크 루프:** 각 Dock을 관리하는 태스크는 `tokio::select!` 매크로를 사용하여 견고한 이벤트 처리 루프를 구성할 수 있다. `select!`는 여러 비동기 분기(branch)를 동시에 기다리다가, 가장 먼저 완료되는 하나의 분기만 실행한다. 이를 통해 'MQTT 메시지 수신', 'DDS 내부 명령 수신', '상태 확인 타임아웃' 등 여러 종류의 이벤트를 단일 루프 내에서 효율적이고 안전하게 처리할 수 있다.
- **`panic` 격리:** Tokio에서 `tokio::spawn`으로 생성된 태스크는 독립적인 실행 단위다. 만약 하나의 태스크에서 복구 불가능한 오류로 인해 `panic`이 발생하더라도, 이 `panic`은 해당 태스크 내에서 격리된다. 다른 태스크들은 (패닉한 태스크가 공유 자원에 대한 락을 쥐고 있지 않는 한) 영향을 받지 않고 계속해서 정상적으로 실행될 수 있다. 이는 시스템 전체의 안정성을 높이는 중요한 특징이다.


`current_thread` 실행기 기반의 아키텍처는 본질적으로 **단일 CPU 코어의 성능**에 의해 그 처리 용량이 제한된다. 수천 개의 Dock과 동시에 I/O 작업을 수행할 수는 있지만, 모든 데이터 처리, 로직 연산, 직렬화/역직렬화 등의 CPU 작업은 단 하나의 코어에서 순차적으로(물론 협력적 멀티태스킹을 통해 시분할로) 처리된다. 따라서 이 아키텍처는 대부분의 시간을 네트워크 응답 대기에 사용하는 **I/O 바운드** 시나리오에 최적화되어 있으며, CPU 연산량이 많은 **CPU 바운드** 작업에는 적합하지 않다.39

- **수직적 확장(Vertical Scaling):** 가장 간단한 확장 방법은 더 빠른 클럭 속도와 더 큰 캐시를 가진 CPU를 사용하는 것이다. 이는 한계가 명확하지만 초기 단계에서는 효과적일 수 있다.
- **수평적 확장(Horizontal Scaling):** "다수의 Dock"이라는 요구사항을 충족하고 단일 코어의 성능 한계를 근본적으로 극복하기 위한 가장 현실적이고 강력한 전략은 **수평적 확장**이다.
  1. **개념:** 단일 스레드로 동작하는 관제 소프트웨어 인스턴스를 여러 개 실행한다. 예를 들어, 16코어 서버가 있다면, 16개의 관제 소프트웨어 프로세스를 각각 실행하여 모든 CPU 코어를 활용할 수 있다.
  2. **작업 분할:** 전체 DJI Dock 집합을 여러 그룹으로 나눈 뒤, 각 관제 소프트웨어 인스턴스가 하나의 그룹을 전담하도록 한다. 예를 들어, 1600개의 Dock이 있다면 각 인스턴스는 100개의 Dock을 담당한다.
  3. **동적 할당 및 조정:** 어떤 인스턴스가 어떤 Dock 그룹을 담당할지 결정하는 메커니즘이 필요하다.
     - **정적 분할:** 설정 파일을 통해 각 인스턴스에 담당할 Dock의 시리얼 번호 목록을 명시적으로 할당하는 가장 간단한 방법.
     - **동적 분할:** 더 유연하고 고가용성을 제공하는 방법. 모든 인스턴스가 Supabase 데이터베이스나 ZooKeeper/etcd와 같은 외부 코디네이션 서비스를 참조한다. 인스턴스가 시작될 때, 코디네이션 서비스에 자신을 등록하고 아직 아무도 담당하지 않는 Dock 그룹을 할당받는다. 만약 특정 인스턴스가 비정상 종료되면, 다른 인스턴스가 이를 감지하고 해당 인스턴스가 담당하던 Dock 그룹을 인계받는 '페일오버(failover)' 로직을 구현할 수 있다.

이러한 수평적 확장 모델을 통해, 각 인스턴스는 단일 스레드의 단순성과 안정성을 그대로 유지하면서도 시스템 전체적으로는 서버의 모든 CPU 코어를 활용하여 이론상 거의 무한한 수의 Dock을 지원할 수 있다. 이는 단일 스레드 모델의 본질적 한계를 우회하는 가장 성숙하고 검증된 아키텍처 패턴이다.



본 보고서에서 제안한 스레드리스 아키텍처는 Rust의 `async/await`와 Tokio의 `current_thread` 실행기를 핵심 기반으로 삼아, 사용자가 제시한 "스레드 회피"라는 엄격한 제약 조건을 만족시키면서도 다수의 DJI Dock 2를 실시간으로 관제할 수 있는 현실적이고 강력한 방안임을 확인했다.

이 아키텍처는 협력적 멀티태스킹 모델을 채택함으로써, I/O 작업이 지배적인 드론 관제 시나리오에서 극도로 높은 동시성을 효율적으로 처리할 수 있다. 또한, 전통적인 멀티스레딩에서 발생하는 데이터 경쟁, 교착 상태, 복잡한 동기화 문제들을 설계 단계에서부터 원천적으로 제거하여 시스템의 전반적인 안정성과 예측 가능성을 크게 향상시킨다. 블로킹 호출의 위험성이라는 협력적 멀티태스킹의 내재적 한계는 `spawn_blocking`과 같은 명확한 회피 전략을 통해 관리할 수 있으며, 단일 코어의 성능 한계는 수평적 확장 전략을 통해 극복할 수 있다. 따라서 제안된 아키텍처는 안정성, 성능, 확장성의 균형을 맞춘 합리적인 설계라 결론 내릴 수 있다.


이상의 분석을 바탕으로, 프로젝트에 사용할 최종 기술 스택을 다음과 같이 권장한다.

- **비동기 런타임:** `tokio` (반드시 `flavor = "current_thread"` 옵션과 함께 사용)
- **MQTT 클라이언트:** `rumqttc` (비동기 `AsyncClient` 및 `Eventloop` 모델 활용)
- **내부 데이터 버스:** `dust-dds` (순수 Rust 구현 및 `async` API 지원)
- **백엔드/데이터베이스:** `Supabase` (Rust 클라이언트로 `supabase-rs` 또는 `suparust` 사용)
- **데이터 직렬화:** `serde` 및 `serde_json` (타입-안전한 JSON 처리)

이 스택은 각 영역에서 검증된 고품질의 라이브러리들로 구성되어 있으며, 모두 Rust의 비동기 생태계와 원활하게 통합되어 제안된 아키텍처를 구현하는 데 최적의 조합을 제공한다.


복잡한 시스템을 성공적으로 구축하기 위해서는 점진적이고 단계적인 접근이 필수적이다. 다음은 권장하는 단계별 구현 로드맵이다.

1. **Phase 1 (코어 기능 구현 및 검증):**
   - 목표: 단일 DJI Dock과의 기본적인 MQTT 통신 검증.
   - 작업:
     - Tokio `current_thread` 런타임 환경 설정.
     - `rumqttc`를 사용하여 MQTT 브로커에 연결하고, 특정 Dock의 `osd` 및 `state` 토픽을 구독.
     - 수신된 메시지를 `serde_json`으로 파싱하여 콘솔에 출력.
     - 간단한 서비스(예: `drone_arm`)를 `services` 토픽으로 전송하는 기능 구현.
2. **Phase 2 (데이터 영속화 및 상태 관리):**
   - 목표: 수신된 데이터를 저장하고, 다수의 Dock 상태를 관리하는 기반 마련.
   - 작업:
     - `supabase-rs` 클라이언트를 연동하여 Supabase 데이터베이스에 연결.
     - 수신된 OSD 및 상태 데이터를 비동기적으로 데이터베이스에 기록하는 로거 태스크 구현.
     - 메모리 내에서 여러 Dock의 상태를 `HashMap<String, DroneState>`과 같은 자료구조로 관리하는 '상태 관리자' 구현.
3. **Phase 3 (내부 아키텍처 고도화):**
   - 목표: DDS를 도입하여 컴포넌트 간 통신을 리팩토링하고, 복잡한 로직 분리.
   - 작업:
     - `dust-dds`를 프로젝트에 통합.
     - 주요 내부 데이터 모델(Telemetry, MissionStatus 등)에 대한 DDS 토픽 정의.
     - MQTT 수신부, 상태 관리자, DB 로거 간의 통신을 기존의 직접 호출 방식에서 DDS pub/sub 모델로 전환.
     - 임무 관리, UI 업데이트 등 복잡한 비즈니스 로직을 별도의 DDS 구독 태스크로 분리.
4. **Phase 4 (확장성 확보 및 안정화):**
   - 목표: 대규모 운영을 위한 수평적 확장 전략 설계 및 시스템 안정화.
   - 작업:
     - 5.3절에서 논의된 수평적 확장 전략(정적 또는 동적 분할)을 설계하고 프로토타입 구현.
     - 오류 처리 로직 강화: 네트워크 단절, `panic` 복구 등 다양한 예외 상황에 대한 테스트 및 코드 보강.
     - 로깅 및 모니터링 시스템(`tracing` 크레이트 등)을 도입하여 운영 중인 시스템의 상태를 관찰할 수 있는 기반 마련.41
5. **Phase 5 (고급 기능 추가):**
   - 목표: 핵심 기능 외 부가적인 고급 기능 구현.
   - 작업:
     - 4.4절에서 설계한 영상 스트림 제어(signaling) 기능 구현 및 미디어 서버 연동.
     - WPML 기반의 복잡한 웨이라인(wayline) 임무 생성, 업로드, 실행, 모니터링 기능 구현.32
     - 사용자 인증 및 권한 관리 기능 고도화.

이러한 단계적 접근을 통해 팀은 각 단계에서 명확한 목표를 가지고 개발을 진행할 수 있으며, 조기에 시스템의 핵심 기능을 검증하고 점진적으로 복잡성을 추가해 나감으로써 프로젝트의 리스크를 효과적으로 관리할 수 있을 것이다.


1. The State of Async Rust: Runtimes, accessed August 11, 2025, https://corrode.dev/blog/async/
2. How Asynchronous Programming Works in Rust – Futures and Async/Await Explained with Examples - freeCodeCamp, accessed August 11, 2025, https://www.freecodecamp.org/news/how-asynchronous-programming-works-in-rust/
3. Async/await vs threads/atomics and when you use each? : r/rust - Reddit, accessed August 11, 2025, https://www.reddit.com/r/rust/comments/jgpvi3/asyncawait_vs_threadsatomics_and_when_you_use_each/
4. Making the Tokio scheduler 10x faster | Tokio - An asynchronous Rust runtime, accessed August 11, 2025, https://tokio.rs/blog/2019-10-scheduler
5. Tokio A Deep Dive into Concurrency Vs. Parallelism - Ian Bull, accessed August 11, 2025, https://ianbull.com/posts/tokio-parallel/
6. async/await - Asynchronous Programming in Rust, accessed August 11, 2025, https://rust-lang.github.io/async-book/03_async_await/01_chapter.html
7. Async/Await in Rust: A Beginner's Guide | by Leapcell - Medium, accessed August 11, 2025, https://leapcell.medium.com/async-await-in-rust-a-beginners-guide-8752d2c2abbf
8. Understanding Async Await in Rust: From State Machines to Assembly Code - EventHelix, accessed August 11, 2025, https://www.eventhelix.com/rust/rust-to-assembly-async-await
9. Can someone explain how runtimes for async and await work on an assembly level?, accessed August 11, 2025, https://users.rust-lang.org/t/can-someone-explain-how-runtimes-for-async-and-await-work-on-an-assembly-level/130413
10. async/.await Primer - Asynchronous Programming in Rust, accessed August 11, 2025, https://rust-lang.github.io/async-book/01_getting_started/04_async_await_primer.html
11. async-rs/async-std: Async version of the Rust standard library - GitHub, accessed August 11, 2025, https://github.com/async-rs/async-std
12. Tokio - An asynchronous Rust runtime, accessed August 11, 2025, https://tokio.rs/
13. Building an Async Runtime with mio - Blog - Tweede golf, accessed August 11, 2025, https://tweedegolf.nl/en/blog/114/building-an-async-runtime-with-mio
14. Async in depth | Tokio - An asynchronous Rust runtime, accessed August 11, 2025, https://tokio.rs/tokio/tutorial/async
15. [post] Tasks are the wrong abstraction : r/rust - Reddit, accessed August 11, 2025, https://www.reddit.com/r/rust/comments/1cedhxi/post_tasks_are_the_wrong_abstraction/
16. rumqtt/rumqttc/examples/topic_alias.rs at main - GitHub, accessed August 11, 2025, https://github.com/bytebeamio/rumqtt/blob/main/rumqttc/examples/topic_alias.rs
17. rumqtt/rumqttc/README.md at main - GitHub, accessed August 11, 2025, https://github.com/bytebeamio/rumqtt/blob/main/rumqttc/README.md
18. AsyncClient in rumqttc - Rust - Docs.rs, accessed August 11, 2025, https://docs.rs/rumqttc/latest/rumqttc/struct.AsyncClient.html
19. Rumqttc Internals | RUMQTT - Bytebeam, accessed August 11, 2025, [https://rumqtt.bytebeam.io/docs/rumqttc/Developer%20Guide/rumqttc%20internals/](https://rumqtt.bytebeam.io/docs/rumqttc/Developer Guide/rumqttc internals/)
20. MQTT - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/mqtt.html
21. Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/topic-definition.html
22. Atostek/RustDDS: Rust implementation of Data Distribution Service - GitHub, accessed August 11, 2025, https://github.com/Atostek/RustDDS
23. s2e-systems/dust-dds: Rust implementation of the Data Distribution Service (DDS) - GitHub, accessed August 11, 2025, https://github.com/s2e-systems/dust-dds
24. dust_dds - Rust - Docs.rs, accessed August 11, 2025, https://docs.rs/dust_dds
25. cyclonedds_rs - Rust - Docs.rs, accessed August 11, 2025, https://docs.rs/cyclonedds-rs
26. Supabase | The Postgres Development Platform., accessed August 11, 2025, https://supabase.com/
27. supabase_rs - crates.io: Rust Package Registry, accessed August 11, 2025, https://crates.io/crates/supabase_rs
28. suparust - Rust - Docs.rs, accessed August 11, 2025, https://docs.rs/suparust
29. Thing Model - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/overview/basic-concept/thing-model.html
30. Device Property List - Cloud API, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock2/properties.html
31. Properties - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/pilot-to-cloud/mqtt/rc-plus/properties.html
32. Wayline Management - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-wayline-management.html
33. Wayline Management - Event | Cloud API, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock2/wayline.html
34. Device Management - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-device-management.html
35. Live Stream - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/pilot-feature-set/pilot-livestream.html
36. Live Stream - Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/feature-set/dock-feature-set/dock-livestream.html
37. Service | Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock1/live.html
38. Service | Cloud API - DJI Developer, accessed August 11, 2025, https://developer.dji.com/doc/cloud-api-tutorial/en/api-reference/dock-to-cloud/mqtt/dock/dock3/live.html
39. async thread vs std thread - Stack Overflow, accessed August 11, 2025, https://stackoverflow.com/questions/78541829/async-thread-vs-std-thread
40. Rust Concurrency: When to Use (and Avoid) Async Runtimes | by Leapcell - Medium, accessed August 11, 2025, https://leapcell.medium.com/rust-concurrency-when-to-use-and-avoid-async-runtimes-43556cff6b62
41. Observing Tokio - RustLab, accessed August 11, 2025, https://rustlab.it/talks/observing-tokio
42. Cloud-API-Doc/docs/en/60.api-reference/00.dji-wpml/10.overview.md at master / dji-sdk ... - GitHub, accessed August 11, 2025, https://github.com/dji-sdk/Cloud-API-Doc/blob/master/docs/en/60.api-reference/00.dji-wpml/10.overview.md
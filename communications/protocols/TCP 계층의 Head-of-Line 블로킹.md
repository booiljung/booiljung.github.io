# TCP 계층의 Head-of-Line 블로킹에 대한 심층 고찰

## 제1장: 서론 - Head-of-Line 블로킹의 본질과 계층적 이해

### 0.1  HOL 블로킹의 정의와 원리

Head-of-Line(HOL) 블로킹은 컴퓨터 네트워킹 및 패킷 교환 시스템에서 발생하는 근본적인 성능 저하 현상으로, 큐(Queue)의 첫 번째(Head)에 위치한 패킷이 어떠한 이유로 지연될 경우, 그 뒤에 있는 후속 패킷들의 처리까지 모두 막는 병목 현상을 지칭한다.1 이는 큐잉 이론(Queueing Theory)의 관점에서 선입선출(First-In, First-Out, FIFO) 원칙을 따르는 시스템에서 필연적으로 발생할 수 있는 문제이다. 후속 패킷들이 첫 번째 패킷과는 무관한 목적지로 향하거나, 이미 처리될 준비가 완료되었음에도 불구하고, 단지 큐의 순서 때문에 대기해야 하는 비효율을 초래한다.2

이 현상은 다양한 시스템에서 관찰된다. 예를 들어, 입력 버퍼(Input-buffered)를 사용하는 네트워크 스위치에서는 특정 출력 포트가 혼잡하여 첫 번째 패킷이 대기할 경우, 다른 비어있는 출력 포트로 가야 할 후속 패킷들까지 입력 버퍼에서 대기하게 된다.1 또한, HTTP/1.1의 파이프라이닝(Pipelining) 기술이나, 신뢰성 있는 전송 프로토콜에서 발생하는 순서가 어긋난 패킷(Out-of-order packet)의 처리 과정에서도 HOL 블로킹은 핵심적인 성능 제약 요소로 작용한다.1

HOL 블로킹의 발생에는 두 가지 핵심 요소가 기여한다. 첫째는 **단일 큐(Single Queue) 구조**로, 여러 목적지나 여러 논리적 흐름을 가진 패킷들이 단 하나의 대기열에서 순서를 기다리는 구조이다. 둘째는 **엄격한 순서 유지(Packet Order)의 필요성**으로, 프로토콜이나 시스템이 데이터의 순차적인 처리를 강제할 때 발생한다.2 이 두 가지 조건이 충족될 때, 큐의 선두에 있는 요소의 지연은 전체 시스템의 처리량(Throughput) 감소와 지연 시간(Latency) 증가로 직접 이어진다.

이 현상을 더 깊이 이해하기 위해서는 단순한 '지연' 현상을 넘어, 시스템의 '결합(Coupling)' 문제로 재해석할 필요가 있다. HOL 블로킹은 논리적으로 독립적인 작업 단위들이 단일한 순차적 자원에 강하게 결합될 때 발생하는 문제의 표면적 증상이다. 예를 들어, 서로 다른 웹 리소스를 요청하는 독립적인 HTTP 요청들은 단일 연결이라는 공유 자원에 결합된다. 마찬가지로, 서로 다른 데이터를 전송하는 논리적 스트림들은 TCP의 단일 순서 번호 공간(Sequence Number Space)이라는 자원에 결합된다. 이처럼 강력한 결합 구조 하에서는 한 요소에서 발생한 문제가 결합된 다른 모든 요소로 전파되는 현상이 필연적으로 발생한다. 따라서 HOL 블로킹의 근본적인 해결책은 단순히 큐 관리 방식을 개선하는 것을 넘어, 이 결합을 끊어내고 각 작업 단위에 독립적인 생명주기를 부여하는 아키텍처의 변화를 요구한다.

### 0.2  계층적 관점에서의 HOL 블로킹: 애플리케이션 vs. 전송 계층

HOL 블로킹은 네트워크 프로토콜 스택의 여러 계층에서 각기 다른 원인으로 발생하며, 이를 명확히 구분하는 것이 문제의 본질을 이해하는 데 매우 중요하다.

#### 0.2.1  애플리케이션 계층 HOL 블로킹

애플리케이션 계층 HOL 블로킹은 프로토콜의 의미론적 제약(Semantic Constraint)으로 인해 발생한다. 대표적인 예는 HTTP/1.1이다. 이 프로토콜은 클라이언트가 서버에 여러 요청을 보낼 때, 서버가 요청을 받은 순서대로 응답을 전송해야 한다는 엄격한 규칙을 가지고 있다.6 만약 첫 번째 요청에 대한 응답(예: 데이터베이스 조회가 필요한 동적 콘텐츠)이 생성되는 데 오랜 시간이 걸리면, 이미 처리가 완료된 후속 요청들(예: 작은 크기의 정적 이미지)에 대한 응답까지 모두 대기해야 한다.8 이 경우, 블로킹의 원인은 하위 네트워크 계층의 패킷 손실이나 지연이 아니라, 순차적 응답을 강제하는 애플리케이션 프로토콜 자체의 설계에 있다.

#### 0.2.2  전송 계층 HOL 블로킹

전송 계층 HOL 블로킹은 프로토콜의 기계적인 신뢰성 보장 메커니즘(Mechanical Reliability Mechanism)에서 비롯된다. 대표적인 예는 전송 제어 프로토콜(Transmission Control Protocol, TCP)이다. TCP는 데이터의 신뢰성 있는 순차적 전달(In-order Delivery)을 보장하는 것을 최우선 목표로 한다.11 이를 위해 모든 데이터 조각(세그먼트)에 순서 번호를 부여하고, 수신 측은 이 순서에 맞춰 데이터를 재조립한다. 만약 중간에 하나의 TCP 세그먼트가 손실되면, 수신 측은 해당 세그먼트가 재전송되어 도착할 때까지 이미 수신한 후속 세그먼트들을 버퍼에 쌓아두고 상위 계층(애플리케이션 계층)으로 전달하지 않는다.5

이 문제는 여러 독립적인 데이터 스트림을 단일 TCP 연결 위에서 다중화(Multiplexing)할 때 심각해진다. 예를 들어, HTTP/2는 여러 개의 독립적인 리소스 요청을 스트림(Stream)이라는 논리적 단위로 나누어 하나의 TCP 연결로 전송한다. 이때, 특정 스트림에 속한 TCP 세그먼트 하나가 손실되면, TCP 계층은 이 손실을 복구할 때까지 해당 연결을 통해 전송되는 모든 스트림의 데이터를 차단한다.6 즉, 논리적으로는 완전히 독립적인 다른 스트림들까지 단 하나의 패킷 손실 때문에 멈춰 서게 되는 것이다.

이처럼 두 계층의 HOL 블로킹은 근본 원인이 다르며, 상호작용을 통해 전체 시스템 성능에 복합적인 영향을 미친다. 특히, HTTP/2가 애플리케이션 계층의 HOL 블로킹을 멀티플렉싱으로 해결하자, 이전에는 여러 TCP 연결로 분산되어 있던 문제가 단일 TCP 연결로 집중되면서 전송 계층의 HOL 블로킹 문제가 오히려 더욱 두드러지는 결과를 낳았다.13 이는 한 계층의 최적화가 어떻게 다른 계층의 근본적인 한계를 드러내는지를 보여주는 중요한 사례이며, 이 보고서에서 다룰 논의의 핵심적인 출발점이 된다.

## 1.  TCP의 신뢰성 보장 메커니즘과 HOL 블로킹의 태동

### 1.1  TCP의 핵심 설계: 연결 지향형 신뢰성 바이트 스트림

TCP는 인터넷 프로토콜 스위트(Internet Protocol Suite)의 핵심 프로토콜 중 하나로, 그 설계 철학은 IP(Internet Protocol) 계층이 제공하는 비신뢰적이고 비연결지향적인 데이터그램(Datagram) 서비스 위에 신뢰성 있는 종단 간(End-to-End) 통신 채널을 구축하는 데 있다.11 TCP가 애플리케이션에게 제공하는 핵심적인 추상화는 '연결 지향형 신뢰성 바이트 스트림(Connection-Oriented Reliable Byte Stream)'이다.

- **연결 지향형(Connection-Oriented)**: 데이터 전송에 앞서, 송신자와 수신자 간에 논리적인 연결을 설정하는 3-way 핸드셰이크 과정을 거친다. 이 과정을 통해 양측은 데이터 전송에 필요한 파라미터(예: 초기 순서 번호)를 교환하고 상태를 동기화한다.11
- **신뢰성(Reliable)**: TCP는 패킷 손실, 중복, 순서 바뀜과 같은 네트워크의 불완전성을 감지하고 이를 자체적으로 해결한다. 데이터가 손상되지 않고, 손실 없이, 중복되지 않게 목적지에 전달되는 것을 보장한다.14
- **바이트 스트림(Byte Stream)**: TCP는 애플리케이션으로부터 받은 데이터를 구조 없는 연속적인 바이트들의 흐름으로 취급한다. 애플리케이션이 정의한 메시지의 경계(Message Boundary)를 인식하지 않는다. 이 데이터 스트림은 TCP에 의해 적절한 크기의 세그먼트(Segment)로 분할되어 IP 패킷에 담겨 전송된다.11

수신 측 TCP는 네트워크를 통해 도착한 이 세그먼트들을 재조립하여 원래의 순서대로 바이트 스트림을 복원한 후, 이를 수신 애플리케이션에 전달할 책임을 진다. 바로 이 '순서대로 복원'이라는 책임이 TCP의 강력한 신뢰성의 근간이자, 동시에 전송 계층 HOL 블로킹이 잉태되는 지점이다.

### 1.2  순서 번호(Sequence Number)와 확인 응답(ACK)의 역할

TCP의 신뢰성은 순서 번호(Sequence Number)와 확인 응답(Acknowledgement, ACK)이라는 두 가지 핵심 메커니즘을 통해 구현된다.

- **순서 번호**: TCP 연결을 통해 전송되는 모든 바이트는 논리적인 순서 번호를 갖는다. 송신자는 데이터를 세그먼트로 분할할 때, 각 세그먼트의 헤더에 해당 세그먼트가 담고 있는 데이터의 첫 번째 바이트에 해당하는 순서 번호를 기록한다. 이 순서 번호는 데이터 스트림 내에서 각 바이트의 고유한 위치를 나타낸다.12
- **확인 응답**: 수신자는 세그먼트를 성공적으로 수신하면, 송신자에게 확인 응답(ACK)을 보낸다. TCP의 ACK는 누적(Cumulative) 방식이다. 즉, 수신자는 순서대로 성공적으로 수신한 마지막 바이트의 다음 바이트 순서 번호를 ACK 번호로 사용하여 응답한다. 예를 들어, 1부터 1000번 바이트까지 순서대로 수신했다면, 수신자는 "이제 1001번 바이트를 기대한다"는 의미로 ACK 번호 1001을 보낸다.15

이 '긍정 확인 응답과 재전송(Positive Acknowledgment with Retransmission)' 기법은 TCP 신뢰성의 핵심이다.11 송신자는 ACK를 받음으로써 데이터가 성공적으로 전달되었음을 확인하고, 일정 시간 동안 ACK를 받지 못하면 데이터가 손실되었다고 판단하여 재전송한다.

이 과정에서 수신 측 TCP 스택의 동작 방식이 HOL 블로킹을 유발한다. 수신 측은 세그먼트를 수신할 때 순서 번호를 확인하여 순서에 맞게 정렬한다. 만약 1001번부터 2000번까지의 데이터를 담은 세그먼트가 손실되고, 2001번부터 3000번까지의 데이터를 담은 세그먼트가 먼저 도착했다면, 수신 측은 2001-3000번 데이터를 즉시 애플리케이션으로 전달하지 않는다. 대신, 이 세그먼트를 수신 버퍼(Receive Buffer)에 임시로 저장하고, 비어있는 1001-2000번 데이터가 도착하기를 기다린다.5 후속 데이터가 아무리 많이 도착하더라도, 스트림의 중간에 '구멍'이 있는 한, 그 구멍 이후의 데이터는 상위 계층으로 전달되지 않고 대기 상태에 놓인다. 이것이 바로 TCP 계층에서 HOL 블로킹이 발생하는 정확한 메커니즘이다.

### 1.3  전송 계층 HOL 블로킹의 근본 원인 심층 분석

전송 계층 HOL 블로킹의 근본 원인은 TCP의 설계 철학과 그로 인해 탄생한 '바이트 스트림' 추상화 모델에 깊이 뿌리박고 있다. TCP는 '정확한 전달(Accurate Delivery)'을 '시기적절한 전달(Timely Delivery)'보다 우선시하는 철학을 바탕으로 설계되었다.11 이로 인해 순서가 어긋난 메시지나 손실된 메시지의 재전송을 기다리는 과정에서 수 초에 달하는 지연이 발생할 수 있으며, 이는 실시간성이 중요한 애플리케이션에는 적합하지 않은 특성이다.

문제의 본질은 TCP의 '바이트 스트림' 추상화가 상위 애플리케이션 계층의 '메시지' 또는 '스트림'이라는 의미론적 경계를 완전히 무시한다는 점에서 비롯된다. HTTP/2와 같은 현대적인 프로토콜은 웹페이지를 구성하는 HTML, CSS, JavaScript 파일 등을 각각 독립적인 '스트림'으로 간주하여 논리적으로 분리하고, 이를 단일 TCP 연결을 통해 다중화한다.5 애플리케이션 계층에서는 이들이 서로 독립적인 자원임을 명확히 인지하고 있다.

하지만 이 데이터들이 TCP 계층으로 전달되는 순간, 이러한 모든 의미론적 정보는 소실된다. TCP에게는 HTML 프레임, CSS 프레임, JS 프레임의 구분이 없다. 모든 데이터는 그저 하나의 거대한, 순서가 정해진 바이트의 흐름일 뿐이다.1 TCP는 어떤 바이트가 렌더링에 필수적인 CSS에 속하는지, 어떤 바이트가 덜 중요한 광고 스크립트에 속하는지 전혀 알지 못한다. TCP가 유일하게 인지하고 판단하는 기준은 '순서 번호'라는 단일 차원뿐이다.

이 "의미론적 정보의 손실(Loss of Semantic Information)"이 치명적인 결과를 낳는다. 덜 중요한 자원(예: 광고 이미지)에 해당하는 TCP 세그먼트 하나가 네트워크에서 손실되면, TCP는 순서 번호에 따라 이 '구멍'을 메우기 위해 전체 데이터 스트림의 전송을 중단시킨다. 그 결과, 이미 수신 버퍼에 도착해 있는 매우 중요한 자원(예: 핵심 CSS 파일)에 해당하는 후속 세그먼트들까지도 애플리케이션 계층으로 전달되지 못하고 하염없이 대기하게 된다.5

결론적으로, TCP HOL 블로킹은 TCP의 버그나 결함이라기보다는, 그 핵심적인 '바이트 스트림' 추상화 모델이 현대적인 다중화 애플리케이션 프로토콜의 요구사항과 근본적으로 불일치하기 때문에 발생하는 필연적인 현상이다. 애플리케이션은 독립적인 메시지 단위로 사고하지만, TCP는 모든 것을 하나의 연속된 흐름으로 환원시켜 처리하기 때문에 이러한 병목이 발생한다. 이 문제를 해결하기 위해서는 전송 계층 자체가 애플리케이션의 '스트림' 개념을 인지하고 독립적으로 처리할 수 있는 새로운 패러다임이 필요했으며, 이는 훗날 QUIC 프로토콜의 탄생으로 이어진다.

## 2.  TCP 패킷 손실 탐지 및 재전송 메커니즘 심층 분석

TCP의 신뢰성은 손실된 패킷을 정확히 탐지하고 효율적으로 재전송하는 능력에 달려있다. 그러나 이 손실 탐지 메커니즘 자체가 HOL 블로킹으로 인한 지연을 증폭시키는 주요 원인이 된다. TCP는 주로 두 가지 방식, 즉 재전송 타임아웃(RTO)과 빠른 재전송(Fast Retransmit)을 통해 패킷 손실을 인지한다. 이 메커니즘들은 본질적으로 불완전한 정보에 기반한 추론 시스템이며, 이로 인한 보수적인 동작이 성능 저하를 야기한다.

### 2.1  재전송 타임아웃 (Retransmission Timeout, RTO)

재전송 타임아웃(RTO)은 TCP 손실 탐지의 가장 기본적인 최후의 보루이다. 송신자가 특정 세그먼트를 전송할 때, 그 세그먼트에 대한 확인 응답(ACK)을 기다리기 위한 타이머를 시작한다.15 만약 이 타이머가 만료될 때까지 유효한 ACK가 도착하지 않으면, 송신자는 해당 세그먼트가 네트워크에서 손실되었다고 가정하고 동일한 세그먼트를 재전송한다.17

RTO 값은 고정되어 있지 않고, 네트워크 상태에 따라 동적으로 계산된다. 이는 주로 연결의 왕복 시간(Round-Trip Time, RTT)을 기반으로 한다. Van Jacobson이 제안한 고전적인 알고리즘에 따르면, RTO는 평활화된 RTT(Smoothed RTT, SRTT)와 RTT의 변동성(RTT Variation, RTTVar)을 고려하여 계산된다.19 일반적인 공식은 다음과 같다.
$$
RTO = SRTT + 4 \times RTTVar
$$
여기서 SRTT는 이전 RTT 측정값들의 지수 가중 이동 평균(EWMA)이며, RTTVar는 RTT 측정값의 편차를 나타낸다. 이 공식은 네트워크 지연이 변동하는 상황에서도 불필요한 재전송을 피하기 위해 충분한 여유를 두도록 설계되었다.

그러나 RTO의 문제는 그 값이 매우 길 수 있다는 점이다. 초기 연결 시 RTT 정보가 없기 때문에, RTO의 초기값은 보수적으로 1초 또는 3초와 같은 긴 값으로 설정된다.16 단 하나의 패킷 손실이 발생하여 RTO에 의존해야 하는 상황이 되면, 데이터 전송은 최소 1초 이상 완전히 중단된다. 더욱이, RTO가 발생하면 TCP는 이를 심각한 네트워크 혼잡의 신호로 간주한다. 그 결과, 손실된 패킷을 재전송할 뿐만 아니라, '지수적 백오프(Exponential Backoff)' 알고리즘에 따라 RTO 값을 2배로 늘린다.19 만약 재전송된 패킷마저 손실되면 다음 RTO는 2초, 4초, 8초 등으로 기하급수적으로 증가하여 HOL 블로킹으로 인한 지연을 극단적으로 악화시킨다.

### 2.2  빠른 재전송 (Fast Retransmit)과 중복 ACK (Duplicate ACKs)

RTO의 긴 대기 시간을 피하기 위해 고안된 더 정교한 메커니즘이 바로 빠른 재전송(Fast Retransmit)이다.15 이 기법은 후속 패킷의 도착 정보를 이용하여 패킷 손실을 더 빠르게 추론한다.

동작 원리는 다음과 같다. 수신자는 항상 순서대로 수신한 마지막 바이트의 다음 바이트를 요청하는 누적 ACK를 보낸다. 만약 송신자가 세그먼트 1, 2, 3, 4, 5를 순서대로 보냈는데, 세그먼트 2가 손실되고 3, 4, 5는 정상적으로 도착했다고 가정하자. 수신자는 세그먼트 1을 받고 ACK 2를 보낸다. 그 후 세그먼트 3을 받지만, 여전히 2를 기다리고 있으므로 다시 ACK 2를 보낸다. 세그먼트 4와 5가 도착할 때도 마찬가지로 계속해서 ACK 2를 보낸다. 이렇게 동일한 ACK 번호를 가진 ACK가 반복적으로 수신되는 것을 '중복 ACK(Duplicate ACK)'라고 한다.15

TCP는 특정 세그먼트에 대해 3개의 중복된 ACK(즉, 최초의 ACK를 포함하여 총 4개의 동일한 ACK)를 수신하면, 해당 세그먼트가 손실되었을 가능성이 매우 높다고 판단한다. 이 경우, 송신자는 RTO 타이머가 만료되기를 기다리지 않고 즉시 해당 세그먼트를 재전송한다.15 이 메커니즘은 RTO에 비해 훨씬 빠르게 손실을 복구할 수 있어 '빠른 재전송'이라 불린다.

하지만 빠른 재전송에도 명백한 한계가 존재한다. 이 메커니즘이 동작하려면 손실된 패킷 이후에 충분한 수의 후속 패킷(최소 3개)이 수신자에게 도착하여 중복 ACK를 유발해야 한다. 따라서 다음과 같은 상황에서는 빠른 재전송이 동작하지 않고 결국 RTO에 의존하게 된다 19:

- **전송 윈도우의 끝에서 발생한 손실 (Tail Loss)**: 전송할 데이터의 마지막 부분을 보내는 중에 패킷이 손실되면, 중복 ACK를 유발할 후속 패킷이 없으므로 빠른 재전송이 트리거되지 않는다.
- **작은 전송 윈도우**: 혼잡 윈도우(Congestion Window) 크기가 작아 한 번에 전송하는 패킷의 수가 적을 경우, 하나의 패킷이 손실되어도 3개의 중복 ACK를 생성할 만큼의 후속 패킷이 없을 수 있다.
- **심각한 버스트 손실(Burst Loss)**: 연속된 여러 패킷이 한꺼번에 손실되면, 중복 ACK를 생성할 패킷 자체가 없어지게 된다.

### 2.3  패킷 손실이 HOL 블로킹을 악화시키는 과정

TCP의 손실 탐지 및 재전송 메커니즘이 HOL 블로킹과 결합하여 성능을 저하시키는 과정은 다음과 같은 악순환을 따른다.

1. **패킷 손실 발생**: 단일 TCP 세그먼트가 네트워크에서 손실된다.
2. **수신자 버퍼링**: 수신자는 손실된 세그먼트 이후에 도착하는 모든 순서가 어긋난 세그먼트들을 수신 버퍼에 쌓아두고, 상위 계층으로의 데이터 전달을 중단한다. 이것이 HOL 블로킹의 시작이다.
3. **송신자 대기**: 송신자는 손실을 인지하기 위해 대기한다. 운이 좋다면 후속 패킷들이 3개의 중복 ACK를 유발하여 빠른 재전송이 동작한다. 그렇지 않다면, 기나긴 RTO 타이머가 만료될 때까지 기다려야 한다. 이 대기 시간 동안 전체 TCP 연결의 데이터 흐름은 완전히 멈춘다.
4. **재전송 및 복구**: 송신자가 마침내 손실을 인지하고 해당 세그먼트를 재전송한다.
5. **블로킹 해소**: 수신자가 재전송된 세그먼트를 성공적으로 수신하면, 비로소 버퍼에 대기 중이던 모든 후속 세그먼트들과 함께 순서를 맞춰 재조립한 후, 상위 애플리케이션 계층으로 데이터를 전달한다. HOL 블로킹이 해소된다.

이 전체 과정에서 핵심은, 단 하나의 TCP 세그먼트 손실이 복구될 때까지 해당 TCP 연결을 통해 전송되는 모든 논리적 데이터 스트림(예: HTTP/2의 모든 스트림)이 예외 없이 중단된다는 점이다.6 이는 마치 고속도로의 한 차선에서 발생한 작은 사고가 전체 고속도로의 모든 차선을 마비시키는 것과 같다.

이러한 TCP의 손실 탐지 방식은 근본적으로 '모호함(Ambiguity)'과 '불확실성(Uncertainty)'에 기반한 추론 시스템이라는 한계를 가진다. RTO 타이머의 만료는 패킷 손실을 의미할 수도 있지만, 극심한 네트워크 지연일 수도 있다. TCP는 이 둘을 구분할 방법이 없기에 최악의 경우(손실)를 가정하고 보수적으로 대응한다. 마찬가지로, 중복 ACK는 패킷 손실의 강력한 징후이지만, 네트워크에서의 일시적인 패킷 재정렬(Reordering) 때문에 발생할 수도 있다. '3'이라는 임계값은 수많은 경험을 통해 정립된 합리적인 추정치(Heuristic)일 뿐, 완벽한 확신을 제공하지는 않는다.19

이러한 불확실성은 TCP가 불필요한 재전송으로 네트워크를 혼잡하게 만드는 것을 피하기 위해 보수적으로 동작하도록 강제한다. 즉, 손실을 확신하기 위해 더 오래 기다리거나(RTO), 더 많은 증거(3개의 중복 ACK)를 요구하는 것이다. 바로 이 보수적인 태도가 HOL 블로킹 상황에서 발생하는 지연 시간을 필연적으로 증폭시키는 구조적 원인이 된다.

## 3.  애플리케이션 계층에서의 HOL 블로킹: HTTP/1.1에서 HTTP/2로의 진화와 한계

전송 계층의 HOL 블로킹이 TCP의 근본적인 특성에서 비롯된다면, 애플리케이션 계층의 HOL 블로킹은 프로토콜의 설계와 의미론에서 발생한다. HTTP 프로토콜의 진화 과정은 이 애플리케이션 계층의 문제를 해결하려는 노력의 역사였으며, 그 과정에서 역설적으로 전송 계층의 한계를 더욱 명확하게 드러내는 계기가 되었다.

### 3.1  HTTP/1.1: 순차적 처리와 파이프라이닝의 도전

초기 웹을 위해 설계된 HTTP/1.0은 요청 하나당 하나의 TCP 연결을 생성하고, 응답을 받은 후 연결을 종료하는 단순한 모델을 사용했다. 이는 매 요청마다 TCP의 3-way 핸드셰이크와 느린 시작(Slow Start)을 반복해야 하는 극심한 비효율을 낳았다. HTTP/1.1은 '지속 연결(Persistent Connections)'을 도입하여 하나의 TCP 연결을 여러 요청과 응답에 재사용할 수 있도록 개선했다.6

하지만 지속 연결 환경에서도 HTTP/1.1은 기본적으로 엄격한 순차적 처리 모델을 따랐다. 클라이언트는 하나의 요청을 보내고 그에 대한 응답을 완전히 수신한 후에야 다음 요청을 보낼 수 있었다.8 이는 명백한 애플리케이션 계층의 HOL 블로킹을 유발했다. 예를 들어, 웹페이지를 로드할 때 큰 이미지를 먼저 요청하면, 그 이미지가 모두 다운로드될 때까지 뒤따르는 작은 CSS나 JavaScript 파일 요청은 시작조차 할 수 없었다.

이 문제를 완화하기 위해 **HTTP 파이프라이닝(Pipelining)**이라는 기술이 HTTP/1.1 명세에 포함되었다. 파이프라이닝은 클라이언트가 이전 요청에 대한 응답을 기다리지 않고, 여러 요청을 연속해서 하나의 TCP 연결에 보내는 것을 허용했다.4 이는 네트워크의 왕복 지연 시간을 줄여 성능을 향상시킬 잠재력을 가졌다.

그러나 파이프라이닝은 치명적인 한계를 가지고 있었다. 바로 서버가 응답을 보낼 때는 반드시 요청이 도착한 순서(FIFO)를 지켜야 한다는 점이다.5 만약 첫 번째 요청이 데이터베이스 조회와 같이 처리 시간이 긴 동적 콘텐츠이고, 두 번째 요청이 캐시된 작은 정적 파일이라 하더라도, 서버는 첫 번째 요청의 처리가 모두 끝나고 응답 전송이 완료될 때까지 두 번째 응답을 보낼 수 없었다.7 결국, 파이프라이닝은 HOL 블로킹 문제를 해결하지 못하고, 단지 블로킹이 발생하는 지점을 클라이언트에서 서버(또는 연결 파이프라인)로 옮겼을 뿐이다.

더욱이, 파이프라이닝은 현실 세계에서 수많은 문제를 야기했다. 많은 구형 프록시 서버들이 파이프라이닝을 제대로 지원하지 않아 응답 순서를 뒤섞거나 연결을 끊어버리는 등 오작동을 일으켰다.22 이러한 구현의 복잡성과 불안정성 때문에 대부분의 웹 브라우저는 파이프라이닝 기능을 기본적으로 비활성화하거나 아예 지원을 중단했다.4

결과적으로 HTTP/1.1 시대의 현실적인 HOL 블로킹 완화책은 기술적 진보가 아닌, 일종의 '우회책'이었다. 웹 브라우저들은 동일한 도메인에 대해 6개에서 8개에 이르는 다수의 병렬 TCP 연결을 생성하여 동시에 여러 리소스를 요청했다.8 이는 HOL 블로킹의 영향을 줄이는 데는 효과적이었지만, 각 연결마다 TCP 핸드셰이크, TLS 핸드셰이크, 느린 시작을 반복해야 했기 때문에 서버와 클라이언트 양쪽에 상당한 리소스 부담과 불필요한 지연을 초래하는 근시안적인 해결책에 불과했다.

### 3.2  HTTP/2: 멀티플렉싱을 통한 애플리케이션 계층 HOL 블로킹 해결

HTTP/2는 HTTP/1.1의 근본적인 문제들을 해결하기 위해 완전히 새로운 패러다임을 도입했다. 그 핵심은 **멀티플렉싱(Multiplexing)**이다.4 HTTP/2는 단 하나의 TCP 연결 위에서 여러 개의 요청과 응답이 동시에, 순서에 상관없이 교차하며 전송될 수 있도록 허용한다.10

이러한 멀티플렉싱은 새로운 바이너리 프레이밍(Binary Framing) 계층을 통해 구현된다. 모든 HTTP 메시지(요청 또는 응답)는 더 작은 단위인 **프레임(Frame)**으로 분할된다. 각 프레임에는 자신이 속한 논리적 흐름을 식별하는 **스트림 ID(Stream ID)**가 부여된다.5 예를 들어, 클라이언트가 HTML, CSS, JS 파일을 동시에 요청하면 각각 스트림 1, 3, 5가 할당될 수 있다. 서버는 이 세 파일에 대한 응답 데이터를 프레임 단위로 쪼개어 준비되는 대로 전송한다. 네트워크 상에서는 HTML 응답 프레임, CSS 응답 프레임, JS 응답 프레임이 뒤섞여(Interleaving) 전송될 수 있다.6 수신 측은 각 프레임의 스트림 ID를 보고 원래의 스트림으로 재조립한다.

이 구조 덕분에, 하나의 스트림(예: 큰 비디오 파일)에 대한 응답이 지연되더라도 다른 스트림(예: 작은 텍스트 파일)의 프레임 전송을 전혀 막지 않는다. 이로써 HTTP/1.1을 괴롭혔던 애플리케이션 계층의 HOL 블로킹 문제는 효과적으로 해결되었다.

나아가 HTTP/2는 **스트림 우선순위 지정(Stream Prioritization)** 기능을 도입하여 클라이언트가 각 스트림의 중요도를 서버에 알릴 수 있게 했다.10 예를 들어, 브라우저는 웹페이지 렌더링에 필수적인 CSS 파일에 높은 우선순위를, 상대적으로 덜 중요한 광고 이미지에는 낮은 우선순위를 부여할 수 있다. 서버는 이 정보를 바탕으로 제한된 네트워크 자원을 더 중요한 리소스에 우선적으로 할당하여 사용자 인지 성능(Perceived Performance)을 최적화할 수 있다.

### 3.3  HTTP/2의 근본적 한계: TCP HOL 블로킹 문제의 부각

HTTP/2는 멀티플렉싱을 통해 애플리케이션 계층의 HOL 블로킹을 성공적으로 해결했지만, 이 해결책은 역설적으로 숨어 있던 더 근본적인 문제를 수면 위로 끌어올렸다. 바로 **전송 계층의 TCP HOL 블로킹**이다.5

HTTP/2의 설계는 모든 다중화된 스트림을 단 하나의 TCP 연결 위에 올리는 것을 전제로 한다.10 이는 HTTP/1.1의 다중 연결 방식에 비해 연결 설정 오버헤드를 줄이고, 단일 연결에 대한 혼잡 상태를 더 정확하게 관리할 수 있다는 장점이 있다. 하지만 이 단일 TCP 연결이 '아킬레스건'이 되었다.

TCP 계층에서 단 하나의 패킷이라도 손실되면, TCP의 엄격한 순서 보장 메커니즘이 동작한다. TCP 스택은 손실된 패킷이 재전송되어 도착할 때까지, 이미 수신된 모든 후속 패킷들의 처리를 중단하고 버퍼에 대기시킨다. 문제는 TCP가 HTTP/2의 '스트림' 개념을 전혀 인지하지 못한다는 점이다. TCP의 관점에서는 모든 스트림의 프레임들이 그저 하나의 연속된 바이트 스트림일 뿐이다. 따라서 손실된 패킷이 어떤 스트림에 속해 있었는지와 무관하게, 전체 TCP 연결이 멈춰 선다. 그 결과, 손실된 패킷과 전혀 관련이 없는 건강한 상태의 다른 모든 HTTP/2 스트림들까지 강제로 대기 상태에 빠지게 된다.6

이 현상은 HTTP/1.1 시대와 비교했을 때 문제의 성격을 바꾸어 놓았다. HTTP/1.1에서는 6개의 병렬 연결 중 하나가 패킷 손실로 막히더라도, 나머지 5개의 연결은 계속해서 데이터를 전송할 수 있었다. 즉, 피해 범위가 해당 연결에 국한되었다. 그러나 HTTP/2에서는 모든 리소스가 의존하는 단 하나의 연결이 전체의 운명을 좌우하는 '단일 장애점(Single Point of Failure)'이 되어버렸다.13 패킷 손실이 잦은 무선 네트워크나 불안정한 인터넷 환경에서 이는 HTTP/2의 성능을 심각하게 저하시키는 요인이 되었다.

결론적으로, HTTP/2의 등장은 HOL 블로킹 문제의 '책임 전가(Shifting the Blame)' 과정을 명확히 보여준다. HTTP/1.1의 문제는 명백히 애플리케이션 프로토콜의 설계 결함이었다. HTTP/2는 이 결함을 완벽하게 수정했지만, 그 결과는 전송 계층인 TCP가 새로운 병목 지점임을 명백히 드러내는 것이었다. 기술 발전 과정에서 한 계층의 최적화가 어떻게 하위 계층의 근본적인 한계를 노출시키는지를 보여주는 교과서적인 사례라 할 수 있다. 이로써 네트워크 커뮤니티는 문제의 본질이 단순히 'HTTP'가 아니라, 'TCP 위에서 동작하는 다중화된 프로토콜'이라는 더 깊은 차원에 있음을 깨닫게 되었고, 이는 TCP를 대체할 새로운 전송 프로토콜, 즉 QUIC의 개발을 촉발하는 직접적인 계기가 되었다.8

## 4.  TCP HOL 블로킹의 근본적 해결책: QUIC 프로토콜

HTTP/2가 전송 계층의 HOL 블로킹이라는 한계에 부딪히면서, 문제의 근본적인 해결을 위해서는 TCP 자체를 넘어서는 새로운 접근이 필요하다는 공감대가 형성되었다. 그 결과로 탄생한 것이 바로 QUIC(Quick UDP Internet Connections) 프로토콜이다. QUIC은 TCP가 가진 제약을 회피하고 현대 웹 환경에 최적화된 전송 계층을 제공하기 위해 패러다임의 전환을 이룬 혁신적인 프로토콜이다.

### 4.1  패러다임의 전환: 왜 TCP가 아닌 UDP인가?

TCP를 직접 개선하려는 시도는 과거에도 존재했다. 예를 들어, 다중 경로 TCP(Multipath TCP, MPTCP)는 여러 네트워크 경로를 동시에 사용하여 성능과 안정성을 높이려는 시도였다. 그러나 이러한 TCP 확장 노력은 두 가지 큰 장벽에 부딪혔다.

첫째, **커널 의존성**이다. TCP는 운영체제(OS)의 커널에 깊숙이 내장되어 있다. TCP의 동작 방식을 변경하려면 커널 코드를 수정해야 하며, 이는 전 세계 수많은 OS와 장비에 걸쳐 업데이트를 배포해야 함을 의미한다. 이는 매우 느리고 어려운 과정이다.24

둘째, **프로토콜 화석화(Protocol Ossification)** 문제이다. 인터넷상의 수많은 중간 장비(Middlebox)들, 예를 들어 방화벽, NAT, 로드 밸런서 등은 TCP 헤더의 특정 필드 값을 기반으로 동작하도록 고정되어 있다. TCP에 새로운 옵션을 추가하거나 기존 필드의 의미를 변경하면, 이들 중간 장비가 해당 패킷을 비정상으로 간주하여 차단하거나 변조할 수 있다.26 이로 인해 TCP의 새로운 기능 배포는 사실상 불가능에 가까웠다.

QUIC 개발자들은 이러한 문제를 정면으로 돌파하는 대신, 완전히 다른 경로를 선택했다. 바로 TCP를 버리고 UDP(User Datagram Protocol)를 기반으로 새로운 전송 프로토콜을 구축하는 것이었다. UDP는 최소한의 기능(포트 번호 지정, 체크섬)만을 제공하는 단순한 프로토콜로, 대부분의 네트워크 환경에서 자유롭게 통과가 허용된다.28 QUIC은 이 UDP 데이터그램 위에 TCP의 신뢰성 있는 전송, 흐름 제어, 혼잡 제어 기능과 TLS의 암호화 기능, 그리고 HTTP/2의 스트림 다중화 기능을 모두 사용자 공간(User Space)에서 직접 재구현했다.29

이러한 접근은 다음과 같은 결정적인 이점을 제공했다:

- **빠른 배포와 진화**: QUIC은 브라우저나 애플리케이션 라이브러리 형태로 배포될 수 있다. 이는 OS 업데이트를 기다릴 필요 없이 새로운 기능을 신속하게 사용자에게 배포하고 실험할 수 있음을 의미한다.26
- **중간 장비 회피**: QUIC은 UDP를 사용하고, 패킷 헤더를 포함한 대부분의 정보를 암호화한다. 이로 인해 중간 장비들이 QUIC 트래픽의 내용을 해석하거나 간섭하기가 매우 어려워져 프로토콜 화석화 문제를 회피할 수 있다.32

결론적으로, UDP를 선택한 것은 신뢰성을 포기한 것이 아니라, 경직된 인터넷 환경에서 혁신을 지속하기 위한 전략적 결정이었다.

### 4.2  QUIC의 핵심 아키텍처

QUIC은 TCP HOL 블로킹을 해결하고 현대 네트워크 환경에 적응하기 위해 몇 가지 혁신적인 아키텍처를 도입했다.

- **독립적인 스트림(Independent Streams)**: QUIC 아키텍처의 가장 핵심적인 요소이다. TCP와 달리, QUIC은 전송 계층 자체에서 '스트림'의 개념을 완벽하게 인지하고 지원한다. 각 QUIC 스트림은 자신만의 독립적인 흐름 제어(Flow Control) 로직과 데이터 전달 순서를 가진다. 따라서 한 스트림에서 패킷 손실이 발생하여 데이터 재전송이 필요하더라도, 이는 해당 스트림에만 영향을 미칠 뿐, 다른 스트림들은 아무런 방해 없이 데이터를 계속 전송하고 처리할 수 있다.6 이것이 바로 TCP HOL 블로킹을 근본적으로 해결하는 메커니즘이다. 이로써 '신뢰성'의 개념은 '단일 스트림의 완벽한 순서 보장'이라는 경직된 개념에서 '각 스트림의 독립적인 순서 보장'이라는 유연하고 다차원적인 개념으로 재정의되었다.
- **연결 ID(Connection ID)**: TCP 연결은 송신자와 수신자의 IP 주소 및 포트 번호로 구성된 4-튜플(4-tuple)에 의해 식별된다. 이로 인해 클라이언트의 IP 주소가 바뀌면(예: Wi-Fi에서 셀룰러 네트워크로 전환) 기존 TCP 연결은 끊어지고 새로 연결을 맺어야 한다. 반면, QUIC은 연결을 식별하기 위해 4-튜플 대신 **연결 ID(CID)**라는 고유 식별자를 사용한다.35 이 CID는 클라이언트의 IP 주소나 포트가 변경되더라도 동일하게 유지되므로, 네트워크 환경이 바뀌어도 연결이 끊기지 않고 원활하게 유지되는 **연결 마이그레이션(Connection Migration)**이 가능하다.38 이는 모바일 사용자 경험을 획기적으로 개선하는 기능이다.
- **내장된 암호화(Integrated Encryption)**: QUIC은 TLS 1.3을 프로토콜의 필수적인 부분으로 통합했다. QUIC 핸드셰이크는 전송 계층의 연결 설정과 암호화 키 교환을 동시에 수행한다.28 이로 인해 TCP와 TLS가 별도의 핸드셰이크를 거쳐야 했던 것에 비해 연결 설정 지연 시간이 크게 단축된다. 특히, 이전에 연결했던 서버와 다시 연결할 때는 0-RTT 핸드셰이크를 통해 지연 시간을 거의 없앨 수도 있다.38 또한, 패킷 번호와 같은 전송 계층 메타데이터까지 암호화하여 개인 정보 보호를 강화하고 중간 장비의 간섭 가능성을 더욱 줄인다.26

### 4.3  HTTP/3: QUIC 위에서의 진정한 멀티플렉싱

HTTP/3는 전송 계층으로 TCP 대신 QUIC을 사용하도록 정의된 HTTP의 세 번째 주요 버전이다.8 HTTP/3는 HTTP/2가 도입했던 스트림, 서버 푸시, 헤더 압축과 같은 핵심적인 애플리케이션 계층 개념들을 대부분 계승한다. 하지만 그 기반이 되는 전송 프로토콜이 바뀜으로써 근본적인 성능 향상을 이루었다.

HTTP/3에서는 HTTP/2의 각 스트림이 QUIC의 독립적인 전송 스트림에 직접 매핑된다.26 이는 HTTP/2가 애플리케이션 계층에서만 구현했던 멀티플렉싱이 이제 전송 계층 수준에서 완벽하게 지원됨을 의미한다. 따라서 TCP HOL 블로킹은 원천적으로 발생하지 않는다.40

또한, 헤더 압축 방식도 개선되었다. HTTP/2의 HPACK은 순차적인 헤더 처리를 가정하기 때문에, 스트림 간 독립성을 가진 QUIC 환경에서는 잠재적인 HOL 블로킹을 유발할 수 있었다. 예를 들어, 한 스트림에서 동적 테이블을 업데이트하는 헤더 프레임이 손실되면, 그 업데이트에 의존하는 다른 스트림의 헤더를 디코딩할 수 없게 된다. 이를 해결하기 위해 HTTP/3는 **QPACK**이라는 새로운 헤더 압축 방식을 도입했다. QPACK은 각 스트림이 독립적으로 헤더를 디코딩할 수 있도록 설계되어, 헤더 압축 과정에서 발생할 수 있는 미세한 HOL 블로킹까지 방지한다.8

아래 표는 HTTP 프로토콜 버전별 HOL 블로킹 문제의 진화 과정을 요약한 것이다.

| 특징 (Feature)                       | HTTP/1.1 (Pipelining) | HTTP/2                         | HTTP/3 (QUIC)          |
| ------------------------------------ | --------------------- | ------------------------------ | ---------------------- |
| **발생 계층 (Layer of Occurrence)**  | 애플리케이션 계층     | **전송 계층 (TCP)**            | **해결됨 (Resolved)**  |
| **주요 원인 (Primary Cause)**        | 엄격한 FIFO 응답 순서 | 단일 TCP 연결, 순서 보장       | -                      |
| **해결 방식 (Resolution Mechanism)** | 병렬 TCP 연결 (우회)  | 멀티플렉싱 (애플리케이션 계층) | **독립적 QUIC 스트림** |
| **남은 문제점 (Remaining Issues)**   | TCP 연결 오버헤드     | TCP HOL 블로킹                 | CPU 오버헤드, UDP 차단 |

이 표는 HOL 블로킹 문제 해결의 여정을 명확하게 보여준다. HTTP/1.1의 문제는 애플리케이션의 순차 처리 규칙에 있었고, HTTP/2는 이를 멀티플렉싱으로 해결했지만 문제는 TCP 계층으로 전이되었다. 최종적으로 HTTP/3와 QUIC이 독립적인 스트림을 통해 전송 계층의 문제까지 근본적으로 해결했음을 알 수 있다.

## 5.  QUIC의 신뢰성 및 성능 최적화 메커니즘

QUIC은 UDP 위에 신뢰성 있는 전송 서비스를 구축하기 위해 TCP의 여러 개념을 차용하면서도, TCP가 가진 근본적인 비효율성을 개선하기 위한 독자적인 메커니즘들을 다수 포함하고 있다. 이러한 메커니즘들은 HOL 블로킹 해결을 넘어 전반적인 성능 최적화에 기여한다.

### 5.1  패킷과 프레임 구조

QUIC의 데이터 전송 단위는 **패킷(Packet)**이며, 각 패킷은 헤더와 하나 이상의 **프레임(Frame)**으로 구성된 페이로드를 포함한다.36 이는 HTTP/2의 프레임 개념을 전송 계층으로 가져온 것과 유사하지만, 그 역할과 구조에서 중요한 차이가 있다.

**TCP 세그먼트 헤더와의 비교**:

- **TCP 세그먼트 헤더**: 순서 번호, 확인 응답 번호, 윈도우 크기, 포트 번호 등 연결 전체의 상태를 관리하는 정보가 포함된다. 이 정보들은 단일 바이트 스트림을 제어하기 위해 설계되었다.
- **QUIC 패킷 헤더**: 연결 ID와 패킷 번호 등 패킷 자체를 식별하고 암호 해독에 필요한 최소한의 정보만을 담는다. 연결 설정 중에는 상세 정보가 포함된 **Long Header**를, 연결이 수립된 후에는 오버헤드를 줄인 **Short Header**를 사용하여 효율성을 높인다.36
- **QUIC 프레임**: 실제 데이터와 스트림별 제어 정보는 프레임 내부에 캡슐화된다. 예를 들어, `STREAM` 프레임에는 스트림 ID, 데이터의 오프셋(Offset), 그리고 실제 데이터가 포함된다.30 이처럼 패킷 헤더(연결 단위)와 프레임(스트림 단위)의 역할을 구조적으로 분리함으로써, 각 스트림이 독립적으로 관리될 수 있는 기반을 마련한다.

이러한 구조는 단일 패킷 안에 여러 스트림에 속한 데이터를 담을 수 있게 하여 패킷 전송 효율을 높이는 동시에, 각 스트림의 독립성을 보장하는 QUIC의 핵심 설계 철학을 반영한다.

### 5.2  손실 탐지 및 복구: 재전송 모호성 해결

QUIC은 TCP의 손실 탐지 및 복구 메커니즘을 크게 개선하여, 더 빠르고 정확한 복구를 가능하게 한다. 가장 중요한 개선점은 **재전송 모호성(Retransmission Ambiguity)** 문제의 해결이다.

- **TCP의 재전송 모호성**: TCP에서는 원본 세그먼트와 그 재전송 세그먼트가 동일한 순서 번호를 사용한다. 이로 인해 송신자가 ACK를 수신했을 때, 이 ACK가 원본 전송에 대한 응답인지, 아니면 재전송에 대한 응답인지 명확히 구분하기 어렵다. 이 모호성은 RTT를 정확하게 측정하는 것을 방해하고, 불필요한 재전송(Spurious Retransmission)을 유발하여 성능을 저하시킬 수 있다.43
- **QUIC의 해결책**: QUIC은 패킷을 재전송할 때, 손실된 패킷의 내용을 새로운 패킷에 담아 전송하면서 항상 **새로운, 단조롭게 증가하는 패킷 번호(Monotonically Increasing Packet Number)**를 부여한다.44 패킷 번호는 전송 순서를 직접적으로 나타내므로, 수신된 ACK는 항상 특정 시점에 전송된 특정 패킷을 명확하게 가리킨다. 이로써 재전송 모호성이 원천적으로 제거된다. 그 결과, QUIC은 다음과 같은 이점을 얻는다:
  - **정확한 RTT 측정**: ACK가 어떤 패킷에 대한 응답인지 명확하므로, RTT를 매우 정확하게 측정할 수 있다.47
  - **향상된 손실 탐지**: TCP의 SACK(Selective Acknowledgment)보다 훨씬 많은 ACK 범위(최대 256개)를 `ACK` 프레임에 담을 수 있어, 패킷 재정렬이나 높은 손실률 환경에서도 어떤 패킷이 수신되었고 어떤 패킷이 손실되었는지를 더 정확하고 빠르게 파악할 수 있다.31
  - **수신자 지연 보정**: QUIC의 `ACK` 프레임에는 **'ACK Delay'** 필드가 포함된다. 이 필드는 수신자가 패킷을 받은 시점과 그에 대한 ACK를 전송한 시점 사이의 시간을 마이크로초 단위로 기록한다. 송신자는 이 값을 RTT 측정값에서 빼줌으로써, 순수한 네트워크 왕복 시간과 수신 측의 처리 지연을 분리할 수 있다. 이는 특히 사용자 공간에서 동작하여 처리 지연이 가변적일 수 있는 QUIC 구현에서 RTT 측정의 정확도를 높이는 데 매우 중요하다.48

### 5.3  혼잡 제어: CUBIC에서 BBR로

혼잡 제어는 네트워크의 붕괴를 막고 가용 대역폭을 효율적으로 사용하기 위한 핵심 기술이다. QUIC은 혼잡 제어 알고리즘을 플러그인 방식으로 교체할 수 있는 유연성을 제공하며, 전통적인 손실 기반 방식에서 벗어난 새로운 모델 기반 방식을 주류로 이끌었다.

- **손실 기반 혼잡 제어 (Loss-based, 예: CUBIC)**: CUBIC은 현재 대부분의 운영체제에서 TCP의 기본 혼잡 제어 알고리즘으로 사용된다. 이 방식의 기본 철학은 네트워크가 감당할 수 있는 한계, 즉 **패킷 손실**이 발생할 때까지 전송 속도(혼잡 윈도우)를 점진적으로 늘리는 것이다.51 손실이 감지되면, 이를 네트워크 혼잡의 명백한 신호로 간주하고 전송 속도를 급격히 줄인다. 이 방식은 단순하고 안정적이지만, 두 가지 큰 문제점을 가진다. 첫째, 네트워크 경로상의 라우터 버퍼를 가득 채우려는 경향이 있어 불필요한 지연, 즉 **버퍼블로트(Bufferbloat)**를 유발한다. 둘째, 실제로는 혼잡이 없더라도 무선 링크의 간섭 등으로 인해 발생하는 무작위 패킷 손실에도 과민하게 반응하여 불필요하게 전송 속도를 낮춘다.53
- **모델 기반 혼잡 제어 (Model-based, 예: BBR)**: Google이 개발하여 QUIC과 함께 널리 보급된 BBR(Bottleneck Bandwidth and Round-trip propagation time)은 완전히 다른 접근 방식을 취한다. BBR은 패킷 손실을 혼잡의 주된 신호로 사용하지 않는다. 대신, 네트워크 경로의 두 가지 핵심 특성, 즉 **병목 대역폭(Bottleneck Bandwidth, BtlBw)**과 **최소 왕복 전파 시간(Round-trip Propagation Time, RTprop)**을 지속적으로 측정하여 네트워크 모델을 구축한다.53 BBR은 이 모델을 기반으로, 버퍼를 거의 채우지 않으면서도 측정된 병목 대역폭을 최대한 활용할 수 있는 최적의 전송 속도를 계산하여 데이터를 전송(Pacing)한다.56 RTT가 급격히 증가하면 이를 큐가 쌓이는 신호로 보고 속도를 조절한다. 이 방식은 지연 시간을 낮게 유지하면서도 높은 처리량을 달성할 수 있으며, 특히 패킷 손실이 반드시 혼잡을 의미하지 않는 현대의 네트워크 환경에서 CUBIC보다 월등한 성능을 보인다.52

아래 표는 CUBIC과 BBR 알고리즘의 핵심적인 차이점을 비교한다.

| 특징 (Feature)                | CUBIC (Loss-based)                                | BBR (Model-based)                                   |
| ----------------------------- | ------------------------------------------------- | --------------------------------------------------- |
| **기반 철학 (Philosophy)**    | 손실이 발생할 때까지 속도를 높여 가용 대역폭 탐색 | 경로의 대역폭과 RTT를 모델링하여 최적 전송률 유지   |
| **혼잡 신호 (Signal)**        | **패킷 손실**, 중복 ACK                           | **RTT 증가**, **전송률 정체**                       |
| **주요 장점 (Advantages)**    | 널리 배포되고 검증됨, TCP 표준                    | 높은 처리량과 낮은 지연 시간 동시 달성, 손실에 강함 |
| **주요 단점 (Disadvantages)** | 버퍼블로트 유발, 무작위 손실에 취약               | 다른 흐름(CUBIC)과의 공정성 문제(BBRv1), 복잡성     |

이 표는 QUIC과 BBR이 대표하는 혼잡 제어의 패러다임 전환을 명확히 보여준다. BBR은 네트워크의 한계점(손실)을 찾는 대신, 네트워크의 용량 자체를 모델링하는 방식으로 전환함으로써, 특히 변동성이 큰 현대 네트워크 환경에서 더 지능적이고 효율적인 대응을 가능하게 한다.

### 5.4  0-RTT 연결 설정과 재전송 공격

QUIC의 가장 혁신적인 성능 향상 기능 중 하나는 **0-RTT(Zero Round-Trip Time) 연결 재개**이다. 클라이언트가 이전에 성공적으로 연결했던 서버에 다시 접속할 경우, QUIC은 암호화 및 전송 핸드셰이크 과정을 완전히 생략하고, 연결을 시작하는 첫 번째 패킷에 바로 애플리케이션 데이터(예: HTTP GET 요청)를 포함하여 보낼 수 있다.31

- **성능적 이점**: 이 기능은 연결 설정에 필요한 최소 1-RTT의 지연 시간을 제거한다. 이는 특히 RTT가 수백 밀리초에 달하는 모바일 네트워크나 위성 통신 환경에서 웹페이지 로드 시간이나 API 응답 시간을 획기적으로 단축시키는 효과를 가져온다.38

- **보안적 트레이드오프: 재전송 공격(Replay Attack)**: 0-RTT는 성능을 위해 보안성을 일부 희생하는 트레이드오프를 가진다. 0-RTT 데이터는 이전 세션에서 캐시해 둔 pre-shared key(PSK)로 암호화되는데, 이 키를 모르는 공격자는 데이터의 내용을 해독하거나 변조할 수는 없다.62 하지만 공격자는 네트워크 상에서 이 0-RTT 패킷을 가로채서 복사한 뒤, 서버에 여러 번 그대로 재전송할 수 있다. 이를 

  **재전송 공격**이라 한다.62 만약 0-RTT 데이터에 담긴 요청이 멱등성(Idempotent)을 가지지 않는 작업, 예를 들어 '상품 구매', '송금', '데이터 삭제'와 같은 요청이라면, 이 공격으로 인해 해당 작업이 의도치 않게 여러 번 실행되는 심각한 부작용을 초래할 수 있다.64

- **완화 전략**: 이러한 위험 때문에 0-RTT 사용에는 엄격한 제약이 따른다.

  1. **멱등성 요청 제한**: 대부분의 구현체는 기본적으로 멱등성이 보장되는 HTTP GET 요청에 대해서만 0-RTT를 허용한다. POST와 같은 요청은 재전송 공격에 취약하므로 0-RTT로 보낼 수 없다.62
  2. **서버의 재전송 방어**: 서버는 수신한 0-RTT 데이터를 즉시 처리하지 않고, 핸드셰이크가 완료된 후에 처리하거나, 재전송을 탐지할 수 있는 제한된 시간 창(window)을 두는 등의 방어 메커니즘을 구현할 수 있다.
  3. **클라이언트 재시도 유도**: CDN이나 서버는 재전송 공격의 위험이 있다고 판단되는 0-RTT 요청에 대해 `425 Too Early`라는 HTTP 상태 코드로 응답할 수 있다. 이 응답을 받은 클라이언트는 0-RTT 시도를 포기하고, 완전한 1-RTT 핸드셰이크를 거친 후 해당 요청을 다시 보내게 된다.59

결론적으로 0-RTT는 강력한 성능 향상 도구이지만, 애플리케이션 개발자와 서버 운영자는 재전송 공격의 위험을 명확히 인지하고, 멱등성이 보장되는 안전한 요청에 대해서만 선택적으로 적용해야 한다.

## 6.  프로토콜 진화의 실제적 영향: 성능 벤치마크 및 적용 사례

TCP에서 QUIC으로의 진화는 단순히 이론적인 개선에 그치지 않고, 실제 인터넷 환경에서 측정 가능한 성능 향상과 새로운 애플리케이션의 등장을 이끌었다. 이 장에서는 HTTP/3의 성능을 벤치마크를 통해 분석하고, QUIC이 어떻게 HTTP를 넘어 다른 프로토콜의 혁신을 촉진하고 있는지 살펴본다.

### 6.1  HTTP/3 성능 분석: HTTP/2와의 비교

HTTP/3의 성능은 네트워크 환경에 따라 HTTP/2와 비교하여 상이한 결과를 보인다. 이는 QUIC의 설계 자체가 특정 환경에서의 약점을 보완하는 데 초점을 맞추고 있기 때문이다.

- **이상적인 네트워크 환경**: 유선 연결과 같이 대역폭이 충분하고 패킷 손실률이 0에 가까운 안정적인 네트워크 환경에서는 HTTP/3의 성능 이점이 크지 않거나, 오히려 HTTP/2보다 약간 저하될 수 있다. 이는 QUIC이 사용자 공간에서 동작하면서 발생하는 CPU 오버헤드와 모든 패킷에 대한 암호화 처리 부담 때문이다.65 고도로 최적화된 운영체제 커널의 TCP 스택이 더 효율적으로 동작할 수 있는 것이다. 일부 초기 벤치마크에서는 특정 구현체(Nginx)에서 HTTP/3가 HTTP/2보다 느리게 측정되기도 했다.67
- **열악한 네트워크 환경 (패킷 손실, 높은 지연)**: QUIC의 진정한 가치는 네트워크 품질이 저하될 때 드러난다.
  - **패킷 손실**: 한 연구에 따르면, 패킷 손실률이 1%만 발생해도 CUBIC 기반의 TCP 처리량은 70% 이상 급감하는 반면, BBR 기반의 QUIC은 약 8.5%의 감소에 그쳐 월등한 회복탄력성을 보였다.52 또 다른 연구에서는 15%의 패킷 손실이 있는 모바일 네트워크 환경에서 HTTP/3가 HTTP/2보다 55% 더 나은 페이지 로드 시간을 기록했다고 보고했다.61 이는 QUIC의 HOL 블로킹 해결과 향상된 손실 복구 메커니즘 덕분이다.
  - **높은 지연**: 클라이언트와 서버 간의 지리적 거리가 멀어 RTT(왕복 시간)가 높은 경우, HTTP/3의 성능 향상은 더욱 극적이다. 0-RTT/1-RTT 핸드셰이크는 연결 설정 시간을 크게 단축시킨다. 한 벤치마크에서는 뉴욕에서 서버에 접속할 때보다 3배 이상 먼 인도에서 접속할 때 HTTP/3의 속도 향상 폭이 3.5배 이상 커지는 결과를 보였다.68 이는 RTT가 높은 환경일수록 핸드셰이크 왕복 횟수를 줄이는 것이 전체 성능에 미치는 영향이 크기 때문이다.
- **모바일 환경 및 연결 마이그레이션**: 모바일 기기 사용자가 Wi-Fi와 셀룰러 네트워크 사이를 이동할 때, TCP 연결은 IP 주소 변경으로 인해 끊어지고 재설정되어야 한다. 이 과정에서 사용자는 서비스 중단을 경험하게 된다. 반면, QUIC의 **연결 마이그레이션** 기능은 연결 ID를 기반으로 하므로, 네트워크 인터페이스가 바뀌어도 연결을 끊김 없이 유지한다.38 이는 비디오 스트리밍이나 실시간 통신과 같이 지속적인 연결이 중요한 모바일 애플리케이션의 사용자 경험을 획기적으로 개선한다.69
- **초기 배포 현황 및 성과**: 이러한 장점 덕분에 QUIC과 HTTP/3는 빠르게 확산되었다. Google, Meta, Cloudflare와 같은 대규모 인터넷 기업들이 초기 도입을 주도했으며, Google 검색에서는 3%의 페이지 로드 시간(PLT) 개선, YouTube에서는 18%의 리버퍼링 시간 감소 효과를 보고했다.70 2024년 9월 기준으로, 전 세계 상위 1,000만 개 웹사이트 중 34%가 HTTP/3를 지원하며, 주요 웹 브라우저의 95% 이상이 이를 지원하고 있어 사실상 표준 기술로 자리 잡았다.72

이러한 결과들은 QUIC의 성능 우위가 '평균' 속도의 향상보다는 '꼬리 지연(Tail Latency)'의 감소에 있음을 시사한다. 즉, 이상적인 조건에서의 최고 속도 경쟁보다는, 네트워크 상태가 예측 불가능하게 나빠지는 상황에서 최악의 성능 저하를 방지하는 '회복탄력성(Resilience)'에 QUIC의 진정한 가치가 있다. 이는 웹 성능의 패러다임을 '최고 속도'에서 '안정적인 예측 가능성'으로 전환시키는 중요한 변화이다.

### 6.2  QUIC의 새로운 지평: 신규 프로토콜 활성화

QUIC은 HTTP/3를 위한 전송 계층을 넘어, 그 자체로 하나의 강력한 플랫폼이 되어 다양한 프로토콜의 혁신을 이끌고 있다. QUIC이 제공하는 저지연, 보안, 다중화, 이동성 지원은 기존 프로토콜들이 TCP 위에서 겪었던 한계를 극복할 새로운 기회를 제공한다.

- **DNS-over-QUIC (DoQ)**: DNS는 인터넷의 핵심 인프라이지만, 전통적으로 암호화되지 않은 UDP를 사용해 개인정보 유출에 취약했다. 이를 해결하기 위해 DoT(DNS-over-TLS)와 DoH(DNS-over-HTTPS)가 등장했지만, 각각 TCP HOL 블로킹과 HTTP 오버헤드라는 단점을 안고 있었다. DoQ(RFC 9250)는 DNS 쿼리를 QUIC을 통해 전송하는 표준이다.74 각 DNS 쿼리를 별도의 QUIC 스트림에 매핑함으로써 병렬 처리를 극대화하고 HOL 블로킹을 제거한다. 또한, UDP 기반이면서도 암호화와 신뢰성을 보장하여 DoT의 성능 문제와 DoH의 복잡성을 동시에 해결하는 이상적인 대안으로 평가받고 있다.76
- **WebTransport**: WebSockets는 오랫동안 웹에서 실시간 양방향 통신을 위한 표준이었지만, TCP 기반이기에 HOL 블로킹에 취약하고, 단일 스트림만 지원하는 한계가 있었다. WebTransport는 QUIC을 기반으로 하는 차세대 웹 통신 API이다.79 WebTransport의 가장 큰 특징은 하나의 연결 내에서 여러 개의 독립적인 스트림을 생성할 수 있고, TCP처럼 신뢰성 있는 순차적 데이터 전송을 위한 **스트림(Streams)**과 UDP처럼 신뢰성 없이 순서도 보장되지 않는 빠른 데이터 전송을 위한 **데이터그램(Datagrams)**을 동시에 지원한다는 점이다.80 이를 통해 개발자는 실시간 게임에서 플레이어의 위치 정보처럼 일부 손실되어도 괜찮은 데이터는 데이터그램으로 빠르게 보내고, 채팅 메시지처럼 반드시 전달되어야 하는 데이터는 스트림으로 안정적으로 보내는 등 애플리케이션의 요구에 맞춰 전송 방식을 유연하게 선택할 수 있다.82
- **SMB over QUIC**: SMB(Server Message Block)는 주로 내부망(LAN)에서 파일 공유를 위해 사용되는 프로토콜로, 전통적으로 TCP 445번 포트를 사용한다. 이 포트는 인터넷상에서 보안상의 이유로 대부분 차단되어 있어, 원격 파일 접근을 위해서는 복잡한 VPN 설정이 필요했다. Microsoft는 Windows Server 2022부터 **SMB over QUIC**을 도입했다.83 이는 SMB 트래픽 전체를 QUIC 터널로 캡슐화하여 인터넷 친화적인 UDP 443번 포트를 통해 전송하는 기술이다. 이를 통해 사용자들은 VPN 없이도 방화벽을 쉽게 통과하여 인터넷을 통해 안전하고 안정적으로 파일 서버에 접근할 수 있게 되었다.84 QUIC의 내장된 TLS 1.3 암호화가 VPN 수준의 보안을 제공하며, 연결 마이그레이션 기능은 모바일 사용자의 연결 안정성을 보장한다.86

이러한 사례들은 QUIC이 단지 더 빠른 HTTP를 위한 기술이 아니라, 인터넷 전송 계층의 근본적인 혁신을 이끄는 플랫폼으로서 기능하고 있음을 명확히 보여준다.

## 7.  결론 - 전송 계층의 미래와 남은 과제

### 8.1. HOL 블로킹 해결의 여정 요약

Head-of-Line 블로킹 문제 해결의 여정은 인터넷 프로토콜 진화의 핵심적인 서사 중 하나이다. 본 보고서는 이 문제의 근원을 TCP의 핵심 설계 철학인 '엄격한 순서 보장을 통한 신뢰성 확보'에서 찾았다. 이 설계는 단일 바이트 스트림 추상화 모델을 낳았고, 이는 본질적으로 전송 계층 HOL 블로킹의 씨앗이 되었다.

초기 웹 환경에서 HTTP/1.1은 애플리케이션 계층에서 순차적 처리라는 자체적인 HOL 블로킹 문제를 가지고 있었다. 이 문제를 해결하기 위해 등장한 HTTP/2의 멀티플렉싱은 역설적으로 문제의 본질을 더 깊은 곳으로 전이시켰다. 애플리케이션 계층의 문제가 해결되자, 모든 스트림이 의존하는 단일 TCP 연결의 HOL 블로킹 문제가 전면에 부상하며 새로운 병목으로 작용했다.

이 근본적인 문제를 해결하기 위해 등장한 QUIC은 패러다임의 전환을 이뤄냈다. UDP를 기반으로 사용자 공간에서 신뢰성과 보안을 재구현하고, 전송 계층 자체에 '독립적인 스트림' 개념을 도입함으로써, QUIC은 수십 년간 지속된 HOL 블로킹 문제를 마침내 근본적으로 해결했다. HTTP/3는 이러한 QUIC의 능력 위에서 진정한 의미의 다중화를 실현한 애플리케이션 프로토콜이다.

### 7.1  현재의 기술적 과제

QUIC은 성공적으로 전송 계층의 새로운 표준으로 자리 잡고 있지만, 그 과정에서 새로운 기술적 과제들이 드러나고 있다.

- **미들박스 간섭 및 UDP 차단**: QUIC의 가장 큰 배포 장애물 중 하나는 여전히 일부 네트워크 환경에 존재하는 중간 장비(미들박스)이다. 많은 기업 방화벽이나 구형 네트워크 장비는 알려지지 않은 UDP 트래픽, 특히 웹 트래픽에 사용되지 않던 UDP 443 포트를 통한 트래픽을 차단하거나 속도를 제한하는 정책을 가지고 있다.87 이 경우, QUIC 연결 시도가 실패하고 클라이언트는 HTTP/2나 HTTP/1.1과 같은 TCP 기반의 레거시 프로토콜로 대체(Fallback)하게 되어 QUIC의 이점을 전혀 누릴 수 없게 된다.89

- **CPU 오버헤드 및 성능**: QUIC은 프로토콜의 복잡한 로직(흐름 제어, 혼잡 제어, 암호화 등)을 커널이 아닌 사용자 공간 라이브러리에서 처리한다. 이는 커널과 사용자 공간 간의 빈번한 데이터 복사와 컨텍스트 전환을 유발하여, 고도로 최적화된 커널 TCP 스택에 비해 더 높은 CPU 사용량을 보인다.66 특히 초고속 네트워크 환경(수 Gbps 이상)이나, CPU 자원이 제한된 모바일 및 IoT 기기에서는 이 오버헤드가 실제 처리량(Throughput)을 저하시키는 병목 현상으로 작용할 수 있다.65 이 문제를 해결하기 위해 QUIC의 암호화 및 패킷 처리 일부를 네트워크 카드(NIC)에서 처리하는 하드웨어 오프로딩(Hardware Offloading) 기술에 대한 연구가 진행되고 있다.93

- **암호화 트래픽의 가시성 문제**: QUIC의 설계 철학은 '기본적으로 암호화(Encrypted by Default)'이다. 패킷 번호를 포함한 대부분의 전송 메타데이터까지 암호화함으로써 사용자의 프라이버시를 보호하고 중간 장비의 간섭을 막는다. 하지만 이는 네트워크 관리자와 보안 담당자에게는 큰 도전 과제이다. 기존의 네트워크 모니터링 및 장애 분석 도구들은 TCP 헤더의 정보를 분석하여 RTT, 패킷 손실, 재전송 등을 파악했지만, 암호화된 QUIC 트래픽에서는 이러한 정보에 접근할 수 없다.94 이는 트래픽 분석, 성능 문제 해결, 보안 위협 탐지를 매우 어렵게 만든다.95

- **관측 가능성(Observability) 확보 노력**: 이러한 가시성 문제를 해결하기 위해 IETF QUIC 워킹 그룹을 중심으로 다양한 노력이 이루어지고 있다. **qlog**는 QUIC 엔드포인트에서 발생하는 상세한 이벤트(패킷 송수신, 손실 탐지, 혼잡 윈도우 변화 등)를 표준화된 JSON 형식으로 기록하는 로깅 스키마이다.98 이 로그 파일을 

  **qvis**와 같은 시각화 도구로 분석하면, 암호화된 트래픽의 내부 동작을 상세히 파악할 수 있다.99 또한, RTT와 같은 핵심 성능 지표를 외부에서도 측정할 수 있도록 **스핀 비트(Spin Bit)**라는 1비트 플래그를 QUIC 헤더에 추가하는 방안이 논의되었다. 스핀 비트는 RTT마다 값이 바뀌도록 설계되어 외부 관찰자가 RTT를 추정할 수 있게 하지만, 개인정보 유출 가능성에 대한 우려로 선택적(Optional) 기능으로 남아 있어 실제 채택률은 제한적이다.101

### 7.2  QUIC의 향후 확장 기능 전망

QUIC은 그 자체로 완성된 프로토콜이 아니라, 지속적으로 진화하는 전송 계층 혁신의 플랫폼이다. IETF QUIC 워킹 그룹은 QUIC의 핵심 기능 위에 다양한 확장 기능을 표준화하는 작업을 활발히 진행하고 있다.

- **다중 경로 (Multipath QUIC)**: MPTCP의 개념을 QUIC에 도입하는 것으로, 단일 QUIC 연결이 스마트폰의 Wi-Fi와 셀룰러 네트워크 같은 여러 경로를 동시에 활용할 수 있게 하는 기술이다. 이를 통해 가용 대역폭을 통합하여 전송 속도를 높이고, 한쪽 경로에 장애가 발생하더라도 다른 경로를 통해 통신을 끊김 없이 이어가는 '심리스 핸드오버(Seamless Handover)'를 구현할 수 있다.103 이는 특히 이동성이 중요한 모바일 환경에서 연결 안정성을 획기적으로 향상시킬 것으로 기대되며, 현재 IETF에서 핵심 확장 기능으로 활발히 논의 중이다.104
- **전방 오류 수정 (Forward Error Correction, FEC)**: FEC는 데이터 패킷과 함께 중복(Parity) 패킷을 전송하여, 수신 측에서 일부 패킷이 손실되더라도 재전송을 기다리지 않고 즉시 복구할 수 있게 하는 기술이다.107 이는 RTT가 매우 길어 재전송 비용이 극도로 높은 위성 통신이나, 극도의 저지연이 요구되는 실시간 미디어 스트리밍과 같은 특정 시나리오에서 유용할 수 있다.108 아직 표준화 초기 단계에 있지만, QUIC의 유연한 프레임 구조는 이러한 고급 기능을 통합하기에 용이하다.

QUIC의 등장은 '성능'과 '보안'의 관계를 재정립했다. 전통적으로 보안(암호화)은 성능을 저해하는 오버헤드로 여겨졌다. 그러나 QUIC의 사례는 역설적으로 강력한 암호화가 어떻게 성능 향상의 전제 조건이 될 수 있는지를 보여주었다. QUIC의 전면적인 암호화는 프로토콜의 진화를 가로막던 '중간 장비에 의한 화석화' 문제를 해결하는 유일한 열쇠였다. 이 보안이라는 방패 덕분에 QUIC은 UDP 위에서 자유롭게 배포되고, 독립적 스트림, 0-RTT, 연결 마이그레이션과 같은 혁신적인 성능 향상 기능들을 현실화할 수 있었다. 이제, 이렇게 향상된 성능과 유연성은 다시 SMB over QUIC과 같은 새로운 보안 아키텍처를 가능하게 하고 있다. 이처럼 보안이 성능을 위한 길을 열고, 성능이 다시 보안의 적용 범위를 넓히는 선순환 구조를 만들어낸 것이야말로 QUIC이 전송 계층의 미래에 던지는 가장 중요한 시사점일 것이다. QUIC은 단순히 더 빠른 프로토콜을 넘어, 인터넷의 근본적인 진화를 이끄는 새로운 플랫폼으로 기능할 것이다.

#### 참고자료

1. en.wikipedia.org, accessed August 6, 2025, https://en.wikipedia.org/wiki/Head-of-line_blocking
2. Understanding Head-of-Line Blocking in Networking - JumpCloud, accessed August 6, 2025, https://jumpcloud.com/it-index/what-is-head-of-line-blocking
3. Head of Line blocking - Monospace Mentor Wiki - Obsidian Publish, accessed August 6, 2025, https://publish.obsidian.md/monospacementor/Notes/Head+of+Line+blocking
4. Head-of-line blocking - Glossary | MDN, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Glossary/Head_of_line_blocking
5. Head-of-line (HOL) blocking in HTTP/1 and HTTP/2 | by Abhishek Varshney, accessed August 6, 2025, https://engineering.cred.club/head-of-line-hol-blocking-in-http-1-and-http-2-50b24e9e3372
6. The Evolution of HTTP: Refining Head-of-Line Blocking Solutions with HTTP/3 - Shuru, accessed August 6, 2025, https://shurutech.com/the-evolution-of-http/
7. How HTTP/2 Fixes HTTP/1.1 Pipelining Issues | by Jyoti Sheoran - Medium, accessed August 6, 2025, https://jyotibhambhu.medium.com/how-http-2-fixes-http-1-1-pipelining-issues-6362377f7c3b
8. http - How does HTTP2 solve Head of Line blocking (HOL) issue ..., accessed August 6, 2025, https://stackoverflow.com/questions/45583861/how-does-http2-solve-head-of-line-blocking-hol-issue
9. rmarx/holblocking-blogpost: Blogpost on Head-of-Line blocking from HTTP/1 to HTTP/3, accessed August 6, 2025, https://github.com/rmarx/holblocking-blogpost
10. How HTTP/2 Addresses Head-of-Line (HOL) Blocking - DEV ..., accessed August 6, 2025, https://dev.to/devcorner/how-http2-addresses-head-of-line-hol-blocking-3om2
11. Transmission Control Protocol - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/Transmission_Control_Protocol
12. SPDY Head of Line blocking - Stack Overflow, accessed August 6, 2025, https://stackoverflow.com/questions/25221954/spdy-head-of-line-blocking
13. One issue is head of line blocking in TCP when multiplexing requests over a sing, accessed August 6, 2025, https://news.ycombinator.com/item?id=18429017
14. Packet loss - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/Packet_loss
15. Retransmission Rules for TCP | Baeldung on Computer Science, accessed August 6, 2025, https://www.baeldung.com/cs/tcp-retransmission-rules
16. Packet loss, accessed August 6, 2025, https://allegro-packets.com/en/blog-posts/packetloss
17. TCP RTOs: Retransmission Timeouts & Application Performance Degradation - Extrahop, accessed August 6, 2025, https://www.extrahop.com/blog/retransmission-timeouts-rtos-application-performance-degradation
18. TCP Retransmission - Tutorialspoint, accessed August 6, 2025, https://www.tutorialspoint.com/tcp-retransmission
19. Packet Loss Detection Algorithms - GeeksforGeeks, accessed August 6, 2025, https://www.geeksforgeeks.org/computer-networks/packet-loss-detection-algorithms/
20. Connection management in HTTP/1.x - MDN Web Docs, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Connection_management_in_HTTP_1.x
21. Head-of-Line Blocking: Explanation and Designing a Flexible Network Layer - Medium, accessed August 6, 2025, https://medium.com/@aditimishra_541/head-of-line-blocking-explanation-and-designing-a-flexible-network-layer-a1b44cde01be
22. HTTP pipelining - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/HTTP_pipelining
23. What is the difference between HTTP/1.1 pipelining and HTTP/2 multiplexing?, accessed August 6, 2025, https://stackoverflow.com/questions/34478967/what-is-the-difference-between-http-1-1-pipelining-and-http-2-multiplexing
24. How Does QUIC Differ from TCP? - IO River, accessed August 6, 2025, https://www.ioriver.io/questions/how-does-quic-differ-from-tcp
25. QUIC for the kernel - LWN.net, accessed August 6, 2025, https://lwn.net/Articles/1029851/
26. QUIC - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/QUIC
27. A QUIC progress report - APNIC Blog, accessed August 6, 2025, https://blog.apnic.net/2025/06/17/a-quic-progress-report/
28. The Road to QUIC - The Cloudflare Blog, accessed August 6, 2025, https://blog.cloudflare.com/the-road-to-quic/
29. How does QUIC ensure reliability if it uses UDP? - Quora, accessed August 6, 2025, https://www.quora.com/How-does-QUIC-ensure-reliability-if-it-uses-UDP
30. What Are QUIC and HTTP/3? - F5 Networks, accessed August 6, 2025, https://www.f5.com/glossary/quic-http3
31. QUIC Wire Layout Specification - Google Docs, accessed August 6, 2025, https://docs.google.com/document/d/1WJvyZflAO2pq77yOLbp9NsGjC1CHetAXV8I0fQe-B_U
32. What is QUIC? How Does It Boost HTTP/3? - CDNetworks, accessed August 6, 2025, https://www.cdnetworks.com/blog/media-delivery/what-is-quic/
33. Thoughts on QUIC? : r/networking - Reddit, accessed August 6, 2025, https://www.reddit.com/r/networking/comments/1e8p6mz/thoughts_on_quic/
34. RFC 9114: HTTP/3, accessed August 6, 2025, https://www.rfc-editor.org/rfc/rfc9114.html
35. What is QUIC? Understand The Protocol - Check Point Software, accessed August 6, 2025, https://www.checkpoint.com/cyber-hub/network-security/what-is-quic/
36. IETF QUIC v1 Design, accessed August 6, 2025, https://www.cse.wustl.edu/~jain/cse570-21/ftp/quic/index.html
37. Design and Evaluation - Multipath QUIC, accessed August 6, 2025, https://multipath-quic.org/conext17-deconinck.pdf
38. What is HTTP/3? - Cloudflare, accessed August 6, 2025, https://www.cloudflare.com/learning/performance/what-is-http3/
39. QUIC vs TCP: “The Ultimate Benchmark for Modern Network Performance” | by Abhi Patel, accessed August 6, 2025, https://medium.com/@abhi.patel.7172/quic-vs-tcp-the-ultimate-benchmark-for-modern-network-performance-22be6e5af66d
40. RFC 9114 - HTTP/3 - IETF Datatracker, accessed August 6, 2025, https://datatracker.ietf.org/doc/html/rfc9114
41. Open sourcing h3i: a command line tool and library for low-level HTTP/3 testing and debugging - The Cloudflare Blog, accessed August 6, 2025, https://blog.cloudflare.com/h3i/
42. quic.xargs.org, accessed August 6, 2025, [https://quic.xargs.org/#:~:text=Each%20QUIC%20packet%20contains%20a,to%20establish%20a%20secure%20connection.](https://quic.xargs.org/#:~:text=Each QUIC packet contains a,to establish a secure connection.)
43. QUIC vs. TCP—Development and Monitoring Guide - Catchpoint, accessed August 6, 2025, https://www.catchpoint.com/http2-vs-http3/quic-vs-tcp
44. QUIC Loss Detection and Congestion Control - IETF, accessed August 6, 2025, https://www.ietf.org/archive/id/draft-ietf-quic-recovery-27.html
45. QUIC vs TCP: A Performance Evaluation over LTE with NS-3 - Scirp.org., accessed August 6, 2025, https://www.scirp.org/journal/paperinformation?paperid=113892
46. RFC 9002: QUIC Loss Detection and Congestion Control, accessed August 6, 2025, https://quicwg.org/base-drafts/rfc9002.html
47. QUIC at 10,000 feet - Google Docs, accessed August 6, 2025, https://docs.google.com/document/d/1gY9-YNDNAB1eip-RTPbqphgySwSNSDHLq9D5Bty4FSU/mobilebasic
48. QUIC Loss Detection and Congestion Control, accessed August 6, 2025, https://greenbytes.de/tech/webdav/draft-ietf-quic-recovery-14.html
49. The Illustrated QUIC Connection: Every Byte Explained, accessed August 6, 2025, https://quic.xargs.org/
50. Explicitly communicate and include ACK delay in RTO computations · Issue #981 · quicwg/base-drafts - GitHub, accessed August 6, 2025, https://github.com/quicwg/base-drafts/issues/981
51. BBR: Congestion-Based Congestion Control - Communications of the ACM, accessed August 6, 2025, https://cacm.acm.org/practice/bbr-congestion-based-congestion-control/
52. Path Quality: Is BBR the Future of Congestion ... - ThousandEyes, accessed August 6, 2025, https://www.thousandeyes.com/blog/path-quality-brr-future-congestion-avoidance
53. BBR TCP (Bottleneck Bandwidth and RTT) - GÉANT federated confluence, accessed August 6, 2025, https://wiki.geant.org/pages/viewpage.action?pageId=121340614&src=contextnavpagetreemode
54. Exploring the BBRv2 Congestion Control Algorithm for use on Data Transfer Nodes - Internet2, accessed August 6, 2025, https://internet2.edu/wp-content/uploads/2022/12/techex22-AdvancedNetworking-ExploringtheBBRv2CongestionControlAlgorithm-Tierney.pdf
55. BBR TCP (Bottleneck Bandwidth and RTT) - GÉANT federated confluence, accessed August 6, 2025, https://wiki.geant.org/pages/viewpage.action?pageId=121340614
56. BBR congestion control [LWN.net], accessed August 6, 2025, https://lwn.net/Articles/701165/
57. Integration and Evaluation of QUIC and TCP-BBR in longhaul Science Data Transfers - EPJ Web of Conferences, accessed August 6, 2025, https://www.epj-conferences.org/articles/epjconf/pdf/2019/19/epjconf_chep2018_08026.pdf
58. TCP BBR - Exploring TCP congestion control | by Andree Toonk - Medium, accessed August 6, 2025, https://atoonk.medium.com/tcp-bbr-exploring-tcp-congestion-control-84c9c11dc3a9
59. Even faster connection establishment with QUIC 0-RTT resumption - The Cloudflare Blog, accessed August 6, 2025, https://blog.cloudflare.com/even-faster-connection-establishment-with-quic-0-rtt-resumption/
60. RFC 9001: Using TLS to Secure QUIC, accessed August 6, 2025, https://www.rfc-editor.org/rfc/rfc9001.html
61. The Next-Gen Protocol: HTTP3 and its Impact on Web Performance - CloudPanel, accessed August 6, 2025, https://www.cloudpanel.io/blog/http3-vs-http2/
62. Lightning-Fast Requests with Early Data | Akamai, accessed August 6, 2025, https://www.akamai.com/blog/edge/lightning-fast-requests-with-early-data
63. 0-RTT sounds nice, until you get to appendix E.5. Everyone should read this - Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=16667036
64. How Resilient is QUIC to Security and Privacy Attacks? - arXiv, accessed August 6, 2025, https://arxiv.org/html/2401.06657v3
65. QUIC is not Quick Enough over Fast Internet - arXiv, accessed August 6, 2025, https://arxiv.org/html/2310.09423v2
66. QUIC and HTTP/3 : Too big to fail?! - Web Performance Calendar, accessed August 6, 2025, https://calendar.perfplanet.com/2018/quic-and-http-3-too-big-to-fail/
67. HTTP/2 vs. HTTP/3 performance benchmark - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=-0yu_zOFilg
68. HTTP/3 is Fast! - Request Metrics, accessed August 6, 2025, https://requestmetrics.com/web-performance/http3-is-fast/
69. QUIC vs. TCP: Detailed Comparison of Transport Layer Protocols, accessed August 6, 2025, https://cheapsslweb.com/blog/quic-vs-tcp-detailed-comparison-of-transport-layer-protocols/
70. A first look at HTTP/3 adoption and performance - ArTS, accessed August 6, 2025, https://arts.units.it/retrieve/2d466be3-5a20-41b8-afe0-ae2a0291c3e2/1-s2.0-S0140366422000421-main-Post_print.pdf
71. Measuring QUIC vs TCP on mobile and desktop | APNIC Blog, accessed August 6, 2025, https://blog.apnic.net/2018/01/29/measuring-quic-vs-tcp-mobile-desktop/
72. HTTP/3 - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/HTTP/3
73. Why HTTP/3 is eating the world | APNIC Blog, accessed August 6, 2025, https://blog.apnic.net/2023/09/25/why-http-3-is-eating-the-world/
74. RFC 9250: DNS over Dedicated QUIC Connections, accessed August 6, 2025, https://www.rfc-editor.org/rfc/rfc9250.html
75. RFC 9250 - DNS over Dedicated QUIC Connections - IETF Datatracker, accessed August 6, 2025, https://datatracker.ietf.org/doc/rfc9250/
76. How Does DoQ Work? - UK DNS Privacy Project, accessed August 6, 2025, https://dnsprivacy.org.uk/docs/how-it-works/encryption_methods/doq.html
77. A Datagram Extension to DNS over QUIC: Proven Resource Conservation in the Internet of Things - arXiv, accessed August 6, 2025, https://arxiv.org/html/2504.09200v1
78. New DNS over QUIC protocol makes encrypted DNS traffic faster and more efficient - SIDN, accessed August 6, 2025, https://www.sidn.nl/en/news-and-blogs/new-dns-over-quic-protocol-makes-encrypted-dns-traffic-faster-and-more-efficient
79. WebTransport API - MDN Web Docs - Mozilla, accessed August 6, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API
80. What is WebTransport and can it replace WebSockets? - Ably, accessed August 6, 2025, https://ably.com/blog/can-webtransport-replace-websockets
81. Building Real-Time Web Apps with WebTransport (Replacing WebSockets?), accessed August 6, 2025, https://dev.to/mukhilpadmanabhan/building-real-time-web-apps-with-webtransport-replacing-websockets-3348
82. Using WebTransport [...] means that you don't have to worry about head-of-line... | Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=32873367
83. SMB over QUIC - Microsoft Learn, accessed August 6, 2025, https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-over-quic
84. SMB Over QUIC Introduced by Microsoft - Visuality Systems, accessed August 6, 2025, https://visualitynq.com/resources/articles/smb-over-quic-and-azure-edition/
85. Secure Access to Windows File Shares Over the Internet with SMB Over QUIC – No VPN Needed! - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=h2DrA-LytjQ&pp=0gcJCfwAo7VqN5tD
86. SMB over QUIC Testing Guide – Part I - StarWind, accessed August 6, 2025, https://www.starwindsoftware.com/blog/smb-over-quic-testing-guide-part-1/
87. Is this sustainable though? Won't the middle boxes adapt? - Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=25970429
88. Managing the QUIC Protocol - Zscaler Help Portal, accessed August 6, 2025, https://help.zscaler.com/zia/managing-quic-protocol
89. nanog: Re: QUIC traffic throttled on AT&T residential - Seclists.org, accessed August 6, 2025, https://seclists.org/nanog/2020/Feb/327
90. USG and QUIC | Ubiquiti Community, accessed August 6, 2025, https://community.ui.com/questions/USG-and-QUIC/3adbc24d-5268-4463-94f2-f43e87e40775
91. Comparing TCP and QUIC - Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=33431669
92. Continuous benchmarking of open-source QUIC implementations - Networked Systems Group, accessed August 6, 2025, https://nsg.ee.ethz.ch/files/public/theses/2024-quic_benchmark_suite/thesis-1.pdf
93. QUIC Kernel and Device Offload - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=IAvQhJSm6O8
94. Debugging QUIC and HTTP/3 with qlog and qvis - Chair of Network Architectures and Services, accessed August 6, 2025, https://www.net.in.tum.de/fileadmin/TUM/NET/NET-2021-05-1/NET-2021-05-1_06.pdf
95. Define Traffic to Decrypt - Palo Alto Networks, accessed August 6, 2025, https://docs.paloaltonetworks.com/pan-os/11-0/pan-os-admin/decryption/define-traffic-to-decrypt
96. QUIC Decryption - Cisco Secure Essentials, accessed August 6, 2025, https://secure.cisco.com/secure-firewall/docs/quic-decryption
97. Solving Encrypted Traffic Challenges with Prisma Access Browser - Palo Alto Networks Blog, accessed August 6, 2025, https://www.paloaltonetworks.com/blog/sase/solving-encrypted-traffic-challenges-with-prisma-access-browser/
98. QUIC event definitions for qlog - IETF Datatracker, accessed August 6, 2025, https://datatracker.ietf.org/doc/draft-ietf-quic-qlog-quic-events/
99. QVis - QUIC and HTTP/3 visualization tools - GitHub, accessed August 6, 2025, https://github.com/quiclog/qvis
100. qvis: tools and visualizations for QUIC and HTTP/3, accessed August 6, 2025, https://qvis.quictools.info/
101. draft-ietf-quic-spin-exp-01, accessed August 6, 2025, https://datatracker.ietf.org/doc/html/draft-ietf-quic-spin-exp
102. Does It Spin? On the Adoption and Use of QUIC's Spin Bit - arXiv, accessed August 6, 2025, https://arxiv.org/pdf/2310.02599
103. I found [1] which describes the architectural difference between MPTCP and QUIC,... | Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=40091664
104. Multipath QUIC, accessed August 6, 2025, https://multipath-quic.org/
105. QUIC Working Group, accessed August 6, 2025, https://quicwg.org/
106. draft-ietf-quic-multipath-15 - Multipath Extension for QUIC - IETF Datatracker, accessed August 6, 2025, https://datatracker.ietf.org/doc/draft-ietf-quic-multipath/
107. FEC Extension for QUIC - IETF, accessed August 6, 2025, https://www.ietf.org/id/draft-zheng-quic-fec-extension-00.html
108. QUIC future | HTTP/3 explained - README, accessed August 6, 2025, https://http3-explained.haxx.se/en/quic-future
109. Forward Error Correction (FEC). - Media over QUIC, accessed August 6, 2025, https://quic.video/blog/forward-error-correction
# UART 통신



본 보고서는 범용 비동기화 송수신기(Universal Asynchronous Receiver/Transmitter, UART)에 대한 심층적이고 포괄적인 기술적 고찰을 목적으로 한다. UART는 현대 전자공학 및 임베디드 시스템 분야에서 가장 기본적이면서도 널리 사용되는 직렬 통신 프로토콜 중 하나이다. 1 본 문서는 UART의 기본 작동 원리, 데이터 프레임의 구조적 분석, 보율(Baud Rate)의 개념과 클럭 정확도의 중요성, 그리고 TTL, RS-232, RS-485와 같은 다양한 물리 계층 표준에 이르기까지 그 기술적 근간을 상세히 탐구한다.

나아가, 본 보고서는 단순한 기능적 설명을 넘어 각 기술 요소가 지니는 공학적 의미와 설계상의 트레이드오프(trade-off)를 분석적으로 조명한다. I2C, SPI와 같은 다른 주요 직렬 통신 프로토콜과의 비교 분석을 통해 각 기술의 장단점과 적합한 응용 분야를 명확히 제시한다. 또한, 디버그 콘솔, GPS 모듈 연동 등 실제 응용 사례를 구체적으로 살펴보고, 현장에서 빈번하게 발생하는 프레이밍 오류, 오버런 오류 등의 문제점과 그 해결 방안을 체계적으로 다룬다. 이를 통해 독자들이 UART에 대한 피상적인 이해를 넘어, 시스템 설계와 디버깅 과정에서 마주할 수 있는 실질적인 문제에 대처할 수 있는 깊이 있는 통찰을 얻도록 하는 것을 최종 목표로 한다.


USB, 이더넷(Ethernet)과 같은 고속, 고기능성 인터페이스가 데이터 통신의 주류로 자리 잡은 현대에도 불구하고, UART는 그 특유의 가치로 인해 임베디드 시스템 분야에서 여전히 핵심적인 위상을 차지하고 있다. 4 그 이유는 역설적이게도 UART의 본질적 한계에서 비롯된 장점, 즉 극도의 단순성, 저렴한 구현 비용, 그리고 검증된 신뢰성에 있다. 4

고속 통신 프로토콜은 복잡한 소프트웨어 스택과 상당한 하드웨어 리소스를 요구하며, 이는 시스템 부팅 초기 단계나 자원이 극히 제한된 저가형 마이크로컨트롤러(MCU) 환경에서는 큰 부담으로 작용한다. 4 반면, UART는 최소한의 하드웨어(일반적으로 TX, RX 두 개의 핀)와 몇 개의 레지스터 설정만으로 즉각적인 통신 채널을 구축할 수 있다. 7 이러한 특성 덕분에 UART는 시스템의 가장 근원적인 상태를 모니터링하고 제어하는 '최후의 보루'와 같은 디버깅 인터페이스, 즉 디버그 콘솔(Debug Console)로서 대체 불가능한 역할을 수행한다. 9 커널 부팅 메시지나 펌웨어의 로그를 출력하는 데 UART만큼 간단하고 확실한 방법은 없다.

또한, 수많은 센서 모듈, GPS, 블루투스 모듈 등 주변 장치와의 통신에서 UART는 여전히 표준 인터페이스로 통용된다. 9 나아가, 대부분의 최신 집적 회로(IC)는 UART 기능에 동기 통신 기능까지 통합한 USART(Universal Synchronous/Asynchronous Receiver/Transmitter) 형태로 통신 블록을 내장하고 있어, 그 활용성과 범용성은 더욱 확장되고 있다. 1 이처럼 UART의 '단순함'은 기술적 복잡성이 날로 증가하는 현대 임베디드 환경 속에서 오히려 '필수불가결함'이라는 역설적인 가치를 창출하며 그 생명력을 굳건히 유지하고 있다.



데이터 전송 방식은 크게 병렬(Parallel)과 직렬(Serial) 통신으로 구분된다. 병렬 통신은 데이터의 각 비트에 해당하는 다수의 전송 라인을 사용하여 여러 비트를 동시에 전송하는 방식이다. 3 예를 들어 8비트 데이터를 전송하기 위해 8개의 데이터 라인을 사용하는 것이다. 이 방식은 단위 시간당 많은 양의 데이터를 보낼 수 있어 속도가 빠르다는 장점이 있으나, 다수의 배선이 필요하여 비용이 증가하고 회로가 복잡해진다. 7 또한, 전송 거리가 길어질수록 각 라인 간의 신호 도착 시간 차이(skew)나 상호 간섭(crosstalk)으로 인해 신호 무결성을 유지하기 어렵다는 치명적인 단점을 가진다. 14

반면, 직렬 통신은 단 하나의 데이터 라인을 통해 데이터를 비트 단위로 순차적으로 전송하는 방식이다. 7 배선 수가 획기적으로 줄어들어 비용이 저렴하고, 장거리 전송에 매우 유리하다. 14 UART를 비롯하여 I2C, SPI, USB 등 현대의 거의 모든 통신 프로토콜은 직렬 통신 방식을 기반으로 한다. 속도 면에서는 병렬 통신에 비해 느릴 수 있으나, 기술의 발전으로 수십 Gbps에 이르는 고속 직렬 통신이 가능해지면서 이러한 단점은 상당 부분 극복되었다.


직렬 통신은 다시 동기(Synchronous) 방식과 비동기(Asynchronous) 방식으로 나뉜다. UART는 이름에서 알 수 있듯이 비동기 통신을 사용한다. 비동기 통신의 가장 핵심적인 특징은 송신기와 수신기가 데이터 타이밍을 맞추기 위한 별도의 클럭 신호(Clock Signal)를 공유하지 않는다는 점이다. 1

동기 통신에서는 마스터 장치가 생성하는 클럭 신호에 맞추어 슬레이브 장치가 데이터를 송수신하므로 비트 단위의 완벽한 동기화가 보장된다. 17 이는 데이터 전송 효율을 높여 고속 통신에 유리하지만, 데이터를 전송하지 않을 때에도 클럭 신호는 계속 유지되어야 하며, 무엇보다 추가적인 클럭 라인이 필요하다는 단점이 있다. 15

비동기 통신은 이러한 클럭 라인을 제거하여 하드웨어를 단순화하는 대신, 다른 방법으로 동기화를 구현한다. UART에서는 두 가지 핵심적인 약속을 통해 이를 해결한다. 첫째, 송신기와 수신기는 사전에 통신 속도, 즉 **보율(Baud Rate)**을 동일하게 설정해야 한다. 7 둘째, 전송할 데이터의 앞뒤에 **시작 비트(Start Bit)**와 **정지 비트(Stop Bit)**라는 특별한 신호를 붙여 데이터 프레임(Frame)의 시작과 끝을 명확히 구분한다. 20 수신기는 시작 비트의 신호 변화(edge)를 감지하여 데이터 수신을 시작하고, 미리 약속된 보율에 맞춰 각 비트를 샘플링(sampling)함으로써 데이터를 복원한다. 15

이러한 비동기 방식의 본질적인 약점은 '클럭의 부재'에서 오는 누적 타이밍 오차 가능성이다. 송신기와 수신기의 내부 클럭 생성기(oscillator)는 완벽하게 동일할 수 없으므로 미세한 주파수 차이(clock drift)가 존재한다. 22 이 오차는 프레임 내에서 비트가 진행될수록 누적되어, 마지막 비트를 샘플링할 때쯤에는 오차가 가장 커지게 된다. 23 만약 프레임의 길이가 무한하다면 이 누적 오차는 결국 샘플링 실패로 이어질 것이다. UART 프로토콜은 이 문제를 '프레임 단위 재동기화'라는 지극히 실용적인 방식으로 해결한다. 즉, 데이터를 5~9비트의 짧은 단위로 나누고, 각 프레임의 시작을 알리는 시작 비트의 하강 에지를 기준으로 수신기 타이머를 매번 재설정한다. 4 이를 통해 장시간 통신에서 발생할 수 있는 누적 타이밍 오차 문제를 원천적으로 방지한다. 이는 클럭 라인을 제거하여 얻는 단순성의 대가로, 약간의 오버헤드(시작/정지 비트)를 감수하는 효율적인 공학적 타협의 결과물이다.


UART 하드웨어의 핵심적인 역할은 병렬 데이터와 직렬 데이터 간의 상호 변환이다. 이 기능은 내부적으로 **시프트 레지스터(Shift Register)**에 의해 수행된다. 1

송신 과정에서, CPU는 내부 데이터 버스를 통해 8비트(일반적인 경우) 단위의 병렬 데이터를 UART의 송신 버퍼(Transmit Buffer)에 기록한다. UART의 송신 로직은 이 병렬 데이터를 시프트 레지스터로 로드한 후, 시작 비트, 패리티 비트(선택 사항), 정지 비트를 추가하여 완전한 데이터 프레임을 구성한다. 13 그 다음, 설정된 보율에 맞춰 시프트 레지스터의 내용을 한 비트씩 TX(Transmit) 핀으로 밀어내어 직렬 데이터 스트림을 생성한다. 24

수신 과정은 이의 역순으로 진행된다. UART의 수신 로직은 RX(Receive) 핀으로 들어오는 직렬 데이터 스트림을 지속적으로 감시하다가, 시작 비트를 감지하면 보율에 맞춰 후속 비트들을 순차적으로 샘플링하여 수신 시프트 레지스터에 저장한다. 24 정지 비트까지 수신하여 하나의 완전한 프레임이 완성되면, 시프트 레지스터에 저장된 데이터 비트들을 병렬 데이터로 변환하여 수신 버퍼(Receive Buffer)로 옮긴다. CPU는 이 수신 버퍼를 읽어 데이터를 획득하게 된다. 13


UART는 데이터 송신을 위한 TX 핀과 수신을 위한 RX 핀, 이렇게 두 개의 독립적인 데이터 라인을 사용한다. 7 이 두 라인은 서로에게 영향을 주지 않고 독립적으로 동작하기 때문에, 데이터의 송신과 수신이 동시에 이루어질 수 있다. 이를 **전이중(Full-Duplex)** 통신 방식이라고 한다. 5

전이중 통신을 위해서는 두 장치 간의 연결이 교차 형태로 이루어져야 한다. 즉, 장치 A의 TX 핀은 장치 B의 RX 핀에 연결되고, 장치 A의 RX 핀은 장치 B의 TX 핀에 연결되어야 한다. 7 또한, 두 장치는 반드시 공통 접지(Common Ground)를 공유해야 안정적인 신호 레벨 기준을 확보할 수 있다. 이와 대비되는 개념으로, 하나의 통신 채널을 통해 양방향 통신이 가능하지만 한 번에 한 방향으로만 전송할 수 있는 방식을 반이중(Half-duplex), 한 방향으로만 전송이 가능한 방식을 단방향(Simplex) 통신이라고 한다. 5


UART 통신은 **프레임(Frame)**이라고 불리는 표준화된 패킷 단위로 데이터를 전송한다. 4 이 프레임의 구조는 송신기와 수신기 간의 약속이며, 성공적인 통신을 위해 양측이 동일한 프레임 구조를 사용하도록 설정되어야 한다. 4 하나의 프레임은 통신 라인의 유휴 상태(Idle State)에서 시작하여, 시작 비트, 데이터 비트, 선택적 패리티 비트, 그리고 정지 비트의 순서로 구성된다. 1

**UART 데이터 프레임 구조 다이어그램**

아래는 일반적인 UART 데이터 프레임(8-N-1 구성: 데이터 8비트, 패리티 없음, 정지 1비트)의 논리적 신호 흐름을 시각적으로 나타낸 것이다.

```
                  <-- 1 Bit Time -->
                  _________________   ___________   _______   _______   _______   _______   _______   _______   _______   _______   ___________
Idle (High) _____| | | | | | | | | | | | | | | | | | | | | |________ Idle (High)

| Start Bit | | Data Bit 0| | Data 1| | Data 2| | Data 3| | Data 4| | Data 5| | Data 6| | Data 7| | Stop Bit |
| (Low) | | (LSB) | | | | | | | | | | | | (MSB) | | (High) |
|_________________| |___________| |_______| |_______| |_______| |_______| |_______| |_______| |_______| |___________|
```

- **유휴 상태 (Idle State):** 데이터 전송이 없는 동안 통신 라인은 지속적으로 논리적 '1'(High) 상태를 유지한다. 4 이 상태는 수신기가 항상 통신 라인의 상태를 감시하여, 송신기의 고장이나 케이블 단선과 같은 물리적 연결 문제를 감지할 수 있게 해주는 중요한 역할을 한다. 5
- **시작 비트 (Start Bit):** 프레임의 시작을 알리는 신호. 유휴 상태(High)에서 논리적 '0'(Low)으로 1비트 시간만큼 전압이 떨어지는 것으로 표현된다. 수신기는 이 High에서 Low로의 하강 에지(falling edge)를 감지하고, 이를 기준으로 내부 타이머를 초기화하여 이후 들어올 데이터 비트들을 샘플링할 타이밍을 동기화한다. 1
- **데이터 비트 (Data Bits):** 실제 전송하고자 하는 정보를 담고 있는 부분이다. 시작 비트 바로 뒤에 이어진다. 4
- **패리티 비트 (Parity Bit):** 데이터 전송 오류를 검출하기 위한 선택적 비트. 데이터 비트와 정지 비트 사이에 위치한다. 4
- **정지 비트 (Stop Bit):** 프레임의 끝을 알리는 신호. 통신 라인을 다시 유휴 상태인 High로 복귀시킨다. 1


시작 비트는 비동기 통신에서 동기화의 기준점 역할을 하는 가장 중요한 요소이다. 20 통신 라인이 항상 High 상태로 유휴 상태를 유지하다가 Low로 떨어지는 명확한 신호 변화를 통해, 수신기는 새로운 데이터 프레임이 도착했음을 인지한다. 1 이 1비트 길이의 Low 신호는 수신 UART에게 "지금부터 약속된 속도로 데이터를 읽을 준비를 하라"는 신호를 보내는 것과 같다. 수신기는 이 시작 비트의 중앙 지점을 기준으로 삼아, 이후 데이터 비트들의 중앙 지점을 계산하여 샘플링을 수행한다. 33


데이터 비트는 사용자가 전송하고자 하는 실제 유효한 데이터(payload)를 담고 있다. 4 UART 설정에 따라 프레임당 5, 6, 7, 8, 또는 9개의 데이터 비트를 전송할 수 있다. 1 역사적으로 5비트는 Baudot 코드, 7비트는 ASCII 문자를 표현하는 데 사용되었으며, 현대에는 8비트(1 바이트) 데이터 전송이 가장 보편적으로 사용된다. 4 9비트 데이터는 주소 지정이나 특수 제어 목적으로 사용되는 멀티프로세서 통신 모드 등에서 활용될 수 있다. 27

데이터 비트의 전송 순서는 특별히 명시되지 않는 한, **최하위 비트(LSB, Least Significant Bit)가 가장 먼저** 전송되는 LSB-first 방식을 따른다. 4 예를 들어, 7비트 ASCII 코드로 대문자 'S'를 전송한다고 가정하자. 'S'의 ASCII 값은 82이고, 이진수로는 `1010010`이다. (자료 4의 1010011은 오기로 보이며, ASCII 'S'는 0x53이 아닌 0x52이다.) 이를 LSB-first 순서로 뒤집으면 `0100101`이 되며, 이 순서대로 통신 라인을 통해 전송된다. 수신기는 이 비트들을 받아 다시 원래 순서로 재조립한다.


패리티 비트는 데이터 전송 과정에서 발생할 수 있는 오류를 검출하기 위한 간단한 메커니즘으로, 선택적으로 사용된다. 1 이는 데이터 비트 전송이 끝난 후, 정지 비트가 시작되기 전에 삽입된다. 4 패리티는 '짝수(Even)'와 '홀수(Odd)' 두 가지 방식이 있으며, '사용 안 함(None)'으로 설정할 수도 있다. 1

- **짝수 패리티 (Even Parity):** 데이터 비트 내의 '1'의 개수를 센 후, 패리티 비트를 포함한 전체 '1'의 개수가 짝수가 되도록 패리티 비트 값을 '0' 또는 '1'로 결정한다. 예를 들어 데이터 비트 '1100101'('S'의 LSB-first)에는 '1'이 4개 있으므로 이미 짝수이다. 따라서 짝수 패리티 비트는 '0'이 된다. 4
- **홀수 패리티 (Odd Parity):** 전체 '1'의 개수가 홀수가 되도록 패리티 비트 값을 결정한다. 위 예시에서 홀수 패리티를 사용한다면, '1'의 개수를 홀수로 만들기 위해 패리티 비트는 '1'이 되어야 한다. 4

수신기는 데이터를 받은 후 동일한 방식으로 패리티를 계산하여 수신된 패리티 비트와 비교한다. 만약 두 값이 다르다면, 전송 중에 오류가 발생했음을 알 수 있다. 그러나 패리티 비트는 한계가 명확하다. 오직 홀수 개의 비트(1개, 3개 등)에서 오류가 발생했을 때만 감지할 수 있으며, 짝수 개의 비트(2개, 4개 등)에서 오류가 발생하면 '1'의 총 개수에 변화가 없으므로 오류를 감지하지 못한다. 4 또한 오류를 '검출'할 뿐, '수정'할 수는 없다. 이 때문에 높은 신뢰성이 요구되는 시스템에서는 CRC(Cyclic Redundancy Check)나 체크섬(Checksum)과 같은 더 강력한 오류 검출 방식을 상위 프로토콜 레벨에서 구현하는 것이 일반적이다. 27


정지 비트는 하나의 데이터 프레임이 끝났음을 명확히 알리는 역할을 한다. 1 이 비트는 통신 라인을 다시 논리적 '1'(High), 즉 유휴 상태로 되돌린다. 4 정지 비트의 길이는 1, 1.5, 또는 2비트 시간으로 설정할 수 있다. 1

- **1 정지 비트:** 가장 보편적으로 사용되는 설정이다.
- **1.5 또는 2 정지 비트:** 정지 비트의 길이를 늘리는 것은 수신기가 수신된 데이터를 처리하고 다음 프레임을 받을 준비를 할 수 있도록 추가적인 시간을 확보해주는 역할을 한다. 4 이는 특히 저속의 기계식 텔레타이프 시절에 필요했던 기능의 유산이며, 현대의 고속 전자 장치에서는 2개의 정지 비트를 사용하는 경우가 드물다. 4 하지만 매우 높은 보율로 통신할 때 수신측의 처리 지연을 보완하기 위해 사용될 수 있다. 35

정지 비트는 다음 프레임의 시작 비트와 명확히 구분되기 위해 반드시 High 상태여야 한다. 만약 수신기가 정지 비트가 있어야 할 위치에서 Low 신호를 감지하면, 이를 **프레이밍 오류(Framing Error)**로 간주한다.



보율(Baud Rate)은 통신 채널을 통해 1초 동안 전송되는 신호의 심볼(symbol) 또는 상태 변화의 횟수를 나타내는 단위이다. 7 UART와 같이 논리 '0'과 '1'의 두 가지 상태만을 사용하는 이진(binary) 디지털 통신에서는 하나의 심볼이 하나의 비트(bit) 정보를 전달하므로, 보율은 초당 비트 수(bps, bits per second)와 사실상 동일한 의미로 사용된다. 37 예를 들어, '9600 보율'은 초당 9600개의 비트를 전송하는 속도를 의미한다.

비동기 통신인 UART에서 보율은 클럭 신호의 역할을 대신하는 핵심적인 동기화 파라미터이다. 성공적인 통신을 위해서는 데이터를 보내는 송신기와 데이터를 받는 수신기가 반드시 사전에 동일한 보율로 설정되어 있어야 한다. 5 만약 양측의 보율이 다르면 수신기는 잘못된 시간 간격으로 신호를 샘플링하게 되어 데이터를 올바르게 복원할 수 없게 된다. 일반적으로 사용되는 표준 보율에는 4800, 9600, 19200, 38400, 57600, 115200 bps 등이 있다. 5


마이크로컨트롤러(MCU) 내부의 UART 모듈은 시스템의 주 클럭(fosc)을 기반으로 원하는 보율에 해당하는 타이밍 신호를 생성한다. 이는 내부의 보율 생성기(Baud Rate Generator)를 통해 이루어지며, 프로그래머는 특정 레지스터 값을 설정하여 이 생성기를 제어한다. 1

예를 들어, 널리 사용되는 AVR ATmega 시리즈 MCU에서는 `UBRRn` (USART Baud Rate Register)이라는 16비트 레지스터를 통해 보율을 설정한다. 7 일반적인 비동기 모드에서 보율을 계산하는 공식은 다음과 같다.
$$
\text{BAUD} = \frac{f_{osc}}{16 \times (UBRRn + 1)}
$$
이 공식을 `UBRRn`에 대해 정리하면, 원하는 보율을 얻기 위해 레지스터에 설정해야 할 값을 계산할 수 있다.
$$
UBRRn = \frac{f_{osc}}{16 \times \text{BAUD}} - 1
$$
여기서 공식에 포함된 숫자 '16'은 UART 수신기가 사용하는 **오버샘플링(oversampling)** 기법과 관련이 있다. 수신기는 단순히 비트 시간의 중앙에서 한 번만 샘플링하는 것이 아니라, 설정된 보율보다 16배(또는 8배) 빠른 내부 클럭을 사용하여 각 비트 구간을 여러 번 샘플링한다. 1 이를 통해 노이즈로 인한 순간적인 신호 왜곡의 영향을 줄이고, 가장 안정적인 지점(비트의 중앙)을 더 정확하게 찾아내어 데이터 수신의 신뢰도를 높인다. 28


송신기와 수신기가 동일한 보율(예: 9600 bps)로 설정되었다고 하더라도, 각 장치의 내부 클럭 소스(oscillator)가 가진 미세한 오차로 인해 실제 생성되는 보율은 약간의 차이를 보일 수 있다. 23 예를 들어 송신기는 9610 bps, 수신기는 9590 bps로 동작할 수 있다. 이 미세한 속도 차이는 프레임의 시작 부분에서는 문제가 되지 않지만, 비트가 진행됨에 따라 샘플링 시점의 오차를 누적시킨다. 23

프레임의 첫 번째 비트인 시작 비트의 하강 에지를 기준으로 동기화가 이루어진 후, 각 비트의 샘플링 시점은 이 기준점으로부터 계산된다. 만약 수신기의 클럭이 송신기보다 느리다면, 샘플링 시점은 점차 비트의 끝 쪽으로 밀려나게 된다. 반대로 수신기 클럭이 더 빠르면 샘플링 시점은 비트의 시작 쪽으로 당겨진다. 이 누적 오차가 비트 시간의 절반을 넘어서면, 수신기는 엉뚱한 비트 구간을 샘플링하게 되어 프레이밍 오류(Framing Error)를 발생시킨다. 마지막 데이터 비트와 정지 비트가 이 누적 오차에 가장 취약하다. 23

이러한 이유로, 안정적인 UART 통신을 위해서는 양측 장치 간의 보율 오차가 특정 허용 범위 이내여야 한다. 일반적으로 이 허용 오차는 **±2%** 이내로 알려져 있다. 42 일부 자료에서는 최대 ±10%까지 가능하다고 언급하기도 하지만 27, 이는 통신 환경이 매우 이상적일 때의 이론적인 값에 가까우며, 실제 시스템에서는 더 엄격한 마진을 두는 것이 안전하다. 높은 정밀도가 요구되는 고속 통신이나 신뢰성이 중요한 애플리케이션에서는, 온도나 공급 전압에 따라 주파수가 변동하기 쉬운 MCU 내부의 RC 오실레이터 대신, 안정성이 높은 외부 크리스탈 오실레이터(crystal oscillator)를 클럭 소스로 사용하는 것이 강력히 권장된다. 22


보율 오차가 통신 신뢰도에 미치는 영향을 정량적으로 분석할 수 있다. 송신기와 수신기 간의 보율 오차율을 E(%)라고 가정하자. 이상적인 샘플링 지점은 각 비트 시간의 50% 지점이다. 신호 왜곡이나 노이즈를 고려하여, 샘플링이 비트 중앙의 ±20% 구간(즉, 비트 시작 후 30% ~ 70% 구간) 내에서 이루어져야 안정적이라고 가정하자. 이는 샘플링 시점이 비트 경계로부터 최소 30%의 시간적 마진을 가져야 함을 의미한다.

8-N-1 (데이터 8비트, 패리티 없음, 정지 1비트) 구성의 경우, 시작 비트 이후 총 9개의 비트(데이터 8개 + 정지 1개)가 전송되는 동안 타이밍 오차가 누적된다. 마지막 비트(여기서는 정지 비트)가 샘플링될 때까지의 누적된 시간 오차는 9×(비트 시간)×(E/100) 이다. 이 누적 오차가 허용 마진인 0.3×(비트 시간) 보다 작아야 한다. 23

이를 부등식으로 표현하면 다음과 같다.
$$
(N_{data} + N_{parity} + N_{stop}) \times \frac{E}{100} < (\text{허용 마진})
$$
8-N-1 구성과 30% 마진을 적용하면,
$$
(8 + 0 + 1) \times \frac{E}{100} < 0.3 \implies 9 \times \frac{E}{100} < 0.3 \implies E < \frac{30}{9} \approx 3.33\%
$$
이 간략한 분석은 8-N-1 통신에서 양측의 보율 오차가 약 3.33% 이내여야 신뢰성 있는 통신이 가능함을 보여준다. 23 이 분석은 보율을 선택하는 행위가 단순히 통신 속도를 결정하는 것을 넘어, 통신 거리, 노이즈 환경, 시스템 클럭의 안정성까지 종합적으로 고려하여 '통신 신뢰도'와의 균형을 맞추는 과정임을 명확히 한다. 높은 보율은 각 비트의 시간 폭을 줄여 타이밍 오차에 더 민감하게 만들고, 장거리 전송 시 신호 감쇠를 심화시킨다. 32 따라서 개발자는 애플리케이션의 요구사항과 물리적 제약 조건 사이에서 최적의 균형점을 찾아 보율을 신중하게 선택해야 한다.


UART는 데이터 프레임의 구조와 타이밍을 정의하는 논리적인 프로토콜, 즉 데이터 링크 계층(OSI 모델의 2계층)에 해당한다. UART 자체는 신호를 전송하기 위한 물리적인 전압 레벨이나 전류 사양을 직접 정의하지 않는다. 2 실제 하드웨어 간의 통신을 위해서는 이 논리적 신호를 물리적 신호로 변환해주는 물리 계층(1계층) 표준이 필요하다. 대표적인 물리 계층 표준으로는 TTL, RS-232, RS-485가 있다.


각 표준은 전압 레벨, 노이즈 내성, 통신 거리, 네트워크 토폴로지 등에서 뚜렷한 차이를 보인다.

- **TTL (Transistor-Transistor Logic):** MCU나 기타 로직 IC의 핀에서 직접 출력되는 신호 레벨을 의미한다. 일반적으로 접지(0V)를 논리적 '0'(Low)으로, 공급 전압 Vcc(주로 3.3V 또는 5V)를 논리적 '1'(High)로 사용한다. 43 이는 별도의 드라이버 회로가 필요 없어 구현이 가장 간단하지만, 전압 스윙 폭이 작아 외부 노이즈에 매우 취약하다. 따라서 통신 거리는 동일한 PCB 내 또는 수 미터 이내의 매우 짧은 거리로 제한된다. 43

- **RS-232 (Recommended Standard 232):** 과거 PC의 직렬 포트(COM Port)에서 널리 사용되던 전통적인 인터페이스 표준이다. 14 RS-232는 노이즈 내성을 높이기 위해 훨씬 높은 전압 레벨을 사용한다. 논리 '1'(Mark)을 -3V에서 -15V 사이의 음의 전압으로, 논리 '0'(Space)을 +3V에서 +15V 사이의 양의 전압으로 정의한다. 44 TTL 신호와 비교했을 때 전압 레벨뿐만 아니라 논리적 극성(polarity)도 반대이다. 이 때문에 TTL 레벨의 UART 신호를 RS-232 장치와 연결하기 위해서는 반드시 MAX232와 같은 레벨 변환 및 반전(inverting) 드라이버 IC가 필요하다. 43 RS-232는 기본적으로 1:1 점대점(point-to-point) 통신만을 지원하며, 최대 통신 거리는 표준상 약 15미터(50피트)로 제한된다. 14

- **RS-485:** 산업 자동화, 공정 제어 등 열악한 환경에서 장거리, 다중 장치 통신을 위해 개발된 강력한 표준이다. 14 RS-485의 가장 큰 특징은 

  **차동 신호(Balanced Differential Signaling)** 방식을 사용하는 것이다. 3 이 방식은 하나의 신호를 전송하기 위해 두 개의 와이어(보통 A와 B로 표기)를 사용한다. 하나의 라인에 신호가 실리면 다른 라인에는 그와 정반대 극성의 신호가 실린다. 수신기는 두 라인 간의 전압 차이를 측정하여 논리 상태를 판단한다. 예를 들어, (A - B) > +200mV 이면 논리 '1', (A - B) < -200mV 이면 논리 '0'으로 인식한다. 48 이 방식은 두 라인에 동일하게 유입되는 공통 모드 노이즈(common-mode noise)를 효과적으로 상쇄시키므로 노이즈 내성이 매우 뛰어나다. 이 덕분에 RS-485는 최대 1200미터(4000피트)의 장거리 통신이 가능하며, 하나의 버스에 최대 32개의 장치(드라이버 및 리시버)를 연결하는 멀티드롭(Multi-drop) 네트워크를 구성할 수 있다. 48 통상적으로 2선식 반이중(Half-duplex) 통신을 기본으로 한다.

다음 표는 세 가지 물리 계층 표준의 핵심적인 특성을 비교 요약한 것이다.

| 특성                | TTL (Transistor-Transistor Logic) | RS-232                           | RS-485                      |
| ------------------- | --------------------------------- | -------------------------------- | --------------------------- |
| **신호 방식**       | 단일 종단 (Single-ended)          | 단일 종단 (Single-ended), 반전됨 | 차동 (Differential)         |
| **전압 레벨 ('1')** | Vcc (3.3V 또는 5V)                | -3V ~ -15V                       | A-B > +200mV                |
| **전압 레벨 ('0')** | 0V (GND)                          | +3V ~ +15V                       | A-B < -200mV                |
| **통신 방식**       | 전이중 (Full-duplex)              | 전이중 (Full-duplex)             | 반이중 (Half-duplex, 2선식) |
| **최대 통신 거리**  | ~수 미터                          | ~15 미터 (50 피트)               | ~1200 미터 (4000 피트)      |
| **최대 장치 수**    | 1:1 (점대점)                      | 1:1 (점대점)                     | 32 (멀티드롭, 표준)         |
| **노이즈 내성**     | 낮음                              | 중간                             | 매우 높음                   |
| **주요 용도**       | 칩 간 통신, 보드 내 연결          | 구형 PC 주변기기, 모뎀           | 산업 자동화, 센서 네트워크  |


수신 장치의 데이터 처리 속도가 송신 장치의 전송 속도를 따라가지 못할 경우, 수신 버퍼가 가득 차서 새로 들어오는 데이터가 유실되는 **오버런 오류(Overrun Error)**가 발생할 수 있다. 41 흐름 제어(Flow Control)는 이러한 데이터 유실을 방지하기 위해, 수신기가 자신의 상태를 송신기에게 알려 데이터 전송을 일시적으로 중지시키거나 재개하도록 요청하는 메커니즘이다. 51

- **하드웨어 흐름 제어 (RTS/CTS):** 가장 신뢰성 높은 방식으로, 데이터 라인 외에 RTS(Request To Send)와 CTS(Clear To Send)라는 두 개의 추가 제어선을 사용한다. 41 수신기는 자신의 수신 버퍼에 여유가 있을 때 RTS 신호를 활성화(Assert, 보통 Low 상태)하여 "데이터를 보내도 좋다"고 알린다. 송신기는 상대방의 CTS 핀(자신의 RTS와 교차 연결됨)이 활성화된 것을 확인하고 데이터를 전송한다. 수신 버퍼가 거의 차면 수신기는 RTS 신호를 비활성화(De-assert, High 상태)하고, 송신기는 이를 감지하여 전송을 즉시 중단한다.
- **소프트웨어 흐름 제어 (XON/XOFF):** 별도의 하드웨어 라인 없이, 기존의 데이터 라인(TX/RX)을 통해 특수한 제어 문자(ASCII 코드)를 전송하여 흐름을 제어한다. 41 수신 버퍼가 거의 차면 수신기는 송신기에게 XOFF(Transmit Off, 0x13) 문자를 보낸다. 송신기는 XOFF 문자를 수신하면 전송을 멈춘다. 이후 수신 버퍼에 다시 여유가 생기면, 수신기는 XON(Transmit On, 0x11) 문자를 보내 전송 재개를 요청한다. 구현이 간단하지만, 제어 문자가 실제 데이터와 혼동될 수 있는 가능성이 있고, 제어 문자가 전송되고 처리되는 데 시간이 걸려 반응이 느리다는 단점이 있다.


과거에는 16550A와 같은 독립적인 UART IC가 널리 사용되었다. 1 이러한 IC들은 내부에 FIFO(First-In, First-Out) 버퍼를 가지고 있어, CPU가 매 바이트 수신 시마다 즉각적으로 반응하지 않아도 되도록 하여 시스템의 부하를 크게 줄여주었다. 1

현대에 이르러서는 거의 모든 마이크로컨트롤러가 하나 이상의 UART(또는 기능을 확장한 USART) 모듈을 칩 내부에 기본적으로 포함하고 있다. 2 예를 들어, ATmega328(Arduino Uno)은 1개의 UART를, ATmega2560(Arduino Mega)은 4개의 UART를 내장하고 있다. 26 이러한 내장형 모듈은 MCU 내부 버스에 직접 연결되어 레지스터를 통해 쉽게 제어할 수 있다.

만약 MCU에 UART 포트가 없거나 필요한 수보다 부족할 경우, 일반 GPIO(General Purpose Input/Output) 핀을 소프트웨어로 직접 제어하여 UART 신호 타이밍을 흉내 내는 **소프트웨어 UART(Software UART 또는 Bit-banging)** 기법을 사용할 수 있다. 26 Arduino의 

`SoftwareSerial` 라이브러리가 대표적인 예이다. 이 방법은 추가 하드웨어 없이 유연하게 UART 포트를 확장할 수 있다는 장점이 있지만, 정밀한 타이밍을 소프트웨어로 유지해야 하므로 CPU에 상당한 부하를 주며, 하드웨어 UART만큼 정밀하거나 안정적이지는 않다. 26


임베디드 시스템 설계에서 통신 프로토콜을 선택하는 것은 시스템의 성능, 비용, 복잡도를 결정하는 중요한 과정이다. UART는 I2C, SPI와 함께 가장 널리 사용되는 칩 간(inter-chip) 통신 프로토콜이며, 각각의 고유한 특성으로 인해 서로 다른 응용 분야에 최적화되어 있다.


다른 프로토콜과 비교하기에 앞서, UART의 고유한 장단점을 명확히 할 필요가 있다.

**장점:**

- **하드웨어 단순성:** 송수신을 위해 단 두 개의 와이어(TX, RX)만 필요하므로 배선이 간단하다. 8
- **구현 용이성:** 별도의 클럭 신호가 없어 하드웨어 구성이 단순하고, 프로토콜 자체가 직관적이어서 소프트웨어 구현이 용이하다. 18
- **전이중 통신:** 송신과 수신 라인이 분리되어 있어 데이터의 동시 송수신(Full-duplex)이 가능하다. 40
- **범용성 및 호환성:** 오랜 역사만큼이나 수많은 장치에서 표준으로 지원되며, PC와의 연결이 용이하다. 4
- **물리 계층 유연성:** RS-232, RS-485 등 다양한 물리 계층 드라이버와 결합하여 통신 거리와 노이즈 내성을 필요에 따라 확장할 수 있다. 12

**단점:**

- **비동기 방식의 한계:** 클럭 신호가 없으므로 송수신 양측이 정밀하게 일치하는 보율을 사용해야 하며, 클럭 소스의 정확도가 통신 신뢰도에 직접적인 영향을 미친다. 22
- **제한된 토폴로지:** 기본적으로 1:1 점대점 통신을 위해 설계되었다. 다수의 장치를 연결하는 멀티드롭 네트워크를 구성하기 위해서는 RS-485와 같은 별도의 물리 계층이나 복잡한 소프트웨어적 처리가 필요하다. 53
- **주소 지정 부재:** 프로토콜 자체에 여러 장치를 구분하기 위한 주소 지정(Addressing) 메커니즘이 내장되어 있지 않다. 22
- **상대적으로 느린 속도:** 시작 비트와 정지 비트로 인한 오버헤드가 존재하며, 일반적으로 SPI나 고속 I2C 모드에 비해 최대 전송 속도가 낮다. 22
- **고정된 프레임 크기:** 데이터 프레임의 길이가 최대 9비트로 제한되어 있어, 긴 데이터 블록을 전송하기에는 비효율적이다. 8


I2C는 필립스(현 NXP)에서 개발한 동기식 직렬 프로토콜로, 주로 동일한 PCB 상의 칩들 간 통신에 사용된다. 15

- **통신선:** I2C는 데이터 라인(SDA, Serial Data)과 클럭 라인(SCL, Serial Clock) 단 두 개의 선만을 사용한다. 58 이 두 라인은 오픈 드레인(Open-drain) 방식으로 동작하므로, 외부 풀업(pull-up) 저항이 필수적이다.
- **클럭 및 동기화:** 마스터 장치가 SCL 라인을 통해 클럭을 제공하는 동기식 방식이다. 따라서 보율 설정이 필요 없으며 통신이 안정적이다. 17
- **토폴로지 및 주소 지정:** I2C의 가장 큰 특징은 **멀티-마스터, 멀티-슬레이브** 버스 구조이다. 고유한 7비트(또는 10비트) 주소를 사용하여 단 두 개의 라인으로 최대 127개의 슬레이브 장치를 연결하고 개별적으로 통신할 수 있다. 22 이는 UART가 1:1 통신에 국한되는 것과 대조적이다.
- **속도:** 표준 모드(100 kbps), 고속 모드(400 kbps), Fast-mode Plus(1 Mbps), 고속 모드(3.4 Mbps) 등 다양한 속도를 지원한다. 22
- **주요 용도:** 다수의 저속 센서(온도, 습도, 가속도), EEPROM, 실시간 클럭(RTC), I/O 확장기 등을 MCU에 연결하는 데 매우 효율적이다. 12


SPI는 모토로라에서 개발한 고속 동기식 직렬 프로토콜로, 전이중 통신을 지원한다. 56

- **통신선:** 일반적으로 4개의 선을 사용한다: SCLK(Serial Clock), MOSI(Master Out Slave In), MISO(Master In Slave Out), SS(Slave Select). 56
- **클럭 및 동기화:** 마스터가 SCLK 라인을 통해 클럭을 제공하는 동기식 방식이다.
- **토폴로지 및 주소 지정:** **단일 마스터, 다중 슬레이브** 구조가 일반적이다. 마스터는 통신하고자 하는 특정 슬레이브의 SS 라인을 활성화(보통 Low)하여 장치를 선택한다. 따라서 슬레이브 장치가 N개일 경우, 데이터 및 클럭 라인 외에 N개의 SS 라인이 추가로 필요하다. 42 이는 I2C처럼 2선으로 다수 장치를 제어하는 것에 비해 핀 소모가 많다는 단점이 있다.
- **속도:** 세 프로토콜 중 가장 빠르다. 구조가 단순하고 푸시-풀(push-pull) 드라이버를 사용하기 때문에 수십 Mbps 이상의 매우 높은 데이터 전송률을 쉽게 달성할 수 있다. 12
- **주요 용도:** 고속 데이터 스트리밍이 요구되는 플래시 메모리, SD 카드, 고해상도 ADC/DAC, 그래픽 LCD 디스플레이 등과의 인터페이스에 널리 사용된다. 12

다음 표는 UART, I2C, SPI의 핵심적인 특성을 비교하여 각 프로토콜의 선택 기준을 명확히 제시한다.

| 특성            | UART                                | I2C                             | SPI                                 |
| --------------- | ----------------------------------- | ------------------------------- | ----------------------------------- |
| **통신선 수**   | 2선 (TX, RX)                        | 2선 (SDA, SCL)                  | 4선 이상 (MOSI, MISO, SCLK, n x SS) |
| **클럭**        | 없음 (비동기)                       | 마스터 제공 (동기)              | 마스터 제공 (동기)                  |
| **통신 방식**   | 점대점 (Point-to-Point)             | 멀티-마스터, 멀티-슬레이브 버스 | 단일 마스터, 다중 슬레이브 버스     |
| **속도**        | 낮음 ~ 중간 (최대 ~수 Mbps)         | 중간 (최대 3.4 Mbps)            | 높음 (최대 수십 Mbps 이상)          |
| **주소 지정**   | 없음 (소프트웨어 구현 필요)         | 하드웨어 주소 지정 (7/10비트)   | 슬레이브 선택(SS) 라인으로 지정     |
| **데이터 방향** | 전이중 (Full-Duplex)                | 반이중 (Half-Duplex)            | 전이중 (Full-Duplex)                |
| **장점**        | 구현 단순, 저비용, 장거리 확장성    | 최소 핀으로 다수 장치 연결      | 최고 속도, 간단한 프로토콜          |
| **단점**        | 속도 제한, 1:1 통신, 보율 동기화    | 속도 제한, 풀업 저항 필요       | 슬레이브 증가 시 핀 소모 많음       |
| **주요 응용**   | 디버그 콘솔, GPS, 블루투스, PC 통신 | 저속 센서, EEPROM, RTC          | 플래시 메모리, SD카드, ADC, LCD     |



UART는 그 단순성과 범용성 덕분에 다양한 임베디드 애플리케이션에서 핵심적인 역할을 수행한다.


임베디드 시스템 개발 과정에서 가장 기본적인 UART의 활용처는 디버그 콘솔이다. 9 임베디드 리눅스 보드나 MCU 펌웨어는 부팅 과정, 시스템 상태, 오류 메시지 등의 로그를 UART를 통해 출력한다. 10 개발자는 PC에 USB-to-UART 변환기를 연결하고, PuTTY, Tera Term, minicom과 같은 터미널 에뮬레이션 프로그램을 사용하여 이 로그를 실시간으로 모니터링할 수 있다. 61 또한, 터미널을 통해 명령어를 입력하여 시스템을 제어하거나 디버깅 작업을 수행할 수도 있다. 이는 그래픽 인터페이스나 네트워크가 동작하지 않는 시스템의 최하단 레벨에서도 접근할 수 있는 강력하고 필수적인 디버깅 채널이다. 62


GPS 모듈(예: u-blox NEO-6M)은 위성으로부터 수신한 위치, 속도, 시간 등의 정보를 NMEA (National Marine Electronics Association)라는 표준 프로토콜 형식의 텍스트 문장으로 변환하여 UART 인터페이스를 통해 주기적으로 출력한다. 63 Arduino나 Raspberry Pi와 같은 개발 보드는 이 UART 신호를 수신하여 NMEA 문장을 파싱(parsing)하고, 위도, 경도 등의 필요한 정보를 추출하여 활용한다. 연결은 간단하다. GPS 모듈의 TX 핀을 MCU의 RX 핀에, VCC와 GND를 각각 전원과 접지에 연결하면 된다. 63 수신된 데이터는 `$GPRMC`, `$GPGGA`와 같은 식별자로 시작하는 문장들이며, 라이브러리(예: TinyGPS++)를 사용하면 이를 쉽게 해석할 수 있다. 63


HC-06과 같은 저가형 블루투스 모듈은 스마트폰이나 PC와의 무선 시리얼 통신을 가능하게 해주는 브리지 역할을 한다. 67 이 모듈은 내부적으로 블루투스 통신 스택을 처리하고, 최종적으로 수신된 데이터를 UART 인터페이스를 통해 MCU로 전달한다. 67 MCU는 블루투스 모듈을 일반적인 UART 장치처럼 다룰 수 있다. 예를 들어, 안드로이드 앱에서 특정 문자('1' 또는 '0')를 전송하면, HC-06이 이를 수신하여 UART를 통해 Arduino로 전달하고, Arduino는 이 문자에 따라 LED를 켜거나 끄는 동작을 수행할 수 있다. 67 이는 복잡한 무선 프로토콜을 직접 다루지 않고도 간단하게 무선 제어 시스템을 구현할 수 있게 해준다.


공장 자동화나 빌딩 관리 시스템과 같이 노이즈가 많고 통신 거리가 긴 환경에서는 TTL 레벨의 UART를 직접 사용할 수 없다. 이때 UART 신호를 RS-485 신호로 변환해주는 컨버터 모듈(주로 MAX485 IC 기반)이 사용된다. 70 MCU의 UART 포트를 이 컨버터에 연결하면, 차동 신호 방식의 강력한 노이즈 내성과 최대 1.2km의 장거리 통신 능력을 확보할 수 있다. 49 또한, RS-485의 멀티드롭 특성을 이용하여 하나의 마스터 MCU가 여러 개의 슬레이브 장치(센서, 액추에이터 등)를 제어하는 산업용 네트워크를 효율적으로 구축할 수 있다. 73


UART 통신은 단순하지만, 설정이 잘못되거나 물리적 환경이 열악할 경우 다양한 오류에 직면할 수 있다. 오류의 원인을 정확히 이해하는 것은 신뢰성 있는 시스템을 구축하는 데 필수적이다.


- **원인 및 증상:** 프레이밍 오류는 수신기가 정지 비트가 있어야 할 시점에서 논리적 '0'(Low)을 감지했을 때 발생한다. 74 이는 수신기가 데이터 프레임의 끝을 제대로 인식하지 못했음을 의미한다. 가장 흔한 원인은 송신기와 수신기 간의 

  **보율 불일치**이다. 74 또한, 불안정한 클럭 소스로 인한 타이밍 드리프트, 긴 케이블이나 노이즈로 인한 신호 왜곡도 원인이 될 수 있다. 76

- **해결 방안:**

  1. **설정 확인:** 양측 장치의 보율, 데이터 비트 수, 패리티 설정, 정지 비트 수가 완벽하게 일치하는지 최우선으로 확인한다. 74
  2. **클럭 소스 검토:** MCU의 시스템 클럭 주파수 설정이 정확한지, 보율 계산에 사용된 분주비가 올바른지 검토한다. 가능하다면 안정적인 외부 크리스탈을 사용한다. 75
  3. **물리적 연결 점검:** 공통 접지(Common Ground)가 제대로 연결되었는지 확인한다. 케이블이 너무 길거나 노이즈가 심한 환경이라면, 쉴드 케이블을 사용하거나 RS-485와 같은 더 강력한 물리 계층으로 변경하는 것을 고려한다.
  4. **신호 분석:** 오실로스코프나 로직 분석기를 사용하여 TX 라인의 신호 파형을 직접 관찰하고, 비트 폭을 측정하여 실제 보율이 설정값과 일치하는지 확인한다. 75


- **원인 및 증상:** 오버런 오류는 수신 UART의 내부 수신 버퍼(Receive Buffer Register, RDR)에 있는 데이터를 CPU가 읽어 가기 전에, 다음 데이터가 시프트 레지스터에서 RDR로 이동하려고 할 때 발생한다. 즉, 수신 속도를 처리 속도가 따라가지 못해 데이터가 덮어씌워지는 상황이다. 7 이는 주로 CPU가 다른 높은 우선순위의 인터럽트를 처리하느라 UART 수신 인터럽트 서비스 루틴(ISR)을 제때 실행하지 못할 때 발생한다. 52
- **해결 방안:**
  1. **ISR 최적화:** UART 수신 ISR 내에서는 시간이 오래 걸리는 작업(예: `printf`, 복잡한 연산, 긴 지연)을 절대 수행해서는 안 된다. ISR의 역할은 오직 수신 레지스터에서 데이터를 읽어와 링 버퍼(Ring Buffer)와 같은 임시 저장 공간에 빠르게 저장하는 것으로 최소화해야 한다. 실제 데이터 처리는 메인 루프(main loop)에서 수행하도록 구조를 변경한다. 52
  2. **FIFO 및 DMA 활용:** 최신 MCU의 UART는 하드웨어 FIFO(First-In, First-Out) 버퍼를 내장하고 있다. 이 FIFO를 활성화하면 여러 바이트가 수신될 때까지 CPU의 개입이 필요 없어 ISR 호출 빈도를 줄일 수 있다. 28 더 나아가, DMA(Direct Memory Access)를 사용하면 CPU를 전혀 거치지 않고 UART 수신 데이터를 메모리의 지정된 버퍼로 직접 전송할 수 있어, 오버런 문제를 근본적으로 해결할 수 있다. 28
  3. **오류 복구:** 오버런 오류가 발생하면, 관련 오류 플래그를 소프트웨어적으로 클리어하고 수신 버퍼를 비우는 복구 로직을 구현하여 통신이 중단되지 않도록 해야 한다. 52


- **원인 및 증상:** 수신된 데이터 프레임의 '1'의 개수를 기반으로 계산한 패리티 값과, 프레임에 포함되어 온 패리티 비트의 값이 일치하지 않을 때 발생한다. 7 이는 전송 경로상의 노이즈로 인해 데이터 비트 중 하나가 0에서 1로, 또는 1에서 0으로 변경되었음을 시사한다. 22
- **해결 방안:**
  1. **노이즈 원인 제거:** 물리적 연결을 점검하고, 접지를 강화하거나 쉴드 케이블을 사용하는 등 노이즈 유입 경로를 차단한다.
  2. **강력한 오류 검출 도입:** 패리티는 매우 기본적인 오류 검출 방법이다. 신뢰성이 중요하다면, 상위 프로토콜 레벨에서 CRC나 체크섬과 같은 더 강력한 오류 검출 코드를 데이터 패킷에 추가하여 사용해야 한다. 수신측은 CRC를 검사하여 데이터의 무결성을 확인하고, 오류가 발견되면 송신측에 데이터 재전송을 요청하는 메커니즘을 구현하는 것이 바람직하다. 34

이러한 UART 오류들은 단순히 버그가 아니라, 프로토콜 자체의 근본적인 설계 특성과 한계를 드러내는 지표이다. 프레이밍 오류는 '클럭 부재'라는 비동기 통신의 본질에서, 오버런 오류는 '내장된 흐름 제어 부재'에서, 패리티 오류는 '단순한 오류 검출 방식'의 한계에서 비롯된다. 따라서 이러한 오류들을 디버깅하는 과정은 개발자에게 비동기 통신의 이론적 배경과 실제 구현 사이의 관계를 깊이 이해하게 만드는 중요한 학습 경험을 제공한다.



범용 비동기화 송수신기(UART)는 수십 년에 걸쳐 검증된, 가장 기본적이면서도 신뢰성 높은 직렬 통신 프로토콜이다. 그 핵심은 별도의 클럭 신호 없이, 단 두 개의 데이터 라인(TX, RX)만으로 안정적인 전이중(Full-duplex) 통신을 구현하는 데 있다. 이는 시작 비트와 정지 비트를 이용한 프레임 기반의 재동기화 메커니즘과, 송수신 양단 간에 사전에 약속된 보율(Baud Rate)이라는 두 가지 단순한 규칙을 통해 달성된다.

UART의 가장 큰 미덕은 '범용성(Universal)'과 '단순성'에 있다. 1 데이터 프레임의 구조(데이터 비트, 패리티, 정지 비트 수)와 통신 속도(보율)를 유연하게 설정할 수 있으며, TTL, RS-232, RS-485 등 다양한 물리 계층 표준과 결합하여 짧은 칩 간 통신부터 수 킬로미터에 이르는 산업용 네트워크까지 광범위한 요구사항에 대응할 수 있다. 이러한 특성 덕분에 UART는 복잡한 하드웨어나 소프트웨어 스택 없이도 최소한의 리소스로 즉각적인 통신 채널을 구축할 수 있는, 가장 접근하기 쉬운 통신 솔루션으로 자리매김하였다.


데이터 처리량이 폭발적으로 증가하고 모든 것이 연결되는 사물 인터넷(IoT)과 고성능 임베디드 컴퓨팅 시대에도 UART의 가치는 퇴색되지 않을 것이다. 오히려 그 역할은 더욱 명확해질 전망이다. USB나 이더넷과 같은 고속 인터페이스가 메인 데이터 경로를 담당하는 동안, UART는 시스템의 근간을 지탱하는 필수적인 보조 인터페이스로서의 역할을 더욱 공고히 할 것이다.

구체적으로, UART는 다음과 같은 영역에서 지속적으로 중요한 역할을 수행할 것이다.

1. **시스템 저수준 제어 및 디버깅:** 복잡한 운영체제나 드라이버가 로드되기 전, 부트로더 단계에서의 시스템 상태를 모니터링하고 제어하는 유일한 창구로서의 역할은 대체 불가능하다.
2. **안전 필수 시스템(Safety-Critical System):** 주 통신 시스템에 장애가 발생했을 때, 독립적이고 신뢰성 높은 비상 통신 채널 또는 로깅 채널로서 활용될 수 있다.
3. **저전력 및 저비용 장치:** 수많은 저가형 센서, 액추에이터, 그리고 배터리로 동작하는 저전력 장치들에게 UART는 여전히 가장 효율적이고 경제적인 통신 수단이다.

결론적으로, UART는 비록 최고 속도를 자랑하지는 않지만, 그 단순성, 신뢰성, 저비용이라는 본질적인 강점을 바탕으로 미래의 복잡한 임베디드 시스템 속에서도 자신의 고유한 생태계를 유지하며 발전해 나갈 것이다. UART의 작동 원리를 깊이 이해하는 것은 다른 모든 직렬 통신 프로토콜을 학습하기 위한 견고한 기초를 제공하며, 이는 시대를 막론하고 모든 임베디드 시스템 엔지니어가 갖추어야 할 기본적인 소양으로 남을 것이다. 10


1. UART - 위키백과, 우리 모두의 백과사전, 8월 8, 2025에 액세스, https://ko.wikipedia.org/wiki/UART
2. UART란 무엇인가? - RAKU, 8월 8, 2025에 액세스, https://rakuraku.tistory.com/47
3. 비동기 시리얼(직렬) 통신의 종류와 개념 - 기록하는 개발자, 8월 8, 2025에 액세스, https://powerdeng.tistory.com/15
4. Understanding UART - Rohde & Schwarz, 8월 8, 2025에 액세스, https://www.rohde-schwarz.com/kr/products/test-and-measurement/essentials-test-equipment/digital-oscilloscopes/understanding-uart_254524.html
5. Understanding UART | Rohde & Schwarz, 8월 8, 2025에 액세스, https://www.rohde-schwarz.com/cz/products/test-and-measurement/essentials-test-equipment/digital-oscilloscopes/understanding-uart_254524.html
6. What are the pros and cons to using USART for debugging embedded systems? [closed], 8월 8, 2025에 액세스, https://electronics.stackexchange.com/questions/292423/what-are-the-pros-and-cons-to-using-usart-for-debugging-embedded-systems
7. [시리얼 통신] UART, 8월 8, 2025에 액세스, [https://sroongzi.tistory.com/entry/%EC%8B%9C%EB%A6%AC%EC%96%BC-%ED%86%B5%EC%8B%A0-UART](https://sroongzi.tistory.com/entry/시리얼-통신-UART)
8. What is the UART communication protocol - Soldered Electronics, 8월 8, 2025에 액세스, https://soldered.com/learn/what-is-the-uart-communication-protocol/
9. UART ep01: UART란 무엇인가? (What is UART) - YouTube, 8월 8, 2025에 액세스, https://www.youtube.com/watch?v=kCvFmXLENds
10. UART in embedded system: what it is and how to use - Proculus Technologies, 8월 8, 2025에 액세스, https://www.proculustech.com/the-essential-role-of-uart-protocols-in-embedded-systems
11. UART Protocols and Applications in Embedded Systems, 8월 8, 2025에 액세스, https://www.embedded.com/understanding-the-uart/
12. UART vs SPI vs I2C : r/embedded - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/embedded/comments/1816dor/uart_vs_spi_vs_i2c/
13. UART 통신 - Basic Electronic Circuit - 티스토리, 8월 8, 2025에 액세스, https://e-circuit.tistory.com/16
14. 직렬 통신 - [정보통신기술용어해설], 8월 8, 2025에 액세스, http://www.ktword.co.kr/test/view/view.php?no=3575
15. 통신 방식에 따른 분류 (serial/parallel, 동기/비동기...) - ibin's study, 8월 8, 2025에 액세스, https://ibin-study.github.io/posts/Communication_type/
16. 직렬 통신 - 위키백과, 우리 모두의 백과사전, 8월 8, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EC%A7%81%EB%A0%AC_%ED%86%B5%EC%8B%A0](https://ko.wikipedia.org/wiki/직렬_통신)
17. [통신기초] 시리얼 통신 (Serial Communication) - LowLevel Lab - 티스토리, 8월 8, 2025에 액세스, https://machinejw.tistory.com/15
18. UART 통신 이론, 8월 8, 2025에 액세스, https://how-to-make-a-quadcopter.tistory.com/1
19. 통신 방식의 비교 - 기록하는 개발자, 8월 8, 2025에 액세스, https://powerdeng.tistory.com/11
20. UART 란? - velog, 8월 8, 2025에 액세스, [https://velog.io/@lutein/UART-%EB%9E%80](https://velog.io/@lutein/UART-란)
21. UART 통신 이론 - 공린이 탈출기 - 티스토리, 8월 8, 2025에 액세스, https://shek.tistory.com/41
22. I2C vs SPI vs UART: A Comprehensive Comparison - Wevolver, 8월 8, 2025에 액세스, https://www.wevolver.com/article/i2c-vs-uart
23. UART Baud Rate: How Accurate Does It Need to Be? - Technical Articles, 8월 8, 2025에 액세스, https://www.allaboutcircuits.com/technical-articles/the-uart-baud-rate-clock-how-accurate-does-it-need-to-be/
24. Universal asynchronous receiver-transmitter - Wikipedia, 8월 8, 2025에 액세스, https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter
25. ko.wikipedia.org, 8월 8, 2025에 액세스, [https://ko.wikipedia.org/wiki/UART#:~:text=UART%EB%8A%94%20%EC%9D%BC%EB%B0%98%EC%A0%81%EC%9C%BC%EB%A1%9C%20%EC%BB%B4%ED%93%A8%ED%84%B0,%EC%88%98%20%EC%9E%88%EB%8F%84%EB%A1%9D%20%EC%95%BD%EC%86%8D%EB%90%98%EC%96%B4%20%EC%9E%88%EB%8B%A4.](https://ko.wikipedia.org/wiki/UART#:~:text=UART는 일반적으로 컴퓨터,수 있도록 약속되어 있다.)
26. UARTs - Serial Communication - SparkFun Learn, 8월 8, 2025에 액세스, https://learn.sparkfun.com/tutorials/serial-communication/uarts
27. UART: A Hardware Communication Protocol Understanding Universal Asynchronous Receiver/Transmitter | Analog Devices, 8월 8, 2025에 액세스, https://www.analog.com/en/resources/analog-dialogue/articles/uart-a-hardware-communication-protocol.html
28. Universal Asynchronous Receiver Transmitter (UART) - C29x Academy, 8월 8, 2025에 액세스, https://dev.ti.com/tirex/explore/node?node=A__AI6PtSRZGz-plQ.ZOvtVpQ__C29X-ACADEMY__KgQzuuf__LATEST
29. Uart Full Duplex vs Half Duplex - Electrical Engineering Stack Exchange, 8월 8, 2025에 액세스, https://electronics.stackexchange.com/questions/228135/uart-full-duplex-vs-half-duplex
30. Concepts of Serial Communication UART Receiver Block Diagram, 8월 8, 2025에 액세스, https://www.ele.uri.edu/~qyang/ele547/LectureSerialComm.pdf
31. en.wikipedia.org, 8월 8, 2025에 액세스, [https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter#:~:text=A%20UART%20frame%20consists%20of,set%20employed%2C%20represent%20the%20character.](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter#:~:text=A UART frame consists of,set employed%2C represent the character.)
32. Understanding the frame format of a UART signal - Arduino Forum, 8월 8, 2025에 액세스, https://forum.arduino.cc/t/understanding-the-frame-format-of-a-uart-signal/1299422
33. UART Baud Rate selection. : r/FPGA - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/FPGA/comments/sx91bg/uart_baud_rate_selection/
34. What are your best practices to make UART comms reliable and robust? - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/embedded/comments/1d45tel/what_are_your_best_practices_to_make_uart_comms/
35. UART Communication - Wiring, Signal, Data Frame, and Speed, 8월 8, 2025에 액세스, https://datacapturecontrol.com/articles/data-communication/wired/uart/intro
36. Understanding Baud Rate: Why is it Important? - Wevolver, 8월 8, 2025에 액세스, https://www.wevolver.com/article/baud-rates
37. UART Communication Explained – Digilent Blog, 8월 8, 2025에 액세스, https://digilent.com/blog/uart-explained/
38. digilent.com, 8월 8, 2025에 액세스, [https://digilent.com/blog/uart-explained/#:~:text=Another%20important%20parameter%20for%20UART,within%2010%25%20of%20each%20other.](https://digilent.com/blog/uart-explained/#:~:text=Another important parameter for UART,within 10% of each other.)
39. UART Baud Rate and Output Rate - SBG Support Center, 8월 8, 2025에 액세스, https://support.sbg-systems.com/sc/kb/latest/technology-insights/uart-baud-rate-and-output-rate
40. UART 통신 프로토콜 (개념), 8월 8, 2025에 액세스, https://dandelion07.tistory.com/28
41. [UART 강좌] 05강 UART의 특징 - YouTube, 8월 8, 2025에 액세스, https://www.youtube.com/watch?v=LMZZC3AW3do
42. UART, SPI, & I2C - EEVblog, 8월 8, 2025에 액세스, https://www.eevblog.com/forum/news/uart-spi-i2c/
43. What's difference between RS232 and UART in context of programming ? | All About Circuits, 8월 8, 2025에 액세스, https://forum.allaboutcircuits.com/threads/whats-difference-between-rs232-and-uart-in-context-of-programming.168639/
44. What are the Differences Between TTL and True RS232 - Honeywell Support Portal, 8월 8, 2025에 액세스, https://sps-support.honeywell.com/s/article/What-are-the-differences-between-TTL-and-a-True-RS232-serial-interface
45. What is the difference between RS232, RS485 and TTL - NiceRF, 8월 8, 2025에 액세스, https://www.nicerf.com/news/rs232-rs485-and-ttl.html
46. Is RS-232 UART??? : r/arduino - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/arduino/comments/1dq1b60/is_rs232_uart/
47. What's the Difference Between RS-232 and RS-485? - Technical Articles - All About Circuits, 8월 8, 2025에 액세스, https://www.allaboutcircuits.com/technical-articles/whats-the-difference-between-rs-232-and-rs-485/
48. Key Differences between RS-485 and RS-232 Serial Protocols - Circuit Digest, 8월 8, 2025에 액세스, https://circuitdigest.com/article/key-differences-between-rs-485-and-rs-232-serial-protocols
49. What Is the difference between RS232 and RS485? - ExpertDAQ, 8월 8, 2025에 액세스, https://expertdaq.com/en/faq/difference-between-rs232-and-rsS485/
50. RS232 and RS485 difference | Serial Protocols Compared - Virtual Serial Port Driver, 8월 8, 2025에 액세스, https://www.virtual-serial-port.org/article/what-is-serial-port/rs232-vs-rs485.html
51. AN0059.0: UART Flow Control - Silicon Labs, 8월 8, 2025에 액세스, https://www.silabs.com/documents/public/application-notes/an0059.0-uart-flow-control.pdf
52. how to handle HAL_UART_ERROR_ORE? - STMicroelectronics Community, 8월 8, 2025에 액세스, https://community.st.com/t5/stm32cubemx-mcus/how-to-handle-hal-uart-error-ore/td-p/481259
53. UART - Data Engineer - 티스토리, 8월 8, 2025에 액세스, https://aconitegreen.tistory.com/m/10
54. [Hardware] UART 통신과 물리적 연결, 8월 8, 2025에 액세스, https://sunrinjuntae.tistory.com/117
55. UART(Universal asynchronous receiver/transmitter) - Mr.HB이야기 - 티스토리, 8월 8, 2025에 액세스, https://gudgud.tistory.com/12
56. UART vs. SPI vs. I2C: Routing & Layout Guidelines | Blogs - Altium Resources, 8월 8, 2025에 액세스, https://resources.altium.com/p/i2c-vs-spi-vs-uart-how-layout-these-common-buses
57. Understanding and Selecting in 2024: I2C, SPI, UART Explained - Parlez-vous Tech, 8월 8, 2025에 액세스, https://www.parlezvoustech.com/en/comparaison-protocoles-communication-i2c-spi-uart/
58. Understanding Sensor Interfaces - UART, I2C, SPI and CAN - Blues Developers, 8월 8, 2025에 액세스, https://dev.blues.io/blog/blues-university-understanding-sensor-interfaces-uart-i2c-spi-can/
59. Good resources for understanding the differences between UART, SPI I2C and when you would use each? - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/embedded/comments/11a8r05/good_resources_for_understanding_the_differences/
60. I2C vs SPI vs UART – Introduction and Comparison of their Similarities and Differences, 8월 8, 2025에 액세스, https://www.totalphase.com/blog/2021/12/i2c-vs-spi-vs-uart-introduction-and-comparison-similarities-differences/
61. UART Terminal: A Web-Based Serial Communication Tool Built with React - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/embedded/comments/1f77bf2/uart_terminal_a_webbased_serial_communication/
62. How to use UART1 as a console ? - Jetson TX2 - NVIDIA Developer Forums, 8월 8, 2025에 액세스, https://forums.developer.nvidia.com/t/how-to-use-uart1-as-a-console/56682
63. Serial Communication between Arduino and GPS Module - Lonely Binary, 8월 8, 2025에 액세스, https://lonelybinary.com/en-uk/blogs/learn/how-gps-works
64. UART GPS NEO-6M - Waveshare Wiki, 8월 8, 2025에 액세스, https://www.waveshare.com/wiki/UART_GPS_NEO-6M
65. Using UART instead of USB | Adafruit Ultimate GPS with gpsd, 8월 8, 2025에 액세스, https://learn.adafruit.com/adafruit-ultimate-gps-on-the-raspberry-pi/using-uart-instead-of-usb
66. Using UART to read GPS module - Studs, 8월 8, 2025에 액세스, https://studs.io/tut/UartGps.html
67. Tutorial - Using HC06 Bluetooth to Serial Wireless UART Adaptors With Arduino, 8월 8, 2025에 액세스, https://www.instructables.com/Tutorial-Using-HC06-Bluetooth-to-Serial-Wireless-U-1/
68. Learn Coding with Arduino IDE – HC-06 Bluetooth Module - osoyoo.com, 8월 8, 2025에 액세스, https://osoyoo.com/2017/10/25/arduino-lesson-hc-06-bluetooth-module/
69. How to set up Itead HC-06 to Arduino, 8월 8, 2025에 액세스, https://forum.arduino.cc/t/how-to-set-up-itead-hc-06-to-arduino/392212
70. UART TTL To RS485 Two-Way Converter - iFuture Technology, 8월 8, 2025에 액세스, https://ifuturetech.org/product/uart-ttl-to-rs485-two-way-converter/
71. How to Use UART TTL to RS485 Two-way Converter: Examples, Pinouts, and Specs, 8월 8, 2025에 액세스, https://docs.cirkitdesigner.com/component/5c2e4fe3-06e2-4526-bc80-4116870ee74c/uart-ttl-to-rs485-two-way-converter
72. UART TTL to RS485 Two-way Converter - Elecrow, 8월 8, 2025에 액세스, https://www.elecrow.com/uart-ttl-to-rs485-twoway-converter-p-1545.html
73. UART to RS-485 converter - RobotShop, 8월 8, 2025에 액세스, https://www.robotshop.com/products/sfe-uart-to-rs-485-converter
74. UART Framing Error Between Two STM32 : r/embedded - Reddit, 8월 8, 2025에 액세스, https://www.reddit.com/r/embedded/comments/1kxw8ob/uart_framing_error_between_two_stm32/
75. UART Framing Error on STM32F1 - STMicroelectronics Community, 8월 8, 2025에 액세스, https://community.st.com/t5/stm32-mcus-products/uart-framing-error-on-stm32f1/td-p/434711
76. Intermittent Uart Frame Errors - Nordic DevZone, 8월 8, 2025에 액세스, https://devzone.nordicsemi.com/f/nordic-q-a/118535/intermittent-uart-frame-errors
77. Getting Out Of A UART Framing Error - Raspberry Pi Forums, 8월 8, 2025에 액세스, https://forums.raspberrypi.com/viewtopic.php?t=380931
78. MSP430 UART overrun Error - MSP low-power microcontroller forum - TI E2E, 8월 8, 2025에 액세스, https://e2e.ti.com/support/microcontrollers/msp430/f/msp-low-power-microcontroller-forum/546168/msp430-uart-overrun-error
79. HAL UART Overrun causes it to never receive data again in normal and interrupt modes, 8월 8, 2025에 액세스, https://community.st.com/t5/stm32cubeide-mcus/hal-uart-overrun-causes-it-to-never-receive-data-again-in-normal/td-p/223838
80. 실무자가 들려주는 UART/ USART 통신 - 앤카이브, 8월 8, 2025에 액세스, https://anchive.tistory.com/7
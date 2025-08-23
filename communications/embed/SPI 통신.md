# SPI 통신



SPI(Serial Peripheral Interface)는 직렬 주변기기 인터페이스의 약어로, 마이크로컨트롤러(Master)와 센서, 메모리, 디스플레이와 같은 다양한 주변 장치(Slave) 간의 데이터 통신을 위해 설계된 동기식 직렬 통신 프로토콜이다.1 SPI는 주로 근거리, 특히 단일 회로 기판(on-board) 내의 칩 간 통신에 최적화되어 있다.4 이는 RS-232C나 CAN 버스와 같이 장거리 통신을 목적으로 하는 프로토콜들과 근본적으로 다른 설계 철학을 반영하며, 칩 간 통신에서 요구되는 높은 속도와 낮은 복잡성을 달성하는 데 주안점을 두었음을 시사한다. 이러한 개발 배경은 SPI 통신의 주요 장점인 고속 전송과 단순한 인터페이스를 가능케 하는 동시에, 거리 제약 및 노이즈에 대한 취약성이라는 단점의 근본적인 원인이 된다.


SPI 통신은 그 구조와 동작 방식에서 몇 가지 핵심적인 특징을 갖는다. 첫째, **마스터-슬레이브(Master-Slave) 구조**로 동작한다.1 마스터는 통신을 시작하고 제어하는 주체로, 보통 MCU(Microcontroller Unit)가 그 역할을 수행한다.3 슬레이브는 마스터의 제어를 받는 주변 장치이며, SPI 통신은 한 개의 마스터와 한 개 이상의 슬레이브로 구성된다.1

둘째, **동기식(Synchronous) 통신** 방식이다.2 마스터는 통신에 필요한 클럭 신호를 직접 생성하고(`SCLK`), 이 클럭 신호를 기준으로 데이터 송수신 타이밍을 동기화한다.2 이 방식은 송수신 장치 간에 미리 설정된 통신 속도(Baud Rate)를 맞춰야 하는 비동기식(Asynchronous) 통신(예: UART)과는 달리, 별도의 타이밍 동기화 과정이 필요 없어 통신 속도 오차 문제를 회피할 수 있다.11 이로 인해 SPI는 이론적으로 `최대 클럭에 대한 제한이 없어` 고속 통신에 매우 유리하다.4 하지만 실제 통신 속도는 통신 라인의 신호 무결성, 마스터의 출력 용량, 그리고 슬레이브 장치의 처리 능력에 의해 결정된다.3

셋째, **전이중(Full-duplex) 통신**을 지원한다.2 이는 마스터와 슬레이브가 동시에 데이터를 송수신할 수 있음을 의미하며, SPI의 고속 통신을 가능케 하는 핵심적인 특징이다.

2. 하드웨어 구성 및 필수 신호


SPI 통신은 최소 네 개의 필수 신호선으로 구성된 4-와이어 버스를 사용한다.2 이 네 개의 신호선은 각각의 고유한 기능을 수행하며, 이 구조 덕분에 SPI는 전이중 통신이 가능하다. 각 핀의 기능과 역할은 다음 표에 요약되어 있다.

| 핀 명칭         | 전체 이름                  | 마스터 역할 | 슬레이브 역할 | 신호 방향      | 주요 기능                                           |
| --------------- | -------------------------- | ----------- | ------------- | -------------- | --------------------------------------------------- |
| **SCLK**        | Serial Clock               | 출력(Out)   | 입력(In)      | Master -->> Slave | 통신 타이밍 동기화를 위한 클럭 신호 생성 및 전송 2  |
| **MOSI**        | Master Out Slave In        | 출력(Out)   | 입력(In)      | Master -->> Slave | 마스터가 슬레이브에게 데이터를 전송 2               |
| **MISO**        | Master In Slave Out        | 입력(In)    | 출력(Out)     | Slave -->> Master | 슬레이브가 마스터에게 데이터를 전송 2               |
| **SS** / **CS** | Slave Select / Chip Select | 출력(Out)   | 입력(In)      | Master -->> Slave | 마스터가 특정 슬레이브를 선택, 활성화 시키는 신호 2 |


- **SCLK (Serial Clock):** 마스터가 생성하여 슬레이브로 보내는 동기화 클럭 신호이다.2 데이터 전송은 이 클럭의 엣지(Edge)에 동기화되어 이루어진다.
- **MOSI (Master Out Slave In):** 마스터가 슬레이브로 데이터를 내보내는 단방향 데이터 라인이다.2 마스터 측에서는 출력(Out), 슬레이브 측에서는 입력(In) 역할을 한다.3
- **MISO (Master In Slave Out):** 슬레이브가 마스터로 데이터를 내보내는 단방향 데이터 라인이다.2 마스터 측에서는 입력(In), 슬레이브 측에서는 출력(Out) 역할을 한다.3
- **SS (Slave Select) / CS (Chip Select):** 마스터가 통신을 원하는 특정 슬레이브를 선택하는 데 사용되는 신호선이다.2 이 신호는 일반적으로 `Active-Low`로 동작하며, 마스터가 특정 슬레이브의 `SS/CS` 핀을 `LOW` 상태로 만들 때 해당 슬레이브가 활성화되어 마스터의 클럭과 데이터에 반응한다.4

SPI 통신은 `1:N` 통신이 가능하다는 장점을 갖지만, I2C 통신처럼 슬레이브를 식별하기 위한 별도의 `주소값`이 프로토콜 자체에 내장되어 있지 않다.4 대신 마스터는 `SS/CS`라는 별도의 아웃밴드(out-band) 신호선을 통해 통신할 슬레이브를 선택한다.4 이 방식은 슬레이브 수가 늘어날수록 마스터에 필요한 `SS/CS` 핀의 수도 비례하여 증가한다는 물리적 한계로 이어진다.2 이러한 배선 복잡성은 SPI 통신이 소수의 장치를 연결하는 근거리 통신에 주로 사용되는 이유를 뒷받침한다.4



SPI 통신의 핵심은 마스터와 슬레이브 양단에 존재하는 **시프트 레지스터(Shift Register)**를 이용한 데이터 교환 방식이다.4 마스터가 `SCLK` 클럭 신호를 생성하면, 매 클럭 주기마다 마스터의 시프트 레지스터에 저장된 데이터가 1비트씩 `MOSI` 라인을 통해 슬레이브로 밀려나간다.4 이와 동시에, 슬레이브의 시프트 레지스터에 저장된 데이터도 1비트씩 `MISO` 라인을 통해 마스터로 밀려들어온다. 이 과정을 추상화하여 표현하면 다음과 같다.
$$
D_{M\_in,i} = D_{S\_out,i-1}, \quad D_{S\_in,i} = D_{M\_out,i-1}
$$
여기서 D는 데이터를 의미하고, M은 마스터, S는 슬레이브, in은 입력, out은 출력, 그리고 i는 클럭 주기를 나타낸다. 이 수식은 i번째 클럭 주기에서 마스터의 입력 데이터는 슬레이브의 직전 출력 데이터(i−1 주기)와 같고, 슬레이브의 입력 데이터는 마스터의 직전 출력 데이터와 같음을 보여준다.

데이터 전송은 일반적으로 최상위 비트(`MSB`, Most Significant Bit)부터 이루어지지만, 장치에 따라 최하위 비트(`LSB`, Least Significant Bit)부터 전송하는 경우도 있으므로, 반드시 해당 장치의 데이터시트를 확인해야 한다.4


SPI 통신은 **클럭 극성(CPOL)**과 **클럭 위상(CPHA)**이라는 두 가지 파라미터를 조합하여 총 네 가지의 통신 모드를 정의한다.9 통신에 참여하는 모든 장치는 동일한 모드로 설정되어야만 정상적인 데이터 교환이 가능하다.21

- **CPOL (Clock Polarity):** 통신이 없는 유휴(Idle) 상태에서 `SCLK`의 기본 논리 레벨을 정의한다.9
  - `CPOL = 0`: 유휴 상태에서 `SCLK`가 `LOW` 상태이다.9
  - `CPOL = 1`: 유휴 상태에서 `SCLK`가 `HIGH` 상태이다.9
- **CPHA (Clock Phase):** `SCLK`의 어느 엣지에서 데이터를 샘플링(Sampling)할지, 즉 데이터를 읽을 시점을 정의한다.9
  - `CPHA = 0`: `SCLK`의 `첫 번째 엣지(Leading Edge)`에서 데이터를 샘플링한다.9
  - `CPHA = 1`: `SCLK`의 `두 번째 엣지(Trailing Edge)`에서 데이터를 샘플링한다.9

이 두 파라미터의 조합으로 형성되는 네 가지 SPI 통신 모드는 다음 표와 같다.23

| 통신 모드  | CPOL (클럭 극성) | CPHA (클럭 위상) | 유휴 상태 클럭 | 데이터 샘플링 엣지       |
| ---------- | ---------------- | ---------------- | -------------- | ------------------------ |
| **Mode 0** | 0                | 0                | Low            | 첫 번째 엣지 (상승 엣지) |
| **Mode 1** | 0                | 1                | Low            | 두 번째 엣지 (하강 엣지) |
| **Mode 2** | 1                | 0                | High           | 첫 번째 엣지 (하강 엣지) |
| **Mode 3** | 1                | 1                | High           | 두 번째 엣지 (상승 엣지) |


통신 모드는 `SS` 신호가 활성화된 이후의 클럭과 데이터의 타이밍 관계를 결정한다. 예를 들어, `Mode 0 (CPOL=0, CPHA=0)`의 경우, `SS` 신호가 `LOW`로 전환되면 `SCLK`는 유휴 상태인 `LOW`를 유지한다.17 이후 `SCLK`가 `LOW`에서 `HIGH`로 바뀌는 첫 번째 상승(Rising) 엣지에서 데이터를 샘플링하며, `HIGH`에서 `LOW`로 바뀌는 하강(Falling) 엣지에서 데이터를 출력한다.17

클럭과 데이터의 관계를 수학적으로 표현하면 다음과 같다.

- Mode 0의 경우, 데이터는 클럭의 첫 번째 상승 엣지에서 샘플링되며, 이는 $t_{sample} = t_{rise,1}$로 표현할 수 있다.
- Mode 1의 경우, 데이터는 클럭의 첫 번째 하강 엣지에서 샘플링되므로 $t_{sample} = t_{fall,1}$이 된다.

과거에는 `CPOL`과 `CPHA` 값이 하나의 값으로 미리 지정되어 설계된 SPI 코어가 존재하여, 서로 다른 모드를 사용하는 장치들 간의 통신이 불가능했다.21 이 경우 하드웨어를 재설계해야 하는 문제가 발생하였다. 하지만 오늘날의 마이크로컨트롤러는 `SPCR(SPI Control Register)`과 같은 제어 레지스터를 통해 `CPOL`과 `CPHA` 값을 프로그래밍 방식으로 유연하게 설정할 수 있게 함으로써 이러한 호환성 문제를 해결하였다. 이는 SPI 통신이 단순한 하드웨어 프로토콜을 넘어, 소프트웨어적으로 유연하게 제어되는 방식으로 진화하였음을 보여준다.18



- **고속 통신:** SPI는 클럭 신호에 대한 이론적인 제한이 없어 통신 속도가 빠르다.4

  `Push-Pull` 출력 방식을 사용하여 신호 무결성을 확보하고 고속 데이터 전송을 지원한다.4

- **전이중 통신:** `MOSI`와 `MISO` 신호선이 분리되어 있어 동시에 데이터를 주고받을 수 있으므로 통신 효율이 매우 높다.2

- **간단한 하드웨어 처리:** 별도의 시작/정지 비트나 주소 지정 프로토콜이 필요 없어 하드웨어 인터페이스 처리 과정이 간단하다.4

- **저전력 소모:** I2C 통신과 비교하여 상대적으로 낮은 전력을 소비한다.4


- **배선 복잡성:** 슬레이브의 수가 증가할수록 각각의 슬레이브를 선택하기 위한 `SS/CS` 라인이 비례적으로 늘어나 배선이 복잡해지는 물리적인 한계가 존재한다.2
- **오류 검사 기능 부재:** SPI 프로토콜 자체에는 `CRC(Cyclic Redundancy Check)`와 같은 오류 검사(`Error Checking`) 기능이 정의되어 있지 않다.4 따라서 데이터의 무결성은 전적으로 하드웨어적 안정성에 의존하게 된다. 하지만 이 한계는 일부 IC의 경우 SPI 명령어 구조에 CRC 코드를 포함시켜 보완할 수 있다.25 이는 SPI의 높은 프로토콜 유연성을 활용하여 응용 계층에서 기능을 확장한 사례이다.4
- **하드웨어 흐름 제어 부재:** 슬레이브가 마스터의 전송 속도를 제어할 수 있는 하드웨어적 메커니즘이 없어, 마스터는 슬레이브가 데이터를 처리하는 데 필요한 시간을 고려하여 클럭 속도를 조절해야 한다.4
- **거리 및 노이즈의 제약:** 비교적 짧은 거리(칩 간)에서 동작하며, 외부 노이즈 스파이크에 영향을 받기 쉬운 경향이 있다.1



SPI 통신은 I2C 및 UART와 같은 다른 주요 직렬 통신 프로토콜과 비교했을 때, 각각의 뚜렷한 장단점을 갖는다. 다음 표는 세 가지 프로토콜의 핵심 특징을 비교한다.

| 특징          | SPI (Serial Peripheral Interface)      | I2C (Inter-Integrated Circuit)                   | UART (Universal Asynchronous Receiver/Transmitter) |
| ------------- | -------------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| **통신 방식** | 동기식, 전이중                         | 동기식, 반이중                                   | 비동기식, 전이중                                   |
| **배선 수**   | 최소 4개 (SCLK, MOSI, MISO, SS)        | 2개 (SCL, SDA)                                   | 2개 (Tx, Rx)                                       |
| **통신 속도** | 빠름 (이론적 제한 없음, 10Mbps~20Mbps) | 상대적으로 느림 (Standard 100Kbps, Fast 400Kbps) | 느림 (Baud Rate에 의존, 최대 115,200bps)           |
| **통신 구조** | 1 Master - 1:N Slave                   | 1:N Master - 1:N Slave                           | 1:1 통신                                           |
| **주소 지정** | `SS/CS` 신호선 사용 (하드웨어)         | 7비트 또는 10비트 주소 사용 (소프트웨어)         | 없음                                               |
| **장점**      | 고속, 전이중, 간단한 하드웨어          | 배선 단순, 다수 마스터 지원                      | 하드웨어 단순, 별도 클럭선 불필요                  |
| **단점**      | 배선 복잡, 오류 검사 부재              | 반이중, 통신 속도 느림                           | 비동기식으로 인한 타이밍 오차 가능성               |


SPI와 I2C는 모두 마이크로컨트롤러와 주변 장치 간의 근거리 통신에 주로 사용된다. SPI는 `전이중 통신`과 `고속`이 필요한 응용 분야(예: ADC, 플래시 메모리)에 적합한 반면 4, I2C는 단 2개의 신호선만을 사용하고 `주소 지정`을 통해 다수의 슬레이브를 관리할 수 있어 `배선 단순화`가 중요한 저속 센서 네트워크에 주로 활용된다.11

반면 UART는 `클럭선`이 없는 `비동기식` 통신이라는 특징 덕분에 하드웨어 구성이 가장 단순하다.11 그러나 송수신 양단이 동일한 `Baud Rate`를 사용해야 하므로 타이밍 오차의 영향을 받을 수 있으며, 통신 속도도 상대적으로 느리다.11 결국 임베디드 시스템 설계자는 단순히 속도만을 고려할 것이 아니라, 전체 시스템의 배선 복잡도, 전력 소모, 오류 허용치 등 종합적인 요소를 고려하여 최적의 통신 프로토콜을 선택해야 한다.



마이크로컨트롤러(MCU)에서 SPI 통신을 구현하기 위해서는 다음과 같은 일반적인 절차를 거친다.

1. **GPIO 핀 설정:** SPI 통신에 사용될 `SCLK`, `MOSI`, `MISO`, `SS` 핀들을 각각의 역할에 맞는 입출력 모드로 설정한다.24
2. **SPI 레지스터 설정:** MCU 내부의 SPI 제어 레지스터를 설정하여 SPI 통신을 활성화하고, 해당 MCU를 `마스터 모드`로 동작하도록 설정한다.18 또한, 주변 장치에 맞는 `CPOL`, `CPHA` 값을 설정하고, 클럭 분주비(Prescaler)를 조절하여 적절한 통신 속도를 결정한다.23
3. **통신 시작:** 통신을 원하는 슬레이브의 `SS` 핀을 `LOW`로 설정하여 해당 장치를 활성화하고, 데이터 전송 및 수신을 진행한다.18

이 과정에서 가장 중요한 것은 통신 대상 장치의 `데이터시트`를 정확하게 분석하는 것이다. 데이터시트는 `CPOL/CPHA` 설정(통신 모드), 데이터 전송 순서(MSB/LSB), 그리고 장치를 제어하기 위한 고유의 명령어 프로토콜을 명시하고 있어, 이를 토대로 MCU의 레지스터를 정확하게 설정해야만 정상적인 통신이 가능하다.15


마이크로컨트롤러는 SPI 통신을 위한 전용 레지스터와 함수를 제공한다. 예를 들어, ATmega328P MCU의 경우 다음과 같은 C 코드를 통해 SPI를 활성화하고 마스터로 설정할 수 있다.18

```
SPCR = (1 << SPE) | (1 << MSTR) | (1 << SPR1) | (1 << SPR0); // SPI 활성화, 마스터 모드, 클럭 분주비 설정
```

STM32와 같은 현대의 MCU는 `HAL(Hardware Abstraction Layer)` 라이브러리를 통해 통신을 보다 쉽게 구현할 수 있다. `HAL_SPI_TransmitReceive()` 함수를 사용하면 데이터 송신과 수신을 동시에 처리할 수 있다.28

```
HAL_SPI_TransmitReceive(&hspi1, T_buffer, R_buffer, 8, 500); // 8비트 데이터 동시 송수신
```

실제 구현 시에는 `DMA(Direct Memory Access)` 모드를 사용할 때 타이밍 문제로 인해 데이터가 잘못 수신되는 경우가 발생할 수 있다. 이 경우 `delay`를 추가하거나, 함수 호출 시 `타임아웃(timeout)` 값을 충분히 늘리는 방식으로 문제를 해결할 수 있다.30 또한, `max7219` LED 디스플레이 컨트롤러와 같은 일부 장치는 단순히 데이터를 보내는 것이 아니라, 먼저 `주소(Address)` 역할을 하는 명령어를 보내고, 그 다음 `설정값(Data)`을 보내는 복합적인 프로토콜을 사용하므로, 이러한 장치 고유의 통신 규격을 준수해야 한다.29


SPI 통신은 다양한 임베디드 시스템에서 활용된다.

- **센서:** 고속의 데이터 샘플링이 필요한 대기압 및 고도 센서(`BMP183`) 31, 온도 및 모션 센서 3 등은 SPI를 통해 마이크로컨트롤러와 통신한다.
- **메모리 및 저장 장치:** 시프트 레지스터 4나 SD카드와 같은 저장 장치와의 고속 데이터 교환에 사용된다.
- **디스플레이:** `ILI9341` 또는 `ST7789Vi`와 같은 컨트롤러 칩을 탑재한 TFT LCD 모듈은 SPI 인터페이스를 사용하여 적은 수의 핀으로도 고해상도 이미지를 빠르게 처리할 수 있다.27
- **복합 시스템:** 디지털 카메라 시스템의 경우, 프로세서(마스터)는 SPI를 통해 카메라 센서(슬레이브)를 제어하고, 센서로부터 고해상도 이미지를 실시간으로 고속 수신한다.3 이러한 실시간 데이터 처리는 SPI의 고속 전이중 통신 특성 덕분에 가능하다.



SPI 통신은 `고속`, `전이중`, 그리고 `간단한` 하드웨어 인터페이스를 제공하여 `근거리 칩 간 통신`이라는 특정 분야에서 독보적인 위치를 차지한다.4 특히 `SCLK`를 이용한 동기식 통신은 통신 속도 오차 문제를 회피하고 전송 안정성을 높이는 데 기여한다.11

하지만 SPI는 슬레이브 수에 따라 `SS/CS` 라인이 비례하여 증가하는 `배선 복잡성`과 프로토콜 자체에 `오류 검사` 기능이 부재하다는 한계를 갖는다.2 이러한 단점은 응용 분야의 특성과 `소프트웨어` 및 `하드웨어 설계`를 통해 극복될 수 있다. 예를 들어, CRC와 같은 오류 검사 기능은 통신 프로토콜 상위 계층에서 추가할 수 있으며 25, 배선 복잡성은 1:1 또는 소수의 슬레이브를 사용하는 시스템에 SPI를 적용함으로써 해소할 수 있다. 즉, SPI 통신은 그 한계와 장점을 명확히 이해하고, 다른 프로토콜(I2C, UART)과 비교하여 시스템의 목적에 맞는 최적의 통신 방식을 선택하는 것이 임베디드 시스템 설계의 핵심임을 재차 확인한다.


IoT, 자율주행 로봇, 실시간 모니터링 시스템 등 데이터의 신속한 처리가 필수적인 현대 기술 분야에서 SPI의 `고속성`과 `안정성`은 여전히 중요한 가치를 지닌다.3 기술이 발전함에 따라 데이터 전송 속도는 더욱 빨라질 것이며, 이는 신호 무결성(Signal Integrity) 문제의 중요성을 더욱 부각시킨다. 따라서 미래에는 PCB 레이아웃 최적화, 임피던스 매칭, 노이즈 감소 기술 등 하드웨어 설계의 완성도를 높이는 것이 SPI 통신의 성능과 신뢰성을 보장하는 데 더욱 중요한 요소가 될 것이다.34 SPI는 고속 근거리 통신의 표준으로서, 앞으로도 다양한 임베디드 시스템에서 중추적인 역할을 수행할 것으로 전망된다.


1. 실무자가 들려주는 SPI 통신 - 앤카이브, 8월 10, 2025에 액세스, https://anchive.tistory.com/5
2. SPI(Serial Peripheral Interface Bus)란? - Mr.HB이야기 - 티스토리, 8월 10, 2025에 액세스, https://gudgud.tistory.com/30
3. SPI는 무엇입니까?직렬 주변 장치 인터페이스가 설명되었습니다, 8월 10, 2025에 액세스, https://www.ariat-tech.kr/blog/what-is-spi-serial-peripheral-interface-explained.html
4. [시리얼 통신] SPI(Serial Peripheral Interface)통신이란? - 외부 저장소 - 티스토리, 8월 10, 2025에 액세스, https://aliencoder.tistory.com/92
5. SPI 통신 (1) - 날아보자 - 티스토리, 8월 10, 2025에 액세스, [https://treeroad.tistory.com/entry/SPI-%ED%86%B5%EC%8B%A0-1](https://treeroad.tistory.com/entry/SPI-통신-1)
6. 여러 장치 간의 연결을 단순화하기 위해 SPI를 사용하는 이유 및 방법, 8월 10, 2025에 액세스, https://www.digikey.kr/ko/articles/why-how-to-use-serial-peripheral-interface-simplify-connections-between-multiple-devices
7. I2C와 SPI 통신, MOSI, MISO, SCL, SS, SDA 등의 핀 정리 - 어중이 기록 창고 - 티스토리, 8월 10, 2025에 액세스, https://thinkanother.tistory.com/13
8. 전이중 통신 SPI(Serial Peripheral Interface) - 개발대장 KET - 티스토리, 8월 10, 2025에 액세스, https://ket-jm.tistory.com/65
9. SPI - Serial Peripheral Interface 통신 - 잇!(IT) 블로그 - 티스토리, 8월 10, 2025에 액세스, https://insoobaik.tistory.com/571
10. 직렬 주변기기 인터페이스 버스 - 위키백과, 우리 모두의 백과사전, 8월 10, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EC%A7%81%EB%A0%AC_%EC%A3%BC%EB%B3%80%EA%B8%B0%EA%B8%B0_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4_%EB%B2%84%EC%8A%A4](https://ko.wikipedia.org/wiki/직렬_주변기기_인터페이스_버스)
11. H/W 통신 프로토콜의 모든 것 Part.1 - UART, SPI, I2C, 8월 10, 2025에 액세스, [https://rangvest.tistory.com/entry/HW-%ED%86%B5%EC%8B%A0-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C%EC%9D%98-%EB%AA%A8%EB%93%A0-%EA%B2%83-Part1-UART-SPI-I2C](https://rangvest.tistory.com/entry/HW-통신-프로토콜의-모든-것-Part1-UART-SPI-I2C)
12. 전이중 통신, SPI 통신이란? - 한국전자기술, 8월 10, 2025에 액세스, [https://k-rnd.com/%EC%A0%84%EC%9D%B4%EC%A4%91-%ED%86%B5%EC%8B%A0-spi-%ED%86%B5%EC%8B%A0%EC%9D%B4%EB%9E%80/](https://k-rnd.com/전이중-통신-spi-통신이란/)
13. k-rnd.com, 8월 10, 2025에 액세스, [https://k-rnd.com/%EC%A0%84%EC%9D%B4%EC%A4%91-%ED%86%B5%EC%8B%A0-spi-%ED%86%B5%EC%8B%A0%EC%9D%B4%EB%9E%80/#:~:text=SPI%20%ED%86%B5%EC%8B%A0%EC%9D%80%20%EC%B5%9C%EB%8C%80%20Clock,%ED%95%98%EC%97%AC%20%EC%86%90%EC%89%BD%EA%B2%8C%20%EC%82%AC%EC%9A%A9%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.](https://k-rnd.com/전이중-통신-spi-통신이란/#:~:text=SPI 통신은 최대 Clock,하여 손쉽게 사용할 수 있습니다.)
14. Serial 통신 비교(UART vs SPI vs I2C) - 삼손스 - 티스토리, 8월 10, 2025에 액세스, https://samsons.tistory.com/7
15. SPI(Serial Peripheral Interface)란? - velog, 8월 10, 2025에 액세스, [https://velog.io/@lutein/SPISerial-Peripheral-Interface%EB%9E%80](https://velog.io/@lutein/SPISerial-Peripheral-Interface란)
16. SPI 통신 - 어중이 기록 창고 - 티스토리, 8월 10, 2025에 액세스, https://thinkanother.tistory.com/17
17. Back to Basics: SPI (Serial Peripheral Interface) - Technical Articles, 8월 10, 2025에 액세스, https://www.allaboutcircuits.com/technical-articles/spi-serial-peripheral-interface/
18. [AVR] ATmega328P SPI 구현 - kyuhyuk.kr, 8월 10, 2025에 액세스, https://kyuhyuk.kr/article/avr/2022/08/28/AVR-ATmega328P-SPI
19. UART, I2C, SPI 통신 - 편하게 보는 전자공학 블로그, 8월 10, 2025에 액세스, https://kkhipp.tistory.com/163
20. SPI 통신 - 창고 - 티스토리, 8월 10, 2025에 액세스, [https://duksoo.tistory.com/entry/SPI-%ED%86%B5%EC%8B%A0](https://duksoo.tistory.com/entry/SPI-통신)
21. KR101447154B1 - Fpga를 이용한 범용 spi 코어의 동작 방법 - Google Patents, 8월 10, 2025에 액세스, https://patents.google.com/patent/KR101447154B1/ko
22. Understanding SPI Clock Signals: Polarity, Phase, Clock Edges, and SPI Modes, 8월 10, 2025에 액세스, https://www.totalphase.com/blog/2024/07/understanding-spi-clock-signals-polarity-phase-clock-edges-spi-modes/
23. Why Different modes are provided in SPI communication? - Stack Overflow, 8월 10, 2025에 액세스, https://stackoverflow.com/questions/43155025/why-different-modes-are-provided-in-spi-communication
24. STM32 ] SPI 통신 사용하기 - 개준생의 공부 일지 - 티스토리, 8월 10, 2025에 액세스, https://eteo.tistory.com/160
25. CRC와 인터럽트를 이용한 SPI 통신 오류 검출 : r/embedded - Reddit, 8월 10, 2025에 액세스, https://www.reddit.com/r/embedded/comments/ql7nxf/spi_communication_error_checking_with_crc_and/?tl=ko
26. 통신오류검출 - 보안을 사랑하는, 8월 10, 2025에 액세스, https://hj-kwon.tistory.com/7
27. Embedded System (2) : 라즈베리파이를 이용한 통신, I2C, SPI 방식을 통한 프로그램 구현, 8월 10, 2025에 액세스, https://lg960214.tistory.com/69
28. STM32F103 SPI 통신 코드 작성하기 - DKMIN, 8월 10, 2025에 액세스, [https://dkeemin.com/stm32f1xx-spi-%ED%86%B5%EC%8B%A0-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0/](https://dkeemin.com/stm32f1xx-spi-통신-코드-작성하기/)
29. STM32 - SPI통신 MAX7219 7-segment 모듈 - rs29 - 티스토리, 8월 10, 2025에 액세스, https://rs29.tistory.com/13
30. SPI 통신 질문 - 인프런 | 커뮤니티 질문&답변, 8월 10, 2025에 액세스, [https://www.inflearn.com/community/questions/974375/spi-%ED%86%B5%EC%8B%A0-%EC%A7%88%EB%AC%B8](https://www.inflearn.com/community/questions/974375/spi-통신-질문)
31. BMP183 대기압 및 고도 센서 -SPI 통신(Adafruit BMP183 SPI Barometric Pressure & Altitude Sensor) - 가치창조기술, 8월 10, 2025에 액세스, [https://m.vctec.co.kr/product/bmp183-%EB%8C%80%EA%B8%B0%EC%95%95-%EB%B0%8F-%EA%B3%A0%EB%8F%84-%EC%84%BC%EC%84%9C-spi-%ED%86%B5%EC%8B%A0adafruit-bmp183-spi-barometric-pressure-altit/5175/](https://m.vctec.co.kr/product/bmp183-대기압-및-고도-센서-spi-통신adafruit-bmp183-spi-barometric-pressure-altit/5175/)
32. 2.2 SPI TFT LCD 모듈 (2.2 SPI TFT LCD Module) - 가치창조기술, 8월 10, 2025에 액세스, [https://m.vctec.co.kr/product/22-spi-tft-lcd-%EB%AA%A8%EB%93%88-22-spi-tft-lcd-module/5136/](https://m.vctec.co.kr/product/22-spi-tft-lcd-모듈-22-spi-tft-lcd-module/5136/)
33. 2.4인치 프리미엄 SPI 저항성 TFT 디스플레이 - Newhaven Display, 8월 10, 2025에 액세스, https://newhavendisplay.com/ko/2-4-inch-premium-spi-resistive-tft-display/
34. [SI/PI]신호 무결성 해부 : 소개 - 지식저장소 - 티스토리, 8월 10, 2025에 액세스, [https://knowingstorage.tistory.com/entry/SIPI%EC%8B%A0%ED%98%B8-%EB%AC%B4%EA%B2%B0%EC%84%B1-%ED%95%B4%EB%B6%80-%EC%86%8C%EA%B0%9C](https://knowingstorage.tistory.com/entry/SIPI신호-무결성-해부-소개)
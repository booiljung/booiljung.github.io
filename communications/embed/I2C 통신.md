# I2C 통신
[임베디드 통신](./index.md)



I2C(Inter-Integrated Circuit)는 1982년 필립스 반도체(Philips Semiconductors, 現 NXP Semiconductors)에 의해 개발된 동기식 직렬 통신 프로토콜이다.1 흔히 '아이-스퀘어-씨($I^2C$)' 또는 '아이-투-씨(I-two-C)'로 발음되며, TWI(Two-Wire Interface)라는 별칭으로도 불린다.3 2021년, I2C 사양 개정 7판에서는 기존의 '마스터(Master)/슬레이브(Slave)' 용어를 정치적 올바름을 고려하여 '컨트롤러(Controller)/타겟(Target)'으로 변경하였으나, 기술적인 역할과 정의는 동일하게 유지된다.2 본 보고서에서는 독자의 폭넓은 이해를 돕기 위해 전통적인 용어와 새로운 용어를 병기하여 서술한다.

I2C의 개발은 1980년대 반도체 기술의 발전과 그에 따른 물리적, 경제적 제약에서 비롯되었다. 집적 회로(IC)의 기능이 복잡해짐에 따라 필요한 핀(pin)의 수가 기하급수적으로 증가했고, 이는 IC 패키지의 크기 증대와 생산 비용 상승으로 직결되었다.5 또한, 인쇄 회로 기판(PCB) 상에서 다수의 배선(trace)이 필요해지면서 회로 설계의 복잡도가 증가하고 노이즈 간섭에 취약해지는 문제가 대두되었다.7 I2C는 이러한 문제에 대한 실용적이고 경제적인 해결책으로 탄생했다. 즉, I2C의 등장은 단순히 기술적 우위를 추구한 결과가 아니라, 당시 반도체 산업이 직면한 비용, 공간, 복잡성이라는 현실적 제약을 극복하기 위한 필연적인 설계 철학의 발현이었다.

초기 I2C는 디지털-아날로그 변환기(DAC)와 같은 오디오 칩셋과 제어 칩셋 간의 사운드 데이터 전송을 목적으로 설계되었으나, 그 범용성과 효율성 덕분에 곧 다양한 IC 간의 통신 규격으로 자리 잡았다.1 최소한의 배선으로 다수의 장치를 연결할 수 있다는 장점은 오늘날 사물 인터넷(IoT) 시대의 저전력, 소형화, 저비용 요구사항과 정확히 부합하며, 40년이 넘는 역사에도 불구하고 I2C가 여전히 임베디드 시스템의 핵심 통신 프로토콜로 널리 사용되는 이유를 설명해준다.5


I2C 프로토콜은 다음과 같은 네 가지 핵심적인 특징으로 정의된다.

- **2-Wire 인터페이스 (Two-Wire Interface)**: I2C 통신은 단 두 개의 신호선만을 사용한다. 하나는 실제 데이터 비트를 전송하는 직렬 데이터 라인(SDA, Serial Data Line)이고, 다른 하나는 데이터 전송의 타이밍을 동기화하는 직렬 클록 라인(SCL, Serial Clock Line)이다.3 이 구조는 I2C의 가장 근본적인 특징으로, 핀 수 절감이라는 개발 목표를 직접적으로 달성한다.9
- **동기식 통신 (Synchronous Communication)**: 모든 데이터 전송은 마스터(컨트롤러)가 생성하는 SCL 클록 신호에 맞춰 동기화된다. 이는 비동기식 통신인 UART(Universal Asynchronous Receiver-Transmitter)와 달리 통신 당사자 간에 사전에 전송 속도(Baudrate)를 약속할 필요가 없음을 의미한다.3 클록 신호가 데이터와 함께 전송되므로 통신 안정성이 높고 시간에 대해 자유롭다.1
- **다중 장치 버스 (Multi-device Bus)**: I2C는 버스(bus) 형태로 구성되어, 단일 버스에 하나의 마스터와 다수의 슬레이브(타겟)를 연결할 수 있다. 7비트 주소 체계를 기준으로 이론상 최대 127개의 슬레이브 장치를 연결할 수 있으며, 각 슬레이브는 고유한 주소(address)를 통해 식별된다.3 마스터가 특정 슬레이브의 주소를 버스에 전송하면, 해당 주소를 가진 슬레이브만이 응답하여 통신에 참여한다.4
- **반이중 통신 (Half-Duplex Communication)**: 데이터 송신과 수신이 하나의 SDA 라인을 공유하므로, 데이터 전송은 양방향으로 가능하지만 동시에 이루어질 수는 없다.11 즉, 한 시점에는 마스터가 슬레이브에게 데이터를 보내거나(쓰기), 슬레이브가 마스터에게 데이터를 보내는(읽기) 동작 중 하나만 가능하다.13
- **다중 마스터 지원 (Multi-Master Support)**: I2C 버스는 하나 이상의 마스터가 존재할 수 있는 다중 마스터 환경을 지원한다. 두 개 이상의 마스터가 동시에 버스 사용을 시도할 경우, 데이터 손상 없이 버스 제어권을 결정하는 중재(Arbitration) 메커니즘이 프로토콜에 내장되어 있다.2


I2C 프로토콜의 유연성과 안정성은 그 독특한 물리적 계층 설계에 깊이 뿌리내리고 있다. 특히 오픈 드레인(Open-Drain) 구조와 풀업 저항(Pull-up Resistor)의 조합은 I2C의 핵심 기능을 가능하게 하는 전기적 기반을 형성한다.


앞서 언급했듯이, I2C 버스는 두 개의 신호선으로 구성된다.8

- **SDA (Serial Data Line)**: 마스터와 슬레이브 간의 실제 데이터 비트가 전송되는 양방향 라인이다. 데이터는 이 라인을 통해 직렬로 전송된다.3
- **SCL (Serial Clock Line)**: 마스터가 생성하는 클록 신호를 전달하는 라인이다. SDA 라인에 실린 데이터 비트는 SCL 클록의 주기에 맞춰 샘플링된다. 이를 통해 모든 장치 간의 통신이 동기화된다.4

이 두 라인은 모두 양방향성을 가지며, 전기적으로는 오픈 드레인 또는 오픈 컬렉터(Open-Collector) 출력 구조를 갖는다.13


오픈 드레인 구조는 I2C의 물리적 계층을 이해하는 데 가장 중요한 개념이다. 이는 단순히 전기적 특성을 넘어, I2C 프로토콜의 핵심 기능인 다중 마스터 중재와 클럭 스트레칭을 가능하게 하는 근본적인 메커니즘이기 때문이다.

- **원리**: I2C 장치의 SDA 및 SCL 핀 출력단은 NMOS 트랜지스터의 드레인(BJT의 경우 컬렉터)에만 연결되어 있고, 전원($V_{CC}$) 쪽은 전기적으로 개방(open)되어 있다.17 이 때문에 장치는 트랜지스터를 켜서(ON) 라인을 접지(GND) 쪽으로 당겨 LOW 상태를 만드는 것만 가능하다.12 트랜지스터를 끄면(OFF) 출력 핀은 높은 임피던스(high-impedance) 상태가 되어 사실상 버스에서 분리된다. 어떤 장치도 라인을 $V_{CC}$ 쪽으로 능동적으로 밀어 올려 HIGH 상태를 만들 수는 없다.18
- **"Wired-AND" 논리**: 이 구조로 인해 I2C 버스는 'Wired-AND'라는 독특한 전기적 특성을 갖게 된다. 버스에 연결된 여러 장치 중 단 하나라도 라인을 LOW로 당기면, 전체 버스 라인의 전압은 LOW가 된다.19 버스 라인이 HIGH 상태가 되기 위해서는 버스에 연결된 모든 장치가 자신의 출력 트랜지스터를 꺼서 라인을 해제해야만 한다. 이는 모든 입력이 '1'(해제 상태)일 때만 출력이 '1'(HIGH)이 되는 논리 AND 연산과 동일하다.
- **신호 충돌(Bus Contention) 방지**: 오픈 드레인 구조의 가장 큰 장점은 신호 충돌을 원천적으로 방지한다는 것이다. 푸시-풀(push-pull) 출력 구조에서는 한 장치가 HIGH를 출력하고 다른 장치가 LOW를 출력하면 $V_{CC}$에서 GND로 직접 전류가 흐르는 단락(short circuit)이 발생하여 장치가 손상될 수 있다. 하지만 I2C에서는 어떤 장치도 HIGH를 능동적으로 구동하지 않으므로 이러한 충돌이 발생하지 않는다.18 이 특성은 여러 장치가 동시에 버스를 제어하려는 상황(다중 마스터 중재)에서 데이터 손상 없이 안전하게 제어권을 이양하는 기반이 된다.


- **필요성**: 오픈 드레인 구조에서 모든 장치가 라인을 해제했을 때(유휴 상태 또는 논리 '1' 전송 시), 라인은 어느 곳에도 연결되지 않은 플로팅(floating) 상태가 된다. 이 상태는 전압 레벨이 불확실하여 논리 오류를 유발할 수 있다.22 풀업 저항은 SDA와 SCL 라인을 각각 전원($V_{CC}$)에 연결하여, 버스가 구동되지 않을 때 안정적인 HIGH 상태를 유지시켜주는 역할을 한다.3 대부분의 마이크로컨트롤러는 내부 풀업 저항을 제공하지만, 이는 저항값이 너무 커(약 40-50 kΩ) 안정적인 통신을 보장하기 어려우므로, 일반적으로 외부 풀업 저항을 사용하는 것이 권장된다.3

- **저항값 선정의 중요성**: 풀업 저항의 값($R_p$)은 I2C 통신의 속도와 신호 품질을 결정하는 매우 중요한 파라미터이다. 저항값은 너무 크지도, 너무 작지도 않은 적절한 범위 내에서 선택되어야 한다.

  - **저항값이 너무 클 경우 (Weak Pull-up)**: 버스 라인에는 PCB 트레이스, 핀, 연결된 장치들로 인한 기생 커패시턴스($C_b$)가 존재한다. 풀업 저항은 이 커패시턴스와 함께 RC 회로를 형성한다. 저항값이 크면 RC 시정수가 커져 커패시터를 충전하는 데 더 오랜 시간이 걸린다. 이는 신호의 상승 시간($t_r$)을 길게 만들어 파형의 모서리가 뭉툭해지는 결과를 낳는다.23 통신 속도가 빠를수록 한 클록 주기가 짧아지므로, 상승 시간이 규정된 시간을 초과하면 신호가 HIGH로 인식되기 전에 다음 클록이 시작되어 타이밍 위반 및 통신 오류를 유발한다.27
  - **저항값이 너무 작을 경우 (Strong Pull-up)**: 장치가 라인을 LOW로 당길 때, 풀업 저항을 통해 $V_{CC}$에서 GND로 흐르는 전류($I_{OL}$)를 싱크(sink)해야 한다. 저항값이 작으면 이 전류가 커진다. 만약 이 전류가 장치의 최대 싱크 전류 용량을 초과하면 장치가 손상될 수 있으며, 규정된 LOW 레벨 출력 전압($V_{OL(max)}$)을 만족시키지 못해 신호가 '0'으로 제대로 인식되지 않을 수 있다.23

- **최소 저항값 ($R_{p(min)}$) 계산**: 최소 풀업 저항값은 장치가 LOW 신호를 안정적으로 생성할 수 있는 능력에 의해 결정된다. 옴의 법칙에 따라 다음과 같이 계산할 수 있다.29
  $$
  R_{p(min)} = \frac{V_{CC} - V_{OL(max)}}{I_{OL}}
  $$
  여기서 $V_{CC}$는 풀업 저항에 연결된 공급 전압, $V_{OL(max)}$는 데이터시트에 명시된 최대 LOW 레벨 출력 전압(예: 0.4 V), $I_{OL}$은 장치가 싱크할 수 있는 최대 전류(예: 3 mA)이다.28

- **최대 저항값 ($R_{p(max)}$) 계산**: 최대 풀업 저항값은 신호의 상승 시간($t_r$)이 I2C 사양을 만족해야 한다는 조건에 의해 결정된다. RC 회로의 충전 방정식을 기반으로 다음과 같이 근사화할 수 있다.29
  $$
  R_{p(max)} = \frac{t_r}{0.8473 \times C_b}
  $$
  여기서 $t_r$은 해당 속도 모드의 최대 허용 상승 시간(예: Fast-mode에서 300 ns)이고, $C_b$는 버스의 총 커패시턴스이다. 계수 0.8473은 전압이 30%에서 70%까지 상승하는 데 걸리는 시간을 계산하기 위해 사용된다.28

  $C_b$는 버스에 연결된 모든 요소(IC 핀, PCB 트레이스, 커넥터 등)의 커패시턴스를 합산한 값으로, 정확한 측정이 어려울 경우 경험적인 값을 사용하거나 데이터시트를 참조하여 추정해야 한다.

일반적으로 2 kΩ에서 10 kΩ 사이의 값이 널리 사용되며, 4.7 kΩ이 표준처럼 쓰이는 경우가 많다.17 하지만 고속 통신이나 버스 부하가 큰 환경에서는 반드시 계산을 통해 최적의 값을 선정해야 한다.


I2C 통신은 물리적 계층의 전기적 신호 위에 잘 정의된 논리적 프로토콜 구조를 통해 이루어진다. 이 구조는 마스터-슬레이브 아키텍처를 기반으로 하며, 통신의 시작과 끝, 장치 선택, 데이터 교환 및 오류 확인을 위한 명확한 규칙들로 구성된다.


I2C 버스는 통신을 주도하는 마스터와 그에 응답하는 슬레이브라는 두 가지 역할로 구성된다.2

- **마스터 (Master / Controller)**: 통신의 주체로서 SCL 클록 신호를 생성하고, 통신을 시작하는 START 조건과 종료하는 STOP 조건을 발생시킨다. 마스터는 통신하고자 하는 특정 슬레이브의 주소를 버스에 전송하여 통신을 개시한다.9
- **슬레이브 (Slave / Target)**: 버스에 연결된 수동적인 장치로, 자신의 고유 주소가 마스터에 의해 호출될 때만 통신에 참여한다. 마스터의 클록 신호에 동기화되어 마스터의 요청에 따라 데이터를 수신하거나(쓰기 동작) 전송한다(읽기 동작).9


데이터 전송과 제어 신호를 구분하기 위해 I2C는 특별한 전기적 신호 조건을 사용한다. 이 조건들은 SCL이 HIGH 상태일 때 SDA의 상태가 변하는 것으로 정의되며, 이는 일반적인 데이터 전송 규칙(SCL이 HIGH일 때 SDA는 안정)의 예외이다.8

- **START 조건**: 통신의 시작을 알리는 신호로, SCL 라인이 HIGH인 상태에서 SDA 라인이 HIGH에서 LOW로 전이(falling edge)될 때 발생한다.4 이 신호가 감지되면 버스에 연결된 모든 슬레이브 장치들은 통신이 시작됨을 인지하고, 이어지는 주소 비트를 수신할 준비를 한다. 이로써 버스는 유휴(idle) 상태에서 사용 중(busy) 상태로 전환된다.13
- **STOP 조건**: 통신의 종료를 알리는 신호로, SCL 라인이 HIGH인 상태에서 SDA 라인이 LOW에서 HIGH로 전이(rising edge)될 때 발생한다.4 이 신호는 현재의 데이터 전송이 모두 완료되었음을 의미하며, 버스는 다시 유휴 상태로 돌아가 다른 마스터가 사용할 수 있게 된다.18
- **Repeated START 조건**: 하나의 통신 트랜잭션이 끝났음을 알리는 STOP 조건을 보내지 않고, 곧바로 새로운 통신을 시작하기 위해 보내는 START 조건이다. 전기적으로는 START 조건과 동일하지만, 버스가 이미 사용 중인 상태에서 발생한다는 차이점이 있다.13 이 조건은 주로 복합적인 통신 시퀀스에서 유용하게 사용된다. 예를 들어, 특정 슬레이브의 특정 레지스터에서 데이터를 읽고자 할 때, 먼저 쓰기 모드로 해당 레지스터 주소를 전송한 뒤, STOP 없이 Repeated START를 보내고 읽기 모드로 전환하여 데이터를 수신하는 방식이다. 이는 다중 마스터 환경에서 다른 마스터에게 버스 제어권을 빼앗기지 않고 연속적인 작업을 보장하는 데 필수적이다.8


마스터는 버스에 연결된 수많은 슬레이브 중 통신할 대상을 정확히 지정해야 한다. 이를 위해 I2C는 주소 지정 방식을 사용한다.

- **주소 프레임 (Address Frame)**: START 조건 직후에 전송되는 첫 번째 바이트(8비트)를 주소 프레임이라 한다. 이 프레임에는 슬레이브의 주소와 데이터 전송 방향 정보가 포함된다.13

- **7비트 주소 지정**: 가장 널리 사용되는 표준 방식으로, 주소 프레임의 상위 7비트($A_6$~$A_0$)가 슬레이브의 고유 주소를 나타낸다.3 이 7비트는 

  $2^7 = 128$개의 고유 주소를 생성할 수 있다. 이 중 일부 주소는 특수 목적(예: General Call, 10비트 주소 지정 등)으로 예약되어 있어, 실제로는 약 112개의 슬레이브 장치를 연결할 수 있다.1

- **10비트 주소 지정**: 더 많은 장치를 버스에 연결해야 할 필요성이 생기면서 도입된 확장 주소 체계이다.2 10비트 주소는 두 개의 바이트에 걸쳐 전송된다. 첫 번째 바이트는 

  `11110`이라는 5비트의 특수 코드로 시작하여, 이것이 10비트 주소임을 모든 장치에 알린다. 그 뒤에 주소의 상위 2비트와 R/W 비트가 따라오고, 두 번째 바이트에 나머지 하위 8비트 주소가 전송된다.15 10비트 주소 장치는 

  `11110` 코드가 기존 7비트 주소와 겹치지 않기 때문에 7비트 주소 장치와 동일한 버스에 공존할 수 있다.15


- **R/W 비트 (Read/Write Bit)**: 주소 프레임의 마지막 8번째 비트로, 데이터 전송의 방향을 결정한다.3
  - **`0` (LOW)**: 마스터가 슬레이브에게 데이터를 쓰는(Write) 동작을 의미한다. 즉, 마스터가 송신자, 슬레이브가 수신자가 된다.10
  - **`1` (HIGH)**: 마스터가 슬레이브로부터 데이터를 읽는(Read) 동작을 의미한다. 즉, 마스터가 수신자, 슬레이브가 송신자가 된다.10
- **데이터 바이트 (Data Byte)**: 주소 프레임 이후에 전송되는 실제 정보 단위로, 8비트로 구성된다. 데이터는 항상 최상위 비트(MSB, Most Significant Bit)부터 순차적으로 전송된다.18
- **ACK/NACK (Acknowledge/Not Acknowledge)**: I2C 프로토콜의 신뢰성을 보장하는 핵심적인 피드백 메커니즘이다. 주소 또는 데이터 바이트가 8비트 전송된 후, 9번째 클록 펄스 구간에서 수신 측이 송신 측으로 응답을 보낸다.3
  - **ACK (Acknowledge)**: 수신 측이 8비트 데이터를 성공적으로 수신했음을 알리는 신호이다. 수신 측은 9번째 클록 펄스가 HIGH인 동안 SDA 라인을 LOW로 당겨 ACK 신호를 생성한다.10 송신 측은 이 ACK 신호를 확인하고 다음 데이터 바이트를 전송하거나 통신을 계속 진행한다.
  - **NACK (Not Acknowledge)**: 수신 측이 응답하지 않거나 데이터를 더 이상 받을 수 없음을 알리는 신호이다. 9번째 클록 펄스 동안 SDA 라인이 HIGH 상태로 유지된다.3 NACK은 다음과 같은 상황에서 발생할 수 있다:
    1. 마스터가 호출한 주소에 해당하는 슬레이브가 버스에 없을 경우.
    2. 슬레이브가 현재 다른 작업을 처리 중이어서 통신할 준비가 되지 않았을 경우.
    3. 슬레이브가 전송된 데이터나 명령을 이해하지 못했을 경우.
    4. 마스터가 슬레이브로부터 데이터를 읽는(Read) 동작을 할 때, 마지막 데이터를 수신한 후 더 이상 데이터가 필요 없음을 알리기 위해 의도적으로 NACK을 보낼 경우.18


I2C 통신의 전체 과정은 이러한 기본 요소들의 조합으로 이루어진다.

- **쓰기(Write) 동작 시퀀스**: 마스터가 슬레이브의 특정 레지스터에 데이터를 쓰는 가장 일반적인 시퀀스는 다음과 같다.10
  1. ``: 마스터가 통신 시작을 알린다.
  2. ``: 마스터가 대상 슬레이브 주소와 쓰기 비트(0)를 전송한다.
  3. `[ACK]`: 주소가 일치하는 슬레이브가 응답한다.
  4. `[레지스터 주소]`: 마스터가 데이터를 쓸 내부 레지스터 주소를 전송한다.
  5. `[ACK]`: 슬레이브가 레지스터 주소를 수신했음을 응답한다.
  6. `[데이터]`: 마스터가 실제 데이터를 전송한다.
  7. `[ACK]`: 슬레이브가 데이터를 수신했음을 응답한다. (여러 바이트를 쓸 경우 이 과정이 반복된다.)
  8. ``: 마스터가 통신 종료를 알린다.
- **읽기(Read) 동작 시퀀스**: 마스터가 슬레이브의 특정 레지스터에서 데이터를 읽는 시퀀스는 쓰기 동작보다 조금 더 복잡하며, Repeated START를 활용한다.8
  1. ``: 마스터가 통신 시작을 알린다.
  2. ``: 먼저 쓰기 모드로 진입하여 읽고자 하는 레지스터를 지정해야 한다.
  3. `[ACK]`: 슬레이브가 응답한다.
  4. `[읽을 레지스터 주소]`: 마스터가 데이터를 읽어올 레지스터 주소를 전송한다.
  5. `[ACK]`: 슬레이브가 응답한다.
  6. ``: 마스터가 버스 제어권을 유지한 채 통신 방향을 전환하기 위해 Repeated START를 보낸다.
  7. ``: 마스터가 동일 슬레이브 주소와 읽기 비트(1)를 전송한다.
  8. `[ACK]`: 슬레이브가 읽기 요청에 응답하고, 이제부터 데이터 송신자 역할을 한다.
  9. `[데이터 수신]`: 슬레이브가 지정된 레지스터의 데이터를 전송하고, 마스터는 이를 수신한다.
  10. `[ACK]`: 마스터가 데이터를 잘 받았고, 다음 데이터를 계속 받기를 원함을 알린다. (여러 바이트를 읽을 경우 이 과정이 반복된다.)
  11. `[마지막 데이터 수신]`: 마스터가 마지막으로 원하는 데이터를 수신한다.
  12. `[NACK]`: 마스터가 더 이상 데이터를 받지 않겠다는 의미로 NACK을 보낸다.
  13. ``: 마스터가 통신 종료를 알린다.


I2C는 단순한 데이터 전송을 넘어, 실제 복잡한 임베디드 환경에서 발생할 수 있는 다양한 상황에 대응하기 위한 정교한 고급 기능들을 포함하고 있다. 클럭 스트레칭과 다중 마스터 중재는 I2C가 이기종 장치 간의 속도 차이를 극복하고, 여러 제어 장치가 하나의 버스를 효율적으로 공유할 수 있게 하는 핵심 메커니즘이다. 이러한 기능들은 I2C의 비대칭적 유연성을 보여주며, 마스터 주도형 프로토콜의 한계를 넘어 실용적인 강건함을 제공한다.


- **정의**: 클럭 스트레칭은 슬레이브 장치가 마스터의 클록 속도를 따라가지 못할 때, SCL 라인을 강제로 LOW 상태로 유지하여 데이터 전송을 일시적으로 중지시키는 기능이다.12 이는 슬레이브가 데이터 처리(예: 센서 값 측정 완료, EEPROM 쓰기 완료)나 다음 전송 데이터 준비를 위해 추가적인 시간을 확보할 수 있도록 한다.14 이는 마스터의 일방적인 통제에 대한 슬레이브의 '수동적 저항'으로, 시스템 내 다양한 성능을 가진 장치들의 공존을 가능하게 하는 중요한 유연성 확보 장치이다.

- **동작 원리**: 클럭 스트레칭은 I2C의 오픈 드레인 구조와 'Wired-AND' 특성에 기반한다.

  1. 정상적인 통신 중 마스터는 SCL 클록을 생성하며 HIGH와 LOW를 반복한다.
  2. 슬레이브가 추가 처리 시간이 필요하다고 판단하면(예: 수신한 바이트를 처리한 후), 마스터가 다음 클록을 위해 SCL 라인을 HIGH로 해제하려는 시점에도 불구하고 슬레이브는 자신의 SCL 핀을 계속 LOW 상태로 유지한다.19
  3. 'Wired-AND' 논리에 따라, 슬레이브가 라인을 LOW로 잡고 있는 한 전체 SCL 라인은 LOW 상태를 유지한다.
  4. 마스터는 SCL 라인을 HIGH로 만들려고 시도했지만, 라인이 여전히 LOW임을 감지하고 슬레이브가 라인을 해제하여 HIGH가 될 때까지 대기 상태에 들어간다.20
  5. 슬레이브가 준비를 마치면 SCL 라인을 해제하고, 그제서야 풀업 저항에 의해 라인이 HIGH로 상승하며 통신이 재개된다.

- **구현 및 주의사항**: 클럭 스트레칭은 주로 바이트 수신 후 ACK를 보내기 전이나, 읽기 요청에 대한 데이터를 전송하기 전에 발생한다.31 NXP의 LPI2C 모듈과 같은 고급 I2C 컨트롤러는 주소 수신 후, 데이터 송/수신 전 등 다양한 시점에서 클럭 스트레칭을 정교하게 제어할 수 있는 기능을 제공한다.30

  그러나 모든 마스터 장치가 클럭 스트레칭을 올바르게 지원하는 것은 아니다. 일부 간단한 I2C 구현이나 비트뱅잉(bit-banging) 방식의 마스터는 SCL 라인의 상태를 확인하지 않고 일방적으로 클록을 출력하려 할 수 있어 통신 오류를 유발할 수 있다. 특히 라즈베리 파이에 사용된 BCM2835 SoC는 클럭 스트레칭 요청을 놓치는 하드웨어 버그가 있는 것으로 알려져 있어, 해당 기능을 사용하는 데 제약이 따를 수 있다.32 따라서 클럭 스트레칭을 사용하는 시스템을 설계할 때는 마스터와 슬레이브 모두 해당 기능을 완벽하게 지원하는지 반드시 확인해야 한다.


- **개요**: I2C 버스는 여러 개의 마스터 장치가 하나의 버스를 공유할 수 있도록 설계되었다.2 다중 마스터 중재는 둘 이상의 마스터가 거의 동시에 버스 사용을 시도할 때, 데이터의 충돌이나 손상 없이 단 하나의 마스터만이 버스 제어권을 획득하도록 보장하는 절차이다.14 이는 시스템의 확장성과 복잡한 상호작용을 가능하게 하는 일종의 '민주적 절차'에 비유할 수 있다.
- **동작 원리**: 중재 과정 역시 오픈 드레인 구조와 'Wired-AND' 특성을 기반으로 비파괴적인(non-destructive) 방식으로 이루어진다.
  1. 버스가 유휴 상태일 때, 두 개 이상의 마스터가 거의 동시에 START 조건을 생성하고 데이터 전송을 시작할 수 있다.
  2. 각 마스터는 SDA 라인에 비트를 전송하면서, 동시에 SDA 라인의 실제 전압 레벨을 지속적으로 모니터링한다.20
  3. 두 마스터가 전송하는 데이터 비트가 동일한 동안에는(예: 동일한 슬레이브 주소를 호출하는 경우) 아무런 문제 없이 통신이 진행되며, 두 마스터 모두 자신이 버스를 제어하고 있다고 믿는다.33
  4. 어느 시점에서 한 마스터는 HIGH('1')를 전송(라인을 해제)하고 다른 마스터는 LOW('0')를 전송(라인을 GND로 당김)하는 순간이 발생한다.
  5. 'Wired-AND' 원리에 따라 버스의 실제 상태는 LOW가 된다.
  6. HIGH를 전송하려던 마스터는 자신이 의도한 레벨과 버스의 실제 레벨이 다름을 감지한다. 이는 다른 마스터가 버스를 사용 중임을 의미하므로, 이 마스터는 즉시 중재에서 패배했음을 인지하고 데이터 전송을 중단하며 SDA와 SCL 라인에서 손을 뗀다. 이후 이 마스터는 다른 마스터의 통신이 끝날 때까지 슬레이브 모드처럼 동작하며 버스를 감시한다.33
  7. LOW를 전송했던 마스터는 자신이 의도한 값과 버스의 실제 값이 일치하므로, 다른 마스터가 경쟁 중이었음을 전혀 인지하지 못한 채 자신의 통신을 중단 없이 계속 진행한다.20 이 과정에서 승리한 마스터의 데이터는 전혀 손상되지 않는다.
- **클럭 동기화 (Clock Synchronization)**: 중재 과정에서는 SDA 라인뿐만 아니라 SCL 라인도 동기화된다. 여러 마스터가 각자의 클록을 생성할 때, SCL 라인 역시 'Wired-AND'로 연결되어 있다. 따라서 한 마스터라도 SCL을 LOW로 당기면 라인은 LOW가 된다. 결과적으로 생성되는 통합 클록은 모든 마스터 클록의 LOW 기간 중 가장 긴 기간과 HIGH 기간 중 가장 짧은 기간을 따르게 된다.20 만약 두 마스터의 클록 속도가 다르다면, 일반적으로 더 느린 클록을 가진 마스터가 SCL을 더 오랫동안 LOW로 유지하므로 중재에서 승리할 가능성이 높다.34

이러한 클럭 스트레칭과 다중 마스터 중재 기능은 I2C가 단순한 점대점 통신 프로토콜을 넘어, 복잡하고 동적인 실제 임베디드 환경에서 매우 강건하고 유연하게 동작할 수 있도록 만드는 핵심적인 요소이다.


I2C 프로토콜은 시대의 요구에 부응하며 지속적으로 발전해왔다. 특히 통신 속도는 임베디드 시스템의 성능 향상과 맞물려 꾸준히 개선되었으며, 이는 다양한 속도 모드(Speed Mode)의 등장으로 이어졌다. 각 모드는 최대 전송 속도뿐만 아니라 전기적 사양, 타이밍 요구사항, 호환성 등에서 차이를 보이므로, 시스템 설계자는 애플리케이션의 요구사항에 맞춰 최적의 모드를 선택해야 한다.


I2C는 1982년 최초 등장 시 최대 100 kbit/s의 속도를 지원하는 **Standard-mode(Sm)**로 시작되었다.2 1992년, 마이크로컨트롤러와 주변 장치의 성능이 향상됨에 따라 최대 400 kbit/s를 지원하는 **Fast-mode(Fm)**가 도입되었다.35 이후 더 높은 대역폭에 대한 요구가 증가하면서 1 Mbit/s의 **Fast-mode Plus(Fm+)**, 3.4 Mbit/s의 **High-speed mode(Hs-mode)**, 그리고 5 Mbit/s의 단방향 통신을 지원하는 **Ultra-fast mode(UFm)**가 순차적으로 표준에 추가되었다.2


각 속도 모드는 단순히 클록 주파수를 높이는 것을 넘어, 그 속도를 안정적으로 구현하기 위한 고유한 전기적, 논리적 특징을 가진다.

- **Standard-mode (Sm)**: 최대 100 kbit/s의 속도를 지원하는 I2C의 기본 모드이다. 가장 넓은 호환성을 가지며, 대부분의 I2C 장치에서 지원된다.2

- **Fast-mode (Fm)**: 최대 400 kbit/s의 속도를 지원한다. Standard-mode와 프로토콜 구조는 동일하지만, 상승/하강 시간, 셋업/홀드 시간 등 타이밍 파라미터를 더 엄격하게 규정하여 속도를 높였다.2 입력단에 스파이크 필터와 슈미트 트리거를 내장하여 신호 무결성을 강화한 장치들이 많다.36

- **Fast-mode Plus (Fm+)**: 최대 1 Mbit/s의 속도를 지원한다. Fast-mode보다 더 강력한 출력 드라이버(최대 20-30 mA 싱크)와 더 낮은 값의 풀업 저항 사용을 전제로 한다. 이를 통해 더 빠른 상승/하강 시간을 달성하고, 최대 허용 버스 커패시턴스를 400 pF에서 550 pF로 늘려 더 많은 장치를 연결하거나 더 긴 버스를 구성할 수 있게 한다.2

- **High-speed mode (Hs-mode)**: 최대 3.4 Mbit/s의 고속 통신을 지원한다. 기존의 오픈 드레인 방식만으로는 이 속도를 달성하기 어려워, 마스터가 SCL 라인에 전류 소스를 이용한 **활성 풀업(active pull-up)**을 사용하여 상승 시간을 강제로 단축시킨다.2 Hs-mode 통신을 시작하기 위해서는, 먼저 Standard-mode나 Fast-mode에서 

  `00001xxx` 형태의 **마스터 코드(master code)**를 전송하여 버스 상의 Hs-mode 지원 슬레이브들을 고속 모드로 전환시켜야 한다. 이 마스터 코드는 일반 슬레이브 주소와 다르므로 다른 장치들은 이를 무시한다. 이 과정에서 버스 중재가 완료되므로, 실제 고속 데이터 전송 구간에서는 중재나 클럭 스트레칭이 허용되지 않는다.2

- **Ultra-fast mode (UFm)**: 최대 5 Mbit/s의 초고속 전송을 지원하지만, 이는 **단방향(unidirectional) 쓰기 전용** 모드이다.36 속도 극대화를 위해 기존 I2C의 여러 특징을 과감히 생략했다. ACK/NACK, 클럭 스트레칭, 중재 기능이 없으며, 오픈 드레인 대신 **푸시-풀(push-pull)** 출력 방식을 사용하여 풀업 저항 없이도 빠른 신호 전환을 구현한다.2 이로 인해 다른 모드와는 전기적으로 호환되지 않는다. 주로 LED 디스플레이 제어와 같이 데이터 피드백이 필요 없고, 전송 오류가 치명적이지 않은 애플리케이션에 사용된다.2


다양한 속도 모드를 지원하는 장치들을 하나의 버스에 혼용할 경우, 호환성 문제를 신중하게 고려해야 한다.

- **하위 호환성**: Standard-mode, Fast-mode, Fast-mode Plus, High-speed mode는 기본적으로 하위 호환성을 가진다. 즉, 더 빠른 모드를 지원하는 장치는 더 느린 모드의 버스에서도 정상적으로 동작할 수 있다.36 예를 들어, Fm+ 장치는 Sm 버스에서 100 kbit/s로 통신할 수 있다.
- **상위 호환성 부재**: 반대로, 느린 모드만 지원하는 장치를 빠른 모드의 버스에 연결하는 것은 위험하다. Standard-mode 장치는 Fast-mode의 빠른 클록 속도를 따라가지 못해 데이터를 놓치거나 오동작하여 버스 전체를 예측 불가능한 상태로 만들 수 있다.36
- **설계 시 고려사항**:
  - 혼합 속도 버스를 설계할 때는 버스 전체의 속도를 가장 느린 장치에 맞춰야 한다.
  - 만약 서로 다른 속도 그룹을 분리해야 할 경우, I2C 멀티플렉서나 스위치를 사용하여 버스를 물리적으로 분할하는 방법을 고려할 수 있다.43
  - 통신 속도가 높아질수록 풀업 저항 값 선정, 버스 커패시턴스 관리, PCB 트레이스 길이 및 라우팅 등 신호 무결성에 대한 고려가 훨씬 더 중요해진다.29

다음 표는 각 I2C 통신 속도 모드의 주요 특징을 요약하여 비교한 것이다.

| 특징                     | Standard-mode (Sm) | Fast-mode (Fm) | Fast-mode Plus (Fm+)     | High-speed mode (Hs)       | Ultra-fast mode (UFm)  |
| ------------------------ | ------------------ | -------------- | ------------------------ | -------------------------- | ---------------------- |
| **최대 속도**            | 100 kbit/s         | 400 kbit/s     | 1 Mbit/s                 | 3.4 Mbit/s                 | 5 Mbit/s               |
| **방향성**               | 양방향             | 양방향         | 양방향                   | 양방향                     | 단방향 (Write-only)    |
| **드라이버 방식**        | 오픈 드레인        | 오픈 드레인    | 오픈 드레인 (강화)       | 오픈 드레인 + 활성 풀업    | 푸시-풀                |
| **최대 버스 커패시턴스** | 400 pF             | 400 pF         | 550 pF                   | 100 pF (Hs) / 400 pF (F/S) | 명시 안됨              |
| **주요 특징**            | 기본 표준          | 타이밍 강화    | 드라이버 강화, 버스 확장 | 마스터 코드 필요           | ACK/중재/스트레칭 없음 |
| **하위 호환성**          | -                  | Sm 호환        | Fm/Sm 호환               | Fm+/Fm/Sm 호환             | 없음                   |


I2C는 임베디드 시스템에서 널리 사용되는 통신 프로토콜이지만, 유일한 선택지는 아니다. 시스템 설계자는 종종 SPI(Serial Peripheral Interface), UART(Universal Asynchronous Receiver-Transmitter)와 같은 다른 직렬 통신 프로토콜들과 I2C를 비교하여 특정 애플리케이션의 요구사항에 가장 적합한 방식을 선택해야 한다. 각 프로토콜은 핀 수, 속도, 통신 방식, 토폴로지 등에서 명확한 장단점을 가지므로, 이들의 차이점을 이해하는 것은 효율적인 시스템 설계를 위한 필수적인 과정이다.


- **I2C (Inter-Integrated Circuit)**: 2개의 선(SDA, SCL)을 사용하는 **동기식, 반이중, 다중 마스터/다중 슬레이브** 버스 프로토콜이다. 주소 기반으로 슬레이브를 선택하며, 핀 수 절약과 다수의 장치를 간단하게 연결하는 데 최적화되어 있다.44
- **SPI (Serial Peripheral Interface)**: 최소 4개의 선(MOSI, MISO, SCLK, SS)을 사용하는 **동기식, 전이중** 통신 프로토콜이다. 개별적인 칩 선택(SS, Slave Select) 라인을 통해 슬레이브를 선택하며, 고속의 연속적인 데이터 스트리밍에 매우 강력한 성능을 보인다.46
- **UART (Universal Asynchronous Receiver-Transmitter)**: 2개의 선(RX, TX)을 사용하는 **비동기식, 전이중** 통신 프로토콜이다. 클록 라인 없이, 미리 약속된 속도(Baudrate)와 시작/정지 비트를 사용하여 데이터를 교환한다. 주로 두 장치 간의 점대점(point-to-point) 통신이나 장거리 통신에 사용된다.46


각 프로토콜의 특성을 핵심 지표별로 상세히 비교하면 그 차이점이 더욱 명확해진다.

- **핀 수 (Wiring)**:
  - **I2C**: 2개. 핀 효율성이 가장 뛰어나다. 슬레이브 수가 늘어나도 핀 수는 변하지 않는다.49
  - **UART**: 2개. 점대점 통신에 한정된다.50
  - **SPI**: 최소 4개. 슬레이브가 하나 추가될 때마다 마스터의 SS 핀이 하나씩 더 필요하므로(4+N), 많은 수의 슬레이브를 연결할 경우 하드웨어 복잡성이 급격히 증가한다.44
- **속도 (Speed)**:
  - **SPI**: 일반적으로 가장 빠르다. 수십 MHz 이상의 클록 속도를 지원하며, 프로토콜 오버헤드가 적어 높은 데이터 처리량을 보인다.46
  - **I2C**: SPI보다 느리다. 풀업 저항에 의존하는 오픈 드레인 구조와 주소 지정, ACK/NACK 등의 프로토콜 오버헤드가 속도를 제한한다. High-speed mode에서 3.4 Mbps까지 가능하지만, 일반적인 SPI 속도에는 미치지 못한다.44
  - **UART**: 일반적으로 가장 느리다. 비동기 방식의 한계와 시작/정지 비트 오버헤드로 인해 최대 속도가 수 Mbps 수준으로 제한된다.48
- **통신 방식 (Duplex)**:
  - **SPI, UART**: 데이터 송신 라인(MOSI/TX)과 수신 라인(MISO/RX)이 분리되어 있어 **전이중(Full-duplex)** 통신이 가능하다. 즉, 송신과 수신이 동시에 일어날 수 있다.46
  - **I2C**: 단일 데이터 라인(SDA)을 공유하므로 **반이중(Half-duplex)** 통신만 가능하다.49
- **장치 연결 및 토폴로지 (Topology)**:
  - **I2C**: 진정한 **다중 장치 버스 구조**를 가진다. 여러 마스터와 여러 슬레이브가 동일한 2개의 선에 병렬로 연결된다.55
  - **SPI**: 마스터를 중심으로 각 슬레이브가 개별 SS 라인으로 연결되는 **스타(Star) 구조**가 일반적이다.
  - **UART**: 두 장치가 1:1로 직접 연결되는 **점대점(Point-to-Point) 구조**이다.56
- **오류 처리 및 신뢰성 (Error Handling & Reliability)**:
  - **I2C**: 각 바이트 전송 후 수신 측이 보내는 **ACK/NACK** 신호를 통해 데이터 수신 여부를 즉시 확인할 수 있어 프로토콜 수준에서 높은 신뢰성을 제공한다.46
  - **UART**: 선택적으로 **패리티 비트(Parity bit)**를 사용하여 단일 비트 오류를 검출할 수 있다.44
  - **SPI**: 프로토콜 자체에는 데이터 수신 확인이나 오류 검출 메커니즘이 내장되어 있지 않다. 신뢰성 확보를 위해서는 상위 소프트웨어 계층에서 별도의 핸드셰이킹(handshaking)이나 CRC(Cyclic Redundancy Check)와 같은 방법을 구현해야 한다.44

다음 표는 I2C, SPI, UART 프로토콜의 주요 특징을 종합적으로 비교하여 보여준다. 이 표를 통해 설계자는 특정 애플리케이션의 제약 조건(예: 핀 수, 요구 속도, 연결 장치 수)에 따라 가장 적합한 통신 프로토콜을 전략적으로 선택할 수 있다.

| 특징                  | I2C (Inter-Integrated Circuit)          | SPI (Serial Peripheral Interface)          | UART (Universal Asynchronous R/T) |
| --------------------- | --------------------------------------- | ------------------------------------------ | --------------------------------- |
| **핀 수**             | 2 (SDA, SCL)                            | 4+N (MOSI, MISO, SCLK, SS)                 | 2 (RX, TX)                        |
| **통신 방식**         | 동기식, 반이중                          | 동기식, 전이중                             | 비동기식, 전이중                  |
| **속도**              | 상대적으로 느림 (최대 5 Mbps)           | 매우 빠름 (수십 Mbps 이상)                 | 느림 (최대 ~5 Mbps)               |
| **장치 연결**         | 다중 마스터, 다중 슬레이브              | 단일 마스터, 다중 슬레이브                 | 점대점 (1:1)                      |
| **슬레이브 선택**     | 7/10비트 주소                           | 개별 SS(Slave Select) 라인                 | 없음 (직접 연결)                  |
| **하드웨어 복잡성**   | 낮음 (풀업 저항 필요)                   | 높음 (슬레이브 수에 따라 핀 증가)          | 낮음                              |
| **소프트웨어 복잡성** | 중간 (주소/ACK 처리 필요)               | 낮음 (프로토콜 오버헤드 적음)              | 낮음                              |
| **장점**              | 핀 효율성, 다중 장치 연결 용이          | 고속, 전이중, 단순한 프로토콜              | 단순함, 장거리 가능, 클록 불필요  |
| **단점**              | 속도 제한, 반이중, 버스 커패시턴스 민감 | 많은 핀 필요, 프로토콜 수준 오류 확인 부재 | 속도 제한, 1:1 통신만 가능        |


I2C 프로토콜의 이론적 이해를 넘어, 실제 임베디드 시스템에서 어떻게 구현되고 활용되는지 살펴보는 것은 매우 중요하다. 본 장에서는 대표적인 개발 플랫폼인 아두이노(Arduino)와 라즈베리 파이(Raspberry Pi)에서의 I2C 구현 방법을 알아보고, 널리 사용되는 센서 및 주변 장치와의 연동 사례를 통해 I2C의 실용성을 구체적으로 탐구한다.


- 아두이노 (Arduino):

  아두이노 환경에서 I2C 통신은 내장된 Wire.h 라이브러리를 통해 매우 직관적으로 구현할 수 있다.57 이 라이브러리는 복잡한 I2C 프로토콜의 저수준(low-level) 동작들을 추상화하여 사용자가 쉽게 마스터 또는 슬레이브 장치를 구현할 수 있도록 돕는다.

  - **핀 구성**: 대부분의 아두이노 보드(Uno, Nano 등)에서 I2C 핀은 아날로그 핀 A4(SDA)와 A5(SCL)에 할당되어 있다. Mega나 Due와 같은 보드는 별도의 SDA/SCL 핀(디지털 20, 21)을 가지고 있다.57
  - **주요 함수**:
    - `Wire.begin()`: I2C 버스를 초기화하고, 선택적으로 슬레이브 주소를 인자로 전달하여 슬레이브 모드로 동작하게 한다.
    - `Wire.beginTransmission(address)`: 지정된 주소의 슬레이브와 통신을 시작한다.
    - `Wire.write(data)`: 전송 버퍼에 데이터를 쓴다.
    - `Wire.endTransmission()`: 버퍼의 데이터를 실제로 전송하고 통신을 종료한다.
    - `Wire.requestFrom(address, byteCount)`: 지정된 주소의 슬레이브로부터 특정 바이트 수만큼의 데이터를 요청한다.
    - `Wire.read()`: 슬레이브로부터 수신한 데이터를 읽는다.57

- 라즈베리 파이 (Raspberry Pi):

  리눅스 기반의 싱글 보드 컴퓨터인 라즈베리 파이에서도 I2C 통신을 쉽게 활성화하고 사용할 수 있다.

  - **인터페이스 활성화**: 기본적으로 I2C 인터페이스는 비활성화되어 있으므로, 터미널에서 `sudo raspi-config` 명령을 실행하여 'Interfacing Options' 메뉴에서 I2C를 활성화해야 한다.60
  - **장치 스캔**: I2C 인터페이스가 활성화되면, `i2c-tools` 패키지를 설치(`sudo apt-get install i2c-tools`)한 후 `i2cdetect -y 1` (또는 구형 모델의 경우 `i2cdetect -y 0`) 명령을 사용하여 버스에 연결된 장치들의 주소를 스캔할 수 있다.61 이는 하드웨어 연결 상태를 확인하는 데 매우 유용하다.
  - **파이썬 프로그래밍**: 파이썬에서는 `smbus` (또는 `smbus2`) 라이브러리를 사용하여 I2C 장치를 제어하는 것이 일반적이다. 이 라이브러리는 I2C 버스에 바이트나 워드 단위로 데이터를 읽고 쓰는 간단한 함수들을 제공한다.60


I2C는 다양한 센서, 메모리, 실시간 클록 등 저속 주변 장치들과 마이크로컨트롤러를 연결하는 데 널리 사용된다.

- **MPU-6050 (3축 가속도 및 3축 자이로 센서)**:
  - **개요**: 가속도와 각속도를 측정하는 6축 관성 측정 장치(IMU)로, 로봇의 자세 제어, 드론, 모션 인식 등 다양한 분야에 활용된다. I2C 인터페이스를 통해 센서 데이터를 쉽게 읽어올 수 있다.59
  - **아두이노 구현**: VCC, GND, SDA(A4), SCL(A5) 핀을 연결하고, 'Adafruit MPU6050'과 같은 라이브러리를 사용하면 복잡한 레지스터 설정을 직접 하지 않고도 가속도($g$)와 각속도(°/s) 값을 쉽게 얻을 수 있다.64 코드는 센서를 초기화하고, 측정 범위를 설정한 뒤, 루프 내에서 지속적으로 센서 값을 읽어 시리얼 모니터에 출력하는 형태로 구성된다.59
  - **라즈베리 파이 구현**: 라즈베리 파이의 3.3V, GND, GPIO 2(SDA), GPIO 3(SCL)에 연결한다. `mpu6050-raspberrypi`와 같은 파이썬 라이브러리를 설치하면 간단한 코드로 센서 데이터를 딕셔너리 형태로 읽어올 수 있다.60
- **24LC256 (EEPROM)**:
  - **개요**: 256 Kbit(32 Kbyte) 용량의 비휘발성 메모리로, 전원이 꺼져도 데이터를 유지해야 하는 설정값, 로그 데이터 등을 저장하는 데 사용된다. I2C를 통해 특정 주소에 데이터를 읽고 쓸 수 있다.66
  - **아두이노 구현**: 24LC256은 64바이트 페이지 단위로 쓰기 동작이 이루어진다. 바이트 단위 쓰기 함수와 페이지 단위 쓰기 함수를 직접 구현하거나, 기존 라이브러리를 활용할 수 있다.67 주소 핀(A0, A1, A2)을 접지하면 기본 I2C 주소는 0x50이 된다.68
  - **라즈베리 파이 구현**: 하드웨어 연결 후, 리눅스 커널에서 `at24` 드라이버를 로드하도록 설정하면, EEPROM이 `/sys/class/i2c-adapter/i2c-1/1-0050/eeprom`과 같은 경로의 파일 시스템으로 마운트된다. 이후 `echo`와 `cat` 또는 `hexdump`와 같은 표준 리눅스 명령어를 사용하여 파일에 데이터를 쓰거나 읽는 것처럼 EEPROM에 접근할 수 있다.69
- **DS1307 (실시간 클록, RTC)**:
  - **개요**: 코인 셀 배터리를 통해 주 전원이 꺼져도 현재 시간을 계속 유지하는 장치이다. 데이터 로깅, 예약 작업 등 정확한 시간 정보가 필요한 프로젝트에 필수적이다. DS1307은 5V에서 동작하는 가장 대중적인 RTC 중 하나이다.58
  - **아두이노 구현**: 'Adafruit RTClib' 라이브러리를 사용하는 것이 일반적이다. 이 라이브러리는 컴파일 시점의 PC 시간을 RTC에 설정하거나, 수동으로 시간을 설정하고, 현재 시간을 `DateTime` 객체 형태로 읽어오는 편리한 함수들을 제공한다.58
  - **라즈베리 파이 구현**: 라즈베리 파이는 내장 RTC가 없어 인터넷(NTP 서버)을 통해 시간을 동기화한다. 인터넷 연결이 없는 환경에서는 외부 RTC가 필수적이다. `/boot/firmware/config.txt` 파일에 `dtoverlay=i2c-rtc,ds1307`을 추가하여 부팅 시 커널이 DS1307 드라이버를 로드하도록 설정한다. 이후 `hwclock` 명령어를 사용하여 시스템 시간을 RTC에 쓰거나(`hwclock -w`), RTC의 시간을 시스템 시간으로 읽어올 수 있다(`hwclock -s`).62
- **LM75 (온도 센서)**:
  - **개요**: I2C 인터페이스를 가진 디지털 온도 센서로, 0.125°C의 해상도를 제공한다. 시스템의 온도 모니터링, 팬 제어, 과열 방지 등에 사용된다.73
  - **아두이노 구현**: `LM75.h`와 같은 라이브러리를 사용하면 센서의 I2C 주소(기본 0x48)를 지정하고 `getTemp()`와 같은 함수를 호출하여 간단하게 섭씨 온도를 부동소수점 값으로 얻을 수 있다.75 코드는 `Wire.h` 라이브러리를 포함하고, 시리얼 통신을 초기화한 뒤, 루프에서 주기적으로 온도를 읽어 출력하는 구조를 가진다.75


I2C는 널리 사용되고 비교적 간단한 프로토콜이지만, 실제 구현 과정에서는 다양한 문제에 직면할 수 있다. 특히 I2C의 물리 계층은 오픈 드레인 구조와 풀업 저항에 의존하기 때문에 시스템 구성에 매우 민감하다. 장치가 하나 추가되거나 배선이 길어질 때마다 버스 커패시턴스가 변하고, 이는 신호 품질에 직접적인 영향을 미친다. 따라서 I2C 디버깅의 첫걸음은 프로토콜의 논리적 오류를 의심하기보다, 물리 계층의 전기적 신호가 건전한지를 먼저 확인하는 것이 가장 효율적인 접근법이다.


I2C 통신 실패의 대부분은 기본적인 하드웨어 설정 오류에서 비롯된다.43 문제가 발생했을 때 가장 먼저 확인해야 할 사항은 다음과 같다.

- **배선 오류**: SDA와 SCL 라인이 서로 바뀌어 연결되지 않았는지, VCC와 GND가 올바르게 연결되었는지 확인한다. 이는 가장 흔하면서도 가장 쉽게 해결할 수 있는 문제이다.77
- **풀업 저항 누락 또는 부적절한 값**: SDA와 SCL 라인에 풀업 저항이 연결되어 있는지 확인한다. 저항이 없으면 라인이 HIGH 상태를 유지할 수 없다. 저항값이 너무 크면 고속에서 신호 상승 시간이 길어져 문제가 되고, 너무 작으면 장치의 싱크 전류 용량을 초과할 수 있다.43
- **주소 불일치**: 마스터가 호출하는 슬레이브 주소가 실제 장치의 주소와 일치하는지 확인한다. 데이터시트를 통해 기본 주소를 확인하고, 주소 설정 핀(A0, A1, A2)이 올바르게 연결되었는지 점검한다.43
- **전원 및 접지 문제**: 버스에 연결된 모든 장치가 동일한 전압 레벨에서 동작하는지, 그리고 공통 접지(common ground)로 연결되어 있는지 확인한다. 특히 여러 보드를 케이블로 연결할 때 접지 레벨 차이로 인해 통신 오류가 발생할 수 있다.43


하나의 I2C 버스에 동일한 주소를 가진 슬레이브 장치가 두 개 이상 연결되면 주소 충돌이 발생하여 정상적인 통신이 불가능하다.80 이 문제를 해결하기 위한 방법은 다음과 같다.

- **I2C 멀티플렉서/스위치 사용**: TCA9548A와 같은 I2C 멀티플렉서 IC를 사용하는 것이 가장 일반적인 해결책이다. 마스터는 먼저 멀티플렉서의 주소로 명령을 보내 원하는 채널(하위 버스)을 선택한다. 그러면 마스터의 I2C 버스가 선택된 채널에만 물리적으로 연결되어, 해당 채널에 연결된 슬레이브와 충돌 없이 통신할 수 있다.43
- **장치 주소 변경**: 많은 I2C 장치들은 하드웨어 핀(A0, A1, A2 등)의 HIGH/LOW 상태를 조합하여 주소의 일부를 변경할 수 있는 기능을 제공한다. 이를 통해 동일한 종류의 장치를 최대 8개까지 하나의 버스에 연결할 수 있다.43 일부 장치는 소프트웨어 명령을 통해 I2C 주소를 동적으로 변경하는 기능을 지원하기도 한다.83
- **다중 I2C 버스 사용**: 마이크로컨트롤러가 두 개 이상의 독립적인 I2C 포트를 지원하는 경우, 주소가 충돌하는 장치들을 서로 다른 버스에 분리하여 연결하는 것이 가장 간단하고 확실한 방법이다.82


오실로스코프나 로직 분석기로 파형을 관찰했을 때, 신호가 깨끗한 사각파가 아닌 왜곡된 형태로 나타나는 경우 신호 무결성 문제를 의심해야 한다.

- **느린 상승 시간**: 파형의 상승 에지가 완만하게 나타나는 현상. 주된 원인은 버스 커패시턴스가 높거나 풀업 저항값이 너무 큰 것이다. 해결책은 더 작은 값(더 강한)의 풀업 저항으로 교체하거나, 버스 길이를 줄이고 연결된 장치 수를 줄여 커패시턴스를 낮추는 것이다. 또는 I2C 버퍼/리피터를 사용하여 버스를 분리할 수도 있다.27
- **크로스토크 및 노이즈**: SDA와 SCL 라인이 서로에게 영향을 주거나, 모터와 같은 외부 노이즈 소스로 인해 신호에 글리치(glitch)나 스파이크가 나타나는 현상. 해결책으로는 SDA와 SCL 라인을 최대한 짧고 멀리 떨어뜨려 배선하고, 중요한 경우 두 라인 사이에 접지 트레이스를 배치하는 것이 좋다. 또한 저항-커패시터(RC) 필터를 추가하거나, 슈미트 트리거 입력이 있는 장치를 사용하여 노이즈 내성을 높일 수 있다.26
- **언더슛/오버슛 (Ringing)**: 신호의 에지가 급격하게 변할 때 전압이 목표 레벨을 지나쳤다가 다시 돌아오는 현상. 주로 긴 배선으로 인한 임피던스 불일치 및 반사 때문에 발생한다. SDA/SCL 라인에 수십 옴 정도의 작은 직렬 저항을 추가하여 에지 속도를 완만하게 만들면 완화될 수 있다.85
- **버스 잠김 (Bus Stuck/Hang)**: 통신 오류나 노이즈로 인해 슬레이브가 SDA 또는 SCL 라인을 LOW 상태로 계속 잡고 놓아주지 않는 상태. 이 경우 버스는 더 이상 동작하지 않는다. 소프트웨어적인 해결책으로, 마스터가 I2C 통신을 시작하기 전에 SDA 핀의 상태를 확인하고, 만약 LOW라면 SCL 핀을 수동으로 토글(최대 9번)하여 슬레이브가 내부 상태 머신을 리셋하고 라인을 해제하도록 유도하는 방법을 시도할 수 있다.87


단순한 전기적 문제를 넘어 프로토콜 수준의 오류를 분석하기 위해서는 로직 분석기가 매우 강력한 도구가 된다.88

- **분석 과정**:
  1. **연결 및 캡처**: 로직 분석기의 채널을 각각 SDA와 SCL 라인에 연결하고, 통신이 이루어지는 동안 신호를 캡처한다.
  2. **프로토콜 디코딩**: 대부분의 로직 분석기 소프트웨어는 I2C 프로토콜 디코더를 내장하고 있다. 캡처된 파형을 START, STOP, 주소, 데이터, ACK/NACK 등 의미 있는 프로토콜 요소로 자동 해석하여 보여준다.89
  3. **문제 식별**: 디코딩된 결과를 통해 다음과 같은 문제들을 직관적으로 파악할 수 있다.
     - **잘못된 주소**: 마스터가 의도하지 않은 주소를 보내고 있지는 않은가?
     - **NACK 발생**: 특정 슬레이브가 주소에 대해 NACK으로 응답하는가? 이는 주소 오류, 슬레이브의 비활성 상태 등을 의미한다. 데이터 전송 중간에 NACK이 발생했다면, 슬레이브가 데이터를 처리할 수 없거나 버퍼가 가득 찼을 수 있다.89
     - **타이밍 위반**: 셋업 시간, 홀드 시간 등 I2C 타이밍 규격을 만족하는지 측정하고 분석할 수 있다.
     - **예기치 않은 데이터**: 슬레이브가 예상과 다른 데이터를 보내거나, 마스터가 잘못된 데이터를 보내고 있는지 확인할 수 있다.

로직 분석기를 사용하면 눈에 보이지 않는 디지털 통신의 흐름을 시각화하여 문제의 근본 원인을 빠르고 정확하게 찾아낼 수 있다.



1982년에 탄생한 I2C 프로토콜이 40년이 넘는 세월 동안 수많은 기술적 변화 속에서도 살아남아 오늘날 임베디드 시스템의 표준 인터페이스로 굳건히 자리 잡은 이유는 그 설계 철학의 명확성과 실용성에 있다. I2C의 핵심 가치는 **'최소한의 자원으로 최대한의 연결성'**을 제공하는 데 있다. 단 두 개의 배선만으로 다수의 장치를 연결할 수 있는 탁월한 핀 효율성은 IC 패키지와 PCB 공간이 제한적인 현대의 소형화된 전자기기에서 여전히 강력한 장점으로 작용한다.

또한, 오픈 드레인 구조를 기반으로 한 중재 및 클럭 스트레칭과 같은 견고한 프로토콜 메커니즘은 다양한 성능과 제조사의 장치들이 하나의 버스에서 안정적으로 공존할 수 있는 기반을 제공했다. 이는 방대한 I2C 지원 장치 생태계를 구축하는 원동력이 되었으며, 개발자들은 수많은 센서, 메모리, 주변 장치들을 마치 레고 블록처럼 쉽게 시스템에 통합할 수 있게 되었다. 이러한 단순성, 효율성, 그리고 확장성이 I2C의 생명력을 유지하는 핵심 동력이다.


I2C는 모든 상황에 적합한 만능 해결책은 아니다. 그 특성을 정확히 이해하고 전략적으로 활용할 때 그 가치가 극대화된다.

- **최적의 적용 분야**: I2C는 고속 데이터 스트리밍보다는 다수의 저속 주변 장치를 제어하고 모니터링하는 데 가장 적합하다. 즉, 데이터 처리량(throughput)보다는 **연결성(connectivity)**이 중요한 애플리케이션에서 빛을 발한다. 온도 센서, 습도 센서, 가속도계, RTC, EEPROM, I/O 확장기, 소형 디스플레이, 전원 관리 IC(PMIC) 등은 I2C의 대표적인 활용 분야이다.
- **설계 시 핵심 고려사항**: I2C 기반 시스템을 성공적으로 설계하기 위해서는 프로토콜의 논리적 흐름뿐만 아니라 물리 계층의 특성을 반드시 고려해야 한다. 특히 **버스 총 커패시턴스**를 예측하고, 이를 기반으로 **적절한 풀업 저항 값**을 계산하는 과정은 설계 초기 단계에서부터 신중하게 이루어져야 한다. 이는 시스템의 안정성과 직결되는 문제이며, 많은 디버깅 시간을 절약해 줄 수 있다. 또한, 다중 마스터나 클럭 스트레칭과 같은 고급 기능을 사용할 경우, 버스에 연결된 모든 장치가 해당 기능을 완벽하게 지원하는지 데이터시트를 통해 검증하는 절차가 필수적이다.


I2C의 성공에도 불구하고, 더 높은 속도, 더 낮은 전력 소모, 그리고 더 향상된 기능에 대한 요구는 계속해서 증가해왔다. 이러한 요구에 부응하기 위해 MIPI Alliance는 I2C의 후속 표준으로 **I3C (Improved Inter-Integrated Circuit)**를 개발했다.36

I3C는 I2C의 핵심적인 장점인 2-wire 구조와 버스 토폴로지를 계승하면서도 여러 측면에서 획기적인 개선을 이루었다.

- **속도 향상**: 푸시-풀과 오픈 드레인을 혼합 사용하는 방식으로 표준 데이터 전송률을 12.5 Mbps까지 높였으며, 고속 모드에서는 더 높은 속도를 지원한다.
- **전력 효율 개선**: 오픈 드레인 방식보다 효율적인 푸시-풀 방식을 주로 사용하여 정적 전력 소모를 크게 줄였다.
- **인-밴드 인터럽트 (In-Band Interrupts)**: 기존 I2C에서는 인터럽트를 위해 별도의 핀이 필요했지만, I3C는 동일한 I2C 버스 라인을 통해 슬레이브가 마스터에게 인터럽트를 요청할 수 있는 기능을 제공하여 핀 수를 더욱 절약한다.
- **동적 주소 할당 (Dynamic Address Assignment)**: 마스터가 버스에 연결된 슬레이브들에게 동적으로 주소를 할당할 수 있어, 주소 충돌 문제를 근본적으로 해결하고 핫-플러깅(hot-plugging)을 용이하게 한다.

무엇보다 중요한 것은 I3C가 **I2C와의 하위 호환성**을 유지한다는 점이다.36 기존의 I2C 슬레이브 장치들은 약간의 제약 조건 하에서 I3C 버스에 함께 연결될 수 있다. 이는 I2C의 방대한 생태계를 점진적으로 I3C로 전환할 수 있는 현실적인 마이그레이션 경로를 제공한다.

결론적으로, I2C는 지난 수십 년간 임베디드 세계의 보이지 않는 혈관 역할을 충실히 수행해왔으며, 그 설계 철학은 여전히 유효하다. 앞으로 I3C가 점차 그 자리를 대체해 나가겠지만, I2C가 구축한 2-wire 버스 통신의 패러다임은 I3C를 통해 계속해서 진화하며 미래의 임베디드 시스템에서도 핵심적인 역할을 수행할 것이다.


1. I2C - 나무위키, 8월 10, 2025에 액세스, https://namu.wiki/w/I2C
2. I²C - Wikipedia, 8월 10, 2025에 액세스, [https://en.wikipedia.org/wiki/I%C2%B2C](https://en.wikipedia.org/wiki/I²C)
3. I2C란? (엄청 쉽게 설명) - 엔지니어스 - Engineeus - 티스토리, 8월 10, 2025에 액세스, https://mickael-k.tistory.com/184
4. I2C 통신(TWI)이란? - 공대누나의 일상과 전자공학, 8월 10, 2025에 액세스, https://gdnn.tistory.com/104
5. I²C - 위키백과, 우리 모두의 백과사전, 8월 10, 2025에 액세스, [https://ko.wikipedia.org/wiki/I%C2%B2C](https://ko.wikipedia.org/wiki/I²C)
6. All About the I2C Standard & Protocol. How I2C Works | Circuit Crush, 8월 10, 2025에 액세스, https://www.circuitcrush.com/i2c-tutorial/
7. I2C Communication Background - Total Phase, 8월 10, 2025에 액세스, https://www.totalphase.com/support/articles/200349156-i2c-background/
8. I2C Protocol (2-Wire Interface) in a nut shell - EmbedJournal, 8월 10, 2025에 액세스, https://embedjournal.com/two-wire-interface-i2c-protocol-in-a-nut-shell/
9. What is I2C? - everything RF, 8월 10, 2025에 액세스, https://www.everythingrf.com/community/what-is-i2c
10. 제일 쉽게 I2C를 알아보자, 8월 10, 2025에 액세스, [https://semiconwide.tistory.com/entry/%EC%A0%9C%EC%9D%BC-%EC%89%BD%EA%B2%8C-I2C%EB%A5%BC-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90](https://semiconwide.tistory.com/entry/제일-쉽게-I2C를-알아보자)
11. I2C 통신이란? - Mr.HB이야기 - 티스토리, 8월 10, 2025에 액세스, https://gudgud.tistory.com/28
12. I2C-BUS - 노땅엔진니어의 로봇 이야기 - 티스토리, 8월 10, 2025에 액세스, https://duvallee.tistory.com/4
13. I2C Communication Protocol - GeeksforGeeks, 8월 10, 2025에 액세스, https://www.geeksforgeeks.org/computer-organization-architecture/i2c-communication-protocol/
14. I2C Communication Protocol and How It Works - Latest News from Seeed Studio, 8월 10, 2025에 액세스, https://www.seeedstudio.com/blog/2022/09/02/i2c-communication-protocol-and-how-it-works/
15. I2C - SparkFun Learn, 8월 10, 2025에 액세스, https://learn.sparkfun.com/tutorials/i2c/all
16. Peripheral과의 통신 - 3. I2C - 코독코독 - 티스토리, 8월 10, 2025에 액세스, [https://coderdocument.tistory.com/entry/Peripheral%EA%B3%BC%EC%9D%98-%ED%86%B5%EC%8B%A0-3-I2C](https://coderdocument.tistory.com/entry/Peripheral과의-통신-3-I2C)
17. twi,iic,i2c 통신에서 풀업 저항값 변화에 따른 파형 - 토순이, 8월 10, 2025에 액세스, https://enng.tistory.com/56
18. Understanding the I2C Bus - Texas Instruments, 8월 10, 2025에 액세스, https://www.ti.com/lit/pdf/slva704
19. clock stretching - EEVblog, 8월 10, 2025에 액세스, https://www.eevblog.com/forum/microcontrollers/clock-stretching/
20. Introduction to I2C: Advanced topics | Video | TI.com, 8월 10, 2025에 액세스, https://www.ti.com/video/6243124608001
21. I2C (Inter-Integrated Circuit) - 정리 - 티스토리, 8월 10, 2025에 액세스, https://yjkim9635.tistory.com/5
22. [ Network ] 02. I2C통신에 관하여, 8월 10, 2025에 액세스, [https://coder-in-war.tistory.com/entry/Network-02-I2C%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC](https://coder-in-war.tistory.com/entry/Network-02-I2C에-관하여)
23. [ 풀업 저항 ] Pull up resistor 을 선정 방법 - 동그리의일상, 8월 10, 2025에 액세스, [https://donggreen.tistory.com/entry/Pull-up-resistor-%ED%92%80%EC%97%85-%EC%A0%80%ED%95%AD%EC%9D%84-%EC%84%A0%EC%A0%95%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95](https://donggreen.tistory.com/entry/Pull-up-resistor-풀업-저항을-선정하는-방법)
24. I2C 통신(I2C Communication) - DIY를 위한 AI - 티스토리, 8월 10, 2025에 액세스, https://ai4diy.tistory.com/entry/I2C-Communication
25. Pull-up resistors (I2C) - Wire (I2C) | Reference - Particle docs, 8월 10, 2025에 액세스, https://docs.particle.io/reference/device-os/api/wire-i2c/pull-up-resistors-i2c/
26. How did I debug I2C communication failure? - MadMachine, 8월 10, 2025에 액세스, [https://docs.madmachine.io/blog/A%20deeper%20dive%20into%20I2C%20usage%20and%20ways%20to%20filter%20noises.](https://docs.madmachine.io/blog/A deeper dive into I2C usage and ways to filter noises.)
27. How to debug I2C through waveform analysis - Texas Instruments, 8월 10, 2025에 액세스, https://www.ti.com/lit/slyt770
28. If and When Do I²C Pull-Up Values Matter? -- Workbench Wednesdays 83, 8월 10, 2025에 액세스, https://community.element14.com/challenges-projects/element14-presents/workbenchwednesdays/w/documents/28424/if-and-when-do-i2c-pull-up-values-matter----workbench-wednesdays-83
29. I2C Bus Pull-Up Resistor Calculation - Texas Instruments, 8월 10, 2025에 액세스, https://www.ti.com/lit/pdf/slva689
30. LPI2C Clock Stretching in RT1010 - NXP Semiconductors, 8월 10, 2025에 액세스, https://www.nxp.com/docs/en/application-note/AN13472.pdf
31. Re: MK22 master handling i2c clock stretching - NXP Community, 8월 10, 2025에 액세스, https://community.nxp.com/t5/Kinetis-Microcontrollers/MK22-master-handling-i2c-clock-stretching/m-p/1740616/highlight/true
32. Learning I2C: What is Clock Stretching? | by Ben Gillett | CodeX | Medium, 8월 10, 2025에 액세스, https://medium.com/codex/learning-i2c-what-is-clock-stretching-704eb4ce7e6c
33. I2C Arbitration | Prodigy Technovations - YouTube, 8월 10, 2025에 액세스, https://www.youtube.com/watch?v=HNkUwyACuL0
34. I2C Clock arbitration - Interface forum - Interface - TI E2E support ..., 8월 10, 2025에 액세스, https://e2e.ti.com/support/interface-group/interface/f/interface-forum/478364/i2c-clock-arbitration
35. AN10216-01 I2C Manual - NXP Semiconductors, 8월 10, 2025에 액세스, https://www.nxp.com/docs/en/application-note/AN10216.pdf
36. I2C-bus specification and user manual - NXP Semiconductors, 8월 10, 2025에 액세스, https://www.nxp.com/docs/en/user-guide/UM10204.pdf
37. Standard Mode - I2C-Bus.org, 8월 10, 2025에 액세스, https://www.i2c-bus.org/standard-mode/
38. Fast Mode - I2C-Bus.org, 8월 10, 2025에 액세스, https://www.i2c-bus.org/fastmode/
39. 1-MHz I2C-bus control on longer buses, 8월 10, 2025에 액세스, https://www.nxp.com/docs/en/brochure/75015687.pdf
40. Fast Mode Plus - I2C-Bus.org, 8월 10, 2025에 액세스, https://www.i2c-bus.org/fast-mode-plus/
41. Basics of I2C: The I2C Protocol, 8월 10, 2025에 액세스, https://www.ti.com/content/dam/videos/external-videos/en-us/8/3816841626001/6235789966001.mp4/subassets/basics-of-i2c-the-i2c-protocol-presentation.pdf
42. I2C Ultra Fast-mode (UFm) Debuts on NXP Devices, 8월 10, 2025에 액세스, https://www.nxp.com/company/about-nxp/smarter-world-videos/I2C-ULTRA-FASTMODE-UFM
43. Issues with the I²C (Inter-IC) Bus and How to Solve Them - DigiKey, 8월 10, 2025에 액세스, https://www.digikey.com/en/articles/issues-with-the-i2c-bus-and-how-to-solve-them
44. UART vs I2C vs SPI – Communication Protocols and Uses - Latest ..., 8월 10, 2025에 액세스, https://www.seeedstudio.com/blog/2019/09/25/uart-vs-i2c-vs-spi-communication-protocols-and-uses/
45. SPI vs. I2C vs. UART: Choosing the Right Communication Protocol | by Anjanashibu, 8월 10, 2025에 액세스, https://medium.com/@anjanashibu142/spi-vs-i2c-vs-uart-choosing-the-right-communication-protocol-e79109d7b64c
46. I2C vs SPI vs UART – Introduction and Comparison of their ..., 8월 10, 2025에 액세스, https://www.totalphase.com/blog/2021/12/i2c-vs-spi-vs-uart-introduction-and-comparison-similarities-differences/
47. I2C vs SPI: A Comprehensive Comparison and Analysis - Wevolver, 8월 10, 2025에 액세스, https://www.wevolver.com/article/i2c-vs-spi-protocols-differences-pros-cons-use-cases
48. I2C vs SPI vs UART: A Comprehensive Comparison - Wevolver, 8월 10, 2025에 액세스, https://www.wevolver.com/article/i2c-vs-uart
49. i2c, SPI and UART compared - Renzo Mischianti, 8월 10, 2025에 액세스, https://mischianti.org/i2c-spi-and-uart-compared/
50. Understanding and Selecting in 2024: I2C, SPI, UART Explained - Parlez-vous Tech, 8월 10, 2025에 액세스, https://www.parlezvoustech.com/en/comparaison-protocoles-communication-i2c-spi-uart/
51. Why is it important to learn SPI, I2C, UART, etc. ? : r/embedded - Reddit, 8월 10, 2025에 액세스, https://www.reddit.com/r/embedded/comments/cmzrlu/why_is_it_important_to_learn_spi_i2c_uart_etc/
52. 44. SPI vs I2C Speed Comparison - EWskills, 8월 10, 2025에 액세스, https://www.ewskills.com/task/spi-vs-i2c-speed-comparison/59
53. Difference Between Half-Duplex vs Full-Duplex - Total Phase, 8월 10, 2025에 액세스, https://www.totalphase.com/blog/2022/10/difference-between-half-duplex-vs-full-duplex/
54. Why do I2C and SPI require more than one wire? Since full-duplex communication over one wire is possible, what is the factor that limits having full-duplex using only one wire? - Reddit, 8월 10, 2025에 액세스, https://www.reddit.com/r/embedded/comments/1ceap55/why_do_i2c_and_spi_require_more_than_one_wire/
55. Communication protocols: UART, I2C and SPI - HiBit, 8월 10, 2025에 액세스, https://www.hibit.dev/posts/102/communication-protocols-uart-i2c-and-spi
56. UART vs. Other Serial Communication Protocols: A Comparison with SPI and I2C, 8월 10, 2025에 액세스, https://iotebyte.wordpress.com/2023/05/25/uart-vs-other-serial-communication-protocols-a-comparison-with-spi-and-i2c/
57. Inter-Integrated Circuit (I2C) Protocol - Arduino Documentation, 8월 10, 2025에 액세스, https://docs.arduino.cc/learn/communication/wire
58. DS1307 Real Time Clock Breakout Board Kit - Adafruit, 8월 10, 2025에 액세스, https://cdn-learn.adafruit.com/downloads/pdf/ds1307-real-time-clock-breakout-board-kit.pdf
59. Arduino and MPU6050 Accelerometer and Gyroscope Tutorial, 8월 10, 2025에 액세스, https://howtomechatronics.com/tutorials/arduino/arduino-and-mpu6050-accelerometer-and-gyroscope-tutorial/
60. How to Use the MPU6050 With the Raspberry Pi 4 : 3 Steps ..., 8월 10, 2025에 액세스, https://www.instructables.com/How-to-Use-the-MPU6050-With-the-Raspberry-Pi-4/
61. Raspberry Pi SPI and I2C Tutorial - SparkFun Learn, 8월 10, 2025에 액세스, https://learn.sparkfun.com/tutorials/raspberry-pi-spi-and-i2c-tutorial/all
62. Adding a Real Time Clock (RTC) to the Raspberry Pi - Pi My Life Up, 8월 10, 2025에 액세스, https://pimylifeup.com/raspberry-pi-rtc/
63. Raspberry Pi and 24LC256, 8월 10, 2025에 액세스, https://forums.raspberrypi.com/viewtopic.php?t=153951
64. How to Connect MPU6050 to Arduino UNO - Instructables, 8월 10, 2025에 액세스, https://www.instructables.com/How-to-Connect-MPU6050-to-Arduino-UNO/
65. Lesson 05: Gyroscope & Accelerometer Module (MPU6050) - SunFounder's Documentations!, 8월 10, 2025에 액세스, https://docs.sunfounder.com/projects/umsk/en/latest/02_arduino/uno_lesson05_mpu6050.html
66. EEPROM 24LC256- Reading and Writing Arduino Sketch | by J3 | Jungletronics - Medium, 8월 10, 2025에 액세스, https://medium.com/jungletronics/eeprom-24lc256-reading-and-writing-arduino-sketch-bdfb6e5a3b13
67. 24lc256 Arduino - GitHub Gist, 8월 10, 2025에 액세스, https://gist.github.com/b0bf974b5a925cb214df
68. QuentinCG/Arduino-I2C-EEPROM-library - GitHub, 8월 10, 2025에 액세스, https://github.com/QuentinCG/Arduino-I2C-EEPROM-library
69. Raspberry Pi I2C 256K EEPROM Tutorial - Digitalpeer, 8월 10, 2025에 액세스, https://www.digitalpeer.com/blog/raspberry-pi-i2c-256k-eeprom-tutorial
70. Introduction - Adafruit DS1307 Library 1.0 documentation, 8월 10, 2025에 액세스, https://docs.circuitpython.org/projects/ds1307/en/latest/
71. DS1307 Real Time Clock Breakout Board Kit - Adafruit Learning System, 8월 10, 2025에 액세스, https://learn.adafruit.com/ds1307-real-time-clock-breakout-board-kit?view=all
72. Set Up Real Time Clock (RTC) on Raspberry Pi - Instructables, 8월 10, 2025에 액세스, https://www.instructables.com/Set-up-Real-Time-Clock-RTC-on-Raspberry-Pi/
73. LM75B LM75C Digital Temperature Sensor and Thermal Watchdog with Two-Wire Interface - Mouser Electronics, 8월 10, 2025에 액세스, http://www.mouser.com/ds/2/282/snis153a-123211.pdf
74. LM75B Digital temperature sensor and thermal watchdog - NXP Semiconductors, 8월 10, 2025에 액세스, https://www.nxp.com/docs/en/data-sheet/LM75B.pdf
75. LM75 Temperature Sensor Module: Pinout and Interfacing with ..., 8월 10, 2025에 액세스, https://microcontrollerslab.com/lm75-temperature-sensor-module-pinout-interfacing-with-arduino/
76. How to Use LM75: Examples, Pinouts, and Specs - Cirkit Designer Docs, 8월 10, 2025에 액세스, https://docs.cirkitdesigner.com/component/d25260ee-61ff-4e79-9218-7e7093466760/lm75
77. Solving common I2C Communication Setup Issues - Semify, 8월 10, 2025에 액세스, https://www.semify-eda.com/post/solving-common-i2c-communication-setup-issues
78. Common Problems In Systems - I2C-Bus.org, 8월 10, 2025에 액세스, https://www.i2c-bus.org/i2c-primer/common-problems/
79. Understanding I2C Errors | Dev Center - Electric Imp, 8월 10, 2025에 액세스, https://developer.electricimp.com/resources/i2cerrors
80. Troubleshooting I2C - Texas Instruments, 8월 10, 2025에 액세스, https://www.ti.com/lit/pdf/scaa106
81. wiki.dfrobot.com, 8월 10, 2025에 액세스, [https://wiki.dfrobot.com/How_to_Resolve_I2C_Address_Conflicts_in_Embedded_Systems#:~:text=An%20I2C%20multiplexer%20is%20a,address%20by%20selecting%20different%20channels.](https://wiki.dfrobot.com/How_to_Resolve_I2C_Address_Conflicts_in_Embedded_Systems#:~:text=An I2C multiplexer is a,address by selecting different channels.)
82. How to Resolve I2C Address Conflicts in Embedded Systems, 8월 10, 2025에 액세스, https://wiki.dfrobot.com/How_to_Resolve_I2C_Address_Conflicts_in_Embedded_Systems
83. I2C address conflict workaround through delayed signal propagation or delayed powerup - Electrical Engineering Stack Exchange, 8월 10, 2025에 액세스, https://electronics.stackexchange.com/questions/737206/i2c-address-conflict-workaround-through-delayed-signal-propagation-or-delayed-po
84. I2C Dynamic Addressing - Texas Instruments, 8월 10, 2025에 액세스, https://www.ti.com/lit/pdf/scaa137
85. Weird I2C Problems - arduino - Electronics Stack Exchange, 8월 10, 2025에 액세스, https://electronics.stackexchange.com/questions/716363/weird-i2c-problems
86. How to get rid of this reflection on falling edge of i2c SDA line? - Electronics Stack Exchange, 8월 10, 2025에 액세스, https://electronics.stackexchange.com/questions/258521/how-to-get-rid-of-this-reflection-on-falling-edge-of-i2c-sda-line
87. What are the common problems with I2C communication? : r/embedded - Reddit, 8월 10, 2025에 액세스, https://www.reddit.com/r/embedded/comments/1d7s8y3/what_are_the_common_problems_with_i2c/
88. Debugging the I2C Interface Between Arduino UNO and Sensor BMP280 Using a $10 Logic Analyzer - YouTube, 8월 10, 2025에 액세스, https://www.youtube.com/watch?v=eMb0bNoprEM
89. How to efficiently debug I2C communication with a logic analyzer ..., 8월 10, 2025에 액세스, https://www.youtube.com/watch?v=9GHKuFoOnVc
90. I2C : Protocol Decoding using Saleae USB Logic Analyzer - YouTube, 8월 10, 2025에 액세스, https://www.youtube.com/watch?v=Zv5AFM35ZTw


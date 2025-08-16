# USB HID 프로토콜

## 서론

USB HID(Human Interface Device) 프로토콜은 현대 컴퓨팅 환경에서 너무나 당연하게 여겨지는 '플러그 앤 플레이(Plug-and-Play)' 개념을 실현한 핵심 기술이다. USB 이전 시대에는 키보드, 마우스 같은 단순한 주변기기조차도 각기 다른 포트와 전용 드라이버, 수동 설정이 필요했다. 이러한 혼란을 종식시키기 위해 등장한 HID 클래스는 인간과 컴퓨터 간의 상호작용을 위한 보편적이고 표준화된 규약을 제공하는 것을 목표로 했다.1 그 결과, 어떤 제조사의 제품이든 표준을 준수하기만 하면 별도의 드라이버 설치 없이 운영체제에 연결하는 즉시 작동하게 되었다.1

하지만 HID의 진정한 힘은 단순히 키보드와 마우스에 국한되지 않는다는 점에서 드러난다. 이 프로토콜의 설계는 본질적으로 매우 유연하여, 게임 컨트롤러, 터치스크린, 오디오 볼륨 조절기, 심지어는 무정전 전원 공급장치(UPS)나 의료용 계측기기 같은 복잡한 장비까지도 기술(describe)할 수 있다.4 이처럼 광범위한 확장성은 HID 프로토콜의 핵심적인 특징이다.

이 보고서의 중심 논지는 HID를 이토록 강력하고 편리하게 만든 바로 그 설계 철학, 즉 장치가 스스로의 기능과 데이터 형식을 정의하는 **자기-서술(self-describing) 특성**이 동시에 가장 심각한 보안 취약점의 근원이 된다는 점이다. 호스트 컴퓨터가 주변기기가 "나는 키보드다"라고 선언하는 것을 근본적으로 신뢰하는 이 모델은, BadUSB와 같은 하드웨어 기반 공격을 가능하게 하는 아킬레스건이 된다.8 이처럼 편리성과 보안성 사이의 본질적인 긴장 관계는 HID 프로토콜을 이해하는 데 있어 가장 중요한 관점이며, 이 보고서 전체를 관통하는 주제가 될 것이다.

본 문서는 HID 프로토콜의 근간을 이루는 구조적 원리부터 시작하여(1부), 데이터 구조의 기술적 해부(2부), 실제 응용 및 구현 사례(3부), 그리고 최신 기술로의 확장(4부)을 거쳐, 마지막으로 이 프로토콜이 내포한 치명적인 보안 위협과 그 대응 전략(5부)까지 심층적으로 분석할 것이다.

------

## 1. 부: HID 프로토콜의 구조와 철학

HID 프로토콜의 본질을 이해하기 위해서는 단순히 기능적인 측면을 넘어, 그 기반에 깔린 설계 철학을 파악해야 한다. 이 철학은 '어떻게 작동하는가' 뿐만 아니라 '어떻게 생각하는가'에 대한 답을 제시한다.

### 1.1  자기-서술(Self-Describing)의 철학: 장치가 스스로를 소개하는 방식

HID 프로토콜의 가장 핵심적인 원칙은 장치 스스로가 자신의 기능과 데이터 형식을 설명하는 정보를 담고 있다는 점이다.11 호스트 운영체제는 연결된 장치의 특정 모델에 대한 사전 지식을 가질 필요가 없다.13 장치가 USB 포트에 연결되면, 호스트는 열거(enumeration) 과정에서 **디스크립터(Descriptor)**라고 불리는 일련의 데이터 구조를 장치로부터 요청한다. 이 중 HID에서 가장 중요한 것이 바로 **리포트 디스크립터(Report Descriptor)**인데, 이는 장치가 호스트와 주고받을 데이터 패킷, 즉 **리포트(Report)**의 구조를 정의하는 일종의 설계도 역할을 한다.3

이러한 자기-서술 방식은 몇 가지 중요한 목표를 달성하기 위해 의도적으로 설계되었다. 데이터 공간을 절약하기 위해 최대한 간결해야 하고, 미래의 새로운 기능을 수용할 수 있도록 확장 가능해야 하며, 소프트웨어 애플리케이션이 알지 못하는 정보를 건너뛸 수 있게 하여 하위 호환성을 보장해야 했다.11 이 덕분에 범용 애플리케이션은 장치별 드라이버 없이도 HID 장치의 데이터를 해석하고 활용할 수 있게 된다.

### 1.2  핵심 구성 요소: 디스크립터(Descriptors), 리포트(Reports), 그리고 Usage의 역할

HID 통신은 세 가지 핵심 개념을 중심으로 이루어진다.

- **디스크립터(Descriptors):** 장치가 열거 과정에서 호스트에게 제공하는 구조화된 정보 블록이다. 이 정보에는 제조사 ID(VID)와 제품 ID(PID) 같은 기본 정보부터 시작해서, 장치의 클래스가 HID라는 사실, 그리고 데이터의 구체적인 형식까지 모든 것이 포함된다.2
- **리포트(Reports):** 실제 작동 중에 호스트와 장치 간에 교환되는 데이터 패킷이다. 이 패킷은 단순한 데이터 덩어리가 아니라, 리포트 디스크립터에 의해 그 구조가 엄격하게 정의된다.5 예를 들어, 마우스 리포트는 버튼 상태를 나타내는 비트들과 X, Y축의 움직임을 나타내는 바이트들로 구성된다.15
- **Usage Tables:** 리포트 안의 데이터에 의미를 부여하는 표준화된 사전이다. **Usage Page**는 "일반 데스크톱 컨트롤(Generic Desktop Controls)"과 같은 큰 카테고리를 정의하고, 그 페이지 내의 **Usage ID**는 "마우스(Mouse)"나 "키보드(Keyboard)"와 같은 특정 기능을 정의한다.11 이 시스템 덕분에 호스트는 특정 비트 스트림이 키보드 입력인지, 조이스틱의 움직임인지 정확히 구분할 수 있다.

### 1.3  데이터 전송 방식: 왜 인터럽트 전송(Interrupt Transfer)을 사용하는가?

USB는 네 가지 데이터 전송 유형을 정의한다: 컨트롤(Control), 인터럽트(Interrupt), 벌크(Bulk), 아이소크로너스(Isochronous).18 HID 장치는 데이터 리포트를 교환하기 위해 주로 **인터럽트 전송**을 사용한다.3

인터럽트 전송은 **보장된 지연 시간(Guaranteed Latency)**을 제공하기 때문에 HID에 이상적이다. 호스트는 미리 정해진 간격(예: 수 밀리초마다)으로 장치를 주기적으로 폴링(polling)한다. 만약 장치에 키 입력과 같은 새로운 데이터가 있으면 전송하고, 없으면 NAK(Negative Acknowledgment) 신호로 응답한다.17 이는 작고, 비주기적이며, 신속한 응답이 필요한 데이터에 완벽하게 부합한다. 또한 CRC를 통한 오류 감지와 재전송 시도를 지원하여 데이터의 신뢰성을 보장한다.18

반면 다른 전송 방식들은 HID에 적합하지 않다. **컨트롤 전송**은 디스크립터 요청과 같은 초기 설정 및 구성에 사용되지만, 지속적인 데이터 스트림에는 부적합하다.3

**벌크 전송**은 USB 플래시 드라이브처럼 지연 시간보다는 높은 처리량이 중요한 대용량 데이터 전송을 위해 설계되었다. 버스 대역폭이 보장되지 않아 마우스나 키보드의 즉각적인 반응성을 구현할 수 없다.19

**아이소크로너스 전송**은 오디오나 비디오 스트리밍처럼 실시간성이 중요하여 데이터 재전송을 포기하는 대신 대역폭을 보장한다. 데이터 손실이 허용되므로 모든 입력이 정확히 전달되어야 하는 HID에는 사용할 수 없다.18

이처럼 인터럽트 전송 방식은 HID 프로토콜의 핵심 요구사항인 '낮은 지연 시간'과 '높은 신뢰성'을 모두 만족시키는 유일한 선택지다.

| 전송 유형 (Transfer Type)        | 주요 사용 사례                 | 지연 시간 (Latency) | 대역폭 보장 (Bandwidth Guarantee) | 데이터 무결성 (Data Integrity) | HID 사용 예시                                 |
| -------------------------------- | ------------------------------ | ------------------- | --------------------------------- | ------------------------------ | --------------------------------------------- |
| **컨트롤 (Control)**             | 장치 설정, 열거, 명령 전송     | 보장되지 않음       | 보장됨 (일부)                     | 보장됨 (오류 감지 및 재시도)   | 디스크립터 요청, LED 상태 설정                |
| **인터럽트 (Interrupt)**         | 작고 빠른 응답이 필요한 데이터 | 보장됨 (폴링 간격)  | 보장됨                            | 보장됨 (오류 감지 및 재시도)   | 키보드 입력, 마우스 움직임 (주요 데이터 전송) |
| **벌크 (Bulk)**                  | 대용량 데이터 전송             | 보장되지 않음       | 보장되지 않음                     | 보장됨 (오류 감지 및 재시도)   | 사용하지 않음                                 |
| **아이소크로너스 (Isochronous)** | 실시간 데이터 스트리밍         | 보장됨              | 보장됨                            | 보장되지 않음 (재시도 없음)    | 사용하지 않음                                 |

여기서 주목할 점은 인터럽트 전송의 '인터럽트'라는 용어가 실제 동작 방식과는 다소 거리가 있다는 것이다. 전통적인 CPU 인터럽트처럼 장치가 능동적으로 호스트의 작업을 중단시키는 것이 아니다. USB 아키텍처의 근본적인 특징은 모든 통신을 호스트가 주도한다는 점이다. 즉, 호스트가 주기적으로 장치를 '폴링'하여 데이터를 요청하는 방식이다.3 '인터럽트'라는 이름은 이 전송 방식이 처리하는 데이터의 성격, 즉 일반적으로 시스템 인터럽트를 유발할 만한 이벤트성 정보를 다룬다는 의미에서 붙여졌다. 이 호스트 주도 폴링 방식은 USB 버스의 안정성을 확보하기 위한 핵심적인 설계 결정이다. 만약 결함이 있거나 악의적인 장치가 무분별하게 신호를 보낼 수 있다면 전체 시스템이 마비될 수 있기 때문이다. 장치는 호스트가 물어볼 때까지 기다려야만 한다.18

또한, 이 구조적 철학은 HID의 가장 큰 보안 문제와 직결된다. '드라이버 없는' 플러그 앤 플레이를 구현하기 위한 전제 조건은 호스트가 장치가 제공하는 디스크립터를 신뢰하는 것이다.1 이것은 버그나 설계상 실수가 아니라, 편리성을 위해 만들어진 근본적인 약속이다. BadUSB 공격은 바로 이 신뢰 모델을 직접적으로 악용한다.23 공격자는 장치의 펌웨어를 조작하여 자신을 '키보드'라고 거짓으로 소개하고, 호스트는 프로토콜에 따라 충실히 키보드 드라이버를 로드하여 공격자에게 시스템 제어권을 넘겨주게 된다. 즉, HID의 가장 위대한 특징이 보안의 관점에서는 가장 치명적인 약점이 되는 것이다.

------

## 2. 부: 기술적 해부: 디스크립터와 리포트

HID 프로토콜의 실제 동작을 이해하기 위해서는 그 핵심을 이루는 데이터 구조, 즉 디스크립터와 리포트를 정밀하게 분해해 볼 필요가 있다. 이 부분은 프로토콜의 기술적 심장부라 할 수 있다.

### 2.1  정보의 계층 구조: Standard USB Descriptors

HID 고유의 디스크립터를 살펴보기 전에, 모든 USB 장치가 공통으로 사용하는 표준 디스크립터의 계층 구조를 이해하는 것이 우선이다. 장치가 연결되면 호스트는 다음과 같은 중첩된 구조의 디스크립터들을 순차적으로 요청하여 장치의 전체적인 그림을 파악한다.2

- **장치 디스크립터 (Device Descriptor):** 최상위에 위치하며, 장치에 대한 가장 기본적인 정보를 담는다. 제조사 ID(VID), 제품 ID(PID), 그리고 장치가 가질 수 있는 설정(Configuration)의 개수 등이 포함된다.3
- **구성 디스크립터 (Configuration Descriptor):** 장치의 특정 동작 모드를 정의한다. 대부분의 장치는 하나의 설정만을 가지지만, 이론적으로는 고전력 모드와 저전력 모드처럼 여러 설정을 가질 수 있다. 이 디스크립터는 해당 설정이 몇 개의 인터페이스를 포함하는지 명시한다.16
- **인터페이스 디스크립터 (Interface Descriptor):** 장치가 어떤 종류의 '클래스'에 속하는지를 선언하는 매우 중요한 부분이다. HID 장치의 경우, `bInterfaceClass` 필드 값이 `0x03`으로 설정된다.7 이 값을 통해 호스트 OS는 해당 장치의 제어를 범용 HID 클래스 드라이버에게 위임해야 함을 인지한다.2 하나의 장치는 여러 인터페이스를 가질 수 있다. 예를 들어, 마이크가 달린 USB 헤드셋은 소리 입출력을 위한 오디오 클래스 인터페이스와 볼륨 조절 버튼을 위한 HID 클래스 인터페이스를 동시에 가질 수 있다.5
- **엔드포인트 디스크립터 (Endpoint Descriptor):** 인터페이스 내에서 실제 데이터가 오가는 통신 채널(파이프)을 정의한다. HID 인터페이스는 항상 존재하는 컨트롤 엔드포인트(Endpoint 0) 외에, 최소한 장치에서 호스트로 데이터를 보내기 위한 **인터럽트 IN 엔드포인트**를 하나 가진다. 선택적으로, 키보드 LED 상태 제어와 같이 호스트에서 장치로 데이터를 받기 위한 **인터럽트 OUT 엔드포인트**를 가질 수도 있다.5
- **HID 디스크립터 (HID Descriptor):** 인터페이스 디스크립터 내부에 중첩되는 특별한 디스크립터로, 가장 핵심적인 **리포트 디스크립터**의 위치와 크기 정보를 담고 있다.3

### 2.2  HID의 심장, 리포트 디스크립터(Report Descriptor)

리포트 디스크립터는 HID 프로토콜에서 가장 복잡하면서도 강력한 부분이다. 이것은 정적인 데이터 구조가 아니라, 사실상 작은 스택 기반 프로그램을 형성하는 **아이템(Item)**들의 연속된 시퀀스다. 호스트의 HID 파서는 이 디스크립터를 '실행'함으로써 장치가 보내는 리포트의 정확한 비트/바이트 단위 레이아웃을 학습한다.12

아이템은 1바이트의 접두사(아이템의 종류와 크기 정의)와 0~4바이트의 데이터로 구성되며 5, 세 가지 유형으로 나뉜다.

- **메인 아이템 (Main Items):** Input, Output, Feature와 같이 데이터 필드 자체를 정의하거나, Collection, End Collection처럼 관련 아이템들을 그룹화한다.5
- **글로벌 아이템 (Global Items):** `Usage Page`, `Logical Minimum/Maximum`(값의 범위), `Report Size`(필드당 비트 수), `Report Count`(필드 개수) 등, 이후에 나오는 모든 메인 아이템에 공통으로 적용될 속성을 정의한다.5
- **로컬 아이템 (Local Items):** 바로 다음에 오는 단 하나의 메인 아이템에만 적용될 속성을 정의한다. 가장 중요한 것은 컨트롤의 구체적인 의미를 지정하는 `Usage`이다.5

이 아이템들이 어떻게 조합되는지 이해하기 위해 간단한 3버튼 마우스의 리포트 디스크립터를 단계별로 분석해 보자.5

1. `Usage Page (Generic Desktop)`와 `Usage (Mouse)`를 선언하여 이 장치가 '마우스'임을 정의한다.
2. `Collection (Application)`으로 컨트롤 그룹을 시작한다.
3. `Usage Page (Button)`으로 버튼 정의를 시작함을 알린다. `Usage Minimum (1)`부터 `Usage Maximum (3)`까지 3개의 버튼을 지정한다. `Logical Minimum (0)`, `Logical Maximum (1)`, `Report Size (1)`, `Report Count (3)`을 통해 이들이 각각 1비트 크기를 가지는 3개의 필드임을 정의한다. 마지막으로 `Input (Data,Var,Abs)` 아이템으로 이 버튼 필드들을 최종 리포트에 포함시킨다.
4. 다음 필드를 바이트 경계에 맞추기 위해 5비트의 패딩(padding)을 추가한다.12
5. 다시 `Usage Page (Generic Desktop)`를 선언하고 `Usage (X)`와 `Usage (Y)`를 지정한다. `Logical Minimum (-127)`, `Logical Maximum (127)`, `Report Size (8)`, `Report Count (2)`를 통해 이들이 각각 8비트 크기의 부호 있는 정수 값 두 개임을 정의한다. `Input (Data,Var,Rel)` 아이템으로 이 상대 좌표 필드들을 리포트에 포함시킨다.
6. `End Collection`으로 모든 정의를 마친다.

이 과정을 거치면 호스트는 이제 이 마우스가 보내는 데이터 패킷의 첫 3비트는 버튼 상태, 그 다음 5비트는 패딩, 그리고 이어지는 두 바이트는 각각 X, Y축의 상대적 움직임을 나타낸다는 사실을 정확히 알게 된다.

| 아이템 유형 | 아이템 이름       | 16진수 코드 (예시) | 기능 및 설명                                                 |
| ----------- | ----------------- | ------------------ | ------------------------------------------------------------ |
| **Main**    | `Input`           | `0x81`             | 장치에서 호스트로 전송되는 데이터 필드를 정의 (예: 버튼, 축) |
|             | `Output`          | `0x91`             | 호스트에서 장치로 전송되는 데이터 필드를 정의 (예: LED)      |
|             | `Feature`         | `0xB1`             | 호스트가 읽고 쓸 수 있는 구성/상태 필드를 정의 (예: 감도 설정) |
|             | `Collection`      | `0xA1`             | 관련 아이템들을 논리적/물리적 그룹으로 묶음                  |
|             | `End Collection`  | `0xC0`             | `Collection` 그룹을 닫음                                     |
| **Global**  | `Usage Page`      | `0x05`             | 이후에 나올 `Usage`들의 상위 카테고리를 지정                 |
|             | `Logical Minimum` | `0x15`             | 데이터 필드가 가질 수 있는 논리적 최솟값을 정의              |
|             | `Logical Maximum` | `0x25`             | 데이터 필드가 가질 수 있는 논리적 최댓값을 정의              |
|             | `Report Size`     | `0x75`             | 각 데이터 필드의 크기를 비트 단위로 지정                     |
|             | `Report Count`    | `0x95`             | 동일한 속성을 가진 데이터 필드의 개수를 지정                 |
| **Local**   | `Usage`           | `0x09`             | 데이터 필드의 구체적인 용도(의미)를 지정                     |
|             | `Usage Minimum`   | `0x19`             | 연속된 `Usage` 범위의 시작점을 지정                          |
|             | `Usage Maximum`   | `0x29`             | 연속된 `Usage` 범위의 끝점을 지정                            |

### 2.3  데이터의 표준어, Usage Tables

HID Usage Tables 문서는 HID 프로토콜의 '사전'과 같은 역할을 한다.11 이 문서는 리포트 디스크립터에 사용되는 데이터 필드에 의미론적 가치를 부여하는 방대하고 표준화된 상수(`Usage`) 목록을 정의한다.

`Usage Page`는 일반 데스크톱(Generic Desktop), 키보드/키패드(Keyboard/Keypad), 시뮬레이션 컨트롤(Simulation Controls), 게임 컨트롤(Game Controls), 소비자 가전(Consumer), 의료 기기(Medical Instrument) 등과 같은 상위 레벨의 그룹을 나타낸다.11

`Usage ID`는 해당 페이지 내의 구체적인 항목을 가리킨다. 예를 들어, 'Generic Desktop' 페이지 내에서 `0x01`은 포인터, `0x02`는 마우스, `0x04`는 조이스틱, `0x06`은 키보드를 의미한다.12

이러한 `Usage Page`와 `Usage ID`의 2단계 조합은 사실상 상상할 수 있는 거의 모든 종류의 입출력 제어 장치에 대해 고유한 32비트 식별자를 생성할 수 있게 해주며, 이는 프로토콜의 엄청난 확장성을 보장한다.12 새로운 기술이 등장함에 따라 HID 사용법 테이블 검토 요청(HUTRR)이라는 공식 절차를 통해 새로운 Usage가 지속적으로 추가되고 있다.11

| Usage Page (16진수 & 이름)         | 예시 Usage ID (16진수 & 이름)                                |
| ---------------------------------- | ------------------------------------------------------------ |
| `0x01` Generic Desktop             | `0x02` Mouse, `0x04` Joystick, `0x06` Keyboard, `0x80` System Control |
| `0x07` Keyboard/Keypad             | `0x04` 'a' and 'A', `0x2C` Spacebar, `0xE0` LeftControl      |
| `0x02` Simulation Controls         | `0x01` Flight Simulation Device, `0xBA` Rudder, `0xC4` Throttle |
| `0x0C` Consumer                    | `0xCD` Play/Pause, `0xE9` Volume Up, `0xEA` Volume Down      |
| `0x40` Medical Instrument          | `0x01` Medical Ultrasound, `0x30` Body Temperature           |
| `0xF1D0` FIDO Alliance             | `0x01` U2F Authenticator Device                              |
| `0xFF00` - `0xFFFF` Vendor-Defined | 제조사가 독자적으로 정의하여 사용                            |

### 2.4  리포트의 세 가지 유형: Input, Output, Feature

리포트 디스크립터에 의해 정의된 데이터 패킷은 `Input`, `Output`, `Feature`라는 세 가지 메인 아이템에 따라 세 가지 유형으로 구분된다.3

- **입력 리포트 (Input Report):** **장치에서 호스트로** 전송되는 데이터. 키 입력, 마우스 움직임, 조이스틱 위치 등 가장 일반적인 형태의 데이터 전송에 사용된다.3
- **출력 리포트 (Output Report):** **호스트에서 장치로** 전송되는 데이터. 키보드의 Caps Lock/Num Lock LED를 켜거나 끄는 것, 게임 컨트롤러에 포스 피드백을 전달하는 것, 또는 장치에 부착된 LCD에 정보를 표시하는 등의 용도로 사용된다.3
- **기능 리포트 (Feature Report):** 호스트가 **읽거나 쓸 수 있는** 데이터. 장치 감도 설정, 보정(calibration) 데이터 검색, 장치 시리얼 번호 확인 등, 일반적인 입출력 스트림에 포함되지 않는 구성 정보를 주고받는 데 사용된다.3

이러한 기술적 구조를 통해, 리포트 디스크립터는 단순한 데이터 구조 정의를 넘어 데이터 패킷을 기술하기 위한 일종의 도메인 특화 언어(Domain-Specific Language, DSL)처럼 기능한다. `Report Size`, `Report Count` 같은 글로벌 아이템을 통해 동적으로 컨트롤 배열을 정의하고, `Collection` 아이템으로 관련 데이터를 그룹화하는 방식은 정적인 C언어 구조체와는 근본적으로 다르다. 이러한 프로그래밍 가능한 특성이 바로 HID 프로토콜의 핵심적인 유연성과 확장성을 부여하는 기술적 기반이다. USB-IF에서 "HID Descriptor Tool" `[15, 27]`이나 Microsoft에서 "Waratah" `11`와 같은 공식 툴을 제공하는 것 자체가 이 디스크립터의 복잡성을 방증한다.

결과적으로 "Human Interface Device"라는 이름은 역사적인 유물에 가깝게 되었다. UPS, 온도계, 산업용 컨트롤러 등 인간과 직접적인 상호작용이 없는 수많은 장치들이 HID 클래스를 채택하고 있다.6 그 이유는 HID가 제공하는 표준화되고 드라이버가 필요 없는, 신뢰성 있는 저지연 데이터 전송 메커니즘이 이런 장치들의 간단한 상태 정보(온도, 배터리 잔량 등)를 보고하는 데 매우 효율적이기 때문이다. 복잡한 커스텀 드라이버를 개발하는 대신, 간단한 리포트 디스크립터를 정의하는 것만으로 모든 주요 운영체제에 내장된 견고한 HID 인프라를 활용할 수 있는 것이다.1 프로토콜은 이미 그 이름이 가진 한계를 넘어 진화했다.

------

## 3. 부: 응용과 구현

1부와 2부에서 다룬 이론적 개념들이 실제 세계의 장치와 코드에서 어떻게 구현되는지 구체적으로 살펴본다.

### 3.1  표준 장치 구현: 키보드, 마우스, 게임 컨트롤러

가장 흔하게 접할 수 있는 HID 장치들은 표준화된 리포트 디스크립터 구조를 따른다.1

- **마우스:** 일반적인 마우스 리포트는 총 3~4바이트로 구성된다. 첫 바이트는 버튼 상태를 나타내는 비트 필드(보통 3~5개 비트 사용), 두 번째와 세 번째 바이트는 각각 X축과 Y축의 상대적 이동량을 나타내는 부호 있는 정수 값이다.15
- **키보드:** 표준 '부트 프로토콜' 키보드 리포트는 총 8바이트로 구성된다. 첫 바이트는 Ctrl, Shift, Alt, GUI(Windows/Command 키) 같은 모디파이어 키의 상태를 비트맵으로 나타낸다. 두 번째 바이트는 예약되어 있으며, 나머지 6바이트는 동시에 눌린 최대 6개의 일반 키 스캔 코드를 담는 배열로 사용된다.6
- **게임 컨트롤러:** 조이스틱이나 게임패드는 마우스와 키보드의 개념을 확장한 형태다. 여러 개의 아날로그 축(X, Y, Z, 회전 등)과 다수의 디지털 버튼을 포함하는 복합적인 리포트 디스크립터를 가진다. HID 표준 덕분에 과거 게임 포트 시절과 달리 대부분의 최신 게임 컨트롤러는 별도의 드라이버 없이도 운영체제에서 즉시 인식된다.4

#### 3.1.1  심층 분석: N-Key Rollover(NKRO)의 "신화"와 진실

많은 사용자들이 USB 키보드는 최대 6개의 키만 동시에 입력할 수 있다고 알고 있지만, 이는 절반만 맞는 이야기다. 6키 동시 입력 제한(6-Key Rollover, 6KRO)은 USB HID 프로토콜 자체의 근본적인 한계가 아니라, **부트 프로토콜(Boot Protocol)**이라는 특정 규격의 특징이다.6 이 부트 프로토콜은 운영체제의 복잡한 HID 파서가 로드되기 전, BIOS와 같은 리소스가 제한된 환경에서도 키보드를 사용할 수 있도록 매우 단순하게 설계되었다.6

진정한 **N-Key Rollover(NKRO)**, 즉 모든 키를 동시에 입력할 수 있는 기능은 부트 프로토콜이 아닌, 완전한 기능을 갖춘 운영체제가 해석할 수 있는 별도의 리포트 디스크립터를 정의함으로써 구현된다. 이 방식은 6바이트 키 코드 배열 대신, 키보드의 모든 키에 해당하는 비트맵을 사용한다. 예를 들어, 104키 키보드라면 104비트(13바이트) 크기의 배열을 리포트로 보내 각 키의 눌림/떨어짐 상태를 1 또는 0으로 직접 표현한다.32

일부 키보드는 여러 개의 가상 키보드 인터페이스를 가진 복합 장치로 자신을 등록하여 NKRO를 구현하기도 한다.33 하지만 이 방식은 운영체제의 HID 파서, 특히 macOS에서 문제를 일으키는 경우가 있어 완벽한 해결책은 아니다.35 이 NKRO 문제는 범용적이지만 기능이 제한된 호환성(부트 프로토콜)과 모든 기능을 지원하지만 더 복잡한 구현(네이티브 NKRO 프로토콜) 사이의 트레이드오프를 보여주는 완벽한 사례다.

### 3.2  운영체제와의 상호작용: The HID Class Driver

HID 장치가 컴퓨터에 연결될 때 일어나는 일련의 과정은 운영체제의 **HID 클래스 드라이버**가 핵심적인 역할을 수행한다.

1. 장치가 연결되면, OS의 범용 USB 드라이버가 초기 열거를 수행한다. 장치/구성 디스크립터를 읽어 `bInterfaceClass` 필드가 `0x03`(HID)임을 확인한다.2
2. 이 정보를 바탕으로 OS는 범용 **HID 클래스 드라이버**(Windows의 경우 `hidusb.sys`와 같은 드라이버)를 로드한다.37 이 드라이버가 이후의 모든 통신을 담당한다.
3. HID 클래스 드라이버는 장치로부터 HID 디스크립터와 가장 중요한 **리포트 디스크립터**를 요청하여 파싱한다.2 이를 통해 장치의 데이터 형식을 완벽하게 이해하게 된다.
4. 마지막으로, 클래스 드라이버는 애플리케이션이 표준 API(Windows의 경우 HID IOCTL)를 통해 상호작용할 수 있는 가상의 장치 객체를 생성한다.37 이 추상화 계층 덕분에 응용 소프트웨어는 하드웨어의 구체적인 모델(로지텍 마우스인지, MS 마우스인지)을 전혀 알 필요 없이 OS에 "마우스 데이터 줘"라고 요청하기만 하면 된다.1

이 HID 클래스 드라이버는 주변기기 호환성의 숨은 영웅이라 할 수 있다. 장치가 스스로를 설명하는 것도 중요하지만, 그 설명을 해석할 수 있는 범용 번역기가 없다면 무용지물이다. OS 제조사가 직접 관리하는 이 단일 드라이버는 모든 유효한 리포트 디스크립터를 파싱하도록 설계되어, 하드웨어 제조사들에게 매우 안정적이고 예측 가능한 개발 목표를 제공한다. 제조사들은 더 이상 여러 OS를 위한 드라이버를 개발하고 배포, 유지보수하는 데 골머리를 앓을 필요 없이, 유효한 리포트 디스크립터를 만드는 데만 집중하면 된다. 이는 하드웨어 제작의 진입 장벽을 극적으로 낮춰, 틈새 시장에서의 혁신을 촉진하는 중요한 기반이 되었다.3

### 3.3  표준을 넘어서: 커스텀 HID 장치 개발

HID의 진정한 잠재력은 표준 장치를 넘어선 커스텀 애플리케이션에서 발현된다. 호스트 측에 별도의 드라이버가 필요 없다는 점 때문에, 취미 개발자나 특정 목적의 제품을 만드는 엔지니어들에게 가장 선호되는 프로토콜 중 하나다.3

- **임베디드 플랫폼:** ATmega32U4 칩을 사용하는 아두이노 레오나르도나 프로 마이크로, 그리고 STM32와 같은 마이크로컨트롤러는 커스텀 HID 장치를 제작하는 데 널리 사용된다. 이들을 위한 라이브러리는 디스크립터를 정의하고 리포트를 전송하는 복잡한 과정을 크게 단순화시켜 준다.16 간단한 커스텀 키보드 제작 41부터 아날로그 조이스틱 센서를 이용한 마우스 제어 31까지 응용 분야는 무궁무진하다.
- **비전통적 응용 사례:**
  - **의료 및 실험 장비:** 혈당 측정기나 디지털 온도계는 복잡한 드라이버 없이도 HID 프로토콜을 이용해 측정값을 컴퓨터로 전송하여 기록하고 분석할 수 있다.6
  - **산업/오디오 제어:** 산업용 기계나 오디오/비디오 편집 소프트웨어를 제어하기 위한 커스텀 슬라이더와 노브가 장착된 컨트롤 서페이스를 만들 수 있다.5
  - **제조사 정의 장치:** HID 사양은 제조사가 독자적인 `Usage Page`를 정의하는 것을 명시적으로 허용한다. 이를 통해 제조사들은 표준 HID 전송 방식을 활용하면서도 완전히 독자적인 기능을 가진 장치를 만들 수 있다.7 Microsoft의 SuperMUTT 테스트 장치가 좋은 예다.47

이러한 응용 사례들은 HID가 단순한 입력 장치 프로토콜을 넘어, 저지연 소량 데이터 교환을 위한 범용적인 '드라이버리스' 솔루션으로 자리 잡았음을 명확히 보여준다.

------

## 4. 부: 현대적 확장과 미래

HID 프로토콜은 탄생한 지 수십 년이 지났지만, 새로운 기술에 맞춰 진화하며 그 생명력을 이어가고 있다. 이는 HID가 단순히 USB에 종속된 규격이 아니라, 그 자체로 견고한 프로토콜임을 증명한다.

### 4.1  유선을 넘어: HID over I²C 프로토콜

HID는 더 이상 USB 버스에만 갇혀 있지 않다. Microsoft가 주도하여 표준화한 **HID over I²C** 프로토콜은 노트북이나 태블릿 내부의 부품 간 통신에 널리 사용되는 저속 버스인 I²C 상에서 HID를 사용할 수 있도록 확장한 것이다.48

이 기술의 주된 동기는 효율성이다. 노트북에 내장된 터치패드나 센서 허브를 위해 완전한 USB 호스트 컨트롤러를 사용하는 것은 전력 소모와 설계 복잡성 측면에서 낭비다. I²C는 훨씬 더 간단하고 저전력으로 동일한 목적을 달성할 수 있는 대안이다.50

하지만 I²C는 USB와 같은 네이티브 '플러그 앤 플레이' 열거 메커니즘이 없기 때문에, 장치 발견 방식에 변화가 필요했다. HID over I²C에서는 장치의 존재, I²C 주소, 그리고 인터럽트를 위한 GPIO 핀 정보가 시스템의 **ACPI(Advanced Configuration and Power Interface) 테이블**에 정적으로 정의된다.51

운영체제(Windows, Linux 등)는 부팅 시 이 ACPI 테이블을 읽어 장치의 존재를 인지한다. Windows는 `HIDI2C.sys`라는 전용 드라이버를 사용해 I²C 및 GPIO 컨트롤러 드라이버와 통신하며 장치를 관리하고 52, Linux는 디바이스 트리(Device Tree)를 통해 설정된 `i2c-hid` 커널 모듈을 사용한다.50 핵심은 전송 계층만 I²C로 바뀌었을 뿐, 리포트 디스크립터를 파싱하고 데이터를 처리하는 상위 HID 로직은 그대로 재사용된다는 점이다.

### 4.2  인증의 새로운 표준: FIDO2와 WebAuthn

FIDO2는 기존의 비밀번호를 대체하기 위해 등장한 강력하고 피싱에 저항성을 가진 최신 인증 표준이다.53 이 표준은 브라우저를 위한 JavaScript API인 W3C의 **WebAuthn**과, 인증장치와 클라이언트 간의 통신 프로토콜인 FIDO 얼라이언스의 **CTAP(Client to Authenticator Protocol)**으로 구성된다.53

여기서 HID의 역할이 빛을 발한다. YubiKey와 같은 물리적 보안키가 컴퓨터의 브라우저와 통신하기 위해서는 표준화된 방법이 필요하다. 만약 모든 보안키가 각기 다른 드라이버를 요구한다면 FIDO2의 보급은 불가능했을 것이다. 이 문제를 해결하기 위해 보안키 제조사들은 **FIDO U2F/FIDO2 HID 프로토콜**을 구현한다. 즉, 보안키가 호스트 컴퓨터에 자신을 범용 HID 장치로 등록하는 것이다.55

실제 인증 과정에서 사용되는 CTAP 메시지(챌린지, 서명 등)는 표준 HID 리포트 내부에 캡슐화되어 인터럽트 엔드포인트를 통해 전송된다. 브라우저는 WebAuthn API를 통해 이 HID 장치와 통신하며 등록 및 인증 절차를 수행한다.55 이것은 HID가 상위 레벨의 애플리케이션 프로토콜을 위한 순수한 **전송 계층(transport layer)**으로 활용되는 대표적인 사례로, HID의 드라이버 없는 범용성이 얼마나 강력한지를 보여준다.

이러한 현대적 확장은 HID가 USB에서 분리되어 범용적인 '드라이버리스 전송 프로토콜'로 진화했음을 명확히 보여준다. HID over I²C와 FIDO2에서의 활용 사례는 HID의 핵심 가치가 더 이상 USB 물리 계층에 있지 않음을 증명한다. 그 진정한 가치는 표준화되고 자기-서술적인 데이터 형식과, 그로 인해 보장되는 광범위한 운영체제 지원에 있다. HID는 이제 USB, I2C, 블루투스와 같은 다양한 물리적 채널 위에서 터널링되어 드라이버 없는 통신 채널을 제공하는 추상적인 프로토콜로 자리매김했다.

HID의 미래는 이러한 **추상화**와 **전문화**라는 두 가지 방향으로 나아갈 것이다. FIDO2의 경우처럼 HID는 더 복잡한 프로토콜을 위한 투명한 '파이프' 역할을 하며 추상화될 것이다. 동시에, 커스텀 게임 컨트롤러나 센서 허브의 경우처럼 HID의 기본적인 구성 요소(버튼, 축 등)는 점점 더 전문화되고 복잡한 가상 장치를 만드는 데 사용될 것이다. 이 두 가지 경향은 HID 프로토콜이 기초적인 토대이자 범용적인 전송 수단으로서의 역할을 모두 수행할 만큼 성숙하고 견고하다는 것을 보여주며, 미래에도 그 중요성이 계속될 것임을 시사한다.

------

## 5. 부: 보안: 취약점과 대응 전략

HID 프로토콜의 우아하고 편리한 설계 이면에는 어두운 그림자가 존재한다. 이 장에서는 HID의 신뢰 모델을 악용하는 치명적인 공격과 그에 대한 방어 전략을 분석한다.

### 5.1  가장 큰 위협, BadUSB와 키스트로크 주입(Keystroke Injection)

BadUSB 공격은 USB 플래시 드라이브에 저장된 데이터를 노리는 것이 아니다. 이 공격의 목표는 **장치 자체의 마이크로컨트롤러 펌웨어**다.8 공격자는 이 펌웨어를 악의적으로 재프로그래밍하여 장치가 자신의 정체를 속이도록 만든다.

공격 메커니즘은 다음과 같다. 펌웨어가 변조된 장치(플래시 드라이브, 마우스, 심지어 충전 케이블까지)가 컴퓨터에 연결되면, 악성 펌웨어는 호스트에 자신이 '키보드'라는 HID 디스크립터를 제공한다.9 호스트 운영체제는 이 디스크립터를 신뢰하고 표준 키보드 드라이버를 로드한다. 그 순간, 이 악성 장치는 시스템에 대한 키보드 제어권을 획득한다. 이후 인간은 상상할 수 없는 속도로 키 입력을 주입하여 명령 프롬프트나 PowerShell을 열고, 악성코드를 다운로드하여 실행하거나, 데이터를 유출하고, 랜섬웨어를 설치하는 등 치명적인 행위를 수행할 수 있다.9

BadUSB 공격이 특히 위험한 이유는 다음과 같다.

- **전통적인 백신으로 탐지 불가:** 백신 소프트웨어는 저장 공간의 파일을 검사할 뿐, 컨트롤러 칩의 펌웨어를 검사하지 않는다. 따라서 장치를 포맷해도 악성 펌웨어는 그대로 남는다.24
- **자동 실행 방어 무력화:** 이 공격은 저장된 프로그램을 실행하는 것이 아니라 신뢰받는 장치(키보드)를 에뮬레이션하는 방식이므로, AutoRun/AutoPlay 기능을 비활성화해도 아무 소용이 없다.24
- **USB 저장 장치 차단 정책 우회:** 기업에서 USB 대용량 저장 장치를 차단하는 정책을 사용하더라도, 악성 장치는 자신을 '키보드'나 '마우스'로 등록하기 때문에 이 정책을 쉽게 우회한다. 키보드와 마우스는 업무에 필수적이므로 거의 항상 허용되기 때문이다.10

실제로 FIN7과 같은 사이버 범죄 조직은 선물이나 공식 서한으로 위장한 악성 USB 드라이브를 우편으로 발송하여 소매, 호텔, 방위 산업체 등을 공격하는 데 BadUSB를 반복적으로 사용해왔다.9

### 5.2  방어 전략: A Multi-Layered Approach

BadUSB 공격에 대한 단 하나의 완벽한 해결책은 없다.58 효과적인 방어는 기술적, 물리적, 정책적 제어를 결합한 다층적 접근을 요구한다.

- **하드웨어 레벨 방어:**
  - **펌웨어 서명(Firmware Signing):** 가장 강력한 기술적 해결책이다. 장치 펌웨어에 암호화된 서명을 적용하고, 장치 하드웨어는 유효한 서명이 없는 펌웨어의 실행을 거부한다. 이는 펌웨어의 무단 수정을 원천적으로 차단하는 방법이다.59 높은 보안 수준을 요구하는 USB 드라이브들이 이 방식을 채택한다.
- **소프트웨어 레벨 방어:**
  - **장치 화이트리스팅/정책 강제:** Linux의 **USBGuard**와 같은 도구는 관리자가 VID/PID나 시리얼 번호를 기반으로 허용된 USB 장치 목록을 엄격하게 관리할 수 있게 해준다. 또한 화면이 잠겨 있을 때 새로운 장치의 연결을 차단하여 자리를 비운 사이 발생하는 공격을 막을 수 있다.58
  - **행동 분석/키 입력 동역학:** 공격 스크립트는 인간과 달리 완벽하게 균일하고 초인적인 속도로 타이핑한다. 이 점에 착안하여 키 입력 사이의 시간 간격을 모니터링하고, 비정상적으로 빠르거나 일정한 패턴이 감지되면 해당 장치를 악성으로 판단하여 차단할 수 있다. Google의 **UKIP(USB Keystroke Injection Protection)**이나 **Keyblock** 프로젝트가 이 원리를 이용한다.58 다만, 타이핑이 매우 빠른 사용자로 인한 오탐을 줄이기 위한 세심한 튜닝이 필요하다.58
- **물리적 및 정책적 방어:**
  - **사용자 교육:** 가장 간단하지만 가장 중요한 방어선이다. 출처가 불분명하거나 공공장소에서 발견한 USB 장치를 절대 컴퓨터에 연결하지 않도록 교육해야 한다.10 모의 훈련을 포함한 지속적인 보안 인식 캠페인이 효과적이다.10
  - **물리적 포트 차단:** 높은 보안이 요구되는 환경에서는 사용하지 않는 USB 포트를 물리적으로 막거나 에폭시로 봉인하는 것도 극단적이지만 확실한 방법이다.23

이러한 방어 전략은 HID 생태계가 근본적으로 가지고 있는 '기본적으로 신뢰(Trust-by-Default)' 모델에서 '제로 트러스트(Zero-Trust)' 모델로 전환해야 할 필요성을 시사한다. 현재의 방어책들은 대부분 공격이 발생한 후에 반응하는 방식이다. 진정으로 효과적인 장기적 방어는 이 패러다임을 역전시키는 것, 즉 모든 새로운 장치를 명시적으로 확인하기 전까지는 신뢰하지 않는 것이다. USBGuard와 같은 도구는 이러한 제로 트러스트 접근법을 소프트웨어적으로 구현한 예다. 장치는 기본적으로 차단되고, 사용자가 의식적으로 허용 정책을 만들어야만 작동한다. 이는 '알려진 악성만 차단'하는 방식에서 '알려진 선의만 허용'하는 방식으로 보안 태세를 전환하는 것이며, 이는 훨씬 더 견고한 보안 원칙이다.60

| 방어 계층      | 기술                             | 동작 원리                                                    | 장점                                                         | 단점 / 한계                                                  |
| -------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **하드웨어**   | 펌웨어 서명                      | 펌웨어에 암호화 서명을 적용하여, 서명되지 않은 펌웨어의 실행을 하드웨어 수준에서 차단. | 원천적인 펌웨어 변조 방지. 가장 강력한 보안.                 | 비용이 높고, 모든 장치에 적용되지 않음.                      |
| **소프트웨어** | 장치 화이트리스팅 (예: USBGuard) | 허용된 장치(VID/PID, 시리얼) 목록을 만들고, 목록에 없는 모든 장치를 차단. | 매우 효과적인 접근 통제. 화면 잠금 시 새로운 장치 차단 가능. | 초기 설정 및 지속적인 관리가 필요. 새로운 장치 사용 시 불편. |
| **소프트웨어** | 행동 분석 (예: UKIP)             | 비정상적으로 빠르거나 일정한 키 입력 속도를 감지하여 악성 스크립트 실행을 차단. | 기존 장치 사용에 영향을 주지 않으면서 공격을 탐지 가능.      | 빠른 타이핑 사용자로 인한 오탐 가능성. 공격자가 속도를 조절하여 우회 시도 가능. |
| **정책/인간**  | 사용자 교육 및 인식              | 출처 불명의 USB 장치를 연결하지 않도록 사용자에게 지속적으로 교육하고 경각심을 고취. | 비용이 낮고, 모든 종류의 사회 공학적 공격에 대한 기본 방어선. | 인간의 실수는 불가피하며, 100% 준수를 보장할 수 없음.        |
| **물리적**     | USB 포트 차단                    | 사용하지 않는 USB 포트를 물리적으로 봉인하여 악성 장치의 연결 자체를 불가능하게 만듦. | 완벽한 물리적 차단.                                          | 유연성이 전혀 없음. 필요한 경우에도 포트를 사용할 수 없음.   |

------

## 6. 결론

USB HID 프로토콜은 현대 컴퓨팅의 발전에 지대한 공헌을 했다. **표준화**를 통해 드라이버 지옥을 종식시켰고, **리포트 디스크립터라는 DSL**을 통해 놀라운 **유연성**을 확보했으며, **단순성**을 통해 하드웨어 혁신의 진입 장벽을 낮췄다.1 이 세 가지 핵심 가치는 HID가 수십 년이 지난 지금도 여전히 핵심적인 기술로 남아있는 이유다.

그러나 이 보고서가 지속적으로 강조했듯이, HID의 가장 위대한 특징인 '기본적으로 신뢰' 모델은 동시에 가장 큰 취약점이기도 하다. 편리성과 보안성 사이의 이 본질적인 긴장 관계는 앞으로도 계속될 프로토콜의 숙명이다.

결론적으로, HID는 더 이상 단순히 USB에 국한된 레거시 프로토콜이 아니다. I²C로의 성공적인 확장과 FIDO2와 같은 최신 보안 프로토콜의 전송 계층으로서의 채택은 HID가 살아 숨 쉬며 진화하는 인터페이스 표준임을 증명한다. 미래에도 HID는 계속해서 확장되는 수많은 장치들과 그들을 제어하는 소프트웨어 사이를 연결하는, 보이지 않지만 필수적인 다리 역할을 수행할 것이다.

#### **참고 자료**

1. usb 프로토콜 분석 - 까망눈 연구소, accessed July 24, 2025, https://jeongzero.oopy.io/fad066a1-fe95-487e-a826-003aee42a0cc
2. USB 기초지식, accessed July 24, 2025, https://t1.daumcdn.net/tistoryfile/fs5/11_29_17_5_blog130798_attach_0_23.pdf?download
3. How USB HID Makes Plug-and-Play Devices Work | Acroname, accessed July 24, 2025, https://acroname.com/blog/how-usb-hid-makes-plug-and-play-devices-work
4. USB 장치 클래스, accessed July 24, 2025, [https://m.champway.com.tw/ko/%EB%89%B4%EC%8A%A4/news/usb-device-classes](https://m.champway.com.tw/ko/뉴스/news/usb-device-classes)
5. USB Device HID Class | Overview | Universal Serial Bus (USB) | v1 ..., accessed July 24, 2025, https://docs.silabs.com/protocol-usb/1.2.0/protocol-usb-hid/
6. USB human interface device class - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/USB_human_interface_device_class
7. Using the HID class eases the job of writing USB device drivers - EDN Network, accessed July 24, 2025, https://www.edn.com/using-the-hid-class-eases-the-job-of-writing-usb-device-drivers/
8. BadUSB 취약점 분석 및 대응 방안, accessed July 24, 2025, https://koreascience.kr/article/JAKO201708733754166.pdf
9. BadUSB - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/BadUSB
10. HID attacks - PwC, accessed July 24, 2025, https://www.pwc.com/mt/en/publications/technology/hid-attacks.html
11. Human Interface Devices (HID) Specifications and Tools - USB-IF, accessed July 24, 2025, https://www.usb.org/hid
12. Understanding HID report descriptors - Who-T, accessed July 24, 2025, http://who-t.blogspot.com/2018/12/understanding-hid-report-descriptors.html
13. HID report descriptors and Linux | Marcus Folkesson Blog, accessed July 24, 2025, https://www.marcusfolkesson.se/blog/hid-report-descriptors/
14. HID(휴먼 인터페이스 디바이스)용 Windows 디바이스 드라이버 개발 - Learn Microsoft, accessed July 24, 2025, https://learn.microsoft.com/ko-kr/windows-hardware/drivers/hid/
15. Tutorial about USB HID Report Descriptors, accessed July 24, 2025, [https://programel.ru/files/Tutorial%20about%20USB%20HID%20Report%20Descriptors%20_%20Eleccelerator.pdf](https://programel.ru/files/Tutorial about USB HID Report Descriptors _ Eleccelerator.pdf)
16. USB(Universal Serial Bus) 프로토콜의 기초 지식 - 안경잡이개발자 - 티스토리, accessed July 24, 2025, https://ndb796.tistory.com/393
17. USB Human Interface Devices - OSDev Wiki, accessed July 24, 2025, https://wiki.osdev.org/USB_Human_Interface_Devices
18. USB in a NutShell - Chapter 4 - Endpoint Types - Beyondlogic, accessed July 24, 2025, https://www.beyondlogic.org/usbnutshell/usb4.shtml
19. How to Send USB Bulk Transfer Requests - Windows drivers - Learn Microsoft, accessed July 24, 2025, https://learn.microsoft.com/en-us/windows-hardware/drivers/usbcon/usb-bulk-and-interrupt-transfer
20. Bulk Transfers - USB Component - Keil, accessed July 24, 2025, https://www.keil.com/pack/doc/mw/usb/html/_u_s_b__bulk__transfers.html
21. What is the best USB transfer type (bulk, interrupt, Isochronous Transfers) to be used for implementing a USB oscilloscope? - Electrical Engineering Stack Exchange, accessed July 24, 2025, https://electronics.stackexchange.com/questions/189533/what-is-the-best-usb-transfer-type-bulk-interrupt-isochronous-transfers-to-b
22. WinUsb Interrupt and Bulk Transfer - NTDEV - OSR Developer Community, accessed July 24, 2025, https://community.osr.com/t/winusb-interrupt-and-bulk-transfer/32506
23. BadUSB: What is it and how to avoid it | ManageEngine DataSecurity Plus, accessed July 24, 2025, https://www.manageengine.com/data-security/security-threats/bad-usb.html
24. (PDF) BadUSB, the threat hidden in ordinary objects - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/331876425_BadUSB_the_threat_hidden_in_ordinary_objects
25. Part 5 - Example Device - USB Made Simple, accessed July 24, 2025, https://www.usbmadesimple.co.uk/ums_5.htm
26. Introduction to HID report descriptors - The Linux Kernel documentation, accessed July 24, 2025, https://docs.kernel.org/hid/hidintro.html
27. USB HID Report Descriptors – IBEX Resources, accessed July 24, 2025, https://ibex.tech/resources/geek-area/communications/usb/usb-hid-report-descriptors
28. Implementing a USB HID keyboard device from a USB - STMicroelectronics Community, accessed July 24, 2025, https://community.st.com/t5/stm32-mcus/implementing-a-usb-hid-keyboard-device-from-a-usb-hid-mouse/ta-p/793535
29. USB HID Class - XMOS, accessed July 24, 2025, https://www.xmos.com/file/an00129-usb-hid-class?version=latest
30. 키보드를 누르면 무슨 일이 일어날까? - velog, accessed July 24, 2025, [https://velog.io/@dhkim01/%ED%82%A4%EB%B3%B4%EB%93%9C%EB%A5%BC-%EB%88%84%EB%A5%B4%EB%A9%B4-%EB%AC%B4%EC%8A%A8-%EC%9D%BC%EC%9D%B4-%EC%9D%BC%EC%96%B4%EB%82%A0%EA%B9%8C](https://velog.io/@dhkim01/키보드를-누르면-무슨-일이-일어날까)
31. STM32 USB HID Custom Joystick/Gamepad - Phil's Lab #149 - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=umkD1piCNvc&pp=0gcJCf0Ao7VqN5tD
32. Myths about USB NKRO and how USB HID works - devever, accessed July 24, 2025, https://www.devever.net/~hl/usbnkro
33. Enabling N-key rollover on keyboard library (Leonardo) : r/arduino - Reddit, accessed July 24, 2025, https://www.reddit.com/r/arduino/comments/7lczcy/enabling_nkey_rollover_on_keyboard_library/
34. How to Program N-key Rollover atmega32u4? - Arduino Forum, accessed July 24, 2025, https://forum.arduino.cc/t/how-to-program-n-key-rollover-atmega32u4/938418
35. how many keys for custom HID descriptor? - Deskthority, accessed July 24, 2025, https://deskthority.net/viewtopic.php?t=11249
36. Myths about USB NKRO and how USB HID works | Hacker News, accessed July 24, 2025, https://news.ycombinator.com/item?id=22977077
37. 미니드라이버 및 HID클래스 드라이버 - Windows drivers - Learn Microsoft, accessed July 24, 2025, https://learn.microsoft.com/ko-kr/windows-hardware/drivers/hid/minidriver-operations
38. HID over USB 개요 - Windows drivers - Learn Microsoft, accessed July 24, 2025, https://learn.microsoft.com/ko-kr/windows-hardware/drivers/hid/hid-over-usb
39. Creating a custom USB HID to control Windows functions : r/learnprogramming - Reddit, accessed July 24, 2025, https://www.reddit.com/r/learnprogramming/comments/10qoy47/creating_a_custom_usb_hid_to_control_windows/
40. Example 2: HID Mouse and Keyboard - Pro Micro & Fio V3 Hookup Guide - SparkFun Learn, accessed July 24, 2025, https://learn.sparkfun.com/tutorials/pro-micro--fio-v3-hookup-guide/example-2-hid-mouse-and-keyboard
41. Building a USB - HID Keyboard (DIY on a prototyping board) - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=xALyq82QZIM&pp=0gcJCfwAo7VqN5tD
42. Example: Joystick Mouse | Pro Trinket as a USB HID Mouse | Adafruit Learning System, accessed July 24, 2025, https://learn.adafruit.com/pro-trinket-usb-hid-mouse/example-joystick-mouse
43. Real-time USB HID Logger: Log HID Values to Files, Excel, Database - AGG Software, accessed July 24, 2025, https://www.aggsoft.com/usb-hid-logger/
44. USB 복합장치 제작 - Engineer's LAB - 티스토리, accessed July 24, 2025, https://nexp.tistory.com/2555
45. Human Interface Device - What exactly is HID? | Guntermann & Drunck GmbH, accessed July 24, 2025, https://www.gdsys.com/en/service/campus/glossar/hid
46. Modifying USB Generic HID Example Code for Custom Report Descriptors, accessed July 24, 2025, https://community.nxp.com/t5/Kinetis-Microcontrollers/Modifying-USB-Generic-HID-Example-Code-for-Custom-Report/td-p/900868
47. Custom HID device sample - Learn Microsoft, accessed July 24, 2025, https://learn.microsoft.com/en-us/samples/microsoft/windows-universal-samples/customhiddeviceaccess/
48. I²C - Wikipedia, accessed July 24, 2025, [https://en.wikipedia.org/wiki/I%C2%B2C](https://en.wikipedia.org/wiki/I²C)
49. hid-over-i2c-protocol-spec-v1-0.docx - Microsoft, accessed July 24, 2025, https://download.microsoft.com/download/7/d/d/7dd44bb7-2a7a-4505-ac1c-7227d3d96d5b/hid-over-i2c-protocol-spec-v1-0.docx
50. Human-Interfacing Devices: HID Over I2C - Hackaday, accessed July 24, 2025, https://hackaday.com/2024/04/17/human-interfacing-devices-hid-over-i2c/
51. hid-over-i2c.txt, accessed July 24, 2025, https://www.kernel.org/doc/Documentation/devicetree/bindings/input/hid-over-i2c.txt
52. Architecture and overview for HID over the I2C transport - GitHub, accessed July 24, 2025, https://github.com/MicrosoftDocs/windows-driver-docs/blob/staging/windows-driver-docs-pr/hid/architecture-and-overview.md
53. FIDO2: Passwordless Authentication Standard - FIDO Alliance, accessed July 24, 2025, https://fidoalliance.org/fido2/
54. What Is FIDO2 and How Does It Work? FIDO Authentication Explained - Hideez, accessed July 24, 2025, https://hideez.com/blogs/news/fido2-explained
55. 표준 안내서 - HID 프로토콜 규맷 - 한국정보통신기술협회, accessed July 24, 2025, https://www.tta.or.kr/data/2017_standard/article-type_C_17.html
56. How to Protect Against BadUSB Attacks - ManageEngine Device Control Plus, accessed July 24, 2025, https://www.manageengine.com/device-control/badusb.html
57. 우편으로 USB 드라이브와 기프트 카드 발송하는 해커 그룹 - FIN7 | 분석맨의 '창조', accessed July 24, 2025, https://kr.analysisman.com/2020/04/fin7-usb.html
58. USB Keystroke Injection Protection | Google Open Source Blog, accessed July 24, 2025, https://opensource.googleblog.com/2020/03/usb-keystroke-injection-protection.html
59. IronKey Solutions Against BadUSB, accessed July 24, 2025, https://www.ironprotector.com/Solution-BadUSB.asp
60. Keyblock: a software architecture to prevent keystroke injection attacks, accessed July 24, 2025, https://sol.sbc.org.br/index.php/sbseg/article/download/19526/19354/
61. SQL Injection Prevention: 6 Ways to Protect Your Stack - eSecurity Planet, accessed July 24, 2025, https://www.esecurityplanet.com/threats/how-to-prevent-sql-injection-attacks/




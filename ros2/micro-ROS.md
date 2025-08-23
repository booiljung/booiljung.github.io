# micro-ROS



로봇 운영체제(Robot Operating System, ROS)는 그 이름에도 불구하고 전통적인 의미의 운영체제(OS)가 아니다.1 ROS는 운영체제 위에 설치되어 로봇 애플리케이션 개발을 위한 라이브러리와 도구의 집합, 즉 소프트웨어 프레임워크 또는 미들웨어(middleware)로서 기능한다.2 이 프레임워크는 이기종 하드웨어 간의 통신을 표준화하고, 소프트웨어 컴포넌트의 재사용성을 극대화하며, 복잡한 로봇 시스템의 개발을 가속화하는 데 핵심적인 역할을 수행해왔다.4

초기 ROS 1 시대에는 마이크로컨트롤러(MCU)와 같은 저사양 임베디드 장치를 ROS 네트워크에 연결하기 위해 `rosserial`이라는 패키지가 사용되었다.6 `rosserial`은 시리얼 통신을 통해 ROS 메시지를 래핑하고 전송하는 프로토콜을 제공했지만, 이는 본질적으로 MCU를 ROS 시스템의 수동적인 데이터 종단점(endpoint)으로 취급하는 방식이었다.6 MCU는 ROS의 핵심 개념인 노드(node), 서비스(service), 파라미터(parameter) 등을 직접적으로 이해하거나 실행할 수 없었으며, 단지 호스트 PC와의 브릿지(bridge)를 통해 제한적인 데이터만 교환할 수 있었다. 이로 인해 MCU는 ROS 생태계의 완전한 구성원, 즉 '일등 시민(first-class citizen)'으로 인정받지 못했다.9

이러한 한계를 극복하고 산업계의 요구사항을 반영하기 위해 등장한 ROS 2는 실시간성(real-time), 보안, 분산 시스템 지원 등 상용화 수준의 기능을 대폭 강화했다.2 데이터 분산 서비스(Data Distribution Service, DDS)를 기본 미들웨어로 채택하여 노드 간의 통신 성능과 안정성을 높였다.2 하지만 이러한 기능 강화는 상당한 리소스 요구를 동반했다. ROS 2는 최소 수백 메가바이트(MB)의 RAM을 필요로 하므로, 수십에서 수백 킬로바이트(KB)의 RAM을 가진 일반적인 MCU에는 직접 배포하는 것이 불가능했다.5 이로 인해 로봇 시스템의 가장 말단에서 센서를 읽고 액추에이터를 제어하는 핵심적인 역할을 수행하는 MCU는 여전히 ROS 2 생태계와 분리되어 있었다.


이러한 기술적 간극을 메우기 위해 탄생한 것이 바로 micro-ROS이다. micro-ROS의 핵심 개발 목표는 리소스가 극도로 제한된 MCU에 ROS 2를 이식하여, 임베디드 장치를 ROS 2 생태계의 완전한 구성원으로 통합하는 것이다.13 이는 단순히 메시지를 전달하는 것을 넘어, MCU가 자체적으로 ROS 2 노드로서 동작하며, 발행(publication)/구독(subscription), 서비스, 액션(action) 등 ROS 2의 모든 통신 패러다임을 완벽하게 지원하는 것을 의미한다.7

micro-ROS의 개발 철학은 세 가지 핵심 원칙에 기반한다. 첫째, **ROS 2와의 완벽한 호환성 및 통합**이다. micro-ROS는 기존 ROS 2 도구(예: Rviz, RQT)와 API를 그대로 사용하여 접근하고 제어할 수 있어야 한다.9 둘째,  **ROS 2 코드의 MCU로의 손쉬운 이식성**이다. 표준 ROS 2 애플리케이션 코드를 최소한의 노력으로 micro-ROS 환경으로 이식할 수 있도록 API 호환성을 유지한다.9 셋째,  **장기적인 유지보수성 확보**이다. 이를 위해 micro-ROS는 독자적인 스택을 구축하는 대신, ROS 2의 핵심 라이브러리인 RCL(ROS Client Library), RMW(ROS Middleware Interface) 등을 최대한 재사용하여 ROS 2의 발전과 보조를 맞출 수 있도록 설계되었다.2

이러한 접근 방식은 로봇 아키텍처 설계 철학의 근본적인 변화를 의미한다. 과거 중앙 집중식 고성능 컴퓨터가 주변의 저사양 MCU들을 '제어'하던 'Master-Slave' 모델에서, 이제는 센서와 액추에이터에 가장 가까운 MCU조차도 분산 시스템의 '지능형 노드'로서 독립적으로 참여하는 'Peer-to-Peer' 모델로의 전환을 가능하게 한다. 이는 각 컴포넌트의 모듈성과 독립성을 높이고, 저수준 제어 로직을 하드웨어에 더 가깝게 배치하여 통신 지연을 줄이며, 시스템 전체의 실시간성과 고장 감내(fault tolerance) 능력을 향상시키는 중요한 진보다. 따라서 micro-ROS는 'ROS 2를 MCU에 이식하는 기술'을 넘어, '차세대 분산 지능형 로봇 시스템을 구축하기 위한 핵심 아키텍처 패턴'을 제시하는 것으로 평가할 수 있다.


micro-ROS는 단순한 서드파티 프로젝트가 아니다. 이는 ROS 2 기술 운영 위원회(Technical Steering Committee) 산하의 공식 워킹 그룹인 '임베디드 워킹 그룹(Embedded Working Group)'의 지원을 받으며 개발되는 공식 프로젝트이다.16 또한, eProsima 사의 Vulcanexus와 같이 주요 ROS 2 배포판에는 기본 임베디드 툴킷으로 포함되어, 그 공식적인 위상을 공고히 하고 있다.17 이는 micro-ROS가 ROS 2 생태계의 미래 발전 방향과 긴밀하게 연결되어 있으며, 지속적인 지원과 업데이트가 보장됨을 시사한다.


micro-ROS의 아키텍처는 ROS 2와의 긴밀한 통합을 유지하면서도 임베디드 환경의 제약을 극복하기 위한 독창적인 설계를 채택하고 있다. 이는 계층적 구조, 클라이언트-에이전트 모델, 그리고 최적화된 라이브러리를 통해 구현된다.


micro-ROS의 가장 큰 구조적 특징은 ROS 2 아키텍처를 그대로 계승한다는 점이다.17 이는 완전히 새로운 스택을 만드는 대신, 기존 ROS 2의 검증된 구성 요소를 최대한 재사용하여 안정성과 호환성을 확보하기 위함이다. 아키텍처 다이어그램을 보면, 사용자 애플리케이션부터 ROS 클라이언트 라이브러리(RCL), ROS 유틸리티(RCUTILS)에 이르는 상위 계층은 표준 ROS 2와 동일한 코드를 공유한다.2

차이점은 하위 계층에 집중된다. 일반적인 ROS 2가 Linux나 Windows와 같은 범용 운영체제(GPOS)와 표준 DDS 미들웨어 위에서 동작하는 반면, micro-ROS는 실시간 운영체제(RTOS) 또는 베어메탈(bare-metal) 환경과 리소스 제약 환경에 최적화된 DDS-XRCE 미들웨어 위에서 동작한다.16 공식 문서에서 진한 파란색으로 표시되는 컴포넌트(예: 

`rclc`, Micro XRCE-DDS Client)는 micro-ROS를 위해 특별히 개발된 부분이며, 연한 파란색으로 표시되는 컴포넌트(예: `rcl`, `rcutils`)는 표준 ROS 2 스택에서 그대로 가져온 부분이다.7 이 구조 덕분에 개발자는 ROS 2에서 사용하던 개념과 API를 거의 그대로 micro-ROS 환경에서도 사용할 수 있다.


micro-ROS는 MCU의 제한된 자원을 효율적으로 사용하기 위해 **클라이언트-에이전트(Client-Agent)** 모델을 채택한다.8

- **micro-ROS Client:** MCU 측에서 실행되는 소프트웨어로, 실제 ROS 2 노드의 역할을 수행한다. 이 클라이언트는 사용자 애플리케이션과 함께 펌웨어에 포함되며, eProsima Micro XRCE-DDS Client 라이브러리를 내장하고 있다.11 클라이언트는 ROS 2 통신을 위한 요청을 생성하여 에이전트로 전송하는 역할을 담당한다.
- **micro-ROS Agent:** 호스트 PC(주로 Linux 환경)의 ROS 2 시스템에서 실행되는 독립적인 노드다. 이 에이전트는 MCU의 클라이언트와 ROS 2의 DDS 데이터스페이스(dataspace) 사이를 연결하는 중개자(broker) 또는 프록시(proxy) 역할을 수행한다.17 클라이언트로부터 XRCE-DDS 프로토콜을 통해 요청을 받으면, 이를 표준 DDS 메시지로 변환하여 ROS 2 네트워크에 전파하고, 반대로 ROS 2 네트워크의 메시지를 수신하여 클라이언트로 전달한다. 하나의 에이전트는 여러 개의 클라이언트와 동시에 통신할 수 있어, 다수의 MCU를 단일 호스트 PC에 연결하는 분산 시스템을 쉽게 구성할 수 있다.23


에이전트의 가장 중요한 역할 중 하나는 계산 집약적인 작업을 MCU 대신 처리하는 **오프로딩(offloading)**이다. ROS 2 시스템에서 새로운 노드가 네트워크에 참여하거나 다른 노드를 찾기 위해 수행하는 '검색(Discovery)' 프로세스는 상당한 계산과 네트워크 트래픽을 유발한다. MCU가 이 작업을 직접 수행하는 것은 큰 부담이므로, 에이전트가 이 역할을 대신 수행한다.2

또한, ROS 2의 강력한 기능인 서비스 품질(Quality of Service, QoS) 설정은 다양한 통신 시나리오에 맞춰 신뢰성, 내구성 등을 조절하는 복잡한 메커니즘을 포함한다. 에이전트는 이러한 QoS 관리를 담당하여 클라이언트가 더 가볍고 단순하게 동작할 수 있도록 돕는다.2 이 외에도 에이전트는 ROS 2 그래프(graph) 관리와 같은 특정 기능을 확장하여 제공하기도 한다.17

이 클라이언트-에이전트 아키텍처는 MCU의 부담을 획기적으로 줄여 저사양 하드웨어에서도 ROS 2를 구동할 수 있게 만드는 핵심 설계 결정이다. 하지만 이 설계는 본질적인 트레이드오프를 내포한다. 클라이언트의 경량화를 위해 에이전트에 대한 의존성이 극도로 높아지면서, 에이전트와 둘 사이의 전송 계층(예: UART, UDP)이 시스템 전체의 **단일 실패점(Single Point of Failure)** 및 성능 병목 지점이 될 수 있다. 실제 사용자 포럼에서는 에이전트가 내부 상태를 알 수 없는 '블랙박스'처럼 동작하여, 클라이언트와의 연결이 끊어졌을 때 이를 감지하고 안전하게 대처하기 어렵다는 심각한 문제가 제기된다.24 이는 프로토타입 수준을 넘어 상용 제품을 개발할 때, 개발자가 에이전트와 클라이언트 간의 연결 상태를 지속적으로 확인하는 '하트비트(heartbeat)'나 '워치독(watchdog)'과 같은 메커니즘을 반드시 직접 구현해야 함을 시사한다. 이 문제는 커뮤니티에서도 인지하고 있으며, `HARD_LIVELINESS_CHECK`와 같은 기능 도입이 논의되고 있다.24


micro-ROS의 사용자 API는 C 언어 기반의 클라이언트 라이브러리를 통해 제공된다. 이는 두 개의 패키지, `rcl`과 `rclc`의 조합으로 구성된다.26

- **`rcl` (ROS Client Library):** ROS 2의 가장 기본적인 C API를 제공하는 라이브러리다. micro-ROS는 `rcl`의 데이터 구조와 알고리즘을 가능한 한 그대로 사용하여 ROS 2와의 호환성을 유지하고 래퍼(wrapper)로 인한 런타임 오버헤드를 피한다.26

- **`rclc` (RCL C Extension):** `rcl` 위에 구축된 경량 확장 라이브러리로, MCU 환경에 특화된 최적화와 편의 기능을 제공한다.2

  `rclcpp`(C++)나 `rclpy`(Python)가 `rcl` 위에 새로운 객체 지향 계층을 추가하는 것과 달리, `rclc`는 새로운 타입 계층을 만들지 않고 `rcl`의 타입을 직접 사용하면서 프로그래밍을 더 쉽게 만들어주는 함수들을 제공하는 방식이다.26

`rclc`의 가장 중요한 최적화는 **메모리 관리**에 있다. `rclc`는 초기화 단계 이후에는 런타임 중에 동적 메모리 할당을 전혀 사용하지 않도록 설계되었다(zero dynamic memory allocation at runtime).2 모든 필요한 메모리는 컴파일 시점에 정적으로 할당되거나 초기화 시점에 미리 할당된 풀(pool)에서 관리된다. 이는 메모리 단편화를 방지하고, 메모리 할당에 걸리는 시간이 예측 불가능해지는 문제를 원천적으로 차단하여 시스템의 결정론적(deterministic) 동작과 실시간성을 보장하는 데 결정적인 역할을 한다. 또한, 

`rclc`는 임베디드 시스템에서 널리 사용되는 스케줄링 패턴(예: Sense-Plan-Act)을 효율적으로 구현할 수 있도록 세밀한 제어가 가능한 전용 실행기(Executor)를 제공한다.5


ROS 2와 마찬가지로 micro-ROS도 미들웨어 교체가 가능한 RMW(ROS Middleware) 인터페이스를 가지고 있다.17 기본 미들웨어로는 eProsima사가 개발한 

**Micro XRCE-DDS**가 사용된다.2 이는 국제 표준화 기구 OMG(Object Management Group)가 제정한 

**DDS-XRCE(DDS for eXtremely Resource Constrained Environments)** 표준을 준수한 구현체다.20

DDS-XRCE는 표준 DDS가 가진 풍부한 기능을 경량화하여 리소스가 극도로 제한된 환경에서도 DDS의 데이터 중심(data-centric) 통신 모델을 사용할 수 있도록 설계되었다. 주요 특징은 다음과 같다 16:

- **클라이언트-서버 아키텍처:** 앞서 설명한 클라이언트-에이전트 모델을 기반으로 동작한다.
- **낮은 리소스 소모:** 동적 메모리 할당을 배제하고, 프로파일(profile) 개념을 도입하여 빌드 시점에 불필요한 기능을 제거함으로써 메모리 사용량을 최소화할 수 있다.
- **전송 계층 독립성(Transport Agnostic):** 특정 통신 방식에 종속되지 않고 UDP, TCP, 시리얼(UART), CAN 버스 등 다양한 전송 계층 위에서 동작할 수 있다.17

이처럼 micro-ROS는 ROS 2의 검증된 아키텍처를 계승하면서도, 클라이언트-에이전트 모델과 최적화된 라이브러리, 경량 미들웨어를 통해 임베디드 환경의 제약을 극복하는 정교한 시스템 설계를 보여준다.


micro-ROS는 특정 하드웨어에 종속되지 않고 다양한 플랫폼에서 동작할 수 있도록 설계되었으며, 광범위한 하드웨어 및 소프트웨어 생태계를 지원한다.


micro-ROS를 구동하기 위한 하드웨어의 가장 중요한 제약 조건은 프로세서 아키텍처와 메모리 용량이다.

- **프로세서:** 주로 **32비트 아키텍처**의 마이크로컨트롤러(MCU)를 대상으로 한다.13 ARM Cortex-M 시리즈나 Tensilica Xtensa 시리즈 등이 대표적이다.
- **메모리(RAM):** 최소 **수십 킬로바이트(KB)의 RAM**이 필요하다.7 애플리케이션의 복잡도, 사용하는 ROS 기능(토픽 수, 메시지 크기 등), 미들웨어 설정에 따라 실제 요구량은 달라지므로, 사용자는 애플리케이션에 맞게 메모리 사용량을 튜닝해야 한다.17 예를 들어, 2KB의 RAM을 가진 Arduino UNO(ATmega328P)는 사용이 불가능하며, 최소한 32KB RAM을 가진 Arduino Zero(ATSAMD21G18)나 96KB RAM을 가진 Arduino Due(AT91SAM3X8E) 수준의 사양이 요구된다.7


micro-ROS 프로젝트는 다양한 벤더의 개발 보드를 공식적으로 지원하며, 이는 장기적인 지원(Long-Term Support, LTS)과 안정적인 동작을 보장함을 의미한다.13 이 외에도 커뮤니티에 의해 지원되는 다양한 보드들이 존재하지만, 이에 대한 공식적인 지원은 보장되지 않는다.13

공식 지원 보드는 프로젝트 계획 단계에서 어떤 하드웨어와 소프트웨어 스택을 선택할지 결정하는 데 매우 중요한 기준이 된다. 아래 표는 주요 공식 지원 보드의 사양과 지원 환경을 정리한 것으로, 개발자가 자신의 프로젝트 요구사항(예: 최소 RAM, 특정 RTOS 지원, 이더넷 통신 필요 여부)에 맞는 최적의 보드를 신속하게 선택할 수 있도록 돕는다. 이 표는 여러 문서에 흩어져 있는 정보를 통합하여 실용적인 의사결정 매트릭스를 제공한다.13

| 보드 이름 (Board Name)        | MCU (Core @ Speed)           | RAM (kB)    | 플래시 (Flash) | 공식 지원 RTOS                | 지원 통신 방식 (Transports)   |      |
| ----------------------------- | ---------------------------- | ----------- | -------------- | ----------------------------- | ----------------------------- | ---- |
| **Renesas EK-RA6M5**          | ARM Cortex-M33 @ 200 MHz     | 512         | 2 MB           | FreeRTOS, ThreadX, Bare-metal | UDP, UART, USB-CDC            |      |
| **Espressif ESP32**           | Xtensa LX6 Dual-core         | 520         | 4 MB           | FreeRTOS                      | UART, WiFi UDP, Ethernet UDP  |      |
| **Arduino Portenta H7**       | ARM Cortex-M7 + M4 Dual-core | 8192 (8 MB) | 16 MB          | - (Arduino)                   | USB, WiFi UDP                 |      |
| **Raspberry Pi Pico**         | ARM Cortex-M0+ Dual-core     | 264         | Up to 16 MB    | - (Pico SDK)                  | USB, UART                     |      |
| **ROBOTIS OpenCR 1.0**        | ARM Cortex-M7 (STM32F7)      | 320         | 1 MB           | - (Arduino)                   | USB, UART                     |      |
| **Teensy 4.0/4.1**            | ARM Cortex-M7 (iMXRT1062)    | 1024        | 2 MB           | - (Arduino)                   | USB, UART, Ethernet UDP (4.1) |      |
| **Crazyflie 2.1 Drone**       | ARM Cortex-M4 (STM32F405)    | 192         | 1 MB           | FreeRTOS                      | Custom Radio Link             |      |
| **STM32L4 Discovery kit IoT** | ARM Cortex-M4 (STM32L4)      | 128         | 1 MB           | Zephyr                        | USB, UART, Ethernet UDP       |      |
| **Olimex STM32-E407**         | ARM Cortex-M4F (STM32F407)   | 196         | 1 MB           | Zephyr, FreeRTOS, NuttX       | USB, UART, Ethernet UDP       |      |

표 1: 공식 지원 micro-ROS 하드웨어 플랫폼 및 구성 13


micro-ROS는 특정 운영체제나 개발 환경에 종속되지 않고 높은 이식성을 제공한다.

- **실시간 운영체제(RTOS):** 임베디드 시스템에서 널리 사용되는 **FreeRTOS, Zephyr, NuttX**를 공식적으로 지원한다.7 또한, C 표준 라이브러리와 POSIX(Portable Operating System Interface) 호환 인터페이스를 제공하는 모든 RTOS에 이식할 수 있도록 설계되었다.7
- **베어메탈(Bare-metal):** 운영체제를 사용하지 않는 환경에서도 micro-ROS를 직접 구동할 수 있다.7 이 경우, 하드웨어 제어를 위한 드라이버는 개발자가 직접 구현해야 한다.
- **개발 툴체인 통합:** 개발 편의성을 높이기 위해 다양한 벤더의 통합 개발 환경(IDE) 및 툴체인과 긴밀하게 통합되어 있다. **Arduino IDE, PlatformIO, Espressif의 ESP-IDF, STM32CubeIDE/MX, Renesas e2 Studio** 등 널리 사용되는 개발 환경을 위한 지원 패키지와 예제를 제공하여, 개발자가 익숙한 환경에서 쉽게 micro-ROS 애플리케이션을 개발하고 빌드, 플래시할 수 있도록 돕는다.17

이처럼 micro-ROS는 다양한 하드웨어와 소프트웨어 플랫폼을 아우르는 폭넓은 생태계를 구축함으로써, 개발자에게 높은 유연성과 선택의 폭을 제공하고 있다.


micro-ROS의 기술적 위상과 특징을 명확히 이해하기 위해서는 기존의 유사 기술 및 관련 기술과의 비교 분석이 필수적이다. 여기서는 ROS 1 시대의 표준이었던 ROSserial과의 차이점, 그리고 IoT 생태계의 핵심 프로토콜인 MQTT와의 관계를 심층적으로 분석한다.


ROS 1 기반의 임베디드 시스템을 개발해 본 경험이 있는 개발자에게는 `rosserial`이 익숙한 도구일 것이다. micro-ROS는 `rosserial`의 개념적 후계자로 볼 수 있지만, 두 기술 사이에는 단순한 버전 차이를 넘어서는 근본적인 차이가 존재한다. 이는 ROS 1에서 ROS 2로의 패러다임 전환을 그대로 반영한다.

- **대상 생태계 및 기본 철학:** 가장 근본적인 차이는 `rosserial`이 **ROS 1**을 위해 설계된 반면, micro-ROS는 **ROS 2**를 위해 설계되었다는 점이다.6

  `rosserial`은 MCU를 ROS 네트워크의 단순한 주변 장치로 취급하여 시리얼 통신을 통해 메시지를 전달하는 '브릿지' 역할에 충실했다. 반면, micro-ROS는 MCU 자체를 ROS 2 네트워크의 완전한 '노드'로 참여시키는 것을 목표로 한다.7

- **지원 기능의 범위:** `rosserial`은 메시지 발행/구독과 간단한 서비스 호출 기능만을 제공했다. 하지만 micro-ROS는 이를 넘어 **파라미터, 액션, 라이프사이클 관리, 타이머, QoS 설정** 등 ROS 2의 거의 모든 핵심 개념을 MCU 상에서 직접 지원한다.6 이는 MCU에서 훨씬 더 복잡하고 정교한 로직을 수행할 수 있게 함을 의미한다.

- **통신 프로토콜 및 신뢰성:** `rosserial`은 자체적인 커스텀 프로토콜을 사용하며, 통신 방식도 UART에 거의 한정되어 있었다.7 반면, micro-ROS는 표준화된 

  **XRCE-DDS 프로토콜**을 기반으로 하며, HDLC 프레이밍과 CRC-16-CCITT 체크섬을 사용하여 데이터 무결성과 신뢰성을 높였다.30 또한, UART뿐만 아니라 UDP(WiFi, Ethernet), SPI, CAN 등 훨씬 다양한 통신 방식을 지원하여 시스템 설계의 유연성을 크게 향상시켰다.6

아래 표는 두 기술의 주요 특징을 직접적으로 비교하여, micro-ROS가 제공하는 구체적인 기술적 이점을 명확하게 보여준다. 이 표는 ROS 1에서 ROS 2로의 마이그레이션을 고려하는 개발자에게 왜 micro-ROS를 선택해야 하는지에 대한 강력한 근거를 제공한다.

| 기능 (Feature)    | ROSserial                        | micro-ROS                                                |      |
| ----------------- | -------------------------------- | -------------------------------------------------------- | ---- |
| **대상 ROS 버전** | ROS 1                            | ROS 2                                                    |      |
| **OS 지원**       | Bare-metal                       | NuttX, FreeRTOS, Zephyr, Bare-metal                      |      |
| **통신 아키텍처** | 브릿지 방식 (Bridged)            | 브릿지 방식 (Bridged)                                    |      |
| **메시지 포맷**   | ROS 1 기반                       | CDR (from DDS) 기반                                      |      |
| **통신 링크**     | UART                             | UART, SPI, IP(UDP/TCP), CAN, WiFi, 6LoWPAN, Bluetooth 등 |      |
| **통신 프로토콜** | 커스텀 프로토콜                  | XRCE-DDS (또는 다른 RMW 구현체)                          |      |
| **기반 코드**     | 독립적 구현 (C++)                | RCL 기반 (C)                                             |      |
| **Node API**      | 커스텀 `rosserial` API           | RCL, RCLC 기반 API                                       |      |
| **콜백 실행**     | 메시지 수신 순서대로 순차적 실행 | ROS 2 Executor 또는 MCU 최적화 Executor 선택 가능        |      |
| **타이머**        | 미지원                           | ROS 2 기본 타이머 지원                                   |      |
| **라이프사이클**  | 미지원                           | 부분 지원 (전체 지원 예정)                               |      |
| **시간 동기화**   | 커스텀 방식                      | NTP/PTP 지원                                             |      |
| **신뢰성**        | 기본 미지원                      | Reliable QoS 지원 (XRCE-DDS)                             |      |

표 2: 기능 및 프로토콜 비교: micro-ROS vs. ROSserial 6


micro-ROS와 MQTT(Message Queuing Telemetry Transport)는 경쟁 관계가 아니라, 현대 로보틱스 시스템에서 각기 다른 역할을 수행하는 **상호 보완적인 관계**에 있다. 이 둘을 결합하는 아키텍처는 시스템의 통신 능력을 로봇 내부에서부터 클라우드까지 확장하는 효과적인 방법을 제시한다.

- **역할 분담:**

  - **micro-ROS (DDS-XRCE):** 주로 **로봇 내부(intra-robot)** 또는 로컬 네트워크 내의 통신을 담당한다. 고성능 데이터(카메라, 라이다) 교환과 저지연 실시간 제어에 강점을 가진 DDS의 특성을 물려받아, 로봇의 각 컴포넌트 간의 긴밀하고 빠른 상호작용에 사용된다.2
  - **MQTT:** 주로 **로봇 외부(robot-to-cloud)** 또는 원격지와의 통신을 담당한다. 경량 메시징, 발행/구독 모델, 불안정한 네트워크 환경에서의 신뢰성 있는 메시지 전달 능력 덕분에 IoT 장치나 원격 관제 시스템, 엔터프라이즈 백엔드(MES, ERP 등)와의 연동에 매우 적합하다.11

- 일반적인 연동 아키텍처:

  가장 일반적인 통합 패턴은 ROS 2 네트워크 내에 'MQTT 브릿지' 노드를 두는 것이다. 전체 데이터 흐름은 다음과 같다.11

  1. **MCU (micro-ROS Client):** 센서 데이터를 읽어 `micro-ROS Agent`로 전송한다.
  2. **호스트 PC (micro-ROS Agent):** 수신한 XRCE-DDS 메시지를 표준 DDS 메시지로 변환하여 로컬 ROS 2 네트워크에 발행한다.
  3. **호스트 PC (ROS 2 MQTT Bridge Node):** 로컬 ROS 2 네트워크에서 특정 토픽을 구독하고, 이 메시지를 MQTT 메시지 형식(예: JSON)으로 변환한다.
  4. **MQTT Broker:** 브릿지 노드로부터 받은 MQTT 메시지를 수신하여, 해당 토픽을 구독하고 있는 원격 클라이언트들에게 전달한다.
  5. **원격 시스템 (Cloud, Enterprise Systems):** MQTT Broker를 통해 로봇의 상태 데이터를 수신하거나, 반대로 로봇에게 명령을 보내기 위해 MQTT 메시지를 발행한다.

이러한 계층적 통신 스택 설계는 현대 로보틱스 시스템이 요구하는 다양한 통신 요구사항을 효과적으로 해결하는 성숙한 엔지니어링 접근 방식을 보여준다. 즉, '하나의 프로토콜이 모든 것을 해결한다'는 접근법 대신, 각 통신 계층의 역할(실시간 제어, 로컬 데이터 교환, 원격 모니터링)에 가장 적합한 기술을 지능적으로 조합하는 것이다. 이는 micro-ROS가 단독으로 사용되는 기술이 아니라, 더 큰 분산 시스템의 '실시간 엣지(real-time edge)' 계층을 담당하는 핵심 컴포넌트임을 명확히 보여준다.


micro-ROS의 실용성을 평가하기 위해서는 통신 성능, 메모리 사용량, 그리고 실시간성과 같은 정량적인 성능 지표를 분석하는 것이 중요하다. 공식 문서 및 관련 연구에서는 이러한 성능 특성에 대한 다양한 벤치마킹 결과를 제공한다.


통신 성능은 시스템의 반응성과 데이터 처리 능력에 직접적인 영향을 미친다.

- **처리량(Throughput):** 벤치마킹 결과에 따르면, 통신 처리량은 사용하는 전송 매체에 크게 의존한다. 예상대로 **이더넷(Ethernet)**이 가장 높은 처리량을 제공하지만, 이는 높은 전력 소모를 동반한다. 반면, **시리얼(UART)** 통신은 처리량이 낮지만 훨씬 적은 에너지를 소모하는 장점이 있다.32 따라서 개발자는 애플리케이션의 데이터 전송 요구량과 전력 제약 사이에서 적절한 균형점을 찾아 전송 매체를 선택해야 한다.
- **지연 시간(Latency):** 클라이언트와 에이전트 간의 왕복 지연 시간(Round-Trip Time, RTT)은 시스템의 실시간성을 결정하는 핵심 요소다.33 한 연구에서는 로봇 팔의 물리적 모델과 디지털 트윈(Digital Twin) 간에 micro-ROS를 통해 메시지를 교환했을 때, 평균 77.67ms의 지연 시간이 측정되었다.34 또 다른 연구에서는 ABB의 mYuMi 로봇과 micro-ROS 기반 MCU 간의 통신 안정성을 평가하며, RTT가 대부분 안정적으로 유지됨을 확인했다.35 이러한 지연 시간은 네트워크 환경, 메시지 크기, 에이전트의 부하 등 다양한 요인에 의해 영향을 받는다.


리소스가 제한된 MCU 환경에서 메모리 점유율은 시스템 구현 가능성을 결정하는 중요한 척도다.

- **정적 메모리(Static Memory):** 컴파일 시점에 결정되는 메모리 사용량으로, 코드와 전역 변수 등이 차지하는 공간이다. 벤치마킹 결과에 따르면, 6LoWPAN과 같이 복잡한 네트워크 프로토콜 스택을 사용할 경우 정적 메모리 사용량이 크게 증가하는 경향을 보인다.32
- **동적 메모리(Dynamic Memory):** 프로그램 실행 중에 힙(heap) 영역에서 할당 및 해제되는 메모리다. micro-ROS는 실시간성을 보장하기 위해 런타임 중의 동적 메모리 할당을 의도적으로 회피한다. 대부분의 동적 할당은 **초기화 단계에서만 발생**하며, 할당되는 메모리 블록의 크기나 개수도 많지 않다.17 이는 런타임 중 메모리 단편화나 예측 불가능한 지연을 방지하는 데 기여한다.
- **전체 메모리 사용량:** eProsima의 발표에 따르면, 512 바이트 크기의 메시지를 발행/구독하는 간단한 애플리케이션의 경우, Micro XRCE-DDS 클라이언트는 약 75KB의 플래시 메모리와 3KB의 RAM을 사용한다.21 이는 micro-ROS가 수십 KB 수준의 RAM을 가진 MCU에서도 충분히 구동될 수 있음을 보여준다.


micro-ROS의 핵심적인 장점 중 하나는 실시간 시스템을 지원하는 것이다.

- **결정론적 동작(Deterministic Behavior):** RTOS 중 하나인 NuttX의 스케줄러를 추적한 벤치마킹 결과에 따르면, micro-ROS 애플리케이션의 작업 실행 순서는 매번 동일하게 유지되어 **결정론적인 동작**을 보였다.32 이는 동일한 입력에 대해 항상 동일한 시간 내에 동일한 출력을 보장해야 하는 실시간 제어 시스템에 필수적인 특성이다.
- **빠른 컨텍스트 스위칭:** 동일한 벤치마크에서 RTOS의 컨텍스트 스위칭 시간은 평균 **10 마이크로초(μs)**로 매우 빠르게 측정되었다.32 이는 여러 작업을 신속하게 전환하며 병렬적으로 처리할 수 있음을 의미한다.
- **`rclc Executor`를 통한 스케줄링:** micro-ROS가 제공하는 `rclc Executor`는 단순한 콜백 실행기를 넘어, 임베디드 시스템에서 요구되는 정교한 스케줄링 패턴을 지원하도록 설계되었다. 예를 들어, 센서 데이터를 받아 처리하고 액추에이터를 구동하는 'Sense-Plan-Act' 파이프라인의 지연 시간을 최소화하기 위해 콜백의 실행 순서를 보장하거나, 여러 주기로 동작하는 센서 데이터들을 동기화하고, 시간에 민감한 작업을 다른 작업보다 먼저 처리하도록 우선순위를 부여하는 등의 제어가 가능하다.5

하지만, micro-ROS를 사용한다고 해서 실시간성이 자동으로 보장되는 것은 아니다. 이는 프레임워크 자체의 기능뿐만 아니라, 개발자가 선택하는 전송 계층, RTOS 설정, 그리고 애플리케이션 설계 방식에 의해 크게 좌우되는 **'시스템 전체의 속성'**이다. 예를 들어, 한 연구에서는 micro-ROS의 기본 실행 모델이 작업의 우선순위를 고려하지 않아, 높은 우선순위의 작업이 낮은 우선순위의 작업 때문에 지연될 수 있는 '우선순위 역전(priority inversion)'과 유사한 문제가 발생할 수 있음을 지적했다. 그리고 이를 해결하기 위해 우선순위 기반의 체인-인식 스케줄러(PoDS, Priority-driven and chain-aware scheduler)를 별도로 제안하기도 했다.12

결론적으로, micro-ROS는 실시간 애플리케이션을 구축하기 위한 강력한 '도구'와 '기반'을 제공하지만, 진정한 실시간 시스템을 구축하기 위해서는 개발자가 ①빠르고 안정적인 전송 매체를 선택하고, (2)RTOS의 스케줄링 정책을 깊이 이해하며, (3)`rclc Executor`를 정교하게 설정하고, ④애플리케이션의 콜백 함수가 시스템을 블로킹하지 않도록 주의 깊게 설계하는 등 시스템 전반에 걸친 종합적인 노력이 필수적이다.


micro-ROS는 학술 연구 단계를 넘어 산업 현장과 다양한 실제 로봇 프로젝트에 활발하게 적용되며 그 실용성을 입증하고 있다.


주요 로봇 및 자동화 기업들은 micro-ROS를 자사 제품과 솔루션에 통합하여 시스템의 모듈성과 효율성을 높이고 있다.

- **Bosch Rexroth:** 독일의 자동화 전문 기업인 보쉬 렉스로스는 자사의 오프로드(off-road) 차량 제어 솔루션에 micro-ROS를 도입했다. 이를 통해 저수준의 센서 및 액추에이터 제어, 실시간 제어 루프, 안전 기능 등을 ROS 2 생태계와 원활하게 통합하였다.5
- **ABB:** 글로벌 로봇 기업 ABB는 자사의 차세대 모바일 협업 로봇인 'mYuMi' 프로토타입에 micro-ROS를 적용했다. mYuMi의 복잡한 시스템에서 외부 센서(예: 거리 측정 센서)와 액추에이터(예: 서보 모터)를 제어하기 위해 MCU를 사용하고, 이를 micro-ROS를 통해 메인 컴퓨터의 ROS 2 시스템과 연결함으로써 전체 시스템 아키텍처를 모듈화하고 개발 효율을 높였다.35
- **Renesas:** 반도체 기업 Renesas는 자사의 RA 시리즈 MCU를 홍보하기 위한 데모에서 micro-ROS를 적극 활용한다. 대표적으로 로봇 팔 'OpenManipulator'를 제어하는 데모를 통해, 자사 MCU와 micro-ROS의 결합이 실제 로봇 애플리케이션에서 얼마나 효과적으로 동작하는지를 보여주었다.19
- **CAPRA Robotics:** 덴마크의 로봇 스타트업인 CAPRA Robotics는 자사의 상용 자율 이동 로봇(AMR)인 'CAPRA HIRCUS'에 micro-ROS를 채택하여, 로봇의 저수준 제어 시스템을 ROS 2 기반의 고수준 소프트웨어와 통합했다.25
- **기타 산업 분야:** 이 외에도 폴란드의 Łukasiewicz-PIAP 연구소는 스마트 창고 물류 시스템에 micro-ROS를 적용했으며, Hydrasystem은 고정밀 GNSS 모듈에 micro-ROS를 통합하여 ROS 2와의 연동을 구현하는 등 다양한 산업 분야에서 활용 사례가 등장하고 있다.36


학계에서도 micro-ROS는 활발한 연구 주제로 다루어지고 있으며, 로봇 공학의 여러 분야에서 새로운 가능성을 탐색하는 데 사용되고 있다.

- **이동 로봇 2D 맵핑:** micro-ROS를 탑재한 이동 로봇을 제작하고, 이 로봇이 수집한 센서 데이터를 ROS 2의 SLAM(Simultaneous Localization and Mapping) 패키지와 연동하여 2D 지도를 생성하는 연구가 수행되었다.37 이는 저사양 MCU가 고수준의 로봇 지능 알고리즘과 어떻게 협력할 수 있는지를 보여주는 좋은 예다.
- **모바일 로봇 플랫폼 아키텍처 연구:** 한 박사 과정 연구에서는 기존의 단일 컴퓨터 기반 제어 시스템을 가진 이동 로봇을 micro-ROS 기반의 분산 제어 시스템으로 전환하고, 이 과정에서 발생하는 성능과 기능성의 변화를 평가하는 연구를 계획하고 있다.2
- **디지털 트윈(Digital Twin):** 실제 로봇 팔의 움직임을 가상 공간에 실시간으로 복제하는 디지털 트윈을 구축하는 데 ROS와 함께 micro-ROS가 활용되었다. 이 경우, 로봇 팔의 각 관절에 연결된 MCU가 micro-ROS를 통해 관절 상태를 실시간으로 발행하고, 디지털 트윈은 이 정보를 구독하여 움직임을 동기화한다.34
- **로봇 기구학(Kinematics) 분산 처리:** 로봇 팔의 순기구학 및 역기구학 계산과 같이 상대적으로 복잡한 연산을 MCU에서 직접 수행할 것인지, 아니면 MCU는 단순한 관절 제어만 담당하고 기구학 계산은 `ros2_control` 프레임워크를 통해 호스트 PC에서 처리할 것인지에 대한 아키텍처적 논의가 활발하게 이루어지고 있다.38

이러한 적용 사례들을 종합해 보면, 현재 micro-ROS 기술이 가장 확실한 가치를 제공하는 영역은 '저수준 하드웨어 제어'와 '분산된 하드웨어의 ROS 2 생태계 통합'이라는 것을 알 수 있다. 산업계와 학계 모두 MCU를 센서 데이터 수집과 액추에이터 구동을 위한 '하드웨어 추상화 계층'으로 활용하고, SLAM, 내비게이션, 기구학 계산과 같은 복잡한 알고리즘은 여전히 호스트 PC에서 처리하는 구조를 주로 채택하고 있다. 이는 현재 micro-ROS의 기술적 '스위트 스폿(sweet spot)'이 어디인지를 명확히 보여준다. 앞으로 MCU의 성능이 더욱 향상되고 micro-ROS의 기능(예: C++ API 지원)이 확장됨에 따라, 더 복잡한 지능형 로직이 MCU, 즉 엣지(edge) 단으로 이동하는 '완전한 엣지 컴퓨팅' 사례가 점차 늘어날 것으로 전망된다.


micro-ROS 프로젝트는 개발자들이 쉽게 기술을 배우고 적용할 수 있도록 풍부한 데모와 튜토리얼을 제공한다. 공식 데모에는 이동 로봇 Kobuki, 소형 드론 Crazyflie, ToF(Time-of-Flight) 거리 센서, 로봇 팔 OpenManipulator-X, 모션 플래닝 프레임워크 MoveIt 2 연동 등이 포함되어 있어, 다양한 로봇 애플리케이션에서의 활용법을 익힐 수 있다.40 또한 Arduino, Teensy, ESP32, STM32 등 대중적인 개발 보드에 대한 상세한 시작 가이드를 제공하여 커뮤니티의 참여와 기술 확산을 촉진하고 있다.40


어떤 기술이든 실제 프로젝트에 적용했을 때 드러나는 장점과 한계가 있다. micro-ROS 역시 사용자 커뮤니티의 경험을 통해 그 실용적인 가치와 해결해야 할 과제들이 명확해지고 있다.


사용자들이 공통적으로 언급하는 micro-ROS의 가장 큰 장점은 개발 생산성과 시스템 유연성이다.

- **완벽한 ROS 2 통합:** micro-ROS 노드는 표준 ROS 2 노드와 동일하게 취급되므로, Rviz, RQT, `ros2` 커맨드라인 도구 등 기존 ROS 2의 강력한 개발 및 디버깅 도구를 그대로 활용할 수 있다.7 이는 개발 과정을 크게 단축시키고, 기존 ROS 2 코드베이스에 새로운 임베디드 장치를 쉽게 통합할 수 있게 해준다.24
- **코드 재사용성 및 모듈성:** 과거에는 MCU 종류나 통신 방식이 바뀔 때마다 호스트 PC의 드라이버 코드를 수정하거나 새로 작성해야 했다. 하지만 micro-ROS는 하나의 에이전트가 여러 종류의 MCU 클라이언트를 동시에 처리할 수 있으므로, 하드웨어에 종속적인 드라이버 코드의 중복을 크게 줄일 수 있다. 이는 시스템의 모듈성을 높이고 유지보수를 용이하게 한다.24
- **폭넓은 하드웨어 및 통신 지원:** 다양한 MCU, RTOS, 그리고 UART, UDP, CAN 등 여러 통신 방식을 지원하기 때문에, 프로젝트의 요구사항에 맞춰 최적의 하드웨어와 통신 방식을 유연하게 선택하고 조합할 수 있다.6
- **임베디드 시스템의 고유 장점 활용:** 일반적인 리눅스 기반의 싱글 보드 컴퓨터(SBC)에 비해, MCU는 **저전력, 저발열, 빠른 부팅 시간**이라는 명확한 장점을 가진다.42 micro-ROS는 이러한 MCU의 장점을 ROS 2 생태계 안에서 온전히 누릴 수 있게 해준다.2


장점만큼이나 실사용자들이 겪는 어려움과 한계점도 명확하게 드러나고 있으며, 이는 주로 운영 신뢰성 및 디버깅과 관련이 있다.

- **에이전트의 "블랙박스" 특성:** 사용자들이 겪는 가장 큰 문제점으로, **에이전트가 내부 상태나 클라이언트와의 연결 상태에 대한 충분한 정보를 제공하지 않는다는 점**이 꼽힌다.24 예를 들어, MCU의 전원이 꺼지거나 시리얼 케이블이 빠져 통신이 두절되었을 때, 에이전트는 이를 명시적으로 알려주지 않는다. 이로 인해 호스트 PC의 상위 노드들은 하드웨어가 비정상 상태임을 인지하지 못하고 계속 동작하여 위험한 상황을 초래할 수 있다. 이는 특히 상용 시스템에서 치명적인 약점이 될 수 있다.

- **디버깅의 복잡성:** 문제가 발생했을 때, 원인이 클라이언트(MCU) 측에 있는지, 에이전트 측에 있는지, 아니면 둘 사이의 통신 계층에 있는지 파악하기가 까다로울 수 있다. 에이전트라는 중간 계층이 존재하기 때문에 문제 해결 과정이 더 복잡해진다.24 또한, MCU에서 발생하는 로그(log)를 ROS 2의 표준 로깅 시스템(

  `rosout`)과 완벽하게 통합하는 기능이 아직 미흡하여, 디버깅 워크플로우에 불편함을 줄 수 있다.24

- **연결 상태 관리의 부담:** 앞서 언급한 '블랙박스' 문제 때문에, 안정적인 시스템을 구축하기 위해서는 개발자가 직접 **연결 상태를 감시하는 로직을 추가로 구현**해야 한다. 예를 들어, MCU가 주기적으로 'heartbeat' 메시지를 보내고, 호스트 PC의 별도 노드가 이 메시지를 감시하여 일정 시간 동안 수신되지 않으면 시스템을 안전하게 정지시키는 등의 '우회책(workaround)'이 필요하다.24 이러한 추가 개발 부담은 micro-ROS 도입으로 얻는 개발 편의성의 이점을 일부 상쇄시킬 수 있다.

- **미성숙한 기능:** 런타임 중에 파라미터를 동적으로 설정하는 기능이나 표준화된 로깅 등 일부 ROS 2의 편의 기능들이 아직 micro-ROS에 완전히 구현되지 않았다.24

이러한 장점과 한계를 종합해 보면, micro-ROS는 현재 **'개발 편의성'과 '운영 신뢰성' 사이의 긴장 관계**에 놓여 있음을 알 수 있다. 개발 초기 단계에서는 ROS 2와의 완벽한 통합이라는 막대한 편의성을 제공하지만, 시스템을 안정적으로 운영하고 문제를 해결해야 하는 후반 단계에서는 추상화된 에이전트 계층이 오히려 장애물이 될 수 있는 것이다. 이는 micro-ROS가 학술 연구 및 프로토타이핑 단계를 넘어, 높은 신뢰성을 요구하는 상용 제품 수준으로 나아가는 '과도기'에 있음을 시사한다. 다행히 이러한 한계점들은 커뮤니티에 의해 인지되고 있으며, 이를 해결하기 위한 `HARD_LIVELINESS_CHECK`와 같은 기능들이 활발히 논의되고 있다.25 이는 프레임워크가 사용자의 피드백을 통해 점차 성숙해지고 있음을 보여주는 긍정적인 신호이며, 개발자는 이 프레임워크를 '완성된 제품'이 아닌 '진화하는 생태계'로 이해하고 접근해야 한다.


micro-ROS는 빠르게 발전하는 프로젝트로, 현재의 한계를 극복하고 미래 로보틱스의 요구사항을 충족시키기 위한 다양한 연구와 개발이 진행 중이다. 그 발전 방향은 소프트웨어적 기능 개선을 넘어, 미들웨어 혁신과 하드웨어 가속이라는 두 가지 큰 축으로 전개되고 있다.


ROS 2 임베디드 워킹 그룹(Embedded WG)을 중심으로 다음과 같은 기능 개선이 활발하게 논의 및 개발되고 있다.

- **신뢰성 향상:** 사용자들이 가장 큰 문제로 지적한 연결 안정성 문제를 해결하기 위해, 클라이언트의 활성 상태(Liveliness)를 에이전트가 주기적으로 확인하고 연결이 끊어졌을 때 관련 리소스를 정리하는 기능(`HARD_LIVELINESS_CHECK`)이 개발되고 있다.24
- **C++ API 지원:** 현재 micro-ROS는 C 언어 API(`rclc`)만 공식적으로 제공하지만, 많은 ROS 개발자들이 C++에 익숙하기 때문에 MCU 환경에 최적화된 C++ API 지원에 대한 논의가 꾸준히 진행되고 있다.43 이는 코드의 추상화 수준을 높이고 개발 생산성을 향상시키는 데 기여할 것이다.
- **신규 기능 및 플랫폼 확장:** 노드 그래프 관리 기능 강화, 멀티스레딩 지원, 대용량 메시지 처리 능력 개선, 사용자가 직접 통신 방식을 구현할 수 있는 커스텀 전송 계층 기능 등 소프트웨어 스택이 지속적으로 개선되고 있다.25 또한, RTEMS, Microsoft Azure RTOS 등 새로운 RTOS에 대한 지원을 추가하며 플랫폼 생태계를 확장하고 있다.25


현재 micro-ROS의 아키텍처는 DDS-XRCE와 에이전트에 크게 의존하고 있다. 하지만 DDS는 본래 유선 로컬 네트워크 환경에 최적화되어 있어, Wi-Fi나 4G/5G와 같이 지연 시간이 길고 불안정한 무선 네트워크 또는 인터넷을 통한 원격 통신에는 약점을 보인다.44

이러한 한계를 근본적으로 해결하기 위해 등장한 것이 **Zenoh**이다. Zenoh는 ZettaScale Technologies에서 개발한 차세대 통신 미들웨어로, **MCU에서부터 엣지 컴퓨터, 클라우드 서버에 이르기까지 단일 프로토콜로 끊김 없는(end-to-end) 통신을 제공하는 것**을 목표로 한다.45 Zenoh는 DDS의 한계를 극복하고 다양한 네트워크 환경에서 높은 성능과 낮은 오버헤드를 보여주어, ROS 2의 차세대 기본 미들웨어(Tier 1 RMW)로 공식 채택되었다.3 Zenoh가 micro-ROS 생태계에 완전히 통합되면, 현재의 복잡한 클라이언트-에이전트-브릿지 구조가 훨씬 단순화되고, MCU와 클라우드 간의 직접적인 통신이 가능해져 현재 micro-ROS가 가진 많은 신뢰성 및 성능 관련 문제점들이 자연스럽게 해결될 수 있다. 이는 micro-ROS의 미래에 가장 큰 영향을 미칠 변화 중 하나로 주목받고 있다.46


소프트웨어 최적화를 넘어, 통신 자체를 하드웨어의 영역으로 옮기려는 연구 또한 활발히 진행 중이다. 이는 **FPGA(Field-Programmable Gate Array)**와 같은 재구성 가능한 하드웨어를 사용하여 ROS 2의 통신 스택을 가속하는 접근 방식이다.

'ROS 2 on a Chip'이라는 비전 아래 진행되는 이 연구들은 ROS 2의 메시지 전달 인프라를 하드웨어 칩에 직접 구현하여, 소프트웨어로는 도달하기 힘든 수준의 성능을 달성하는 것을 목표로 한다.47 FPGA로 구현된 프로토타입은 기존 CPU 기반의 소프트웨어 구현에 비해 

**통신 속도를 평균 62배 이상, 에너지 효율을 500배 이상 향상**시켰으며, 최대 지연 시간(worst-case latency)을 **30,000배 이상** 줄여 수 마이크로초(μs) 단위의 결정론적이고 등시성(isochronous) 있는 통신을 보장할 수 있음을 보여주었다.48 이러한 하드웨어 가속 기술은 극도로 낮은 지연 시간이 요구되는 고속 정밀 제어, 다개체 로봇 군집 제어 등 미래 로보틱스 응용 분야의 새로운 지평을 열 것으로 기대된다.


micro-ROS는 유럽 연합(EU)의 연구 혁신 프로젝트를 통해 시작된 만큼, 유럽 내 로보틱스 기술 생태계 구축에 중요한 역할을 하고 있다. 장기적인 비전은 micro-ROS를 중심으로 ROS 2 로봇의 모델링, 프로그래밍, 시뮬레이션, 네트워크 성능 테스트, 클라우드/엣지 연동, MCU 통합 등을 모두 아우르는 **완전한 형태의 유럽형 ROS 배포판 모델**을 구축하는 것이다.50 이는 기술 주권을 확보하고 유럽 로봇 산업의 경쟁력을 강화하려는 전략적 목표와 맞닿아 있다.

이처럼 micro-ROS의 미래는 현재의 소프트웨어적 한계를 극복하기 위한 미들웨어 혁신과 하드웨어 가속이라는 담대한 비전을 향해 나아가고 있다. 이는 임베디드 로보틱스의 패러다임을 또 한 번 바꿀 잠재력을 가지고 있으며, 개발자들은 이러한 기술 동향을 주시하며 미래 시스템 아키텍처의 확장성을 고려해야 할 것이다.



본 고찰을 통해 분석한 바와 같이, micro-ROS는 리소스가 제한된 마이크로컨트롤러를 현대적인 ROS 2 생태계에 통합하는 가장 성숙하고 표준적인 솔루션으로 자리매김했다. 이는 단순한 라이브러리를 넘어, 센서와 액추에이터 단까지 지능을 확장하는 분산 지능형 로봇 시스템을 구축하기 위한 핵심 아키텍처 패턴을 제시한다.

현재 micro-ROS는 ROS 2와의 완벽한 통합을 통해 높은 '개발 편의성'을 제공한다는 강력한 장점을 가지고 있다. 하지만 에이전트의 블랙박스 특성이나 디버깅의 복잡성과 같은 문제로 인해, 높은 수준의 '운영 신뢰성'을 요구하는 상용 제품에 적용하기에는 아직 성숙 과정에 있는 과도기적 기술이라고 평가할 수 있다. 그럼에도 불구하고 활발한 커뮤니티와 명확한 발전 로드맵을 통해 이러한 한계점들을 꾸준히 개선해 나가고 있다.


새로운 로봇 프로젝트에 micro-ROS 도입을 고려하는 개발자에게 다음과 같은 사항을 제언한다.

1. **적극적인 도입 고려:** 신규 ROS 2 기반 프로젝트에서 MCU를 사용해야 한다면, `rosserial`과 같은 레거시 방식을 고수하거나 독자적인 통신 프로토콜을 개발하는 데 시간을 낭비하기보다, **micro-ROS를 최우선으로 고려해야 한다.** 이는 ROS 2 생태계와의 완벽한 통합, 커뮤니티 지원, 장기적인 발전 가능성 측면에서 가장 합리적인 선택이다.
2. **신뢰성 최우선 설계:** micro-ROS의 가장 큰 약점은 에이전트와의 연결 안정성 문제다. 따라서 프로토타입 단계를 넘어선 개발에서는, **연결 단절을 감지하고 이에 대응하는 안전 장치(Fail-safe) 및 복구 메커니즘을 애플리케이션 레벨에서 반드시 설계**해야 한다. 주기적인 하트비트 체크, 워치독 타이머 구현 등은 필수적으로 고려되어야 한다.
3. **하드웨어 및 전송 계층의 신중한 선택:** 시스템의 실시간 요구사항과 데이터 처리량을 면밀히 분석하여, 이에 맞는 성능과 전력 효율을 제공하는 MCU와 통신 방식을 프로젝트 초기에 신중하게 선택해야 한다. UART의 간편함에만 의존하기보다, 필요하다면 UDP(이더넷, Wi-Fi)나 CAN-FD와 같은 더 빠르고 안정적인 통신 방식을 검토해야 한다.
4. **커뮤니티와의 지속적인 소통:** micro-ROS는 매우 빠르게 발전하는 오픈소스 프로젝트다. 따라서 GitHub 저장소, ROS Discourse 포럼, 임베디드 워킹 그룹 회의록 등 공식 채널을 통해 최신 개발 동향, 버그 수정, 새로운 기능에 대한 정보를 지속적으로 확인하고, 문제 발생 시 커뮤니티에 적극적으로 질문하고 참여하는 자세가 중요하다.18


micro-ROS는 ROS 2의 영향력을 대형 산업용 로봇과 고성능 컴퓨터의 영역을 넘어, 우리 주변의 모든 사물에 내장될 수 있는 소형 임베디드 장치까지 확장하는 기폭제 역할을 하고 있다. 이는 로봇 기술의 적용 범위를 IoT, 스마트 가전, 웨어러블 기기, 소형 자율 로봇 등 무한한 영역으로 넓히는 중요한 전환점이다.

앞으로 Zenoh와 같은 차세대 미들웨어, 그리고 'ROS 2 on a Chip'과 같은 하드웨어 가속 기술과의 결합은 현재 micro-ROS가 가진 기술적 한계를 뛰어넘어, 더욱 지능적이고, 즉각적으로 반응하며, 에너지 효율적인 차세대 로봇 시스템의 등장을 예고하고 있다. micro-ROS는 의심할 여지 없이 그 거대한 변화의 중심에 서 있을 것이며, 임베디드 로보틱스의 미래를 이끌어갈 핵심 기술로 계속해서 발전해 나갈 것이다.


1. ROS: Robot Operating System에 대해 알아보기 - HMG Developers, accessed July 6, 2025, https://developers.hyundaimotorgroup.com/blog/364
2. Overview Analysis of Micro-ROS System as an Embedded Solution for Microcontrollers in Automatics and Robotics Applications, accessed July 6, 2025, http://pe.org.pl/articles/2024/12/48.pdf
3. the official ROS docs, accessed July 6, 2025, https://docs.ros.org/
4. ROS 기본 개념, accessed July 6, 2025, https://dawon-project.tistory.com/m/4
5. Micro-ROS – bringing the most popular robotics ... - Bosch Global, accessed July 6, 2025, https://www.bosch.com/stories/bringing-robotics-middleware-onto-tiny-microcontrollers/
6. Comparison to related approaches - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/overview/comparison/
7. [Micro-ROS] 특징 - Interactics - 티스토리, accessed July 6, 2025, [https://huroint.tistory.com/entry/Micro-ROS-%ED%8A%B9%EC%A7%95](https://huroint.tistory.com/entry/Micro-ROS-특징)
8. Micro-ROS - Not Black Magic, accessed July 6, 2025, https://notblackmagic.com/bitsnpieces/micro-ros/
9. Robot Operating System (ROS) | PDF | Microcontroller - Scribd, accessed July 6, 2025, https://www.scribd.com/document/737515102/Robot-Operating-System-ROS
10. ROS 2 Key Challenges and Advances: A Survey of ROS 2 Research, Libraries, and Applications - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/384942758_ROS_2_Key_Challenges_and_Advances_A_Survey_of_ROS_2_Research_Libraries_and_Applications
11. MQTT & micro-ROS: Building Efficient Robotics Applications | by EMQ Technologies, accessed July 6, 2025, https://emqx.medium.com/mqtt-micro-ros-building-efficient-robotics-applications-150a7ff40df4
12. Improving Real-Time Performance of Micro-ROS with Priority-Driven Chain-Aware Scheduling - MDPI, accessed July 6, 2025, https://www.mdpi.com/2079-9292/13/9/1658
13. micro-ROS.github.io/_docs/overview/hardware/index.md at master / micro-ROS/micro-ROS.github.io / GitHub, accessed July 6, 2025, https://github.com/micro-ROS/micro-ROS.github.io/blob/master/_docs/overview/hardware/index.md
14. www.renesas.com, accessed July 6, 2025, [https://www.renesas.com/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus/micro-ros-solutions#:~:text=micro%2DROS%20is%20an%20open,robots%2C%20IoT%20sensors%20and%20devices.](https://www.renesas.com/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus/micro-ros-solutions#:~:text=micro-ROS is an open,robots%2C IoT sensors and devices.)
15. Introduction to the micro-ROS Framework - FIWARE, accessed July 6, 2025, https://www.fiware.org/2020/05/18/introduction-to-the-micro-ros-framework/
16. Features and Architecture - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/overview/features/
17. 6. micro-ROS Documentation - Vulcanexus 1.0.0 documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/humble/rst/microros_documentation/index.html
18. micro-ROS | ROS 2 for microcontrollers, accessed July 6, 2025, https://micro.ros.org/
19. ROS 2 Embedded Working Group - meeting #25 - March 2023 - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=wm055hb7Xeg
20. micro-ROS: a robotic framework - FIWARE, accessed July 6, 2025, https://www.fiware.org/wp-content/uploads/2020/04/Micro-ROS-Information-Flyer.pdf
21. [Micro-ROS] Micro XRCE-DDS 특징 - Interactics - 티스토리, accessed July 6, 2025, [https://huroint.tistory.com/entry/Micro-ROS-Micro-XRCE-DDS-%ED%8A%B9%EC%A7%95](https://huroint.tistory.com/entry/Micro-ROS-Micro-XRCE-DDS-특징)
22. Architecture and Implementation of Micro-ROS with OpenAMP on an Heterogeneous Multi-core Processor, accessed July 6, 2025, https://sasimi.jp/new/sasimi2024/files/archive/pdf/p26_R1-5.pdf
23. Micro-ROS - Floris Jousselin - GitHub Pages, accessed July 6, 2025, https://robotcopper.github.io/micro-ROS/
24. Our conclusions on trying to implement micro-ros in our robot ..., accessed July 6, 2025, https://discourse.ros.org/t/our-conclusions-on-trying-to-implement-micro-ros-in-our-robot/24690
25. Latest Embedded topics - ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/c/embedded/9?page=1
26. Introduction to Client Library - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/concepts/client_library/introduction/
27. Overview - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/tutorials/programming_rcl_rclc/overview/
28. Supported Hardware | micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/overview/hardware/
29. ROS 2 on Microcontrollers with micro-ROS & Zephyr // Zephyr Tech Talk #011 - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=E8KZEuUCtVA
30. Micro XRCE-DDS compared to rosserial | micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/concepts/middleware/rosserial/
31. MQTT & micro-ROS: Building Efficient Robotics Applications | EMQ, accessed July 6, 2025, https://www.emqx.com/en/blog/mqtt-and-micro-ros
32. Results | micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/concepts/benchmarking/results/
33. 6.4. Benchmarking - Vulcanexus 1.0.0 documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/microros_documentation/benchmarking/benchmarking.html
34. Unity and ROS as a Digital and Communication Layer for Digital Twin Application: Case Study of Robotic Arm in a Smart Manufacturing Cell - MDPI, accessed July 6, 2025, https://www.mdpi.com/1424-8220/24/17/5680
35. MICRO-ROS FOR MOBILE ROBOTICS SYSTEMS - MDU, accessed July 6, 2025, https://mdh.diva-portal.org/smash/get/diva2:1670378/FULLTEXT01.pdf
36. Robot Operating System (ROS): The Complete Reference [7] 3031090616, 9783031090615, accessed July 6, 2025, https://dokumen.pub/robot-operating-system-ros-the-complete-reference-7-3031090616-9783031090615.html
37. 2D Mapping of Mobile Robot Based on micro-ROS - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/366263361_2D_Mapping_of_Mobile_Robot_Based_on_micro-ROS
38. ros2 - micro_ros and control - Robotics Stack Exchange, accessed July 6, 2025, https://robotics.stackexchange.com/questions/109658/micro-ros-and-control
39. micro-ROS enabled robot and kinematics - Embedded - ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/t/micro-ros-enabled-robot-and-kinematics/22662
40. Overview - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/tutorials/core/overview/
41. Why don't we use ROS? - ROS General - Open Robotics Discourse, accessed July 6, 2025, https://discourse.ros.org/t/why-dont-we-use-ros/3161
42. Micro-ROS vs multiple compute modules running ROS2? - Reddit, accessed July 6, 2025, https://www.reddit.com/r/ROS/comments/ouomkn/microros_vs_multiple_compute_modules_running_ros2/
43. ROS 2 Embedded WG meetings, accessed July 6, 2025, https://discourse.ros.org/t/ros-2-embedded-wg-meetings/15460
44. [2309.07496] Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS2 Systems - arXiv, accessed July 6, 2025, https://arxiv.org/abs/2309.07496
45. ROSCon 2024 Highlights: Conversations with Dexory, Asimovo, and ZettaScale - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=z1qAGeU1nFo
46. ROSCon 2025, accessed July 6, 2025, https://roscon.ros.org/
47. ROS 2-centric - KRS 1.0 documentation - GitHub Pages, accessed July 6, 2025, https://xilinx.github.io/KRS/sphinx/build/html/docs/features/ros2centric.html
48. ROS 2 on a Chip, Achieving Brain-Like Speeds and Efficiency in Robotic Networking - arXiv, accessed July 6, 2025, https://arxiv.org/html/2404.18208v1
49. ROS 2 on a Chip, Achieving Brain-Like Speeds and Efficiency in Robotic Networking - arXiv, accessed July 6, 2025, https://arxiv.org/abs/2404.18208
50. micro-ROS - ROS 2 into microcontrollers | PPT - SlideShare, accessed July 6, 2025, https://www.slideshare.net/slideshow/microros-ros-2-into-microcontrollers/257160077




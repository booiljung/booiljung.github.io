[드론 FCC](./index.md)
# BeagleBone AI-64 기반 PX4 비행 제어 컴퓨터 개발 방안에 대한 심층 고찰



전통적인 드론 시스템의 핵심인 비행 제어 컴퓨터(Flight Controller, FC)는 주로 마이크로컨트롤러(MCU)를 기반으로 설계되었다. 이러한 MCU 기반 FC는 안정적인 비행 제어라는 본연의 임무에 충실하지만, 복잡한 자율 비행 알고리즘, 실시간 인공지능(AI) 추론, 다중 센서 데이터의 정교한 융합과 같은 고수준의 연산을 수행하기에는 컴퓨팅 성능의 명백한 한계를 드러낸다. 이 한계를 극복하기 위해 업계에서는 실시간 제어를 담당하는 FC와 고수준 연산을 처리하는 컴패니언 컴퓨터(Companion Computer)를 분리하는 이중 보드 아키텍처가 보편적으로 채택되어 왔다.1 그러나 이러한 구조는 시스템의 물리적 부피와 무게를 증가시킬 뿐만 아니라, 두 보드 간의 통신 병목으로 인한 지연 시간(latency) 발생, 전력 소모 증가, 그리고 전체 시스템의 복잡성 증대라는 새로운 문제들을 야기한다.

이러한 맥락에서, BeagleBone AI-64와 같은 고성능 이종(heterogeneous) 시스템 온 칩(SoC) 기반의 단일 보드 컴퓨터(SBC)는 새로운 가능성을 제시한다. 이 보드들은 강력한 애플리케이션 프로세서(AP), 실시간 코프로세서, 그리고 AI 가속기를 단일 칩에 통합함으로써, 기존의 분산된 두 역할을 하나의 플랫폼으로 통합하여 차세대 드론 아키텍처를 구현할 강력한 잠재력을 지닌다.4 이는 단순히 하드웨어를 통합하는 것을 넘어, 각기 다른 특성을 가진 코어들이 고속 내부 버스를 통해 데이터를 유기적으로 공유하며 동작하는 새로운 패러다임으로의 전환을 의미한다. AI 기반의 상황 인식, 동적 경로 계획, 실시간 궤적 추종, 그리고 정밀 모터 제어가 하나의 보드 내에서 긴밀하게 결합될 수 있는 '통합 지능형 제어' 시스템의 등장은 외부 통신 병목 현상을 원천적으로 제거하고, 전력 효율을 극대화하며, 시스템 전체의 반응성을 획기적으로 향상시킬 수 있다.


BeagleBone AI-64의 강력한 성능에도 불구하고, 이를 드론의 단독 FC로 활용하기 위해서는 근본적인 기술적 난제를 해결해야 한다. 보드의 주 프로세서인 Arm Cortex-A72 코어에서 동작하는 Debian과 같은 범용 운영체제(General-Purpose Operating System, GPOS)는 본질적으로 평균 처리량(average throughput)을 높이는 데 최적화되어 있어, 스케줄링의 비결정성(non-determinism)을 내포한다. 즉, 특정 작업의 실행 시간이 예측 불가능하며, 예기치 않은 지연이 발생할 수 있다.7 반면, 드론의 비행 제어, 특히 자세를 안정시키는 내부 루프는 수백 헤르츠(Hz)에 달하는 주기로 오차 없이 정밀하게 실행되어야 하는 경성 실시간(hard real-time) 요구사항을 갖는다.8 이 제어 루프의 주기를 단 한 번이라도 놓치거나 실행 시간의 변동성(jitter)이 클 경우, 기체는 불안정해지거나 최악의 경우 추락에 이를 수 있다.

따라서 본 고찰의 핵심 기술 문제는 'GPOS의 유연성 및 고성능'과 '비행 제어의 경성 실시간성'이라는 상충하는 두 요구사항 간의 기술적 간극을 BeagleBone AI-64라는 단일 플랫폼 내에서 어떻게 효과적으로 메울 것인가에 있다.


본 보고서는 BeagleBone AI-64의 독특한 이종 멀티코어 아키텍처를 심층적으로 분석하고, 세계적으로 가장 널리 사용되는 오픈소스 오토파일럿인 PX4를 이 플랫폼에 성공적으로 포팅하기 위한 구체적인 방안을 제시하는 것을 목적으로 한다. 이를 위해, PX4의 실시간성 요구사항을 충족시키기 위한 다각적인 접근법을 고찰하고, 각 방안의 기술적 타당성과 장단점을 비교 분석한다.

궁극적으로 본 보고서는 Cortex-A72, Cortex-R5F, 그리고 Programmable Real-Time Unit (PRU) 등 이종 코어들의 역할을 최적으로 분담하는 하이브리드 시스템 아키텍처를 제안한다. 또한, 개발 과정 전반에 걸친 기술적 로드맵을 제시하고, 보드에 내장된 AI 가속기와 로봇 운영체제(ROS2)를 연동하여 단순한 비행 제어를 넘어선 지능형 자율 비행 시스템으로 확장하는 방안까지 탐구한다. 이 보고서는 기존 MCU 기반 FC의 한계를 넘어 고성능 컴퓨팅과 AI를 비행 제어에 직접 통합하고자 하는 고급 엔지니어와 연구자들에게 실질적인 기술적 지침을 제공할 것이다.


BeagleBone AI-64를 드론의 FC로 활용하기 위한 타당성을 평가하기 위해서는 먼저 그 핵심 두뇌인 Texas Instruments (TI) TDA4VM SoC의 아키텍처를 비행 제어의 관점에서 깊이 있게 이해해야 한다. TDA4VM은 본래 자동차의 첨단 운전자 보조 시스템(ADAS)이나 산업용 로보틱스와 같이 높은 신뢰성과 저지연 처리가 요구되는 애플리케이션을 위해 설계된 K3 멀티코어 아키텍처 플랫폼에 기반한다.10 이 SoC의 가장 큰 특징은 단일 칩 내에 목적이 다른 여러 종류의 프로세싱 코어를 집적한 이종 멀티코어 구조라는 점이다.


TDA4VM SoC는 단순히 연산 속도가 빠른 코어를 여러 개 모아놓은 것이 아니라, 각기 다른 특성과 강점을 지닌 코어들을 유기적으로 결합하여 시스템 전체의 효율과 성능을 극대화하도록 설계되었다. 이는 개발자에게 단순한 선택지를 제공하는 것을 넘어, "고수준 작업은 AP에서, 실시간 제어는 실시간 코어에서, 정밀 I/O는 전용 프로세서에서 처리하라"는 명확한 설계 지침을 제시한다. 이 설계 철학을 이해하고 따르는 것이 성공적인 FC 개발의 첫걸음이다.

- **Dual-core Arm Cortex-A72 (최대 2.0GHz):** 이 64비트 애플리케이션 프로세싱 유닛(APU)은 시스템의 '중앙 두뇌' 역할을 수행한다. 32KB L1 데이터 캐시, 48KB L1 명령어 캐시, 그리고 1MB의 공유 L2 캐시를 갖추고 있어 13, Debian GNU/Linux와 같은 고기능 운영체제를 원활하게 구동할 수 있다.14 드론 시스템에서는 복잡한 항법 알고리즘(예: SLAM), 임무 계획, ROS(Robot Operating System) 노드 실행, 지상관제소(GCS)와의 고속 데이터 통신(MAVLink over IP), 그리고 방대한 양의 비행 데이터 로깅과 같은 고수준 작업을 처리하는 데 이상적이다.15

- **Six Arm Cortex-R5F MCUs (최대 1.0GHz):** 이 32비트 실시간 마이크로컨트롤러(RT-MCU) 코어들은 시스템의 '소뇌'와 같은 역할을 담당하며, 실시간 처리와 기능 안전(functional safety)을 위해 특별히 설계되었다. 각 코어는 독립적으로 또는 두 개가 하나의 쌍으로 록스텝(lock-step) 모드로 동작하여 연산 오류를 감지할 수 있다. 이 코어들은 캐시 미스로 인한 예측 불가능한 지연을 피하기 위해 Tightly-Coupled Memory(TCM)를 사용하며 18, Linux 커널의 스케줄링 지연에 전혀 영향을 받지 않는 완전한 결정론적(deterministic) 연산 환경을 제공한다. 따라서 Zephyr나 FreeRTOS와 같은 실시간 운영체제(RTOS)를 직접 포팅하여, IMU 데이터 융합(EKF)이나 자세/위치 제어 루프와 같은 PX4의 핵심적인 비행 제어 로직을 실행하기에 가장 적합한 후보이다.10

- **Dual-core Programmable Real-Time Unit and Industrial Communication SubSystem (PRU-ICSSG):** 시스템의 '손과 발'에 해당하는 이 초저지연 I/O 프로세서는 TDA4VM 아키텍처의 백미라 할 수 있다. 보드에는 2개의 PRU-ICSSG 모듈이 탑재되어 있으며, 각 모듈은 여러 개의 32비트 RISC 코어(PRU 및 RTU)를 포함한다.11 PRU 코어는 200MHz 이상으로 동작하며 파이프라인이 없어, 모든 명령어가 단일 클럭 사이클(약 5 나노초) 내에 예측 가능하게 실행된다.19 가장 중요한 특징은 GPIO 핀을 레지스터 조작을 통해 단일 사이클에 직접 제어할 수 있다는 점이다. 이는 Linux의 사용자 공간에서는 상상할 수 없는 나노초(

  ns) 단위의 정밀한 타이밍 제어를 가능하게 한다.8 드론 FC에서는 다음과 같은 경성 실시간 I/O 작업에 독보적인 성능을 발휘한다.

  - **모터/서보 제어:** ESC(Electronic Speed Controller)를 제어하기 위한 정밀한 PWM(Pulse Width Modulation), DShot, 또는 기타 디지털 프로토콜 신호를 생성한다.
  - **RC 수신기 입력:** SBUS, PPM, Crossfire(CRSF) 등 다양한 RC 수신기 프로토콜의 시리얼 프레임을 정밀한 타이밍으로 디코딩한다.
  - **보조 센서 인터페이스:** Quadrature Encoder Pulse (QEP)와 같은 고속의 디지털 센서 입력을 처리한다.

- **AI 및 비전 가속기:** TDA4VM은 이름에서 알 수 있듯이 AI 연산에 특화되어 있다. C7x 부동소수점 벡터 DSP와 Matrix Multiply Accelerator(MMA)는 결합하여 최대 8 TOPS(Tera Operations Per Second)의 AI 추론 성능을 제공한다.4 이 하드웨어는 TI의 Deep Learning(TIDL) 라이브러리를 통해 효율적으로 활용될 수 있다.21 또한, Vision Processing Accelerators(VPAC)는 카메라 센서로부터 들어오는 원시(raw) 이미지 데이터에 대한 전처리(예: Image Signal Processing, ISP)를 하드웨어 수준에서 가속하여 Cortex-A72의 부하를 크게 줄여준다. 이러한 가속기들은 비전 기반 자율 비행, 실시간 객체 탐지 및 추적, 시각적 SLAM(Simultaneous Localization and Mapping)과 같은 고급 기능을 드론에 구현하는 데 핵심적인 역할을 한다.


BeagleBone AI-64는 강력한 내부 프로세싱 능력과 더불어, 다양한 외부 센서 및 액추에이터와 연결되기 위한 풍부한 I/O 인터페이스를 제공한다.

- **범용 I/O (GPIO, I2C, SPI, UART):** P8과 P9에 위치한 두 개의 46핀 Cape 확장 헤더를 통해 다수의 I/O 핀에 접근할 수 있다. 이 헤더들은 BeagleBone Black과의 하드웨어 Cape 호환성을 어느 정도 유지하도록 설계되었다.11 TDA4VM SoC는 10개의 I2C, 11개의 SPI, 12개의 UART 등 풍부한 시리얼 인터페이스를 내장하고 있어 11, IMU, 기압계, 자력계, GNSS(GPS), 텔레메트리 모듈 등 드론에 필요한 거의 모든 종류의 센서를 동시에 연결하기에 충분하다. 하지만, 특정 핀에서 원하는 I/O 기능을 사용하기 위해서는 Linux 커널의 Device Tree Overlay(.dtbo 파일)를 수정하여 핀 기능 다중화(pinmux) 설정을 변경해야 한다. 이는 특히 여러 개의 UART나 SPI 버스를 동시에 활성화해야 할 때 복잡한 작업이 될 수 있으며, 관련 커뮤니티 포럼에서도 이에 대한 논의가 활발히 이루어지고 있다.23
- **PWM 및 타이머:** TDA4VM SoC는 6개의 ePWM(Enhanced Pulse-Width Modulation) 서브시스템, 3개의 eCAP(Enhanced Capture) 모듈, 3개의 eQEP(Enhanced Quadrature Encoder Pulse) 모듈을 내장하고 있다.10 이론적으로는 이 하드웨어 타이머들을 사용하여 모터 및 서보 제어가 가능하지만, Linux의 표준 드라이버를 통해 이들을 제어할 경우 커널 스케줄링 지연으로 인해 PRU를 사용하는 것만큼의 정밀도와 저지연성을 보장하기는 어렵다. 따라서 정밀한 모터 제어를 위해서는 PRU를 활용하는 것이 권장된다.
- **메모리 및 스토리지:** 4GB의 LPDDR4 RAM과 16GB의 내장 eMMC 플래시 스토리지는 고기능 OS, 복잡한 ROS 애플리케이션, 고해상도 지도 데이터, 그리고 장시간의 비행 로그를 저장하기에 충분한 공간과 성능을 제공한다.13 MicroSD 카드 슬롯을 통해 저장 공간을 추가로 확장하거나, 부팅 미디어로 활용할 수도 있다.
- **전원 및 물리적 사양:** 보드의 안정적인 동작을 위해서는 5V 전압에 최소 3A 이상의 전류를 공급할 수 있는 전원이 필수적이다.10 USB Type-C 포트를 통해서도 전원 공급이 가능하지만, 여러 주변 장치를 연결할 경우 외부 DC 잭을 사용하는 것이 권장된다. 보드의 크기는 104mm x 83mm이며, 방열판을 포함한 순수 무게는 약 193g이다.10 이는 일반적인 취미용 소형 드론보다는, 충분한 탑재 공간과 추력을 가진 중대형급 연구 개발용 드론 플랫폼에 더 적합한 물리적 사양임을 시사한다.

아래 표는 비행 제어 컴퓨터 개발자의 관점에서 BeagleBone AI-64의 핵심 기술 사양을 요약한 것이다.

**Table 1: BeagleBone AI-64 핵심 기술 사양 (비행 제어 관점)**

| 구성 요소 (Component)                 | 세부 사양 (Specification)                  | 드론 FC에서의 주요 역할 (Primary Role in Drone FC)        | 관련 소스 (Source Snippets) |
| ------------------------------------- | ------------------------------------------ | --------------------------------------------------------- | --------------------------- |
| **APU (Application Processing Unit)** | Dual-core Arm Cortex-A72 @ 2.0GHz          | 고수준 항법, 임무 계획, ROS 노드, GCS 통신                | 13                          |
| **RT-MCU (Real-Time MCU)**            | 6x Arm Cortex-R5F @ 1.0GHz                 | 핵심 제어 루프, 센서 융합 (RTOS 환경)                     | 11                          |
| **I/O Processor**                     | 2x PRU-ICSSG (Programmable Real-Time Unit) | 정밀 PWM/DShot 생성, RC 신호 디코딩                       | 8                           |
| **AI Accelerator**                    | C7x DSP + MMA (최대 8 TOPS)                | 실시간 객체 탐지, 시각적 SLAM                             | 4                           |
| **Memory/Storage**                    | 4GB LPDDR4, 16GB eMMC, MicroSD 슬롯        | OS, 애플리케이션, 비행 로그, 지도 데이터 저장             | 15                          |
| **Key Interfaces**                    | I2C, SPI, UART, ePWM, eQEP, CAN, CSI, DSI  | IMU/GPS/센서 연결, 모터/서보 제어, 카메라/디스플레이 연결 | 10                          |


BeagleBone AI-64라는 강력한 하드웨어 플랫폼에 생명을 불어넣기 위해서는 그에 걸맞은 정교한 소프트웨어, 즉 오토파일럿 스택이 필요하다. PX4는 모듈화된 설계와 플랫폼 독립성으로 인해 이러한 고성능 보드에 이식하기에 가장 적합한 오픈소스 오토파일럿 중 하나이다.25


PX4의 아키텍처는 '반응형 시스템(reactive system)'이라는 설계 철학에 기반한다. 이는 시스템의 모든 기능이 독립적이고 교체 가능한 컴포넌트(모듈)로 분리되어 있으며, 이들 간의 통신이 비동기식 메시지 패싱을 통해 이루어짐을 의미한다.26

- **모듈화 구조:** PX4는 센서 드라이버, 상태 추정기(estimator), 제어기(controller), 항법(navigator) 등 다양한 기능들이 각각 독립적인 태스크 또는 스레드로 실행되는 구조를 가진다. 예를 들어, `sensors` 모듈은 IMU 데이터를 읽고, `ekf2` 모듈은 이 데이터를 받아 기체의 자세와 위치를 추정하며, `mc_att_control` 모듈은 추정된 자세와 목표 자세 간의 오차를 바탕으로 모터 출력을 계산한다.25

- **uORB 메시징 버스:** 이 모듈들 간의 통신은 uORB(micro Object Request Broker)라는 이름의 publish-subscribe 메시징 버스를 통해 이루어진다.26 특정 모듈이 

  `sensor_combined`라는 토픽(topic)에 새로운 IMU 데이터를 발행(publish)하면, 이 토픽을 구독(subscribe)하고 있던 다른 모든 모듈들(예: `ekf2`, `logger`)이 즉시 해당 데이터를 받아 자신의 작업을 수행할 수 있다. 이 방식은 각 모듈 간의 의존성을 낮추고, 시스템 전체를 병렬화하며, 데이터의 흐름을 명확하게 만들어 유연성과 확장성을 극대화한다.27

- **POSIX 호환성:** PX4의 미들웨어는 NuttX, Linux, macOS 등 POSIX(Portable Operating System Interface) 호환 API를 제공하는 모든 운영체제 위에서 동작하도록 설계되었다. Linux 환경에서는 각 PX4 모듈이 별도의 프로세스나 스레드로 실행되며, uORB는 프로세스 간 통신을 위해 공유 메모리(shared memory)를 기반으로 구현된다.26


PX4는 이미 다수의 Linux 기반 SBC를 지원하고 있으며, 이는 BeagleBone AI-64 포팅 작업에 중요한 선례가 된다. 공식적으로 또는 실험적으로 지원되는 보드 목록에는 Raspberry Pi (Navio2, PilotPi 쉴드 사용)와 BeagleBone Blue가 포함되어 있다.31 이러한 기존 포팅 사례의 소스 코드는 새로운 보드 지원 계층(Board Support Layer)을 개발하는 데 있어 매우 유용한 참조 자료가 된다.

PX4 포팅 가이드에 따르면, Linux 기반 보드의 포팅은 NuttX 기반 보드(예: Pixhawk 시리즈)의 포팅과 중요한 차이점을 보인다. NuttX의 경우, PX4 빌드 프로세스가 운영체제 자체를 포함하여 전체 펌웨어 이미지를 생성한다. 반면, Linux 보드의 경우 PX4는 이미 보드에 설치된 Linux 이미지를 전제로 하며, OS와 커널의 구성은 PX4의 빌드 범위에 포함되지 않는다.30 대신, 포팅 개발자는 해당 Linux 시스템이 I2C, SPI 등의 버스를 통해 센서에 접근할 수 있도록 필요한 커널 드라이버나 userspace I/O 라이브러리를 사전에 준비해야 할 책임이 있다.

이러한 PX4의 접근 방식은 개발자에게 더 큰 유연성을 제공하는 동시에, 더 큰 책임을 부여한다. NuttX 기반 FC에서는 PX4 프레임워크가 실시간 스케줄링을 포함한 저수준 제어를 상당 부분 담당한다. 그러나 Linux 환경에서 PX4는 자신을 하나의 '사용자 공간 애플리케이션'으로 간주하고, 비행 제어에 필수적인 '실시간성 보장'이라는 중대한 책임을 운영체제, 즉 포팅 개발자에게 전가한다. 따라서 BeagleBone AI-64에 PX4를 성공적으로 포팅하는 것은 단순히 코드를 컴파일하고 실행하는 것을 넘어, Linux 시스템 자체를 비행 제어에 적합한 '실시간 시스템'으로 개조하는 심도 있는 엔지니어링 과제를 포함한다. 이 점을 인지하지 못하면, 프로젝트는 겉보기에는 동작하는 것처럼 보이지만 실제 비행에서는 예측 불가능한 지연으로 인해 실패할 확률이 매우 높다.


BeagleBone AI-64를 새로운 PX4 타겟으로 추가하기 위해서는 다음과 같은 체계적인 절차를 따라야 한다. 기존의 `beaglebone/blue` 33나 `raspberrypi/pilotpi` 35 디렉토리 구조를 참고하여 진행한다.

1. **보드 설정 파일 디렉토리 생성:** PX4 소스 코드의 최상위 디렉토리에서 `boards/beaglebone/ai-64/` 와 같은 새로운 디렉토리 구조를 생성한다. 이 디렉토리는 BeagleBone AI-64와 관련된 모든 설정 파일들을 담는 공간이 된다.

2. **빌드 설정 파일 (`.px4board`) 작성:** `boards/beaglebone/ai-64/` 디렉토리 내에 `beaglebone_ai64_default.px4board` 파일을 생성한다. 이 파일은 보드의 기본 정보를 정의하고, PX4의 각 기능(GPS, Telemetry, RC 등)이 어떤 시리얼 포트(예: `/dev/ttyS2`, `/dev/ttyS4`)에 매핑되는지를 명시한다.36

3. **CMake 빌드 스크립트 추가:** `cmake/configs/` 디렉토리에 `posix_beaglebone_ai64_default.cmake` 파일을 추가한다. 이 스크립트는 `make beaglebone_ai64_default` 명령이 실행될 때 호출되며, AArch64 크로스 컴파일러 툴체인을 지정하고, 보드에 특화된 컴파일 옵션이나 라이브러리 링크 설정을 정의하는 역할을 한다.34

4. **초기화 및 부팅 스크립트 (`rcS`) 구성:** `ROMFS/px4fmu_common/init.d/` 디렉토리에 위치한 `rcS` 파일은 PX4가 부팅될 때 실행되는 메인 셸 스크립트이다.29 이 스크립트를 직접 수정하거나, 보드별로 분기되는 새로운 스크립트(

   `rc.board_sensors` 등)를 생성하여 BeagleBone AI-64에 연결된 센서 드라이버(IMU, 기압계 등), PWM 출력 드라이버, 그리고 기타 필수 모듈들이 올바른 순서로 시작되도록 설정해야 한다.36 예를 들어, 

   `mpu9250 start -I /dev/i2c-1`과 같은 명령을 통해 특정 I2C 버스에 연결된 IMU 드라이버를 실행시킬 수 있다.

5. **보드별 설정 헤더 (`board_config.h`) 정의:** `boards/beaglebone/ai-64/src/` 디렉토리에 `board_config.h` 헤더 파일을 생성한다. 이 파일은 소프트웨어 코드 내에서 하드웨어에 종속적인 설정들을 정의하는 데 사용된다. 예를 들어, 보드의 상태를 표시하는 LED의 GPIO 핀 번호, 부저(buzzer) 핀, 전원 관리 관련 핀 등을 `#define` 지시어를 통해 상수로 정의한다.36

이러한 절차를 통해 PX4 빌드 시스템은 BeagleBone AI-64를 새로운 하드웨어 타겟으로 인식하고, 해당 보드에 맞는 실행 파일을 생성할 수 있게 된다. 그러나 이것은 포팅의 시작일 뿐이며, 진정한 도전은 다음 장에서 논의할 실시간 제어 구현에 있다.



앞서 언급했듯이, 표준 Linux 커널은 드론 비행 제어와 같은 경성 실시간 애플리케이션에 내재적인 한계를 가진다. 커널은 시스템의 평균적인 응답성과 전체 처리량을 극대화하도록 설계되었기 때문에, 개별 태스크의 실행 완료 시간을 보장하지 않는다.7 높은 우선순위를 가진 실시간 태스크라 할지라도, 커널 내부의 긴 임계 구역(critical section) 처리, 다른 디바이스 드라이버의 인터럽트 서비스 루틴(ISR) 실행, 또는 페이지 폴트(page fault)와 같은 예측 불가능한 이벤트로 인해 수 밀리초(ms)에 달하는 지연(latency)을 겪을 수 있다.38

드론의 자세 제어 루프는 일반적으로 250Hz에서 400Hz(주기 4ms ~ 2.5ms)로 동작하며, 이는 매 주기마다 센서 데이터를 읽고, 상태를 추정하며, 모터 출력을 계산하여 업데이트해야 함을 의미한다. 이 과정에서 발생하는 지연 시간의 최대값(worst-case execution time, WCET)과 변동성(jitter)이 엄격하게 제어되지 않으면, 제어 루프가 불안정해져 기체의 진동을 유발하거나 심각한 경우 제어 불능 상태에 빠져 추락으로 이어질 수 있다. 따라서 BeagleBone AI-64의 Cortex-A72 코어에서 PX4를 안정적으로 구동하기 위해서는, 표준 Linux의 비결정성 문제를 해결하고 예측 가능한 실시간성을 확보하는 것이 가장 중요한 기술적 과제이다. 이를 해결하기 위한 세 가지 주요 방안을 심층적으로 고찰한다.


가장 직접적이고 널리 알려진 접근법은 표준 Linux 커널을 실시간 커널로 변환하는 PREEMPT_RT 패치를 적용하는 것이다.

- **원리:** PREEMPT_RT 패치는 Linux 커널의 스케줄링 메커니즘을 근본적으로 수정하여 실시간 성능을 향상시킨다. 주요 변경 사항은 다음과 같다: (1) 커널 코드 내에서 선점을 비활성화하는 임계 구역의 대부분을 선점 가능하도록 변경한다. (2) 하드웨어 인터럽트 핸들러(Hard-IRQ)를 우선순위를 가진 커널 스레드(Threaded IRQs)로 변환하여, 높은 우선순위의 실시간 태스크가 낮은 우선순위의 인터럽트 처리보다 먼저 실행될 수 있도록 한다. (3) 스핀락(spinlock)과 같은 동기화 메커니즘을 우선순위 상속(priority inheritance)을 지원하는 실시간 뮤텍스(rtmutex)로 대체하여, 우선순위 역전(priority inversion) 문제를 방지한다. 이러한 변경들을 통해 PREEMPT_RT는 Linux를 연성 실시간(soft real-time) 시스템에서 경성 실시간에 근접한(near hard real-time) 시스템으로 탈바꿈시킨다.39

- **성능 예측:** 시스템의 실시간 성능을 정량적으로 평가하는 데는 `cyclictest`라는 표준 벤치마크 도구가 널리 사용된다. `cyclictest`는 주기적으로 실행되도록 설정된 실시간 스레드가 목표한 기상 시간과 실제로 깨어난 시간 사이의 차이, 즉 스케줄링 지연 시간을 마이크로초(μs) 단위로 정밀하게 측정한다.42 BeagleBone AI-64와 유사한 ARM 기반 SBC에서 PREEMPT_RT 커널을 테스트한 연구 결과들은 매우 고무적이다. 예를 들어, Raspberry Pi 3 및 BeagleBone AI에서 PREEMPT_RT 패치를 적용했을 때 최대 지연 시간이 150µs 이내로 크게 감소했으며 9, Cortex-A53 기반 시스템에서는 최대 17µs, Cortex-A9 시스템에서는 54µs라는 우수한 결과가 보고되기도 했다.45 Cortex-A72의 성능을 고려할 때, 잘 튜닝된 시스템에서는 최대 지연 시간을 수십 µs 수준으로 유지할 수 있을 것으로 기대된다. 이는 400Hz 제어 루프의 주기인 2500µs에 비해 충분히 짧은 시간이므로, Cortex-A72 코어에서 PX4의 핵심 제어 루프를 직접 실행할 수 있는 이론적 가능성을 강력하게 시사한다.

- **구현:** BeagleBone AI-64용 Linux 커널 소스 트리를 확보한 후, 해당 커널 버전에 맞는 PREEMPT_RT 패치 파일을 다운로드하여 적용해야 한다. 그 후, 커널 설정 유틸리티(`make menuconfig`)를 실행하여 `General setup ---> Preemption Model` 메뉴에서 `Fully Preemptible Kernel (Real-Time)` 옵션을 선택하고, 새로운 커널 이미지를 크로스 컴파일해야 한다. 이 과정은 커널 빌드에 대한 깊은 이해를 필요로 한다.39

- **한계:** PREEMPT_RT 패치는 커널 자체의 스케줄링 지연을 획기적으로 줄여주지만, 만능 해결책은 아니다. 부실하게 작성된 디바이스 드라이버가 장시간 동안 인터럽트를 비활성화하거나, 시스템 관리 인터럽트(SMI)와 같은 펌웨어 레벨의 이벤트가 발생하면 PREEMPT_RT 커널이라도 속수무책으로 지연이 발생할 수 있다.6 또한, ESC 제어를 위한 PWM 신호 생성과 같이 수십 나노초(

  ns) 단위의 극도로 정밀한 타이밍 제어가 요구되는 작업에는 여전히 부적합하다.


PREEMPT_RT가 해결할 수 없는 초정밀, 초저지연 I/O 요구사항을 해결하기 위한 최적의 솔루션은 TDA4VM에 내장된 PRU를 활용하는 것이다.

- **원리:** PRU는 Cortex-A72 ARM 코어와는 완전히 독립적으로 동작하는 200MHz 32비트 RISC 프로세서이다. PRU의 명령어 셋은 파이프라인 구조가 없어 모든 명령어가 예측 가능한 결정론적 시간(5ns) 내에 완료된다. 특히, GPIO 핀의 상태를 읽거나 쓰는 작업을 단일 클럭 사이클에 수행할 수 있는 능력은 PRU의 가장 강력한 특징이다. 이는 Linux 커널을 거치지 않고 하드웨어를 직접 제어함으로써 나노초 수준의 정밀한 I/O 타이밍을 보장한다.8

- **Linux와의 연동 아키텍처:** 최신 Linux 커널은 `remoteproc`이라는 표준 프레임워크를 통해 PRU와 같은 원격 프로세서(remote processor)를 관리한다. ARM 코어에서 실행되는 `pru_rproc` 커널 드라이버는 사용자 공간의 요청에 따라 지정된 PRU 펌웨어(.out 파일)를 파일 시스템(`/lib/firmware`)에서 찾아 PRU의 명령어 메모리에 로드하고, PRU의 실행을 시작하거나 정지시키는 역할을 한다.46 ARM과 PRU 간의 데이터 통신은 

  `rpmsg`(Remote Processor Messaging) 프레임워크를 통해 이루어진다. `rpmsg`는 양방향 통신을 위해 공유 메모리 영역에 `vring`이라는 링 버퍼 구조를 생성하고, 새로운 메시지가 도착했음을 알리기 위해 하드웨어 인터럽트(mailbox)를 사용한다.49

- **적용 시나리오:**

  - **정밀 PWM/DShot 생성:** PX4의 믹서(mixer) 모듈이 계산한 최종 모터 출력 값(예: 1000~2000µs 범위의 펄스 폭)을 `rpmsg`를 통해 PRU로 전달한다. PRU 펌웨어는 이 값을 받아, 내부 타이머와 GPIO 직접 제어 기능을 이용해 마이크로초(μs) 단위의 오차도 없는 정확한 PWM 또는 DShot 신호를 생성하여 ESC로 출력한다.
  - **RC 수신기 프로토콜 디코딩:** SBUS, PPM, CRSF와 같은 시리얼 기반 RC 프로토콜은 정밀한 타이밍에 의존하여 각 채널의 데이터를 인코딩한다. PRU 펌웨어가 UART 핀의 상태 변화를 나노초 단위로 감시하여 프로토콜을 디코딩하고, 최종적으로 추출된 16개 또는 그 이상의 채널 값만을 `rpmsg`를 통해 ARM 코어로 전달한다. 이를 통해 ARM 코어는 복잡한 비트-뱅잉(bit-banging) 작업에서 완전히 해방된다.

- **개발:** PRU 펌웨어는 일반적으로 C 언어와 약간의 어셈블리어를 혼용하여 작성하며, TI에서 제공하는 전용 컴파일러(`clpru`)를 사용하여 빌드한다.52 ARM과의 통신을 위한 

  `rpmsg` 채널이나 공유 메모리 영역과 같은 리소스는 펌웨어 코드 내에 `resource_table`이라는 특수한 데이터 구조를 통해 정의되어야 하며, `pru_rproc` 드라이버는 펌웨어를 로드할 때 이 테이블을 읽어 필요한 시스템 리소스를 설정해준다.48 TI의 PRU Software Support Package와 다양한 GitHub 예제들79이 개발에 큰 도움이 된다.


가장 높은 수준의 안정성과 결정성이 요구되는 시스템을 위해서는 Cortex-R5F 실시간 코어를 활용하는 방안을 고려할 수 있다.

- **원리:** Cortex-R5F는 실시간 응답성과 기능 안전을 위해 설계된 고신뢰성 코어다. 부동소수점 연산 유닛(FPU)과 함께, 캐시 미스로 인한 예측 불가능한 지연 없이 코드와 데이터에 접근할 수 있는 Tightly-Coupled Memory(TCM)를 내장하고 있어, 매우 예측 가능한 실행 시간을 보장한다.10
- **아키텍처:** 이 방안의 핵심은 R5F 코어 중 하나에 Zephyr나 FreeRTOS와 같은 경량의 실시간 운영체제(RTOS)를 포팅하고, PX4의 핵심 비행 제어 모듈(센서 드라이버, EKF, 제어기 등)을 이 RTOS 상에서 직접 실행하는 것이다. Cortex-A72에서 실행되는 Linux 시스템과는 OpenAMP 프레임워크나 `rpmsg`를 통해 통신한다. 이 구조는 사실상 BeagleBone AI-64 보드 내부에 별도의 고성능 FC를 소프트웨어적으로 구현하는 것과 같다.
- **장점:** 비행 제어의 핵심 로직을 Linux 운영체제의 스케줄링, 드라이버 문제, 메모리 관리 등 모든 잠재적 불안정성 요인으로부터 완벽하게 분리할 수 있다. 이는 최고의 안정성과 결정성을 확보할 수 있는 가장 확실한 방법이다.
- **단점:** 세 가지 방안 중 개발 복잡도가 압도적으로 높다. R5F용 RTOS 포팅, 해당 RTOS 환경에 맞게 PX4의 저수준 모듈들을 수정하고 이식하는 작업, 그리고 A72의 Linux와 R5F의 RTOS 간에 안정적인 통신 인터페이스를 개발하는 것 모두 상당한 전문성과 엔지니어링 노력을 요구한다. TI의 Processor SDK RTOS가 관련 드라이버와 예제를 일부 제공하지만 53, 이를 PX4라는 거대한 프레임워크와 통합하는 것은 전혀 다른 차원의 과제이다.


세 가지 방안을 각각 독립적으로 사용하는 것보다, 각 코어의 장점을 극대화하고 단점을 상호 보완하는 하이브리드 아키텍처를 구성하는 것이 가장 이상적인 해결책이다. 하나의 방식에만 의존하는 것은 비효율적이거나, 불안정하거나, 혹은 지나치게 복잡한 시스템을 낳을 수 있다.

- **제안 아키텍처 및 역할 분담:**
  - **Cortex-A72 (Linux + PREEMPT_RT):** 시스템의 '전략가' 또는 '두뇌' 역할을 수행한다. 고수준 임무 계획, SLAM, AI 가속기를 활용한 객체 탐지, GCS와의 통신, 비행 로그 기록 등 비결정성이 일부 허용되지만 높은 컴퓨팅 성능이 요구되는 작업을 전담한다. PX4의 상위 레벨 모듈(예: `navigator`, `commander`, `mavlink`)과 모든 ROS 노드가 이 코어에서 실행된다.
  - **PRU-ICSSG:** 시스템의 '반사 신경' 또는 '손과 발' 역할을 맡는다. 경성 실시간 I/O를 완벽하게 처리한다. 8개 이상의 모터/서보를 위한 정밀 PWM/DShot 신호를 생성하고, RC 수신기 입력을 지연 없이 디코딩하며, 기타 고속 디지털 센서 인터페이스를 담당한다.
  - **Cortex-R5F (선택적 고급 구성):** 시스템의 '안전 관리자' 또는 '소뇌' 역할을 한다. 최고 수준의 신뢰성이 요구되는 프로젝트에서 선택적으로 도입한다. IMU 데이터 수집, EKF를 통한 상태 추정, 그리고 자세/위치 제어 루프와 같은 비행 안정화의 가장 핵심적인 로직을 전담한다. 이 경우, A72는 목표 궤적(trajectory)이나 고수준 명령을 R5F에 전달하고, R5F는 이를 안정적으로 추종하는 제어 명령을 계산하여 PRU로 직접 전달하는 계층적 제어 구조를 형성할 수 있다.

아래 표는 각 실시간 제어 구현 방안의 특징을 비교 분석한 것이다.

**Table 2: 실시간 제어 구현 방안 비교 분석**

| 구현 방안 (Method)  | 핵심 기술 (Core Technology)                | 장점 (Pros)                                                  | 단점 (Cons)                                                  | 최적 적용 분야 (Best Use-Case)                               |
| ------------------- | ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **PREEMPT_RT 커널** | Linux 커널 패치, 스케줄러 개선             | 상대적으로 구현 용이, 기존 Linux 앱/라이브러리와 높은 호환성 | 완벽한 결정성 보장 불가, 부실한 드라이버에 취약, 초정밀 타이밍 불가 | 고수준 항법, 임무 계획, ROS 노드 등 연성 실시간(soft real-time) 작업 |
| **PRU 활용**        | 독립적인 200MHz RISC 코어, 단일 사이클 I/O | 나노초 단위 정밀도, 완벽한 결정성, 경성 실시간 I/O 보장      | 별도 펌웨어 개발 필요, 제한된 메모리 및 연산 능력, 복잡한 알고리즘 부적합 | PWM/DShot 신호 생성, RC 프로토콜 디코딩, QEP 등 정밀 I/O     |
| **Cortex-R5F 활용** | 고신뢰성 실시간 MCU 코어, TCM              | Linux와 완벽히 분리된 결정론적 환경, 높은 신뢰성 및 기능 안전 | 개발 복잡도 매우 높음, RTOS 포팅 및 이종 코어 간 통신 인터페이스 구현 필요 | 핵심 비행 제어 루프(자세/위치 제어), 센서 융합(EKF)          |

이 분석을 통해, BeagleBone AI-64의 잠재력을 최대한 활용하기 위해서는 PREEMPT_RT 커널을 기본으로 채택하고, 정밀 I/O는 PRU에 위임하는 하이브리드 방식이 가장 현실적이고 효율적인 출발점임을 알 수 있다. Cortex-R5F의 활용은 프로젝트의 요구사항과 가용 리소스에 따라 추후에 고려할 수 있는 고급 확장 옵션으로 남겨둘 수 있다.


이론적인 아키텍처 설계를 마쳤다면, 다음 단계는 이를 실제로 구현하기 위한 시스템 통합과 효율적인 개발 환경을 구축하는 것이다. 이 과정은 PX4 드라이버 개발, 크로스 컴파일 환경 설정, 그리고 타겟 보드에서의 자동 실행 설정으로 구성된다.


PX4 프레임워크에 새로운 하드웨어를 통합하기 위해서는 해당 하드웨어와 통신하고, 그 결과를 표준화된 uORB 토픽으로 발행하는 드라이버를 개발해야 한다.55 BeagleBone AI-64 기반 FC를 위해서는 최소한 IMU 센서 드라이버와 PWM 출력 드라이버가 필수적이다.

- **IMU 센서 드라이버:** 드론에 널리 사용되는 InvenSense MPU-9250이나 Bosch BMI088과 같은 IMU 센서는 일반적으로 I2C 또는 SPI 버스를 통해 마스터 프로세서와 통신한다. 드라이버 개발의 첫 단계는 BeagleBone AI-64의 Cape 헤더 핀맵을 참조하여 56, 사용할 I2C 또는 SPI 버스를 Device Tree Overlay를 통해 활성화하는 것이다. 활성화가 완료되면, Linux 파일 시스템에 

  `/dev/i2c-X` 또는 `/dev/spidevX.Y` 와 같은 디바이스 노드가 생성된다. PX4의 IMU 드라이버는 이 디바이스 노드를 `open()`, `read()`, `write()`, `ioctl()`과 같은 표준 POSIX 파일 I/O 함수를 사용하여 접근하는 사용자 공간(userspace) 드라이버 형태로 작성된다. 드라이버는 주기적으로(예: 1kHz) 센서의 레지스터를 읽어 가속도와 자이로스코프의 원시(raw) 데이터를 획득한 후, 이를 PX4가 사용하는 표준 uORB 토픽인 `sensor_gyro`와 `sensor_accel`로 발행(publish)해야 한다.

- **PWM 출력 드라이버:** IV장에서 제안한 하이브리드 아키텍처에 따라, PWM 출력은 PRU가 전담한다. 따라서 Cortex-A72에서 실행되는 PX4의 PWM 출력 드라이버(`prumsg_pwm_out`과 같은 이름으로 신규 개발)는 실제 PWM 신호를 생성하는 대신, PRU 펌웨어와의 통신을 중계하는 역할을 맡는다. 이 드라이버는 PX4의 제어기 모듈들이 발행하는 `actuator_outputs` uORB 토픽을 구독한다. 새로운 `actuator_outputs` 메시지를 수신할 때마다, 드라이버는 그 안에 담긴 각 모터의 출력 값(예: -1.0 ~ 1.0 범위)을 추출하여 `rpmsg` 통신 채널을 통해 PRU로 전송한다. PRU 펌웨어는 이 값을 수신하여 실제 ESC가 요구하는 PWM 또는 DShot 신호로 변환하고 GPIO 핀으로 출력한다. 이 구조는 실시간성과 비실시간성 작업의 역할을 명확히 분리한다.


개발 작업은 일반적으로 고성능의 x86_64 아키텍처 PC에서 이루어지지만, 최종 실행 파일은 BeagleBone AI-64의 Arm AArch64 아키텍처에서 동작해야 한다. 따라서 크로스 컴파일(cross-compilation) 환경 구축은 필수적이다.57

개발용 PC가 Ubuntu Linux 환경이라면, `apt` 패키지 관리자를 통해 AArch64 크로스 컴파일 툴체인을 간단히 설치할 수 있다.

```Bash
sudo apt update
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

59

툴체인 설치 후, PX4의 빌드 시스템인 CMake에게 타겟 보드를 빌드할 때 이 크로스 컴파일러를 사용하도록 알려주어야 한다. 이는 앞서 생성한 `cmake/configs/posix_beaglebone_ai64_default.cmake` 파일 내에 툴체인 경로와 타겟 시스템 정보를 명시함으로써 이루어진다.37 올바르게 설정되었다면, 개발자는 PC에서 `make beaglebone_ai64_default` 명령을 실행하는 것만으로 BeagleBone AI-64에서 실행 가능한 PX4 바이너리를 생성할 수 있다.


개발된 커스텀 Linux 커널(PREEMPT_RT 패치 적용)과 PX4 애플리케이션을 타겟 보드에 배포하고, 시스템 부팅 시 자동으로 실행되도록 설정해야 한다.

- **eMMC 플래싱:** 개발 초기 단계에서는 SD 카드로 부팅하여 테스트하는 것이 편리하지만, 최종 배포 시에는 안정성과 속도를 위해 내장 eMMC에 이미지를 설치해야 한다. 이는 일반적으로 부팅 가능한 'flasher' 이미지를 SD 카드에 구운 뒤, 특정 부트 버튼을 누른 상태로 보드에 전원을 인가하여 eMMC로 이미지를 복사하는 방식으로 진행된다.61

- **부트 로더 설정:** BeagleBone AI-64는 U-Boot 부트 로더를 사용한다. U-Boot는 기본적으로 microSD 카드를 먼저 스캔하고, 부팅 가능한 `extlinux.conf` 파일이 있으면 해당 미디어로 부팅을 시도한다. 따라서 eMMC로 항상 부팅하도록 강제하려면, SD 카드를 제거하거나 U-Boot의 부팅 순서 환경 변수를 수정해야 할 수 있다.64

- **`systemd` 서비스 등록:** 현대적인 Linux 배포판은 `systemd`를 init 시스템으로 사용한다. 시스템 부팅 후 PX4 애플리케이션이 자동으로 실행되도록 하려면 `systemd` 서비스를 생성하고 등록해야 한다.

  1. `/etc/systemd/system/` 디렉토리에 `px4.service`와 같은 이름의 유닛 파일을 생성한다.

  2. 이 파일 내에 PX4 실행 파일의 경로, 시작에 필요한 부팅 스크립트(`rcS`)의 위치, 그리고 실행 사용자 등을 명시한다.

     Ini, TOML

     ```
     [Unit]
     Description=PX4 Autopilot Service
     After=network.target
     
     
     ExecStart=/usr/bin/px4 -s /etc/px4/rc.d/rcS
     Restart=always
     User=root
     
     [Install]
     WantedBy=multi-user.target
     ```

  3. `sudo systemctl enable px4.service` 명령을 실행하여 이 서비스가 부팅 시 자동으로 활성화되도록 설정한다. `sudo systemctl start px4.service` 명령으로 즉시 서비스를 시작할 수도 있다.11

이러한 개발 환경과 배포 프로세스를 구축하는 과정에서, 개발 워크플로우의 효율성은 프로젝트의 성패에 큰 영향을 미친다. BeagleBone AI-64와 같이 커널, PRU 펌웨어, PX4 애플리케이션 등 여러 소프트웨어 구성요소를 가진 복잡한 시스템에서는 이들 중 하나만 수정해도 크로스 컴파일, 바이너리 패키징, 타겟 보드로의 전송(`scp` 또는 `rsync` 사용), 서비스 재시작 또는 재부팅, 그리고 테스트라는 긴 과정을 반복해야 한다. 특히 커널이나 부트로더 수정 시에는 eMMC 플래싱이 필요할 수 있어 이 '컴파일-배포-테스트' 사이클의 시간 소요가 매우 커질 수 있다.63 따라서, 단순히 툴체인을 설치하는 것을 넘어, 이 반복적인 사이클을 최대한 자동화하고 시간을 단축하는 스크립트(예: `make upload` 명령 최적화, `rsync`를 이용한 빠른 증분 동기화, `ssh`를 통한 원격 서비스 재시작 자동화)를 구축하는 것이 프로젝트 생산성을 결정짓는 핵심적인 요소가 된다. 이는 PX4가 BeagleBone Blue와 같은 기존 보드에서 `make upload`를 어떻게 구현했는지 분석하고 33, 이를 BeagleBone AI-64 환경에 맞게 재구현하는 작업을 포함할 수 있다.


BeagleBone AI-64의 진정한 가치는 안정적인 비행 제어를 넘어, 내장된 강력한 AI 가속기와 고성능 AP를 활용하여 지능형 자율 비행 기능을 단일 보드에서 구현하는 데 있다. 이는 TI의 Edge AI SDK와 ROS2 생태계를 통합함으로써 실현될 수 있다.


BeagleBone AI-64는 TI의 Processor SDK와 완벽하게 호환되며, 여기에는 AI 추론을 위한 핵심 라이브러리인 TIDL(TI Deep Learning)이 포함되어 있다.21

- **TIDL의 역할:** TIDL은 TensorFlow-Lite, ONNX, Caffe 등 표준 딥러닝 프레임워크를 사용하여 학습된 신경망 모델을 입력받아, 이를 TDA4VM SoC에 내장된 C7x DSP와 MMA(Matrix Multiply Accelerator) 하드웨어에서 최적으로 실행될 수 있는 형태로 컴파일하고 가속하는 역할을 한다.70 이 과정을 통해 AP인 Cortex-A72의 부하를 최소화하면서 최대 8 TOPS에 달하는 높은 AI 추론 성능을 달성할 수 있다.
- **PX4 시스템과의 통합:** PX4 시스템 내에서 AI 기능을 통합하기 위한 일반적인 접근법은, 카메라(USB 또는 MIPI-CSI)로부터 영상 프레임을 지속적으로 획득하고, 이 프레임을 TIDL API를 통해 AI 가속기에 전달하여 추론(예: 객체 탐지, 의미론적 분할)을 수행한 뒤, 그 결과를 uORB 토픽(예: `obstacle_distance`, `object_detected`)으로 발행하는 별도의 PX4 모듈을 개발하는 것이다. 이 모듈은 시스템의 다른 부분(예: 충돌 방지 모듈)이 AI 추론 결과를 활용하여 비행 경로를 수정할 수 있도록 한다.


로봇 애플리케이션 개발의 사실상 표준인 ROS(Robot Operating System), 특히 최신 버전인 ROS2와의 통합은 BeagleBone AI-64의 활용성을 극대화하는 핵심 열쇠이다.

- **uXRCE-DDS 브릿지:** PX4는 ROS2와의 원활한 통신을 위해 `uXRCE-DDS` 클라이언트를 미들웨어에 내장하고 있다. 이 클라이언트는 PX4 내부의 uORB 메시지를 ROS2의 기본 통신 시스템인 DDS(Data Distribution Service) 메시지로 실시간 변환(또는 그 반대)하는 역할을 한다. 이를 통해 PX4와 ROS2 노드들은 별도의 변환 과정 없이 마치 동일한 통신 시스템을 사용하는 것처럼 직접적으로 데이터를 주고받을 수 있다.1
- **통합 아키텍처:** BeagleBone AI-64의 Debian Linux 환경에 ROS2를 설치하고, `px4_ros_com` 패키지를 활용하여 PX4와 ROS2 간의 통신 브릿지를 설정한다.73 이렇게 구성된 시스템에서 PX4는 실시간 비행 제어와 안정화를 담당하고, ROS2는 SLAM, 고급 경로 계획, 컴퓨터 비전, 조작(manipulation) 등 ROS 생태계의 방대한 오픈소스 라이브러리와 알고리즘을 활용하는 고수준 작업을 처리한다. Cortex-A72의 강력한 성능은 여러 ROS2 노드를 동시에 실행하기에 충분하다.


AI 가속기와 ROS2를 통합하여 시각 기반 장애물 회피 시스템을 구현하는 과정은 다음과 같은 데이터 파이프라인으로 개념화할 수 있다.

1. **카메라 데이터 수집:** ROS2의 카메라 드라이버 노드(예: `v4l2_camera_node`)가 MIPI-CSI 또는 USB 포트에 연결된 카메라로부터 영상 프레임을 지속적으로 획득하여 ROS2의 표준 이미지 토픽인 `/image_raw`로 발행한다.
2. **AI 기반 객체 탐지:** 이 시스템의 핵심인 'AI 추론 노드'는 `/image_raw` 토픽을 구독한다. 이 노드는 수신된 이미지 데이터를 TIDL 라이브러리를 통해 TDA4VM의 C7x/MMA 가속기로 전달하여, 미리 학습된 객체 탐지 모델(예: YOLO, SSD)을 실행한다. 추론 결과(탐지된 객체의 종류, 위치, 경계 상자)는 새로운 ROS2 토픽인 `/detected_objects`로 발행된다.
3. **장애물 정보 변환:** 또 다른 '인지(Perception) 노드'는 `/detected_objects` 토픽과, 필요하다면 스테레오 카메라나 LiDAR로부터 얻은 깊이 정보를 함께 구독한다. 이 노드는 2D 이미지 상의 객체 정보를 3D 공간상의 장애물 위치와 크기로 변환하고, 이를 PX4가 이해할 수 있는 MAVLink의 `OBSTACLE_DISTANCE` 메시지 형식이나 그에 상응하는 uORB 토픽 형식으로 가공한다.
4. **PX4로 데이터 전송:** 가공된 장애물 정보는 `px4_ros_com`의 uXRCE-DDS 브릿지를 통해 PX4 내부의 `obstacle_distance` uORB 토픽으로 실시간 전달된다.
5. **회피 기동:** PX4에 내장된 충돌 방지(Collision Prevention) 모듈은 `obstacle_distance` 토픽을 구독하고 있다가, 새로운 장애물 정보가 들어오면 이를 기반으로 현재 비행 경로를 안전하게 수정하여 충돌을 회피하는 제어 명령을 생성한다.

이처럼 온보드 AI 기능을 통합하는 것은 단순히 알고리즘을 실행하는 것을 넘어, 시스템 전체의 '데이터 흐름'을 최적화하는 복잡한 엔지니어링 문제이다. 성능의 병목은 AI 모델의 추론 속도 자체뿐만 아니라, 카메라에서부터 최종 PX4 제어 모듈에 이르기까지 여러 프로세스(ROS 노드)와 통신 계층(ROS2 DDS, uXRCE-DDS)을 거치면서 발생하는 데이터 복사 및 전송 오버헤드에서 발생할 가능성이 크다. TI의 Robotics SDK는 이러한 문제를 해결하기 위해 GStreamer와 OpenVX를 활용한 최적화된 플러그인과 모듈을 제공하는데 53, 이는 메모리 복사를 최소화(zero-copy)하고 하드웨어 가속기를 최대한 활용하여 데이터 파이프라인의 효율을 극대화하기 위함이다. 따라서 성공적인 AI 통합은 단순히 ROS 노드를 작성하는 것을 넘어, GStreamer, V4L2, OpenVX, DMA와 같은 저수준 API를 이해하고 활용하여 시스템 전체의 데이터 흐름을 분석하고 최적화하는 깊이 있는 시스템 레벨 엔지니어링 역량을 요구한다.



본 고찰을 통해 분석한 결과, BeagleBone AI-64는 PX4 오토파일럿을 기반으로 하는 차세대 고성능, 지능형 드론 비행 제어 컴퓨터(FC)를 개발하기 위한 매우 유망하고 기술적으로 타당한 플랫폼이라 결론 내릴 수 있다. 강력한 64비트 애플리케이션 프로세서(Cortex-A72), 신뢰성 높은 실시간 코어(Cortex-R5F), 초저지연 I/O 프로세서(PRU), 그리고 하드웨어 AI 가속기(C7x/MMA)를 단일 SoC에 집적한 이종 아키텍처는 기존의 'FC + 컴패니언 컴퓨터' 분리 구조가 가진 물리적, 성능적 한계를 극복하고, 두 역할을 하나의 보드로 통합하여 그 이상의 성능을 발휘할 수 있는 독보적인 잠재력을 지니고 있다


이러한 잠재력을 현실화하기 위한 프로젝트의 성공 여부는 TI TDA4VM SoC의 이종 아키텍처를 얼마나 깊이 이해하고, 각 프로세싱 코어의 고유한 특성에 맞는 역할을 부여하는 최적의 하이브리드 소프트웨어 아키텍처를 구현하는가에 달려 있다. 본 보고서에서 제안하는 가장 이상적인 역할 분담은 다음과 같다.

1. **Cortex-A72 (PREEMPT_RT Linux):** 고수준 항법, 임무 계획, ROS 노드, AI 추론 관리 등 복잡한 연산을 담당한다.
2. **PRU-ICSSG:** 모터 제어를 위한 정밀 PWM/DShot 신호 생성과 RC 수신기 입력 처리 등 경성 실시간 I/O를 전담한다.
3. **Cortex-R5F (선택적 고급 구성):** EKF, 자세/위치 제어 루프 등 비행 안정화의 핵심 로직을 담당하여 최고의 신뢰성을 확보한다.

이러한 아키텍처 구현에 있어 가장 큰 기술적 난제는 단연 개발의 복잡성이다. PREEMPT_RT 커널의 빌드 및 튜닝, PRU 펌웨어 개발, 이종 코어 간의 통신(remoteproc/rpmsg), 그리고 이 모든 것을 PX4 프레임워크와 통합하고 안정적으로 디버깅하는 과정은 상당한 수준의 임베디드 시스템 전문성을 요구한다.


이러한 복잡성을 관리하고 프로젝트의 성공 확률을 높이기 위해, 다음과 같은 단계적 개발 접근법을 제언한다.

- **1단계 (기본 포팅 및 안정화):**
  - **목표:** BeagleBone AI-64의 Linux 환경에서 PX4 애플리케이션을 성공적으로 빌드하고 실행하여, GCS(QGroundControl)와 기본 통신(MAVLink over TCP/IP)이 가능하도록 한다.77
  - **세부 과업:** AArch64 크로스 컴파일 환경을 구축하고, PX4의 신규 보드 지원 파일을 생성한다. PREEMPT_RT 패치를 적용한 커널을 빌드하여 설치한다. 이 단계에서는 보드의 I/O를 직접 사용하는 대신, 검증된 외부 FC(예: Pixhawk)를 USB 또는 UART로 연결하여 BeagleBone AI-64를 컴패니언 컴퓨터처럼 활용하며 고수준 제어 알고리즘을 테스트할 수 있다.1
- **2단계 (실시간 I/O 구현 및 독립 비행):**
  - **목표:** BeagleBone AI-64가 외부 FC 없이 독립적인 비행 제어 컴퓨터로 기능하도록 한다.
  - **세부 과업:** PRU 펌웨어를 개발하여 정밀한 PWM/DShot 신호를 생성하고, 이를 제어하는 PX4의 PWM 출력 드라이버를 `rpmsg`를 통해 구현한다. 보드의 I2C/SPI 버스를 통해 IMU, 기압계 등의 핵심 센서를 직접 연결하고, 해당 센서들을 위한 PX4 드라이버를 개발한다. 센서 데이터가 EKF로 정상적으로 전달되고, 제어 루프를 통해 모터가 안정적으로 제어되는 것을 확인하여 첫 독립 비행을 달성한다.
- **3단계 (지능화 및 기능 확장):**
  - **목표:** 안정적인 비행이 검증된 플랫폼 위에 AI와 ROS2를 통합하여 지능형 자율 비행 기능을 구현한다.
  - **세부 과업:** TI의 Edge AI SDK를 설치하고 TIDL을 활용하여, 학습된 AI 모델을 TDA4VM의 하드웨어 가속기에서 실행한다. ROS2와 uXRCE-DDS 브릿지를 설정하여 PX4와 ROS2 노드 간의 통신 파이프라인을 구축한다. 이를 바탕으로 시각 기반 장애물 회피, 정밀 착륙, 객체 추적 등 BeagleBone AI-64의 진정한 잠재력을 발휘하는 고급 애플리케이션을 개발한다.

이러한 체계적이고 단계적인 접근법을 통해 개발팀은 기술적 리스크를 효과적으로 관리하고, 복잡한 이종 시스템을 성공적으로 구축하여 차세대 지능형 드론 개발의 새로운 지평을 열 수 있을 것이다.


1. Raspberry Pi Companion with Pixhawk | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/companion_computer/pixhawk_rpi
2. How To Connect PixHawk to Raspberry Pi and NVIDIA Jetson - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=nIuoCYauW3s
3. Raspberry Pi based Autonomous drone - element14 Community, accessed August 6, 2025, https://community.element14.com/products/raspberry-pi/b/blog/posts/raspberry-pi-based-autonomous-drone
4. BeagleBoard BeagleBone® AI-64 - Mouser Electronics, accessed August 6, 2025, https://www.mouser.com/new/beagleboardorg/beagleboard-beaglebone-ai-64/
5. Top 5 Companion Computers for UAVs | ModalAI, Inc., accessed August 6, 2025, https://www.modalai.com/blogs/blog/top-5-companion-computers-for-uavs
6. Is it possible to working real time with Linux? : r/embedded - Reddit, accessed August 6, 2025, https://www.reddit.com/r/embedded/comments/10p23zm/is_it_possible_to_working_real_time_with_linux/
7. Real-Time I/O - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/books/beaglebone-cookbook/08realtime/realtime.html
8. Real time processing on the Beaglebone AI - Mender, accessed August 6, 2025, https://mender.io/blog/real-time-processing-on-the-beaglebone-ai
9. Performance Assessment of Linux Kernels with PREEMPT_RT on ARM-Based Embedded Devices - MDPI, accessed August 6, 2025, https://www.mdpi.com/2079-9292/10/11/1331
10. Design and Specifications - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagle.cc/boards/beaglebone/ai-64/03-design-and-specifications.html
11. BeagleBone AI-64 - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/beaglebone-ai-64.pdf
12. TDA4VM data sheet, product information and support | TI.com - Texas Instruments, accessed August 6, 2025, https://www.ti.com/product/TDA4VM
13. BeagleBone® AI-64 - BeagleBoard, accessed August 6, 2025, https://www.beagleboard.org/boards/beaglebone-ai-64
14. BEAGL-BONE-AI-64 by BeagleBoard.org Foundation | TI.com, accessed August 6, 2025, https://www.ti.com/tool/BEAGL-BONE-AI-64
15. BEAGLEBOARD 102110646 Single Board Computer, BeagleBone, AI-64, TDA4VM, accessed August 6, 2025, https://vilros.com/products/beagleboard-102110646-single-board-computer-beaglebone-ai-64-tda4vm-arm-cortex-a72
16. BeagleBone AI-64 comes with TDA4VM SoC from Texas Instruments - Linux Gizmos, accessed August 6, 2025, https://linuxgizmos.com/beaglebone-ai-64-comes-with-tda4vm-soc-from-texas-instruments/
17. Introduction - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/boards/beaglebone/ai-64/01-introduction.html
18. BeagleBone AI-64 - Zephyr Project Documentation, accessed August 6, 2025, https://docs.zephyrproject.org/latest/boards/beagle/beaglebone_ai64/doc/index.html
19. PRU-ICSS / PRU_ICSSG Getting Starting Guide on Linux - Texas Instruments, accessed August 6, 2025, https://www.ti.com/lit/pdf/sprace9
20. First steps with the BeagleBone PRU - boxysean.com, accessed August 6, 2025, https://boxysean.com/blog/2012/08/12/first-steps-with-the-beaglebone-pru/
21. Getting Started with the TDA4VM Edge AI Starter Kit - Hackster.io, accessed August 6, 2025, https://www.hackster.io/whitney-knitter/getting-started-with-the-tda4vm-edge-ai-starter-kit-c05531
22. BeagleBone cape interface spec - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/latest/boards/capes/cape-interface-spec.html
23. Enable UART 1 on BB AI-64 - General Discussion - BeagleBoard Forum, accessed August 6, 2025, https://forum.beagleboard.org/t/enable-uart-1-on-bb-ai-64/32997
24. How can I enable the 5 UARTs of the beaglebone AI-64? - BeagleBoard Forum, accessed August 6, 2025, https://forum.beagleboard.org/t/how-can-i-enable-the-5-uarts-of-the-beaglebone-ai-64/35602
25. PX4 Autopilot Software - GitHub, accessed August 6, 2025, https://github.com/PX4/PX4-Autopilot
26. PX4 Architectural Overview | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/concept/architecture
27. PX4: A Node-Based Multithreaded Open Source Robotics Framework for Deeply Embedded Platforms - Ethz, accessed August 6, 2025, https://people.inf.ethz.ch/pomarc/pubs/MeierICRA15.pdf
28. Architectural Overview / PX4 Development Guide - shnuzxd, accessed August 6, 2025, https://shnuzxd.gitbooks.io/px4-development-guide/en/concept/architecture.html
29. PX4 Research Log [4] – A first look at PX4 architecture, example code, uORB and NSH script - UAV Lab @ Sydney, accessed August 6, 2025, https://uav-lab.org/2016/08/02/px4-research-log4-a-first-look-at-px4-architecture/
30. Flight Controller Porting Guide | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/hardware/porting_guide
31. Drone Apps & APIs | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/robotics/
32. Raspberry Pi 2/3/4 Navio2 Autopilot | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/flight_controller/raspberry_pi_navio2.html
33. BeagleBone Blue | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/flight_controller/beaglebone_blue
34. Flight Controller Porting Guide / Devguide, accessed August 6, 2025, https://bresch.gitbooks.io/devguide/content/en/debug/porting-guide.html
35. PX4-user_guide/ja/flight_controller/raspberry_pi_pilotpi.md at main - GitHub, accessed August 6, 2025, https://github.com/PX4/PX4-user_guide/blob/master/ja/flight_controller/raspberry_pi_pilotpi.md
36. NuttX Board Porting Guide | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/hardware/porting_guide_nuttx.html
37. Building the Code / PX4 Development Guide - shnuzxd, accessed August 6, 2025, https://shnuzxd.gitbooks.io/px4-development-guide/en/setup/building_px4.html
38. Intel energizes decades-old real-time Linux kernel project | Hacker News, accessed August 6, 2025, https://news.ycombinator.com/item?id=30476723
39. Building a Real-time Linux Kernel in Ubuntu with PREEMPT_RT - acontis, accessed August 6, 2025, https://www.acontis.com/en/building-a-real-time-linux-kernel-in-ubuntu-preemptrt.html
40. Tag: preempt_rt - robskelly.com, accessed August 6, 2025, https://robskelly.com/tag/preempt_rt/
41. Real-Time Performance in Linux: Harnessing PREEMPT_RT for Embedded Systems - RunTime Recruitment, accessed August 6, 2025, https://runtimerec.com/wp-content/uploads/2024/10/real-time-performance-in-linux-harnessing-preempt-rt-for-embedded-systems_67219ae1.pdf
42. realtime:documentation:howto:tools:cyclictest:start [Wiki], accessed August 6, 2025, https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/cyclictest/start
43. A Comparison of Scheduling Latency in Linux, PREEMPT RT, and LITMUSRT - People at MPI-SWS, accessed August 6, 2025, https://people.mpi-sws.org/~bbb/papers/pdf/ospert13.pdf
44. Cyclictest latency results comparison for Raspberry Pi with Linux... - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/figure/Cyclictest-latency-results-comparison-for-Raspberry-Pi-with-Linux-kernels-with-PREEMPT-RT_tbl2_352041533
45. Preempt-RT Latency Benchmarking of the Cortex-A53 processor - Enclustra, accessed August 6, 2025, https://www.enclustra.com/assets/files/download/Preempt-RT-Latency-Benchmarking-of-the-Cortex-A53-Processor-Paul-Thomas-AMSC-Embedde_Linux_Conference_Europe_2018.pdf
46. Remote Processor Framework - The Linux Kernel documentation, accessed August 6, 2025, https://docs.kernel.org/staging/remoteproc.html
47. PRU RemoteProc documentation - Google Groups, accessed August 6, 2025, https://groups.google.com/g/beagleboard/c/_LJ7rI1eISM
48. 3.5.3.1. RemoteProc and RPMsg - Processor SDK Linux for AM335X Documentation - Texas Instruments, accessed August 6, 2025, https://software-dl.ti.com/processor-sdk-linux/esd/docs/latest/linux/Foundational_Components/PRU-ICSS/Linux_Drivers/RemoteProc_and_RPMsg.html
49. PRU-framework/drivers/remoteproc/pruss_remoteproc.c at master - GitHub, accessed August 6, 2025, https://github.com/shubhi1407/PRU-framework/blob/master/drivers/remoteproc/pruss_remoteproc.c
50. Are there good resources for understanding remoteproc and rpmsg are doing? How are they related to openAMP? : r/linux - Reddit, accessed August 6, 2025, https://www.reddit.com/r/linux/comments/yebqjh/are_there_good_resources_for_understanding/
51. 3.6.3.1. RemoteProc - Processor SDK Linux for AM57X Documentation - Texas Instruments, accessed August 6, 2025, https://software-dl.ti.com/processor-sdk-linux/esd/AM57X/08_02_00_04/exports/docs/linux/Foundational_Components/PRU-ICSS/Linux_Drivers/RemoteProc.html
52. How to run C programs on the BeagleBone's PRU microcontrollers - Ken Shirriff's blog, accessed August 6, 2025, http://www.righto.com/2016/09/how-to-run-c-programs-on-beaglebones.html
53. SDK Components - BeagleBone AI-64 - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/boards/beaglebone/ai-64/edge_ai_apps/sdk_components.html
54. PROCESSOR-SDK-J721E Software development kit (SDK) | TI.com, accessed August 6, 2025, https://www.ti.com/tool/PROCESSOR-SDK-J721E
55. Driver Development | PX4 Guide (main), accessed August 6, 2025, https://docs.px4.io/main/en/middleware/drivers
56. Expansion - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/boards/beaglebone/ai-64/04-expansion.html
57. How to cross compile for ARM? - Ask Ubuntu, accessed August 6, 2025, https://askubuntu.com/questions/250696/how-to-cross-compile-for-arm
58. How to setup environment to develop for aarch64 target using x86_64 machine? - Reddit, accessed August 6, 2025, https://www.reddit.com/r/Compilers/comments/1kjzef6/how_to_setup_environment_to_develop_for_aarch64/
59. Ubuntu Development Environment | PX4 User Guide (v1.14), accessed August 6, 2025, https://docs.px4.io/v1.14/en/dev_setup/dev_env_linux_ubuntu.html
60. Unable to build px4 current version by cross-compilation on macOS Mojave 10.14.6 / Issue #13172 - GitHub, accessed August 6, 2025, https://github.com/PX4/Firmware/issues/13172
61. Quick Start Guide - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagleboard.org/latest/boards/beaglebone/ai-64/02-quick-start.html
62. Additional Support Information - BeagleBoard Documentation, accessed August 6, 2025, https://docs.beagle.cc/boards/beaglebone/ai-64/06-support.html
63. Beaglebone AI-64 can not boot both from eMMC and mircoSD card - BeagleBoard Forum, accessed August 6, 2025, https://forum.beagleboard.org/t/beaglebone-ai-64-can-not-boot-both-from-emmc-and-mircosd-card/34491
64. BBAI-64 Boot order - General Discussion - BeagleBoard Forum, accessed August 6, 2025, https://forum.beagleboard.org/t/bbai-64-boot-order/33129
65. BeagleBone AI-64: Distro Boot Overview - General Discussion, accessed August 6, 2025, https://forum.beagleboard.org/t/beaglebone-ai-64-distro-boot-overview/32227
66. BeagleBone AI-64 Setup Guide | Prepare Devices - Viam Documentation, accessed August 6, 2025, https://docs.viam.com/operate/reference/prepare/beaglebone-setup/
67. HDMI issue with beaglebone AI 64 - General Discussion - BeagleBoard Forum, accessed August 6, 2025, https://forum.beagleboard.org/t/hdmi-issue-with-beaglebone-ai-64/41297
68. PX4 BeagleBone® Blue-based quadcopter - Aviumtechnologies Blog, accessed August 6, 2025, https://blog.aviumtechnologies.com/topics/px4-autopilot/px4-beaglebone-r-blue-based-quadcopter
69. Texas Instruments SK-TDA4VM - Edge Impulse Documentation, accessed August 6, 2025, https://docs.edgeimpulse.com/docs/edge-ai-hardware/cpu-+-ai-accelerators/sk-tda4vm
70. Deep learning with Jacinto TDA4x processors | Video | TI.com - Texas Instruments, accessed August 6, 2025, https://www.ti.com/video/6195166916001
71. TDA4VM: TIDL: How to use API - Processors forum - TI E2E, accessed August 6, 2025, https://e2e.ti.com/support/processors-group/processors/f/processors-forum/1004254/tda4vm-tidl-how-to-use-api
72. PX4 ROS 2 Navigation Interface | PX4 Guide (main) - PX4 docs, accessed August 6, 2025, https://docs.px4.io/main/en/ros2/px4_ros2_navigation_interface.html
73. PX4/px4_ros_com: ROS2/ROS interface with PX4 through a Fast-RTPS bridge - GitHub, accessed August 6, 2025, https://github.com/PX4/px4_ros_com
74. Object detection using ROS2 in SDK-TDA4VM - Processors forum - TI E2E, accessed August 6, 2025, https://e2e.ti.com/support/processors-group/processors/f/processors-forum/1314516/tda4vm-object-detection-using-ros2-in-sdk-tda4vm
75. TexasInstruments/edgeai-robotics-sdk: Robotics SDK provides a ROS-based robotics software development environment for Texas Instruments Edge AI Processors, including AM62A, TDA4VM, AM67A, AM68A, and AM69A. - GitHub, accessed August 6, 2025, https://github.com/TexasInstruments/edgeai-robotics-sdk
76. Process This: Build AI-powered robots with TI's NEW Robotics SDK - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=o0Tm1u3CwdY
77. Using QGroundControl over Wi-Fi - Clover, accessed August 6, 2025, https://clover.coex.tech/en/gcs_bridge.html
78. Problems with TCP connection via 4G dongle - QGroundControl - PX4 Discussion Forum, accessed August 6, 2025, https://discuss.px4.io/t/problems-with-tcp-connection-via-4g-dongle/25282
79. PRU Cookbook - BeagleBoard, accessed August 6, 2025, https://beagleboard.org/static/prucookbook/
80. jmacd/bbb-pru-examples: BeagleBone Black PRU examples - GitHub, accessed August 6, 2025, https://github.com/jmacd/bbb-pru-examples
81. jadonk/pruduino: AM335x PRU remoteproc firmware with command interface for BeagleBone - GitHub, accessed August 6, 2025, https://github.com/jadonk/pruduino
82. pru c++ shared mem - General Discussion - BeagleBoard Forum, accessed August 6, 2025, https://forum.beagleboard.org/t/pru-c-shared-mem/35095
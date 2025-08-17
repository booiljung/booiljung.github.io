# NVIDIA Jetson 플랫폼으로의 PX4 비행 스택 이식

## 서론

"NVIDIA Jetson에 PX4를 이식할 수 있는가?"라는 질문은 단순한 기술적 호기심을 넘어, 무인 항공 시스템(UAS) 아키텍처의 근본적인 패러다임에 대한 탐구를 담고 있습니다. 결론부터 말하자면, Jetson에서 PX4 비행 스택을 네이티브로 실행하는 것은 이론적으로는 가능하지만, 이는 극도로 높은 기술적 난이도를 수반하며 소수의 고도로 전문화된 연구 프로젝트에나 적합한 도전 과제입니다. 따라서 대부분의 상업용 및 연구용 애플리케이션에서는 이 방식을 권장하지 않습니다.

현재 업계 표준이자 가장 안정적이며 효율적인 접근 방식은 Jetson을 고성능 '컴패니언 컴퓨터(Companion Computer)'로 활용하고, 비행 제어는 Pixhawk와 같은 별도의 전용 비행 컨트롤러(FCU)에 맡기는 것입니다. 이 보고서는 이 두 가지 대조적인 아키텍처를 심층적으로 분석하여, 각 방식의 기술적 기반, 장단점, 그리고 현실적인 구현 가능성을 평가하는 것을 목표로 합니다.

본 보고서는 다음과 같은 구조로 전개됩니다. 첫째, 현재 가장 널리 채택되고 있는 '컴패니언 컴퓨터 모델'의 아키텍처, 하드웨어 통합, 그리고 소프트웨어 스택을 상세히 분석하여 현존하는 최상의 솔루션을 조명합니다. 둘째, 사용자의 질문에 직접적으로 답하기 위해, PX4를 Jetson에 '네이티브로 이식하는 모델'이 마주하게 될 근본적인 기술적 장벽, 특히 실시간 운영체제(RTOS)의 필요성과 드라이버 개발의 복잡성을 심도 있게 탐구합니다. 마지막으로, 두 아키텍처를 종합적으로 비교 분석하고, 이를 바탕으로 개발 목적과 상황에 맞는 명확하고 전략적인 권고안을 제시할 것입니다. 이 분석을 통해 개발자들은 Jetson의 강력한 컴퓨팅 성능을 PX4 기반 드론 시스템에 통합하기 위한 최적의 경로를 이해하고, 정보에 입각한 아키텍처 설계를 할 수 있게 될 것입니다.

## 1.  확립된 패러다임: 고성능 컴패니언 컴퓨터로서의 Jetson

오늘날 고성능 드론 시스템을 구축할 때 가장 널리 채택되고 검증된 아키텍처는 NVIDIA Jetson을 컴패니언 컴퓨터로 활용하는 것입니다. 이 모델은 시스템의 안정성과 개발 효율성을 극대화하는 최적의 설계로 평가받고 있습니다.

### 1.1 아키텍처 철학: 관심사의 분리

이 아키텍처의 핵심 철학은 '관심사의 분리(Separation of Concerns)'입니다. 이는 시스템의 각기 다른 요구사항을 전문화된 하드웨어에 위임하는 설계 원칙입니다.

- **비행 제어 (실시간 영역):** 비행 안정성 유지, 센서 데이터의 고속 처리, 모터 제어와 같이 밀리초 단위의 정밀한 시간 제어가 필수적인 '하드 실시간(Hard Real-time)' 작업은 NuttX와 같은 실시간 운영체제(RTOS)가 탑재된 전용 마이크로컨트롤러(MCU) 기반의 비행 컨트롤러(FCU), 예를 들어 Pixhawk가 전담합니다.1 FCU는 시스템의 '뇌간' 또는 '소뇌'처럼 자율적인 생명 유지 기능을 수행합니다.
- **고수준 연산 (비실시간 영역):** 반면, 컴퓨터 비전, 인공지능 기반 경로 계획, 복잡한 페이로드 제어, 군집 비행 알고리즘 등 방대한 계산량을 요구하지만 엄격한 실시간성이 필요 없는 작업은 Jetson과 같은 강력한 임베디드 리눅스 시스템이 담당합니다.3 Jetson은 시스템의 '대뇌' 역할을 하며, 지능적인 의사결정을 내립니다.

이러한 역할 분담은 시스템의 강건성(Robustness)을 극대화합니다. 예를 들어, Jetson에서 실행되는 고부하의 객체 탐지 알고리즘이 일시적으로 멈추거나 오류를 일으키더라도, FCU는 독립적으로 비행 자세를 안정적으로 유지할 수 있습니다. 이는 비행 안전과 직결되는 매우 중요한 설계적 이점입니다.

### 1.2 하드웨어 통합 및 인터페이스

FCU와 Jetson을 물리적으로 연결하고 통합하는 방식은 이 아키텍처의 성공적인 구현에 있어 핵심적인 요소입니다.

#### 1.2.1 표준 통신 링크

두 프로세서 간의 통신은 주로 다음과 같은 표준화된 인터페이스를 통해 이루어집니다.

- **직렬 통신 (UART):** 가장 보편적이고 기본적인 연결 방식입니다. Pixhawk의 `TELEM` 포트와 Jetson의 GPIO 헤더에 있는 UART 핀(TX, RX)을 연결하여 MAVLink 메시지를 교환합니다.5 설정이 간단하고 대부분의 보드에서 지원된다는 장점이 있습니다.
- **이더넷 (Ethernet):** 더 높은 대역폭과 안정적인 연결이 필요할 때 권장되는 방식입니다.7 특히 다중 센서 데이터 스트리밍이나 고해상도 영상 전송 시 UART보다 월등한 성능을 제공합니다.
- **CAN 버스 (CAN Bus):** 차동 신호 방식을 사용하여 전기적 노이즈에 강하고, 여러 장치를 하나의 버스에 연결할 수 있어 시스템 배선을 간소화할 수 있는 강력한 대안입니다.8

#### 1.2.2 최상의 구현 사례: Holybro Pixhawk Jetson 베이스보드

이러한 통합 과정을 극적으로 간소화한 대표적인 상용 제품이 바로 'Holybro Pixhawk Jetson 베이스보드'입니다.8 이 제품은 Jetson Orin Nano/NX 모듈과 Pixhawk Autopilot Bus (PAB) 규격의 FCU 모듈(예: Pixhawk 6C)을 단일 캐리어 보드에 장착할 수 있도록 설계되었습니다.8

이 보드의 설계는 컴패니언 컴퓨터 아키텍처의 모범 사례를 명확하게 보여줍니다.

- **통합 인터페이스:** 보드 내에 이더넷 스위치(RTL8367S)가 내장되어 있어 Jetson과 Pixhawk가 별도의 케이블링 없이 이더넷으로 통신할 수 있습니다. 또한 Jetson의 UART1 포트가 Pixhawk의 `TELEM2` 포트에 직접 연결되어 있어 직렬 통신도 즉시 사용 가능합니다.8
- **독립 전원부:** Jetson과 Pixhawk는 각각 독립된 전원 회로를 통해 전력을 공급받습니다. 이는 한쪽의 전원 문제가 다른 쪽에 영향을 미치는 것을 방지하여 시스템 안정성을 높입니다.8
- **풍부한 I/O:** USB 3.0, MIPI CSI 카메라 입력, NVMe SSD 슬롯 등 Jetson의 고성능 I/O를 외부로 쉽게 확장할 수 있도록 지원하여, 개발자가 복잡한 주변 장치를 손쉽게 연결할 수 있게 합니다.8

이러한 상용 통합 보드의 존재 자체가 컴패니언 컴퓨터 모델이 단순한 이론적 권장 사항을 넘어, 실제 제품 개발 현장에서 검증되고 채택된 업계 표준임을 강력하게 시사합니다. 기업들은 안정적이고 수요가 명확한 아키텍처에만 이러한 정교한 제품 개발 투자를 진행하기 때문입니다.

### 1.3 소프트웨어 및 통신 스택

하드웨어 통합만큼 중요한 것이 두뇌(Jetson)와 뇌간(FCU)이 원활하게 소통하게 하는 소프트웨어 스택입니다.

#### 1.3.1 프로토콜

- **MAVLink:** 드론 구성 요소 간의 통신을 위한 사실상의 표준 프로토콜입니다. 원격 측정 데이터(Telemetry), 명령(Commands), 미션 정보 등을 교환하는 데 사용되는 경량의 헤더 전용 라이브러리입니다.3
- **uXRCE-DDS (micro-ROS):** ROS 2 생태계와의 직접적인 통합을 위한 현대적인 대안입니다. PX4의 내부 메시징 시스템인 µORB와 ROS 2의 DDS 네트워크를 직접 연결하여, MAVLink 기반 브리지보다 높은 성능과 네이티브에 가까운 통합을 제공합니다.3

#### 1.3.2 API 및 프레임워크

- **MAVSDK:** Python, C++, Swift 등 다양한 프로그래밍 언어에서 MAVLink 시스템과 상호작용할 수 있도록 지원하는 현대적이고 사용하기 쉬운 SDK입니다.3 간단한 스크립트로 드론을 제어하는 것부터 복잡한 애플리케이션 개발까지 폭넓게 사용됩니다.
- **ROS/ROS 2와 MAVROS:** 학계 및 고급 로보틱스 연구에서 가장 널리 사용되는 프레임워크입니다. MAVROS는 MAVLink와 ROS 간의 포괄적인 브리지 역할을 하며, FCU의 모든 기능을 ROS 토픽, 서비스, 파라미터로 노출시켜 ROS의 방대한 라이브러리와 도구를 활용할 수 있게 합니다.3

이처럼 MAVSDK, MAVROS와 같은 전체 소프트웨어 생태계는 컴패니언 컴퓨터와 비행 컨트롤러 간의 클라이언트-서버 관계를 전제로 설계되었습니다. 즉, 직렬 또는 네트워크 링크를 통해 원격 장치와 통신하는 것을 기본 가정으로 합니다. 만약 PX4를 Jetson에 네이티브로 이식한다면, 이 아키텍처는 붕괴됩니다. Jetson의 애플리케이션은 더 이상 원격 장치와 통신하는 것이 아니라, 동일한 머신에서 실행되는 다른 프로세스(PX4 비행 스택)와 통신해야 하므로, 기존의 MAVSDK/MAVROS 대신 새로운 형태의 프로세스 간 통신(IPC) 메커니즘이 필요하게 됩니다. 이는 네이티브 이식이 단순히 하드웨어 변경을 넘어, 고수준 소프트웨어 스택의 근본적인 재설계를 요구함을 의미합니다.

## 2.  미개척 영역: Jetson에서의 PX4 네이티브 실행

사용자의 질문에 대한 직접적인 탐구는 PX4를 Jetson에 네이티브로 이식하는, 즉 Jetson을 단일 보드 비행 컴퓨터로 사용하는 개념에서 시작됩니다. 이 접근 방식은 이론적으로는 가능성의 문을 열어두고 있지만, 실제 구현에는 엄청난 실질적 장벽이 존재합니다.

### 2.1 PX4 이식성의 기반

PX4가 다양한 하드웨어 플랫폼으로 이식될 수 있는 잠재력을 가진 것은 그 아키텍처의 핵심적인 설계 철학 덕분입니다.

- **POSIX 호환 API:** PX4는 스레딩, 파일 입출력 등에서 표준 POSIX(Portable Operating System Interface) API를 따르도록 설계되었습니다. 이는 운영체제에 대한 의존성을 최소화하고, NuttX, Linux, macOS 등 다양한 POSIX 호환 시스템에서 코드를 재사용할 수 있게 하는 의도적인 선택입니다.1
- **하드웨어 추상화 계층 (HAL):** ArduPilot과 마찬가지로 PX4는 핵심 자동 비행 로직과 특정 하드웨어 드라이버를 분리하기 위해 하드웨어 추상화 계층(Hardware Abstraction Layer)을 사용합니다.13 덕분에 개발자는 HAL만 새로 구현하면 동일한 비행 스택 코드를 새로운 보드에서 실행할 수 있습니다. 네이티브 이식의 노력은 대부분 이 보드 지원 계층에 집중됩니다.
- **µORB 미들웨어:** PX4 내부의 발행-구독(Publish-Subscribe) 메시징 시스템인 µORB는 센서, 추정기, 컨트롤러 등 각 모듈을 분리(decoupling)하여 시스템의 모듈성과 이식성을 향상시킵니다.1

### 2.2 공식 PX4 리눅스 포팅 가이드 분석

이러한 네이티브 이식의 가능성을 공식적으로 열어주는 문서가 바로 PX4의 '비행 컨트롤러 포팅 가이드'입니다.15 이 가이드는 새로운 보드 포트를 위한 파일 구조(`boards/VENDOR/MODEL/`)와 빌드 설정 파일(`.px4board`)의 구성을 설명합니다.

그러나 이 가이드에서 가장 주목해야 할 부분은 리눅스 기반 보드에 대한 언급입니다. 가이드는 "리눅스 보드는 OS 및 커널 설정을 포함하지 않습니다. 이는 보드에 사용 가능한 리눅스 이미지에 의해 이미 제공되며, **이 리눅스 이미지는 관성 센서를 즉시(out of the box) 지원해야 합니다**"라고 명시하고 있습니다.15

이 문장은 사용자의 질문에 대한 답변의 핵심을 담고 있습니다. PX4 개발팀은 PX4 소프트웨어 자체는 이식 가능하지만, 비행 제어에 적합한 실시간성과 센서 지원을 갖춘 리눅스 환경을 구축하는 엄청난 과업은 전적으로 포팅을 시도하는 개발자의 책임이라고 선언하는 것과 같습니다. 이는 네이티브 포트가 이론적으로는 가능하지만, 그 과정에서 가장 어렵고 핵심적인 부분(실시간 커널 및 드라이버)은 개발자가 처음부터 해결해야 함을 의미하며, 그 난이도를 암묵적으로 경고하고 있습니다.

### 2.3 가장 큰 난관: 실시간 결정성

Jetson에 기본으로 제공되는 L4T(Linux for Tegra)의 표준 리눅스 커널이 비행 제어에 근본적으로 부적합한 이유는 '실시간 결정성(Real-time Determinism)'의 부재 때문입니다.

- **스케줄러 지연 시간 (Scheduler Latency):** 일반적인 OS(GPOS)는 평균적인 처리량(throughput)을 최적화하도록 설계되었기 때문에, 특정 작업이 언제 실행될지 정확히 보장하지 않습니다.
- **비결정적 인터럽트 처리:** 하드웨어 인터럽트 처리가 다른 커널 작업에 의해 지연될 수 있어, 센서 데이터를 읽는 타이밍이 불규칙해질 수 있습니다.

비행 컨트롤러는 수백 헤르츠(Hz)의 일정한 주기로 센서 값을 읽고 모터 출력을 업데이트해야 하며, 이때 지터(jitter, 시간 변동)가 최소화되어야 합니다. 단 한 번의 마감 시간(deadline)을 놓치는 것이 기체 불안정으로 이어져 치명적인 추락을 유발할 수 있습니다. 이러한 이유로, 표준 리눅스 커널을 하드 실시간 성능을 보장하는 RTOS로 변환하는 작업이 네이티브 이식의 가장 중요하고도 어려운 첫 번째 관문이 됩니다.

## 3.  기술 심층 분석: Jetson에서 PREEMPT_RT로 실시간 성능 확보하기

Jetson에서 PX4를 네이티브로 실행하기 위한 협상 불가능한 전제 조건은 표준 리눅스 커널을 하드 실시간 운영체제로 변환하는 것입니다. 이를 위한 유일하고 현실적인 해결책은 `PREEMPT_RT` (Real-Time Preemption) 패치를 적용하는 것입니다.

### 3.1 실시간 리눅스(PREEMPT_RT) 소개

`PREEMPT_RT` 패치는 리눅스 커널을 거의 완전한 선점형(fully preemptible)으로 만드는 코드 수정 모음입니다. 이 패치는 커널 내의 스핀락(spinlock) 대부분을 뮤텍스(mutex)로 변환하고, 인터럽트 핸들러를 우선순위를 가진 커널 스레드로 이동시키는 등의 변경을 통해 커널 코드 실행 중에도 높은 우선순위의 실시간 작업이 즉시 끼어들 수 있도록 보장합니다. 이를 통해 시스템의 최대 지연 시간(latency)을 극적으로 줄여, 비행 제어와 같은 하드 실시간 애플리케이션에 적합한 환경을 만듭니다.16

### 3.2 실시간 Jetson 커널 구축 경로

Jetson에 실시간 커널을 구축하는 방법은 크게 두 가지로, 각각 장단점과 복잡성을 가집니다.

#### 3.2.1 "고전적인" 수동 구축 방법

이는 개발자가 커널 빌드의 모든 과정을 직접 제어하는 전통적인 방식입니다. 여러 튜토리얼을 종합한 일반적인 작업 흐름은 다음과 같습니다.18

1. **소스 확보:** NVIDIA 개발자 사이트에서 대상 Jetpack 버전에 맞는 L4T 드라이버 패키지, 샘플 루트 파일 시스템, 커널 소스, 그리고 ARM64용 크로스 컴파일 툴체인을 모두 다운로드합니다.19 버전 호환성을 맞추는 것이 매우 중요합니다.
2. **패치 적용:** 다운로드한 커널 소스 디렉토리 내에서 제공되는 스크립트(`./scripts/rt-patch.sh apply-patches`)를 실행하여 `PREEMPT_RT` 패치를 적용합니다.19
3. **커널 설정 (`menuconfig`):** 텍스트 기반의 설정 인터페이스를 통해 커널의 동작을 세밀하게 조정하는 핵심 단계입니다. 실시간 성능을 위해 반드시 설정해야 하는 주요 옵션은 다음과 같습니다.18
   - `General setup -> Preemption Model` -> `Fully Preemptible Kernel (Real-Time)`
   - `General setup -> Timer subsystem -> Timer tick handling` -> `Full dynticks system (tickless)`
   - `Kernel Features -> Timer frequency` -> `1000 HZ` (낮은 지연 시간을 위해 일반적으로 선택)
4. **컴파일 및 설치:** 설정이 완료되면, 커널 이미지(Image), 디바이스 트리 블롭(DTBs), 그리고 커널 모듈들을 크로스 컴파일합니다. 생성된 결과물들을 `Linux_for_Tegra` 디렉토리 내의 올바른 위치에 정확히 복사한 후, 전체 시스템 이미지를 생성하여 SD 카드에 플래싱하거나 Jetson에 직접 플래싱합니다.22

#### 3.2.2 "현대적인" APT를 이용한 방법

NVIDIA는 최신 Jetpack 버전에서 이 과정을 단순화하는 방법을 제공합니다.24

1. Jetson 보드에서 직접 `apt` 소스 리스트 파일(`sources.list.d`)에 NVIDIA의 RT 커널 저장소를 추가합니다.
2. `sudo apt update` 명령으로 패키지 목록을 갱신합니다.
3. `sudo apt install nvidia-l4t-rt-kernel nvidia-l4t-rt-kernel-headers...` 와 같은 명령어로 사전 빌드된 RT 커널과 관련 패키지를 설치합니다.
4. 부트로더 설정 파일(`/boot/extlinux/extlinux.conf`)을 수정하여 부팅 시 기본 커널로 RT 커널이 선택되도록 설정합니다.

이 방법은 수동 빌드의 복잡성을 크게 줄여주지만, 개발자가 커널 설정을 세밀하게 제어할 수 없다는 단점이 있습니다.

이 두 가지 방법의 존재와, 특정 버전에 따른 문제 해결을 논의하는 수많은 포럼 게시물들은 숨겨진 비용, 즉 '유지보수 부담'을 드러냅니다.19 네이티브 포트 경로를 선택하는 개발자는 자신의 프로젝트를 특정 버전의 맞춤형 커널에 종속시키게 됩니다. NVIDIA가 새로운 Jetpack을 출시할 때마다 RT 패치가 깨지거나, 전체 빌드 및 검증 과정을 처음부터 다시 반복해야 할 위험이 있습니다. 이는 일회성 노력이 아니라, 지속적인 엔지니어링 투자를 요구하는 중대한 약속이며, 시스템을 매우 취약하게 만들 수 있습니다.

## 4.  하드웨어 인터페이스: 네이티브 포트를 위한 센서 및 액추에이터 드라이버 개발

실시간 커널 구축이라는 첫 번째 거대한 산을 넘었다고 해도, 두 번째로 그에 못지않은 난관이 기다리고 있습니다. 바로 PX4가 Jetson에 연결된 물리적 센서 및 액추에이터와 통신하도록 만드는 드라이버 개발입니다.

### 4.1 드라이버 아키텍처의 간극 메우기

네이티브 포트의 핵심 과제는 근본적으로 다른 두 가지 드라이버 모델 사이의 간극을 메우는 것입니다.

- **MCU 기반 드라이버 (NuttX):** Pixhawk와 같은 MCU 기반 보드에서 PX4 드라이버는 I2C나 SPI 버스를 통해 센서의 레지스터에 직접 접근합니다. 즉, 메모리 맵 입출력(Memory-Mapped I/O)을 통해 하드웨어를 매우 낮은 수준에서 직접 제어합니다.
- **리눅스 기반 드라이버 (Jetson):** 리눅스에서는 모든 하드웨어 접근이 커널에 의해 중재됩니다. PX4와 같은 사용자 공간(user-space) 애플리케이션은 하드웨어에 직접 접근할 수 없으며, 대신 커널이 제공하는 추상화된 인터페이스, 즉 `/dev/i2c-1`이나 `/dev/spidev1.0`과 같은 디바이스 파일을 통해야 합니다. 애플리케이션은 `open()`, `read()`, `write()`, `ioctl()`과 같은 시스템 콜을 사용하여 커널에 요청을 보내고, 커널 드라이버가 실제 하드웨어 통신을 수행합니다.

PX4의 아키텍처 다이어그램을 보면, 가장 하단에 하드웨어와 직접 통신하는 드라이버 계층이 존재합니다.1 네이티브 리눅스 포트는 이 계층 전체를 리눅스의 드라이버 모델에 맞게 완전히 새로 작성해야 함을 의미합니다.

### 4.2 리눅스용 PX4 드라이버 통합을 위한 개념적 로드맵

현재까지 이 과정에 대한 완전한 공개 예제가 없으므로, 이는 이론적인 로드맵입니다.

1. **대상 센서 식별:** Jetson의 I2C/SPI 버스에 연결될 IMU, 기압계, 자력계 등의 물리적 주소와 버스 번호를 파악합니다.
2. **커널 드라이버 확인:** Jetson의 리눅스 커널이 해당 센서들을 인식하고, 대응하는 `/dev/` 노드를 생성하는 데 필요한 드라이버를 포함하고 있는지 확인해야 합니다. 이것이 바로 포팅 가이드에서 언급한 "관성 센서를 즉시 지원해야 한다"는 요구사항의 실체입니다.15
3. **PX4 사용자 공간 드라이버 개발:** 각 센서에 대해 PX4 소스 트리(`src/drivers`) 내에 새로운 C++ 클래스를 작성해야 합니다. 이 새로운 드라이버는 다음과 같이 동작합니다.
   - PX4 드라이버의 표준 인터페이스(`init()`, `start()` 등)를 구현합니다.
   - 주기적인 실행 루프 내에서, 리눅스의 해당 디바이스 파일(예: `/dev/i2c-1`)을 `open()`하고, `ioctl()`을 사용하여 통신할 슬레이브 주소를 설정한 뒤, `read()`/`write()`로 데이터를 주고받습니다.
   - 커널로부터 읽어온 원시 데이터를 파싱하여 물리량으로 변환합니다.
   - 최종적으로, 가공된 센서 데이터를 `sensor_accel`, `sensor_gyro`와 같은 적절한 µORB 토픽에 발행(publish)하여 PX4의 다른 모듈(예: EKF2 추정기)이 사용할 수 있도록 합니다.

이 고된 과정은 비행에 필요한 모든 단일 센서에 대해 반복되어야 합니다.

이 과정에서 간과하기 쉬운 또 다른 중대한 도전 과제는 바로 액추에이터(모터, 서보) 제어입니다. 비행 제어는 여러 개의 PWM(Pulse Width Modulation) 신호를 동시에, 높은 주파수(ESC의 경우 400Hz 이상)로, 그리고 매우 낮은 지터(jitter)로 생성해야 합니다. Pixhawk에 사용되는 STM32와 같은 MCU는 이러한 작업을 위해 특별히 설계된 하드웨어 타이머 주변장치를 내장하고 있습니다. 반면, Jetson의 범용 입출력(GPIO) 핀은 정밀한 하드웨어 타이밍 PWM 신호를 여러 채널에서 동시에 생성하도록 설계되지 않았습니다. 소프트웨어적으로 이를 구현하려 하면 상당한 CPU 자원을 소모하고, 비행 제어에 치명적인 지터를 유발할 수 있습니다. 따라서 현실적인 네이티브 포트 솔루션은 USB나 SPI를 통해 연결되는 별도의 외부 PWM 생성기 보드를 필요로 할 가능성이 매우 높습니다. 아이러니하게도 이는 단일 보드 솔루션이라는 본래의 목적을 일부 훼손하며, 또 다른 드라이버 개발의 복잡성을 추가하는 결과를 낳습니다.

## 5.  비교 분석 및 전략적 권고

지금까지의 심층 분석을 종합하여, 두 가지 Jetson 통합 아키텍처의 장단점을 명확히 비교하고 개발자를 위한 실질적인 전략을 제시합니다.

### 5.1 Jetson 통합 아키텍처 비교 분석표



두 접근 방식의 복잡한 트레이드오프를 한눈에 파악할 수 있도록 다음 표를 제시합니다. 이 표는 최종 권고안을 뒷받침하는 핵심적인 근거 자료입니다.

| 평가 항목                              | 컴패니언 컴퓨터 모델                                         | 네이티브 PX4 포트 모델                                       | 우위                   |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------- |
| **비행 안정성 및 안전성**              | 물리적, 논리적으로 분리된 아키텍처로 최고 수준의 안정성 보장. 고수준 작업 실패가 비행 제어에 영향 없음. | 단일 시스템 실패가 전체 시스템 마비로 이어질 수 있음 (Single Point of Failure). | **컴패니언 모델**      |
| **실시간 성능**                        | 전용 RTOS(NuttX)가 하드 실시간 성능을 완벽하게 보장.         | `PREEMPT_RT` 패치를 적용한 GPOS. 상당한 개선이 있으나, 전용 RTOS 수준의 보장은 어려움. | **컴패니언 모델**      |
| **개발 노력 및 시간**                  | 수일 ~ 수주. 검증된 하드웨어와 방대한 소프트웨어 생태계 활용. | 수개월 ~ 수년. 커널, 드라이버, 시스템 통합 등 모든 것을 처음부터 개발. | **컴패니언 모델**      |
| **소프트웨어 생태계 및 커뮤니티 지원** | MAVSDK, ROS, QGroundControl 등 성숙한 생태계와 방대한 커뮤니티 지원. | 거의 전무. 모든 문제를 자체적으로 해결해야 함.               | **컴패니언 모델**      |
| **시스템 복잡성 및 유지보수**          | 표준 소프트웨어와 커널 사용으로 단순하고 예측 가능.          | 맞춤형 커널과 드라이버로 인해 매우 복잡하며, Jetpack 업데이트 시마다 파손 위험. | **컴패니언 모델**      |
| **하드웨어 비용 (BOM)**                | FCU 추가 비용 발생.                                          | 이론적으로는 비용 절감 가능하나, 외부 PWM 보드 등 추가 주변장치 필요 시 상쇄됨. | **상황에 따라 다름**   |
| **크기, 무게, 전력 (SWaP)**            | FCU로 인한 약간의 증가.                                      | 이론적으로는 유리하나, 추가 주변장치 필요 시 큰 차이 없음.   | **상황에 따라 다름**   |
| **연구 유연성**                        | 고수준 AI/비전 연구에 최적화.                                | 실시간 리눅스, 하드웨어/소프트웨어 통합 등 저수준 시스템 연구에 독특한 기회 제공. | **네이티브 포트 모델** |

### 5.2 위험 대비 보상 평가

- **컴패니언 컴퓨터 모델:** **낮은 위험, 높은 보상.** 이는 두 플랫폼의 장점만을 결합한 검증되고 강력한 아키텍처입니다. 개발자는 비행 제어 플랫폼을 재발명하는 대신, Jetson에서의 고수준 애플리케이션 개발에 집중할 수 있습니다.
- **네이티브 포트 모델:** **극도로 높은 위험, 불확실한 보상.** 외부 PWM 보드와 같은 필수 주변장치를 추가하면 잠재적인 SWaP(크기, 무게, 전력) 이점은 거의 사라집니다. 이 경로의 '보상'은 주로 학술적인 성취, 즉 포트 자체의 성공에 있습니다. 제품 중심의 개발 프로젝트에서는 이러한 위험을 정당화하기 어렵습니다.

### 5.3 최종 권고안

#### 5.3.1 상용 제품, 양산 및 신뢰성 중심 프로젝트

**컴패니언 컴퓨터 모델**을 명백하게 권고합니다. Pixhawk 6C와 같은 기성품 FCU와 Holybro 베이스보드 같은 통합 캐리어를 사용하십시오.3 엔지니어링 자원은 Jetson에서 실행될 애플리케이션 소프트웨어(컴퓨터 비전, AI, 자율 비행 알고리즘 등) 개발에 집중하는 것이 가장 효율적이고 안전한 전략입니다.

#### 5.3.2 고급 학술 연구 및 R&D

**네이티브 포트 모델**은 연구 목표가 '비행 제어를 위한 실시간 리눅스'나 '새로운 하드웨어/소프트웨어 통합 설계' 자체를 탐구하는 경우에만 고려할 수 있습니다. 이 경우, 해당 프로젝트는 커널 엔지니어링, 드라이버 개발, 실시간 시스템에 대한 깊은 전문 지식을 요구하는 수년간의 R&D 과제로 취급되어야 하며, 앞서 논의된 모든 기술적 난제와 지속적인 유지보수 부담을 완전히 인지한 상태에서 시작해야 합니다.

## 6. 결론

사용자의 질문 "Jetson에 PX4를 이식할 수 있는가?"에 대한 기술적인 답변은 "이론적으로는 가능하다"입니다. 그러나 실용적이고 전략적인 답변은 "대부분의 목적을 위해서는 시도해서는 안 된다"입니다. 이 보고서에서 심층적으로 분석한 바와 같이, PX4를 Jetson에 네이티브로 이식하는 작업은 실시간 커널 구축, 모든 센서와 액추에이터를 위한 드라이버의 완전한 재작성 등 해결해야 할 거대한 기술적 장벽들을 포함하고 있습니다.

반면, Jetson을 컴패니언 컴퓨터로, Pixhawk와 같은 전용 FCU를 실시간 제어기로 사용하는 아키텍처는 타협의 산물이 아니라, 현재 기술 수준에서 가장 최적화된 설계입니다. 이 모델은 각 플랫폼의 강점을 극대화하고, 월등한 신뢰성, 성숙한 소프트웨어 생태계, 그리고 압도적으로 낮은 개발 오버헤드를 제공합니다.

결론적으로, PX4의 네이티브 Jetson 포트는 심층적인 학술 탐구의 영역에 남아 있으며, 오늘날 강력하고 지능적인 무인 시스템을 구축하기 위한 실용적인 경로는 아닙니다. 개발자들은 검증되고 지원받는 컴패니언 컴퓨터 아키텍처를 채택함으로써, 위험을 최소화하고 혁신적인 애플리케이션 개발에 역량을 집중할 수 있을 것입니다.

#### **참고 자료**

1. PX4: A Node-Based Multithreaded Open Source Robotics Framework for Deeply Embedded Platforms - Ethz, accessed July 3, 2025, https://people.inf.ethz.ch/pomarc/pubs/MeierICRA15.pdf
2. PX4: A node-based multithreaded open source robotics framework for deeply embedded platforms - SciSpace, accessed July 3, 2025, https://scispace.com/pdf/px4-a-node-based-multithreaded-open-source-robotics-50m35kksn5.pdf
3. Companion Computers | PX4 Guide (main) - PX4 docs, accessed July 3, 2025, https://docs.px4.io/main/en/companion_computer/
4. Top 5 Companion Computers for UAVs | ModalAI, Inc., accessed July 3, 2025, https://www.modalai.com/blogs/blog/top-5-companion-computers-for-uavs
5. Px4 Jetson Nano Connection - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/px4-jetson-nano-connection/271119
6. How to Connect Jetson Nano to Pixhawk - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/how-to-connect-jetson-nano-to-pixhawk/80189
7. Using a Companion Computer with Pixhawk Controllers | PX4 Guide (main), accessed July 3, 2025, https://docs.px4.io/main/en/companion_computer/pixhawk_companion.html
8. Holybro Pixhawk Jetson Baseboard | PX4 Guide (main) - PX4 docs, accessed July 3, 2025, https://docs.px4.io/main/en/companion_computer/holybro_pixhawk_jetson_baseboard.html
9. PX4-user_guide/tr/companion_computer/holybro_pixhawk_jetson_baseboard.md at main - GitHub, accessed July 3, 2025, https://github.com/PX4/PX4-user_guide/blob/main/tr/companion_computer/holybro_pixhawk_jetson_baseboard.md
10. Get Started with PX4 and ROS 2 using Holybro's Jetson Baseboard, accessed July 3, 2025, https://px4.io/get-started-with-holybro-jetson/
11. Companion Computers - Dev documentation - ArduPilot, accessed July 3, 2025, https://ardupilot.org/dev/docs/companion-computers.html
12. Connection of pixhawk CubeOrange to Nvidia Jetson AGX Orin via PX4-ROS 2 Bridge using UART, accessed July 3, 2025, https://discuss.px4.io/t/connection-of-pixhawk-cubeorange-to-nvidia-jetson-agx-orin-via-px4-ros-2-bridge-using-uart/29206
13. Learning ArduPilot - Introduction - Dev documentation, accessed July 3, 2025, https://ardupilot.org/dev/docs/learning-ardupilot-introduction.html
14. Master Thesis PX4 autopilot customization for non-standard gimbal and UWB peripherals - Politecnico di Torino, accessed July 3, 2025, https://webthesis.biblio.polito.it/15919/1/tesi.pdf
15. Flight Controller Porting Guide | PX4 Guide (main) - PX4 docs, accessed July 3, 2025, https://docs.px4.io/main/en/hardware/porting_guide.html
16. How crucial is realtime kernel? : r/linuxaudio - Reddit, accessed July 3, 2025, https://www.reddit.com/r/linuxaudio/comments/oih3ew/how_crucial_is_realtime_kernel/
17. Building a real-time kernel for the NVIDIA Jetson TK1 - JetsonHacks, accessed July 3, 2025, https://jetsonhacks.com/2015/05/19/building-a-real-time-kernel-for-the-nvidia-jetson-tk1/
18. Including RT Patch, Getting the Backup Image and Transferring the Root File-System on DSBOX-N2 - Forecr.io, accessed July 3, 2025, https://www.forecr.io/blogs/bsp-development/including-rt-patch-getting-the-backup-image-and-transferring-the-root-file-system-on-dsbox-n2
19. Applying a PREEMPT-RT patch to JetPack 4.5 on Jetson Nano - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/applying-a-preempt-rt-patch-to-jetpack-4-5-on-jetson-nano/168428
20. RT Kernel on Jetson Nano - Simon Ghyselincks, accessed July 3, 2025, https://chipnbits.github.io/content/projects/RLUnicycle/rtkernel/rtpatch.html
21. hmxf/RTJetson: Preempt-RT Kernel Build Guide for NVIDIA Development Board - GitHub, accessed July 3, 2025, https://github.com/hmxf/RTJetson
22. NVIDIA Jetson Linux Developer Guide : Kernel Customization, accessed July 3, 2025, [https://docs.nvidia.com/jetson/l4t/Tegra%20Linux%20Driver%20Package%20Development%20Guide/kernel_custom.html](https://docs.nvidia.com/jetson/l4t/Tegra Linux Driver Package Development Guide/kernel_custom.html)
23. Build the Real-Time Kernel - Jetson AGX Orin - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/build-the-real-time-kernel/229571
24. Setting up Realtime Kernel on Jetson - Oren Bell, accessed July 3, 2025, https://orenbell.com/setting-up-realtime-kernel-on-jetson/
25. Installing Real-Time Kernel - NVIDIA Jetson Linux Developer Guide, accessed July 3, 2025, https://docs.nvidia.com/jetson/archives/r36.4.4/DeveloperGuide/SD/Kernel/RealTimeKernel.html
26. How to add rt kernel in the vesion of 35.2.1? - Jetson Xavier NX - NVIDIA Developer Forums, accessed July 3, 2025, https://forums.developer.nvidia.com/t/how-to-add-rt-kernel-in-the-vesion-of-35-2-1/252446
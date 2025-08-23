# PX4 오토파일럿 v1.15 아키텍처 심층 분석

**Executive Summary:** 본 보고서는 PX4 오토파일럿 버전 1.15의 소프트웨어 아키텍처에 대한 포괄적이고 심층적인 분석을 제공한다. PX4는 POSIX 호환 실시간 운영체제(RTOS) 위에 구축된 반응형(Reactive), 모듈형(Modular) 설계를 특징으로 한다. 시스템은 크게 두 개의 기능적 계층, 즉 하드웨어 추상화 및 통신을 담당하는 **미들웨어(Middleware)**와 상태 추정 및 제어 알고리즘을 포함하는 **플라이트 스택(Flight Stack)**으로 구성된다. v1.15 릴리스는 PX4의 진화 과정에서 중대한 전환점을 제시한다. 이 버전은 PX4를 주로 독립적으로 운영되던 오토파일럿에서 고도로 통합된 로보틱스 플랫폼으로 변모시키는 핵심적인 변화들을 포함한다. 특히, 동료(Peer) 수준의 ROS 2 인터페이스 도입과 핵심 상태 추정기인 EKF2의 상당한 성숙은 이러한 변화의 중심에 있다. 본 문서는 PX4의 근본적인 설계 철학부터 시작하여 각 계층의 구성 요소, 실행 모델, 그리고 v1.15의 주요 아키텍처 변화가 갖는 기술적, 전략적 의미를 상세히 탐구함으로써 개발자와 연구자에게 심도 있는 이해를 제공하는 것을 목표로 한다.


PX4 시스템 전체를 관통하는 핵심 원칙을 이해하는 것은 개별 구성 요소를 분석하기 위한 필수적인 선행 과정이다. 이 원칙들은 단순히 구성 요소가 *무엇*인지를 넘어, *왜* 그렇게 설계되었는지를 설명하며, 시스템의 유연성, 확장성, 그리고 견고성의 기반을 이룬다.


PX4 아키텍처의 가장 근본적인 특징은 "반응형(Reactive)" 시스템으로 정의된다는 점이다.1 이는 전통적인 순차적, 시간 트리거 방식의 비행 제어 루프와는 근본적으로 다른 접근 방식이다. 시스템은 비동기적이며 이벤트 기반으로 작동하며, 주로 새로운 데이터가 가용해질 때 이에 반응하여 동작한다.

이러한 반응형 설계는 시스템의 모든 기능이 교체 가능하고 재사용 가능한 구성 요소, 즉 "모듈(Modules)"로 분할되어 있기에 가능하다.1 각 모듈은 독립적인 프로그램으로, 셸(Shell)을 통해 런타임에도 개별적으로 시작, 중지, 교체될 수 있다 (`<module_name> start/stop`).1 이러한 극단적인 모듈성은 VTOL과 같은 새로운 형태의 기체를 위한 신속한 개발, 테스트, 그리고 맞춤화를 가능하게 하는 핵심적인 동력이다.5 이 모듈들 간의 통신은 비동기 메시지 패싱(Asynchronous Message Passing)을 통해 이루어지며, 이는 2.1절에서 상세히 다룰 uORB 메커니즘에 의해 구현된다.1 이 구조는 시스템이 가변적인 작업 부하에 효과적으로 대처할 수 있게 한다.1


PX4는 NuttX, Linux, macOS, QuRT와 같이 POSIX(Portable Operating System Interface) API를 제공하는 다양한 운영체제에서 실행된다.1 이는 개발자들이 일반적인 Unix 지식과 경험을 재사용할 수 있도록 하기 위한 전략적인 설계 결정이다.5 이 결정은 시스템 전반에 깊은 영향을 미친다.

첫째, 시스템 검사와 명령어 실행을 위한 Unix와 유사한 셸(예: NuttShell 'NSH')을 제공한다.1 이를 통해 개발자는 런타임에 시스템의 상태를 동적으로 확인하고(`top`, `uorb top` 등), 모듈을 직접 제어할 수 있다. 둘째, 멀티스레드 디자인 패턴을 자연스럽게 활용할 수 있게 한다.5 셋째, NuttX와 같은 POSIX 호환 실시간 운영체제(RTOS)는 FIFO 스케줄링과 같은 실시간 스케줄링 기능을 제공하여, 제어 작업에 필수적인 시간 결정성을 보장한다.1

이러한 POSIX 중심의 접근 방식은 전통적인 임베디드 펌웨어 개발의 높은 진입 장벽을 낮추는 효과를 가져온다. 많은 임베디드 시스템이 특정 RTOS와 하드웨어에 대한 전문 지식을 요구하는 반면, PX4는 `pthreads`, 파일 디스크립터, 명령줄 셸과 같은 친숙한 개념을 도입했다. 이는 특히 ROS(Robot Operating System)와 Linux 환경에 익숙한 광범위한 로보틱스 커뮤니티의 개발자들이 더 쉽게 PX4 생태계에 참여하고 기여할 수 있는 길을 열어주었다. uORB라는 발행-구독 시스템이 ROS 토픽과 개념적으로 유사하다는 점 5 역시 이러한 전략의 일환으로 볼 수 있다. 따라서 POSIX 호환성은 단순히 기술적 선택을 넘어, 개발자 생태계를 확장하고 혁신을 가속화하기 위한 장기적인 전략의 핵심이다. v1.15에서 도입된 ROS 2 인터페이스는 이러한 장기 전략의 자연스러운 귀결이자 완성이라고 할 수 있다.


PX4의 또 다른 핵심 철학은 멀티콥터, 고정익, VTOL, 로버, 보트, 잠수함 등 지원되는 모든 종류의 기체에 대해 단일 코드베이스(Single Codebase)를 공유한다는 것이다.1

이러한 접근은 기체별 로직(예: 컨트롤러, 믹서)을 특정 모듈로 추상화하고, 상태 추정, 내비게이션, 통신 프로토콜과 같은 상위 수준의 구성 요소는 모든 플랫폼에서 공유함으로써 달성된다.4 빌드 시스템은 `make <board_name>_<variant>` (예: `make px4_fmu-v5_default`)와 같은 명령어를 사용하여 이 공유 코드베이스를 특정 하드웨어 타겟에 맞게 컴파일한다.10

이 "단일 코드베이스" 철학은 강력한 선순환 구조를 만들어낸다. 예를 들어, 한 개발자가 고정익 기체를 위해 EKF2 추정기의 GPS 융합 로직을 개선했다고 가정해보자. EKF2 모듈은 모든 플랫폼에서 공유되기 때문에 4, 이 개선 사항은 별도의 포팅 작업 없이 즉시 멀티콥터, VTOL, 로버 등 다른 모든 기체에 적용된다. 중앙 저장소를 통해 관리되는 이러한 기능과 버그 수정의 교차 적용은 개발의 힘을 배가시키는 승수 효과(Force Multiplier)를 낳는다. 이는 분기되어 기체별로 관리되는 프로젝트들과 비교할 때, 전체 플랫폼이 훨씬 더 빠르고 견고하게 성숙할 수 있도록 하는 원동력이 되며, 다양한 애플리케이션 전반에 걸쳐 높은 수준의 품질과 신뢰성을 보장한다.7


PX4 아키텍처의 미들웨어 계층은 비행 제어 로직 자체와는 독립적으로, 하드웨어 통합과 내/외부 통신을 위한 기반 서비스를 제공하는 "범용 로보틱스 계층(general robotics layer)"이다.1 이 계층은 PX4를 단순한 오토파일럿을 넘어 다양한 자율 로봇을 위한 플랫폼으로 기능하게 하는 핵심이다.



uORB(micro Object Request Broker)는 모든 모듈 간 통신에 사용되는 내부 비동기 발행-구독(Publish-Subscribe) 메시징 시스템이다.1 이는 PX4의 중추 신경계와 같으며, 시스템의 반응형, 모듈형 설계를 가능하게 하는 기술적 근간이다.


uORB에서 발행자(Publisher)는 `vehicle_attitude`와 같이 의미적으로 이름이 부여된 메시지 채널인 '토픽(Topic)'을 광고(advertise)하고, 구독자(Subscriber)는 이 토픽을 구독하여 데이터를 수신한다.1 발행자는 구독자를 기다리지 않으며, 그 반대도 마찬가지다. 이 비동기적 특성은 모든 작업과 통신이 완전히 병렬화될 수 있도록 보장한다.1 시스템이 반응형이라는 것은 곧 모듈들이 새로운 데이터가 uORB를 통해 발행되는 즉시 업데이트된다는 것을 의미한다.1


NuttX와 같은 RTOS 타겟에서 uORB는 공유 메모리(Shared Memory)를 기반으로 하며, 전체 PX4 미들웨어는 단일 주소 공간(Single Address Space)에서 실행된다.1 이는 마이크로컨트롤러의 제한된 자원에서 매우 효율적인 통신을 가능하게 하지만, 모든 모듈 간에 메모리가 공유되므로 스레드 안전성(thread-safe)을 신중하게 고려한 접근이 필요하다.

구현 자체는 비동기성을 달성하기 위해 발행자와 구독자 사이에 별도의 버퍼를 사용하는 잠금 없는(lock-free) 방식으로 최적화되어 있다.13 메시지 형식은 `/msg` 디렉토리 내의 `.msg` 파일에 정의되며, 이 파일들은 빌드 시점에 C/C++ 코드로 자동 변환된다.13 개발자는 셸에서 `uorb top` 명령어를 실행하여 실시간으로 토픽 발행률과 구독자 정보를 확인함으로써 시스템의 데이터 흐름을 직관적으로 파악하고 디버깅할 수 있다.1

uORB를 단일 공유 메모리 주소 공간에 구현한 것은 실용적인 트레이드오프의 결과다. 이는 별도의 주소 공간이 제공하는 메모리 보호 기능보다는, 마이크로컨트롤러에서의 성능과 낮은 리소스 사용량을 우선시한 결정이다. 단일 주소 공간에서는 한 모듈의 버그(예: 버퍼 오버플로우)가 다른 모듈의 메모리를 손상시켜 전체 시스템을 불안정하게 만들 수 있는 잠재적 위험이 존재한다. 이는 안전이 중요한 소프트웨어에서 상당한 리스크다. 그러나 별도의 주소 공간을 구현하려면 많은 마이크로컨트롤러에 없는 메모리 관리 장치(MMU)가 필요하며, 프로세스 간 통신(IPC)의 컨텍스트 스위칭으로 인한 상당한 오버헤드가 발생한다. 따라서 이 설계는 자원이 제한된 하드웨어에서 성능을 극대화하기 위한 계산된 결정이다. 문서에서 "최소한의 노력으로" 각 모듈을 별도의 주소 공간에서 실행하는 것이 가능하도록 시스템이 설계되었다고 언급한 점 1은 아키텍트들이 이러한 트레이드오프를 인지하고 있으며, 더 강력한 하드웨어에서 메모리 보호 기능을 강화하는 방향으로의 미래 진화를 염두에 두고 설계했음을 시사한다.



PX4의 소프트웨어 아키텍처는 크게 4개의 계층으로 나뉜다. 그 중 하위 두 계층은 디바이스 드라이버로 구성된다. 가장 낮은 계층에는 특정 IMU나 버스 유형에 대한 하드웨어 종속적인 코드가 포함되고, 그 위 계층에는 해당 하드웨어를 `/dev/accel0`과 같은 POSIX 표준 디바이스 노드로 추상화하여 시스템에 노출하는 드라이버가 위치한다.5 이 구조는 하드웨어의 복잡성을 숨기고 일관된 인터페이스를 제공한다.


PX4는 I2C, SPI, UART, DroneCAN 등 다양한 버스를 통해 연결되는 방대한 종류의 센서와 액추에이터를 지원한다.7 이러한 주변 장치들을 위한 드라이버는 모듈 형태로 구현된다. 드라이버는 하드웨어에서 원시 데이터를 읽고, 필요한 보정(calibration)을 적용한 후, 처리된 데이터를 `sensor_gyro`와 같은 uORB 토픽으로 발행한다.


대부분의 드라이버는 NuttX의 `HPWORK`와 같은 공유된 고우선순위 작업 큐(Work Queue)에서 실행된다.20 이는 하드웨어 인터럽트를 효율적으로 처리하기 위한 "bottom half" 역할을 수행하기 위함이다. 인터럽트가 발생하면 최소한의 작업만 인터럽트 서비스 루틴(ISR)에서 처리하고, 나머지 데이터 처리 및 발행 작업은 작업 큐로 위임하여 중요한 제어 작업의 블로킹(blocking)을 방지하고 시스템의 응답성을 유지한다.



MAVLink는 QGroundControl과 같은 지상 통제 시스템(GCS)이나 컴패니언 컴퓨터와의 통신을 위한 주력 프로토콜이다.7 이 경량 메시징 프로토콜은 드론 생태계를 위해 설계되었다.

PX4 내의 `mavlink` 모듈은 브리지(bridge) 역할을 한다. 이 모듈은 `vehicle_attitude`와 같은 내부 uORB 토픽을 구독하고, 이를 MAVLink 메시지로 변환하여 시리얼 또는 UDP 링크를 통해 외부로 전송한다.13 반대로, `COMMAND_LONG`과 같은 MAVLink 명령을 수신하면 이를 `vehicle_command`와 같은 적절한 uORB 토픽으로 발행하여 시스템 내부의 다른 모듈들이 처리할 수 있도록 한다.13 MAVLink는 하이브리드 통신 패턴을 사용하는데, 텔레메트리 데이터는 다수의 수신자가 받을 수 있도록 멀티캐스트(multicast) 방식으로 스트리밍되고, 파라미터나 미션과 같이 신뢰성이 요구되는 데이터는 재전송을 포함한 점대점(point-to-point) 방식으로 전송된다.25

`mavlink` 모듈을 단순한 uORB-MAVLink 변환기로 설계한 것은 핵심 플라이트 스택을 외부 통신 프로토콜의 복잡성으로부터 격리시키는 중요한 아키텍처 결정이다. 컨트롤러나 추정기와 같은 플라이트 스택 모듈들은 오직 uORB 토픽에 대해서만 알면 되며, MAVLink의 존재 자체를 인지할 필요가 없다.1

`mavlink` 모듈이 이 변환을 전담함으로써 4 강력한 "관심사의 분리(Separation of Concerns)"가 달성된다. 만약 MAVLink 프로토콜이 업데이트되거나 심지어 다른 프로토콜로 대체되더라도, 오직 `mavlink` 모듈만 수정하면 된다. 핵심 비행 알고리즘은 전혀 영향을 받지 않는다. 이는 매우 견고하고 미래 지향적인 설계를 의미하며, v1.15의 새로운 uXRCE-DDS 브리지 역시 동일한 원칙에 따라 설계되었다. 즉, 핵심 시스템이 자신이 상호작용하는 외부 생태계에 구애받지 않도록 하는 또 다른 "번역기" 모듈인 것이다. 이러한 모듈성이 플라이트 스택의 근본적인 재작성 없이 ROS 2와의 원활한 통합을 가능하게 했다.


v1.15에서 도입되고 크게 개선된 이 미들웨어는 내부 uORB 세계와 외부 ROS 2/DDS 세계를 연결하는 다리 역할을 한다.14 컴패니언 컴퓨터에서는 micro XRCE-DDS 에이전트가 실행되고, 비행 컨트롤러에서는 클라이언트가 실행된다. 이 브리지는 uORB 메시지와 ROS 2 메시지 간의 직접적이고 양방향적인 교환을 가능하게 하여, 사실상 두 메시징 시스템을 상호 운용 가능하게 만든다.14 이것이 바로 새로운 ROS 2 인터페이스 라이브러리의 기술적 기반이다.


플라이트 스택은 미들웨어 계층이 제공하는 서비스를 기반으로, 기체를 실제로 비행시키는 작업을 수행하는 알고리즘의 집합이다.1 여기에는 상태 추정, 유도, 제어 로직이 포함된다.



ECL/EKF2(Extended Kalman Filter)는 PX4의 주력 상태 추정 모듈이다. 이 정교한 알고리즘은 다양한 노이즈가 섞인 센서 데이터를 융합하여 기체의 상태(자세, 위치, 속도 등)에 대한 안정적이고 정확한 추정치를 생성한다.27


이는 EKF2의 핵심적인 혁신 중 하나이다. 센서 측정값이 도착하는 즉시 융합하는 대신, EKF는 지연된 "융합 시간 지평(fusion time horizon)" 위에서 실행된다. 각 센서의 데이터는 FIFO 버퍼에 저장되며, EKF는 이 버퍼들로부터 데이터를 검색하여 모든 측정값이 동일한 시점에 정렬되도록 융합한다. 이는 GPS 데이터가 IMU 데이터보다 훨씬 늦게 도착하는 것과 같은 각 센서의 서로 다른 지연 시간을 보상한다.28

`EKF2_*_DELAY` 파라미터들이 이 보상 시간을 제어한다.


- **IMU:** IMU 데이터(델타 각도 및 델타 속도)는 센서 측정 간의 상태 예측(prediction)에 사용된다. 이는 가장 높은 빈도의 입력이며 추정의 근간을 형성한다.31
- **지자계 센서 (Magnetometer):** 요(Yaw)/헤딩 추정에 사용된다. `EKF2_MAG_TYPE` 파라미터는 융합 모드를 제어하는데, 자기장 이상에 강건한 단순 요 각도 융합 방식부터 자기장 오프셋을 추정할 수 있는 더 정확한 3축 융합 방식까지 다양하다.31
- **고도 (Height):** 주 고도 소스가 필요하다. 이는 기압계, GPS, 거리 측정기, 또는 외부 비전 시스템이 될 수 있으며, `EKF2_HGT_REF` 및 `EKF2_*_CTRL` 파라미터로 제어된다.28
- **위치/속도 (Position/Velocity):** GPS는 위치 및 속도 업데이트를 위한 가장 일반적인 소스이다. `EKF2_AID_MASK` 파라미터는 GPS, 비전 등 다양한 센서 유형의 융합을 활성화하거나 비활성화하는 비트마스크이다.29


PX4는 여러 개의 EKF2 인스턴스를 병렬로 실행할 수 있으며, 각 인스턴스는 서로 다른 IMU를 사용한다 (`EKF2_MULTI_IMU`).28 선택기(selector) 메커니즘이 각 인스턴스의 상태를 모니터링하며, 주 EKF에 장애가 발생하면 건강한 EKF로 페일오버(failover)를 수행하여 센서 고장에 대한 견고성을 크게 향상시킨다.



PX4는 표준적인 계단식 제어 아키텍처를 사용한다. 이는 외부 루프(outer loop)의 출력이 다음 내부 루프(inner loop)의 설정값(setpoint)이 되는 구조다.5 이 방식은 복잡한 제어 문제를 더 단순한 단계들로 분리한다. 멀티콥터의 일반적인 데이터 흐름은 다음과 같다: 

**위치(Position) -->> 속도(Velocity) -->> 자세(Attitude) -->> 각속도(Angular Rate) -->> 액추에이터(Actuators)**.5


- **위치 컨트롤러:** 위치 설정값과 현재 추정 위치를 입력받아 목표 속도 설정값을 출력하는 간단한 P-제어기이다.32 게인은 `MPC_XY_P` 파라미터로 제어된다.

- **속도 컨트롤러:** 속도 설정값과 추정 속도를 입력받아 목표 가속도 설정값을 출력하는 PID 제어기이다.32 게인은 

  `MPC_XY_VEL_P_ACC`, `MPC_XY_VEL_I_ACC` 등으로 제어된다.


- **자세 컨트롤러:** 쿼터니언(quaternion)으로 표현된 자세 설정값과 추정 자세를 입력받아 목표 각속도 설정값을 출력하는 P-제어기이다.32
- **각속도 컨트롤러:** 각속도 설정값과 자이로 센서로부터 추정된 각속도를 입력받아 토크(torque) 명령을 출력하는 PID 제어기이다.32 이는 가장 내부의 가장 중요한 제어 루프이다.

계단식 제어 아키텍처는 단순히 표준적인 관행을 따르는 것을 넘어, 모듈성과 튜닝 용이성을 본질적으로 향상시키는 설계이다. 각 제어 루프는 거의 독립적으로 튜닝하고 검증할 수 있어, 기체 제어라는 복잡한 문제를 체계적으로 단순화한다. 예를 들어, 가장 안쪽 루프인 각속도 컨트롤러는 자이로 데이터만을 사용하여 먼저 튜닝하여 기체의 근본적인 안정성을 확보할 수 있다. 각속도 루프가 안정화되면, 그 위에 자세 루프를 튜닝하며 자세 설정값을 추종하도록 P-게인에만 집중할 수 있다. 이러한 과정은 속도 및 위치 루프로 계속 이어진다.32 이 계층적 접근 방식은 튜닝 문제를 분리시킨다. 만약 기체가 위치 유지 모드에서 진동한다면, 문제는 근본적인 각속도 안정화가 아니라 외부 루프(위치/속도 게인)나 그 기반이 되는 상태 추정치에 있을 가능성이 높다. 이러한 체계적인 접근법은 신뢰할 수 있는 성능을 달성하는 데 매우 중요하다.


컨트롤러에서 나온 최종 토크 및 추력 명령은 `control_allocator` 모듈로 전달된다. 이 모듈은 기체의 지오메트리(geometry) 파일에 정의된 물리적 구성에 따라 이러한 추상적인 명령을 특정 액추에이터 출력(예: 모터 PWM 값)으로 변환한다.32 이는 제어 로직을 물리적인 기체 구성으로부터 분리시켜 재사용성을 극대화한다.


고정익 항공기의 경우, 위치 제어를 위해 총 에너지 제어 시스템(Total Energy Control System, TECS)이 사용된다. TECS는 스로틀과 피치 각도 설정값을 제어하여 고도와 대기속도를 동시에 관리한다.32



이 모듈은 기체의 중앙 두뇌 역할을 한다. 시동(arming)/해제(disarming), 비행 모드 전환, 페일세이프(failsafe) 동작 등 기체의 전반적인 상태를 관리하는 상태 머신(state machine)이다.35 GCS(MAVLink 경유)나 RC 조종기로부터 명령을 수신하고 어떤 제어 루프를 활성화할지 결정한다.

`commander` 모듈이 중앙 상태 머신으로서 기능하는 것은 중요한 안전 기능이다. 모든 모드 전환과 시동 로직을 중앙에서 관리함으로써, 충돌하는 명령을 방지하고 페일세이프 조치가 항상 최우선 순위를 갖도록 보장한다. 예를 들어, 사용자가 GCS를 통해 "미션" 모드를 명령하는 동시에 배터리 부족 페일세이프가 트리거되는 상황을 상상해보자. 중앙 중재자가 없다면 이러한 충돌하는 요청은 예측 불가능한 행동으로 이어질 수 있다. `commander` 모듈 35이 바로 이 중재자 역할을 한다. 이 모듈은 모든 명령 소스와 상태 업데이트(예: 

`battery_status`)를 구독하고, 페일세이프 조건이 사용자나 미션 명령을 무시하도록 하는 엄격한 상태 전이 로직을 구현한다.37 이는 안전이 나중에 추가된 기능이 아니라, 아키텍처의 핵심 원칙임을 보여준다. v1.15의 새로운 ROS 2 인터페이스가 이 페일세이프 상태 머신과 명시적으로 통합된다는 점 14은 이러한 안전 원칙이 외부에서 정의된 사용자 모드에까지 확장됨을 증명한다.


이 모듈은 상위 수준의 유도(guidance)를 담당한다. 미션 웨이포인트나 "이륙 지점으로 복귀(Return to Launch)"와 같은 다른 상위 수준 목표를 입력받아, 이를 위치 컨트롤러에 공급되는 위치 또는 속도 설정값으로 변환한다.4


이 섹션에서는 소프트웨어 모듈이 실제로 기본 운영체제에 의해 어떻게 실행되는지, 그리고 관련된 트레이드오프는 무엇인지 탐구한다.


PX4는 모듈이 실행될 수 있는 두 가지 독특한 방식을 제공하며, 이는 성능과 리소스 소비 간의 트레이드오프를 나타낸다.1


모듈은 전용 태스크로 실행될 수 있다. 이는 해당 모듈이 NuttX 스케줄러에 의해 관리되는 자신만의 스레드, 스택, 그리고 프로세스 우선순위를 갖는다는 것을 의미한다.6

- **장점:** uORB 토픽을 폴링(polling)하거나 잠시 멈추는(sleep) 것과 같은 블로킹(blocking) 작업을 수행할 수 있다. 전용 스택은 다른 태스크와의 간섭을 방지하며, 우선순위를 세밀하게 조정할 수 있다.
- **단점:** 각 태스크마다 전용 스택으로 인해 더 많은 RAM을 소비한다. 태스크 수가 많아지면 컨텍스트 스위칭 오버헤드가 증가할 수 있다.
- **사용 예:** 종종 이벤트를 기다려야 하는 `commander` 모듈은 태스크에 적합한 좋은 예이다. `src/templates/template_module` 예제는 태스크 생성 방법을 보여준다.39


모듈은 공유 작업 큐에서 실행될 수 있다. 이는 모듈의 작업 항목(work item)이 단일 공유 워커 스레드에 의해 서비스되는 큐에 추가된다는 것을 의미한다.1

- **장점:** 많은 모듈이 동일한 스택을 공유하므로 RAM 측면에서 매우 리소스 효율적이다. 태스크 스위칭이 적다.6

- **단점:** 작업 큐의 태스크들은 협력적으로 동작해야 하며, 블로킹하거나 잠들 수 없다.1 빠르게 실행하고 제어권을 양보해야 하므로, 오랜 시간이 걸리는 계산에는 부적합하다.

- **사용 예:** 빠른 읽기와 발행 작업이 필요한 센서 드라이버들은 낮은 지연 시간으로 인터럽트를 처리하기 위해 종종 `HPWORK`와 같은 고우선순위 작업 큐에 배치된다.20 `src/examples/work_item` 예제는 이 구현 패턴을 보여준다.39

이러한 이중 실행 모델(태스크 vs. 작업 큐)의 존재는 임베디드 시스템의 근본적인 딜레마, 즉 복잡하고 상태를 가지는 애플리케이션에 대한 요구와 RAM 및 CPU 사이클의 엄격한 제약 사이의 긴장 관계에 대한 아키텍처적 대응이다. 단순한 시스템이라면 모든 것을 하나의 모델로 처리할 수도 있었겠지만, PX4가 두 가지 독특한 모델을 지원하고 문서화했다는 사실 39은 두 가지 모두에 대한 명확한 필요성을 인식했음을 나타낸다. 모드 전환(`commander`)이나 미션 처리(`navigator`)와 같은 상위 수준 로직은 본질적으로 상태를 가지며 이벤트를 기다려야 하므로, 블로킹이 가능한 전용 `태스크`에 완벽하게 부합한다.35 반면, I2C 센서를 읽는 것과 같은 저수준 하드웨어 상호작용은 빠르고, 논블로킹(non-blocking)이며, 최소한의 오버헤드를 가져야 하므로 `작업 큐 태스크`에 완벽하게 부합한다.1 따라서 이 아키텍처는 개발자에게 지능적인 트레이드오프를 할 수 있는 도구를 제공한다. 이는 단순한 것부터 복잡한 것까지 광범위한 애플리케이션을 위해 설계된 성숙하고 잘 고려된 시스템의 징표이다. I2C/SPI 버스를 위한 특정 작업 큐를 만드는 것에 대한 논의 21는 이것이 활발한 최적화가 이루어지는 영역임을 더욱 잘 보여준다.


Linux와 같은 POSIX 시스템에서 PX4는 `SCHED_FIFO`와 같은 실시간 스케줄링 정책을 활용하여 자신의 스레드가 일반 시스템 프로세스보다 높은 우선순위를 갖도록 보장한다.41 NuttX에서는 스케줄러가 태스크의 우선순위를 지정하여 내부 루프 컨트롤러와 같이 가장 중요한 작업들의 실시간 성능을 보장한다.40 uORB를 통한 데이터의 가용성은 모듈 실행의 주된 동인이 된다.40

개발자는 모듈의 `CMakeLists.txt` 파일에서 `px4_add_module` 함수를 사용하여 각 태스크의 우선순위와 스택 크기를 정의할 수 있다.42


이 표는 새로운 모듈을 구현하는 방법을 선택해야 하는 개발자에게 매우 중요하다. 두 실행 모델을 핵심 기술 및 성능 지표에 따라 직접 비교하여 명확한 의사 결정 프레임워크를 제공한다. 이는 여러 소스 1에 흩어져 있는 복잡한 정보를 단일의 실행 가능한 참조 자료로 요약한 것이다.

| 기능              | 태스크 (Task)                    | 작업 큐 (Work Queue)                     | 일반적인 사용 사례                                        |
| ----------------- | -------------------------------- | ---------------------------------------- | --------------------------------------------------------- |
| **리소스 사용량** | 높음 (태스크당 전용 스택)        | 낮음 (공유 스택)                         | 메모리가 제한된 환경에서의 드라이버 및 간단한 주기적 작업 |
| **스케줄링**      | 선점형, 우선순위 기반            | 협력형, 비선점형 (큐 내에서)             | 실시간 제어 루프, 상태 머신                               |
| **블로킹/폴링**   | 허용 (`poll()`, `sleep()` 등)    | 허용되지 않음                            | 이벤트 대기가 필요한 모듈 (예: GCS 명령)                  |
| **실행 컨텍스트** | 자신만의 스레드                  | 공유 워커 스레드                         | 다수의 간단한 모듈을 효율적으로 실행                      |
| **구현 복잡도**   | 약간 높음 (스레드 생명주기 관리) | 더 간단함 (작업 항목 스케줄링)           | 빠른 개발 및 프로토타이핑                                 |
| **대표 모듈**     | `commander`, `navigator`         | `bmi088` (IMU 드라이버), `land_detector` | -                                                         |


이 섹션은 PX4의 역량을 재정의하는 v1.15의 구체적이고 중요한 변화들에 초점을 맞춘 보고서의 분석적 핵심이다.



v1.15는 PX4가 ROS 2와 상호작용하는 방식을 근본적으로 바꾸는 실험적인 라이브러리를 도입했다.14 이전 방식은 외부 제어를 위한 단일의 일반적인 모드였던 "오프보드 모드(Offboard Mode)"에 의존했다. 새로운 인터페이스는 ROS 2 노드가 PX4 내부 모드와 **동등한 수준의(peer)** 자체 비행 모드를 정의할 수 있도록 한다.14


- **이름이 지정되고 선택 가능한 모드:** ROS 2 모드는 GCS에 표시되는 고유한 이름을 가지므로, 사용자는 "Position", "Mission"과 같은 내장 모드처럼 이를 선택할 수 있다.38
- **페일세이프와의 통합:** 이러한 외부 모드는 `commander`의 페일세이프 상태 머신 및 시동 전 점검(arming checks)에 직접 통합된다.14 이는 기존 오프보드 모드에 비해 안전성과 견고성 측면에서 엄청난 개선이다.
- **모드 실행기 (Mode Executors):** "모드 실행기"라는 개념이 도입되었다. 이는 ROS 2에서 실행되는 상태 머신으로, 하나의 모드를 소유하고 다른 모드를 활성화할 수 있어, 컴패니언 컴퓨터에서 복잡하고 다단계적인 행동을 조율할 수 있게 한다.38


이는 고급 자율 비행 동작을 만드는 과정을 극적으로 단순화한다. 개발자는 이제 핵심 PX4 펌웨어를 수정하지 않고도, 컴퓨터 비전 기반 착륙과 같은 복잡한 로직을 표준 ROS 2 애플리케이션으로 작성하면서 PX4 플라이트 스택의 완전한 안전성과 통합의 이점을 누릴 수 있다.14


v1.15는 단순히 외부 인터페이스만 개선한 것이 아니다. 자율 비행의 근간이 되는 상태 추정기의 신뢰성과 성능을 대폭 향상시켰다.


릴리스 노트는 EKF2의 수렴 특성, 수치적 안정성, 그리고 계산 효율성의 주요 개선을 강조한다.14 특히 리 군(Lie group) 이론을 사용하여 쿼터니언 불확실성을 더 정확하게 기술함으로써 추정의 정밀도를 높였다.


- 새로운 `EKF2_MAG_TYPE` 모드인 "Init"이 추가되었다. 이 모드에서는 지자계 센서가 필터의 초기 헤딩을 설정하는 데에만 사용된다. 그 후에는 자이로와 GNSS와 같은 전역 위치 업데이트만으로 헤딩이 전파(propagate)된다.14 이는 자기적으로 열악한 환경에서 운용할 때 매우 유용하다.
- 이제 모든 융합 모드에서 지자계 센서 바이어스 추정이 허용되어, 시간이 지남에 따라 헤딩 정확도가 향상된다.14


지자계 센서의 일관성을 확인하는 로직(`EKF2_MAG_CHECK`)이 재작업되었으며, 자기장 강도(`EKF2_MAG_CHK_STR`)와 경사각(`EKF2_MAG_CHK_INC`)에 대한 허용 오차를 제어하는 새로운 파라미터가 추가되었다.14 이를 통해 비행 중 자기장 간섭을 더 신뢰성 있게 감지하고 복구할 수 있다.

v1.15에서 EKF2 추정기와 ROS 2 인터페이스의 병렬적인 발전은 우연이 아니다. 이는 더 복잡하고 신뢰할 수 있는 자율성을 가능하게 한다는 동일한 목표의 양면과 같다. 비전 기반 내비게이션이나 동적 장애물 회피와 같은 고급 자율 작업은 두 가지를 필요로 한다: 무거운 계산을 실행할 플랫폼(ROS 2를 탑재한 컴패니언 컴퓨터)과 극도로 신뢰성 높은 고성능 상태 추정치. 새로운 ROS 2 인터페이스 14는 복잡한 로직을 개발하고 배포할 플랫폼을 제공하고, EKF2의 향상 14은 그 로직이 의존하는 견고한 상태 추정치를 제공한다. 자율 알고리즘은 그 기반이 되는 위치 및 헤딩 추정치가 신뢰할 수 없을 때 무용지물이 되기 때문이다. 따라서 v1.15는 전체론적인 시스템 업그레이드로 볼 수 있다. 비행 컨트롤러는 더 견고한 "실시간 모션 서버"가 되고, 컴패니언 컴퓨터는 "애플리케이션 두뇌"가 된다. 개선된 인터페이스와 개선된 추정기는 시너지를 발휘하여 전체 시스템의 역량을 한 단계 끌어올린다.


이 표는 v1.15로 업그레이드하는 사용자를 위한 실용적인 가이드 역할을 한다. 가장 중요한 EKF2 변경 사항을 분리하고, 그 기능과 변경의 이유를 설명한다. 추정기 튜닝은 복잡하고 안전에 직결되는 작업이므로, 이러한 새로운 기능을 이해하는 것은 최적의 성능과 신뢰성을 달성하는 데 필수적이다.

| 파라미터               | v1.14 동작 (또는 해당 없음) | v1.15 기능                                                | 근거 / 영향                                            |
| ---------------------- | --------------------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| **`EKF2_MAG_TYPE`**    | 기존 융합 모드들            | 초기화 후 지자계 센서 없이 비행하기 위한 "Init" 모드 추가 | 자기적으로 거부된(denied) 환경에서의 견고한 운용 가능  |
| **`EKF2_MAG_CHECK`**   | 구 버전의 점검 로직         | 더 세분화된 점검을 포함하는 재작업된 로직                 | 자기 간섭에 대한 더 나은 감지 및 회복력                |
| **`EKF2_MAG_CHK_STR`** | 해당 없음                   | 자기장 강도 허용 오차를 정의하는 신규 파라미터            | 특정 지리적 위치나 예상 간섭 수준에 맞게 튜닝 가능     |
| **`EKF2_MAG_CHK_INC`** | 해당 없음                   | 자기장 경사각을 점검하는 신규 파라미터                    | 상태 점검에 새로운 차원을 추가하여 이상 감지 능력 향상 |

지자계 센서를 위한 "Init" 모드의 도입은 PX4의 운영 철학이 크게 성숙했음을 나타낸다. 이는 실제 전문적인 사용 사례에서는 완벽한 센서 데이터를 항상 사용할 수 없으며, 시스템이 자신의 의존성을 우아하게 줄여나갈 수 있어야 함을 인정한 것이다. 과거 많은 드론 시스템의 가정은 안정적인 헤딩을 위해 지속적이고 깨끗한 지자계 신호가 필요하다는 것이었다. 그러나 교량 점검이나 실내 비행과 같은 실제 운용 환경은 종종 이 가정을 위반하여 성능 저하나 시동 거부로 이어졌다. "Init" 모드 14는 이에 대한 실용적인 해결책이다. "비행 중에는 자기장이 신뢰할 수 없을 수 있음을 인정하지만, 지상에서 초기 헤딩 참조를 얻기 위해 필요하다"는 접근 방식을 취한다. 이는 이상적인 센서 환경을 위한 설계에서, 현실 세계의 도전적인 환경을 위한 설계로의 전환을 보여준다. 이는 플랫폼의 운용 범위를 넓히고 산업용 애플리케이션을 위한 더 실행 가능한 도구로 만들어준다.


- **시뮬레이션 분리:** 시뮬레이션 환경(예: Gazebo)과 PX4 SITL 인스턴스가 이제 분리되었다. 이들은 독립적으로 실행될 수 있으며, 네트워크를 통해 서로 다른 호스트에서 실행될 수도 있다.14 이는 복잡한 시뮬레이션 설정과 CI/CD 파이프라인의 유연성을 크게 향상시킨다.
- **내비게이션 개선:** 일반적인 "내비게이션에 대한 주요 개선"이 언급되었으며, 이는 EKF2 향상 및 더 나은 경로 계획 또는 페일세이프 로직과 관련될 가능성이 높다.14 지오펜스(Geofence) 개선 44이 구체적인 예시 중 하나이다.



본 보고서의 분석을 종합하면, PX4 v1.15는 점진적인 업데이트가 아닌 전략적인 도약으로 평가할 수 있다. 이 버전은 모듈성과 반응성이라는 아키텍처의 핵심 원칙을 공고히 하면서, 동시에 더 넓은 로보틱스 생태계와의 통합을 위한 역량을 대폭 확장했다. 그 결과, 전문적 및 연구용 애플리케이션을 위해 준비된 성숙하고 견고하며 고도로 확장 가능한 플랫폼이 탄생했다. 단일 코드베이스와 POSIX 호환성이라는 기반 위에 구축된 미들웨어와 플라이트 스택은, uORB라는 효율적인 내부 통신 시스템을 통해 유기적으로 결합된다. 태스크와 작업 큐라는 이중 실행 모델은 개발자에게 리소스와 기능 간의 균형을 맞출 수 있는 유연성을 제공한다. v1.15는 여기에 ROS 2와의 심층적인 통합과 EKF2의 근본적인 성능 향상을 더함으로써, PX4를 단순한 비행 제어기를 넘어 복잡한 자율 시스템의 핵심 구성 요소로 자리매김시켰다.


v1.15의 변화는 PX4의 미래 발전 방향을 명확하게 제시한다. 특히 uXRCE-DDS 브리지를 중심으로 한 미들웨어 계층의 중요성이 커짐에 따라, "온보드(onboard)" 로직과 "오프보드(offboard)" 로직 간의 경계는 계속해서 희미해질 것으로 예상된다. PX4는 분산된 대규모 로봇 시스템 내에서 실시간 비행 제어를 담당하는 최고의 구성 요소로 자신을 포지셔닝하고 있다.

향후 개발은 더욱 긴밀한 통합, 외부 명령에 대한 더 정교한 페일세이프 처리, 그리고 핵심 펌웨어 외부에서 정의되는 더 광범위한 상위 수준 자율 행동 지원에 초점을 맞출 가능성이 높다. EKF2의 지속적인 개선과 함께, 비전 및 기타 비전통적 센서 데이터를 융합하는 능력은 더욱 중요해질 것이다. 궁극적으로 PX4는 개발자들이 복잡한 자율 비행 문제를 해결하는 데 집중할 수 있도록, 안정적이고 신뢰할 수 있으며 고도로 연결된 비행 제어의 기반을 제공하는 방향으로 계속 진화할 것이다.


1. PX4 Architectural Overview | PX4 Guide (v1.15) - PX4 docs, accessed July 15, 2025, https://docs.px4.io/v1.15/en/concept/architecture.html
2. PX4 Architectural Overview | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/concept/architecture.html
3. PX4 Architectural Overview / Devguide - bresch, accessed July 15, 2025, https://bresch.gitbooks.io/devguide/content/en/concept/architecture.html
4. Architectural Overview / PX4 Development Guide - shnuzxd, accessed July 15, 2025, https://shnuzxd.gitbooks.io/px4-development-guide/en/concept/architecture.html
5. PX4: A Node-Based Multithreaded Open Source Robotics Framework for Deeply Embedded Platforms - Ethz, accessed July 15, 2025, https://people.inf.ethz.ch/pomarc/pubs/MeierICRA15.pdf
6. 1. PX4 software system introduction 2. PX4 Architecture - RflySim, accessed July 15, 2025, https://rflysim.com/doc/en/RflySimAPIs/2.RflySimUsage/0.ApiExps/e1_RflySimSoftwareReadme/Firmware/Readme.pdf
7. Basic Concepts | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/getting_started/px4_basic_concepts.html
8. Releases | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/releases/index.html
9. PX4 Development | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/development/development.html
10. Building PX4 Software | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/dev_setup/building_px4.html
11. Building the Code / PX4 Development Guide - shnuzxd, accessed July 15, 2025, https://shnuzxd.gitbooks.io/px4-development-guide/en/setup/building_px4.html
12. Building PX4 Software | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/dev_setup/building_px4.html
13. Modules Reference: Communication | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/modules/modules_communication.html
14. PX4-Autopilot v1.15 Release Notes | PX4 Guide (main) - PX4 docs, accessed July 15, 2025, https://docs.px4.io/main/en/releases/1.15.html
15. PX4 Autopilot User Guide | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/
16. Getting Started | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/dev_setup/getting_started.html
17. Ubuntu Development Environment | PX4 Guide (v1.15), accessed July 15, 2025, https://docs.px4.io/v1.15/en/dev_setup/dev_env_linux_ubuntu.html
18. PX4 System Architecture | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/concept/px4_systems_architecture.html
19. Adding a Frame Configuration | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/dev_airframes/adding_a_new_frame.html
20. Work Queues - NuttX latest documentation, accessed July 15, 2025, https://nuttx.apache.org/docs/latest/reference/os/wqueue.html
21. PX4 general work queue / Issue #8814 - GitHub, accessed July 15, 2025, https://github.com/PX4/Firmware/issues/8814
22. MAVLink - Clover, accessed July 15, 2025, https://clover.coex.tech/en/mavlink.html
23. PX4 and the art of Custom MAVLink Messages - Hamish Willee, Auterion - YouTube, accessed July 15, 2025, https://www.youtube.com/watch?v=qfDGdgIAU5g
24. MAVLink Messaging | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/mavlink/index.html
25. MAVLink Developer Guide, accessed July 15, 2025, https://mavlink.io/en/
26. Protocol Overview - MAVLink Guide, accessed July 15, 2025, https://mavlink.io/en/about/overview.html
27. Position Stabilization of Drones in Indoor Environments: Evaluation of Sensors and Techniques and Integration into the PX4 Flight Control Software - POLITECNICO DI TORINO, accessed July 15, 2025, https://webthesis.biblio.polito.it/28500/1/tesi.pdf
28. Using PX4's Navigation Filter (EKF2) | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/advanced_config/tuning_the_ecl_ekf.html
29. ECL/EKF Overview & Tuning - PX4 docs, accessed July 15, 2025, https://docs.px4.io/v1.11/en/advanced_config/tuning_the_ecl_ekf.html
30. EKF2 Estimation System - Dev documentation - ArduPilot, accessed July 15, 2025, https://ardupilot.org/dev/docs/ekf2-estimation-system.html
31. ecl EKF / px4-devguide - bkueng, accessed July 15, 2025, https://bkueng.gitbooks.io/px4-devguide/en/tutorials/tuning_the_ecl_ekf.html
32. Controller Diagrams | PX4 Guide (main) - PX4 docs, accessed July 15, 2025, https://docs.px4.io/v1.15/en/flight_stack/controller_diagrams.html
33. Control Allocation (Mixing) | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/concept/control_allocation.html
34. Controller Diagrams | PX4 User Guide (v1.13), accessed July 15, 2025, https://docs.px4.io/v1.13/en/flight_stack/controller_diagrams.html
35. Modules Reference: System | PX4 Guide (main), accessed July 15, 2025, https://docs.px4.io/main/en/modules/modules_system.html
36. PX4-Autopilot/src/modules/commander/Commander.cpp at main - GitHub, accessed July 15, 2025, https://github.com/PX4/PX4-Autopilot/blob/master/src/modules/commander/Commander.cpp
37. PX4 Firmware: src/modules/commander/state_machine_helper.cpp Source File, accessed July 15, 2025, https://px4.github.io/Firmware-Doxygen/df/d25/state__machine__helper_8cpp_source.html
38. PX4 ROS 2 Control Interface - GitHub, accessed July 15, 2025, https://github.com/PX4/PX4-user_guide/blob/main/en/ros2/px4_ros2_control_interface.md
39. PX4-user_guide/ja/modules/module_template.md at main - GitHub, accessed July 15, 2025, https://github.com/PX4/PX4-user_guide/blob/main/ja/modules/module_template.md?plain=1
40. Main loop and master schedule - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 15, 2025, https://discuss.px4.io/t/main-loop-and-master-schedule/7731
41. PX4 process isolation on Linux boards / Issue #16140 / PX4/PX4-Autopilot - GitHub, accessed July 15, 2025, https://github.com/PX4/PX4-Autopilot/issues/16140
42. PX4-Autopilot/cmake/px4_add_module.cmake at main - GitHub, accessed July 15, 2025, https://github.com/PX4/PX4-Autopilot/blob/master/cmake/px4_add_module.cmake
43. Cmake help - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 15, 2025, https://discuss.px4.io/t/cmake-help/4523
44. PX4 v1.15 public changes - what needs docs? - PX4 Autopilot, accessed July 15, 2025, https://discuss.px4.io/t/px4-v1-15-public-changes-what-needs-docs/39850


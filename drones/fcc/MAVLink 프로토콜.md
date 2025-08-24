# MAVLink 프로토콜
[드론 FCC](./index.md)


이 부분에서는 MAVLink의 근원, 설계 철학, 그리고 기능을 가능하게 하는 기술적 메커니즘을 추적하며 MAVLink의 '무엇'과 '왜'에 대한 근본적인 질문에 답한다. 이는 특히 보안 및 다른 프로토콜과의 비교 논의에서 프로토콜의 능력과 한계에 영향을 미치는 장단점을 이해하는 데 매우 중요하다.



MAVLink(Micro Air Vehicle Link)는 2009년 초 로렌츠 마이어(Lorenz Meier)에 의해 처음 출시되었다.1 이 시점은 대규모 상업용 드론 붐이 일어나기 전으로, 이는 MAVLink의 뿌리가 자원이 제한된 소형 항공기(micro air vehicles)에 초점을 맞춘 학계 및 취미 활동 커뮤니티에 있음을 시사한다.

프로토콜의 라이선스 구조는 광범위한 채택에 결정적인 역할을 했다. 메시지 정의 XML 파일과 생성된 C 언어 라이브러리는 MIT 라이선스 하에 제공되며, 생성기 툴체인은 LGPLv3 라이선스를 따른다.1 특히 라이브러리에 대한 MIT 라이선스의 허용적 성격은 소스 코드 공개 의무 없이 비공개 소스 애플리케이션에서도 사용할 수 있게 하여 상업적 채택을 촉진하는 핵심 요인이었다.5

MAVLink는 리눅스 재단 산하의 프로젝트인 드론코드 프로젝트(Dronecode Project)의 거버넌스 하에 운영된다.1 이는 표준에 대한 벤더 중립적인 환경을 제공하여 협업을 촉진하고 지속적인 개발과 유지보수를 보장한다. 초기 개발 동기는 다양한 드론 플랫폼이 확산됨에 따라 소형 무인 항공기를 위한 표준화되고 효율적인 통신 방법에 대한 필요성에서 비롯되었다.3


MAVLink는 명시적으로 "매우 가벼운 메시징 프로토콜"로 설계되었다.4 이는 프로토콜의 주요 설계 제약 조건이자 아키텍처 전반에 걸쳐 반복되는 주제이다. 제한된 RAM과 플래시 메모리를 가진 자원 제약적 시스템(예: 8비트 AVR 마이크로컨트롤러)에 최적화된 헤더 전용(header-only) 메시지 마샬링 라이브러리이다.2

효율성은 최소한의 오버헤드를 통해 달성된다. MAVLink 1은 패킷당 단 8바이트의 오버헤드를 가지며, MAVLink 2는 14바이트의 오버헤드를 가진다.1 이러한 바이너리 직렬화 방식은 XML이나 JSON과 같은 텍스트 기반 형식보다 훨씬 효율적이다.3

프로토콜은 하이브리드 발행-구독(publish-subscribe) 및 점대점(point-to-point) 설계 패턴을 따른다.4 자세, 위치와 같은 높은 빈도의 원격 측정 데이터 스트림은 멀티캐스트 방식(토픽 모드)으로 발행되는 반면, 미션이나 파라미터와 같은 설정 또는 명령 및 제어(C2) 하위 프로토콜은 보장된 전달을 위해 재전송 기능이 있는 점대점 방식으로 통신한다.16

이러한 설계 배경과 거버넌스 구조는 MAVLink의 진화를 정의하는 근본적인 긴장 관계를 드러낸다. 자원이 제한된 오픈소스 환경에서 탄생했기 때문에, 핵심 설계는 강력한 내장 보안과 같은 기능보다는 효율성을 우선시했다. 드론코드의 관리는 표준화를 보장하지만, 네이티브 암호화와 같은 급진적인 변경에 대해서는 커뮤니티 프로세스를 거쳐야 하므로 독점적인 개발보다 속도가 느릴 수 있다. 이 하이브리드 통신 모델은 매우 현명한 절충안이다. 이는 저대역폭 링크를 포화시킬 수 있는 고주파 원격 측정 패킷에 대한 모든 응답 확인(acknowledgement) 오버헤드를 피하면서도, 미션 업로드나 파라미터 변경과 같은 중요한 저주파 작업에 대해서는 신뢰성을 제공한다. 이는 MAVLink가 다양하고 까다로운 통신 채널에서 효과적으로 작동할 수 있게 하는 핵심적인 아키텍처 결정이다.4



MAVLink 1.0은 2013년경 널리 채택되었다.18 이 버전은 간단한 8바이트 헤더와 2바이트 체크섬을 특징으로 한다.2 메시지 ID 필드가 단일 바이트이기 때문에 메시지 ID가 256개(0-255)로 제한되었다.2 또한 인증이나 서명과 같은 네이티브 보안 기능이 전혀 없어 스푸핑 및 재전송 공격에 취약했다.20


MAVLink 2.0은 2017년 초부터 주요 사용자들이 채택하기 시작했다.18 패킷 헤더는 14바이트로 확장되었고, 선택적으로 13바이트의 서명을 추가할 수 있다.2 주요 특징은 다음과 같다: 24비트 메시지 ID(1,600만 개 이상의 메시지 지원), 인증을 위한 패킷 서명, 호환성을 깨지 않고 기존 메시지를 확장하는 기능, 그리고 페이로드 끝의 0으로 채워진 바이트를 절단(truncation)하는 기능이다.19

MAVLink 2 라이브러리는 하위 호환성을 갖도록 설계되어 MAVLink 1 패킷을 파싱하고 전송할 수 있다.16 이는 생태계의 원활한 전환을 보장하기 위한 중요한 설계 결정이었다.


버전 감지는 패킷의 첫 바이트로 처리된다: MAVLink 1은 `0xFE`, MAVLink 2는 `0xFD`이다.2 시스템이 MAVLink 1으로 통신을 시작하다가 MAVLink 2 패킷을 수신하면 MAVLink 2로 전환하는 "핸드셰이크" 프로토콜이 존재한다.18

`AUTOPILOT_VERSION` 메시지는 이 기능을 명시적으로 알리기 위해 `MAV_PROTOCOL_CAPABILITY_MAVLINK2` 플래그를 포함한다.18

그러나 이 협상 방식은 문제의 원인이 될 수 있다. 예를 들어, ArduPilot 4.2의 변경으로 인해 포트가 MAVLink 2로 설정되면 협상 핸드셰이크를 기대하는 구형 장치와의 호환성이 깨지면서 즉시 MAVLink 2를 전송하기 시작했다.24 이는 다양한 생태계에서 버전 전환의 실제적인 어려움을 보여준다. 또한, 시스템은 동일한 채널에서 서명된 MAVLink 2와 서명되지 않은 MAVLink 2, 또는 MAVLink 1과 MAVLink 2를 혼합하여 사용할 수 없다. 레거시 MAVLink 1 주변기기와 최신 서명된 MAVLink 2 시스템에 연결하려면 별도의 채널(예: 다른 직렬 포트)이 필요하다.18

MAVLink 1에서 2로의 전환은 대규모 분산 오픈소스 프로젝트 내에서 진화를 관리하는 사례 연구이다. 하위 호환성에 대한 집중은 생태계의 분열을 막는 데 가장 중요했다. 그러나 협상 로직의 미묘한 변화 24는 신중한 계획에도 불구하고 예기치 않은 변경이 발생하여 레거시 하드웨어를 사용하는 사용자에게 실제적인 문제를 일으킬 수 있음을 보여준다. "채널당 하나의 버전" 규칙 18은 중요한 아키텍처적 영향을 미친다. 비행 컨트롤러는 레거시 MAVLink 1 OSD를 위해 하나의 UART 포트를, 서명되지 않은 MAVLink 2 컴패니언 컴퓨터를 위해 다른 포트를, 그리고 서명된 MAVLink 2 장거리 라디오를 위해 세 번째 포트를 할당해야 할 수 있다. 이는 하드웨어 복잡성과 설정 부담을 증가시키며, 프로토콜 수준의 결정이 물리적 수준에 직접적인 결과를 낳는다는 것을 보여준다.



MAVLink 1.0 패킷은 고정된 8바이트 헤더로 구성된다. 여기에는 프레임 시작(1바이트), 페이로드 길이(1바이트), 패킷 시퀀스(1바이트), 시스템 ID(1바이트), 컴포넌트 ID(1바이트), 메시지 ID(1바이트)가 포함된다. 그 뒤에 페이로드(0-255바이트)와 2바이트 CRC 체크섬이 온다.2

`패킷 시퀀스` 번호는 패킷 손실 감지를 가능하게 한다.2

`시스템 ID`(SYSID)와 `컴포넌트 ID`(COMPID)는 송신자를 식별하여, 네트워크상에서 최대 255개의 시스템이 각각 여러 컴포넌트(예: 자동 조종 장치, IMU, 짐벌)를 가질 수 있도록 한다.2


MAVLink 2.0에서는 헤더가 확장되었다. 구조는 다음과 같다: 프레임 시작(1바이트), 페이로드 길이(1바이트), 비호환성 플래그(1바이트), 호환성 플래그(1바이트), 패킷 시퀀스(1바이트), 시스템 ID(1바이트), 컴포넌트 ID(1바이트), 메시지 ID(3바이트). 그 뒤에 페이로드(0-255바이트), 2바이트 CRC, 그리고 선택적인 13바이트 서명이 온다.2

- `비호환성 플래그(Incompatibility flags)`: 이 플래그의 비트가 설정되면, 수신 라이브러리는 해당 기능을 이해해야만 패킷을 처리할 수 있다. 이해하지 못하면 패킷을 폐기해야 한다. 주요 예는 서명 플래그이다.2
- `호환성 플래그(Compatibility flags)`: 이해하지 못하더라도 무시할 수 있는 기능을 나타내는 비트이다.2
- `메시지 ID`: 24비트로 확장되어 가능한 고유 메시지의 수가 대폭 증가했다.2
- `서명(Signature)`: 메시지 인증을 위해 추가되는 선택적 13바이트 필드이다.2

다음 표는 MAVLink 1과 MAVLink 2의 패킷 구조를 비교하여 주요 변경 사항을 명확히 보여준다.

| 필드 이름       | MAVLink 1 (바이트, 목적)                        | MAVLink 2 (바이트, 목적)                                     | 주요 변경점                                       |
| --------------- | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| 프레임 시작     | 0 (1), `0xFE`로 전송 시작을 알림                | 0 (1), `0xFD`로 전송 시작을 알림                             | 시작 바이트 변경으로 버전 구분                    |
| 페이로드 길이   | 1 (1), 페이로드의 길이(n)                       | 1 (1), 페이로드의 길이(n)                                    | 변경 없음                                         |
| 비호환성 플래그 | -                                               | 2 (1), 수신 측이 반드시 이해해야 하는 기능 플래그            | MAVLink 2의 새로운 기능 (예: 서명) 지원           |
| 호환성 플래그   | -                                               | 3 (1), 이해하지 못해도 무시할 수 있는 기능 플래그            | MAVLink 2의 새로운 기능 지원                      |
| 패킷 시퀀스     | 2 (1), 패킷 손실 감지를 위한 전송 시퀀스 카운터 | 4 (1), 패킷 손실 감지를 위한 전송 시퀀스 카운터              | 변경 없음 (위치만 변경)                           |
| 시스템 ID       | 3 (1), 송신 시스템 식별                         | 5 (1), 송신 시스템 식별                                      | 변경 없음 (위치만 변경)                           |
| 컴포넌트 ID     | 4 (1), 송신 컴포넌트 식별                       | 6 (1), 송신 컴포넌트 식별                                    | 변경 없음 (위치만 변경)                           |
| 메시지 ID       | 5 (1), 페이로드의 의미와 디코딩 방법을 정의     | 7-9 (3), 페이로드의 의미와 디코딩 방법을 정의                | 8비트에서 24비트로 확장되어 메시지 공간 대폭 증가 |
| 페이로드        | 6 ~ (n+6) (n), 메시지 데이터                    | 10 ~ (n+10) (n), 메시지 데이터                               | MAVLink 2는 끝의 0 바이트를 절단할 수 있음        |
| 체크섬 (CRC)    | (n+7) ~ (n+8) (2), 패킷 무결성 검사             | (n+11) ~ (n+12) (2), 패킷 무결성 검사                        | 변경 없음 (위치만 변경)                           |
| 서명            | -                                               | (n+13) ~ (n+25) (13), 신뢰할 수 있는 출처임을 확인하는 선택적 서명 | MAVLink 2의 핵심 보안 기능 추가                   |



MAVLink 메시지는 하드코딩되지 않고 XML 파일에 정의된다.4 각 XML 파일은 하나의 "다이얼렉트(dialect)"를 정의한다.4

`common.xml`은 대부분의 시스템에 유용한 표준 정의를 포함하며 MAVLink 프로젝트에서 관리한다.26 자동 조종 장치는 상호운용성을 보장하기 위해 가능한 한 이 공통 메시지를 사용해야 한다.26 ArduPilot과 같은 특정 비행 스택은 `common.xml`을 포함하고 시스템별 메시지를 추가하는 자체 다이얼렉트(예: `ardupilotmega.xml`)를 가지고 있다.27 PX4는 기본적으로 `development.xml` 또는 `common.xml`을 사용한다.29 XML 스키마는 메시지, 필드(타입, 이름, 단위 포함), 그리고 열거형(enum)을 정의한다.31


MAVLink 툴체인, 주로 `mavgen` 파이썬 스크립트는 이 XML 파일들을 파싱하여 특정 언어에 맞는 라이브러리(C의 헤더 파일, 파이썬의 클래스 등)를 자동으로 생성한다.1 이는 C, C++11, 파이썬, 자바, C#, Rust 등 다양한 언어를 지원하여 크로스플랫폼 개발을 가능하게 한다.1


임베디드 프로세서의 메모리 정렬(alignment)에 최적화하고 패딩 바이트를 방지하기 위해, MAVLink는 직렬화("over-the-wire" 형식) 전에 필드를 재정렬한다.2 필드는 네이티브 데이터 크기에 따라 내림차순으로 정렬된다(예: `uint64_t`가 먼저, `uint8_t`가 마지막). 안정 정렬(stable sort)이 사용되므로, 동일한 크기의 필드들은 XML 정의에서의 상대적 순서를 유지한다.22

이는 중요하지만 종종 혼란을 야기하는 기능이다. 이는 프로토콜 사양의 일부가 되는 구현 세부사항이며, 커스텀 구현은 호환성을 위해 이 정렬을 정확하게 복제해야 한다.36 MAVLink 2는 추가적인 최적화를 도입했다: 페이로드 끝의 비어있는(0으로 채워진) 바이트는 절단되어, 사용되지 않는 선택적 필드가 있는 메시지의 패킷 크기를 줄인다.22


각 패킷에는 전송 오류(패킷 손상)를 확인하기 위한 16비트 CRC(CRC-16/MCRF4XX)가 있다.2 두 번째로, 더 미묘한 무결성 검사가 존재하여 송신자와 수신자가 동일한 메시지 정의를 사용하고 있는지 확인한다. 

`CRC_EXTRA`라는 특별한 "시드" 값은 메시지 정의 자체(메시지 이름, 필드 타입, 필드 이름, *재정렬된* 와이어 형식의 필드 순서 및 배열 길이)로부터 계산된다.16

이 `CRC_EXTRA` 바이트는 최종 패킷 CRC가 계산되기 전에 데이터 스트림에 추가된다. 만약 GCS와 드론이 동일한 메시지 ID에 대해 다른 XML 정의를 가지고 있다면, `CRC_EXTRA` 값이 달라지고 최종 CRC가 불일치하여 패킷이 폐기된다.22 이는 프로토콜 초기 버전에서 문제가 되었던, 일치하지 않는 메시지 구조로 인한 데이터 손상을 방지한다.36

`CRC_EXTRA` 메커니즘은 분산된 생태계에서 "다이얼렉트 표류(dialect drift)" 문제에 대한 훌륭하고 실용적인 해결책이다. 이는 메시지의 구조적 정의의 해시를 런타임 체크섬에 내장하여, 전체 스키마를 각 패킷과 함께 전송하지 않고도 호환성을 강제하는 경량적이면서도 강력한 방법을 제공한다. 반면, 필드 재정렬 기능은 양날의 검과 같다. 이는 C 기반 임베디드 시스템에서 정렬 문제를 피함으로써 메모리와 처리 주기를 절약하는 매우 효과적인 최적화이다.36 그러나 MAVLink 파서를 처음부터 구현하는 사람에게는 안정 정렬 알고리즘을 완벽하게 복제해야 하므로 복잡성을 크게 증가시킨다. 이는 구현별 최적화(C 구조체 정렬)를 추상적인 프로토콜 사양과 혼합하는 것으로, 개발자 커뮤니티에서 논쟁의 여지가 있는 부분이다.37 이는 C/임베디드 세계에 대한 설계 편향을 보여준다.


이 부분은 이론에서 응용으로 넘어가, MAVLink가 주요 오픈소스 자동 조종 장치, 지상 관제국, 개발자 도구에서 어떻게 사용되는지 살펴본다. MAVLink 기반 시스템을 구축하고 운영하는 실제적인 현실을 탐구한다.



MAVLink는 자동 조종 장치와 GCS 또는 컴패니언 컴퓨터와 같은 외부 구성 요소 간의 통신을 위한 사실상의 표준이다.3 이는 다른 제조업체의 구성 요소 간 상호운용성을 제공한다.4 두 선도적인 오픈소스 비행 스택인 PX4와 ArduPilot 모두 MAVLink를 주요 외부 통신 프로토콜로 사용한다.3


PX4는 uORB라는 내부 발행-구독 메시징 시스템을 사용한다.29 PX4에서의 MAVLink 통신은 내부 uORB 토픽과 외부 MAVLink 메시지 간의 변환을 담당하는 브리지 역할을 하는 전용 `mavlink` 모듈에 의해 처리된다.29 새로운 MAVLink 메시지 스트림을 추가하려면, 개발자는 일반적으로 uORB 토픽을 구독하고 해당 MAVLink 메시지를 채워 전송하는 `MavlinkStream` 클래스를 생성한다.30 PX4의 빌드 시스템은 지정된 다이얼렉트 XML 파일(SITL의 경우 기본적으로 `development.xml`, 하드웨어의 경우 `common.xml`)로부터 필요한 MAVLink C 헤더 파일을 생성한다.29 QGroundControl의 MAVLink 콘솔은 PX4 시스템 셸(NuttShell/NSH)과 직접 상호 작용할 수 있게 하여 강력한 디버깅 인터페이스를 제공한다.40


ArduPilot 또한 GCS 통신 및 컴패니언 컴퓨터 통합을 위해 MAVLink를 광범위하게 사용한다.27 PX4의 uORB와 달리 ArduPilot의 내부 아키텍처는 다르지만, MAVLink 인터페이스는 동일한 목적을 수행한다. 다양한 기체 유형(콥터, 비행기, 로버)에 대한 유도 모드(Guided mode) 작동을 위한 잘 정의된 지원 명령어 세트를 가지고 있다.27 파라미터 처리는 핵심적인 MAVLink 기능이다. ArduPilot은 전체 파라미터 목록 검색(`PARAM_REQUEST_LIST`), 단일 파라미터 읽기(`PARAM_REQUEST_READ`), 파라미터 설정(`PARAM_SET`)을 지원한다.42 ArduPilot은 `common.xml`을 포함하지만 자체 기능 세트에 특화된 많은 사용자 정의 메시지와 파라미터를 추가하는 `ardupilotmega.xml` 다이얼렉트를 유지 관리한다.27

PX4(uORB 기반)와 ArduPilot의 서로 다른 내부 아키텍처는 MAVLink가 추상화 계층으로서 성공했음을 보여준다. 내부적인 차이에도 불구하고, 두 시스템 모두 외부적으로 "MAVLink를 사용"하기 때문에 QGroundControl과 같은 동일한 GCS 및 MAVSDK와 같은 단일 SDK와 통신할 수 있다.38 MAVLink는 비행 제어 시스템을 지상 제어 및 주변기기 생태계로부터 효과적으로 분리시킨다. 이는 오픈소스 드론 커뮤니티에 있어 중요한 전략적 결정이었다.



QGroundControl(QGC)은 MAVLink를 지원하는 모든 기체와 작동하도록 설계된 모든 기능을 갖춘 GCS이다.38 PX4와 ArduPilot을 모두 지원한다.38 QGC는 비행 제어, 임무 계획, 기체 설정/파라미터 구성, 원격 측정 데이터 표시 등 모든 주요 기능에 MAVLink를 사용한다.38 MAVLink Inspector는 개발자를 위한 강력한 QGC 도구로, 모든 수신 MAVLink 메시지와 그 필드를 실시간으로 검사하고 차트로 표시할 수 있다.45 이는 사용자 정의 메시지 구현을 디버깅하는 데 매우 유용하다. QGC는 다양한 비행 스택과의 호환성을 극대화하기 위해 기본적으로 `all.xml` 다이얼렉트를 포함하도록 사용자 정의할 수 있다.46 또한 QGC는 JSON 파일을 통해 구성된 화면 또는 조이스틱 버튼을 통해 사용자 정의 MAVLink 명령을 전송할 수 있어 유연한 사용자 정의 작업을 가능하게 한다.47


컴패니언 컴퓨터(예: Raspberry Pi, NVIDIA Jetson)는 드론에 고급 기능(예: 컴퓨터 비전, AI)을 추가하는 데 사용된다.48 이들은 직렬 또는 UDP 연결을 통해 비행 컨트롤러와 통신하며, MAVLink를 사용하여 원격 측정 데이터를 수신하고 고수준 명령을 전송한다.48 이 설정을 통해 컴패니언 컴퓨터는 모든 비행 데이터(자세, GPS 등)에 접근하고, 예를 들어 유도 모드(ArduPilot)에서 속도 설정점을 보내거나 오프보드 모드(PX4)를 사용하여 기체를 제어할 수 있다.48


여러 MAVLink 구성 요소(예: 비행 컨트롤러, 컴패니언 컴퓨터, 짐벌, GCS)가 있는 복잡한 시스템에서는 메시지를 올바르게 라우팅해야 한다. ArduPilot에는 어떤 SYSID/COMPID 쌍이 어떤 통신 채널(예: Telem1, Telem2, USB)에 있는지 학습하고 대상 메시지를 그에 따라 전달하는 내장 라우팅 클래스가 있다.52

더 복잡한 시나리오, 특히 컴패니언 컴퓨터에서는 `mavlink-router`가 여러 엔드포인트(UART, UDP, TCP) 간에 MAVLink 트래픽을 라우팅할 수 있는 전용 오픈소스 도구이다.53 예를 들어, 비행 컨트롤러에서 단일 직렬 스트림을 받아 Wi-Fi를 통해 GCS로 브로드캐스트하고 동시에 컴패니언 컴퓨터의 로컬 프로세스로 전달할 수 있다.54

`mavlink-router`는 또한 비행 컨트롤러에서 SD 카드를 물리적으로 꺼낼 필요 없이 오프보드 로깅과 같은 유용한 기능을 제공한다.54

MAVLink 라우팅의 개념은 프로토콜이 단순한 점대점 링크에서 시스템들의 시스템을 위한 진정한 네트워킹 프로토콜로 진화했음을 보여준다. `mavlink-router` 55와 ArduPilot의 내장 로직 52과 같은 도구의 필요성은 드론이 더 복잡해짐에 따라 통신 아키텍처도 단순한 "파이프"에서 관리되는 "버스"로 진화해야 함을 보여준다. 온보드 복잡성이 증가함에 따라 네트워킹 기능이 추상화되어 비행에 필수적이지 않은 더 강력한 프로세서(컴패니언 컴퓨터)로 이동하는 명확한 아키텍처 패턴을 관찰할 수 있다.



Pymavlink는 MAVLink의 공식 파이썬 구현이며 MAVLink 프로젝트의 핵심 부분이다.35

`mavgen` 생성기와 연결 생성 및 메시지 송수신을 위한 `mavutil` 라이브러리를 포함한다.58 이는 저수준의 세분화된 제어를 제공하여 개발자가 모든 MAVLink 메시지를 구성하고 파싱할 수 있게 한다. 스크립팅, 테스트 및 프로토콜과 직접 상호 작용해야 하는 애플리케이션에 자주 사용된다.50 예제들은 직렬 또는 UDP를 통한 연결, 하트비트 대기, 명령 전송(`command_long_send`), 비행 모드 변경 등에 사용됨을 보여준다.58


MAVSDK는 MAVLink 기체를 제어하기 위한 간단하고 비동기적인 API를 제공하는 현대적인 고수준 라이브러리이다.43 C++로 작성되었으며 파이썬, 스위프트, 자바 등을 위한 래퍼(wrapper)를 제공한다.43 이는 개별 MAVLink 메시지의 복잡성을 추상화한다. 개발자는 이륙을 위해 특정 파라미터가 포함된 

`COMMAND_LONG` 메시지를 보내는 대신, 간단히 `action.takeoff()`를 호출한다.61 모바일 GCS나 고수준 컴패니언 컴퓨터 소프트웨어와 같은 애플리케이션을 구축하기 위해 설계되었다. 원시 프로토콜 수준에서 작업할 필요가 없는 대부분의 애플리케이션 개발자에게 권장되는 경로이다.43 MAVSDK는 크로스플랫폼이며 드론코드 생태계의 일부로 유지 관리된다.43 최근 MAVSDK 3.0의 출시는 활발한 개발과 성숙을 나타낸다.64

Pymavlink와 MAVSDK의 존재는 소프트웨어 엔지니어링의 전형적인 계층화 패턴을 보여주며 MAVLink 생태계의 성숙을 반영한다. Pymavlink는 "프로토콜 수준" 도구인 반면, MAVSDK는 "애플리케이션 수준" 프레임워크이다. 초기에 개발자들은 MAVLink 상호작용을 직접 스크립팅할 방법이 필요했고, 이는 Pymavlink로 이어졌다. 이는 강력하지만 깊은 프로토콜 지식을 요구한다. 더 복잡한 애플리케이션이 구축되면서 공통 패턴(연결, 시동, 이륙, 착륙, 임무 수행)이 나타났다. 모든 애플리케이션에서 이 복잡하고 저수준의 MAVLink 메시지 처리를 반복하는 것은 비효율적이고 오류가 발생하기 쉽다. 이로 인해 MAVSDK 개발로 이어진 고수준 추상화의 필요성이 대두되었다. MAVSDK는 MAVLink의 복잡한 세부 사항을 숨기는 깨끗하고 현대적인 비동기 API를 제공하여 애플리케이션 프로그래머의 개발 속도를 높이고 신뢰성을 향상시킨다. 이 두 계층의 툴체인은 서로 다른 개발자 요구를 충족시킨다: 프로토콜 전문가/테스터는 Pymavlink를 사용하고, 애플리케이션 개발자는 MAVSDK를 사용한다.


이 부분은 MAVLink의 보안 상태에 대한 집중적이고 심층적인 검토를 제공한다. 학술 연구, 프로토콜 사양 및 실제 구현을 종합하여 취약점과 대응책을 평가한다.



MAVLink 1.0은 네이티브 기밀성(암호화)이나 인증 기능을 제공하지 않는다.20 모든 데이터는 평문으로 전송되어 도청(패킷 스니핑)에 취약하다.20 인증 부재는 메시지 스푸핑(공격자가 가짜 명령 주입), 재전송 공격(공격자가 유효한 명령을 기록하고 재전송), 중간자 공격(PITM) 등 광범위한 공격을 가능하게 한다.20 2014년의 초기 연구는 보안되지 않은 ArduPilot 시스템에 대한 성공적인 공격을 시연하여 기밀성, 무결성, 가용성을 침해했다.66 간단한 공격은 특정 라디오의 NetID 처리 설계 결함을 이용하여 드론을 탈취할 수도 있었다.12


- **퍼징(Fuzzing)**: 연구자들은 MAVLink 구현에 퍼즈 테스트를 적용하여, 자동 조종 시스템을 다운시킬 수 있는 소프트웨어 취약점을 찾기 위해 변형되거나 무작위적인 패킷을 전송했다.7 이는 프로토콜 수준의 공격을 넘어 구현 수준의 공격으로 나아간다.

- **정형 기법(Formal Methods)**: 최근 연구에서는 정제된 다자 세션 타입(RMPSTs)을 사용하여 MAVLink 프로토콜의 의도된 동작을 공식적으로 모델링한다. 이는 개별적으로는 유효한 메시지 시퀀스가 안전하지 않은 상태로 이어지는 "은밀한 공격"을 탐지할 수 있는데, 이는 단순한 패킷 검사로는 놓칠 수 있는 취약점이다.70 예를 들어, 

  `MISSION_COUNT` 메시지 뒤에 올바른 수의 미션 아이템 메시지가 오는지 확인하는 것이다.70

- **침입 탐지 시스템(IDS)**: 연구자들은 머신러닝(LSTM 모델)을 사용하여 MAVLink 트래픽의 순서와 빈도를 분석하여, 공격의 구체적인 내용을 알지 못하더라도 거짓 MAVLink 주입 공격을 나타내는 이상 징후를 탐지하는 네트워크 수준 IDS(예: MUVIDS)를 제안했다.69

MAVLink 보안에 대한 학술 연구 궤적은 정교함에서 뚜렷한 진전을 보여준다. 이는 기본적인 프로토콜 수준의 공격(스푸핑을 통한 탈취 12)을 입증하는 것에서부터, 미묘한 구현 버그 찾기(퍼징 7), 메시지 시퀀스의 복잡한 논리적 결함 식별(정형 기법 70), 그리고 마지막으로 행동 기반, AI 기반 방어 메커니즘 개발(IDS 69)으로 이동했다. 이는 더 성숙한 IT 분야에서 볼 수 있는 보안의 '고양이와 쥐' 게임이 드론 생태계에서도 벌어지고 있음을 보여준다.



MAVLink 2는 신뢰할 수 있는 출처로부터의 메시지임을 확인(인증)하고 변조되지 않았음을 확인(무결성)하기 위해 선택적 메시지 서명을 도입했다.13 서명된 패킷은 `incompatibility_flags` 필드의 플래그로 표시되며 13바이트의 서명이 추가된다.2 서명은 6바이트 링크 ID, 48비트 타임스탬프, 그리고 48비트 서명 해시로 구성된다.25

해시는 비밀 키와 전체 패킷(헤더, 페이로드, CRC), 링크 ID, 타임스탬프를 연결한 값으로부터 계산된다. SHA-256 해시의 첫 6바이트(48비트)가 사용된다.25

`타임스탬프`는 링크가 설정된 이후의 마이크로초를 나타내는 48비트 값이다. 수신자가 마지막으로 수신한 메시지보다 오래된 타임스탬프를 가진 메시지를 거부하므로 간단한 재전송 공격을 방지하는 데 사용된다.25

`링크 ID`는 각 통신 채널에 대한 고유 ID로, 한 링크(예: 보안 USB 연결)에서 기록된 명령이 다른 링크(예: 비보안 무선 링크)에서 재전송되는 교차 채널 재전송 공격을 방지하기 위해 설계되었다.23


비밀 키는 신뢰하는 당사자(예: GCS 및 자동 조종 장치) 간에 공유되는 32바이트 값이다.25 키는 안전하게 관리되어야 한다. USB와 같은 보안된 물리적 링크를 통해서만 전송되어야 하는 `SETUP_SIGNING` 메시지를 사용하여 설정할 수 있다.23 공개적으로 접근 가능한 MAVLink 파라미터나 로그 파일에 저장되어서는 안 된다.25 C (`mavgen`) 및 파이썬 (`pymavlink`) 라이브러리 모두 서명 활성화, 키 설정, 타임스탬프 관리를 위한 함수를 제공한다.72 Mission Planner는 ArduPilot과의 서명 설정을 위한 GUI를 제공한다.75


서명은 *선택 사항*이다. 생태계는 완전히 서명되지 않은 MAVLink 2 모드로 작동할 수 있다. 구현은 특정 서명되지 않은 패킷을 조건부로 수락하는 메커니즘을 제공해야 한다. 예를 들어, 서명을 지원하지 않는 원격 측정 라디오의 `RADIO_STATUS` 메시지는 링크가 작동하기 위해 수락되어야 한다.23 이 콜백 메커니즘(`accept_unsigned_callback`)은 잘못 구성될 경우 잠재적인 약점이 될 수 있다. 잘못 서명된 패킷을 수락하는 시스템은 사용자에게 링크가 안전하지 않다는 명확하고 눈에 띄는 경고를 제공해야 한다.25

MAVLink의 메시지 서명은 인증 및 재전송 공격을 해결하는 잘 설계된 암호학적 통제 수단이지만, 그 보안은 전적으로 사용자의 운영 규율에 달려 있다. 선택적 성격과 일부 서명되지 않은 패킷을 수용해야 하는 필요성은 취약한 보안 경계를 만든다. 프로토콜 설계자들은 전체 생태계를 깨뜨리거나 큰 부담을 주지 않으면서 인증을 추가해야 했다. 그 해결책은 선택적 서명이었다. 재전송 공격을 핵심 위협으로 정확히 식별하고, 단조롭게 증가하는 타임스탬프를 서명 해시에 포함시켰다. 교차 채널 재전송 공격을 식별하고, 링크별 ID를 해시에 포함시켰다. 일부 레거시 구성 요소가 패킷에 서명할 수 없다는 것을 인식하고, 특정 서명되지 않은 메시지를 통과시키기 위한 콜백을 만들었다. 그 결과, 암호학적으로는 건전하지만 실제 효과는 사용자가 실제로 활성화하는지, 보안 채널을 통해 키를 설정하는지, 개발자가 서명되지 않은 패킷 예외 규칙을 올바르게 구현하는지에 따라 달라지는 메커니즘이 탄생했다. 이러한 운영 단계 중 하나라도 실패하면 전체 메커니즘이 무용지물이 될 수 있어 "깨지기 쉬운" 보안 기능이 된다.



공식 MAVLink 프로토콜은 메시지 암호화를 *제공하지 않는다*.6 이는 암호화가 상당한 계산 오버헤드와 지연 시간을 추가하기 때문에, 프로토콜이 경량이고 효율적이어야 한다는 철학에 뿌리를 둔 의도적인 설계 선택이다.12 커뮤니티의 오랜 합의는 저전력 8비트 시스템을 배제하지 않기 위해 보안은 프로토콜 자체에 내장하기보다는 전송 시스템(예: 암호화된 무선 링크)에서 처리해야 한다는 것이었다.12


네이티브 암호화의 부재는 이제 특히 민감한 군사 또는 상업 작전에서 주요 취약점으로 널리 인식되고 있다.76 수많은 학술 연구에서 애플리케이션 계층에서 암호화를 통합하는 방법을 제안하고 구현했다. 이는 MAVLink 페이로드를 패킹하고 전송하기 전에 암호화하는 것을 포함한다.65 연구자들은 AES-CTR, ChaCha20, Speck-CTR 등 다양한 표준 암호를 실제 드론 하드웨어에서 실행되는 MAVLink에 성공적으로 통합하고 테스트했다.77 MAVShield라는 새로운 경량 암호는 MAVLink를 위해 특별히 제안되었으며, 기존 암호보다 전력 소비와 CPU 부하를 약간만 증가시키면서 더 나은 성능을 보인다고 주장한다.78


초기 분석에 따르면 보안되지 않은 MAVLink 통신조차도 측정 가능한 전력 소비 및 지연 시간 오버헤드를 추가하며, 이는 자원 집약적인 공격 중에 악화된다.66 암호화를 구현하면 추가 오버헤드가 발생한다. AES-CTR을 통합한 한 연구에서는 암호화/복호화 시간이 짧고 메모리 및 CPU 부하가 약간만 증가하는 것으로 나타났다.79 MAVShield 연구는 실제 드론에서 배터리 전력 소비 5.6%, CPU 부하 2.9% 증가로 오버헤드를 정량화하여, 현대적인 경량 암호가 유능한 비행 컨트롤러에서 실제 배포에 실용적이 되고 있음을 시사한다.78

MAVLink 커뮤니티는 암호화와 관련하여 기로에 서 있다. "보안은 전송 계층의 책임"이라는 원래의 철학 12은 드론이 전송이 안전하지 않을 수 있는 IP 네트워크(Wi-Fi, LTE, 위성 통신)에 점점 더 많이 연결되면서 유지하기 어려워지고 있다. 학계는 애플리케이션 계층 암호화가 실현 가능하다는 것을 증명하고 있지만 78, MAVLink 프로토콜 자체 내에는 공식적이고 표준화된 구현이 없다. 이는 MAVLink의 미래에 대한 중요한 전략적 과제를 제기한다. MAVLink가 상호운용성이라는 핵심 가치를 훼손하지 않으면서 암호화를 표준화하지 못하면, 벤더와 연구자들이 각자의 호환되지 않는 보안 계층을 구현하게 되어 생태계가 파편화될 위험이 있다.


이 부분은 특화된 MAVLink 마이크로서비스와 복잡한 실제 시나리오에서의 응용을 탐구하며, 단순한 원격 측정 스트리밍을 넘어서는 프로토콜의 유연성을 보여준다.



기본 MAVLink 메시지는 "발사 후 망각(fire-and-forget)" 방식이지만, 명령 프로토콜(Command Protocol)은 중요한 작업에 대해 보장된 전달을 제공한다.17 이는 명령(`MAV_CMD` 열거형 값)과 그 파라미터를 `COMMAND_LONG` 또는 `COMMAND_INT` 메시지에 패키징하여 작동한다. 수신자는 `COMMAND_ACK` 메시지로 응답해야 한다.17 송신자가 시간 초과 내에 ACK를 받지 못하면 자동으로 명령을 재전송한다.17 이는 UDP와 같은 비신뢰성 링크에서 신뢰성을 제공한다. 이 프로토콜은 수신자가 `MAV_RESULT_IN_PROGRESS` 상태의 `COMMAND_ACK` 메시지를 주기적으로 보내 장기 실행 명령을 지원한다.17


카메라 프로토콜(현재 v2)은 카메라 페이로드의 명령 및 제어를 표준화한다.81 이는 임시방편적인, 벤더별 제어 방법을 넘어서는 중요한 진전이다. 카메라 식별(`CAMERA_INFORMATION`), 설정 조회(`CAMERA_SETTINGS`), 상태(`STORAGE_INFORMATION`, `VIDEO_STREAM_INFORMATION`)를 위한 메시지를 정의한다.81 사진 촬영(`MAV_CMD_IMAGE_START_CAPTURE`), 비디오 녹화 시작/중지, 줌/초점 제어와 같은 작업을 위해 명령(`MAV_CMD`)이 사용된다.81

핵심 기능은 카메라 정의 파일(Camera Definition File)로, 카메라의 모든 파라미터를 설명하는 XML 파일이다(HTTP 또는 MAVLink FTP를 통해 카메라에서 호스팅됨). 이를 통해 QGroundControl과 같은 GCS는 특정 기능에 대한 사전 지식 없이도 모든 호환 카메라에 대한 전체 사용자 인터페이스를 동적으로 생성할 수 있다.81

`mavlink-camera-manager`는 컴패니언 컴퓨터에서 실행되어 다양한 소스(USB, IP 카메라)의 비디오를 관리하고 스트리밍하며 MAVLink 호환 카메라로 노출시키는 서버 애플리케이션이다.83

카메라 및 명령 프로토콜은 MAVLink의 "마이크로서비스" 계층을 대표한다. 이는 프로토콜이 단순한 메시지 전달 라이브러리에서 복잡하고 상태를 갖는 상호작용을 정의하는 프레임워크로 성숙했음을 보여준다. 특히 카메라 정의 파일은 페이로드에 대한 진정한 "플러그 앤 플레이" 상호운용성을 촉진하는 강력한 개념이다. 이 설계는 GCS가 특정 카메라 하드웨어에 대해 미리 프로그래밍될 필요 없이, 정의 파일을 파싱하고 UI를 렌더링하는 방법만 알면 되도록 하여 GCS와 카메라 하드웨어를 분리한다. 이는 매우 확장 가능하고 유연한 아키텍처 패턴이다.



MAVLink는 많은 드론 스웜 연구 플랫폼에서 기본 통신 프로토콜로 사용된다.84 아키텍처는 다양하다. 중앙 집중식 제어는 GCS가 스웜의 각 드론에 개별 명령을 보내는 것을 포함한다(예: Mission Planner의 스웜 인터페이스 85). 분산 제어는 더 발전된 형태로, 드론들이 서로 통신하여 행동을 조율한다. 이는 종종 각 드론의 컴패니언 컴퓨터에서 실행되는 편대 비행, 충돌 회피, 작업 할당 알고리즘을 포함한다.51 이러한 분산 시스템에서 MAVLink는 GCS와 스웜 간의 통신에 사용될 수 있으며, 저지연 드론 간 통신에는 다른 프로토콜(예: ESP-NOW)이 사용될 수 있다.51 MDS 3와 같은 프로젝트는 드론 측 애플리케이션과 클라우드 백엔드 간의 임무 관리를 위해 MAVLink를 사용한다.86


BVLOS(Beyond Visual Line of Sight) 운용은 종종 LTE, C-Band 라디오, 위성 통신(SATCOM)과 같은 통신 기술의 조합을 사용하여 장거리에 걸쳐 매우 신뢰성 있는 명령 및 제어(C2) 링크를 필요로 한다.88 MAVLink는 낮은 대역폭 요구 사항 때문에 이러한 시나리오에 적합하다.90 UDP는 이러한 다양한 링크 유형과의 광범위한 호환성 때문에 종종 선택되는 전송 프로토콜이다.92

핵심 과제는 링크 품질을 관리하고 다른 통신 경로 간에 전환하는 것이다. 고급 시스템은 "링크 집행 관리자(Link Executive Manager)"를 사용하여 각 링크(LTE, SATCOM 등)의 성능을 모니터링하고 사용 가능한 최상의 경로를 통해 MAVLink 트래픽을 동적으로 라우팅하여 단일의 고신뢰성 본딩 C2 링크를 생성한다.88 위성 통신과 같은 고지연 링크에서 대역폭을 더욱 줄이기 위해, 사용자 정의 MAVLink 라이브러리를 사용하여 메시지를 필터링하고 가장 중요한 데이터만 전송할 수 있다.89

다음 표는 BVLOS 비행에 사용되는 다양한 통신 기술을 요약하고 MAVLink와의 통합 방법을 설명한다.

| 기술                   | 일반적인 범위           | 대역폭    | 지연 시간 | 주요 장점                                      | 주요 과제                                            | MAVLink 통합 방법                                            |
| ---------------------- | ----------------------- | --------- | --------- | ---------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| 4G/5G LTE              | 셀룰러 네트워크 범위 내 | 높음      | 낮음      | 넓은 커버리지, 고대역폭, 기존 인프라 활용      | 네트워크 혼잡, 외딴 지역의 커버리지 부족, 운영 비용  | IP 링크를 통한 UDP, 링크 집행 관리자에 의해 관리 89          |
| 위성 통신(SATCOM)      | 전 세계적               | 낮음-중간 | 높음      | 진정한 글로벌 커버리지, 인프라 독립적          | 높은 지연 시간, 높은 운영 비용, 낮은 대역폭          | IP 링크를 통한 UDP, 대역폭 최적화를 위한 메시지 필터링 88    |
| C-Band/ISM Band 라디오 | 수십 km (가시선)        | 중간      | 매우 낮음 | 저지연, 낮은 운영 비용, 높은 신뢰성(가시선 내) | 가시선(LOS) 요구 사항, 지형에 의한 방해, 제한된 범위 | 직렬 또는 IP 링크를 통한 UDP, 링크 집행 관리자의 일부 88     |
| 다중 링크 본딩         | 결합된 링크의 범위      | 결합됨    | 가변적    | 높은 신뢰성, 단일 장애점 없음, 동적 경로 선택  | 시스템 복잡성 증가, 추가 하드웨어/소프트웨어 필요    | 링크 집행 관리자가 여러 UDP 스트림을 관리하고 최상의 경로로 MAVLink 패킷을 라우팅 88 |


이 마지막 부분은 MAVLink를 더 넓은 로봇 공학 생태계의 맥락에 놓고, 주요 대안과 비교하며, 현재 동향과 커뮤니티 논의를 바탕으로 미래 개발 경로를 분석한다.



- **MAVLink**: 저대역폭, 자원 제약 환경에 최적화된 경량의 점대점 및 발행/구독 원격 측정 프로토콜이다. 드론 세계에서 성숙하고 잘 정의된 표준이다.9
- **DDS (Data Distribution Service)**: 데이터 중심, 실시간, 발행/구독 미들웨어를 위한 OMG 표준이다. 내장된 서비스 품질(QoS) 정책, 동적 발견, 확장 가능한 개방형 아키텍처 시스템에 중점을 두어 MAVLink보다 기능이 풍부하다.95 ROS 2의 기본 통신 미들웨어이다.96
- **ROS 2**: DDS가 통신 계층인 완전한 로봇 공학 프레임워크이다. 통신뿐만 아니라 로봇 공학의 모든 측면을 위한 방대한 도구, 라이브러리, 패키지 생태계를 제공한다.96
- **주요 차이점**: MAVLink는 *프로토콜*인 반면, DDS는 *미들웨어 표준*이고 ROS 2는 *프레임워크*이다. MAVLink에서 사용자 정의 메시지를 구현하는 것은 비교적 간단하지만(XML 편집), DDS를 통해 PX4의 내부 uORB 토픽과 통합하는 것은 다른 작업 흐름을 필요로 한다.98 MAVLink는 더 간단하고 가볍다. DDS/ROS 2는 더 강력하고 유연하며 복잡하다.


- **MAVROS**: MAVLink를 ROS 1에 연결하는 원래의 브리지이다. 자동 조종 장치의 MAVLink 메시지를 ROS 토픽으로 변환하고 그 반대도 수행하는 ROS 노드로 작동한다.96 이는 중요한 인프라지만 때로는 병목 현상이나 제거해야 할 복잡한 종속성으로 간주된다.50
- **Micro-XRCE-DDS 브리지**: PX4를 ROS 2에 연결하는 현대적인 접근 방식이다. PX4의 `uxrce_dds_client`와 컴패니언 컴퓨터의 `micro_xrce_dds_agent`로 구성된다.98 이를 통해 PX4의 내부 uORB 토픽을 ROS 2 DDS 네트워크에 직접 공유할 수 있어 MAVLink-MAVROS 변환 계층보다 더 빠르고 네이티브한 통합을 제공한다.96

MAVLink와 DDS/ROS 2의 관계는 단순한 경쟁이 아니라 두 가지 다른 패러다임이 수렴하는 것이다. MAVLink는 "에지"(저전력 자동 조종 장치 및 제한된 무선 링크)에 최적화되어 있는 반면, DDS/ROS 2는 "두뇌"(강력한 컴패니언 컴퓨터 및 더 넓은 로봇 공학 시스템)의 표준이다. MAVROS 및 XRCE-DDS와 같은 브리지의 존재는 이러한 수렴의 증거이다. 개발자에게는 두 가지 경로가 있다: GCS 및 간단한 주변 장치와의 높은 상호운용성을 위해 전통적인 MAVLink를 사용하거나, ROS 2 기반 컴패니언 컴퓨터와의 긴밀한 통합을 위해 고성능 XRCE-DDS 브리지를 사용하는 것이다. 이 선택은 근본적인 아키텍처적 절충을 반영한다.



PX4 및 MAVLink와 같은 핵심 프로젝트에 수천 개의 커밋이 이루어지는 등 생태계는 여전히 매우 활발하다.64 MAVSDK는 3.0 버전에 도달하여 상당한 성숙과 새로운 기능을 보여준다.64 FMUv6X-RT와 같은 새로운 비행 컨트롤러는 더 많은 처리 능력과 이더넷과 같은 인터페이스를 제공하여 미래의 프로토콜 요구에 영향을 미칠 것이다.64 Skydio와 같은 상용 제품은 RAS-A 상호운용성 프로파일과 같은 표준에 대한 MAVLink 준수를 개선하고 있으며, 이는 방위 및 상업 부문에서 MAVLink의 중요성을 보여준다.103 ArduPilot 개발자 회의에서는 유도 모드 시간 초과 개선, 비행 모드 로직 개선, 새로운 원격 측정 유형에 대한 바인딩 추가 등 지속적인 점진적 개선이 이루어지고 있다.104


2019년 발표에서는 강력한 보안(인증 및 암호화), IP 링크의 잠재력 활용, 공식적인 서비스 품질, 더 유연한 라우팅 등 주요 한계와 미래 작업 영역을 강조했다.105 2011년의 초기 로드맵 논의 106는 프로토콜 수준 ACK(명령 프로토콜이 됨) 및 확장된 메시지 형식(MAVLink 2가 됨)과 같은 많은 핵심 아이디어가 구현되었지만 기본 아키텍처는 안정적으로 유지되었음을 보여준다. 점진적인 개선을 넘어서는 명확하고 공개적인 미래 지향적 로드맵의 부재는 잠재적인 약점이다. 미래 방향의 대부분은 개발자 논의 및 MAVSDK와 같은 관련 프로젝트에서 유추된다.63


MAVLink의 강점은 단순성, 효율성, 그리고 방대한 설치 기반에 있다. 이는 드론-GCS 및 기본 주변 장치 통신을 위한 명실상부한 공용어이다.10 약점은 현대적이고 고도로 연결된 보안 시스템의 요구에 적응하는 데 어려움을 겪는다는 점이다. 고성능 온보드 프로세스 간 통신에서는 DDS/ROS 2에 의해 밀려나고 있다.

MAVLink의 미래 경로는 공존의 길로 보인다. 제한된 링크와 다양한 하드웨어 간의 광범위한 상호운용성을 보장하는 데 지배적인 프로토콜로 남을 가능성이 높다. 동시에 고급 시스템의 경우, 고대역폭, 저지연 통신을 처리하는 XRCE-DDS 브리지와 함께 사용될 것이다.

MAVLink 커뮤니티의 가장 중요한 과제는 기밀성(암호화)을 위한 해결책을 표준화하는 것이다. 공식적으로 승인된 접근 방식이 없다면, 벤더와 연구자들이 각자의 호환되지 않는 보안 계층을 구현함에 따라 생태계가 파편화될 위험이 있으며, 이는 프로토콜의 핵심 가치 제안인 상호운용성을 훼손하게 될 것이다. MAVLink는 자체적인 성공의 희생양이다. 광범위한 채택과 하위 호환성에 대한 집중은 네이티브 암호화 및 QoS를 갖춘 "MAVLink 3.0"과 같은 급진적인 진화를 매우 어렵게 만든다. 따라서 그 미래는 DDS와 같은 신흥 기술을 *대체*하는 것이 아니라 *전문화*하고 *공존*하는 것일 가능성이 높다. MAVLink의 역할은 *유일한* 프로토콜에서 *상호운용성* 프로토콜로 변화하고 있다. 그 지속적인 관련성은 특정 작업을 위한 가장 효율적이고 보편적인 표준으로 계속 남는 것에 달려 있다.


1. MAVLink 개발자 안내서, accessed July 16, 2025, https://mavlink.io/ko/
2. MAVLink - Wikipedia, accessed July 16, 2025, https://en.wikipedia.org/wiki/MAVLink
3. Journal Paper - Micro Air Vehicle Link (MAVLink) in a Nutshell: A Survey - CISTER, accessed July 16, 2025, https://cister-labs.pt/docs/micro_air_vehicle_link_(mavlink)_in_a_nutshell__a_survey/1551/view.pdf
4. MAVLink Developer Guide, accessed July 16, 2025, https://mavlink.io/en/
5. MAVLink Developer Guide - Hamish Willee, accessed July 16, 2025, https://hamishwillee.gitbooks.io/ham_mavdevguide/en/
6. Frequently Asked Questions (FAQ) - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/about/faq.html
7. Security Analysis of the Drone Communication Protocol: Fuzzing the MAVLink protocol 1 Introduction 2 Related Work - COSIC - KU Leuven, accessed July 16, 2025, https://cosicdatabase.esat.kuleuven.be/backend/publications/files/conferencepaper/2667
8. UAS 상호운용성 향상을 위한 STANAG 4586과 MAVLink 프로토콜 비교분석 및 개선방안 연구 A Study on the Ana, accessed July 16, 2025, https://jkimst.org/upload/pdf/KIMST-2020-23-6-618.pdf
9. mavlink - Dictor's blog, accessed July 16, 2025, https://kimdictor.kr/tags/mavlink/
10. MAVLink: The Lingua Franca of Drone Communication - Codemonk, accessed July 16, 2025, https://codemonk.io/blog/mavlink-the-lingua-franca-of-drone-communication/
11. MAVLink - 위키백과, 우리 모두의 백과사전, accessed July 16, 2025, https://ko.wikipedia.org/wiki/MAVLink
12. Hijacking drones with a MAVLink exploit - Blogs - diydrones, accessed July 16, 2025, https://diydrones.com/profiles/blogs/hijacking-quadcopters-with-a-mavlink-exploit
13. 자주 묻는 질문 - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/ko/about/faq.html
14. MAVLink Guide, accessed July 16, 2025, https://mavlink.io/
15. 드론 서버 개발 관련 유용한 링크들 - Go for IT - 티스토리, accessed July 16, 2025, [https://claremont.tistory.com/entry/%EB%93%9C%EB%A1%A0-%EC%84%9C%EB%B2%84-%EA%B0%9C%EB%B0%9C-%EA%B4%80%EB%A0%A8-%EC%9C%A0%EC%9A%A9%ED%95%9C-%EB%A7%81%ED%81%AC%EB%93%A4](https://claremont.tistory.com/entry/드론-서버-개발-관련-유용한-링크들)
16. Protocol Overview - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/about/overview.html
17. Command Protocol - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/services/command.html
18. MAVLink Versions, accessed July 16, 2025, https://mavlink.io/en/guide/mavlink_version.html
19. MAVLink 2 / ham_mavdevguide - Hamish Willee, accessed July 16, 2025, https://hamishwillee.gitbooks.io/ham_mavdevguide/kr/guide/mavlink_2.html
20. MAVLink Protocol | Drone Wolf Playbook, accessed July 16, 2025, [https://dronewolf.darkwolf.io/How%20They%20Work/Protocols/mavlink](https://dronewolf.darkwolf.io/How They Work/Protocols/mavlink)
21. CVE-2020-10282 Detail - NVD, accessed July 16, 2025, https://nvd.nist.gov/vuln/detail/CVE-2020-10282
22. Packet Serialization - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/guide/serialization.html
23. mavlink/doc/MAVLink2.md at master - GitHub, accessed July 16, 2025, https://github.com/Parrot-Developers/mavlink/blob/master/doc/MAVLink2.md
24. Changing mavlink version negotiation behaviour for protocol setting "2" - Copter 4.2, accessed July 16, 2025, https://discuss.ardupilot.org/t/changing-mavlink-version-negotiation-behaviour-for-protocol-setting-2/83074
25. Message Signing (Authentication) - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/guide/message_signing.html
26. MAVLINK Common Message Set (common.xml), accessed July 16, 2025, https://mavlink.io/en/messages/common.html
27. MAVLink Interface - Dev documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/dev/docs/mavlink-commands.html
28. Dialect: ArduPilotMega - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/messages/ardupilotmega.html
29. MAVLink Messaging | PX4 User Guide (v1.14), accessed July 16, 2025, https://docs.px4.io/v1.14/en/middleware/mavlink.html
30. Mavlink - Auterion, accessed July 16, 2025, https://auterion.zendesk.com/hc/en-us/articles/15728115389468-Mavlink
31. How to Define MAVLink Messages & Enums, accessed July 16, 2025, https://mavlink.io/en/guide/define_xml_element.html
32. MAVLink XML File Schema / Format, accessed July 16, 2025, https://mavlink.io/en/guide/xml_schema.html
33. [드론 만들기] MAVLink 설치하기 - 광필이 좋아요! - 티스토리, accessed July 16, 2025, https://kwangpil.tistory.com/80
34. PX4-Devguide/ko/middleware/mavlink.md at master - GitHub, accessed July 16, 2025, https://github.com/PX4/PX4-Devguide/blob/master/ko/middleware/mavlink.md
35. MAVLink - 오늘의AI위키, AI가 만드는 백과사전, accessed July 16, 2025, https://wiki.onul.works/w/MAVLink
36. 직렬화(Serialization) / ham_mavdevguide - Hamish Willee, accessed July 16, 2025, https://hamishwillee.gitbooks.io/ham_mavdevguide/kr/guide/serialization.html
37. Document field sorting and MAVLINK_CRC_EXTRA calculation / Issue #48 - GitHub, accessed July 16, 2025, https://github.com/mavlink/mavlink/issues/48
38. QGroundControl – Drone Control – Ground Control Station for Small Air – Land – Water Autonomous Unmanned Systems, accessed July 16, 2025, https://qgroundcontrol.com/
39. MAVLink 메시징 | PX4 오토파일럿 사용자 설명서 (v1.13), accessed July 16, 2025, https://docs.px4.io/v1.13/ko/middleware/mavlink.html
40. MAVLink Console (Analyze View) | QGC Guide (4.3), accessed July 16, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/analyze_view/mavlink_console.html
41. MAVLink Basics - Dev documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/dev/docs/mavlink-basics.html
42. Getting and Setting Parameters - Dev documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/dev/docs/mavlink-get-set-params.html
43. MAVSDK (main) | MAVSDK Guide, accessed July 16, 2025, https://mavsdk.mavlink.io/
44. QGroundControl을 이용한 패롯 아나피 드론 제어를 위한 MAVLink 프로토콜 수정 및 추가, accessed July 16, 2025, https://seo.goover.ai/report/202409/go-public-report-ko-3b539121-12e8-4ddc-823e-8c4679a8696e-0-0.html
45. MAVLink Inspector | QGC Guide (4.3), accessed July 16, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/analyze_view/mavlink_inspector.html
46. MAVLink Customisation | QGC Guide (master), accessed July 16, 2025, https://docs.qgroundcontrol.com/master/en/qgc-dev-guide/custom_build/mavlink.html
47. Sending custom MAVLink commands with QGC - Blue Robotics Community Forums, accessed July 16, 2025, https://discuss.bluerobotics.com/t/sending-custom-mavlink-commands-with-qgc/17070
48. Companion Computers - Dev documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/dev/docs/companion-computers.html
49. Companion Computer / GitBook - ArduSub, accessed July 16, 2025, https://www.ardusub.com/introduction/hardware-options/required-hardware/companion-computer.html
50. Sending Messages to Flight Controller Using PyMAVLink | by Sanjana Kumari | Medium, accessed July 16, 2025, https://medium.com/@sanjana_dev9/sending-messages-to-flight-controller-using-pymavlink-74ffa40d3d39
51. Decentralize Collision avoidance system for swarm drones using ESP32 - ArduCopter, accessed July 16, 2025, https://discuss.ardupilot.org/t/decentralize-collision-avoidance-system-for-swarm-drones-using-esp32/130844
52. MAVLink Routing in ArduPilot - Dev documentation, accessed July 16, 2025, https://ardupilot.org/dev/docs/mavlink-routing-in-ardupilot.html
53. Companion Computers - Better explain MAVLink routing and setup / Issue #574 - GitHub, accessed July 16, 2025, https://github.com/PX4/Devguide/issues/574
54. Using mavlink-router to route MAVLink streams over the network | 404warehouse, accessed July 16, 2025, https://404warehouse.net/2018/08/25/using-mavlink-router-to-route-mavlink-streams/
55. mavlink-router/mavlink-router: Route mavlink packets between endpoints - GitHub, accessed July 16, 2025, https://github.com/mavlink-router/mavlink-router
56. 6. Install and setup mavlink-router (connecting drone to ground station), accessed July 16, 2025, https://bellergy.com/6-install-and-setup-mavlink-router/
57. ArduPilot/pymavlink: python MAVLink interface and utilities - GitHub, accessed July 16, 2025, https://github.com/ArduPilot/pymavlink
58. Pymavlink / GitBook - ArduSub, accessed July 16, 2025, https://www.ardusub.com/developers/pymavlink.html
59. Pymavlink / GitBook - GitHub Pages, accessed July 16, 2025, https://williangalvani.github.io/ardusub-gitbook/developers/pymavlink.html
60. pymavlink/examples/magtest.py at master - GitHub, accessed July 16, 2025, https://github.com/ArduPilot/pymavlink/blob/master/examples/magtest.py
61. C++ QuickStart | MAVSDK Guide, accessed July 16, 2025, https://mavsdk.mavlink.io/main/en/cpp/quickstart.html
62. Python QuickStart | MAVSDK Guide, accessed July 16, 2025, https://mavsdk.mavlink.io/main/en/python/quickstart.html
63. Controlling MAVLink drones with MAVSDK - Jonas Vautherin - PX4 Developer Summit 2019, accessed July 16, 2025, https://www.youtube.com/watch?v=GXt-eXJ9vfg
64. The 2024 Year in Review - Dronecode Foundation, accessed July 16, 2025, https://dronecode.org/the-2024-year-in-review/
65. MAVLINK Encryption and other Security related aspects of Ardupilot, accessed July 16, 2025, https://discuss.ardupilot.org/t/mavlink-encryption-and-other-security-related-aspects-of-ardupilot/15857
66. UAS의 C2용 MAVLink 프로토콜 취약점 분석 - cuashub.com, accessed July 16, 2025, [https://cuashub.com/ko/%EC%BD%98%ED%85%90%EC%B8%A0/uas%EC%9D%98-c2%EC%97%90-%EB%8C%80%ED%95%9C-mavlink-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C%EC%9D%98-%EC%B7%A8%EC%95%BD%EC%84%B1-%EB%B6%84%EC%84%9D/](https://cuashub.com/ko/콘텐츠/uas의-c2에-대한-mavlink-프로토콜의-취약성-분석/)
67. Vulnerability Analysis of the MAVLink Protocol for Command and Control of Unmanned Aircraft - AFIT Scholar, accessed July 16, 2025, https://scholar.afit.edu/context/etd/article/1613/viewcontent/AFIT_ENG_14_M_50_Marty_ADA598977.pdf
68. Vulnerability Analysis of the MAVLink Protocol for C2 of UAS - cuashub.com, accessed July 16, 2025, https://cuashub.com/en/content/vulnerability-analysis-of-the-mavlink-protocol-for-c2-of-uas/
69. MUVIDS: False MAVLink Injection Attack Detection in Communication for Unmanned Vehicles, accessed July 16, 2025, https://www.ndss-symposium.org/wp-content/uploads/autosec2021_23036_paper.pdf
70. [논문 리뷰] Enforcing MAVLink Safety & Security Properties Via Refined Multiparty Session Types, accessed July 16, 2025, https://www.themoonlight.io/ko/review/enforcing-mavlink-safety-security-properties-via-refined-multiparty-session-types
71. Enforcing MAVLink Safety & Security Properties Via Refined Multiparty Session Types, accessed July 16, 2025, https://arxiv.org/html/2501.18874v2
72. C Message Signing (mavgen) - Hamish Willee, accessed July 16, 2025, https://hamishwillee.gitbooks.io/ham_mavdevguide/content/v/ext_param/en/mavgen_c/message_signing_c.html
73. Packet Signing - MAVProxy documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/mavproxy/docs/modules/signing.html
74. Message Signing (Pymavlink) - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/mavgen_python/message_signing.html
75. MAVLink2 Signing - Copter documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/copter/docs/common-MAVLink2-signing.html
76. (PDF) MAVLink Protocol: A Survey of Security Threats and Countermeasures, accessed July 16, 2025, https://www.researchgate.net/publication/385670293_MAVLink_Protocol_A_Survey_of_Security_Threats_and_Countermeasures
77. MAVSec: Securing the MAVLink Protocol for Ardupilot/PX4 Unmanned Aerial Systems - CISTER, accessed July 16, 2025, http://www.cister.isep.ipp.pt/docs/mavsec__securing_the_mavlink_protocol_for_ardupilot_px4_unmanned_aerial_systems/1504/view.pdf
78. A Novel Cipher for Enhancing MAVLink Security: Design, Security Analysis, and Performance Evaluation Using a Drone Testbed - arXiv, accessed July 16, 2025, https://arxiv.org/html/2504.20626v1
79. Enhancing MAVLink Security: Implementation and Performance Evaluation of Encryption on a Drone Testbed - IEEE Computer Society, accessed July 16, 2025, https://www.computer.org/csdl/proceedings-article/icoin/2025/10993000/26JHPsbABgY
80. Vulnerability Analysis of the MAVLink Protocol for Command and Control of Unmanned Aircraft - CORE, accessed July 16, 2025, https://core.ac.uk/download/pdf/277527627.pdf
81. Camera Protocol v2 - MAVLink Guide, accessed July 16, 2025, https://mavlink.io/en/services/camera.html
82. MAVLINK interface for UAV cameras, accessed July 16, 2025, https://www.drone-thermal-camera.com/mavlink-interface-for-uav-cameras/
83. MAVLink Camera Manager Service - GitHub, accessed July 16, 2025, https://github.com/mavlink/mavlink-camera-manager
84. Message passing interface for swarm communication via MAVLink. - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/figure/Message-passing-interface-for-swarm-communication-via-MAVLink_fig9_384838663
85. Swarming - Mission Planner documentation - ArduPilot, accessed July 16, 2025, https://ardupilot.org/planner/docs/swarming.html
86. alireza787b/mavsdk_drone_show: All in one Drone Show and Smart Swarm Solutin for PX4, accessed July 16, 2025, https://github.com/alireza787b/mavsdk_drone_show
87. Deployable Swarms: Decentralized Multi-Agent Path Planning and Control | NIMBUS Lab, accessed July 16, 2025, https://nimbus.unl.edu/deployable-swarms-decentralized-multi-agent-path-planning-and-control/
88. Conducting Extended BVLOS Operations in challenging terrain leveraging path and link diversity for highly reliable C2 - FAA, accessed July 16, 2025, [https://www.faa.gov/uas/programs_partnerships/BAA/BAA004-uAvionix%E2%80%93Conducting-Extended-BVLOS-Operations-in-challenging-terrain.pdf](https://www.faa.gov/uas/programs_partnerships/BAA/BAA004-uAvionix–Conducting-Extended-BVLOS-Operations-in-challenging-terrain.pdf)
89. University of Southern Denmark Towards SORA-Compliant BVLOS Communication Andersen, Frederik Mazur; Terkildsen, Kristian Husum;, accessed July 16, 2025, https://portal.findresearcher.sdu.dk/files/184743985/AcroBrwEx_Towards_SORA_compliant_BVLOS_communication_final_1_.pdf_ADW5B1E.tmp.pdf
90. Drone Communication Systems - Fly Eye, accessed July 16, 2025, https://www.flyeye.io/drone-technology-communication/
91. MAVLink Messaging | PX4 Guide (main), accessed July 16, 2025, https://docs.px4.io/main/en/mavlink/index.html
92. Enabling Technologies for the Navigation and Communication of UAS Operating in the Context of BVLOS - MDPI, accessed July 16, 2025, https://www.mdpi.com/2079-9292/13/2/340
93. Ultra-Reliable & Secure BVLOS Communications Technology for Mission-Critical Drones & Robotics - Defense Advancement, accessed July 16, 2025, https://www.defenseadvancement.com/company/elsight/
94. What is Mavlink? And what can it do for me? - Grey Arrows Drone Club, accessed July 16, 2025, https://greyarro.ws/t/what-is-mavlink-and-what-can-it-do-for-me/19451
95. DDS Standard - Data Distribution Service for Complex Systems - RTI, accessed July 16, 2025, https://www.rti.com/products/dds-standard
96. Transitioning from ROS 1 (ArduPilot) to ROS 2 (PX4): A Complete Mental Model | by Sidharth Mohan Nair | Medium, accessed July 16, 2025, https://medium.com/@sidharthmohannair/transitioning-from-ros-1-ardupilot-to-ros-2-px4-a-complete-mental-model-98c54758f569
97. Custom flight modes using PX4 and ROS2 - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=boLXuAxdF0U
98. Latency Reduction and Packet Synchronization in Low-Resource Devices Connected by DDS Networks in Autonomous UAVs - MDPI, accessed July 16, 2025, https://www.mdpi.com/1424-8220/23/22/9269
99. Basic understanding of MAVROS, Quadrocopter, UAV - Robotics Stack Exchange, accessed July 16, 2025, https://robotics.stackexchange.com/questions/73377/basic-understanding-of-mavros-quadrocopter-uav
100. Mavlink、DDS and uORB - ArduCopter - ArduPilot Discourse, accessed July 16, 2025, https://discuss.ardupilot.org/t/mavlink-dds-and-uorb/128456
101. Working with MAVlink messages in ROS without Mavros | by James - Medium, accessed July 16, 2025, https://james1345.medium.com/working-with-mavlink-messages-in-ros-without-mavros-88a055973fdf
102. uXRCE-DDS (PX4-ROS 2/DDS Bridge) | PX4 Guide (main), accessed July 16, 2025, https://docs.px4.io/main/en/middleware/uxrce_dds.html
103. Skydio X10D ATO: v37.1.182 - 30 October 2024, accessed July 16, 2025, https://support.skydio.com/hc/en-us/articles/30483032985499-Skydio-X10D-ATO-v37-1-182-30-October-2024
104. Dev Call Jan 6, 2025 - Development Team - ArduPilot Discourse, accessed July 16, 2025, https://discuss.ardupilot.org/t/dev-call-jan-6-2025/128445
105. Roadmap Discussion, accessed July 16, 2025, https://px4.io/wp-content/uploads/2019/06/MAVLink-Considerations-for-future-iterations.pdf
106. MAVLink: Flight Control Language vs Transport Layer (or both with clear distinctions), accessed July 16, 2025, https://groups.google.com/g/mavlink/c/O0YpUmBScj0



